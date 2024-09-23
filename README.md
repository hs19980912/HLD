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