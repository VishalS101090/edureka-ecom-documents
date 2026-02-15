# MSA Training – Project Ports Reference

This document lists all ports used across projects in the MSA_training workspace.

---

## Application services (server.port)

| Project | Port | Description |
|---------|------|-------------|
| **discovery-server** | 5001 | Eureka discovery server |
| **api-gateway** | 5002 | Spring Cloud API Gateway |
| **product-service** | 5003 | Product microservice |
| **order-service** | 5004 | Order microservice |
| **inventory-service** | 5005 | Inventory microservice |
| **customer-service** | 5006 | Customer microservice |
| **payment-service** | 5007 | Payment microservice |

**Source:** `application.properties` / `application-stag.properties` in each project.

---

## Infrastructure (Docker Compose)

### Documents/docker-compose.yml & docker_files/docker-compose.yml

| Service | Port | Description |
|---------|------|-------------|
| **Zookeeper** | 2181 | Kafka coordination (ZOOKEEPER_CLIENT_PORT) |
| **Kafka** | 9092 | Kafka broker |
| **Mongo** | 27017 | MongoDB (Documents/docker-compose.yml only) |

### projects/product-service/docker-compose.yml

| Service | Port | Description |
|---------|------|-------------|
| **Mongo (product-mongo)** | 27017 | MongoDB for product-service |

---

## Referenced ports (dependencies)

Ports that applications connect to (not necessarily defined in this repo):

| Port | Used by | Purpose |
|------|---------|---------|
| 27017 | product-service, order-service | MongoDB connection |
| 9092 | order-service, inventory-service | Kafka bootstrap (spring.kafka.bootstrap-servers) |
| 5001 | api-gateway, product-service, order-service, inventory-service, customer-service, payment-service | Eureka (discovery-server) |

---

## Quick reference – port summary

| Port | Component |
|------|-----------|
| 2181 | Zookeeper |
| 5001 | Discovery Server (Eureka) |
| 5002 | API Gateway |
| 5003 | Product Service |
| 5004 | Order Service |
| 5005 | Inventory Service |
| 5006 | Customer Service |
| 5007 | Payment Service |
| 9092 | Kafka |
| 27017 | MongoDB |

---
*Generated from workspace scan. Update this document when adding or changing services or ports.*
