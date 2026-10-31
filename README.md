# Die Chroniken von Rubor – Website

Campaign wiki for our D&D group. This repo holds **content only** (Markdown + assets). The site is built with [Quartz 4](https://quartz.jzhao.xyz/), a static site generator, and served by GitHub Pages.

## How a build works

Every push to `main` runs [.github/workflows/deploy.yml](.github/workflows/deploy.yml):

1. Check out the Quartz fork [`fanosta/quartz`](https://github.com/fanosta/quartz) (default branch, not pinned) to `quartz/`.
2. Check out this repo to `quartz/content/` with **full git history**, which Quartz uses for page created/modified dates.
3. Run `npm ci` with Node 22.
4. Copy [.config/quartz.config.ts](.config/quartz.config.ts) and [.config/quartz.layout.ts](.config/quartz.layout.ts) over the fork's defaults.
5. `sed` the `baseUrl` to `<owner>.github.io/<repo>`.
6. If `_attachments/icon.png` exists, copy it to `quartz/static/icon.png` (favicon).
7. `npx quartz build`, which writes static HTML/CSS/JS to `quartz/public/`.
8. Upload `public/` and deploy it with `actions/deploy-pages`.

GitHub repo setting: *Settings → Pages → Source: GitHub Actions*.

**Use the fork, not upstream Quartz.** The layout uses `Component.FrontMatterList()`, which upstream doesn't have.

## Building locally / on another server

```sh
git clone https://github.com/fanosta/quartz && cd quartz
npm ci
git clone git@github.com:dnd-graz/rubor.git content   # full clone, not --depth 1
cp content/.config/quartz.config.ts content/.config/quartz.layout.ts .
npx quartz build --serve      # preview on http://localhost:8080
npx quartz build              # or: static output in public/
```

To host somewhere other than GitHub Pages:

- Set `baseUrl` in `quartz.config.ts` to the new host and path, without the protocol (e.g. `example.org/rubor`). It's used for absolute links, the sitemap and the RSS feed.
- Serve `public/` as static files. Links have no extension (`/Personen/Ezekiel` → `Ezekiel.html`), so the server must resolve them. For nginx: `try_files $uri $uri.html $uri/ =404;`. Quartz also emits a `404.html`.
- The `tmp_data/` and `raw_audio/` folders hold GM-only material. They're gitignored, so they never reach CI, but **a local build from a working copy would publish them**. Build from a clean clone, or add them to `ignorePatterns`.

## Content conventions

- Pages live in `Charaktere/`, `Personen/`, `Orte/` and `Kapitel/`, with `index.md` as the start page. The folder structure is the site navigation.
- Frontmatter: `title`, optional `aliases` (each alias gets a redirect page), `tags`. `draft: true` excludes a page from the build.
- Links are Obsidian-style `[[Page]]` / `[[Page|Text]]` / `[[Page#Heading]]`, resolved by shortest unique path, so folder names aren't needed.
- Images go in `_attachments/` and audio in `audio/`. Embed them with `![[file.jpg|320]]` / `![[file.mp3]]`.
- Paths matched by `ignorePatterns` in the config (`private`, `templates`, `.obsidian`, this README) aren't published.
- All site content is in German. The site is read by the players, so nothing goes in that they haven't learned at the table.
