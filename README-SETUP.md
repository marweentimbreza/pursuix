# pursuix v2 — setup guide

From zero to a live site with a private writing desk at `myspace.pursuix.com`.
Total time: roughly 30–45 minutes. Do the steps in order — each one feeds the next.

There are exactly **three placeholders** to edit: your repo name and auth Worker URL
in `public/admin/config.yml` (steps 1 and 4), and your Web3Forms access key in
`src/pages/contact.astro` (step 8).

---

## Step 0 — Lock your GitHub account first

Your GitHub account is now the key to your website, so harden it before anything else.
Go to GitHub → Settings → Password and authentication → enable **two-factor
authentication** (an authenticator app or passkey, not SMS). This is the single most
important security step in this whole guide.

## Step 1 — Push this project to GitHub

Replace the contents of your existing site repo with this project (or create a fresh
repo — either works, just be consistent about the name).

```bash
cd pursuix-site
git init
git add .
git commit -m "pursuix v2 — terminal redesign + writing desk"
git branch -M main
git remote add origin https://github.com/marweentimbreza/YOUR-REPO-NAME.git
git push -u origin main --force   # --force only if overwriting the old repo
```

Then open `public/admin/config.yml` and set `repo:` to that same `owner/name`. Commit
and push again.

## Step 2 — Point Cloudflare Pages at the repo

In the Cloudflare dashboard → Workers & Pages → your Pages project (or Create → Pages
→ Connect to Git if starting fresh):

- **Production branch:** `main`
- **Framework preset:** Astro
- **Build command:** `npm run build`
- **Output directory:** `dist`

Save and deploy. Your existing `pursuix.com` custom domain keeps working. From now on,
every commit to `main` automatically rebuilds the site — this is the pipe the CMS uses.

## Step 3 — Create the GitHub OAuth App

GitHub → Settings → Developer settings → OAuth Apps → **New OAuth App**:

- **Application name:** `pursuix writing desk`
- **Homepage URL:** `https://pursuix.com`
- **Authorization callback URL:** leave a placeholder for now (e.g. `https://example.com`);
  you'll fill in the real one after step 4.

Note the **Client ID** and generate a **Client Secret**. The secret is shown once —
copy it somewhere safe. It must never be committed to the repo.

## Step 4 — Deploy the auth gateway (a tiny Cloudflare Worker)

Sveltia CMS needs a small server-side helper to complete GitHub's OAuth handshake,
because the client secret can't live in a static page. Sveltia provides one:

1. Open `https://github.com/sveltia/sveltia-cms-auth` and use its **Deploy to
   Cloudflare Workers** button (or `wrangler deploy` if you prefer the CLI).
2. In the Worker's Settings → Variables and Secrets, add:
   - `GITHUB_CLIENT_ID` — from step 3
   - `GITHUB_CLIENT_SECRET` — from step 3 (add as a **Secret**, not plain text)
   - `ALLOWED_DOMAINS` — `pursuix.com,myspace.pursuix.com`
     (this stops other websites from borrowing your auth gateway)
3. Note the Worker URL, e.g. `https://sveltia-cms-auth.yourname.workers.dev`.
4. Go back to the GitHub OAuth App and set the callback URL to
   `https://<your-worker-url>/callback`.
5. Set `base_url:` in `public/admin/config.yml` to the Worker URL. Commit and push.

The Worker holds the only secret in the whole system, stores nothing, and simply
exchanges OAuth codes for tokens that live in *your* browser session.

## Step 5 — Wire up the subdomain

Cloudflare dashboard → your Pages project → **Custom domains** → Add
`myspace.pursuix.com`. Since your DNS is already on Cloudflare, the CNAME and TLS
certificate are created automatically.

Then make the subdomain land on the writing desk: Rules → **Redirect Rules** → create:

- **If** Hostname equals `myspace.pursuix.com` AND URI Path equals `/`
- **Then** static redirect to `https://myspace.pursuix.com/admin/`, status 302

Result: visiting `myspace.pursuix.com` drops you at the editor.

## Step 6 — The second gate: Cloudflare Access (recommended)

This is the "password" you asked for — done properly. Free for up to 50 users.

Cloudflare dashboard → Zero Trust → Access → Applications → **Add an application**
→ Self-hosted:

- **Application domain:** `myspace.pursuix.com` (path: leave empty to cover all)
  — add a second domain entry for `pursuix.com/admin` too.
- **Policy:** Allow → Include → Emails → *your email address only*.
- Session duration: 24h is a sane default.

Now the admin URL won't even load without a one-time code sent to your email.
Bots, scanners, and brute-forcers hit Cloudflare's wall before your page exists
to them. Access also logs every login attempt (Zero Trust → Logs).

## Step 7 — Write something

Visit `https://myspace.pursuix.com` → pass the Access check → **Sign in with
GitHub** → authorize once. You'll see "Notes & Research" with your network-traffic
draft in it. Write, hit **Publish**, and watch: Sveltia commits the markdown to
`src/content/notes/`, Cloudflare Pages rebuilds, and the note is live on
`pursuix.com` about a minute later. No manual uploads, ever again — and every post
is a commit in your repo's history.

## Step 8 — Wire the contact form

The contact page uses Web3Forms — the same service your original site was set up
for. Go to https://web3forms.com, enter the email you want messages delivered to,
and you'll instantly receive an access key. Paste it into `src/pages/contact.astro`
replacing `YOUR-WEB3FORMS-ACCESS-KEY-HERE`, commit, push. Submissions now land in
your inbox; the form includes a honeypot field that silently filters spam bots.

---

## Site map

| URL | Page |
|---|---|
| `/` | Home — hero, sitemap, latest notes |
| `/blog/` | All notes & research (posts at `/blog/<slug>/`) |
| `/projects/` | Projects (honest empty state + roadmap) |
| `/about/` | Whoami, certs, links |
| `/contact/` | Say-hello form |

`/blogs` and `/project` redirect to the right places automatically (see
`public/_redirects`), so old habits and typos still land.

---

## Day-2 notes

- **Adding images:** use the editor's image button — files land in `public/uploads/`.
- **Drafts:** the `draft` toggle shows a "draft · updating" badge on the site rather
  than hiding the note, matching your learning-in-the-open ethos. If you'd rather
  hide drafts, filter them in `src/pages/index.astro`.
- **Local preview:** `npm install`, then `npm run dev` → http://localhost:4321
- **Read `SECURITY.md`** for the threat model and what to check quarterly.
