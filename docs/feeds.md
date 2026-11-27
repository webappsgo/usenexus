# Public feeds

GoSpool serves Atom and RSS feeds for public newsgroups. The feeds contain
article subjects and group names; they do not include article bodies or
private-group content.

| URL | Format | Content type |
|---|---|---|
| `/feed.xml` | Atom 1.0 | `application/atom+xml; charset=utf-8` |
| `/rss.xml` | RSS 2.0 | `application/rss+xml; charset=utf-8` |
| `/groups/{group}/feed.xml` | Atom 1.0 for one public group | `application/atom+xml; charset=utf-8` |
| `/groups/{group}/rss.xml` | RSS 2.0 for one public group | `application/rss+xml; charset=utf-8` |

The global feeds combine active articles from public groups and suppress
duplicate Message-IDs from cross-posts. A private group feed returns `404`,
including for signed-in users. Article links are absolute URLs based on the
validated request host. Feed text is XML-safe and bounded to 140 characters
for titles and 280 characters for summaries.

Configure the global entry limit and channel title in `server.yml`:

```yaml
server:
  seo:
    feed:
      max_items: 50
      title: ""
```

`max_items` must be between 1 and 200. An empty title uses `GoSpool`. Feed
responses use a five-minute public cache lifetime, ETags, and
`Last-Modified` when at least one dated article is available. Both feed routes
support conditional requests and `HEAD`.
