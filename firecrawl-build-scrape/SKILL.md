---
name: firecrawl-build-scrape
dust_skill_id: skl_kQKMTruJKLd0
availability: workspace_users
---

# firecrawl-build-scrape

**Description:** Implement Firecrawl's `/scrape` (single page) and `/crawl` (multi-page) endpoints in application code when you already know the target URL(s). Use for "fetch this page as markdown/structured data" or "crawl this whole site" integrations.

> ⚠️ Attached reference files not exported: `references/scrape-formats.md`, `references/crawl-options.md`.

## Instructions

# Firecrawl Build — Scrape & Crawl
Use this skill when the integration starts from a **known URL**: a single page (`scrape`) or an entire site/section (`crawl`).

## When to Use This vs. Search/Interact
| Starting point | Skill |
|---|---|
| You already know the URL(s) | `firecrawl-build-scrape` (this skill) |
| You only have a query/topic | `firecrawl-build-search` |
| You need to click/fill/navigate before extracting | `firecrawl-build-interact` |

## Quick Start
1. Ensure `FIRECRAWL_API_KEY` is set (see `firecrawl-build-onboarding` if not)
2. Read the source-of-truth doc for your language: docs.firecrawl.dev/agent-source-of-truth/{node|python|rust|java|elixir|curl}
3. Choose `scrape` for one page, `crawl` for a whole site/section

### Scrape a single page (Node/TypeScript)
```typescript
import FirecrawlApp from "@mendable/firecrawl-js";
const app = new FirecrawlApp({ apiKey: process.env.FIRECRAWL_API_KEY });

const doc = await app.scrapeUrl("https://example.com/pricing", {
  formats: ["markdown", "links"],
});
```

### Crawl a whole site (Python)
```python
from firecrawl import FirecrawlApp
app = FirecrawlApp(api_key=os.environ["FIRECRAWL_API_KEY"])

job = app.crawl_url(
    "https://example.com",
    limit=100,
    scrape_options={"formats": ["markdown"]},
)
```
Crawl is async server-side; poll the job status or use the SDK's polling helper until it completes, then read `job.data`.

## Key Parameters
**scrape:** `formats` (`markdown`, `html`, `links`, `screenshot`, structured `extract` with a schema); `onlyMainContent` to strip nav/footer noise; `waitFor` for JS-heavy pages.
**crawl:** `limit` (max pages); `includePaths` / `excludePaths` to scope the crawl; `scrapeOptions` applied to every page found; `maxDepth`.

## Implementation Checklist
1. Pick `scrape` for one known URL, `crawl` when you need many pages under a path
2. Choose the minimal `formats` needed — `markdown` is usually enough for LLM consumption; add `extract` with a schema when you need structured fields
3. For crawl jobs, handle the async nature: submit, poll/wait, then read results — don't assume a synchronous response
4. Scope crawls with `includePaths`/`excludePaths` and a sane `limit` to avoid runaway credit usage
5. Handle pages that fail to scrape (paywalled, JS-broken, 404) without failing the whole batch

## After Implementation
Run a smoke test against a real target URL/site from the product's actual use case and confirm the returned content is usable (not boilerplate/nav noise) before shipping.
