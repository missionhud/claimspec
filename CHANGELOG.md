# Changelog

All notable changes to ClaimSpec are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/); the spec uses Semantic Versioning.

## [0.1.0-alpha] — 2026-06-05

Initial public draft. **Not stable** — the schema may change between alphas.

### Added
- **The core move:** everything is a claim with a `confidence` and a `register`
  (`record` | `inference`).
- **The never-1.0-inference invariant** and the **phase-transition guardrail**
  (records supersede; registers don't graduate by confidence) — the central safety
  properties.
- **Object ontology:** records (`observation`, `evidence`), the inference maturity
  ladder (`hypothesis`→…→`law`), and register-less prompts.
- **Provenance spine:** event-as-truth ledger, the three-axis Actor model
  (mechanism / agency / identity), and the two input edges (`derived_from` / `drew_on`).
- **Disclosure** (egress closure-walk): AI-involvement disjunction (sticky-upward),
  external-marks union, sources union.
- **Composition** with W3C PROV, in-toto/SLSA, OpenLineage; **compliance mapping** to
  EU AI Act Art. 12 / Art. 50.
- `spec/v0/spec.md`, `spec/v0/schema.json` (JSON Schema 2020-12), an Extended-tier
  example, dual licensing (CC-BY-4.0 docs / MIT code), and governance scaffolding.

### Provenance
- Derived from the Mission HUD Discovery claim/register + provenance-spine design,
  extracted into a vendor-neutral standard (sibling to AppSpec).
