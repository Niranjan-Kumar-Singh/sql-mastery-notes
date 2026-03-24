# ⛓️ Topic 10.9: Joining 3 or More Tables

Real-world queries rarely involve just two tables. A complete order invoice needs `Customers`, `Orders`, `Order_Items`, and `Products` — that's 4 tables. A hospital report might join 6 or 7 tables. Chaining joins together is one of the most essential skills in SQL.

---

## 1. Definition

**Multi-Table Join:** A query that chains multiple `JOIN` clauses, where each link connects a new table to a table already in the chain. Each new join typically uses a Foreign Key from a previously joined table.

---

## 2. Why This Concept Exists

Proper normalization splits data into focused tables (Phase 16). An `Orders` table doesn't store the customer's name or the product's description — it only stores IDs that reference other tables. To rebuild a meaningful result (like an invoice), you must traverse the relationship chain:

```
Orders → Customers (for customer name)
      → Order_Items (for what was ordered)
               → Products (for product details)
```

---

## 3. How It Works — Sequential Join Execution (PRO LEVEL)

MySQL doesn't join all 4 tables at once. It executes joins **sequentially**:

1. `Customers JOIN Orders` → Temporary Result 1
2. `Temp1 JOIN Order_Items` → Temporary Result 2
3. `Temp2 JOIN Products` → Final Result

**But** MySQL's Query Optimizer may **reorder** this sequence! It calculates the cost of each join order and picks the one that minimises intermediate result set sizes — this is called **Join Order Optimization**. You can force an order using `STRAIGHT_JOIN` if needed.

```sql
-- Check which join order MySQL actually chose:
EXPLAIN SELECT c.Name, p.Product_Name
FROM Customers c
JOIN Orders o ON c.Customer_ID = o.Customer_ID
JOIN Order_Items oi ON o.Order_ID = oi.Order_ID
JOIN Products p ON oi.Product_ID = p.Product_ID;
-- Look at the 'id' and 'table' columns to see the actual join execution order.
```

---

## 4. Syntax / Implementation — The Chain Pattern

```sql
-- Full order invoice (4 tables)
SELECT 
    c.Customer_Name,
    c.Email,
    o.Order_ID,
    o.Order_Date,
    p.Product_Name,
    oi.Quantity,
    oi.Quantity * p.Price AS Line_Total,
    o.Status
FROM Customers c
JOIN Orders o       ON c.Customer_ID = o.Customer_ID   -- Link 1: Customer → Order
JOIN Order_Items oi ON o.Order_ID = oi.Order_ID         -- Link 2: Order → Item
JOIN Products p     ON oi.Product_ID = p.Product_ID     -- Link 3: Item → Product
WHERE o.Order_ID = 10452
ORDER BY p.Product_Name;
```

---

## 5. Mixing JOIN Types in a Chain

You can mix INNER JOINs and LEFT JOINs in the same query — each join is independent:

```sql
SELECT 
    e.First_Name,
    d.Dept_Name,
    b.Building_Name,
    pr.Review_Score
FROM Employees e
INNER JOIN Departments d ON e.Dept_ID = d.Dept_ID         -- Must have a dept
LEFT JOIN Buildings b ON d.Building_ID = b.Building_ID    -- Dept might not have a building
LEFT JOIN Performance_Reviews pr ON e.Emp_ID = pr.Emp_ID  -- Employee might have no review yet
WHERE e.Status = 'Active';
```

---

## 6. Real-Life Examples

**Example 1 — Hospital Patient Invoice (5 tables):**
```sql
SELECT 
    p.Patient_Name,
    d.Doctor_Name,
    t.Treatment_Name,
    t.Cost,
    w.Ward_Name,
    ins.Insurance_Provider
FROM Patients p
INNER JOIN Doctors d        ON p.Doctor_ID = d.Doctor_ID
INNER JOIN Treatments t     ON p.Treatment_ID = t.Treatment_ID
INNER JOIN Wards w          ON p.Ward_ID = w.Ward_ID
LEFT JOIN Insurance ins     ON p.Insurance_ID = ins.Insurance_ID;
```

**Example 2 — School Report Card (4 tables):**
```sql
SELECT 
    s.Student_Name,
    sub.Subject_Name,
    t.Teacher_Name,
    g.Marks_Obtained,
    g.Grade
FROM Students s
INNER JOIN Grades g         ON s.Student_ID = g.Student_ID
INNER JOIN Subjects sub     ON g.Subject_ID = sub.Subject_ID
INNER JOIN Teachers t       ON sub.Teacher_ID = t.Teacher_ID
ORDER BY s.Student_Name, sub.Subject_Name;
```

**Example 3 — E-commerce (Product + Supplier + Category):**
```sql
SELECT 
    p.Product_Name,
    cat.Category_Name,
    sup.Supplier_Name,
    sup.Country,
    p.Stock,
    p.Price
FROM Products p
INNER JOIN Categories cat   ON p.Category_ID = cat.Category_ID
INNER JOIN Suppliers sup    ON p.Supplier_ID = sup.Supplier_ID
WHERE p.Stock < 10
ORDER BY p.Stock ASC;
-- Low-stock products with their category and supplier info for reordering
```

---

## 7. Common Mistakes

- **The INNER JOIN trap in a chain:** If you have 5 INNER JOINs and the last table has no match for a particular row, that entire row disappears from the result — even if the first 4 joins were successful. Use `LEFT JOIN` for optional relationships.
- **Missing indexes on all FK columns:** In a 4-table join, if even ONE join column lacks an index, performance degrades exponentially. Always index every Foreign Key.
- **Unclear aliases:** In a 5-table join, using single letters (`a`, `b`, `c`, `d`, `e`) makes the query unreadable. Use meaningful abbreviations (`cust`, `ord`, `item`, `prod`, `sup`).

---

## 8. Tips & Best Practices

- **Format multi-table joins vertically:** Align the `JOIN` keywords and `ON` conditions in a column for readability (see examples above).
- **Run EXPLAIN on any multi-table query** before it goes to production — look for `type: ALL` which signals a missing index.
- **Add one join at a time when debugging:** If results look wrong, temporarily remove the last join and check if the preceding result is correct. Isolate the problem join.

---

## 9. Mini Practice Tasks

- **Task 1:** Write a 3-table JOIN connecting `Books(Book_ID, Title, Author_ID, Publisher_ID)`, `Authors(Author_ID, Author_Name)`, and `Publishers(Publisher_ID, Publisher_Name)`. Display the title, author name, and publisher name.
- **Task 2:** In a 4-table INNER JOIN chain, if the 3rd table has no matching row, what happens to the entire result row? How would you fix this to keep partial results?
- **Task 3:** Why does MySQL's Query Optimizer sometimes execute a 4-table join in a different order than you wrote the `FROM ... JOIN ... JOIN ... JOIN` chain?

---
