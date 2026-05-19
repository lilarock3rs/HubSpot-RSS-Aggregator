# Example prompts

## Bootstrap (skill asks for feeds)

```
Use hubspot-rss-hubdb-aggregator from scratch. Ask me which RSS feeds to use before creating HubDB rows.
After serverless deploy, share the POST URL so I can add a HubSpot workflow timer.
```

## Provide feeds when asked

```
Use these RSS sources:
1. HubSpot Blog — https://blog.hubspot.com/marketing/rss.xml
2. TechCrunch — https://techcrunch.com/feed/
```

## Workflow timer setup

```
I deployed sync-rss. Give me the full POST URL and steps to create a scheduled HubSpot workflow that calls it daily.
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
