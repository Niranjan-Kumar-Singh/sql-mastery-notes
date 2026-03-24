# 🧩 Topic 14.1: Temporary Tables vs. CTEs (The WITH Clause)

As queries grow complex, you need to store intermediate results before doing the final computation. Two tools exist for this: **Temporary Tables** (the old way) and **Common Table Expressions (CTEs)** (the modern, preferred way). Knowing both — and when to use each — is essential.

---

## 1. Definitions

- **Temporary Table:** A real physical table created in the database that exists only for the duration of the current user session. It lives in the `tmp` directory on disk, can be indexed, and can be reused across multiple queries in the session.
- **CTE (Common Table Expression):** A named, temporary result set created using the `WITH` keyword, valid only for the duration of the single SQL statement it precedes.

---

## 2. Why They Exist

Without either tool, complex queries become deeply nested and unreadable:

```sql
-- ❌ Without CTE: Deeply nested, hard to debug
SELECT Name, Dept_Avg FROM
    (SELECT Name, Dept_ID, (SELECT AVG(Salary) FROM Employees e2 WHERE e2.Dept_ID = e.Dept_ID) AS Dept_Avg
     FROM Employees e) t
WHERE Salary > Dept_Avg;

-- ✅ With CTE: Clean, readable, named blocks
WITH Dept_Averages AS (
    SELECT Dept_ID, AVG(Salary) AS Dept_Avg FROM Employees GROUP BY Dept_ID
)
SELECT e.Name, da.Dept_Avg
FROM Employees e
JOIN Dept_Averages da ON e.Dept_ID = da.Dept_ID
WHERE e.Salary > da.Dept_Avg;
```

---

## 3. CTE vs. Temporary Table — Side-by-Side Comparison

| Feature | CTE | Temporary Table |
|---|---|---|
| **Keyword** | `WITH name AS (...)` | `CREATE TEMPORARY TABLE name ...` |
| **Scope** | Single SQL statement only | Entire session (until disconnect or DROP) |
| **Storage** | Memory (usually — may spill to disk) | Memory or disk (temp directory) |
| **Indexable** | ❌ Cannot add custom indexes | ✅ Can add indexes for performance |
| **Reusable** | Within the same WITH block only | Any subsequent query in the session |
| **Writeable** | ❌ Read-only | ✅ Can INSERT, UPDATE, DELETE |
| **Auto-cleanup** | ✅ Vanishes after the query ends | ❌ Must explicitly DROP or wait for session end |
| **Recursive** | ✅ Supports recursion via WITH RECURSIVE | ❌ Not natively recursive |
| **Syntax complexity** | Lower — no CREATE/DROP needed | Higher — must CREATE, use, then DROP |

---

## 4. CTE Syntax — Complete Cheat Sheet

```sql
-- Basic CTE
WITH Sales_2024 AS (
    SELECT Customer_ID, SUM(Amount) AS Total_Spent
    FROM Orders
    WHERE YEAR(Order_Date) = 2024
    GROUP BY Customer_ID
)
SELECT c.Customer_Name, s.Total_Spent
FROM Customers c
JOIN Sales_2024 s ON c.Customer_ID = s.Customer_ID
ORDER BY s.Total_Spent DESC;

-- Multiple CTEs in one WITH block (separate with commas)
WITH 
High_Earners AS (
    SELECT Emp_ID, Name, Salary, Dept_ID 
    FROM Employees 
    WHERE Salary > 100000
),
Dept_Info AS (
    SELECT Dept_ID, Dept_Name, Manager_ID 
    FROM Departments
)
SELECT h.Name, d.Dept_Name, h.Salary
FROM High_Earners h
JOIN Dept_Info d ON h.Dept_ID = d.Dept_ID;

-- Chained CTEs (Step 2 references Step 1)
WITH 
Step1 AS (
    SELECT Product_ID, SUM(Quantity) AS Total_Sold FROM Order_Items GROUP BY Product_ID
),
Step2 AS (
    SELECT p.Product_Name, s.Total_Sold, p.Price * s.Total_Sold AS Revenue
    FROM Products p
    JOIN Step1 s ON p.Product_ID = s.Product_ID
)
SELECT * FROM Step2 WHERE Revenue > 100000 ORDER BY Revenue DESC;
```

---

## 5. Temporary Table Syntax

```sql
-- Create a temporary table
CREATE TEMPORARY TABLE Temp_High_Value_Orders AS
SELECT * FROM Orders WHERE Total_Amount > 10000;

-- Add an index to the temp table (this is the main reason to prefer temp tables over CTEs for heavy processing)
ALTER TABLE Temp_High_Value_Orders ADD INDEX idx_customer (Customer_ID);

-- Use it like a normal table
SELECT c.Customer_Name, t.Total_Amount
FROM Customers c
JOIN Temp_High_Value_Orders t ON c.Customer_ID = t.Customer_ID;

-- Query again (CTEs can't do this — they vanish after one query)
SELECT COUNT(*) FROM Temp_High_Value_Orders WHERE Status = 'Shipping';

-- Clean up (optional — auto-cleaned on session disconnect)
DROP TEMPORARY TABLE IF EXISTS Temp_High_Value_Orders;
```

---

## 6. How CTEs Work Internally — Materialized vs. Non-Materialized (PRO LEVEL)

When MySQL encounters a CTE, it makes a decision:

**Non-Materialized (Merge):** The optimizer "inlines" the CTE into the main query as if you wrote a subquery. No temporary data is stored. This is the default for simple CTEs.

**Materialized:** MySQL stores the CTE result in a temporary memory buffer. This happens when:
- The CTE is referenced **multiple times** in the query.
- The CTE contains `DISTINCT`, `GROUP BY`, or aggregate functions.
- The optimizer determines that materialization is more efficient.

```sql
-- This CTE will likely be materialized (referenced twice)
WITH Monthly_Revenue AS (
    SELECT MONTH(Date) AS Mo, SUM(Amount) AS Total FROM Sales GROUP BY Mo
)
SELECT Mo, Total FROM Monthly_Revenue WHERE Total > 50000
UNION ALL
SELECT Mo, Total FROM Monthly_Revenue WHERE Mo <= 3;
-- MySQL materializes Monthly_Revenue once, then uses it for both queries
```

---

## 7. Real-Life Examples

**Example 1 — Executive Dashboard: Step-by-Step Data Processing:**
```sql
WITH 
-- Step 1: Calculate per-customer revenue
Customer_Revenue AS (
    SELECT Customer_ID, SUM(Total) AS Lifetime_Value
    FROM Orders WHERE Status = 'Completed'
    GROUP BY Customer_ID
),
-- Step 2: Segment customers by value
Customer_Segments AS (
    SELECT c.Customer_Name, cr.Lifetime_Value,
        CASE 
            WHEN cr.Lifetime_Value > 100000 THEN 'Platinum'
            WHEN cr.Lifetime_Value > 50000 THEN 'Gold'
            WHEN cr.Lifetime_Value > 10000 THEN 'Silver'
            ELSE 'Bronze'
        END AS Tier
    FROM Customers c
    JOIN Customer_Revenue cr ON c.Customer_ID = cr.Customer_ID
)
-- Step 3: Final report
SELECT Tier, COUNT(*) AS Customer_Count, AVG(Lifetime_Value) AS Avg_Value
FROM Customer_Segments
GROUP BY Tier
ORDER BY Avg_Value DESC;
```

**Example 2 — When Temp Table is Better (Complex Batch Processing):**
```sql
-- Process 100k transactions in multiple stages, each needing indexes
CREATE TEMPORARY TABLE Flagged_Transactions AS
SELECT * FROM Transactions WHERE Amount > 100000 OR Country NOT IN ('IN', 'US', 'UK');

-- Now index for fast lookups in subsequent queries
ALTER TABLE Flagged_Transactions ADD INDEX idx_user (User_ID);

-- Query 1: By user
SELECT User_ID, COUNT(*) FROM Flagged_Transactions GROUP BY User_ID;
-- Query 2: By country
SELECT Country, SUM(Amount) FROM Flagged_Transactions GROUP BY Country;
-- Both queries reuse the same indexed temp table — much faster than two CTEs!
```

---

## 8. Common Mistakes

- **Trying to use a CTE after the semicolon:** `WITH CTE AS (...) SELECT ... ; SELECT * FROM CTE` — the second SELECT fails. A CTE is only valid for the statement it precedes.
- **Forgetting commas between multiple CTEs:** `WITH A AS (...) B AS (...)` is a syntax error. Use `WITH A AS (...), B AS (...)`.
- **Using a Temp Table when a CTE would do:** Temp tables require CREATE and DROP, add complexity, and leave behind temp data if your application crashes mid-execution.
- **Not indexing Temp Tables:** If you use a Temp Table for performance but don't add indexes, it may be slower than a CTE.

---

## 9. Tips & Best Practices

- **Default to CTEs** for complex queries — they're self-cleaning, readable, and require no setup/teardown.
- **Use Temporary Tables** when: (a) you need to run multiple different queries against the same intermediate result, (b) you need custom indexes on the intermediate result, or (c) the intermediate result set is very large and referenced many times.
- **Name your CTEs descriptively:** `Step1` is bad. `Customer_Revenue_2024` is good. The CTE name appears in the query; make it self-documenting.

---

## 10. Mini Practice Tasks

- **Task 1:** Write a query using TWO CTEs: (a) `Top_Products` (products with price > ₹1000), and (b) `Product_Sales_Summary` (total quantity sold per product). Then join them to show sales stats for top products only.
- **Task 2:** When is a `TEMPORARY TABLE` a better choice than a CTE? Give two specific scenarios.
- **Task 3:** If a CTE is referenced 3 times in the same query, does MySQL execute the CTE 3 times, or does it cache the result? What is this caching behavior called?

---
