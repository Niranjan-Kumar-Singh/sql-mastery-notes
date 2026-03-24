# 🔄 Topic 12.2: Updatable vs. Non-Updatable Views

A View is usually used for **reading** data. But in certain cases, you can actually `INSERT`, `UPDATE`, or `DELETE` data **through** a View, and those changes are automatically applied to the underlying base table. This is called an Updatable View.

---

## 1. Definitions

- **Updatable View:** A view where there is a clear, 1-to-1 mapping between rows in the view and rows in the underlying base table. MySQL can translate a DML statement on the view into a DML statement on the base table.
- **Non-Updatable View:** A view that combines, aggregates, or filters data in ways that MySQL cannot "reverse" back to a single base table row. You can only `SELECT` from these.

---

## 2. Rules — When is a View Updatable?

A view is updatable **only when** it avoids all of these elements:

| Disqualifier | Why It Prevents Updates |
|---|---|
| `DISTINCT` | Rows are merged — no 1-to-1 mapping |
| `GROUP BY` / `HAVING` | Multiple base rows → 1 view row |
| Aggregate functions (`SUM`, `AVG`, `COUNT`) | Derived values — no base row to update |
| `UNION` or `UNION ALL` | Rows from multiple tables |
| Subquery in the SELECT list | Derived values |
| Joins on write side | MySQL can't determine which table to update |

**Simple rule:** If a view is a straightforward column selection + optional WHERE filter from a SINGLE table, it is updatable.

---

## 3. How It Works — Query Rewriting (PRO LEVEL)

When you run `UPDATE my_view SET col = val WHERE id = 5`, MySQL:
1. Translates the view's definition to find the underlying base table.
2. Rewrites the `UPDATE my_view` as `UPDATE base_table`.
3. Executes the rewritten statement on the actual table.

This happens transparently — the application code that does `UPDATE my_view` doesn't need to know it's actually updating `base_table`.

---

## 4. Syntax / Implementation

```sql
-- Create a simple updatable view (one table, no aggregation)
CREATE VIEW Active_Employees AS
SELECT Emp_ID, First_Name, Last_Name, Salary, Department, Status
FROM Employees
WHERE Status = 'Active';

-- ✅ You CAN update through this view (straightforward single-table view):
UPDATE Active_Employees SET Salary = 75000 WHERE Emp_ID = 101;
-- MySQL actually runs: UPDATE Employees SET Salary = 75000 WHERE Emp_ID = 101

-- ✅ You CAN insert through this view:
INSERT INTO Active_Employees (First_Name, Last_Name, Salary, Department, Status)
VALUES ('Priya', 'Sharma', 60000, 'IT', 'Active');

-- ✅ You CAN delete through this view:
DELETE FROM Active_Employees WHERE Emp_ID = 202;

-- Create a non-updatable view (uses GROUP BY → cannot update)
CREATE VIEW Dept_Salary_Summary AS
SELECT Department, COUNT(*) AS Employee_Count, AVG(Salary) AS Avg_Salary
FROM Employees GROUP BY Department;

-- ❌ This will FAIL — non-updatable view:
UPDATE Dept_Salary_Summary SET Avg_Salary = 80000 WHERE Department = 'IT';
-- ERROR 1288: The target table Dept_Salary_Summary is not updatable
```

---

## 5. `WITH CHECK OPTION` — Controlling What Updates Are Allowed

This is the most powerful feature for updatable views. It ensures that any `INSERT` or `UPDATE` through the view must satisfy the view's own `WHERE` clause.

```sql
-- View shows only 'Active' employees
CREATE VIEW Active_Employees AS
SELECT * FROM Employees WHERE Status = 'Active'
WITH CHECK OPTION;

-- ❌ This UPDATE fails — it would make the row disappear from the view!
UPDATE Active_Employees SET Status = 'Inactive' WHERE Emp_ID = 5;
-- ERROR 1369: CHECK OPTION failed — this would make the row invisible in the view

-- ✅ This UPDATE succeeds — row still satisfies the view's WHERE clause
UPDATE Active_Employees SET Salary = 80000 WHERE Emp_ID = 5;
```

**Why is this useful?** Without `WITH CHECK OPTION`, a user writing to the view could accidentally "delete" data from their own view by changing the very column the view filters on. `WITH CHECK OPTION` is a safety net.

---

## 6. CASCADED vs. LOCAL Check Option

For views built on top of other views:

```sql
-- View B built on top of View A with LOCAL check option:
-- Only View B's own WHERE clause is checked (View A's is not).
CREATE VIEW View_B AS SELECT * FROM View_A WHERE Salary > 50000
WITH LOCAL CHECK OPTION;

-- View C built on top of View A with CASCADED check option (default):
-- Both View C's and View A's WHERE clauses must be satisfied.
CREATE VIEW View_C AS SELECT * FROM View_A WHERE Salary > 50000
WITH CASCADED CHECK OPTION;
```

---

## 7. Real-Life Examples

**Example 1 — Masked Employee View (read/write restricted):**
```sql
CREATE VIEW IT_Department_View AS
SELECT Emp_ID, First_Name, Last_Name, Email, Salary
FROM Employees
WHERE Department = 'IT' AND Status = 'Active'
WITH CHECK OPTION;

-- IT manager can update salaries of IT team through the view
UPDATE IT_Department_View SET Salary = 90000 WHERE Emp_ID = 303;

-- But can't accidentally move someone to a different department through this view:
UPDATE IT_Department_View SET Department = 'HR' WHERE Emp_ID = 303;
-- ERROR: This would remove them from the IT view — WITH CHECK OPTION blocks it!
```

**Example 2 — Application-Facing View for Security:**
```sql
-- The application only ever interacts with this safe view, not the base table
CREATE VIEW Customer_Profile_View AS
SELECT Customer_ID, Name, Email, Phone, City, Registration_Date
FROM Customers
WHERE Account_Status = 'Active'
WITH CHECK OPTION;
-- Application can UPDATE customer's phone/city through the view
-- Cannot deactivate accounts through the view (would violate WITH CHECK OPTION)
```

---

## 8. Common Mistakes

- **Trying to update a non-updatable view:** Always check — if the view has `GROUP BY`, `DISTINCT`, or aggregate functions, it cannot be updated. Update the base table directly.
- **Not using `WITH CHECK OPTION`:** Without it, you can update a row to no longer match the view's filter — effectively making the row "invisible" from the view without realizing it.
- **Over-using updatable views in large applications:** Most senior architects recommend using views **only for reading**. For writes, use Stored Procedures (Topic 14.4) which can enforce complex business rules with conditional logic.

---

## 9. Tips & Best Practices

- **Add `WITH CHECK OPTION` to any updatable view that has a `WHERE` clause** — it prevents logic errors where an update makes a row "fall out" of the view silently.
- **Test updatability explicitly:** Run `SELECT IS_UPDATABLE FROM information_schema.VIEWS WHERE TABLE_NAME = 'your_view_name'` to check if MySQL considers your view updatable.
- **Prefer Stored Procedures for writes in critical systems.** A Stored Procedure can enforce business rules (e.g., "can't reduce salary by more than 20%") that a view cannot.

---

## 10. Mini Practice Tasks

- **Task 1:** Create an updatable view `High_Stock_Products` that shows only products where `Stock > 100`. Add `WITH CHECK OPTION`. Then: (a) write a valid UPDATE through the view, and (b) write an UPDATE that would fail due to `WITH CHECK OPTION`.
- **Task 2:** Why is a View with `GROUP BY` non-updatable? If you change a `Department`'s average salary, which base table rows would MySQL need to change?
- **Task 3:** What is the difference between `WITH LOCAL CHECK OPTION` and `WITH CASCADED CHECK OPTION`?

---
