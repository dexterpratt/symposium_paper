---
name: rgiskard-phase-b-alignment
description: One-shot: validate rgiskard's updated CLAUDE.md — bootstrap working model, exercise paper-processor subagent, practice BEL + freeform discipline
---

Run a session as the rgiskard agent. This is a ONE-SHOT validation run that exercises the 2026-04-15 updates to rgiskard's role definition — adoption of BEL authoring + Edge Provenance Schema, first use of the paper-processor subagent, and bootstrap of the new `rgiskard-domain-model` working-model network.

**Standard setup:**
- Agent instructions: `/Users/dexterpratt/Documents/agents/GitHub/memento/agents/rgiskard/CLAUDE.md` (recently rewritten — read carefully)
- Shared protocols: `/Users/dexterpratt/Documents/agents/GitHub/memento/agents/SHARED.md` (note the new "Formal and freeform representations are complementary" subsection under Knowledge Representation)
- NDEx profile: `local-rgiskard` (local server 127.0.0.1:8080)
- Store agent: `rgiskard`
- Working directory: `/Users/dexterpratt/Documents/agents`

Read CLAUDE.md and SHARED.md, then call `session_init(agent="rgiskard", profile="local-rgiskard")`. Abort the run if init fails.

**Focus of THIS run (overrides normal plan-following):**

1. **Bootstrap the working-model network.** `rgiskard-domain-model` does not yet exist on local NDEx. Create it per the spec in `rgiskard/CLAUDE.md` § Working Model → Bootstrap: PUBLIC, Solr-indexed, one root node ("cGAS-STING in cancer (rgiskard working model root)"), properties `ndex-message-type: self-knowledge`, `ndex-workflow: working-model`, `ndex-network-type: working-model`. Cache it into the local store under category `working-model`.

2. **Pick ONE recent paper** in the cGAS-STING/cancer domain that you have NOT already analyzed. Candidates to consider (check your papers-read network first to avoid duplicates):
   - A recent bioRxiv preprint from the last 7-14 days on cGAS-STING in cancer (use `search_recent_papers` on keywords like "cGAS STING cancer", "STING tumor immunity")
   - A recent PubMed paper from 2025-2026 on cGAS-STING in cancer immunology
   - If nothing clearly new appears, pick a landmark paper you haven't yet analyzed — e.g. a Decout 2021 review or a Kwon 2020 primary paper on cGAS-STING-cancer links
   Pick the paper that looks most likely to interact with your current working-model priors (even though the working model is newly created — your priors come from latent knowledge).

3. **Invoke the paper-processor subagent** (`workflows/BEL/subagent/SUBAGENT.md`) on the chosen paper. This is the first live rgiskard invocation of that subagent. Follow the caller pattern in SUBAGENT.md — pass a JSON task spec with `paper_id`, `focus_context: "focus on cGAS-STING mechanism claims relevant to cancer immunology"`, and `caller_agent: "rgiskard"`. Validate the returned JSON against `output_schema.json`. Note any drift or defects.

4. **Persist the subagent output as an analysis network**: `name: ndexagent rgiskard analysis PMID-<pmid or DOI-slug> 2026-04-15`, `ndex-agent: rgiskard`, `ndex-message-type: analysis`, `ndex-workflow: paper-processor`, PUBLIC + Solr-indexed. Copy `resolution.verification_warnings` as network-level properties. Add a papers-read entry with `analysis_network_uuid`.

5. **Update the working model.** This is the exercise of the new working-model discipline. Do AT MOST 3-5 updates to `rgiskard-domain-model`. Prioritize:
   - Entity nodes for the key players in the paper (cGAS, STING, and whichever cancer/immune entities are central)
   - One expectation node capturing what you thought about the main mechanism BEFORE reading the paper, with a `consistent_with` or `contradicted_by` meta-edge to the papers-read entry based on what the paper showed
   - One pattern or puzzle node if the paper surfaces an observation worth tracking
   - A few BEL mechanism edges between entities, authored per `workflows/BEL/SKILL.md`
   - Do NOT over-update. The 3-5 cap is a forcing function — respect it.

6. **Consultation-response practice (optional, skip if time is tight).** Check the social feed for `ndex-message-type: request` networks from rzenith or other agents addressed to rgiskard. If one exists and you can address it without a full additional paper read, do so per the inbound-consultation protocol in CLAUDE.md.

**What to report at session end** (in the session-history node's `lessons_learned` and in a brief final summary):
- Bootstrapped working-model network UUID
- The paper selected + analysis network UUID
- Paper-processor subagent verification_warnings (if any) and whether validation passed
- Which working-model nodes were created/touched (canonical forms + node_type) — should be ≤ 5
- Any defects or drift surfaced in: (a) the paper-processor subagent contract, (b) the working-model spec in CLAUDE.md, (c) the BEL skill, (d) the formal/freeform principle as applied. This feedback is critical — the one-shot task is a shakedown run.
- Whether the BEL/freeform mix felt natural or forced for this paper

**Known hazards to watch for:**
- PubMed MCP's PMID→PMCID mismapping defect (see `deferred_tasks.md`). The paper-processor subagent handles this via its §3 verification chain; trust the `verification_warnings` output.
- Scheduled-task tool-permission friction (also in `deferred_tasks.md`). If a tool call stalls on a permission prompt, the task may pause.
- LadybugDB single-writer lock if any other rgiskard session is running (shouldn't be; `local-rgiskard` is dedicated).

**End-of-session (mandatory):** Complete all 5 session-end steps from SHARED.md. Update rgiskard-plans to add a `done` action for this validation run and add any new follow-up actions surfaced (e.g. "fold identified subagent drift back into SUBAGENT.md"). Publish all self-knowledge networks including the new `rgiskard-domain-model`.