# 🔑 Topic 2.3: Primary Key and Candidate Key

In a database table with millions of rows, how does MySQL find the *exact one record* you're looking for? The answer is **Keys** — columns or sets of columns that uniquely identify rows.

---

## 1. Candidate Key — Every Possible Unique Identifier

A **Candidate Key** is any column (or combination of columns) that can **uniquely identify every row** in a table with no duplicates and no NULLs.

A table can have **multiple** Candidate Keys.

**Example — `Users` table:**
- `User_ID` → Always unique ✅ → Candidate Key
- `Email` → Each user has a different email ✅ → Candidate Key
- `Phone_Number` → Always unique ✅ → Candidate Key
- `Username` → Always unique ✅ → Candidate Key
- `First_Name` → NOT unique (multiple Johns) ❌ → NOT a Candidate Key

---

## 2. Primary Key — The Chosen One

A **Primary Key (PK)** is the single Candidate Key you officially designate as the table's main identifier. All other qualifying Candidate Keys that are NOT chosen become **Alternate Keys**.

```
All Unique Columns → Candidate Keys
                          ↓
          Pick ONE → Primary Key
   The remaining ones → Alternate Keys
```

**The 3 Rules of a Primary Key:**
1. **Unique** — No two rows can have the same value
2. **NOT NULL** — It can never be empty
3. **Immutable** — Its value should never change (once assigned, stays forever)

---

## 3. Why These Rules Matter (PRO LEVEL)

- **Why immutable?** The Primary Key is used as a Foreign Key in other tables. If you change a Customer's `Customer_ID`, every related `Order`, `Invoice`, and `Shipment` record that references it must also change — a cascading nightmare. Use surrogate IDs (auto-incremented numbers) that never change.
- **Why not NULL?** MySQL builds the **Clustered Index** around the Primary Key. NULL values cannot be indexed or located reliably — they'd make the entire table unlookupable.

---

## 4. Syntax / Implementation

```sql
-- Single column Primary Key
CREATE TABLE Customers (
    Customer_ID INT AUTO_INCREMENT PRIMARY KEY,  -- Chosen PK
    Email VARCHAR(100) UNIQUE,                    -- Candidate Key (not chosen → Alternate Key)
    Phone VARCHAR(15) UNIQUE,                     -- Candidate Key (not chosen → Alternate Key)
    Name VARCHAR(100) NOT NULL
);

-- Composite Primary Key (two columns together form the PK)
CREATE TABLE Order_Items (
    Order_ID INT,
    Product_ID INT,
    Quantity INT,
    PRIMARY KEY (Order_ID, Product_ID)  -- Neither column alone is unique; together they are
);

-- Check existing primary key
SHOW KEYS FROM Customers WHERE Key_name = 'PRIMARY';
```

---

## 5. Real-Life Example — Which Column to Choose as PK?

```
Table: Employees
Columns with unique values: Emp_ID, Email, National_ID, Passport_Number

❌ Email as PK: People change emails — violates immutability rule
❌ National_ID as PK: Not available for foreign nationals — violates NOT NULL rule  
❌ Passport_Number as PK: Passports expire and change — violates immutability
✅ Emp_ID (AUTO_INCREMENT INT) as PK: Never null, never changes, always unique
```

**Best Practice:** In almost every real table, use a **surrogate key** — a meaningless `INT AUTO_INCREMENT` column (`ID`) — as the Primary Key. It satisfies all 3 rules by design.

---

## 6. Common Mistakes

- **Using Email as a Primary Key:** People change emails. Changing a PK cascades through all related tables — a dangerous and expensive operation.
- **Thinking a table can have multiple Primary Keys:** A table has exactly ONE Primary Key (though it can be composite — spanning multiple columns).
- **Confusing Primary Key with Unique Key:** A Unique Key also prevents duplicates but allows NULL and doesn't become the clustered index anchor.

---

## 7. Mini Practice Tasks

- **Task 1:** A `Students` table has these unique columns: `Student_ID`, `Email`, `Aadhar_Number`, `Roll_Number`. List all Candidate Keys. Which one should be the Primary Key and why?
- **Task 2:** Can a table have zero Primary Keys? What problem does that cause for joining and updating data?
- **Task 3:** Write the SQL to create a `Bookings` table where the combination of `Room_ID` and `Booking_Date` forms the Primary Key (composite key — a room can only have one booking per date).

---
