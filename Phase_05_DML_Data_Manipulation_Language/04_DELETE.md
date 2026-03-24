# 🗑️ Topic 5.4: DELETE (Removing Specific Records Safely)

We can add rows (`INSERT`), we can change rows (`UPDATE`). The final atomic operation in DML is physical removal. When a user cancels their account or a product is discontinued, we need to remove individual rows from the table. This is the job of **`DELETE`**.

---

## 1. Definition

**`DELETE` (DML):** A Data Manipulation Language command that permanently removes one or more specifically targeted rows from a table, while leaving the table structure and all other rows completely untouched.

---

## 2. Why This Concept Exists

The lifecycle of most data entities eventually ends. An expired promotional coupon is redeemed and must be removed. A spam comment is reported and must be deleted. A test data entry was created by mistake and must be cleaned up. `DELETE` gives us the surgical precision to remove exactly the rows we specify.

---

## 3. Why We Use It

*   **Row-Level Removal:** Unlike `TRUNCATE` (which nukes everything) and `DROP` (which destroys the table itself), `DELETE` is the only command precise enough to target one single specific row.
*   **Foreign Key Awareness:** `DELETE` is the only destruction command that is fully aware of, and correctly processes, Foreign Key constraints and cascade rules.
*   **Rollback Safety:** Because `DELETE` is DML, it writes to the Undo-Log, making it the only safe, reversible deletion command.

---

## 4. When to Use It

*   When you need to remove **specific rows** with a targeted `WHERE` clause.
*   When the table has **Foreign Key children** that must be properly cascade-deleted or restricted (use `DELETE`, not `TRUNCATE`).
*   When you need the option to **undo** the deletion via `ROLLBACK`.

---

## 5. How It Works (Row-by-Row Undo-Log - PRO LEVEL)

This is why `DELETE` is fundamentally slower than `TRUNCATE` on large datasets — but why it is irreplaceable:

For every single row `DELETE` targets:
1.  MySQL reads the row from the Buffer Pool.
2.  MySQL writes a **full backup copy** of that row into the **Undo-Log**.
3.  MySQL marks the row as "deleted" in the B-Tree page (it's not immediately physically removed — it becomes a "ghost" marked for purge).
4.  InnoDB's background **Purge Thread** eventually reclaims the ghost space.
5.  Foreign Key triggers are evaluated row by row (cascade, restrict, set null — all fire here).

This process makes `DELETE` safe, transactional, and FK-aware — but it means deleting 10 million rows writes 10 million Undo-Log entries, making it excruciatingly slow compared to `TRUNCATE`.

---

## 6. Syntax / Implementation (Cheat Sheet)

```sql
-- Targeted delete (The safe, correct way)
DELETE FROM table_name WHERE condition;

-- Delete everything (But keep the table structure)
-- ⚠️ Think twice — use TRUNCATE for large tables instead
DELETE FROM table_name;
```

---

## 7. Real-Life Examples

**The Airbnb Review System:**
1.  A guest's review is flagged as abusive spam.
2.  A moderation system fires: `DELETE FROM Reviews WHERE Review_ID = 58432 AND is_flagged = TRUE;`
3.  The review is permanently removed. The Listing and all other reviews are untouched.

---

## 8. SQL Examples (MySQL Execution)

```sql
-- Setup
CREATE TABLE Comments (
    Comment_ID INT PRIMARY KEY AUTO_INCREMENT,
    Post_ID INT,
    Author_Name VARCHAR(100),
    Body TEXT,
    is_spam BOOLEAN DEFAULT FALSE
);

-- 1. Delete a single specific row
DELETE FROM Comments WHERE Comment_ID = 42;

-- 2. Delete using a condition (Clean up all spam at once)
DELETE FROM Comments WHERE is_spam = TRUE;

-- 3. Safe transactional delete (Senior pattern — always use this in production!)
BEGIN;
    DELETE FROM Comments WHERE Post_ID = 99;
    -- Review the number of rows affected before committing
COMMIT;
-- ROLLBACK; -- Use this if the wrong rows were deleted!

-- 4. Delete with ORDER BY and LIMIT (Purge only the oldest 100 logs, not everything)
DELETE FROM System_Logs 
ORDER BY Created_At ASC 
LIMIT 100;
```

---

## 9. Common Mistakes

*   **Forgetting the `WHERE` Clause (The Fatal Mistake):**
    ```sql
    DELETE FROM Users;
    -- RESULT: All 5 million users are deleted, one-by-one, writing 5 million Undo-Log entries.
    -- This could take 45 minutes and consume all your server RAM.
    -- If not wrapped in a transaction, it cannot be rolled back!
    ```

*   **Not Wrapping DELETE in a Transaction:** Most non-MySQL-Workbench clients run in `auto-commit` mode by default. This means the moment `DELETE FROM Users WHERE User_ID = 7` hits Enter, it commits instantly to disk without a safety window. **Always wrap important DELETE statements in `BEGIN;` ... `COMMIT;`** so you have a review moment before making it permanent.

---

## 10. Tips & Best Practices (Pro-Level)

**The Soft-Delete Pattern (Never Actually Deleting in Production):**

Senior backend engineers at companies handling financial or legal data almost *never* run the physical `DELETE` command on critical entities like `Users`, `Orders`, or `Transactions`.

Why? Because regulator compliance laws (GDPR, SOX, HIPAA) require audit histories. If a bank truly deleted a transaction, they would break the law.

*The Enterprise Solution: The Soft-Delete Architecture (Revisited)*
```sql
-- Add a boolean flag during table design
ALTER TABLE Users ADD COLUMN is_deleted BOOLEAN NOT NULL DEFAULT FALSE;
ALTER TABLE Users ADD COLUMN deleted_at TIMESTAMP NULL;

-- "Delete" the user (without actually deleting them)
UPDATE Users 
SET is_deleted = TRUE, deleted_at = NOW() 
WHERE User_ID = 88;

-- Query only "visible" users everywhere in the backend code
SELECT * FROM Users WHERE is_deleted = FALSE;
```
The user is completely hidden from the application, but all their payment history, order receipts, and account data are safely preserved on disk for legal auditing. 

---

## 11. Mini Practice Tasks

*   **Task 1:** Write the SQL statement to permanently delete all products from a `Products` table where the product's `Stock` column equals exactly `0`.
*   **Task 2:** You need to delete only the 5 oldest rows from an `Error_Logs` table (identified by `Created_At`). Write the DELETE statement that uses both `ORDER BY` and `LIMIT` to achieve this precisely without deleting anything else.

---
