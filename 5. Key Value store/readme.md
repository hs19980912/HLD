# Key-Value store  | Amazon Dynamo DB

## 1. Goals
- SCALABILITY
- DE-CENTRALIZATION
- EVENTUAL CONSISTENCY

## 2. STEPS
1. **Partition**: `Consistent hashing` 
2. **Replication**: `Coordinator`, `Preference List`
3. **GET and PUT requests**: `R + W > N`
4. **Data Versioning**: `Vector Clocks`, `Client side conflict resolution`, `Eventual Consistency`
5. **GOSSIP protocol**: `Node sending periodic status update`
6. **Merkel Tree**: `Hash tree`

## Client-Side Conflict Resolution

Yes, in systems like **Amazon DynamoDB** (and other **Dynamo-inspired** databases like Riak), the **client** is often responsible for resolving conflicts when **vector clocks** detect multiple conflicting versions of a data item. The server uses vector clocks to detect the conflicts but doesn't resolve them automatically. Instead, it leaves the conflict resolution to the client application for the following reasons:

### Conflict Detection in DynamoDB
In distributed systems with **eventual consistency**, conflicts can occur when multiple clients write to different replicas of the same data concurrently. When such a conflict happens, DynamoDB (or any Dynamo-inspired system) tracks the different versions of the data using **vector clocks**. 

- **Vector Clock**: Each version of the data has a vector clock, which allows the system to detect whether one version happened before another (i.e., a causal relationship) or if the versions are concurrent (i.e., conflicting).

- **Version Divergence**: When a conflict is detected (i.e., concurrent updates happen at different replicas), DynamoDB does **not** automatically decide which version should win. Instead, it stores **both versions** of the object and flags the conflict.

### Client-Side Conflict Resolution
When a client reads a data item that has conflicting versions (as indicated by vector clocks), DynamoDB returns **multiple versions** of the data to the client. The client is then responsible for resolving the conflict by applying domain-specific logic. The resolution can take many forms, such as:

- **Merge the Versions**: The client might merge the conflicting versions. For instance, if the conflicting versions represent changes to different fields of an object, the client can combine the fields into a single resolved version.

- **Last Write Wins (LWW)**: The client can choose to use a **last-write-wins** strategy, where the version with the latest timestamp is selected as the winner. However, this approach can lead to data loss if the updates are not truly independent.

- **Custom Application Logic**: The client can use any application-specific conflict resolution logic. For example, in an e-commerce application, the client might prioritize resolving stock updates by taking the higher value in case of conflicting inventory counts.

- **Manual Resolution**: In some cases, the system might prompt a human operator to resolve the conflict, especially when automated resolution is not feasible.

### Why Client-Side Conflict Resolution?
There are several reasons why Dynamo-style databases leave conflict resolution to the client:

- **Application-Specific Logic**: Different applications have different rules for resolving conflicts. For example, in a shopping cart system, merging items from both carts might be appropriate, whereas in a bank transaction system, resolving to the latest balance might be needed. The server has no knowledge of the application’s specific requirements, so it delegates the conflict resolution to the client.

- **Flexibility**: By allowing clients to resolve conflicts, the system provides flexibility in handling different types of data and use cases. Some data may benefit from merging, while others may require strict last-write-wins semantics or manual intervention.

- **Eventual Consistency**: In systems with eventual consistency, clients are expected to tolerate temporary inconsistencies and handle them appropriately. Conflict resolution is part of this model, where clients play an active role in maintaining consistency.

### How It Works in Practice
Here’s how the conflict detection and resolution process works in a Dynamo-inspired system like DynamoDB:

1. **Write Operation**: Two clients write to the same key (data item) on different replicas. Each write is tagged with a vector clock.

2. **Conflict Detection**: When the replicas eventually synchronize their data, the system detects that both replicas have different versions of the same data (due to conflicting vector clocks).

3. **Read Operation**: When a client reads the data, it returns all conflicting versions (along with their vector clocks).

4. **Client Resolves Conflict**: The client application inspects the versions and applies its logic to resolve the conflict (e.g., merging fields or selecting one version as the winner).

5. **Write Back Resolved Version**: The client writes back the resolved version to the database, updating the data