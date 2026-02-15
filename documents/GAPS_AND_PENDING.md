# E-Commerce Microservices – Gaps and Pending Tasks

This document summarizes the comparison between **requirements** (from `Final_Project_Master_Guide.md`, `Business_Microservices_Setup.md`, and `Project_Ports_Reference.md`) and the **current implementation** in `git_src`, and lists completed fixes and remaining optional work.

---

## Requirement Sources

- **Final_Project_Master_Guide.md** – Main implementation guide (Eureka, Gateway, services, Swagger, execution plan).
- **Business_Microservices_Setup.md** – Product, Order, Inventory setup and Kafka flow.
- **Project_Ports_Reference.md** – Port allocation.
- **shared-events.md** – Event DTO and topic naming guidance.

**Note:** The PDF `project_- spring microservices -E-commerce application.pdf` was not found in the workspace. The above markdown documents were used as the requirement baseline.

---

## Services Implemented (git_src)

| Service              | Port | Eureka | MongoDB | Kafka        | Status |
|----------------------|------|--------|---------|--------------|--------|
| discovery-server     | 5001 | Server | -       | -            | OK     |
| api-gateway          | 5002 | Client | -       | -            | OK     |
| product-service      | 5003 | Client | product-db | -         | OK     |
| order-service        | 5004 | Client | order-db  | Producer    | OK     |
| inventory-service    | 5005 | Client | inventory-db | Consumer | OK     |
| customer-service     | 5006 | Client | customer-db | -        | OK     |
| payment-service      | 5007 | Client | payment_db | Consumer  | OK     |

---

## Gaps Fixed in This Pass

1. **Order service application name**  
   - **Was:** `spring.application.name=Order Service` (space, wrong for gateway route `/order-service/...`).  
   - **Fixed:** `spring.application.name=order-service`.  
   - **File:** `git_src/edureka-ecom-order-service/src/main/resources/application.properties`.

2. **Product service application name**  
   - **Was:** `spring.application.name=product_service` (underscore).  
   - **Fixed:** `spring.application.name=product-service`.  
   - **File:** `git_src/edureka-ecom-product-service/src/main/resources/application.properties`.

3. **Feign client for Product**  
   - **Was:** `@FeignClient(name = "product_service", ...)`.  
   - **Fixed:** `@FeignClient(name = "product-service", ...)`.  
   - **File:** `git_src/edureka-ecom-order-service/.../client/ProductClient.java`.

4. **Customer service MongoDB database name**  
   - **Was:** `customer_db`.  
   - **Fixed:** `customer-db` (aligned with Master Guide).  
   - **File:** `git_src/edureka-ecom-customer-service/src/main/resources/application.properties`.

5. **Project_Ports_Reference.md**  
   - **Was:** Missing customer-service (5006) and payment-service (5007).  
   - **Fixed:** Added both to application services table, referenced ports, and quick reference.  
   - **File:** `documents/Project_Ports_Reference.md`.

6. **API Gateway – optional Swagger aggregation (Master Guide Part 4)**  
   - **Was:** No SpringDoc or Swagger UI config at gateway.  
   - **Fixed:** Added `springdoc-openapi-starter-webmvc-ui` to gateway `pom.xml` and configured `springdoc.swagger-ui.urls` for product-service, order-service, customer-service, inventory-service, payment-service.  
   - **Files:** `git_src/edureka-ecom-api-gateway/pom.xml`, `application.properties`.

---

## Intentional Differences (No Change)

- **Kafka topic name**  
  - Docs mention `notificationTopic`; code uses `orderPlacedTopic`.  
  - **Decision:** Kept `orderPlacedTopic` (more descriptive; used in order, inventory, and payment services).

- **OrderPlacedEvent payload**  
  - Docs show minimal `orderNumber`, `email`.  
  - Implementation includes `orderId`, `skuCode`, `quantity`, `amount` for inventory deduction and payment.  
  - **Decision:** Richer event kept as-is.

- **Payment service MongoDB**  
  - Uses `payment_db` (underscore). Master Guide shows `payment-db`.  
  - **Decision:** Left as `payment_db` to avoid breaking existing data; can be aligned later if desired.

---

## Optional / Future Work

1. **Resilience (Circuit breaker / Retry)**  
   - No Resilience4j or similar in the codebase.  
   - **Suggestion:** Add `spring-cloud-starter-circuitbreaker-resilience4j` and `@CircuitBreaker`/`@Retry` on Feign clients in order-service for Product, Customer, and Inventory calls.

2. **Shared events module**  
   - `shared-events.md` recommends a shared Maven module for event DTOs.  
   - **Current:** Each service has its own `OrderPlacedEvent` (order, inventory, payment).  
   - **Suggestion:** Introduce `shared-events` (or similar) and depend on it from order-, inventory-, and payment-service to avoid drift.

3. **Centralized configuration**  
   - Not required by the current docs; all config is per-service `application.properties`.  
   - **Suggestion:** Optional Spring Cloud Config Server for shared and environment-specific settings.

4. **Actuator / health**  
   - Not explicitly required in the guides.  
   - **Suggestion:** Add `spring-boot-starter-actuator` and expose `health` (and optionally `info`) for Eureka health checks and monitoring.

5. **API Gateway – path prefix**  
   - Master Guide verification step: `POST http://localhost:5002/order-service/api/orders`.  
   - With discovery locator and lower-case-service-id, the correct path is `/order-service/api/orders` (gateway strips `/order-service` and routes to the order-service instance).  
   - **Status:** Correct as long as all services use hyphenated names (now fixed for order-service and product-service).

6. **Topic naming in docs**  
   - Optional: Update `Final_Project_Master_Guide.md` and `Business_Microservices_Setup.md` to mention `orderPlacedTopic` (or document that implementation uses `orderPlacedTopic` instead of `notificationTopic`).

---

## Verification Checklist (from Master Guide)

After starting infrastructure and all services:

1. **Eureka (http://localhost:5001)**  
   - Expect 6 application instances: API-GATEWAY, PRODUCT-SERVICE, ORDER-SERVICE, INVENTORY-SERVICE, CUSTOMER-SERVICE, PAYMENT-SERVICE (names may appear in uppercase).

2. **Place order via Gateway**  
   - `POST http://localhost:5002/order-service/api/orders`  
   - Body example: `{"skuCode":"<valid-sku>","quantity":1,"email":"<valid-customer-email>"}`  
   - Prerequisites: Product with that `skuCode`, Customer with that `email`, Inventory with sufficient stock.

3. **Logs**  
   - Order service: order placed and event published.  
   - Inventory service: stock deduction for order.  
   - Payment service: payment processed for order.

4. **Swagger**  
   - Per-service: e.g. http://localhost:5003/swagger-ui.html (product), http://localhost:5004/swagger-ui.html (order).  
   - Gateway aggregation: http://localhost:5002/swagger-ui.html (when all services are up).

---

*Last updated from codebase and document review. Update this file when closing further gaps or adding new services.*
