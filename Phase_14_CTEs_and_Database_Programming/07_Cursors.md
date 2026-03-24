# 🚣 Topic 14.7: Cursors

Standard SQL works on **Sets** (all rows at once). But sometimes, you need an algorithm that behaves like an "Excel Macro" or a "Python Loop," processing rows **one by one**. This is the role of a **Cursor**.

---

## 1. Definition

**Cursor:** A database object that allows you to traverse the rows of a result set one by one, performing custom logic on each specific row.

---

## 2. Why This Concept Exists

*   **Row-Level Logic:** "For each employee, if their tenure > 5 years, calculate complex tax; else, calculate simpler tax." (When a single SQL `UPDATE` is too simple).
*   **External Integration:** Calling an external API or procedure for every row found in a search.
*   **Complex Data Transformation:** Reading data from Table A, doing 5 checks, and sparingly inserting into Tables B, C, or D.

---

## 3. The Cursor Lifecycle (The 4 Steps)

To use a cursor inside a Stored Procedure, you must follow this strict sequence:

1.  **DECLARE:** Define the query the cursor will read.
2.  **OPEN:** Execute the query and prepare to read.
3.  **FETCH:** Retrieve the "Current Row" and move the pointer to the next one.
4.  **CLOSE:** Release the memory associated with the cursor.

---

## 4. Syntax / Implementation

```sql
DELIMITER //

CREATE PROCEDURE ProcessHighSalaries()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_name VARCHAR(100);
    
    -- 1. DECLARE
    DECLARE cur1 CURSOR FOR SELECT Name FROM Employees WHERE Salary > 100000;
    
    -- Handler for when no more rows exist
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- 2. OPEN
    OPEN cur1;

    read_loop: LOOP
        -- 3. FETCH
        FETCH cur1 INTO emp_name;
        IF done THEN LEAVE read_loop; END IF;
        
        -- CUSTOM LOGIC PER ROW
        CALL LogHighEarner(emp_name);
    END LOOP;

    -- 4. CLOSE
    CLOSE cur1;
END //

DELIMITER ;
```

---

## 5. How It Works (The Memory Pointer - PRO LEVEL)

A cursor is a **Memory Pointer** to a temporary result set. 
- **Efficiency:** Cursors in MySQL are "As-and-When" (Non-Scrollable). They can only move forward, not backward.
- **Performance DANGER:** Cursors are notoriously slow. SQL engines are optimized for math on "Sets" (Columns). A Cursor forces the engine to behave like a standard CPU loop, which is significantly slower than a vectorized SQL operation.

---

## 6. Common Mistakes

*   **Forgetting the HANDLER:** If you don't declare a `CONTINUE HANDLER FOR NOT FOUND`, your loop will never end, or it will throw an error as soon as it reaches the last row.
*   **Not Closing:** Leaving cursors open consumes server RAM. Always `CLOSE` your cursor before the procedure ends.

---

## 7. Tips & Best Practices (Pro-Level)

**The Golden Rule: Avoid Cursors.**
90% of the time, a task you think requires a cursor can be done with a clever `CASE` statement inside an `UPDATE` or a `JOIN`. Only use cursors when the row-level logic is so complex that standard SQL cannot express it.

---

## 8. Mini Practice Tasks

*   **Task 1:** List the four stages of a Cursor's lifecycle.
*   **Task 2:** Why are Cursors typically discouraged for large-scale data processing in SQL?

---
