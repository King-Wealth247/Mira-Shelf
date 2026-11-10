# supabase/ — database & server-side functions

Never bundled into the mobile app. The **service-role key** is used only here and in operator
scripts, never shipped to devices or committed.

## Migration sequence (Phase 2 — Section 9.8)

| Migration | Content |
|-----------|---------|
| `0001_types_and_schemas` | schemas, extensions (`pg_trgm`, optional `vector`), enums, default privilege changes |
| `0002_identity_tables` | `profiles`, `user_roles`, `admin_audit_logs` + append-only trigger |
| `0003_catalogue_tables` | `categories`, `books` (legal gate), `book_categories`, `book_chunks`, import tables, indexes |
| `0004_user_activity_tables` | `reading_progress`, `reading_sessions`, `saved_books`, conversations, messages, `ai_usage_events` |
| `0005_triggers` | `handle_new_user`, last-super-admin guard, conversation immutability, `updated_at`, search vector |
| `0006_role_helpers_and_rpc` | role helpers, `set_user_role`, reading RPCs, retrieval, admin stats, `create_conversation` |
| `0007_rls` | RLS enablement, privileges, policies |
| `0008_storage` | buckets and storage policies |
| `0009_seed_categories` | the 10 categories of Section 5.2 (idempotent) |

> Migrations are **not yet written** — they are Phase 2. Never edit a deployed migration; add a new one.

## Edge Functions

| Function | Responsibility |
|----------|----------------|
| `ai-chat` | validate, rate-limit, load conversation under user JWT, retrieve passages, call adapter, verify, persist |
| `admin-books` | validate book form, signed upload URLs, sanitise uploads, publish gate, audit |
| `admin-roles` | verify Super Admin, confirm input, call `set_user_role` |
| `delete-account` | delete the caller's auth user (cascade) |

## Local commands

```bash
supabase start          # bring up the local stack (Docker)
supabase db reset       # apply all migrations on an empty DB
supabase test db        # run the pgTAP RLS + function suite (BLOCKING in CI)
```
