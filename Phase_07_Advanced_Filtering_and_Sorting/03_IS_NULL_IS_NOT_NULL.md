# ∅ Topic 7.3: IS NULL and IS NOT NULL

`NULL` is the most misunderstood value in all of SQL. It is not zero. It is not an empty string. It is not `FALSE`. It is the mathematical concept of **"Unknown"** — and it requires completely special operators to detect.

---

## 1. Definition

*   **`IS NULL`:** Returns `TRUE` if the column's value is missing/unknown (NULL). Returns `FALSE` for any actual value including zero, empty string, or `FALSE`.
*   **`IS NOT NULL`:** Returns `TRUE` if the column contains any actual value. Returns `FALSE` only for `NULL`.

---

## 2. Why NULL Is Special (The Three-Valued Logic System)

Standard Boolean logic has two states: **TRUE** and **FALSE**.

SQL operates on **Three-Valued Logic (3VL):** TRUE, FALSE, and **UNKNOWN**.

`NULL` participates as UNKNOWN in every comparison:
- `NULL = NULL` → UNKNOWN (not TRUE!)
- `NULL != NULL` → UNKNOWN
- `NULL > 5` → UNKNOWN
- `NULL + 5` → NULL (NULL is contagious in arithmetic!)

Because of 3VL, the `=` operator can **never** detect NULL. Only `IS NULL` can.

---

## 3. Why We Use It

*   **Missing data detection:** Find all customers who haven't provided a phone number (`WHERE Phone IS NULL`).
*   **Data quality audits:** Find any row with unexpected empty fields in critical columns.
*   **Conditional joins:** Outer joins (Phase 10) produce NULL for non-matching rows — `IS NULL` is used to identify those specifically.

---

## 4. How It Works (NULL in the B-Tree - PRO LEVEL)

In InnoDB, NULL values are stored differently than regular values:
- Each row has a **NULL bitmap** in its row header — a compact bit array flagging which columns are NULL.
- The B-Tree index for a nullable column **includes NULL entries** (stored as special markers at one end of the index).
- This means `WHERE column IS NULL` CAN use an index — MySQL scans the NULL-marker portion of the B-Tree instead of doing a full table scan.

---

## 5. Syntax / Implementation

```sql
-- Find rows where column is NULL
SELECT * FROM table WHERE column IS NULL;

-- Find rows where column has any value
SELECT * FROM table WHERE column IS NOT NULL;

-- NEVER DO THIS — it always returns nothing!
SELECT * FROM table WHERE column = NULL;   -- ❌ Always 0 rows
SELECT * FROM table WHERE column != NULL;  -- ❌ Always 0 rows
```

---

## 6. Real-Life Examples

**The HR Missing Data Audit:**
Before payroll runs, an HR system checks for employees with missing bank account details:
```sql
SELECT Emp_ID, First_Name, Last_Name
FROM Employees
WHERE Bank_Account_Number IS NULL
ORDER BY Last_Name;
```

**The Post-Join NULL Detection (Left Join pattern):**
Find all customers who have NEVER placed an order:
```sql
SELECT c.Customer_Name
FROM Customers c
LEFT JOIN Orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Order_ID IS NULL;  -- NULL means no matching order row exists
```

---

## 7. SQL Examples (MySQL Execution)

```sql
-- 1. Find employees with no assigned city
SELECT First_Name FROM Employees WHERE City IS NULL;

-- 2. Find employees who DO have a manager assigned
SELECT First_Name FROM Employees WHERE Manager_ID IS NOT NULL;

-- 3. Count how many employees are missing phone numbers
SELECT COUNT(*) AS Missing_Phones FROM Employees WHERE Phone IS NULL;

-- 4. IFNULL: Replace NULL with a default for display purposes
-- (Does not change stored data!)
SELECT First_Name, IFNULL(City, 'City Not Provided') AS City_Display
FROM Employees;
```

---

## 8. Common Mistakes

*   **`= NULL` and `!= NULL` never work:** As explained in Section 5, these always return zero rows due to 3VL. This is the most common beginner NULL bug.
*   **NULL in aggregate functions:** `COUNT(column)` ignores NULL values! `COUNT(Phone)` counts only non-NULL phones, while `COUNT(*)` counts all rows. This is a common source of subtle reporting errors.
    ```sql
    SELECT COUNT(*) AS Total_Employees,
           COUNT(Phone) AS Employees_With_Phone  -- Ignores NULLs!
    FROM Employees;
    ```

---

## 9. Tips & Best Practices (Pro-Level)

**The `COALESCE` Function — The Professional NULL Handler:**
`COALESCE(val1, val2, val3, ...)` returns the first non-NULL value from the list. It is the most versatile NULL-handling tool:

```sql
-- Display the first available contact method for each employee
SELECT First_Name,
       COALESCE(Mobile_Phone, Office_Phone, Email, 'No Contact Info') AS Contact
FROM Employees;
```

**Always consider NOT NULL design:** Every NULL in a database represents a question mark about your data. Design columns as `NOT NULL DEFAULT 'Unknown'` or `NOT NULL DEFAULT 0` where possible — it makes queries far simpler and avoids 3VL bugs entirely.

---

## 10. Mini Practice Tasks

*   **Task 1:** Write a query to find any employee in the `Employees` table whose `Department` column has not been filled in yet (is NULL).
*   **Task 2:** Why does `COUNT(Email)` return a different number than `COUNT(*)` when some rows have a NULL Email? What does each version actually count?

---
