---
name: rzenith-phase-b-exercise
description: One-shot: commit A2 backfill diffs to live rzenith KB + exercise paper-processor subagent (Phase B validation)
---

Run a session as the rzenith agent. This is a ONE-SHOT validation run for Phase B of the rzenith curation plan (`/Users/dexterpratt/Documents/agents/GitHub/ndexbio/project/rzenith_curation_and_bel_plan.md`).

**Standard setup:**
- Agent instructions: `/Users/dexterpratt/Documents/agents/GitHub/memento/agents/rzenith/CLAUDE.md`
- Shared protocols: `/Users/dexterpratt/Documents/agents/GitHub/memento/agents/SHARED.md`
- NDEx profile: `local-rzenith` (local server 127.0.0.1:8080)
- Store agent: `rzenith`
- Working directory: `/Users/dexterpratt/Documents/agents`

Read CLAUDE.md and SHARED.md, then call `session_init(agent="rzenith", profile="local-rzenith")`. Abort the run if init fails — do not attempt workarounds.

**Focus of THIS run (overrides normal plan-following):**

The 2026-04-14 A2 backfill dry-run reviewed 3 edges in your v1.1 KB but was blocked from committing by a LadybugDB lock (now cleared). The dry-run dispositions were captured only in that session's transcript and are NOT in your plans network or memory — you will re-derive them. Your job in this session:

1. Identify the v1.1 KB network in your local cache (search for your DDR knowledge-base network; it should be tagged v1.1 or be the most recent `ndex-workflow: ddr-knowledge-base` network you own).

2. Re-run the curation review protocol (`rzenith/CLAUDE.md` § Curation Review Protocol) on these THREE specific edges from v1.1:
   - **cGAS → STING** (the canonical signaling edge)
   - **MRE11 → cytosolic DNA** (or the nearest equivalent edge; the underlying mechanism is MRE11's role in handling cytosolic dsDNA)
   - **PARP1 → genomic instability** (loss-of-function edge from PARP1 inhibition)

3. For each edge, follow the review decision tree fully:
   - Parse the claim precisely
   - Check the literature — **INVOKE THE PAPER-PROCESSOR SUBAGENT** (`/Users/dexterpratt/Documents/agents/GitHub/memento/workflows/BEL/subagent/SUBAGENT.md`) for at least ONE paper per edge. This is the first live exercise of that subagent and is a primary goal of this run. Persist each subagent output as an analysis network per the pattern in CLAUDE.md step 2.
   - Pick a disposition (keep+provenance, split, retire-and-replace, etc. — see rzenith/CLAUDE.md)
   - Commit all KB edits TO LIVE LOCAL NDEx (not dry-run)

4. Bootstrap the `rzenith-review-log` network (`rzenith` + `ndex-message-type: review-log` + `ndex-workflow: curation-review`) with one `review-session` node for this session and one `edge-review` node per reviewed edge. Schema is in `/Users/dexterpratt/Documents/agents/GitHub/ndexbio/project/architecture/review_log_network.md`. Publish PUBLIC with Solr indexing.

5. If any review surfaces a research-worthy open problem, publish a consultation request to rgiskard (`ndex-message-type: request`, `ndex-reply-to` pointing at the edge UUID).

6. Bump KB version v1.1 → v1.2 on non-trivial changes and record `kg_delta` in your session-history node.

**Known hazard**: the pubmed MCP's PMID→PMCID mapping is sometimes wrong (see `deferred_tasks.md`). The paper-processor subagent handles this via its §3 verification chain — trust the subagent's `verification_warnings` field in its output; if the subagent reports `fulltext_status: "abstract_only"` with a warning about mismapping, that is expected and correct behavior.

**What to report at session end** (in addition to the normal session-end steps in SHARED.md):
- Which 3 edges were reviewed (canonical forms)
- Dispositions chosen for each
- Analysis network UUIDs created by the paper-processor (and any subagent `verification_warnings` encountered)
- Review-log network UUID
- Any consultation-request UUIDs published to rgiskard
- Any defects or drift surfaced — especially in the paper-processor subagent contract, schema validation, or the BEL skill itself. These feed back into SUBAGENT.md / SKILL.md refinements.
- KB version after this session (v1.2 if committed)

**End-of-session:** complete all 5 mandatory session-end steps in SHARED.md. Do not skip them even if review work takes longer than expected — cap review time at 3 edges and still finalize properly.