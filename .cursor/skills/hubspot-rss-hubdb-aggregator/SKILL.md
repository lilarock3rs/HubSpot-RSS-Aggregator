---
name: hubspot-rss-hubdb-aggregator
description: >-
  Builds and maintains a HubSpot RSS-to-HubDB-to-CMS workflow using HubSpotDev MCP
  and local code. Creates HubDB tables (rss_feeds, rss_articles), CMS serverless
  sync-rss, and HubL module rss-feed-list. Use when the user mentions RSS, feeds,
  HubDB, sync articles, hubspot-rss, or CMS news aggregator.
---

# HubSpot RSS → HubDB → CMS

## Prerequisites

- HubSpot CLI authenticated: `hs auth`
- MCP server **HubSpotDev** enabled in Cursor
- Portal with **CMS + HubDB** (Professional/Enterprise)
- Project root: use `absoluteCurrentWorkingDirectory` = workspace absolute path

## Workflow (follow in order)

### 1. Documentation (MCP — always first)

Call `search-docs` then `fetch-doc` for:

- HubDB API (create table, rows, publish)
- CMS serverless functions (overview + getting started)

Do not guess API shapes; use fetched docs.

### 2. RSS sources (ask the user — required)

**Before creating or seeding `rss_feeds`, ask the user which RSS sources to use.**

Use `AskQuestion` or a short conversational prompt. Collect for each feed:

| Field | Required | Example |
|-------|----------|---------|
| `name` | yes | HubSpot Blog |
| `feed_url` | yes | https://blog.hubspot.com/marketing/rss.xml |
| `enabled` | default true | true |

Rules:

- Do **not** use demo feeds from `hubdb/rss_feeds.json` unless the user accepts defaults or skips.
- If the user gives one URL without a name, derive a readable name from the domain.
- Confirm the list back to the user before writing HubDB rows or updating `hubdb/rss_feeds.json`.
- After confirmation, either:
  - Update `hubdb/rss_feeds.json` `rows` and run `hs hubdb create` (new portal), or
  - Add rows in HubDB UI / HubDB API for an existing table, then **Publish** `rss_feeds`.

Example confirmation message:

```
Planned RSS sources:
1. HubSpot Blog — https://blog.hubspot.com/marketing/rss.xml
2. NASA Breaking News — https://www.nasa.gov/rss/dyn/breaking_news.rss

Proceed with these feeds?
```

### 3. HubDB tables (CLI — not an MCP tool)

**MCP does not create HubDB tables.** Use HubSpot CLI after `fetch-doc` on HubDB CLI commands:

```bash
./scripts/create-hubdb-tables.sh
# or:
hs hubdb create --path hubdb/rss_feeds.json
hs hubdb create --path hubdb/rss_articles.json
```

Source files: `hubdb/rss_feeds.json`, `hubdb/rss_articles.json`. Populate `rss_feeds.json` rows from **step 2** (user-confirmed sources), not hardcoded demos.

| Table | Purpose |
|-------|---------|
| `rss_feeds` | Sources: name, feed_url, enabled, last_synced_at |
| `rss_articles` | Items: title, link, summary, published_at, feed_name, guid, image_url |

Column definitions: see [reference.md](reference.md). Legacy `*.schema.json` files are reference-only.

**Publish** both tables in HubSpot UI (Content > HubDB > Publish) so the live site and module see data.

### 4. CMS serverless (MCP scaffold + code)

Call MCP `create-cms-function` with:

- `functionsFolder`: `sync-rss`
- `filename`: `sync-rss.js`
- `endpointMethod`: `POST`
- `endpointPath`: `/sync-rss`
- `dest`: `{workspace}/cms/sync-rss.functions`

Implement logic in generated file (see `cms/sync-rss.functions/` in repo):

- Read enabled feeds from HubDB `rss_feeds`
- Fetch and parse each RSS URL
- Upsert into `rss_articles` (dedupe by `guid` or `link`)
- Max 20 items per feed per run
- Publish `rss_articles` when done
- Use `HUBSPOT_PRIVATE_APP_TOKEN` from HubSpot secrets

**Do not** call `upload-project` unless the user explicitly requests deploy.

After upload/deploy, **always share the serverless sync URL** with the user in this format:

```markdown
## RSS sync endpoint (for manual curl or HubSpot workflow timer)

**Method:** POST
**URL:** https://{cms-domain}/_hcms/api/sync-rss?portalid={portalId}
**Headers:** Content-Type: application/json
**Body:** {} (empty JSON object is fine)

Example curl:
curl -X POST "https://{cms-domain}/_hcms/api/sync-rss?portalid={portalId}" \
  -H "Content-Type: application/json"
```

Ask for **CMS domain** and **portal ID** if unknown (`hs account list`, or Settings → Account Defaults).  
If the user wants scheduled sync, point them to [reference.md — HubSpot workflow timer](reference.md#hubspot-workflow-timer) and paste the URL above — do **not** create the workflow unless they ask.

### 5. CMS module (MCP scaffold + HubL)

Call MCP `create-cms-module` with:

- `userSuppliedName`: `rss-feed-list` (ask if unclear)
- `reactType`: `false` (HubL)
- `moduleLabel`: `RSS Feed List`
- `contentTypes`: `SITE_PAGE,LANDING_PAGE`
- `dest`: `{workspace}/cms/rss-feed-list.module`

Edit `module.html` to use `hubdb_table_rows()` — see repo template.

### 6. Validate (MVP)

1. Feeds exist in `rss_feeds` (from step 2) with `enabled = true`
2. `POST` to the shared sync URL (step 4)
3. Confirm rows in `rss_articles` (published)
4. Add module to a landing page; preview

Debug failures: MCP `get-cms-serverless-function-logs`.

## MCP rules

| Rule | Action |
|------|--------|
| Names / feeds | Never invent `userSuppliedName` or RSS URLs; **ask user for feeds** |
| Serverless URL | After deploy, **always output** POST URL + curl for workflow timer |
| Upload/deploy | Ask before `upload-project` or `deploy-project` |
| Docs | `search-docs` → `fetch-doc` before HubDB/serverless API work |
| Paths | Pass full absolute paths to MCP tools |

## Out of scope (unless user asks)

- Creating the HubSpot workflow in the UI (skill provides URL + steps only)
- React modules
- CRM sync

Scheduled sync is supported **via HubSpot workflow timer** calling the shared serverless URL (see reference.md).

## Repo layout

```
.cursor/skills/hubspot-rss-hubdb-aggregator/
hubdb/*.schema.json
cms/sync-rss.functions/
cms/rss-feed-list.module/
scripts/seed-feeds.example.json
README.md
```

## Examples

See [examples.md](examples.md).
