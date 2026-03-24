# 📏 Topic 7.1: The BETWEEN Operator (Inclusive Ranges)

In Topic 6.5 we learned that ranges can be expressed using `AND`: `WHERE Salary >= 50000 AND Salary <= 100000`. This works perfectly, but it is verbose. MySQL provides a cleaner, more readable shorthand for exactly this purpose: **`BETWEEN`**.

---

## 1. Definition

**`BETWEEN A AND B`:** A range operator that checks whether a value falls within an inclusive boundary — meaning the boundary values themselves (`A` and `B`) are included in the match.

It is syntactic sugar equivalent to: `column >= A AND column <= B`.

---

## 2. Why This Concept Exists

Long `AND`-chained range queries are readable for engineers but verbose in complex SQL. `BETWEEN` was standardized in ANSI SQL to make range filtering self-documenting and concise.

---

## 3. The Inclusive Boundary Rule (Critical!)

`BETWEEN` is always **inclusive on both ends**:
- `WHERE Salary BETWEEN 50000 AND 100000` matches salaries of **exactly** 50,000, exactly 100,000, and everything in between.
- There is no "exclusive" version of `BETWEEN`. If you need exclusive boundaries, use `>` and `<` directly.

---

## 4. How It Works (Index Range Scan - PRO LEVEL)

When the column in a `BETWEEN` condition is indexed, MySQL performs an efficient **Index Range Scan**:

1.  MySQL navigates the B-Tree index to find the leaf node matching the lower bound (`A`).
2.  It then **sequentially reads forward** through the B-Tree leaf nodes until it hits a value exceeding the upper bound (`B`).
3.  It stops and returns the matched range.

This is dramatically faster than a full table scan because only a contiguous slice of the B-Tree is read.

---

## 5. Syntax / Implementation

```sql
-- Numeric range (Inclusive)
SELECT * FROM table WHERE column BETWEEN low_value AND high_value;

-- Negation: Rows OUTSIDE the range
SELECT * FROM table WHERE column NOT BETWEEN low_value AND high_value;

-- Date range (Extremely common in real applications)
SELECT * FROM Orders WHERE Order_Date BETWEEN '2024-01-01' AND '2024-12-31';
```

---

## 6. Real-Life Examples

**The Hotel Booking System:**
Find all available rooms priced between ₹3,000 and ₹8,000 per night:
```sql
SELECT Room_ID, Room_Type, Price_Per_Night
FROM Hotel_Rooms
WHERE Price_Per_Night BETWEEN 3000 AND 8000
  AND is_available = TRUE
ORDER BY Price_Per_Night ASC;
```

---

## 7. SQL Examples (MySQL Execution)

```sql
-- 1. Salary range
SELECT First_Name, Salary
FROM Employees
WHERE Salary BETWEEN 50000 AND 90000;

-- 2. Date range (Financial Year 2023-24)
SELECT Order_ID, Order_Date, Total_Amount
FROM Orders
WHERE Order_Date BETWEEN '2023-04-01' AND '2024-03-31';

-- 3. NOT BETWEEN (Exclude a range)
SELECT Product_Name, Price
FROM Products
WHERE Price NOT BETWEEN 100 AND 500;

-- 4. Character range (Alphabetical — less common but valid)
SELECT First_Name FROM Employees WHERE First_Name BETWEEN 'A' AND 'M';
-- Returns names starting with A through M alphabetically (depends on collation)
```

---

## 8. Common Mistakes

*   **The Date Boundary Trap:** If you use `BETWEEN '2024-01-01' AND '2024-12-31'` on a `DATETIME` column (not just `DATE`), the upper bound `'2024-12-31'` is treated as `'2024-12-31 00:00:00'`. Any records from December 31st afternoon/evening are excluded! 

    ```sql
    -- ❌ Misses Dec 31st records after midnight
    WHERE Order_DateTime BETWEEN '2024-01-01' AND '2024-12-31'
    
    -- ✅ Correct for DATETIME columns
    WHERE Order_DateTime BETWEEN '2024-01-01 00:00:00' AND '2024-12-31 23:59:59'
    -- Or even better:
    WHERE Order_DateTime >= '2024-01-01' AND Order_DateTime < '2025-01-01'
    ```

---

## 9. Tips & Best Practices (Pro-Level)

**Always verify the order of bounds:** `BETWEEN A AND B` requires `A <= B`. If you accidentally write `BETWEEN 100 AND 50`, MySQL returns zero rows — it doesn't automatically swap them.

---

## 10. Mini Practice Tasks

*   **Task 1:** Write a query to find all employees hired between January 1, 2021 and December 31, 2023. Return `First_Name` and `Hire_Date`.
*   **Task 2:** Explain why `WHERE Order_DateTime BETWEEN '2024-01-01' AND '2024-12-31'` can miss orders placed on December 31st, 2024 at 3:00 PM.

---
