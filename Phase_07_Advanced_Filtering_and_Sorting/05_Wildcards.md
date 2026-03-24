# 🃏 Topic 7.5: Wildcards (% and _)

The `LIKE` operator alone isn't very useful — it needs **Wildcards** to work. Wildcards are special placeholder characters that stand in for unknown characters in a pattern. MySQL provides exactly two.

---

## 1. The Two MySQL Wildcards

| Wildcard | What it represents | Example Pattern | Example Matches |
|---|---|---|---|
| **`%`** (Percent) | **Zero, one, or many** characters (any length) | `'A%'` | `'Apple'`, `'A'`, `'Air Force'` |
| **`_`** (Underscore) | **Exactly one** character | `'B_y'` | `'Boy'`, `'Bay'`, `'Buy'` — NOT `'By'` or `'Body'` |

---

## 2. Pattern Reference

Memorize these common patterns with their meanings:

| Pattern | Meaning | Example Matches | Does NOT Match |
|---|---|---|---|
| `'a%'` | Starts with 'a' | `apple`, `air`, `a` | `banana`, `pear` |
| `'%a'` | Ends with 'a' | `banana`, `area`, `india` | `apple`, `orange` |
| `'%or%'` | Contains 'or' anywhere | `orange`, `fork`, `word` | `apple`, `mango` |
| `'_r%'` | 'r' is the second character | `tree`, `iron`, `army` | `root`, `river` |
| `'a_%'` | Starts with 'a' and at least 2 chars long | `apple`, `at` | `a` (only 1 char) |
| `'a%o'` | Starts with 'a' and ends with 'o' | `audio`, `auto`, `avocado` | `apple`, `art` |
| `'___'` | Exactly 3 characters | `cat`, `dog`, `SQL` | `at`, `java` |
| `'A__'` | Starts with 'A', exactly 3 chars total | `Ali`, `Amy`, `Art` | `Alice`, `A` |

---

## 3. Real-Life Examples

**1. Find all customers from cities starting with 'New':**
```sql
SELECT * FROM Customers WHERE City LIKE 'New%';
-- Matches: New York, New Delhi, Newcastle, Newark
```

**2. Find all users with exactly 5-character usernames:**
```sql
SELECT * FROM Users WHERE Username LIKE '_____';
-- Matches: 'admin', 'ninja', 'alice' (5 underscores = exactly 5 chars)
```

**3. Find product codes in the format 'AB-###' (AB-dash-three digits):**
```sql
SELECT * FROM Products WHERE Code LIKE 'AB-___';
-- Matches: AB-100, AB-999, AB-XYZ
```

**4. Find all emails NOT from common providers (contains a dot before 'com'):**
```sql
SELECT * FROM Users WHERE Email LIKE '%.%.com';
-- Matches: user@mail.example.com (has two dots before end)
```

**5. Find employees hired in the year 2023:**
```sql
SELECT * FROM Employees WHERE Hire_Date LIKE '2023%';
-- Matches: 2023-01-15, 2023-12-31 (if stored as YYYY-MM-DD string)
```

---

## 4. Combining `%` and `_` Together

You can freely combine both wildcards in a single pattern for more precise control:

```sql
-- Find all plates: exactly 7 chars, starts with 'DL', ends with 'G'
SELECT * FROM Vehicles WHERE Plate_Number LIKE 'DL____G';
-- DL (2 fixed) + ____ (4 wild chars) + G (1 fixed) = 7 total

-- Find all 10-digit phone numbers starting with 98:
SELECT * FROM Contacts WHERE Phone LIKE '98________';
-- 98 + 8 underscores = 10 digits total
```

---

## 5. The Escape Character (PRO LEVEL)

What if your data literally contains a `%` or `_` character? Since these are wildcards, MySQL would misinterpret them. You must **escape** them.

**Default escape: Backslash `\`**

```sql
-- Find products with '50%' in the name (like "Discount 50%")
SELECT * FROM Products WHERE Name LIKE '%50\%%';
--                                          ^^^ The \% is a literal percent sign

-- Find columns with an underscore literally (not as a wildcard)
SELECT * FROM Files WHERE Filename LIKE '%\_backup%';
--                                        ^^ Literal underscore
```

**Custom Escape Character (using `ESCAPE` keyword):**

```sql
-- Use '#' as the escape character instead of '\'
SELECT * FROM Products WHERE Name LIKE '%50#%%' ESCAPE '#';
```

Using a custom escape character is cleaner in cases where backslashes already have meaning in your programming language strings.

---

## 6. Performance Notes

| Pattern Type | Index Used? | Speed |
|---|---|---|
| `'Prefix%'` | ✅ Yes (B-Tree prefix scan) | Fast |
| `'_____'` (exact length) | ❌ No | Full scan |
| `'%Suffix'` | ❌ No | Full scan |
| `'%Middle%'` | ❌ No | Full scan |

**Rule:** Whenever possible, put the known characters at the **beginning** of the pattern (`'prefix%'`) so MySQL can use an index. Any pattern that starts with `%` or `_` forces a full table scan.

---

## 7. Common Mistakes

- **Confusing `%` and `_`:** `'a%'` matches strings starting with 'a' of any length. `'a_'` only matches strings starting with 'a' with exactly 2 characters total.
- **Using too many underscores:** `LIKE '_____'` can be hard to count and maintain. If you need exact-length validation, a `CHAR_LENGTH()` check in the `WHERE` clause is clearer.
- **Not escaping special characters in user input:** If your application passes user input directly into `LIKE`, a malicious user could type `%` or `_` to match more rows than expected. Always sanitize inputs.

---

## 8. Tips & Best Practices

- Prefer `%prefix` searches over `%suffix%` or `%middle%` patterns. They're dramatically faster on indexed columns.
- When you find yourself building very complex wildcard patterns, consider using **REGEXP** (Topic 7.6) — it's more powerful and readable for complex string validation.
- PostgreSQL uses `ILIKE` for case-insensitive LIKE; MySQL uses `LIKE` with collation. Don't mix up the syntax when switching databases.

---

## 9. Mini Practice Tasks

- **Task 1:** Write a pattern using wildcards to find all Canadian postal codes that start with 'M5' and are exactly 6 characters long (e.g., `M5V2A1`).
- **Task 2:** Write a query to find all `Product_Name` values that contain a literal underscore `_` character (for example, "USB_Cable").
- **Task 3:** What is the difference between `LIKE 'X%'` and `LIKE 'X_'`? Write an example where one matches a value but the other doesn't.

---
