# Module Guide

## Backend Modules
- **access** – permissions and portfolio sharing
- **account** – accounts and balances
- **asset** – asset profiles and watch lists
- **order** – buy/sell activities
- **portfolio** – portfolio metrics
- **user** – authentication and subscriptions
- **admin** – maintenance endpoints and queues

Each module exposes a NestJS `Module` composed of controllers, services and DTOs. Shared logic resides in [`libs/common`](../libs/common/). Each module exposes controllers under `/api/v1/`.

## Frontend Structure
Angular features are grouped under `apps/client/src/app`. Reusable UI widgets reside in `libs/ui`. Modules communicate via services and RxJS stores.

## REST API Surface
Routes are versioned under `/api/v1`.

| Feature | Sample Route | Response |
|---------|--------------|---------|
| Auth | `POST /auth/login` | `{ token: string }` |
| Orders | `POST /orders` | `{ id, status }` |
| Portfolio | `GET /portfolio/summary` | `{ value, allocation }` |

Standard errors use shape `{ statusCode, message }` with appropriate HTTP codes.
Below is an excerpt of REST routes grouped by feature:

| Feature | Routes |
|---------|-------|
| **auth** | `POST /auth/google`, `POST /auth/anonymous` |
| **account** | `GET /account`, `POST /account`, `DELETE /account/:id` |
| **order** | `GET /order`, `POST /order`, `PUT /order/:id`, `DELETE /order/:id` |
| **portfolio** | `GET /portfolio/overview`, `GET /portfolio/history` |
| **symbol** | `GET /symbol/lookup`, `GET /symbol/:source/:symbol` |
| **admin/queue** | `POST /admin/queue/enqueue`, `GET /admin/queue/stats` |

Example: create an order
```http
POST /order
Content-Type: application/json
{
  "accountId": "uuid",
  "symbol": "AAPL",
  "type": "BUY",
  "quantity": 10,
  "unitPrice": 125,
  "currency": "USD"
}
```
Returns `201` with JSON body `{ "id": "uuid", "symbol": "AAPL", ... }`. Error responses follow `{ "statusCode": 403, "message": "Forbidden" }`.

Status codes: `200` success, `201` created, `400` invalid data, `403` unauthorized, `404` not found, `500` server error.

## Decision Log
- **Nx** chosen for workspace management and affected builds.
- **NestJS** provides modular structure and TypeScript type safety.
- **Prisma** used for type-safe ORM and migrations.
- **Bull/Redis** handle background jobs and caching.
Alternatives like TypeORM and RabbitMQ were considered but rejected for maturity and simplicity reasons.


## Extensibility Example: Asset Analysis Job
1. Create `analysis.processor.ts` under `apps/api/src/services/queues/asset-analysis/` implementing `@Processor` with Bull.
2. Register the queue in `common/config.ts` and add a module similar to `data-gathering.module.ts`.
3. Implement `AssetAnalysisService` performing computations and storing results.
4. Schedule jobs via `DataGatheringService.addJobToQueue()`.
5. Expose results through a new controller under `asset-analysis`.

This pattern mirrors existing queue processors and keeps domain logic isolated in services.

Last verified on 2025-06-22 @ commit 2634a4fd.
