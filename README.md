# Agentic GTM Infrastructure

> **Patent Pending** — The systems and processes described in this repository are the subject of a pending patent application. All rights reserved. See [LICENSE](LICENSE) for details.

A multi-agent system being built to scale a startup's go-to-market operations. Specialized AI agents coordinate through a shared runtime to handle market research, outbound, brand, infrastructure reliability, and orchestration — replacing fragmented manual workflows with an autonomous operations layer.

## What This Is

This repo documents the architecture and agent design patterns behind a production multi-agent GTM system. Each agent owns a specific domain, communicates via a shared message bus (Discord), persists state in PostgreSQL, and operates semi-autonomously under human oversight.

No proprietary code is included. This is a reference architecture.

## System Overview

```
+---------------------------------------------------------+
|                    Human Operator                        |
|              (strategy + final authority)                |
+----------------------------+----------------------------+
                             |
               +-------------v--------------+
               |    Agent: Chief of Staff    |
               |    (triage + delegation)    |
               +--+------+------+------+----+
                  |      |      |      |
       +----------+  +---+--+  ++---------+  +---+-----+
       | Market   |  | GTM  |  | Designer |  |  ARE     |
       | Research |  | Ops  |  | /Brand   |  | (Agent  |
       | /Writer  |  |      |  |          |  | Reliab.)|
       +---------+   +------+  +----------+  +---------+
```

## Agents

| Agent | Domain | Key Integrations |
|-------|--------|-----------------|
| [Chief of Staff](agents/chief-of-staff.md) | Triage, delegation, daily standups, agent orchestration | Discord, PostgreSQL, runtime API |
| [Market Researcher / Writer](agents/market-researcher.md) | Market research, cold email copy, follow-ups, LinkedIn messaging, A/B variants | LLM API, CRM enrichment tools |
| [GTM Ops](agents/gtm-ops.md) | Workflow automation, sending infrastructure, analytics pipelines | n8n, PostgreSQL, Metabase dashboards |
| [Designer/Brand](agents/brand.md) | Visual identity, landing page aesthetics, brand consistency | Design tools, frontend frameworks |
| [ARE (Agent Reliability Engineer / Security)](agents/are.md) | Agent reliability, incident response, system health, security auditing | PostgreSQL, monitoring, GitHub |

## Stack

- **Agent Development:** Claude Code (primary development environment for building and iterating on agents)
- **Runtime:** TypeScript-based agent framework with WebSocket gateway
- **Coordination:** Discord (channel-per-agent pattern)
- **State:** PostgreSQL (operational database + analytics database)
- **Workflows:** n8n (self-hosted, webhook-driven)
- **Dashboards:** Metabase (connected to analytics DB, feeds into Pathmode for strategic context)
- **Strategic Layer:** Pathmode (receives analytics from Metabase; provides feedback loops for both GTM performance and agent system improvements)
- **Hosting:** Mix of local services and cloud deployments

## Feedback Loops

This system is designed with two distinct feedback loops:

### GTM Performance Loop
```
Outbound Copy --> Send Infrastructure --> Campaign Events --> Analytics DB
      ^                                                          |
      |                                                          v
      +--- Copy Optimization <--- Metabase Dashboards <--- Pathmode
```
Campaign data flows into analytics, surfaces in Metabase, gets synthesized in Pathmode for strategic context, and feeds back into copy and targeting decisions.

### Agent System Loop
```
Agent Behavior --> Heartbeats/Logs --> Operational DB --> Metabase
      ^                                                      |
      |                                                      v
      +--- Identity File Updates <--- ARE Review <--- Pathmode
```
Agent performance data flows into dashboards, gets reviewed in Pathmode alongside GTM metrics, and informs improvements to agent configurations, identity files, and coordination patterns.

## Daily Standups

The system runs automated daily standups coordinated by the Chief of Staff agent:

- **Morning brief:** System health check, overnight incident summary, pending task queue review
- **Agent status reports:** Each agent surfaces key metrics from their domain (pipeline stats, delivery health, open incidents)
- **Priority alignment:** Chief of Staff synthesizes reports and flags blockers, dependencies, or drift from weekly objectives
- **Operator digest:** Human operator receives a consolidated brief in Discord — can intervene, reprioritize, or acknowledge

Standups create a daily rhythm that keeps the system self-aware without requiring the operator to actively monitor every channel.

## Scaling Architecture

This system is designed to scale horizontally:

```
Phase 1 (current):  5 agents, single operator, one GTM vertical
Phase 2:            Add domain agents (sales ops, customer success, content)
Phase 3:            Multi-client — replicate the agent stack per client/vertical
Phase 4:            Agent marketplace — plug-and-play agents with standardized interfaces
```

Adding a new agent follows a standard pattern:
1. Create a workspace directory with identity files (IDENTITY.md, SOUL.md, TOOLS.md, AGENTS.md)
2. Register in the runtime config
3. Bind to a Discord channel
4. Add a database record
5. Update the Chief of Staff's awareness file
6. Define owned tables (if any)

The identity file pattern means agents are modular — you can spin up a new agent in hours, not weeks.

## Design Principles

1. **Single-owner domains** — Each agent owns one operational area. No overlap.
2. **Human-in-the-loop** — Agents draft; humans approve. No unsupervised sends.
3. **Shared state, separate concerns** — All agents read/write to the same databases but own different tables.
4. **Observable by default** — Every agent logs heartbeats, task completions, and failures to a central store.
5. **Personality is a feature** — Agents have distinct operational styles defined in identity files. This isn't cosmetic — it shapes how they prioritize, communicate, and escalate.
6. **Feedback-driven** — Both the GTM output and the agent system itself improve through structured feedback loops.

## License

Copyright (c) 2026 Britney Duffy. All Rights Reserved. Patent Pending. See [LICENSE](LICENSE).
