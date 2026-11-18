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

## 4. From an empty WordPress to a live site (step by step)
"You" = Alex. Everything else is Claude. Each step ends with what you type to hand over. Files land in `/mnt/project-files/sites/<site>/`.

**Step 1. Create the site at the host (you, ~10 min)**
- SiteGround: Site Tools > WordPress > Install & Manage > new WordPress. Cloudways: Add Application > WordPress. Any host works if it runs current WordPress with HTTPS.
- Use a staging or temporary domain first; the real domain comes at the end.
- Note the admin username. Keep the host's cache plugin installed and active (SiteGround: Speed Optimizer; Cloudways: Breeze), so Claude can purge cache.

**Step 2. Connect Claude to the site (you, ~5 min)**
- wp-admin > Users > Profile > Application Passwords > name "Claude" > Add > copy the code.
- Project settings > Environment > gear: add the domain to **Allowed domains**, then **Add secret**: type Basic, the domain only, path prefix `/wp-json/`, your username + the code.
- Type: **"New site <url> is ready"**. wp-kit checks access, versions, routes and cache, writes `profile.env`, and tests with a throwaway draft.

**Step 3. Install the stack with Blueprint (you, ~10 min, in the browser)**
- Get the zips: ask **"Build the Blueprint and Suite zips"** (they land in `/mnt/project-files/plugin-zips/`).
- New site: Plugins > Add New > Upload > Blueprint zip > Activate.
- Blueprint source (alexg241): TwoEleven > Blueprint > Blueprint source > **New setup code** (one use, 60 minutes).
- New site: TwoEleven > Blueprint > New site > paste the code > **Check what it will do** > Apply. This installs Etch, the Etch theme, ACSS and SCF, and copies ACSS settings, Etch settings, components and field groups.
- On the result screen: activate ACSS (one click) and paste the Etch license (Copy button; Etch can't be automated yet).
- Type: **"Stack done on <site>"**. Claude checks versions and that Etch, ACSS and SCF answer.

**Step 4. Our plugins with Suite (you upload once; Claude does the rest)**
- New site: Plugins > Add New > Upload > Suite zip > Activate.
- Type: **"Set up our plugins on <site>"**. Claude writes `site.json` (tracking, languages, accessibility, watcher, speed), shows a dry run, applies it after your OK, and reads it back. Keys go through Vault only.

**Step 5. Plan the site (Claude, you approve)**
- Type: **"Use the web kit to plan <site>"**, plus any brief, brand or old site link.
- Claude writes `brief.md`, `brand.md` and `blueprint.md`: pages, content types and fields, media collections, components.
- After your OK: custom post types and fields (SCF), media collections, compressed images with alt text, and an empty draft per page.

**Step 6. Build each page (Claude, two gates for you)**
| | You type | What happens | You step in? |
|---|---|---|---|
| a | "Build the <page> page" | copywriter + SEO write `copy.md`. | **Gate:** approve the copy |
| b | (continues) | design-director writes `design-spec.yaml`; it must pass the spec check. | Approve the design if asked |
| c | (continues) | etch-kit builds the draft through its gates (section 5). | No |
| d | (continues) | Review at 375 and 1440 width, then audits (SEO, content, accessibility, tracking). Each fix gets an owner. | Read the one-line report |
| e | (continues) | Lingua translates once the copy is approved, if the site is bilingual. | Approve translations if asked |

**Step 7. Go live (you, with Claude)**
- Host: point the real domain at the site and turn on SSL (DNS is yours).
- Type: **"Go live"**. Claude publishes, turns search indexing on, purges cache, and checks every page at the real domain.
- After launch: Pulse watches updates and uptime; an Etch or ACSS update triggers a web-lab re-test.

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
Section 4, steps 1 and 2. Without the Allowed domains entry the site returns 403.
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
