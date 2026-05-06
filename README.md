# spring-cloud-config-repo

This repository is the **Git backend for Spring Cloud Config Server**. It contains only YAML files — no Java code. The Config Server clones this repo on startup and serves its content to all microservices over HTTP.

### Structure

```
spring-cloud-config-repo/
├── application.yml           ← shared config for all services (Eureka URL, etc.)
├── product-service.yml       ← overrides for product-service (port 8081, H2 datasource)
├── inventory-service.yml     ← overrides for inventory-service (port 8082)
├── order-service.yml         ← overrides for order-service (port 8083)
└── api-gateway.yml           ← overrides for api-gateway (port 8080, routing rules)
```

Config Server resolves files by `{application-name}.yml`. If a service is named `product-service`, it receives both `application.yml` (shared base) and `product-service.yml` (specific overrides) merged together.

---

## Project Overview

A learning project for exploring Spring Cloud through a minimal microservices-based e-commerce system.

---

## Goal

Build a distributed system covering the core Spring Cloud components: centralized configuration, service discovery, inter-service communication, API gateway, and resilience.

---

## Architecture

```
[HTTP Client]
      |
[API Gateway :8080]
      |
      ├──→ [Product Service :8081]
      ├──→ [Inventory Service :8082]
      └──→ [Order Service :8083]
                  |
                  ├──→ [Product Service]   (via OpenFeign)
                  └──→ [Inventory Service] (via OpenFeign)

All services register on  → [Eureka Discovery Server :8761]
All services read config  → [Config Server :8888]
Config Server reads from  → [this config-repo on GitHub]
```

---

## Repository Structure (Polyrepo)

7 repositories, to be created in this order:

| # | Repository | Purpose |
|---|---|---|
| 1 | `spring-cloud-config-repo` | YAML configuration files, read by Config Server via Git |
| 2 | `spring-cloud-config-server` | Reads the config-repo and exposes it to all services |
| 3 | `spring-cloud-discovery-server` | Eureka — service registry |
| 4 | `spring-cloud-api-gateway` | Single entry point, routes requests to microservices |
| 5 | `spring-cloud-product-service` | Product CRUD |
| 6 | `spring-cloud-inventory-service` | Inventory CRUD |
| 7 | `spring-cloud-order-service` | Order orchestration, calls product and inventory |

---

## Development Order

Each step depends on the previous one:

1. **config-repo** — create the Git repo with YAML configuration files
2. **config-server** — points to config-repo, exposes it via HTTP
3. **discovery-server** — Eureka, other services register here
4. **product-service** — simple CRUD, pure REST
5. **inventory-service** — same, simple CRUD
6. **order-service** — calls product and inventory via OpenFeign
7. **api-gateway** — routes to all services via Eureka
8. *(optional)* **Resilience4j on order-service** — circuit breaker if inventory is down

---

## Startup Order (runtime)

```
1. config-server      :8888
2. discovery-server   :8761
3. product-service    :8081
4. inventory-service  :8082
5. order-service      :8083
6. api-gateway        :8080
```

---

## Tech Stack

| Technology | Version |
|---|---|
| Java | 21 |
| Spring Boot | 3.3.5 |
| Spring Cloud | 2023.0.3 |

---

## Config Repo Structure

This repository contains only YAML files — no Java code:

```
spring-cloud-config-repo/
├── application.yml           ← shared config for all services
├── product-service.yml
├── inventory-service.yml
├── order-service.yml
└── api-gateway.yml
```

`application.yml` defines shared settings (e.g. Eureka URL). Each service-specific file overrides or extends it (e.g. server port, datasource).

---

## API Endpoints

### Product Service
| Method | Path | Description |
|---|---|---|
| GET | `/api/products` | List all products |
| GET | `/api/products/{id}` | Get product by ID |
| POST | `/api/products` | Create a product |

### Inventory Service
| Method | Path | Description |
|---|---|---|
| GET | `/api/inventory/{productId}` | Get stock for a product |
| PUT | `/api/inventory/{productId}/decrease` | Decrease stock |

### Order Service
| Method | Path | Description |
|---|---|---|
| POST | `/api/orders` | Place an order (calls product + inventory internally) |

---

## Spring Cloud Concepts Covered

| Concept | Component | Repository |
|---|---|---|
| Centralized configuration | Spring Cloud Config | config-server + config-repo |
| Service discovery | Eureka | discovery-server |
| Client-side load balancing | Spring Cloud LoadBalancer | built into Feign and Gateway |
| Inter-service communication | OpenFeign | order-service |
| API Gateway / routing | Spring Cloud Gateway | api-gateway |
| Circuit Breaker | Resilience4j | order-service (optional) |
