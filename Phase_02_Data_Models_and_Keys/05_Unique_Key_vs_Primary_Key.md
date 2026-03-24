# 🆔 Topic 2.5: Unique Key vs. Primary Key

Both `PRIMARY KEY` and `UNIQUE KEY` prevent duplicate values in a column. But they serve fundamentally different purposes, and confusing them is one of the most common beginner database design mistakes.

---

## 1. Definitions

- **Primary Key (PK):** The **main row identifier**. Physically organizes how data is stored on disk. Every table must have exactly one. Cannot be NULL.
- **Unique Key:** A **business constraint**. Ensures no two rows have the same value in a column (like Email or Username), but doesn't make it the row's identity. A table can have many. Allows NULL (one per column in most engines).

---

## 2. The Full Comparison Table

| Feature | Primary Key | Unique Key |
|---|---|---|
| **Number per table** | Exactly ONE | Unlimited |
| **NULL allowed** | ❌ Never | ✅ One NULL per column |
| **Index type created** | Clustered Index (rows stored in PK order) | Non-Clustered Index (separate data structure) |
| **Main purpose** | Row identity — linking tables (FK references) | Business uniqueness constraint |
| **Can be a Foreign Key target** | ✅ Yes (most common) | ✅ Yes (but unusual) |
| **Auto-incremented** | Often (`AUTO_INCREMENT INT`) | Rarely |

---

## 3. Why This Difference Matters (PRO LEVEL)

### The Clustered Index Effect
In MySQL/InnoDB, the Primary Key determines the **physical storage order of rows on disk**. This is called the **Clustered Index**. When you query `WHERE Customer_ID = 500`, MySQL navigates the B+ Tree of the clustered index directly to that row — extremely fast.

A Unique Key creates a **separate B+ Tree** (Non-Clustered Index) that stores the unique column values along with pointers back to the actual row. One extra lookup compared to PK.

### The NULL Difference
The `UNIQUE` constraint allows NULL because SQL defines NULL as "unknown" — and no two unknown values can be said to be equal. So technically, `NULL ≠ NULL` in SQL, meaning multiple NULLs don't violate uniqueness.

---

## 4. Syntax / Implementation

```sql
-- A table with one Primary Key and two Unique Keys
CREATE TABLE Users (
    User_ID    INT AUTO_INCREMENT PRIMARY KEY,  -- PK: row identity, never null, never changes
    Email      VARCHAR(100) UNIQUE NOT NULL,     -- Unique Key: business constraint (required)
    Username   VARCHAR(50)  UNIQUE,              -- Unique Key: business constraint (optional)
    First_Name VARCHAR(50)  NOT NULL,
    Created_At TIMESTAMP DEFAULT NOW()
);

-- Add a Unique Key to an existing table
ALTER TABLE Users ADD CONSTRAINT uq_phone UNIQUE (Phone);

-- See all unique keys on a table
SHOW INDEX FROM Users;
-- Key_name: 'PRIMARY' (the PK), 'Email', 'Username' (the Unique Keys)
```

---

## 5. Real-Life Examples

**Example 1 — When to use UNIQUE (not PK):**
```sql
CREATE TABLE Products (
    Product_ID  INT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate PK
    SKU         VARCHAR(50) UNIQUE NOT NULL,       -- Business unique identifier (changes rarely)
    Barcode     VARCHAR(20) UNIQUE,               -- Another unique business code
    Product_Name VARCHAR(200) NOT NULL
);
-- Why not use SKU as PK? SKUs can change when products are reformulated, repackaged, etc.
-- Using an immutable Product_ID as PK is safer.
```

**Example 2 — Why Email as PK is a bad idea:**
```sql
-- ❌ BAD DESIGN: Email as Primary Key
CREATE TABLE Users (
    Email VARCHAR(100) PRIMARY KEY,  -- Problem: Emails change!
    Name VARCHAR(100)
);
-- If user changes email: must UPDATE primary key → cascades to ALL foreign key references
-- Could break thousands of records in Orders, Sessions, Logs tables

-- ✅ GOOD DESIGN: Surrogate PK + Email as Unique Key
CREATE TABLE Users (
    User_ID INT AUTO_INCREMENT PRIMARY KEY,
    Email VARCHAR(100) UNIQUE NOT NULL,  -- Business constraint, not identity
    Name VARCHAR(100)
);
```

---

## 6. Common Mistakes

- **Using a business value (Email, Phone, SSN) as PK:** Business values change. Surrogate auto-increment IDs never change. Always use a meaningless `ID` as PK.
- **Forgetting `NOT NULL` on Unique Keys:** A `UNIQUE` column without `NOT NULL` allows NULL values, which can silently cause issues if your application expects the column to always have a value.
- **Thinking UNIQUE constraint slows down queries:** The opposite — UNIQUE keys create indexes that **speed up** lookups on those columns.

---

## 7. Tips & Best Practices

- **Default to `INT AUTO_INCREMENT PRIMARY KEY`** for every table — a surrogate key that satisfies all PK requirements by design.
- **Add `UNIQUE NOT NULL` for business identifiers** like email, username, phone, national ID — values that must be unique and non-empty but aren't the row's identity.
- **Never expose the Primary Key in public-facing systems** (e.g., URLs like `/user/123`). Use a separate `UUID` or public-facing unique token instead.

---

## 8. Mini Practice Tasks

- **Task 1:** A `Products` table has `Product_ID` (auto-increment) and `Barcode` (13-digit, always unique). Which should be the Primary Key? Write the `CREATE TABLE` SQL.
- **Task 2:** What is a Clustered Index? Why does the Primary Key in InnoDB create one, but a Unique Key doesn't?
- **Task 3:** If a `UNIQUE` column allows NULL but a `PRIMARY KEY` doesn't, how many NULL values can a UNIQUE column have? Why does SQL treat this as valid?

---
