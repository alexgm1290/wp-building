# WP + Etch build playbook

Written 2026-11-18 so future Alex (and any Claude thread) can pick this up cold.
Status words below: **live** = saved and usable now, **planned** = agreed, not built yet.

## 1. The goal
An agent builds and runs a whole WordPress site without you at the Mac. It controls WordPress (pages, media, custom post types, plugins), Etch (pages, classes, components, loops, media) and ACSS (classes and settings), and it brings in our design, copy, SEO and audit skills to build pages from a site blueprint. You only step in at the gates: approving copy and design, and saying "go live".

## 2. The skills
Think of it like a building crew: one foreman, two tradespeople, a test bench, two reference books and the specialists.

| Skill | Role | Status |
|---|---|---|
| **web-kit** | The foreman. Plans the site (blueprint), runs every step in order, stops at your gates. Start here. | planned rewrite (today it is the old router) |
| **wp-kit** | Does anything in WordPress: pages, posts, media, CPTs/fields, plugins, our plugin suite, onboarding a new site. | live |
| **etch-kit** | Builds and edits Etch pages, classes, components, loops. Enforces Etch best practices with gates. | live |
| **web-lab** | Test bench. Proves a new feature or an Etch/ACSS update on the sandbox before real builds use it. | live |
| **etch-expert** | Rule book for Etch. The one list of rules is `etch-expert/reference/guardrails.md`. | live (update to Etch 1.8 planned) |
| **acss-expert** | Rule book for ACSS names, tokens and settings. | live (settings mode planned) |
| **design-director** | Visual direction, the design spec, and the review at phone and desktop width. Absorbs art-director. | live (merge planned) |
| **copywriter** | All words on the page. | live |
| **seo-geo-auditor** | Titles, meta, headings, schema; checked before writing. | live |
| Other auditors | site-content-auditor, tracking-auditor, security-auditor, accessibility: the final audit. | live |

## 3. The plugins (ours; all merged to main in twoeleven-studio, 2026-11-18)
| Plugin | What it does in the build |
|---|---|
| **Blueprint** | Turns a blank WordPress into our stack: installs Etch, the Etch theme, ACSS and SCF, and copies ACSS settings, Etch settings, components and field groups from the blueprint site. Has Undo. |
| **Suite** | Installs and configures Signal, Lingua, Clearway, Pulse and Pace from one file per site (`site.json`), over the API. Keys live in Vault, never in the file. |
| **Signal** | Tracking IDs and cookie consent. |
| **Lingua** | Translation (English/Spanish). |
| **Clearway** | Accessibility fixes and scans. |
| **Pace** | Performance advice. It does not compress images; the agent compresses them before upload. |
| **Pulse** | Watches versions, updates and security. An Etch or ACSS update triggers a web-lab re-test. |

## 4. Build a site from blank WordPress
Each step writes one file into `/mnt/project-files/sites/<site>/`, so any thread can pick up where the last stopped.

| # | You type | What happens | You step in? |
|---|---|---|---|
| 0 | "Add a new site: <url>" | wp-kit walks you through the password and secret (section 6), then reads the site and writes its profile. | Yes, once: create the password and secret |
| 1 | "Use the web kit to plan <site>" | web-kit writes `blueprint.md`: pages, CPTs and fields, ACSS settings, components, media collections. | Approve the blueprint |
| 2 | (continues) | Blueprint installs the stack; Suite applies `site.json`. Both run a dry run first and show what will change. | Upload plugin zips if missing |
| 3 | "Build the <page> page" | `brief.md`, then copywriter and SEO write `copy.md`. | **Gate:** approve the copy |
| 4 | (continues) | design-director writes `design-spec.yaml`; it must pass the spec check. | Approve the design if asked |
| 5 | (continues) | etch-kit builds a **draft** on staging/sandbox through its gates (section 5). | No |
| 6 | (continues) | Review at 375 and 1440 width, then audits (SEO, content, accessibility, tracking). Each fix gets an owner. | Read the one-line report |
| 7 | "Go live" | Publish, purge cache, check the live page. | **Gate:** only your typed words |

## 5. The rules
- **Etch best practices are enforced, not suggested.** etch-kit has gates a build cannot skip:
  - Before writing: a strict check. New pages must have zero violations; edits may not add any. It blocks hand-written HTML, inline styles, plain images, unknown classes and ACSS names that don't exist on that site.
  - After writing: what WordPress stored must equal what was sent (catches a wiped page or stripped scripts).
  - Before sign-off: a review against the rest of the rules at phone and desktop width. Any failure = not done.
  - New rules go into `etch-expert/reference/guardrails.md` and, when a script can check them, into the lint.
- **Path order:** REST API first, then WordPress Abilities, then WP-CLI (needs the Mac), then WPCodeBox only as the last resort.
- **Staging or sandbox first.** Live, publish, delete and DNS only on your typed words.
- **PRs:** Claude merges its own PRs when ready (no conflicts, checks green).
- **Per-site folders:** hostnames, page IDs, colours and fonts live in `sites/<site>/`, never inside a skill, so the skills work on any client site.
- **Secrets** only in the cloud environment or Vault. Never in chat, files or memory.

## 6. Set up a new site (one time per site)
1. In the site's wp-admin: Users > Profile > Application Passwords > name it "Claude" > Add > copy the code.
2. In this project: Project settings > Environment > gear > Add secret. Type Basic, allowed website = the site's domain only, path prefix `/wp-json/`, username + that code.
3. Same screen: add the domain to **Allowed domains** (without this the site returns 403).
4. Tell Claude "done". wp-kit checks the role, versions, available routes and cache, writes `sites/<site>/profile.env`, and tests with a throwaway draft.
Test sites: **alexg241.sg-host.com** is the blueprint source (read-only for tests). Make a SiteGround staging copy of it as the **sandbox** for web-lab.

## 7. Still open (as of 2026-11-18)
- **ACSS settings:** no API to read/write them. Fix planned: a small Suite route for ACSS settings + CSS rebuild. Until then: WP-CLI on the Mac.
- **Cache purge:** the SiteGround purge route only exists while the SG Optimizer plugin is active. Keep it active on built sites.
- **Blueprint apply** only runs from WP-CLI or the admin screen; a cloud route is planned.
- **Builder round-trip** (open in Etch, save, compare) still needs a browser; web-lab will test a cloud login for it.
- **Sandbox site** not created yet. The new scripts have not run against a real site yet.
- **To do:** rewrite web-kit; update etch-expert to Etch 1.8; ACSS settings mode; merge art-director into design-director; fix old skill names in the auditors.

## 8. Where things live
- Plans and research: `/mnt/project-files/skills-audit/` (target-architecture.md is the full plan; blueprint-site-probe.md lists every proven API route).
- Skill drafts: `/mnt/project-files/skills-drafts/`.
- Scripts, fixtures, rules: repo `alexgm1290/twoeleven-studio`, folder `skills/` (etch-page-editor/scripts, etch-builder, etch-expert/reference, acss-expert).
- Plugin specs: same repo, `plugins/*/SPEC.md`.
- This playbook: `/mnt/project-files/playbook/` and repo `alexgm1290/wp-building`.
