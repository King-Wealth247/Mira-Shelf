# DP-2 — Distribution Jurisdictions & Rights Policy Decision Record

> **Owner:** Project owner / legal contact. **Needed by:** end of Phase 1. **Blocks:** Phase 5 import.
> This is a project working rule, **not legal advice** — confirm with a qualified legal contact for
> the intended distribution jurisdictions **before import begins**. Record the outcome in
> [ADR-0003](adr/0003-dp2-rights-policy.md).

## 1. Target distribution jurisdictions (A-04)

- Confirmed jurisdiction(s): _______________
- Confirmed by: _______________  Date: __________

## 2. Conservative rights policy (Section 7.2) — adopt / adjust

| Rule | Requirement (default) | Adopted as-is? | Adjustment / notes |
|------|-----------------------|----------------|--------------------|
| R-1 Underlying work | Author died ≥76 yrs before import year (before 1950 for 2026) **and** first published ≤1930 | ☐ | |
| R-2 Translations | Translator meets R-1, or explicit CC0/public-domain dedication | ☐ | |
| R-3 Added material | Intros/notes/typesetting only with CC0/PD dedication, else removed/replaced | ☐ | |
| R-4 Illustrations | Kept only if illustrator/artwork meets R-1 or has dedication, else stripped | ☐ | |
| R-5 Covers | Source cover only if artwork + composition rights verified, else project-generated | ☐ | |
| R-6 Open licences | CC BY etc. only if copying/storage/redistribution permitted, attribution shown, share-alike acceptable | ☐ | |
| R-7 Unverifiable | If any element unverifiable, do not import that edition | ☐ | |

## 3. Approved sources (Section 7.3)

- [ ] Project Gutenberg — follow automated-access guidance (official feeds/mirrors, not page crawling)
- [ ] Standard Ebooks — record cover artwork credit from colophon
- [ ] Other repositories (Wikisource, Internet Archive, national libraries) — per-title explicit evidence

## 4. Sign-off

- Project owner: _______________  Date: __________
- Legal contact: _______________  Date: __________

**Gate reminder:** the DB `books_available_gate` CHECK enforces the result technically — a book cannot
become `available` without verified rights, verified/generated cover, content file, format, checksum,
manifest and ≥20-char evidence. Verified by `T-IMP-02`, `T-IMP-03`.
