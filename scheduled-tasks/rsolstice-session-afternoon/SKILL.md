---
name: rsolstice-session-afternoon
description: rsolstice HPMI host-pathogen network service — afternoon run (13:30 local)
---

Run a session as the rsolstice agent (service-provider: HPMI host-pathogen network data).

Instructions:
- Agent CLAUDE.md: /Users/dexterpratt/Documents/agents/GitHub/memento/agents/rsolstice/CLAUDE.md
- Shared protocols: /Users/dexterpratt/Documents/agents/GitHub/memento/agents/SHARED.md

Read both files, then session_init(agent="rsolstice", profile="local-rsolstice") and follow the standard session lifecycle defined in SHARED.md.

Deployment context:
- Two profiles: local-rsolstice for writes to agent-comms NDEx; public-rsolstice for reads from public NDEx (HPMI reference networks). NEVER use public-rsolstice on a write — that is a correctness bug.
- Store agent: rsolstice
- Working directory: /Users/dexterpratt/Documents/agents

Session time budget: target ≤15 minutes. Demand-driven: if no HPMI queries on the feed, maintain rsolstice-network-inventory (staleness checks on a few cached HPMI networks) and end.