# 🔗 Topic 2.4: Foreign Key — The Link Between Tables

Imagine a library. Each book has a unique ID. When you borrow a book, your borrowing record says "*Book ID: 42*" — it doesn't repeat the book's full name, author, and price. It just references the book by its ID. That reference is exactly what a **Foreign Key** is in a database.

---

## 1. Definition

A **Foreign Key (FK)** is a column (or group of columns) in one table that **references the Primary Key of another table**. It creates a mathematical link between the two tables, enforcing that no "orphan" data can exist.

- **Parent Table:** The table being referenced (the one with the Primary Key).
- **Child Table:** The table doing the referencing (the one with the Foreign Key).

---

## 2. Why This Concept Exists

Without foreign keys, your data becomes a mess of unconnected islands. Imagine:

- You delete a `Customer` from the `Customers` table.
- But their `Orders` still exist in the `Orders` table, now pointing to a customer that doesn't exist.

This is called **Orphan Data**, and it silently corrupts your entire application. The Foreign Key constraint prevents this from ever happening.

---

## 3. The Rules of a Foreign Key

1. The value in the FK column **must exist** in the referenced Primary Key column (or be `NULL`).
2. You **cannot delete** a parent row if a child row still references it (unless you configure a special action).
3. The FK column and the PK column it references must have the **same data type**.

---

## 4. Referential Actions (`ON DELETE` / `ON UPDATE`)

When you delete or update a parent row, MySQL needs to know what to do with the child rows. You configure this with special keywords:

| Action | Description | Use When |
|---|---|---|
| `RESTRICT` | 🚫 Block the delete/update if child rows exist. | You want strict, safe protection (Default). |
| `CASCADE` | ⛓️ Automatically delete/update child rows too. | Orders should vanish when a customer is deleted. |
| `SET NULL` | Set the FK column in child rows to `NULL`. | You want to keep the child data but remove the link. |
| `NO ACTION` | Same as `RESTRICT` in MySQL (checked at end of transaction). | Advanced transactional scenarios. |

---

## 5. Syntax / Implementation

```sql
CREATE TABLE Customers (
    Customer_ID INT PRIMARY KEY,
    Name        VARCHAR(100) NOT NULL
);

CREATE TABLE Orders (
    Order_ID    INT PRIMARY KEY AUTO_INCREMENT,
    Customer_ID INT,                           -- The Foreign Key Column
    Total       DECIMAL(10, 2),
    
    CONSTRAINT fk_customer                    -- Give the FK a name (best practice!)
        FOREIGN KEY (Customer_ID)             -- Column in THIS (child) table
        REFERENCES Customers(Customer_ID)     -- Points to PK in parent table
        ON DELETE CASCADE                     -- If customer is deleted, orders are too
        ON UPDATE RESTRICT                    -- Prevent changing a Customer_ID in use
);
```

---

## 6. Real-Life Example

**The School Database:**

| Students Table (Parent) | Enrollments Table (Child) |
|---|---|
| `Student_ID` (PK) = 101 | `Student_ID` (FK) = 101 → References Student 101 |
| `Student_ID` (PK) = 102 | `Student_ID` (FK) = 103 → ❌ ERROR! Student 103 doesn't exist! |

The Foreign Key constraint would **reject** the second insertion immediately, preventing corrupt data from ever entering the system.

---

## 7. Viewing and Dropping Foreign Keys

```sql
-- See all foreign keys on a table
SHOW CREATE TABLE Orders;

-- Remove a foreign key (you need its name!)
ALTER TABLE Orders DROP FOREIGN KEY fk_customer;

-- Add a foreign key to an existing table
ALTER TABLE Orders 
ADD CONSTRAINT fk_customer 
FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID);
```

---

## 8. Common Mistakes

- **Forgetting `ON DELETE` actions:** If you don't specify any action, MySQL defaults to `RESTRICT`. This can surprise beginners when a delete query fails with an error like `ERROR 1451: Cannot delete or update a parent row: a foreign key constraint fails`.
- **Wrong data type:** If the PK is `INT UNSIGNED` but your FK is just `INT`, MySQL will silently (or noisily) refuse to create the constraint. They must match **exactly**.
- **No index on FK column:** MySQL auto-creates an index on the FK column for performance. But if you ever manually `DROP` that index, JOIN queries on that relationship will become very slow.

---

## 9. Tips & Best Practices

- **Always name your foreign keys** using `CONSTRAINT fk_tablename_column`. This makes it much easier to `DROP` or debug them later.
- **Draw your relationships first.** In complex systems with 10+ tables, knowing which table is a Parent vs. a Child before writing any SQL is crucial to avoid circular dependency errors.

---

## 10. Mini Practice Tasks

- **Task 1:** Create two tables: `Authors` (Author_ID, Name) and `Books` (Book_ID, Title, Author_ID). Add a foreign key from `Books.Author_ID` to `Authors.Author_ID` with `ON DELETE CASCADE`. What happens if you delete the author?
- **Task 2:** What error would MySQL give you if you tried to `INSERT INTO Books VALUES (1, 'My Book', 999)` when there is no author with ID 999?

---
