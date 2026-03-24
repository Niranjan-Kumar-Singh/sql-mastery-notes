# 🔭 Topic 15.4: Transaction Isolation Levels

Isolation is the "I" in ACID. When thousands of concurrent users all read and write data simultaneously, how "sheltered" is your transaction from what others are doing? Isolation Levels let you configure this trade-off: **more isolation = safer data, but more locks and slower performance.**

---

## 1. The 3 Concurrency Problems (What We're Protecting Against)

Before understanding isolation levels, understand the problems they solve:

### Problem 1: Dirty Read
User A reads data that User B has **modified but not yet committed**. If User B rolls back, User A has read "fake" data that technically never existed.

```
B: UPDATE Balance = 0 (not committed yet)
A: SELECT Balance → sees 0  ← DIRTY READ
B: ROLLBACK (Balance goes back to 1000)
A: Made a decision based on a balance of 0 that never existed!
```

### Problem 2: Non-Repeatable Read
User A reads the same row twice and gets **different values** because User B committed a change between the two reads.

```
A: SELECT Balance → 1000
B: UPDATE Balance = 500; COMMIT
A: SELECT Balance again → 500  ← Different! Non-Repeatable Read
```

### Problem 3: Phantom Read
User A runs a query twice and gets a **different number of rows** because User B inserted or deleted rows between the two reads.

```
A: SELECT * FROM Orders WHERE Amount > 500 → 10 rows
B: INSERT INTO Orders (Amount=800); COMMIT
A: Same SELECT again → 11 rows  ← An extra "phantom" row appeared!
```

---

## 2. The 4 Isolation Levels

MySQL InnoDB supports all four standard SQL isolation levels:

| Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Performance |
|---|---|---|---|---|
| **READ UNCOMMITTED** | ❌ Possible | ❌ Possible | ❌ Possible | 🚀 Fastest |
| **READ COMMITTED** | ✅ Prevented | ❌ Possible | ❌ Possible | ⚡ Fast |
| **REPEATABLE READ** | ✅ Prevented | ✅ Prevented | ✅ Prevented* | 🔶 Moderate |
| **SERIALIZABLE** | ✅ Prevented | ✅ Prevented | ✅ Prevented | 🐢 Slowest |

*MySQL's REPEATABLE READ also prevents Phantom Reads (unique to InnoDB's MVCC implementation — better than the SQL standard requires)

**Default in MySQL: REPEATABLE READ** — the best balance of safety and performance.

---

## 3. How It Works — MVCC (PRO LEVEL)

How does MySQL prevent Dirty Reads without locking every row being read? **Multi-Version Concurrency Control (MVCC).**

When a transaction starts:
1. MySQL takes an invisible "**Snapshot**" of the database state at that exact moment.
2. All reads within the transaction see data from **that snapshot**, regardless of what other sessions commit.
3. Other sessions read their own snapshots simultaneously.
4. Writes still use row-level locks — but reads never block writes, and writes never block reads.

**This is why MySQL is fast:** Reads and writes don't block each other. MVCC ensures consistency without the performance penalty of locking every read.

---

## 4. Each Level Explained Simply

### READ UNCOMMITTED ("Danger Mode")
Reads data even from uncommitted transactions. **Never use in production.**
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- Scenario: Bank balance shows ₹0 because another session is mid-transfer (not committed).
-- You might deny a customer credit based on a temporary phantom ₹0 balance.
```

### READ COMMITTED ("Modern Standard")
Only reads committed data. Prevents dirty reads. Widely used in OLTP systems.
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Scenario: Each SELECT in your transaction shows the latest committed data.
-- Good for: Most business applications, reporting dashboards.
```

### REPEATABLE READ ("MySQL Default")
Your transaction always sees the same data — frozen at the moment the transaction began. Prevents dirty reads AND non-repeatable reads.
```sql
-- Default — usually no need to set explicitly
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- Scenario: A financial report transaction sees consistent totals even as new transactions are posted.
```

### SERIALIZABLE ("Fort Knox Mode")
Every transaction runs as if it's the only one in the system. Completely safe but very slow — places shared locks on everything read.
```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Scenario: A government revenue calculation that absolutely cannot have any variation.
-- Tradeoff: Application throughput drops significantly under load.
```

---

## 5. Syntax / Implementation

```sql
-- Check current isolation level
SELECT @@transaction_isolation;  -- MySQL 8.0+
SELECT @@tx_isolation;           -- MySQL 5.7 (old syntax)

-- Change for current session only
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Change globally (all new connections)
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Change for next transaction only
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
-- ... this transaction runs under SERIALIZABLE ...
COMMIT;
-- Next transaction returns to session default
```

---

## 6. Real-Life Examples

**Example 1 — Banking: Use REPEATABLE READ (Default)**
```sql
-- Generating a bank statement for the month
-- Must see consistent balances throughout the entire report generation
START TRANSACTION;  -- Takes a snapshot at this moment
SELECT SUM(Amount) FROM Transactions WHERE Month(Date) = 3;
SELECT AVG(Balance) FROM Accounts;
COMMIT;
-- Even if other transactions commit during this, our report is from the snapshot 
```

**Example 2 — E-commerce: READ COMMITTED is Fine**
```sql
-- Product availability check - always want to see latest committed stock
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT Stock FROM Products WHERE Product_ID = 45;
-- If another session just committed a stock update, we see it immediately (good!)
```

---

## 7. Common Mistakes

- **Using SERIALIZABLE everywhere "to be safe":** This destroys database throughput. Web applications under SERIALIZABLE cannot handle concurrent users. Use REPEATABLE READ (default) for safety and performance.
- **Assuming READ UNCOMMITTED is okay for "read-only" use:** Even read-only operations can make wrong decisions based on dirty data. Never use READ UNCOMMITTED for business logic.
- **Not understanding that MySQL's REPEATABLE READ also prevents Phantoms:** MySQL's MVCC implementation prevents phantom reads at REPEATABLE READ level — this is better than the ANSI SQL standard requires. Don't upgrade to SERIALIZABLE just to prevent phantoms.

---

## 8. Tips & Best Practices

- **Stick with REPEATABLE READ (default)** for 99% of applications — MySQL chose it as default for good reason.
- **Use READ COMMITTED** if your application is "read-heavy" and doesn't need consistent snapshots (e.g., live dashboards).
- **Use SERIALIZABLE only for critical financial calculations** that require absolute consistency, and be prepared to limit concurrent access.
- **Never use READ UNCOMMITTED in production** — there is no legitimate use case for reading uncommitted data in a business system.

---

## 9. Mini Practice Tasks

- **Task 1:** What is a "Dirty Read"? Write a timeline showing how it can occur and what harm it could cause in a bank balance scenario.
- **Task 2:** MySQL's default isolation level is REPEATABLE READ. What problem does this specifically prevent compared to READ COMMITTED? Give a concrete example.
- **Task 3:** A team argues about using SERIALIZABLE for their e-commerce site "to prevent any data issues." Why is this a bad idea, and what should they use instead?

---
