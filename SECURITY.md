# Security design — pursuix v2

You asked for a password and defenses against brute force. This document explains why
the design gives you something stronger than a password, and maps the whole system
against the OWASP Top 10. Since you're studying blue team: this file *is* a small
threat-modeling exercise. Steal the method.

## The core decision: no custom auth, anywhere

The most dangerous thing a small project can do is invent its own login. A password
check in static-site JavaScript is decorative — the "secret" ships to every visitor in
view-source. A server-side password system means you now own hashing, salting, rate
limiting, lockouts, session tokens, and reset flows, and a mistake in any one of them
is a breach. So this architecture has **zero passwords of its own**:

- **Authentication** is delegated to GitHub OAuth. Logging into the writing desk *is*
  logging into GitHub — which also satisfies "link it to my GitHub profile."
- **Authorization** is your repo's permission model: the CMS publishes by committing,
  and GitHub only accepts commits from accounts with write access. That's you.
- **Perimeter** (optional but recommended, step 6 of setup): Cloudflare Access demands
  a one-time email code before the admin page even loads.

An attacker must now defeat GitHub's account security (with your 2FA enabled) *and*
Cloudflare Access tied to your inbox — two of the most heavily defended login systems
on the internet — instead of one password form written by us on a weekend.

## Brute force & friends, specifically

| Attack | Why it fails here |
|---|---|
| Password brute-forcing | No password exists to guess. GitHub logins are rate-limited, monitored, and 2FA-protected by GitHub's own systems. |
| Credential stuffing (reused passwords from other breaches) | Same — and 2FA makes a leaked GitHub password alone useless. |
| Bots hammering the admin URL | Cloudflare Access returns them a login wall at the edge; your page never even renders. Attempts are logged in Zero Trust → Logs. |
| Stolen OAuth client secret | Lives only in the Worker's encrypted secret store — never in the repo, never in the browser. Rotate it in one minute if ever in doubt. |
| Someone else using your auth Worker | `ALLOWED_DOMAINS` restricts the OAuth flow to pursuix.com origins. |
| Session theft via XSS | Mitigated by the CSP in `_headers` (below) and by Astro's default output escaping. |

## OWASP Top 10 mapping

- **A01 Broken Access Control** — publishing rights = GitHub repo write access; the
  admin UI has no privileges of its own. Cloudflare Access adds a second, independent
  allow-list (your email only).
- **A02 Cryptographic Failures** — TLS everywhere via Cloudflare; HSTS with
  `includeSubDomains` in `_headers` prevents downgrade to HTTP. No secrets at rest in
  the repo or the browser beyond a session-scoped token.
- **A03 Injection** — no database, no server-side code of ours, so no SQL/command
  injection surface. The XSS variant: Astro escapes all template output by default and
  markdown is compiled at build time (raw HTML in markdown is not rendered by
  default — don't change that). The CSP is the safety net: scripts only from `self`
  (plus unpkg on `/admin/*` only), no framing, no external form posts.
- **A04 Insecure Design** — the design principle is minimal attack surface: a static
  site (nothing to exploit at runtime), a single stateless Worker, identity delegated
  to hardened providers.
- **A05 Security Misconfiguration** — `_headers` sets nosniff, frame-denial, referrer
  policy, permissions policy, HSTS, CSP. The admin is `noindex`. Verify after each big
  change at securityheaders.com. Cloudflare's free WAF/Bot Fight Mode can be toggled on
  for extra edge filtering.
- **A06 Vulnerable & Outdated Components** — the surface is small (Astro at build time,
  Sveltia in the admin). Enable GitHub **Dependabot alerts** on the repo; consider
  pinning the Sveltia CDN version in `public/admin/index.html` and bumping it
  deliberately. Run `npm audit` when you update.
- **A07 Identification & Authentication Failures** — the headline decision above:
  delegate, don't build. Your action item: GitHub 2FA (setup step 0), and prefer
  passkeys where offered.
- **A08 Software & Data Integrity Failures** — every publish is a signed-in commit;
  the repo history is your tamper-evident audit log. The build runs from the repo
  only. CDN supply-chain risk on the admin script is real but bounded to `/admin/*`
  by the CSP; pin the version if you want it tighter.
- **A09 Logging & Monitoring Failures** — you have three logs for free: GitHub's
  security log (logins, OAuth grants), the repo commit history (every content change,
  by whom, when), and Cloudflare Access logs (every attempt to reach the admin).
  A good blue-team habit: skim all three monthly.
- **A10 SSRF** — no server that fetches user-supplied URLs exists in this system.

## Residual risks — stated honestly

1. **Your GitHub account is the crown jewel.** Its compromise = site compromise.
   2FA/passkey is non-negotiable; review authorized OAuth apps quarterly.
2. **CDN supply chain** for the Sveltia script (see A06/A08). Pinning trades
   auto-updates for stability — reasonable either way at this scale.
3. **Your email account** guards Cloudflare Access codes and GitHub recovery.
   Harden it with the same 2FA discipline.
4. **This site is public content by design.** Never paste secrets, tokens, or
   internal details into a note — the repo history remembers forever.

## Quarterly checklist

- [ ] GitHub → Settings → Security log: anything unfamiliar?
- [ ] GitHub → Applications: still only the OAuth apps you recognize?
- [ ] Cloudflare Zero Trust → Logs: any repeated denied attempts on the admin?
- [ ] `npm update` locally, rebuild, and check Dependabot alerts.
- [ ] Re-test headers at securityheaders.com after any config change.
