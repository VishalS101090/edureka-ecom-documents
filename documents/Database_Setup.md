# Microservices Project: Database Setup Guide (MongoDB)

**Pattern:** Database-per-service

**Technology:** MongoDB (NoSQL)

**Goal:** Configure each microservice with its own dedicated database to ensure loose coupling and independent scaling.

---

## Concept: Database per Service

In this architecture, no service should access another service's database directly. Each service owns its database and collections:

- Product Service — `product_db`
- Order Service — `order_db`
- Inventory Service — `inventory_db`
- Customer Service — `customer_db`
- Payment Service — `payment_db`

Note: MongoDB creates databases and collections lazily. Spring Boot will create the database/collection when you save the first record.

---

## Product Service (Catalog)

- Database: `product_db`
- Collection: `products`

### Configuration
File: `src/main/resources/application.properties`

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/product_db
```

### Data Model (example)
File: `src/main/java/com/edureka/product/model/Product.java`

```java
package com.edureka.product.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.math.BigDecimal;

@Document(collection = "products")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    @Id
    private String id;
    private String name;
    private String description;
    private BigDecimal price;
    private String category;
    private String imageUrl;
}
```

---

## Order Service (Transactions)

- Database: `order_db`
- Collection: `orders`

### Configuration
File: `src/main/resources/application.properties`

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/order_db
```

### Data Models
File: `src/main/java/com/edureka/order/model/Order.java`

```java
package com.edureka.order.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.time.LocalDateTime;
import java.util.List;

@Document(collection = "orders")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    @Id
    private String id;
    private String orderNumber;
    private String customerEmail;
    private List<OrderLineItems> orderLineItems;
    private String status;
    private LocalDateTime orderDate;
}
```

File: `src/main/java/com/edureka/order/model/OrderLineItems.java`

```java
package com.edureka.order.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.math.BigDecimal;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderLineItems {
    private String skuCode;
    private BigDecimal price;
    private Integer quantity;
}
```

---

## Inventory Service (Stock Management)

- Database: `inventory_db`
- Collection: `inventory`

### Configuration
File: `src/main/resources/application.properties`

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/inventory_db
```

### Data Model
File: `src/main/java/com/edureka/inventory/model/Inventory.java`

```java
package com.edureka.inventory.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "inventory")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Inventory {
    @Id
    private String id;
    private String skuCode;
    private Integer quantity;
}
```

---

## Customer Service (User Profiles)

- Database: `customer_db`
- Collection: `customers`

### Configuration
File: `src/main/resources/application.properties`

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/customer_db
```

### Data Model
File: `src/main/java/com/edureka/customer/model/Customer.java`

```java
package com.edureka.customer.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "customers")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Customer {
    @Id
    private String id;
    private String firstName;
    private String lastName;
    private String email;
    private String phoneNumber;
    private Address address;
}
```

File: `src/main/java/com/edureka/customer/model/Address.java`

```java
package com.edureka.customer.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
}
```

---

## Payment Service (Transactions)

- Database: `payment_db`
- Collection: `payments`

### Configuration
File: `src/main/resources/application.properties`

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/payment_db
```

### Data Model
File: `src/main/java/com/edureka/payment/model/Payment.java`

```java
package com.edureka.payment.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Document(collection = "payments")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment {
    @Id
    private String id;
    private String orderId;
    private String transactionId;
    private BigDecimal amount;
    private String paymentStatus;
    private LocalDateTime paymentDate;
}
```

---

## Verification Steps

1. Start your Spring Boot services (infrastructure and business services).
2. Insert data (POST a product or place an order) using Postman or curl.
3. Open MongoDB Compass and connect to `mongodb://localhost:27017`.
4. Click Refresh; you should see the databases:

- `product_db`
- `order_db`
- `inventory_db`
- `customer_db`
- `payment_db`

