# scripts/ — operator tools

Run from a developer machine or the EC2 companion host with the **service-role key supplied
through environment variables** — never from the mobile app, never committed.

| Tool | Phase | Purpose |
|------|-------|---------|
| `bootstrap-super-admin/` | 3 | One-time Super Admin creation; aborts if one exists (unless `--recovery`); audits `role.bootstrap`. |
| `seed-dev-users/` | 3 | Create 5 users + 2 admins + 1 super admin (local/isolated only); idempotent; four production guards. |
| `verify/` | 3, 5 | `seed:verify` (roles) and import `verify` (200-book final checks). |
| `import-books/` | 5 | Manifest-driven import: fetch, validate, sanitise, normalise, dedupe, store, chunk, publish. |
| `manifest/` | 5 | Frozen `approved_books.csv` + rights-evidence checklist template. |

All scripts must call an `assertDevelopmentEnvironment()` guard before any network call where they
could touch a non-local project (Section 10.4).
