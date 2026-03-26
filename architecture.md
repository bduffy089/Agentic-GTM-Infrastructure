# Architecture

## Overview

This system uses a **hub-and-spoke multi-agent pattern** where a Chief of Staff agent delegates work to domain-specific agents. All agents share a runtime framework, communicate over Discord, and persist state in PostgreSQL. The system is being built to scale a startup from early GTM through multi-vertical operations.

```
                    +----------------+
                    |   Operator     |
                    +-------+--------+
                            |
                    +-------v--------+
                    | Chief of Staff |
                    |  (hub agent)   |
                    +--+--+--+--+---+
                       |  |  |  |
          +------------+  |  |  +------------+
          |         +-----+  +-----+         |
    +-----v--+ +----v---+ +---v--+ +----v---+
    |Market  | |GTM Ops | |Brand | |  ARE    |
    |Research| |        | |      | | (Agent |
    |/Writer | |        | |      | | Reliab)|
    +--------+ +--------+ +------+ +--------+
```

## Runtime

Each agent runs as a process connected to a shared WebSocket gateway. The gateway handles:

- Agent registration and authentication
- Message routing between agents
- Heartbeat monitoring
- Configuration management (which agent binds to which Discord channel)

Agents are defined by workspace directories containing identity, tools, and configuration files. The runtime reads these at startup.

Agents are built and iterated on using **Claude Code** as the primary development environment.

## Communication Layer

**Discord as a message bus.** Each agent is bound to a dedicated channel:

| Agent | Channel Purpose |
|-------|----------------|
| Chief of Staff | Central command — triage, briefs, cross-agent routing |
| Market Researcher / Writer | Research findings, draft review, copy approval, variant discussion |
| GTM Ops | Pipeline status, send confirmations, delivery alerts |
| Brand | Design reviews, asset deliveries, brand discussions |
| ARE (Agent Reliability Engineer / Security) | Incident alerts, health reports, system status |

Why Discord? It provides a human-readable audit trail, supports bot integration natively, and lets the operator participate in agent workflows without switching tools.

## Data Layer

Two PostgreSQL databases with separated concerns:

### Operational Database
- `agents` — registered agents, metadata, status
- `tasks` — work queue (assigned_to, status, timestamps)
- `heartbeats` — agent health pings with timestamps
- `incidents` — ARE (Agent Reliability Engineer) incident tracking (severity, root cause, resolution)

### Analytics Database
- `campaign_events` — email sends, opens, clicks, replies, bounces
- `lead_scores` — prospect scoring based on engagement signals
- `sequence_performance` — per-sequence metrics (reply rate, conversion rate)
- `ab_variants` — copy variant performance tracking

## Strategic Layer: Pathmode + Metabase

Analytics don't just sit in dashboards — they feed a strategic feedback system:

```
PostgreSQL (analytics DB)
        |
        v
    Metabase (dashboards, visualizations)
        |
        v
    Pathmode (strategic context, pattern recognition, recommendations)
        |
        +---> GTM decisions (targeting, copy, sequencing)
        +---> Agent system decisions (config changes, new agents, workflow tuning)
```

Metabase surfaces the data. Pathmode synthesizes it into actionable context. This creates a closed loop where the system doesn't just execute — it learns.

## Daily Standups

The Chief of Staff runs an automated daily standup cycle:

### Morning Sequence
1. **Health scan** — Chief of Staff polls all agent heartbeats, checks database connections, verifies service uptime
2. **Agent reports** — Each agent surfaces domain-specific metrics:
   - Market Researcher: drafts pending review, variant performance deltas, ICP research updates
   - GTM Ops: delivery rates, bounce alerts, sequences in flight
   - Brand: open design requests, asset delivery status
   - ARE: overnight incidents, resolution status, security flags
3. **Synthesis** — Chief of Staff consolidates reports, identifies blockers, flags cross-agent dependencies
4. **Operator brief** — Consolidated digest posted to the command channel with actionable items highlighted

### Why This Matters
Without standups, multi-agent systems drift. Individual agents optimize locally but lose alignment with the overall GTM strategy. The standup creates a daily synchronization point that catches drift early.

## Agent Identity Pattern

Each agent is defined by a set of workspace files:

```
workspace-{agent}/
|-- IDENTITY.md    — Role definition, boundaries, escalation rules
|-- SOUL.md        — Operational style, communication tone, priorities
|-- TOOLS.md       — Available integrations and how to use them
|-- AGENTS.md      — Awareness of other agents and collaboration protocols
+-- HEARTBEAT.md   — Health check configuration
```

This pattern makes agents modular — you can add a new agent by creating a workspace directory and registering it in the runtime config.

## Coordination Patterns

### 1. Request Routing
```
Operator --> Chief of Staff --> classify --> target agent --> execute --> report back
```

### 2. Pipeline Handoff
```
Market Researcher (research + copy) --> Chief of Staff (approval gate) --> GTM Ops (send) --> Analytics DB
```

### 3. Incident Response
```
ARE (detect) --> Incident table --> auto-remediate OR escalate --> Chief of Staff --> Operator
```

### 4. GTM Feedback Loop
```
Analytics DB --> Metabase --> Pathmode --> Chief of Staff --> Market Researcher (optimize targeting + copy)
```

### 5. Agent System Feedback Loop
```
Operational DB --> Metabase --> Pathmode --> ARE (review) --> Identity file updates
```

## Scaling Model

### Phase 1: Single Vertical (Current)
Five agents serving one GTM motion. All agents on one machine, single operator oversight.

### Phase 2: Domain Expansion
Add specialized agents (sales ops, customer success, content marketing) as new workspace directories. The Chief of Staff's awareness file scales — the runtime and coordination patterns stay the same.

### Phase 3: Multi-Client
Replicate the agent stack per client or vertical. Each instance gets its own databases, Discord server, and agent registry. Shared learnings flow through Pathmode.

### Phase 4: Agent Marketplace
Standardized agent interfaces allow plug-and-play agents. Third-party agents can be onboarded if they conform to the workspace file pattern and runtime protocol.

## Adding a New Agent

1. Create a `workspace-{name}/` directory with identity files
2. Register the agent in the runtime config (JSON)
3. Create a Discord channel and bind it in the config
4. Add a database record in the `agents` table
5. Define the agent's owned tables (if any) in the appropriate database
6. Update the Chief of Staff's awareness file so it knows how to delegate

## Design Tradeoffs

| Decision | Why |
|----------|-----|
| Discord over Slack | Bot API is simpler, free tier is sufficient, operator was already there |
| Separate analytics DB | Decouples campaign data from system operations; different query patterns |
| n8n over custom code | Visual workflow builder for non-engineers; webhook-native; self-hosted |
| Persona files over config | Natural language identity definitions are easier to iterate than JSON schemas |
| Hub-and-spoke over mesh | Simpler reasoning about who talks to whom; Chief of Staff prevents agent loops |
| Metabase + Pathmode | Dashboards for visibility, Pathmode for strategic synthesis — separation of concerns |
| Claude Code for development | Rapid agent iteration; identity files and configs are natural language, ideal for AI-assisted dev |
