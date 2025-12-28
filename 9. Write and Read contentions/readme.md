
### "Read" and "Write" contentions are different problems in distributed systems

- Reads want visibility and consistency.
- Writes want ordering and conflict resolution. 
- **Note:** Scaling writes is a much harder problem to solve. It requires ordering, conflict resolution and agreement across nodes. These properties inherently demand coordination, which introduces latency and serialization.  
- **Read contention:**
    - Problem:
        - Many clients wants to read same data.
        - Reads dont modify state.
        - Blocking reads kills throughput.
    - Challenges:
        - Stale reads
        - Inconsistent snapshots
        - Replica lag
    - Typical Solutions:
        - MVCC (Readers dont block writers)
        - Replication (Scale read horizontally)
        - Caching (Redis, CDN..) (Avoid DB entirely)
        - Read-only snapshots (Consistent analytics)
        - Eventual consistency (Trades freshness for scale)
- **Write contention:**
    - Problem:
        - Multiple clients want to modify the same logical data.
        - Writes must preserve correctness.
    - Challenges:
        - Lost updates
        - Ordering
        - Conflicts
        - Distributed coordination
    - Typical solutions:
        - Leader-based wirtes (Single authority)
        - Optimistic concurrency(CAS/version) (Detect conflicts)
        - Pessimistic locking (Prevents conflicts)
        - Sharding (Reduces contention domain)
        - CRDTs (Conflict-free Replicated Data Types) (Avoid conflicts entirely) 


## Read Contention

- **Transaction**
    - In releational databases, a transaction is a logical unit of work that groups one or more SQL operations so they are treated as one single indivisible operation. 
    - Every relational database transaction guarantees ACID.
- **DB Locking**
    - No other transaction updates the locked Row.
    - 2 types:
        1. Shared lock (Read purpose) (S)
        2. Exclusive lock (Write purpose) (X)
    - If a Tx has Shared Lock: 
        - Other Tx can aquire a Shared lock
        - Other Tx cannot aquire an Exclusive lock
    - If a Tx has Exclusive lock:
        - Other Tx cannot aquire Shared lock
        - Other Tx cannot aquire Exclusive lock
- **Isolation level**
    - SQL isolation levels define how visible other transactions changes are to a running transaction.
    - Isolation level in a way determins the concurrecncy level a system can handle.
    - **Note:** Isolation levels are defined by SQL, but the concurrency and isolation concepts they represent apply broadly across both SQL and non-SQL databases.
    - **Read anamolies**:
        - **Dirty read**: A Tx reads uncommitted data written by another Tx (Reading data that was never committed).
        - **Non-repeatable read**: A Tx reads the same row twice and gets different values (Reading commited data that later got changed).
        - **Pantom read**: Same query predicate, different rows returned after another transaction’s committed insert/delete.
    - **Isolation levels:**
        ```
        | Isolation Level  | Dirty Reads  | Non-Repeatable Reads | Phantom Reads |
        | ---------------- | ------------ | -------------------- | ------------- |
        | Read Uncommitted | Possible     | Possible             | Possible      | High concurrency
        | Read Committed   | Not possible | Possible             | Possible      |
        | Repeatable Read  | Not possible | Not possible         | Possible      |
        | Serializable     | Not possible | Not possible         | Not possible  | Low concurrency
        ```
        - Read Uncommitted: Almost never used.
        - Read Committed: Most commonly used in real world.
        - Repeatable Read: MVCC-based repeatable read (Snapshot Isolation) is also very common.
        - Serializable: Used only in specific critical cases. 
    - **Locking Strategy:**
        - **Note:** Modern databases uses MVCC based isolation levels.
        1. **Read uncommited:** 
            - No locks.
        2. **Read Commited: MVCC based**  
            - **Read Committed = “What’s true right now?”**  
            - **Reads:** 
                - No shared lock → Read from a snapshot (Snapshot is per statement) → Never block writes
                - Performance:
                    - Snapshot is per statement
                    - After a statement finishes: snapshot is gone.
                    - Old versions become obsolete quickly.
                    - Vacuum/GC can clean aggersively. Hence better performance than MVCC based "Repeatable Read".   
            - **Writes:** 
                - Acquire X lock on row → creates new version → lock held until commit → Prevents other writers
        3. **Repeatable Read: MVCC based**  
            - **Repeatable Read = “What was true when I started?”**    
            - **Reads:**
                - Snapshot is held for entire transaction.
                - Even if the transaction is idle: Old versions must be kept alive.
                - Garbage collection is blocked, Old versions pile up.
            - **Writes:**
                - Same as Read committed.
        4. **Serializable:**
            - Serializable isolation guarantees that the outcome of concurrent transactions is equivalent to some serial (one-after-another) execution order.
            - Serializable does NOT mean:
                - Transactions literally run one-by-one
                - The DB uses a single thread
                - Reads always block writes
            - How Serializable Is Implemented (Two Main Ways):
                1. **Strict Two-Phase Locking (2PL) (Pessimistic Serializable) - Old approach**
                    - HEAVY BLOCKING
                    - Transactions acquire locks before accessing data, it locks all matching rows, and the range/predicate itself so new matching rows can be added later.
                    - Locks are held until commit.
                    - Uses range, predicate and range locks.
                    - Prevents conflicting operations upfront.
                    - Major Shortcomings:
                        - Heavy blocking - Reads block writes, Writes block read.
                        - Low concurrency.
                2. **Serializable Snapshot Isolation (SSI) - Modern Approach**
                    - AVOIDS READ BLOCKING
                    - Used by modern MVCC DBs + MVCC + optimistic validation.
                    - Transactions run using snapshots (like Repeatable Read)
                    - DB tracks:
                        - Read → write dependencies.
                        - Write → read dependencies.
                        - Predicate reads.
                    - DB builds a dependency graph.
                        - If a cycle appears, then the Txs are not serializable.
                        - DB aborts one transaction / Retries.
                    - Why this prevents all anomalies
                        - Any invariant violation implies a non-serializable schedule.
                        - Such non-serializable schedules are detected.
                        - One transaction is rolled back.
                        - Business correctness emerges automatically.
                    - Why SSI is still expensive
                        - Dependency tracking overhead
                        - Abort + retry cost
                        - predicate conflicts are complex.
                        - But it avoids read blocking which is crucial to scale.

- **MVCC terminology: (Multiversion Concurrency Control)**
    1. **Snapshot:**
        - A snapshot is a logical view of the database that defines which committed changes are visible to a transaction at a specific point in time. IT IS NOT A COPY OF THE DATA. Think of them more like a filter.
        - **Snapshots Are Cheap to Create**
            - Creating a snapshot means:
                - Recording a few transactions IDs
                - Updating visibility metadata
            - No data copying → O(1) operation
        - **Snapshots Are Expensive to Hold**
            - As long as your snapshot exists:
                - Old row versions cannot be deleted.
                - Garbage collection must wait.
            - Thats why long transactions hurt MVCC systems.




## Write contention
- One important gyan ki baat: 
    - **In distributed systems, locks are expensive. Retries are cheaper than waiting.**

- 2 fundamentally different mindsets to deal with write contention:
    1. **Pessimistic concurrency control**: 
    2. **Optimistic concurrency control**

- __Pessimistic locking__:
    - What is a lock?
        - It is simply **Shared state + enforcement logic** that everyone agrees to respect.
    - Where does this lock live?
        - On some authoritative component that all writers must talk to.
    - In distributed systems, this authority can live in three main places:
        1. Locks inside a Database (Row / Table Locks)
        2. Distributed Locks (External Lock Service)
        3. Leader-Based Systems (Implicit Locking)
    - In pessimistic locking, reads blocks write and write blocks reads.
    - **Locks inside a Database (Row / Table Locks)**:
        - f
    - **Distributed Locks (External Lock Service)**:
        - f
    - **Leader-Based Systems (Implicit Locking)**:
        - Most important one in modern distributed systems.
        - Instead of explicit locks, Only one node is allowed to write. Others are forbidden "by design".
        - Mental model: “You don’t need locks if only one writer exists.” The leader implicitly holds the lock.
        - Examples:
            - Kafka → partition leader
            - Raft / Paxos → leader node
            - Primary-replica databases
            - Dynamo-style quorum leaders
        - **Kafka**: Each partition has one leader, Producers write only to leader. Followers replicate, No write-write conflict possible.
        - **Primary DB**: Writes go only to primary. Replicas are read-only. Primary crash → new leader elected.
        - Why this scales better:
            - No per-record locks.
            - No waiting clients.
            - Serialization happens naturally.
        - Real-world usage:
            - Event logs
            - Messaging systems
            - Metadata services
            - Consensus systems


- **Optimistic concurrency**
    - 

