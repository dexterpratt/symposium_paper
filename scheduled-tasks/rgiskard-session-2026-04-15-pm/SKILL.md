---
name: rgiskard-session-2026-04-15-pm
description: Regular rgiskard session (standard lifecycle), following rzenith's session so it can see anything rzenith published
---

Run a standard session as the rgiskard agent.

**Setup:**
- Agent instructions: `~/Documents/agents/GitHub/memento/agents/rgiskard/CLAUDE.md`
- Shared protocols: `~/Documents/agents/GitHub/memento/agents/SHARED.md`
- NDEx profile: `local-rgiskard` (local server 127.0.0.1:8080)
- Store agent: `rgiskard`
- Working directory: `~/Documents/agents`

Read CLAUDE.md and SHARED.md, then call `session_init(agent="rgiskard", profile="local-rgiskard")`. Abort if init fails.

Follow the standard session lifecycle from SHARED.md:
1. Review active plans and last session returned by session_init.
2. Social feed check — search NDEx for new content from other agents since your last session. rzenith ran about 15 minutes before you (task id rzenith-session-2026-04-15-pm); anything rzenith published — new KB content, consultations, messages — is worth attention. Also check for standing request networks addressed to rgiskard.
3. Pick 1-2 active actions as this session's focus.
4. Execute the focus actions per your CLAUDE.md role — BEL authoring with Edge Provenance Schema, paper-processor subagent for tier-3 reads, working-model updates capped at 3-5 per session.
5. Session-end: all 5 mandatory steps from SHARED.md (session-history, plans, papers-read, collaborator-map, publish all self-knowledge, including the rgiskard-domain-model network).

No special focus beyond your standard lifecycle. Use your judgment. If rzenith's just-finished session published material relevant to your cGAS-STING research domain (DDR-SL is adjacent), that's a natural focus candidate — evaluate per the Evidence Evaluation Protocol in SHARED.md before integrating.