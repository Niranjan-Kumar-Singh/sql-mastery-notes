# 🎯 Topic 11.5: Subqueries in the WHERE Clause

The most common place to use a subquery is the `WHERE` clause. It allows you to build a dynamic filter where the criteria isn't a fixed number, but the result of another search.

---

## 1. Two Main Categories

### A. Single-Row Subqueries
Used with standard operators (`=`, `<>`, `>`, `<`, `>=`, `<=`).
**Condition:** The inner query MUST return exactly **one row and one column**.

```sql
-- Find employees hired after 'Alice'
SELECT First_Name FROM Employees 
WHERE Hire_Date > (SELECT Hire_Date FROM Employees WHERE First_Name = 'Alice');
```

### B. Multiple-Row Subqueries
Used when the inner query can return more than one result.
**Must use:** `IN`, `ANY`, `ALL`, or `EXISTS`.

```sql
-- Find employees in departments located in 'Mumbai'
SELECT First_Name FROM Employees
WHERE Dept_ID IN (SELECT Dept_ID FROM Departments WHERE City = 'Mumbai');
```

---

## 2. Why This Concept Exists

In many systems, the information you filter by is a moving target. You can't hardcode `WHERE Dept_ID = 5` because the ID for "Mumbai Engineering" might change. A subquery locks onto the *logic* (Find the ID for Mumbai) and uses that to filter the Employees.

---

## 3. How It Works (Semi-Join Optimization - PRO LEVEL)

When you use `WHERE column IN (Subquery)`, modern MySQL doesn't always run the inner query first. The Optimizer often rewrites the query into a **Semi-Join**.

Instead of: "Run Query 1, then use results for Query 2"
It does: "Join Table A and Table B, but stop as soon as a match is found (don't create duplicate rows)."

This optimization makes `WHERE IN` subqueries nearly as fast as `INNER JOINs` in many scenarios.

---

## 4. Syntax / Implementation

```sql
-- Single-row subquery
SELECT * FROM Products 
WHERE Price > (SELECT AVG(Price) FROM Products);

-- Multiple-row subquery
SELECT * FROM Customers
WHERE Customer_ID NOT IN (SELECT DISTINCT Customer_ID FROM Orders);
```

---

## 5. Real-Life Examples

**The Top Earner Report:**
"Find the employee with the maximum salary in each department." (This requires a Correlated subquery or a JOIN, but for the overall max it's simple):
```sql
SELECT Name, Salary
FROM Employees
WHERE Salary = (SELECT MIN(Salary) FROM Employees);
```

---

## 6. Common Mistakes

*   **Subquery returning multiple rows for a single-row operator:**
    `WHERE ID = (SELECT ID FROM ...)`. If the inner query returns 2 IDs, the whole query crashes with "Subquery returns more than 1 row". 
    **Fix:** Use `IN` instead of `=` whenever there's a chance of multiple matches.

---

## 7. Tips & Best Practices (Pro-Level)

**The JOIN vs. Subquery Debate:**
Most `WHERE IN` subqueries can be rewritten as `INNER JOINs`.
- **Joins** are generally preferred for clarity and performance when you need columns from *both* tables.
- **Subqueries** are cleaner when you only need data from the *outer* table and the inner query is just a "membership check."

---

## 8. Mini Practice Tasks

*   **Task 1:** Write a query to find all employees whose `Department` is the same as the department of the employee with `Emp_ID = 101`.
*   **Task 2:** Why is it safer to use `IN` rather than `=` when using a subquery in the `WHERE` clause?

---
