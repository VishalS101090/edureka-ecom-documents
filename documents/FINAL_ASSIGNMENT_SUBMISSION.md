# E-Commerce Microservices Platform - Final Assignment Submission

**Project Name:** E-Commerce Microservices Application  
**Technology Stack:** Spring Boot 3, Spring Cloud, MongoDB, Apache Kafka, Docker, Kubernetes  
**Submission Date:** February 17, 2026  
**Author:** MSA Training Project

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [System Requirements & Setup](#2-system-requirements--setup)
3. [Technology Stack](#3-technology-stack)
4. [Microservices Architecture](#4-microservices-architecture)
5. [Service Configurations](#5-service-configurations)
6. [Communication Patterns](#6-communication-patterns)
7. [Resilience Patterns](#7-resilience-patterns)
8. [Observability & Monitoring](#8-observability--monitoring)
9. [Database Configuration](#9-database-configuration)
10. [Deployment Process](#10-deployment-process)
11. [Service Connectivity & Testing](#11-service-connectivity--testing)
12. [Troubleshooting Guide](#12-troubleshooting-guide)
13. [Service Structure & Patterns](#13-service-structure--patterns)
14. [Appendix](#14-appendix)

---

## 1. Project Overview

### 1.1 Introduction
This project implements a complete e-commerce platform using microservices architecture with Spring Boot 3 and Spring Cloud. The system consists of 7 microservices (2 infrastructure + 5 business services) that demonstrate industry-standard patterns for distributed systems.

### 1.2 Key Features Implemented
- ✅ **Service Discovery** - Netflix Eureka for service registry
- ✅ **API Gateway** - Spring Cloud Gateway for routing
- ✅ **Database per Service** - Independent MongoDB databases
- ✅ **Synchronous Communication** - REST APIs with OpenFeign clients
- ✅ **Asynchronous Messaging** - Apache Kafka for event-driven architecture
- ✅ **Resilience Patterns** - Circuit Breaker, Retry, Timeout with Resilience4j
- ✅ **Distributed Tracing** - Zipkin with Micrometer
- ✅ **API Documentation** - Swagger/OpenAPI with gateway aggregation
- ✅ **Health Monitoring** - Spring Boot Actuator with Prometheus metrics
- ✅ **Containerization** - Docker images for all services
- ✅ **Orchestration** - Kubernetes manifests with liveness/readiness probes

### 1.3 Project Compliance
**Status:** 100% Complete - All functional and non-functional requirements met

---

## 2. System Requirements & Setup

### 2.1 Software Prerequisites

#### Java Development Kit
- **JDK Version:** 17 (eclipse-temurin recommended)
- **Verification:** `java -version`

#### Build Tool
- **Apache Maven:** 3.8+
- **Verification:** `mvn -version`

#### Database
- **MongoDB:** 7.0
- **Port:** 27017
- **Installation:** Download from https://www.mongodb.com/try/download/community
- **Verification:** `MongoDB Compass`

#### Message Broker
- **Apache Kafka:** 7.6.0 (Confluent Platform)
- **Zookeeper:** 7.6.0 (bundled)
- **Ports:** Kafka 9092, Zookeeper 2181

#### Tracing System
- **Zipkin:** Latest
- **Port:** 9411

#### Container Platform
- **Docker Desktop:** Latest
- **Kubernetes:** Enabled in Docker Desktop

#### Development IDE
- **IntelliJ IDEA:** Ultimate Edition
- **VS Code:** With Java extensions (alternative)

### 2.2 Infrastructure Setup Using Docker Compose

All external infrastructure (Kafka, Zookeeper, Zipkin) is managed via Docker Compose.

**Location:** `edureka-ecom-documents/docker_files/docker-compose.yml`

**Services Included:**
- Zookeeper (Port 2181)
- Kafka (Port 9092)
- Zipkin (Port 9411)
- MongoDB (Optional - can use local installation)

**Start Infrastructure:**
```bash
cd edureka-ecom-documents/docker_files
docker-compose up -d
```

**Verify Running Containers:**
```bash
docker ps
```

**Stop Infrastructure:**
```bash
docker-compose down
```

---

## 3. Technology Stack

### 3.1 Core Frameworks & Libraries

| Technology | Version | Purpose |
|------------|---------|---------|
| **Spring Boot** | 3.5.10 | Application framework |
| **Spring Cloud** | 2025.0.1 | Microservices infrastructure |
| **Java** | 17 | Programming language |
| **Maven** | 3.8+ | Build & dependency management |

### 3.2 Spring Cloud Components

| Component | Artifact | Usage |
|-----------|----------|-------|
| **Eureka Server** | spring-cloud-starter-netflix-eureka-server | Service registry |
| **Eureka Client** | spring-cloud-starter-netflix-eureka-client | Service discovery client |
| **Gateway** | spring-cloud-starter-gateway | API Gateway routing |
| **OpenFeign** | spring-cloud-starter-openfeign | Declarative REST client |
| **LoadBalancer** | spring-cloud-starter-loadbalancer | Client-side load balancing |
| **Resilience4j** | spring-cloud-starter-circuitbreaker-resilience4j | Circuit breaker, retry, timeout |

### 3.3 Data & Messaging

| Technology | Version | Purpose |
|------------|---------|---------|
| **MongoDB** | 7.0 | NoSQL database (database per service) |
| **Apache Kafka** | 7.6.0 | Distributed event streaming |
| **Spring Data MongoDB** | 3.5.10 | MongoDB integration |
| **Spring Kafka** | 3.5.10 | Kafka integration |

### 3.4 Observability & Documentation

| Technology | Purpose |
|------------|---------|
| **Zipkin** | Distributed tracing |
| **Micrometer Tracing** | Tracing abstraction with Brave bridge |
| **Spring Boot Actuator** | Health checks, metrics, monitoring |
| **Prometheus** | Metrics format (via Actuator) |
| **SpringDoc OpenAPI** | Swagger/OpenAPI 3 documentation |

### 3.5 Utilities

| Library | Purpose |
|---------|---------|
| **Lombok** | Reduce boilerplate code (@Data, @Slf4j) |
| **Jackson** | JSON serialization/deserialization |

---

## 4. Microservices Architecture

### 4.1 Service Overview

The application consists of 7 microservices deployed across 7 ports (5001-5007).

| Service | Port | Type | Database | Kafka Role | Description |
|---------|------|------|----------|------------|-------------|
| **discovery-server** | 5001 | Infrastructure | - | - | Eureka service registry |
| **api-gateway** | 5002 | Infrastructure | - | - | Single entry point, routing |
| **product-service** | 5003 | Business | product-db | - | Product catalog management |
| **order-service** | 5004 | Business | order-db | Producer | Order orchestration |
| **inventory-service** | 5005 | Business | inventory-db | Consumer | Stock management |
| **customer-service** | 5006 | Business | customer-db | - | Customer profile management |
| **payment-service** | 5007 | Business | payment_db | Consumer | Payment processing |

### 4.2 Architecture Diagram (Conceptual)

```
                                    ┌─────────────────┐
                                    │   Client Apps   │
                                    │  (Web/Mobile)   │
                                    └────────┬────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │  API Gateway    │
                                    │   (Port 5002)   │
                                    └────────┬────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
                    ▼                        ▼                        ▼
          ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
          │ Product Service  │    │ Customer Service │    │  Order Service   │
          │   (Port 5003)    │    │   (Port 5006)    │    │   (Port 5004)    │
          │    MongoDB       │    │    MongoDB       │    │    MongoDB       │
          └──────────────────┘    └──────────────────┘    └────────┬─────────┘
                                                                     │
                                                          Publishes Event
                                                                     │
                                                            ┌────────▼────────┐
                                                            │  Kafka Topic    │
                                                            │ orderPlacedTopic│
                                                            └────────┬────────┘
                                                                     │
                                                   ┌─────────────────┴─────────────────┐
                                                   │                                   │
                                                   ▼                                   ▼
                                         ┌──────────────────┐              ┌──────────────────┐
                                         │Inventory Service │              │ Payment Service  │
                                         │   (Port 5005)    │              │   (Port 5007)    │
                                         │    MongoDB       │              │    MongoDB       │
                                         └──────────────────┘              └──────────────────┘

                                    ┌─────────────────────────────────────┐
                                    │     Discovery Server (Eureka)       │
                                    │          (Port 5001)                │
                                    │  All services register here         │
                                    └─────────────────────────────────────┘

                                    ┌─────────────────────────────────────┐
                                    │     Distributed Tracing (Zipkin)    │
                                    │          (Port 9411)                │
                                    └─────────────────────────────────────┘
```

### 4.3 Service Dependencies

**Discovery Server:**
- No dependencies (runs standalone)

**API Gateway:**
- Depends on: Discovery Server

**Business Services:**
- All depend on: Discovery Server, MongoDB, Zipkin
- Order Service additionally depends on: Kafka, Product Service, Customer Service, Inventory Service
- Inventory Service additionally depends on: Kafka
- Payment Service additionally depends on: Kafka

---

## 5. Service Configurations

### 5.1 Discovery Server (Eureka)

**Port:** 5001  
**Purpose:** Service registry and discovery

**Key Properties (`application.properties`):**
```properties
spring.application.name=discovery-server
server.port=5001

# Eureka Configuration
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

# Distributed Tracing
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans

# Actuator Configuration
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
management.health.livenessState.enabled=true
management.health.readinessState.enabled=true
```

**Main Class Annotation:**
```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

**Dashboard Access:** http://localhost:5001

---

### 5.2 API Gateway

**Port:** 5002  
**Purpose:** Single entry point for all client requests

**Key Properties (`application.properties`):**
```properties
spring.application.name=API-GATEWAY
server.port=5002

# Eureka Configuration
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true

# Gateway Discovery Locator (Auto-routing)
spring.cloud.gateway.discovery.locator.enabled=true
spring.cloud.gateway.discovery.locator.lower-case-service-id=true

# Explicit Routes with Load Balancing
spring.cloud.gateway.routes[0].id=product-service
spring.cloud.gateway.routes[0].uri=lb://product-service
spring.cloud.gateway.routes[0].predicates[0]=Path=/product-service/**
spring.cloud.gateway.routes[0].filters[0]=StripPrefix=1

spring.cloud.gateway.routes[1].id=order-service
spring.cloud.gateway.routes[1].uri=lb://order-service
spring.cloud.gateway.routes[1].predicates[0]=Path=/order-service/**
spring.cloud.gateway.routes[1].filters[0]=StripPrefix=1

spring.cloud.gateway.routes[2].id=inventory-service
spring.cloud.gateway.routes[2].uri=lb://inventory-service
spring.cloud.gateway.routes[2].predicates[0]=Path=/inventory-service/**
spring.cloud.gateway.routes[2].filters[0]=StripPrefix=1

spring.cloud.gateway.routes[3].id=customer-service
spring.cloud.gateway.routes[3].uri=lb://customer-service
spring.cloud.gateway.routes[3].predicates[0]=Path=/customer-service/**
spring.cloud.gateway.routes[3].filters[0]=StripPrefix=1

spring.cloud.gateway.routes[4].id=payment-service
spring.cloud.gateway.routes[4].uri=lb://payment-service
spring.cloud.gateway.routes[4].predicates[0]=Path=/payment-service/**
spring.cloud.gateway.routes[4].filters[0]=StripPrefix=1

# CORS Configuration
spring.cloud.gateway.globalcors.cors-configurations[/**].allowed-origins=*
spring.cloud.gateway.globalcors.cors-configurations[/**].allowed-methods=*
spring.cloud.gateway.globalcors.cors-configurations[/**].allowed-headers=*

# Swagger Aggregation
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.urls[0].name=Customer Service
springdoc.swagger-ui.urls[0].url=/customer-service/v3/api-docs
springdoc.swagger-ui.urls[1].name=Product Service
springdoc.swagger-ui.urls[1].url=/product-service/v3/api-docs
springdoc.swagger-ui.urls[2].name=Order Service
springdoc.swagger-ui.urls[2].url=/order-service/v3/api-docs
springdoc.swagger-ui.urls[3].name=Inventory Service
springdoc.swagger-ui.urls[3].url=/inventory-service/v3/api-docs
springdoc.swagger-ui.urls[4].name=Payment Service
springdoc.swagger-ui.urls[4].url=/payment-service/v3/api-docs

# Distributed Tracing & Actuator
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
```

**Routing Pattern:**
- URL: `http://localhost:5002/<service-name>/<endpoint>`

---

### 5.3 Product Service

**Port:** 5003  
**Purpose:** Product catalog management  
**Database:** product-db

**Key Properties (`application.properties`):**
```properties
spring.application.name=product-service
server.port=5003

# MongoDB
spring.data.mongodb.uri=mongodb://localhost:27017/product-db

# Eureka
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Distributed Tracing & Actuator
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
management.health.livenessState.enabled=true
management.health.readinessState.enabled=true
```

**REST Endpoints:**
- POST `/api/products` - Create product
- GET `/api/products` - Get all products
- GET `/api/products/{id}` - Get product by ID
- GET `/api/products/sku/{skuCode}` - Get product by SKU code

**Domain Model:**
```java
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
    private String skuCode;
}
```

---

### 5.4 Order Service

**Port:** 5004  
**Purpose:** Order orchestration with resilience patterns  
**Database:** order-db  
**Kafka:** Producer (orderPlacedTopic)

**Key Properties (`application.properties`):**
```properties
spring.application.name=order-service
server.port=5004

# MongoDB
spring.data.mongodb.uri=mongodb://localhost:27017/order-db

# Kafka Producer
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Eureka
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Circuit Breaker Configuration (Product, Customer, Inventory)
resilience4j.circuitbreaker.instances.productService.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.productService.wait-duration-in-open-state=30000
resilience4j.circuitbreaker.instances.productService.sliding-window-size=10
resilience4j.circuitbreaker.instances.productService.minimum-number-of-calls=5
resilience4j.circuitbreaker.instances.productService.automatic-transition-from-open-to-half-open-enabled=true

# (Similar configuration for customerService and inventoryService)

# Retry Configuration
resilience4j.retry.instances.productService.max-attempts=3
resilience4j.retry.instances.productService.wait-duration=1000
resilience4j.retry.instances.productService.enable-exponential-backoff=true
resilience4j.retry.instances.productService.exponential-backoff-multiplier=2

# Timeout Configuration
resilience4j.timelimiter.instances.productService.timeout-duration=3s

# Distributed Tracing & Actuator
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=health,info,metrics,prometheus,circuitbreakers,circuitbreakerevents
management.endpoint.health.show-details=always
management.health.circuitbreakers.enabled=true
management.endpoint.health.probes.enabled=true
```

**Feign Clients:**
```java
@FeignClient(name = "product-service")
public interface ProductClient {
    @GetMapping("/api/products/sku/{skuCode}")
    ProductResponse getProductBySkuCode(@PathVariable String skuCode);
}

@FeignClient(name = "customer-service")
public interface CustomerClient {
    @GetMapping("/api/customers/email/{email}")
    CustomerResponse getCustomerByEmail(@PathVariable String email);
}

@FeignClient(name = "inventory-service")
public interface InventoryClient {
    @PutMapping("/api/inventory/deduct/{skuCode}")
    void deductInventory(@PathVariable String skuCode, @RequestParam Integer quantity);
}
```

**Service Method with Resilience:**
```java
@CircuitBreaker(name = "productService", fallbackMethod = "placeOrderFallback")
@Retry(name = "productService")
@TimeLimiter(name = "productService")
public CompletableFuture<String> placeOrder(OrderRequest request) {
    // Validate product, customer, inventory via Feign clients
    // Save order to MongoDB
    // Publish OrderPlacedEvent to Kafka
    return CompletableFuture.completedFuture("Order placed successfully");
}

public CompletableFuture<String> placeOrderFallback(OrderRequest request, Exception e) {
    return CompletableFuture.completedFuture(
        "Order service is temporarily unavailable. Please try again later."
    );
}
```

**REST Endpoints:**
- POST `/api/orders` - Place order (with resilience)
- GET `/api/orders` - Get all orders
- GET `/api/orders/{id}` - Get order by ID

**Event Published:**
```java
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

---

### 5.5 Inventory Service

**Port:** 5005  
**Purpose:** Stock management  
**Database:** inventory-db  
**Kafka:** Consumer (orderPlacedTopic, group: inventory-group)

**Key Properties (`application.properties`):**
```properties
spring.application.name=inventory-service
server.port=5005

# MongoDB
spring.data.mongodb.uri=mongodb://localhost:27017/inventory-db

# Kafka Consumer
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=inventory-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*

# Eureka
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Distributed Tracing & Actuator
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
```

**Kafka Listener:**
```java
@Service
@Slf4j
public class InventoryListener {
    
    @KafkaListener(topics = "orderPlacedTopic", groupId = "inventory-group")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Received order event: {}", event.getOrderNumber());
        // Update inventory stock
    }
}
```

**REST Endpoints:**
- GET `/api/inventory/check` - Check stock availability
- POST `/api/inventory` - Create/update inventory
- PUT `/api/inventory/deduct/{skuCode}` - Deduct stock

---

### 5.6 Customer Service

**Port:** 5006  
**Purpose:** Customer profile management  
**Database:** customer-db

**Key Properties (`application.properties`):**
```properties
spring.application.name=customer-service
server.port=5006

# MongoDB
spring.data.mongodb.uri=mongodb://localhost:27017/customer-db

# Eureka
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true

# Distributed Tracing & Actuator
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
```

**REST Endpoints:**
- POST `/api/customers` - Create customer
- GET `/api/customers/email/{email}` - Get customer by email
- GET `/api/customers/{id}` - Get customer by ID

**Domain Model:**
```java
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

---

### 5.7 Payment Service

**Port:** 5007  
**Purpose:** Payment processing  
**Database:** payment_db  
**Kafka:** Consumer (orderPlacedTopic, group: payment-group)

**Key Properties (`application.properties`):**
```properties
spring.application.name=payment-service
server.port=5007

# MongoDB
spring.data.mongodb.uri=mongodb://localhost:27017/payment_db

# Kafka Consumer (Different group ID from inventory)
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=payment-group
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*

# Eureka
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# Distributed Tracing & Actuator
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
```

**Kafka Listener:**
```java
@Service
@Slf4j
public class PaymentListener {
    
    @KafkaListener(topics = "orderPlacedTopic", groupId = "payment-group")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        log.info("Processing payment for order: {}", event.getOrderNumber());
        // Create payment record
    }
}
```

**REST Endpoints:**
- POST `/api/payments` - Create payment
- GET `/api/payments/{id}` - Get payment by ID
- GET `/api/payments/order/{orderId}` - Get payments for order

---

## 6. Communication Patterns

### 6.1 Synchronous Communication (REST + OpenFeign)

**Pattern:** Request-response communication for immediate data needs

**Implementation:**
- Order Service uses Feign clients to call Product, Customer, and Inventory services
- Client-side load balancing via Spring Cloud LoadBalancer
- Declarative REST clients with `@FeignClient` annotation

**Advantages:**
- Immediate response
- Strong consistency
- Simple error handling

---

### 6.2 Asynchronous Communication (Apache Kafka)

**Pattern:** Event-driven architecture for loosely coupled services

**Implementation:**
- **Topic:** `orderPlacedTopic`
- **Producer:** Order Service
- **Consumers:** Inventory Service (inventory-group), Payment Service (payment-group)
- **Serialization:** JSON (JsonSerializer/JsonDeserializer)

**Event Flow:**
```
Order Service → Kafka Topic → Inventory Service
                            → Payment Service
```

**Consumer Groups:**
- `inventory-group` - Ensures only one Inventory Service instance processes each event
- `payment-group` - Ensures only one Payment Service instance processes each event

**Advantages:**
- Loose coupling between services
- Resilience to downstream failures
- Scalability (multiple consumers per group)
- Temporal decoupling (consumers can be offline temporarily)

---

### 6.3 Service Discovery

**Pattern:** Dynamic service registration and discovery

**Implementation:**
- All services register with Eureka Server on startup
- Services discover each other by application name
- API Gateway and Feign clients use service names (not IPs/ports)

**Load Balancing:**
- Spring Cloud LoadBalancer automatically distributes requests across service instances
- Round-robin strategy by default

---

## 7. Resilience Patterns

### 7.1 Circuit Breaker

**Purpose:** Prevent cascading failures by failing fast when downstream services are unhealthy

**Implementation:** Resilience4j in Order Service

**Configuration:**
```properties
resilience4j.circuitbreaker.instances.productService.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.productService.wait-duration-in-open-state=30000
resilience4j.circuitbreaker.instances.productService.sliding-window-size=10
resilience4j.circuitbreaker.instances.productService.minimum-number-of-calls=5
```

**States:**
1. **CLOSED** (Normal) - Requests pass through
2. **OPEN** (Tripped) - Fast-fail without calling downstream service (50% failure rate reached)
3. **HALF_OPEN** (Testing) - After 30 seconds, allows limited calls to test recovery

**Usage:**
```java
@CircuitBreaker(name = "productService", fallbackMethod = "placeOrderFallback")
public CompletableFuture<String> placeOrder(OrderRequest request) {
    // Call downstream services
}
```

---

### 7.2 Retry Pattern

**Purpose:** Automatically retry failed requests to handle transient failures

**Configuration:**
```properties
resilience4j.retry.instances.productService.max-attempts=3
resilience4j.retry.instances.productService.wait-duration=1000
resilience4j.retry.instances.productService.enable-exponential-backoff=true
resilience4j.retry.instances.productService.exponential-backoff-multiplier=2
```

**Behavior:**
- Attempt 1: Immediate
- Attempt 2: After 1 second
- Attempt 3: After 2 seconds (exponential backoff)

**Usage:**
```java
@Retry(name = "productService")
public CompletableFuture<String> placeOrder(OrderRequest request) {
    // Will retry up to 3 times on failure
}
```

---

### 7.3 Timeout Pattern

**Purpose:** Prevent indefinite waiting for slow downstream services

**Configuration:**
```properties
resilience4j.timelimiter.instances.productService.timeout-duration=3s
```

**Behavior:**
- If downstream call takes more than 3 seconds, timeout exception is thrown
- Prevents thread pool exhaustion

**Usage:**
```java
@TimeLimiter(name = "productService")
public CompletableFuture<String> placeOrder(OrderRequest request) {
    // Must complete within 3 seconds
}
```

---

### 7.4 Fallback Methods

**Purpose:** Provide graceful degradation when resilience patterns are triggered

**Implementation:**
```java
public CompletableFuture<String> placeOrderFallback(OrderRequest request, Exception e) {
    log.error("Order placement failed, executing fallback", e);
    return CompletableFuture.completedFuture(
        "Order service is temporarily unavailable. Please try again later."
    );
}
```

**Triggers:**
- Circuit breaker is OPEN
- All retry attempts exhausted
- Timeout exceeded

---

## 8. Observability & Monitoring

### 8.1 Distributed Tracing (Zipkin)

**Purpose:** Track requests across multiple microservices

**Implementation:**
- All 7 services send traces to Zipkin at http://localhost:9411
- 100% sampling rate (all requests traced)

**Configuration (All Services):**
```properties
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
```

**Dependencies:**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

**Zipkin Dashboard:** http://localhost:9411/zipkin/

**Features:**
- View trace timeline across services
- Identify performance bottlenecks
- Debug distributed transactions
- Visualize service dependencies

---

### 8.2 Health Checks (Spring Boot Actuator)

**Purpose:** Monitor service health and readiness

**Configuration (All Services):**
```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
management.health.livenessState.enabled=true
management.health.readinessState.enabled=true
```

**Endpoints:**
```
# Health check
GET http://localhost:{port}/actuator/health

# Liveness probe (Kubernetes)
GET http://localhost:{port}/actuator/health/liveness

# Readiness probe (Kubernetes)
GET http://localhost:{port}/actuator/health/readiness
```

**Health Indicators:**
- **diskSpace** - Disk space availability
- **mongo** - MongoDB connection status (business services)
- **ping** - Application is running

---

### 8.3 Metrics (Prometheus)

**Purpose:** Expose metrics for monitoring systems

**Endpoint:**
```
GET http://localhost:{port}/actuator/prometheus
```

**Metrics Included:**
- JVM metrics (memory, threads, GC)
- HTTP server metrics (requests, duration, errors)
- Circuit breaker metrics (calls, failures, state)
- Custom application metrics

**Integration:**
- Prometheus can scrape these endpoints
- Grafana can visualize Prometheus data

---

### 8.4 API Documentation (Swagger/OpenAPI)

**Purpose:** Interactive API documentation for all services

**Individual Service Swagger:**
```
http://localhost:5003/swagger-ui.html  # Product Service
http://localhost:5004/swagger-ui.html  # Order Service
http://localhost:5005/swagger-ui.html  # Inventory Service
http://localhost:5006/swagger-ui.html  # Customer Service
http://localhost:5007/swagger-ui.html  # Payment Service
```

**Aggregated at Gateway:**
```
http://localhost:5002/swagger-ui.html
```

**Features:**
- Try out API endpoints directly from browser
- View request/response schemas
- See all available operations
- Copy cURL commands

**Dependency:**
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.2.0</version>
</dependency>
```

---

## 9. Database Configuration

### 9.1 Database per Service Pattern

**Principle:** Each microservice owns its database - no direct database access between services

**Implementation:**

| Service | Database Name | Collections |
|---------|---------------|-------------|
| Product Service | product-db | products |
| Order Service | order-db | orders |
| Inventory Service | inventory-db | inventory |
| Customer Service | customer-db | customers |
| Payment Service | payment_db | payments |

**MongoDB Connection:**
```properties
spring.data.mongodb.uri=mongodb://localhost:27017/{database-name}
```

**Auto-creation:**
- MongoDB creates databases and collections lazily on first write
- No manual database setup required

---

### 9.2 Data Models

**Product:**
```java
@Document(collection = "products")
public class Product {
    @Id private String id;
    private String name;
    private String description;
    private BigDecimal price;
    private String skuCode;
}
```

**Order:**
```java
@Document(collection = "orders")
public class Order {
    @Id private String id;
    private String orderNumber;
    private String customerEmail;
    private List<OrderLineItem> lineItems;
    private String status;
    private LocalDateTime orderDate;
}
```

**Inventory:**
```java
@Document(collection = "inventory")
public class Inventory {
    @Id private String id;
    private String skuCode;
    private Integer quantity;
}
```

**Customer:**
```java
@Document(collection = "customers")
public class Customer {
    @Id private String id;
    private String firstName;
    private String lastName;
    private String email;
    private String phoneNumber;
    private Address address;
}
```

**Payment:**
```java
@Document(collection = "payments")
public class Payment {
    @Id private String id;
    private String orderId;
    private String transactionId;
    private BigDecimal amount;
    private String paymentStatus;
    private LocalDateTime paymentDate;
}
```

---

### 9.3 MongoDB Verification

**Using MongoDB Compass:**
1. Connect to `mongodb://localhost:27017`
2. After running services and creating data, verify the following databases exist:
   - product-db
   - order-db
   - inventory-db
   - customer-db
   - payment_db

**Using MongoDB Shell (mongosh):**
```bash
mongosh
show dbs
use product-db
show collections
db.products.find()
```

---

## 10. Deployment Process

### 10.1 Local Development Deployment

**Prerequisites:**
1. Infrastructure running (MongoDB, Kafka, Zookeeper, Zipkin)

**Startup Order:**
```bash
# 1. Start infrastructure
cd edureka-ecom-documents/docker_files
docker-compose up -d

# 2. Start Discovery Server
cd git_src/edureka-ecom-discovery-server
mvn spring-boot:run

# 3. Wait for Eureka to fully start (10-20 seconds)

# 4. Start API Gateway
cd ../edureka-ecom-api-gateway
mvn spring-boot:run

# 5. Start Business Services (any order)
cd ../edureka-ecom-product-service
mvn spring-boot:run

cd ../edureka-ecom-order-service
mvn spring-boot:run

cd ../edureka-ecom-inventory-service
mvn spring-boot:run

cd ../edureka-ecom-customer-service
mvn spring-boot:run

cd ../edureka-ecom-payment-service
mvn spring-boot:run
```

**Verification:**
1. Check Eureka Dashboard: http://localhost:5001
2. Verify all 6 services are registered
3. Check Zipkin: http://localhost:9411

---

### 10.2 Containerization (Docker)

**Dockerfile Pattern (All Services):**
```dockerfile
FROM eclipse-temurin:17-jdk
LABEL authors="vishal"
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE {port}
RUN apt-get update && apt-get install -y gcc curl
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build Steps for Each Service:**
```bash
# 1. Build JAR
cd git_src/edureka-ecom-{service-name}
mvn clean package -DskipTests

# 2. Build Docker image
docker build -t {your-dockerhub-username}/{service-name}:latest .

# 3. Push to Docker Hub
docker push {your-dockerhub-username}/{service-name}:latest
```

---

### 10.3 Kubernetes Deployment

**Prerequisites:**
- Kubernetes cluster (Docker Desktop)
- kubectl configured
- MongoDB, Kafka, Zookeeper, Zipkin deployed in cluster

**Kubernetes Manifest Pattern:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {service-name}-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {service-name}
  template:
    metadata:
      labels:
        app: {service-name}
    spec:
      containers:
        - name: {service-name}
          image: {dockerhub-user}/{service-name}:latest
          imagePullPolicy: Always
          ports:
            - containerPort: {port}
          env:
            - name: SPRING_DATA_MONGODB_URI
              value: "mongodb://mongodb:27017/{db-name}"
            - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
              value: "http://discovery-service:5001/eureka/"
            - name: SPRING_KAFKA_BOOTSTRAP_SERVERS
              value: "kafka:9092"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: {port}
            initialDelaySeconds: 60
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: {port}
            initialDelaySeconds: 40
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: {service-name}
spec:
  selector:
    app: {service-name}
  type: ClusterIP  # LoadBalancer for gateway only
  ports:
    - protocol: TCP
      port: {port}
      targetPort: {port}
```

**Deploy All Services:**
```bash
# Deploy infrastructure first
kubectl apply -f git_src/edureka-ecom-discovery-server/k8s/discovery-server.yaml
kubectl apply -f git_src/edureka-ecom-api-gateway/k8s/api-gateway.yaml

# Deploy business services
kubectl apply -f git_src/edureka-ecom-product-service/k8s/product-service.yaml
kubectl apply -f git_src/edureka-ecom-order-service/k8s/order-service.yaml
kubectl apply -f git_src/edureka-ecom-inventory-service/k8s/inventory-service.yaml
kubectl apply -f git_src/edureka-ecom-customer-service/k8s/customer-service.yaml
kubectl apply -f git_src/edureka-ecom-payment-service/k8s/payment-service.yaml
```

**Verify Deployment:**
```bash
# Check pods
kubectl get pods

# Check services
kubectl get services

# Get logs if pod fails
kubectl logs {pod-name}

# Describe pod for events
kubectl describe pod {pod-name}
```

**Access Services:**
```bash
# Get external IP for LoadBalancer services
kubectl get service api-gateway-service
```

---

## 11. Service Connectivity & Testing

### 11.1 Health Check Verification

**Check All Services:**
```bash
curl http://localhost:5001/actuator/health  # Discovery
curl http://localhost:5002/actuator/health  # Gateway
curl http://localhost:5003/actuator/health  # Product
curl http://localhost:5004/actuator/health  # Order
curl http://localhost:5005/actuator/health  # Inventory
curl http://localhost:5006/actuator/health  # Customer
curl http://localhost:5007/actuator/health  # Payment
```

---

### 11.2 Service Registry Verification

**Check Eureka Dashboard:**
```
http://localhost:5001
```

---

### 11.3 API Testing

**Create Product:**
```bash
curl -X POST http://localhost:5002/product-service/api/products \
  -H "Content-Type: application/json" \
  -d '{...}'
```

**Get All Products:**
```bash
curl http://localhost:5002/product-service/api/products
```

**Create Customer:**
```bash
curl -X POST http://localhost:5002/customer-service/api/customers \
  -H "Content-Type: application/json" \
  -d '{...}'
```

**Create Inventory:**
```bash
curl -X POST http://localhost:5002/inventory-service/api/inventory \
  -H "Content-Type: application/json" \
  -d '{...}'
```

**Place Order (Tests Entire Flow):**
```bash
curl -X POST http://localhost:5002/order-service/api/orders \
  -H "Content-Type: application/json" \
  -d '{...}'
```

**Verify Event Processing:**
Check Order Service, Inventory Service, and Payment Service logs for event processing confirmation.

---

### 11.4 Distributed Tracing Verification

1. Execute any API request via Gateway
2. Open Zipkin: http://localhost:9411/zipkin/
3. Click "Run Query"
4. Click on a trace to see:
   - Total duration
   - Time spent in each service
   - Inter-service calls
   - Any errors

---

### 11.5 Resilience Testing

**Test Circuit Breaker:**
1. Stop Product Service
2. Try placing order
3. Check Order Service actuator: `curl http://localhost:5004/actuator/health`
4. Verify circuit breaker state

**Test Retry:**
1. Simulate network issue or slow service
2. Check logs for retry attempts with exponential backoff
3. Verify fallback is triggered after max attempts

**Test Timeout:**
1. Simulate slow downstream service
2. Verify timeout exception after configured duration
3. Verify fallback is triggered

---

## 12. Troubleshooting Guide

### 12.1 Infrastructure Debugging Commands

**Docker Infrastructure:**
```bash
docker ps
docker logs {container-name}
docker restart {container-name}
```

**Kubernetes Deployment:**
```bash
kubectl get pods
kubectl logs {pod-name}
kubectl describe pod {pod-name}
```

**Service Health:**
```bash
curl http://localhost:{port}/actuator/health
```

**Service Registry:**
```bash
# Check Eureka Dashboard
http://localhost:5001
```

---

## 13. Service Structure & Patterns

### 13.1 Standard Service Structure

All business services follow a consistent layered architecture:

```
src/main/java/com/edureka/{service-name}/
├── {ServiceName}Application.java      # Main class with @SpringBootApplication
├── config/                             # Configuration classes
│   └── FeignConfig.java               # Feign configuration (if applicable)
├── model/                              # Domain entities
│   ├── Product.java                   # MongoDB entity (@Document)
│   └── Address.java                   # Embedded document
├── dto/                                # Data Transfer Objects
│   ├── ProductRequest.java
│   └── ProductResponse.java
├── repository/                         # Data access layer
│   └── ProductRepository.java         # MongoDB repository
├── service/                            # Business logic
│   ├── ProductService.java            # Interface
│   └── ProductServiceImpl.java        # Implementation
├── controller/                         # REST endpoints
│   └── ProductController.java         # @RestController
├── client/                             # Feign clients (order-service only)
│   ├── ProductClient.java
│   ├── CustomerClient.java
│   └── InventoryClient.java
├── event/                              # Event DTOs (Kafka)
│   └── OrderPlacedEvent.java
└── listener/                           # Kafka consumers
    └── OrderListener.java

src/main/resources/
├── application.properties              # Configuration
└── application-{profile}.properties    # Profile-specific config (optional)
```

---

### 13.2 Design Patterns Implemented

| Pattern | Implementation | Services |
|---------|----------------|----------|
| **Service Registry** | Eureka Server | discovery-server |
| **API Gateway** | Spring Cloud Gateway | api-gateway |
| **Database per Service** | Separate MongoDB | All business services |
| **Circuit Breaker** | Resilience4j | order-service |
| **Retry** | Resilience4j | order-service |
| **Timeout** | Resilience4j | order-service |
| **Client-Side Load Balancing** | Spring Cloud LoadBalancer | api-gateway, order-service |
| **Event-Driven** | Apache Kafka pub/sub | order, inventory, payment |
| **Distributed Tracing** | Zipkin + Micrometer | All services |
| **Health Checks** | Spring Boot Actuator | All services |
| **API Documentation** | Swagger/OpenAPI | All business services |
| **Repository Pattern** | Spring Data MongoDB | All business services |
| **Layered Architecture** | Controller→Service→Repository | All business services |

---

### 13.3 Dependency Management

**Parent POM Configuration:**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.10</version>
</parent>

<properties>
    <java.version>17</java.version>
    <spring-cloud.version>2025.0.1</spring-cloud.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**Common Dependencies Across Services:**
- spring-boot-starter-web (REST)
- spring-cloud-starter-netflix-eureka-client (Discovery)
- micrometer-tracing-bridge-brave (Tracing)
- zipkin-reporter-brave (Zipkin)
- spring-boot-starter-actuator (Monitoring)
- lombok (Code generation)
- springdoc-openapi-starter-webmvc-ui (Swagger)

**Service-Specific Dependencies:**
- **Product/Customer Services:** spring-boot-starter-data-mongodb
- **Order Service:** +spring-cloud-starter-openfeign, spring-kafka, resilience4j
- **Inventory/Payment Services:** +spring-kafka, spring-boot-starter-data-mongodb

---

## 14. Appendix

### 14.1 Port Reference

| Port | Service | Protocol |
|------|---------|----------|
| 2181 | Zookeeper | TCP |
| 5001 | Discovery Server | HTTP |
| 5002 | API Gateway | HTTP |
| 5003 | Product Service | HTTP |
| 5004 | Order Service | HTTP |
| 5005 | Inventory Service | HTTP |
| 5006 | Customer Service | HTTP |
| 5007 | Payment Service | HTTP |
| 9092 | Kafka | TCP |
| 9411 | Zipkin | HTTP |
| 27017 | MongoDB | TCP |

---

### 14.2 Important URLs

**Infrastructure:**
- Eureka Dashboard: http://localhost:5001
- Zipkin UI: http://localhost:9411/zipkin/

**API Gateway:**
- Gateway Health: http://localhost:5002/actuator/health
- Aggregated Swagger: http://localhost:5002/swagger-ui.html

**Business Service Health:**
- http://localhost:5003/actuator/health (Product)
- http://localhost:5004/actuator/health (Order)
- http://localhost:5005/actuator/health (Inventory)
- http://localhost:5006/actuator/health (Customer)
- http://localhost:5007/actuator/health (Payment)

**Individual Swagger UIs:**
- http://localhost:5003/swagger-ui.html (Product)
- http://localhost:5004/swagger-ui.html (Order)
- http://localhost:5005/swagger-ui.html (Inventory)
- http://localhost:5006/swagger-ui.html (Customer)
- http://localhost:5007/swagger-ui.html (Payment)

---

### 14.3 Kafka Topics

| Topic | Producer | Consumers | Purpose |
|-------|----------|-----------|---------|
| orderPlacedTopic | order-service | inventory-service (inventory-group), payment-service (payment-group) | Notify services when order is placed |

---

### 14.4 Maven Commands

**Build Service:**
```bash
mvn clean package
mvn clean package -DskipTests  # Skip tests
```

**Run Service:**
```bash
mvn spring-boot:run
```

**Update Dependencies:**
```bash
mvn versions:display-dependency-updates
```

---

### 14.5 Docker Commands

**Build Image:**
```bash
docker build -t {image-name}:{tag} .
```

**Run Container:**
```bash
docker run -p {host-port}:{container-port} {image-name}:{tag}
```

**List Images:**
```bash
docker images
```

**List Containers:**
```bash
docker ps -a
```

**Remove Container:**
```bash
docker rm {container-id}
```

**Remove Image:**
```bash
docker rmi {image-name}:{tag}
```

**Push to Docker Hub:**
```bash
docker login
docker push {dockerhub-username}/{image-name}:{tag}
```

---

### 14.6 Kubernetes Commands

**Apply Manifest:**
```bash
kubectl apply -f {manifest-file.yaml}
```

**Get Resources:**
```bash
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get all
```

**Describe Resource:**
```bash
kubectl describe pod {pod-name}
kubectl describe service {service-name}
```

**View Logs:**
```bash
kubectl logs {pod-name}
kubectl logs -f {pod-name}  # Follow logs
kubectl logs {pod-name} --previous  # Previous container logs
```

**Delete Resource:**
```bash
kubectl delete pod {pod-name}
kubectl delete -f {manifest-file.yaml}
```

**Port Forward:**
```bash
kubectl port-forward pod/{pod-name} {local-port}:{pod-port}
kubectl port-forward service/{service-name} {local-port}:{service-port}
```

**Execute Command in Pod:**
```bash
kubectl exec -it {pod-name} -- /bin/sh
```

**Scale Deployment:**
```bash
kubectl scale deployment {deployment-name} --replicas=3
```

---

### 14.7 Glossary

| Term | Definition |
|------|------------|
| **Circuit Breaker** | Pattern to prevent cascading failures by failing fast when downstream service is unhealthy |
| **Discovery Server** | Service registry (Eureka) that enables dynamic service discovery |
| **Event-Driven Architecture** | Architecture pattern where services communicate via events (Kafka) |
| **Feign Client** | Declarative REST client for synchronous service-to-service communication |
| **Health Check** | Endpoint to verify service health and readiness |
| **Kafka** | Distributed event streaming platform for asynchronous messaging |
| **Load Balancer** | Distributes requests across multiple instances of a service |
| **Microservice** | Independently deployable service focused on a single business capability |
| **Resilience** | Ability to handle failures gracefully and recover |
| **Service Mesh** | Infrastructure layer for service-to-service communication (not implemented) |
| **Tracing** | Tracking requests across multiple services for debugging |
| **Zipkin** | Distributed tracing system |

---

## Conclusion

This E-Commerce Microservices Platform demonstrates a production-ready implementation of modern microservices architecture using Spring Boot 3 and Spring Cloud. The system includes:

✅ **Complete Service Ecosystem** - 7 microservices (2 infrastructure + 5 business)  
✅ **Robust Communication** - Synchronous (REST/Feign) and Asynchronous (Kafka)  
✅ **Enterprise Resilience** - Circuit Breaker, Retry, Timeout, Fallback  
✅ **Full Observability** - Distributed Tracing, Health Checks, Metrics, API Documentation  
✅ **Production Deployment** - Dockerfiles and Kubernetes manifests for all services  
✅ **Database Isolation** - Database per service pattern with MongoDB  
✅ **Scalability** - Load balancing, service discovery, horizontal scaling support  

---

