---
name: itfits-blog-publisher
description: Researches and drafts itfits brand encyclopedia pages, then publishes via the blog API as unpublished drafts. Use when writing blog posts for itfits, publishing brand articles, working with /blog/, or the blog_api_token workflow.
---

# itfits blog publisher

This file and `automation/SKILL.md` must stay identical. Cloud automations should load **`automation/SKILL.md`** (committed in the git repo). Local Cursor can use this path.

Draft **one new brand per run** after backfill is empty. When the API `backfill` list is not empty (new section added to the skill, e.g. ambassadors), **this run must write every missing page for every listed brand**, then stop. Do not write only one brand’s ambassadors and wait for the next cron. Do not skip backfill to start a new brand. Once `backfill` is `[]`, never re-check old brands unless a new slug is added to `blogBackfillPageSlugs()`.

Save as `published: false`. Never set `published: true`. Wait for the user to approve before publishing.

Do not open a pull request unless you must edit skill files.

## Workspace

Canonical local folder: `/Users/embisda/Documents/_Project/Web/itfits.vakhromeev.com`. Site files are in `www/`. If this checkout has no skill file, look for `automation/SKILL.md` then `.cursor/skills/itfits-blog-publisher/SKILL.md`. Do not create a new project. Do not use a Downloads copy.

## Cadence

- After a skill/automation change that adds a page type: one run fills **all** existing brands’ missing pages (`backfill`).
- After that, one **new** catalog brand per run (full encyclopedia, including the new page types).
- If API `next` is `null` and `backfill` is `[]`, the catalog has nothing left. Stop.

## Source of truth

Do **not** keep a hardcoded brand list in this skill or in the automation prompt.

- What exists in the wardrobe admin catalog = `catalog` from the API (`GET` with token), same brands as https://itfits.vakhromeev.com/admin/brands.php
- What to write now = API `backfill` first (all items), else API `next` (one new brand).
- Map catalog line names to a parent house (Calvin Klein Jeans → Calvin Klein + extra page). Do not invent a second house.

## New section on existing brands

Whenever a new page type is added to the encyclopedia (for example `ambassadors`):

1. Add it to `blogSectionLabels()` / this skill.
2. Add the slug to `blogBackfillPageSlugs()` in `www/includes/blog.php` so API `backfill` lists every existing brand that lacks the page.
3. The **next automation run** writes all `backfill` pages in that single run (`PUT`, `published: false`). Then later runs write new brands.
4. Do not POST a full brand for backfill (that 409s).

## Run protocol (mandatory)

Token: environment `BLOG_API_TOKEN`, or `blog_api_token` in `www/config/config.php`. Never print it. Call `https://itfits.vakhromeev.com/api/blog.php?token=` (the host often strips `Authorization`). Do not use MySQL.

Treat timeout, 5xx, 401, empty body, or non-JSON as **failure**. Failure ≠ “brand has no pages”.

1. `GET https://itfits.vakhromeev.com/api/blog.php?next=1&token=`
2. On failure: wait 45s, retry up to 3 times, then **stop**. Do not guess the next brand.
3. If `backfill` is a non-empty array: for **each** item, `GET ?brand={slug}&token=`, then `PUT` every slug in `missing_pages` with `published: false` and a proper `brand` display name (never the slug). When all items are done, summarize and **stop**. Do not write a new brand in the same run.
4. If `backfill` is `[]` and `next` is `null`: tell the user the catalog queue is empty and **stop**.
5. `GET …/api/blog.php?brand={next.slug}&token=`
6. If that call fails: **stop**.
7. If `pages` is non-empty: **stop** (do not POST).
8. If `pages` is empty: research and draft that one brand (plus `next.extra_page` if present), including ambassadors when sources exist.
9. Immediately before POST, repeat the brand GET. If it fails or `pages` is non-empty: **stop**.
10. `POST` with `published: false` and `brand` as the human name from the catalog (e.g. `Ben Sherman`, never `bensherman`). Never send `overwrite: true`.
11. Summarize brand + slugs saved. If POST returns 409, report it and **stop**.

Wait at least 45s between API calls if the host rate-limits.

## Subbrands in the catalog

Wardrobe names are not always the house name. Write the **parent**, plus one extra page about the line. API `next.extra_page` is `{slug, title}` when the catalog has a line.

| If admin catalog has | Write hub as | Extra page |
|---|---|---|
| Calvin Klein Jeans | Calvin Klein (`calvinklein`) | slug `jeans` — линия Calvin Klein Jeans |
| Marciano By Guess | Guess (`guess`) | slug `marciano` — линия Marciano By Guess |

Same pattern later: Polo Ralph Lauren → Ralph Lauren + page `polo`, etc. Infer parent from the name.

Extra page: history of that line, how it relates to the house, typical products. Title like «Calvin Klein Jeans». Hub links «подробнее» to it.

## Pages (skip if sources do not support the section)

| API `slug` | Public URL | Role |
|---|---|---|
| `index` | `/blog/{brand}` | Hub. H1 = brand name. Short “what they are famous for / what they make now” (2–4 sentences). |
| `founder` | `/blog/{brand}/founder` | Who founded, how they started, other businesses of the founder. |
| `history` | `/blog/{brand}/history` | First product, how it got popular, most famous product. |
| `culture` | `/blog/{brand}/culture` | Music, musicians, subcultures, exhibitions, charity — only sourced facts. |
| `ambassadors` | `/blog/{brand}/ambassadors` | Known people who were the face of the brand or promoted it by choice, not because they were paid. Skip hired campaign models unless they are also independently associated with the house. Skip if you cannot source unpaid / iconic association. |
| `logo` | `/blog/{brand}/logo` | Meaning and evolution of the mark. Describe it in words; do not insert a logo image in HTML. |
| `owner` | `/blog/{brand}/owner` | Current owner, stock. |
| `stores` | `/blog/{brand}/stores` | Moscow offline, SPb offline, Russia online. |
| *(subbrand)* | `/blog/{brand}/{slug}` | Only when the catalog has a distinct line (Jeans, Marciano, …). |

On the hub, link to other sections in ordinary encyclopedia language («Подробнее об истории», «об амбассадорах»). Never explain how the site, the admin catalog, templates, or CMS insert logos or store fields.

## Voice, sources, forbidden

- Russian, **Вы**, encyclopedic but not stiff. Never invent; omit empty sections.
- If a source is in English (or another language), use it and write the article in Russian. Do not leave English body copy.
- Sources to search (in addition to official sites, marketplaces, and books):
  - https://www.wikipedia.org/
  - https://bre.ruwiki.ru/
  - http://www.scholarpedia.org/
  - https://www.infoplease.com/
  - https://www.rubricon.com/
  - https://www.britannica.com/
  - https://www.encyclopedia.com/
  - https://discover.hubpages.com/
  - https://citizendium.org/
- No `<script>`, JSON-LD, OG, or breadcrumbs in `content`.
- Do not mention the Russia–Ukraine war, invasion, «СВО», sanctions, or political statements about that conflict. Closed shops: status only.
- Do not write about technical aspects of creating the articles or how the blog is built. Readers must never be told how the site “подставляет” a logo, where files live, what a template does, or what the admin catalog is. Forbidden examples (and anything like them): «актуальный логотип на сайте itfits берётся из админского каталога», «картинку знака в HTML этой страницы мы не ставим — шаблон уже показывает его сверху», «картинку логотипа в этот текст мы не ставим: на сайте её подставляет каталог», «эта страница хранится как slug», «в API поле published». If you have nothing encyclopedic to say about the mark, describe its history and meaning only.

## Images

- Add a `<figure>` **only** when the photo is unique to that page and you have a free stable URL. If nothing fits, **no image**.
- Search order for photos (keep going until a **reusable** license is found: CC BY/SA, public domain, official press still with clear reuse, Wikimedia, or the encyclopedia source itself allows reuse of that file):
  1. Wikimedia Commons API (`list=search` namespace 6) and Wikipedia infobox / `File:` on EN/IT/RU pages.
  2. The encyclopedias listed above (Wikipedia, Britannica, Encyclopedia.com, Citizendium, Scholarpedia, Infoplease, Rubricon, bre.ruwiki.ru, HubPages) — use their images **only** when the page or file states reuse is allowed. Do not hotlink a copyrighted Britannica/Encyclopedia.com still just because it illustrates the topic.
  3. Openverse / Flickr Commons (`license=cc` or `pd`), Europeana, Internet Archive images.
  4. Official brand heritage / about pages, Massimo Osti Archive, company press kits — **only** if the page states reuse; otherwise skip (portraits of living/recent designers are usually © archive).
  5. National portraits / museum collections (NPG, Rijksmuseum, MET Open Access).
- Do not hotlink random magazine scans, Instagram, or WWD “courtesy of” shots.
- Do not reuse the catalog logo or the same photo on several pages.
- Hub and logo pages: do not duplicate the brand mark as an `<img>` in content.
- Captions left-aligned (template CSS). One figure max per page unless culture truly needs two distinct sourced photos.
- `og_image`: catalog `logo_url` if present, or the page’s unique still (not a GIF). Do not mention this choice in the article text.

## SEO

- `seo_title` ~70 chars, `seo_description` 140–160, one H1 from `title`.
- `faq`: 3–6 items this page only, same wording as body; skip if you cannot.

## Publish

1. Token from env or `www/config/config.php` — never copy it into chat or this skill.
2. `POST https://itfits.vakhromeev.com/api/blog.php?token=` for a new brand; `PUT …?brand={slug}&page={slug}&token=` for a missing section.
3. `published: false`. Hub slug `index`.
4. After approval, set `published: true` with PUT/PATCH (not a new POST of the whole brand).

See [reference.md](reference.md).
