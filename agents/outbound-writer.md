# Agent: Outbound Writer

## Function

Generates outbound GTM copy — cold emails, follow-up sequences, LinkedIn DMs, and multi-variant messaging. Focuses on copywriting and message strategy, not sending or tracking.

## Responsibilities

- Write cold outbound email sequences (initial + follow-ups)
- Generate A/B copy variants for testing
- Draft LinkedIn connection requests and DMs
- Adapt messaging tone by ICP segment
- Hand off approved copy to the GTM Ops agent for sending

## Framework

- **Runtime:** TypeScript agent framework with WebSocket connection
- **Communication:** Discord (dedicated outbound channel)
- **LLM Integration:** Claude API for copy generation with structured prompts
- **Data Sources:** Enrichment platforms for prospect context (industry, role, company size)
- **Output Format:** Structured JSON with subject line, body, CTA, and variant metadata

## Workflow

1. Receive a brief from the coordinator (target ICP, campaign goal, tone)
2. Pull prospect context from enrichment data
3. Generate copy variants with the LLM
4. Post drafts to Discord for human review
5. On approval, hand off to GTM Ops for sequencing and sending

## Key Design Decisions

- This agent writes copy only — it never sends anything directly
- All copy goes through human approval before entering any sending pipeline
- Variants are tagged with metadata so downstream analytics can attribute performance
- Prompt templates are versioned and stored alongside the agent's identity files
