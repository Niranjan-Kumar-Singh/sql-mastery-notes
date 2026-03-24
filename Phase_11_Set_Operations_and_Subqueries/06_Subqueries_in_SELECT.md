# 📊 Topic 11.6: Subqueries in the SELECT Clause

Sometimes you need to add a single calculated value to every row of your result set. For example, next to every employee's salary, you want to show the **company-wide average salary** for comparison.

This is where a **Subquery in the SELECT clause** is used.

---

## 1. Definition

**Scalar Subquery in SELECT:** A nested query placed in the column list of the `SELECT` statement. It must return exactly **one row and one column** (a scalar value) for every row in the outer query.

---

## 2. Why This Concept Exists

*   **Comparison Columns:** "My Salary vs The Average."
*   **Running Counts:** "Product Name vs Total Sales of that product."
*   **Business Logic:** "Show total orders for this customer as a column in the customers table."

---

## 3. How It Works (Row-by-Row Execution - PRO LEVEL)

A subquery in the `SELECT` clause behaves like a **Loop**:
1.  MySQL executes the outer query.
2.  For **every row** in the outer result set, it pauses and runs the inner query.
3.  The result of the inner query is "pasted" as a value for that row.
4.  Move to the next row.

**Performance DANGER:** If your outer query has 1 million rows, and you have a subquery in the `SELECT` clause, MySQL will execute that subquery **1 million times**. 

**Senior Tip:** Only use subqueries in `SELECT` for small result sets. For large ones, always use a **JOIN** or a **Window Function** (Phase 13).

---

## 4. Syntax / Implementation

```sql
SELECT 
    First_Name, 
    Salary,
    (SELECT AVG(Salary) FROM Employees) AS Average_Company_Salary
FROM Employees;
```

---

## 5. Real-Life Examples

**The "Market Share" Column:**
Find every brand and its total sales, plus the total sales of ALL brands in one view.
```sql
SELECT 
    Brand_Name, 
    SUM(Sales_Amount) AS Brand_Total,
    (SELECT SUM(Sales_Amount) FROM Sales) AS Global_Total
FROM Sales
GROUP BY Brand_Name;
```

---

## 6. Common Mistakes

*   **Subquery returning multiple rows:** If the scalar subquery returns 2 rows (e.g., if you forget a filter), the whole query fails with: "Subquery returns more than 1 row."
*   **Forgetting Aliases:** Always give the subquery column a name using `AS Alias_Name`, otherwise the header in your report will be the literal SQL string of the subquery!

---

## 7. Tips & Best Practices (Pro-Level)

**Subqueries vs. LEFT JOIN for Counts:**
A subquery is often easier to write than a Join when you just want a simple count:
```sql
-- Subquery Version
SELECT 
    Name, 
    (SELECT COUNT(*) FROM Orders o WHERE o.Cust_ID = c.ID) AS OrderCount
FROM Customers c;

-- Join Version (Requires GROUP BY)
SELECT c.Name, COUNT(o.ID)
FROM Customers c
LEFT JOIN Orders o ON c.ID = o.Cust_ID
GROUP BY c.ID;
```
For small datasets, most developers prefer the subquery for its simplicity.

---

## 8. Mini Practice Tasks

*   **Task 1:** Write a query that shows the `Product_Name` and its `Price`, alongside a third column showing the price of the **cheapest** product in the entire catalog.
*   **Task 2:** Why is a subquery in the `SELECT` clause potentially slower than a regular Join?

---
