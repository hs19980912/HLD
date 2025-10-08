# HLD
## 1. Network Protocols
 - Network Protocols
 - what is Client server model
 - web sockets
 - Peer to Peer model
 - HTTP vs TCP vs UDP vs FTP vs SMTP(POP, IMAP)

##
  - Application Layer
    - Client Server protocal
      - HTTP (Stateless)
      - FTP
      - SMTP
      - Web Sockets (Stateful)
    - P2P protocol
      - WebRTC

## 2. CAP Theorem
 - CAP Theorem
 - Never Tradeoff P (Partition tolerance)

## 3. Monolith vs Microservice
 - Monolithic and Microservices architecture.
 - Decomposition Pattern
     - By Business capibility
     - by sub domain

## 4. Strangler Pattern
    Generally used when refactoring the code
    i.e migrating from monolothic to microservice architecture.

    Key Components:
    1. Incremental Replacement
    2. Proxy layer

   ![image](https://github.com/hs19980912/HLD/assets/63532987/cfd19c2c-fe4c-4517-ab46-f938e3ef6430)


## Database management in Microservices
    There are 2 ways of database management:
    1. separate DB for each service.
    2. single DB for the entire application.

## Separate DB for each microservice is the preferred way
    It has several advantages like decoupling, scalability, tailored data model etc.

    But it also has 2 major complexities in seperate DB approach.
    1. Maintaining the transactional property i.e ACID properties over distributed DBs.
    2. Join operations.
  
    SAGA pattern and CQRS overcomes these complexities in separate DB approach.

## SAGA pattern - solving transactional integrity
    SAGA pattern solves the challenge of a transaction spanning a distributed DB. It helps maintain data consistency across services using sequence of local transactions.

    SAGA - Sequence of local events

## TODO - Types of Sagas 
  - SAGA
    - Choreography: Each service manages its own transactions and listens to events from other services.
    - Orchestration: A centralized orchestrator manages the transaction flow and handles compensation logic.
    
## CQRS - Solving joins in distributed DBs


## Microservice



####  🧩 1. “Can I assume that in any backend framework, there will be separate REST controllers for each of the microservices?”

✅ **Yes, absolutely.**

Each **microservice** is an **independent deployable unit**, and inside it, you’ll have:

* A **Controller Layer** → defines REST endpoints
* A **Service Layer** → contains business logic
* A **Repository Layer** → talks to the database or cache

For example, in **Spring Boot**:

```java
// ProductController.java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService productService;

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        return ResponseEntity.ok(productService.createProduct(product));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return ResponseEntity.ok(productService.getProduct(id));
    }
}
```

So, yes — you can confidently say:

> “Each microservice has its own controller (or multiple controllers) that expose REST APIs relevant to that bounded context.”

---

#### 🔄 2. “How do microservices talk to each other in such an **Orchestration Pattern**?”

Great — in **Orchestration**, there is a **central coordinator** (the “orchestrator”) that *actively calls* other services.

### Example:

The **Order Service** orchestrates the flow:

1. Receives the checkout request from client
2. Calls **Cart Service** (REST) → fetch cart items
3. Calls **Inventory Service** (REST or gRPC) → reserve items
4. Calls **Payment Service** → initiate payment
5. Calls **Notification Service** → send order confirmation

📡 **Communication is synchronous** most of the time —
typically **HTTP REST** or **gRPC**, depending on latency and reliability needs.

So, yes —
In orchestration:

* One service (the orchestrator) knows the sequence of calls
* Communication is **direct** and **request–response** based (usually REST or gRPC)

---

#### 🕊️ 3. “In Choreography pattern, can I assume that majority of the communication between microservices happens using Queues?”

✅ **Exactly right.**

In a **Choreography pattern**, there’s **no central coordinator** —
services communicate **indirectly** through **events**.

### Example:

In our shopping cart system:

1. `Order Service` publishes `OrderCreated` event → Queue/Topic (Kafka, RabbitMQ)
2. `Inventory Service` consumes that event → decrements stock → emits `StockReserved`
3. `Payment Service` listens to `StockReserved` → initiates payment → emits `PaymentCompleted`
4. `Notification Service` listens to `PaymentCompleted` → sends email

Here:

* Communication = **Asynchronous**
* Medium = **Event Bus / Message Queue** (Kafka, RabbitMQ, SQS, etc.)
* Each service only **reacts to events** it cares about.

So yes, your statement is 100% correct 👏

> In choreography, the services mostly communicate through asynchronous events published on message queues or topics.

---

## 🧠 Summary

| Pattern           | Coordination                                   | Communication Type             | Example Medium       |
| ----------------- | ---------------------------------------------- | ------------------------------ | -------------------- |
| **Orchestration** | Central orchestrator (one service drives flow) | **Synchronous** (REST/gRPC)    | HTTP, gRPC           |
| **Choreography**  | Decentralized (event-driven)                   | **Asynchronous** (event-based) | Kafka, RabbitMQ, SQS |

---
