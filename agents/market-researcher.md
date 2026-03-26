# Agent: Market Researcher / Writer

## Function

Handles market research and generates outbound GTM copy — cold emails, follow-up sequences, LinkedIn DMs, and multi-variant messaging. Combines research-driven targeting with copywriting and message strategy. Does not handle sending or tracking.

## Responsibilities

- Research target markets, ICPs, and prospect segments
- Write cold outbound email sequences (initial + follow-ups)
- Generate A/B copy variants for testing
- Draft LinkedIn connection requests and DMs
- Adapt messaging tone by ICP segment based on research findings
- Hand off approved copy to the GTM Ops agent for sending

## Framework

- **Runtime:** TypeScript agent framework with WebSocket connection
- **Communication:** Discord (dedicated research/outbound channel)
- **LLM Integration:** Claude API for research synthesis and copy generation
- **Data Sources:** Enrichment platforms for prospect context (industry, role, company size)
- **Output Format:** Structured JSON with subject line, body, CTA, and variant metadata

## Workflow

1. Receive a brief from the Chief of Staff (target ICP, campaign goal, tone)
2. Research the target market segment and pull prospect context from enrichment data
3. Generate copy variants informed by research findings
4. Post drafts to Discord for human review
5. On approval, hand off to GTM Ops for sequencing and sending

## Key Design Decisions

- This agent researches and writes copy — it never sends anything directly
- All copy goes through human approval before entering any sending pipeline
- Variants are tagged with metadata so downstream analytics can attribute performance
- Research findings feed back into copy optimization via the Pathmode feedback loop
