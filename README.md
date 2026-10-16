# Pernee help centre (`Pernee-code/docs`)

**This repo is the only source of the help centre (2 Sep 2026).** Mintlify's
git source points here — `Pernee-code/docs`, deploy branch `main`, content
directory empty. The copy that lived at `docs/` in the `pernee-ai` monorepo,
which Mintlify built from until then, is removed (pernee-ai #282); do not
recreate it. A change is live once it is on `main` here and Mintlify has
rebuilt — the origin at `https://pernee.mintlifysite.com/help` shows what was
last built, and `pernee.com/help` proxies it.


Mintlify content root. `docs.json` is the site config, and the content
directory in Mintlify's dashboard is empty: this repo's root is the site.

## How this reaches pernee.com/help

Mintlify hosts the site and the marketing site
([`Pernee-code/website`](https://github.com/Pernee-code/website)) reverse-proxies
`/help` to it, the same way it proxies `/blog`, so the URL bar stays on `pernee.com`. Mintlify is
not deployed by Vercel and has no Vercel project.

Two things have to agree, and they are the whole setup:

1. **Mintlify knows it lives under `/help`.** In the dashboard, Custom domain →
   "Host at" is set to `pernee.com` + base path `help`. Mintlify then rebuilds
   the docs with `basePath: "/help"`, so every link and asset URL it generates
   already carries the prefix.
2. **The proxy forwards that prefix rather than stripping it** —
   `/help/:path*` → `${HELP_ORIGIN}/help/:path*`. See `next.config.ts`
   in the website repo.

**The origin host is `pernee.mintlifysite.com`, and this is the part that bit us —
twice.** Mintlify serves a deployment from several hostnames and only
`[subdomain].mintlifysite.com` applies the "Host at" base path. Until 2 Sep 2026
the host was `pernee.mintlify.site`; Mintlify retired that domain without
notice, it stopped resolving, and `/help` answered 502 until the proxy was
repointed. If `/help` 502s again, check Mintlify's reverse-proxy guide for the
current host before touching anything else:

| Request | Result |
|---|---|
| `pernee.mintlifysite.com/help` | 200 — the docs, links prefixed `/help/*` |
| `pernee.mintlifysite.com/clients` | 404 (correct — it is at `/help/clients`) |
| `pernee.mintlify.app/` | the docs at the **root**, un-prefixed build |
| `pernee.mintlify.app/help` | 404 |

`.mintlify.app` is the host this repo pointed at for three attempts. It returns
a real, styled page, which is why it looked close to working — but it is the
un-prefixed build, so every link on it leads off `/help`, and serving it at the
`/help` URL made Mintlify's client-side router throw "Error 500 — an unexpected
error occurred". If `/help` breaks again, check the origin host first.

## What is published

Only what is listed in `docs.json`'s navigation. Mintlify does not publish a
file merely for sitting in this directory — but it does parse every `.md` and
`.mdx` under the content root, so an unlisted file can still break the build.

## Internal material does not belong here

Internal briefs (agent role briefs, internal plans) used to sit beside the docs
in the old monorepo and were kept out of `docs.json`. They are not in this
repo, and should not be added to it. Before adding a page, ask whether it is
written for someone outside the company. If it is addressed to an agent or a
teammate, it belongs in a private repo, not here.

## Analytics

`docs.json` → `integrations.ga4` sends page views to the same **GA4** property
as pernee.com and the blog: `549432653` (`G-1CVP1TB4JM`). Mintlify does not
send from preview links.

Unlike the website and the blog, Mintlify's integration has no internal-traffic
guard. The team's own visits to `/help`, and any direct visits to the
`mintlifysite.com` origin, are counted. In GA4, filter on hostname
`pernee.com` to see real `/help` traffic.

## Note on repository visibility

**This repository (`Pernee-code/docs`) is public.** Everything in it is
world-readable on GitHub whether or not Mintlify publishes it. Keeping a file
out of the navigation limits how it is presented and indexed; it does not make
it private.
