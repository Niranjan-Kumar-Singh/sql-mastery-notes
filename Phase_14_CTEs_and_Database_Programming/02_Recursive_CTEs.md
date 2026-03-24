# 🔄 Topic 14.2: Recursive CTEs

Standard SQL is "Iterative." But some data structures—like family trees, organizational charts, or file directories—are "Recursive." To query these, we need a query that can **call itself**. This is a **Recursive CTE**.

---

## 1. Definition

**Recursive CTE:** A CTE that references itself. It consists of two parts:
1.  **The Anchor Member:** The starting point (e.g., the CEO of a company).
2.  **The Recursive Member:** The logic that finds the child rows based on the previous row found (e.g., finding the managers reporting to the CEO).

---

## 2. Why This Concept Exists

*   **Org Charts:** Printing a list of all employees and their levels (CEO=1, VP=2, Manager=3).
*   **Path Finding:** Finding all stops in a flight itinerary.
*   **Hierarchy Expansion:** Flattening a "Folder > Subfolder > File" structure.

---

## 3. How It Works (The Union Loop - PRO LEVEL)

The engine executes `WITH RECURSIVE` in these steps:
1.  Execute the **Anchor Member**. This becomes the "Current Result."
2.  Execute the **Recursive Member** using the "Current Result" as an input.
3.  Append the new matches to the final result set.
4.  **Repeat Step 2-3** using the *new* matches as the next "Current Result."
5.  Stop when no new matches are found.

**The Infinite Loop Guard:** By default, MySQL limits recursion depth to 1000 iterations. This prevents a bug in your SQL from crashing the entire server.

---

## 4. Syntax / Implementation

```sql
WITH RECURSIVE Hierarchy AS (
    -- Anchor: Find the Boss
    SELECT Emp_ID, Name, Manager_ID, 1 AS Level
    FROM Employees WHERE Manager_ID IS NULL
    
    UNION ALL
    
    -- Recursive: Find everyone reporting to the previous level
    SELECT e.Emp_ID, e.Name, e.Manager_ID, h.Level + 1
    FROM Employees e
    INNER JOIN Hierarchy h ON e.Manager_ID = h.Emp_ID
)
SELECT * FROM Hierarchy;
```

---

## 5. Real-Life Examples

**Generating a Number Series (1 to 10):**
```sql
WITH RECURSIVE Numbers AS (
    SELECT 1 AS n -- Start at 1
    UNION ALL
    SELECT n + 1 FROM Numbers WHERE n < 10 -- Increment until 10
)
SELECT * FROM Numbers;
```

---

## 6. Common Mistakes

*   **Forgetting `RECURSIVE`:** If you try to self-reference a CTE without the `RECURSIVE` keyword, MySQL will return an error.
*   **Leaving out the Termination condition:** Without a `WHERE` clause in the recursive part (like `n < 10`), the query will run forever until it hits the `max_execution_time`.

---

## 7. Tips & Best Practices (Pro-Level)

**Path Accumulation:**
You can build a "Breadcrumb" string during recursion to show the full path to a row:
```sql
-- Recursive part:
SELECT ..., CONCAT(h.Path, ' > ', e.Name) FROM ...
```
This is how Professional Databases generate "Category Navigation" paths (Electronics > Laptops > Macbooks).

---

## 8. Mini Practice Tasks

*   **Task 1:** Identify the two main parts of a Recursive CTE and explain their roles.
*   **Task 2:** Write a recursive CTE that prints numbers from `10` down to `1`.

---
