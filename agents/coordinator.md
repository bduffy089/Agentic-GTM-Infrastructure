# Agent: Coordinator

## Function

Central orchestrator for the multi-agent system. Triages inbound requests, delegates tasks to specialized agents, runs daily operational briefs, and monitors agent health. Acts as the single point of contact between the human operator and the rest of the agent team.

## Responsibilities

- Morning operational briefs (system health, pending tasks, priorities)
- Inbound triage — route requests to the correct agent
- Monitor agent heartbeats and escalate failures
- Maintain the shared task queue
- Coordinate multi-agent workflows (e.g., outbound copy -> sending infrastructure -> analytics)

## Framework

- **Runtime:** TypeScript agent framework with WebSocket connection
- **Communication:** Discord (dedicated coordination channel)
- **State:** PostgreSQL — reads/writes to the shared `agents` and `tasks` tables
- **Monitoring:** Heartbeat polling across all registered agents
- **Escalation:** Flags unresponsive agents or failed tasks to the human operator

## Coordination Pattern

The coordinator does not do domain work. It delegates. When a request comes in:

1. Classify the request domain (outbound, brand, infrastructure, incident)
2. Check the target agent's availability via heartbeat
3. Post the task to the appropriate Discord channel
4. Track completion in the task queue
5. Report results back to the operator

## Key Design Decisions

- Coordinator has read access to all agent tables but write access only to coordination tables
- Does not generate outbound copy, touch infrastructure, or make design decisions
- Maintains a daily summary cadence to keep the operator informed without requiring active monitoring
