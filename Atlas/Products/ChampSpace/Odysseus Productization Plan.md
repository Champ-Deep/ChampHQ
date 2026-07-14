---
tags: [effort, product]
status: planning
created: 2026-07-14
company: "[[Champions Group]]"
source-repo: https://github.com/pewdiepie-archdaemon/odysseus
---

# Champ Agent Workspace: Odysseus Productization Plan

> Fork PewDiePie's Odysseus (self-hosted AI workspace, 81.8k stars, AGPL-3.0), rebrand it into the Champ Suite, and point it at the data team's scraping and enrichment needs, running on totally local models. Internal-only tool.

## 1. What Odysseus Is

A self-hosted AI workspace: chat + agents (tools, MCP, shell, skills, memory), Cookbook (hardware-aware local model recommendations, downloads, serving), Deep Research (multi-step web research with source reading and report generation), bundled SearXNG metasearch, model Compare, documents editor, email/notes/calendar extras. Python (51.6%) + JavaScript, Docker Compose deploy with NVIDIA and AMD GPU variants, native macOS/Windows launchers. Active project: 1,896 commits, no tagged releases, `dev` is default and churns fast, `main` is the curated branch.

Why it fits us: it is essentially a self-hosted, local-model version of the agent workspace we keep gluing together from parts. For the data team it means Deep Research + agents + web search with zero data leaving our infrastructure, which matches the `[s:local-only]` posture we already enforce on internal company data.

## 2. Decisions Locked (2026-07-14)

| Decision | Choice |
|---|---|
| Positioning | Champ-themed name around "agent workspace" |
| Deployment | Hybrid: central GPU server for heavy jobs + per-analyst local installs for sensitive one-offs |
| V1 scope | [[Investor Platform - Cadence\|Cadence]] enrichment pipeline + general scraping workbench for the team |
| License posture | Internal-only. No external users, so AGPL-3.0 network clause never triggers. Modifications stay private. |

## 3. Naming Shortlist

Name must say "agent workspace", not "scanner". [[ChampScan]] stays reserved for a future lighter enrichment tool.

| Candidate | Read | Notes |
|---|---|---|
| **ChampSpace** (recommended) | The workspace where Champ agents live | Cleanest mapping to "self-hosted AI workspace", no collisions in the [[Glossary]] |
| ChampBench | The workbench for agents and scraping flows | Slightly more "tool" than "home" |
| ChampCrew | A crew of agents in one place | Fun, but weaker fit for docs/email/notes modules |

Plan uses **ChampSpace** as the working name until you confirm. One word in the glossary changes if you pick differently.

## 4. License Reality (AGPL-3.0)

Internal use is unrestricted: we can rebrand, modify, and keep everything private as long as only Champions Group people use it. Two hard rules baked into the plan: keep `LICENSE` and `ACKNOWLEDGMENTS.md` intact in the fork (attribution survives the reskin), and never expose the instance to external users, clients, or [[Black & Beige]] sub-clients. If ChampSpace ever becomes sellable, that is a new decision gate: either publish our fork's source or clean-room the core. Documented so it is not relitigated.

## 5. Architecture: Keep, Strip, Add

| Layer | Action | Detail |
|---|---|---|
| Chat + Agents (tools, MCP, shell, skills, memory) | **Keep** | The core value. Agents with shell access = self-serve scraping IDE. |
| Cookbook (model serving) | **Keep** | Hardware-aware local model management is exactly the "totally local" requirement. |
| Deep Research + SearXNG | **Keep + tune** | SearXNG config gets our engine list; Deep Research becomes the account research workhorse. |
| Compare | **Keep** | Use it to benchmark local models on extraction quality before standardizing. |
| Documents editor | **Keep** | Free deliverable surface for research reports. |
| Email (IMAP/SMTP) | **Strip/disable** | We have Inbox Pulse + email triage. Avoid a second inbox surface. |
| Notes, Tasks, Calendar, CalDAV | **Strip/disable** | Celsus + Worksuite own this. Reduce surface area. |
| MCP servers dir | **Add** | Wire [[LakeStream API]] (our own scraper), [[Lake B2B Internal Scraping System]], and a Playwright MCP for JS-heavy targets. |
| Extraction presets | **Add** | Cadence field schemas as agent presets/skills: Jobs (17 fields), News/liquidity (8), Funding (5). |

## 6. Phased Plan

### Phase 0: Fork + Audit (Week 1)

Fork to `Champ-Deep/champspace` via SSH deploy key per standing rule (never PATs). Pin to `main` branch, not `dev`, since upstream has no releases and `dev` churns. Stand up a vanilla instance via `docker compose up` on the target server, default port 7000, confirm the Cookbook serves a model end to end. Audit pass: THREAT_MODEL.md and SECURITY.md review, confirm auth + 2FA on, raw model ports not exposed, decide upstream sync cadence (monthly cherry-pick from `main`, not continuous, to keep rebrand diffs manageable).

### Phase 1: Rebrand (Week 1-2, parallel with Phase 0)

White-label checklist: wordmark and logo swap (`docs/odysseus-wordmark.png` and static assets), theme to Champions Group orange + white via the built-in themes system, rename user-facing strings ("Odysseus" to "ChampSpace") in `static/` and templates, landing page (`docs/index.html`) rebuilt as internal onboarding page, service names in compose files and `odysseus-ui.service`. Keep internal module names untouched to minimize merge conflicts with upstream. LICENSE and ACKNOWLEDGMENTS stay.

### Phase 2: Local Model Stack (Week 2-3)

Hybrid topology, totally local at both tiers:

| Tier | Hardware | Serving | Models (validate with Compare) |
|---|---|---|---|
| Central server | GPU box, 24GB+ VRAM minimum (48GB+ ideal), at JSTARS or a dedicated hosted box we control | Cookbook-managed, GPU compose variant | 30B-class workhorse for Deep Research + agent reasoning; small fast 7-14B model for bulk extraction calls |
| Analyst laptops | Team Macs/PCs as-is | Ollama via native install or Docker CPU/Metal | Quantized 7-14B for sensitive one-off research |

Principles: extraction is the high-volume job and small models handle structured field-pulling well, so route bulk Cadence work to the cheap fast model and reserve the big model for research synthesis. Model choices are deliberately not hard-coded in this plan; the Compare module exists to settle it with our own eval set (50 hand-labeled company enrichment samples from [[Harshil Scraping Intern|Harshil]]'s benchmark work). No OpenRouter here: this stack is the explicit exception to the OpenRouter-always rule because the whole point is local-only.

### Phase 3: Cadence Enrichment Integration (Week 3-5)

The 500K-company enrichment flow becomes the proving ground. Build the pipeline as ChampSpace agent presets: input company batch, agent hits SearXNG + [[LakeStream API]] + Playwright MCP, local model extracts the Jobs/News/Funding field schemas, output lands as CSV/JSON for [[Investor Platform - Cadence|Cadence]] ingestion. Slot ChampSpace directly into Harshil's existing benchmark (Claude/Docker/proxies vs Firecrawl/Apify) as the local-first contender; the Monday 4 PM cadence with [[Subbu]] is the natural review checkpoint. Proxies remain external infrastructure: local models does not mean no proxies, bulk scraping still needs rotation, and that is compatible with the privacy posture since proxies see traffic, not our data.

### Phase 4: Team Rollout + Workbench (Week 5-6)

Accounts for [[Uma]] (data lead), [[Sidhu]] (scraping), [[Harshil Scraping Intern|Harshil]] (intern). One 60-minute onboarding session, then a shared preset library: Cadence enrichment preset, prospect deep-research preset, generic "scrape this list" preset. Guardrails: agents get shell access inside the container only, scraping etiquette defaults (rate limits, robots awareness for non-adversarial targets), and a weekly usage review for the first month. Vault note per preset in `Atlas/Products/ChampSpace/` so knowledge stays in Celsus.

## 7. Holes Shot First

| Risk | Mitigation |
|---|---|
| Odysseus is a research workspace, not a bulk scraper. Agent-driven scraping of 500K companies would be slow and expensive. | Use it as the orchestration + extraction brain on sampled/prioritized batches; keep [[Lake B2B Internal Scraping System]] and Sidhu's pipelines for raw volume. ChampSpace enriches, it does not firehose. |
| Upstream churn, no releases, single charismatic maintainer who may lose interest. | Pin to `main`, monthly sync, keep our diff small (branding + presets + MCPs, minimal core edits). If upstream dies, we already own our fork. |
| Local model extraction quality below Claude/GPT on messy pages. | Compare-module bake-off against the labeled set before committing. If a field group underperforms, tighten prompts/schemas per field rather than reaching for cloud. |
| GPU box is a new single point of failure. | Laptop tier keeps the team functional; compose stack redeploys in minutes; weekly config backup to vault-adjacent storage (not the vault itself, per lean policy). |
| Maintenance owner ambiguity. | Name one: recommend Sidhu owns the deployment, Harshil owns preset quality, review at the Monday 4 PM Cadence sync. |
| Scope creep into email/notes/calendar modules. | Disabled at Phase 1. ChampSpace is agents + research + scraping, full stop. |

## 8. Success Metrics (90 days)

Enrichment cost per 1K companies vs the Firecrawl/Apify benchmark, field-level extraction accuracy vs Harshil's labeled set (target: within 5% of cloud-model baseline on Jobs/News/Funding schemas), weekly active use by all 3 data team members, zero external egress of company data during enrichment runs (spot-audit container network logs), and at least one non-Cadence scraping flow built self-serve by the team without Deep's involvement.

## 9. Immediate Next Actions

1. Deep confirms the name (ChampSpace / ChampBench / ChampCrew).
2. Decide the central GPU box: existing hardware at JSTARS vs procuring one. This is the only spend decision in the plan.
3. Fork the repo (SSH deploy key) and stand up vanilla on whatever hardware exists today, even CPU-only, to validate fit before GPU spend.
4. Pull 50 labeled samples from Harshil's benchmark set for the model bake-off.
