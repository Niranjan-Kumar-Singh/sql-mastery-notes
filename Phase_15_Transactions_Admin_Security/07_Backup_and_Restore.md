# 💾 Topic 15.7: Backup and Restore (mysqldump & More)

"Disk drives fail. Ransomware happens. Developers accidentally run `DROP DATABASE`. The only thing that saves you is a backup. The best time to set up backups is before the disaster. The second-best time is right now."

---

## 1. Why Backups Are Non-Negotiable

- **Hardware failure:** Even enterprise SSDs have failure rates. RAID arrays protect against drive failure but not against logical corruption.
- **Human error:** `DELETE FROM Orders WHERE` — deleted too many rows. `DROP TABLE Users` — gone forever. Without a backup, this is catastrophic.
- **Ransomware/security breach:** Attackers encrypt your database and demand payment. A clean backup from before the attack restores everything.
- **Migration:** Moving from local development → staging → production requires reliable, version-consistent copies of your data.

---

## 2. Backup Types

| Type | Tool | Output | Restore Time | Best For |
|---|---|---|---|---|
| **Logical Backup** | `mysqldump` | `.sql` file (CREATE + INSERT statements) | Slower (re-runs SQL) | Most use cases, cross-version migration |
| **Physical Backup** | Percona XtraBackup, file copy | Raw InnoDB `.ibd` files | Very fast | Large DBs, same MySQL version |
| **Binary Log Backup** | MySQL Binary Log | Log of all changes since last backup | Fast (replay) | Point-in-time recovery |

---

## 3. mysqldump — The Most Common Backup Tool

`mysqldump` is a **command-line tool** (run in Terminal/PowerShell, NOT inside MySQL Workbench).

### Exporting (Creating a Backup)

```bash
# Backup a single database
mysqldump -u root -p my_database > backup_2024_03_25.sql

# Backup with compression (saves disk space for large databases)
mysqldump -u root -p my_database | gzip > backup_2024_03_25.sql.gz

# Backup schema ONLY (no data — useful for dev environment setup)
mysqldump -u root -p --no-data my_database > schema_only.sql

# Backup data ONLY (no CREATE TABLE — useful for re-importing into same structure)
mysqldump -u root -p --no-create-info my_database > data_only.sql

# Backup specific tables only
mysqldump -u root -p my_database Users Orders Products > selected_tables.sql

# Backup ALL databases on the server (full server backup)
mysqldump -u root -p --all-databases > full_server_backup.sql

# InnoDB-safe backup (no table lock, consistent snapshot — ALWAYS use this!)
mysqldump -u root -p --single-transaction my_database > safe_backup.sql
```

### Critical Flags Explained

| Flag | What it Does | When to Use |
|---|---|---|
| `--single-transaction` | Uses a START TRANSACTION snapshot — no table locks | Always, for InnoDB tables |
| `--no-data` | Schema only | Setting up dev/staging databases |
| `--no-create-info` | Data only | Re-importing to existing structure |
| `--all-databases` | Every database on server | Full server disaster recovery backup |
| `--compress` | Compresses data during transfer | Remote backup over slow network |

---

## 4. Importing (Restoring a Backup)

```bash
# Restore into an existing database
mysql -u root -p my_database < backup_2024_03_25.sql

# Restore a compressed backup
gunzip < backup_2024_03_25.sql.gz | mysql -u root -p my_database

# Restore ALL databases (all-databases backup)
mysql -u root -p < full_server_backup.sql

# If the database doesn't exist yet, create it first:
mysql -u root -p -e "CREATE DATABASE my_database;"
mysql -u root -p my_database < backup_2024_03_25.sql
```

---

## 5. Binary Log Backups — Point-in-Time Recovery (PRO LEVEL)

For production systems, `mysqldump` alone isn't enough. What if disaster strikes at 3:47 PM but your last backup was at 2 AM? You'd lose 13 hours of data.

**Binary Logs** record every change to the database (every INSERT, UPDATE, DELETE). Combined with a baseline backup, you can restore to any exact point in time.

```sql
-- Check if binary logging is enabled
SHOW VARIABLES LIKE 'log_bin';  -- Should show 'ON'

-- See current binary log file
SHOW MASTER STATUS;

-- Replay binary logs from a specific date/time
-- (Run in terminal after restoring the base backup)
-- mysqlbinlog --start-datetime="2024-03-25 02:00:00" --stop-datetime="2024-03-25 14:47:00" /var/log/mysql/binlog.000045 | mysql -u root -p
```

---

## 6. Real-Life Examples

**Example 1 — Daily Automated Backup (Windows Task Scheduler / Linux Cron):**
```bash
# Linux cron job (runs every night at 2 AM):
# 0 2 * * * mysqldump -u root -p'YourPassword' --single-transaction company_db > /backups/company_db_$(date +\%Y\%m\%d).sql

# Windows Task Scheduler equivalent PowerShell:
# mysqldump -u root -pYourPassword --single-transaction company_db > "D:\Backups\company_db_$(Get-Date -Format 'yyyyMMdd').sql"
```

**Example 2 — Migration: Local → Production Server:**
```bash
# Step 1: Backup local database
mysqldump -u root -p --single-transaction local_app_db > local_backup.sql

# Step 2: Copy to production server
# scp local_backup.sql user@production-server:/backups/

# Step 3: Restore on production
# mysql -u root -p production_app_db < /backups/local_backup.sql
```

---

## 7. Common Mistakes

- **Not using `--single-transaction` on InnoDB tables:** Without it, `mysqldump` locks the entire database during backup, blocking all writes. Production databases cannot afford this.
- **Never testing backups:** A backup you've never restored is not a backup — it's a false sense of security. Test your restore process monthly on a separate machine.
- **Storing backups on the same disk as the database:** If the disk fails, both the database AND the backup are gone. Always store backups off-site (AWS S3, Google Cloud Storage, etc.).
- **Not compressing old backups:** Daily backups accumulate quickly. Automate compression and deletion of backups older than 30/60/90 days.

---

## 8. Tips & Best Practices

- **The 3-2-1 Backup Rule:** 3 copies of data, on 2 different media types, with 1 stored off-site.
- **Always use `--single-transaction`** for InnoDB databases — it's non-locking and consistent.
- **Automate with cron/Task Scheduler** — manual backups get forgotten. Automation is the only reliable backup.
- **Enable binary logging** on production servers for point-in-time recovery capability (essential for all serious production databases).
- **Test your restore process** at least quarterly to ensure backups are actually valid and restorable.

---

## 9. Mini Practice Tasks

- **Task 1:** Write the terminal command to create a backup of a database called `Hospital_Records` to a file named `hospital_backup.sql`, using the safest flag for InnoDB tables.
- **Task 2:** Explain why `--single-transaction` is important when backing up InnoDB tables. What happens without it?
- **Task 3:** You have a nightly `mysqldump` backup but a disaster occurs mid-afternoon. What additional backup mechanism would allow you to recover up to the exact moment the disaster occurred?

---
