# 🗄️ Topic 1.3: Types of Databases (Relational, NoSQL, Hierarchical, Network)

A database isn't a "one-size-fits-all" construct. System architects map completely different data structures and theorems onto different engines. Understanding why we have NoSQL vs SQL is the hallmark of a Senior computing professional.

---

## 1. Definition

1.  **Relational Databases (RDBMS):** Stores data in strict **Tables** linked by Keys. Backed mathematically by Relational Algebra and Set Theory. Highly optimized using B+ Tree indexing. (Examples: **MySQL**, PostgreSQL).
2.  **NoSQL Databases (Non-Relational):** Built for horizontal scaling and schema-less flexibility. 
    *   *Document:* Stores JSON/BSON. (MongoDB)
    *   *Key-Value:* In-memory hash maps for blazing speed. (Redis)
    *   *Graph:* Nodes and Edges for complex relationship mapping. (Neo4j)
    *   *Wide-Column:* Petabyte-scale sparse data matrices. (Cassandra)
3.  **Hierarchical:** Legacy 1960s systems. Data in strict Tree formats (1 Parent -> Many Children). Used exclusively today in mainframes or Windows Registry concepts.
4.  **Network:** Complex graph webs where elements can have Multiple Parents. Superceded by modern Relational capabilities.

---

## 2. Why This Concept Exists

**The CAP Theorem:** In distributed computer science, it is mathematically impossible for a database to guarantee all three of these simultaneously if a network fails:
*   **C**onsistency (Everyone reads the exact same newest data)
*   **A**vailability (The system is always online)
*   **P**artition Tolerance (The system survives network breaks between servers)

Relational (MySQL) usually chooses **Consistency** (ACID). NoSQL databases often choose **Availability** (BASE - Basically Available, Soft state, Eventual consistency). We have different databases because businesses have different CAP theorem priorities!

---

## 3. Why We Use It (Why so many types?)

*   **Bank Account (SQL):** You absolutely *require* Consistency. If you transfer $100, the ledger must perfectly balance instantly. 
*   **YouTube Likes (NoSQL):** You favor *Availability*. It doesn't matter if your screen says "1,005 likes" while your friend's screen says "1,003 likes" (Eventual Consistency). The database prioritizing Availability handles millions of clicks without crashing.

---

## 4. When to Use It

*   **Relational (MySQL):** Financial ledgers, CRM systems, e-commerce transactions, any domain where data strongly interrelates and schema changes are rare and methodical.
*   **NoSQL (MongoDB / Redis):** Rapid application prototyping, IoT sensor data (massive write volumes), caching layers (millisecond responses), real-time chat apps.

---

## 5. How It Works (Storage Mechanics)

*   **Relational (B+ Tree):** Data is organized strictly to fit into memory block pages. To do an update, it must seek the exact disk sector and lock the row safely.
*   **NoSQL (LSM Trees):** Many NoSQL systems use Log-Structured Merge Trees. They never "update" raw disk directly. They just append (add) the new data to the end of a log sequentially, making write speeds astonishingly fast (but reads slightly more complex).

---

## 6. Syntax / Implementation Showdown

*A conceptual look at how the exact same data maps onto different paradigms:*

**Relational (MySQL) - Strict Schema & Normalization:**
```sql
-- Table 1: Users (Contains core info only)
User_ID | Name  | Age
1       | John  | 25

-- Table 2: Orders (Linked by Foreign Key constraint)
Order_ID | User_ID | Item
99       | 1       | Laptop
```

**NoSQL (MongoDB) - Denormalized JSON Document:**
```json
{
  "_id": "5f1a3b4",
  "name": "John",
  "age": 25,
  "orders": [
    { "order_id": 99, "item": "Laptop" }
  ]
}
```
*Pro Insight: The NoSQL document is retrieved in a single disk read, whereas SQL might require multiple disk seeks to JOIN the tables (though mitigated by modern caching).*

---

## 7. Real-Life Architectures

**The Modern Tech Stack:**
*   Users browse Netflix -> The frontend rapidly reads user profiles and session tokens from **Redis (Key-Value NoSQL)** because it's 100x faster than reading from disk.
*   Users watch diverse movies -> Their viewing history is logged to **Cassandra (Wide-Column NoSQL)** to handle millions of simultaneous write actions globally.
*   Netflix processes their monthly subscription payment -> Handled exclusively by a **PostgreSQL/MySQL Cluster (RDBMS)** because billing requires strict ACID transactions.

---

## 8. SQL Examples (MySQL)

Here is a `JOIN`, the fundamental mechanism by which RDBMS calculates relationships on-the-fly without needing to store duplicated JSON documents:

```sql
-- The mathematical Intersection of two sets of data
SELECT u.Name, o.Item 
FROM Users u
INNER JOIN Orders o ON u.User_ID = o.User_ID;
```

---

## 9. Common Mistakes

*   **The "NoSQL is Modern" Fallacy:** Juniors think SQL is dead. In reality, NoSQL handles very specific scaling niches. SQL databases have evolved and actually incorporated NoSQL features (MySQL supports JSON natively!). RDBMS remains the absolute backbone of the software engineering world.
*   **"NoSQL is faster" assumption:** NoSQL is often faster for *Writes* and fetching single denormalized documents. RDBMS is remarkably faster for complex aggregations, filtering, and reporting relying on multi-table analytics.

---

## 10. Tips & Best Practices

*   **Start with RDBMS:** Unless you have a specific, measurable reason that MySQL will fail (e.g., you anticipate writing 100,000 JSON blobs per second), default to PostgreSQL or MySQL. You can scale an RDBMS massively before needing NoSQL.
*   **Polyglot Persistence:** Learn at least one RDBMS (MySQL) and one NoSQL (Redis caching). Using the right tool for the right microservice is elite-level architecture.

---

## 11. Mini Practice Tasks

*   **Task 1:** Look up the "CAP Theorem". Write down which two attributes an RDBMS typically focuses on (Hint: Consistency is definitely one).
*   **Task 2:** Write down the difference between a B-Tree (SQL) and an LSM Tree (Many NoSQL databases) in one sentence regarding how they handle writing data.

---
