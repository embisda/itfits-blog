---
name: itfits-blog-publisher
description: Researches and drafts itfits brand encyclopedia pages, then publishes via the blog API as unpublished drafts. Use when writing blog posts for itfits, publishing brand articles, working with /blog/, Fred Perry, Lacoste, or the blog_api_token workflow.
---

# itfits blog publisher

This file and `automation/SKILL.md` must stay identical. Cloud automations should load **`automation/SKILL.md`** (committed in the git repo). Local Cursor can use this path.

Draft **one brand per run**. Save as `published: false`. Never set `published: true`. Wait for the user to approve before publishing.

Do not open a pull request unless you must edit skill files.

## Workspace

Canonical local folder: `/Users/embisda/Documents/_Project/Web/itfits.vakhromeev.com`. Site files are in `www/`. If this checkout has no skill file, look for `automation/SKILL.md` then `.cursor/skills/itfits-blog-publisher/SKILL.md`. Do not create a new project. Do not use a Downloads copy.

## Cadence

- One brand every 3 days.
- If API `next` is `null`, ask the user for the next brands and **stop**.

## Source of truth

Do **not** pick the brand from the table below and do **not** keep a local “done” list.

- **Already written** (draft or published) = API `queue[].status` not `queued`, or any pages for that slug.
- **What to write now** = API `next` only.
- Queue order in PHP: `blogBrandQueue()` in `www/includes/blog.php`. Keep the table below in sync when you change PHP.

Base order (documentation only):

| Brand | slug |
|---|---|
| Fred Perry | `fredperry` |
| Stone Island | `stoneisland` |
| Ben Sherman | `bensherman` |
| Merc | `merc` |
| Pretty Green | `prettygreen` |
| Lacoste | `lacoste` |
| Ralph Lauren | `ralphlauren` |
| Calvin Klein | `calvinklein` |
| Levi's | `levis` |
| Nike | `nike` |

## Run protocol (mandatory)

Token: environment `BLOG_API_TOKEN`, or `blog_api_token` in `www/config/config.php`. Never print it. Call `https://itfits.vakhromeev.com/api/blog.php?token=` (the host often strips `Authorization`). Do not use MySQL.

Treat timeout, 5xx, 401, empty body, or non-JSON as **failure**. Failure ≠ “brand has no pages”.

1. `GET https://itfits.vakhromeev.com/api/blog.php?next=1&token=`
2. On failure: wait 45s, retry up to 3 times, then **stop**. Do not guess the next brand from the table or from memory.
3. If `next` is `null`: tell the user the queue is empty and **stop**.
4. `GET …/api/blog.php?brand={next.slug}&token=`
5. If that call fails: **stop**. If `pages` is non-empty: **stop** (do not POST; the server would return 409).
6. Research and draft that one brand (plus `next.extra_page` if present).
7. Immediately before POST, repeat step 4. If it fails or `pages` is non-empty: **stop**.
8. `POST` with `published: false`. Never send `overwrite: true`.
9. Summarize brand + slugs saved. If POST returns 409, report it and **stop**.

Wait at least 45s between API calls if the host rate-limits. Do not walk the queue with per-brand GETs after a timeout.

## Subbrands in the catalog

Wardrobe names are not always the house name. Write the **parent**, plus one extra page about the line. API `next.extra_page` is `{slug, title}` when the catalog has a line.

| If admin catalog has | Write hub as | Extra page |
|---|---|---|
| Calvin Klein Jeans | Calvin Klein (`calvinklein`) | slug `jeans` — линия Calvin Klein Jeans |
| Marciano By Guess | Guess (`guess`) | slug `marciano` — линия Marciano By Guess |

Same pattern later: Polo Ralph Lauren → Ralph Lauren + page `polo`, etc. Infer parent from the name; do not invent a second house.

Extra page: history of that line, how it relates to the house, typical products. Title like «Calvin Klein Jeans». Hub links «подробнее» to it.

## Pages (skip if sources do not support the section)

| API `slug` | Public URL | Role |
|---|---|---|
| `index` | `/blog/{brand}` | Hub. H1 = brand name. Short “what they are famous for / what they make now” (2–4 sentences). No logo in HTML — the template shows the catalog logo. |
| `founder` | `/blog/{brand}/founder` | Who founded, how they started, other businesses of the founder. |
| `history` | `/blog/{brand}/history` | First product, how it got popular, most famous product. |
| `culture` | `/blog/{brand}/culture` | Music, musicians, subcultures, exhibitions, charity — only sourced facts. |
| `logo` | `/blog/{brand}/logo` | Meaning and evolution. **Do not put a logo `<img>` in content** — PHP inserts `brands.logo_url` from the admin catalog. Set `og_image` to that URL if present. |
| `owner` | `/blog/{brand}/owner` | Current owner, stock. |
| `stores` | `/blog/{brand}/stores` | Moscow offline, SPb offline, Russia online. |
| *(subbrand)* | `/blog/{brand}/{slug}` | Only when the catalog has a distinct line (Jeans, Marciano, …). |

## Voice, sources, forbidden

- Russian, **Вы**, encyclopedic but not stiff. Never invent; omit empty sections.
- Sources: Wikipedia, official sites, marketplaces, books.
- No `<script>`, JSON-LD, OG, or breadcrumbs in `content`.
- Do not mention the Russia–Ukraine war, invasion, «СВО», sanctions, or political statements about that conflict. Closed shops: status only.

## Images

- Add a `<figure>` **only** when the photo is unique to that page and you have a free stable URL. If nothing fits, **no image**.
- Search order for photos (keep going until a **reusable** license is found: CC BY/SA, public domain, official press still with clear reuse, or Wikimedia):
  1. Wikimedia Commons API (`list=search` namespace 6) and Wikipedia infobox / `File:` on EN/IT/RU pages.
  2. Openverse / Flickr Commons (`license=cc` or `pd`), Europeana, Internet Archive images.
  3. Official brand heritage / about pages, Massimo Osti Archive, company press kits — **only** if the page states reuse; otherwise skip (portraits of living/recent designers are usually © archive).
  4. National portraits / museum collections (NPG, Rijksmuseum, MET Open Access).
- Do not hotlink random magazine scans, Instagram, or WWD “courtesy of” shots.
- Do not reuse the catalog logo or the same photo on several pages.
- Hub and logo pages: catalog logo is already in the template — never duplicate it in HTML.
- Captions left-aligned (template CSS). One figure max per page unless culture truly needs two distinct sourced photos.
- `og_image`: catalog logo, or the page’s unique still (not a GIF).

## SEO

- `seo_title` ~70 chars, `seo_description` 140–160, one H1 from `title`.
- `faq`: 3–6 items this page only, same wording as body; skip if you cannot.

## Publish

1. Token from env or `www/config/config.php` — never copy it into chat or this skill.
2. `POST https://itfits.vakhromeev.com/api/blog.php?token=`
3. `published: false`. Hub slug `index`.
4. After approval, set `published: true` with PUT/PATCH (not a new POST of the whole brand).

See [reference.md](reference.md).
