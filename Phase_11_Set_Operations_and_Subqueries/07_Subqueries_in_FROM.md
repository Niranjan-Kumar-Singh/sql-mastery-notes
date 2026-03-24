# 🏗️ Topic 11.7: Subqueries in the FROM Clause (Derived Tables)

In standard SQL, you join tables. But what if you need to join a table with the **result** of another query? This is a **Derived Table**.

---

## 1. Definition

**Derived Table:** A subquery that appears in the `FROM` clause of an SQL statement. It is a temporary "virtual table" that exists only for the duration of the query.

---

## 2. Why This Concept Exists

*   **Aggregation Before Joining:** Sometimes you need to `SUM` or `COUNT` data before you join it to another table to avoid "Double Counting" (The fan-out problem).
*   **Multi-Step Calculations:** You need to calculate an average, then use that average to calculate a standard deviation, etc.

---

## 3. The Strict Rule: Aliasing (CRITICAL!)

In MySQL, **every derived table MUST have its own alias.** 

```sql
-- ❌ ERROR: "Every derived table must have its own alias"
SELECT * FROM (SELECT * FROM Users);

-- ✅ CORRECT:
SELECT * FROM (SELECT * FROM Users) AS temp_users;
```

---

## 4. How It Works (Materialization - PRO LEVEL)

When MySQL sees a subquery in the `FROM` clause:
1.  It executes the subquery first.
2.  It creates an internal, temporary **Materialized Table** (often in RAM, but spills to disk if it's large).
3.  The outer query then treats this temporary table exactly like a regular table on your disk.

**Performance Tip:** Since a Derived Table is temporary, it usually **does not have indexes**. Joining a large derived table can be very slow. In MySQL 8.0, the optimizer tries to "merge" the subquery into the outer query to avoid materialization whenever possible.

---

## 5. Syntax / Implementation

```sql
SELECT dt.Avg_Salary
FROM (SELECT AVG(Salary) AS Avg_Salary FROM Employees) AS dt;
-- Here, 'dt' is the name we give the virtual result of the subquery.
```

---

## 6. Real-Life Examples

**Combining Totals with Details:**
Show every customer, and next to them, the average order value across the entire company.
```sql
SELECT c.Name, stats.Avg_Val
FROM Customers c
CROSS JOIN (
    SELECT AVG(Total_Amount) AS Avg_Val FROM Orders
) AS stats;
```

---

## 7. Common Mistakes

*   **Missing the Alias:** This is the #1 error beginners make with derived tables.
*   **Ambiguous Column Names:** If your derived table selects `ID` and your outer table has `ID`, you must use the alias to specify: `dt.ID` vs `outer.ID`.

---

## 8. Tips & Best Practices (Pro-Level)

**Derived Tables vs. CTEs:**
A derived table is powerful but can be hard to read because they are "Inside Out" (the inner query is in the middle of the `FROM` clause). 
**Professional developers** prefer **Common Table Expressions (CTEs)** (Topic 14.1) which allow you to define the subquery at the *top* of the query, making it much more readable.

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query where you select from a derived table that calculates the `MIN` and `MAX` salary of the entire company. Give your derived table the alias `Sal_Stats`.
*   **Task 2:** What is "Materialization" in the context of a derived table?

---
