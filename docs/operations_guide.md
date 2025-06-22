# Operations Guide

## Configuration
Required environment variables are defined in [.env.example](../.env.example). Secrets such as `REDIS_PASSWORD` and `POSTGRES_PASSWORD` must be provided via your secret store. Defaults are set in `ConfigurationService` using `envalid`.

Docker Compose files:
- `docker/docker-compose.yml` for production
- `docker/docker-compose.dev.yml` for local development
Override options in a `.env` file placed at repo root.

## Build & Deploy
Key Nx targets:
- `build` – compiles API and client
- `test` – runs Jest suites
- `lint` – checks code style
- `affected` – targets only changed projects
CI runs these steps in [.github/workflows/build-code.yml](../.github/workflows/build-code.yml).

Prisma migrations run via `npm run database:migrate` during container startup. Tags follow `MAJOR.MINOR.PATCH` and trigger Docker image builds.

## Security & Compliance
Authentication uses JWT tokens stored client side. RBAC is enforced via Nest guards. TLS is expected in production. Data is encrypted at rest by Postgres if configured and in transit via HTTPS. Inputs are validated with `class-validator`; common OWASP recommendations apply.

## Testing Strategy
- **Unit tests** with Jest for utilities and services
- **Integration tests** against the NestJS API
- **E2E** tests using Cypress (client-e2e)
Coverage gates are defined in `nx.json` and enforced in CI.

Last verified on 2025-06-22 @ commit 2634a4fd.
