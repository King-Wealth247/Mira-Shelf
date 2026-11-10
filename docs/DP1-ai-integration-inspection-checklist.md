# DP-1 — Existing AI Integration Inspection Checklist

> **Owner:** Backend lead. **Needed by:** end of Phase 1. **Blocks:** Phase 7 (AI).
> Source: Appendix B of the *Cahier des Charges*. This document does **not** know the answers and
> does not claim the integration has been tested. Fill the **Result** column, then record the
> outcome in [ADR-0002](adr/0002-dp1-ai-integration-pattern.md).

- **Inspected by:** _______________  **Date:** __________
- **Integration name / location:** _______________

| # | Item to inspect | Why it matters | Result (fill in) |
|---|-----------------|----------------|------------------|
| 1 | Provider, model family, API style (HTTP API / SDK / internal service) | Determines Pattern A or B | |
| 2 | Where it runs & how reached (public endpoint / private network / local module) | Network exposure & hosting | |
| 3 | How credentials are stored & who can read them | Must never reach the app or DB | |
| 4 | Request/response formats; system-prompt & structured (JSON) output support | Needed for citation verification | |
| 5 | Tool/function calling, memory, retrieval features enabled | **Must be disabled/unused — SEC-AI-02** | |
| 6 | Any direct database or file access the integration already has | **Must be removed for this use** | |
| 7 | Context window & max output length | Sizing of passages & history | |
| 8 | Latency, rate limits, quotas, pricing | Timeouts, budget cap, rate limits | |
| 9 | Data retention, logging, training-use policy of the provider | Privacy statement & legal review | |
| 10 | Existing prompts, guardrails, filters | Reuse where compatible; do not rely on them | |
| 11 | Existing tests & monitoring | Reuse for the adapter contract suite | |
| 12 | Licensing / contractual limits on use in this app | Compliance | |

## Decision (choose one, then update ADR-0002)

- [ ] **Pattern A** — direct from Edge Function (HTTPS API/SDK usable from Deno, server-held key).
- [ ] **Pattern B** — via EC2 AI adapter (needs Node/Python runtime, long-running process, private
      network, or unavailable libraries). Key only on EC2; Edge Function holds adapter shared secret only.
- [ ] **Replacement via change request** — only if SEC-AI-02…SEC-AI-07 cannot be met.

**Gate reminder:** whatever the pattern, the adapter must be a pure text-in/text-out call with no
tools, no DB client, no credentials and no URLs to fetch (SEC-AI-02). Verified by `T-SEC-02`, `T-SEC-14`.
