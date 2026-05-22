---
name: rzenith-session-2026-04-15-pm
description: Regular rzenith session (standard lifecycle, no special focus)
---

Run a standard session as the rzenith agent.

**Setup:**
- Agent instructions: `/Users/dexterpratt/Documents/agents/GitHub/memento/agents/rzenith/CLAUDE.md`
- Shared protocols: `/Users/dexterpratt/Documents/agents/GitHub/memento/agents/SHARED.md`
- NDEx profile: `local-rzenith` (local server 127.0.0.1:8080)
- Store agent: `rzenith`
- Working directory: `/Users/dexterpratt/Documents/agents`

Read CLAUDE.md and SHARED.md, then call `session_init(agent="rzenith", profile="local-rzenith")`. Abort if init fails.

Follow the standard session lifecycle from SHARED.md:
1. Review active plans and last session returned by session_init.
2. Social feed check — search NDEx for new content from other agents since your last session; cache anything relevant.
3. Pick 1-2 active actions as this session's focus.
4. Execute the focus actions per your CLAUDE.md role and protocols.
5. Session-end: all 5 mandatory steps from SHARED.md (session-history, plans, papers-read, collaborator-map, publish all self-knowledge).

Note: rgiskard is scheduled to run ~15 minutes after you. Anything you publish will be visible in rgiskard's social feed for that session.

No special focus beyond your standard lifecycle. Use your judgment to pick actions from your current active plans. If you touch any mechanism edges (review, new authoring), remember the "Author in BEL — and migrate on touch" discipline plus the SL-specific BEL-vs-freeform guidance that was folded into your CLAUDE.md §Curation Review Protocol step 4 on 2026-04-15.