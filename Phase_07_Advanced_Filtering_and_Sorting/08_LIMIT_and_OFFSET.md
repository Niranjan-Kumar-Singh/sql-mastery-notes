# 📄 Topic 7.8: LIMIT and OFFSET (Pagination)

We can filter, we can sort. But what if the sorted result contains 5 million rows and the user's screen can only show 20 at a time? Loading all 5 million rows would crash the browser, kill network bandwidth, and annihilate the server's RAM.

**`LIMIT`** and **`OFFSET`** are MySQL's pagination tools — they allow you to retrieve only a specific "page" of results out of a large sorted dataset.

---

## 1. Definition

*   **`LIMIT n`:** Returns only the first `n` rows of the result set after all other clauses (`WHERE`, `ORDER BY`) have been applied.
*   **`OFFSET n`:** Skips the first `n` rows before starting to return results. Used with `LIMIT` to implement page-by-page navigation.

---

## 2. Why This Concept Exists

Web applications never display raw database dumps. Showing 10, 20, or 50 rows per page is a UX standard. Without `LIMIT`/`OFFSET`, you would need to retrieve millions of rows and discard most of them in application memory — completely wasteful. These clauses push the pagination logic down into the database where it is handled efficiently.

---

## 3. How It Works (Row Scanning Cost - PRO LEVEL)

Here is the critical performance insight that most developers miss:

`OFFSET` does **not** teleport to a row number. MySQL still scans and discards all rows before the offset:

```
LIMIT 10 OFFSET 1000000
```
MySQL reads and discards 1,000,000 rows, then returns the next 10. For large offsets, this gets exponentially slow regardless of indexes.

**The Enterprise Solution — Keyset (Cursor) Pagination:**
Instead of `OFFSET`, use the last seen primary key to filter forward:

```sql
-- Instead of OFFSET 1000000:
-- ❌ Slow OFFSET pagination
SELECT * FROM Products ORDER BY Product_ID LIMIT 10 OFFSET 1000000;

-- ✅ Fast Keyset pagination (use the last_seen_id from the previous page)
SELECT * FROM Products WHERE Product_ID > 1000000 ORDER BY Product_ID LIMIT 10;
-- This uses a B-Tree index seek directly — no scanning of 1M rows!
```

---

## 4. Syntax / Implementation (Pagination Formula)

```sql
-- Basic LIMIT
SELECT * FROM table LIMIT 10;  -- First 10 rows only

-- LIMIT with OFFSET (Two equivalent syntaxes)
SELECT * FROM table LIMIT 10 OFFSET 20;  -- Skip 20, return next 10
SELECT * FROM table LIMIT 20, 10;         -- Same: first number is offset, second is count

-- Pagination Formula:
-- Page N (1-indexed), Page Size = S:
-- LIMIT S OFFSET (N - 1) * S
-- Page 1: LIMIT 20 OFFSET 0
-- Page 2: LIMIT 20 OFFSET 20
-- Page 3: LIMIT 20 OFFSET 40
```

---

## 5. Real-Life Examples

**The Instagram Explore Feed (Paginated infinite scroll):**
```sql
-- Page 1: First 20 posts, most recent first
SELECT Post_ID, Image_URL, Caption
FROM Posts
ORDER BY Posted_At DESC
LIMIT 20 OFFSET 0;

-- Page 2: Next 20 posts
SELECT Post_ID, Image_URL, Caption
FROM Posts
ORDER BY Posted_At DESC
LIMIT 20 OFFSET 20;
```

**The Fast Keyset version (What Instagram actually uses):**
```sql
-- After seeing Post_ID 5000 as the last post on the previous page:
SELECT Post_ID, Image_URL, Caption
FROM Posts
WHERE Post_ID < 5000
ORDER BY Post_ID DESC
LIMIT 20;
-- Direct B-Tree seek — no scanning millions of older posts!
```

---

## 6. SQL Examples (MySQL Execution)

```sql
-- 1. Top 5 highest-paid employees
SELECT First_Name, Salary FROM Employees ORDER BY Salary DESC LIMIT 5;

-- 2. Page 3 of results (page size = 10, pages are 1-indexed)
SELECT First_Name, Department FROM Employees ORDER BY Last_Name ASC LIMIT 10 OFFSET 20;

-- 3. Single row: Get the employee with the longest tenure
SELECT First_Name, Hire_Date FROM Employees ORDER BY Hire_Date ASC LIMIT 1;

-- 4. Random sample of 5 rows (for testing)
SELECT * FROM Employees ORDER BY RAND() LIMIT 5;
-- ⚠️ ORDER BY RAND() is very slow on large tables — shuffles entire dataset in memory!
```

---

## 7. Common Mistakes

*   **Deep Pagination Performance Cliff:** Pages 1-10 are fast. Page 10,000 (`OFFSET 100000`) becomes extremely slow because MySQL must scan 100,000 rows just to discard them. Use Keyset Pagination for deep scrolling.
*   **LIMIT without ORDER BY gives non-deterministic results:** `SELECT * FROM Products LIMIT 10` returns 10 rows in storage order — which can change between queries. Always pair `LIMIT` with `ORDER BY` to get a consistent, predictable page.
*   **`LIMIT 20, 10` syntax confusion:** The two-argument version `LIMIT offset, count` (offset first) is the reverse of the `LIMIT count OFFSET offset` syntax. This trips up many developers.

---

## 8. Tips & Best Practices (Pro-Level)

**Implementing a `COUNT` for Total Pages:**
A proper paginated API also needs to tell the frontend how many total pages exist:

```sql
-- Step 1: Get the total count (for "Showing page 3 of 47")
SELECT COUNT(*) AS Total_Results
FROM Employees
WHERE Department = 'Engineering';

-- Step 2: Get this page's data
SELECT First_Name, Salary
FROM Employees
WHERE Department = 'Engineering'
ORDER BY Salary DESC
LIMIT 20 OFFSET 40;  -- Page 3
```
*Note: Running COUNT(*) + a data query means 2 round-trips. In high-traffic APIs, engineers cache the COUNT result separately and refresh it on a schedule rather than querying it with every page load.*

---

## 9. Mini Practice Tasks

*   **Task 1:** Write the SQL query to retrieve page 5 of products (each page shows 15 products), sorted by `Price` from lowest to highest.
*   **Task 2:** Explain why `LIMIT 10 OFFSET 500000` is slow even when the target column is indexed. What is the enterprise alternative pattern called?

---
