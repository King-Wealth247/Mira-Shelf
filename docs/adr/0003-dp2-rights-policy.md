# ADR-0003: DP-2 — distribution jurisdictions and conservative rights policy

- **Status:** Proposed (PENDING — awaiting project owner + legal confirmation)
- **Date:** <to fill>
- **Deciders:** Project owner / legal contact (owner), project team
- **Related:** DP-2; Spec Section 7 (7.2 R-1..R-7); A-04; `docs/DP2-rights-policy-decision.md`

## Context

Only books whose redistribution rights are verified per specific digital edition may be imported.
Public-domain status varies by jurisdiction. Until confirmed, the conservative rights rule in
Section 7.2 applies. This decision gates the Phase 5 import of 200 books.

## Decision

**PENDING.** To record:
- The confirmed list of target distribution jurisdictions (A-04).
- Whether the conservative policy (R-1…R-7) is adopted as-is or adjusted, with legal sign-off.

Default working rule until confirmed (Section 7.2): author died ≥76 years before import year (died
before 1950 for 2026 imports) **and** first published 1930 or earlier (US public domain as of
2026-01-01); translations/added material/illustrations/covers per R-2…R-5; CC BY etc. per R-6;
if any element is unverifiable, do not import (R-7).

## Consequences

- Narrower/wider jurisdictions change which titles are eligible, not the process.
- The DB legal gate (`books_available_gate` CHECK) enforces the result technically.

## Alternatives considered

Recorded once legal confirmation is captured in `docs/DP2-rights-policy-decision.md`.
