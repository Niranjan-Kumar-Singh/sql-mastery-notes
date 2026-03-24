# 🪟 Topic 13.1: Introduction to Window Functions (The OVER Clause)

Before MySQL 8.0, if you wanted to see each employee's salary alongside the department's total salary, you had to use a complex JOIN or a Subquery. **Window Functions** changed everything by allowing "Aggregations without Collapsing."

---

## 1. Definition

**Window Function:** A function that performs a calculation across a set of table rows that are somehow related to the current row. Unlike regular aggregate functions, window functions **do not** group rows into a single output row; each row in the original query remains visible.

---

## 2. The `OVER()` Clause

The "magic" of a window function is the `OVER()` clause. It tells MySQL: *"This is a window function; here is how to define the window of rows it should look at."*

**Basic Anatomy:**
```sql
SELECT 
    column_name, 
    FUNCTION() OVER (PARTITION BY ... ORDER BY ...) 
FROM table;
```

---

## 3. Why This Concept Exists

*   **Row-Level Context:** Compare an individual's performance against the group without losing the individual's data.
*   **Sequential Logic:** Calculate running totals, moving averages, or rank rows.
*   **Performance:** Window functions are often much faster and more memory-efficient than the equivalent `SELF JOIN` or `CORRELATED SUBQUERY`.

---

## 4. How It Works (The Multi-Pass Execution - PRO LEVEL)

Internal Execution Flow:
1.  **From/Where/Group By:** MySQL performs standard filtering and grouping first.
2.  **Window Pass:** MySQL takes the result set and **sorts it** in memory based on the `OVER(ORDER BY ...)` clause.
3.  **Calculation:** It iterates through the rows, maintaining a "sliding window" of data to perform the requested math.
4.  **Selection:** The final result is returned to the user.

**Crucial Note:** Window functions are among the last things executed in a query. You cannot use a window function in a `WHERE` or `HAVING` clause because those clauses run **before** the window is calculated.

---

## 5. Syntax / Implementation

```sql
SELECT 
    Employee_Name, 
    Salary, 
    AVG(Salary) OVER() AS Global_Avg_Salary
FROM Employees;
-- Every row will now show the employee's name, their salary, AND the overall average.
```

---

## 6. Real-Life Examples

**Departmental Comparisons:**
```sql
SELECT 
    Name, 
    Department, 
    Salary,
    SUM(Salary) OVER(PARTITION BY Department) AS Dept_Total_Payroll
FROM Employees;
```
*Note: `PARTITION BY` is like `GROUP BY`, but for windows.*

---

## 7. Common Mistakes

*   **Confusing with GROUP BY:** If you use `GROUP BY`, you lose the individual rows. If you use `OVER()`, you keep them.
*   **Trying to filter by Window results in WHERE:** 
    ```sql
    -- ❌ ERROR
    SELECT Name, RANK() OVER(...) as rnk FROM Employees WHERE rnk = 1;
    ```
    **Fix:** You must wrap the query in a **CTE** or **Subquery** to filter by a window result.

---

## 8. Tips & Best Practices (Pro-Level)

**Window Function vs. CTEs:**
If you find yourself using the same window definition (`PARTITION BY... ORDER BY...`) multiple times in one query, MySQL 8.0 allows you to define a **Named Window** to keep your code clean:

```sql
SELECT 
    RANK() OVER w, 
    DENSE_RANK() OVER w
FROM Sales
WINDOW w AS (PARTITION BY Region ORDER BY Sales_Amount DESC);
```

---

## 9. Mini Practice Tasks

*   **Task 1:** What is the fundamental difference in output between an aggregate function used with `GROUP BY` vs. one used with `OVER()`?
*   **Task 2:** Write a query that shows the `Product_Name` and `Price`, along with a column showing the average price across all products.

---
