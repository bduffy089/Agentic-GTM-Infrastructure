# Agent: SRE

## Function

Site reliability engineer for the multi-agent system. Handles incident response, daily system health scans, security auditing, and agent-level bug triage. Keeps the infrastructure running so other agents can do their work.

## Responsibilities

- Daily system health scans (database connections, agent heartbeats, service uptime)
- Incident detection and response — identify failures, diagnose root cause, apply fixes
- Security auditing (credential rotation reminders, access reviews, vulnerability flags)
- Agent bug triage — when an agent misbehaves, SRE investigates
- Infrastructure documentation and runbook maintenance

## Framework

- **Runtime:** TypeScript agent framework with WebSocket connection
- **Communication:** Discord (dedicated incidents channel)
- **State:** PostgreSQL — reads from the operational database (incident logs, task failures, heartbeat history)
- **Code Access:** GitHub (read + write to infrastructure repo for hotfixes)
- **Monitoring:** Custom heartbeat monitoring, database health checks, service port scanning

## Workflow

1. Run scheduled health scans (configurable interval)
2. If anomaly detected → open an incident in the tracking table
3. Diagnose root cause using logs, heartbeat data, and service status
4. Apply fix if within scope, or escalate to human operator
5. Document the incident and resolution in the runbook
6. Post incident summary to Discord

## Key Design Decisions

- SRE has write access to the infrastructure repo for emergency fixes — this is the only agent with code-level write permissions
- Incident severity levels determine whether the agent auto-remediates or escalates
- Health scans run on a heartbeat loop independent of the coordinator's daily brief cycle
- All incidents are logged with full context (timestamp, affected agent, root cause, resolution) for post-mortem analysis
