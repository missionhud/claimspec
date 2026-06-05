# Governance

ClaimSpec is stewarded by **Atmosphérique Pty Ltd** under the **Mission HUD** brand.
This document describes how the specification evolves.

## Principles

- **Thin and load-bearing.** ClaimSpec is a semantic data contract, not an engine.
  Proposals that push reasoning, storage, ML, or UI into the spec are out of scope.
  A standard is the easiest thing to over-engineer; we keep it minimal.
- **Profile, don't reinvent.** Where an established standard exists (W3C PROV,
  in-toto/SLSA, OpenLineage), ClaimSpec aligns with it and standardizes only the
  novel epistemic layer.
- **Forever Backwards Read.** Any tool that can read a vN document can read every
  future vN.x document. Breaking changes require a new major version.

## RFC process

Substantive changes go through a lightweight RFC:

1. Open an issue using the **RFC** template (`.github/ISSUE_TEMPLATE/rfc.yml`).
2. Discussion in the open. A maintainer assigns `accepted` / `declined` / `deferred`.
3. Accepted RFCs land as a PR against the next `spec/vN/` working draft, with a
   `CHANGELOG.md` entry and (where applicable) conformance test updates.

## Versioning policy

- **Semver.** `0.x` is pre-stable; the schema may change between alphas.
- **Major versions** are separated by long, deliberate intervals (≥ 18 months target,
  matching AppSpec) with published migration helpers.

## Stability & open governance milestone

ClaimSpec begins as a single-steward open standard. Once there are **3+ independent
implementers**, governance will migrate to a **W3C Community Group** (the same path as
AppSpec), with a published charter and decision process.

## Maintainers

- Atmosphérique Pty Ltd (Mission HUD). Contact via `SECURITY.md` for security matters
  and the issue tracker for everything else.
