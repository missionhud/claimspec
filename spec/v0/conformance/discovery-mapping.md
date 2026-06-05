# Conformance mapping — Mission HUD Discovery ↔ ClaimSpec v0

_The first conformance exercise. Discovery is the **source** the spec was extracted from, so this is reconciliation + stress-test, not a rebuild. Verdict: **v0 holds**, with one field to add (`review_ref`), one clarification (`family` is derived), and confirmation that the lean-core-plus-extensions design is right._

Discovery schema refs: `supabase/migrations/00001_initial_schema.sql` (objects), `20250721…register_field.sql` (register), `20250718…citations_lineage.sql` (lineage_events), `20250722…provenance_agency.sql` (3-axis actor + edges + review_ref), `20250723…egress_disclosure.sql` (disclosure).

---

## What matches exactly (no change needed)

- **`register`** (`record` | `inference`) and the **invariant** — Discovery's `discovery_objects_inference_not_certain` CHECK is `register = 'record' OR confidence_score < 1.0`, identical to ClaimSpec §3. ✅ The central safety property is byte-for-byte the same.
- **The 3-axis actor model** — Discovery's `20250722` migration adds `actor_agency (human|ai|hybrid)` alongside `actor_type (user|ai_agent|system)` and `actor_name`/`model_used`, with the *sticky-upward* hybrid rule documented in the migration itself. ✅ = ClaimSpec §6.4 + §7.
- **Event-as-truth** — `lineage_events` is "the append-only PRODUCTION-ACT LEDGER"; the object stamp is a projection. ✅ = §6.1.
- **The two input edges** — `derived_from` (predecessor version) vs `input_object_ids`/`input_citation_ids` (sources drawn on). ✅ = §6.3.
- **Disclosure** — `published_surfaces.disclosure` JSONB with floor fields `aiInvolved`/`externalMarks` (non-strippable) + sources/review_refs at relevant depth. ✅ = §7.

## Field mapping

### Claim ↔ `discovery_objects`
| ClaimSpec | Discovery | Note |
|---|---|---|
| `id` | `id` (uuid) | ✅ |
| `type` | `object_type` (open TEXT, no DB enum) | ✅ open/extensible — matches §5 |
| `register` | `register` | ✅ exact |
| `confidence` | `confidence_score` | **naming** |
| `content` | `core_text`/`summary`/`description`/`type_specific_fields` | Discovery structures the payload; maps to opaque `content` |
| `agency` | (projected from `lineage_events.actor_agency`) | ✅ projection |
| `superseded_by` | (via supersede event; `status`/version) | confirm a supersede edge exists at the projection |
| `derived_from` | `lineage_events.derived_from` | ✅ |
| `drew_on` | `lineage_events.input_object_ids` + `input_citation_ids` | ✅ (union) |
| `created_by` | genesis `lineage_events` row | ✅ |

### Event ↔ `lineage_events`
| ClaimSpec | Discovery | Note |
|---|---|---|
| `id` | `id` | ✅ |
| `subject` | `object_id` | **naming** |
| `family` | — | **divergence — see below** |
| `type` | `event_type` | **naming** |
| `actor.*` | `actor_type`/`actor_agency`/`actor_name`/`model_used` | ✅ (`actor_name`↔`actor_id`) |
| `inputs.derived_from` | `derived_from` | ✅ |
| `inputs.drew_on` | `input_object_ids` + `input_citation_ids` | ✅ |
| `at` | `created_at` | **naming** |
| _(none)_ | `review_ref` | **spec should add — see below** |
| _(none)_ | `confidence_before/after`, `details`, `token_count`, `cost_estimate` | Discovery impl detail (powers `confidence_history`) |

### Disclosure ↔ `published_surfaces.disclosure`
| ClaimSpec | Discovery | Note |
|---|---|---|
| `subject` | (the published surface) | ✅ |
| `ai_involved` | `aiInvolved` (floor, non-strippable) | **naming** (camel vs snake) |
| `external_marks` | `externalMarks` (floor) | **naming** |
| `sources` | sources/review_refs at relevant depth | ✅ |

---

## Divergences — and which side moves

1. **Field naming (cosmetic): map at the boundary, do NOT rename the DB.**
   `confidence_score`↔`confidence`, `object_type`↔`type`, `object_id`↔`subject`, `event_type`↔`type`, `actor_name`↔`actor_id`, `created_at`↔`at`, `aiInvolved`↔`ai_involved`.
   → **Neither side renames internally.** ClaimSpec is the *interchange* contract; Discovery keeps its DB names and translates at the ClaimSpec **projection** (the REST/MCP egress) — exactly the provenance-spine "objects are a projection" law. The projection *is* the conformance surface.

2. **`review_ref` — SPEC MOVES (add it).**
   Discovery already carries `review_ref` on `lineage_events` (the Art. 50(4) human-control-exemption record). ClaimSpec v0 only mentions review-refs inside Disclosure. → **Add an optional `review_ref` to the ClaimSpec `Event`** (and keep its union into Disclosure.sources). Discovery has the better primitive; the spec should adopt it.

3. **`family` — SPEC CLARIFIES (derived, not stored).**
   ClaimSpec v0 has `family` (genesis/production/epistemic/relational/lifecycle) as a first-class Event field; Discovery stores flat `event_type`. → **Define `family` as a coarse grouping *derivable* from `type`** (a published mapping), so an implementer with flat event types is still conformant; the projection assigns family. (Keep `family` for query ergonomics; make it non-authoritative.)

4. **Discovery's domain richness — VALIDATES the lean-core design; formalize the extension mechanism.**
   Discovery adds, beyond ClaimSpec's core:
   - `confidence_signals` (weighted dimensions composing the score — source_authority, evidence_strength, consensus, recency, replication, methodology_quality),
   - `relationships` (a general typed claim-to-claim graph beyond lineage),
   - `maturity_score` (a continuous maturity *alongside* the `hypothesis→…→law` type ladder),
   - domain object fields (`emotion_tags`, `opportunity_score`, `frontier_distance`, `mission_relevance`, …),
   - `confidence_history` (a cached projection of confidence-over-time).
   → **None belong in ClaimSpec core.** They confirm the §2 `content` + the "thin core, conformant extensions" decision. **Action: define a clean *extension* convention** (namespaced keys under `content` / an `x-` extension block, AppSpec-style) so Discovery's richness is *declared* rather than ad-hoc. Optionally add a non-normative `confidence_breakdown` hint and a `maturity` hint as Standard-tier extensions.

---

## Resolution (applied in v0.1.0-alpha.2)

1. **`Event.review_ref`** — ✅ **ADDED** (optional; Art. 50(4); unions into `Disclosure.sources`). The reference implementation had the better primitive; the spec adopted it.
2. **`family` as derived** — ⏸ **HELD** for the **second implementer** (Builder's lean embed) to confirm against a flat-`event_type` model before the schema changes. Flagged provisional in §6.2; do not depend on `family` being authoritative.
3. **Extension convention** — ✅ **ADDED** as §11 (`x-` namespaced keys, validator-ignored, non-authoritative). Discovery's `confidence_signals`, `relationships`, `maturity`, and domain fields are now *declared* extensions rather than ad-hoc — core stays thin.

Everything else conforms as-is. The act of building Discovery's ClaimSpec projection (REST/MCP egress) is the executable conformance test; **Builder is the second implementer that confirms `family`.**

## Conformance verdict

**ClaimSpec v0 survives its first real implementer with one additive field, one clarification, and one extension convention — and Discovery reaches Extended tier.** No invariant moved; the core held. This is exactly the validation the "≥2 implementers before stabilizing" rule is for — Builder (the lean embed) is the natural second.
