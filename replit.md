# SecurePrint AI

India's most secure document printing platform — replaces WhatsApp document sharing between customers and print shops with AES-256 encrypted, privacy-first, auto-deleting document workflows.

## Run & Operate

- `pnpm --filter @workspace/secureprint run dev` — run the frontend (port 26145, preview path `/`)
- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — for OTP hashing and cookie signing

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + TailwindCSS + Framer Motion + shadcn/ui + Recharts + Wouter
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Fonts: Plus Jakarta Sans (display) + Inter (body)

## Where things live

- `lib/api-spec/openapi.yaml` — API contract source of truth
- `lib/db/src/schema/` — Drizzle table definitions (users, shops, documents, orders, queue, subscriptions, audit logs, reviews)
- `artifacts/api-server/src/routes/` — Express route handlers (auth, documents, orders, shops, queue, analytics, admin, ai, subscriptions)
- `artifacts/secureprint/src/` — React frontend (pages, components, hooks)

## Architecture decisions

- **Cryptographic deletion:** Documents are deleted by clearing `encryptedData` + `encryptionKeyRef` fields, with an audit hash stored for the privacy receipt — the key is destroyed, making recovery impossible even if disk sectors remain
- **Anonymous order IDs:** Customer identity is decoupled from print shop access; shops only see `ANON-XXXXXX` identifiers
- **AES-256 encryption placeholder:** File data flows through an encryption layer (simulated in dev; in production, integrate AWS KMS or Vault for key management)
- **Signed one-time URLs:** Document access requires a fresh signed URL per print job, expiring in 5 minutes
- **OTP auth:** No passwords — email OTP flow with SHA-256 hashed tokens; sessions stored in memory (production: use Redis)
- **Role-based routing:** `customer`, `shop_owner`, `admin` roles drive UI and API access

## Product

- **Landing page** (`/`) — hero, how it works, stats, testimonials, FAQ, pricing preview, footer
- **Customer upload flow** (`/upload`) — drag-and-drop, AI sensitivity detection, print options wizard, shop selector, QR delivery
- **Order tracking** (`/order/:id`) — status timeline, secure QR display, countdown to auto-deletion, privacy receipt
- **Print shop dashboard** (`/shop/dashboard`) — KPI cards, queue overview, recent orders
- **Analytics** (`/shop/analytics`) — revenue charts, peak hours, status breakdown
- **AI Assistant** (`/ai-assistant`) — document analysis, price estimation, natural language chat
- **Admin panel** (`/admin`) — shop approvals, user management, audit logs, platform stats
- **Pricing** (`/pricing`) — Free/Basic/Professional/Enterprise plans with monthly/yearly toggle

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- After changing `lib/db/src/schema/`, run `pnpm run typecheck:libs` before typechecking `artifacts/api-server`
- After changing `lib/api-spec/openapi.yaml`, run `pnpm --filter @workspace/api-spec run codegen`
- The `50mb` body limit on Express supports base64-encoded file uploads; do not lower it
- Orval auto-derives `<OperationId>Response` Zod names for response schemas — never name component schemas with that pattern or you'll get TS2308 collisions

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
