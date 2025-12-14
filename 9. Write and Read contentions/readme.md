- One imp gyan ki baat: 
    - **In distributed systems, locks are expensive. Retries are cheaper than waiting.**

- 2 fundamentally different mindsets to deal with write contention:
    1. **Pessimistic locking**: 
    2. **Optimistic concurrency**

- __Pessimistic locking__:
    - What is a lock?
        - It is simply **Shared state + enforcement logic** hat everyone agrees to respect.
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