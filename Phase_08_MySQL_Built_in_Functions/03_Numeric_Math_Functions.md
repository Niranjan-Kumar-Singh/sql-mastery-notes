# 🔢 Topic 8.3: Numeric & Math Functions

Databases aren't just for storing text; they are calculators. From calculating tax on a product to generating random samples for an A/B test, **Math Functions** are essential for processing numeric data.

---

## 1. Definition

**Numeric & Math Functions:** Built-in SQL functions that perform arithmetic calculations on input numbers (integer, decimal, or float) and return a numeric result.

---

## 2. The Core Math Functions Reference

| Function | Description | Example | Result |
|---|---|---|---|
| `ROUND(n, d)` | Rounds to `d` decimal places | `ROUND(15.678, 2)` | 15.68 |
| `CEIL(n)` | Rounds UP to nearest integer | `CEIL(15.1)` | 16 |
| `FLOOR(n)` | Rounds DOWN to nearest integer| `FLOOR(15.9)` | 15 |
| `ABS(n)` | Absolute value (positive) | `ABS(-42)` | 42 |
| `MOD(n, m)` | Modulo (remainder of n/m) | `MOD(10, 3)` | 1 |
| `POWER(n, m)` | `n` raised to power `m` | `POWER(2, 3)` | 8 |
| `RAND()` | Random float between 0 and 1 | `RAND()` | 0.457... |

---

## 3. Why This Concept Exists

In financial systems, you can't just display a value like `15.999999996` — you need to round it to 2 decimal places for cents. In logistical systems, if a crate can hold 10 items and you have 11 items, you need `CEIL(1.1)` to know you need 2 crates. Math belongs in the data layer where it can be applied to millions of rows simultaneously.

---

## 4. How It Works (Floating Point Inaccuracy - PRO LEVEL)

This is a critical "Gotcha" for senior engineers:

**ROUND with FLOAT/DOUBLE:** 
Because `FLOAT` and `DOUBLE` use binary scientific notation (Topic 3.2), some numbers cannot be represented exactly.
```sql
-- Theoretically: 0.1 + 0.2 = 0.3
SELECT ROUND(0.1 + 0.2, 1);
```
In some edge cases involving complex math, the floating point error might lead `ROUND` to behave unexpectedly. **Always use `DECIMAL` for currency and financial totals** to ensure exact math with these functions.

---

## 5. Real-Life Examples

**Example 1: E-commerce Tax Calculation**
```sql
-- Calculate 18% GST and round to 2 decimal places
SELECT 
    Product_Name, 
    Price,
    ROUND(Price * 0.18, 2) AS Tax_Amount,
    Price + ROUND(Price * 0.18, 2) AS Final_Price
FROM Products;
```

**Example 2: Shipping Logic (CEIL)**
Find how many shipping boxes are needed if each box holds 5 items:
```sql
SELECT 
    Order_ID, 
    Total_Items,
    CEIL(Total_Items / 5.0) AS Boxes_Required
FROM Orders;
```

---

## 6. SQL Examples (MySQL Execution)

```sql
-- 1. Handling remainders (MOD)
-- Find all 'Even' Order IDs
SELECT Order_ID FROM Orders WHERE MOD(Order_ID, 2) = 0;

-- 2. Random Sampling (RAND)
-- Pick 1 random winner from the Users table
SELECT Username FROM Users ORDER BY RAND() LIMIT 1;
-- ⚠️ WARNING: ORDER BY RAND() is O(N log N) — extremely slow on large tables!

-- 3. Generating a random integer between a range (1 to 100)
SELECT FLOOR(RAND() * 100) + 1 AS Random_Number;
```

---

## 7. Common Mistakes

*   **Confusing CEIL and ROUND:** `CEIL(10.1)` is 11, but `ROUND(10.1)` is 10. Use `CEIL` when you can never have a "partial" unit (like a shipping box or a car).
*   **Using `ROUND` too early in a calculation:** If you round every intermediate step of a long multiplication, you introduce "rounding drift" errors. Always perform full math and round only the final result for display.

---

## 8. Tips & Best Practices (Pro-Level)

**The `RAND(seed)` Deterministic Randomness:**
The `RAND()` function can take an optional "seed" integer. `RAND(5)` will return the *exact same* sequence of "random" numbers every time it's called. This is invaluable for debugging and for generating reproducible test data.

```sql
SELECT RAND(100), RAND(100); -- Both rows will have the exact same value!
```

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that calculates the `Monthly_Salary` (Salary / 12) from the `Employees` table and rounds it to the nearest whole number.
*   **Task 2:** Use the `MOD` function to identify all employees with an odd-numbered `Emp_ID`.

---
