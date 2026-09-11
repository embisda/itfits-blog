# itfits blog API and research

## Auth

```
GET/POST/PUT/PATCH/DELETE https://itfits.vakhromeev.com/api/blog.php?token=
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
  "backfill": [
    {
      "name": "Stone Island",
      "slug": "stoneisland",
      "missing_pages": ["ambassadors"],
      "extra_page": null
    }
  ],
  "queue": []
}
```

`next` is the next catalog brand with **zero** pages. `backfill` is every existing encyclopedia brand that still lacks a newly added section (currently `ambassadors`). `status` is `queued` | `draft` | `published` | `mixed`.

If `backfill` is not empty, `PUT` **all** those missing pages in one run, then stop. Do not POST a full brand. Do not write `next` in the same run.

After `backfill` is `[]`, write `next` as a full new brand. `next` is `null` when the catalog has no unwritten brands.

Always send `brand` as the human catalog name (`Ben Sherman`), never the slug (`bensherman`).

Confirm emptiness for a **new** brand: `GET /api/blog.php?brand=…&token=` → `pages` must be `[]` before POST.

Timeout / 401 / 5xx / non-JSON: stop. Do not treat that as an empty encyclopedia.

## POST body

New brand only. If the brand already exists, API returns **409** unless `"overwrite": true` (automation must never send overwrite).

```json
{
  "brand": "Lacoste",
  "brand_slug": "lacoste",
  "published": false,
  "pages": [
    {
      "slug": "index",
      "title": "Lacoste",
      "excerpt": "Одно предложение для оглавления.",
      "seo_title": "Lacoste — бренд и крокодил | itfits",
      "seo_description": "140–160 символов про эту страницу.",
      "og_image": "https://…",
      "related_brands": ["fredperry", "ralphlauren", "calvinklein"],
      "faq": [
        {"question": "Вопрос как в тексте", "answer": "Ответ как в тексте"}
      ],
      "content": "<p>2–4 предложения. <a href=\"/blog/lacoste/history\">Подробнее об истории</a>.</p>"
    },
    {
      "slug": "founder",
      "title": "Основатель Lacoste",
      "excerpt": "Короткая подпись для содержания на хабе.",
      "seo_title": "…",
      "seo_description": "…",
      "faq": [],
      "content": "<p>…</p>"
    }
  ]
}
```

`related_brands` only on `index`. Child pages omit it.

Update one page: `PUT /api/blog.php?brand=lacoste&page=history&token=`.

Delete: `DELETE /api/blog.php?brand=lacoste&page=founder&token=` or whole brand without `page`.

Canonical hub URL is `/blog/{brand}`, never `/blog/{brand}/index` (that 301s).

Human queue and drafts: `/admin/blog.php`.

## Research checklist

- Confirm each claim against a named source before writing. Prefer the encyclopedias in SKILL.md (Wikipedia, Britannica, Encyclopedia.com, Citizendium, Scholarpedia, Infoplease, Rubricon, bre.ruwiki.ru, HubPages) plus official sites. English sources are fine: translate into Russian.
- Images from those encyclopedias only when reuse is allowed; otherwise Commons / CC / museum OA.
- Stores: only shops/sites you can verify for Moscow, St. Petersburg, Russia online. If none, skip `stores`. Closed shops: status only, no political explanation.
- Culture: music/musicians, subculture, exhibition, charity — omit unsourced names. Skip the page if empty.
- Ambassadors: people independently associated with the house or historically its face; not paid campaign-only models. Skip the page if unsourced.
- Subbrand page when `next.extra_page` is set, or the catalog name is a line (Calvin Klein Jeans → parent + `/jeans`).
- Owner/stock: use current corporate parent; do not guess ticker.
- Logo page: describe the mark; do not put a logo `<img>` in content.
- Images: only if unique and sourced; never duplicate the catalog logo; skip rather than filler.
- Similar brands on hub: pick 3–5 from the catalog/queue that are actually close.
- Never mention the Russia–Ukraine war or related political framing.
- Never describe how itfits stores logos, templates, slugs, or the admin catalog inside article copy.
- Before the next brand: use API `next` only. Do not keep a local brand list.

## HTML allowed in content

`p, br, h2, h3, h4, ul, ol, li, strong, b, em, i, a, img, blockquote, figure, figcaption`

One H1 comes from `title` in the template — do not put `<h1>` in content. Hub H1 is the brand name; hub `title` can match the brand name.
