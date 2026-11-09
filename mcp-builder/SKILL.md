---
name: mcp-builder
dust_skill_id: skl_9UFRqNDiUIrh
availability: workspace_users
note: Matches Anthropic's publicly published "mcp-builder" skill.
---

# mcp-builder

**Description:** Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Use when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) or Node/TypeScript (MCP SDK).

> ⚠️ Attached reference files not exported here: `reference/evaluation.md`, `reference/mcp_best_practices.md`, `reference/node_mcp_server.md`, `reference/python_mcp_server.md`, `scripts/connections.py`, `scripts/evaluation.py`, `scripts/example_evaluation.xml`, `scripts/requirements.txt`, `LICENSE.txt`.

## Instructions

# MCP Server Development Guide

## Overview
Create MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. The quality of an MCP server is measured by how well it enables LLMs to accomplish real-world tasks.

---

# Process

## 🚀 High-Level Workflow
Creating a high-quality MCP server involves four main phases:

### Phase 1: Deep Research and Planning

#### 1.1 Understand Modern MCP Design
**API Coverage vs. Workflow Tools:** Balance comprehensive API endpoint coverage with specialized workflow tools. Workflow tools can be more convenient for specific tasks, while comprehensive coverage gives agents flexibility to compose operations. Performance varies by client—some clients benefit from code execution that combines basic tools, while others work better with higher-level workflows. When uncertain, prioritize comprehensive API coverage.

**Tool Naming and Discoverability:** Clear, descriptive tool names help agents find the right tools quickly. Use consistent prefixes (e.g., `github_create_issue`, `github_list_repos`) and action-oriented naming.

**Context Management:** Agents benefit from concise tool descriptions and the ability to filter/paginate results. Design tools that return focused, relevant data. Some clients support code execution which can help agents filter and process data efficiently.

**Actionable Error Messages:** Error messages should guide agents toward solutions with specific suggestions and next steps.

#### 1.2 Study MCP Protocol Documentation
Navigate the MCP specification: start with the sitemap at `https://modelcontextprotocol.io/sitemap.xml`, then fetch specific pages with `.md` suffix for markdown format (e.g., `https://modelcontextprotocol.io/specification/draft.md`).

Key pages to review: specification overview and architecture; transport mechanisms (streamable HTTP, stdio); tool, resource, and prompt definitions.

#### 1.3 Study Framework Documentation
Recommended stack:
- **Language**: TypeScript (high-quality SDK support and good compatibility in many execution environments e.g. MCPB. Plus AI models are good at generating TypeScript code, benefiting from its broad usage, static typing and good linting tools)
- **Transport**: Streamable HTTP for remote servers, using stateless JSON (simpler to scale and maintain, as opposed to stateful sessions and streaming responses). stdio for local servers.

Load framework documentation:
- **MCP Best Practices**: `reference/mcp_best_practices.md` - Core guidelines
- **For TypeScript (recommended)**: TypeScript SDK README at `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`, plus `reference/node_mcp_server.md`
- **For Python**: Python SDK README at `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`, plus `reference/python_mcp_server.md`

#### 1.4 Plan Your Implementation
**Understand the API:** Review the service's API documentation to identify key endpoints, authentication requirements, and data models. Use web search and WebFetch as needed.

**Tool Selection:** Prioritize comprehensive API coverage. List endpoints to implement, starting with the most common operations.

---

### Phase 2: Implementation

#### 2.1 Set Up Project Structure
See language-specific guides (`reference/node_mcp_server.md`, `reference/python_mcp_server.md`) for project setup: package.json/tsconfig.json or module organization/dependencies.

#### 2.2 Implement Core Infrastructure
Create shared utilities: API client with authentication; error handling helpers; response formatting (JSON/Markdown); pagination support.

#### 2.3 Implement Tools
For each tool:

**Input Schema:** Use Zod (TypeScript) or Pydantic (Python); include constraints and clear descriptions; add examples in field descriptions.

**Output Schema:** Define `outputSchema` where possible for structured data; use `structuredContent` in tool responses (TypeScript SDK feature); helps clients understand and process tool outputs.

**Tool Description:** Concise summary of functionality; parameter descriptions; return type schema.

**Implementation:** Async/await for I/O operations; proper error handling with actionable messages; support pagination where applicable; return both text content and structured data when using modern SDKs.

**Annotations:** `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint` (true/false each).

---

### Phase 3: Review and Test

#### 3.1 Code Quality
Review for: no duplicated code (DRY principle); consistent error handling; full type coverage; clear tool descriptions.

#### 3.2 Build and Test
**TypeScript:** run `npm run build` to verify compilation; test with MCP Inspector: `npx @modelcontextprotocol/inspector`.

**Python:** verify syntax: `python -m py_compile your_server.py`; test with MCP Inspector.

See language-specific guides for detailed testing approaches and quality checklists.

---

### Phase 4: Create Evaluations
After implementing your MCP server, create comprehensive evaluations to test its effectiveness. Load `reference/evaluation.md` for complete evaluation guidelines.

#### 4.1 Understand Evaluation Purpose
Use evaluations to test whether LLMs can effectively use your MCP server to answer realistic, complex questions.

#### 4.2 Create 10 Evaluation Questions
1. **Tool Inspection**: List available tools and understand their capabilities
2. **Content Exploration**: Use READ-ONLY operations to explore available data
3. **Question Generation**: Create 10 complex, realistic questions
4. **Answer Verification**: Solve each question yourself to verify answers

#### 4.3 Evaluation Requirements
Each question must be: **Independent**, **Read-only**, **Complex** (multiple tool calls), **Realistic**, **Verifiable** (single clear answer, string comparison), **Stable** (answer won't change over time).

#### 4.4 Output Format
Create an XML file with this structure:
```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
  <!-- More qa_pairs... -->
</evaluation>
```

---

# Reference Files

## 📚 Documentation Library
Load these resources as needed during development:

**Core MCP Documentation (Load First):**
- MCP Protocol: sitemap at `https://modelcontextprotocol.io/sitemap.xml`, then fetch pages with `.md` suffix
- `reference/mcp_best_practices.md` - naming conventions, response format guidelines (JSON vs Markdown), pagination best practices, transport selection, security and error handling standards

**SDK Documentation (Load During Phase 1/2):**
- Python SDK: `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- TypeScript SDK: `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`

**Language-Specific Implementation Guides (Load During Phase 2):**
- `reference/python_mcp_server.md` - server init, Pydantic models, `@mcp.tool`, working examples, quality checklist
- `reference/node_mcp_server.md` - project structure, Zod schemas, `server.registerTool`, working examples, quality checklist

**Evaluation Guide (Load During Phase 4):**
- `reference/evaluation.md` - question creation guidelines, answer verification strategies, XML format specs, example questions, running an evaluation with provided scripts
