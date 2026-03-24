# ⚖️ Topic 6.7: Comparison Operators (=, <, >, <=, >=, <>, !=)

The `WHERE` clause filters rows based on conditions. But what kind of conditions? The mathematical language of comparison.

Every single `WHERE` condition evaluates a comparison between a column's value and a target. MySQL provides a complete set of **Comparison Operators** that cover every logical relationship between two values.

---

## 1. Definition

**Comparison Operators** are symbols that evaluate the mathematical or lexicographic relationship between two values, returning either `TRUE` or `FALSE`. When `TRUE`, the row is included in the result; when `FALSE`, it is excluded.

---

## 2. The Complete Comparison Operator Reference

| Operator | Name | Meaning | Example |
|---|---|---|---|
| `=` | Equal To | Exact match | `WHERE Status = 'Active'` |
| `!=` | Not Equal To | Excludes a match | `WHERE Status != 'Banned'` |
| `<>` | Not Equal To | Same as `!=` (ANSI standard) | `WHERE Status <> 'Banned'` |
| `>` | Greater Than | Strictly above | `WHERE Salary > 60000` |
| `<` | Less Than | Strictly below | `WHERE Age < 18` |
| `>=` | Greater Than or Equal | At or above | `WHERE Stock >= 1` |
| `<=` | Less Than or Equal | At or below | `WHERE Price <= 999.99` |

---

## 3. Why This Concept Exists

The English language uses phrases like "more than", "at least", "no less than", etc. to describe data conditions. Comparison operators are the mathematical encoding of these natural language expressions, allowing the database to evaluate billions of rows in microseconds using pure CPU arithmetic.

---

## 4. Why We Use It

*   **Range Filtering:** `WHERE Salary BETWEEN 50000 AND 100000` internally uses `>= AND <=`.
*   **Boundary Conditions:** `WHERE Stock > 0` — only show available products.
*   **Exclusion Filtering:** `WHERE Role != 'Admin'` — show non-admin users only.

---

## 5. How It Works (String vs Numeric Comparison - PRO LEVEL)

This is a critical distinction:

**Numeric Comparisons** are pure integer/decimal arithmetic — blazingly fast.

**String Comparisons** depend on the Column's **COLLATION** setting:
- `utf8mb4_unicode_ci` (case insensitive): `'Alice' = 'alice'` evaluates to `TRUE`
- `utf8mb4_bin` (binary/byte-level): `'Alice' = 'alice'` evaluates to `FALSE`

This is why a user login might succeed whether they type `Alice` or `alice` — the `=` operator for VARCHAR columns silently runs a case-insensitive collation comparison, not a raw byte comparison!

```sql
-- Case-insensitive (default collation utf8mb4_unicode_ci)
WHERE Username = 'alice'  -- Matches 'alice', 'Alice', 'ALICE', 'aLiCe'

-- Case-sensitive (explicitly using binary collation)
WHERE Username = BINARY 'alice'  -- Matches ONLY 'alice'
```

**Date Comparisons** compare ISO-8601 formatted strings mathematically:
- `'2024-03-15' > '2024-01-01'` evaluates to `TRUE` (March is after January).

---

## 6. Syntax / Implementation

```sql
-- Equality
WHERE Status = 'Active'

-- Inequality (both are equivalent, != is more common)
WHERE Status != 'Suspended'
WHERE Status <> 'Suspended'

-- Numeric ranges
WHERE Salary >= 50000 AND Salary <= 100000

-- Date comparison
WHERE Order_Date > '2024-01-01'

-- Combined
WHERE Price > 0 AND Price <= 999.99 AND Status != 'Discontinued'
```

---

## 7. Real-Life Examples

**The Flight Booking Engine:**
```sql
-- Find available seats in economy class under $500 departing after today
SELECT Flight_ID, Departure_City, Arrival_City, Price
FROM Flights
WHERE Price <= 500.00
  AND Seat_Class = 'Economy'
  AND Departure_Date > CURDATE()
  AND Available_Seats > 0
ORDER BY Price ASC;
```
Every single condition uses a comparison operator, chained with `AND` (Topic 6.6).

---

## 8. SQL Examples (MySQL Execution)

```sql
-- 1. Employees earning exactly 75,000
SELECT First_Name FROM Employees WHERE Salary = 75000.00;

-- 2. Employees NOT in the Engineering department
SELECT First_Name, Department FROM Employees WHERE Department != 'Engineering';

-- 3. High earners above 90k
SELECT First_Name, Salary FROM Employees ORDER BY Salary DESC;

-- 4. Employees hired on or after Jan 1st 2022
SELECT First_Name, Hire_Date FROM Employees WHERE Hire_Date >= '2022-01-01';

-- 5. Products that need restocking (stock is critically low)
SELECT Product_Name, Stock FROM Products WHERE Stock <= 5;

-- 6. All products in a specific price band
SELECT Product_Name, Price FROM Products WHERE Price > 100 AND Price <= 500;
```

---

## 9. Common Mistakes

*   **Using `=` to check for NULL:** As covered in Topic 6.5, `NULL = NULL` is `NULL` (not TRUE). Always use `IS NULL` / `IS NOT NULL`.
*   **Using `!=` for NULL checking:**
    ```sql
    -- ❌ This does NOT catch NULL rows!
    WHERE Department != 'Engineering'
    -- Employees with NULL Department are excluded from results!
    
    -- ✅ Correct approach to include NULLs:
    WHERE Department != 'Engineering' OR Department IS NULL
    ```
*   **Confusing `=` (comparison) with `=` (assignment in SET):**
    - In `WHERE Status = 'Active'` — this is a comparison (returns TRUE/FALSE).
    - In `UPDATE ... SET Status = 'Active'` — this is an assignment (sets the value).

---

## 10. Tips & Best Practices (Pro-Level)

**`<>` vs `!=`: Choose a Convention and Stick to It**

Both `!=` and `<>` are identical in MySQL. However:
- `!=` is MySQL/PostgreSQL/SQL Server standard and more readable to most developers.
- `<>` is the ANSI SQL standard and more portable across all database engines (Oracle, DB2, etc.).

In enterprise environments where the same SQL might need to run on different DB vendors, use `<>`. In MySQL-only projects, `!=` is preferred for readability.

**Index Usage with Comparison Operators:**
All comparison operators on an indexed column are sargable (can use the index):
- `=`, `<`, `>`, `<=`, `>=` on `WHERE` → B-Tree Range Scan → Fast ✅
- BUT: `!=` and `<>` typically force a full table scan even on indexed columns. MySQL's optimizer usually finds it cheaper to scan the whole table than to use an index for a NOT-EQUAL query, especially when the excluded value is a small minority.

---

## 11. Mini Practice Tasks

*   **Task 1:** Write a query to find all products with a `Price` strictly greater than `200` and less than or equal to `1000`, and where the product is NOT in the `'Discontinued'` category. Use the `Employees` table structure as reference.
*   **Task 2:** Why does `WHERE Department != 'Engineering'` accidentally exclude employees who have a `NULL` value in their `Department` column? How do you write the query correctly to include those NULL rows?

---
