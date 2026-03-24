# 🎭 Topic 6.4: The DISTINCT Keyword (Handling Duplicate Results)

When you run a `SELECT` on a large table, you will often get back result rows where certain column values repeat — the same city appears 500 times, or the same department name appears 200 times. In reporting, this repetition is usually meaningless noise.

The **`DISTINCT`** keyword filters these duplicates out of the result set, showing each unique value exactly once.

---

## 1. Definition

**`DISTINCT`:** A modifier applied immediately after `SELECT` that instructs MySQL to scan the entire result set and remove any rows where the combination of selected column values is duplicated, returning only one instance of each unique value combination.

---

## 2. Why This Concept Exists

A `SELECT` by default returns every matching row — including exact duplicates. In a 5-million-row `Orders` table, asking "Which cities have customers?" without `DISTINCT` would return `New York` listed 200,000 times. Businesses need de-duplicated, unique-value lists for dropdowns, reports, and analytics.

---

## 3. Why We Use It

*   **Generating dropdown options:** A search form needs a list of unique cities — not `'Mumbai'` listed 50,000 times.
*   **Quick cardinality check:** `SELECT COUNT(DISTINCT Department)` tells you how many unique departments exist without manually filtering.
*   **Data quality audits:** `SELECT DISTINCT Email FROM Users` to spot if any email appears more than once (indicating a potential UNIQUE constraint violation on legacy data).

---

## 4. When to Use It

*   When you need a list of **unique values** from a column or combination of columns.
*   As a quick way to inspect data variety during development.
*   **Caution:** `DISTINCT` is computationally expensive on large tables. If performance matters, consider `GROUP BY` instead (which can leverage indexes more efficiently).

---

## 5. How It Works (The Sort-and-Deduplicate Algorithm - PRO LEVEL)

`DISTINCT` internally forces MySQL to perform an additional pass over the result set:

1.  MySQL retrieves all rows matching the query.
2.  It passes the rows through a **temporary in-memory HashTable or Sort** to group identical value combinations.
3.  It returns only one row per unique group.

This extra sorting step means `DISTINCT` adds CPU and memory overhead. On a table with 50 million rows and high cardinality (many unique values), `DISTINCT` can be 10x slower than the equivalent `GROUP BY` on an indexed column.

**The `DISTINCT` vs `GROUP BY` Equivalency:**
```sql
-- These two queries return identical results:
SELECT DISTINCT Department FROM Employees;

SELECT Department FROM Employees GROUP BY Department;
-- GROUP BY CAN use an index more efficiently than DISTINCT in some cases!
```

---

## 6. Syntax / Implementation

```sql
-- Single column
SELECT DISTINCT column_name FROM table_name;

-- Multiple columns (DISTINCT applies to the COMBINATION of values)
SELECT DISTINCT column1, column2 FROM table_name;

-- With COUNT
SELECT COUNT(DISTINCT column_name) FROM table_name;
```

---

## 7. Real-Life Examples

**The E-Commerce Category Filter:**
A fashion website needs to populate a filter sidebar with all unique clothing categories.
```sql
SELECT DISTINCT Category FROM Products ORDER BY Category ASC;
```
Without `DISTINCT`, "Shirts" would appear 85,000 times for 85,000 shirts in the catalogue.

---

## 8. SQL Examples (MySQL Execution)

```sql
-- 1. List all unique departments (no repeats)  
SELECT DISTINCT Department FROM Employees;

-- 2. List all unique city+department combinations
SELECT DISTINCT City, Department FROM Employees;
-- Note: 'New York, Engineering' and 'New York, Marketing' are TWO distinct combinations!

-- 3. Count how many unique cities employees are located in
SELECT COUNT(DISTINCT City) AS Unique_Office_Locations FROM Employees;

-- 4. Find all unique salary tiers (to spot pay bands)
SELECT DISTINCT Salary FROM Employees ORDER BY Salary DESC;
```

---

## 9. Common Mistakes

*   **DISTINCT applies to ALL selected columns, not just the first:** Beginners assume `SELECT DISTINCT City, Department` deduplicates only on `City`. It actually deduplicates on the **unique combination** of City AND Department. `New York, Engineering` and `New York, Marketing` are two distinct rows!
*   **Using DISTINCT instead of fixing a bad JOIN:** Many times when a query returns duplicates, the real cause is a missing JOIN condition or a many-to-many relationship that needs handling. `DISTINCT` hides the symptom but doesn't fix the architectural problem.

---

## 10. Tips & Best Practices (Pro-Level)

**Measuring Column Cardinality (For Index Design):**
Senior engineers use `DISTINCT` to measure **cardinality** — the count of unique values in a column. High cardinality = excellent index candidate. Low cardinality = poor index candidate.

```sql
-- Check cardinality of the 'Department' column
SELECT COUNT(DISTINCT Department) AS Unique_Departments,
       COUNT(*) AS Total_Rows
FROM Employees;
-- If Unique_Departments = 3 and Total_Rows = 500,000, 
-- the Department column has LOW cardinality — a B-Tree index here is mostly useless!
```

---

## 11. Mini Practice Tasks

*   **Task 1:** Write a query to retrieve all unique `City` values from the `Employees` table, sorted alphabetically.
*   **Task 2:** You run `SELECT DISTINCT City, Department FROM Employees`. The result shows two rows: `('Mumbai', 'Engineering')` and `('Mumbai', 'HR')`. Does DISTINCT think these are duplicates? Why or why not?

---
