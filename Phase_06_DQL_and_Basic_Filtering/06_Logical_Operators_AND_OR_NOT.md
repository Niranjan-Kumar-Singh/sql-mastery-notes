# 🔗 Topic 6.6: Logical Operators (AND, OR, NOT)

A single `WHERE` condition often isn't enough. Real business data questions have multiple dimensions: *"Show me Engineering employees who earn more than $80,000 AND are based in Mumbai."*

To handle this, SQL provides **Logical Operators** — mathematical connectives that combine multiple `WHERE` conditions into a single compound filter.

---

## 1. Definition

**Logical Operators** are keywords that combine two or more boolean conditions into a single compound expression:

1.  **`AND`:** Both conditions must be `TRUE` simultaneously for the row to pass.
2.  **`OR`:** At least one condition must be `TRUE` for the row to pass.
3.  **`NOT`:** Inverts a condition — `TRUE` becomes `FALSE` and vice versa.

---

## 2. Why This Concept Exists

Real database queries are rarely one-dimensional. Filtering always involves intersections (`AND`), unions (`OR`), and exclusions (`NOT`). These three operators, originating directly from Boolean Algebra and Set Theory, give SQL its expressive filtering power.

---

## 3. Why We Use It

*   **`AND`:** Narrowing down results (More restrictive — fewer rows returned).
*   **`OR`:** Widening results (More inclusive — more rows returned).
*   **`NOT`:** Excluding specific categories from results.

---

## 4. Operator Truth Table

Understand these tables before writing any complex filter:

**AND:**
| Condition A | Condition B | Result |
|---|---|---|
| TRUE | TRUE | ✅ TRUE |
| TRUE | FALSE | ❌ FALSE |
| FALSE | TRUE | ❌ FALSE |
| FALSE | FALSE | ❌ FALSE |

**OR:**
| Condition A | Condition B | Result |
|---|---|---|
| TRUE | TRUE | ✅ TRUE |
| TRUE | FALSE | ✅ TRUE |
| FALSE | TRUE | ✅ TRUE |
| FALSE | FALSE | ❌ FALSE |

---

## 5. How It Works (Operator Precedence - PRO LEVEL)

This is the #1 source of logical bugs in SQL filters:

MySQL evaluates operators in this strict priority order:
1.  `NOT` (highest precedence)
2.  `AND`
3.  `OR` (lowest precedence)

This means without parentheses, `AND` always binds before `OR`:

```sql
-- What the developer MEANT:
-- "(Marketing OR Engineering) AND salary > 80000"
WHERE Department = 'Marketing' OR Department = 'Engineering' AND Salary > 80000;

-- What MySQL ACTUALLY executes (AND binds first!):
-- "Marketing OR (Engineering AND salary > 80000)"
-- This returns ALL Marketing employees regardless of salary!

-- ✅ The correct way — use parentheses to make intent explicit:
WHERE (Department = 'Marketing' OR Department = 'Engineering') AND Salary > 80000;
```

**Always use parentheses when mixing AND and OR** — rely on explicit grouping, never on implicit precedence.

---

## 6. Syntax / Implementation

```sql
-- AND: Both conditions must be true
SELECT * FROM Employees WHERE Department = 'Engineering' AND Salary > 75000;

-- OR: At least one condition must be true
SELECT * FROM Employees WHERE City = 'Mumbai' OR City = 'Delhi';

-- NOT: Invert a condition
SELECT * FROM Employees WHERE NOT Department = 'HR';

-- Combining all three with parentheses
SELECT * FROM Employees 
WHERE (Department = 'Engineering' OR Department = 'Marketing') 
  AND Salary > 60000 
  AND NOT City = 'Remote';
```

---

## 7. Real-Life Examples

**The Ride-Hailing Surge Pricing Algorithm:**
Find all drivers who are available AND located in a high-demand zone AND haven't completed a trip in the last 5 minutes:
```sql
SELECT Driver_ID, Current_Location
FROM Drivers
WHERE is_available = TRUE
  AND Zone_ID IN (5, 12, 18)
  AND Last_Trip_Completed < DATE_SUB(NOW(), INTERVAL 5 MINUTE);
```

---

## 8. SQL Examples (MySQL Execution)

```sql
-- 1. Find senior engineers with high pay
SELECT First_Name, Department, Salary
FROM Employees
WHERE Department = 'Engineering' AND Salary >= 90000;

-- 2. Find employees in either Mumbai or Bangalore
SELECT First_Name, City
FROM Employees
WHERE City = 'Mumbai' OR City = 'Bangalore';

-- 3. Find everyone EXCEPT HR
SELECT First_Name, Department
FROM Employees
WHERE NOT Department = 'HR';
-- Note: This is equivalent to: WHERE Department != 'HR' (Covered in 6.7)

-- 4. Complex compound filter
SELECT First_Name, City, Salary
FROM Employees
WHERE (City = 'Mumbai' OR City = 'Pune')
  AND Salary BETWEEN 50000 AND 100000
  AND NOT Department = 'Intern';
```

---

## 9. Common Mistakes

*   **Mixing AND and OR without parentheses:** As shown in Section 5, this is the silent logical bug that produces wrong query results that look visually plausible. Always parenthesize mixed conditions.
*   **Using `AND` with `OR` on the same column (Wrong):**
    ```sql
    -- ❌ WRONG — Department can never be BOTH 'HR' AND 'Engineering' simultaneously!
    WHERE Department = 'HR' AND Department = 'Engineering'
    -- This returns zero rows always! Use OR for same-column alternatives.
    
    -- ✅ CORRECT
    WHERE Department = 'HR' OR Department = 'Engineering'
    -- (Or better: WHERE Department IN ('HR', 'Engineering') — see Topic 7.2!)
    ```

---

## 10. Tips & Best Practices (Pro-Level)

**Short-Circuit Evaluation:**
MySQL evaluates `AND` and `OR` conditions using short-circuit logic:
- For `AND`: If the **first condition** is `FALSE`, MySQL skips the second condition entirely.
- For `OR`: If the **first condition** is `TRUE`, MySQL skips the second condition entirely.

*Performance Tip:* Always put the **most selective** (eliminates the most rows) or **cheapest** (uses an index) condition first in an `AND` chain. This maximizes short-circuit skipping and minimizes unnecessary evaluations.

---

## 11. Mini Practice Tasks

*   **Task 1:** Write a query to find all employees who are in the `'Engineering'` department AND have a salary greater than `70,000` AND are NOT based in `'Remote'`.
*   **Task 2:** What is the danger of writing `WHERE City = 'Mumbai' OR City = 'Delhi' AND Salary > 50000`? Rewrite it correctly with explicit parentheses.

---
