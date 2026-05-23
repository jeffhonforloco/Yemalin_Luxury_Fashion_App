# Project Context — Yèmalín

## Identity
Project: Yèmalín
Owner: Tarvico Inc.
Status: Active development (pre-launch)

## Stack
- Frontend: React Native 0.81.5 + React 19 / TypeScript 5.9 / Expo 54 (Expo Router) / React Native Web
- Backend: Node.js / Hono 4.9 / tRPC 11 (type-safe RPC over Hono)
- Database: PostgreSQL via Neon serverless (@neondatabase/serverless + pg)
- Cache: none
- Queue: none
- AI: Anthropic SDK (Claude claude-sonnet-4-20250514 primary)
- Auth: Custom JWT (jose + bcryptjs) — access tokens 7d, refresh tokens 30d, stored in AsyncStorage
- Payments: Stripe (configured, stub implementation)
- Email: SendGrid or Klaviyo (dual support, stub implementation)
- File storage: AWS S3 (optional, configured via env)
- Cloud: Rork AI hosting (EXPO_PUBLIC_RORK_API_BASE_URL)
- Functions: none (Hono handles all API routes)
- CI/CD: none configured

## Key Paths
```
expo/app/              ← Expo Router pages (file-based routing, tab + stack nav)
expo/backend/          ← All backend code
expo/backend/trpc/     ← tRPC routers and procedures
expo/backend/trpc/routes/auth/     ← signup, login, me
expo/backend/trpc/routes/products/ ← getAll, getById, getComingSoon
expo/backend/trpc/routes/orders/   ← create, getMyOrders, updatePayment
expo/backend/trpc/routes/admin/    ← dashboard, emails, carts, analytics, orders
expo/backend/db/       ← PostgreSQL connection pool + model queries
expo/backend/db/models/ ← users.ts, products.ts, orders.ts
expo/backend/auth/     ← JWT signing/verification (jwt.ts)
expo/backend/services/ ← stripe.ts, email.ts (external SDK wrappers)
expo/backend/hono.ts   ← Hono app entry point + tRPC server mount
expo/components/       ← Reusable UI components
expo/providers/        ← React Context providers (Auth, Cart, Marketing, VIP)
expo/lib/              ← Shared utilities (trpc.ts client, reminderSystem, emailStorage)
expo/data/             ← Static seed data (products.ts)
expo/scripts/          ← Migration + utility scripts
```

## Critical Conventions
- Multi-tenancy: not applicable (single-tenant B2C app); user identity is the isolation boundary
- Row-level isolation: application-layer user filter (user_id on orders; no org_id)
- LLM provider priority: Tarvico → anthropic → openai → nemotron → deepseek → llama
- Embeddings: OpenAI text-embedding-3-small (1536 dims); Voyage-3 (1024 dims) via stub
- Agent hierarchy: parent_role TEXT (soft ref, NOT UUID FK) — preserves org-agnostic seeding
- Prompt templates: same template_key, org config wins over system defaults
- Audit logs: phi_access_log writes required even on BAA-blocked missions
- Design tokens: use var(--bg-raised), var(--text-primary) — no hardcoded colors
- VIP tiers: bronze / silver / gold / platinum — derived from total_spent, stored as vip_tier on users table
- tRPC: all API calls go through tRPC client in expo/lib/trpc.ts — no raw fetch to backend
- Auth context: JWT decoded server-side in expo/backend/trpc/create-context.ts — always validate there, not in individual routes
- State management: Zustand 5 for global client state; TanStack Query 5 for server state via tRPC
- Validation: Zod 4 for all input schemas at tRPC procedure boundaries
- DB queries: raw parameterized SQL via pg pool — no ORM

## Environment Variables
DATABASE_URL
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
JWT_SECRET
JWT_EXPIRES_IN
JWT_REFRESH_EXPIRES_IN
API_URL
NODE_ENV
EXPO_PUBLIC_RORK_API_BASE_URL
STRIPE_SECRET_KEY
STRIPE_PUBLISHABLE_KEY
STRIPE_WEBHOOK_SECRET
KLAVIYO_API_KEY
KLAVIYO_LIST_ID
SENDGRID_API_KEY
SENDGRID_FROM_EMAIL
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_S3_BUCKET
GOOGLE_ANALYTICS_ID
FACEBOOK_PIXEL_ID
ADMIN_EMAIL

## Do NOT
- Break existing audit chains
- Break the mission state machine
- Add UUID FKs to agent parent_role
- Remove org_id from any table
- Skip row-level isolation on new tables
- Add cloud-provider-specific SDKs without approval (keep infra swappable)
- Bypass tRPC by calling backend routes directly from client code
- Store JWT secrets or DB credentials in client-side code or AsyncStorage
- Hard-code tier thresholds — VIP logic lives in expo/backend/db/models/users.ts

## Known Deferred Items
- Stripe payment flow: stub in place, full integration pending
- Email sending: SendGrid/Klaviyo stub in place, templates defined, sending not wired
- AWS S3 upload: env configured, upload logic not implemented
- CI/CD pipeline: not yet set up
- No ORM / migration framework: DB schema managed via raw SQL scripts in expo/scripts/

## Last Updated
2026-05-23 — Filled in from codebase exploration (stack, paths, env vars, conventions)
