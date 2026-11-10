# manifest/ — frozen import manifest & rights evidence

## `approved_books.csv`

The frozen source of truth for the Phase 5 import. **Exactly 200** approved rows plus ordered
reserves. Its SHA-256 is recorded in `import_batches` at import time; later changes require a new
manifest version and re-review of changed rows.

Columns (Section 7.5): `manifest_version, seq, source_name, source_identifier, source_url, file_url,
title, authors, language, primary_category, secondary_categories, publication_year, license_type,
rights_status, rights_evidence, rights_verified_by, rights_verified_at, cover_policy, cover_source_url,
reviewer_1, reviewer_2, reserve_rank`.

Example row (illustrative only — **not** a verified entry):

```
1,0001,standard_ebooks,EXAMPLE-SLUG,https://example.org/book,https://example.org/book.epub,Example Title,"Example Author",en,Fiction,"Mystery and Adventure",1900,public_domain,verified_public_domain,"Author d.1899; first pub.1900; no added material",A.Reviewer,2026-11-02,generated,,A.Reviewer,B.Reviewer,
```

## Category targets (must total 200; ±3 tolerance per category, every category represented)

| Category | Target | | Category | Target |
|----------|-------:|-|----------|-------:|
| Fiction (novels, short stories) | 35 | | Education and Learning | 15 |
| Mystery and Adventure | 15 | | Personal Development | 20 |
| Literature (poetry, drama, essays) | 20 | | Biography and Memoir | 15 |
| Philosophy | 20 | | Economics and Society | 10 |
| History | 20 | | Technology and Engineering | 10 |
| Science | 20 | | **Total** | **200** |

## Rights evidence (per title)

Two reviewers independently complete the checklist in `rights-evidence-checklist.md` per title.
Disagreements are resolved by the project owner. Approved rows get `rights_status = verified_*`.
