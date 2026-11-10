# ADR-0002: DP-1 — existing AI integration reuse pattern

- **Status:** Proposed (PENDING — awaiting backend-lead inspection)
- **Date:** <to fill>
- **Deciders:** Backend lead (owner), project team
- **Related:** DP-1; Spec Sections 8.4, 6 (SEC-AI-02..07), Appendix B; `docs/DP1-ai-integration-inspection-checklist.md`

## Context

The project reuses an existing AI integration whose provider, API, hosting and credential handling
are not yet known to the specification. Before Phase 7, the backend lead must inspect it (Appendix B
checklist) and decide how it is wired in. Two patterns are specified; the data-flow rules are
identical for both.

## Decision

**PENDING.** To be set to one of:
- **Pattern A — direct from Edge Function:** integration exposes an HTTPS API/SDK usable from Deno with
  a server-held key. Provider key in Supabase function secrets.
- **Pattern B — via EC2 AI adapter:** integration needs a Node/Python runtime, long-running process,
  private network, or libraries unavailable in Edge Functions. Provider key only on EC2; the Edge
  Function holds only the adapter shared secret; adapter has no database access.
- **Replacement via change request** — only if inspection shows the integration cannot meet
  SEC-AI-02…SEC-AI-07.

## Consequences

- Determines where the AI key lives and the `ai-chat → adapter` call path.
- The `AiProvider.generate(input): output` adapter interface is implemented regardless, with a
  contract test suite (`T-SEC-14`).

## Alternatives considered

Recorded once the inspection (`docs/DP1-ai-integration-inspection-checklist.md`) is complete.
