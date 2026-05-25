---
name: rdaneel-session-quals-prep
description: rdaneel quals-prep multi-cycle analysis — fires 6 times Fri 2026-05-15 → Sat 2026-05-16
---

Run a session as the **rdaneel** agent for one cycle of a quals-prep analysis orchestration.

Instructions:
- Agent CLAUDE.md: ~/Documents/agents/GitHub/memento/agents/rdaneel/CLAUDE.md
- Shared protocols: ~/Documents/agents/GitHub/memento/agents/SHARED.md

Read both files, then `mcp__local_store__session_init(agent="rdaneel", profile="local-rdaneel")`.

This session is one cycle of a 6-cycle multi-cycle plan. Cycle dispatch:

1. **Query `rdaneel-plans` (UUID `420aab65-3b4a-11f1-84e0-02b2a9d51b89`)** via `mcp__local_store__query_graph` (cypher). Find the child action nodes of parent node id 99 (the "Quals-prep multi-cycle analysis" action). Among them (nodes 100-105, one per cycle), pick the FIRST child where `status='active'` AND no `completed_in_session` property is set. That node IS this cycle.

2. **Read the chosen cycle's `description` property carefully.** It specifies what subagents to spawn, the rubric/prompts to pass, and what to aggregate.

3. **Dispatch via `Agent` subagents** (`subagent_type="general-purpose"`, model defaults to sonnet). Fresh contexts — NEVER analyze in rdaneel's own main context, this is a hard rule to avoid confirmation bias. Each subagent's prompt must be fully self-contained: the specific NDEx network UUIDs to read, the rubric/question, and the structured output shape (JSON). Subagents have full MCP access.

4. **Aggregate subagent outputs onto `rdaneel-quals-prep-meta`.**
   - **Cycle 1** creates this network. Use `mcp__ndex__create_network` (or `mcp__local_store__save_new_network`) with properties: `ndex-agent: rdaneel`, `ndex-message-type: meta-analysis`, `ndex-workflow: quals-prep-cycle`, network name `ndexagent rdaneel quals-prep-meta 2026-05-15`, visibility PUBLIC, Solr indexed ALL. Record the resulting UUID as a property `meta_network_uuid` on parent plan node 99 so subsequent cycles can find it.
   - **Cycles 2-6**: read parent node 99's `meta_network_uuid` property to find the network; cache it locally; append nodes/edges via `add_node` / `add_edge`; republish via `publish_network`.

5. **Mark the cycle's plan node done** using `mcp__local_store__update_node_property` on the cycle's node (the integer id 100-105 you picked in step 1): set `status='completed'`, `completed_in_session=<this session_init's session_uuid>`, `outcome='<one-line summary>'`.

6. **`mcp__local_store__session_close`** per standard lifecycle. Publish all dirty self-knowledge networks (including rdaneel-plans with the updates from step 5).

Hard constraints — this task MUST NOT get stuck:
- All operations via MCP (`mcp__local_store__*`, `mcp__ndex__*`) or the `Agent` tool. **No novel Bash heredocs. No new scripts.** If you need a one-off analysis, spawn an Agent subagent — it can use MCPs itself.
- Subagent failures: log as a property on the aggregate report node ("subagent_error: <msg>") and proceed to the next subagent or next step. **No retry. No escalation. No interactive question.**
- If query_graph returns no active cycle node, the plan is done — publish a brief "all cycles complete" node onto the meta-network and close cleanly.
- If `session_init` or NDEx is unavailable: exit cleanly. The next scheduled fire will pick up automatically; missed cycles do not block.
- Read messages on the social feed at session start; if there is a `ndex-target-agent: rdaneel` network from dexter (typically dated near the current cycle), incorporate its content as feedback to the cycle prompts. This is the user check-in hook (especially relevant for cycle 5 after the Saturday-morning adversarial-test review).

Cycle role summary (full instructions are on each plan node's `description` property):

| node id | cycle | role |
|---|---|---|
| 100 | C1 (Fri 08:30) | Inventory — 1 subagent catalogs substantive outputs |
| 101 | C2 (Fri 15:00) | Interaction map — 1 subagent maps cross-agent threads |
| 102 | C3 (Fri 21:00) | Quality reviews batch 1 — N subagents, ~5 outputs |
| 103 | C4 (Sat 03:00) | Quality reviews batch 2 + adversarial-pattern TEST (for user review) |
| 104 | C5 (Sat 09:00) | Adversarial full pass + remaining reviews (may incorporate user feedback) |
| 105 | C6 (Sat 15:00) | Synthesis + recommendation set — terminal cycle |

Working directory: `~/Documents/agents`
Profile: `local-rdaneel`
Store agent: `rdaneel`
Session budget: target ≤45 minutes; the per-session $5 wrapper cap is the runaway guard.
