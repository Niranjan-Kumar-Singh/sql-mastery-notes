# ↔️ Topic 10.5: FULL OUTER JOIN (& How to Simulate it in MySQL)

When you need **everything from both tables** — all records from the left, all records from the right, and matches combined — you need a `FULL OUTER JOIN`. It's the union of LEFT JOIN and RIGHT JOIN.

---

## 1. Definition

**FULL OUTER JOIN:** Returns all rows from **both** the left and right tables.
- Rows that match → combined normally.
- Rows from the left with no match on the right → right-side columns are `NULL`.
- Rows from the right with no match on the left → left-side columns are `NULL`.

---

## 2. Why This Concept Exists

Perfect for **data reconciliation** — situations where you have two lists and want to see everything from both:
- "Compare orders from System A and System B — show everything, highlight differences."
- "List all employees and all managers — show who isn't matched to either side."
- "Merge two product catalogs from two companies after a merger."

---

## 3. The MySQL Limitation (Critical!)

**MySQL does NOT support `FULL OUTER JOIN` natively.**

If you type `FROM T1 FULL OUTER JOIN T2`, MySQL returns a syntax error. This surprises developers moving from PostgreSQL or SQL Server, which do support it.

**Workaround:** Combine a `LEFT JOIN` and a `RIGHT JOIN` using `UNION`.

---

## 4. How It Works — The UNION Simulation (PRO LEVEL)

```
LEFT JOIN result:  [All from A] + [matches from B]
RIGHT JOIN result: [matches from A] + [All from B]
UNION:             Combines both, removes duplicates that appear in both
= FULL OUTER JOIN
```

The key is using `UNION` (not `UNION ALL`) to remove the duplicate matched rows that appear in both LEFT and RIGHT JOIN results.

---

## 5. Syntax / Implementation — MySQL Simulation

```sql
-- Step 1: LEFT JOIN (all from A, matching from B)
SELECT c.Customer_Name, o.Order_ID, o.Total_Amount
FROM Customers c
LEFT JOIN Orders o ON c.Customer_ID = o.Customer_ID

UNION

-- Step 2: RIGHT JOIN (matching from A, all from B)
SELECT c.Customer_Name, o.Order_ID, o.Total_Amount
FROM Customers c
RIGHT JOIN Orders o ON c.Customer_ID = o.Customer_ID;

-- UNION removes rows present in both halves (the matched rows)
-- leaving only the previously unmatched rows from each side
```

---

## 6. Real-Life Examples

**Example 1 — Company Merger: Reconcile Two Client Lists:**
```sql
-- Find ALL clients from both companies.
-- Show which clients are shared, which are exclusive to Company A, which to Company B.
SELECT 
    a.Company_Name AS Client_in_A, 
    b.Company_Name AS Client_in_B
FROM Clients_CompanyA a
LEFT JOIN Clients_CompanyB b ON a.Tax_ID = b.Tax_ID

UNION

SELECT 
    a.Company_Name AS Client_in_A, 
    b.Company_Name AS Client_in_B
FROM Clients_CompanyA a
RIGHT JOIN Clients_CompanyB b ON a.Tax_ID = b.Tax_ID;
-- Result: Shared clients show both names. Exclusive clients show NULL on one side.
```

**Example 2 — Sync Check: Database vs Spreadsheet:**
```sql
-- Find all products either in the database OR in an imported spreadsheet,
-- flagging which exist only in one source (data quality issue)
SELECT db.SKU AS DB_SKU, xl.SKU AS Excel_SKU
FROM Products_DB db
LEFT JOIN Products_Excel xl ON db.SKU = xl.SKU

UNION

SELECT db.SKU AS DB_SKU, xl.SKU AS Excel_SKU
FROM Products_DB db
RIGHT JOIN Products_Excel xl ON db.SKU = xl.SKU;
```

---

## 7. Common Mistakes

- **Using `UNION ALL` instead of `UNION`:** This will produce **duplicate rows** for all matched records (once from the LEFT result, once from the RIGHT result). You MUST use `UNION` (with deduplication).
- **Thinking MySQL 8.0 added FULL OUTER JOIN:** It still hasn't been added — even in MySQL 9.x. The `UNION` simulation is the only way.
- **Forgetting to order the SELECT columns identically:** Both `SELECT` statements in the `UNION` must have the same number of columns in the same order. Column names are taken from the first `SELECT`.

---

## 8. Tips & Best Practices

- **Add a source column** to identify which side a record came from:
```sql
SELECT c.Name, o.Order_ID, 'Customer Only' AS Status
FROM Customers c LEFT JOIN Orders o ON c.ID = o.Cust_ID WHERE o.ID IS NULL
UNION
SELECT c.Name, o.Order_ID, 'Order Only' AS Status
FROM Customers c RIGHT JOIN Orders o ON c.ID = o.Cust_ID WHERE c.ID IS NULL
UNION
SELECT c.Name, o.Order_ID, 'Both' AS Status
FROM Customers c INNER JOIN Orders o ON c.ID = o.Cust_ID;
```
- **Full join is expensive** — it runs two join operations + a deduplication sort. Use it only for reconciliation tasks, not for regular application queries.

---

## 9. Mini Practice Tasks

- **Task 1:** Write a MySQL FULL OUTER JOIN simulation between `Teachers(Teacher_ID, Name)` and `Classes(Class_ID, Teacher_ID, Class_Name)`. Show all teachers and all classes, even unmatched ones.
- **Task 2:** Why does the UNION simulation require `UNION` (not `UNION ALL`) to correctly simulate a FULL OUTER JOIN?
- **Task 3:** A developer writes `FROM A FULL OUTER JOIN B ON A.id = B.id` in MySQL. What error do they get, and what should they write instead?

---
