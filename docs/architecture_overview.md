# Architecture Overview

## System Context
Ghostfolio is an Nx monorepo with a NestJS API and Angular client. PostgreSQL and Redis provide persistence and caching. Background tasks run via Bull queues. The system follows a hexagonal architecture separating domain logic from infrastructure concerns.

## Domain Model
The full Prisma schema lives at [../prisma/schema.prisma](../prisma/schema.prisma). It defines entities such as `User`, `Account`, `Order` and `SymbolProfile`. Key relations:
- `User` owns many `Account` and `Order` records.
- `Account` aggregates `Order` transactions and balance snapshots.
- `SymbolProfile` represents tradable symbols watched by users.
- `Order` links a `User` to a `SymbolProfile` through an `Account`.

Invariants:
- Every `Order` references an existing `SymbolProfile`.
- Deleting a `User` cascades to their accounts and orders.
- Currency and data source enums restrict valid values.

```mermaid
classDiagram
  User <|-- Account
  Account <|-- Order
  User <|-- SymbolProfile
  SymbolProfile <|-- Order
```

## Critical Execution Flows

### 1. User Authentication
```mermaid
sequenceDiagram
  participant U as User
  participant C as Client
  participant A as API
  participant DB as DB
  U->>C: submit credentials
  C->>A: POST /api/v1/auth/login
  A->>DB: verify user
  DB-->>A: user record
  A-->>C: JWT token
  C->>U: session established
```

### 2. Order Placement and Settlement
```mermaid
sequenceDiagram
  participant U as User
  participant C as Client
  participant A as API
  participant Q as Queue
  participant DB as DB
  U->>C: create order
  C->>A: POST /api/v1/orders
  A->>DB: persist order
  A->>Q: enqueue settlement job
  Q->>DB: update balances
  DB-->>Q: confirmation
  Q-->>A: job complete
  A-->>C: order settled
```

### 3. Background Price Update
```mermaid
sequenceDiagram
  participant Cron as Scheduler
  participant Q as Queue
  participant A as API
  participant Provider as DataProvider
  participant DB as DB
  Cron->>Q: enqueue price-update
  Q->>Provider: fetch prices
  Provider-->>Q: price data
  Q->>DB: store prices
  DB-->>Q: ok
  Q-->>A: prices updated
```

## Deployment Topology
```mermaid
graph TD
  Client[Angular SPA]
  API[NestJS API]
  DB[(PostgreSQL)]
  Cache[(Redis)]
  Queue[(Bull)]
  Client -- HTTP --> API
  API -- ORM --> DB
  API -- Cache --> Cache
  API -- Jobs --> Queue
```

Last verified on 2025-06-22 @ commit 2634a4fd.
