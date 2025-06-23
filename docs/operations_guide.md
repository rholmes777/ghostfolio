# Operations Guide

## Configuration
Required environment variables are defined in [.env.example](../.env.example). Secrets such as `REDIS_PASSWORD`, `POSTGRES_PASSWORD`, and `ACCESS_TOKEN_SALT`  must be provided via your secret store. Defaults are set in `ConfigurationService` using `envalid`.

Docker Compose files:
- `docker/docker-compose.yml` for production
- `docker/docker-compose.dev.yml` for local development
Override options in a `.env` file placed at repo root.

## Build & Deploy
Primary Nx targets:
- **Build:** `npm run build:production` → builds both `api` and `client` for container images.
- **Lint:** `npm run lint` → runs ESLint across all projects.
- **Test:** `npm run test` → executes unit tests (Jest).
- **CI Builds:** `nx affected` commands support selective builds in CI.

CI runs these steps in [.github/workflows/build-code.yml](../.github/workflows/build-code.yml).

Prisma migrations run via `npm run database:migrate` during container startup. Tags follow `MAJOR.MINOR.PATCH` and trigger Docker image builds.

## Security & Compliance
Authentication uses JWT tokens stored client side. RBAC is enforced via Nest guards. TLS is expected in production. Data is encrypted at rest (if enabled in Postgres) and in transit via HTTPS. Inputs are validated with `class-validator`; common OWASP recommendations apply.

## Testing Strategy
- **Unit tests** with Jest cover utilities and services
- **Integration tests** validate Prisma interactions and NestJS API
- **E2E** End-to-End tests run with Cypress (client-e2e) against the built Angular app.
Coverage gates are defined in `nx.json` and enforced in CI.


## Extensibility
To add jobs or features, create new Nx libraries and reference them in project configuration. Follow existing naming conventions and update `nx.json` tags to maintain boundaries.

Last verified on 2025-06-22 @ commit 2634a4fd.
