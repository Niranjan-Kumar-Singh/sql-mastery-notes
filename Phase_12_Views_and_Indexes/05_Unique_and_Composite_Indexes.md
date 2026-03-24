# 🗄️ Topic 12.5: Unique and Composite Indexes

Sometimes an index on one column isn't enough. You might need to ensure a **combination** is unique, or speed up queries that always filter by two columns at once.

---

## 1. Unique Index

A **Unique Index** is an index that also enforces a data rule: no two rows can have the same value in this index. 
- While a Primary Key is a Unique Index, a table can have many other Unique Indexes (e.g., `Passport_Number`, `Email`, `Social_Security`).
- **NULL Exception:** In MySQL, a unique index **allows multiple NULLs**. This is because NULL != NULL in SQL logic.

---

## 2. Composite Index (Multi-Column Index)

A **Composite Index** is an index created on two or more columns at once. 

```sql
CREATE INDEX idx_name_city ON Users(Last_Name, City);
```

---

## 3. The "Left-Prefix" Rule (PRO LEVEL)

This is the most critical rule for Composit Indexes. The order of columns matters!
An index on `(Last_Name, City)` can speed up:
- `WHERE Last_Name = 'Singh'` (The first part).
- `WHERE Last_Name = 'Singh' AND City = 'Delhi'` (Both parts).

It will **NOT** speed up:
- `WHERE City = 'Delhi'` (The engine cannot skip the first part of the sorted tree).

**Senior Tip:** Always put the most frequently used column as the **first** column in your composite index.

---

## 4. Why This Concept Exists

*   **Covering Indexes:** By including all the columns a query needs in the index, you avoid the "Double Lookup" to the disk, making the query 10-100x faster.
*   **Optimizing JOINS:** Composite indexes on foreign keys and reference columns can drastically reduce the time it takes to link large tables.

---

## 5. Syntax / Implementation

```sql
-- Creating a Composite Unique Index
CREATE UNIQUE INDEX uq_student_course ON Enrollments(Student_ID, Course_ID);

-- Creating a regular Composite Index
CREATE INDEX idx_sales_date_region ON Sales(Sale_Date, Region);
```

---

## 6. Mini Practice Tasks

*   **Task 1:** What is the "Left-Prefix" rule in composite indexing?
*   **Task 2:** Does a Unique Index in MySQL prevent multiple rows from having a `NULL` value in that column?

---
