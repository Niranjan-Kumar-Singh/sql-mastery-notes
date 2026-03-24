# 👤 Topic 15.5: User Management in MySQL

A MySQL Server is like an apartment building. The `root` account is the building owner who has master keys to every room. But you wouldn't give the delivery person a master key — you give them access to only the lobby. **User Management** is how you issue and revoke these keys.

---

## 1. Definition

**User Management** in MySQL involves creating database user accounts, setting their passwords, and controlling which databases and tables they can access and what actions they can perform.

This is part of **DCL (Data Control Language)** — the security layer of SQL.

---

## 2. Why This Concept Exists

The `root` user in MySQL has unlimited power — it can drop any database, create any user, read any table. Running your web application as `root` is an enormous security risk:

- If a hacker finds a **SQL Injection flaw** in your website, they get root access — they can `DROP DATABASE` your entire business.
- The **Principle of Least Privilege (PoLP)** states: every user account should only have the *minimum permissions needed* to do its job — nothing more.

---

## 3. Creating and Deleting Users

```sql
-- Create a new user (only allowed to log in from localhost)
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';

-- Create a user that can connect from ANY host (less secure)
CREATE USER 'report_user'@'%' IDENTIFIED BY 'ReportPass456!';

-- Remove a user completely
DROP USER 'app_user'@'localhost';

-- Change a user's password
ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'NewPassword789!';

-- See all existing users
SELECT User, Host FROM mysql.user;
```

**Understanding `'user'@'host'`:**
- `'app'@'localhost'` — Can only connect from the same machine as MySQL.
- `'app'@'192.168.1.5'` — Can only connect from that specific IP address.
- `'app'@'%'` — Can connect from any IP (use with caution!).

---

## 4. Granting Privileges

```sql
-- Grant SELECT only (read-only analyst account)
GRANT SELECT ON company_db.* TO 'report_user'@'localhost';

-- Grant INSERT, UPDATE, DELETE on one specific table
GRANT INSERT, UPDATE, DELETE ON company_db.orders TO 'app_user'@'localhost';

-- Grant ALL privileges on one database (for a backend app account)
GRANT ALL PRIVILEGES ON company_db.* TO 'app_user'@'localhost';

-- Grant ALL on everything (DON'T do this for app accounts!)
GRANT ALL PRIVILEGES ON *.* TO 'admin_user'@'localhost';

-- IMPORTANT: Apply the changes immediately
FLUSH PRIVILEGES;
```

**Common Privilege Types:**

| Privilege | Allows |
|---|---|
| `SELECT` | Reading data |
| `INSERT` | Adding rows |
| `UPDATE` | Modifying rows |
| `DELETE` | Removing rows |
| `CREATE` | Creating tables/databases |
| `DROP` | Deleting tables/databases |
| `INDEX` | Creating/dropping indexes |
| `EXECUTE` | Running stored procedures |
| `ALL PRIVILEGES` | Everything above |

---

## 5. Revoking Privileges

```sql
-- Remove SELECT permission from the report user
REVOKE SELECT ON company_db.* FROM 'report_user'@'localhost';

-- Remove ALL privileges (but don't delete the user)
REVOKE ALL PRIVILEGES ON company_db.* FROM 'app_user'@'localhost';

-- Check what privileges a user currently has
SHOW GRANTS FOR 'app_user'@'localhost';
```

---

## 6. Real-Life Architecture Example

For a standard web application, you should create at minimum **3 separate user accounts**:

| User Account | Privileges | Purpose |
|---|---|---|
| `root` | ALL | Only for DB admin tasks, never used by the app |
| `app_user` | SELECT, INSERT, UPDATE, DELETE | Used by the backend application server |
| `report_user` | SELECT only | Used by analytics tools / BI dashboards |

```sql
-- Setting up the app_user for a web backend
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'SecureAppPass!';
GRANT SELECT, INSERT, UPDATE, DELETE ON shop_db.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
```

---

## 7. Common Mistakes

- **Running the web app as `root`:** This is the #1 security mistake. A single SQL Injection vulnerability becomes a full database takeover.
- **Using `%` as the host for sensitive accounts:** `'admin'@'%'` allows connections from the entire internet. Always restrict by IP or use `localhost`.
- **Forgetting `FLUSH PRIVILEGES`:** After manual changes to the `mysql.user` table (using `INSERT/UPDATE` directly), you must flush. Changes made via `GRANT/REVOKE` or `CREATE USER` do not require flushing.

---

## 8. Tips & Best Practices

- **One app, one user:** Never share database credentials between different applications or microservices. If one app is compromised, the others remain safe.
- **Rotate passwords regularly:** Use `ALTER USER` to change database passwords periodically, especially after any team member leaves the company.
- **Audit user access:** Periodically run `SELECT User, Host FROM mysql.user;` and `SHOW GRANTS FOR 'username'@'host';` to audit who has access to what.

---

## 9. Mini Practice Tasks

- **Task 1:** A new junior developer joins your team and only needs to **read** data from a database called `inventory_db`. Write the complete SQL to create their user account and grant appropriate privileges.
- **Task 2:** You run `SHOW GRANTS FOR 'app_user'@'localhost';` and see they have `DROP` privilege. They shouldn't. Write the SQL to fix this.

---
