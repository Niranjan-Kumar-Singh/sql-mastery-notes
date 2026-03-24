# 🥞 Topic 11.2: UNION vs. UNION ALL

Two of the most practical Set Operations in SQL. Both combine result sets from multiple `SELECT` statements into one list — but they differ in a fundamental way that has major performance implications.

---

## 1. Definition

- **`UNION`:** Combines the rows from two or more `SELECT` statements into a single result set and **removes all duplicate rows**. Slower but gives unique results.
- **`UNION ALL`:** Combines rows and **keeps all duplicates** exactly as they appear. Faster and gives a complete combined list.

---

## 2. The Critical Rule — Column Compatibility

Both `SELECT` statements must have:
- The **same number of columns**
- Columns in the **same order** with compatible data types
- **Column names** are taken from the first `SELECT` only

---

## 3. How It Works — Performance Difference (PRO LEVEL)

This is a vital performance distinction:

**`UNION ALL` (Fast — Streaming):**
1. Runs Query 1, streams results directly to the output buffer.
2. Runs Query 2, appends results to the same buffer.
3. Done. No extra processing.

**`UNION` (Slower — Blocking):**
1. Runs Query 1, stores all rows in a temporary table.
2. Runs Query 2, adds rows to the same temporary table.
3. **Performs a full Sort + Deduplicate pass** on the entire temp table.
4. Returns the deduplicated result.

**Rule:** Use `UNION ALL` whenever you know the two result sets don't overlap. Use `UNION` only when duplicate removal is genuinely required.

---

## 4. Syntax / Implementation — Full Cheat Sheet

```sql
-- UNION: Removes duplicate rows
SELECT City FROM Office_Locations
UNION
SELECT City FROM Customer_Addresses;
-- Each city appears only once, even if found in both tables.

-- UNION ALL: Keeps all rows including duplicates
SELECT City FROM Office_Locations
UNION ALL
SELECT City FROM Customer_Addresses;
-- Mumbai appears twice if it's in both tables.

-- Adding a source label (highly recommended for diagnostics!)
SELECT Name, Email, 'Staff' AS Record_Type FROM Employees
UNION ALL
SELECT Name, Email, 'Client' AS Record_Type FROM Customers;

-- Sorting the final combined result (ORDER BY goes at the very end only)
SELECT Product_Name, Price FROM Electronics
UNION ALL
SELECT Product_Name, Price FROM Furniture
ORDER BY Price ASC;
```

---

## 5. Real-Life Examples

**Example 1 — Master Contact List (Marketing):**
```sql
-- Create a unified mailing list from Customers + Suppliers
SELECT Name, Email, Phone, 'Customer' AS Type FROM Customers
UNION
SELECT Name, Email, Phone, 'Supplier' AS Type FROM Suppliers
ORDER BY Name;
-- UNION used because a person might appear in both tables (should appear once)
```

**Example 2 — Transaction History (Accounting):**
```sql
-- ALL credit card transactions from this month's processing batches
SELECT Transaction_ID, Amount, Transaction_Date FROM Batch_Morning
UNION ALL
SELECT Transaction_ID, Amount, Transaction_Date FROM Batch_Evening
UNION ALL
SELECT Transaction_ID, Amount, Transaction_Date FROM Batch_Night
ORDER BY Transaction_Date;
-- UNION ALL used: each batch is independent, no duplicates possible
```

**Example 3 — Multi-Source Search:**
```sql
-- Search results from both blog posts and product names
SELECT 'Blog' AS Source, Title AS Result, Created_At AS Date 
FROM Blog_Posts 
WHERE Title LIKE '%machine learning%'

UNION ALL

SELECT 'Product' AS Source, Product_Name AS Result, Added_On AS Date 
FROM Products 
WHERE Product_Name LIKE '%machine learning%'

ORDER BY Date DESC;
```

---

## 6. Common Mistakes

- **Using UNION when UNION ALL is sufficient:** The silent performance killer. If you're combining `Male_Customers` and `Female_Customers` (by definition, no overlap), using `UNION` wastes time sorting and deduplicating a result that already has no duplicates.
- **Wrong column count:** `SELECT Name FROM T1 UNION SELECT Name, Email FROM T2` fails immediately — column counts must match.
- **Mismatched column meaning:** `SELECT First_Name FROM Employees UNION SELECT Product_Name FROM Products` — technically valid SQL, but the resulting column labeled "First_Name" contains product names. Semantically nonsensical.
- **Adding ORDER BY in the middle:** `SELECT ... ORDER BY col UNION SELECT ...` is a syntax error. The ORDER BY must come at the very end.

---

## 7. Tips & Best Practices

- **Always label your sources** with an extra column like `'Staff' AS Source` — it makes combined results self-explanatory.
- **Use `UNION ALL` by default**, then add `UNION` only if you actually need deduplication. Never use `UNION` "just to be safe."
- **For 3+ sources**, you can chain: `Q1 UNION ALL Q2 UNION ALL Q3 UNION ALL Q4` — all handled in one pass.

---

## 8. Mini Practice Tasks

- **Task 1:** You are merging two transaction tables — `Transactions_2023` and `Transactions_2024`. Every transaction has a unique `Transaction_ID`. Which operator should you use and why — `UNION` or `UNION ALL`?
- **Task 2:** Rewrite this query to add a `'Source'` column showing `'Electronics'` or `'Furniture'` for each row: `SELECT Product_Name FROM Electronics UNION ALL SELECT Product_Name FROM Furniture`
- **Task 3:** Why is `UNION` slower than `UNION ALL`? Describe the internal extra step that `UNION` performs.

---
