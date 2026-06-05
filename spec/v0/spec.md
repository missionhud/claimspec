# ClaimSpec v0 — Specification

**Version:** v0.1.0-alpha · **Schema:** [`schema.json`](./schema.json) (JSON Schema Draft 2020-12)
**Status:** under active design; not stable. **License:** CC-BY-4.0 (this document).

---

## 1. Scope & design principles

ClaimSpec defines the canonical shape and invariants of four things:

1. **Claim** — a unit of knowledge with a `confidence` and a `register`.
2. **Event** — one act that happened (the lineage ledger; the source of truth).
3. **Actor** — who acted, along three orthogonal axes.
4. **Disclosure** — the read-model egress publishes (the closure-walk projection).

Design principles:

- **From data toward theory.** ClaimSpec reasons the way the scientific method does — observe, infer, mature, supersede. *Belief* is not in the vocabulary.
- **Honesty over confidence.** The model makes false certainty *structurally impossible* (§3).
- **Event-as-truth.** The ledger of acts is authoritative; everything else is a projection (§6).
- **Profile, don't reinvent.** The provenance layer aligns with W3C PROV, in-toto/SLSA, and OpenLineage (§9). The novel contribution is the *epistemic* layer.
- **Model, not engine.** Reasoning, storage, and UI are conformant implementations, not part of the spec.

---

## 2. The Claim

A **Claim** is an object in the knowledge graph:

| Field | Type | Notes |
|---|---|---|
| `id` | string | stable, unique identifier |
| `type` | string | the object type (§5); governs allowable `register` |
| `register` | `"record"` \| `"inference"` | what grounds the confidence (§3) |
| `confidence` | number `[0, 1]` | how sure; subject to the invariant in §3 |
| `content` | object | the claim's payload (domain-defined; opaque to the spec) |
| `agency` | `"human"` \| `"ai"` \| `"hybrid"` | responsibility axis (projected from events; §6) |
| `superseded_by` | string \| null | the `id` that supersedes this claim, if any (§4) |
| `derived_from` | string[] | predecessor *versions* (the version DAG; §6.3) |
| `drew_on` | string[] | sources *cited / composed* (§6.3) |
| `created_by` | Event ref | the genesis event (§6) |

`agency`, `superseded_by`, and `confidence` are **projections** — recomputable by replaying the claim's event stream (§6). They are a cache the ledger can always rebuild; nothing on the Claim is independently authoritative.

---

## 3. The two registers, and the invariant

### `record` — constitutive truth
True because it was **observed, recorded, or decided**. Confidence is *trust in provenance* — a first-party record (a payment that occurred, a decision that was made, a measurement taken) is legitimately `1.0`. Records are **WORM**: superseded, never edited. In the engine they are *inert evidence* — reasoned **from**, never **about**.

### `inference` — empirical truth
True because it is **inferred from evidence**. Confidence is *proportional to evidence* and lives in **`(0, 1)` forever** — it asymptotes toward, but never reaches, `1.0`. Inferences are **revisable** — matured or challenged as evidence accrues.

### The invariant (normative)

> A Claim with `register: "inference"` **MUST** have `confidence < 1.0`.
> Equivalently: `register == "record" OR confidence < 1.0`.

This is the **seduction-of-certainty** guard. "We're 99.9% sure X" must never be encoded as certainty; only what we *author or record* can be known with certainty.

### The phase-transition guardrail (normative)

> A Claim changes `register` **only by being recorded** — a phase transition — **never by accumulating `confidence`.**

This is the **risk-vs-issue** rule: a risk becomes an issue by *happening*, not by its probability creeping to 99%. An inference at `0.999` is still an inference; it does not *graduate* into a record. To cross registers, a new **record** event must occur (an observation, a decision). Different registers are different *kinds of thing*, not two readings on one slider.

> **Why categorical, not a float:** if `record` were merely `confidence = 1.0`, someone eventually sets an empirical inference to `1.0` and false certainty leaks in. Making `register` a *type* makes that impossible — and is the spec's central safety property.

---

## 4. Decisions are records that supersede

A human decision ("ship variant B") is a `record` — it *constitutively happened*. A later decision ("revert to A") is a **new** record that **supersedes** the first in the projection of *what is in force now*; it never edits or falsifies the original. This is event sourcing: **immutable history, provisional present.**

There is **no special `decision` type.** A recorded payment, a filed document, and an authored decision are all `record` claims, differentiated by **provenance** (where from) and **agency** (human / ai / system) — never by a bespoke type. Fact-ness lives in `register`, not the type catalog.

---

## 5. The object ontology

`type` is open and extensible (domain vocabularies may add types via the extension mechanism, analogous to AppSpec library descriptors). The **core ontology**, sorted by truth-maker:

**Records** (`register: record`, may reach `1.0`):
- `observation` — something observed/recorded/decided (the generic record).
- `evidence` — a recorded artifact cited in support of inferences.

**Inferences** (`register: inference`, `(0,1)` forever) — a *maturity ladder*:
- `hypothesis` → `pattern` → `insight` → `theory` → `principle` → `law`
  (increasing evidential support and generality; **never** certainty).

**Prompts** (register-less — they ask, they don't assert):
- `question`, `gap`, `contradiction`, `assumption`, `risk`.

Implementations **MUST** enforce: record-types carry `register: record`; inference-ladder types carry `register: inference`; prompt types carry no `register`. Domain extensions **MUST** declare which register (if any) their types take.

---

## 6. The provenance spine

### 6.1 One law: event-as-truth

> **The event ledger is the truth; Claims are a projection over it.**

`events` is an append-only ledger of **acts**. A Claim's provenance stamp (`confidence`, `agency`, `superseded_by`, `register`) is a **read-model recomputable by replaying its event stream.** This is the same event-sourcing law (immutable history, provisional present) applied to provenance.

```
   events  (append-only ledger = TRUTH; one row per ACT)
      │  replay / project
      ▼
   Claims  (projection / cache: confidence · register · agency · superseded_by)
      │  walk closure at publish
      ▼
   Disclosure manifest  (egress read-model: ai-involved? · external marks · sources)
```

### 6.2 Events — five families

Every event belongs to one of five **behavioural families**: `genesis` · `production` · `epistemic` · `relational` · `lifecycle`. Two acts are first-class for the model: `superseded` (§4) and `egress_disclosed` (§7). An Event carries: `id`, `family`, `type`, `subject` (the Claim id acted on), `actor` (§6.4), `inputs` (§6.3), and `at` (timestamp).

### 6.3 Two input edges

`inputs` is two distinct edges that the egress closure-walk must tell apart:

- **`derived_from`** — my predecessor *version* (the version DAG).
- **`drew_on`** — the sources I *cited or composed*.

### 6.4 Actors — three orthogonal axes

A single "actor type" was secretly doing three independent jobs; they vary independently (an AI agent drafts → a human edits → agency becomes `hybrid` while the mechanism stays `ai_agent`), so they cannot collapse into one enum:

| Axis | Field | Answers | Drives |
|---|---|---|---|
| **Mechanism** | `actor_type` (`user` / `ai_agent` / `system`) | *how* it ran | (operational) |
| **Agency** | `actor_agency` (`human` / `ai` / `hybrid`) | *who is responsible* | **EU AI Act Art. 50** |
| **Identity** | `actor_id` (+ `model_used`) | *which specific* actor | ownership / audit (**Art. 12**) |

---

## 7. Disclosure (egress closure-walk)

Agency is **recorded per act (local)** but **disclosed per egress (global)**. When a Claim is published, the egress walks its full transitive closure and resolves:

- **AI-involvement** = the **disjunction** of all `agency` in the closure — **sticky-upward**: a later human act never launders an AI ancestor. *"Human-assembled" ≠ "human-authored."*
- **External marks** = the **union** of marks across the closure, preserved intact.
- **Sources / review-refs** = the **union** (Art. 12 attribution; supports the Art. 50(4) exemption).

A `Disclosure` manifest is the read-model of this walk for a published artifact.

---

## 8. Conformance tiers

- **Core** — §2 Claim with `id`, `type`, `register`, `confidence`; enforces the §3 invariant.
- **Standard** — Core + the §5 ontology + the §6.3 lineage edges.
- **Extended** — Standard + the §6 event ledger + the §6.4 three-axis Actor model + §7 Disclosure + §9 compliance mapping.

A document declares its tier; validators check accordingly.

---

## 9. Composition with existing standards

ClaimSpec's provenance layer is a **profile** that aligns with — and should interoperate with — established standards. It standardizes only the *epistemic* layer on top.

- **W3C PROV (PROV-O):** Claim ↔ `prov:Entity`, Event ↔ `prov:Activity`, Actor ↔ `prov:Agent`; `derived_from` ↔ `prov:wasDerivedFrom`; `drew_on` ↔ `prov:used`; the genesis event ↔ `prov:wasGeneratedBy`. A ClaimSpec graph SHOULD be expressible as PROV.
- **in-toto / SLSA:** a `record` of type `evidence` MAY carry or reference a signed in-toto attestation (e.g., SLSA build provenance) as its provenance content — binding the epistemic layer to supply-chain provenance.
- **OpenLineage:** for data-derived claims, `drew_on` edges SHOULD be reconcilable with OpenLineage dataset lineage.

**What is NOT in those standards, and is ClaimSpec's contribution:** `register` (record/inference) as a type; the never-1.0-inference invariant; the phase-transition guardrail; the inference maturity ladder; and the AI-agency *sticky-upward* disclosure model.

---

## 10. Compliance mapping (informative)

- **EU AI Act Art. 12 (record-keeping).** The append-only event ledger is the *automatic, tamper-evident, lifetime* record the article requires; the Identity axis (`actor_id` + `model_used`) plus the human approver recorded on the relevant decision event satisfy "which model" and "who verified." Retention ≥ 6 months (longer where other law applies).
- **EU AI Act Art. 50 (transparency / AI disclosure).** The Agency axis plus the §7 sticky-upward AI-involvement disclosure provide per-artifact disclosure of AI involvement; the union of sources/review-refs supports the Art. 50(4) exemption.

This mapping is informative; a conformant implementation's auditability is established against its own risk assessment, not by ClaimSpec alone.

---

## Appendix A — normative keywords

The keywords **MUST**, **MUST NOT**, **SHOULD**, and **MAY** are used per RFC 2119. The normative invariants are: §3 (the never-1.0-inference invariant), §3 (the phase-transition guardrail), §4 (records supersede, never edit), §5 (register-by-type enforcement), and §6.1 (event-as-truth).
