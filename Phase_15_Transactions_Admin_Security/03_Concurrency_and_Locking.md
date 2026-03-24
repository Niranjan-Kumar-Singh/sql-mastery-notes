# 🔒 Topic 15.3: Concurrency & Locking Concepts

When thousands of users are using your database simultaneously, things get complicated. Two users trying to buy the **last concert ticket** at the same time, two bank transfers happening on the same account simultaneously — without proper locking, data gets corrupted. Locking is how MySQL maintains order.

---

## 1. The Problem Without Locking — The Lost Update

Here's what happens without locking in a bank system:

```
Time 1ms: User A reads Balance: ₹10,000
Time 2ms: User B reads Balance: ₹10,000
Time 3ms: User A withdraws ₹3,000 → writes ₹7,000
Time 4ms: User B withdraws ₹2,000 → writes ₹8,000  ← Overwrites User A's update!
Final Balance: ₹8,000  ← Should be ₹5,000! ₹3,000 vanished!
```

This is called a **Lost Update** — one of the most catastrophic concurrency bugs.

---

## 2. The Two Types of Locks

| Lock Type | Symbol | Also Called | Who Can Read? | Who Can Write? |
|---|---|---|---|---|
| **Shared Lock** | S-Lock | Read Lock | Anyone | Nobody (until S released) |
| **Exclusive Lock** | X-Lock | Write Lock | Nobody | Only the lock owner |

**Lock Compatibility Matrix:**
- S + S = Compatible (two readers can coexist) ✅
- S + X = Incompatible (can't read while someone writes) ❌
- X + X = Incompatible (can't have two writers) ❌

---

## 3. Lock Granularity — What Gets Locked?

| Granularity | What it Locks | Best For |
|---|---|---|
| **Table Lock** | Entire table | Bulk imports, backups (MyISAM default) |
| **Row Lock** | Only the specific row(s) being modified | High-concurrency web apps (InnoDB default) |
| **Gap Lock** | A range between rows (prevents phantom rows) | Repeatable Read isolation level |

**InnoDB uses Row-Level locking by default** — this is why MySQL can handle thousands of concurrent users modifying different rows without blocking each other.

---

## 4. How It Works — The Lock Lifecycle (PRO LEVEL)

When MySQL executes an `UPDATE`:
1. MySQL requests an **Exclusive (X) Lock** on the target row(s).
2. If another session holds a **Shared or Exclusive lock** on that row → current session **waits** (enters a sleep state).
3. MySQL periodically checks if the waiting thread has exceeded `innodb_lock_wait_timeout` (default: 50 seconds). If so, it throws `ERROR 1205: Lock wait timeout exceeded`.
4. Once the conflicting lock is released → waiting session gets the X-lock, performs the UPDATE, and releases it at `COMMIT`.

---

## 5. Deadlocks — When Two Sessions Block Each Other

A **Deadlock** occurs when:
- Session A holds Lock on Row 1, waiting for Row 2
- Session B holds Lock on Row 2, waiting for Row 1
- Both sessions wait forever → deadlock!

**MySQL's Deadlock Resolution:** MySQL automatically detects this cycle and kills the transaction with the smallest undo log (least work done), rolling it back and releasing its locks. The other session can then proceed.

```sql
-- Check recent deadlock info
SHOW ENGINE INNODB STATUS;
-- Look for "LATEST DETECTED DEADLOCK" section
```

---

## 6. Implementation — Explicitly Locking Rows

```sql
-- Pessimistic Lock: Lock during the SELECT phase (you know you'll update it soon)
START TRANSACTION;
SELECT * FROM Accounts WHERE Account_ID = 101 FOR UPDATE;
-- Now no other session can read or write Account 101 until you COMMIT or ROLLBACK
UPDATE Accounts SET Balance = Balance - 5000 WHERE Account_ID = 101;
COMMIT;  -- Lock released here

-- Shared Lock: Allow others to read, but block writes (useful for generating reports)
START TRANSACTION;
SELECT * FROM Inventory WHERE Product_ID = 55 LOCK IN SHARE MODE;
-- Others can SELECT, but no UPDATE/DELETE until you COMMIT
COMMIT;
```

---

## 7. Real-Life Examples

**Example 1 — Last Ticket Problem (Concert Booking):**
```sql
START TRANSACTION;

-- Lock the ticket row BEFORE checking availability
SELECT Tickets_Available FROM Events WHERE Event_ID = 200 FOR UPDATE;
-- At this point, every other session trying to book the same event waits!

-- Check and book
UPDATE Events SET Tickets_Available = Tickets_Available - 1 WHERE Event_ID = 200 AND Tickets_Available > 0;
INSERT INTO Bookings (User_ID, Event_ID) VALUES (501, 200);

COMMIT;
-- Only after COMMIT does the next waiting session get to check availability!
```

**Example 2 — Avoiding Deadlocks (Always Lock in the Same Order):**
```sql
-- BAD: Session A locks Account 1 then Account 2. Session B locks Account 2 then Account 1. → Deadlock
-- GOOD: Always lock accounts in low-to-high ID order to prevent circular waits:
START TRANSACTION;
SELECT * FROM Accounts WHERE Account_ID IN (101, 202) ORDER BY Account_ID FOR UPDATE;
-- Both accounts locked in same order by everyone → no deadlock possible
UPDATE Accounts SET Balance = Balance - 1000 WHERE Account_ID = 101;
UPDATE Accounts SET Balance = Balance + 1000 WHERE Account_ID = 202;
COMMIT;
```

---

## 8. Common Mistakes

- **Leaving long-running transactions open:** Every lock held longer than necessary blocks other sessions. Open → Work → Commit as fast as possible.
- **Not handling lock wait timeouts in application code:** When `innodb_lock_wait_timeout` triggers, MySQL throws an error. Application code MUST catch this error and retry or report it gracefully instead of crashing.
- **Table-level locking in InnoDB:** Commands like `LOCK TABLES Employees WRITE` bypass row-level locking and lock the entire table. Appropriate for maintenance, catastrophic in production applications.

---

## 9. Tips & Best Practices

- **Keep transactions short:** The best defence against lock contention is minimising the time between `START TRANSACTION` and `COMMIT`.
- **Use `FOR UPDATE` for read-then-write patterns:** If your app reads a value and then decides whether to update it, use `SELECT ... FOR UPDATE` to lock the row during the read — otherwise another session might change it between your read and your write.
- **Monitor lock waits:** `SHOW ENGINE INNODB STATUS` and `information_schema.INNODB_TRX` table show active transactions and their lock waits.

---

## 10. Mini Practice Tasks

- **Task 1:** Explain what a "Lost Update" is and show a timeline of events that causes it. How does locking prevent it?
- **Task 2:** What is a Deadlock? Write a scenario involving two sessions and two rows that results in a deadlock. How does MySQL resolve it?
- **Task 3:** When would you use `SELECT ... FOR UPDATE` vs `SELECT ... LOCK IN SHARE MODE`? Give one real-world example for each.

---
