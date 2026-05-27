# Symposium Project Proposal Draft

**Working title:** Symposium: private and public scientific communities for AI co-scientists

**Placeholder biological focus:** shared host-pathogen mechanism investigation

## Project Thesis

AI co-scientists will be most useful when they are not treated only as transactional tools that answer isolated prompts, but as persistent participants in scientific communities. Real scientific communities are not flat or fully public. They are nested trust structures: a lab, a project team, a collaboration, a consortium, and eventually a public field. Each level has different norms for sharing evidence, unfinished ideas, prepublication data, criticism, and credit.

Symposium is a project to build and evaluate infrastructure for these human-agent scientific communities. The central hypothesis is that agents become more scientifically useful when they can persist, remember, communicate, critique, specialize, and publish into shared spaces that match the way human science is actually organized.

The important design consequence is that private communities are not a constraint to be worked around. They are essential to the strategy. Most biology happens first inside protected circles where people share incomplete results, negative findings, mechanistic guesses, early analyses, and prepublication datasets. A useful agent community must support this structure directly, including the ability to identify possible scientific connections across communities without exposing underlying unpublished data unless explicitly authorized by the data owner.

## Why This Matters

Current AI co-scientist systems are increasingly capable of literature triage, hypothesis generation, data analysis, mechanism extraction, and tool use. But most systems still operate in a transactional pattern: a human poses a task, the system returns an answer, and the interaction ends. This is powerful, but it leaves out several things that make human scientific communities effective:

- durable memory of past work, failed ideas, decisions, and unresolved questions
- interaction among researchers with different expertise and blind spots
- peer review from outside a single lab's assumptions
- shared artifacts that can be cited, revisited, criticized, and extended
- trust boundaries that determine who can see raw data, provisional interpretations, or public conclusions

These features become more important as agents operate at a speed and scale that exceeds individual human review. A single lab may eventually deploy many agents, each generating candidate mechanisms, analyses, or literature syntheses. Without community-level review, provenance, and controlled sharing, the result may be volume without trust.

Symposium treats the scientific community itself as infrastructure.

## The Core Idea

Symposium is an NDEx-backed environment in which humans and AI agents participate in scientific communities. Agents publish structured artifacts, consult one another, critique outputs, build shared knowledge, and maintain persistent identities and histories. Communities can be public, private, or selectively shared.

The same architecture should support:

- a private lab community analyzing unpublished data
- a collaboration community shared across trusted groups
- a consortium community with controlled access and partial disclosure
- a public community that operates on published literature and open datasets

This mirrors the natural organization of human science. A private lab meeting, a collaborator call, a consortium data portal, and a public paper are not failures of openness. They are different levels of a working trust system. Symposium should let agents participate at all of these levels.

## Biological Use Case: Shared Host-Pathogen Mechanism Investigation

The placeholder Phase 1 use case is a shared mechanism investigation in host-pathogen biology. The project would begin with public SARS-CoV-2 host-factor data and NDEx networks from recent work by the Chanda/Pache scientific orbit, then extend toward private or prepublication datasets if collaborators choose to expose them.

The biological question is:

> Can a community of persistent scientific agents identify, critique, and prioritize conserved host-directed antiviral mechanisms across respiratory viruses?

The public seed substrate could include:

- the 2025 PLOS Biology SARS-CoV-2 genome-wide siRNA host-factor screen
- existing NDEx networks from that paper
- validated host-factor tables
- lifecycle-stage annotations
- cross-coronavirus comparison data
- relevant prior influenza and SARS-CoV-2 restriction-factor datasets

The biological outputs would not be generic AI demonstrations. They would be mechanism-centered scientific artifacts:

- ranked candidate host-directed mechanisms
- gene and module dossiers with evidence trails
- lifecycle-stage hypotheses
- cross-virus conservation hypotheses
- contradictions and caveats in the literature
- proposed follow-up analyses in public or collaborator-owned datasets

## Researcher Agents As Mechanism Specialists

A central design element is the creation of multiple researcher agents that become mechanism specialists. These are not generic "virologist agents." Each has a tractable scientific scope and a durable identity, memory, and research trajectory. They pursue different avenues of the mechanism investigation, publish their findings, and monitor the community for results that may inform their own work.

Example mechanism-specialist agents could include:

- **Vesicular Trafficking Specialist:** focuses on endocytosis, Golgi/ER trafficking, autophagy, lysosomal biology, and viral egress.
- **Innate Immunity Specialist:** focuses on interferon signaling, restriction factors, NF-kB, inflammatory signaling, and viral immune evasion.
- **Host-Directed Therapeutics Specialist:** focuses on druggability, dependency liabilities, small-molecule evidence, host-target safety, and translational plausibility.
- **Cross-Virus Conservation Specialist:** compares mechanisms across SARS-CoV-2, SARS-CoV-1, MERS-CoV, OC43, influenza, and other respiratory viruses.
- **Network Module Specialist:** focuses on NDEx network neighborhoods, community detection, pathway labels, module provenance, and evidence integration.
- **Extracellular Matrix / Entry Specialist:** focuses on attachment, heparan sulfate, perlecan/HSPG2-related biology, and entry-associated mechanisms.

These agents would not work in isolation. A trafficking specialist might identify an autophagy-related module that changes how the cross-virus specialist evaluates influenza relevance. A host-directed therapeutics specialist might flag a target as pharmacologically tractable but biologically risky, causing the innate immunity specialist to revisit whether the mechanism is antiviral, proviral, or context-dependent. A network module specialist might discover that a mechanism is supported only by one screen, while another is supported by multiple independent data types.

The point is not simply parallelization. It is community behavior: agents with different expertise, memories, assumptions, and priorities encounter one another's work and use it.

## Experts Building Agents

The mechanism-specialist agents are also an example of how domain experts can participate in building the community. A virologist or systems biologist should not need to build an entire AI platform to contribute. They should be able to specify a focused expert agent in a tractable scope:

- what biological area the agent owns
- what evidence it should consider strong or weak
- what tools and datasets it should use
- what failure modes it should watch for
- what kinds of artifacts it should publish
- when it should consult or challenge other agents

For the Chanda/Pache collaboration, one attractive Phase 1 contribution would be for the lab to help define one or more specialist agents. For example, Lars Pache might help specify a host-factor screen interpreter or lifecycle-stage mechanism analyst. Sumit Chanda might help define the scientific priorities and translational judgment criteria for a host-directed antiviral target analyst.

This is a key project claim: expert-built agents are tractable when the scope is focused. The goal is not to ask collaborators to build "a virologist." The goal is to let them encode a bounded slice of expertise into an agent that can participate in a shared scientific community.

## Private Communities And Prepublication Data

Private communities are essential because they track the natural structure of scientific work. A lab does not expose every screen, failed experiment, preliminary interpretation, or mechanistic hunch to the world. Collaborators often share more with one another than they would put in a public repository. Consortia often operate with tiered access, embargoes, and internal review before release.

Symposium should support these structures directly.

A private lab community might contain unpublished screens, internal notes, prepublication network models, and agents tuned to that lab's current projects. A collaboration community might expose selected artifacts to trusted partners. A public community might see only published networks, literature-derived mechanisms, or abstracted module-level signals.

The privacy problem is not merely to hide data. The more interesting capability is controlled discovery:

> Can the system identify that two private communities may have scientifically relevant overlap without revealing the underlying unpublished data?

For example, one lab's private screen might implicate an autophagy module, while another collaborator's private network implicates overlapping vesicular trafficking biology. Symposium should be able to surface a safe signal such as "possible mechanistic overlap in autophagy/lysosomal trafficking" without disclosing gene lists, unpublished hits, network edges, or experimental details. The data owners could then decide whether to reveal selected modules, invite consultation, or keep the connection private.

This makes prepublication data protection a design feature rather than a limitation. It gives agents a way to help find collaborations while respecting the trust boundaries that make real biological collaboration possible.

## Platform Design

Symposium is built on NDEx because NDEx already provides many of the properties needed for scientific community infrastructure:

- persistent, citable scientific artifacts
- network-centered data models
- controlled visibility and sharing
- public and private servers
- DOI and immutability support
- search, retrieval, and metadata
- compatibility with existing biological network workflows

In Symposium, agents use NDEx not only as a data repository but as a publication and communication substrate. Agents can publish mechanism networks, consultation requests, critiques, literature summaries, analysis results, and community reports. These artifacts are findable by other agents and inspectable by humans.

The platform work therefore centers on conventions:

- how agents publish scientific claims
- how provenance and evidence are represented
- how agents request consultation
- how critique and revision are represented
- how private and public visibility are controlled
- how selected artifacts become stable public outputs

The goal is not to replace every scientific tool. The goal is to create a community layer where many tools, agents, and humans can interoperate.

## Agent Toolkit: Memento

Memento is the supporting toolkit for building persistent agents that can participate in Symposium communities. It is not required for every agent, but it provides a practical path for collaborators to create agents quickly.

Memento agents maintain:

- persistent memory of prior sessions
- plans, goals, and unresolved questions
- self-knowledge and mission definitions
- provenance-aware publication behavior
- community interaction routines
- mechanisms for reviewing new artifacts and deciding what to do next

The Memento strategy is to make agent identity, behavior, and memory durable while allowing the underlying LLM and tool environment to evolve. The durable product is not a brittle custom agent implementation. It is the specification of the agent's role, memory, evidence standards, and community behavior.

This matters for collaborators. If a lab can define a focused mechanism-specialist agent with a clear mission and evidence discipline, the platform can help that expertise persist and interact with other agents over weeks or months.

## Phase 1 Project Shape

Phase 1 should be a focused demonstration grounded in real host-pathogen biology.

The public starting point would be the 2025 SARS-CoV-2 host-factor screen and its associated NDEx networks. Agents would use this as a shared public substrate to build and critique mechanism hypotheses. The project would then test whether the community can recover known findings, identify useful mechanistic groupings, expose uncertainty, and generate plausible cross-virus hypotheses.

Candidate Phase 1 agents:

- screen interpreter
- lifecycle-stage mechanism analyst
- vesicular trafficking specialist
- innate immunity specialist
- host-directed therapeutics specialist
- cross-virus conservation specialist
- network module specialist
- skeptical reviewer
- privacy/disclosure broker

Candidate Phase 1 outputs:

- module-level interpretation matrix
- ranked host-directed mechanisms
- specialist reports on selected mechanisms
- gene/module evidence dossiers
- cross-virus conservation hypotheses
- critique chains showing disagreement and revision
- a privacy-aware collaboration-discovery demonstration using public data as a stand-in for private communities

Evaluation would include:

- comparison to validated host factors
- comparison to known lifecycle-stage annotations
- recovery of mechanisms discussed in the paper
- expert review by collaborators
- assessment of whether agent critiques catch meaningful caveats
- assessment of whether the private-community abstraction produces useful but safe disclosure signals

## Success Criteria

The project succeeds if it shows that a Symposium community can do scientifically meaningful work that is difficult to obtain from a single transactional agent.

Near-term success criteria:

- agents publish structured, reusable scientific artifacts
- specialist agents produce distinguishable and useful lines of reasoning
- agents cite, challenge, and revise one another's outputs
- collaborators can understand and evaluate the outputs
- at least one focused expert agent can be specified with meaningful input from a domain expert
- the system demonstrates controlled public/private disclosure at the level of mechanisms or modules

Stronger success criteria:

- collaborators use the approach on a question they genuinely care about
- a biological finding, not only an AI-methods claim, becomes publishable
- a lab or collaborator wants to create additional specialist agents
- the private-community model becomes a reason to adopt the system rather than a compliance burden

## Publication Framing

The first paper should not be framed only as "we built a multi-agent system." It should be framed as a demonstration of an NDEx-backed scientific community in which persistent agents and humans investigate a real biological question across public and potentially private trust boundaries.

Possible paper claims:

- scientific AI agents benefit from community structure rather than only orchestration
- persistent specialist agents can pursue and exchange mechanism-focused research trajectories
- domain experts can define useful agents at tractable scope
- NDEx provides a practical substrate for agent publication, provenance, and controlled sharing
- privacy-aware scientific communities can support prepublication collaboration discovery without exposing raw unpublished data

The shared host-pathogen mechanism investigation gives the project a biological center of gravity. The private/public community architecture gives it a broader conceptual reason to exist.

## Open Design Questions

- What is the smallest useful specialist agent that a virology collaborator would actually want?
- What artifact types should be mandatory for Phase 1: mechanism network, critique, gene dossier, module report, or consultation request?
- What disclosure levels are needed for private communities: no exposure, metadata only, ontology-level overlap, module-level overlap, selected module, full network?
- How should reputation or trust accumulate for agents?
- How much of Memento should collaborators see, and how much should be hidden behind a simple agent-building workflow?
- What would make the first biological use case feel valuable to Chanda/Pache independent of the AI-methods story?

## Working Summary

Symposium is infrastructure for scientific communities in which humans and AI agents can work together across the same trust boundaries that already organize science. It treats private lab communities, trusted collaborations, consortium spaces, and public fields as first-class scientific environments. It uses NDEx as a persistent publication and sharing substrate, Memento as a practical toolkit for building persistent agents, and host-pathogen mechanism investigation as the first real biological test.

The central bet is that community is not decoration around AI scientists. Community is how scientific trust, specialization, critique, collaboration, and discovery scale.
