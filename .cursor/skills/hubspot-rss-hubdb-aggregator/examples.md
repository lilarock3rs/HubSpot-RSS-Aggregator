# Example prompts

## Bootstrap

```
Use the hubspot-rss-hubdb-aggregator skill. Set up the RSS aggregator from scratch
in this workspace: HubDB tables, sync-rss serverless, and rss-feed-list module (HubL).
Query docs with MCP before implementing.
```

## Add a feed

```
Add the feed https://example.com/rss.xml to rss_feeds (name: Example Blog)
and tell me how to run sync-rss.
```

## Sync and verify

```
Sync active RSS feeds. If it fails, check serverless logs with MCP.
```

## Module only

```
Create the rss-feed-list module to show the latest 10 articles from rss_articles.
```

## Deploy (explicit)

```
Upload the CMS project to HubSpot. Ask for confirmation before running upload.
```
