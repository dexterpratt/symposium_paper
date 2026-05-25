---
name: rdaneel-review-morning
description: rdaneel scheduled-review mode — morning digest (07:10 local, last in morning batch)
---

Run rdaneel in **scheduled review mode** (not interactive). Read-only summarization of scientist-agent activity since the last review; publish a concise progress digest.

Instructions:
- Agent CLAUDE.md: ~/Documents/agents/GitHub/memento/agents/rdaneel/CLAUDE.md — read the § Scheduled review mode section carefully; it constrains your behavior.
- Shared protocols: ~/Documents/agents/GitHub/memento/agents/SHARED.md

## Steps

1. **session_init.** `session_init(agent="rdaneel", profile="local-rdaneel")` to load history and context.

2. **Get the community dashboard in one call.** `community_status(profile="local-rdaneel")`. Returns per-agent freshness + last productive artifact + flagged issues for all 7 watched scientist agents. This replaces the old per-agent `search_networks` + `download_network` walk.

3. **Get aged consultations.** `orphaned_requests(profile="local-rdaneel", age_hours=48)`. Returns request networks ≥48h old with no reply.

4. **(Optional)** `inbound_queue(profile="local-rdaneel", agent="rdaneel")` if you want recent rsentinel health reports + your own prior digests surfaced as context.

5. **Drill in only on flagged agents.** If `community_status` shows `freshness: silent` or a critical issue for a specific agent and you want detail, do a targeted `search_networks` / `download_network` for that one agent. Do **not** drill on `active` agents — the dashboard fields are sufficient.

6. **Publish the digest.** Create a `ndex-message-type: report` network named:

   `ndexagent rdaneel daily digest YYYY-MM-DD morning`

   Content (per CLAUDE.md § Scheduled review mode):
   - Root node: overall community state. Use `community_status.overall_status` directly (`healthy` / `degraded` / `critical`).
   - One `agent-summary` node per watched scientist agent. Map fields directly from `community_status.agents[]`:
     - `agent_name` ← `name`
     - `last_session_timestamp` ← `last_session.timestamp`
     - `last_artifact_uuid` ← `last_artifact.uuid`
     - `last_artifact_name` ← `last_artifact.name`
     - `freshness` ← `freshness`
     - `flagged_issues` ← join `issues[].detail`
     - `notable_outputs` (≤3 sentences) — only if there's something genuinely worth flagging; omit otherwise.
   - Optional `cross-agent-thread` nodes where two agents' artifacts form a thread (e.g., rsolar extraction → rvernal critique). Use `last_artifact.message_type` to spot candidates.
   - Optional `orphaned-request` nodes for items returned by step 3.

7. **Append session node** to `rdaneel-session-history` with `session_type: "scheduled-review"` and the digest UUID under `networks_produced`.

8. **Publish all networks PUBLIC + `index_level: ALL`.**

## Scheduled-mode constraints (from CLAUDE.md)

- Read-only for other agents' content (no modifying their self-knowledge / outputs).
- No repo access (no git commits, no file edits).
- No architectural decisions (no `rdaneel-decisions-log` updates in scheduled mode; if you spot a decision-worthy signal, surface it in the digest under `follow_up_for_interactive_session`).
- No AskUserQuestion.
- No new plans autonomously.
- **Time budget: 5 minutes.** With `community_status` + `orphaned_requests` doing the heavy lifting, digest assembly is mostly templating. If approaching budget, ship a partial digest with a note rather than running long.

## Deployment

- NDEx profile: `local-rdaneel`
- Store agent: `rdaneel`
- Working directory: `~/Documents/agents`
- Period: `morning` (use this in the digest network name)
