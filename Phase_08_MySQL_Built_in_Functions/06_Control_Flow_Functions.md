# 🚦 Topic 8.6: Control Flow Functions

Sometimes you need "IF/THEN" logic in your SQL. If a value is NULL, show "N/A". If one value equals another, return NULL. **Control Flow Functions** allow the database to make basic logic decisions as it processes each row.

---

## 1. Definition

**Control Flow Functions:** SQL functions that evaluate conditions and return different values based on those conditions. They allow for "procedural" logic inside a declarative query.

---

## 2. Core Control Flow Reference

| Function | Description | Example | Result |
|---|---|---|---|
| `IF(expr, t, f)` | If `expr` is true, return `t`, else `f` | `IF(1>2, 'Yes', 'No')` | 'No' |
| `IFNULL(val, default)`| Returns `default` if `val` is NULL | `IFNULL(NULL, 0)` | 0 |
| `NULLIF(a, b)` | Returns NULL if `a=b`, else returns `a` | `NULLIF(5, 5)` | NULL |
| `COALESCE(a, b, ...)` | Returns the first non-NULL value | `COALESCE(NULL, 5, 2)`| 5 |

---

## 3. Why This Concept Exists

Data is messy. 
*   **`IFNULL`** prevents apps from crashing when they try to display a NULL name or salary.
*   **`NULLIF`** prevents "Division by Zero" errors by turning 0 into NULL before a divide.
*   **`IF`** allows for quick categorization (e.g., classifying salary as "High" or "Low") without using the more verbose `CASE`.

---

## 4. How It Works (Short-Circuit Logic - PRO LEVEL)

A critical point for both `IF` and `COALESCE` is **Short-Circuit Evaluation**:
MySQL evaluates the arguments from left to right and **stops** as soon as it finds the answer.

```sql
SELECT COALESCE('Found', (SELECT complex_expensive_query));
-- MySQL will NEVER run the expensive subquery because the first argument is already non-NULL.
```
This is a vital performance optimization for senior developers when dealing with multi-step fallback logic.

---

## 5. Real-Life Examples

**Example 1: The "N/A" Fallback**
```sql
SELECT 
    First_Name, 
    IFNULL(Phone, 'No Phone Provided') AS Contact_Info
FROM Employees;
```

**Example 2: Preventing Division by Zero**
```sql
-- If Sold_Units is 0, NULLIF turns it to NULL. 
-- Any number / NULL = NULL (prevents crash).
SELECT 
    Total_Revenue / NULLIF(Sold_Units, 0) AS Price_Per_Unit
FROM Sales;
```

**Example 3: Hierarchical Fallback (COALESCE)**
```sql
-- Find first available contact method
SELECT 
    First_Name, 
    COALESCE(Work_Phone, Mobile_Phone, Email, 'Untraceable') AS Preferred_Contact
FROM Employees;
```

---

## 6. COALESCE vs IFNULL (The Pro Choice)

*   `IFNULL(a, b)` takes only **two** arguments. It is MySQL-specific.
*   `COALESCE(a, b, c, ...)` takes **unlimited** arguments. It is **ANSI SQL Standard**.

**Senior Tip:** Always prefer `COALESCE`. It is more powerful and makes your code portable to other databases like PostgreSQL or SQL Server.

---

## 7. Common Mistakes

*   **Using `IF` for complex branching:** While `IF()` is great for simple "one or the other" logic, it becomes unreadable when nested. For 3 or more conditions, use `CASE` (Topic 8.7).
*   **Assuming `IFNULL` works on empty strings:** `IFNULL('', 'N/A')` returns `''` (the empty string). To MySQL, an empty string is NOT NULL. Use `NULLIF` combined with `COALESCE` to handle this: `COALESCE(NULLIF(col, ''), 'N/A')`.

---

## 8. Tips & Best Practices (Pro-Level)

**Logical Mapping with IF:**
```sql
-- Quickly flag status
SELECT 
    Username, 
    IF(is_active = 1, '✅ Active', '❌ Deactivated') AS Status
FROM Users;
```

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that returns the `City` of an employee, but if the city is NULL, return `'Remote'`.
*   **Task 2:** Use `COALESCE` to return the first non-NULL value among columns `Bonus`, `Commission`, and `0`.

---
