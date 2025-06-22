# Module Guide

## Overview
The repository contains multiple Nx applications and libraries. `apps/api` implements the NestJS backend while `apps/client` hosts the Angular UI. Shared logic lives in `libs/common` and `libs/ui`.

## Backend Modules
- **access** – manage portfolio sharing
- **account** – account CRUD and balances
- **asset** – asset profiles and watch lists
- **order** – order placement and tagging
- **portfolio** – computes metrics and snapshots
- **user** – registration and profile management
- **admin** – maintenance endpoints and queues

Each module exposes controllers under `/api/v1/`.

## Frontend Structure
Angular features are grouped under `apps/client/src/app`. Reusable UI widgets reside in `libs/ui`. Modules communicate via services and RxJS stores.

## API Surface
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

## Extensibility Example: Asset Analysis Job
1. Create `analysis.processor.ts` under `apps/api/src/services/queues/asset-analysis/` implementing `@Processor` with Bull.
2. Register the queue in `common/config.ts` and add a module similar to `data-gathering.module.ts`.
3. Implement `AssetAnalysisService` performing computations and storing results.
4. Schedule jobs via `DataGatheringService.addJobToQueue()`.
5. Expose results through a new controller under `asset-analysis`.

This pattern mirrors existing queue processors and keeps domain logic isolated in services.

Last verified on 2025-06-22 @ commit 2634a4fd23074adcf5add56123bd7d02863b90e9.
