# Microservices Project: Business Services Implementation

**Phase:** Step B — Implementing Business Logic
**Goal:** Build the core e-commerce services using **MongoDB** for storage and **Apache Kafka** for asynchronous communication.

## Architecture
- Product Service — Manages product catalog (REST API + MongoDB)
- Order Service — Handles orders (REST API + MongoDB + Kafka Producer)
- Inventory Service — Updates stock (Kafka Consumer)

---

## Prerequisites
Before starting these services, ensure the following are running:

1. **Infrastructure:** Eureka Server (5001) and API Gateway (5002)
2. **Database:** MongoDB (27017)
3. **Messaging:** Apache Kafka (9092) and Zookeeper (2181)

---

## Product Service (product-service)

**Role:** Product catalog (source of truth)

### Generate Project
- Artifact: `product-service`
- Dependencies: `Spring Web`, `Spring Data MongoDB`, `Eureka Discovery Client`, `Lombok`

### Configuration
File: `src/main/resources/application.properties`

```properties
server.port=5003
spring.application.name=product-service

# MongoDB Connection
spring.data.mongodb.uri=mongodb://localhost:27017/product-db

# Eureka Registration
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
```

### Domain Model
File: `src/main/java/com/edureka/product/model/Product.java`

```java
package com.edureka.product.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "products")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    @Id
    private String id;
    private String name;
    private String description;
    private Double price;
}
```

### Repository
File: `src/main/java/com/edureka/product/repository/ProductRepository.java`

```java
package com.edureka.product.repository;

import com.edureka.product.model.Product;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface ProductRepository extends MongoRepository<Product, String> {
}
```

### Controller
File: `src/main/java/com/edureka/product/controller/ProductController.java`

```java
package com.edureka.product.controller;

import com.edureka.product.model.Product;
import com.edureka.product.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductRepository productRepository;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product createProduct(@RequestBody Product product) {
        return productRepository.save(product);
    }

    @GetMapping
    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }
}
```

---

## Order Service (order-service)

**Role:** Accepts orders, persists them, and publishes events to Kafka

### Generate Project
- Artifact: `order-service`
- Dependencies: `Spring Web`, `Spring Data MongoDB`, `Spring for Apache Kafka`, `Eureka Discovery Client`, `Lombok`

### Configuration
File: `src/main/resources/application.properties`

```properties
server.port=5004
spring.application.name=order-service

# MongoDB Connection (Separate DB)
spring.data.mongodb.uri=mongodb://localhost:27017/order-db

# Kafka Producer Config
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Note: Start MongoDB and Kafka via the repository's docker-compose.yml (see docker-compose.yml)
```

### Domain Model
File: `src/main/java/com/edureka/order/model/Order.java`

```java
package com.edureka.order.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.math.BigDecimal;

@Document(collection = "t_orders")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    @Id
    private String id;
    private String orderNumber;
    private String skuCode;
    private BigDecimal price;
    private Integer quantity;
    private String email;
}
```

### Event DTO
File: `src/main/java/com/edureka/order/event/OrderPlacedEvent.java`

```java
package com.edureka.order.event;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderPlacedEvent {
    private String orderNumber;
    private String email;
}
```

### Repository
File: `src/main/java/com/edureka/order/repository/OrderRepository.java`

```java
package com.edureka.order.repository;

import com.edureka.order.model.Order;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface OrderRepository extends MongoRepository<Order, String> {
}
```

### Controller (with Kafka)
File: `src/main/java/com/edureka/order/controller/OrderController.java`

```java
package com.edureka.order.controller;

import com.edureka.order.event.OrderPlacedEvent;
import com.edureka.order.model.Order;
import com.edureka.order.repository.OrderRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.*;
import java.util.UUID;

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private KafkaTemplate<String, OrderPlacedEvent> kafkaTemplate;

    @PostMapping
    public String placeOrder(@RequestBody Order order) {
        order.setOrderNumber(UUID.randomUUID().toString());
        orderRepository.save(order);
        kafkaTemplate.send("notificationTopic", new OrderPlacedEvent(order.getOrderNumber(), order.getEmail()));
        return "Order Placed Successfully";
    }
}
```

---

## Inventory Service (inventory-service)

**Role:** Listens for order events and updates stock

### Generate Project
- Artifact: `inventory-service`
- Dependencies: `Spring Web`, `Spring for Apache Kafka`, `Eureka Discovery Client`, `Lombok`

### Configuration
File: `src/main/resources/application.properties`

```properties
server.port=5005
spring.application.name=inventory-service

# Kafka Consumer Config
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=inventory-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*

eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
```

### Event DTO (copy)
File: `src/main/java/com/edureka/inventory/event/OrderPlacedEvent.java`

```java
package com.edureka.inventory.event;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderPlacedEvent {
    private String orderNumber;
    private String email;
}
```

### Kafka Listener
File: `src/main/java/com/edureka/inventory/listener/InventoryListener.java`

```java
package com.edureka.inventory.listener;

import com.edureka.inventory.event.OrderPlacedEvent;
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class InventoryListener {

    @KafkaListener(topics = "notificationTopic", groupId = "inventory-group")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Received Notification for Order - {}", event.getOrderNumber());
        // Update inventory logic goes here
    }
}
```

---

## Testing the Flow

1. Start infrastructure: Eureka, API Gateway
2. Start dependencies: MongoDB, Kafka (+ Zookeeper)
3. Start business services: Product, Order, Inventory

Verify registration: open http://localhost:5001 and confirm services are UP

Place an order via the API Gateway (example):

```http
POST http://localhost:5004/order-service/api/orders
Content-Type: application/json

{
  "skuCode": "iphone_13",
  "price": 1000,
  "quantity": 1,
  "email": "test@edureka.com"
}
```

Check logs for Inventory Service — you should see: "Received Notification for Order - [UUID]"
