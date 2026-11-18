# How agentic can the build flow get? (2026-11-18)

Short answer: **almost everything up to a reviewed staging site can run unattended. Go-live, money, licenses and client-facing claims should stay human.** Fully agentic end to end is possible technically but is a bad idea for client sites: one wrong claim, price or legal line published unattended costs more than the minutes saved.

Facts this rests on: `skills-audit/blueprint-site-probe.md` (what REST can do), plugin code on `twoeleven-studio` main, PR #58 (REST scripts + gates). Items marked *unverified* need a check before we build on them.

## 1. Where Alex's hands are needed today
| Step | Can it be automated? | How | Verdict |
|---|---|---|---|
| Create the site at the host | **Cloudways: yes** (public REST API creates and clones apps; API key in Vault). **SiteGround: no public API** for shared/cloud accounts (*unverified for their newer plans*). | Cloudways: `wp-kit` host adapter. SiteGround: keep manual, or use a Cloudways "golden" clone (below). | Automate on Cloudways; manual on SiteGround |
| Install stack (Etch, ACSS, SCF, design system) | Yes | **Golden template site**: one fully set up site (Blueprint, Suite, Etch, ACSS, SCF, Claude user) that the host clones. Removes Blueprint apply and zip uploads per site. Second route: a Blueprint `apply` ability so the cloud can run it. | Biggest single win |
| Upload first plugin zip | Not over WordPress REST (it only installs wordpress.org slugs) | Golden clone carries it; or the host's file manager/SSH | Gone with golden clone |
| Application Password + cloud secret | Half | Golden clone already has the Claude user; a fresh password per site still has to become a cloud secret. Whether the environment can be updated by an agent is *unverified* (a remote environment API exists; secret writes not tested). Until then Alex pastes it, 2 min. | Keep human for now; check the API |
| Allowed domains | Same as above | Use one wildcard-able staging domain per host (e.g. all sandboxes under one host domain) if wildcards are accepted (*unverified*) | Check |
| Etch license | Not yet | No documented constant or command; read Etch's code for an activation route. ACSS already has `ACSS_LICENSE_KEY`. | Automate after code read |
| ACSS settings change | Not yet | Small Suite route: ACSS settings import + CSS rebuild. WPCodeBox only as last resort. | Build it (plugin PR) |
| Copy, design direction approval | Should not be automated | Gate | **Human by design** |
| Go live, DNS, delete, send, buy | Should not be automated | Gate | **Human by design** |

## 2. Quality risks of unattended builds
| Risk | Today | Add |
|---|---|---|
| Etch rule violations | Strict lint before write, read-back after (PR #58) | Make every judgement rule that can be checked a lint rule; add each violation from real builds as a test case |
| Wiped or stripped pages (REST traps) | Gate 2 + snapshots | Daily snapshot of all pages to project files; one-command site-wide restore |
| Design quality ("looks wrong" but passes lint) | Human eye | Screenshot at 375/1440 compared with the approved mockup (pixel diff + a written design review); first page per site always reviewed by Alex, later pages only when the diff is large |
| Copy accuracy and claims | content-review compliance pass | Every number, price or claim must cite a source in `copy.md`; uncited = blocked |
| Etch/ACSS updates break builds | Manual | Pulse sees the update; web-lab runs fixtures + four section types on the sandbox; builds pause until green |
| Silent regressions after launch | None | Weekly screenshot diff of live pages; Pulse uptime |

## 3. Platform limits
| Limit | Mitigation |
|---|---|
| Cloud network allowlist per domain | Add the domain at onboarding (one line); check wildcards |
| Some threads can't reach a site (this one couldn't reach alexg241) | Put the site domain in every environment used for builds; the probe step reports it at once |
| No browser tab for builder-only features (round-trip gate, `etch.assets` in the builder) | Headless Chromium with a dedicated low-risk "claude-bot" login, its password as a second secret for `/wp-login.php` and `/wp-admin/` (web-lab test). Most media work already runs over REST |
| Cache purge only while SG Optimizer is active | Keep the host cache plugin active on every build; profile stores the purge route |
| Long builds burn context and usage | One page per thread; hand-off files per step; Sonnet for audits/copy, Opus for build/design; resume from files, not chat history |
| Host captchas on cloud IPs | Verify on staging domains; live checks from Pulse |

## 4. Recommended autonomy level
**Unattended:** onboarding probe, site plan draft, CPTs and fields, media (compress, alt text, collections), page builds on staging, all gates, audits and fixes, translations, web-lab re-tests, PR merges.
**Needs Alex:** site plan approval, copy and claims, first design per site, go-live, DNS, licenses and purchases, deletes on live, anything sent to a client.
That is roughly 90% of the hours automated, with 4 short gates per site.

## 5. Roadmap (cheapest wins first)
1. **This week, agent only:** sandbox copy of alexg241; first real run of the PR #58 scripts; build the Blueprint and Suite zips; screenshot baseline + diff in etch-kit's review; Pulse to web-lab trigger. *Needs Alex:* create the sandbox (5 min).
2. **Next:** golden template site + clone procedure (Cloudways API, SiteGround manual copy). Cuts per-site hands-on from ~30 to ~5 minutes.
3. **Plugin PRs:** Blueprint `apply` ability; Suite `acss` settings route; ACSS key via constant; Blueprint into the Suite catalog.
4. **Checks:** environment API for secrets/allowed domains; wildcard domains; Etch license code read; headless bot login for builder-only features.
5. **Cloudways adapter** in wp-kit: create or clone the app from the agent.
