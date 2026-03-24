# 🏁 Topic 15.2: TCL — Transactions (COMMIT, ROLLBACK, SAVEPOINT)

Transaction Control Language (TCL) gives you a **safety net** for your DML operations. It lets you group multiple SQL statements into one atomic unit of work — either all succeed together, or none of them do. This is the feature that protects bank transfers, e-commerce checkouts, and every critical operation from partial failures.

---

## 1. The Core TCL Commands

| Command | What It Does |
|---|---|
| `START TRANSACTION` / `BEGIN` | Marks the beginning of a safe unit of work. Pauses autocommit. |
| `COMMIT` | Permanently saves all changes made since `START TRANSACTION`. |
| `ROLLBACK` | Discards all changes made since `START TRANSACTION`. Restores previous state. |
| `SAVEPOINT name` | Creates a named checkpoint inside a transaction. |
| `ROLLBACK TO name` | Rolls back only to the named savepoint (not the whole transaction). |
| `RELEASE SAVEPOINT name` | Removes a savepoint (optional cleanup). |

---

## 2. Why This Concept Exists

- **Human Error Protection:** You accidentally run `DELETE FROM Orders` without a `WHERE` clause. If you were inside a transaction, just type `ROLLBACK` and every row comes back instantly.
- **Logical Dependency:** A bank transfer must debit Account A AND credit Account B. If only the debit succeeds, money disappears. Wrapping both in a transaction ensures they either both succeed or both fail together.
- **Integrity During Crashes:** Combined with ACID properties (Topic 15.1), if the server crashes mid-transaction, MySQL uses its Redo/Undo logs to either complete or roll back the transaction cleanly on restart.

---

## 3. How Autocommit Works (Critical Understanding)

By default, MySQL runs in **Autocommit Mode**:
```sql
SELECT @@autocommit; -- Returns 1 (ON)
```

In autocommit mode, **every single SQL statement is immediately and permanently committed** to disk. There is no undo.

You have two ways to take control:

**Option 1: Use `START TRANSACTION` (Recommended — Explicit)**
```sql
START TRANSACTION;        -- Pauses autocommit just for this block
UPDATE Accounts SET Balance = Balance - 100 WHERE ID = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE ID = 2;
COMMIT;                   -- Now both updates are permanent
```

**Option 2: Turn off autocommit for the session**
```sql
SET autocommit = 0;       -- All future statements need manual COMMIT
UPDATE Products SET Price = 500 WHERE ID = 3;
COMMIT;
SET autocommit = 1;       -- Always turn it back on!
```

---

## 4. Complete Syntax Examples

### Basic Transaction
```sql
START TRANSACTION;

-- Step 1: Debit sender
UPDATE Accounts SET Balance = Balance - 1000 WHERE Account_ID = 101;

-- Step 2: Credit receiver
UPDATE Accounts SET Balance = Balance + 1000 WHERE Account_ID = 202;

-- Check: Did both work correctly?
SELECT Balance FROM Accounts WHERE Account_ID IN (101, 202);

COMMIT;   -- Looks good — make it permanent!
-- OR:
-- ROLLBACK;  -- Something went wrong — undo everything!
```

### Using SAVEPOINTS (Checkpoint-Based Recovery)
```sql
START TRANSACTION;

INSERT INTO Orders (Customer_ID, Total) VALUES (5, 250.00);
SAVEPOINT after_order;   -- ← Checkpoint: order is saved here

INSERT INTO Order_Items (Order_ID, Product_ID, Qty) VALUES (LAST_INSERT_ID(), 12, 2);
SAVEPOINT after_items;   -- ← Checkpoint: items are saved here

INSERT INTO Invoices (Order_ID, Due_Date) VALUES (LAST_INSERT_ID(), '2024-12-31');
-- Something goes wrong in invoices...

ROLLBACK TO after_items; -- ← Undo only the invoice, keep order + items ✅

COMMIT;   -- Permanently save: order + items (invoice is discarded)
```

---

## 5. Real-Life Example — E-commerce Checkout

```sql
START TRANSACTION;

-- 1. Reduce stock
UPDATE Products SET Stock = Stock - 2 WHERE Product_ID = 45;

-- 2. Check if stock went negative (data validation)
SELECT Stock FROM Products WHERE Product_ID = 45 INTO @new_stock;
IF @new_stock < 0 THEN
    ROLLBACK;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Out of stock!';
END IF;

-- 3. Create the order
INSERT INTO Orders (Customer_ID, Status, Total) VALUES (88, 'Pending', 199.98);

-- 4. Create shipping record
INSERT INTO Shipments (Order_ID, Address) VALUES (LAST_INSERT_ID(), '42 Baker St');

COMMIT;  -- All 4 steps succeeded → commit everything atomically
```

---

## 6. Common Mistakes

- **Leaving transactions open too long:** While a transaction is open, InnoDB holds **Row Locks** on every row you touched. Other database connections that try to update those same rows will be **blocked** until you `COMMIT` or `ROLLBACK`. Long-running transactions can freeze entire parts of your application.
- **Forgetting TCL doesn't protect DDL:** `COMMIT` and `ROLLBACK` only protect DML (`INSERT`, `UPDATE`, `DELETE`). DDL commands like `CREATE TABLE`, `DROP TABLE`, and `ALTER TABLE` **always auto-commit immediately** in MySQL — they cannot be rolled back.
- **Not handling errors in application code:** If your Python/Node.js app throws an exception mid-transaction, you must catch it and explicitly call `ROLLBACK`. Otherwise the transaction might stay open and locks accumulate.

---

## 7. DDL vs DML — What Can Be Rolled Back?

| Command Type | Can Be Rolled Back? | Examples |
|---|---|---|
| DML | ✅ Yes | `INSERT`, `UPDATE`, `DELETE` |
| DDL | ❌ No (auto-commits) | `CREATE TABLE`, `DROP TABLE`, `ALTER TABLE`, `TRUNCATE` |
| DCL | ❌ No | `GRANT`, `REVOKE` |

---

## 8. Tips & Best Practices

- **Wrap ALL multi-step operations in transactions:** Any time your operation touches more than one table or row, use `START TRANSACTION`.
- **Use SAVEPOINT for long batch operations:** If you're processing 1,000 rows in a loop, set a SAVEPOINT every 100 rows so you don't have to restart entirely if something fails at row 850.
- **Keep transactions as short as possible:** Open → Work → Commit. The longer a transaction lives, the more it holds locks and blocks other users.

---

## 9. Mini Practice Tasks

- **Task 1:** What is the difference between `COMMIT` and `ROLLBACK`? Write a complete transaction that transfers ₹500 from `Account_ID = 1` to `Account_ID = 2`, and only commits if both `UPDATE` statements succeed.
- **Task 2:** Why can't you `ROLLBACK` a `DROP TABLE` command even if you're inside a `START TRANSACTION` block?
- **Task 3:** Write a script that starts a transaction, inserts a record into a `Logs` table, creates a `SAVEPOINT`, inserts a second record, and then **rolls back only the second insert** using `ROLLBACK TO SAVEPOINT`.

---
