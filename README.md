<div align="center">

# 🗄️ The Complete SQL Mastery Notes

### A comprehensive, beginner-friendly, professional-quality guide to SQL & MySQL  
*From absolute zero to advanced analytics — one phase at a time.*

---

![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql&logoColor=white)
![Markdown](https://img.shields.io/badge/Format-Markdown-lightgrey?style=for-the-badge&logo=markdown)
![Phases](https://img.shields.io/badge/Phases-16-success?style=for-the-badge)
![Topics](https://img.shields.io/badge/Topics-80%2B-orange?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-purple?style=for-the-badge)

</div>

---

## 📖 About This Repository

This is a **personal, handcrafted SQL curriculum** — built from scratch with the goal of mastering SQL from the very basics all the way to advanced production-level concepts. These notes are:

- ✅ **Beginner-friendly** — plain English, real-world analogies, no unnecessary jargon
- ✅ **Technically deep** — every topic includes PRO-LEVEL internals (B+ Trees, MVCC, WAL, InnoDB mechanics)
- ✅ **Comprehensive** — 80+ topics across 16 structured Phases covering every major SQL concept
- ✅ **Practical** — every note has syntax cheat sheets, real-life examples, common mistakes, and 3 practice tasks
- ✅ **Production-ready** — covers security (PoLP, GRANT/REVOKE), performance (EXPLAIN, indexing), and advanced patterns

> **Perfect for:** Students starting their SQL journey, developers brushing up on gaps, or anyone who wants solid, Google-able reference notes.

---

## 🗺️ Curriculum Overview

Start with `00_SQL_Master_Roadmap.md` for the complete learning path, then follow the 16 phases in order.

| Phase | Topic | Files | Key Concepts |
|:-----:|-------|:-----:|--------------|
| 📘 **01** | [Database Fundamentals](./Phase_01_Database_Fundamentals/) | 8 | DBMS, RDBMS, File Systems vs DB, DB Architecture, MySQL Installation |
| 📘 **02** | [Data Models & Keys](./Phase_02_Data_Models_and_Keys/) | 9 | ER Model, PK, FK, Composite, Surrogate, Alternate, Super Keys |
| 📘 **03** | [SQL Sublanguages & Data Types](./Phase_03_SQL_Sublanguages_and_Data_Types/) | 5 | DDL/DML/DQL/DCL/TCL, Numeric, String, Date, Boolean types |
| 📘 **04** | [DDL & Constraints](./Phase_04_DDL_and_Constraints/) | 9 | CREATE, ALTER, DROP, PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT |
| 📘 **05** | [DML — Data Manipulation](./Phase_05_DML_Data_Manipulation_Language/) | 5 | INSERT, UPDATE, DELETE, REPLACE, INSERT IGNORE |
| 📘 **06** | [DQL & Basic Filtering](./Phase_06_DQL_and_Basic_Filtering/) | 7 | SELECT, WHERE, DISTINCT, Comparison & Logical Operators |
| 📗 **07** | [Advanced Filtering & Sorting](./Phase_07_Advanced_Filtering_and_Sorting/) | 8 | BETWEEN, IN, IS NULL, LIKE, Wildcards, REGEXP, ORDER BY, LIMIT/OFFSET |
| 📗 **08** | [MySQL Built-in Functions](./Phase_08_MySQL_Built_in_Functions/) | 7 | String, Date, Numeric, Aggregate, CASE, COALESCE |
| 📗 **09** | [Aggregations & Grouping](./Phase_09_Aggregations_and_Grouping/) | 6 | COUNT, SUM, AVG, MIN, MAX, GROUP BY, HAVING, ROLLUP |
| 📙 **10** | [Multi-Table Queries — Joins](./Phase_10_Multi_Table_Queries_Joins/) | 9 | INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF, NATURAL JOIN, 3+ Tables |
| 📙 **11** | [Set Operations & Subqueries](./Phase_11_Set_Operations_and_Subqueries/) | 10 | UNION, INTERSECT, EXCEPT, Subqueries (WHERE/SELECT/FROM), EXISTS, ANY/ALL |
| 📙 **12** | [Views & Indexes](./Phase_12_Views_and_Indexes/) | 7 | Views, Updatable Views, Clustered/Secondary/Composite Indexes, EXPLAIN |
| 📕 **13** | [Window Functions](./Phase_13_Window_Functions/) | 8 | OVER, PARTITION BY, ROW_NUMBER, RANK, DENSE_RANK, LEAD, LAG, NTILE |
| 📕 **14** | [CTEs & Database Programming](./Phase_14_CTEs_and_Database_Programming/) | 8 | CTEs, Recursive CTEs, Stored Procedures, Functions, Triggers, Cursors, Error Handling |
| 📕 **15** | [Transactions, Admin & Security](./Phase_15_Transactions_Admin_Security/) | 7 | ACID, TCL, Locking, Isolation Levels, GRANT/REVOKE, User Management, Backup |
| 📕 **16** | [Database Design & Normalization](./Phase_16_Database_Design_Normalization/) | 6 | Anomalies, 1NF → 2NF → 3NF → BCNF, Denormalization |

---

## 📂 Repository Structure

```
📦 SQL-Mastery-Notes
 ├── 📄 00_SQL_Master_Roadmap.md          ← Start here! Full learning path
 │
 ├── 📁 Phase_01_Database_Fundamentals/
 │   ├── 01_Data_Information_Database_DBMS.md
 │   ├── 02_File_Systems_vs_DBMS.md
 │   ├── 03_Types_of_Databases.md
 │   └── ... (8 files)
 │
 ├── 📁 Phase_02_Data_Models_and_Keys/
 ├── 📁 Phase_03_SQL_Sublanguages_and_Data_Types/
 ├── 📁 Phase_04_DDL_and_Constraints/
 ├── 📁 Phase_05_DML_Data_Manipulation_Language/
 ├── 📁 Phase_06_DQL_and_Basic_Filtering/
 ├── 📁 Phase_07_Advanced_Filtering_and_Sorting/
 ├── 📁 Phase_08_MySQL_Built_in_Functions/
 ├── 📁 Phase_09_Aggregations_and_Grouping/
 ├── 📁 Phase_10_Multi_Table_Queries_Joins/
 ├── 📁 Phase_11_Set_Operations_and_Subqueries/
 ├── 📁 Phase_12_Views_and_Indexes/
 ├── 📁 Phase_13_Window_Functions/
 ├── 📁 Phase_14_CTEs_and_Database_Programming/
 ├── 📁 Phase_15_Transactions_Admin_Security/
 └── 📁 Phase_16_Database_Design_Normalization/
```

---

## 📚 What Each Note Contains

Every single note file in this repository follows a consistent, professional structure:

```
📄 Topic Name
 ├── 📌 Definition           — What is it, in plain English
 ├── 💡 Why It Exists        — The real-world problem it solves
 ├── ⚙️  How It Works         — Internal mechanics (PRO LEVEL)
 ├── 💻 Syntax Cheat Sheet   — Full SQL examples ready to use
 ├── 🌍 Real-Life Examples   — 2-3 business scenarios (e-commerce, banking, HR)
 ├── ❌ Common Mistakes       — What beginners (and seniors!) get wrong
 ├── ✅ Best Practices        — Pro tips from industry experience
 └── 🏋️  Practice Tasks       — 3 tasks to test understanding
```

---

## 🚀 Getting Started

### Prerequisites
- MySQL 8.0+ installed ([Download MySQL](https://dev.mysql.com/downloads/mysql/))
- MySQL Workbench (optional, recommended for beginners)
- Any Markdown viewer (GitHub renders these perfectly!)

### Recommended Learning Path

```
Phase 01 → 02 → 03 → 04 → 05     ← Foundation (Weeks 1-2)
Phase 06 → 07 → 08 → 09           ← Querying (Weeks 3-4)
Phase 10 → 11 → 12                 ← Intermediate (Weeks 5-6)
Phase 13 → 14 → 15 → 16           ← Advanced / Production (Weeks 7-8)
```

> 💡 **Tip:** Don't just read — open MySQL Workbench alongside and actually run every code example!

---

## 🎯 Topics Covered At a Glance

<details>
<summary><strong>📘 Foundation Phases (01–05)</strong></summary>

- What is Data, Information, Database, and DBMS?
- Why Relational Databases beat File Systems
- SQL vs MySQL — the language vs the tool
- All MySQL Data Types — INT, DECIMAL, VARCHAR, TEXT, DATE, TIMESTAMP, BOOLEAN
- DDL — CREATE TABLE, ALTER TABLE, DROP TABLE, TRUNCATE
- DML — INSERT (single/bulk), UPDATE, DELETE, REPLACE, INSERT IGNORE
- All Constraints — PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK, DEFAULT
</details>

<details>
<summary><strong>📗 Querying Phases (06–09)</strong></summary>

- SELECT statement and the full query execution pipeline
- WHERE, DISTINCT, comparison operators, logical operators (AND/OR/NOT)
- LIKE, Wildcards, REGEXP/RLIKE patterns
- BETWEEN, IN, IS NULL / IS NOT NULL
- ORDER BY (single/multi-column), LIMIT, OFFSET, keyset pagination
- String functions: CONCAT, LENGTH, SUBSTRING, UPPER, LOWER, TRIM, REPLACE
- Date functions: NOW(), CURDATE(), DATEDIFF(), DATE_FORMAT(), TIMESTAMPDIFF()
- Aggregate functions: COUNT, SUM, AVG, MIN, MAX
- GROUP BY, HAVING, WITH ROLLUP (multidimensional aggregates)
</details>

<details>
<summary><strong>📙 Intermediate Phases (10–12)</strong></summary>

- All 7 JOIN types with visual diagrams and real examples
- UNION, UNION ALL, INTERSECT, EXCEPT (Set theory in SQL)
- Subqueries in WHERE, SELECT, FROM clauses
- Correlated Subqueries — how they work and their O(N²) performance danger
- EXISTS and NOT EXISTS — and the critical NOT IN NULL trap
- Views — creating, updating, securing data with views
- Indexes — B+ Tree, Clustered, Secondary, Composite, Full-Text
- EXPLAIN — reading query execution plans
</details>

<details>
<summary><strong>📕 Advanced Phases (13–16)</strong></summary>

- Window Functions — OVER(), PARTITION BY, all ranking/value/aggregate functions
- LEAD/LAG for time-series and sequential analysis
- CTEs — simple, chained, and recursive
- Stored Procedures with IN/OUT/INOUT parameters and DELIMITER
- User-Defined Functions (UDFs), Triggers, Cursors
- Error Handling with DECLARE HANDLER and transaction rollback patterns
- ACID Properties and the Redo/Undo Log system
- Transaction Isolation Levels — Dirty Reads, Non-Repeatable Reads, Phantoms
- Row-Level Locking, Shared vs Exclusive Locks, Deadlocks
- GRANT / REVOKE and the Principle of Least Privilege
- User Management and Host-Restricted Accounts
- mysqldump Backup & Restore — full cheat sheet
- 1NF → 2NF → 3NF → BCNF Normalization
- Denormalization and real-world performance tradeoffs
</details>

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **MySQL 8.0** | The SQL engine used for all examples |
| **MySQL Workbench** | GUI for running queries and managing databases |
| **InnoDB** | MySQL's default storage engine (ACID-compliant) |
| **Markdown** | All notes written in clean, readable Markdown |

---

## 📈 Progress Tracker

Use this to track your learning progress:

- [ ] Phase 01 — Database Fundamentals
- [ ] Phase 02 — Data Models & Keys
- [ ] Phase 03 — SQL Sublanguages & Data Types
- [ ] Phase 04 — DDL & Constraints
- [ ] Phase 05 — DML
- [ ] Phase 06 — DQL & Basic Filtering
- [ ] Phase 07 — Advanced Filtering & Sorting
- [ ] Phase 08 — MySQL Built-in Functions
- [ ] Phase 09 — Aggregations & Grouping
- [ ] Phase 10 — Multi-Table Queries (Joins)
- [ ] Phase 11 — Set Operations & Subqueries
- [ ] Phase 12 — Views & Indexes
- [ ] Phase 13 — Window Functions
- [ ] Phase 14 — CTEs & Database Programming
- [ ] Phase 15 — Transactions, Admin & Security
- [ ] Phase 16 — Database Design & Normalization

---

## 🤝 Contributing

Found a mistake? Have a suggestion to improve a note? Contributions are welcome!

1. Fork this repository
2. Create a branch: `git checkout -b fix/topic-name`
3. Make your changes
4. Submit a Pull Request with a clear description of what you changed and why

---

## 📄 License

This project is licensed under the **MIT License** — feel free to use, share, and build upon these notes for personal or educational purposes. A credit link back to this repo is appreciated! 🙏

---

## 👤 Author

<div align="center">

### Niranjan Kumar Singh

*Passionate about learning databases, backend development, and building things that matter.*

---

[![Email](https://img.shields.io/badge/Email-niranjansingh1419%40gmail.com-red?style=for-the-badge&logo=gmail&logoColor=white)](mailto:niranjansingh1419@gmail.com)
[![Instagram](https://img.shields.io/badge/Instagram-niranjan.ks.in-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/niranjan.ks.in)
[![GitHub](https://img.shields.io/badge/GitHub-Follow%20Me-181717?style=for-the-badge&logo=github)](https://github.com)

---

*"The best way to learn SQL is to write SQL. Open Workbench. Run the code. Break things. Fix them."*

</div>

---

<div align="center">

⭐ **If these notes helped you, please give this repository a star!** ⭐  
*It helps others discover this resource and motivates continued improvements.*

</div>
