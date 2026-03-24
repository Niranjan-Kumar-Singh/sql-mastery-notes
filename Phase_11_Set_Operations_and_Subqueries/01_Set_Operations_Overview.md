# 📚 Topic 11.1: Set Operations Overview

In Phase 10, we learned Joins — which combine tables **horizontally** (adding columns). In Phase 11, we learn Set Operations — which combine result sets **vertically** (adding rows).

---

## 1. Definition

**Set Operations:** Mathematical operations that take two or more `SELECT` result sets and combine them into a single result set based on set theory principles (Union, Intersection, and Difference).

---

## 2. The 3 Rules of Set Compatibility (CRITICAL!)

Unlike a Join, which can connect any two tables, Set Operations require the result sets to be **Compatible**. If these three rules aren't met, MySQL will throw an error:

1.  **Same Number of Columns:** Both `SELECT` statements must return the exact same number of columns.
2.  **Compatible Data Types:** The columns in the same position must have similar data types (e.g., you can't combine a `DATE` column with an `INT` column).
3.  **Same Order:** Even if names are different, the *meaning* of the data in each position should be the same for the result to make sense.

---

## 3. The Core Set Operators

| Operator | Concept | MySQL Support |
|---|---|---|
| **UNION** | Combines rows and removes duplicates | ✅ Native |
| **UNION ALL** | Combines rows, keeps duplicates | ✅ Native |
| **INTERSECT** | Rows common to both sets | ❌ Simulated (or 8.0.31+) |
| **EXCEPT / MINUS**| Rows in Set A but NOT in Set B | ❌ Simulated (or 8.0.31+) |

---

## 4. Why This Concept Exists

Real-world data often lives in fragmented tables that are structurally identical but logically separate.
- **Example:** You have a `Current_Employees` table and an `Archived_Employees` table. To get a list of "Everyone who has ever worked here," you need to stack these two tables vertically.

---

## 5. How It Works (The Result Set Merger - PRO LEVEL)

When you run a Set Operation:
1.  MySQL executes the first query and stores the result in a temporary memory buffer.
2.  It executes the second query.
3.  **For UNION:** It performs a **Sort-and-Deduplicate** pass to remove rows that appear in both sets.
4.  **For UNION ALL:** It simply appends the second result to the first — much faster!

---

## 6. Syntax / Implementation

```sql
SELECT col1, col2 FROM TableA
UNION
SELECT col1, col2 FROM TableB;
```

---

## 7. Real-Life Examples

**The Master Contact List:**
Combining a `Customers` table and a `Suppliers` table into one single mailing list.
```sql
SELECT Name, Email, 'Customer' AS Category FROM Customers
UNION
SELECT Name, Email, 'Supplier' AS Category FROM Suppliers;
```

---

## 8. Common Mistakes

*   **Column Mismatch:** `SELECT Name FROM T1 UNION SELECT Name, Age FROM T2` will fail because the column counts don't match (1 vs 2).
*   **Assuming Header Names come from the second query:** In a Set Operation, the final column headers are **always** taken from the **first** query in the chain.

---

## 9. Tips & Best Practices (Pro-Level)

**Sorting a Set Result:**
If you want to sort the combined result, you only place the `ORDER BY` at the very end of the **last** query. It will apply to the entire unified result set.

```sql
SELECT Name FROM T1 
UNION 
SELECT Name FROM T2 
ORDER BY Name ASC; -- Sorts the final combined list
```

---

## 10. Mini Practice Tasks

*   **Task 1:** List the 3 strict rules a query must follow to be eligible for a `UNION` operation.
*   **Task 2:** Write a query that combines `Products` from two different databases (e.g., `Electronics_DB` and `Furniture_DB`) into a single list of `Product_Name` and `Price`.

---
