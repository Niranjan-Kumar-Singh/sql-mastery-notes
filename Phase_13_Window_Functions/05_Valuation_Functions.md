# 🏷️ Topic 13.5: Valuation Functions

Beyond ranking and moving forward/backward, sometimes you need to find a specific value within a partition, like the **First**, **Last**, or **Nth** value. These are **Valuation Functions**.

---

## 1. Definition

- **`FIRST_VALUE(col)`:** Returns the value of the specified column from the first row in the window frame.
- **`LAST_VALUE(col)`:** Returns the value from the last row in the window frame.
- **`NTH_VALUE(col, n)`:** Returns the value from the `n`-th row in the window frame.

---

## 2. Why This Concept Exists

*   **Benchmarking:** "Compare every employee's salary to the highest salary in their department."
*   **Gap Analysis:** Finding the very first order date for a customer and showing it on every order row.
*   **Data Consistency:** Identifying the "Gold Standard" row in a group of duplicates.

---

## 3. How It Works (Frame Targeting - PRO LEVEL)

These functions are highly dependent on the **Window Frame** (Topic 13.8). 
By default, the window frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This can lead to unexpected results for `LAST_VALUE()` (it will just return the value of the current row!). 

**Senior Tip:** When using `LAST_VALUE`, you almost always need to redefine the frame to include the entire partition:
```sql
LAST_VALUE(Sal) OVER(PARTITION BY Dept ORDER BY Sal ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

---

## 4. Syntax / Implementation

```sql
SELECT 
    Name, 
    Salary, 
    Department,
    FIRST_VALUE(Salary) OVER(PARTITION BY Department ORDER BY Salary DESC) AS Max_Dept_Salary
FROM Employees;
-- This adds a column showing the highest salary of the department for every employee.
```

---

## 5. Real-Life Examples

**Finding the Second Best:**
```sql
SELECT 
    Name, 
    Exam_Score,
    NTH_VALUE(Name, 2) OVER(ORDER BY Exam_Score DESC) AS Silver_Medalist
FROM Students;
```

---

## 6. Common Mistakes

*   **Null Handling:** If the N-th row doesn't exist (e.g., asking for `NTH_VALUE(..., 5)` in a partition of only 3 rows), it returns `NULL`.
*   **Ignoring the Frame:** As mentioned in Section 3, `LAST_VALUE` often behaves like the current row if the frame isn't expanded to `UNBOUNDED FOLLOWING`.

---

## 7. Tips & Best Practices (Pro-Level)

**Valuation vs. Max/Min:**
You might think `FIRST_VALUE(Salary ORDER BY DESC)` is the same as `MAX(Salary) OVER()`. 
**The Difference:** `MAX()` only returns the column it's running on. `FIRST_VALUE()` allows you to reach into the row and grab **another** column from the winner.
Example: "Show me the `Name` of the person who has the `MAX(Salary)`."

---

## 8. Mini Practice Tasks

*   **Task 1:** Write a query that shows all products and a column displaying the name of the **first** product added to each `Category`.
*   **Task 2:** Why does `LAST_VALUE()` frequently return the current row's value instead of the actual last value in the partition?

---
