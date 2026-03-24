# ✅ Topic 3.5: MySQL Data Types - Boolean / TINYINT(1)

The final core Data Type we need to master before writing our `CREATE TABLE` scripts is the concept of True and False. 

How do we securely tell a computer that a Youtube video is "Public" or "Private"? Do we type out the word "Public" into a `VARCHAR` string? Absolutely not. We use the most microscopic, efficient data type in computer science: the **Boolean**.

---

## 1. Definition

A **Boolean** is a strict logical data type that represents exactly one of two possible states:
*   **True** (On / Yes / 1)
*   **False** (Off / No / 0)

---

## 2. Why This Concept Exists

Because software engineering is inherently binary (transistors are either on or off). We need a mechanism to mathematically answer binary statements in a Database without wasting disk space. 

If we have a `Users` table, and we want to track if a user is an Administrator, storing the 5-letter text string `'Admin'` across 10 million rows wastes 50,000,000 bytes of hard drive space. Storing a simple Boolean `1` or `0` takes a fraction of that memory.

---

## 3. Why We Use It

*   **Extreme Size Optimization:** True/False calculations are phenomenally fast for CPUs.
*   **UI Toggles:** Booleans are natively mapped directly to Front-End Application Checkboxes and Toggle Switches (e.g., A toggle switch in your iPhone settings menu maps to exactly one Boolean in a database).

---

## 4. When to Use It

*   **Flags:** Any column name that conceptually starts with `is_`, `has_`, or `can_`.
    *   `is_active`
    *   `is_deleted`
    *   `has_paid`
    *   `can_comment`

---

## 5. How It Works (The Massive MySQL Illusion - PRO LEVEL)

This is a legendary trivia question for MySQL Database Administrators:
**Question:** *"How many bytes does the MySQL `BOOLEAN` data type physically cost?"*
**Answer:** *"MySQL does not actually have a native BOOLEAN data type!"*

If you literally type `BOOLEAN` or `BOOL` into your MySQL table creation script, the Engine secretly intercepts your code, deletes the word `BOOLEAN`, and replaces it with **`TINYINT(1)`**. 

MySQL treats Booleans strictly as tiny math integers. It saves the number `1` for True, and the number `0` for False. Consequently, "Booleans" in MySQL cost exactly 1-Byte of memory.

---

## 6. Syntax / Implementation (Cheat Sheet)

Because MySQL secretly uses `TINYINT(1)`, these two table creation scripts are mathematically 100% identical under the hood:

```sql
-- Script 1:
CREATE TABLE Users (is_banned BOOLEAN);

-- Script 2:
CREATE TABLE Users (is_banned TINYINT(1));
```
*Note: The `(1)` in `TINYINT(1)` does NOT mean 1 byte. It is purely a visual display hint telling the MySQL terminal to only print 1 digit on the screen.*

---

## 7. Real-Life Examples

1.  **Netflix Subscription:** You click "Cancel Subscription". The database executes an update changing your `is_premium` column from exactly `1` to `0`. Your videos instantly lock.
2.  **Amazon Product:** If a TV sells out completely, the database changes the `is_in_stock` Boolean from `TRUE` to `FALSE`. The website UI sees this and replaces the "Add to Cart" button with "Out of Stock".

---

## 8. SQL Examples (MySQL Execution)

MySQL provides `TRUE` and `FALSE` keywords to make coding easier even though it compiles to `1` and `0`.

```sql
CREATE TABLE Products (
    Product_ID INT PRIMARY KEY AUTO_INCREMENT,
    Product_Name VARCHAR(100),
    
    -- We set the Default to zero! By default, a new product is NOT physically shipped yet.
    is_shipped BOOLEAN DEFAULT FALSE 
);

-- I insert a product. I don't provide the Boolean. It automatically defaults to 0 (False).
INSERT INTO Products (Product_Name) VALUES ('Laptop');

-- An employee scans a barcode at the post office. We update the flag to 1 (True)!
UPDATE Products SET is_shipped = TRUE WHERE Product_ID = 1;
```

---

## 9. Common Mistakes

*   **Assuming MySQL enforces strict True/False:** Because MySQL secretly uses `TINYINT` under the hood, the Database Engine will bizarrely let you insert the number `8` or `12` into a "Boolean" column! In MySQL, `0` is mathematically evaluated as False. **Every single other number on the planet is mathematically evaluated as True.** Avoid inserting numbers into Boolean columns; strictly use the words `TRUE` and `FALSE`.
*   **Creating a "Status" Boolean:** If an order can be "Pending", "Shipped", or "Delivered", you cannot use a Boolean! Booleans only handle TWO states. (For 3 or more states, you must use an `ENUM` or a standard `TINYINT` measuring from 1 to 3).

---

## 10. Tips & Best Practices (Pro-Level)

**The "Soft Delete" Architecture:**
Senior backend engineers almost *never* run the destructive `DELETE` command on a user or transaction. 

If a customer clicks "Delete My Account", you do not actually delete them from the hard drive (because you need their historical ledger data for taxes!). Instead, you create a boolean column `is_deleted`. When they click delete, you run an `UPDATE is_deleted = TRUE`.

Then, on your backend web app, you write every query to say: `SELECT * FROM Users WHERE is_deleted = FALSE`. You have successfully hidden the user from everyone, perfectly simulating a deletion, while safely keeping all their data hidden in the database for legal auditing!

---

## 11. Mini Practice Tasks

*   **Task 1:** You create a column `is_verified BOOLEAN`. You accidentally insert the number `-500` into it. If your Python backend checks `if is_verified:`, does the backend think the user is verified or unverified? (Hint: See Section 9 regarding the number 0).
*   **Task 2:** Describe the "Soft Delete" logic to an imaginary junior developer. Why do we flip an `is_deleted` Boolean rather than just dropping the row entirely?

---
