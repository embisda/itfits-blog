# Cursor automation prompt

Paste this into the automation instructions. Bind the run to **this git repository** (not a Downloads folder). Set secret `BLOG_API_TOKEN` to the same value as `blog_api_token` in `www/config/config.php`. Never put the token in the prompt.

After PHP changes are on the live host, the first API call must be `?next=1`.

```
Use the skill at automation/SKILL.md (same content as .cursor/skills/itfits-blog-publisher/SKILL.md). If the workspace root does not contain that file, search the repo for it. Do not create a new project.

Token: environment variable BLOG_API_TOKEN. Never print it. Call https://www.itfits.vakhromeev.com/api/blog.php with ?token=. Do not use MySQL. Do not keep a hardcoded brand list.

First call: GET https://www.itfits.vakhromeev.com/api/blog.php?next=1&token=

Ignore backfill even if it is not empty. Never PUT missing pages onto brands that already have encyclopedia pages. Never audit or rewrite old brands unless this chat explicitly names a brand and asks to rewrite it.

Write only next (one new catalog brand, full set of pages that sources support). If next is null, STOP. GET ?brand={slug}&token= first; if pages is not empty, STOP. Repeat that GET immediately before POST. Never send overwrite: true.

Save with published: false. Never set published: true.

Follow the Writing section of the skill: max two screens per page; hub = чем знамениты и чем занимаются сейчас; required H2 outlines; always try collaborations, cinema (docs + fiction, Kinopoisk/IMDb links), and ad campaigns; ambassadors unpaid vs paid; stores H2 Москва / Санкт-Петербург / Онлайн в России; hub faq = Содержание (названия разделов + excerpt); figure under each H2/H3 only when it matches; end of article Источники with links; do not say the text is from Wikipedia; named-person quotes OK.

Articles: Russian, Вы. English sources are OK — translate. Never tell the reader how the site, template, or admin catalog works. Do not close sentences with the template «…, а не …»; vary the wording.

Wikimedia photos: original file URL without /thumb/ and without 800px.

Do not open a pull request unless you must edit skill files. Summarize which brand and slugs you saved.
```
