---
generated: '2026-09-19'
method: generated
name: Find a professional-services firm and request an introduction
description: Search TESSA's professional-services directory (marketing and compliance registries), read a firm's dossier, then request a warm introduction or a quote — using TESSA's anonymous MCP server, with the REST twins where they exist.
api: openapi/tessa-tech-agent-directory-openapi.yml
mcp: mcp/tessa-tech-mcp.yml
operations: [firm_profile__slug__profile_json_get, firm_request_introduction__slug__request_introduction_post]
tools: [find_professional_services_firm, get_firm_profile, request_introduction, request_quote]
source: >-
  Grounded in the captured tools/list (mcp/tessa-tech-mcp-tools-list.json) and the OpenAPI; operationIds verified
  verbatim. Auth per authentication/tessa-tech-authentication.yml, errors per errors/tessa-tech-problem-types.yml,
  conventions per conventions/tessa-tech-conventions.yml, bindings per mcp/tessa-tech-tool-crosswalk.yml.
---

# Find a professional-services firm and request an introduction

Use TESSA's directory to locate a firm, read its profile, and hand off a lead. The MCP server is the complete surface; REST covers the profile read and the per-firm introduction write.

## Connect
- MCP endpoint: `https://aiagent.tessa.tech/mcp/` (Streamable HTTP, anonymous). Send `Accept: application/json, text/event-stream`, call `initialize`, echo the returned `mcp-session-id` header on every later request, then `notifications/initialized`. See `conventions/tessa-tech-conventions.yml` (transports).
- REST base: `https://aiagent.tessa.tech` for TESSA's own routes; firm routes (`/{slug}/...`) are host-scoped and answer **421** there — call them on `https://complianceregistry.net` or `https://marketingregistry.org`. See `errors/tessa-tech-problem-types.yml` (421).
- No credential is required anywhere in this flow (`authentication/tessa-tech-authentication.yml`).

## Steps
1. **Search** — MCP `find_professional_services_firm` with `industry` (`marketing` | `compliance` | keyword), optional `region` (city/state contains), `capability` (keyword), `limit` (integer). Returns matching firms with `slug`, website and location. There is no REST equivalent with these filters; the static index is `registry_index_registry_json_get` (`GET /registry.json`, registry hosts only).
2. **Read the dossier** — MCP `get_firm_profile` `{slug}` (required), or REST `firm_profile__slug__profile_json_get` (`GET /{slug}/profile.json` on a registry host). Both return description, headquarters, founded year, employee band, services, locations and directory meta (publisher, claim URL, takedown URL). The A2A form of the same record is `firm_card__slug__agent_card_json_get`.
3. **Hand off the lead** — pick ONE:
   - Warm intro: MCP `request_introduction` with `prospect_email` (required), `target_firm_slug`, and optionally `prospect_name`, `prospect_org`, `project_brief`, `requested_window`. REST twin: `firm_request_introduction__slug__request_introduction_post` (`POST /{slug}/request-introduction`, registry host). TESSA logs the lead and emails the firm with TESSA Cc'd.
   - Budget/scope inquiry: MCP `request_quote` with `firm_slug`, `prospect_email`, `scope_summary` (all required) and optional `budget_band` (`under_5k` | `5k_25k` | `25k_100k` | `100k_plus` | `open` | free text). MCP-only.

## Rules
- **Fire once.** No idempotency key exists on any of these writes (`conventions/` idempotency.coverage: none). A retry after a timeout emails the firm twice. Confirm from the tool's acknowledgement instead of retrying.
- **Nothing here can be undone** (`conventions/` reversibility: none). Get the human's consent to share `prospect_email` before step 3.
- Omitting `target_firm_slug` on `request_introduction` routes the lead to TESSA itself, not to a directory firm.
- Validation failures come back as FastAPI `{"detail":[{"loc","msg","type"}]}` (422) on REST and as JSON-RPC errors on MCP.
