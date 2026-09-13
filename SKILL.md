---
name: itfits-blog-publisher
description: Researches and drafts itfits brand encyclopedia pages, then publishes via the blog API as unpublished drafts. Use when writing blog posts for itfits, publishing brand articles, working with /blog/, or the blog_api_token workflow.
---

# itfits blog publisher

This file and `automation/SKILL.md` must stay identical. Cloud automations load **`automation/SKILL.md`**. Local Cursor can use this path.

Two layers in this file:

1. **Writing** — how an article looks and what it contains.
2. **Automation** — how a run picks a brand and saves a draft.

API shapes and HTML allowlist: [reference.md](reference.md).

---

# Writing

## Goal

Write a Russian encyclopedia of clothing brands for itfits. One catalog house = one hub plus child pages. Reader-facing copy only. Never invent. Skip a page if sources cannot support it.

## Length

Each page: **no more than two screens of body text** on a laptop (about 3 500–5 000 characters of visible HTML text, not counting FAQ, Contents, or the sources list). Hub is shorter: **2–4 sentences**. Do not pad. If there is more material, keep only the facts that earn an H2.

## Voice

Russian, **Вы**, encyclopedic but not stiff. Translate any non-Russian source; do not leave English body copy.

H1 comes from `title` (template). Do **not** put `<h1>` in `content`. Child headings in `content` start at **h2**.

Culture `title` / H1 is always **`{название бренда} в культуре`** — the same formula on every house (`Calvin Klein в культуре`, `Fila в культуре`). Never «Культура Calvin Klein», «Культура дома», or a free paraphrase. The hub TOC label stays «В культуре» (`blogSectionLabels()`). Other child H1s stay the existing templates (`Основатель {бренд}`, `История {бренд}`, …).

## Do not rewrite old brands (default)

Default work is **new brands only** (API `next`, zero pages).

Do **not** audit, complete, or rewrite brands that already have pages. Ignore API `backfill`, `missing_pages`, and gaps on old houses.

Rewrite only when the user **explicitly asks** (see «How the user asks for a rewrite» below). Do not expand one named brand into a catalog-wide backfill unless they say «все уже написанные бренды».

## Hub vs child pages

The hub is **not** a retelling of founder/history/culture.

Hub text (2–4 sentences) = **чем бренд знаменит** и **чем занимается сейчас**. Then ordinary encyclopedia links («Подробнее об истории», «об амбассадорах»). Details live on child pages.

The visible **Содержание** on `/blog/{brand}` is built from child pages automatically (section title + `excerpt`). Do not duplicate that list inside hub `content`.

## Pages and H2 outline

Skip the whole page if you cannot fill its required H2s with sourced facts.

| API `slug` | URL | Role and H2 |
|---|---|---|
| `index` | `/blog/{brand}` | Hub. `title` / H1 = brand name. 2–4 sentences: чем знамениты + чем занимаются сейчас. Links to children. No extra H2 unless a subbrand line needs «подробнее». |
| `founder` | `/blog/{brand}/founder` | **h2 = имя основателя** — биография и как начал этот бизнес. Второй **h2 «Другие бренды основателя»** только если у человека реально были другие марки / компании в моде; иначе не ставить. |
| `history` | `/blog/{brand}/history` | H2 = ключевые вещи и события страницы (изделия, даты, переломы), не «Ранние годы». Пример для Alpha Industries: «Аляска N-3B», «Бомбер MA-1», «Как куртка стала гражданской». |
| `culture` | `/blog/{brand}/culture` | `title` / H1 = **`{бренд} в культуре`**. H2 = названия **субкультур**, которые котируют бренд, и/или **музыкальных жанров**, если бренд котируется у известных исполнителей. Только проверяемые факты. |
| `ambassadors` | `/blog/{brand}/ambassadors` | Две группы (пустую не писать). **h2 «По собственному выбору»** — носили / показывали марку, потому что она им близка, без рекламного контракта как главной причины. **h2 «Представляли бренд за деньги»** — кампании, контракты, paid face of the brand. Внутри группы **h3 = имя**. Нужен хотя бы один sourced человек; иначе страницу пропустить. |
| `logo` | `/blog/{brand}/logo` | По порядку, пропускать пустые: **h2 «Первый логотип»**, **h2 «Эволюция логотипа»**, **h2 «Текущий логотип»**, **h2 «Другая символика»** (орёл, крокодил, патч, крой — то, по чему дом узнают без слова-марки). Знак **словами**; `<img>` логотипа в контент не ставить. |
| `owner` | `/blog/{brand}/owner` | **h2 = название текущего владельца** (группа / фонд / частное лицо). **h2 «Бренд на бирже»** — тикер и биржа, если бумага есть; если дом частный — так и написать, не угадывать тикер. Можно уточнить, что на бирже торгуется **родитель**, а не сама марка. |
| `stores` | `/blog/{brand}/stores` | Три обязательных h2, даже если блок короткий: **«Москва»**, **«Санкт-Петербург»**, **«Онлайн в России»**. Сначала монобренды (собственный магазин / официальный сайт марки). Если монобренда нет — мультибренды, где марка реально продаётся. Только проверяемые точки. Закрыто: статус одной фразой, без политики. Нет ни одной проверяемой точки ни в одном городе и онлайн — страницу пропустить. |
| *(линия)* | `/blog/{brand}/{slug}` | Только если каталог/API дал линию. История линии, связь с домом, типичные продукты. Заголовок как «Calvin Klein Jeans». H2 по тому же принципу, что у history. |

### Subbrands in the catalog

Wardrobe name is not always the house. Write the **parent**, plus one extra page. API `next.extra_page` is `{slug, title}`.

| Catalog name | Hub | Extra page |
|---|---|---|
| Calvin Klein Jeans | Calvin Klein (`calvinklein`) | `jeans` |
| Marciano By Guess | Guess (`guess`) | `marciano` |

Same later: Polo Ralph Lauren → Ralph Lauren + `polo`. Infer parent from the name. Do not invent a second house.

## Sources

Confirm every claim against a named source **before** writing. Preferred: official sites, marketplaces, books, plus:

- https://www.wikipedia.org/
- https://bre.ruwiki.ru/
- http://www.scholarpedia.org/
- https://www.infoplease.com/
- https://www.rubricon.com/
- https://www.britannica.com/
- https://www.encyclopedia.com/
- https://discover.hubpages.com/
- https://citizendium.org/

**In the body:** do **not** write that the text is taken from Wikipedia, Britannica, or any encyclopedia («по данным Википедии», «согласно Encyclopedia.com»). Use the fact, not the catalog.

If a topic has **no** sourced fact, **omit it**. Do not narrate the gap. Forbidden (and anything like them): «отдельных программ в этот текст не включаем», «проверяемых данных нет, поэтому не пишем», «энциклопедии об этом молчат», «крупных благотворительных программ не нашли». Silence is enough.

**Quotes are allowed** when a **real named person** said it (журналист, писатель, основатель, дизайнер, представитель бренда): «Как говорил Рене Лакост, …», «В интервью Vogue креативный директор N сказал, что …». Do not invent quotes.

**At the end of the page**, when you used web/print sources, add a sources list (does not count toward the two-screen limit):

```html
<h2>Источники</h2>
<ul>
<li><a href="https://example.com/page">Короткое название источника</a></li>
</ul>
```

Wikipedia and other encyclopedias **may** appear in this list. Other in-body links: internal `/blog/...` and store URLs on `stores`.

## Forbidden in copy

- `<script>`, JSON-LD, OG, breadcrumbs in `content`.
- Russia–Ukraine war, invasion, «СВО», sanctions, political framing. Closed shops: status only.
- How itfits, templates, the admin catalog, slugs, or `published` work. Forbidden (and anything like them): «логотип берётся из админского каталога», «картинку в HTML не ставим — шаблон уже показывает».

## Images

- Wikimedia: **never** `/thumb/` URLs with a pixel width (especially `800px-`). Use the original: `https://upload.wikimedia.org/wikipedia/commons/{hash}/{file}` or `Special:FilePath/Filename.jpg` with **no** `?width=`. If a thumb is unavoidable, only `500px-` has been reliable.
- Search until a **reusable** license exists (CC BY/SA, PD, press still with clear reuse, Wikimedia, or the encyclopedia page allows reuse of that file):
  1. Wikimedia Commons API (`list=search` ns 6) and Wikipedia `File:` on EN/IT/RU.
  2. Encyclopedias above — only if reuse is stated. Do not hotlink a copyrighted Britannica still.
  3. Openverse / Flickr Commons / Europeana / Internet Archive.
  4. Official heritage / press kits — only if reuse is stated.
  5. Museum OA (NPG, Rijksmuseum, MET Open Access).
- No magazine scans, Instagram, or WWD “courtesy of”.
- Do not reuse the catalog logo or the same photo on several pages.
- Hub and logo pages: do not duplicate the brand mark as `<img>` in content.
- A `<figure>` may sit under **any** H2, including several on one page, **only if** the photo matches that heading **and** the adjacent paragraph (джинсы Marilyn → эти джинсы, не куртка; хип-хоп / Nas → портрет, не витрина магазина). If nothing licensed matches, skip the image. Never pad with a random product, storefront, or “brand-flavored” still.
- Captions left-aligned.
- `og_image`: catalog `logo_url` if present, else the page’s unique still (not a GIF). Do not mention this in the article.

## SEO and fields

- `seo_title` ~70 characters, `seo_description` 140–160, one H1 from `title`.
- `excerpt` on **every child page**: one sentence. This is the summary in hub **Содержание**.
- Hub `faq`: same as Содержание. One item per child page you actually wrote. `question` = section title (`Основатель`, `История`, `В культуре`, … — `blogSectionLabels()`). `answer` = that page’s `excerpt`. Do not invent extra Q&A on the hub. Do not duplicate Contents in hub HTML.
- Child-page `faq`: optional 3–6 items from **that** page’s body; skip if you cannot.
- `related_brands`: only on `index`, 3–5 close houses from the catalog.

---

# Automation

Save as `published: false`. Never `published: true`. The user publishes from admin.

Do not open a pull request unless you must edit skill files.

## Workspace

Canonical folder: `/Users/embisda/Documents/_Project/Web/itfits.vakhromeev.com`. Site files: `www/`. Look for `automation/SKILL.md` then `.cursor/skills/itfits-blog-publisher/SKILL.md`. Do not create a new project. Do not use a Downloads copy.

## Cadence

- One **new** catalog brand per run (full encyclopedia for that house, including extra line page when API sends `extra_page`).
- If API `next` is `null`: catalog has no unwritten brands. Stop.
- Adding a new page type: put it in `blogSectionLabels()` / this skill so **future** new brands get it. Do **not** turn on catalog-wide backfill. Old brands get the new section only if the user asks to rewrite that house.

## Source of truth

Do **not** keep a hardcoded brand list.

- Catalog = API `catalog` / https://www.itfits.vakhromeev.com/admin/brands.php
- What to write = API `next` only (one house with zero pages), unless the user asked for a rewrite.
- Ignore `backfill`. Map catalog line names to a parent house.

## Run protocol (mandatory)

Token: env `BLOG_API_TOKEN` or `blog_api_token` in `www/config/config.php`. Never print it. Call `https://www.itfits.vakhromeev.com/api/blog.php?token=` (host often strips `Authorization`). Do not use MySQL.

Timeout, 5xx, 401, empty body, or non-JSON = **failure**. Failure ≠ “brand has no pages”.

1. `GET https://www.itfits.vakhromeev.com/api/blog.php?next=1&token=`
2. On failure: wait 45s, retry up to 3 times, then **stop**. Do not guess the next brand.
3. Ignore `backfill` even if the array is not empty.
4. If `next` is `null`: tell the user the catalog queue is empty and **stop**.
5. `GET …/api/blog.php?brand={next.slug}&token=`
6. If that call fails: **stop**.
7. If `pages` is non-empty: **stop** (do not POST, do not fill missing slugs).
8. If `pages` is empty: research and draft that one brand (plus `next.extra_page` if present).
9. Immediately before POST, repeat the brand GET. If it fails or `pages` is non-empty: **stop**.
10. `POST` with `published: false` and `brand` as the human catalog name (`Ben Sherman`, never `bensherman`). Never send `overwrite: true`.
11. Summarize brand + slugs saved. If POST returns 409, report it and **stop**.

Wait at least 45s between API calls if the host rate-limits.

## How the user asks for a rewrite

Default cron/chat does **not** touch old brands.

Treat as a rewrite **only** if the user names the house (or says all existing houses) **and** asks to rewrite / переписать / пересобрать / дописать недостающие страницы. Example phrases:

- «Перепиши Stone Island по новым правилам и создай страницы, которых ещё не было»
- «Пересобери Alpha Industries: обнови существующие разделы и допиши амбассадоров, если их нет»
- «Пройди все уже написанные бренды по новым правилам и допиши недостающие страницы»

Then, for each named brand (or each encyclopedia brand if they said «все»):

1. `GET ?brand={slug}&token=`
2. Rewrite **existing** pages with `PUT …?brand={slug}&page={slug}&token=`, `published: false`. Never POST the whole brand again. Never `overwrite: true`.
3. For sections in this skill that are still missing and sources support them: `PUT` those new pages too (`published: false`).
4. Stop after the named set. Do not continue through API `next` in the same rewrite request unless they also asked for new brands.

If they only say «напиши следующий бренд» — that is **not** a rewrite.
