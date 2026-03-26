# GTM Automation Architecture

A multi-agent system for automating go-to-market operations. Five specialized AI agents coordinate through a shared runtime to handle outbound, brand, infrastructure, reliability, and orchestration — replacing fragmented manual workflows with an autonomous operations layer.

## What This Is

This repo documents the architecture and agent design patterns behind a production multi-agent GTM system. Each agent owns a specific domain, communicates via a shared message bus (Discord), persists state in PostgreSQL, and operates semi-autonomously under human oversight.

No proprietary code is included. This is a reference architecture.

## System Overview

```
┌─────────────────────────────────────────────────────┐
│                   Human Operator                     │
│              (strategy + final authority)             │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │    Agent: Coordinator    │
          │  (triage + delegation)   │
          └─┬──────┬──────┬──────┬─┘
            │      │      │      │
     ┌──────▼┐ ┌──▼───┐ ┌▼────┐ ┌▼─────┐
     │Outbound│ │GTM   │ │Brand│ │ SRE  │
     │Writer  │ │Ops   │ │     │ │      │
     └────────┘ └──────┘ └─────┘ └──────┘
```

## Agents

| Agent | Domain | Key Integrations |
|-------|--------|-----------------|
| [Coordinator](agents/coordinator.md) | Triage, delegation, daily briefs, agent orchestration | Discord, PostgreSQL, runtime API |
| [Outbound Writer](agents/outbound-writer.md) | Cold email copy, follow-ups, LinkedIn messaging, A/B variants | LLM API, CRM enrichment tools |
| [GTM Ops](agents/gtm-ops.md) | Workflow automation, sending infrastructure, analytics pipelines | n8n, PostgreSQL, analytics dashboards |
| [Brand](agents/brand.md) | Visual identity, landing page aesthetics, brand consistency | Design tools, frontend frameworks |
| [SRE](agents/sre.md) | Incident response, system health, security auditing, bug triage | PostgreSQL, monitoring, GitHub |

## Architecture Docs

- [architecture.md](architecture.md) — Coordination patterns, data flow, and runtime design

## Stack

- **Runtime:** TypeScript-based agent framework with WebSocket gateway
- **Coordination:** Discord (channel-per-agent pattern)
- **State:** PostgreSQL (operational database + analytics database)
- **Workflows:** n8n (self-hosted, webhook-driven)
- **Dashboards:** Metabase (connected to analytics DB)
- **Hosting:** Mix of local services and cloud deployments

## Design Principles

1. **Single-owner domains** — Each agent owns one operational area. No overlap.
2. **Human-in-the-loop** — Agents draft; humans approve. No unsupervised sends.
3. **Shared state, separate concerns** — All agents read/write to the same databases but own different tables.
4. **Observable by default** — Every agent logs heartbeats, task completions, and failures to a central store.
5. **Personality is a feature** — Agents have distinct operational styles defined in identity files. This isn't cosmetic — it shapes how they prioritize, communicate, and escalate.

## License

MIT
