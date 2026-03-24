# 🏗️ Topic 8.2: Advanced String Functions

Beyond basic concatenation and case conversion, MySQL offers functions for precision string manipulation — essential for data cleaning, ETL (Extract, Transform, Load) processes, and report formatting.

---

## 1. Definition

**Advanced String Functions:** Higher-order text manipulation tools used for finding positions, padding strings to fixed lengths, and extracting data from either end of a string.

---

## 2. The Advanced String Functions Reference

| Function | Description | Example | Result |
|---|---|---|---|
| `REVERSE(str)` | Flips a string backward | `REVERSE('SQL')` | 'LQS' |
| `LOCATE(sub, str)` | Returns position of substring | `LOCATE('@', 'a@b.com')` | 2 |
| `LEFT(str, n)` | Returns first `n` characters | `LEFT('Database', 4)` | 'Data' |
| `RIGHT(str, n)` | Returns last `n` characters | `RIGHT('Database', 4)` | 'base' |
| `LPAD(str, n, pad)`| Pads left to length `n` | `LPAD('42', 5, '0')` | '00042' |
| `RPAD(str, n, pad)`| Pads right to length `n` | `RPAD('42', 5, '*')` | '42***' |

---

## 3. Why This Concept Exists

In enterprise systems, you often deal with:
1.  **Fixed-width files:** Legacy systems often require IDs to be exactly 10 digits (`0000001234`).
2.  **Data Masking:** Displaying only the last 4 digits of a credit card (`************4455`).
3.  **Parsing:** Finding a specific delimiter (like `@` or `,`) to chop up a string.

---

## 4. How It Works (The Padding Algorithm - PRO LEVEL)

When using `LPAD` or `RPAD`, if the original string is **longer** than the specified length `n`, MySQL will **truncate** the string to fit the length.

```sql
SELECT LPAD('Enterprise', 5, '*');
-- Result: 'Enter' (The string was truncated!)
```
Professional developers always check that the target length covers the maximum possible length of the data to avoid accidental truncation.

---

## 5. Real-Life Examples

**Example 1: Credit Card Masking**
```sql
-- Mask a 16-digit card, showing only the last 4 digits
SELECT 
    CONCAT(REPEAT('*', 12), RIGHT(Card_Number, 4)) AS Masked_Card
FROM Transactions;
-- Result: ************4455
```

**Example 2: Fixed-Width ID Generation**
```sql
-- Ensuring every Invoice ID is exactly 10 characters for a bank file
SELECT LPAD(Invoice_ID, 10, '0') AS Bank_Padded_ID
FROM Invoices;
-- Result: '0000009842'
```

---

## 6. SQL Examples (MySQL Execution)

```sql
-- 1. Find the domain from an email address
-- Using LOCATE to find @ and SUBSTRING to get everything after it
SELECT 
    SUBSTRING(Email, LOCATE('@', Email) + 1) AS Domain
FROM Users;

-- 2. Reverse a string (Useful for palindromes or obscure encoding tasks)
SELECT REVERSE('racecar'); -- 'racecar'

-- 3. Extracting file extension
-- (Finding the last '.' is harder in pure SQL, often uses REVERSE + LOCATE trick)
SELECT REVERSE(LEFT(REVERSE('manual.pdf'), LOCATE('.', REVERSE('manual.pdf')) - 1)) AS Extension;
```

---

## 7. Common Mistakes

*   **Confusing `LEFT`/`RIGHT` with `SUBSTRING`:** `LEFT(str, 3)` is just a shorthand for `SUBSTRING(str, 1, 3)`. They are interchangeable, but `LEFT` is more readable for simple prefix extraction.
*   **Locating non-existent substrings:** `LOCATE('x', 'abc')` returns **0** (not NULL). Always check for 0 results when using `LOCATE` in logic.

---

## 8. Tips & Best Practices (Pro-Level)

**Handling Multi-value columns (The "Anti-Pattern" Search):**
Sometimes developers store comma-separated values in a single column (e.g., `Tags = 'sql,mysql,db'`). To find if 'mysql' is in that string, `LOCATE` is more efficient than `LIKE`:

```sql
SELECT * FROM Table WHERE LOCATE('mysql', Tags) > 0;
```
*Note: This is still an architectural anti-pattern! You should use a mapping table instead (Normalization - Phase 16).*

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that pads the `Emp_ID` to exactly 8 digits using leading zeros.
*   **Task 2:** Use `LOCATE` and `SUBSTRING` to extract the username part (before the `@`) of an email address.

---
