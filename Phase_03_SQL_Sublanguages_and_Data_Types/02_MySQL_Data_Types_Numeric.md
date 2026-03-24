# 🔢 Topic 3.2: MySQL Data Types - Numeric

Before we physically construct a table, we must tell MySQL exactly how much RAM and hard drive memory to reserve for every single column. This is called picking a **Data Type**. 

The most common, heavily optimized data format in computer science is the Number. Let's master the **Numeric Data Types**.

---

## 1. Definition

Numeric Data Types are strictly formatted blocks of memory reserved exclusively for counting, sorting, and performing mathematical calculations (like `SUM()` or `AVG()`). 

In MySQL, numbers are heavily segmented into 3 main categories:
1.  **Exact Integers** (Whole numbers with zero decimals: `1`, `50`)
2.  **Approximate Floating-Points** (Science/Calculus decimals: `3.14159`)
3.  **Exact Fixed-Points** (Financial currency decimals: `$50.25`)

---

## 2. Why This Concept Exists

Why can't we just dump everything into a universal `NUMBER` type? Because memory is expensive.

If you store the number `1` as a massive 8-Byte decimal, it takes up 8 times more physical space on the magnetic disk than if you properly stored it as a tiny 1-Byte integer. When your database hits 500 million rows, choosing the wrong numeric type can instantly bloat your server disk by 500 unnecessary Gigabytes, slowing your entire system to a crawl.

---

## 3. Why We Use It

*   **Computational Speed:** CPUs process `INTEGER` math blisteringly fast natively.
*   **Disk Efficiency:** By rigorously picking the exact right "size" of the number, we pack the B-Tree disk pages much tighter. 
*   **Data Integrity:** If you force a column to be an `INT`, MySQL will natively crash if a user accidentally tries to insert the string `'Apple'`, shielding your data from garbage entries.

---

## 4. When to Use It

*   **Integers:** Used for Primary Key IDs, counts, ages, inventory levels, years.
*   **Decimals:** Used *strictly* for money, accounting ledgers, and billing.
*   **Floats:** Used for geographic GPS coordinates, scientific measurements, temperature arrays.

---

## 5. How It Works (The Memory Parsing)

Here is exactly how much RAM MySQL physically allocates when you choose an Integer type. Memorize this logic:

| Data Type | Disk Space | Signed Range (Default) | UNSIGNED Range (Positive Only) | Best Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **TINYINT** | 1 Byte | -128 to 127 | 0 to 255 | Age, Boolean Flags (0/1) |
| **SMALLINT** | 2 Bytes | -32,768 to 32,767 | 0 to 65,535 | Max passengers on a ship |
| **MEDIUMINT**| 3 Bytes | -8.3 Million to 8.3 M | 0 to 16.7 Million | City Populations |
| **INT** | 4 Bytes | -2 Billion to +2 Billion| 0 to 4.2 Billion | **Primary Keys! (Default)** |
| **BIGINT** | 8 Bytes | -9 Quintillion to +9 Q | 0 to 18 Quintillion | Global system IDs (Twitter) |

---

## 6. Syntax / Implementation (Handling Decimals)

Integers are easy. Decimals get slightly tricky. 

**`DECIMAL(M, D)`** (The Money format)
*   **M (Precision):** The *TOTAL* number of digits allowed entirely.
*   **D (Scale):** The number of digits allowed *AFTER* the decimal point.
*   *Example:* `DECIMAL(5, 2)` means the maximum number it can possibly hold is `999.99` (5 total digits, 2 after the dot).

**`FLOAT` and `DOUBLE`** (The Science format)
*   They simply take up 4-Bytes (`FLOAT`) or 8-Bytes (`DOUBLE`) and dynamically slide the decimal point mathematically wherever it needs to go for extreme precision.

---

## 7. Real-Life Examples

1.  **Product Price:** `DECIMAL(8,2)` (Max price is `$999,999.99`).
2.  **Number of Wheels on a Car:** `TINYINT` (Max is 255 wheels. 1 Byte. Perfect!).
3.  **Views on a YouTube Video:** `BIGINT` ("Baby Shark" has 14 Billion views. A regular `INT` maxes out at 4.2 Billion. MySQL would crash violently without `BIGINT`!)

---

## 8. SQL Examples (MySQL Engine Execution)

```sql
CREATE TABLE Products (
    Product_ID INT PRIMARY KEY AUTO_INCREMENT,
    
    -- We know an age will never be negative. Let's save memory!
    Required_Age TINYINT UNSIGNED,
    
    -- We use exact Fixed-Point for currency to avoid rounding errors
    Price DECIMAL(10,2),
    
    -- We use Float for GPS coordinates because extreme micro-rounding doesn't matter
    GPS_Longitude FLOAT
);
```

---

## 9. Common Mistakes

*   **Saving Phone Numbers or Zip Codes as INT:** This is a brutal architectural error. The Zip code for Boston is `02108`. If you insert that into an `INT` column, MySQL applies integer math to it, stripping the leading zero, and saves `2108`. The Zip code is destroyed. **Never use Numeric types for numbers that don't do math! Use VARCHAR for Zip codes and Phones.**
*   **Using FLOAT for Money:** `FLOAT` uses approximate CPU binary-fractions. If you add `$0.10 + $0.20` using `FLOAT` data types, MySQL will often spit back `$0.30000000000000004`. Over thousands of bank transactions, these floating-point rounding errors will destroy an accounting ledger. *Always* use `DECIMAL` for money!

---

## 10. Tips & Best Practices (Pro-Level)

*   **The Magic of `UNSIGNED`:** By default, every Integer in MySQL reserves exactly half of its memory capacity for negative numbers. If you declare `TINYINT`, you get `-128 to 127`. If you are tracking a user's Age, they can never be negative 15 years old. By explicitly declaring `TINYINT UNSIGNED`, you tell MySQL: *"Shift the memory block completely strictly to the positive side."* You instantly double your storage maximum (`0 to 255`) for absolutely 0 extra bytes of RAM cost! 
*   Always declare your `id INT AUTO_INCREMENT Primary Keys` as `UNSIGNED`. It instantly upgrades your table's max limit from 2 Billion rows to 4.2 Billion rows for free!

---

## 11. Mini Practice Tasks

*   **Task 1:** You are building an NBA Basketball statistics database. A player can score points in a single game. What is the most memory-efficient Data Type (including the `UNSIGNED` modifier) you should use for `Points_Scored`? (Hint: Does anyone score more than 200?)
*   **Task 2:** Write down the exact `DECIMAL(M,D)` format required if you want to store a billionaire's bank account with a maximum possible value of `$9,999,999,999.99` (nine billion).

---
