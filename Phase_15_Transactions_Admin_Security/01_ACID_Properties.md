# 🏗️ Topic 15.1: ACID Properties

When you send ₹10,000 from your bank account, two things must happen simultaneously: your balance decreases by ₹10,000, and the recipient's balance increases by ₹10,000. If the server crashes between these two steps, where does the money go? **ACID Properties** are the mathematical guarantees that prevent this nightmare.

---

## 1. Definition

**ACID** is a set of four properties that every reliable database transaction must satisfy. These properties are what separate a production-grade database (MySQL, PostgreSQL, Oracle) from a simple file or spreadsheet.

---

## 2. The Four ACID Properties

| Property | One-Line Description | Technical Meaning |
|---|---|---|
| **A — Atomicity** | "All or Nothing" | A transaction is treated as a single unit. Either ALL operations succeed, or NONE of them are applied. No partial completion. |
| **C — Consistency** | "Valid Before and After" | Every transaction brings the database from one valid state to another. All constraints (PRIMARY KEYs, FOREIGN KEYs, CHECK, NOT NULL) are always enforced. |
| **I — Isolation** | "Independent Transactions" | Concurrent transactions are invisible to each other while in progress. The result is as if they ran sequentially. |
| **D — Durability** | "Permanent Once Committed" | Once a `COMMIT` is executed, the data is permanently saved — even if the server loses power one millisecond later. |

---

## 3. Each Property — Deep Dive

### A — Atomicity
```sql
START TRANSACTION;
UPDATE Accounts SET Balance = Balance - 10000 WHERE Account_ID = 1;  -- Step 1
UPDATE Accounts SET Balance = Balance + 10000 WHERE Account_ID = 2;  -- Step 2
COMMIT;
-- If Step 2 fails (e.g., Account 2 doesn't exist), ROLLBACK reverses Step 1 too.
-- The ₹10,000 doesn't vanish — it's as if neither step ever ran.
```

### C — Consistency
```sql
-- Example: A FOREIGN KEY constraint enforces Consistency
ALTER TABLE Orders ADD CONSTRAINT fk_cust 
FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID);

-- This INSERT will FAIL (violates Consistency — no Customer_ID=999 exists):
INSERT INTO Orders (Customer_ID, Total) VALUES (999, 500);
-- ERROR 1452: Cannot add or update a child row: a foreign key constraint fails
```

### I — Isolation  
```
User A (in a transaction): reads Balance = ₹50,000
User B: transfers ₹20,000 into User A's account and commits
User A: reads Balance again → still sees ₹50,000 (isolated from B's change!)
User A: commits → now sees the updated ₹70,000
```
(This is REPEATABLE READ isolation — MySQL's default)

### D — Durability
MySQL writes every committed transaction to the **Redo Log** on disk **before** confirming success. Even if the power fails immediately after a COMMIT, MySQL reads the Redo Log on restart and completes the write.

---

## 4. How MySQL Achieves ACID — The Redo/Undo Log System (PRO LEVEL)

| ACID Property | MySQL Mechanism |
|---|---|
| **Atomicity** | Undo Log — stores the "before" values so ROLLBACK can reverse changes |
| **Consistency** | Constraint checking (FK, PK, CHECK, NOT NULL) at commit time |
| **Isolation** | MVCC (Multi-Version Concurrency Control) — each transaction reads from a snapshot |
| **Durability** | Redo Log (Write-Ahead Log) — changes written to Redo Log before the actual table page |

**The Redo Log (WAL — Write-Ahead Log):**
1. Before MySQL writes to the actual InnoDB `.ibd` data file, it first writes the change to the Redo Log file.
2. On crash recovery, MySQL reads the Redo Log and "replays" any committed-but-not-yet-written transactions.
3. This allows MySQL to confirm COMMIT instantly (Redo Log write is fast — sequential) rather than waiting for random disk writes to the data file.

---

## 5. Real-Life Examples

**Example 1 — E-commerce Checkout (All 4 ACID properties in action):**
```sql
START TRANSACTION;

-- Step 1: Reduce stock (will be rolled back if anything fails) → ATOMICITY
UPDATE Products SET Stock = Stock - 2 WHERE Product_ID = 45;

-- Step 2: Create order (FOREIGN KEY check ensures Customer exists) → CONSISTENCY
INSERT INTO Orders (Customer_ID, Total, Status) VALUES (123, 1499.98, 'Pending');

-- Step 3: Other transactions can't see the reduced stock yet → ISOLATION
INSERT INTO Order_Items (Order_ID, Product_ID, Qty) VALUES (LAST_INSERT_ID(), 45, 2);

COMMIT;  -- Data now permanently saved on disk → DURABILITY
```

**Example 2 — Hospital Patient Record (Atomicity):**
```sql
START TRANSACTION;

-- Admit patient and allocate a bed simultaneously
INSERT INTO Admissions (Patient_ID, Ward_ID, Admit_Date) VALUES (789, 12, NOW());
UPDATE Wards SET Beds_Available = Beds_Available - 1 WHERE Ward_ID = 12;

-- If the ward is already full (Beds_Available < 0 via CHECK constraint):
-- MySQL rolls back BOTH changes. Patient not admitted AND no bed count change.
COMMIT;
```

---

## 6. Which Storage Engine Supports ACID?

| Storage Engine | ACID Compliant? | Notes |
|---|---|---|
| **InnoDB** | ✅ Yes — Full ACID | MySQL's default since version 5.5 |
| **MyISAM** | ❌ No | No transactions, no rollback |
| **MEMORY** | ❌ No | Data lost on server restart |

**Always use InnoDB** for any table that handles business-critical data.

---

## 7. Common Mistakes

- **Using MyISAM for important tables:** MyISAM has no transaction support — no ROLLBACK. If a query fails mid-way, partial changes are permanent. Check `SHOW CREATE TABLE tablename` to verify engine is InnoDB.
- **Assuming autocommit = ACID protection:** Autocommit means every statement is its own transaction. It provides Atomicity per statement, but NOT for multi-step operations. Always use explicit `START TRANSACTION` for multi-step work.
- **Thinking ACID prevents applications from crashing:** ACID protects the database layer. Application code can still have bugs — ACID just ensures the database remains consistent even when the application fails.

---

## 8. Tips & Best Practices

- **Always use InnoDB** for any table that stores user data, financial records, or any business-critical information.
- **Wrap multi-step operations in transactions:** `START TRANSACTION ... COMMIT` is the mechanism that activates ACID's Atomicity guarantee across multiple statements.
- **Test rollback scenarios:** Include test cases that deliberately fail mid-transaction to verify your application correctly rolls back and leaves the database clean.

---

## 9. Mini Practice Tasks

- **Task 1:** Explain all 4 ACID properties using the analogy of a bank transfer. For each property, describe what would go wrong if that specific property was violated.
- **Task 2:** Which MySQL mechanism (Redo Log vs Undo Log) handles Atomicity, and which handles Durability? Explain the difference.
- **Task 3:** A table uses `MyISAM` storage engine. A developer runs an `UPDATE` that modifies 5,000 rows and the server crashes after updating 2,000 rows. What happens? How does the answer change if the table uses `InnoDB`?

---
