# ⚔️ Topic 9.5: WHERE vs. HAVING

This is a classic technical interview question. While both `WHERE` and `HAVING` are used for filtering, they operate on completely different data states and at different times during query execution.

---

## 1. Summary Comparison Table

| Feature | WHERE | HAVING |
|---|---|---|
| **What it filters** | Raw individual rows. | Summarized group rows. |
| **When it runs** | **Before** `GROUP BY`. | **After** `GROUP BY`. |
| **Aggregate usage** | **Forbidden** (cannot use SUM, AVG, etc.) | **Allowed** (designed for SUM, AVG, etc.) |
| **Performance** | **Faster** (reduces rows before math). | **Slower** (math runs on all rows first). |
| **Indexes** | Can utilize B-Tree indexes. | Cannot utilize indexes effectively. |

---

## 2. The Logic of Execution

Imagine a company with 10,000 employees across 10 departments. You want to find departments with an average salary > $80k, but you only care about employees in New York.

1.  **WHERE City = 'New York'**: Filters out 8,000 employees. Only 2,000 remain.
2.  **GROUP BY Department**: Splits the 2,000 employees into 10 buckets.
3.  **Calculate AVG(Salary)**: Runs math on those buckets.
4.  **HAVING AVG(Salary) > 80000**: Discards buckets that don't meet the criteria.

**Key Insight:** `WHERE` shrinks the input. `HAVING` shrinks the output.

---

## 3. When to Use Which?

*   **Use `WHERE`** if you want to exclude data **before** it gets counted/summed.
*   **Use `HAVING`** if you want to filter based on the result of a **count/sum**.

---

## 4. Pro-Level Example: Combining Both

```sql
-- Find high-performing departments, but only count full-time staff
SELECT 
    Department, 
    COUNT(*) AS FT_Staff, 
    AVG(Salary) AS Avg_Sal
FROM Employees
WHERE Employment_Type = 'Full-Time'  -- Filter raw rows
GROUP BY Department                  -- Group them
HAVING Avg_Sal > 70000;              -- Filter groups
```

---

## 5. Performance Implications (PRO LEVEL)

Senior Engineers always prefer `WHERE` over `HAVING` whenever possible. 

*   **`WHERE`** is **Individually Selective**. It can use an Index to skip Millions of rows.
*   **`HAVING`** is **Post-Processing**. The database must have already done the hard work of reading rows and calculating aggregates before `HAVING` can throw them away.

**Optimization Rule:** If a condition can be written in `WHERE`, it *must* be in `WHERE`.

---

## 6. Mini Practice Tasks

*   **Task 1:** Identify the error in this query: `SELECT Department, SUM(Salary) FROM Employees WHERE SUM(Salary) > 100000 GROUP BY Department;`
*   **Task 2:** Describe the "Logical Pipeline" of a query that uses both `WHERE` and `HAVING`. Which one executes first?

---
