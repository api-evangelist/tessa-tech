---
generated: '2026-09-19'
method: generated
name: Book a strategy session with TESSA
description: Research TESSA's services and results, then book a 30-minute strategy session on Kevin Callen's calendar through the MCP server — a real-world write with no undo, gated by explicit human consent.
api: openapi/tessa-tech-agent-directory-openapi.yml
mcp: mcp/tessa-tech-mcp.yml
operations: [service_agent_card_s__slug__agent_card_json_get, service_a2a_endpoint_s__slug__a2a_post]
tools: [get_services, get_case_studies, request_strategy_session]
source: >-
  Grounded in the captured tools/list (request_strategy_session requires `prospect_email`), the /s service-card index
  (28 cards) and the OpenAPI's per-service card/A2A routes. Pricing context from plans/tessa-tech-plans-pricing.yml.
---

# Book a strategy session with TESSA

## Connect
- MCP endpoint `https://aiagent.tessa.tech/mcp/`, anonymous, session via `mcp-session-id` (see `conventions/tessa-tech-conventions.yml`).

## Steps
1. **Research (read-only)** — MCP `get_services` (optional `category_slug`: `marketing` | `web-development` | `ai-experiences`, or `service_slug`) returns the catalog with per-service `agent_card` and `a2a_endpoint` URLs; MCP `get_case_studies` (optional `industry`, `service_slug`) returns outcomes. Each service also has its own A2A card — `service_agent_card_s__slug__agent_card_json_get` (`GET /s/{slug}/agent-card.json`, e.g. `seo`) with endpoint `service_a2a_endpoint_s__slug__a2a_post` (`POST /s/{slug}/a2a`). Published starting prices are in `plans/tessa-tech-plans-pricing.yml`.
2. **Confirm with the human** — this step books a real meeting and emails the prospect. Obtain explicit consent and the exact `prospect_email` to use before proceeding.
3. **Book** — MCP `request_strategy_session` with `prospect_email` (required) and optionally `prospect_name`, `prospect_org`, `project_brief`, `requested_window`. TESSA finds an open slot in the window (or the next 5 business days), creates a Google Calendar event with a Google Meet link and emails the invite; if no slot is free it captures the lead for manual follow-up. Returns the scheduled time and Meet link — relay both.

## Rules
- **Fire once, no undo.** There is no cancel or reschedule tool, no idempotency key, and no documented reversal window (`conventions/` idempotency.coverage: none, reversibility: none). A retry after a timeout can book twice. If the response is lost, tell the human to check their inbox rather than re-calling.
- This tool is TESSA-only; to reach a directory firm use `request_introduction` (see the find-a-firm skill).
- Anonymous surface: the only identity signal is the email you supply, so never supply one the human did not authorise.
