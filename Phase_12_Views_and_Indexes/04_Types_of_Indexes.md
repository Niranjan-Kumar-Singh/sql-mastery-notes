# 🗂️ Topic 12.4: Types of Indexes in MySQL

You now know that an Index is like a phone book for your data. But just like there are different types of reference books (alphabetical directory, topic index, reverse lookup), there are different **types of indexes** optimized for different situations.

---

## 1. Overview

MySQL (InnoDB engine) supports the following major index types:

| Index Type | Primary Use | Unique? |
|---|---|---|
| **Primary Index (Clustered)** | The table's main identifier (PK) | ✅ Yes |
| **Unique Index** | Enforce uniqueness on a non-PK column | ✅ Yes |
| **Regular Index (Secondary)** | Speed up `WHERE` / `JOIN` / `ORDER BY` | ❌ No |
| **Composite Index** | Speed up multi-column filters | ❌ / ✅ |
| **Full-Text Index** | Search natural language inside long text | ❌ No |
| **Spatial Index** | Geographic/location data queries | ❌ No |

---

## 2. Primary Index (Clustered Index)

The **Primary Index** is automatically created when you define a `PRIMARY KEY`. It is special because in InnoDB, the table's data itself is physically stored **in the order of this index** — it IS the table. This is why it is called a **Clustered Index**.

- There can only be **one** Clustered Index per table (because data can only be sorted one way on disk).
- All other indexes are stored separately and contain a "pointer back" to this primary row.

```sql
CREATE TABLE Products (
    Product_ID INT PRIMARY KEY,   -- Clustered Index created automatically here
    Name       VARCHAR(100),
    Price      DECIMAL(10, 2)
);
```

---

## 3. Unique Index

A **Unique Index** ensures all values in the indexed column(s) are distinct, while also providing fast lookup speed. It is created automatically when you add a `UNIQUE` constraint.

```sql
-- Method 1: Via UNIQUE constraint (recommended)
CREATE TABLE Users (
    User_ID INT PRIMARY KEY,
    Email   VARCHAR(100) UNIQUE   -- Automatically creates a Unique Index on Email
);

-- Method 2: Explicit creation
CREATE UNIQUE INDEX idx_unique_email ON Users(Email);
```

**Key Fact:** A `UNIQUE` index *does* allow one `NULL` value (because `NULL != NULL` in SQL). This is the only exception to the "all values must be distinct" rule.

---

## 4. Regular (Non-Unique / Secondary) Index

A regular index speeds up search queries on columns that are NOT necessarily unique. This is the most common type you'll add manually.

**When to add one:**
- Columns frequently used in `WHERE` clauses.
- Columns used in `JOIN` conditions.
- Columns used in `ORDER BY` or `GROUP BY`.

```sql
-- Speed up filtering employees by department
CREATE INDEX idx_dept ON Employees(Department);

-- Now this query is fast even on millions of rows:
SELECT * FROM Employees WHERE Department = 'Engineering';
```

---

## 5. Composite Index

A **Composite Index** covers **two or more columns**. It is extremely powerful for queries that filter on multiple conditions simultaneously.

```sql
-- A composite index on (Last_Name, First_Name)
CREATE INDEX idx_name ON Employees(Last_Name, First_Name);
```

**The Leftmost Prefix Rule (Critical!):**

MySQL can use a Composite Index for queries that filter using the **leftmost column(s)** of the index:

```sql
-- ✅ USES the index (filters on Last_Name first)
SELECT * FROM Employees WHERE Last_Name = 'Smith';

-- ✅ USES the index (filters on both columns)
SELECT * FROM Employees WHERE Last_Name = 'Smith' AND First_Name = 'Alice';

-- ❌ DOES NOT USE the index (skips Last_Name, jumps to First_Name only)
SELECT * FROM Employees WHERE First_Name = 'Alice';
```

**Rule:** Put the **highest selectivity column** (the one that filters out the most rows) **first** in a composite index.

---

## 6. Full-Text Index

A **Full-Text Index** is designed for searching natural language content — like searching through article bodies, product descriptions, or blog posts. Standard `LIKE '%keyword%'` is painfully slow for long text; Full-Text is not.

```sql
-- Create the Full-Text Index
ALTER TABLE Articles ADD FULLTEXT INDEX ft_body (Title, Body);

-- Use it with MATCH...AGAINST for blazing-fast text search
SELECT * FROM Articles 
WHERE MATCH(Title, Body) AGAINST('database optimization' IN NATURAL LANGUAGE MODE);
```

**Limitation:** Only works on `CHAR`, `VARCHAR`, and `TEXT` columns with the InnoDB (or MyISAM) storage engine.

---

## 7. Common Mistakes

- **Indexing every column:** More indexes = slower `INSERT/UPDATE/DELETE` (because MySQL must update each index file). Only index columns that are actually searched or sorted.
- **Ignoring the Leftmost Prefix Rule:** Creating a composite index `(City, Country)` and then writing `WHERE Country = 'India'` — MySQL won't use it! Add a separate index on `Country` if needed.
- **Not checking with EXPLAIN:** Always run `EXPLAIN SELECT ...` before and after adding an index to verify MySQL is actually using it.

---

## 8. Mini Practice Tasks

- **Task 1:** You have an `Orders` table with columns `Customer_ID`, `Status`, and `Order_Date`. You frequently run: `SELECT * FROM Orders WHERE Customer_ID = 5 AND Status = 'Pending'`. What type of index should you create, and on which columns?
- **Task 2:** What is the key difference between a **Clustered Index** and a **Non-Clustered (Secondary) Index** in InnoDB?

---
