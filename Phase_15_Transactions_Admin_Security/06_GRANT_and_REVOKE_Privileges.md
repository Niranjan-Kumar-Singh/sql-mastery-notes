# 🔑 Topic 15.6: GRANT and REVOKE Privileges (DCL)

Creating a user gives them the ability to connect to MySQL. But by default, a new user has **zero permissions** — they can't even see a `SHOW DATABASES` list. `GRANT` and `REVOKE` let you give users exactly the access they need — no more, no less.

---

## 1. Definition

- **`GRANT`:** Gives a specific privilege (or set of privileges) to a user on a specific database, table, or column.
- **`REVOKE`:** Removes a previously granted privilege from a user.
- **DCL (Data Control Language):** The category of SQL commands that controls **who can do what** in the database.

---

## 2. The Privilege Hierarchy — Four Levels of Granularity

Privileges can be granted at 4 different scopes, from broadest to most specific:

| Level | Syntax | What it Controls |
|---|---|---|
| **Global** | `ON *.*` | All databases and all tables on the server. Use only for DBAs. |
| **Database** | `ON mydb.*` | All tables within one specific database. |
| **Table** | `ON mydb.users` | One specific table in one specific database. |
| **Column** | `ON mydb.users (Name, Email)` | Specific columns within a table (most granular). |

---

## 3. Complete Privilege Reference

| Privilege | What it Allows |
|---|---|
| `SELECT` | Reading / querying data |
| `INSERT` | Adding new rows |
| `UPDATE` | Modifying existing rows |
| `DELETE` | Removing rows |
| `CREATE` | Creating new tables/databases |
| `DROP` | Deleting tables/databases |
| `INDEX` | Creating and dropping indexes |
| `ALTER` | Modifying table structure |
| `EXECUTE` | Running stored procedures and functions |
| `GRANT OPTION` | Passing privileges to other users |
| `ALL PRIVILEGES` | Every privilege (root-level access) |

---

## 4. Syntax / Implementation — Full Cheat Sheet

```sql
-- Grant specific privileges on a specific table
GRANT SELECT, INSERT, UPDATE ON company_db.Customers TO 'api_user'@'localhost';

-- Grant read-only access to an entire database
GRANT SELECT ON company_db.* TO 'reporter'@'%';
-- '%' means the user can connect from ANY host

-- Grant all privileges on one database (app admin level)
GRANT ALL PRIVILEGES ON company_db.* TO 'app_admin'@'localhost';

-- Grant column-level access (hide sensitive data)
GRANT SELECT (Employee_ID, First_Name, Department) ON company_db.Employees TO 'hr_viewer'@'localhost';
-- hr_viewer can see names but NOT salary or personal details!

-- Allow a user to grant their permissions to others (rare, use carefully!)
GRANT SELECT ON company_db.* TO 'team_lead'@'localhost' WITH GRANT OPTION;

-- Revoke a specific permission
REVOKE DELETE ON company_db.* FROM 'api_user'@'localhost';

-- Revoke all permissions from a user
REVOKE ALL PRIVILEGES ON *.* FROM 'old_employee'@'localhost';

-- Check what privileges a user currently has
SHOW GRANTS FOR 'api_user'@'localhost';

-- Apply changes immediately (usually auto-applied, but good practice)
FLUSH PRIVILEGES;
```

---

## 5. How It Works (Grant Tables — PRO LEVEL)

MySQL stores all privilege information in special system tables inside the `mysql` database:
- `mysql.user` — global-level grants
- `mysql.db` — database-level grants
- `mysql.tables_priv` — table-level grants
- `mysql.columns_priv` — column-level grants

When a user tries to execute a query, MySQL checks these tables in order (global → database → table → column). The first matching grant at any level determines access.

This is why `FLUSH PRIVILEGES` is sometimes needed after direct table edits — it reloads these tables into memory. When using `GRANT`/`REVOKE`, MySQL auto-flushes, so it's rarely needed.

---

## 6. Real-Life Examples

**Example 1 — The Three-Role Pattern (Most Production Systems):**
```sql
-- Role 1: Read-only reporter (data analytics, dashboards)
GRANT SELECT ON analytics_db.* TO 'dashboard_user'@'%';

-- Role 2: Application API user (CRUD but no structure changes)
GRANT SELECT, INSERT, UPDATE, DELETE ON app_db.* TO 'api_user'@'app_server_ip';

-- Role 3: DBA (everything, but NOT on production during business hours!)
GRANT ALL PRIVILEGES ON *.* TO 'dba_user'@'localhost' WITH GRANT OPTION;
```

**Example 2 — Hiding Salary Data from HR Staff:**
```sql
-- HR can see employee name, department, start date — but NOT salary
GRANT SELECT (Emp_ID, First_Name, Last_Name, Department, Start_Date) 
ON company_db.Employees 
TO 'hr_staff'@'localhost';

-- HR can still UPDATE only the department field
GRANT UPDATE (Department) ON company_db.Employees TO 'hr_staff'@'localhost';
```

---

## 7. The Principle of Least Privilege (PoLP)

**Always give users the minimum permissions needed to do their job.**

| Role | Appropriate Privileges |
|---|---|
| Frontend App | SELECT, INSERT, UPDATE (never DROP/CREATE/DELETE) |
| Reporting Tool | SELECT only |
| Backend API | SELECT, INSERT, UPDATE, DELETE (on specific tables only) |
| DBA / Admin | ALL PRIVILEGES (limited to internal network only) |
| Read-only Analyst | SELECT only |

**Never use the `root` account in application code.** If your application's connection credentials are leaked, the attacker should have the minimum possible access.

---

## 8. Common Mistakes

- **Granting `ALL PRIVILEGES ON *.*` to application users:** If the app is compromised, the attacker becomes a database admin. Grant only what the app needs.
- **Using `root@%` (root accessible from any host):** This means root is accessible from the internet. Always restrict sensitive users to `localhost` or a specific IP.
- **Forgetting to REVOKE when employees leave:** Create an offboarding checklist that includes `REVOKE ALL PRIVILEGES` and `DROP USER` for departing team members.

---

## 9. Mini Practice Tasks

- **Task 1:** Create a user `reporting_bot` that can connect from anywhere (`%`) and can only `SELECT` from all tables in the `sales_db` database.
- **Task 2:** An employee named Priya has left the company. she was user `priya@localhost`. Write the commands to: (a) revoke all her privileges, and (b) drop her user account.
- **Task 3:** Why is column-level `GRANT` useful in a healthcare database? Give a specific scenario with patient data.

---
