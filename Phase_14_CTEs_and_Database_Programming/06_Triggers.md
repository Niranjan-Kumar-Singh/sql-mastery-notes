# 🔫 Topic 14.6: Triggers (BEFORE/AFTER Logic)

What if you want the database to **Automatically** do something whenever a specific event happens? For example: *"When a user is deleted, automatically copy their record to an 'Audit_Archive' table."* This is the role of a **Trigger**.

---

## 1. Definition

**Trigger:** A named database object that is associated with a table and activates (fires) when a particular event (`INSERT`, `UPDATE`, or `DELETE`) occurs for that table.

---

## 2. Why This Concept Exists

*   **Audit Logging:** Recording who changed what value and when.
*   **Data Consistency:** Ensuring that if the `Stock` in the `Products` table changes, the `Orders` table remains valid.
*   **Automated Calculation:** Updating a "Total Balance" column every time a "Transaction" is added.

---

## 3. The 6 Timing Combinations

A trigger is defined by its **Timing** and its **Event**:

| Timing | Event | Typical Use Case |
|---|---|---|
| **BEFORE** | `INSERT` | Validating data before it reaches the table. |
| **AFTER** | `INSERT` | Updating a summary table or audit log. |
| **BEFORE** | `UPDATE` | Capturing the "Old Color" before it changes. |
| **AFTER** | `UPDATE` | Syncing data to another system. |
| **BEFORE** | `DELETE` | Checking if the deletion is allowed. |
| **AFTER** | `DELETE` | Moving the record to an archive. |

---

## 4. Syntax: OLD and NEW Keywords

Inside a trigger, you have access to two special virtual rows:
- **`OLD`:** The row data **before** the change (available in `UPDATE` and `DELETE`).
- **`NEW`:** The row data **after** the change (available in `INSERT` and `UPDATE`).

```sql
DELIMITER //

CREATE TRIGGER before_salary_update
BEFORE UPDATE ON Employees
FOR EACH ROW
BEGIN
    -- If the new salary is lower than old, throw error
    IF NEW.Salary < OLD.Salary THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Salary cannot decrease!';
    END IF;
END //

DELIMITER ;
```

---

## 5. How It Works (The Atomic Hook - PRO LEVEL)

A trigger executes within the **same transaction** as the statement that fired it.
1.  User runs `UPDATE Employees`.
2.  MySQL locks the row.
3.  The `BEFORE UPDATE` trigger fires.
4.  The actual `UPDATE` happens.
5.  The `AFTER UPDATE` trigger fires.
6.  The transaction commits.

**Senior Insight:** Because it's in the same transaction, if a trigger fails, the main statement **also fails and rolls back**.

---

## 6. Real-Life Examples

**The Audit Log Pattern:**
```sql
CREATE TRIGGER audit_product_change
AFTER UPDATE ON Products
FOR EACH ROW
BEGIN
    INSERT INTO Product_Log(Prod_ID, Old_Price, New_Price, Change_Time)
    VALUES (OLD.ID, OLD.Price, NEW.Price, NOW());
END //
```

---

## 7. Common Mistakes

*   **Recursive Triggers:** Creating a trigger on Table A that updates Table A... which fires the trigger again... and again. This will cause an error or a crash.
*   **Performance:** Triggers run for **every single row** being updated. If you update 1 million rows, the trigger runs 1 million times. Too many triggers can slow down your database significantly.

---

## 8. Tips & Best Practices (Pro-Level)

**Implicit Logic Danger:**
Triggers are "Silent." A developer might run an `UPDATE` and be confused why a value in another table also changed. **Pro-Tip:** Document your triggers clearly. Many teams prefer putting logic in **Stored Procedures** because they are "Explicit" (you have to call them) rather than "Silent."

---

## 9. Mini Practice Tasks

*   **Task 1:** What is the difference between the `OLD` and `NEW` keywords in a trigger?
*   **Task 2:** Write a trigger `after_user_insert` that inserts a welcome message into a `Notifications` table whenever a new user is added to the `Users` table.

---
