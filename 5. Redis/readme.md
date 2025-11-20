# 🟥 **REDIS**

## 🔹 **1. What is Redis?**

Redis is an **open-source, in-memory, key–value data structure store** used as:

* Cache
* Database
* Message broker
* Real-time data engine

Main characteristic → **In-memory = extremely fast (microseconds).**

---

# 🔹 **2. Redis Is Not Just Key-Value — It Exposes Many Data Structures**

### **Core Data Structures**

1. **Strings** → Cache, counters
2. **Hashes** → Store objects (user profile, product metadata)
3. **Lists** → Queues, jobs, activity feeds
4. **Sets** → Unique items, tags, groups
5. **Sorted Sets (ZSET)** → **Leaderboards, ranking, sliding-window rate limits**
6. **Bitmaps** → User activity tracking, flags
7. **HyperLogLog** → Approximate unique counts (UV)
8. **GeoSpatial Index** → Nearest drivers, delivery agents, nearby stores
9. **Streams** → Event logs, message queues, consumer groups
10. **Pub/Sub** → Real-time notifications, chat

### **Redis Modules (additional types)**

* RedisJSON
* RedisSearch (fulltext + secondary indexes)
* RedisBloom (Bloom/Cuckoo filters)
* RedisGraph
* RedisTimeSeries

---

# 🔹 **3. Redis = In-Memory Engine**

* Redis stores data **primarily in RAM**, making it ultra-fast
* But it can **persist to disk** using:

  * RDB snapshots
  * AOF (Append Only File) logging

You do **not** handle memory, sorting, indexing manually → **Redis manages everything internally.**

---

# 🔹 **4. Client–Server Model**

* Redis is a **server** running separately (local machine, pod, VM).
* Your microservice uses **Redis client libraries** (Jedis, Lettuce, redis-py, go-redis) to communicate.

Flow:

```
Your Microservice → Redis Client Library → Redis Server (port 6379)
```

---

# 🔹 **5. Redis in Microservices**

Your service uses Redis like a database:

* Business logic → service layer
* Repository layer → interacts with Redis
* Redis client library is the “ORM-like” abstraction for Redis
* Redis handles RAM, persistence, eviction, atomicity internally

---

# 🔹 **6. Redis Deployment**

* You *can* run Redis **in the same pod** as your app (for dev)
* In production → **Redis runs as a separate pod / server**
* Why?

  * Redis is *stateful*, your application is *stateless*
  * Scaling apps shouldn’t duplicate data
  * Redis needs dedicated memory/CPU

---

# 🔹 **7. Most Important Redis Use Cases**

### ✔ **Caching Layer**

Speed up reads, offload DB.

### ✔ **Leaderboards (Sorted Sets)**

* Realtime ranking
* Online games
* Competition apps

### ✔ **Rate Limiting**

* Sliding Window → **uses Sorted Sets**
* Fixed Window → counters
* Token Bucket → counters + TTL
* Leaky Bucket → counters + timers

### ✔ **Geospatial Queries (GeoIndex)**

Used in:

* Uber / Ola
* Swiggy / Zomato
* Logistics (FedEx, DHL)
* Tinder, Bumble
* Google Maps-like “near me” features

### ✔ **Queues & Messaging**

* Lists → simple queues
* Streams → persistent queues + consumer groups
* Pub/Sub → real-time push events

### ✔ **Analytics**

* HyperLogLog → Unique visitors (UV)
* Bitmaps → Daily active users, login streaks

### ✔ **Session Store**

High-speed session storage for web apps.

---

# 🔹 **8. Why Use Redis Instead of In-Memory Maps?**

Redis gives:

* Network access (cross-service)
* Atomic operations
* Persistence (AOF/RDB)
* Eviction policies (LRU, LFU)
* Replication + clustering
* High performance and reliability
* Data shared across multiple microservices

Much safer than using a HashMap or TreeMap inside your app.

---

# 🔹 **9. Redis Performance**

* Single-threaded event loop with I/O multiplexing
* Every operation is **O(1) or O(log n)**
* Predictable latency: usually **<1 ms**

---

# 🔹 **10. Open Source vs Managed Redis**

Redis (core engine) → **Open source**
Redis as a service → **Paid** (Redis Enterprise, AWS ElastiCache, Azure Redis, GCP MemoryStore)

---

# 🔹 **11. Important Sorted Set Use Cases**

1. **Leaderboard** (MOST famous use case)
2. **Sliding Window Rate Limiter**
3. **Time-series events**
4. **Priority queues**
5. **Analytics sorted by value (top N)**

---

# 🔹 **12. Redis is a “Mature In-Memory Data Structure Engine”**

You can think of it as:

> “Redis = a production-grade, network-accessible collection of optimized data structures (Lists, HashMaps, Sets, Sorted Sets, Streams, etc.) stored in-memory.”

You could implement these in your app — but Redis does it **faster, safer, and with persistence + clustering**.

---

# 🟩 **THE REDIS CRASH SHEET — ONE LINE PER TOPIC**

* **What is Redis?** In-memory data structure store, super fast.
* **Primary trait?** In-memory.
* **Why so fast?** RAM + optimized data structures.
* **Key types?** Strings, Hash, List, Set, SortedSet, Bitmap, HLL, Stream, Geo.
* **Main uses?** Cache, leaderboard, rate limit, pub/sub, sessions, analytics, geospatial.
* **Leaderboards?** Yes → SortedSets.
* **Rate Limiting?** Sliding Window → SortedSets.
* **Geo queries?** Yes → GeoIndex built over SortedSets.
* **Persistence?** RDB + AOF.
* **Deployment?** Microservice uses client library → Redis server.
* **Why not HashMap?** Redis gives safety, atomicity, persistence, sharing, scaling.
* **Open source?** Yes.
* **Managed services?** ElastiCache, Redis Enterprise.

---
