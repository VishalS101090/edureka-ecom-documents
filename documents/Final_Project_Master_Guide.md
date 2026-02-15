Microservices Final Project: Master Implementation Guide

Project: E-Commerce Microservices Platform
Tech Stack: Spring Boot 3, Spring Cloud, MongoDB, Apache Kafka, Docker

Service Registry: Port 5001
Gateway: Port 5002

---

## Part 1 — Shared Configuration (Prerequisites)

Before starting, ensure MongoDB (27017) and Kafka (9092) are running.

- Unified Eureka URL: all services must register with:

  http://localhost:5001/eureka/

---

## Part 2 — Infrastructure Services

### Discovery Server (Eureka)

- Port: 5001
- Dependency: Eureka Server
- Main class: add `@EnableEurekaServer` to the Spring Boot application
- File: `src/main/resources/application.properties`

```properties
server.port=5001
spring.application.name=discovery-server
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```

### API Gateway (Spring Cloud Gateway)

- Port: 5002
- Dependencies: Gateway, Eureka Discovery Client
- File: `src/main/resources/application.properties`

```properties
server.port=5002
spring.application.name=api-gateway

# Register with Eureka (Discovery Server)
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Auto-routing
spring.cloud.gateway.discovery.locator.enabled=true
spring.cloud.gateway.discovery.locator.lower-case-service-id=true

# (Optional) Swagger/OpenAPI aggregation
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.urls[0].name=product-service
springdoc.swagger-ui.urls[0].url=/product-service/v3/api-docs
springdoc.swagger-ui.urls[1].name=order-service
springdoc.swagger-ui.urls[1].url=/order-service/v3/api-docs
```

---

## Part 3 — Business Microservices

### Product Service (product-service)

- Port: 5003
- Dependencies: Spring Web, Spring Data MongoDB, Eureka Client, Lombok, SpringDoc OpenAPI
- File: `src/main/resources/application.properties`

```properties
server.port=5003
spring.application.name=product-service
spring.data.mongodb.uri=mongodb://localhost:27017/product-db
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
```

- Model: `Product.java` (id, name, description, price)
- Controller: `ProductController.java` (GET /api/products, POST /api/products)

### Order Service (order-service)

- Port: 5004
- Dependencies: Spring Web, Spring Data MongoDB, Spring for Apache Kafka, Eureka Client, Lombok, SpringDoc OpenAPI
- File: `src/main/resources/application.properties`

```properties
server.port=5004
spring.application.name=order-service
spring.data.mongodb.uri=mongodb://localhost:27017/order-db
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Kafka producer
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

Code (Kafka producer example):

```java
@Autowired
private KafkaTemplate<String, OrderPlacedEvent> kafkaTemplate;

@PostMapping("/api/orders")
public String placeOrder(@RequestBody Order order) {
    repository.save(order);
    kafkaTemplate.send("notificationTopic", new OrderPlacedEvent(order.getOrderNumber(), order.getEmail()));
    return "Order Placed";
}
```

### Inventory Service (inventory-service)

- Port: 5005
- Dependencies: Spring Web, Eureka Client, Spring for Apache Kafka, Lombok
- File: `src/main/resources/application.properties`

```properties
server.port=5005
spring.application.name=inventory-service
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Kafka consumer
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=inventory-group
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

Listener example:

```java
@KafkaListener(topics = "notificationTopic", groupId = "inventory-group")
public void updateStock(OrderPlacedEvent event) {
    log.info("Updating stock for order: {}", event.getOrderNumber());
}
```

### Customer Service (customer-service)

- Port: 5006
- Dependencies: Spring Web, Spring Data MongoDB, Eureka Client, Lombok
- File: `src/main/resources/application.properties`

```properties
server.port=5006
spring.application.name=customer-service
spring.data.mongodb.uri=mongodb://localhost:27017/customer-db
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
```

Controller: standard CRUD for customer entities.

### Payment Service (payment-service)

- Port: 5007
- Dependencies: Spring Web, Spring Data MongoDB, Spring for Apache Kafka, Eureka Client, Lombok
- File: `src/main/resources/application.properties`

```properties
server.port=5007
spring.application.name=payment-service
spring.data.mongodb.uri=mongodb://localhost:27017/payment-db
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Kafka consumer (same topic, different group)
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=payment-group
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

Listener example:

```java
@KafkaListener(topics = "notificationTopic", groupId = "payment-group")
public void processPayment(OrderPlacedEvent event) {
    log.info("Processing payment for order: {}", event.getOrderNumber());
    // persist payment to MongoDB
}
```

---

## Part 4 — API Documentation (Swagger / OpenAPI)

Add SpringDoc dependency to services that should expose docs (Product, Order, etc.):

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.2.0</version>
</dependency>
```

Access docs after services start:

- Product Service: http://localhost:5003/swagger-ui.html
- Order Service: http://localhost:5004/swagger-ui.html
- (Gateway aggregation) http://localhost:5002/webjars/swagger-ui/index.html

---

## Part 5 — Final Execution Plan

1. Start Infrastructure (strict order):
   - Discovery Server (verify at http://localhost:5001)
   - API Gateway

2. Start Business Services:
   - Product Service
   - Order Service
   - Inventory Service
   - Customer Service
   - Payment Service

3. Verification:
   - Refresh http://localhost:5001 — you should see 6 instances: API-GATEWAY, PRODUCT-SERVICE, ORDER-SERVICE, INVENTORY-SERVICE, CUSTOMER-SERVICE, PAYMENT-SERVICE.
   - Test end-to-end: POST to http://localhost:5002/order-service/api/orders with body {"skuCode":"test-item","email":"user@test.com"}
   - Check logs: Order Service logs "Order Placed", Inventory Service logs "Updating stock...", Payment Service logs "Processing payment..."
