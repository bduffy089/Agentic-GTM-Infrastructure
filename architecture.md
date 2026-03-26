# Architecture

## Overview

This system uses a **hub-and-spoke multi-agent pattern** where a central coordinator delegates work to domain-specific agents. All agents share a runtime framework, communicate over Discord, and persist state in PostgreSQL.

```
                    ┌──────────────┐
                    │   Operator   │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  Coordinator  │
                    │  (hub agent)  │
                    └──┬──┬──┬──┬──┘
                       │  │  │  │
          ┌────────────┘  │  │  └────────────┐
          │         ┌─────┘  └─────┐         │
    ┌─────▼──┐ ┌────▼───┐ ┌───▼──┐ ┌────▼───┐
    │Outbound│ │GTM Ops │ │Brand │ │  SRE   │
    │Writer  │ │        │ │      │ │        │
    └────────┘ └────────┘ └──────┘ └────────┘
```

## Runtime

Each agent runs as a process connected to a shared WebSocket gateway. The gateway handles:

- Agent registration and authentication
- Message routing between agents
- Heartbeat monitoring
- Configuration management (which agent binds to which Discord channel)

Agents are defined by workspace directories containing identity, tools, and configuration files. The runtime reads these at startup.

## Communication Layer

**Discord as a message bus.** Each agent is bound to a dedicated channel:

| Agent | Channel Purpose |
|-------|----------------|
| Coordinator | Central command — triage, briefs, cross-agent routing |
| Outbound Writer | Draft review, copy approval, variant discussion |
| GTM Ops | Pipeline status, send confirmations, delivery alerts |
| Brand | Design reviews, asset deliveries, brand discussions |
| SRE | Incident alerts, health reports, system status |

Why Discord? It provides a human-readable audit trail, supports bot integration natively, and lets the operator participate in agent workflows without switching tools.

## Data Layer

Two PostgreSQL databases with separated concerns:

### Operational Database
- `agents` — registered agents, metadata, status
- `tasks` — work queue (assigned_to, status, timestamps)
- `heartbeats` — agent health pings with timestamps
- `incidents` — SRE incident tracking (severity, root cause, resolution)

### Analytics Database
- `campaign_events` — email sends, opens, clicks, replies, bounces
- `lead_scores` — prospect scoring based on engagement signals
- `sequence_performance` — per-sequence metrics (reply rate, conversion rate)
- `ab_variants` — copy variant performance tracking

## Agent Identity Pattern

Each agent is defined by a set of workspace files:

```
workspace-{agent}/
├── IDENTITY.md    — Role definition, boundaries, escalation rules
├── SOUL.md        — Operational style, communication tone, priorities
├── TOOLS.md       — Available integrations and how to use them
├── AGENTS.md      — Awareness of other agents and collaboration protocols
└── HEARTBEAT.md   — Health check configuration
```

This pattern makes agents modular — you can add a new agent by creating a workspace directory and registering it in the runtime config.

## Coordination Patterns

### 1. Request Routing
```
Operator → Coordinator → classify → target agent → execute → report back
```

### 2. Pipeline Handoff
```
Outbound Writer (copy) → Coordinator (approval gate) → GTM Ops (send) → Analytics DB
```

### 3. Incident Response
```
SRE (detect) → Incident table → auto-remediate OR escalate → Coordinator → Operator
```

### 4. Feedback Loop
```
Analytics DB → GTM Ops (surface metrics) → Coordinator → Outbound Writer (optimize copy)
```

## Adding a New Agent

1. Create a `workspace-{name}/` directory with identity files
2. Register the agent in the runtime config (JSON)
3. Create a Discord channel and bind it in the config
4. Add a database record in the `agents` table
5. Define the agent's owned tables (if any) in the appropriate database
6. Update the coordinator's awareness file so it knows how to delegate

## Design Tradeoffs

| Decision | Why |
|----------|-----|
| Discord over Slack | Bot API is simpler, free tier is sufficient, operator was already there |
| Separate analytics DB | Decouples campaign data from system operations; different query patterns |
| n8n over custom code | Visual workflow builder for non-engineers; webhook-native; self-hosted |
| Persona files over config | Natural language identity definitions are easier to iterate than JSON schemas |
| Hub-and-spoke over mesh | Simpler reasoning about who talks to whom; coordinator prevents agent loops |
