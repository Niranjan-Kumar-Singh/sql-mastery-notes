# 🧮 Topic 14.5: User-Defined Functions (UDFs)

Stored Procedures are for **Actions**. **User-Defined Functions (UDFs)** are for **Calculations**. They allow you to extend the MySQL library with your own custom logic that can be used directly inside a `SELECT` statement.

---

## 1. Definition

**UDF:** A stored routine that returns a **single value**. Unlike procedures, functions can be called anywhere a built-in function (like `UPPER()` or `ROUND()`) can be used.

---

## 2. Why This Concept Exists

*   **Formula Standardization:** "Calculate the GST Tax on this amount."
*   **Business Logic in SELECT:** "Convert this numeric score into a letter grade (A, B, C)."
*   **Cleaner Queries:** Instead of repeating a 10-line `CASE` statement in every report, you wrap it in a function.

---

## 3. Function vs. Procedure (The Core Difference)

| Feature | Stored Procedure | UDF (Function) |
|---|---|---|
| **Return** | Can return 0, 1, or many results. | Must return exactly one value. |
| **Usage** | Called with `CALL`. | Used in `SELECT`, `WHERE`, `JOIN`. |
| **Parameters**| `IN`, `OUT`, `INOUT`. | Only `IN` parameters allowed. |
| **Actions** | Can modify tables (`INSERT/UPDATE`). | Usually Read-Only (Safe). |

---

## 4. Syntax / Implementation

```sql
DELIMITER //

CREATE FUNCTION GetTaxAmount(price DECIMAL(10,2)) 
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN price * 0.18;
END //

DELIMITER ;

-- Usage:
SELECT Product_Name, GetTaxAmount(Price) FROM Products;
```

---

## 5. DETERMINISTIC vs. NOT DETERMINISTIC (PRO LEVEL)

When creating a function, you **must** specify its nature for the binary logger:
- **DETERMINISTIC:** The function always returns the same result for the same input (e.g., `price * 0.18`).
- **NOT DETERMINISTIC:** The result can change even if the input stays the same (e.g., a function that returns `NOW()` or reads from a table).

**Why this matters:** If a function is deterministic, MySQL can cache the result. If it's not, MySQL must re-calculate it for every single row, which is slower.

---

## 6. Real-Life Examples

**The "Credit Level" Calculator:**
```sql
CREATE FUNCTION CreditLevel(amount INT) 
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    IF amount > 50000 THEN RETURN 'PLATINUM';
    ELSEIF amount > 20000 THEN RETURN 'GOLD';
    ELSE RETURN 'SILVER';
    END IF;
END //
```

---

## 7. Common Mistakes

*   **Forgetting to Return:** Every path in your function logic (every `IF/ELSE`) MUST eventually lead to a `RETURN` statement.
*   **Trying to perform DML:** You generally cannot run `INSERT`, `UPDATE`, or `DELETE` inside a function. Use a procedure for that.

---

## 8. Tips & Best Practices (Pro-Level)

**Built-in vs. UDF Performance:**
Never create a UDF for something MySQL already does (like `CONCAT` or `DATE_DIFF`). Native C++ functions in the MySQL engine are 10-20x faster than custom UDFs. Only build a UDF for unique business logic.

---

## 9. Mini Practice Tasks

*   **Task 1:** What is the primary difference between a Stored Procedure and a User-Defined Function?
*   **Task 2:** Write a function `GetFullName` that takes `FirstName` and `LastName` as parameters and returns them concatenated with a space in between.

---
