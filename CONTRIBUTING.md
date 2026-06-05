# Contributing to ClaimSpec

Thanks for your interest. ClaimSpec is an open standard for claims, confidence, and
provenance. We welcome implementers, reviewers, and proposers.

## Ways to contribute

- **Implement it.** Build a reader/writer/validator and tell us where the spec is
  unclear or wrong. Real implementations are the best feedback — and 3+ independent
  ones trigger the move to open W3C-CG governance (see `GOVERNANCE.md`).
- **Propose a change.** Open an **RFC** issue (`.github/ISSUE_TEMPLATE/rfc.yml`).
- **Report a defect.** Ambiguity, contradiction, or a schema/example mismatch — open
  an issue.

## What's in scope vs out

- **In scope:** the data model, invariants, the object ontology, the provenance/actor
  model, disclosure, alignment with PROV/in-toto/OpenLineage, conformance tests.
- **Out of scope:** reasoning algorithms, storage, ML, UI, domain-specific type
  catalogs (those belong in conformant *implementations* or domain extensions).

## Ground rules

- **Keep it thin.** Justify additions against the "thin and load-bearing" principle.
- **Don't break Forever Backwards Read** without a major version + migration notes.
- **Preserve the invariants** in `spec.md` §3–§6.1 — they are the safety properties.
- Update `schema.json`, a `conformance/` example, and `CHANGELOG.md` with any change.

## Licensing of contributions

By contributing you agree your spec contributions are licensed CC-BY-4.0 and code
contributions MIT, consistent with this repository's dual license.
