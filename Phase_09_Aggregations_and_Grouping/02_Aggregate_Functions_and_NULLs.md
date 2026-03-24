# 🕳️ Topic 9.2: Aggregate Functions and NULLs

NULLs behave very strangely in aggregate functions. If you don't understand how MySQL handles missing data during summarization, your financial reports and averages will be mathematically incorrect.

---

## 1. The Golden Rule of Aggregation

**Virtually all aggregate functions EXCEPT `COUNT(*)` ignore `NULL` values entirely.**

They treat NULL as if the row simply doesn't exist for the purpose of that calculation.

---

## 2. How NULLs Affect Specific Functions

| Function | Behavior with NULL |
|---|---|
| `COUNT(*)` | Counts every row, including rows where all columns are NULL. |
| `COUNT(col)`| Counts only rows where `col` is NOT NULL. |
| `SUM(col)` | Ignores NULLs. (Total of 10, 20, NULL is 30). |
| `AVG(col)` | **The Trap:** Calculates `SUM(col) / COUNT(col)`. It ignores both the value AND the count of NULL rows. |
| `MIN/MAX` | Ignores NULLs. |

---

## 3. The "AVG" Trap (Critical Math Warning)

The most dangerous behavior is in `AVG`. Consider this table:
| Employee | Commissions |
|---|---|
| A | 100 |
| B | 200 |
| C | NULL (Did not earn any) |

*   **Mathematical Average (Commission per employee):** `(100 + 200 + 0) / 3 = 100`
*   **MySQL `AVG(Commissions)`:** `(100 + 200) / 2 = 150`

Because MySQL ignores Employee C's NULL, the average appears **higher** than it actually is across the whole company.

**✅ Professional Solution:** Use `IFNULL` or `COALESCE` to turn NULLs into 0 before averaging:
```sql
SELECT AVG(IFNULL(Commissions, 0)) AS Correct_Avg_Commission FROM Employees;
```

---

## 4. How It Works (Optimizer Optimization - PRO LEVEL)

Why does `COUNT(*)` exist separately from `COUNT(column)`?
- **`COUNT(*)`** is highly optimized in the InnoDB engine. It registers a request for the metadata of the table. If an index exists, it scans the smallest index possible to count keys.
- **`COUNT(column)`** forces MySQL to check the NULL bitmap of every single row to see if that specific column is NULL before incrementing the counter. This is significantly more CPU-intensive on large tables.

**Senior Tip:** If you just want a row count, **always use `COUNT(*)`**. It is the most performant way to count total rows.

---

## 5. SQL Examples (MySQL Execution)

```sql
-- 1. Difference between * and column count
SELECT 
    COUNT(*) AS Total_Rows,
    COUNT(Phone) AS Rows_With_Phone
FROM Employees;
-- Rows_With_Phone will be lower if some employees have NULL phones.

-- 2. Handling NULLs in SUM
-- Total income (Salary + Commission)
-- Problem: Salary + NULL = NULL
SELECT SUM(Salary + IFNULL(Commission, 0)) FROM Employees;

-- 3. MIN/MAX are unaffected by NULLs
SELECT MAX(Bonus) FROM Employees; -- Correctly finds highest non-NULL bonus
```

---

## 6. Common Mistakes

*   **Thinking `SUM` of a NULL column returns 0:** If every row in a group has a NULL value for that column, `SUM()` returns `NULL`, not `0`. 
    ```sql
    -- To ensure you get 0 instead of NULL:
    SELECT IFNULL(SUM(Salary), 0) FROM Employees WHERE Department = 'EmptyDept';
    ```

---

## 7. Tips & Best Practices (Pro-Level)

**`COUNT(1)` vs `COUNT(*)`:**
You will often see `SELECT COUNT(1) FROM table`. 
In modern MySQL (8.0+), the Optimizer treats `COUNT(1)` and `COUNT(*)` as **identical**. Neither is faster. `COUNT(*)` is preferred as it is the standard SQL syntax.

---

## 8. Mini Practice Tasks

*   **Task 1:** You have 10 rows. 3 rows have a NULL value in the `Email` column. What will `COUNT(*)` return? What will `COUNT(Email)` return?
*   **Task 2:** Write a query that calculates the average `Bonus` but ensures that employees with a NULL bonus are treated as having a `0` bonus in the calculation.

---
