# 📊 Topic 13.6: Aggregate Window Functions

In Phase 9, we used `SUM()` and `AVG()` to collapse data. Now, we use them as **Window Functions** to perform cumulative math over time.

---

## 1. Definition

**Aggregate Window Functions:** Standard aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) used with an `OVER()` clause. 

---

## 2. Why This Concept Exists

*   **Running Totals:** "Show my daily sales and the cumulative total of sales so far this month."
*   **Moving Averages:** "Show the 7-day average of stock prices to smooth out volatility."
*   **Contextual Ratios:** "Show this employee's salary divided by the department's total."

---

## 3. How It Works (The Sliding Accumulator - PRO LEVEL)

When `ORDER BY` is present inside `OVER()`, an aggregate function becomes "Incremental":
1.  MySQL starts at Row 1. The SUM is Row 1.
2.  Move to Row 2. The SUM is Row 1 + Row 2.
3.  Move to Row 3. The SUM is Previous SUM + Row 3.

**Result:** You get a **Running Total**.

---

## 4. Syntax / Implementation

```sql
-- Running Total of Sales
SELECT 
    Date, 
    Sales_Amount,
    SUM(Sales_Amount) OVER(ORDER BY Date) AS Cumulative_Sales
FROM Daily_Sales;

-- Departmental Running Total
SELECT 
    Dept, 
    Name, 
    Salary,
    SUM(Salary) OVER(PARTITION BY Dept ORDER BY Hire_Date) AS Dept_Accumulated_Payroll
FROM Employees;
```

---

## 5. Real-Life Examples

**The 3-Day Moving Average:**
```sql
SELECT 
    Date, 
    Price,
    AVG(Price) OVER(ORDER BY Date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS 3Day_Avg
FROM Stock_Prices;
```

---

## 6. Common Mistakes

*   **Forgetting Partition:** If you are calculating Running Totals for Customers, and forget `PARTITION BY Customer_ID`, the totals will "bleed" from one customer into the next one.
*   **Logical Duplicates:** If two dates are identical, `SUM(...) OVER(ORDER BY Date)` will sum **both** rows into the total for the first date. Use a unique tie-breaker (like `ID`) in your `ORDER BY` for precise increments.

---

## 7. Tips & Best Practices (Pro-Level)

**Window Aggregates vs. Self Joins:**
In older SQL, to get a running total, you had to join Table A to Table B where `B.Date <= A.Date`. 
On a table of 10,000 rows, this Join creates **50 Million comparison rows**. 
On that same table, a Window Function performs a single sort and a single pass. The performance improvement is **exponential**.

---

## 8. Mini Practice Tasks

*   **Task 1:** Write a query that calculates a running total of `Expenses` for each `Project_ID`, ordered by `Transaction_Date`.
*   **Task 2:** How do you calculate a moving average using the `AVG()` function as a window function?

---
