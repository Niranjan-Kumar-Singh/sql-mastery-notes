# 📦 Topic 11.4: Introduction to Subqueries

Up until now, we have written "Flat" queries. Now we start nesting. A **Subquery** is simply a query inside another query. It is the SQL equivalent of a "callback function" or a "nested loop."

---

## 1. Definition

**Subquery (Inner Query):** A `SELECT` statement that is nested inside another statement (called the Outer Query). The inner query executes first and provides its result to the outer query.

---

## 2. Why This Concept Exists

Sometimes the data you need for a filter depends on another search. 
- **The Question:** "Who earned more than the average salary?"
- **The Problem:** You don't know the average salary until you query it first.
- **The Solution:** A subquery calculates the `AVG(Salary)`, and the outer query uses that number to filter employees.

---

## 3. Position and Terminology

Subqueries can be placed in three main areas:
1.  **WHERE Clause:** Filter rows based on the result of another query.
2.  **SELECT Clause:** Generate a dynamic column value for every row.
3.  **FROM Clause:** Treat the result of a query as if it were a temporary table (Derived Table).

---

## 4. How It Works (The Execution Pipeline - PRO LEVEL)

Generally, MySQL executes subqueries in a **Bottom-Up** fashion:
1.  The **Inner Query** runs once.
2.  Its result set (a value, a list, or a table) is stored in memory.
3.  The **Outer Query** runs, using that stored result to evaluate its conditions.

*Note: For "Correlated Subqueries" (Topic 11.8), this changes to a Top-Down approach.*

---

## 5. Basic Syntax Template

```sql
SELECT column_name
FROM table_name
WHERE column_name OPERATOR (SELECT column_name FROM table_name);
--                          ^--- Subquery is always inside (Parentheses)
```

---

## 6. Real-Life Examples

**"Find the most expensive product":**
```sql
SELECT Product_Name, Price
FROM Products
WHERE Price = (SELECT MAX(Price) FROM Products);
-- The inner query finds (e.g.) 5000. 
-- Then outer query becomes: WHERE Price = 5000.
```

---

## 7. Common Mistakes

*   **Forgetting Parentheses:** Subqueries **must** be wrapped in `(...)`.
*   **Single-row vs. Multi-row mismatch:**
    ```sql
    -- ❌ ERROR if the subquery returns more than 1 row!
    WHERE Salary = (SELECT Salary FROM Employees WHERE Dept = 'Sales');
    ```
    If a subquery can return multiple rows, you **must** use `IN`, `ANY`, or `ALL` instead of `=`.

---

## 8. Tips & Best Practices (Pro-Level)

**Formatting for Readability:**
Always indent your subqueries. A flat query is easy to read, but nested queries become "Sphaghetti SQL" very quickly if not properly formatted.

```sql
SELECT Name
FROM Users
WHERE User_ID IN (
    SELECT User_ID 
    FROM Logins 
    WHERE Login_Date = CURDATE()
);
```

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that finds all employees whose `Salary` is greater than the overall `AVG(Salary)` of the company.
*   **Task 2:** What is the difference between an "Inner Query" and an "Outer Query"?

---
