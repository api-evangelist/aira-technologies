---
name: Track Aira Technologies RAN announcements and partnerships
description: >-
  Pull Aira Technologies' press releases and blog posts about AI-native RAN automation, operator
  and vendor partnerships, funding and alliance membership from the public read-only content API.
  No credentials required.
api: openapi/aira-technologies-posts-api-openapi.yml
operations: [searchContent, listPosts, getPost, listCategories, getUser]
---

# Track Aira Technologies RAN announcements and partnerships

50 published posts at capture — 35 in the `Press Release` category, 15 in `Blog` — covering Naavik
and Naavik One, RANGPT, xApps, and named work with Nokia, Broadcom, Snowflake, Ericsson, Tech
Mahindra, VMware, Dell and Intel, plus the $14.5M Series B and AI-RAN Alliance membership.

Base URL: `https://aira-technology.com/wp-json`

## Authentication

None. Anonymous HTTPS. The collection answers `Allow: GET`.

## Steps

1. **Find the announcement** — `searchContent`
   `GET /wp/v2/search?search=<terms>&per_page=20`
   Returns lightweight `{id, title, url, type, subtype}` records across every public post type.
   Branch on `subtype`: `post` for news, `article` for a technical piece, `events` for a
   conference, `page` for marketing. Each result's `_links.self` gives the correctly typed URL.
   Verified live: `search=naavik` returned 12 objects.

2. **Or walk the archive by date** — `listPosts`
   `GET /wp/v2/posts?per_page=100&orderby=date&order=desc&_fields=id,date,modified,slug,title,link,categories,author`
   `X-WP-Total` was 50 at capture. Narrow with `after=<ISO8601>` / `before=<ISO8601>`, or
   `modified_after` to pick up silent edits to an existing release.

3. **Split announcements from opinion** — `listCategories`, then filter
   `GET /wp/v2/categories` returns 5 terms. At capture: `press-release` id 3 (35 posts),
   `blog` id 4 (15), `wireless` id 6 (2), `uncategorized` id 1 (1), `whitepaper` id 5 (0).
   `GET /wp/v2/posts?categories=3` is the press-release wire; `categories=4` is the blog.
   The `whitepaper` category is empty — whitepapers live in the `article` post type instead
   (see `aira-technologies-read-technical-library.md`).

4. **Retrieve the body** — `getPost`
   `GET /wp/v2/posts/{id}?_fields=id,date,modified,title,content,excerpt,link,categories`
   `content.rendered` is HTML, not markdown. `excerpt.rendered` is a usable summary and is far
   cheaper than the full body when you are classifying a hundred releases.

5. **Attribute the byline carefully** — `getUser`
   `GET /wp/v2/users/{id}` resolves `author`. Only 4 author records exist and two of them are
   agency accounts whose display name is an email address — they are the PR agency, not an Aira
   spokesperson. Treat `author` as a publishing account, not as a quotable person; the quotable
   names are inside `content.rendered`.

## Conventions that will bite you

- **`per_page` caps at 100.** 101 or more returns `400 rest_invalid_param` with the bound in
  `data.params.per_page`.
- **Paginate off the headers, not off a guess.** `X-WP-Total` and `X-WP-TotalPages` are exposed via
  `Access-Control-Expose-Headers`, and the `Link` header carries `rel="next"`.
- **Reads can be 30 days stale.** Responses carry `Cache-Control: public, max-age=0,
  s-maxage=2592000`. Compare `modified` against your last sync rather than trusting freshness.
- **Match on `code`, never `message`.** Errors are `{code, message, data:{status}}`. See
  `errors/aira-technologies-problem-types.yml`.
- **There is no rate-limit signal.** No `RateLimit-*`, no `X-RateLimit-*`, no `Retry-After` on any
  observed response. Self-throttle; Cloudflare fronts the origin and will act silently.
- **Everything is read-only.** An unauthenticated `POST` returns `401 rest_cannot_create`, not 405.
  Do not retry it.

## What this is not

This is the company's marketing content, not a Naavik or RANGPT API. Aira publishes no product API,
no developer portal and no specification for anything it sells. Do not present this surface to a
user as "the Aira API".
