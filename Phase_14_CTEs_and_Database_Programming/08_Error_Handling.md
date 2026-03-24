# 🚨 Topic 14.8: Error Handling in MySQL (DECLARE HANDLER)

In any real-world stored procedure, things can go wrong. A duplicate entry, a foreign key violation, or a failed connection will crash your stored procedure mid-execution. **Error Handling** lets you catch these failures and respond gracefully instead of crashing.

---

## 1. The Problem Without Error Handling

Consider this stored procedure:

```sql
CREATE PROCEDURE AddUser(IN p_email VARCHAR(100))
BEGIN
    INSERT INTO Users(Email) VALUES (p_email);
    INSERT INTO User_Profiles(Email) VALUES (p_email);
END;
```

If the first `INSERT` succeeds but the second fails (e.g., duplicate), your procedure crashes halfway through — leaving an orphaned `Users` row with no `Profile`. This is data corruption.

---

## 2. Definition

**`DECLARE ... HANDLER`:** A MySQL statement used inside Stored Procedures to catch specific errors or conditions and execute a recovery block of code, rather than letting the error propagate and crash the procedure.

---

## 3. Key Terminology

| Term | What It Means |
|---|---|
| **SQLEXCEPTION** | Any general SQL error (catches most errors). |
| **SQLWARNING** | Any SQL warning (non-fatal, like data truncation). |
| **NOT FOUND** | Fired when a `SELECT INTO` or `CURSOR FETCH` returns no data. |
| **SQLSTATE** | A 5-character error code (e.g., `'23000'` = duplicate entry). |
| **CONTINUE** | After the handler runs, execution continues from the next statement. |
| **EXIT** | After the handler runs, the `BEGIN...END` block exits immediately. |

---

## 4. Basic Syntax

```sql
DECLARE handler_type HANDLER 
FOR condition_name
handler_statement;
```

- **`handler_type`**: `CONTINUE` or `EXIT`
- **`condition_name`**: `SQLEXCEPTION`, `SQLWARNING`, `NOT FOUND`, or a specific `SQLSTATE 'XXXXX'`
- **`handler_statement`**: What to do when caught (can be a single statement or `BEGIN...END` block)

---

## 5. Real-Life Examples

### Example 1: Catching a Duplicate Entry (`EXIT` Handler)

```sql
DELIMITER //

CREATE PROCEDURE SafeInsertUser(IN p_email VARCHAR(100), IN p_name VARCHAR(100))
BEGIN
    -- Declare a variable to track if an error happened
    DECLARE exit_flag INT DEFAULT 0;
    
    -- Handler for duplicate key error (SQLSTATE 23000)
    DECLARE EXIT HANDLER FOR SQLSTATE '23000'
    BEGIN
        SELECT 'ERROR: A user with this email already exists.' AS Message;
    END;
    
    -- If we get here, no duplicate exists — proceed with the insert
    INSERT INTO Users(Email, Name) VALUES (p_email, p_name);
    SELECT 'SUCCESS: User added.' AS Message;
    
END //

DELIMITER ;

-- Call it:
CALL SafeInsertUser('alice@gmail.com', 'Alice');   -- Works fine first time
CALL SafeInsertUser('alice@gmail.com', 'Alice B'); -- Shows friendly error, doesn't crash!
```

---

### Example 2: `NOT FOUND` Handler for Cursor Loops

When you use a **Cursor** (Topic 14.7) and reach the end of data, MySQL fires a `NOT FOUND` condition. Without a handler, it becomes an error.

```sql
DELIMITER //

CREATE PROCEDURE ProcessAll()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE cur_name VARCHAR(100);
    
    DECLARE cursor_emp CURSOR FOR SELECT Name FROM Employees;
    
    -- When cursor hits the last row, set 'done' to TRUE instead of crashing
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN cursor_emp;
    read_loop: LOOP
        FETCH cursor_emp INTO cur_name;
        IF done THEN
            LEAVE read_loop;   -- Cleanly exits the loop
        END IF;
        -- Process cur_name here...
    END LOOP;
    CLOSE cursor_emp;
END //

DELIMITER ;
```

---

### Example 3: General Exception with Rollback (The Standard Pattern)

This is the **gold standard** pattern for safe multi-step transactions:

```sql
DELIMITER //

CREATE PROCEDURE TransferFunds(IN sender_id INT, IN receiver_id INT, IN amount DECIMAL(10,2))
BEGIN
    -- If anything goes wrong in this procedure, roll back everything
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'ERROR: Transfer failed. No money moved.' AS Status;
    END;
    
    START TRANSACTION;
    
    UPDATE Accounts SET Balance = Balance - amount WHERE Account_ID = sender_id;
    UPDATE Accounts SET Balance = Balance + amount WHERE Account_ID = receiver_id;
    
    COMMIT;
    SELECT 'SUCCESS: Transfer complete.' AS Status;
    
END //

DELIMITER ;
```

---

## 6. CONTINUE vs EXIT: When to Use Each

| Handler Type | After the Handler Code Runs... | Best For |
|---|---|---|
| `CONTINUE` | Execution resumes at the next statement after the one that caused the error | Soft errors you want to log and skip (e.g., cursor loops) |
| `EXIT` | The current `BEGIN...END` block is exited immediately | Fatal errors that should stop all further processing |

---

## 7. Declaring Named Conditions (Advanced)

Instead of using raw `SQLSTATE` codes, you can name them for readability:

```sql
-- Give a friendly name to SQLSTATE 23000 (duplicate entry)
DECLARE Duplicate_Entry CONDITION FOR SQLSTATE '23000';

-- Now use the friendly name in your handler
DECLARE EXIT HANDLER FOR Duplicate_Entry
BEGIN
    SELECT 'Duplicate detected — please use a unique value.' AS Error;
END;
```

---

## 8. Common Mistakes

- **Declaring handlers after other statements:** `DECLARE` statements (variables, cursors, handlers) MUST come at the very **top** of a `BEGIN...END` block, before any regular SQL statements. If you put `DECLARE HANDLER` after an `INSERT`, MySQL will give a syntax error.
- **Using `CONTINUE` for transaction errors:** If an `INSERT` fails partially and you `CONTINUE`, the next statement might act on inconsistent data. For transaction-based procedures, **always use `EXIT` + `ROLLBACK`**.
- **Catching everything with `SQLEXCEPTION` in development:** During debugging, catching all exceptions can hide real errors. Be specific about which `SQLSTATE` codes you handle during development.

---

## 9. Useful SQLSTATE Codes Reference

| SQLSTATE | MySQL Error | Description |
|---|---|---|
| `'23000'` | 1062 | Duplicate entry for unique/primary key |
| `'22001'` | 1406 | Data too long for column |
| `'42S02'` | 1146 | Table doesn't exist |
| `'42S22'` | 1054 | Unknown column name |
| `'45000'` | – | Generic custom error (used with `SIGNAL`) |

---

## 10. Mini Practice Tasks

- **Task 1:** Write a stored procedure `SafeDelete(IN p_id INT)` that tries to delete a user from the `Users` table. If the deletion fails (e.g., because of a foreign key constraint), catch the `SQLEXCEPTION` and return a friendly error message instead of crashing.
- **Task 2:** What is the difference between `DECLARE EXIT HANDLER` and `DECLARE CONTINUE HANDLER`? Give one scenario where each would be the right choice.

---
