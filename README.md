# docs/

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

Mintlify hosts the site and `apps/website` reverse-proxies `/help` to it, the
same way it proxies `/blog`, so the URL bar stays on `pernee.com`. Mintlify is
not deployed by Vercel and has no Vercel project.

Two things have to agree, and they are the whole setup:

1. **Mintlify knows it lives under `/help`.** In the dashboard, Custom domain →
   "Host at" is set to `pernee.com` + base path `help`. Mintlify then rebuilds
   the docs with `basePath: "/help"`, so every link and asset URL it generates
   already carries the prefix.
2. **The proxy forwards that prefix rather than stripping it** —
   `/help/:path*` → `${HELP_ORIGIN}/help/:path*`. See
   `apps/website/next.config.ts`.

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

## What is deliberately not published

These files live here because this is where the repo keeps them, **not**
because they belong on a docs site. They are internal and are kept out of
`docs.json`:

- `agents/veepee.md`, `agents/princee.md`, `agents/pernesh.md` — the canonical
  briefs for the engineering roles. They name the Jira instance and board, the
  compliance posture, internal file layouts, and which class of defect is
  treated as most severe. That last one is a map for anyone looking for a way
  in.
- `tech-knowledge-base-plan.md` — an internal proposal, including an inventory
  of internal systems and their status.

Before adding a page to the navigation, ask whether it is written for someone
outside the company. If it is addressed to an agent or a teammate, it is not
documentation — it is an internal brief that happens to be Markdown.

## Note on repository visibility

`Pernee-code/pernee-ai` is a **public** repository, so everything above is
already world-readable on GitHub regardless of what Mintlify serves. Keeping a
file out of the navigation limits how it is presented and indexed; it does not
make it private.
