# 🔼🔽 Topic 7.7: Sorting Data — ORDER BY (ASC, DESC, Multiple Columns)

Filtering with `WHERE` gives us the right rows. But unless we sort them, MySQL returns them in no guaranteed order — whatever physical order it encountered them on disk. For any user-facing list (a leaderboard, a product catalog, an invoice history), the sort order is critical.

**`ORDER BY`** is MySQL's universal sort clause.

---

## 1. Definition

**`ORDER BY`:** A clause appended to a `SELECT` statement that instructs MySQL to sort the result rows by one or more specified columns in either Ascending (`ASC`) or Descending (`DESC`) order before returning them.

---

## 2. Why This Concept Exists

Without `ORDER BY`, the order of rows returned by a `SELECT` is completely indeterminate — MySQL will return rows in whatever order is most convenient for the storage engine (often insertion order, or the order from a particular index scan). For any meaningful data presentation, explicit sorting is required.

---

## 3. ASC vs DESC

*   **`ASC` (Ascending, Default):** Smallest → Largest. `A → Z` for text. `1 → 9` for numbers. Oldest → Newest for dates.
*   **`DESC` (Descending):** Largest → Smallest. `Z → A` for text. `9 → 1` for numbers. Newest → Oldest for dates.

---

## 4. How It Works (Filesort vs Index Sort - PRO LEVEL)

Sorting is one of the most expensive database operations. MySQL has two strategies:

**1. Index Sort (Fast):** If the `ORDER BY` column is indexed and MySQL is already using that index for the `WHERE` clause, MySQL reads rows in pre-sorted B-Tree order — zero extra sorting work needed. The `EXPLAIN` output shows `Using index` — this is ideal.

**2. Filesort (Slow):** If the `ORDER BY` column is not indexed (or is different from the WHERE index), MySQL must retrieve all matching rows, dump them into a temporary memory buffer, and sort them using an in-memory quicksort algorithm. If the buffer overflows RAM, MySQL spills to a temporary disk file — hence the name "Filesort". This appears as `Using filesort` in `EXPLAIN` output — a performance warning for large tables.

```sql
-- Check if ORDER BY triggers a filesort:
EXPLAIN SELECT First_Name FROM Employees WHERE Department = 'Engineering' ORDER BY Salary DESC;
-- If 'Extra' column shows 'Using filesort' → consider adding an index on (Department, Salary)
```

---

## 5. Syntax / Implementation

```sql
-- Single column, ascending (default)
SELECT * FROM table ORDER BY column ASC;

-- Single column, descending
SELECT * FROM table ORDER BY column DESC;

-- Multiple columns (Primary sort, then secondary sort for ties)
SELECT * FROM table ORDER BY column1 ASC, column2 DESC;

-- Sort by column position number (less readable, avoid in production)
SELECT First_Name, Salary FROM Employees ORDER BY 2 DESC;

-- Sort by an alias (works because ORDER BY evaluates after SELECT)
SELECT Salary * 12 AS Annual_Pay FROM Employees ORDER BY Annual_Pay DESC;
```

---

## 6. Real-Life Examples

**The Leaderboard:**
```sql
-- Game leaderboard: highest score first, alphabetical by name on ties
SELECT Username, High_Score, Level
FROM Players
ORDER BY High_Score DESC, Username ASC
LIMIT 10;
```

**The Transaction History Page:**
```sql
-- Most recent transactions first (standard banking UX)
SELECT Transaction_ID, Amount, Transaction_Date
FROM Transactions
WHERE Customer_ID = 1045
ORDER BY Transaction_Date DESC;
```

---

## 7. SQL Examples (MySQL Execution)

```sql
-- 1. Highest earners first
SELECT First_Name, Salary FROM Employees ORDER BY Salary DESC;

-- 2. Alphabetical by last name, then first name (for ties)
SELECT First_Name, Last_Name FROM Employees ORDER BY Last_Name ASC, First_Name ASC;

-- 3. Most recently hired first
SELECT First_Name, Hire_Date FROM Employees ORDER BY Hire_Date DESC;

-- 4. Department alphabetically, then salary DESC within each department
SELECT First_Name, Department, Salary
FROM Employees
ORDER BY Department ASC, Salary DESC;

-- 5. WHERE + ORDER BY together
SELECT First_Name, Salary
FROM Employees
WHERE Department = 'Engineering'
ORDER BY Salary DESC;
```

---

## 8. Common Mistakes

*   **Expecting a consistent default order without `ORDER BY`:** Beginners assume MySQL always returns rows in insertion order. It does NOT — the order is non-deterministic and can change between queries, MySQL versions, or after an `UPDATE`. Never rely on implicit ordering.
*   **`ORDER BY` with `LIMIT` and no index:** The database must sort the entire result set before it can apply the `LIMIT`. On 10M rows without an index, this triggers a massive filesort even if you only want 10 rows.

---

## 9. Tips & Best Practices (Pro-Level)

**The Composite Index for `WHERE + ORDER BY`:**
The best performance happens when a single composite index covers both the `WHERE` column and the `ORDER BY` column:

```sql
-- Create a composite index covering both filter and sort
CREATE INDEX idx_dept_salary ON Employees (Department, Salary);

-- This query can now be served entirely from the index (no filesort!)
SELECT First_Name, Salary
FROM Employees
WHERE Department = 'Engineering'
ORDER BY Salary DESC;
-- EXPLAIN shows: type=ref, key=idx_dept_salary, Extra=Using index -- Perfect!
```

---

## 10. Mini Practice Tasks

*   **Task 1:** Write a query that retrieves all employees from the `Employees` table, sorted by `Department` alphabetically (ASC), and within each department sorted by `Salary` from highest to lowest (DESC).
*   **Task 2:** What is "Filesort" in MySQL, and when does it occur? How can you avoid it?

---
