---
generated: '2026-09-19'
method: generated
name: Assess a website's AI agent readiness
description: Score any website 0-100 for AI-agent discoverability with TESSA's deterministic assess_ai_readiness tool, then cross-check the result against the discovery documents TESSA itself publishes.
api: openapi/tessa-tech-agent-directory-openapi.yml
mcp: mcp/tessa-tech-mcp.yml
operations: [well_known_agent_card__well_known_agent_card_json_get, mcp_server_manifest__well_known_mcp_server_json_get]
tools: [assess_ai_readiness, get_services]
source: >-
  Grounded in the captured tools/list (assess_ai_readiness requires `url`), the agent card skill
  ai-readiness-assessment, and the OpenAPI's well-known routes. Read-only; nothing in this skill creates a lead.
---

# Assess a website's AI agent readiness

A read-only flow: run TESSA's readiness check on a URL and interpret it.

## Connect
- MCP endpoint `https://aiagent.tessa.tech/mcp/` — anonymous; `initialize` then echo `mcp-session-id`. See `conventions/tessa-tech-conventions.yml`.
- A2A alternative: the same capability is skill `ai-readiness-assessment` on the card at `https://aiagent.tessa.tech/.well-known/agent-card.json` (JSON-RPC `message/send`, inputMode `application/json`).

## Steps
1. **Run the check** — MCP `assess_ai_readiness` `{url}` (required, string). Returns a 0-100 score with category breakdowns (structured data, metadata quality, agent discoverability, AI-bot friendliness) and recommendations. The tool text states it is a "pure deterministic check, fast, no LLM call".
2. **Know what it looks for** — per the card skill description the checks cover JSON-LD/schema coverage, a well-known agent card, llms.txt, robots.txt AI directives and metadata quality. You can reproduce the discoverability part yourself: `GET /.well-known/agent-card.json` (`well_known_agent_card__well_known_agent_card_json_get`), `GET /.well-known/mcp/server.json` (`mcp_server_manifest__well_known_mcp_server_json_get`), `/llms.txt`, `/robots.txt` on the target host.
3. **Optional: map findings to services** — MCP `get_services` with `category_slug` `ai-experiences` lists TESSA's AI Agent Readiness offering (quoted per engagement per `plans/tessa-tech-plans-pricing.yml`). Do NOT call any lead tool unless the human asks — see the booking skill.

## Rules
- Read-only: safe to retry.
- The score is TESSA's own rubric, not a standard; present it as such alongside the raw checks in step 2.
- Errors: a missing `url` is a JSON-RPC invalid-params error; a bad session is HTTP 400 "Missing session ID" (`errors/tessa-tech-problem-types.yml`).
