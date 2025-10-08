```
product-service/
├── pom.xml
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/product/
│       │       ├── ProductServiceApplication.java
│       │       ├── controller/
│       │       │   └── ProductController.java
│       │       ├── service/
│       │       │   └── ProductService.java
│       │       ├── repository/
│       │       │   └── ProductRepository.java
│       │       └── model/
│       │           └── Product.java
│       └── resources/
│           ├── application.properties
│           └── data.sql (optional seed data)
```



# 🧩 Product Microservice Flow Diagram

```
Client (Browser / Mobile App)
         |
         | HTTP Request (POST /products) (Only Admin will make this call)
         v
+--------------------+
| ProductController  |  <--- REST Controller Layer
|  - Receives request|
|  - Validates input |
+--------------------+
         |
         v
+--------------------+
| ProductService     |  <--- Service Layer (Business Logic)
|  - Validates data  |
|  - Applies rules   |
|  - Calls repository|
+--------------------+
         |
         v
+--------------------+
| ProductRepository  |  <--- Data Access Layer (JPA)
|  - save()          |
|  - findById()      |
|  - findAll()       |
+--------------------+
         |
         v
+--------------------+
| MySQL Database     |  <--- Product Table
|  - id              |
|  - name            |
|  - price           |
|  - quantity        |
+--------------------+
         |
         v
HTTP Response (201 Created + Product JSON)
         |
         v
Client receives confirmation with product data
```

---

# 🔹 Step-by-Step Flow Explanation

1. **Client → Controller**

   * Client sends POST /products with JSON body:

     ```json
     {"name":"iPhone 16","price":100000,"quantity":10}
     ```
   * `ProductController` parses JSON into a `Product` object.

2. **Controller → Service**

   * `ProductController` calls `productService.createProduct(product)`.
   * `ProductService` performs any business validation:

     * Price > 0
     * Name not empty
     * Quantity > 0

3. **Service → Repository**

   * `ProductService` calls `productRepository.save(product)` to persist data.
   * `ProductRepository` uses **Spring Data JPA** to generate the correct SQL automatically.

4. **Repository → Database**

   * Hibernate generates SQL like:

     ```sql
     INSERT INTO products (name, price, quantity) VALUES ('iPhone 16', 100000, 10);
     ```
   * Database stores the product and generates an `id`.

5. **Repository → Service → Controller**

   * Repository returns the saved `Product` object (with `id` populated) to `ProductService`.
   * Service may apply post-processing (optional).
   * Controller wraps the `Product` in a `ResponseEntity` with HTTP status **201 CREATED**.

6. **Controller → Client**

   * JSON response sent back to client:

     ```json
     {
       "id": 1,
       "name": "iPhone 16",
       "price": 100000,
       "quantity": 10
     }
     ```

---

# 🔹 Key Notes for Interview

* **Separation of Concerns:** Controller → Service → Repository → DB
* **ResponseEntity** allows flexible response handling (status, headers, body)
* **JPA/Hibernate** automatically maps `Product` entity → DB table
* **HTTP Status:** 201 Created for POST
* **Optional:** `@Transactional` in service to ensure DB consistency