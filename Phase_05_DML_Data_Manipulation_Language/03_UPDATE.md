# ✏️ Topic 5.3: UPDATE (Modifying Specific Records Safely)

`INSERT INTO` adds entirely new rows. 

But data changes over time. A user changes their email address. A product price drops. An order status advances from "Pending" to "Shipped". We don't want to delete the old row and re-insert a new one — we want to surgically modify the specific cell that changed. This is exactly what **`UPDATE`** does.

---

## 1. Definition

**`UPDATE` (DML):** A Data Manipulation Language command that modifies one or more column values within one or more existing rows. It never creates new rows or removes any rows — it purely changes cell values in-place within the existing table grid.

---

## 2. Why This Concept Exists

Data is always in motion. In any live application, far more `UPDATE` statements are executed every second than `INSERT` statements. Every "Like" on a Facebook post increments a counter. Every password change rewrites a single column. Every GPS coordinate update on a delivery app overwrites the coordinates. `UPDATE` is the beating heart of mutable, living data.

---

## 3. Why We Use It

*   **State Machines:** An order progressing from `'Pending'` → `'Processing'` → `'Shipped'` → `'Delivered'` is 3 UPDATE operations on a single `Status` column.
*   **Incremental Counters:** A page view counter, a `Login_Count`, or a product `Stock` level are all decremented or incremented via UPDATE.
*   **User Profile Edits:** Every time a user saves changes to their profile, a single `UPDATE Users SET ... WHERE User_ID = X` fires.

---

## 4. When to Use It

*   When the data you want to change physically **already exists** as a row in the table.
*   Do NOT use `UPDATE` if you need to create a new entity — use `INSERT INTO`.

---

## 5. How It Works (The Undo-Log Safety Net - PRO LEVEL)

This is a critical difference from DDL commands: `UPDATE` is a **safe** DML command.

When MySQL executes an `UPDATE`, InnoDB:
1.  **Reads** the current row from disk/buffer.
2.  **Writes the old version** of the row into the **Undo-Log** (a hidden rollback file on disk).
3.  **Writes the new version** of the row into the Buffer Pool.
4.  If a `ROLLBACK` is triggered before `COMMIT`, MySQL reads the Undo-Log and restores the original row perfectly.

This is the core DML safety guarantee — `UPDATE` can always be undone if wrapped in a `BEGIN...ROLLBACK` transaction block!

---

## 6. Syntax / Implementation (Cheat Sheet)

```sql
UPDATE table_name
SET column1 = new_value1, column2 = new_value2
WHERE condition;
```

**⚠️ The `WHERE` clause is NOT optional in production!** Without it, MySQL will update every single row in the table simultaneously!

---

## 7. Real-Life Examples

**The Amazon Order Shipping System:**
1.  A warehouse worker scans an order barcode.
2.  The warehouse app fires: `UPDATE Orders SET Status = 'Shipped', Shipped_At = NOW() WHERE Order_ID = 10452;`
3.  The customer's app immediately shows "Your order has been shipped!"

---

## 8. SQL Examples (MySQL Execution)

```sql
-- Setup
CREATE TABLE Employees (
    Emp_ID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100),
    Department VARCHAR(50),
    Salary DECIMAL(10,2),
    is_active BOOLEAN DEFAULT TRUE
);

-- 1. Update a single column for a specific row
UPDATE Employees
SET Department = 'Engineering'
WHERE Emp_ID = 5;

-- 2. Update multiple columns at once
UPDATE Employees
SET Salary = 85000.00, Department = 'Senior Engineering'
WHERE Emp_ID = 5;

-- 3. Update using math (Giving everyone a 10% raise!)
UPDATE Employees
SET Salary = Salary * 1.10
WHERE Department = 'Engineering';

-- 4. Safe Soft-Delete Update (NOT a full delete — see Phase 4 Soft-Delete)
UPDATE Employees
SET is_active = FALSE
WHERE Emp_ID = 12;
```

---

## 9. Common Mistakes

*   **Forgetting the `WHERE` Clause (The #1 Most Dangerous Mistake in SQL):**
    ```sql
    -- The developer meant to update just Bob's salary!
    UPDATE Employees SET Salary = 200000.00;
    -- RESULT: Every single employee in the company now earns $200,000!
    ```
    *This is the most catastrophic junior mistake possible. Always double-check your `WHERE` clause before executing in production!*

*   **Wrapping UPDATE in a Transaction for Safety:**
    Before running any large `UPDATE`, always begin a transaction so you have an escape hatch:
    ```sql
    BEGIN;
    UPDATE Employees SET Salary = 200000 WHERE Emp_ID = 7;
    -- Review the result before confirming:
    SELECT Salary FROM Employees WHERE Emp_ID = 7;
    COMMIT; -- Only commit if it looks correct!
    -- ROLLBACK; -- Fire this instead if something looks wrong!
    ```

---

## 10. Tips & Best Practices (Pro-Level)

**The Row Lock During UPDATE:**
When MySQL executes an `UPDATE`, InnoDB places an exclusive **X-Lock** (Write Lock) on the specific rows being modified. This means no other database thread can read or write those same rows until the transaction commits.

In high-traffic systems like Twitter, if you fire `UPDATE Tweets SET Like_Count = Like_Count + 1 WHERE Tweet_ID = 1;` 50,000 times per second, the rows will be X-Locked in a queue. This is normal and correct — InnoDB processes the lock queue sequentially to guarantee accurate counting.

*Senior engineers avoid this bottleneck by using MySQL's atomic `UPDATE` directly rather than a read-calculate-write pattern from the application backend.*

---

## 11. Mini Practice Tasks

*   **Task 1:** Write the exact SQL command to update a user's email address from `'old@mail.com'` to `'new@mail.com'` for the user with `User_ID = 99` in a `Users` table.
*   **Task 2:** Describe the consequence of accidentally running an `UPDATE` statement without a `WHERE` clause on a live production `Customers` table with 2 million rows.

---
