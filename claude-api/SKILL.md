---
name: claude-api
dust_skill_id: skl_SmIOse1lqDGv
availability: workspace_users
note: Condensed — original is ~58KB covering per-language SDK docs (Python, TypeScript, Java, Go, Ruby, C#, PHP, cURL), Managed Agents, caching, model migration, etc. Full content available via Dust (describe_skill on skl_SmIOse1lqDGv).
---

# claude-api

**User-facing description:** Reference for the Claude API / Anthropic SDK — model ids, pricing, params, streaming, tool use, MCP, agents, caching, token counting, model migration.

TRIGGER — read BEFORE opening the target file; don't skip because it "looks like a one-liner" — whenever: the prompt names Claude/Anthropic in any form (Claude, Anthropic, Fable, Opus, Sonnet, Haiku, `anthropic`, `@anthropic-ai`, `claude-*`, `us.anthropic.*`, `[1m]`); the user asks about an LLM (pricing/model choice/limits/caching) — never answer from memory; OR the task is LLM-shaped with provider unstated (agent/MCP/tool-definition/multi-agent/RAG/LLM-judge/computer-use; generate/summarize/extract/classify/rewrite/converse over NL; debugging refusals/cutoffs/streaming/tool-calls/tokens).

SKIP only when another provider is being worked on (overrides all triggers): OpenAI/GPT/Gemini/Llama/Mistral/Cohere/Ollama named in the query; OR `grep -rE 'openai|langchain_openai|google.generativeai|genai|mistralai|cohere|ollama'` over the project hits (run this grep FIRST if no provider named — don't Read the file).

## Instructions (condensed)

# Building LLM-Powered Applications with Claude
This skill helps you build LLM-powered applications with Claude. Choose the right surface based on your needs, detect the project language, then read the relevant language-specific documentation.

## Before You Start
Scan the target file (or prompt/project) for non-Anthropic provider markers (`import openai`, `langchain_openai`, `OpenAI(`, `gpt-4`, `gpt-5`, files like `agent-openai.py`, or explicit provider-neutral instructions). If found, stop and ask whether to switch to Claude or keep it non-Claude. Do not edit a non-Anthropic file with Anthropic SDK calls.

## Output Requirement
Code must call Claude through:
1. **The official Anthropic SDK** for the project's language (`anthropic`, `@anthropic-ai/sdk`, `com.anthropic.*`, etc.) — default whenever a supported SDK exists.
2. **Raw HTTP** (curl/requests/fetch/httpx) — only when explicitly requested, the project is shell/cURL, or no SDK exists for the language.

Never mix the two, never fall back to OpenAI-compatible shims. **Never guess SDK usage** — function/class names, signatures and import paths must come from explicit documentation (the skill's per-language files or official SDK repos/docs listed in `shared/live-sources.md`); WebFetch if not documented. Do not infer Ruby/Java/Go/PHP/C# APIs from cURL shapes or other languages' SDKs.

## Defaults (unless user requests otherwise)
- Model: Claude Opus 4.8, exact string `claude-opus-4-8`
- Adaptive thinking (`thinking: {type: "adaptive"}`) for anything remotely complicated
- Streaming for any request with long input/output or high `max_tokens` (prevents timeouts); use `.get_final_message()` / `.finalMessage()` if you don't need individual stream events

## Subcommands
If the user request is a bare subcommand string, match it against the Subcommands table:
| Subcommand | Action |
|---|---|
| `migrate` | Migrate existing Claude API code to a newer model. Read `shared/model-migration.md` immediately: Step 0 (confirm scope — ask which files/directories first), Step 1 (classify each file), then per-target breaking changes. Execute, don't summarize. Ask which target model if unstated. |

## Language Detection
Infer from project files:
- `*.py`, `requirements.txt`, `pyproject.toml` → Python → `python/`
- `*.ts`/`*.tsx`, `package.json`, `tsconfig.json` → TypeScript → `typescript/`
- `*.js`/`*.jsx` (no `.ts`) → TypeScript (same SDK) → `typescript/`
- `*.java`, `pom.xml`, `build.gradle` → Java → `java/`
- `*.kt`/`*.kts`, `build.gradle.kts` → Java SDK (Kotlin) → `java/`
- `*.scala`, `build.sbt` → Java SDK (Scala) → `java/`
- `*.go`, `go.mod` → Go → `go/`
- `*.rb`, `Gemfile` → Ruby → `ruby/`
- `*.cs`, `*.csproj` → C# → `csharp/`
- `*.php`, `composer.json` → PHP → `php/`

If multiple languages detected, infer from the current file/question or ask. If language can't be inferred, use AskUserQuestion (Python/TypeScript/Java/Go/Ruby/cURL/C#/PHP) or default to Python with a note. For unsupported languages (Rust, Swift, C++, Elixir...), suggest cURL examples from `curl/` and offer Python/TypeScript as reference. For cURL/raw HTTP, read from `curl/`.

### Language-Specific Feature Support
| Language | Tool Runner | Managed Agents | Notes |
|---|---|---|---|
| Python | Yes (beta) | Yes (beta) | `@beta_tool` decorator |
| TypeScript | Yes (beta) | Yes (beta) | `betaZodTool` + Zod |
| Java | Yes (beta) | Yes (beta) | Beta tool use with annotated classes |
| Go | Yes (beta) | Yes (beta) | `BetaToolRunner` in `toolrunner` pkg |
| Ruby | Yes (beta) | Yes (beta) | `BaseTool` + `tool_runner` in beta |
| C# | Yes (beta) | Yes (beta) | `BetaToolRunner` + raw JSON schema |
| PHP | Yes (beta) | Yes (beta) | `BetaRunnableTool` + `toolRunner()` |
| cURL | N/A | Yes (beta) | Raw HTTP, no SDK features |

Managed Agents are persistent: create once (`agents.create`), store the ID, reference it from every `sessions.create` — don't recreate in the request path. The `ant` CLI can create agents/environments from version-controlled YAML (`shared/anthropic-cli.md`). C# has beta Managed Agents support via `client.Beta.Agents`.

## Which Surface Should I Use?
Start simple — default to the simplest tier; only reach for agents when the task genuinely needs open-ended, model-driven exploration.

| Use Case | Tier | Surface | Why |
|---|---|---|---|
| Classification, summarization, extraction, Q&A | Single LLM call | Claude API | One request, one response |
| Batch processing or embeddings | Single LLM call | Claude API | Specialized endpoints |
| Multi-step pipelines, code-controlled logic | Workflow | Claude API + tool use | You orchestrate the loop |
| Custom agent with your own tools | Agent | Claude API + tool use | Maximum flexibility |
| Server-managed stateful agent with workspace | Agent | Managed Agents | Anthropic runs the loop + hosts the sandbox |
| Persisted, versioned agent configs | Agent | Managed Agents | Agents are stored objects; sessions pin to a version |
| Long-running multi-turn agent with file mounts | Agent | Managed Agents | Per-session containers, SSE events, Skills + MCP |

Managed Agents = Anthropic runs the loop AND hosts the tool-execution container. Claude API + tool use = you host the compute (tool runner for automatic looping, or manual loop for approval gates/custom logging/conditional execution).

**Cloud-provider access:** Claude Platform on AWS has same-day API parity (Managed Agents work, except self-hosted sandboxes). Amazon Bedrock, Google Vertex AI, Microsoft Foundry do **not** support Managed Agents or server-side tools — use Claude API + tool use there.

### Decision Tree (summary)
0. Provider: first-party/AWS Platform → full surface; Bedrock/Vertex/Foundry → Claude API only (no Managed Agents).
1. Single call → Claude API.
2. Want Anthropic to run the loop + host a per-session container (bash/file ops/code)? → Managed Agents.
3. Workflow, code-orchestrated, your own tools → Claude API + tool use.
4. Open-ended agent, your own tools, you host compute → Claude API + tool use (manual or tool-runner loop).

> ⚠️ This file is a condensed summary of the trigger/routing logic only. The full skill includes complete per-language SDK references (`python/`, `typescript/`, `java/`, `go/`, `ruby/`, `csharp/`, `php/`, `curl/`), caching guidance, token counting, pricing tables, and the full model-migration guide (`shared/model-migration.md`) — re-fetch via Dust's `building_agents_and_skills__describe_skill` (id `skl_SmIOse1lqDGv`) for the complete ~58KB reference.
