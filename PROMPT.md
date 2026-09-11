# Cursor automation prompt

Paste this into the automation instructions. Bind the run to **this git repository** (not a Downloads folder). Set secret `BLOG_API_TOKEN` to the same value as `blog_api_token` in `www/config/config.php`. Never put the token in the prompt.

After PHP changes are on the live host, the first API call must be `?next=1`.

```
Use the skill at automation/SKILL.md (same content as .cursor/skills/itfits-blog-publisher/SKILL.md). If the workspace root does not contain that file, search the repo for it. Do not create a new project.

Token: environment variable BLOG_API_TOKEN. Never print it. Call https://itfits.vakhromeev.com/api/blog.php with ?token=. Do not use MySQL. Do not keep a hardcoded brand list.

First call: GET https://itfits.vakhromeev.com/api/blog.php?next=1&token=

If backfill is a non-empty array: this run is a skill-change backfill. For EVERY item in backfill, PUT every missing_pages slug (published: false) with brand set to the human name (Ben Sherman, not bensherman). Do all brands in this one run. Then STOP. Do not start a new brand in the same run.

If backfill is empty: write only next (one new catalog brand, full set of pages). If next is null, STOP. GET ?brand={slug}&token= first; if pages is not empty, STOP. Repeat that GET immediately before POST. Never send overwrite: true.

Save with published: false. Never set published: true.

Articles: Russian, Вы. English sources are OK — translate. Encyclopedias listed in the skill plus official sites. Never tell the reader how the site, template, or admin catalog works. Never write that a logo is “подставляется” or taken from the catalog. Describe the mark’s history only.

Ambassadors: people who were the face of the brand or promoted it by choice, not for money. Skip a page if unsourced.

Do not open a pull request unless you must edit skill files. Summarize which brands and slugs you saved.
```
