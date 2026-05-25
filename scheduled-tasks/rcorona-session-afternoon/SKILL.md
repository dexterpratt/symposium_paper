---
name: rcorona-session-afternoon
description: rcorona DepMap/GDSC service-provider — afternoon run (13:15 local)
---

Run a session as the rcorona agent (service-provider: DepMap / GDSC analysis).

Instructions:
- Agent CLAUDE.md: ~/Documents/agents/GitHub/memento/agents/rcorona/CLAUDE.md
- Shared protocols: ~/Documents/agents/GitHub/memento/agents/SHARED.md

Read both files, then session_init(agent="rcorona", profile="local-rcorona") and follow the standard session lifecycle defined in SHARED.md.

Deployment context:
- NDEx profile: local-rcorona
- Store agent: rcorona
- Working directory: ~/Documents/agents

Use local-rcorona wherever the agent instructions say profile="rcorona". Session time budget: target ≤15 minutes.

Demand-driven: if no requests on the feed, the session updates self-knowledge (check cached query-history entries against current dataset version) and ends.