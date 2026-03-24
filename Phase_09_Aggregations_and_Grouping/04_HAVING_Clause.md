# 🧼 Topic 9.4: The HAVING Clause (Filtering Aggregates)

You know how to filter rows using `WHERE`. But what if you want to find departments where the **total** salary is greater than $500,000? 

You might try: `WHERE SUM(Salary) > 500000`. 
**This will crash.** 

Why? Because the `WHERE` clause runs **before** the aggregation happens. We need a way to filter the results **after** the groups are calculated. That tool is **`HAVING`**.

---

## 1. Definition

**`HAVING`:** A filter clause applied after the `GROUP BY` phase. It allows you to filter the final "group rows" based on the results of aggregate functions.

---

## 2. Why This Concept Exists

Aggregate values (like SUMs and AVGs) don't exist until the `GROUP BY` is finished. Since SQL processes the `WHERE` clause first to decide which rows to include in the calculation, `WHERE` can never "see" the final result of a `SUM`. `HAVING` was created as the "WHERE for groups."

---

## 3. How It Works (Order of Execution - PRO LEVEL)

This is the single most important sequence to memorize in SQL. This is how the database engine actually executes your query:

1.  **FROM / JOIN:** Assemble the raw data.
2.  **WHERE:** Filter individual raw rows. (Discard unneeded rows).
3.  **GROUP BY:** Bundle the remaining rows into groups.
4.  **AGGREGATE:** Run SUM, AVG, COUNT, etc. on those groups.
5.  **HAVING:** Filter the resulting groups!
6.  **SELECT:** Pick which columns to show.
7.  **ORDER BY / LIMIT:** Sort and restrict the final output.

Because **Step 5 (HAVING)** happens after **Step 4 (AGGREGATE)**, it is the only place in the query where aggregate results can be filtered.

---

## 4. Syntax / Implementation

```sql
SELECT Department, SUM(Salary) AS Total_Pay
FROM Employees
GROUP BY Department
HAVING Total_Pay > 500000; -- Filters based on the SUM result
```

---

## 5. Real-Life Examples

**Example 1: Identifying "Big Spenders"**
Find customers who have placed more than 10 orders:
```sql
SELECT Customer_ID, COUNT(*) AS Order_Count
FROM Orders
GROUP BY Customer_ID
HAVING Order_Count > 10;
```

**Example 2: Managing Low Stock by Category**
Find categories where the average stock level is less than 5 units:
```sql
SELECT Category, AVG(Stock) AS Avg_Stock
FROM Products
GROUP BY Category
HAVING Avg_Stock < 5;
```

---

## 6. Common Mistakes

*   **Using HAVING when WHERE would be faster:**
    ```sql
    -- ❌ SLOW (Calculates SUM for ALL departments, then filters)
    SELECT Department, SUM(Salary) FROM Employees GROUP BY Department HAVING Department = 'Sales';
    
    -- ✅ FAST (Filters rows FIRST, then only runs math on Sales)
    SELECT Department, SUM(Salary) FROM Employees WHERE Department = 'Sales' GROUP BY Department;
    ```
    If you are filtering by a **regular column**, use `WHERE`. If you are filtering by an **aggregate result**, use `HAVING`.

---

## 7. Tips & Best Practices (Pro-Level)

**HAVING without GROUP BY:**
Interestingly, you can use `HAVING` without a `GROUP BY` clause. In this case, it treats the entire table as one single group.
```sql
-- Only returns data if the whole company headcount is > 100
SELECT COUNT(*) FROM Employees HAVING COUNT(*) > 100;
```

---

## 8. Mini Practice Tasks

*   **Task 1:** Write a query that shows all `Departments` from the `Employees` table that have more than 5 employees.
*   **Task 2:** Find all `Cities` where the average salary is greater than `75,000`.

---
