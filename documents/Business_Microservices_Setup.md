# Microservices Project: Business Services Implementation

**Phase:** Step B — Implementing Business Logic
**Goal:** Build the core e-commerce services using **MongoDB** for storage and **Apache Kafka** for asynchronous communication.

## Architecture
- Product Service — Manages product catalog (REST API + MongoDB)
- Customer Service — Manages customer information (REST API + MongoDB)
- Order Service — Handles orders (REST API + MongoDB + Kafka Producer)
- Inventory Service — Updates stock (Kafka Consumer)
- Payment Service — Processes payments (Kafka Consumer + MongoDB)

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

## Customer Service (customer-service)

**Role:** Customer management (source of truth for customer data)

### Generate Project
- Artifact: `customer-service`
- Dependencies: `Spring Web`, `Spring Data MongoDB`, `Eureka Discovery Client`, `Lombok`

### Configuration
File: `src/main/resources/application.properties`

```properties
server.port=5006
spring.application.name=customer-service

# MongoDB Connection
spring.data.mongodb.uri=mongodb://localhost:27017/customer-db

# Eureka Registration
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
```

### Domain Model
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
}
```

### Repository
File: `src/main/java/com/edureka/customer/repository/CustomerRepository.java`

```java
package com.edureka.customer.repository;

import com.edureka.customer.model.Customer;
import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.List;

public interface CustomerRepository extends MongoRepository<Customer, String> {
    List<Customer> findByEmail(String email);
}
```

### Controller
File: `src/main/java/com/edureka/customer/controller/CustomerController.java`

```java
package com.edureka.customer.controller;

import com.edureka.customer.model.Customer;
import com.edureka.customer.repository.CustomerRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.Map;
import java.util.UUID;

@RestController
@RequestMapping("/api/customers")
public class CustomerController {

    @Autowired
    private CustomerRepository repository;

    @PostMapping
    public ResponseEntity<?> createCustomer(@RequestBody Customer customer) {
        if (customer == null || customer.getEmail() == null || customer.getEmail().isBlank()) {
            return ResponseEntity.badRequest()
                    .body(Map.of("error", "Customer email is required"));
        }
        customer.setId(UUID.randomUUID().toString());
        Customer saved = repository.save(customer);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    @GetMapping("/email/{email}")
    public ResponseEntity<?> getCustomerByEmail(@PathVariable String email) {
        return repository.findByEmail(email).stream()
                .findFirst()
                .<ResponseEntity<?>>map(ResponseEntity::ok)
                .orElse(ResponseEntity.status(HttpStatus.NOT_FOUND)
                        .body(Map.of("error", "Customer not found")));
    }

    @GetMapping("/{id}")
    public ResponseEntity<?> getCustomerById(@PathVariable String id) {
        return repository.findById(id)
                .<ResponseEntity<?>>map(ResponseEntity::ok)
                .orElse(ResponseEntity.status(HttpStatus.NOT_FOUND)
                        .body(Map.of("error", "Customer not found")));
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
import java.math.BigDecimal;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderPlacedEvent {
    private String orderNumber;
    private String orderId;
    private String email;
    private String skuCode;
    private Integer quantity;
    private BigDecimal amount;
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
import java.math.BigDecimal;
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
        order.setId(UUID.randomUUID().toString());
        order.setOrderNumber(UUID.randomUUID().toString());
        orderRepository.save(order);
        
        // Calculate total amount
        BigDecimal totalAmount = order.getPrice().multiply(BigDecimal.valueOf(order.getQuantity()));
        
        // Publish event to Kafka for Payment and Inventory services
        kafkaTemplate.send("orderPlacedTopic", new OrderPlacedEvent(
            order.getOrderNumber(),
            order.getId(),
            order.getEmail(),
            order.getSkuCode(),
            order.getQuantity(),
            totalAmount
        ));
        
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
import java.math.BigDecimal;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderPlacedEvent {
    private String orderNumber;
    private String orderId;
    private String email;
    private String skuCode;
    private Integer quantity;
    private BigDecimal amount;
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

    @KafkaListener(topics = "orderPlacedTopic", groupId = "inventory-group")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Received Order Notification - Order: {}, SKU: {}, Quantity: {}", 
                 event.getOrderNumber(), event.getSkuCode(), event.getQuantity());
        // Update inventory logic goes here (deduct quantity from stock)
    }
}
```

---

## Payment Service (payment-service)

**Role:** Listens for order events and processes payments

### Generate Project
- Artifact: `payment-service`
- Dependencies: `Spring Web`, `Spring Data MongoDB`, `Spring for Apache Kafka`, `Eureka Discovery Client`, `Lombok`

### Configuration
File: `src/main/resources/application.properties`

```properties
server.port=5007
spring.application.name=payment-service

# MongoDB Connection
spring.data.mongodb.uri=mongodb://localhost:27017/payment_db

# Kafka Consumer Config
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=payment-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*

eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
```

### Domain Model
File: `src/main/java/com/edureka/payment/model/Payment.java`

```java
package com.edureka.payment.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.math.BigDecimal;

@Document(collection = "payments")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment {
    @Id
    private String id;
    private String orderId;
    private BigDecimal amount;
    private String status;
}
```

### Repository
File: `src/main/java/com/edureka/payment/repository/PaymentRepository.java`

```java
package com.edureka.payment.repository;

import com.edureka.payment.model.Payment;
import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.Optional;

public interface PaymentRepository extends MongoRepository<Payment, String> {
    Optional<Payment> findByOrderId(String orderId);
}
```

### Event DTO (copy from Order Service)
File: `src/main/java/com/edureka/payment/event/OrderPlacedEvent.java`

```java
package com.edureka.payment.event;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.math.BigDecimal;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderPlacedEvent {
    private String orderNumber;
    private String orderId;
    private String email;
    private String skuCode;
    private Integer quantity;
    private BigDecimal amount;
}
```

### Kafka Listener
File: `src/main/java/com/edureka/payment/listener/PaymentListener.java`

```java
package com.edureka.payment.listener;

import com.edureka.payment.event.OrderPlacedEvent;
import com.edureka.payment.model.Payment;
import com.edureka.payment.repository.PaymentRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class PaymentListener {

    @Autowired
    private PaymentRepository repository;

    @KafkaListener(topics = "orderPlacedTopic", groupId = "payment-group")
    public void processPayment(OrderPlacedEvent event) {
        log.info("Payment Service: Processing payment for Order {}", event.getOrderNumber());
        
        Payment payment = new Payment();
        payment.setOrderId(event.getOrderId() != null ? event.getOrderId() : event.getOrderNumber());
        payment.setAmount(event.getAmount() != null ? event.getAmount() : java.math.BigDecimal.ZERO);
        payment.setStatus("SUCCESS");
        
        repository.save(payment);
        log.info("Payment Saved Successfully");
    }
}
```

### Controller (Optional - for querying payments)
File: `src/main/java/com/edureka/payment/controller/PaymentController.java`

```java
package com.edureka.payment.controller;

import com.edureka.payment.model.Payment;
import com.edureka.payment.repository.PaymentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.Map;

@RestController
@RequestMapping("/api/payments")
public class PaymentController {

    @Autowired
    private PaymentRepository repository;

    @GetMapping("/{id}")
    public ResponseEntity<?> getPaymentById(@PathVariable String id) {
        return repository.findById(id)
                .<ResponseEntity<?>>map(ResponseEntity::ok)
                .orElse(ResponseEntity.status(HttpStatus.NOT_FOUND)
                        .body(Map.of("error", "Payment not found")));
    }

    @GetMapping("/order/{orderId}")
    public ResponseEntity<?> getPaymentByOrderId(@PathVariable String orderId) {
        return repository.findByOrderId(orderId)
                .<ResponseEntity<?>>map(ResponseEntity::ok)
                .orElse(ResponseEntity.status(HttpStatus.NOT_FOUND)
                        .body(Map.of("error", "Payment not found for order")));
    }
}
```

---

## Testing the Flow

### Prerequisites
1. Start infrastructure services:
   - Eureka Server (port 5001)
   - API Gateway (port 5002)

2. Start dependencies:
   - MongoDB (port 27017)
   - Zookeeper (port 2181)
   - Kafka (port 9092)

3. Start business services:
   - Product Service (port 5003)
   - Order Service (port 5004)
   - Inventory Service (port 5005)
   - Customer Service (port 5006)
   - Payment Service (port 5007)

### Verification Steps

1. **Verify Service Registration**  
   Open http://localhost:5001 in your browser and confirm all services are registered and showing status UP.

2. **Create a Customer**  
   ```http
   POST http://localhost:5006/api/customers
   Content-Type: application/json

   {
     "firstName": "John",
     "lastName": "Doe",
     "email": "john.doe@edureka.com"
   }
   ```

3. **Create a Product**  
   ```http
   POST http://localhost:5003/api/products
   Content-Type: application/json

   {
     "name": "iPhone 13",
     "description": "Latest Apple smartphone",
     "price": 999.99
   }
   ```

4. **Place an Order**  
   ```http
   POST http://localhost:5004/api/orders
   Content-Type: application/json

   {
     "skuCode": "iphone_13",
     "price": 999.99,
     "quantity": 1,
     "email": "john.doe@edureka.com"
   }
   ```

5. **Check Kafka Event Processing**  
   - Check Inventory Service logs — you should see: "Received Order Notification - Order: [UUID], SKU: iphone_13, Quantity: 1"
   - Check Payment Service logs — you should see: "Payment Service: Processing payment for Order [UUID]" and "Payment Saved Successfully"

6. **Verify Payment Was Created**  
   ```http
   GET http://localhost:5007/api/payments/order/{orderId}
   ```
   Replace `{orderId}` with the order ID from step 4. You should receive a payment record with status "SUCCESS".

### Expected Flow
1. Customer creates account → stored in customer-db
2. Product is created → stored in product-db
3. Order is placed → stored in order-db
4. OrderPlacedEvent published to Kafka topic `orderPlacedTopic`
5. Inventory Service consumes event → logs order details and updates stock
6. Payment Service consumes event → creates payment record in payment_db with SUCCESS status
