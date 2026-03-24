# 7 Spring — App-of-Apps

Coordination workspace for the 7 Spring application suite. This repo does not contain the product code for every app — it owns the shared packages, local test harness, cross-repo documentation, and orchestration scripts that keep the platform moving in a consistent direction.

## Repo Map

The 7 Spring platform is organized into **channel apps** (frontends that users interact with), **domain services** (backends that own business logic and data), and **shared infrastructure** (packages, migrations, and tooling in this repo).

### Employee Web Apps

Internal tools used by staff and operators. All deployed on **Vercel**.

| Repo | Purpose | Hosting |
| --- | --- | --- |
| `7s-inventory` | Inventory management — items, counts, purchasing, vendors, and financial operations | Vercel |
| `7s-host-terminal` | Host desk — door events, greeting, coat check, table management, and reservations | Vercel |
| `7s-marketing` | Marketing operations — flyers, campaigns, digests, and outbound communications | Vercel |
| `7s-membership` | Membership operations — member records, cards, visits, standing, and coordinator tools | Vercel |
| `7s-landing` | Platform shell, launcher, and home for the canonical Supabase project (migrations, edge functions, seeds) | Vercel |
| `7s-cafe-retail` | Café and retail point-of-sale operations | Vercel |

### Member-Facing Apps

Apps used by members (clients). Housed in a single monorepo.

| Repo | App | Purpose | Hosting |
| --- | --- | --- | --- |
| `7s-client-apps` | `apps/web` | Member web portal — profile, events, concierge, LFG, backgammon | Vercel |
| `7s-client-apps` | `apps/mobile` | Member mobile app (Expo / React Native) | App Store |

The `7s-client-apps` monorepo also contains shared internal packages: `packages/contracts` (generated API contracts), `packages/ui` (shared component library), and `packages/backgammon-engine`.

### Employee Mobile App

| Repo | Purpose | Hosting |
| --- | --- | --- |
| `7s-mobile` | Internal mobile app for inventory and host workflows (Expo / React Native) | App Store (internal distribution) |

### Backend Services

| Repo | Purpose | Hosting |
| --- | --- | --- |
| `7s-client-backend` | Authoritative member platform backend — memberships, billing, Stripe, host APIs, concierge, LFG, member health, published events/flyers, notifications. Includes a separate worker process for async jobs. | Render |
| `7s-operations-backend` | Internal operations backend — inventory APIs, purchasing, vendor management, marketing authoring, campaign sends, and internal reporting | Render |
| `7s-member-auth-admin` | Privileged auth lifecycle service — member enrollment, disenrollment, and login-link generation. Narrow scope by design. | Vercel |

### Future / Planned

| Repo | Purpose | Status |
| --- | --- | --- |
| `7s-bgcom` | Backgammon.com retail integration | TBD |
| `7s-business` | Business financial performance and estimation dashboards | Planned (Vercel) |

### This Repo (`7s-app-of-apps`)

Not a deployable app. This is the platform workspace that ties everything together.

| Directory | Contents |
| --- | --- |
| `packages/` | Shared npm packages published under `@eacooke/*` — see [Shared Packages](#shared-packages) below |
| `docs/` | Architecture docs, runbooks, migration plans, and dev logs |
| `scripts/` | Dev orchestration scripts, testing harnesses, and CI tooling |
| `config/` | Shared configuration |
| `__tests__/` | Cross-repo integration tests |

## Deployment Topography

```
┌─────────────────────────────────────────────────────────┐
│  Vercel                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐  │
│  │Inventory │ │Host Term │ │Marketing │ │Membership │  │
│  └──────────┘ └──────────┘ └──────────┘ └───────────┘  │
│  ┌──────────┐ ┌──────────┐ ┌───────────────┐           │
│  │ Landing  │ │Cafe Retail│ │Client Apps Web│           │
│  └──────────┘ └──────────┘ └───────────────┘           │
│  ┌──────────────┐                                       │
│  │Member Auth   │                                       │
│  │Admin         │                                       │
│  └──────────────┘                                       │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Render                                                 │
│  ┌──────────────────┐  ┌──────────────────────────────┐ │
│  │ Client Backend   │  │ Operations Backend           │ │
│  │ + Worker process │  │                              │ │
│  └──────────────────┘  └──────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Supabase (single shared project)                       │
│  Canonical migrations owned by 7s-landing               │
│  Schema isolation + per-backend database roles          │
│  Schemas: platform, authz, client, membership,          │
│           host_terminal, backend, inventory              │
│  Planned: marketing, published                           │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  App Stores                                             │
│  ┌──────────────────┐  ┌──────────────────┐             │
│  │ Client Mobile    │  │ Ops Mobile       │             │
│  │ (7s-client-apps) │  │ (7s-mobile)      │             │
│  └──────────────────┘  └──────────────────┘             │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  External Services                                      │
│  Stripe · Google Places API · OpenTable                 │
└─────────────────────────────────────────────────────────┘
```

## Architecture Overview

The platform follows a few core principles. For the full specification, see `docs/architecture/platform-target-state-v2.md`.

**Two backends, not ten.** The platform intentionally consolidates into two domain backends rather than one per app. `Client Backend` owns all member-facing truth (memberships, billing, host workflows, and published member-facing projections of events and flyers). `Operations Backend` owns internal operations (inventory, purchasing, marketing authoring). When Operations Backend needs to publish member-facing content, it calls Client Backend APIs — it never writes directly to Client Backend tables. Channel apps stay thin and consume APIs — they don't re-implement domain rules.

**One write owner per domain.** Every database table is assigned to exactly one backend. Cross-domain reads use read-only grants. No cross-schema writes. The full table-to-owner mapping is in `docs/architecture/domain-ownership-matrix.md`.

**Single Supabase project, strict isolation.** Near-term, all data lives in one Supabase project with schema-level and role-level isolation. Each backend connects with its own database role and can only write to its own schemas. A future split into separate public/internal data planes is planned but not yet triggered.

**Canonical migrations live in `7s-landing`.** All Supabase migrations, edge functions, seeds, and project configuration are owned by the `7s-landing` repo. This is the single source of truth for the database schema.

**Channel apps → Backend APIs, not shared database.** Web and mobile frontends consume backend APIs. Direct database access from channel apps is being eliminated where it still exists (notably `7s-mobile`).

## Shared Packages

Published under the `@eacooke` scope via the workspace. Managed with pnpm.

| Package | Description |
| --- | --- |
| `@eacooke/email` | Shared email transport and rendering primitives |
| `@eacooke/operator-auth` | Operator authentication, authorization, and service-to-service credential helpers |
| `@eacooke/platform-context` | Shared platform context fetchers (entities, app access, profiles) |
| `@eacooke/platform-tasks` | Task/workflow queue primitives |
| `@eacooke/backgammon-engine` | Shared backgammon game engine (rules, move generation, state management) |

Build, test, and lint all packages:

```bash
pnpm packages:build
pnpm packages:test
pnpm packages:lint
```

## How Things Connect

```
Member apps ──→ Client Backend ──→ Supabase (client, membership, host_terminal, backend schemas)
                     │
                     ├──→ Stripe (billing)
                     ├──→ Auth Lifecycle Service (enrollment)
                     └──→ Worker (async jobs, notifications, health evaluation)

Ops apps ─────→ Operations Backend ──→ Supabase (inventory schema)
  │                   │
  │                   ├──→ Client Backend (audience resolution, publish projections)
  │                   └──→ Email package (campaign sends)
  │
  └───── Some ops apps also call Client Backend directly for member/host workflows
```

## Development

### Prerequisites

- Node.js (LTS)
- pnpm (`corepack enable && corepack prepare`)
- Docker (for local Supabase and integration tests)
- Expo CLI + Xcode (for mobile development)
- Detox (for mobile E2E testing)

### Common Workflows

Check the local test environment is ready:

```bash
npm run testenv:check
```

Start the shared local Supabase stack:

```bash
npm run supabase:local -- start --app inventory
```

Seed the local database:

```bash
npm run seed:local
```

Run gate checks (contract validation and CI dry-runs):

```bash
npm run gate:contract
npm run gate:pr:dry-run
npm run gate:main:dry-run
```

Orchestrate all local services:

```bash
npm run start-all    # start everything
npm run status       # check what's running
npm run stop-all     # tear it down
npm run start-one    # start a single service
```

## Key Docs

| Document | Location |
| --- | --- |
| Platform Target State V2 | `docs/architecture/platform-target-state-v2.md` |
| Domain Ownership Matrix | `docs/architecture/domain-ownership-matrix.md` |
| Repo Classification | `docs/architecture/repo-classification.md` |
| Schema Governance | `docs/architecture/schema-governance.md` |
| Dev Startup Guide | `docs/dev-startup.md` |
| Testing Port Map | `docs/agent-testing-port-map.md` |
| Supabase Security & RLS | `docs/supabase-security-rls-migration-plan-2026-02-20.md` |

## Transitional & Legacy Repos

These directories exist on disk but are not part of the target-state architecture. See `docs/architecture/repo-classification.md` for full details.

| Directory | Status | Notes |
| --- | --- | --- |
| `7s-backend/` | Transitional | Legacy backend being absorbed into `7s-client-backend`. Do not add features. |
| `7s-client-app/` | Transitional | Original client backend candidate, superseded by `7s-client-backend`. Pending archive. |
| `7s_OS_Proof_of_Work/` | Non-runtime | Architecture evidence and portfolio documentation. |
