# 📊 Topic 8.7: The CASE Statement

When `IF` logic is too simple, we need a robust branching structure that can handle multiple conditions precisely. The **`CASE` statement** is SQL's version of the `if-else if-else` or `switch` statement found in modern programming.

---

## 1. Definition

**`CASE` Statement:** A multi-conditional expression that evaluates a list of conditions and returns a value as soon as a condition is satisfied. It comes in two forms: **Simple CASE** and **Searched CASE**.

---

## 2. Syntax / Implementation

### Form 1: Simple CASE (Like a Switch statement)
Used for checking a single column against specific values.
```sql
CASE column_name
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE default_result
END
```

### Form 2: Searched CASE (The most powerful form)
Used for complex range or boolean conditions.
```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
```

---

## 3. Why This Concept Exists

Business logic often involves "Bucketing":
- Salaries: `< 50k` is Junior, `50-100k` is Mid, `> 100k` is Senior.
- Order Status: `0` is Pending, `1` is Shipped, `2` is Delivered.
`CASE` allows the database to perform this categorization on-the-fly, returning human-readable labels instead of raw numbers.

---

## 4. How It Works (The Order of Evaluation - PRO LEVEL)

The `CASE` statement stops evaluation at the **first** condition that is TRUE.

```sql
SELECT 
    CASE 
        WHEN Salary > 50000 THEN 'Pro'
        WHEN Salary > 80000 THEN 'Expert' -- This will NEVER be reached!
    END
```
Anything over 80,000 is also over 50,000. So every row will be tagged as 'Pro' and the 'Expert' logic will never execute. **Always order your conditions from most specific to most general.**

---

## 5. Real-Life Examples

**Example 1: User Level Classification (Searched CASE)**
```sql
SELECT 
    Username, 
    Total_Orders,
    CASE 
        WHEN Total_Orders >= 50 THEN '🥇 Elite'
        WHEN Total_Orders >= 20 THEN '🥈 Frequent'
        WHEN Total_Orders >= 5  THEN '🥉 Occasional'
        ELSE '🆕 New Customer'
    END AS Customer_Tier
FROM Users;
```

**Example 2: Mapping Numeric Status Codes (Simple CASE)**
```sql
SELECT 
    Order_ID, 
    CASE status_code
        WHEN 0 THEN 'Pending'
        WHEN 1 THEN 'Processing'
        WHEN 2 THEN 'Shipped'
        ELSE 'Unknown'
    END AS Status_Label
FROM Orders;
```

---

## 6. Advanced Use: Conditional Aggregation

`CASE` isn't just for displays. It is used to pivot data during aggregations (Phase 9 preview):

```sql
-- Count how many high vs low earners in one row
SELECT 
    SUM(CASE WHEN Salary >= 100000 THEN 1 ELSE 0 END) AS High_Earners,
    SUM(CASE WHEN Salary < 100000 THEN 1 ELSE 0 END) AS Regular_Earners
FROM Employees;
```

---

## 7. Common Mistakes

*   **Forgetting the `END` keyword:** `CASE` must always finish with `END`.
*   **Order of conditions:** As noted in Section 4, putting broader conditions before specific ones leads to logical bugs.
*   **Forgetting the `ELSE` branch:** If no condition is met and there is no `ELSE`, `CASE` returns `NULL`. Always provide an `ELSE` for defensive coding.

---

## 8. Tips & Best Practices (Pro-Level)

**Using CASE in ORDER BY:**
You can use `CASE` to perform "Custom Sorting" that isn't alphabetical:

```sql
-- Sort by Priority: High first, then Medium, then Low
SELECT * FROM Tasks
ORDER BY 
    CASE Priority
        WHEN 'High' THEN 1
        WHEN 'Medium' THEN 2
        WHEN 'Low' THEN 3
    END ASC;
```

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query using a `Searched CASE` statement to classify employees into three categories based on their `Salary`: `'Low'` (< 40k), `'Medium'` (40k-80k), and `'High'` (> 80k).
*   **Task 2:** Write a query that uses a `Simple CASE` to map `Department` codes (e.g., `'ENG'` to `'Engineering'`, `'MKT'` to `'Marketing'`).

---
