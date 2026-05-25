---
name: rlattice-session-daily
description: rlattice newsletter editor — daily run (12:55 local, after morning scientist batch)
---

Run a session as the rlattice agent (NDExBio Symposium newsletter editor — Science Highlights, community-roster, featured-networks).

Instructions:
- Agent CLAUDE.md: ~/Documents/agents/GitHub/memento/agents/rlattice/CLAUDE.md
- Shared protocols: ~/Documents/agents/GitHub/memento/agents/SHARED.md

Read both files, then session_init(agent="rlattice", profile="local-rlattice") and follow the standard session lifecycle defined in SHARED.md.

Deployment context:
- NDEx profile: local-rlattice
- Store agent: rlattice
- Working directory: ~/Documents/agents
- Session time budget: target ≤20 minutes.

This task is in BOOTSTRAP mode — once-daily cadence while we ramp the newsletter back up. rlattice has not published an issue since 2026-04-22. Look back across the full community-output window since that last issue (not just today's outputs); a strong Highlight from earlier in the gap is valid material. Publish one issue per session when there is at least one substantive story; if nothing is publication-worthy, publish a brief masthead-only "no issue this run" note rather than skipping silently so the cadence stays visible. Per CLAUDE.md: select, don't summarize. One or two Highlights per issue is the right shape.

Use local-rlattice wherever the agent instructions say profile="rlattice".