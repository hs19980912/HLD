
# What Exactly Is Serializable Isolation Level?

## Precise Definition

> **Serializable isolation guarantees that the outcome of concurrent transactions is equivalent to some serial (one-after-another) execution order.**

Key points:

* The serial order is **logical**, not wall-clock
* Transactions may overlap in time
* But the **result must match a valid serial schedule**

This is the **strongest isolation level in SQL**.

---

## What Serializable Guarantees (Nothing Less)

Serializable isolation prevents **all anomalies**:

| Anomaly              | Prevented? |
| -------------------- | ---------- |
| Dirty reads          | ✅          |
| Non-repeatable reads | ✅          |
| Phantom reads        | ✅          |
| Read skew            | ✅          |
| Write skew           | ✅          |
| Lost updates         | ✅          |

If any of these would occur → **transaction must not be allowed to commit**.

---

## The Core Requirement (This Drives Everything)

> **The database must prevent executions for which no serial order exists.**

This requirement is what makes Serializable **hard and expensive**.

---

# Two Ways to Implement Serializable Isolation

There are **two fundamentally different approaches**:

1. **Old / Classic way** → *Pessimistic locking*
2. **Modern way** → *Optimistic + conflict detection (SSI)*

Let’s look at both.

---

## 1️⃣ Older Way: Strict Two-Phase Locking (2PL)

### How it works

* Transactions acquire **locks before accessing data**
* Locks are held **until commit**
* Includes:

  * Row locks
  * Range locks
  * Predicate locks

### What is locked?

Not just rows — **relationships**.

Example:

```sql
SELECT * FROM doctors WHERE on_call = true;
```

DB must lock:

* All matching rows
* The **range/predicate itself**
  So no new matching row can be inserted.

---

### Why It Works

* Conflicts are **prevented upfront**
* No invalid interleavings possible
* Serializability guaranteed by construction

---

### Why It’s Expensive

* Reads block writes
* Writes block reads
* Long transactions stall the system
* Lock contention explodes under concurrency

---

### Characteristics

| Aspect            | Old Serializable (2PL) |
| ----------------- | ---------------------- |
| Conflict handling | Blocking               |
| Read locks        | Yes                    |
| Predicate locks   | Yes                    |
| Aborts            | Rare                   |
| Throughput        | Low under contention   |

---

### Where it was used

* Early relational databases
* Systems where correctness > throughput
* Single-node systems

---

## 2️⃣ Modern Way: Serializable Snapshot Isolation (SSI)

This is how **modern MVCC databases** do Serializable.

---

### Key Idea

> **Let transactions run freely, then abort them if the result cannot be serialized.**

This flips the model:

* Don’t prevent conflicts
* **Detect and resolve them**

---

### How SSI Works (Conceptually)

1. Each transaction runs using **MVCC snapshot**
   (same as Repeatable Read)

2. The DB tracks:

   * Read → write dependencies
   * Write → write dependencies
   * Predicate reads

3. The DB builds a **dependency graph**

4. If a **cycle** appears:

   ```
   T1 →rw T2
   T2 →rw T1
   ```

   → ❌ Not serializable

5. The DB **aborts one transaction**

---

### Why This Prevents All Anomalies

* Any invariant violation implies a non-serializable schedule
* Non-serializable schedules are detected
* One transaction is rolled back

**Business correctness emerges automatically.**

---

### Why SSI Is Still Expensive

* Dependency tracking overhead
* Memory for conflict metadata
* Abort + retry cost
* Predicate conflicts are complex

But it avoids **blocking**, which is crucial for scale.

---

### Characteristics

| Aspect            | Modern Serializable (SSI) |
| ----------------- | ------------------------- |
| Conflict handling | Abort & retry             |
| Read locks        | No                        |
| Predicate locks   | Logical tracking          |
| Aborts            | Common                    |
| Throughput        | High (until contention)   |

---

## Why the Industry Moved from 2PL → SSI

| Problem with 2PL  | SSI Advantage            |
| ----------------- | ------------------------ |
| Blocking          | Non-blocking reads       |
| Lock contention   | Optimistic execution     |
| Distributed locks | Local conflict detection |
| Poor scalability  | Better concurrency       |

SSI is **far more scalable**, especially in:

* High-read workloads
* Distributed systems
* MVCC-based databases

---

## How Serializable Differs from Repeatable Read (Final Contrast)

| Property                   | Repeatable Read | Serializable    |
| -------------------------- | --------------- | --------------- |
| Snapshot                   | Per transaction | Per transaction |
| Write skew                 | ❌ Allowed       | ✅ Prevented     |
| Predicate reasoning        | ❌               | ✅               |
| Aborts for logic conflicts | ❌               | ✅               |
| Complexity                 | Medium          | High            |

---

## Mental Model (Lock This In)

> **Serializable isolation doesn’t make transactions slower —
> it makes *incorrect histories impossible*.**

It pays the cost **only when concurrency threatens correctness**.

---

## Interview-Ready Summary

> Serializable isolation is the strongest SQL isolation level and guarantees that concurrent transactions produce results equivalent to some serial execution. Older systems implemented it using strict two-phase locking with row and predicate locks, which caused heavy blocking. Modern MVCC databases use Serializable Snapshot Isolation, where transactions run optimistically on snapshots and the database detects non-serializable conflict patterns at commit time, aborting transactions when necessary to preserve correctness.

---

Excellent — this is the **last missing mental brick**.
Once *predicate* clicks, **phantom reads, write skew, and SSI all become obvious**.

I’ll go in this order:

1. What does **predicate** mean (plain English)
2. What is a **predicate read** in a database
3. Why row-level tracking is **not enough**
4. How **SSI uses predicate reads**
5. Tie back to your doctors example

---

## 1️⃣ What Does “Predicate” Mean?

### Plain English

A **predicate** is simply a **condition that evaluates to true or false**.

Examples:

* `x > 10`
* `status = 'ACTIVE'`
* `on_call = true`
* `age BETWEEN 18 AND 60`

In SQL, predicates appear in:

```sql
WHERE <predicate>
```

---

## 2️⃣ What Is a Predicate Read?

### Definition

> A **predicate read** is a read operation that depends on a **condition**, not on specific row IDs.

Example:

```sql
SELECT * FROM doctors WHERE on_call = true;
```

This does **not** mean:

* “Read doctor A”
* “Read doctor B”

It means:

> “Read **all rows that satisfy this condition** — now and implicitly in the future.”

That “implicitly in the future” part is critical.

---

## 3️⃣ Why Row-Level Tracking Is Not Enough

Let’s contrast two types of reads.

### Row Read (simple)

```sql
SELECT * FROM doctors WHERE id = 1;
```

* DB knows exactly which row you read
* Conflicts are easy to detect

---

### Predicate Read (dangerous)

```sql
SELECT * FROM doctors WHERE on_call = true;
```

* DB does **not know in advance**:

  * which rows *might* qualify later
* New rows or updates may suddenly match this predicate

This is where **phantoms and write skew come from**.

---

## 4️⃣ Why Predicate Reads Matter (Core Problem)

Let’s see what happens **without predicate awareness**.

### Timeline

```
T1: SELECT COUNT(*) WHERE on_call = true  → sees 2
T2: UPDATE doctors SET on_call=false WHERE id=1
T1: UPDATE doctors SET on_call=false WHERE id=2
```

Row-level view:

* T1 read rows {1,2}
* T2 wrote row {1}

No direct row conflict!

But logically:

* T2 changed the **truth of the predicate** that T1 relied on

This is the **write skew anomaly**.

---

## 5️⃣ What SSI Tracks (This Is the Key)

Serializable Snapshot Isolation tracks **dependencies**, not business logic.

### The three dependency types you mentioned

---

### 🔹 1. Read → Write dependency (rw)

> Transaction T1 **read something** that transaction T2 **later wrote**

Example:

```text
T1 reads: WHERE on_call = true
T2 writes: doctor A → on_call=false
```

T1’s logic depended on data that T2 changed.

---

### 🔹 2. Write → Write dependency (ww)

> Two transactions write the **same row**

Classic conflict:

```text
T1 writes row A
T2 writes row A
```

Handled by row locks or version checks.

---

### 🔹 3. Predicate Read dependency (the subtle one)

> A transaction reads a **condition**, and another transaction writes data that **changes whether rows satisfy that condition**

This is **not** row overlap — it’s **predicate overlap**.

Example:

```sql
-- T1
SELECT * FROM doctors WHERE on_call = true;

-- T2
UPDATE doctors SET on_call = false WHERE name='A';
```

T2 didn’t modify “the SELECT” —
it modified the **truth of the predicate**.

---

## 6️⃣ How SSI Uses Predicate Reads

SSI does **not** check business rules.

Instead, it checks:

> “Did someone write data that would have changed what you *should* have seen?”

If yes:

* That’s a **dangerous structure**
* Combine that with another rw edge → cycle
* Cycle → **abort**

---

## 7️⃣ Doctors Example Revisited (Now Fully Clear)

### Both transactions do:

```sql
SELECT COUNT(*) FROM doctors WHERE on_call = true;
```

→ Both create a **predicate read dependency**

Then:

* T1 writes doctor A (inside predicate)
* T2 writes doctor B (inside predicate)

This forms:

```
T1 →rw T2
T2 →rw T1
```

No serial order exists.

💥 **SSI aborts one transaction**

---

## 8️⃣ Mental Model (Lock This In)

> **Row reads protect values.
> Predicate reads protect assumptions.**

Serializable isolation must protect **assumptions**, not just data.

---

## 9️⃣ Interview-Ready Summary

> A predicate is a logical condition used in a query, typically in a WHERE clause. A predicate read occurs when a transaction reads data based on such a condition rather than specific row identifiers. In Serializable Snapshot Isolation, predicate reads are tracked because other transactions may write data that changes whether rows satisfy the condition, leading to anomalies like write skew or phantom reads. By tracking predicate read–write dependencies, SSI detects non-serializable execution patterns and aborts transactions to preserve correctness.

---