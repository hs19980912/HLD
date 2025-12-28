Lets take the doctors example and see how both transactions look like in Read committed, Repeatable Read and Serializable Isolation levels.

---

# The Doctors-on-Call Example (Setup)

### Business invariant

> **At least one doctor must be on call**

### Initial database state

```
Doctor A → on_call = true
Doctor B → on_call = true
```

### Two concurrent transactions

**T1**

```sql
BEGIN;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- check
UPDATE doctors SET on_call = false WHERE name = 'A';
COMMIT;
```

**T2**

```sql
BEGIN;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- check
UPDATE doctors SET on_call = false WHERE name = 'B';
COMMIT;
```

---

# 1️⃣ Read Committed (MVCC)

### What happens step by step

**T1**

```
SELECT COUNT(*) → sees 2
```

**T2**

```
SELECT COUNT(*) → sees 2
```

(Both queries see the latest committed state at *their own statement time*)

---

**Writes**

```
T1 updates A → commits
T2 updates B → commits
```

### Final state

```
Doctor A → off
Doctor B → off
```

❌ **Invariant violated**

---

### Why Read Committed allows this

* Snapshot is taken **per statement**
* Reads are independent
* DB does **not** reason about relationships between rows
* No conflict at row level

---

### Verdict

❌ Broken invariant
✅ Allowed by Read Committed

---

# 2️⃣ Repeatable Read (MVCC)

### What changes?

* Snapshot is taken **once at transaction start**

---

### Step by step

**T1 snapshot**

```
A = on, B = on
```

**T2 snapshot**

```
A = on, B = on
```

---

**Reads**

```
T1 sees 2
T2 sees 2
```

---

**Writes**

```
T1 updates A → commits
T2 updates B → commits
```

### Final state

```
Doctor A → off
Doctor B → off
```

❌ **Invariant still violated**

---

### Why Repeatable Read still allows this

* Snapshot consistency ≠ serializability
* No row-level conflict
* Writes touch **different rows**
* DB sees no direct conflict

This is the classic **write skew anomaly**.

---

### Verdict

❌ Broken invariant
✅ Allowed by Repeatable Read

---

# 3️⃣ Serializable Isolation (SSI – modern MVCC)

Now things change.

---

### Step by step

**Both transactions**

```
Take snapshot: A = on, B = on
Read predicate: WHERE on_call = true
```

DB internally tracks:

* Predicate read
* Rows later written

---

### Conflict detection

* T1 writes row A (part of T2’s read predicate)
* T2 writes row B (part of T1’s read predicate)

This creates a **cycle**:

```
T1 →rw T2
T2 →rw T1
```

🚨 **No serial order possible**

---

### What the DB does

* Aborts **one** transaction (say T2)
* Commits the other (T1)

---

### Final state

```
Doctor A → off
Doctor B → on
```

✅ **Invariant preserved**

---

### Verdict

✅ Correct behavior
❌ One transaction must retry

---

# Side-by-Side Summary

| Isolation Level | Both Commit?   | Invariant Preserved? | Why                                  |
| --------------- | -------------- | -------------------- | ------------------------------------ |
| Read Committed  | ✅              | ❌                    | Statement-level snapshots            |
| Repeatable Read | ✅              | ❌                    | Snapshot isolation allows write skew |
| Serializable    | ❌ (one aborts) | ✅                    | Detects non-serializable pattern     |

---

## The Big Takeaway (Lock This In)

> **Read Committed and Repeatable Read protect data *values*.
> Serializable protects *relationships between data*.**

That’s the leap in strength.

---

## Interview-Ready One-Liner

> In the doctors-on-call example, both Read Committed and Repeatable Read allow write skew because the transactions update different rows without conflicts. Serializable isolation prevents this by detecting a non-serializable read–write dependency cycle and aborting one transaction, preserving the invariant.


---