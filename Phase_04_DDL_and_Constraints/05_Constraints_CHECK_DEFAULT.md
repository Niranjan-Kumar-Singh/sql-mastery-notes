# 🛡️ Topic 4.5: SQL Constraints - CHECK & DEFAULT

We have secured our tables using structural constraints (`PRIMARY KEY`, `NOT NULL`). Now we must implement pure **Business Logic** at the lowest level of the database using the final two constraints: **`CHECK`** and **`DEFAULT`**.

---

## 1. Definition

1.  **`CHECK` Constraint:** A custom mathematical True/False formula evaluated against incoming data. If the data evaluates to "False" against your formula, the database physically rejects the insertion.
2.  **`DEFAULT` Constraint:** A safety-net fallback value. If a backend Python or Java application completely forgets to send a value for a column, the Database injects the `DEFAULT` value instead of throwing an error or inserting `NULL`.

---

## 2. Why This Concept Exists

If you declare a column `Price INT`, MySQL will happily let you insert `-500` into it because mathematically, `-500` is indeed a valid integer. 

We use these constraints because Data Types only check the *shape* of the data, but `CHECK` constraints verify the actual *sanity* of the data based on CEO business rules.

---

## 3. Why We Use It

*   **`CHECK`:** Protects the company from impossible states (Negative bank account balances, or users whose Age is listed as 999).
*   **`DEFAULT`:** Saves network bandwidth! When an API creates a million users on a Friday, it doesn't need to manually send `is_active = TRUE` a million times across the internet. It can just send names and passwords, and the Database Engine will seamlessly inject the `TRUE` defaults natively to save time.

---

## 4. When to Use Which?

*   Use `CHECK` for numeric boundaries (Prices, Ratings 1-to-5, Ages) or specific String patterns.
*   Use `DEFAULT` for common initial states (Account Creation Datestamps, Initial Balances, Status Flags).

---

## 5. How It Works (The Famous MySQL 5.7 Flaw - PRO LEVEL)

If you are ever interviewing for a Senior Database Engineer role, you must know this historical trap:

Prior to MySQL Version `8.0.16` (released in 2019), MySQL famously **ignored** the `CHECK` constraint! If you wrote `CREATE TABLE... CHECK (Age >= 18)`, the MySQL engine would read your code, successfully compile it, and then completely ignore the math. It would happily let you insert a 5-year-old. 

*Why?* The original storage engine architecture wasn't designed for complex inline math. If you are forced to work on a legacy MySQL 5.7 Server today, you **cannot** trust `CHECK` constraints. You must build complex backend SQL programs called "Triggers" to manually simulate the math. (Don't worry, modern MySQL 8.0 fully enforces `CHECK` beautifully).

---

## 6. Syntax / Implementation (Cheat Sheet)

You declare these exactly on the same row as the Column Definition.

```sql
CREATE TABLE Products (
    Product_Name VARCHAR(100),
    
    -- Using DEFAULT
    Inventory_Count INT DEFAULT 0,
    
    -- Using CHECK
    Price DECIMAL(10,2) CHECK (Price >= 0.99)
);
```

---

## 7. Real-Life Examples

**The YouTube Video Upload Logic:**
*   You upload a video. The `Views` column doesn't need to be sent by the frontend API. It natively triggers a `DEFAULT 0`.
*   The `Like_Ratio` column natively triggers a `DEFAULT 'Unrated'`.
*   The `Video_Length` column fires a `CHECK (Length_Seconds > 0)` ensuring corrupted 0-second video files are immediately rejected by the database.

---

## 8. SQL Examples (MySQL Execution)

Let's formally define a robust table using explicit constraints:

```sql
CREATE TABLE Bank_Accounts (
    Account_ID INT PRIMARY KEY AUTO_INCREMENT,
    Owner_Name VARCHAR(100) NOT NULL,
    
    -- The fallback. Everyone starts with zero dollars.
    Balance DECIMAL(15,2) DEFAULT 0.00,
    
    -- Advanced CHECK constraint. You cannot be negative!
    CONSTRAINT chk_positive_balance CHECK (Balance >= 0.00),
    
    -- Advanced CHECK constraint. A user rating must literally be 1, 2, 3, 4, or 5.
    Customer_Rating INT,
    CONSTRAINT chk_valid_rating CHECK (Customer_Rating BETWEEN 1 AND 5)
);

-- ❌ FATAL ERROR: This fails because the Rating is 9 (violating the CHECK!)
INSERT INTO Bank_Accounts (Owner_Name, Customer_Rating) VALUES ('Alice', 9);

-- ✅ SUCCESS: We completely forgot to send the Balance, so it inserts 0.00!
INSERT INTO Bank_Accounts (Owner_Name, Customer_Rating) VALUES ('Bob', 5);
```

---

## 9. Common Mistakes

*   **Confusing DEFAULT with NOT NULL:** A column can be `NOT NULL` but have no Default. A column can have a Default but legally allow `NULL`. If a Python script literally sends the command `INSERT ... VALUES (NULL)`, the Database will insert the `NULL`! `DEFAULT` only triggers if the column is physically **missing** from the SQL `INSERT` command entirely.
*   **Not Naming the Check Rules:** As shown in Section 8 (`CONSTRAINT chk_positive_balance`), you must explicitly name your `CHECK` rules. If you don't, and the math fails, the server error log will just spit out: *"Constraint 14xqz violated"*. That is useless for an engineer trying to figure out which column crashed.

---

## 10. Tips & Best Practices (Pro-Level)

**Stacking CHECKs with AND:**
You can build deeply complex mathematical validators within a single check statement using basic SQL operators (`AND`, `OR`, `IN`).

```sql
CREATE TABLE Flights (
    -- The flight must take off BEFORE it lands natively!
    Takeoff_Time DATETIME,
    Landing_Time DATETIME,
    
    -- Table-Level CHECK Constraint comparing two columns against each other!
    CONSTRAINT chk_time_logic CHECK (Takeoff_Time < Landing_Time)
);
```

---

## 11. Mini Practice Tasks

*   **Task 1:** Write the column definition for a `Subscribe_To_Newsletter` column. It should be a Boolean (remember Phase 3 fakes this with `TINYINT`), and if the frontend API completely forgets to send a value, it should physically fall back to `FALSE`.
*   **Task 2:** Why must you be deeply cautious deploying a `CHECK` constraint on an ancient server running MySQL version 5.6? (Hint: See Section 5).

---
