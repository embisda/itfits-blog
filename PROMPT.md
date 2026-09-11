# Cursor automation prompt

Paste this into the automation instructions. Bind the run to **this git repository** (not a Downloads folder). Set secret `BLOG_API_TOKEN` to the same value as `blog_api_token` in `www/config/config.php`. Never put the token in the prompt.

After PHP changes are on the live host, the first API call must be `?next=1`.

```
Use the skill at automation/SKILL.md (same content as .cursor/skills/itfits-blog-publisher/SKILL.md). If the workspace root does not contain that file, search the repo for it. Do not create a new project.

Each run: follow API next only. Do not keep a hardcoded brand list.

Token: environment variable BLOG_API_TOKEN. Never print it. Call https://itfits.vakhromeev.com/api/blog.php with ?token= (the host often strips Authorization). Do not use MySQL.

First call: GET https://itfits.vakhromeev.com/api/blog.php?next=1&token=
Write only next.slug. If next.missing_pages is set, PUT those pages only (published: false). If the request fails or times out, wait 45s, retry up to 3 times, then STOP. Do not guess brands. If next is null, STOP.

Then GET ?brand={slug}&token=. If that fails, STOP. For a new brand, if pages is not empty, STOP. Repeat that GET immediately before POST.

One brand or one missing-section backfill per run. Save with published: false. Never set published: true. Never send overwrite: true.

Articles: Russian, Вы. English sources are OK — translate into Russian. Use Wikipedia, Britannica, Encyclopedia.com, Citizendium, Scholarpedia, Infoplease, Rubricon, bre.ruwiki.ru, HubPages plus official sites. Images from those sites only if reuse is allowed; otherwise Commons/CC. Do not write about how the blog or admin catalog works. Do not mention templates, slugs, or “логотип берётся из админки”.

Ambassadors page: people who were the face of the brand or promoted it by choice, not for money. Skip if unsourced.

Do not open a pull request unless you must edit skill files. Summarize which brand and slugs you saved.
```
