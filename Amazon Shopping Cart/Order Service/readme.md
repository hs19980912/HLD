    order-service/
    ├── pom.xml
    ├── src/
    │   └── main/
    │       ├── java/
    │       │   └── com/example/order/
    │       │       ├── OrderServiceApplication.java
    │       │       ├── controller/
    │       │       │   └── OrderController.java
    │       │       ├── service/
    │       │       │   └── OrderService.java
    │       │       ├── repository/
    │       │       │   └── OrderRepository.java
    │       │       ├── model/
    │       │       │   ├── Order.java
    │       │       │   └── OrderItem.java
    │       │       └── client/
    │       │           ├── CartClient.java
    │       │           ├── InventoryClient.java
    │       │           ├── PaymentClient.java
    │       │           └── NotificationClient.java
    │       └── resources/
    │           ├── application.properties
    │           └── data.sql (optional seed data)


### Order service is the orchestrator service