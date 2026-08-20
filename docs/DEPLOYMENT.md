# Deployment and change control

## How production works today

`aatifmulla.me` is a static site on GitHub Pages, fronted by Cloudflare.

```
local edit -> git push origin main -> GitHub Pages builds -> live in 1-3 min
```

There is no build step, no bundler, and no dependencies. `index.html`,
`case-studies.html`, `404.html` and `styles.css` are served as written.

## Current risk (verified 2026-08-10, read-only API queries)

| Control | State |
|---|---|
| `main` branch protection | **None.** `GET /branches/main` returns `"protected": false` |
| Rulesets | **None.** `GET /rulesets` returns `[]` |
| PR required before merge | No |
| Force push to `main` | Allowed |
| `main` deletion | Allowed |
| Required status checks | None |
| Pages source | Branch `main` (implicit `pages-build-deployment` workflow) |
| Deployment approval | None. Every push to `main` publishes automatically |
| Local auth | HTTPS + Git Credential Manager, credentials cached |

**Consequence:** any process on this machine that can run `git push` can change
the public site immediately, with no review, no approval and no rollback gate.
That includes AI coding agents.

## Recommended GitHub settings

Apply in this order. Only you can do this; these are repository settings, not code.

### 1. Protect `main` (highest value)

`Settings -> Branches -> Add branch ruleset` (or classic branch protection).

Target: `main`. Enable:

- **Restrict deletions**
- **Block force pushes**
- **Require a pull request before merging** (approvals: 0 is fine for a solo repo;
  the point is that changes land as reviewable PRs, not silent pushes)
- **Require status checks to pass** -> select `static-checks` once
  `.github/workflows/pr-checks.yml` has run at least once
- **Do not** grant yourself a bypass while testing the setup. Add it back only if
  the friction is genuinely blocking.

Effect: `git push origin main` is rejected. Work must go through a branch and a PR.

### 2. Keep Pages deploying from `main`

Once `main` is protected, "deploy on push to `main`" is safe, because reaching
`main` now requires a PR. Migrating to a custom Actions deploy workflow adds an
environment approval gate but also adds a workflow with `pages: write`
permission, which is a larger attack surface than it removes. Not recommended
unless you want a manual approval step before every publish.

If you do want that gate: `Settings -> Environments -> New environment ->
github-pages -> Required reviewers`. This works with the built-in Pages flow too.

### 3. Optional hardening

- **Require signed commits** on `main` (needs GPG/SSH signing set up locally first).
- **Actions -> General -> Workflow permissions -> Read repository contents**, and
  disable "Allow GitHub Actions to create and approve pull requests".

## Security headers (Cloudflare, not GitHub)

GitHub Pages cannot set response headers. The site currently sends none.
Verified live: no HSTS, CSP, X-Content-Type-Options, X-Frame-Options or
Permissions-Policy.

Add via `Cloudflare -> Rules -> Transform Rules -> Modify Response Header`,
"All incoming requests", **Set static**:

| Header | Value |
|---|---|
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `SAMEORIGIN` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `geolocation=(), camera=(), microphone=(), interest-cohort=()` |

Those four are safe and reversible. Then verify:

```bash
curl -sI https://aatifmulla.me/ | grep -iE 'x-content-type|x-frame|referrer|permissions'
```

**HSTS** has its own toggle: `SSL/TLS -> Edge Certificates -> HSTS`. Start at
`max-age=86400`, with `includeSubDomains` and `preload` OFF. It is the one
setting here that is genuinely hard to undo, and the domain has an unrelated
`n8n` subdomain that `includeSubDomains` would cover.

**CSP** is the highest-value header and the only one that can visibly break the
page, because both pages use inline `<script>` and inline `<style>`. A policy
matching what the site actually loads:

```
default-src 'self'; script-src 'self' 'unsafe-inline' https://static.cloudflareinsights.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data:; connect-src 'self' https://cloudflareinsights.com; frame-ancestors 'self'; base-uri 'self'; form-action 'self'
```

Ship it first as `Content-Security-Policy-Report-Only`, load both pages, confirm
the console shows no violations, then rename the header.

Only three external hosts actually load resources: `fonts.googleapis.com`,
`fonts.gstatic.com`, `static.cloudflareinsights.com`. Every other external domain
in the HTML is a plain outbound link, which CSP fetch directives do not govern.

## Before any change goes out

```bash
python scripts/check_site.py     # structure, links, cache-busters, copy rules, secrets
```

Then bump the cache-buster if you touched CSS or the resume:

- `styles.css?v=NN` in `index.html`, `case-studies.html`, `404.html`
- `resume.pdf?v=YYYY-MMx` in `index.html` and `404.html`

`check_site.py` fails the build if these drift apart.
