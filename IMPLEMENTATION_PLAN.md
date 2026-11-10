# AI-Powered E-Book Library — Implementation Plan

> **Purpose.** This is the working guide for building the AI-Powered E-Book Library described in
> `EBook_Library_Cahier_des_Charges.pdf` (v1.0). It breaks the project into stages, steps and
> trackable tasks, maps each to its requirements and verifying tests, and tracks overall completion.
> Keep this file updated as work proceeds: tick boxes, fill in dates/owners, and recompute the
> completion table at the bottom.

- **Project:** AI-Powered E-Book Library (Mira-Shelf)
- **Stack (fixed):** React Native (TypeScript, Android first) · Supabase (Postgres, Auth, RLS, Storage, Edge Functions) · existing AI integration behind a provider-independent adapter · AWS EC2 companion service
- **Baseline schedule:** 4 weeks + 1 contingency week, team of 1–3 developers
- **Initial catalogue:** exactly 200 rights-verified books
- **Plan status:** Stage 0 bootstrap scaffolded; day-1 DP tracks prepared — ~3% complete (see [Completion Tracker](#completion-tracker))
- **Legend:** `[ ]` not started · `[~]` in progress · `[x]` done · 🔴 critical path · ⚖️ legal gate · 🔒 security-blocking

---

## Table of Contents

1. [How to use this plan](#how-to-use-this-plan)
2. [Guiding principles (non-negotiable)](#guiding-principles-non-negotiable)
3. [Decision points to close first](#decision-points-to-close-first)
4. [Stage 0 — Project bootstrap](#stage-0--project-bootstrap-pre-phase)
5. [Phase 1 — Requirements validation & architecture](#phase-1--requirements-validation--architecture)
6. [Phase 2 — Supabase setup & database](#phase-2--supabase-setup--database-implementation)
7. [Phase 3 — Authentication & role-based authorisation](#phase-3--authentication--role-based-authorisation)
8. [Phase 4 — Design system & navigation](#phase-4--react-native-design-system--navigation)
9. [Phase 5 — Catalogue & legal import pipeline](#phase-5--book-catalogue--legal-import-pipeline-)
10. [Phase 6 — Reading interface & progress](#phase-6--reading-interface--progress-tracking)
11. [Phase 7 — AI integration & secure retrieval](#phase-7--ai-integration--secure-book-retrieval-)
12. [Phase 8 — Saved books & reading history](#phase-8--saved-books--reading-history)
13. [Phase 9 — Admin & Super Admin tooling](#phase-9--administrative-dashboard--book-management)
14. [Phase 10 — Security testing & QA](#phase-10--security-testing--quality-assurance-)
15. [Phase 11 — AWS EC2 deployment & release](#phase-11--aws-ec2-deployment--release-preparation)
16. [Phase 12 — Documentation & final validation](#phase-12--documentation--final-validation)
17. [Critical path & scope-cut order](#critical-path--scope-cut-order)
18. [Acceptance criteria checklist](#acceptance-criteria-checklist)
19. [Final deliverables checklist](#final-deliverables-checklist)
20. [Completion Tracker](#completion-tracker)

---

## How to use this plan

- **One source of truth.** Every task traces to a requirement ID (`FR-*`, `SEC-AI-*`, `NFR-*`) and/or a test ID (`T-*`) from the spec. Do not mark a task done until its listed tests pass.
- **Order matters.** Phases run roughly in sequence, but two long-lead tracks start on **day one in parallel**: the **AI inspection (DP-1)** and the **rights review (Section 7)**. Both are on the critical path.
- **Gates are hard.** Database/RLS tests (Phase 2) and AI-security tests (Phase 7/10) are release-blocking. No blocker or high-severity security defect may remain at release.
- **Tracking.** Update the checkbox, then recompute the per-phase and overall percentages in the [Completion Tracker](#completion-tracker). Suggested weight = number of tasks in the phase.

---

## Guiding principles (non-negotiable)

- [ ] **AI never touches the database.** The model gets pure text in / text out — no tools, no DB client, no credentials, no URLs. Enforced by architecture, server-side authz, RLS, and output verification — *not* by prompt text alone. (Section 6)
- [ ] **Deny by default.** RLS enabled on every table; function execute revoked from `public/anon/authenticated`, then granted per function. (Section 9.1)
- [ ] **Clients read; trusted code writes.** Business-rule writes (progress, sessions, roles, messages, books) go through guarded `SECURITY DEFINER` functions / Edge Functions. (Section 9.1)
- [ ] **Legal gate lives in the database.** A `CHECK` constraint forbids `status='available'` without verified rights, cover, content file, format, checksum, manifest and ≥20-char evidence. (Section 9.4)
- [ ] **No secrets in the app or VCS.** Anon key only in the app; service-role & AI keys only in function secrets / AWS Secrets Manager / SSM / operator shells. (Sections 3.4, 9.6)
- [ ] **Roles are never client-supplied.** `user_roles` is not client-writable; the only write path is `set_user_role` (service_role only). (Sections 4, 10.1)
- [ ] **Exactly 200 books, each rights-verified.** A link is not an import; only validated stored files count. (Section 7)

---

## Decision points to close first

> Development of dependent features must not start before the decision is **recorded in the spec** (bump it to v1.1).

| ID | Decision | Owner | Needed by | Status |
|----|----------|-------|-----------|--------|
| **DP-1** 🔴 | Inspect existing AI integration (provider, API, auth, hosting, limits, retention); choose Pattern A (direct from Edge Function) or B (via EC2 adapter). Use Appendix B checklist. | Backend lead | End of Phase 1 | [ ] **pending** — fillable checklist ready: [`docs/DP1-ai-integration-inspection-checklist.md`](docs/DP1-ai-integration-inspection-checklist.md); decision → [ADR-0002](docs/adr/0002-dp1-ai-integration-pattern.md) |
| **DP-2** ⚖️ | Confirm distribution jurisdictions + conservative rights policy (Section 7.2). | Project owner / legal | End of Phase 1 | [ ] **pending** — decision record ready: [`docs/DP2-rights-policy-decision.md`](docs/DP2-rights-policy-decision.md); decision → [ADR-0003](docs/adr/0003-dp2-rights-policy.md) |
| **DP-3** | Choose EPUB reader library after a 1-day spike on real project deps. | Frontend lead | Phase 4 start / Phase 6 | [ ] |
| **DP-4** | Decide whether embeddings/semantic search are in scope (cost, provider, DB size). | Backend lead | Phase 7 start | [ ] |
| **DP-5** | Select production SMTP provider + domain name for the EC2 companion service. | DevOps | Phase 11 start | [ ] |

---

## Stage 0 — Project bootstrap (pre-phase)

**Goal:** repo, tooling and environments exist so every later phase has a home.
**Status:** mostly done (2026-11-10). Remaining items need a developer machine / Supabase account — see notes.

- [x] Create monorepo skeleton per Section 12 (`app/`, `supabase/`, `services/companion/`, `scripts/`, `deploy/`, `e2e/`, `docs/`). *(git initialised; 53 files)*
- [x] Add `.gitignore`, `.gitattributes`, `.env.example` (placeholders only), README. *(license = TBD, noted in README)*
- [~] Initialise React Native **CLI (bare) TypeScript** project under `app/` — **source tree + `package.json` + `app/README.md` scaffolded; native `android/`/`ios/` generation + `npm install` must be run on a dev machine** (`npx @react-native-community/cli init`). Verify A-01; if Expo-managed, re-verify Section 17.
- [~] Install & pin Supabase CLI + Docker; `supabase init` — **`supabase/config.toml` + folder tree committed; CLI/Docker install is a dev-machine step**, then `supabase start`.
- [x] Stand up CI skeleton (`.github/workflows/ci.yml`): secret-scan (active) + fail-soft stubs for lint/type-check, unit/component, migrations+pgTAP, dependency audit, APK build.
- [x] Configure secret scanning in CI (fails on dev password / service-role key patterns) — `T-DEV-05`.
- [ ] Create three Supabase projects: **local** (Docker), **staging**, **production**; link staging/production with `supabase link`. **⛔ Needs your Supabase account — I can't do this.**
- [x] Set up ADR folder (`docs/adr/`) with `TEMPLATE.md` + `ADR-0001` (stack & repo structure); git initialised.

---

## Phase 1 — Requirements validation & architecture
**Objective:** close open items and fix the architecture. **Dependencies:** none (starts the project).
**Completion criteria:** DP-1 & DP-2 recorded; baseline doc v1.1 issued.

- [ ] 🔴 **Inspect the existing AI integration** using Appendix B (12-item checklist): provider/model/API style, hosting/network, credential storage, request/response formats + JSON output support, tool/memory features (must be disabled — SEC-AI-02), any direct DB/file access (must be removed), context window, latency/limits/pricing, retention/training policy, existing guardrails, existing tests, licensing. → **DP-1** outcome: reuse-as-is / reuse-wrapped-on-EC2 (Pattern B) / replacement via change request.
- [ ] ⚖️ **Confirm jurisdictions & rights policy** with legal contact → **DP-2** (Section 7.2 R-1…R-7).
- [ ] Finalise repository, environments and CI skeleton (carry over from Stage 0 if needed).
- [ ] Start book-candidate selection (~230 candidates across category targets; see Phase 5).
- [ ] Record architecture decisions as ADRs; update Sections 3 & 8 of the spec.
- [ ] Supervisor review of the spec; issue **baseline v1.1**.

**Deliverables:** updated Sections 3 & 8, ADRs, repo with CI.
**Risk:** AI integration opaque or cannot meet SEC-AI-02…07 → raise a documented change request.

---

## Phase 2 — Supabase setup & database implementation
**Objective:** implement the schema and security model. **Dependencies:** Phase 1.
**Completion criteria:** pgTAP suite green in CI; anon access denied everywhere.
**Verifying tests:** all `T-DB-*`, `T-SA-05`, `T-SA-06`, `T-SA-08`.

### 2.1 Project init & config
- [ ] Configure `supabase/config.toml`: email confirmations on (staging/prod), JWT expiry 3600s, refresh-token rotation on, site URL + deep-link redirect URLs, production password min 8; **local** `minimum_password_length = 5` (dev-seed exception, Section 10.4).
- [ ] Create schemas `public` (exposed) and `private` (helpers, not API-exposed); enable `pg_trgm`; conditionally enable `vector` (DP-4).
- [ ] First migration: `alter default privileges in schema public revoke execute on functions from public, anon, authenticated`.

### 2.2 Migrations (0001–0009, run in order)
- [ ] `0001_types_and_schemas` — schemas, extensions, enums (`app_role`, `book_status`, `content_format`, `rights_status`, `cover_rights_status`, `reading_status`), default privilege changes.
- [ ] `0002_identity_tables` — `profiles`, `user_roles`, `admin_audit_logs` + append-only trigger (`block_audit_mutation`).
- [ ] `0003_catalogue_tables` — `categories`, `books` (⚖️ legal gate CHECK), `book_categories` (one-primary partial unique), `book_chunks`, `import_batches`, `import_items`, all indexes (dedupe/source/isbn unique, status, GIN fts, trigram title).
- [ ] `0004_user_activity_tables` — `reading_progress`, `reading_sessions` (one-open partial unique, active_seconds 0..43200), `saved_books`, `chat_conversations`, `chat_messages` (composite FK + idempotency index), `ai_usage_events`.
- [ ] `0005_triggers` — `handle_new_user` (creates profile + role 'user'; role **never** from metadata), last-super-admin guard, conversation immutability, `updated_at`, `books.search_tsv` (title A / authors B / description C, `simple`).
- [ ] `0006_role_helpers_and_rpc` — `is_admin`, `is_super_admin`, `get_my_role`, `set_user_role` (service_role only, 5 guards + audit), reading RPCs (`start/heartbeat/end_reading_session`, `save_reading_progress`), `search_book_chunks` (SECURITY INVOKER), `append_chat_turn` (service_role only), admin stats (`admin_list_users`, `admin_overview`, `admin_user_detail` [writes audit], `admin_top_books`, `admin_recent_activity`), `create_conversation`.
- [ ] `0007_rls` — enable RLS on every table; revoke-then-grant privileges; column-level grant on `books` (hide rights evidence/internal ids); owner-only policies; `book_chunks` "opened-and-available" policy; `ai_usage_events` no policy/no grant (service role only); `admin_audit_logs` Super-Admin read only.
- [ ] `0008_storage` — private buckets `book-content` (30MB; epub/zip/pdf) & `book-covers` (2MB; jpeg/png/webp); read policies tied to `status='available'`; no client write.
- [ ] `0009_seed_categories` — the 10 categories (idempotent) with targets: Fiction 35, Mystery&Adventure 15, Literature 20, Philosophy 20, History 20, Science 20, Education&Learning 15, Personal Development 20, Biography&Memoir 15, Economics&Society 10 (=200).

### 2.3 Tests & verification
- [ ] Write pgTAP suite in `supabase/tests/` (policies, grants, triggers, constraints, storage policies).
- [ ] Verify: role table not client-writable (`T-DB-01`), profiles/progress/saved/sessions/conversation/message RLS (`T-DB-02…07`), book write privileges (`T-DB-08`), audit privileges (`T-DB-09`), storage access denied for draft/archived (`T-DB-10`), anonymous access returns nothing (`T-DB-11`), function grants correct (`T-DB-12`).
- [ ] Verify last-super-admin guard & audit immutability (`T-SA-05/06/08`).
- [ ] CI runs migrations on empty DB + pgTAP **blocking** before merge.

**Risk:** policy gaps → mitigate with test-first policy writing.

---

## Phase 3 — Authentication & role-based authorisation
**Objective:** working auth, roles, and dev accounts. **Dependencies:** Phase 2.
**Completion criteria:** 8 seeded accounts log in with correct roles; guards verified.
**Verifying tests:** `T-AUTH-01…12`, `T-DEV-01…05`, `T-SA-02…04`, `T-SA-10`.

### 3.1 Client auth
- [ ] Supabase client with anon key + **encrypted storage adapter** (Android Keystore via `react-native-keychain`); `autoRefreshToken` on, `detectSessionInUrl` off; never plain AsyncStorage; disable Android `allowBackup`.
- [ ] Registration (`FR-AUTH-01`): validate email/password(≥8)/display name(1–80); role cannot be supplied (trigger assigns 'user').
- [ ] Login + session handling (`FR-AUTH-02`): generic error (no account enumeration); tokens in encrypted storage only.
- [ ] Logout (`FR-AUTH-03`): sign-out, clear session + cached private content, back-button cannot reveal authed screens.
- [ ] Password reset via deep link (`FR-AUTH-04`): identical confirmation for known/unknown email; expired-link recovery.
- [ ] Role resolution `get_my_role()` → navigator selection (UI hint only) (`FR-AUTH-05`).
- [ ] Session expiry handling: one silent refresh then sign-out w/ "session expired" (`FR-AUTH-08`).
- [ ] Account deletion Edge Function (cascade; anonymised audit) (`FR-AUTH-09`, Should).

### 3.2 Server authz & role management
- [ ] `admin-roles` Edge Function: verify JWT → confirm Super Admin → call `set_user_role` + audit in one txn.
- [ ] Grant/revoke Admin flows (`FR-SA-02`, `FR-SA-03`); revocation effective immediately (42501 without re-login).
- [ ] Verify server rejects privileged calls by wrong role (`FR-AUTH-06`, `T-AUTH-09/10`).

### 3.3 Operator scripts & dev accounts (Section 10)
- [ ] `scripts/bootstrap-super-admin` — one-time, aborts if a Super Admin exists unless `--recovery`; writes `role.bootstrap` audit (`FR-SA-07`, `T-SA-10`).
- [ ] `scripts/seed-dev-users` — 5 users, 2 admins, 1 super admin; password from `DEV_SEED_PASSWORD` env; idempotent (`FR-DEV-01`, `T-DEV-01`).
- [ ] `seed:verify` script (`FR-DEV-02`, `T-DEV-02`): stored roles + exactly one Super Admin.
- [ ] **Four-layer production seeding prevention** (`FR-DEV-03`, `T-DEV-03/04`): script env guard, no prod `DEV_SEED_PASSWORD`, exclude from prod artefacts + CI secret scan, pre-release SQL gate (seed-email count = 0).
- [ ] No credentials in VCS (`FR-DEV-04`, `T-DEV-05`).
- [ ] Provision a **second Super Admin** (different person) so the last-super-admin guard is not an operational dead-end.

**Risk:** dev password length vs Auth minimum → ensure local `minimum_password_length = 5`.

---

## Phase 4 — React Native design system & navigation
**Objective:** reusable UI foundation. **Dependencies:** Phase 3 (navigation by role).
**Completion criteria:** all screens can be assembled from shared components.
**Verifying tests:** component tests, `T-UI-01`, `T-NFR-05` (baseline).

- [ ] Theme tokens (Section 11.1): typography scale, colours (navy `#1F3A5F`, teal `#2A7F8E`, surfaces, error/success), 4-pt spacing, radius 8/12, 48dp touch targets; light/sepia(`#F4ECD8`)/dark reader themes; contrast ≥4.5:1.
- [ ] Core components: `BookCard` (2:3 cover, progress ring), list cards, buttons (primary/secondary/text/destructive + disabled/loading), form controls (labelled fields, password show/hide, select sheets, switches), bottom tabs, top app bar, bottom sheets.
- [ ] Feedback patterns: skeleton loaders, spinners, snackbars, inline error banners, empty states (icon + one sentence + one action).
- [ ] Navigators (Section 11.3): Auth stack, User tabs (Home, Library, Assistant, My Books, Profile), User modal stack (Details, Reader, Conversation, History), Admin area (6th tab), Super Admin area (Roles, Audit); `RoleGate` mounts by `get_my_role()` (usability filter only).
- [ ] Deep-link config (password reset intent filter).
- [ ] Accessibility baseline: labels + roles on every control, focus order, cover alt text "Cover of {title}", status not by colour alone, TalkBack, 200% font, reduce-motion.
- [ ] Responsiveness: flex/relative units, 2-col <600dp / 3–4-col above, 720dp tablet max width, keyboard-avoiding containers.
- [ ] Install & verify shared libs per Section 17 compatibility protocol (navigation, react-query, react-hook-form+zod, keychain, async-storage, url-polyfill); one library per purpose; record ADRs.

**Risk:** library compatibility → apply Section 17 protocol (compare peer deps, build debug+release APK, low-end device test, maintenance review, ADR).

---

## Phase 5 — Book catalogue & legal import pipeline ⚖️
**Objective:** populate exactly 200 verified books. **Dependencies:** Phases 2 & 4; DP-2; rights review started Week 1.
**Completion criteria:** `verify` prints PASS with 200 books.
**Verifying tests:** `T-CAT-01…07`, `T-IMP-01…10`.

### 5.1 Catalogue UI & search
- [ ] Home screen (`FR-CAT-01`): Continue-reading carousel, Saved shortcut, AI entry card, category chips, Recently added.
- [ ] Catalogue listing (`FR-CAT-02`): keyset pagination 24/page, infinite scroll, batched signed cover URLs.
- [ ] Search RPC `search_books` (`FR-CAT-03`): debounced 300ms, ≥2 chars, trimmed, ≤100 chars, FTS + trigram, parameterised (no string-built SQL); robustness `T-CAT-04`.
- [ ] Category filter + sorting (`FR-CAT-04`): multi-select chips from active categories; Title/Author A–Z, Recently added, Most read.
- [ ] Book details (`FR-CAT-05`): metadata, own progress, Read/Continue, Save, Ask-AI (disabled until opened).
- [ ] Loading/empty/error states on every data-driven screen (`FR-CAT-06`).

### 5.2 Rights review (two-person, starts Week 1) ⚖️
- [ ] Prepare ~230 candidates to category targets (200 + ~30 reserves).
- [ ] Two independent reviewers per title: author/translator death dates, first publication, edition provenance, illustrations, cover (R-1…R-7); project owner resolves disagreements.
- [ ] Approved rows → `rights_status = verified`.
- [ ] Record per-title evidence a second person can re-check.

### 5.3 Import tool (`scripts/import-books`, Node/TS)
- [ ] Commands: `import --manifest FILE [--dry-run] [--only SEQ,...]`, `verify`, `report --batch ID`.
- [ ] Freeze manifest: exactly 200 approved + ordered reserves → `manifest/approved_books.csv`; record SHA-256.
- [ ] Fetch: official file URL, 30MB size limit, content-type check, timeout+retry, checksum verify when published.
- [ ] Validate & sanitise (`FR-IMP-05`): EPUB container/spine, XHTML well-formedness, strip scripts/handlers/external refs/failing-R4 images+fonts, non-empty text extraction, flag model-directed instructions for human review.
- [ ] Normalise & dedupe (`FR-IMP-04`, `FR-IMP-03`): Unicode NFC, trim, "Surname, Forename"→display, ISO lang, year int, desc ≤4000; four ordered dedupe checks (source id, ISBN, dedupe_key, checksum).
- [ ] Store & index (`FR-IMP-07`): upload to private bucket, build reading manifest, chunks (~500 tokens, 60 overlap, chapter-aligned, FTS vector), optional embeddings (DP-4); book stays `draft`.
- [ ] Covers (`FR-IMP-06`): verified rights or project-generated typographic cover; 300px WebP thumbnail.
- [ ] Idempotent upsert by `(source_name, source_identifier)`; per-item outcome codes in `import_items`; CSV+JSON report; non-zero exit on failure (`FR-IMP-08`).
- [ ] Rights gate enforced (`FR-IMP-02`, `T-IMP-02/03`): DB CHECK blocks `available` without gates; same rule in admin UI + import.
- [ ] Manifest-driven exactly-200 import (`FR-IMP-01`): refuses unreviewed rows; refuses ≠200 for initial batch.
- [ ] Final verification (`FR-IMP-09`): `verify` runs Table 12 checks (count=200, uniqueness, rights gates all rows, content objects present+checksums, covers decodable, retrieval data, category distribution ±3, reader smoke test on 10 random); exits 0 only if all pass.

**Risk:** rights uncertainty removes candidates (use reserves); source access issues (use official mirrors/feeds). **Start review early.**

---

## Phase 6 — Reading interface & progress tracking
**Objective:** integrated reader with progress and sessions. **Dependencies:** Phases 4 & 5 (real books).
**Completion criteria:** reading, resume, time accounting behave per spec on two devices.
**Verifying tests:** `T-RDR-01…10`, `T-NFR-02`, `T-NFR-07`.

- [ ] **DP-3 EPUB reader spike** (1 day on real deps) → choose library; document fallback (sanitised HTML bundle in WebView with same progress interface).
- [ ] Open book (`FR-RDR-01`): 1-hour signed URL (available books only), cache download, open at saved position; `T-RDR-01`, `T-DB-10`.
- [ ] Reader layout/navigation (`FR-RDR-02`): paginated/scroll, TOC drawer, prev/next, font size (5 steps), line spacing, light/sepia/dark; device-persisted prefs; titles wrap.
- [ ] Progress calculator from manifest (`FR-RDR-03`): `progress_percent` formula, clamp 0..100, monotonic `furthest_percent` server-side; `T-RDR-03/04`.
- [ ] Resume (`FR-RDR-04`): load `last_location` from server; later-timestamp-wins vs local; survives reinstall.
- [ ] Mark completed (`FR-RDR-05`): auto at furthest ≥98 or "Mark as finished"; "Read again" → in_progress without erasing history.
- [ ] Reading sessions (`FR-RDR-06`, Section 5.3.3): `start/heartbeat/end_reading_session`; client UUID idempotent; heartbeat 30s while active; server adds `min(now−last_heartbeat, 45s)`, ignores <10s, never exceeds wall-clock; one open session; 5-min stale close; `T-RDR-07/08`.
- [ ] Content-unavailable handling (`FR-RDR-07`): missing/corrupt/expired/archived/no-network → specific message + Retry/Back; auto-renew expired URL once.
- [ ] "Discuss with AI" shortcut from reader (`FR-RDR-08`): reuse latest conversation for this book or create one.
- [ ] Format support (`FR-RDR-09`): EPUB via reader lib; HTML bundle via WebView; PDF only if a catalogue book requires it.
- [ ] Intermittent-network behaviour (`T-NFR-07`): no crash, progress saved on reconnect, time not inflated.

**Risk:** EPUB library issues → HTML-bundle fallback.

---

## Phase 7 — AI integration & secure book retrieval 🔒🔴
**Objective:** grounded, isolated AI assistant. **Dependencies:** Phases 2, 5 (chunks), DP-1; DP-4 for embeddings.
**Completion criteria:** all High AI-security tests pass with stub **and** live integration.
**Verifying tests:** `T-AI-01…16`, `T-SEC-01…14`.

### 7.1 Adapter & function
- [ ] Provider-independent adapter `AiProvider.generate(input): output` around the existing integration (`SEC-AI-10`); contract test suite passes with integration or stub; swapping providers changes no data-flow rule; `T-SEC-14`.
- [ ] `ai-chat` Edge Function pipeline (Section 8.4): authenticate (user_id from JWT, never body) → validate (reject unknown fields, text 1–2000) → rate-limit → authorise (load conversation under caller JWT/RLS; 404 if none; book still available) → idempotency on `client_message_id` → retrieve → build prompt → generate (30s timeout, no tools/DB/secrets) → verify → persist via `append_chat_turn` → respond.
- [ ] Retrieval design (Section 8.5): book convs → `search_book_chunks(book_id, query)` ≤6; general convs → `search_books` ≤8 candidates; FTS baseline (`websearch_to_tsquery`, `ts_rank_cd`, min-rank threshold → empty when below); optional semantic/RRF if DP-4.
- [ ] Prompt structure (Section 8.4 template): fixed server policy → untrusted REFERENCE blocks in randomised delimiters → last 10 messages → user message; structured JSON output `{answer, citations[], books[]}`.

### 7.2 Chat UX
- [ ] AI landing (`FR-AI-01`): suggested prompts, New conversation, 5 recent, always-visible disclaimer.
- [ ] Create conversation (`FR-AI-02`): general = no book; book-bound requires existing `reading_progress` + available book; immutable binding; `T-AI-02/03`.
- [ ] Send/receive (`FR-AI-03`): 1–2000 chars, idempotent on `(conversation_id, client_message_id)`.
- [ ] General reading assistance (`FR-AI-04`): recommend only catalogue ids (server verifies), tappable book cards.
- [ ] Book-specific discussion (`FR-AI-05`): ≤6 passages (~3000 tokens), answer only from them, chapter-label citations.
- [ ] Insufficient-context behaviour (`FR-AI-06`): explicit "not in the text"; never fabricate quotes.
- [ ] History / continue (`FR-AI-07`): list newest-first paginated, load messages 30/page, context = last 10 of *that* conversation only.
- [ ] Failures/timeouts (`FR-AI-08`): server 30s timeout + 1 retry for transient; client 35s; failed message stored with status; retry reuses `client_message_id`; `T-AI-12/13`.
- [ ] Loading/error/rate-limit UI states (`FR-AI-09`).
- [ ] Rate limits/quotas (`FR-AI-10`): 12/10min + 150/day per user, 60/10min per IP, global daily budget cap; counters server-side, not client-resettable; `T-AI-15`.
- [ ] Optional personalisation (`FR-AI-11`, Could): off by default; when on, add only caller's completed titles + primary categories (≤20).

### 7.3 AI security (🔒 release-blocking)
- [ ] No PII solicitation (`SEC-AI-01`): policy + output filter; 30-prompt corpus passes; `T-SEC-01`.
- [ ] Restricted access layer (`SEC-AI-02`): outbound request contains only policy + user message + ≤10 own messages + (≤6 chunks | ≤8 catalogue items); inspection over 20 scenarios; `T-SEC-02`.
- [ ] Strict book scope (`SEC-AI-03`): book_id from stored conversation only; `T-SEC-03`.
- [ ] Cross-user isolation (`SEC-AI-04`): caller-JWT loads; 404 on another user's conversation id; `T-SEC-04/05`.
- [ ] Secret/system-prompt protection (`SEC-AI-05`): output scanner blocks key patterns/JWT-like/policy fragments; `T-SEC-06`.
- [ ] Prompt-injection resistance (`SEC-AI-06`): delimited untrusted blocks, randomised delimiter, import-time scanning; `T-SEC-07/08/09`.
- [ ] Response grounding & quote verification (`SEC-AI-07`): cited ids ⊂ retrieved set; every ≥6-word quote is a normalised substring; 1 regeneration then safe fallback; `T-SEC-10/11`.
- [ ] Out-of-scope requests (`SEC-AI-08`): brief refusal + redirect; `T-SEC-12`.
- [ ] AI audit & monitoring (`SEC-AI-09`): `ai_usage_events` (outcome, latency, flags); no raw prompts/secrets; alert on repeated flags; `T-SEC-13`.

**Risk:** highest-risk phase (provider limits, grounding quality, injection resilience). **Contingency week reserved.**

---

## Phase 8 — Saved books & reading history
**Objective:** personal library features. **Dependencies:** Phases 3 & 6.
**Completion criteria:** all user-specific features isolated by RLS tests.
**Verifying tests:** `T-USR-01…10`.

- [ ] Save/unsave (`FR-USR-01`): idempotent heart toggle; `T-USR-01/02`.
- [ ] Saved books list (`FR-USR-02`): own rows only; `T-USR-03`, `T-DB-04`.
- [ ] Reading history (`FR-USR-03`): view over `reading_progress` + `books`; In-progress/Completed filter.
- [ ] Personal reading dashboard (`FR-USR-04`, Should): totals, 7-day bar chart, in-progress list; `T-USR-05`.
- [ ] Progress privacy (`FR-USR-05`): RLS `user_id = auth.uid()`; `T-USR-06`, `T-DB-03`.
- [ ] Conversation ownership (`FR-USR-06`): owner-only RLS, no admin override; writes via `append_chat_turn`; `T-USR-07/08`, `T-DB-06/07`.
- [ ] Profile & account settings (`FR-USR-07`): edit display name/language/AI-history toggle/theme; change password; role read-only; `T-USR-09`, `T-DB-02`.
- [ ] Delete conversation (`FR-USR-08`, Should): own only; `T-USR-10`.

**Risk:** low.

---

## Phase 9 — Administrative dashboard & book management
**Objective:** Admin & Super Admin tooling. **Dependencies:** Phases 3, 5, 6.
**Completion criteria:** admin workflows pass; audit entries verified.
**Verifying tests:** `T-ADM-01…13`, `T-SA-01…09`.

### 9.1 Statistics & user analytics
- [ ] Dashboard overview (`FR-ADM-01`): KPI cards, top books, recent activity, one chart; `admin_overview`; User session cannot call it.
- [ ] User list (`FR-ADM-02`): paginated 25, search, sort; only permitted fields (Section 4); `admin_list_users`.
- [ ] User reading details (`FR-ADM-03`): identity summary + per-book rows; no tokens/chats/saved-books; `admin_user_detail`.
- [ ] Popular books & recent activity with date filters (`FR-ADM-04`, Should).
- [ ] Audited user-detail access (`FR-ADM-09`): each view writes `user.view` audit row readable by Super Admin only; `T-ADM-13`.

### 9.2 Book management
- [ ] Add book form (`FR-ADM-05`, Section 5.7 fields + validation): drafts, gates, "Make available" only when gates pass; via `admin-books` Edge Function + audit; `T-ADM-06/07/08`.
- [ ] Content & cover upload (`FR-ADM-06`): size/magic-bytes/EPUB-structure checks, sanitise, SHA-256, private bucket, chunks+manifest, rollback on failure; signed upload URLs; `T-ADM-09/10`.
- [ ] Edit/withdraw/archive (`FR-ADM-07`): re-validate on content replace; archived stays in history; audit before/after; no hard delete; `T-ADM-11`.
- [ ] Category management (`FR-ADM-08`, Should): add/rename/reorder/deactivate; unique immutable slugs; `T-ADM-12`.

### 9.3 Super Admin
- [ ] Role-management screen (`FR-SA-01`): unreachable for Admin/User at nav + server; `T-SA-01`.
- [ ] Grant/revoke (`FR-SA-02/03`) with reason ≤200 chars; immediate effect; `T-SA-02/03/04`.
- [ ] Privilege-escalation & last-super-admin protection (`FR-SA-04`): 5 prohibited ops fail; trigger holds against direct SQL; `T-SA-05/06`.
- [ ] Audit history (`FR-SA-05`, Should): read-only append-only list; `T-SA-07/08`, `T-DB-09`.
- [ ] Effect of privilege changes (`FR-SA-06`): live DB lookup; UI refresh within ~1 min; `T-SA-09`.

**Risk:** upload sanitisation complexity.

---

## Phase 10 — Security testing & quality assurance 🔒
**Objective:** verify controls and quality targets. **Dependencies:** Phases 2–9.
**Completion criteria:** no open blocker or high-severity security defect.
**Verifying tests:** all of Section 15.3; NFR validation.

- [ ] Run full RLS suite on staging (`T-DB-01…12`).
- [ ] Run full AI-security suite — stub **and** live adapter run before release (`T-SEC-01…14`); no test reported passed for live until actually run.
- [ ] Authz role matrix: each role × each RPC/function/screen.
- [ ] Dependency audit + secret scan (`T-DEV-05`); review new dependencies.
- [ ] Performance targets (Section 14): catalogue p95 ≤2.0s warm/≤3.5s cold (`T-NFR-01`); reader open p95 ≤4s/1MB (`T-NFR-02`); AI median ≤8s/p95 ≤20s/30s hard timeout (`T-NFR-03`); concurrency 50 users <1% error (`T-NFR-04`).
- [ ] Accessibility WCAG 2.1 AA core flows via TalkBack (`T-NFR-05`).
- [ ] Reliability/recoverability: no progress/message loss on transient failure (`T-NFR-07`); backup restore drill (`T-NFR-06`).
- [ ] Error-handling states (network off, 500, timeout, expired URL, rate limit).
- [ ] Triage defects: blocker / major / minor; close all blockers + high-severity security.

**Risk:** late discovery of design flaws → mitigated by running RLS (Phase 2) and AI (Phase 7) tests early.

---

## Phase 11 — AWS EC2 deployment & release preparation
**Objective:** deploy server components and build release. **Dependencies:** Phase 10; DP-5.
**Completion criteria:** smoke tests pass; release checklist signed.
**Verifying tests:** `T-DEP-01…05`, `T-DEV-04`, `T-NFR-06`.

### 11.1 EC2 companion service
- [ ] Provision: 1× t3.small, 30GB encrypted gp3, Elastic IP, Ubuntu LTS, Docker+Compose, Caddy/Nginx, CloudWatch agent; MFA on root; least-privilege IAM.
- [ ] Secure config (Section 16.3): SG inbound 443 (80 for cert only); SSH closed/restricted or SSM Session Manager; HTTPS via Caddy/Let's Encrypt + HSTS; secrets from Secrets Manager/SSM at container start (none in image/repo); non-root read-only container pinned by digest + image scan.
- [ ] Deploy companion service (import worker, health endpoint, optional AI adapter host for Pattern B, optional internal APK page).
- [ ] Deploy Supabase migrations + Edge Functions to production via CLI (after staging smoke tests).
- [ ] Logging (CloudWatch 30-day), monitoring (status checks, CPU credits, disk >80%, failed health), backups (daily EBS ×7, weekly logical dump, Supabase managed, quarterly restore drill).
- [ ] Smoke/network/rollback tests: `T-DEP-01` (health, import dry-run, ai-chat), `T-DEP-02` (TLS/HSTS/redirect), `T-DEP-03` (only 443 reachable, SSH refused), `T-DEP-04` (rollback ≤10min).

### 11.2 Mobile release
- [ ] Inject `SUPABASE_URL` + anon key at build time from per-env config excluded from Git.
- [ ] Create upload keystore once; store outside repo; configure signing from env.
- [ ] Build: `assembleRelease` (staging APK for testing) and `bundleRelease` (AAB, R8, version code incremented) → Play Internal Testing → closed → production.
- [ ] Release build on clean device (`T-DEP-05`): works, no debug flags, cleartext blocked, backups disabled.
- [ ] Pre-release checklist: all High tests pass; prod seed-email SQL gate = 0 (`T-DEV-04`); privacy info prepared; rollback drill done.

**Risk:** store review delays / signing mistakes → internal testing track + early signing setup.

---

## Phase 12 — Documentation & final validation
**Objective:** complete and validate deliverables. **Dependencies:** all phases.
**Completion criteria:** all acceptance criteria met and signed by supervisor.

- [ ] Setup guide (reproducible by a new dev in <½ day).
- [ ] Database guide (schema, policies, migration process).
- [ ] Security guide (controls, key rotation, incident steps).
- [ ] Testing guide (how to run each suite).
- [ ] Deployment runbooks (EC2 setup, mobile release, update, rollback).
- [ ] Recovery runbook (Super Admin recovery, restore drill).
- [ ] ADRs per major decision; architecture diagram sources included.
- [ ] Final acceptance run against Section 19 checklist; update spec to final version.

**Risk:** docs left to the end → write docs per phase.

---

## Critical path & scope-cut order

**Critical path:** DP-1 → adapter (Phase 7) → AI-security tests (Phase 10) → release (Phase 11).
**Parallel critical track:** rights review → manifest → import (Phase 5) → chunks → grounded retrieval (Phase 7).

**Schedule (baseline 4 + 1 weeks):**
| Week | Phases | Milestone |
|------|--------|-----------|
| 1 | 1, 2, 3 (4 & 5 start) | Architecture agreed, DP-1/DP-2 decided, DB+RLS in place, auth+roles working, rights review under way |
| 2 | 4, 5, 6 | Design system, catalogue, manifest frozen, import dry-run, reader with progress+sessions |
| 3 | 7, 8, 9 | AI grounded retrieval, saved/history, admin dashboard + book mgmt |
| 4 | 10, 11, 12 | Security testing, EC2 deploy, release build, docs, final import verification |
| 5 | Contingency | Absorbs slippage in AI security, legal import or release prep |

**Scope-cut order if time runs short** (supervisor approval; no cut may touch Must requirements, security controls, or the legal gate):
1. Account deletion (`FR-AUTH-09`) → 2. Personal dashboard (`FR-USR-04`) → 3. Category management (`FR-ADM-08`) → 4. Conversation deletion (`FR-USR-08`) → 5. History personalisation (`FR-AI-11`) → 6. Charts → 7. Semantic search.

---

## Acceptance criteria checklist

- [ ] **AC-01** All three roles authenticate with correct permissions. *(T-AUTH-03/07–10, T-DEV-01/02)*
- [ ] **AC-02** Catalogue contains 200 validated, legally importable books. *(T-IMP-01…10)*
- [ ] **AC-03** Users can search, open and read supported books. *(T-CAT-03/05, T-RDR-01/02/10)*
- [ ] **AC-04** Reading progress saved and restored correctly. *(T-RDR-03…08)*
- [ ] **AC-05** Saved books and reading history work. *(T-USR-01…06)*
- [ ] **AC-06** Users create/retrieve/continue their own AI conversations. *(T-AI-02/04/10, T-USR-07/10)*
- [ ] **AC-07** AI gives book-grounded responses; cannot access unauthorised user/admin data. *(T-AI-07…09, T-SEC-01…14)*
- [ ] **AC-08** Admins add books and view permitted statistics. *(T-ADM-01…13)*
- [ ] **AC-09** Super Admin grants/revokes Admin securely. *(T-SA-02…10)*
- [ ] **AC-10** DB policies prevent unauthorised cross-user access. *(T-DB-01…12, T-USR-06…08)*
- [ ] **AC-11** Automated + manual tests pass (High 100%, others ≥95%, no open security defects).
- [ ] **AC-12** Documented build and release process. *(T-DEP-05)*
- [ ] **AC-13** Required deployment components configured and tested. *(T-DEP-01…04, T-DEV-04)*
- [ ] **AC-14** Setup/DB/security/testing/deployment docs complete.

---

## Final deliverables checklist

- [ ] Source code (RN app, Edge Functions, companion service, scripts)
- [ ] Database migrations (0001+, RLS, storage, seed categories)
- [ ] Seed/verify/bootstrap scripts with env guards
- [ ] Import tool + frozen manifest with rights evidence + import report (200 verified)
- [ ] Test results (CI, pgTAP, AI-security stub+live clearly labelled, performance, accessibility)
- [ ] Configuration templates (`.env.example`, `config.toml`, proxy/container config; no secrets)
- [ ] Architecture diagrams (context, app, ER, authz flow, import, AI, AWS; editable sources)
- [ ] Deployment instructions + recovery runbook
- [ ] Documentation set (setup, database, security, testing, admin guide, ADRs)

---

## Completion Tracker

> Update after each work session. Per-phase % = done tasks ÷ total tasks in that phase.
> Overall % = weighted by task count (shown in the **Tasks** column). Round to the nearest whole number.

| Phase | Tasks | Done | % | Status |
|-------|-------|------|---|--------|
| 0 — Bootstrap | 8 | 5 | 63% | In progress (2 partial: RN/Supabase CLI = dev-machine step; 1 blocked: 3 Supabase projects need your account) |
| 1 — Requirements & architecture | 6 | 0 | 0% | Not started |
| 2 — Supabase & database | 16 | 0 | 0% | Not started |
| 3 — Auth & authorisation | 15 | 0 | 0% | Not started |
| 4 — Design system & navigation | 8 | 0 | 0% | Not started |
| 5 — Catalogue & legal import | 20 | 0 | 0% | Not started |
| 6 — Reading interface & progress | 11 | 0 | 0% | Not started |
| 7 — AI integration & retrieval | 24 | 0 | 0% | Not started |
| 8 — Saved books & history | 8 | 0 | 0% | Not started |
| 9 — Admin & Super Admin | 15 | 0 | 0% | Not started |
| 10 — Security testing & QA | 10 | 0 | 0% | Not started |
| 11 — EC2 deploy & release | 10 | 0 | 0% | Not started |
| 12 — Documentation & validation | 8 | 0 | 0% | Not started |
| **Decision points (DP-1…5)** | 5 | 0 | 0% | Artifacts prepared for DP-1 & DP-2 (checklists + ADR stubs); decisions await human input |
| **TOTAL** | **164** | **5** | **3%** | **In progress** |

### Milestone gates (must all pass for release)
- [ ] DP-1 and DP-2 recorded (spec v1.1)
- [ ] pgTAP suite green; anon access denied everywhere
- [ ] 8 seeded accounts log in with correct roles
- [ ] `verify` prints PASS with exactly 200 available books
- [ ] All High AI-security tests pass with stub **and** live adapter
- [ ] No open blocker or high-severity security defect
- [ ] Deployment smoke tests pass; release checklist signed

---

### 📊 Overall project completion: **3%**

*Stage 0 repo/tooling scaffold is in place (git initialised, Section 12 tree, CI with active secret-scan, ADRs). The two day-one parallel tracks are prepared and waiting on human input: DP-1 needs the backend lead to run the AI inspection checklist; DP-2 + the rights review need legal confirmation and two reviewers. Next machine-side steps: generate the native RN project + `npm install`, install Supabase CLI/Docker and `supabase start`, then begin Phase 2 migrations.*

<!--
How to recompute overall %:
  overall = (sum of done tasks across all phases + DP) / 164 * 100
Update each phase row's Done and %, then the TOTAL row, then the headline number above.
-->
