# Microservices Project: Infrastructure Setup Guide

**Project:** E-Commerce Microservices Application  
**Goal:** Set up the foundational "plumbing" services required for microservice communication and routing.  
**Tech Stack:** Spring Boot 3.x, Spring Cloud, Java 17, Maven.

---

## Prerequisites
Before starting, ensure you have the following installed:
* **Java JDK 17** (or 11)
* **Apache Maven**
* **IntelliJ IDEA** (Community or Ultimate)
* **Docker Desktop** (Optional for this specific phase, but needed later)

---

## Part 1: Service Registry (Eureka Server)

**Role:** Acts as the "Phonebook" or DNS for your microservices. Services register here so they can find each other without hardcoded IP addresses.

### Step 1: Generate Project
1.  Go to **[start.spring.io](https://start.spring.io/)**.
2.  **Project:** Maven
3.  **Language:** Java
4.  **Spring Boot:** 3.2.x (Latest Stable)
5.  **Group:** `com.edureka`
6.  **Artifact:** `discovery-server`
7.  **Packaging:** Jar
8.  **Java:** 17
9.  **Dependencies:** Search for and add **Eureka Server**.
10. **Generate**, unzip, and open in IntelliJ.

### Step 2: Main Class Configuration
Open `src/main/java/com/edureka/discoveryserver/DiscoveryServerApplication.java` and add the `@EnableEurekaServer` annotation.

```java
package com.edureka.discoveryserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer; // 1. Import this

@SpringBootApplication
@EnableEurekaServer // 2. Add this annotation to enable the registry
public class DiscoveryServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

### Step 3: Properties Configuration
Rename `application.properties` (if it exists) or create it.

File: `src/main/resources/application.properties`

```properties
# Server Port (Standard for Eureka)
server.port=5001

# Hostname
eureka.instance.hostname=localhost

# Registry Configuration
# false: This is the server, so it doesn't need to register with itself
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

# Default Zone (The URL clients will use to connect)
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

# Logging (Optional: Reduces noise in console)
logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF
```
### Step 4: Run & Verify
Right-click `DiscoveryServerApplication.java` and select Run.

Wait for "Started DiscoveryServerApplication" in the console.

Open Browser: http://localhost:5001

Success: You should see the "Spring Eureka System Status" page.

---
---

## Part 2: API Gateway (Spring Cloud Gateway)
**Role:** The single entry point for all client requests. It routes traffic to the appropriate microservice based on the URL.

### Step 1: Generate Project
1. Go to **[start.spring.io](https://start.spring.io/)**.
2. **Artifact:** `api-gateway`
3. **Dependencies:**
    - Gateway (Spring Cloud Routing) — Important: Do NOT add "Spring Web"
    - Eureka Discovery Client (Spring Cloud Discovery)
4. Generate, unzip, and open in a new window in IntelliJ.

### Step 2: Main Class
Ensure the main class looks like this (standard Spring Boot setup).

```java
package com.edureka.apigateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```
### Step 3: Properties Configuration
We enable Discovery Locator to automatically create routes for registered services.
File: `src/main/resources/application.properties`

```properties
# ==============================================================
# API GATEWAY CONFIGURATION
# ==============================================================

# Server Port (Entry point for all clients)
server.port=5002

# Application Name
spring.application.name=api-gateway

# ==============================================================
# EUREKA CLIENT CONFIGURATION
# ==============================================================
# Tell Gateway where the Phonebook (Eureka) is
eureka.client.service-url.defaultZone=http://localhost:5001/eureka/

# ==============================================================
# ROUTING CONFIGURATION (The "Magic" Part)
# ==============================================================
# 1. Enable automatic routing based on Service ID registered in Eureka
spring.cloud.gateway.discovery.locator.enabled=true

# 2. Allow lowercase access (e.g., /product-service/.. instead of /PRODUCT-SERVICE/..)
spring.cloud.gateway.discovery.locator.lower-case-service-id=true
```

### Step 4: Run & Verify
Ensure Eureka Server is already running.

1. Run `ApiGatewayApplication` (right‑click the main class and Run, or use `mvn spring-boot:run`).
2. Check logs for DiscoveryClient ... completed handshake.
3. Refresh http://localhost:5001 — you should see `API-GATEWAY` listed in the "Instances currently registered" table.

## Part 3: Execution Order (Important!)
Whenever you restart your system, start the applications in this order to ensure connections are established correctly:

1. Discovery Service (Eureka)
    - Wait until it is fully started (approx. 10–20 seconds).
2. API Gateway
    - Wait for it to register with Eureka.
3. Business Microservices
    - Product Service, Order Service, etc. (to be built in the next phase).

---

### External Infrastructure (Docker)

Before running any Java application, you must start the database and messaging system.

### 1. Create `docker-compose.yml`
A ready-to-use `docker-compose.yml` has been added to the repository root. It starts Zookeeper, Kafka and MongoDB which are required by the business services.

See [docker-compose.yml](../docker_files/docker-compose.yml) for the full configuration and run instructions.

### 2. Start Services
Run command: `docker-compose up -d`

### 3. Verify
Run command: `docker ps`
Ensure MongoDB (27017), Kafka (9092), and Zookeeper (2181) are UP.




