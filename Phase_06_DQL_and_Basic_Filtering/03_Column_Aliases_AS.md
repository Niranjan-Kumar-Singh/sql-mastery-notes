# 🏷️ Topic 6.3: Column Aliases (The AS Keyword)

When MySQL returns a result set, column headers default to the raw database column names like `First_Name`, `Salary`, or `COUNT(*)`. These names are great for internal engineering, but an API or report might need clean, human-readable, or differently named headers.

The **`AS` keyword** (Column Alias) solves this elegantly.

---

## 1. Definition

**Column Alias (`AS`):** A temporary, display-only rename applied to a column or expression in a `SELECT` result. The alias exists only within the scope of that single query's result set — it does not change the actual column name in the table.

---

## 2. Why This Concept Exists

Raw database column names often follow snake_case conventions (`first_name`, `emp_salary_usd`) which are internal conventions. API consumers, business reports, and frontend JavaScript code may need different naming (camelCase `firstName`, or full labels like `Employee Salary`). Aliases bridge this gap without altering the schema.

---

## 3. Why We Use It

*   **Readable Reports:** `SELECT COUNT(*) AS Total_Employees` produces a header called "Total_Employees" instead of the cryptic `COUNT(*)`.
*   **Expression Labels:** When computing derived values like `Salary * 1.10`, the resulting column has no natural name. An alias gives it one.
*   **API Contract:** A MySQL column named `usr_pw_hash` can be aliased to `passwordHash` to match a strict JSON API contract without renaming the column.

---

## 4. When to Use It

*   When using **aggregate functions** (`COUNT`, `SUM`, `AVG`): they produce unnamed columns without an alias.
*   When computing **mathematical expressions**.
*   When the column name must match a **specific API or report format**.

---

## 5. How It Works (Parse-Time Resolution - PRO LEVEL)

There is a critical limitation of aliases that senior engineers know:

**Aliases cannot be reused in the same `WHERE` clause.** This is because MySQL parses the `WHERE` clause *before* it evaluates the `SELECT` columns. The alias doesn't "exist" yet at the time `WHERE` is parsed.

```sql
-- ❌ This CRASHES! MySQL doesn't know 'Annual_Salary' yet when evaluating WHERE
SELECT Salary * 12 AS Annual_Salary FROM Employees WHERE Annual_Salary > 60000;

-- ✅ This WORKS: Repeat the expression in WHERE instead
SELECT Salary * 12 AS Annual_Salary FROM Employees WHERE Salary * 12 > 60000;

-- ✅ OR use a subquery/CTE (the cleanest solution)
SELECT * FROM (
    SELECT Salary * 12 AS Annual_Salary, First_Name FROM Employees
) AS Derived
WHERE Annual_Salary > 60000;
```

---

## 6. Syntax / Implementation

```sql
-- Basic alias
SELECT column_name AS alias_name FROM table_name;

-- Alias with spaces (must use quotes)
SELECT Salary AS 'Monthly Salary (USD)' FROM Employees;

-- The 'AS' keyword is actually optional (but always use it for clarity!)
SELECT First_Name fname FROM Employees; -- Works but bad practice
SELECT First_Name AS fname FROM Employees; -- Clear and professional
```

---

## 7. Real-Life Examples

**The HR Payroll Report:**
```sql
SELECT 
    CONCAT(First_Name, ' ', Last_Name) AS Full_Name,
    Department AS Team,
    Salary AS Monthly_Pay,
    Salary * 12 AS Annual_Pay,
    Salary * 12 * 0.30 AS Estimated_Tax
FROM Employees
WHERE Department = 'Engineering';
```

This report is instantly readable by a business manager who has never seen raw SQL column names.

---

## 8. SQL Examples (MySQL Execution)

```sql
-- 1. Aliasing a computed expression
SELECT 
    First_Name,
    Salary,
    Salary * 1.15 AS Salary_After_15_Percent_Raise
FROM Employees;

-- 2. Aliasing aggregate functions (No alias = ugly column header 'COUNT(*)')
SELECT 
    Department,
    COUNT(*) AS Total_Employees,
    AVG(Salary) AS Average_Salary,
    MAX(Salary) AS Highest_Salary
FROM Employees
GROUP BY Department;

-- 3. Aliasing for API JSON compatibility
SELECT 
    First_Name AS firstName,
    Last_Name AS lastName,
    Hire_Date AS hireDate
FROM Employees
WHERE Emp_ID = 42;
```

---

## 9. Common Mistakes

*   **Using alias in WHERE (Causes Error):** As explained in Section 5, the `WHERE` clause is evaluated before `SELECT` aliases are resolved. Always repeat the expression or use a subquery.
*   **Forgetting quotes for aliases with spaces:** `AS Full Name` will crash — MySQL sees two tokens. Always wrap multi-word aliases in single quotes or backticks: `` AS `Full Name` `` or `AS 'Full Name'`.

---

## 10. Tips & Best Practices (Pro-Level)

**`ORDER BY` CAN use aliases (unlike `WHERE`):**
Because `ORDER BY` is evaluated after `SELECT`, aliases are fully visible there:
```sql
SELECT Salary * 12 AS Annual_Salary, First_Name
FROM Employees
ORDER BY Annual_Salary DESC;  -- ✅ This works perfectly!
```

This is extremely useful for sorting by computed values without repeating verbose expressions.

---

## 11. Mini Practice Tasks

*   **Task 1:** Write a SELECT query that retrieves `First_Name`, `Last_Name`, and `Salary`. Alias `Salary` as `'Monthly Gross Pay'`. Then add a computed column `Salary * 12` aliased as `Annual_Gross_Pay`.
*   **Task 2:** Why does `WHERE Annual_Salary > 50000` fail when `Annual_Salary` is defined as an alias in the same query's SELECT clause?

---
