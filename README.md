# Scalable E-Commerce Platform - Microservices Architecture

Scalable e-commerce microservices architecture with API Gateway, 9 decoupled services, Kafka event-driven Saga for checkout, idempotency keys & atomic inventory reservations to prevent overselling & duplicate payments. Features CDN, Redis caching, PCI-DSS payments. Stack: Java 21, Spring Boot, PostgreSQL, Redis, OpenSearch, Docker, Kubernetes.

## 🏗️ Architecture Overview

Client (Web/Mobile) -> CDN/WAF/Load Balancer -> API Gateway -> Microservices (Identity, Catalog, Cart, Order, Payment, Inventory, Notification, Search) -> Kafka Event Bus

**Design Principles:**
- Single Responsibility per Service
- Database per Service (No Shared DB)
- Stateless & Horizontally Scalable
- Event-Driven Async Workflows
- Resilience: Timeouts, Retries, Circuit Breaker, Saga

## 🧩 Microservices

| Service | Store | Responsibility |
|---|---|---|
| API Gateway | - | Auth, Rate-limit, Routing |
| Identity | PostgreSQL | Auth, Users, Roles (Argon2id) |
| Catalog | PostgreSQL + Redis + OpenSearch | Products, Search |
| Cart | Redis | Cart with TTL |
| Order | PostgreSQL | Order lifecycle, Saga |
| Payment | PostgreSQL | Payment intents, PCI-DSS |
| Inventory | PostgreSQL | Atomic Stock Reserve/Commit |
| Notification | PG + Kafka | Email/SMS/Push |
| Search | OpenSearch | Derived search index |

## 🔄 Checkout Flow (Critical Path)

1. Client sends `POST /orders` with idempotency key
2. Order Service creates PENDING order
3. Inventory Service atomically reserves stock
4. Payment Service creates intent with external provider
5. On `PaymentSucceeded` -> Order CONFIRMED -> Inventory Commit -> Notification
6. On failure -> Saga compensates: Cancel Order + Release Inventory
7. Reconciliation job handles missed webhooks

Ensures: No duplicate charges, No overselling

## 🛠️ Tech Stack

- **Language:** Java 21 + Spring Boot
- **Gateway:** Spring Cloud Gateway / Kong
- **Databases:** PostgreSQL, Redis, OpenSearch
- **Messaging:** Apache Kafka (Outbox Pattern + Idempotent Consumers)
- **Infra:** Docker, Kubernetes, CDN, Object Storage
- **Observability:** Prometheus + Grafana, OpenTelemetry
- **Security:** OAuth2/OIDC, RBAC, mTLS, PCI-DSS Tokenization

## 🔐 Security & Reliability

- TLS 1.2+, OIDC short-lived tokens, MFA for admin
- WAF, Rate limiting, Schema validation
- Health checks, Bulkheads, DLQ, Graceful degradation
- SLOs: Catalog 99.9%, Checkout p95 <500ms, Cart p95 <200ms

## 🚀 Getting Started

```bash
docker-compose up -d
# Services will be available via API Gateway at :8080
