# 🛑 Topic 12.6: When NOT to Use Indexes

Indexes make reads faster — but they are not free. Understanding when **not** to index is just as important as knowing when to index. Blindly indexing every column is one of the most common performance anti-patterns in database design.

---

## 1. The Hidden Cost of Every Index

Every index you create adds **three costs** to your database:

| Cost | What it Means |
|---|---|
| **Write Overhead** | Every `INSERT`, `UPDATE`, or `DELETE` must also update the index tree. 10 indexes = 11 write operations per row change |
| **Storage Space** | Index files live on disk. A heavily indexed table can have an index file larger than the data file itself |
| **Optimizer Confusion** | With 20+ indexes, MySQL's optimizer may pick the wrong index, making queries slower than no index at all |

---

## 2. Columns That Should NEVER Be Indexed

### A. Low-Cardinality Columns
**Cardinality** = number of unique values. A column with very few unique values is a poor index candidate.

**Examples:** `Gender` ('Male'/'Female'), `Status` ('Active'/'Inactive'), `is_verified` (TRUE/FALSE), `Payment_Method` ('Cash'/'Card'/'UPI').

**Why it doesn't help:** If 50% of your users are 'Active', an index on `Status` doesn't filter much data. MySQL will likely do a full table scan anyway — and using the index adds overhead since it has to process the index tree first.

```sql
-- ❌ Useless index — half the table is 'Active' anyway
CREATE INDEX idx_status ON Users(Status);

-- ✅ Instead: Partial index concept (not MySQL-native) or composite index
-- with a high-cardinality column first:
CREATE INDEX idx_status_created ON Users(Created_At, Status);
```

### B. Very Small Tables (< 1,000 rows)
If a table has 200 rows, MySQL can read the entire table from memory in microseconds. The overhead of navigating an index tree is actually **slower** than just reading all 200 rows directly. MySQL's optimizer often ignores indexes on small tables for this reason.

### C. Frequently Updated Columns
A `last_seen_at` timestamp updated every time a user makes any action, or a `page_hit_counter` incremented thousands of times per minute — indexing these causes **constant index tree rebuilding**, thrashing the InnoDB buffer pool and CPU.

```sql
-- ❌ Don't index a column that changes every few seconds
CREATE INDEX idx_last_updated ON Sessions(Last_Active_At);
```

### D. Very Long Text Columns
Indexing a full `TEXT` or `BLOB` column (like an article body, user bio, or email content) creates an enormous index that's slow to navigate and wastes disk space. 

```sql
-- ❌ Terrible for a full TEXT column
CREATE INDEX idx_content ON Articles(Content);

-- ✅ Use FULL-TEXT index instead for word-based searching
CREATE FULLTEXT INDEX ft_content ON Articles(Content);
```

---

## 3. The Goldilocks Principle — Good vs. Bad Index Candidates

| Column | Good for Index? | Reason |
|---|---|---|
| `User_ID` (Primary Key) | ✅ Always | Automatically indexed, unique, high cardinality |
| `Email` | ✅ Yes | High cardinality, frequently searched |
| `Order_Date` | ✅ Yes | Range queries, high cardinality |
| `Status` (Active/Inactive) | ❌ No | Low cardinality (only 2 values) |
| `Gender` | ❌ No | Low cardinality (2-3 values) |
| `Article_Body` | ❌ No | Full text — use FULLTEXT index |
| `Last_Ping_At` (updates every 10s) | ❌ No | Too frequently updated |
| `Notes` (mostly NULL) | ❌ No | Sparse data, nulls everywhere |

---

## 4. Real-Life Example — The Over-Indexed Disaster

A junior developer on their first production database creates:
```sql
CREATE INDEX idx1 ON Users(User_ID);      -- Already a PK — duplicate!
CREATE INDEX idx2 ON Users(First_Name);   -- Low cardinality (many Johns)
CREATE INDEX idx3 ON Users(Status);       -- Too few values
CREATE INDEX idx4 ON Users(Bio);          -- TEXT column — terrible!
CREATE INDEX idx5 ON Users(Updated_At);   -- Changes constantly
```

**Result:**
- New user registration takes 800ms instead of 8ms.
- The `Users` table index files are 3× larger than the actual data.
- Bulk imports that should take minutes now take hours.

**Fix:** Drop all redundant and bad indexes, keep only `Email` and `Created_At`:
```sql
DROP INDEX idx1 ON Users;
DROP INDEX idx2 ON Users;
DROP INDEX idx3 ON Users;
DROP INDEX idx4 ON Users;
DROP INDEX idx5 ON Users;
CREATE INDEX idx_email ON Users(Email);          -- High cardinality, frequently searched
CREATE INDEX idx_created ON Users(Created_At);  -- Date range queries
```

---

## 5. How to Check Your Current Indexes

```sql
-- See all indexes on a table
SHOW INDEX FROM Users;
-- Key columns: Key_name, Cardinality, Index_type

-- Find unused indexes (MySQL 8.0+ with performance_schema)
SELECT * FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL 
  AND COUNT_STAR = 0    -- Never used!
  AND OBJECT_SCHEMA = 'your_db_name';

-- Analyze index usage in a specific query
EXPLAIN SELECT * FROM Users WHERE Status = 'Active';
-- If 'key' column shows NULL → index not being used
-- If rows estimate is > 50% of table size → index likely not helpful
```

---

## 6. Common Mistakes

- **Indexing foreign keys on the parent table:** The primary key in the parent table is already indexed. You should index the FK column on the **child** table (e.g., index `Customer_ID` in the `Orders` table, not in `Customers`).
- **Unused duplicate indexes:** Creating `idx_email` and `idx_email_name` on the same table where the first column is `Email` in both — the second index is nearly always redundant.
- **Not removing old indexes:** As the application evolves, some indexes become orphaned (the query patterns that needed them were removed). Unused indexes still incur write overhead.

---

## 7. Tips & Best Practices

- **Index only columns used in WHERE, JOIN ON, or ORDER BY clauses.** If a column never appears in these positions, it doesn't need an index.
- **For low-cardinality + high-cardinality combinations**, use composite indexes with high-cardinality column FIRST: `(Created_At, Status)` is better than `(Status, Created_At)`.
- **Periodically audit your indexes** using the `performance_schema` query above. Remove indexes that haven't been used in 30+ days.

---

## 8. Mini Practice Tasks

- **Task 1:** A developer adds `CREATE INDEX idx_payment ON Orders(Payment_Method)` where Payment_Method has values 'Cash', 'Card', 'UPI', 'EMI'. Is this a good index? Why or why not?
- **Task 2:** Your `Products` table has 150 rows. A query `SELECT * FROM Products WHERE Category = 'Electronics'` is "slow" (takes 2ms). Should you add an index? What would a senior developer say?
- **Task 3:** How does a `FULLTEXT INDEX` differ from a regular `INDEX` for large text columns?

---
