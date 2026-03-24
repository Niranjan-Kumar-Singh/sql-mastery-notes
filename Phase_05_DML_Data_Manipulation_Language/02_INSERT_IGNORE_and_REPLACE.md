# 🔄 Topic 5.2: INSERT IGNORE & REPLACE (MySQL Specific Duplicate Handling)

We know a basic `INSERT INTO` will crash violently with **Error 1062** when you try to insert a row that duplicates a `UNIQUE` or `PRIMARY KEY` value. 

In real-world applications, this is extremely common. A user might refresh the sign-up page, or a background job might re-import a CSV with rows that already exist. We need strategies to handle duplicates gracefully. MySQL gives us two specialized tools: **`INSERT IGNORE`** and **`REPLACE`**.

---

## 1. Definition

1.  **`INSERT IGNORE` (MySQL-specific DML):** Attempts to insert a row. If the row violates a UNIQUE/PK constraint (duplicate), MySQL **silently discards** the conflicting row and continues. No error is thrown. The original row is untouched.

2.  **`REPLACE` (MySQL-specific DML):** Attempts to insert a row. If the row violates a UNIQUE/PK constraint, MySQL **first physically DELETEs** the old conflicting row entirely, and then **INSERTs** the new row fresh in its place. The old data is completely overwritten.

---

## 2. Why This Concept Exists

Production applications are messy. Data pipelines, API retries, and user errors continuously try to re-insert already-existing data. Crashing the entire application every time a duplicate appears is unacceptable. These two commands give engineers clean, non-destructive strategies to handle collisions.

---

## 3. Why We Use It

*   **`INSERT IGNORE`:** Best for data ingestion pipelines. If you are importing 1 million product records from a CSV file and some already exist, you want to quietly skip the duplicates and insert only the new ones.
*   **`REPLACE`:** Best for "Upsert" logic (Update if exists, Insert if not). For example, refreshing a product price feed — if the product is already in the DB, update its price by replacing the entire row.

---

## 4. The Critical Mechanical Difference (READ THIS CAREFULLY)

| Behavior | `INSERT IGNORE` | `REPLACE` |
| :--- | :--- | :--- |
| Duplicate found? | Skips the row silently | Deletes old row, inserts new |
| Existing data preserved? | ✅ Yes, original row untouched | ❌ No, old row is DELETED first |
| AUTO_INCREMENT counter | Not incremented on skip | **Always incremented, even if replacing!** |
| Foreign Key children | Safe — no delete happens | ⚠️ Can trigger cascade deletes on children! |

---

## 5. How It Works (The REPLACE DELETE+INSERT Cycle - PRO LEVEL)

The `REPLACE` command is not magic. Internally, MySQL's InnoDB executes it as a strict two-step DML pipeline:

**Step 1:** MySQL runs `DELETE FROM table WHERE pk = conflict_value;`
**Step 2:** MySQL runs a fresh `INSERT INTO table VALUES (new_data);`

The consequences of this are massive:
- The **AUTO_INCREMENT ticker always advances**, even if replacing. If you started at ID 1 and REPLACE fires on a conflict with ID 1, after the operation, your next new row gets ID 2, but the ticker has internally jumped. After enough REPLACEs, your ID numbers can seem to leap forward inexplicably.
- Any child rows linked via **`ON DELETE CASCADE`** Foreign Keys will be permanently wiped during the DELETE phase!   

---

## 6. Syntax / Implementation

```sql
-- INSERT IGNORE: Skip duplicates silently
INSERT IGNORE INTO table_name (column1, column2) 
VALUES (value1, value2);

-- REPLACE: Delete old, insert new
REPLACE INTO table_name (column1, column2) 
VALUES (value1, value2);
```

---

## 7. Real-Life Examples

**Music Streaming Service — Daily Chart Update:**

*   A background job runs every midnight to upsert the current "#1 Song" in the `Weekly_Charts` table.
*   If this is a brand-new chart entry: `INSERT` normally works.
*   If the same song is #1 for the second week, the row already exists: `REPLACE` fires, deletes the old entry (with last week's stats), and inserts a fresh row with updated streaming counts.

**CSV Product Import — E-Commerce:**
*   A store manager uploads 50,000 rows from a spreadsheet.
*   2,000 of those products already exist in the database. `INSERT IGNORE` silently skips all 2,000 and inserts only the 48,000 genuinely new ones. The manager sees `48000 rows inserted, 2000 warnings` instead of a crash.

---

## 8. SQL Examples (MySQL Execution)

```sql
CREATE TABLE Users (
    User_ID INT PRIMARY KEY AUTO_INCREMENT,
    Email VARCHAR(255) UNIQUE NOT NULL,
    Username VARCHAR(100)
);

INSERT INTO Users (Email, Username) VALUES ('alice@gmail.com', 'Alice');

-- ❌ This normally CRASHES with Error 1062!
INSERT INTO Users (Email, Username) VALUES ('alice@gmail.com', 'AliceUpdated');

-- ✅ INSERT IGNORE: Silently skips. Alice remains 'Alice'. No error.
INSERT IGNORE INTO Users (Email, Username) VALUES ('alice@gmail.com', 'AliceUpdated');
-- Result: 0 rows affected, 1 warning. Original row is SAFE.

-- ✅ REPLACE: Alice's old row is DELETED. New row with 'AliceV2' is inserted.
REPLACE INTO Users (Email, Username) VALUES ('alice@gmail.com', 'AliceV2');
-- Result: 2 rows affected (1 delete + 1 insert). Username is NOW 'AliceV2'.
```

---

## 9. Common Mistakes

*   **Using `REPLACE` when you only want to update one column:** `REPLACE` deletes the entire old row. If your `Users` table has 20 columns, and you `REPLACE` by only specifying 2 columns, all 18 un-specified columns will be set to their `NULL` or `DEFAULT` values! Use `INSERT ... ON DUPLICATE KEY UPDATE` instead for partial updates (covered in Topic 5.3).
*   **Forgetting REPLACE's FK Cascade Impact:** If an `Orders` table has a `FOREIGN KEY ... ON DELETE CASCADE` pointing to `Users`, and you `REPLACE` a User row, the underlying DELETE step will trigger a cascade and permanently wipe all that user's orders!

---

## 10. Tips & Best Practices (Pro-Level)

**The Professional Alternative: `INSERT ... ON DUPLICATE KEY UPDATE`**

For precise, surgical upsert logic, senior MySQL engineers almost always prefer this hybrid command over plain `REPLACE`:

```sql
INSERT INTO Products (Product_Code, Price, Stock)
VALUES ('LAPTOP-01', 999.99, 50)
ON DUPLICATE KEY UPDATE 
    Price = VALUES(Price),   -- Only update the Price column
    Stock = VALUES(Stock);   -- Only update the Stock column
-- Result: If new → INSERT. If duplicate → UPDATE only Price and Stock. Nothing else is touched!
```

*This is the safest way to upsert: it never deletes the row, preserves all other column data, and doesn't trigger cascade deletes.*

---

## 11. Mini Practice Tasks

*   **Task 1:** You have a `Scores` table with a `UNIQUE` constraint on `Player_Name`. A player named "Bob" already has a score. Write the query using `INSERT IGNORE` to attempt inserting Bob again with a new score of 500. What will happen to Bob's existing record?
*   **Task 2:** Explain in your own words why `REPLACE INTO` can be dangerous when the table being modified has child rows linked via `ON DELETE CASCADE` Foreign Keys.

---
