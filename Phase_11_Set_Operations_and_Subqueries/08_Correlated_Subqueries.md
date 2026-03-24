# 🔗 Topic 11.8: Correlated Subqueries

A regular subquery is **self-contained** — it runs once and its result is fed to the outer query. A correlated subquery is fundamentally different — it **references a column from the outer query**, forcing it to re-execute for every single row the outer query processes. Powerful, but potentially slow.

---

## 1. Definition

**Correlated Subquery:** A subquery that uses one or more columns from the **outer query** in its own `WHERE` clause. Because of this dependency, the inner query cannot be run independently — it must be re-evaluated for every row of the outer query.

---

## 2. Why This Concept Exists

Regular subqueries answer questions about the whole dataset:
> "Who earns more than the **company-wide** average salary?"

Correlated subqueries answer row-specific questions:
> "Who earns more than the average salary **of their own department**?"

To answer this, for every employee row being processed, the subquery must "look up" that employee's specific department and calculate just that department's average. It needs a value from the outer row to do its job.

---

## 3. The Identifying Feature

A correlated subquery references a table or alias from the **outer** query. Look for the alias crossing the subquery boundary:

```sql
SELECT e1.Name, e1.Salary, e1.Dept_ID
FROM Employees e1
WHERE e1.Salary > (
    SELECT AVG(e2.Salary)         -- Inner query
    FROM Employees e2
    WHERE e2.Dept_ID = e1.Dept_ID  -- ← e1 is from the OUTER query! This is the correlation!
);
```

---

## 4. How It Works — The Nested Loop (PRO LEVEL)

This is a **nested loop — O(N²) complexity**:

```
For each row in Employees (outer loop):
    Execute: SELECT AVG(Salary) FROM Employees WHERE Dept_ID = [outer row's Dept_ID]
    Return result to outer query filter
    If outer salary > inner AVG → include this row
```

If Employees has 10,000 rows, the inner subquery runs **10,000 times**. For a large table, this is extremely slow.

**Senior engineer insight:** Most correlated subqueries can be rewritten as:
1. **Window Functions** (Phase 13) — single pass, dramatically faster
2. **JOIN to a derived table** — calculate all averages once, then join

```sql
-- ✅ Correlated Subquery (runs AVG 10,000 times):
SELECT Name FROM Employees e1
WHERE Salary > (SELECT AVG(Salary) FROM Employees e2 WHERE e2.Dept_ID = e1.Dept_ID);

-- ✅ Window Function equivalent (runs AVG ONCE):
SELECT Name FROM (
    SELECT Name, Salary, AVG(Salary) OVER(PARTITION BY Dept_ID) AS Dept_Avg
    FROM Employees
) t WHERE Salary > Dept_Avg;
```

---

## 5. Syntax / Implementation

```sql
-- Pattern 1: WHERE column > (correlated aggregate)
SELECT e1.Name, e1.Salary, e1.Dept_ID
FROM Employees e1
WHERE e1.Salary > (
    SELECT AVG(e2.Salary)
    FROM Employees e2
    WHERE e2.Dept_ID = e1.Dept_ID
);

-- Pattern 2: WHERE column = (correlated max/min)
SELECT Order_ID, Customer_ID, Order_Date
FROM Orders o1
WHERE Order_Date = (
    SELECT MAX(o2.Order_Date)
    FROM Orders o2
    WHERE o2.Customer_ID = o1.Customer_ID  -- For this specific customer's orders
);
-- Finds the most recent order for each customer.
```

---

## 6. Real-Life Examples

**Example 1 — Find Employees Earning Above Their Department Average:**
```sql
SELECT 
    e.Name,
    e.Department,
    e.Salary,
    (SELECT AVG(e2.Salary) FROM Employees e2 WHERE e2.Department = e.Department) AS Dept_Avg
FROM Employees e
WHERE e.Salary > (
    SELECT AVG(e3.Salary) FROM Employees e3 WHERE e3.Department = e.Department
);
```

**Example 2 — Find Each Customer's Most Expensive Order:**
```sql
SELECT c.Customer_Name, o.Order_ID, o.Total_Amount
FROM Customers c
INNER JOIN Orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Total_Amount = (
    SELECT MAX(o2.Total_Amount)
    FROM Orders o2
    WHERE o2.Customer_ID = c.Customer_ID  -- For THIS customer specifically
);
```

**Example 3 — Products with Price Above Their Category Average:**
```sql
SELECT p.Product_Name, p.Category_ID, p.Price
FROM Products p
WHERE p.Price > (
    SELECT AVG(p2.Price)
    FROM Products p2
    WHERE p2.Category_ID = p.Category_ID
)
ORDER BY p.Category_ID, p.Price DESC;
```

---

## 7. Common Mistakes

- **Logic error — forgetting the correlation:** If you write the subquery without linking it to the outer row (`WHERE e2.Dept_ID = e1.Dept_ID`), the inner query calculates the global average for all employees — turning a correlated subquery into a regular one and giving completely wrong results.
- **Performance blindness on large tables:** Running a correlated subquery on a 500,000-row table without an index on the correlated column (`Dept_ID` in the inner query) causes 500,000 full table scans — potentially minutes of execution time.

```sql
-- Fix: Index the correlated column in the inner table
CREATE INDEX idx_dept ON Employees(Dept_ID);
```

---

## 8. Tips & Best Practices

- **Use correlated subqueries for "row-specific" comparisons** where the comparison value changes per row.
- **Always check EXPLAIN** before using a correlated subquery in production — look at the `rows` estimate multiplied by the outer table's row count. That's the total work MySQL will do.
- **Prefer Window Functions** when available — they achieve the same result in a single pass instead of N passes:
    - Correlated subquery: O(N²) ← N outer rows × N inner rows
    - Window Function: O(N log N) ← single sort pass

---

## 9. Mini Practice Tasks

- **Task 1:** Write a correlated subquery to find all `Products` whose `Price` is higher than the average price within their `Category_ID`.
- **Task 2:** Rewrite this correlated subquery using a Window Function: `SELECT Name FROM Employees e WHERE Salary > (SELECT AVG(Salary) FROM Employees WHERE Dept_ID = e.Dept_ID)`
- **Task 3:** A correlated subquery on a 100,000-row table runs the inner query 100,000 times. What is the total maximum number of row-reads, and why is this an O(N²) operation?

---
