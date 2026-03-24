# 🔤 Topic 8.1: String Functions (Basic)

In Phase 6, we learned how to select data. But often, the raw data in the database isn't exactly how we want to display it. Strings might be lowercase when they should be uppercase, or we might need to combine two columns into one. 

MySQL provides a powerful suite of **String Functions** to manipulate text on the fly.

---

## 1. Definition

**String Functions:** Built-in SQL functions that take one or more strings as input, perform a transformation (like combining, trimming, or changing case), and return a new string as the result. 

---

## 2. Why This Concept Exists

Data is rarely "ready-to-wear." Names might be stored in separate columns (`First_Name`, `Last_Name`), but a UI needs a single `Full Name`. Strings might contain accidental leading spaces or be in the wrong case for a report. String functions allow the database to handle these transformations, saving the application from doing it in every single frontend component.

---

## 3. The Core String Functions Reference

| Function | Description | Example | Result |
|---|---|---|---|
| `CONCAT(a, b, ...)` | Joins two or more strings | `CONCAT('Hello', ' ', 'SQL')` | 'Hello SQL' |
| `CONCAT_WS(sep, a, b)` | Joins with a separator | `CONCAT_WS('-', '2024', '03', '15')` | '2024-03-15' |
| `UPPER(str)` | Converts to uppercase | `UPPER('mysql')` | 'MYSQL' |
| `LOWER(str)` | Converts to lowercase | `LOWER('ADMIN')` | 'admin' |
| `LENGTH(str)` | Returns length in **bytes** | `LENGTH('SQL')` | 3 |
| `CHAR_LENGTH(str)` | Returns length in **characters** | `CHAR_LENGTH('SQL')` | 3 |
| `SUBSTRING(s, start, len)`| Extracts a portion | `SUBSTRING('Database', 1, 4)` | 'Data' |
| `REPLACE(s, from, to)` | Replaces occurrences | `REPLACE('abc@x.com', 'x.com', 'me.com')`| 'abc@me.com' |
| `TRIM(str)` | Removes leading/trailing spaces| `TRIM('  word  ')` | 'word' |

---

## 4. How It Works (Memory & Multibyte Strings - PRO LEVEL)

This is a critical distinction for modern web development:

**`LENGTH()` vs `CHAR_LENGTH()`**
- **`LENGTH(str)`** measures the size of the string in **bytes**.
- **`CHAR_LENGTH(str)`** measures the size of the string in **characters**.

In the `utf8mb4` character set (which supports Emojis):
```sql
SELECT LENGTH('🚀'), CHAR_LENGTH('🚀');
-- Result: 4, 1
```
An Emoji takes 4 bytes of storage but is only 1 character. If you are validating "Username must be 10 characters or less," you MUST use `CHAR_LENGTH()`. If you use `LENGTH()`, an 8-character string with two emojis will exceed the limit!

---

## 5. Syntax / Implementation

```sql
-- Combining columns with a space
SELECT CONCAT(First_Name, ' ', Last_Name) AS Full_Name FROM Employees;

-- Concatenate With Separator (Handles NULLs better!)
-- If any argument in CONCAT() is NULL, the whole result is NULL.
-- CONCAT_WS() skips NULL values!
SELECT CONCAT_WS(', ', Last_Name, First_Name, City) FROM Employees;

-- Extracting first initial
SELECT SUBSTRING(First_Name, 1, 1) AS Initial FROM Employees;
```

---

## 6. Real-Life Examples

**The Email Generator:**
```sql
-- Generate user emails: first_initial + last_name @ company.com
SELECT 
    LOWER(CONCAT(SUBSTRING(First_Name, 1, 1), Last_Name, '@company.com')) AS Corporate_Email
FROM Employees;
```

---

## 7. Common Mistakes

*   **Using `CONCAT()` when data might be NULL:** As noted, `CONCAT('Hello', NULL)` returns `NULL`. This is a common bug when building address strings. Use `CONCAT_WS()` or `IFNULL()` inside your `CONCAT()`.
*   **Assuming `SUBSTRING` starts at 0:** In SQL, string indexing is **1-based**. `SUBSTRING('SQL', 1, 1)` returns 'S'. (Most programming languages like Python/JS use 0-based indexing).

---

## 8. Tips & Best Practices (Pro-Level)

**Search Performance with Functions:**
Never use a string function on the **left side** of a `WHERE` clause if you can avoid it.
```sql
-- ❌ EXTREMELY SLOW (Kills index usage on First_Name)
WHERE UPPER(First_Name) = 'ALICE'

-- ✅ FAST (Standard equality is usually case-insensitive anyway)
WHERE First_Name = 'Alice'
```
Wrapping a column in a function like `UPPER()` prevents MySQL from using a B-Tree index (Sargability).

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that returns the `Last_Name` in all uppercase and the `First_Name` with only its first three characters.
*   **Task 2:** If you have an emoji like '⚡' in a `utf8mb4` column, what will `LENGTH()` and `CHAR_LENGTH()` return? Why?

---
