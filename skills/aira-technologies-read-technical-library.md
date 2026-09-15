---
name: Read the Aira Technologies technical article and whitepaper library
description: >-
  Retrieve Aira Technologies' long-form technical writing on Open RAN architecture and AI/ML in
  wireless from the `article` custom post type on the public read-only content API. No credentials
  required.
api: openapi/aira-technologies-articles-api-openapi.yml
operations: [listArticles, getArticle, listCategories, listTypes, getOembed]
---

# Read the Aira Technologies technical article and whitepaper library

Aira keeps its long-form technical writing in a site-specific `article` custom post type, served
under `/article/<slug>/` and entirely separate from the news archive. Three articles at capture:
*Unraveling the Open RAN Architecture: A Deep Dive* (2023-06-01), *Wireless Connections: Can AI
Technology Improve Wireless Performance?* (2023-07-19) and *Wireless RAN Networks and Improvements
With AI/ML* (2023-09-26).

Base URL: `https://aira-technology.com/wp-json`

## Authentication

None. Anonymous HTTPS.

## Steps

1. **Confirm the post type still exists** — `listTypes`
   `GET /wp/v2/types`
   `article` and `events` are site-specific types registered by the theme, not WordPress core. If a
   plugin or theme change removes one, its route disappears with no deprecation notice — see
   `lifecycle/aira-technologies-lifecycle.yml`. Check `types.article.rest_base` before assuming the
   path.

2. **List the library** — `listArticles`
   `GET /wp/v2/article?per_page=100&orderby=date&order=desc&_fields=id,date,modified,slug,title,link,categories`
   `X-WP-Total` was 3 at capture.

3. **Retrieve the body** — `getArticle`
   `GET /wp/v2/article/{id}?_fields=id,date,modified,title,content,excerpt,link&_embed=true`
   `content.rendered` is HTML. `_embed=true` inlines the author record and the featured image under
   `_embedded` in the same request instead of an N+1 walk of `_links`.

4. **Cross-check the taxonomy** — `listCategories`
   The `article` type shares the `category` taxonomy with posts. Note that the `whitepaper`
   category (id 5) has a count of **0** — no content is filed under it. If you are looking for
   "the Aira whitepapers", they are these articles plus the linked material on
   `https://aira-technology.com/blogs-whitepapers/`, not a `whitepaper`-tagged collection.

5. **Get embeddable metadata without scraping** — `getOembed`
   `GET /oembed/1.0/embed?url=<article url>`
   Returns oEmbed 1.0 `{version, provider_name, title, author_name, html, thumbnail_url, ...}` for
   any article, post, page or event URL on the host.

## Conventions that will bite you

- **This library is frozen.** The most recent article is dated 2023-09-26 — roughly three years
  stale at capture, while the press-release wire is current to 2026-06-26. Do not present these as
  Aira's current technical position; the company's platform has since moved from RANGPT to Naavik
  and Naavik One.
- **`per_page` caps at 100**; above it, `400 rest_invalid_param`.
- **Reads can be served up to 30 days stale** from the edge (`s-maxage=2592000`). Compare
  `modified`.
- **Some site content is genuinely gated.** `https://aira-technology.com/demos/` renders a
  WordPress login form rather than content. There is no API equivalent of that page and no public
  credential path to it.
