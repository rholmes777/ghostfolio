# Operations Guide

## Configuration
Environment variables are loaded via the `ConfigurationService`. Required keys are shown below (defaults in brackets):

- `ACCESS_TOKEN_SALT`
- `DATABASE_URL`
- `JWT_SECRET_KEY`
- `REDIS_HOST` (`localhost`)
- `REDIS_PORT` (`6379`)
- `REDIS_PASSWORD` (`''`)
- `POSTGRES_DB` (`ghostfolio-db`)
- `POSTGRES_USER` (`user`)
- `POSTGRES_PASSWORD`
- `ROOT_URL` (`http://localhost:3333`)
- OAuth keys: `GOOGLE_CLIENT_ID`, `GOOGLE_SECRET`

Local development uses `.env.dev` and Docker compose files in `docker/`. Override ports via environment variables.

## Build & Deploy
Primary Nx targets:
- `npm run build:production` → builds both `api` and `client` for container images.
- `npm run lint` → runs ESLint across all projects.
- `npm run test` → executes unit tests (Jest).
- `nx affected` commands support selective builds in CI.

Database migrations use Prisma Migrate. On deploy:
```bash
npm run database:migrate
npm run database:seed
```
Release images are tagged with the package.json version (e.g. `ghostfolio/ghostfolio:2.171.0`).

## Decision Log
- **Nx** – manages multiple apps and enforces boundaries.
- **NestJS** – provides modular architecture and DI.
- **Prisma** – type-safe database layer.
- **Bull + Redis** – reliable job queues for data gathering.
Alternatives like TypeORM or Kafka were considered but rejected for higher complexity.

## Security & Compliance
Authentication uses JWT tokens stored in the browser. RBAC is enforced through `HasPermissionGuard`. All traffic is served over HTTPS; secrets such as `JWT_SECRET_KEY` are never committed to Git. PostgreSQL and Redis are expected to run within trusted networks.

## Testing Strategy
- **Unit tests** with Jest cover services and utilities.
- **Integration tests** validate Prisma interactions.
- **E2E tests** run with Cypress against the built Angular app.
Coverage reports run in CI via `nx test --configuration=ci`.

## Extensibility
To add jobs or features, create new Nx libraries and reference them in project configuration. Follow existing naming conventions and update `nx.json` tags to maintain boundaries.

Last verified on 2025-06-22 @ commit 2634a4fd23074adcf5add56123bd7d02863b90e9.
