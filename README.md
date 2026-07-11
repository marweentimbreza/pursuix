# pursuix

Personal site and blog of a defensive-security practitioner — DFIR, detection
engineering, and notes from someone learning to defend, in the open.

**Live:** https://pursuix.com · **Writing desk:** https://myspace.pursuix.com

## Stack

- [Astro](https://astro.build) static site — terminal-styled, markdown content
  collection for notes, built and deployed by Cloudflare Pages on every push.
- [Sveltia CMS](https://github.com/sveltia/sveltia-cms) at `/admin` — a browser
  writing desk that publishes by committing markdown to this repo.
- Auth: GitHub OAuth via a Cloudflare Worker gateway + Cloudflare Access.
  No passwords anywhere. See `SECURITY.md`.

## Layout

| Path | What it is |
|---|---|
| `src/pages/index.astro` | The homepage — hero, notes, projects, whoami |
| `src/pages/notes/[...slug].astro` | Note article template |
| `src/content/notes/` | The notes themselves (markdown, CMS-managed) |
| `public/admin/` | Sveltia CMS app + `config.yml` |
| `public/_headers` | Security headers (CSP, HSTS, etc.) |
| `README-SETUP.md` | Step-by-step deployment guide |
| `SECURITY.md` | Threat model & OWASP mapping |

## Local dev

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # output in dist/
```

Built and maintained in the open.
