# Marketplace Architecture

## 1. Scope and Requirements

This project describes the architecture of an online marketplace where sellers publish products and buyers place orders.

The marketplace supports:

- personalized product feed;
- product catalog management;
- buyer and seller management;
- order creation;
- payment calculation and accounting;
- order status notifications.

This repository contains the architecture and initialization of one service. Marketplace business logic is not implemented.

---

## 2. Domains and Responsibilities

The marketplace is divided into six main domains.

| Domain | Service | Responsibility | Owned Data |
| --- | --- | --- | --- |
| Users | User Service | Buyers, sellers, profiles and roles | Users, profiles, roles |
| Catalog | Catalog Service | Products, descriptions and categories | Products, categories, product metadata |
| Recommendations | Recommendation Service | Personalized product feed | Recommendations and derived preferences |
| Orders | Order Service | Orders, items and order statuses | Orders, order items, order state |
| Payments | Payment Service | Payment calculation and payment status | Payments and accounting records |
| Notifications | Notification Service | Order and payment notifications | Notifications and delivery status |

---

## 3. Service Decomposition

Each domain is placed in a separate service because it has its own responsibility and data.

- **User Service** is responsible for buyers, sellers and their profiles.
- **Catalog Service** is responsible for products and categories.
- **Recommendation Service** is responsible for the personalized product feed.
- **Order Service** is responsible for the order lifecycle.
- **Payment Service** is responsible for payments and payment state.
- **Notification Service** is responsible for notifications and their delivery status.

This separation makes the responsibilities of each service and data ownership explicit.

---

## 4. C4 Container Architecture

The marketplace architecture is described using a C4 Container diagram.

Source:

[architecture/marketplace.c4](architecture/marketplace.c4)

Rendered diagram:

[architecture/marketplace.svg](architecture/marketplace.svg)

![Marketplace C4 Container Diagram](architecture/marketplace.svg)

The diagram contains:

- Buyer;
- Seller;
- Web Client;
- User Service;
- Catalog Service;
- Recommendation Service;
- Order Service;
- Payment Service;
- Notification Service;
- Event Broker;
- a separate data store for each service.

The diagram also shows synchronous HTTP/REST communication and asynchronous event-based communication.

---

## 5. Data Ownership

Each service owns its own data. There are no shared databases between services.

A service does not directly access another service's database. If information from another domain is required, services communicate through APIs or events.

| Service | Owned Data Store | Main Data |
| --- | --- | --- |
| User Service | User DB | Users, profiles, roles |
| Catalog Service | Catalog DB | Products, categories, metadata |
| Recommendation Service | Recommendation DB | Recommendations and preferences |
| Order Service | Order DB | Orders, items, order state |
| Payment Service | Payment DB | Payments and accounting records |
| Notification Service | Notification DB | Notifications and delivery status |

The databases shown in the architecture are conceptual and are not deployed in this project.

---

## 6. Service Communication

### Synchronous Communication

HTTP/REST is used when an immediate response is required.

| Caller | Receiver | Purpose |
| --- | --- | --- |
| Web Client | User Service | Get user information |
| Web Client | Catalog Service | Browse or manage products |
| Web Client | Recommendation Service | Get personalized feed |
| Web Client | Order Service | Create or view orders |
| Order Service | Catalog Service | Get product information |
| Order Service | Payment Service | Initiate payment processing |

### Asynchronous Communication

An Event Broker is used for operations that do not require an immediate response.

Examples:

- Payment Service publishes payment status events.
- Order Service publishes order status events.
- Notification Service receives order and payment events.
- Recommendation Service receives user and catalog activity events.

Asynchronous communication reduces direct dependencies between services.

Its main disadvantage is eventual consistency: different services may not be updated at exactly the same moment. Event-based communication can also make debugging more difficult.

The Event Broker is an architectural element only and is not deployed in this project.

---

## 7. Alternative Architecture 1: Modular Monolith

One alternative is a modular monolith.

Users, Catalog, Recommendations, Orders, Payments and Notifications would be separate modules inside one backend application.

### Advantages

- simpler deployment;
- easier local development;
- fewer network failures;
- simpler transactions.

### Disadvantages

- the whole backend is deployed together;
- individual domains are harder to scale separately;
- modules may become strongly coupled as the project grows.

---

## 8. Alternative Architecture 2: Domain-Oriented Services

The second option is to separate the domains into independent services.

This is the architecture selected for this project.

### Advantages

- clear domain boundaries;
- clear data ownership;
- services can evolve independently;
- individual services can scale separately;
- both synchronous and asynchronous communication can be used.

### Disadvantages

- higher operational complexity;
- network calls can fail;
- asynchronous communication introduces eventual consistency;
- communication between services is harder to debug.

---

## 9. Trade-offs

| Aspect | Modular Monolith | Domain-Oriented Services |
| --- | --- | --- |
| Deployment | One application | Separate services |
| Complexity | Lower | Higher |
| Scaling | Entire backend | Individual services |
| Transactions | Easier | Harder across services |
| Data ownership | Controlled inside application | Explicit per service |
| Network failures | Fewer | More possible failure points |
| Operations | Simpler | More complex |

Neither architecture is always better. The choice depends on the requirements and scale of the system.

---

## 10. Final Architecture Decision

The selected architecture is **Domain-Oriented Services**.

This option was selected because the assignment requires explicit domain boundaries, service responsibilities, data ownership and both synchronous and asynchronous interactions.

Separating the marketplace into services makes these concepts clearly visible in the architecture.

For a small real-world MVP, a modular monolith could be a simpler starting point because it has less operational complexity.

---

## 11. Implemented Service

Only **User Service** is implemented in this repository.

Technology:

- Python 3.12;
- FastAPI;
- Uvicorn.

The service contains one endpoint:

```text
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

No marketplace business logic, database, authentication or CRUD operations are implemented.

The other services, databases and Event Broker exist only in the architecture model.

---

## 12. Running the Project

Requirements:

- Docker;
- Docker Compose.

Run the project from the repository root:

```bash
docker compose up --build
```

The service is available at:

```text
http://localhost:8000
```

Check the health endpoint:

```bash
curl http://localhost:8000/health
```

On Windows PowerShell:

```powershell
curl.exe -i http://localhost:8000/health
```

Stop the project:

```bash
docker compose down
```

---

## 13. Health Check

Endpoint:

```text
GET http://localhost:8000/health
```

Expected status:

```text
HTTP 200 OK
```

Expected response:

```json
{
  "status": "ok"
}
```

The Docker image and User Service were tested locally with Docker Compose. The service started successfully and the `/health` endpoint returned HTTP 200.

---

## 14. Repository Structure

```text
.
├── .gitignore
├── README.md
├── architecture/
│   ├── marketplace.c4
│   └── marketplace.svg
├── docker-compose.yml
└── services/
    └── user-service/
        ├── .dockerignore
        ├── Dockerfile
        ├── requirements.txt
        └── app/
            └── main.py
```