# 🏢 Topic 13.7: PARTITION BY vs. GROUP BY

These two clauses look identical—both separate data based on shared values. But their internal logic and final output are opposites. Understanding this distinction is the mark of a Senior SQL master.

---

## 1. Summary Comparison Table

| Feature | GROUP BY | PARTITION BY (Window) |
|---|---|---|
| **Rows** | Collapses multiple rows into one. | Keeps all original rows visible. |
| **SELECT** | Only grouped or aggregated cols allowed. | Any column from the row is allowed. |
| **Logic** | Summarization / Aggregation. | Analytical / Contextual. |
| **Execution**| Early in the pipeline (shrinks data). | Late in the pipeline (adds columns). |

---

## 2. When to Use Which?

*   **Use `GROUP BY`** when you want a **High-Level Aggregate**.
    - *"What was the total profit per region?"* (Output: 1 row per region).
*   **Use `PARTITION BY`** when you want **Detail + Context**.
    - *"Show every transaction and how much it contributed to the region's total profit."* (Output: 1 row per transaction).

---

## 3. How It Works (The Internal Buffer - PRO LEVEL)

- **`GROUP BY`** uses a **Hash Table or Sort-Group** mechanism to find unique keys and aggregate them. Once a row is collapsed, the original values (like "Transaction ID") are purged from memory to save space.
- **`PARTITION BY`** maintains the full dataset in a **Window Buffer**. It sorts the data but keeps every pointer to the original row. This consumes more RAM but preserves the granularity of the dataset.

---

## 4. Syntax Comparison

```sql
-- COLLAPSED (Summary)
SELECT Department, SUM(Salary) 
FROM Employees 
GROUP BY Department;

-- PRESERVED (Analytical)
SELECT Name, Department, Salary,
       SUM(Salary) OVER(PARTITION BY Department) AS Dept_Total
FROM Employees;
```

---

## 5. Real-Life Examples

**The "Top Performer" Comparison:**
"Show every salesperson, their sales, the average of their team, and the difference." 
This requires `PARTITION BY` because you need the individual `Saleperson_Name` which `GROUP BY` would erase.

```sql
SELECT 
    Name, Team, Sales,
    AVG(Sales) OVER(PARTITION BY Team) AS Team_Avg,
    (Sales - AVG(Sales) OVER(PARTITION BY Team)) AS Performance_Gap
FROM Sales_Rep;
```

---

## 6. Mini Practice Tasks

*   **Task 1:** Describe a scenario where `PARTITION BY` is necessary and `GROUP BY` would not work.
*   **Task 2:** How does the number of rows in the final result set differ between a query using `GROUP BY` and a query using `PARTITION BY`?

---
