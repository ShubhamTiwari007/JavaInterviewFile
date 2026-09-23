# Kafka Consumer + Database: Preventing Duplicate Transactions

## 1. The problem

Suppose a Kafka consumer receives a transaction:

```text
Kafka topic
    |
    v
Consumer
    |
    v
Debit ₹100 from account
    |
    v
Commit Kafka offset
```

The dangerous part is that the database operation and Kafka offset commit are **two separate operations**.

For example:

1. Consumer reads message `TXN-123`.
2. Consumer debits ₹100 from the database.
3. Consumer crashes **before** committing the Kafka offset.
4. Kafka still considers `TXN-123` uncommitted.
5. Consumer restarts one hour later.
6. Kafka delivers `TXN-123` again.
7. Without protection, the consumer debits another ₹100.

Result:

```text
Expected: ₹100 debited
Actual:   ₹200 debited
```

Kafka's normal consumer offset mechanism provides **at-least-once delivery**, so duplicate processing is expected to be possible.

---

## 2. The primary solution: idempotent database processing

Give every business transaction a unique identifier, for example:

```text
transaction_id = TXN-123
```

Store that ID in the database and enforce a **UNIQUE constraint**.

Example:

```sql
CREATE TABLE transactions (
    transaction_id VARCHAR(100) PRIMARY KEY,
    account_id     VARCHAR(100) NOT NULL,
    amount         DECIMAL(18,2) NOT NULL,
    status         VARCHAR(20) NOT NULL,
    created_at     TIMESTAMP NOT NULL
);
```

The important part is:

```text
transaction_id = PRIMARY KEY / UNIQUE
```

This makes the transaction processing idempotent.

---

## 3. Recommended processing flow

The consumer should process a Kafka message approximately like this:

```text
                Kafka
                  |
                  v
          Read transaction
                  |
                  v
       Is transaction_id already
             processed?
            /                    YES           NO
           |             |
           v             v
       Skip debit     Start DB transaction
           |             |
           |             v
           |       Insert transaction
           |             |
           |             v
           |       Debit account
           |             |
           |             v
           |        Commit DB TX
           |             |
           +-------> Commit Kafka offset
```

The key point is that **the database transaction must atomically record that the transaction was processed and perform the financial operation**.

---

## 4. Why the database transaction matters

Do not do this:

```text
1. Check transaction ID
2. If not found, debit account
3. Insert transaction ID
```

There is a race/crash window between these operations.

Instead, use a database transaction:

```text
BEGIN;

INSERT transaction_id = TXN-123;

UPDATE account
SET balance = balance - 100
WHERE account_id = 'A1';

COMMIT;
```

If the insert fails because `TXN-123` already exists, do not perform the debit.

Conceptually:

```pseudo
BEGIN DATABASE TRANSACTION

insert transaction TXN-123

if transaction already exists:
    ROLLBACK
    commit Kafka offset
    return

debit account by ₹100

COMMIT DATABASE TRANSACTION

commit Kafka offset
```

The exact SQL/error-handling mechanism depends on the database.

---

## 5. What happens when the consumer crashes?

### Scenario A: Crash before the DB commit

```text
Kafka message TXN-123
       |
       v
BEGIN DB TX
       |
       v
Debit ₹100
       |
       X  Consumer crashes
       |
       v
DB transaction rolls back
```

The Kafka offset was not committed, so Kafka redelivers `TXN-123`.

The message is processed again:

```text
TXN-123
   |
   v
DB transaction
   |
   v
Debit ₹100
   |
   v
COMMIT
   |
   v
Commit Kafka offset
```

Only one debit occurs.

---

### Scenario B: DB commit succeeds, but consumer crashes before Kafka offset commit

This is the important case.

```text
Kafka: TXN-123
       |
       v
DB transaction
       |
       v
Debit ₹100
       |
       v
DB COMMIT SUCCESS
       |
       X Consumer crashes
       |
       v
Kafka offset NOT committed
```

After restart:

```text
Kafka sends TXN-123 again
       |
       v
Consumer checks transaction_id
       |
       v
TXN-123 already exists
       |
       v
DO NOT debit again
       |
       v
Commit Kafka offset
```

Therefore:

```text
Debit = ₹100
```

not:

```text
Debit = ₹200
```

This is why the **unique transaction ID + atomic DB transaction** is critical.

---

## 6. Example database design

A common pattern is a transaction/processed-events table:

```sql
CREATE TABLE processed_transactions (
    transaction_id VARCHAR(100) PRIMARY KEY,
    account_id     VARCHAR(100) NOT NULL,
    amount         DECIMAL(18,2) NOT NULL,
    status         VARCHAR(20) NOT NULL,
    created_at     TIMESTAMP NOT NULL
);
```

Then the business operation and the record of processing happen in the same database transaction.

For example:

```sql
BEGIN;

INSERT INTO processed_transactions
    (transaction_id, account_id, amount, status, created_at)
VALUES
    ('TXN-123', 'A1', 100.00, 'SUCCESS', CURRENT_TIMESTAMP);

UPDATE accounts
SET balance = balance - 100.00
WHERE account_id = 'A1';

COMMIT;
```

If `TXN-123` already exists, the consumer recognizes it as a duplicate and does not execute the debit.

---

## 7. Do not rely only on the Kafka offset

A common misconception is:

> "If I commit the Kafka offset after processing, duplicates cannot happen."

That is not sufficient.

There is always a failure window:

```text
DB operation succeeds
        |
        X
Consumer crashes
        |
Kafka offset not committed
        |
        v
Message is delivered again
```

Therefore, Kafka offset management alone cannot guarantee that an external database operation happens exactly once.

For financial operations, design the **business operation itself to be idempotent**.

---

## 8. Idempotency key

The ideal message contains a stable unique business identifier:

```json
{
  "transactionId": "TXN-123",
  "accountId": "A1",
  "amount": 100.00,
  "operation": "DEBIT"
}
```

`transactionId` should be generated by the system creating the transaction and remain unchanged if Kafka redelivers the message.

Do not generate a new random ID every time the consumer receives the message.

Bad:

```text
Kafka message TXN-123
   |
   +--> Consumer generates UUID-1 --> debit
   |
   +--> Redelivery generates UUID-2 --> debit again
```

Good:

```text
Kafka message TXN-123
   |
   +--> First delivery  --> TXN-123
   |
   +--> Redelivery      --> TXN-123
```

---

## 9. Account balance protection

For financial systems, also consider protecting the account update itself.

For example:

```sql
UPDATE accounts
SET balance = balance - 100.00
WHERE account_id = 'A1'
  AND balance >= 100.00;
```

Then verify that exactly one row was updated.

This protects against insufficient balance, but it does **not** replace idempotency.

You need both:

```text
Idempotency
    +
Correct transactional database update
```

---

## 10. Kafka consumer configuration

Typical consumer behavior is:

```text
enable.auto.commit = false
```

Then commit the Kafka offset only after the database transaction succeeds.

Conceptually:

```pseudo
record = kafka.poll()

try:
    BEGIN DB TRANSACTION

    if transaction_id already processed:
        ROLLBACK / no-op
    else:
        record transaction_id
        perform debit

        COMMIT DB TRANSACTION

    kafka.commitOffset()

catch error:
    ROLLBACK DB TRANSACTION
    do NOT commit Kafka offset
```

If processing fails, Kafka can redeliver the message.

---

## 11. What about Kafka Exactly-Once Semantics?

Kafka supports transactions and exactly-once semantics (EOS) for certain Kafka-to-Kafka processing patterns.

For example:

```text
Kafka topic A
     |
     v
Kafka transaction
     |
     v
Kafka topic B
```

Kafka can coordinate the consumed offsets and produced Kafka records.

However, your scenario is:

```text
Kafka
  |
  v
Consumer
  |
  v
External Database
```

Kafka cannot automatically make an arbitrary external database transaction atomic with the Kafka offset commit.

Therefore, for a database-backed financial transaction, **idempotency at the database/business layer remains essential**.

---

## 12. Advanced pattern: Transactional Outbox

If your application performs a database change and then needs to publish another Kafka event, consider the **Transactional Outbox Pattern**.

Instead of:

```text
DB update
   |
   v
Kafka publish
```

use:

```text
             DB transaction
             /                       v             v
     Business data    Outbox event
                           |
                           v
                    Outbox publisher
                           |
                           v
                         Kafka
```

The business update and outbox record are committed atomically in the database.

A separate publisher reads the outbox and publishes to Kafka.

This solves a different but related dual-write problem.

---

## 13. Practical architecture

For a financial debit service, a robust design looks like:

```text
                         Kafka
                           |
                           v
                    Consumer Service
                           |
                           v
                 transactionId = TXN-123
                           |
                           v
                +----------------------+
                | Database Transaction |
                |                      |
                | 1. Idempotency check |
                | 2. Record TXN-123    |
                | 3. Debit account     |
                | 4. Commit             |
                +----------------------+
                           |
                           v
                  Commit Kafka Offset
```

The ordering is important:

```text
DB transaction succeeds
        ↓
Kafka offset commit
```

Not:

```text
Kafka offset commit
        ↓
DB transaction
```

If the offset is committed first and the application crashes before the database operation, the message can be lost from the consumer's perspective.

---

## 14. The key failure scenarios

| Failure point | DB result | Kafka offset | What happens |
|---|---|---|---|
| Before DB operation | Nothing | Not committed | Message redelivered |
| During DB transaction | Rolled back | Not committed | Message redelivered |
| DB committed, then crash | Debit exists | Not committed | Message redelivered, idempotency prevents second debit |
| DB committed + offset committed | Debit exists | Committed | Normal completion |
| Offset committed before DB operation | Possibly no debit | Committed | **Potential message loss** |

---

## 15. Recommended rule

For a transactional/financial Kafka consumer, follow this rule:

> **Perform the business operation and record the idempotency key atomically in the database. Commit the Kafka offset only after the database transaction succeeds.**

In simplified form:

```text
Kafka message
     |
     v
Read transactionId
     |
     v
DB transaction
     |
     +--> Already processed?
     |       |
     |      YES ---> No-op
     |       |
     |      NO
     |       |
     |       +--> Record transactionId
     |       |
     |       +--> Debit amount
     |
     v
DB COMMIT
     |
     v
Kafka OFFSET COMMIT
```

### Bottom line

Do not try to guarantee that Kafka delivers the message only once.

Instead, assume:

```text
Kafka may deliver the message more than once.
```

and make the financial operation safe to retry:

```text
Unique transaction ID
        +
Database unique constraint
        +
Atomic DB transaction
        +
Commit Kafka offset after DB commit
```

This gives you **effectively-once business processing** even though the Kafka consumer itself may receive the same message multiple times.
