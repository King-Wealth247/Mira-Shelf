# Mira-Shelf — AI-Powered E-Book Library

A secure digital library (React Native, Android first) backed by Supabase, with an AI reading
assistant that answers only from retrieved passages of the book being read. Three roles
(User, Admin, Super Admin), an auditable privilege model, and a catalogue of exactly 200
rights-verified books.

> **Start here:** [`IMPLEMENTATION_PLAN.md`](IMPLEMENTATION_PLAN.md) is the working guide — stages,
> steps, tasks, and the completion tracker. The authoritative requirements are in
> `EBook_Library_Cahier_des_Charges.pdf` (v1.0).

## Tech stack (fixed)

| Layer | Technology |
|-------|------------|
| Frontend | React Native (TypeScript), Android first |
| Database & auth | Supabase: PostgreSQL, Auth, Row Level Security, Storage |
| Backend services | Supabase Edge Functions + a small EC2 companion service |
| AI | Existing integration behind a provider-independent adapter (DP-1) |
| Deployment | AWS EC2 (server side) + Android release build |

## Repository layout (Section 12 of the spec)

```
Mira-Shelf/
├─ app/                 # React Native application (frontend only — never ships secrets)
├─ supabase/            # Database + server-side functions (never bundled into the app)
│  ├─ migrations/       # 0001 … 0009 SQL
│  ├─ tests/            # pgTAP RLS and function tests
│  └─ functions/        # _shared, ai-chat, admin-books, admin-roles, delete-account
├─ services/companion/  # EC2 companion service (import worker, optional AI adapter host)
├─ scripts/             # operator tools (service-role key from environment)
│  └─ manifest/         # approved_books.csv + rights evidence checklist
├─ deploy/              # proxy config, systemd/compose, monitoring, runbooks
├─ e2e/                 # Maestro / Detox flows
├─ docs/                # setup, security, testing, deployment, ADRs, decision records
└─ .github/workflows/   # CI
```

## Getting started (developer)

See [`docs/SETUP.md`](docs/SETUP.md) once written (Phase 12). Current state is **Stage 0 — bootstrap**.

Prerequisites to install on your machine (not committed here):
- Node.js LTS + a package manager (npm/pnpm)
- React Native CLI toolchain (JDK, Android SDK)
- Docker + Supabase CLI (pin the version in `docs/`)

## Security ground rules (do not break)

- The **anon key** is the only Supabase key in the mobile app. The **service-role key** and AI
  credentials live only in Edge Function secrets / AWS Secrets Manager / SSM / operator shells.
- Never commit secrets. CI secret-scanning blocks the dev password and service-role key patterns.
- Roles are never client-supplied; the AI never reaches the database.

## License

License: **TBD** — to be confirmed by the project owner before any public distribution.
