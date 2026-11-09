---
name: dust-mcp-server
dust_skill_id: skl_OtJNLjxKbt5h
availability: workspace_users
note: Condensed — original is ~25KB of TypeScript code examples. Key structure and steps preserved; full code samples available via Dust (describe_skill on skl_OtJNLjxKbt5h) or the referenced source files (e.g. front/lib/api/actions/servers/github/).
---

# dust-mcp-server

**Description:** Step-by-step guide for creating new internal MCP server integrations in Dust that connect to remote platforms (Jira, HubSpot, Salesforce, etc.). Use when adding a new MCP server, implementing a platform integration, or connecting Dust to a new external service.

## Instructions (condensed)

### File Structure
```
front/lib/api/actions/servers/{provider}/
├── metadata.ts           # Tool metadata and server info using createToolsRecord
├── tools/index.ts        # Tool handlers with exhaustive Record type
├── index.ts              # Server creation and tool registration
├── client.ts             # API client (optional)
└── helpers.ts            # Helper functions (optional)
```

### Registration Files
1. `front/lib/actions/mcp_internal_actions/constants.ts` - Add server config with `metadata: YOUR_SERVER`
2. `front/lib/actions/mcp_internal_actions/servers/index.ts` - Import and register in switch statement

### OAuth Requirements (if the platform requires OAuth)
- OAuth provider must already exist in `front/lib/api/oauth/providers/{provider}.ts`
- OAuth core implementation must exist in `core/src/oauth/providers/{provider}.rs`
- OAuth scopes must be configured for the required API access
- Server's `authorization` field must reference the OAuth provider

### Common Gotchas
- Do not forget to add the server to `AVAILABLE_INTERNAL_MCP_SERVER_NAMES` array
- Server IDs must be stable and unique; never change them once deployed
- Tool stakes must be configured appropriately (`never_ask`, `low`, `medium`, `high`)
- Always implement proper error handling with `Result` types
- Handle OAuth token refresh automatically through the `withAuth` pattern

### Prerequisites — OAuth Configuration (if required)
1. Check whether an OAuth provider exists in `core/src/oauth/providers/` as `{provider}.rs`
2. Check whether a front OAuth provider exists in `front/lib/api/oauth/providers/{provider}.ts`

If not, implement it first in `core` and `front`: create `core/src/oauth/providers/{provider}.rs`, implement the OAuth flow (authorization URL, token exchange, refresh), register the provider in `core/src/oauth/providers/mod.rs`, create `front/lib/api/oauth/providers/{provider}.ts`. See `hubspot.rs` or `jira.rs` for reference.

### Research Phase (before implementation)
1. **API Documentation** — official docs, REST vs GraphQL vs SDK, rate limits, pagination
2. **Authentication Method** — OAuth 2.0 (preferred) vs API key/bearer token; required scopes
3. **Available Operations** — read (list, get, search), write (create, update, delete), special operations (transitions, associations, etc.)

### Step-by-Step Implementation

**1. Create `metadata.ts`** at `front/lib/api/actions/servers/{provider}/metadata.ts`. Define tools via `createToolsRecord({...})` — each tool has `description`, a Zod `schema`, a `stake` (`never_ask`/`low`/`medium`/`high`), and `displayLabels: { running, done }`. Export a `YOUR_PROVIDER_SERVER` object satisfying `ServerMetadata`, with `serverInfo` (name, version, description, `authorization.provider`, `supported_use_cases`, icon, `documentationUrl`), `tools` (mapped from the tools record via `zodToJsonSchema`), and `tools_stakes`.

**2. Create `tools/index.ts`** at `front/lib/api/actions/servers/{provider}/tools/index.ts`. Implement a `ToolHandlers<typeof YOUR_PROVIDER_TOOLS_METADATA>` object: one async handler per tool key, receiving `(params, { authInfo })`. Get the OAuth token via `authInfo?.token`, return `new Err(new MCPError(...))` on failure or `new Ok([{type:"text", text:...}, ...])` on success. Export `TOOLS = buildTools(YOUR_PROVIDER_TOOLS_METADATA, handlers)`.

**3. Create `index.ts`** at `front/lib/api/actions/servers/{provider}/index.ts`. A `createServer(auth, agentLoopContext)` function that calls `makeInternalMCPServer("your_provider")`, loops over `TOOLS` calling `registerTool(auth, agentLoopContext, server, tool, { monitoringName: "your_provider" })`, and returns the server. Default-export it.

**4. Register in `constants.ts`** (`front/lib/actions/mcp_internal_actions/constants.ts`): import `YOUR_PROVIDER_SERVER`, add the server name to `AVAILABLE_INTERNAL_MCP_SERVER_NAMES`, and add a config entry to `INTERNAL_MCP_SERVERS`:
```typescript
your_provider: {
  id: 99, // unique stable ID, never change after deployment
  availability: "manual", // "manual" | "auto" | "auto_hidden_builder"
  allowMultipleInstances: true, // true for OAuth-based integrations
  isRestricted: undefined, // feature-flag/plan gating function if needed
  isPreview: false, // true for beta/preview integrations
  tools_arguments_requiring_approval: undefined,
  tools_retry_policies: undefined,
  timeoutMs: undefined,
  metadata: YOUR_PROVIDER_SERVER,
},
```

**5. Register in `servers/index.ts`** (`front/lib/actions/mcp_internal_actions/servers/index.ts`):
```typescript
case "your_provider":
  return yourProviderServer(auth, agentLoopContext);
```

### Optional: `client.ts` and `helpers.ts`
Add `client.ts` when you need multiple API endpoints, response validation, auth header management, or retry logic. Add `helpers.ts` for `withAuth` wrappers, response rendering, shared transformations. Decision guide:
- no external API → keep everything in `tools/index.ts`
- 1–2 simple API calls → inline, maybe add `helpers.ts`
- several API endpoints → create `client.ts`
- complex response formatting → dedicated rendering helpers

`withAuth` pattern: a helper that extracts `authInfo?.token`, returns `Err(MCPError)` if missing, otherwise runs the action in a try/catch and wraps failures in `MCPError`.

### Alternative: function-based tools
If handlers need direct access to `Authenticator`, create tools through a function instead of a constant — see `front/lib/api/actions/servers/github/tools/index.ts` for a full example.

### Icon
Use an existing similar icon temporarily, then request the final icon from design/Sparkle.

> ⚠️ This file is a condensed summary. The full skill instructions include complete TypeScript code samples for each step (metadata.ts, tools/index.ts, index.ts, withAuth helper) — re-fetch via Dust's `building_agents_and_skills__describe_skill` (id `skl_OtJNLjxKbt5h`) for verbatim code.
