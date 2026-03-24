# 🗣️ Topic 3.1: SQL Sublanguages Overview (DDL, DML, DQL, DCL, TCL)

We have officially passed the theoretical modeling phase. From here out, Phase 3 marks the beginning of writing actual, physical SQL code. 

Before we write our first command, we must categorize the SQL language. SQL has hundreds of commands, but every single one fundamentally falls into one of **5 Sublanguages**. Let's break them down.

---

## 1. Definition

The SQL Language is strictly categorized into 5 specific "Vocabularies" based on the sheer destructive power and purpose of the command:

1.  **DDL (Data Definition Language):** Commands that interact with the physical *Structure* or *Schema* (e.g., Creating a table, deleting a table).
2.  **DML (Data Manipulation Language):** Commands that modify the actual *Data/Rows* inside the structure (e.g., Adding a new user, updating a price).
3.  **DQL (Data Query Language):** Commands that purely *Read* data without changing anything (e.g., Fetching a report).
4.  **DCL (Data Control Language):** Commands that manage *Security* and user access (e.g., Giving a junior dev a password).
5.  **TCL (Transaction Control Language):** Commands that manage *State and Safety* (e.g., Saving your work permanently or undoing a mistake).

---

## 2. Why This Concept Exists

Because deleting a single row of text in an Excel cell is mathematically entirely different from taking an axe and deleting the physical Excel Grid Column itself. 

By categorizing the SQL language, Database Administrators (DBAs) can grant laser-focused security privileges. A normal employee should have permission to use **DML** (Insert data), but they should NEVER have permission to use **DDL** (Drop Table).

---

## 3. Why We Use It

*   **Security Granularity:** We use DCL to lock down DDL, while keeping DML open to the public web application.
*   **Engine Optimization:** MySQL parses these sublanguages entirely differently. DDL commands actually physically alter the magnetic disk file headers, whereas DML commands primarily manipulate the B-Tree memory nodes.

---

## 4. When to Use It

*   **DDL:** Day 1 of the project (Building the tables).
*   **DML:** Day 2 through Day 10,000 (Users signing up and buying things).
*   **DQL:** End of the month (CEO wants a read-only sales report).
*   **DCL:** Only when hiring or firing employees.
*   **TCL:** Every single time money moves between accounts.

---

## 5. How It Works (The Auto-Commit Trap - PRO LEVEL)

This is the most critical architectural difference between DDL and DML in MySQL:
*   **DML is Reversible:** When you run an `UPDATE` or `DELETE` string, the InnoDB storage engine buffers the change. If you realize you made a mistake, you can mathematically undo it by triggering a TCL `ROLLBACK`.
*   **DDL is Permanent (Auto-Commit):** In MySQL, DDL statements (`CREATE`, `DROP`, `ALTER`) force an immediate, irreversible OS-level commit. If you accidentally execute `DROP TABLE Users`, you **cannot** undo it. Your only hope is pulling a backup from Amazon AWS.

---

## 6. Syntax / Implementation (The Core Commands)

Memorize this cheat sheet for interviews!

*   **DDL:** `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`
*   **DML:** `INSERT`, `UPDATE`, `DELETE`
*   **DQL:** `SELECT` *(Note: Many developers just loop DQL under DML for simplicity, but strictly speaking, it's DQL!)*
*   **DCL:** `GRANT`, `REVOKE`
*   **TCL:** `COMMIT`, `ROLLBACK`, `SAVEPOINT`

---

## 7. Real-Life Examples

**Building a House:**
1.  **DDL:** Pouring the concrete foundation and building walls perfectly (`CREATE TABLE`). Tearing down the den (`DROP`). 
2.  **DML:** Bringing your TV and couch inside (`INSERT`). Changing the couch from blue to red (`UPDATE`). Throwing the couch in the garbage (`DELETE`).
3.  **DQL:** Looking through the window from outside to see if the TV is on (`SELECT`).
4.  **DCL:** Giving your best friend a copy of the front door key (`GRANT`).
5.  **TCL:** Saving your video game progress before fighting a boss (`SAVEPOINT`), or reloading your old save file if you die (`ROLLBACK`).

---

## 8. SQL Examples (MySQL Execution)

Here is a quick visualization of the 5 sublanguages in action:

```sql
-- DDL: Defining the structure of the bucket
CREATE TABLE Bank_Vault (ID INT, Balance DECIMAL);

-- DML: Putting something physically inside the bucket
INSERT INTO Bank_Vault (ID, Balance) VALUES (1, 500.00);

-- DQL: Looking inside the bucket
SELECT Balance FROM Bank_Vault WHERE ID = 1;

-- TCL: Permanently saving the insertion to the magnetic disk
COMMIT;

-- DCL: Stopping the Intern from stealing the money
REVOKE ALL PRIVILEGES ON Bank_Vault FROM 'intern_user';
```

---

## 9. Common Mistakes

*   **Confusing DROP, DELETE, and TRUNCATE:** This is the #1 junior developer mistake in SQL.
    *   **`DELETE` (DML):** Erases a specific row (e.g., throwing out the couch). It writes to the undo-log so you can roll it back. Extremely slow for millions of rows.
    *   **`TRUNCATE` (DDL):** Erases *ALL* rows instantly by resetting the file pointer, but leaves the empty table standing. (Selling all your furniture instantly, but keeping the house). Very fast, but **cannot be easily rolled back**!
    *   **`DROP` (DDL):** Erases the rows AND physically incinerates the table and columns off the hard drive. (Bulldozing the entire property). Cannot be rolled back.

---

## 10. Tips & Best Practices

*   **Privilege Principle of Least Privilege (PoLP):** A senior architect never uses the `root` account to run an API web application backend. The backend should only be authenticated using `DML` and `DQL` privileges. If a hacker finds an SQL Injection exploit on your website, they can only `SELECT` or `UPDATE` text; they physically lack the `DDL` authorization to run `DROP TABLE` and destroy your company.
*   **Transactions (TCL):** We will spend an entire Phase on this, but know that wrapping your complex `DML` statements securely in `TCL` (`BEGIN` and `COMMIT`) is what prevents half-finished bank transfers when servers crash.

---

## 11. Mini Practice Tasks

*   **Task 1:** Look at the command `ALTER TABLE Users ADD COLUMN Age INT;`. Does this change the actual data, or the structure? Which of the 5 Sublanguages does this belong to?
*   **Task 2:** Write down the critical difference between `DELETE` and `DROP` using the "Building a House" analogy.

---
