# 🏆 Topic 13.2: Ranking Functions (ROW_NUMBER, RANK, DENSE_RANK)

Ranking is one of the most common tasks in analytics: top 5 customers, 2nd highest salary, rank products by sales. Standard `ORDER BY` sorts data but doesn't assign rank numbers. That's what Ranking Functions do.

---

## 1. The Three Ranking Functions

MySQL 8.0+ provides three ranking functions, each handling **ties (duplicate values)** differently:

| Function | Behavior on Ties | Output for `{100, 90, 90, 80}` |
|---|---|---|
| **`ROW_NUMBER()`** | No ties — every row gets a unique sequential number | 1, 2, 3, 4 |
| **`RANK()`** | Ties get the same rank, next rank **skips** numbers | 1, 2, 2, 4 |
| **`DENSE_RANK()`** | Ties get the same rank, next rank **doesn't skip** | 1, 2, 2, 3 |

---

## 2. Why Each One Exists

- **`ROW_NUMBER()`:** When you need a unique identifier per row (pagination, deduplication, picking exactly the "first" row).
- **`RANK()`:** Olympic/sports-style ranking — two gold medal winners, nobody gets silver, the next person gets bronze.
- **`DENSE_RANK()`:** Salary band ranking — two people with the same salary are "Rank 2", the next unique salary is "Rank 3" (not 4).

---

## 3. How It Works — The Partition Sort (PRO LEVEL)

All ranking functions work identically under the hood:
1. Data is split into groups (partitions) if `PARTITION BY` is specified.
2. Within each partition, rows are **sorted** by the `ORDER BY` clause inside `OVER()`.
3. The ranking math is applied to the sorted partition.
4. **Important:** This is a **blocking operation** — MySQL must have all rows in memory (sorted) before it can assign ranks. On a 50M-row table without careful indexing, this can be slow.

```sql
-- Always pair ranking with an ORDER BY and an index on the ranking column!
-- Add an index: CREATE INDEX idx_salary ON Employees(Salary DESC);
```

---

## 4. Syntax / Implementation — Full Reference

```sql
-- All three in one query for comparison
SELECT 
    Name, 
    Score,
    ROW_NUMBER() OVER(ORDER BY Score DESC) AS Row_Num,
    RANK()       OVER(ORDER BY Score DESC) AS Standard_Rank,
    DENSE_RANK() OVER(ORDER BY Score DESC) AS Compact_Rank
FROM Exam_Results;

-- With PARTITION BY: Rank within each department separately
SELECT 
    Name, 
    Department,
    Salary,
    DENSE_RANK() OVER(PARTITION BY Department ORDER BY Salary DESC) AS Dept_Rank
FROM Employees;

-- Stable ROW_NUMBER tied with a tiebreaker (prevents non-deterministic results)
SELECT 
    Name,
    Score,
    ROW_NUMBER() OVER(ORDER BY Score DESC, Name ASC) AS Unique_Row_Num
FROM Students;
-- Adding 'Name ASC' as tiebreaker ensures consistent results on every run
```

---

## 5. Real-Life Examples

**Example 1 — Top 3 Customers per Region:**
```sql
SELECT * FROM (
    SELECT 
        Customer_Name,
        Region,
        Total_Spent,
        RANK() OVER(PARTITION BY Region ORDER BY Total_Spent DESC) AS Regional_Rank
    FROM Customers
) Ranked
WHERE Regional_Rank <= 3;
-- Shows top 3 spenders in each region. If two are tied for #2, both appear (RANK skips to 4).
```

**Example 2 — Deduplication: Latest Record per User:**
```sql
-- Remove duplicate login records, keep only the most recent per user
SELECT * FROM (
    SELECT 
        User_ID, 
        Login_Time,
        ROW_NUMBER() OVER(PARTITION BY User_ID ORDER BY Login_Time DESC) AS rn
    FROM Login_History
) ranked
WHERE rn = 1;
-- ROW_NUMBER ensures exactly 1 row per user (even if two had same Login_Time)
```

**Example 3 — Salary Band Report:**
```sql
SELECT 
    Name,
    Department,
    Salary,
    DENSE_RANK() OVER(ORDER BY Salary DESC) AS Salary_Band,
    CASE 
        WHEN DENSE_RANK() OVER(ORDER BY Salary DESC) = 1 THEN 'Band A (Executive)'
        WHEN DENSE_RANK() OVER(ORDER BY Salary DESC) <= 3 THEN 'Band B (Senior)'
        ELSE 'Band C (Staff)'
    END AS Pay_Grade
FROM Employees;
```

---

## 6. Which Function to Use — Decision Guide

| Scenario | Best Function | Reason |
|---|---|---|
| Pagination (unique row identifier) | `ROW_NUMBER()` | Every row needs a unique number |
| Sports/competition ranking | `RANK()` | Ties share rank, a gap after |
| Salary bands / continuous ranking | `DENSE_RANK()` | No gaps after ties |
| "Find the Nth highest value" | `DENSE_RANK()` | Handles ties without skipping |
| Deduplication (keep one row per group) | `ROW_NUMBER()` | Only unique sequential numbers |

---

## 7. Common Mistakes

- **Forgetting `ORDER BY` inside `OVER()`:** A ranking function without `ORDER BY` produces a constant rank for all rows (rank 1 everywhere). It's syntactically valid but meaningless.
- **Non-deterministic ties with `ROW_NUMBER()`:** If two students both score 90 and you sort only by score, MySQL may assign row numbers 2 and 3 in any order — different every run. Always add a secondary tiebreaker column.
- **Trying to use ranking in `WHERE`:** `WHERE RANK() OVER(...) = 1` is a syntax error. Wrap it in a subquery or CTE:
    ```sql
    SELECT * FROM (SELECT *, RANK() OVER(ORDER BY Score DESC) AS rnk FROM Students) t
    WHERE t.rnk = 1;
    ```

---

## 8. Tips & Best Practices

- **Prefer `DENSE_RANK()` for most analytical reports.** The gap in `RANK()` (1,2,2,4) often confuses end users who expect contiguous rank numbers.
- **For pagination, always use `ROW_NUMBER()` over `LIMIT/OFFSET`** on large tables — see keyset pagination (Topic 7.8).
- **Add a tiebreaker column** to any `ORDER BY` inside a ranking function to guarantee deterministic, reproducible results.

---

## 9. Mini Practice Tasks

- **Task 1:** Given exam scores `{85, 92, 92, 78, 92}`, write what `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` would produce for each row (ordered descending).
- **Task 2:** Write a query to find the **2nd highest salary** in each department, handling ties correctly (use `DENSE_RANK`). Tables: `Employees(Emp_ID, Name, Department, Salary)`.
- **Task 3:** Why can't you use `WHERE RANK() OVER(...) = 1` directly in a query? What is the correct way to filter by a ranking function result?

---
