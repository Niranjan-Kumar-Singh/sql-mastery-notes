# 📊 Topic 9.1: Aggregate Functions (COUNT, SUM, AVG, MIN, MAX)

So far, we have been retrieving individual rows. But high-level data analysis requires **Summarization**. You don't want to see 10,000 transaction rows; you want to see the `SUM` of all transactions. You don't want to see every employee's salary; you want the `AVG` salary.

**Aggregate Functions** turn thousands of rows into a single meaningful number.

---

## 1. Definition

**Aggregate Functions:** Functions that perform a calculation on a **set** of values (a column across multiple rows) and return a single scalar value. 

---

## 2. The Core Aggregate Functions Reference

| Function | Description | Example |
|---|---|---|
| `COUNT()` | Returns number of items/rows | `COUNT(*)` |
| `SUM()` | Returns total sum of numeric values | `SUM(Salary)` |
| `AVG()` | Returns mathematical average | `AVG(Price)` |
| `MIN()` | Returns smallest value | `MIN(Hire_Date)` |
| `MAX()` | Returns largest value | `MAX(Experience_Years)` |

---

## 3. Why This Concept Exists

Without aggregation, a database is just a storage locker. With aggregation, a database becomes a **Business Intelligence** engine. It allows for the instantaneous generation of reports, dashboards, and KPI metrics (Key Performance Indicators) that drive company decisions.

---

## 4. How It Works (The Accumulator Pattern - PRO LEVEL)

When MySQL runs an aggregate function, it doesn't try to load all rows into memory at once. It uses an **Accumulator Pattern**:

1.  MySQL initializes an internal memory variable (the Accumulator) to zero or NULL.
2.  It iterates through each row in the result set.
3.  For each row, it updates the Accumulator (e.g., adds to the sum, or updates if the new value is smaller for MIN).
4.  Once all rows are processed, it returns the final value of the Accumulator.

**Performance Tip:** Aggregate functions like `MIN` and `MAX` on an indexed column are O(1) — MySQL simply looks at the first or last entry in the B-Tree index without reading any data rows!

---

## 5. Syntax / Implementation

```sql
SELECT 
    COUNT(*) AS Total_Employees,
    SUM(Salary) AS Total_Payroll,
    AVG(Salary) AS Average_Salary,
    MIN(Salary) AS Lowest_Salary,
    MAX(Salary) AS Highest_Salary
FROM Employees;
```

---

## 6. Real-Life Examples

**The E-commerce Dashboard:**
```sql
SELECT 
    COUNT(Order_ID) AS Orders_Today,
    SUM(Total_Amount) AS Daily_Revenue,
    MIN(Total_Amount) AS Smallest_Order,
    AVG(Total_Amount) AS Order_Value_Average
FROM Orders
WHERE Order_Date = CURDATE();
```

---

## 7. Common Mistakes

*   **Mixing Aggregate and Non-Aggregate columns:**
    ```sql
    -- ❌ WRONG (Which First_Name should MySQL show for the average salary?)
    SELECT First_Name, AVG(Salary) FROM Employees;
    ```
    If you select an aggregate function, every other column in the `SELECT` clause MUST either be an aggregate function or be part of a `GROUP BY` clause (Topic 9.3).
*   **Assuming `AVG` ignores zeros:** `AVG` includes 0 in the calculation. If you want to exclude 0s, you must use a `WHERE` clause or turn 0s into NULLs (Topic 9.2).

---

## 8. Tips & Best Practices (Pro-Level)

**`COUNT(DISTINCT column)`:**
You can combine aggregation with `DISTINCT` to count unique occurrences:
```sql
-- Find how many UNIQUE cities our employees come from
SELECT COUNT(DISTINCT City) FROM Employees;
```

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that finds the highest and lowest `Hire_Date` in the `Employees` table.
*   **Task 2:** Calculate the average `Salary` of all employees in the `'Engineering'` department.

---
