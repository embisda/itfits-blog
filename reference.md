# itfits blog API and research

## Auth

```
Authorization: Bearer <blog_api_token from www/config/config.php>
POST / PUT / PATCH / DELETE https://itfits.vakhromeev.com/api/blog.php
```

GET without token returns only published pages. GET with token can read drafts (`?brand=lacoste` or `?brand=lacoste&page=history`).

With token and no `brand`, the list also includes `catalog`: `{name, logo_url}` from the wardrobe `brands` table. Same data: `GET /api/blog.php?catalog=1&token=`. Do not open a remote MySQL port from the laptop.

## POST body

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

Update one page: `PUT /api/blog.php?brand=lacoste&page=history`.

Delete: `DELETE /api/blog.php?brand=lacoste&page=founder` or whole brand without `page`.

Canonical hub URL is `/blog/{brand}`, never `/blog/{brand}/index` (that 301s).

## Research checklist

- Confirm each claim against a named source before writing.
- Stores: only shops/sites you can verify for Moscow, St. Petersburg, Russia online. If none, skip `stores`. Closed shops: status only, no political explanation.
- Culture: music/musicians, subculture, exhibition, charity — omit unsourced names. Skip the page if empty.
- Subbrand page when the catalog name is a line (Calvin Klein Jeans → parent + `/jeans`).
- Owner/stock: use current corporate parent; do not guess ticker.
- Logo page: no `<img>` of the logo in content; PHP uses `brands.logo_url`.
- Images: only if unique and sourced; never duplicate the catalog logo; skip rather than filler.
- Similar brands on hub: pick 3–5 from the queue that are actually close.
- Never mention the Russia–Ukraine war or related political framing.
- Before the next brand: merge `brands` table names (mapped to parents) with the queue; skip brands that already have API pages.

## HTML allowed in content

`p, br, h2, h3, h4, ul, ol, li, strong, b, em, i, a, img, blockquote, figure, figcaption`

One H1 comes from `title` in the template — do not put `<h1>` in content. Hub H1 is the brand name; hub `title` can match the brand name.
