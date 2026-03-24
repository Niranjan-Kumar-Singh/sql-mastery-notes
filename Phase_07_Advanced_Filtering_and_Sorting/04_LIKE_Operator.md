# 🔤 Topic 7.4: Pattern Matching — The LIKE Operator

Comparison operators (`=`, `!=`) match only **exact** values. But real search queries are fuzzy: *"Find all users whose name starts with 'Joh'"* or *"Find all emails from Gmail."* This is pattern matching — and MySQL provides it through the **`LIKE`** operator.

---

## 1. Definition

**`LIKE`:** A string-matching operator used in a `WHERE` clause to search for a specified **pattern** in a column. It returns `TRUE` if the column value matches the pattern, `FALSE` if it doesn't.

Patterns are built using regular characters and two special **Wildcards** (`%` and `_`), which are covered in detail in Topic 7.5.

---

## 2. Why This Concept Exists

Users don't always type complete words. Search bars, autocomplete boxes, and report filters need **partial-text matching**. Without `LIKE`, you would need to retrieve all rows and filter them in your application code — deeply inefficient for millions of records.

---

## 3. Basic Syntax

```sql
-- Standard LIKE
SELECT * FROM Employees WHERE Last_Name LIKE 'Smi%';

-- NOT LIKE: Find rows that do NOT match the pattern
SELECT * FROM Products WHERE SKU NOT LIKE 'DISC%';

-- Using it in a full query context
SELECT Customer_ID, Email
FROM Customers
WHERE Email LIKE '%@gmail.com'
ORDER BY Email;
```

---

## 4. Real-Life Examples

**1. Search Bar — Find products with "wireless" in the name:**
```sql
SELECT Product_Name, Price 
FROM Products
WHERE Product_Name LIKE '%wireless%';
```

**2. Filter — Find all company email addresses (ending in @company.com):**
```sql
SELECT Name, Email
FROM Employees
WHERE Email LIKE '%@company.com';
```

**3. Find usernames that start with "admin":**
```sql
SELECT Username, Last_Login
FROM Users
WHERE Username LIKE 'admin%';
```

**4. Find Indian phone numbers starting with country code +91:**
```sql
SELECT Name, Phone FROM Contacts WHERE Phone LIKE '+91%';
```

---

## 5. Case Sensitivity

In MySQL, `LIKE` is **case-insensitive by default** (when using the default `utf8mb4_unicode_ci` collation).

```sql
-- Both of these return the same rows:
WHERE First_Name LIKE 'alice%'
WHERE First_Name LIKE 'Alice%'
WHERE First_Name LIKE 'ALICE%'
```

To force **case-sensitive** matching, use the `BINARY` keyword:

```sql
-- Only matches 'Admin', NOT 'admin' or 'ADMIN'
SELECT * FROM Users WHERE Username LIKE BINARY 'Admin%';
```

---

## 6. Performance (The Golden Rule — PRO LEVEL)

This is one of the most critical performance facts in all of SQL. How you place your wildcard dramatically changes query speed:

**❌ Leading Wildcard — Forces a Full Table Scan:**
```sql
-- MySQL must scan EVERY row to find those ending in '@gmail.com'
WHERE Email LIKE '%@gmail.com'
```

**✅ Trailing Wildcard — Uses a B-Tree Index (Fast!):**
```sql
-- MySQL can use the index to jump to rows starting with 'Joh'
WHERE First_Name LIKE 'Joh%'
```

**Why?** A B-Tree index is like a phone book. If you know the name starts with *"J"*, you can flip directly to that section. If you only know the name *contains* something, you have to read the entire book from cover to cover.

**Rule of Thumb:**
- If you must search with a leading wildcard (e.g., searching by email domain), consider using a **Full-Text Index** (Topic 12.4) for large tables instead of `LIKE`.

---

## 7. LIKE vs. Exact Match

| Use Case | Best Operator | Reason |
|---|---|---|
| `WHERE Status = 'Active'` | `=` (equality) | Exact match — always faster |
| `WHERE Name LIKE 'Ali%'` | `LIKE` with wildcard | Partial prefix match needed |
| `WHERE Email = 'john@gmail.com'` | `=` (equality) | You know the exact value |

**Rule:** Only use `LIKE` when you genuinely need wildcard pattern matching. For exact comparisons, `=` is always faster.

---

## 8. Common Mistakes

- **Using `LIKE` for exact matches:** `WHERE Status LIKE 'Active'` works but is slower than `WHERE Status = 'Active'`. The DBMS optimizer may not simplify it.
- **Forgetting to escape special characters:** If your pattern contains a literal `%` or `_`, you must escape them. See Topic 7.5.
- **Treating `LIKE` as case-sensitive by default:** Unlike some other databases, MySQL `LIKE` is case-insensitive on most default setups. This can lead to unexpected results.

---

## 9. Tips & Best Practices

- For short, prefix-based searches (autocomplete), use `LIKE 'prefix%'` — it's fast and index-friendly.
- For large text columns (articles, descriptions), prefer `MATCH...AGAINST` with a Full-Text Index over `LIKE '%keyword%'`.
- When debugging performance, run `EXPLAIN SELECT ...` to see if MySQL is performing a full scan or an index scan for your `LIKE` query.

---

## 10. Mini Practice Tasks

- **Task 1:** Write a `SELECT` statement using `LIKE` to find all customers whose `Email` is **not** from the domain `outlook.com`.
- **Task 2:** You want to find all product SKUs that start with `'ELEC'`. Write the query. Which wildcard character should you use, and why is this query faster than `LIKE '%ELEC%'`?
- **Task 3:** How would you search for all employees whose first name contains exactly 5 characters? (Hint: think about which wildcard represents exactly one character.)

---
