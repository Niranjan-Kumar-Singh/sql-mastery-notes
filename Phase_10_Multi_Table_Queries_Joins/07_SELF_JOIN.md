# 🔄 Topic 10.7: SELF JOIN (Joining a Table to Itself)

A join usually connects two **different** tables. But sometimes, the relationship you need exists between rows **within the same table**. An employee's manager is also an employee. A reply to a forum post is also a forum post. This is where **SELF JOIN** comes in.

---

## 1. Definition

**SELF JOIN:** A join where a table is joined with **itself**. To tell the two "copies" apart, the table must be given **two different aliases** in the query.

---

## 2. Why This Concept Exists

Some data is **hierarchical** or **peer-relational** and lives in a single table:
- Employees and their managers (both in `Employees` table)
- Products and their related products (both in `Products` table)
- Forum replies and their parent posts (both in `Posts` table)
- Users referred by other users (both in `Users` table)

Without SELF JOIN, you'd need two separate copies of the table, which is impractical.

---

## 3. How It Works — The Twin Alias Logic (PRO LEVEL)

MySQL treats the query as if two separate copies of the table exist in memory:

```
Employee Table "e1" (as the Employee view)
Employee Table "e2" (as the Manager view)
```

```sql
FROM Employees e1          -- "e1" represents each employee
JOIN Employees e2           -- "e2" represents their manager (same table)
ON e1.Manager_ID = e2.Emp_ID  -- Link: employee's Manager_ID = manager's Emp_ID
```

MySQL internally runs a Nested Loop Join — for each row in `e1`, it looks up the matching row in `e2`.

---

## 4. Syntax / Implementation — Full Cheat Sheet

```sql
-- 1. Employee → Manager (INNER JOIN: excludes CEO who has no manager)
SELECT 
    emp.Name AS Employee_Name,
    mgr.Name AS Manager_Name,
    emp.Department
FROM Employees emp
INNER JOIN Employees mgr ON emp.Manager_ID = mgr.Emp_ID;

-- 2. Employee → Manager (LEFT JOIN: includes CEO with NULL manager)
SELECT 
    emp.Name AS Employee_Name,
    COALESCE(mgr.Name, 'No Manager (CEO)') AS Manager_Name
FROM Employees emp
LEFT JOIN Employees mgr ON emp.Manager_ID = mgr.Emp_ID;

-- 3. Find all peers (employees in the same city)
SELECT 
    e1.Name AS Person1, 
    e2.Name AS Person2, 
    e1.City
FROM Employees e1
INNER JOIN Employees e2 
    ON e1.City = e2.City 
    AND e1.Emp_ID < e2.Emp_ID;  -- Prevents (Alice,Bob) AND (Bob,Alice) duplicates
```

---

## 5. Real-Life Examples

**Example 1 — Organization Chart:**
```sql
SELECT 
    emp.Emp_ID,
    emp.Name AS Employee,
    emp.Job_Title,
    COALESCE(mgr.Name, '— Top of Org —') AS Reports_To
FROM Employees emp
LEFT JOIN Employees mgr ON emp.Manager_ID = mgr.Emp_ID
ORDER BY mgr.Name, emp.Name;
```

**Example 2 — Find Referral Pairs (Users who referred other users):**
```sql
SELECT 
    referrer.Username AS Referrer,
    referred.Username AS New_User,
    referred.Signup_Date
FROM Users referred
INNER JOIN Users referrer ON referred.Referred_By = referrer.User_ID
ORDER BY referred.Signup_Date DESC;
```

**Example 3 — Find Products Priced the Same:**
```sql
SELECT 
    p1.Product_Name AS Product_A,
    p2.Product_Name AS Product_B,
    p1.Price
FROM Products p1
INNER JOIN Products p2 
    ON p1.Price = p2.Price 
    AND p1.Product_ID < p2.Product_ID  -- Only unique pairs, no self-match
ORDER BY p1.Price;
```

---

## 6. Common Mistakes

- **Forgetting aliases — fatal error:** `SELECT Name FROM Employees JOIN Employees` causes "Not unique table/alias" error immediately. You MUST provide two different aliases.
- **Duplicate pair problem:** `ON e1.City = e2.City` without the `e1.ID < e2.ID` filter produces `(Alice, Bob)` AND `(Bob, Alice)` as separate rows. Use `<` or `!=` to control pairing.
- **Not using LEFT JOIN for top-level records:** If the CEO has no manager (`Manager_ID = NULL`), INNER JOIN excludes them. Use LEFT JOIN when hierarchy root records need to appear.

---

## 7. Tips & Best Practices

- **Alias names communicate intent:** Use `emp` and `mgr`, not `a` and `b`. Descriptive aliases make self-join queries much easier to understand at a glance.
- **For deep hierarchies:** A SELF JOIN only shows one level up (employee → manager). For multi-level hierarchy (employee → manager → director → VP), use **Recursive CTEs** (Topic 14.2). Self-joining 5 times for 5 levels is messy and brittle.

---

## 8. Mini Practice Tasks

- **Task 1:** Write a SELF JOIN query on an `Employees(Emp_ID, Name, Manager_ID)` table to list every employee's name alongside their manager's name. Include employees with no manager (e.g., CEO) using an appropriate JOIN type.
- **Task 2:** In the SELF JOIN for finding "same-city pairs," why do we use `e1.Emp_ID < e2.Emp_ID` in the `ON` condition?
- **Task 3:** A `Categories` table has `Category_ID` and `Parent_Category_ID` (for sub-categories). Write a SELF JOIN to list each sub-category alongside its parent category name.

---
