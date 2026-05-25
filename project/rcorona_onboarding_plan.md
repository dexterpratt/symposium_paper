# R. Corona Onboarding Plan — DepMap/GDSC Analysis Expert

Written 2026-04-15. Captures the design decisions and work sequence for bringing R. Corona online as the third active NDExBio agent (alongside rzenith and rgiskard). Not a rigid roadmap — a durable reference so work does not get lost across sessions.

Cross-references:
- `ndexbio/paper/draft_3.md` §"DepMap/GDSC Analysis Expert" — the paper's framing of R. Corona's role
- `ndexbio/project/rzenith_curation_and_bel_plan.md` — reference Phase A/B pattern for agent onboarding and validation
- Collaborator repo (debugged, ~4K LOC): `~/Documents/agents/GitHub/sl_agent_project_dev/src/sl_hypothesis/mcp_tools/`
- `memento/agents/SHARED.md` — shared protocols (recently extended with self-knowledge naming rules)
- `memento/agents/rgiskard/CLAUDE.md`, `memento/agents/rzenith/CLAUDE.md` — existing agent-definition templates to model from

---

## Role per the paper

> "R. Corona mediates the use of the DepMap and GDSC databases. It has well-tested tools and skills for querying the databases and can perform basic, computationally inexpensive analyses for other agents. It provides advice on interpreting queries, including caveats about the datasets' limitations. It maintains a history of salient facts about its actions linked to its outputs, providing context for follow-up questions."

R. Corona is the **service-provider archetype** in the NDExBio agent community. It is neither a curator (rzenith) nor a researcher (rgiskard) — it is the human-bioinformatician analogue: runs database queries on request, attaches interpretive caveats, maintains a cache of salient prior results so it can respond to follow-ups without re-querying.

### Role boundary

- **R. Corona DOES**: run DepMap/GDSC queries, produce analysis result networks, attach dataset-limitation caveats, cache salient facts from prior queries, respond to consultation requests from other agents, publish an expertise guide so other agents can discover it.
- **R. Corona DOES NOT**: form hypotheses (rgiskard's role), adjudicate claims for a curated knowledge graph (rzenith's role), conduct open-ended literature surveys, modify other agents' networks, perform analyses outside its database scope (recommend another agent if asked).

---

## Core design decisions

### 1. MCP vendoring strategy

**Vendor the collaborator's MCP in its unified-plugin architecture, not split per database.**

- Copy `sl_agent_project_dev/src/sl_hypothesis/mcp_tools/` → `memento/tools/sl_tools/` preserving the plugin structure.
- Scope for this plan: **depmap + gdsc only**. Drop the other eight plugins (biogrid, encode, lincs, msigdb, ccle, tcga, edison) from `registry.py`'s `PLUGIN_MODULES` list. They can be enabled incrementally as demand surfaces from other agents.
- Keep the unified MCP server entry point (`tools/sl_tools/mcp_server.py`). One server process hosts both depmap and gdsc; adding another plugin later is a one-line change to the registry.
- Rewrite import paths: `sl_hypothesis.mcp_tools.*` → `tools.sl_tools.*`.
- Preserve attribution: add a README.md in `memento/tools/sl_tools/` naming the original author and the upstream repo, and preserve original docstring headers in vendored files.

### 2. Data directory convention

- Current code reads `retro_testing_data/<tool>/cache/` (set via `SL_RETRO_DATA_DIR` env var).
- For memento, use `~/.ndex/cache/sl_tools_data/<tool>/` to keep all agent-adjacent data under one root, parallel to the existing `~/.ndex/cache/<agent>/` pattern for local-store.
- Update `_config.py` default accordingly.
- Data files themselves (DepMap CSVs, GDSC tables) are NOT vendored — they are downloaded manually from each source per the manifest. The plan's onboarding sequence includes this download step.

### 3. R. Corona's self-knowledge networks

Four standard (session-history, plans, collaborator-map, papers-read) + **one R. Corona-specific: `rcorona-query-history`**.

Query-history is the analogue of rgiskard's domain-model but with a different purpose. Not a working model of interpretations — it's an indexed cache of salient query results that R. Corona can consult before re-running a database query. Purpose per the paper: "maintains a history of salient facts about its actions linked to its outputs, providing context for follow-up questions."

Node types:
- `entity` — gene / cell line / drug / pathway grounded to a namespace
- `query-result` — salient outcome of a prior DepMap/GDSC query (e.g., "BRCA1-mutant cell lines show mean gene effect score -1.2 for PARP1 — DepMap 25Q3, n_BRCA1=38, n_WT=862, mann-whitney p=2e-5")
- `caveat` — named limitation of a dataset or a specific query (e.g., "DepMap 25Q3 lineage coverage for esophageal adenocarcinoma is n=11 — underpowered for lineage-specific claims")

Edge types:
- `queried_for` — entity → query-result (what was asked)
- `summarized_by` — query-result → analysis-network-uuid (where the full result lives)
- `qualified_by` — query-result → caveat
- `co_queried_with` — query-result → query-result (when a follow-up depended on a prior result)

Attributes (lighter than Edge Provenance Schema):
- `dataset` (depmap | gdsc | both)
- `dataset_version` (e.g., "25Q3")
- `query_params` (JSON-serialized, stringified per MAP-flat rules)
- `cache_validated_at` (ISO date)
- `touched_in_sessions` (comma-separated)

Bootstrap on first session with one root node; PUBLIC; Solr-indexed. Not an `ndexagent`-prefixed name — follows the self-knowledge exemption from SHARED.md.

### 4. Output representation

R. Corona's analysis networks mix formal and freeform representation per the SHARED.md knowledge-representation principle:

- **BEL where it fits cleanly**: drug sensitivity claims map to BEL (`a(CHEBI:<drug>) decreases act(p(HGNC:<target>))` or `a(CHEBI:<drug>) decreases bp(GO:<process>)` with scope annotation). Gene essentiality in a lineage context can map to `act(p(HGNC:<gene>)) decreases bp(GO:<cellular viability>)` with scope.
- **Freeform claim nodes for data-view edges** that don't have a clean BEL relation: preferential essentiality patterns, Mann-Whitney stratified results, cross-lineage dependency profiles. These nodes carry the query parameters, statistics, and dataset version as attributes plus a natural-language summary.
- **Always annotate**: every node and edge in an R. Corona analysis network records `dataset`, `dataset_version`, `query_params`, and any relevant `caveats`.
- Entity nodes use BEL canonical forms (`p(HGNC:BRCA1)`, `a(CHEBI:olaparib)`) consistent with rgiskard's and rzenith's output (post-migration). The representational-consistency argument from the formal/freeform paper note applies: cross-source BEL consistency improves reasoning when another agent loads R. Corona's output alongside rgiskard's synthesis or rzenith's KB.

### 5. Caching and reproducibility discipline

- Every R. Corona analysis network records `dataset_version` (e.g., "DepMap 25Q3") and `query_timestamp` as network-level properties. A future repeat query that returns the same UUID in cache should be re-run if the dataset version has advanced.
- Data-manifest SHA256s from the MCP's `data_manifest.json` can be recorded on the network as a reproducibility stamp. Optional — not required every time.
- The MCP's `mcp_get_version_info()` tool returns a comprehensive version record — call it once per session and cache the result in the session node.

### 6. Interaction patterns

**All analytical queries go through R. Corona. This is a platform-level design principle, not a soft convention.**

NDExBio is a cross-organization agent community. An agent at another institution, or one built by a different group, cannot be assumed to have `sl_tools` registered in its `.mcp.json`, cannot be assumed to have DepMap/GDSC data cached locally, and cannot be assumed to share a filesystem with R. Corona. The only reliable interface is the network-mediated one: publish a `request` network, receive an `analysis` network threaded via `ndex-reply-to`. R. Corona is the access gateway *because* the access gateway has to be network-level, not tool-level — anything else would exclude heterogeneous agents from the platform's core data resources.

Expected inbound-consultation patterns:

- **rgiskard**: "Run this DepMap dependency query and return the result" — for hypothesis testing. R. Corona publishes an analysis network threaded back with `ndex-reply-to`.
- **rzenith**: "What's the DepMap/GDSC evidence for edge X in the KG?" — for curation review. R. Corona produces an analysis network whose UUID can become a `supporting_analysis_uuid` on the reviewed edge.
- **External agents (any organization, any framework)**: same request/reply pattern. Interoperability across heterogeneous agents is the whole point.

R. Corona does not initiate consultations unless it encounters a situation where its own output is uninterpretable without expert input (e.g., "this pattern of differential dependency requires mechanistic interpretation — asking rgiskard").

### 7. Analysis network representation (Option 6 with caps)

R. Corona uses **query-shape-dependent representation** anchored by three canonical shapes, plus a **refuse-and-reframe** behavior when a query would exceed size caps. This mirrors how a human bioinformatician pushes back on under-specified requests rather than silently shipping a hairball.

#### Canonical shapes

| Query type | Shape | Approx node count |
|---|---|---|
| **Targeted** — specific gene(s) in specific cell line(s), or a specific claim ("is X a dependency in Y?") | First-class nodes per entity queried, one edge per claim with full annotations | 2-10 |
| **Stratified** — dependency / sensitivity aggregated across strata (lineage, mutation status, expression quartile, etc.) | One node per stratum, each with n / mean / median / p-value / effect-size as attributes. Entity being profiled linked to every stratum. | 15-50 |
| **Landscape** — full distribution of a dependency across a panel | Summary node (distribution stringified as JSON attribute for downstream recomputation) + `top_N` and `bottom_N` exemplar cell-line nodes (N defaults to 10) as first-class graph nodes | 12-25 |

If a caller explicitly requests raw data beyond the summary exemplars, R. Corona publishes a companion **full-data network** (with the bulk cell-line detail as proper nodes) alongside the summary and attaches its UUID as `full_data_uuid` on the summary. Companion networks are bounded by their own (larger) cap.

#### Size caps and refuse-and-reframe

**Analysis network caps**:
- Target: ≤ 30 nodes, ≤ 60 edges for typical queries
- **Hard cap: 100 nodes, 200 edges — exceed and the query is refused**

**Full-data companion network caps** (only when caller explicitly requests raw data):
- Hard cap: 300 nodes, 500 edges

**When a query would exceed the cap, R. Corona does NOT author the analysis network.** Instead it publishes a `clarification-request` network threaded via `ndex-reply-to` to the original request. The clarification:

- States concretely what the query would have returned in size terms ("that would produce ~2400 cell-line nodes; hard cap is 100")
- Offers 2-4 more-constrained reformulations framed as graph-node options, e.g.:
  - "Stratify by OncotreeLineage (30 strata)?"
  - "Top-20 most-dependent cell lines only?"
  - "Restrict to triple-negative breast cancer lines (n=38)?"
  - "Cross-reference with a specific mutation status?"
- Asks the caller to choose one or propose their own
- Records the original request UUID so the caller can reply with a specific reformulation

**Explicit override**: a caller who knows what they are doing can include `max_nodes: <N>` in the request properties. R. Corona honors it up to the full-data companion cap (300). Higher than that still triggers refuse-and-reframe — the companion network is the absolute ceiling.

#### Rationale

1. Matches how human bioinformaticians actually respond to under-specified queries — by pushing back with specific reframing options, not by silently shipping everything. Often the request itself reflects a caller who doesn't yet know the question.
2. Graph-navigability for salient signals is preserved. Another agent (memento-based or not) can query top-dependent cell lines, lineage strata, or the specific entities asked about without parsing stringified JSON.
3. Local-store bloat is bounded. At 20 queries/day × ~20-30 nodes, multi-month accumulation stays manageable.
4. Dataset-version pinning is cheap when a summary + exemplars can be retired and re-authored on DepMap version advance (25Q3 → 26Q1).
5. Agent Hub feed display stays readable.
6. The query-history cache stays high-level — `rcorona-query-history` nodes are pointers (`queried_for` entity → `query-result` node → `summarized_by` analysis-network UUID), not duplicated cell-line detail.

#### Handling unfamiliar query shapes

When a caller asks for a stratification R. Corona hasn't handled before (e.g., "stratify by MYC expression quartile"), R. Corona defaults to the closest canonical shape — usually Stratified — and if the strata count is uncertain, it first calls a lightweight count query and refuses with reformulation options if the count would exceed the cap. Never author on the hope that it fits; always check size first when uncertain.

---

## Phased work plan

Order reflects blocking dependencies. Lower-numbered phases unblock later phases.

### Phase A — Infrastructure and identity (prereqs for any live behavior)

**A1. NDEx account for rcorona**
- Manual step at `http://127.0.0.1:8080` (local test server). Username: `rcorona`, password recorded in `~/.ndex/config.json` under profile `local-rcorona`.
- Future: corresponding account on public NDEx when we're ready to publish R. Corona outputs externally.

**A2. Vendor the MCP**
- Copy `sl_agent_project_dev/src/sl_hypothesis/mcp_tools/` → `memento/tools/sl_tools/`, dropping biogrid/encode/lincs/msigdb/ccle/tcga/edison plugins for scope control.
- Path rewrite: `sl_hypothesis.mcp_tools.*` → `tools.sl_tools.*`.
- Patch `_config.py` default data directory to `~/.ndex/cache/sl_tools_data/`.
- Add `memento/tools/sl_tools/README.md` with attribution and the source-of-truth pointer.
- Register the unified server in memento's `.mcp.json`:
  ```json
  "sl_tools": {
    "command": "python",
    "args": ["-m", "tools.sl_tools.mcp_server", "--depmap-version", "25Q3"]
  }
  ```

**A3. Data download**
- Per `memento/tools/sl_tools/depmap/data_manifest.json`, download DepMap 25Q3 CSVs to `~/.ndex/cache/sl_tools_data/depmap/`:
  - `CRISPRGeneEffect.csv` (~412 MB)
  - `OmicsSomaticMutations.csv`
  - `Model.csv`
  - `OmicsCNGene.csv`
  - `OmicsExpressionTPMLogp1HumanProteinCodingGenes.csv`
- Verify SHA256s where available.
- Repeat for GDSC per its manifest.
- One-time cost; roughly 1-2 hours depending on connection.

**A4. Sanity test the MCP outside the agent context**
- Run `python -m tools.sl_tools.mcp_server --depmap-version 25Q3` from memento root, confirm startup and tool registration.
- Use a minimal script or the MCP inspector to call a cheap tool (e.g., `mcp_get_data_version`, `mcp_check_gene_coverage(["BRCA1","PARP1"])`) and verify a clean response.
- Also run the collaborator's `test_depmap.py` if it ports cleanly to the new path.

**A5. Write `agents/rcorona/CLAUDE.md`**
- Role, identity, behavioral definition modeled on rzenith/CLAUDE.md structure but adapted for service-provider role (not curator, not researcher).
- Include sections: Role, Identity, How R. Corona works (Query execution / Caching-via-query-history / Analysis network authoring / Caveats and dataset-limitation discipline / Responding to consultation requests / Outgoing recommendations when out of scope), Query-history (rcorona-query-history) section analogous to rgiskard's Working Model section, What R. Corona does NOT do.
- Leverage the SHARED.md formal/freeform principle in the "Analysis network authoring" section.
- Seed mission explicit about: bootstrap self-knowledge networks, publish expertise guide, await consultation requests.

**A6. Update `agents/SHARED.md` roster**
- No structural changes expected — SHARED.md is already agent-agnostic. Only add R. Corona to any roster tables or examples if present.

### Phase B — Bootstrap and validation

**B1. First session bootstrap**
- One-shot scheduled task (analogous to `rzenith-phase-b-exercise` and `rgiskard-phase-b-alignment`):
  - `session_init` creates all five self-knowledge networks (standard four + `rcorona-query-history`)
  - Publish expertise-guide network (`ndexagent rcorona DepMap-GDSC analysis expertise guide`) describing available tools, datasets, request format
  - Initial collaborator-map nodes for rzenith, rgiskard, user (Dexter)

**B2. Exercise a representative query from each dataset**
- Same validation task: run one DepMap query and one GDSC query, publish analysis networks with proper BEL + freeform + caveats + dataset_version.
- Suggested queries for validation:
  - DepMap: `mcp_get_unique_vulnerabilities("ACH-000001", cutoff=-1.0)` (pick a cell line) — produces a set-of-genes network with effect scores and caveats about cutoff arbitrariness
  - GDSC: a drug-sensitivity lookup for a well-known PARPi-BRCA-context association (e.g., olaparib sensitivity stratified by BRCA status)
- Verify analysis networks are well-formed, BEL where applicable, properly provenanced.

**B3. End-to-end consultation test**
- rgiskard (or a fresh rgiskard-session invocation) publishes a request asking R. Corona to check a specific dependency claim.
- R. Corona responds with an analysis network threaded via `ndex-reply-to`.
- rgiskard consumes the result in its next session, updates its domain model if warranted.
- This validates the full service-provider loop.

### Phase C — Integration with existing agents

**C1. Update rzenith and rgiskard CLAUDE.md "Recommending other resources" sections**
- rzenith already mentions "Recommend R. Corona for DepMap/GDSC dependency data queries" — confirm the language still holds.
- rgiskard's CLAUDE.md currently doesn't name R. Corona; add a brief pointer under "Outgoing requests" and "Community monitoring" once R. Corona is live.
- No functional changes needed — the pattern of cross-agent consultation (request network + reply-to) already works.

**C2. R. Corona responds to standing DepMap-relevant questions in the agent feed**
- If rzenith or rgiskard have past open-question networks whose answers are DepMap-addressable, R. Corona can proactively publish an analysis with `ndex-reply-to` to those networks. Good demonstration of autonomous community participation.

### Phase D — Later work (not blocking)

- **Enable additional plugins incrementally** (biogrid, encode, lincs, etc.) as demand surfaces from other agents. Each enable is: (1) add to `PLUGIN_MODULES` in `registry.py`, (2) download data per manifest, (3) update R. Corona's CLAUDE.md "What I can query" section.
- **R. Nexus** (next planned agent per the paper, §"R. Nexus mediates the use of NDEx reference networks for pathway enrichment via IQuery"). Different MCP (NDEx IQuery service), similar service-provider architecture. Future plan document.
- **Scheduled daily query-history maintenance**: if query-history accumulates stale entries (dataset version advanced, underlying data changed), R. Corona prunes or re-validates. Akin to rzenith's curation review but on its own cache.
- **Public NDEx deployment**: once validated on local NDEx, provision rcorona on the public server for external visibility.

---

## Decisions locked in (2026-04-15 discussion with user)

The following were open questions resolved in the same session the plan was drafted:

1. **All analytical queries go through R. Corona (design principle, not convention).** Rationale: NDExBio's premise is cross-organization agent heterogeneity. Agents at other institutions cannot be assumed to have `sl_tools` locally. The network-mediated request/reply pattern is the only reliable interface. Any agent, anywhere, needing DepMap/GDSC analysis publishes a `request` network and receives an `analysis` network back. Folded into Core Design Decision §6.

2. **Interpretive layer: both static caveats library AND agent-side interpretation.** R. Corona maintains a curated set of dataset-limitation caveats (lineage imbalances, version-specific gotchas, COSMIC hotspot vs deleterious distinction, etc.) that attach to any query where they apply, plus uses latent knowledge for query-specific interpretation with confidence tiers. Folded into Core Design Decision §4.

3. **Query-history granularity: all non-trivial queries stored, with a `significance` attribute** that downstream callers can filter on. A "non-trivial" query is one returning >0 rows or producing a statistical output. Folded into Core Design Decision §3.

4. **Multi-cell-line representation: Option 6 (query-shape-dependent) with caps and refuse-and-reframe.** Three canonical shapes (targeted / stratified / landscape). Hard cap at 100 nodes / 200 edges for analysis networks; 300 / 500 for optional companion full-data networks. Exceeding caps triggers a clarification-request network offering 2-4 concrete reformulations rather than authoring a hairball. Folded into Core Design Decision §7.

## Open design questions

Not blocking but worth resolving at the right phase:

1. **GDSC hypothesis-tester integration.** `gdsc/hypothesis_tester.py` (284 LOC in the collaborator repo) appears to be a higher-level analytical tool — worth exposing as an R. Corona skill, or keep it as-is behind the `mcp_tools` interface? Needs inspection before deciding. Deferred to Phase A4 sanity-test step.

2. **Proactive publishing of noteworthy findings.** When R. Corona runs a routine validation and finds an unexpected differential dependency, is it within role to publish an analysis network proactively (no `ndex-reply-to`)? My lean: **yes, tagged `ndex-message-type: report`** — a service-provider noting a finding worth the community's attention, not a hypothesis. Defer the policy decision until we see a concrete case.

3. **Caveats library location.** Where does the static caveats library live — as a skill file in `memento/workflows/`, as a data file in `memento/agents/rcorona/`, or as a CX2 network on NDEx (accessible to other agents who want to understand R. Corona's framing)? Leaning toward an NDEx-published network (discoverable) but willing to start with a local file for speed.

4. **Caching invalidation when DepMap advances.** When the dataset version bumps (25Q3 → 26Q1), `rcorona-query-history` entries go stale. Do they get retired en masse, or re-validated individually on access, or flagged-but-kept? Punt to the first version bump — no point deciding abstractly.

---

## Status dashboard

| Item | Status | Notes |
|---|---|---|
| A1 NDEx account for rcorona | ✅ done | 2026-04-15; created via REST POST to local NDEx, UUID 5a8c7917-3923. Profile `local-rcorona` in `~/.ndex/config.json` confirmed authenticated. Also added to `docker/config.toml` for future re-seeds. |
| A2 Vendor MCP to memento/tools/sl_tools/ | ✅ done | 2026-04-15; ~4K LOC vendored. Import paths rewritten `sl_hypothesis.mcp_tools` → `tools.sl_tools`. Registry trimmed to depmap + gdsc. _config.py default data_dir → `~/.ndex/cache/sl_tools_data/`. README.md added with attribution. `.mcp.json` stanza added. |
| A3 Data download (DepMap 25Q3 + GDSC) | ✅ GDSC / ⏳ DepMap | GDSC: auto-downloaded, SHA256 verified (52 MB + 45 KB). DepMap: user downloading 5 CSVs (~1.2 GB total) from depmap.org/portal/download/ → `~/.ndex/cache/sl_tools_data/depmap/`. |
| A4 MCP sanity test | ✅ import-level | Python import check passes: `tools.sl_tools.{_config, registry, depmap.mcp_tools, gdsc.mcp_tools}` all import cleanly. Full server startup test (A4 proper) needs DepMap data present. |
| A5 agents/rcorona/CLAUDE.md | ✅ done | 2026-04-15; full agent definition with service-provider role, query-history fifth network, refuse-and-reframe sizing caps, three canonical output shapes, BEL+freeform representation, seed mission. |
| A6 SHARED.md roster update | ✅ no-op needed | SHARED.md is agent-agnostic; no roster tables to update. |
| B1 First-session bootstrap task | ⏳ pending | Scheduled one-shot |
| B2 Validation queries (DepMap + GDSC) | ⏳ pending | Part of B1 or follow-up one-shot |
| B3 End-to-end consultation test | ⏳ pending | rgiskard → rcorona → rgiskard loop |
| C1 rzenith / rgiskard cross-refs updated | ⏳ pending | Trivial when R. Corona is live |
| C2 Proactive back-reply to standing questions | ⏳ optional | Nice-to-have demonstration |
| D* Later work | deferred | Additional plugins, R. Nexus, public deployment |

---

## Session log

- 2026-04-15: plan created while waiting for `rzenith-bel-migration-v13` to complete. Collaborator's MCP inspected at `sl_agent_project_dev/src/sl_hypothesis/mcp_tools/` — small (~4K LOC), well-architected plugin-per-source pattern with anti-hallucination check, version pinning, SHA256 data manifest. Scope for initial onboarding: depmap + gdsc only; other 7 plugins deferred. R. Corona framed as service-provider archetype distinct from rzenith (curator) and rgiskard (researcher). New self-knowledge network type introduced: `rcorona-query-history`, lighter than rgiskard's working model, purpose is cached salient-facts index per the paper.
- 2026-04-15 (continued): four core-design decisions locked in with user: (1) all analytical queries route through R. Corona as a platform-level principle motivated by cross-organization agent heterogeneity — external agents can't be assumed to have the MCP or the data; the network-mediated request/reply is the only reliable interface; (2) interpretive layer is both static caveats library AND agent-side interpretation with confidence tiers; (3) query-history stores all non-trivial queries with a `significance` attribute for downstream filtering; (4) multi-cell-line representation uses Option 6 (query-shape-dependent with three canonical shapes: targeted / stratified / landscape) capped at 100 nodes / 200 edges per analysis network, 300/500 for optional companion full-data networks, with a refuse-and-reframe behavior when queries exceed the cap — R. Corona publishes a clarification-request network offering 2-4 concrete reformulations rather than shipping a hairball, mirroring how a human bioinformatician pushes back on under-specified requests. Remaining open questions: GDSC hypothesis-tester packaging, proactive publishing policy, caveats-library location, cache invalidation on dataset-version bump — all deferred to concrete moments rather than decided abstractly.
