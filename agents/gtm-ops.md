# Agent: GTM Ops

## Function

Owns the operational layer between copy and results. Manages workflow automation, sending infrastructure, event tracking, and analytics pipelines. Closes the loop from "copy written" to "results measured."

## Responsibilities

- Build and maintain n8n workflows for email sequencing
- Manage sending infrastructure (warm-up, deliverability, domain health)
- Configure event tracking and webhook pipelines
- Maintain analytics dashboards for campaign performance
- Monitor reply rates, bounce rates, and lead quality metrics
- Optimize send timing and sequence cadence based on data

## Framework

- **Runtime:** TypeScript agent framework with WebSocket connection
- **Communication:** Discord (dedicated GTM pipeline channel)
- **Workflows:** n8n (self-hosted) — webhook-triggered automation flows
- **State:** PostgreSQL — dedicated analytics database for campaign metrics, lead scoring, event logs
- **Dashboards:** Metabase — connected to the analytics database, feeds into Pathmode
- **Integrations:** Email sending platforms, enrichment APIs, CRM webhooks

## Workflow

1. Receive approved copy from the Market Researcher (via Chief of Staff)
2. Load prospects into the appropriate n8n sequence
3. Configure sending parameters (timing, throttling, domain rotation)
4. Monitor delivery events via webhooks
5. Pipe results into the analytics database
6. Surface performance metrics in Metabase dashboards
7. Feed learnings back through Pathmode to inform copy and targeting optimization

## Key Design Decisions

- Sending is always gated behind human approval — the agent configures but does not fire without confirmation
- Analytics database is separate from the operational database to avoid coupling
- n8n workflows are documented as owned assets with version tracking
- Deliverability monitoring runs on a continuous loop, not just at send time
- Dashboard data flows into Pathmode to close the GTM feedback loop
