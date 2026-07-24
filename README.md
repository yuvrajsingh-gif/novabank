# NovaBank — Event-Driven Digital Banking System (Microservices + SAGA)

A production-style digital banking backend built with **Spring Boot 3**, **Spring Cloud**, **Apache Kafka**, **Redis**, and **MySQL**. Money transfers are coordinated across services using the **SAGA choreography pattern** over Kafka, with **real-time fraud detection** on the event stream and **API-gateway rate limiting** at the edge.

---

## Services Overview

| Service | Port | Responsibility |
|---|---|---|
| api-gateway | 8080 | Single entry point, routing, rate limiting |
| account-service | 8081 | Account management, balances |
| transaction-service | 8082 | Money transfers, transaction history, SAGA orchestration |
| payment-service | 8083 | Payment orders & webhooks |
| fraud-detection-service | 8084 | Real-time fraud detection via Redis patterns |
| notification-service | 8085 | Transaction & fraud alerts |

---

## Architecture Flow

```
User → API Gateway (rate limiting)
             ↓
    Account / Transaction / Payment Service
             ↓
        Apache Kafka
             ↓
    ┌────────────────────────┐
    │                        │
Fraud Detection      Notification Service
(Redis patterns)     (alerts via email/SMS)
    │
Account Service
(block if fraud)
```

The transfer flow is a **SAGA**: `transaction-service` initiates a transfer, emits an event, waits for the fraud-check result, and only then completes (or compensates) — so a transfer never gets stuck half-applied even if a step fails mid-process.

---

## Kafka Topics

| Topic | Publisher | Consumer |
|---|---|---|
| transaction.initiated | Transaction Service | Fraud Detection |
| fraud.check.result | Fraud Detection | Transaction Service |
| transaction.completed | Transaction Service | Account Service, Notification |
| fraud.detected | Fraud Detection | Account Service, Notification |
| payment.completed | Payment Service | Notification |

---

## Tech Stack

- **Language / Framework:** Java 17, Spring Boot 3, Spring Cloud Gateway, Spring Data JPA
- **Messaging:** Apache Kafka (event-driven SAGA choreography)
- **Cache / Fraud state:** Redis
- **Database:** MySQL 8
- **Resilience:** API-gateway rate limiting
- **Infra:** Docker Compose (Redis, MySQL, Kafka, Zookeeper)

---

## How To Run

### Prerequisites
- JDK 17+
- Maven 3.9+
- Docker Desktop (running)

### Step 1 — Start infrastructure (Redis, MySQL, Kafka, Zookeeper)
```bash
docker-compose up -d
```

### Step 2 — Start each service (one terminal per service)
```bash
cd account-service          && mvn spring-boot:run   # Terminal 1
cd transaction-service      && mvn spring-boot:run   # Terminal 2
cd payment-service          && mvn spring-boot:run   # Terminal 3
cd fraud-detection-service  && mvn spring-boot:run   # Terminal 4
cd notification-service     && mvn spring-boot:run   # Terminal 5
cd api-gateway              && mvn spring-boot:run   # Terminal 6
```

The gateway is then reachable at `http://localhost:8080`.

---

## Concepts Demonstrated

- SAGA choreography for distributed transactions
- Event-driven microservices with Kafka topics
- Real-time, stateful fraud detection with Redis
- API-gateway rate limiting
- Inter-service communication (REST clients)
- Idempotent, resilient money movement
