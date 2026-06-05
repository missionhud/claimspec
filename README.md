# ClaimSpec

**The open standard for claims, confidence, and provenance — the data model of the scientific method.**

ClaimSpec is a vendor-neutral interchange format for **what is known, how sure, and what grounds it** — designed so AI agents, knowledge engines, software-delivery tools, and BI/analytics systems can reason about truth and provenance in one consistent language, **without false certainty and without vendor lock-in.**

It is a sibling to [AppSpec](https://github.com/missionhud/appspec): *AppSpec carries design intent over a provenance foundation; ClaimSpec carries knowledge claims over the same foundation* — adding the epistemic layer AppSpec doesn't need.

> Status: **v0.1.0-alpha.2** — schema under active design. Not yet stable. Published as the shared reference for early implementers.

---

## The one move

> **Everything is a claim with a `confidence` and a `register`.**

- **`confidence`** — *how sure* (one axis, `0 → 1`).
- **`register`** — *what grounds the confidence* (categorical): `record` or `inference`.

ClaimSpec reasons the way science does — **from data toward theory**. The word *belief* is deliberately avoided; it smuggles in faith. The vocabulary is **claim · record · inference**.

| | `record` | `inference` |
|---|---|---|
| True because… | it was **observed / recorded / decided** | it is **inferred from evidence** |
| `confidence` = | trust in **provenance** (first-party = `1.0`) | proportional to **evidence** |
| Ceiling | may reach **`1.0`** | lives in **`(0, 1)` forever** — asymptotes, never `1.0` |
| Mutability | **WORM** — superseded, never edited | revisable — matured or challenged over time |
| Role | **inert evidence** — reasoned *from* | the live graph — reasoned *about* |

**The load-bearing guardrail:** *a claim changes register **only by a phase transition — being recorded — never by accumulating confidence.*** A risk at 99% is still a risk; it becomes an issue by **happening**, not by its probability creeping toward 1. An inference at 0.999 is still an inference — it never graduates into a record. Different registers, not two readings on one slider. (Making `register` a *type*, not `confidence = 1.0`, makes false certainty structurally impossible.)

---

## Why it exists

Every tool in the modern pipeline drops context at the handoff: a build knows *why*, then hands over a diff; CI hands over an artifact; production knows only metrics. The same loss happens in research and analytics — a confident dashboard that has forgotten what it was a claim *about*. ClaimSpec is the thread that survives the handoff: a claim carries its **confidence, its register, and its full lineage** wherever it goes, so downstream actors (and auditors) can always answer *"how do we know this, and how sure are we?"*

It is also a **governance substrate**. An append-only ledger of claims and the acts that produced them is, by construction, the **automatic, tamper-evident, lifetime record** that high-risk AI regulation now demands (EU AI Act Art. 12), with first-class **AI-attribution** (Art. 50) via a three-axis actor model.

---

## What ClaimSpec is — and is not

- **It is** a *semantic data contract*: the shape of a Claim, the lineage Events that produce it, the Actors that act, and the Disclosure manifest that egress publishes. Plus the **invariants** that keep it honest.
- **It is not** an engine. Reasoning algorithms, storage, ML, and UI are *conformant implementations*, not part of the spec — exactly as AppSpec is the format, not a renderer. (Reference engines: [Mission HUD Discovery](https://missionhud.com) for the full Learn core; a lean embeddable model for tools that only need claims + provenance.)

### Composes with existing standards (it does not reinvent them)

ClaimSpec's **provenance layer is a profile of, and aligns with:**
- **W3C PROV** (PROV-O) — Object ↔ Entity, Event ↔ Activity, Actor ↔ Agent; `derived_from`/`drew_on` ↔ `wasDerivedFrom`/`used`.
- **in-toto / SLSA** — a `record` may carry or reference a signed attestation as its provenance evidence.
- **OpenLineage** — for data-derived claims.

The **genuinely new contribution** ClaimSpec standardizes is the **epistemic layer**: `register` (record/inference), confidence-that-can-never-falsely-reach-1.0, the phase-transition guardrail, the inference maturity ladder, and the AI-agency *sticky-upward* disclosure model.

---

## Repository

```
spec/v0/
  spec.md          # the human-readable specification + rationale (start here)
  schema.json      # canonical JSON Schema (Draft 2020-12)
  conformance/     # example documents + (planned) test suite
GOVERNANCE.md      # RFC process, stewardship, migration policy
CONTRIBUTING.md
CHANGELOG.md
LICENSE-CC-BY-4.0  # the specification documents
LICENSE-MIT        # reference implementation code
```

## Conformance tiers

- **Core** — a Claim with `id`, `type`, `register`, `confidence`, and the *never-1.0-inference* invariant.
- **Standard** — Core + the full object ontology + lineage edges (`derived_from`, `drew_on`).
- **Extended** — Standard + the event-sourced ledger + the three-axis Actor model + the Disclosure manifest (egress closure-walk) + the Art-12/50 mapping.

## Versioning

Semver with a **Forever Backwards Read** commitment: any tool that can read ClaimSpec v0 will read every future v0.x document. Major versions are separated by long, deliberate intervals with published migration helpers.

## Licensing

Dual-licensed (the W3C / OpenAPI pattern): **specification documents under CC-BY-4.0**, **reference code under MIT**.

## Stewardship

Published by **Atmosphérique Pty Ltd** under the **Mission HUD** brand. External implementations welcome; a migration to W3C Community Group governance is planned at the **3+ independent implementers** milestone. See `GOVERNANCE.md`.
