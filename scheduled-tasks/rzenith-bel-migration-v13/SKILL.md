---
name: rzenith-bel-migration-v13
description: One-shot: migrate the 6 provenanced legacy-syntax edges in rzenith KB v1.2 to BEL, producing v1.3
---

Run a session as the rzenith agent. This is a ONE-SHOT migration task: convert the 6 provenanced legacy-syntax edges currently in the DDR knowledge base (v1.2) to proper BEL syntax, producing KB v1.3. This is cleanup from the 2026-04-15 Phase B exercise run which added provenance but did NOT migrate syntax.

**Standard setup:**
- Agent instructions: `~/Documents/agents/GitHub/memento/agents/rzenith/CLAUDE.md` (recently updated with the "Author in BEL — and migrate on touch" rule in §Curation Review Protocol step 4 — read it carefully)
- Shared protocols: `~/Documents/agents/GitHub/memento/agents/SHARED.md`
- BEL skill: `~/Documents/agents/GitHub/memento/workflows/BEL/SKILL.md` and `reference/` files
- NDEx profile: `local-rzenith` (127.0.0.1:8080)
- Store agent: `rzenith`
- Working directory: `~/Documents/agents`

Read CLAUDE.md and SHARED.md, then call `session_init(agent="rzenith", profile="local-rzenith")`. Abort if init fails.

**Scope of THIS run — migration only.** Do NOT review new edges. Do NOT invoke the paper-processor subagent (provenance already exists for all 6 target edges). The work is purely syntactic migration to BEL with referential-integrity discipline.

**The 6 edges to migrate** (all in `ndexagent rzenith DDR synthetic lethality knowledge base` uuid 4d23e21b-368a-11f1-84e0-02b2a9d51b89):

| # | Legacy edge | Provenance status | Migration target |
|---|---|---|---|
| 1 | `BRCA1 -[synthetic_lethal_with]-> PARP1` | established, pmid:15829966,15829967 | Synthetic lethality is a domain-specific relation BEL doesn't natively express. Preferred approach: freeform claim node encoding the SL relationship in prose, with entity nodes `p(HGNC:BRCA1)` and `p(HGNC:PARP1)` linked. Alternative: `composite(p(HGNC:BRCA1), p(HGNC:PARP1)) negativeCorrelation bp(GO:"cell viability")` with scope annotation. Choose per your judgment per the BEL skill step 6. |
| 2 | `BRCA2 -[synthetic_lethal_with]-> PARP1` | same | same treatment as #1 |
| 3 | `PARP1 -[inhibition_causes]-> Genomic Instability` | established, pmid:15829966 | Can decompose: `a(CHEBI:"PARP inhibitor") directlyDecreases act(p(HGNC:PARP1))` + a downstream chain, or a single `a(CHEBI:"PARP inhibitor") increases path(DOID:"genomic instability")` with scope. Decomposition preferred if evidence supports each step. |
| 4 | `ATM -[phosphorylates]-> STING` | supported, pmid deferred | Clean BEL: `act(p(HGNC:ATM), ma(kin)) directlyIncreases p(HGNC:STING1, pmod(Ph))`. If the specific residue is known, include it in pmod. |
| 5 | `cGAS -[inhibits]-> RAD51` | supported, pmid deferred | Clean BEL: `p(HGNC:CGAS) decreases act(p(HGNC:RAD51))` (use `directlyDecreases` only if direct binding shown). |
| 6 | `PARP1 -[inhibition_trapping_causes]-> Genomic Instability` | established, pmid:22193524 | New edge from 2026-04-15 split-add; never should have been authored non-BEL. Represent the trapped-complex state. One option: a freeform claim node describing the trapped-PARP-DNA adduct as a cytotoxic protein-DNA complex; connect via BEL edges from `a(CHEBI:"PARP inhibitor")` and `p(HGNC:PARP1)`. Use your judgment. |

**Migration mechanics (per rzenith/CLAUDE.md §Curation Review Protocol step 4):**

For each edge above:
1. Retire the legacy-syntax edge in place: set `evidence_status: superseded`, `superseded_by: <new BEL edge UUID or node identifier>`, add a brief `superseded_rationale`.
2. Author the BEL replacement (or claim node + connecting edges) with the FULL Edge Provenance Schema. Copy forward the existing `evidence_tier`, `pmid`, `scope`, `last_validated`; update `last_validated` to 2026-04-15 since this is a re-validation touch.
3. Entity grounding: replace bare symbols (`BRCA1`, `cGAS`, `ATM`) with proper BEL forms (`p(HGNC:BRCA1)`, etc.). If an entity already exists as a bare node in the KB and is referenced by OTHER non-migrated edges, leave the bare node in place for now (future migration pass) and add the namespace-grounded form as a new node. Do NOT rename bare nodes in place — that breaks graph referential integrity.
4. Log each migration in the review-log as a new `edge-review` node under a new `review-session` node (session_date: 2026-04-15, action tag should include `migrate-to-bel`). Use the existing review-log network `rzenith-review-log` (uuid faf129a8-390e-11f1-84e0-02b2a9d51b89). Do NOT create a new review-log network.

**KB version bump**: update network properties to `ndex-version: 1.3`. Update the description to note "v1.3: BEL migration of 6 provenanced edges from v1.2; legacy non-BEL edges retired in place, BEL equivalents authored. Synthetic lethality edges authored as freeform claim nodes where BEL distorts meaning per SHARED.md knowledge representation principle."

**Representational consistency check**: compare your BEL authoring to rgiskard's `ndexagent rgiskard analysis PMID-38200309 2026-04-15` (uuid 63961232-3912-11f1-84e0-02b2a9d51b89). That network is the reference implementation for proper BEL authoring. Your output should use the same `p()/a()/bp()/path()/complex()/act()` grammar and the same namespace-grounding discipline.

**Session end**: complete all 5 mandatory SHARED.md session-end steps. Add a session-history node for this migration session. Update plans: mark the migration action done; add a follow-up action for migrating any remaining non-BEL edges in the KB (there are 70+ `component_of` and `targeted_by` edges that also need migration, but those are deferred to future sessions; note them in the plan with status `planned`).

**What to report at session end** (in session-history lessons_learned AND in a final summary message):
- KB version after (should be v1.3)
- UUIDs of the 6 new BEL edges/claim nodes
- UUIDs of the 6 retired legacy edges (with evidence_status:superseded confirmed)
- Any BEL expressions you found genuinely awkward — feed these back into the BEL SKILL.md or reference files as amendments worth considering
- Any drift or ambiguity you hit in the new "Author in BEL — and migrate on touch" rule in CLAUDE.md
- Remaining non-BEL edge count in the KB (estimate)

**Known hazards:**
- `set_network_properties` replaces ALL properties (see `ndexbio/project/deferred_tasks.md`). When bumping version, pass the full property list.
- LadybugDB single-writer lock if another rzenith session is active (shouldn't be).