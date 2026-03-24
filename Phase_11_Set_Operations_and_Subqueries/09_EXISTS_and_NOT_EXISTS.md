# 🔦 Topic 11.9: EXISTS and NOT EXISTS

`EXISTS` is one of SQL's most elegant and efficient operators for answering a simple but powerful question: **"Does any matching row exist?"** Unlike `IN`, it doesn't retrieve data — it just checks if data is there.

---

## 1. Definition

- **`EXISTS(subquery)`:** Returns `TRUE` if the subquery returns **at least one row**, `FALSE` if zero rows are returned.
- **`NOT EXISTS(subquery)`:** The opposite — returns `TRUE` if the subquery returns **zero rows** (i.e., no matching data exists).

---

## 2. Why EXISTS Exists — What Problem it Solves

Sometimes you don't need data from the related table. You just need to know: "Is there anything there?"

- "Find customers **who have placed** at least one order." (EXISTS)
- "Find products **that have never been** sold." (NOT EXISTS)
- "Find employees **who have** submitted their performance review." (EXISTS)
- "Find departments **that have no** active employees." (NOT EXISTS)

`IN` builds a full value list and compares. `EXISTS` short-circuits as soon as one match is found — much faster for large datasets.

---

## 3. How It Works — Short-Circuit Evaluation (PRO LEVEL)

**`IN` approach:**
1. Execute the inner query → gets a list of all matching IDs: `(1, 5, 7, 23, 890, ...)`
2. For each outer row, scan the entire list to find a match.
3. Even if the first ID matches, it checks the rest of the list.

**`EXISTS` approach:**
1. For each outer row, execute the inner query.
2. **As soon as ONE matching row is found → immediately return `TRUE` and stop.**
3. Never builds a full list. Uses short-circuit evaluation.

```sql
-- EXISTS only checks for the presence of a row — it doesn't need actual data
-- So always use SELECT 1 (or SELECT NULL) — it communicates intent clearly
WHERE EXISTS (SELECT 1 FROM Orders o WHERE o.Customer_ID = c.Customer_ID)
--                    ^^^
--                    Convention: '1' signals "we don't need real column data"
```

---

## 4. Syntax / Implementation — Full Cheat Sheet

```sql
-- EXISTS: Find customers who have placed at least one order
SELECT c.Customer_Name, c.Email
FROM Customers c
WHERE EXISTS (
    SELECT 1 
    FROM Orders o 
    WHERE o.Customer_ID = c.Customer_ID
);

-- NOT EXISTS: Find customers with zero orders
SELECT c.Customer_Name, c.Email
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 
    FROM Orders o 
    WHERE o.Customer_ID = c.Customer_ID
);

-- EXISTS with additional filter in the subquery
SELECT p.Product_Name
FROM Products p
WHERE EXISTS (
    SELECT 1
    FROM Order_Items oi
    WHERE oi.Product_ID = p.Product_ID
    AND oi.Quantity > 10  -- Only products ordered in bulk at least once
);

-- NOT EXISTS: Find departments with no currently active employees
SELECT d.Dept_Name
FROM Departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM Employees e
    WHERE e.Dept_ID = d.Dept_ID
    AND e.Status = 'Active'
);
```

---

## 5. EXISTS vs. IN vs. NOT IN — Key Differences

| Feature | `IN` | `EXISTS` | `NOT IN` |
|---|---|---|---|
| **Returns** | TRUE if value is in a list | TRUE if any row exists | TRUE if value NOT in list |
| **NULL handling** | **Breaks** with NULLs in subquery | Safe — NULLs don't match | **Breaks** — returns no rows if NULLs in subquery |
| **Performance on large subqueries** | Slower (full list) | Faster (short-circuit) | Very slow (has to check ALL rows) |
| **Best for** | Small static lists | Existence checks on large tables | Use NOT EXISTS instead |

---

## 6. The NULL Trap — Why NOT IN Can Break

This is a critical interview topic:

```sql
-- If Orders table has even ONE row with Customer_ID = NULL:
SELECT Name FROM Customers 
WHERE Customer_ID NOT IN (SELECT Customer_ID FROM Orders);
-- ❌ Returns ZERO rows! 
-- Because NULL comparisons always return UNKNOWN (not TRUE or FALSE)
-- NOT IN (1, 5, NULL) → UNKNOWN for every comparison! 

-- ✅ SAFE alternative — NOT EXISTS handles nulls correctly:
SELECT Name FROM Customers c
WHERE NOT EXISTS (SELECT 1 FROM Orders o WHERE o.Customer_ID = c.Customer_ID);
-- Returns customers with no orders, correctly ignoring NULLs
```

---

## 7. Real-Life Examples

**Example 1 — Inactive Accounts (Never Logged In):**
```sql
SELECT u.Username, u.Email, u.Created_At
FROM Users u
WHERE NOT EXISTS (
    SELECT 1 FROM Login_History lh WHERE lh.User_ID = u.User_ID
)
AND DATEDIFF(NOW(), u.Created_At) > 30;
-- Users who registered 30+ days ago but have NEVER logged in
-- Perfect for re-engagement email campaigns
```

**Example 2 — Stock Replenishment Needed:**
```sql
-- Products that need restocking: sold recently but have low stock
SELECT p.Product_Name, p.Stock, p.Reorder_Level
FROM Products p
WHERE p.Stock <= p.Reorder_Level
AND EXISTS (
    SELECT 1 
    FROM Order_Items oi 
    JOIN Orders o ON oi.Order_ID = o.Order_ID
    WHERE oi.Product_ID = p.Product_ID
    AND o.Order_Date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
);
-- Find products that are low AND were actively sold in the last 30 days
```

**Example 3 — Employees Without Performance Reviews:**
```sql
SELECT e.Emp_ID, e.First_Name, e.Department, e.Manager_ID
FROM Employees e
WHERE e.Status = 'Active'
AND NOT EXISTS (
    SELECT 1
    FROM Performance_Reviews pr
    WHERE pr.Emp_ID = e.Emp_ID
    AND YEAR(pr.Review_Date) = YEAR(NOW())
);
-- Active employees who haven't had a review this year (for HR followup)
```

---

## 8. Common Mistakes

- **Forgetting the correlation link:** `WHERE EXISTS (SELECT 1 FROM Orders)` — without a link (e.g., `WHERE Orders.Customer_ID = Customers.Customer_ID`), the subquery either always returns TRUE (if Orders has any rows) or always FALSE (if empty). The link is what makes it meaningful.
- **Using `SELECT *` inside EXISTS:** Works, but `SELECT 1` is the professional convention — it clearly communicates "we care only about row existence, not column values."
- **Using `NOT IN` instead of `NOT EXISTS`:** The NULL trap (above) can silently return zero rows. Always prefer `NOT EXISTS` over `NOT IN` for subqueries.

---

## 9. Tips & Best Practices

- **Use `NOT EXISTS` instead of `NOT IN`** — always safer due to NULL handling.
- **Use `EXISTS` instead of `COUNT > 0`:** `WHERE (SELECT COUNT(*) FROM T2 WHERE T2.id = T1.id) > 0` scans ALL matching rows and counts them. `EXISTS` stops at the first match — much faster.
- **`EXISTS` is essentially a correlated subquery**, so index the correlated column in the inner table for maximum performance.

---

## 10. Mini Practice Tasks

- **Task 1:** Write an `EXISTS` query to find all `Departments` that have at least one employee with a salary above ₹80,000.
- **Task 2:** Explain the NULL trap with `NOT IN`. Write a scenario with sample data showing why `NOT IN` returns 0 rows when NULLs exist, and show the `NOT EXISTS` alternative that works correctly.
- **Task 3:** Rewrite this query using `EXISTS` instead of `IN`:  
  `SELECT Name FROM Customers WHERE Customer_ID IN (SELECT Customer_ID FROM Orders WHERE Status = 'Pending')`

---
