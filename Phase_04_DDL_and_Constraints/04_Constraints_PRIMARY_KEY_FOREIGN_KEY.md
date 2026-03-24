# 🗝️ Topic 4.4: SQL Constraints - PRIMARY KEY & FOREIGN KEY

We previously clamped down our basic columns using `NOT NULL` and `UNIQUE` to stop duplicate text. Now, we must use constraints to legally lock the tables themselves together.

The **`PRIMARY KEY`** and **`FOREIGN KEY`** constraints are the literal engine brackets that put the "Relational" inside a Relational Database.

---

## 1. Definition

Both of these constraints are rigidly bolted onto tables during the `CREATE TABLE` DDL statement:

1.  **`PRIMARY KEY` Constraint:** Natively forces a specific column to become the ultimate Anchor Point. Mathematically, it applies `NOT NULL` and `UNIQUE` simultaneously, but more importantly, it commands the database engine to build the physical clustered structure of the disk around this column.
2.  **`FOREIGN KEY` Constraint:** Commands the Database Engine to draw an invisible tripwire connecting this column securely to the Primary Key of an adjacent table.

---

## 2. Why This Concept Exists

Writing "Order_ID" on a whiteboard makes clear visual sense to a human designing an ER Model. But the MySQL Engine is just a blank Operating System application; it doesn't know that `Buyer_ID` in an Orders table is mathematically supposed to match `User_ID` in a Users table. 

We must explicitly declare these DDL constraints so MySQL can mathematically lock the tables together and prevent network chaos.

---

## 3. Why We Use It

*   **Primary Key (Entity Integrity):** It gives the specific table its main Anchor point and ensures lightning-fast queries (via the Clustered B-Tree).
*   **Foreign Key (Referential Integrity):** It natively blocks the insertion of "Orphaned Records". The engine will forcefully reject an `INSERT` command if you try to assign an Order to `Buyer_ID = 99` but User 99 does not actually exist.
*   **Referential Actions:** You can tell MySQL exactly what to do when a parent is deleted or updated:
    *   **RESTRICT (Default):** Blocks the action if children exist.
    *   **CASCADE:** Automatically deletes/updates children. (Great for logs).
    *   **SET NULL:** Keeps children but sets their connection to NULL. (Great for history).

---

## 4. When to Use Which?

*   **1 Table = 1 Primary Key Constraint:** Apply it exactly once per table. (Note: A Single PK constraint can wrap around *multiple* columns if it is a Composite Key, but it remains one unified constraint.)
*   **Foreign Key Constraints:** You can apply anywhere from zero to 50 Foreign Key constraints exclusively on the "Child" table that holds the Many-side of a relationship.

---

## 5. How It Works (The Invisible B-Tree Locks - PRO LEVEL)

What does a Foreign Key physically do under the hood during a live transaction?

If an Amazon Node.js backend executes an `INSERT INTO Orders (Buyer_ID) VALUES (5);`, MySQL pauses the insertion. It casts an incredibly fast microscopic **Shared Lock (S-Lock)** onto `User_ID 5` in the entirely separate `Users` table. 
*Why?* To absolutely guarantee that while the `Orders` table is taking 2 milliseconds to finish saving the receipt to the hard drive, a totally different server thread doesn't simultaneously delete the `User`! The Foreign Key uses these microscopic memory locks across tables to perfectly orchestrate global referential integrity.

---

## 6. Syntax / Implementation (Column-Level vs Table-Level)

This is a critical SQL syntax formatting rule:

1.  **Column-Level Constraint:** You write the constraint on the exact same line as the column definition type.
    *   *Example:* `User_ID INT PRIMARY KEY`
    *   *Rule:* You **cannot** use Column-Level syntax to create a Composite Primary Key!
2.  **Table-Level Constraint:** You define all your standard columns blindly first, and then at the very bottom of the script, you formally declare the relational constraints. 
    *   *Example:* `PRIMARY KEY (User_ID, Campaign_ID)`
    *   *Rule:* You **must** use this syntax for any constraints involving multiple columns, and almost always for Foreign Keys.

---

## 7. Real-Life Examples

**The School Application System:**
*   You build a `Classes` table. You place a `PRIMARY KEY` explicitly on the Room Number integer.
*   You build a `Students` table. You place a `PRIMARY KEY` on their Student_ID. You also create an `Assigned_Room` column, and bolt a **Table-Level** `FOREIGN KEY` onto it, wiring it straight back to the `Classes` table.

---

## 8. SQL Examples (MySQL Execution with NAMED Constraints)

As we learned in our previous topic, you should always **NAME** your constraints using the explicit `CONSTRAINT` keyword so your Node/Python server error logs don't look like gibberish!

```sql
-- Creating the PARENT first
CREATE TABLE Authors (
    Author_ID INT,
    
    -- Defining the Anchor (Table-Level format)
    CONSTRAINT pk_author PRIMARY KEY(Author_ID) 
);

-- Creating the CHILD second
CREATE TABLE Books (
    Book_ID INT PRIMARY KEY,
    Title VARCHAR(100),
    Written_By INT,          -- Same byte-size as Parent!
    
    -- Explicitly defining the behavior during Parent deletion
    CONSTRAINT fk_author FOREIGN KEY (Written_By) 
    REFERENCES Authors(Author_ID)
    ON DELETE CASCADE       -- Chain Reaction: Delete Books if Author is deleted
    ON UPDATE RESTRICT      -- Freeze: Block Author_ID changes if Books exist
);
```

---

## 9. Common Mistakes

*   **The Ordering Paradox (Crashing the Database on Day 1):** 
    *   If you type `CREATE TABLE Books` containing a foreign key referencing `Authors`, but you actually haven't scripted the `Authors` table yet... MySQL will crash instantly! *You must always `CREATE` the Parent Table first!*
    *   If you type `DROP TABLE Authors`, but there are still `Books` actively wired to it on the hard drive, MySQL will crash instantly! *You must always `DROP` the Child Table first before destroying the Parent!*
*   **Data Type Mismatch:** As warned natively in Phase 2, if the Parent's PK is a massive `BIGINT`, but the Child's FK is a tiny `INT`, the `FOREIGN KEY` creation script will violently fail. The algebraic binary RAM sizes must perfectly physically map.

---

## 10. Tips & Best Practices (Pro-Level)

**Why Naming Your Foreign Keys is Mandatory:**
If you naively type `FOREIGN KEY (User_ID) REFERENCES Users(ID)`, MySQL will automatically generate a terribly ugly name for the invisible wire, like `orders_ibfk_1`. 

Three years later, your CTO says "We are changing database structures, please remove the foreign key wire." If you try to run the DDL Drop Command (`ALTER TABLE Orders DROP FOREIGN KEY User_ID`), it will **FAIL**! You cannot drop the wire by guessing its original column name; you MUST use its exact system dictionary name! If you didn't explicitly name it yourself as `fk_user`, you will have to dig dangerously through encrypted system metadata tables just to find what MySQL secretly named it.

---

## 11. Mini Practice Tasks

*   **Task 1:** Look at a design for a `Cities` table and a `States` table. Every single City natively belongs to exactly one State. Which table mathematically acts as the Parent? Which table must be `CREATED` first in your SQL script to prevent a crash?
*   **Task 2:** Write the exact Table-Level SQL syntax to formally declare a named Foreign Key constraint `fk_state` applied to a column `State_ID` that points safely to the `States` table (specifically tracking back to its `ID` column). 

---
