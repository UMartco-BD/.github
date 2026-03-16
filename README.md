# Umartco Organization Overview

This document gives a single, practical overview of all projects inside this workspace and how they fit together.

## What This Organization Contains

Umartco is organized as multiple apps and APIs that share business domains (catalog, admin operations, merchant operations, fulfillment, and backups).

- Frontend apps built with Next.js (`clientweb`, `adminweb`, `staffweb`, `storeweb`)
- Backend APIs built primarily with Bun + Express + Prisma (`clientbackend`, `adminstaffbackend`, `storebackend`, `delivery_backend`)
- Supporting service for database backups (`backup-service`)
- Shared static/media assets (`Assets`)

## Project-by-Project Map

### Customer Experience

- `clientweb`
  - Role: Customer-facing storefront (public browsing, product discovery, checkout-facing UI flows).
  - Stack: Next.js.
  - Local run: `bun run dev`.

- `clientbackend`
  - Role: Public/customer API used by `clientweb`.
  - Stack: Bun + Express + Prisma.
  - Default port: `1200` (`src/server.ts`).
  - Main route groups:
    - `/api/auth`
    - `/api/departments`
    - `/api/client`
    - `/api/v1/recommendation`
    - `/v1`

### Admin & Operations

- `adminweb`
  - Role: Admin dashboard frontend.
  - Stack: Next.js.

- `staffweb`
  - Role: Staff operations frontend (warehouse/operations/support workflows).
  - Stack: Next.js.

- `adminstaffbackend`
  - Role: API for admin + staff platforms.
  - Stack: Bun + Express + Prisma.
  - Default port: `5051` (`src/server.ts`).
  - Main route groups:
    - `/api/admin`
    - `/api/upload`

### Merchant / Store Management

- `storeweb`
  - Role: Store/merchant web frontend.
  - Stack: Next.js.
  - Docker runtime port currently configured: `1203`.

- `storebackend`
  - Role: Merchant/admin JWT-protected store backend API.
  - Stack: Bun + Express + Prisma.
  - Default port: `5052` (`src/server.ts`).
  - Main route groups:
    - `/api/merchant`
    - `/api/admin`

### Logistics & Integrations

- `delivery_backend`
  - Role: Delivery-related backend integration service (e.g., courier/pathao integration code lives here).
  - Stack: Bun + TypeScript.

### Platform Maintenance

- `backup-service`
  - Role: Standalone backup utility service.
  - Stack: Python + FastAPI.
  - Purpose: Runs `pg_dump`, sends backups to webhook, and handles cleanup.

## High-Level Data / Request Flow

1. Admin/staff actions happen in `adminweb` or `staffweb` and call `adminstaffbackend`.
2. Product and catalog data is persisted through Prisma into PostgreSQL.
3. Customer app `clientweb` reads public commerce data through `clientbackend`.
4. Merchant/store operations use `storeweb` + `storebackend` (JWT-protected routes).
5. Backups are handled by `backup-service` independently of user-facing traffic.

## Deployments (Current CI/CD Snapshot)

The repository includes GitHub Actions based Docker build/push workflows and Dokploy deploy hooks.

Current GHCR image names in workflows:

- `storeweb` -> `ghcr.io/umartco-bd/storeweb:latest`
- `clientbackend` -> `ghcr.io/umartco-bd/client_backend:latest`
- `storebackend` -> `ghcr.io/umartco-bd/storebackend:latest`

> Note: Tagging strategy is not fully uniform yet (some Dokploy payloads use `latest`, some use commit SHA).

## Shared Technology Summary

- Runtime / server: Bun + Express (majority of APIs)
- ORM: Prisma
- Database: PostgreSQL
- Frontend framework: Next.js
- Container registry: GHCR (`ghcr.io/umartco-bd/...`)
- Deployment trigger: Dokploy webhook per service

## Suggested Conventions For New Services

If new apps/services are added, keep this structure consistent:

- Name by domain + layer (`<domain>web`, `<domain>backend`)
- Keep one service-specific README inside each project
- Keep one organization-level summary in this file
- Prefer consistent Docker tag format (`latest` + `${GIT_SHA}`)
- Document route groups and default port in each backend README
