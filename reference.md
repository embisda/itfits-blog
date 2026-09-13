# itfits blog API and research

Companion to `SKILL.md`. Writing rules live there. This file is the API contract and a short research checklist.

## Auth

```
GET/POST/PUT/PATCH/DELETE https://www.itfits.vakhromeev.com/api/blog.php?token=
```

Prefer `?token=` (the host often strips `Authorization: Bearer`). Token is `BLOG_API_TOKEN` or `blog_api_token` in `www/config/config.php`. Never print it.

GET without token returns only published pages (drafts look like “no pages”). Always use the token to decide what to write.

Do not open a remote MySQL port from the laptop.

## Pick the next brand

```
GET /api/blog.php?next=1&token=
```

```json
{
  "ok": true,
  "next": { "name": "Lacoste", "slug": "lacoste", "extra_page": null, "missing_pages": [] },
  "backfill": [],
  "queue": []
}
```

`next` is the next catalog brand with **zero** pages. `next` is `null` when the catalog has no unwritten brands.

**Ignore `backfill`.** Automation never completes missing sections on brands that already have pages. Rewrite only when the user asks (see SKILL.md «How the user asks for a rewrite»).

Always send `brand` as the human catalog name (`Ben Sherman`), never the slug (`bensherman`).

Confirm emptiness for a **new** brand: `GET /api/blog.php?brand=…&token=` → `pages` must be `[]` before POST.

Timeout / 401 / 5xx / non-JSON: stop. Do not treat that as an empty encyclopedia.

## POST body

New brand only. If the brand already exists, API returns **409** unless `"overwrite": true` (never send overwrite).

```json
{
  "brand": "Lacoste",
  "brand_slug": "lacoste",
  "published": false,
  "pages": [
    {
      "slug": "index",
      "title": "Lacoste",
      "excerpt": "Чем знамениты и чем занимаются сейчас — одно предложение.",
      "seo_title": "Lacoste — бренд и крокодил | itfits",
      "seo_description": "140–160 символов про эту страницу.",
      "og_image": "https://…",
      "related_brands": ["fredperry", "ralphlauren", "calvinklein"],
      "faq": [
        {"question": "Основатель", "answer": "Рене Лакост основал марку после теннисной карьеры."},
        {"question": "История", "answer": "Дом вырос из рубашки-поло и знака крокодила."}
      ],
      "content": "<p>Чем знамениты и чем занимаются сейчас. <a href=\"/blog/lacoste/history\">Подробнее об истории</a>.</p>"
    },
    {
      "slug": "founder",
      "title": "Основатель Lacoste",
      "excerpt": "Одно предложение для Содержание на хабе.",
      "seo_title": "…",
      "seo_description": "…",
      "faq": [],
      "content": "<h2>Рене Лакост</h2><p>…</p><h2>Источники</h2><ul><li><a href=\"https://en.wikipedia.org/wiki/René_Lacoste\">René Lacoste — Wikipedia</a></li></ul>"
    }
  ]
}
```

Hub `faq` = Содержание: `question` is the section title, `answer` is the child `excerpt`. Child pages omit hub-style FAQ unless they have their own 3–6 items from the body.

`related_brands` only on `index`. Child pages omit it.

Update one page (rewrite): `PUT /api/blog.php?brand=lacoste&page=history&token=`.

Delete: `DELETE /api/blog.php?brand=lacoste&page=founder&token=` or whole brand without `page`.

Canonical hub URL is `/blog/{brand}`, never `/blog/{brand}/index` (that 301s).

Human queue and drafts: `/admin/blog.php`.

## Research checklist

- Confirm each claim against a named source before writing.
- In the body: no «взято из Википедии»; named-person quotes OK. End of page: `h2` Источники with links when possible.
- English sources are fine: translate into Russian.
- Images: reusable license only; Wikimedia original file URL, never `/thumb/.../800px-`. Put a figure only when it matches the heading and the paragraph; skip rather than pad.
- Stores: three H2s — Москва, Санкт-Петербург, Онлайн в России. Monobrand first, else multibrand. Skip the page if nothing is verifiable.
- Culture H2s: subcultures and music genres. Omit unsourced names.
- Ambassadors: two groups (unpaid affinity vs paid representation), h3 = names. Skip the page if unsourced.
- Subbrand page when `next.extra_page` is set.
- Owner: current parent; stock H2 even if the answer is «не торгуется». Do not guess a ticker.
- Logo: describe the mark; no logo `<img>` in content.
- Similar brands on hub: 3–5 from the catalog that are actually close.
- Never mention the Russia–Ukraine war or related political framing.
- Never describe how itfits stores logos, templates, slugs, or the admin catalog.
- Next brand: API `next` only. Ignore `backfill`. Do not keep a local brand list.

## HTML allowed in content

`p, br, h2, h3, h4, ul, ol, li, strong, b, em, i, a, img, blockquote, figure, figcaption`

One H1 comes from `title` in the template — do not put `<h1>` in content. Hub H1 is the brand name; hub `title` can match the brand name.
