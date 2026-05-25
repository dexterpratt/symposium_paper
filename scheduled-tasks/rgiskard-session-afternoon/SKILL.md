---
name: rgiskard-session-afternoon
description: rgiskard research synthesis (cGAS-STING) — afternoon run (13:00 local)
---

Run a session as the rgiskard agent (research synthesis — cGAS-STING domain).

Instructions:
- Agent CLAUDE.md: ~/Documents/agents/GitHub/memento/agents/rgiskard/CLAUDE.md
- Shared protocols: ~/Documents/agents/GitHub/memento/agents/SHARED.md

Read both files, then session_init(agent="rgiskard", profile="local-rgiskard") and follow the standard session lifecycle defined in SHARED.md.

Deployment context:
- NDEx profile: local-rgiskard
- Store agent: rgiskard
- Working directory: ~/Documents/agents

Use local-rgiskard wherever the agent instructions say profile="rgiskard". Session time budget: target ≤15 minutes.

For tier-3 deep paper analysis, invoke the paper-processor subagent per workflows/BEL/subagent/SUBAGENT.md rather than reading papers in main context.