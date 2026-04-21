# 7 Spring Engineering

This organization holds the private software systems for 7 Spring and related retail properties.

The platform has been migrated into a structured monorepo. New 7 Spring product work should start in `7s-workspace` unless it clearly belongs to one of the independent retail web properties listed below.

## Current Repositories

| Repository | Role | What belongs here |
| --- | --- | --- |
| [`7s-workspace`](https://github.com/7-Spring/7s-workspace) | Canonical 7 Spring platform monorepo | Member apps, operations apps, backend services, shared packages, contracts, local Supabase infrastructure, seeds, and platform docs |
| [`shop-backgammon`](https://github.com/7-Spring/shop-backgammon) | Backgammon.com retail storefront | Luxury ecommerce storefront built with Next.js and Shopify Storefront API integration |
| [`7s-cafe-retail`](https://github.com/7-Spring/7s-cafe-retail) | 7 Spring cafe and retail web surface | Public cafe/retail web experience, menu/shop/play pages, SEO, and related static assets |

## Platform Monorepo

`7s-workspace` is the source of truth for the 7 Spring product platform. It is a pnpm + Turbo monorepo with explicit service boundaries.

| Area | Location | Purpose |
| --- | --- | --- |
| Member web | `apps/client-web` | Customer-facing web application |
| Member mobile | `apps/client-mobile` | Customer-facing mobile app |
| Ops web | `apps/ops-web` | Internal operations web application |
| Ops mobile | `apps/ops-mobile` | Internal operations mobile app |
| Client backend | `services/client-backend` | Member-facing API and domain logic |
| Operations backend | `services/operations-backend` | Internal operations API and workflows |
| Member auth | `services/member-auth` | Privileged member-auth lifecycle service |
| Shared packages | `packages/*` | UI, contracts, domain helpers, email, links, design tokens, and gameplay engine code |
| Local platform | `infra/*` | Local Docker, Supabase, migrations, seeding, and dev orchestration |

Boundary rules are enforced in the workspace:

- Apps do not import from other apps; shared code moves through packages.
- Services do not import from apps.
- Packages do not import from apps or services.
- Member and ops domain code stay separated unless a shared package is intentionally introduced.

## Independent Retail Properties

`shop-backgammon` and `7s-cafe-retail` are intentionally separate from the 7 Spring platform monorepo. They are brand/retail web properties with different deployment and product lifecycles.

Keep them separate unless they start sharing real runtime contracts with the platform. If shared code becomes necessary, prefer a small package or API boundary over moving either property into `7s-workspace`.

## Cafe Repo Decision

Keep `7s-cafe-retail`.

`7s-cafe-retail-web` appears to be the older duplicate:

- Both repos have the same Vite/React app shape and package name.
- `7s-cafe-retail-web` was last pushed on March 18, 2026.
- `7s-cafe-retail` includes those March commits plus the later April 14, 2026 SEO work.
- The local active checkout is `7s-cafe-retail`.

Recommended cleanup:

1. Confirm any deployment still pointing at `7s-cafe-retail-web` has been moved to `7s-cafe-retail`.
2. Archive `7s-cafe-retail-web`.
3. Keep all future cafe/retail web work in `7s-cafe-retail`.

## Daily Development

Use the README inside each repository for setup details.

For the platform monorepo:

```bash
pnpm install
pnpm infra:start
pnpm bootstrap:env
pnpm infra:seed
pnpm dev
```

Before committing platform changes:

```bash
pnpm check
```

## Legacy Repositories

The old split 7 Spring app and service repositories have been absorbed into `7s-workspace`. Treat them as historical references only. Do not start new product work in legacy repos unless an active migration note explicitly says to do so.
