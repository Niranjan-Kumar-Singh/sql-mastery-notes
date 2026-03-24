# 🖼️ Topic 12.1: Views (Virtual Tables)

In a complex database, you might have a query that joins 5 tables and has 10 filters. Typing that every time is exhausting and error-prone. **Views** let you save that query as a **reusable "virtual table"** — query it like a normal table, and it always shows fresh data.

---

## 1. Definition

**View:** A stored, named SQL query that can be treated exactly like a table. It does **not** physically store data itself — it is a "window" that re-runs its underlying query against the base tables every time you query it.

---

## 2. Why Views Exist

- **Simplicity:** Instead of explaining a complex 30-line JOIN to a junior dev, you say: `SELECT * FROM sales_summary`. 
- **Security:** You can give a user access to a View showing `Name` and `Department`, while hiding `Salary`, `Password`, and `SSN` from the real `Employees` table — without changing any permissions on the base table.
- **Consistency:** If the business rule for "Total Profit" changes, you update it in one View, and every report company-wide is automatically updated.
- **Reusability:** One View can be used by multiple applications, dashboards, and API endpoints.

---

## 3. How It Works (Query Merging — PRO LEVEL)

When you run `SELECT * FROM My_View`, MySQL does NOT retrieve the "view's data" and then filter it. Instead:

1. It takes your query.
2. It retrieves the View's stored SQL definition.
3. It **merges** them into a single optimized execution plan.
4. That combined plan is run against the real base tables.

**Result:** A View is always **up-to-date**, because it re-queries the real tables every time. If a new row is added to `Employees`, the View sees it immediately.

---

## 4. Syntax / Implementation — Full Cheat Sheet

```sql
-- Create a View
CREATE VIEW Active_Senior_Employees AS
SELECT Emp_ID, First_Name, Last_Name, Salary, Hire_Date
FROM Employees
WHERE Status = 'Active' AND Salary > 100000;

-- Use it exactly like a table
SELECT * FROM Active_Senior_Employees;
SELECT First_Name FROM Active_Senior_Employees WHERE First_Name LIKE 'A%';

-- CREATE OR REPLACE: Update definition without DROP first (safer!)
CREATE OR REPLACE VIEW Active_Senior_Employees AS
SELECT Emp_ID, First_Name, Last_Name, Salary, Department, Hire_Date
FROM Employees
WHERE Status = 'Active' AND Salary > 120000;

-- ALTER VIEW (same as CREATE OR REPLACE)
ALTER VIEW Active_Senior_Employees AS
SELECT Emp_ID, First_Name FROM Employees WHERE Status = 'Active';

-- See the VIEW's definition (how it was created)
SHOW CREATE VIEW Active_Senior_Employees;

-- See all views in the current database
SHOW FULL TABLES WHERE Table_Type = 'VIEW';

-- Delete a View (does NOT delete the underlying table data)
DROP VIEW IF EXISTS Active_Senior_Employees;
```

---

## 5. Real-Life Examples

**Example 1 — Security Masking:**
```sql
-- Internal employees table (has sensitive data)
-- View for public API — exposes only non-sensitive columns
CREATE VIEW Public_User_Profiles AS
SELECT User_ID, Username, Bio, Join_Date, Avatar_URL
FROM Users;
-- Passwords, Emails, Phone numbers are completely hidden!
```

**Example 2 — Reusable Complex JOIN:**
```sql
-- This JOIN is needed in 5 different reports. Save it once as a View.
CREATE VIEW Order_Details_View AS
SELECT 
    o.Order_ID,
    c.Customer_Name,
    p.Product_Name,
    oi.Quantity,
    oi.Quantity * p.Price AS Line_Total,
    o.Order_Date
FROM Orders o
JOIN Customers c  ON o.Customer_ID = c.Customer_ID
JOIN Order_Items oi ON o.Order_ID = oi.Order_ID
JOIN Products p   ON oi.Product_ID = p.Product_ID;

-- Now all reports just do:
SELECT * FROM Order_Details_View WHERE Order_Date >= '2024-01-01';
```

---

## 6. Common Mistakes

- **Thinking a View stores data:** A View is just a saved query. It has no data of its own. If you `DROP` the underlying table, the View breaks.
- **Views inside Views:** Technically possible, but chaining Views (View A queries View B which queries View C) can confuse the optimizer and hurt performance.
- **Using Views as a performance solution:** Views do NOT cache results. If the underlying query was slow, the View will be equally slow. For caching, use a Materialized View pattern (a physical table updated by a Trigger or scheduled job).

---

## 7. Tips & Best Practices

- **Name Views clearly:** Use suffixes like `_view` or `_vw` (e.g., `active_orders_vw`) so developers know immediately they're querying a view, not a base table.
- **Use `CREATE OR REPLACE`** instead of `DROP VIEW + CREATE VIEW` — it's atomic and won't leave you with a missing view if something goes wrong.
- **Security tip:** In production, grant application users access only to Views, never directly to base tables. This forces all data access through controlled interfaces.

---

## 8. Mini Practice Tasks

- **Task 1:** Create a View called `Product_Catalog` that returns the `Product_Name` and `Price` for all products currently in stock (`Stock > 0`). Then write a query to use the view to find products cheaper than ₹500.
- **Task 2:** What is the difference between `CREATE VIEW` and `CREATE OR REPLACE VIEW`? When would you use each?
- **Task 3:** Can you `DELETE` rows from a View? Under what conditions? (Hint: Think about Updatable Views in Topic 12.2.)

---
