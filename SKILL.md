---
name: itfits-blog-publisher
description: Researches and drafts itfits brand encyclopedia pages, then publishes via the blog API as unpublished drafts. Use when writing blog posts for itfits, publishing brand articles, working with /blog/, Fred Perry, Lacoste, or the blog_api_token workflow.
---

# itfits blog publisher

Draft one brand at a time. Save as `published: false`. Wait for the user to approve before flipping to published.

## Cadence

- One brand every 3 days.
- After the queue is done, ask the user for the next brands.

## Queue

Base order:

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

Before writing the next brand:

1. `GET https://itfits.vakhromeev.com/api/blog.php?token=` — with token the JSON includes `brands` (blog, including drafts) and `catalog` (`name`, `logo_url` from table `brands`). Or `GET …/api/blog.php?catalog=1&token=`.
2. Skip encyclopedia brands that already have pages (draft or published).
3. Map catalog names to **parent encyclopedia brands** (below). Append any parent not yet in the queue and not yet written.
4. Pick the first unwritten brand in that merged list.

## Subbrands in the catalog

Wardrobe names are not always the house name. Write the **parent**, plus one extra page about the line.

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

1. Token from `www/config/config.php` — never copy it into chat or this skill.
2. `POST https://itfits.vakhromeev.com/api/blog.php` (`Authorization: Bearer` or `?token=` if the host strips the header).
3. `published: false`. Hub slug `index`.
4. After approval, set `published: true`.

See [reference.md](reference.md).
