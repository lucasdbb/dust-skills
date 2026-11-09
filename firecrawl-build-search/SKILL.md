---
name: firecrawl-build-search
dust_skill_id: skl_bnZYnNUMhH6g
availability: workspace_users
---

# firecrawl-build-search

**Description:** Implement Firecrawl's `/search` endpoint in application code to find and optionally scrape web results for a query. Use when building a feature that starts from a search query rather than a known URL.

> ⚠️ Attached reference files not exported: `references/search-parameters.md`, `references/response-schema.md`.

## Instructions

# Firecrawl Build — Search
Use this skill when the integration needs to start from a **query**, not a known URL: "find pages about X", "search then scrape top results", research/aggregation features.

## When to Use This vs. Scrape/Interact
| Starting point | Skill |
|---|---|
| You already know the URL(s) | `firecrawl-build-scrape` |
| You only have a query/topic | `firecrawl-build-search` (this skill) |
| You need to click/fill/navigate before extracting | `firecrawl-build-interact` |

## Quick Start
1. Ensure `FIRECRAWL_API_KEY` is set (see `firecrawl-build-onboarding` if not)
2. Read the source-of-truth doc for your language: docs.firecrawl.dev/agent-source-of-truth/{node|python|rust|java|elixir|curl}
3. Call `search()` with the query and, if you also need page content, set `scrapeOptions`

### Example (Node/TypeScript)
```typescript
import FirecrawlApp from "@mendable/firecrawl-js";
const app = new FirecrawlApp({ apiKey: process.env.FIRECRAWL_API_KEY });

const results = await app.search("best project management tools 2024", {
  limit: 5,
  scrapeOptions: { formats: ["markdown"] }, // optional: also fetch page content
});
```

### Example (Python)
```python
from firecrawl import FirecrawlApp
app = FirecrawlApp(api_key=os.environ["FIRECRAWL_API_KEY"])

results = app.search(
    query="best project management tools 2024",
    limit=5,
    scrape_options={"formats": ["markdown"]},
)
```

## Key Parameters
- `query` — the search string
- `limit` — max number of results
- `scrapeOptions` — optional; when set, each result URL is also scraped and returned with content in the requested formats
- `tbs` — time-based search filter (e.g. restrict to recent results)
- `lang` / `country` — localize the search

## Implementation Checklist
1. Decide whether you need just links/snippets, or full page content (set `scrapeOptions` for the latter)
2. Call `search()` with a narrow, well-formed query — treat it like a real search engine query, not a natural-language question
3. Handle empty result sets gracefully
4. If you need full content for every result, prefer `scrapeOptions` over a second round of individual scrape calls (fewer requests, same credits model)
5. Run a smoke test with a real query and confirm results look relevant

## After Implementation
Test with 2–3 representative queries from the actual product use case, not generic examples, and confirm result relevance and content quality before shipping.
