# 📝 Topic 14.4: Stored Procedures (IN, OUT, INOUT)

Writing queries one-by-one is fine for basic tasks. But for complex business logic (like "Process an Order" which involves 5 tables), you need to group related SQL statements into a single, reusable unit. This is a **Stored Procedure**.

---

## 1. Definition

**Stored Procedure:** A prepared SQL code that you can save so the code can be reused over and over again. It is stored on the database server and can take input parameters and return output values.

---

## 2. Why This Concept Exists

*   **Logic Centralization:** Instead of writing complex "Order Logic" in 3 different apps (Java, Python, PHP), you write it once in the database.
*   **Security:** You can give a user permission to **run** a procedure but **deny** them permission to see the underlying tables.
*   **Performance:** Procedures are pre-compiled by the database engine, reducing network traffic and execution time.

---

## 3. The Three Types of Parameters

Unlike simple scripts, procedures use parameters to talk to the "Outside World":

| Parameter | Function |
|---|---|
| **`IN`** | (Default) Passes a value from the user into the procedure. |
| **`OUT`** | Returns a value from the procedure back to the user. |
| **`INOUT`** | Passes a value in, modifies it, and sends it back. |

---

## 4. Syntax / Implementation

Because procedures contain multiple semicolons, we must change the **DELIMITER** so MySQL knows where the procedure ends.

```sql
DELIMITER //

CREATE PROCEDURE GetEmpByDept(IN dept_id INT, OUT total_emps INT)
BEGIN
    SELECT COUNT(*) INTO total_emps 
    FROM Employees 
    WHERE Department_ID = dept_id;
    
    SELECT * FROM Employees WHERE Department_ID = dept_id;
END //

DELIMITER ;
-- Calling the procedure:
CALL GetEmpByDept(10, @count);
SELECT @count;
```

---

## 5. How It Works (The Execution Plan Cache - PRO LEVEL)

When a procedure is created:
1.  MySQL parses the SQL and checks for syntax errors.
2.  It creates an **optimized execution plan** for the queries inside.
3.  When you `CALL` the procedure, MySQL simply re-runs the existing plan rather than re-optimizing from scratch. This makes procedures significantly faster for complex logic.

---

## 6. Real-Life Examples

**The "Secure Insert" Pattern:**
Allowing a user to add a product without giving them `INSERT` permissions on the table.
```sql
CREATE PROCEDURE AddProduct(IN name VARCHAR(100), IN price DECIMAL(10,2))
BEGIN
    IF price > 0 THEN
        INSERT INTO Products(Product_Name, Price) VALUES (name, price);
    END IF;
END //
```

---

## 7. Common Mistakes

*   **Forgetting the Delimiter:** If you don't change the delimiter, MySQL will stop at the first semicolon inside the code and throw a syntax error.
*   **Parameter Name Clashes:** Don't name your parameter the same as your column name (e.g., `IN ID INT` vs `WHERE ID = ID`). MySQL will get confused! Use prefixes like `p_ID`.

---

## 8. Tips & Best Practices (Pro-Level)

**Minimal Procedures:**
Don't put thousands of lines of code in one procedure. Think of procedures like functions in Python—keep them small, focused, and give them descriptive names. Use Procedures for **Actions** (Delete User, Process Refund) and check out **Functions** (Topic 14.5) for **Calculations**.

---

## 9. Mini Practice Tasks

*   **Task 1:** What is the role of the `DELIMITER` command when creating a stored procedure?
*   **Task 2:** Write a simple stored procedure `CheckStock` that takes a `Product_ID` and returns the `Stock_Count` as an `OUT` parameter.

---
