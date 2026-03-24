# 🧩 Topic 7.6: Regular Expressions in MySQL (REGEXP / RLIKE)

`LIKE` gives us simple, readable wildcard patterns. But what if you need to validate that a phone number has exactly 10 digits? Or that an email has a proper structure? Or filter any string containing a digit?

`LIKE` cannot do this. We need **Regular Expressions (REGEXP)**.

---

## 1. Definition

**`REGEXP` (or its alias `RLIKE`):** A powerful string-matching operator that evaluates a column value against a full **Regular Expression pattern** — a mathematical mini-language for describing complex text patterns.

---

## 2. Why This Concept Exists

`LIKE` only has two wildcards (`%` and `_`). Regular Expressions provide dozens of pattern primitives — character classes, quantifiers, anchors, alternation — enabling arbitrarily complex string validation directly inside SQL without application code.

---

## 3. The Core REGEXP Syntax Reference

| Pattern | Meaning | Example |
|---|---|---|
| `.` | Any single character | `'a.c'` matches 'abc', 'axc' |
| `^` | Start of string | `'^John'` matches strings starting with 'John' |
| `$` | End of string | `'gmail\\.com$'` matches strings ending with 'gmail.com' |
| `*` | 0 or more of previous | `'ab*c'` matches 'ac', 'abc', 'abbc' |
| `+` | 1 or more of previous | `'ab+c'` matches 'abc', 'abbc' but not 'ac' |
| `?` | 0 or 1 of previous | `'colou?r'` matches 'color' and 'colour' |
| `[abc]` | Any one of listed chars | `'[aeiou]'` matches any vowel |
| `[^abc]` | Any char NOT in list | `'[^0-9]'` matches any non-digit |
| `[0-9]` | Any digit | `'[0-9]{10}'` matches exactly 10 digits |
| `{n}` | Exactly n repetitions | `'[0-9]{6}'` matches exactly 6 digits |
| `{n,m}` | Between n and m reps | `'[a-z]{3,6}'` matches 3 to 6 lowercase letters |
| `\|` | OR (alternation) | `'cat\|dog'` matches 'cat' or 'dog' |

---

## 4. How It Works (NFA Engine - PRO LEVEL)

MySQL's `REGEXP` uses a **Non-deterministic Finite Automaton (NFA)** engine (similar to Perl regex). It:
1.  Compiles the regex pattern into a state machine.
2.  Runs each row's column value through the state machine.
3.  Returns TRUE if the state machine reaches an accepting state.

**Performance Caveat:** `REGEXP` always performs a **full table scan** — there is no way for a B-Tree index to accelerate it. For large tables, `REGEXP` should be used as a last resort or applied after initial filtering with sargable conditions.

---

## 5. Syntax / Implementation

```sql
-- Basic REGEXP
SELECT * FROM table WHERE column REGEXP 'pattern';

-- Case sensitivity: REGEXP is case-insensitive by default
-- For case-sensitive:
SELECT * FROM table WHERE column REGEXP BINARY 'Pattern';

-- NOT REGEXP: Exclude matching rows
SELECT * FROM table WHERE column NOT REGEXP 'pattern';
```

---

## 6. Real-Life Examples

**Data Quality Validation:**
```sql
-- Find all rows where the phone number is NOT exactly 10 digits
SELECT * FROM Users WHERE Phone NOT REGEXP '^[0-9]{10}$';

-- Find emails that appear malformed (no @ or no dot after @)
SELECT * FROM Users WHERE Email NOT REGEXP '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$';
```

---

## 7. SQL Examples (MySQL Execution)

```sql
-- 1. Names starting with a vowel
SELECT First_Name FROM Employees WHERE First_Name REGEXP '^[AEIOUaeiou]';

-- 2. Products containing a number anywhere in the name
SELECT Product_Name FROM Products WHERE Product_Name REGEXP '[0-9]';

-- 3. Email addresses that end with .com OR .in
SELECT Email FROM Users WHERE Email REGEXP '\\.(com|in)$';

-- 4. Names that are exactly 4 characters long
SELECT First_Name FROM Employees WHERE First_Name REGEXP '^.{4}$';

-- 5. Match either 'color' or 'colour' (British/American spelling)
SELECT * FROM Articles WHERE Content REGEXP 'colou?r';
```

---

## 8. Common Mistakes

*   **Using REGEXP to replace proper schema design:** If you need to validate that a phone column contains only digits, the correct architectural decision is to store them as `VARCHAR` with a `CHECK` constraint, not to filter with REGEXP at query time.
*   **Escaping in MySQL REGEXP:** In MySQL REGEXP, the backslash must be doubled because MySQL itself interprets one level of backslashes in strings. To match a literal dot: use `'\\.'` (not `'\.'`).

---

## 9. Tips & Best Practices (Pro-Level)

**REGEXP vs LIKE — When to Use Which:**
- Use `LIKE` when the pattern is simple (starts with, ends with, contains keyword).
- Use `REGEXP` only for complex structural patterns (valid email format, exactly N digits, alternation between multiple options).
- Both force full table scans with leading wildcards — prefer indexed exact/prefix lookups when performance is critical.

---

## 10. Mini Practice Tasks

*   **Task 1:** Write a REGEXP query to find all employees whose `First_Name` starts with either `'A'` or `'S'`.
*   **Task 2:** Why does `REGEXP` always cause a full table scan, even when the target column is indexed?

---
