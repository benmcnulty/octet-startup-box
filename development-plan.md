# Octet OIS — Startup in a Box: Website + Prototype

## Context

Octet OIS is announcing its first named beta product: **Startup in a Box** — a locally-hosted organizational intelligence environment (spec: `octet-startup-box-spec.md` v0.7). This plan covers two sequential workstreams: (1) adding a product page and landing page update to the live octetois.com website, then (2) building the full prototype in the octet-startup-box repository.

**Repos:**

- `~/dev/octetois/` — live website (Cloudflare Workers + static assets)
- `~/dev/octet-startup-box/` — new prototype (zero-dep Node.js + vanilla JS)

---

## Workstream 1 — Website Integration

### W1.0: Website Audit Findings

**Routing**: Static HTML auto-served from `public/` via `env.ASSETS.fetch()`. The `html_handling: "auto-trailing-slash"` setting means `public/startup-box.html` will serve at both `/startup-box` and `/startup-box/`. No worker changes needed.

**Design system** (`public/styles.css`, 1344 lines): All tokens present — `--bg-*`, `--text-*`, `--accent-1/2/3`, `--space-*`, `--text-*` (typography), `.btn-primary`, `.btn-secondary`, `.section-inner`, `.section-heading`, `.section-lead`, `.inference-card`, `.tool-card`, `.card-icon-wrap`, `.card-title`, `.card-text`. oklch wide-gamut overrides via `@supports`.

**Navigation**: No `<nav>` exists in header. Only: `.logo` (left) + `.header-controls` (share + theme toggle, right). A `<nav>` must be inserted between them.

**Email capture**: Waitlist form POSTs `{ email }` to `/api/waitlist`. D1 table has `source DEFAULT 'landing'` column. Handler in `src/api/waitlist.js` does NOT currently pass source from the request body — needs a one-line fix.

**Header HTML** (index.html:348-728): `.header-inner` uses `display: flex; justify-content: space-between`.

### W1.1: New Page — `/startup-box`

**Create**: `public/startup-box.html`

Copy the full document shell from `index.html` (head, bg-canvas, floating-hex-cluster, header, footer, app.js). Customize OG meta for this page. Add `<link rel="canonical" href="https://octetois.com/startup-box">`.

**Seven content sections** (all using existing component patterns):

| Section | Pattern | Content Source |
| --------- | --------- | --------------- |
| Hero | `.hero` + circuit SVG | Spec §1: "Startup in a Box" + value prop + Beta badge + In Development indicator + GitHub link |
| What it is | `.section-inner` | §1 + §2: founder's operating system, self-sustaining loop, local inference, autonomous personas |
| How it works | `.inference-grid` (6 cards) | §4: all 6 phases presented for non-technical founders |
| Key capabilities | `.toolchain-grid` (5 cards) | §12 Octet, §11 Memory, §14-15 Meetings, §19 Attachment, §21 Inference |
| Built on | `.section-inner` with badge row | §2 + §25: zero deps, pure web, Ollama, MIT license |
| Get involved | `.waitlist-form` (reuse) | Waitlist with `source: 'startup-box'` |
| Links | `.section-inner` | GitHub repo CTA + raw spec file link |

### W1.2: Landing Page Update

**Modify**: `public/index.html`

Insert a product highlight section **after the marquee** (line 837) and **before the mission section** (line 840).

```html
<section class="product-highlight" id="startup-box">
  <div class="section-inner">
    <span class="inference-badge inference-badge--active">First Beta</span>
    <h2 class="section-heading">Startup in a Box</h2>
    <p class="section-lead">...</p>
    <div class="hero-ctas">
      <a href="/startup-box" class="btn btn-primary">Learn More</a>
      <a href="https://github.com/benmcnulty/octet-startup-box" class="btn btn-secondary">View on GitHub</a>
    </div>
  </div>
</section>
```

**CSS**: `.product-highlight` — constrained width, accent-1 top border, `--bg-secondary` background, `--card-shadow` elevation.

### W1.3: Header Navigation

**Modify**: `public/index.html` AND `public/startup-box.html`

Insert `<nav>` between `.logo` and `.header-controls`:

```html
<nav class="header-nav" aria-label="Main">
  <a href="/startup-box" class="header-nav-link">Startup Box</a>
</nav>
```

**CSS additions** (in `styles.css`):

- `.header-nav` — flex, center-aligned, gap
- `.header-nav-link` — `--text-sm`, `--text-secondary`, hover `--accent-1`

### W1.4: Waitlist Source Fix

**Modify**: `src/api/waitlist.js` — read `body.source`, whitelist `['landing', 'startup-box']`, pass to INSERT.

**Modify**: `public/app.js` — detect page via `location.pathname`, include `source` in POST body.

### W1.5: Sitemap + Robots

**Modify**: `public/sitemap.xml` — add `/startup-box` entry.

### W1.6: Commit + Deploy

Single commit: `feat: add /startup-box product page with nav and landing card`

Verify live:

- `GET /startup-box` returns 200
- Nav link present on both pages
- Product card on landing page links correctly
- Waitlist form on `/startup-box` records `source: 'startup-box'`

---

## Workstream 2 — Prototype

### Dependency Map

```bash
Step 1: Server Foundation (no deps)
  ↓
Step 2: Frontend Shell (needs: SSE, static serving, state API)
  ↓
Step 3: Ingestion + Templates (needs: inferenceAdapter, queue, state, promptLoader; views from Step 2)
  ↓
Step 4: Personas (needs: contextPackager, templateFiller from Step 3)
  ↓
Step 5: Octet + Org Canvas (needs: personas from Step 4)
  ↓
Step 6: Meetings + Attachment (needs: Octet, timeline from Step 5; personas from Step 4)
```

No circular dependencies. Each step produces a testable, committable increment.

### Ambiguity Log

| # | Ambiguity | Resolution |
| --- | --- | --- |
| 1 | §28 says "Cloud inference backends" out of scope, but §21 details all 5 sources | The adapter + config UI are in scope; "cloud-only mode" (no Ollama) is out of scope. Ollama required. |
| 2 | §27 shows `octet-box-data/` inside repo tree, but §20.2 says sibling | §20.2 is authoritative — sibling directory, configurable via `DATA_DIR` env var |
| 3 | Template files referenced but no content provided | Author all ~30 templates as part of each step |
| 4 | Prompt files referenced but no content provided | Author all ~25 prompts iteratively, test with Ollama at each step |
| 5 | 14 scaffold template pairs listed but content unspecified | Author as part of Step 6 (scaffoldManager) |

### Mock/Fixture Strategy

| External Dep | Mock Location | Format |
| --- | --- | --- |
| GitHub REST API | `tests/fixtures/github-api/` | JSON (repos, readme, tree, contents) |
| Ollama `/api/tags` | `tests/fixtures/ollama/tags.json` | JSON model list |
| Ollama `/api/chat` | `tests/fixtures/ollama/chat-*.ndjson` | Streaming NDJSON |
| HTML pages | `tests/fixtures/html/` | Raw HTML files |
| State snapshots | `tests/fixtures/state/` | `state.json` per schema version |

### Risk Register

| # | Risk | Impact | Mitigation |
| --- | --- | --- | --- |
| 1 | Prompt quality on 7B models | Useless output, broken JSON | Author iteratively with Ollama; fallback parsing for non-JSON |
| 2 | Token budget overflow on 4K context | Truncated/garbled context | Hard caps in contextPackager; priority-based truncation; unit tested |
| 3 | SSE connection drops | Lost real-time updates | Server heartbeat 30s; client exponential backoff reconnect |
| 4 | fs.watch platform inconsistency | Missed file changes | 300ms debounce; ENOENT catch; macOS-specific testing |
| 5 | Meeting context exceeds model window | Hallucination/repetition | ContextRollingManager summarizes every 10 exchanges |

### Step Sizing — Hardest Challenge Per Step

| Step | Hardest Part | Approach |
| --- | --- | --- |
| 1 | inferenceQueue: priority + prerequisites + persistence + restart resume | Build queue as isolated module, 15+ unit tests before anything depends on it |
| 2 | CSS grid 3-column layout with 10 switchable views | Define all view containers up front; show/hide via `data-view` + phase gating |
| 3 | ANALYST-driven iterative FetchQueue (multi-round inference) | Strict 10-round/30-file hard cap; JSON parse with fallback; tested with fixtures |
| 4 | Processing state machine with queue gating | State FSM in stateManager; queue checks prerequisites before dequeue |
| 5 | Octet Pulse consensus framework (5 modes) | Implement Ratification first (simplest); add modes incrementally; each mode tested |
| 6 | ContextRollingManager for long meetings | Configurable summary interval; rolling summary replaces raw transcript in context |

### P0: Repository Setup

**Create in `~/dev/octet-startup-box/`:**

```bash
package.json          — type:module, scripts only, zero deps
.gitignore            — octet-box-data/, .env, node_modules/, test-results/
.env.example          — DATA_DIR, PORT, OLLAMA_HOST, PULSE_INTERVAL
README.md             — prerequisites, install, run
CLAUDE.md             — agent session context
```

### Step 1: Server Foundation

**Files** (10):

- `server.js` — `node:http`, static serving, MIME map, router import
- `server/router.js` — route table from §25.2, 501 stubs for unimplemented
- `server/sseEmitter.js` — client registry, broadcast, heartbeat, cleanup
- `server/stateManager.js` — read/write/patch state.json, migration chain, backup
- `server/inferenceAdapter.js` — unified `chat()` routing to source adapters
- `server/inferenceAdapters/ollamaAdapter.js` — streaming NDJSON parsing
- `server/inferenceAdapters/openRouterAdapter.js` — stub
- `server/inferenceAdapters/openAIAdapter.js` — stub
- `server/inferenceAdapters/anthropicAdapter.js` — stub
- `server/inferenceQueue.js` — priority queue, prerequisites, persistence, pause/resume, approval gate
- `server/ollamaClient.js` — /api/tags proxy, health check
- `server/runLogger.js` — JSONL append logger with PackageManifest
- `server/promptLoader.js` — load from defaults + override, YAML frontmatter, interpolation

**Tests**: stateManager, inferenceQueue, sseEmitter, promptLoader (node:test)

**Verify**: `node server.js` starts, serves static, SSE connects, `/api/state` returns initial state, queue persists across restart.

### Step 2: Frontend Shell

**Files** (10):

- `public/index.html` — 3-column grid, topbar, 10 view containers
- `public/css/main.css` — §5 palette, grid layout, topbar, sidebar, right panel
- `public/css/components.css` — cards, buttons, inputs, badges
- `public/css/animations.css` — pulse, shimmer, typing, phase transition
- `public/js/app.js` — SSE + reconnect, view router, phase handler
- `public/js/state/store.js` — client state mirror, subscribe pattern
- `public/js/state/sseHandler.js` — map all 26 SSE events to store
- `public/js/components/phaseProgress.js` — sidebar phase tracker
- `public/js/components/queuePanel.js` — right panel queue tab
- `public/js/components/octetFeed.js` — right panel feed tab

**Verify**: Layout renders, SSE connects, view switching works.

### Step 3: Ingestion + Templates

**Files** (15 server + 5 frontend + 9 templates + 2 prompts):

- `server/ingestors/htmlTextExtractor.js` — pure function, 16K cap
- `server/ingestors/repoDetector.js` — URL pattern matching
- `server/ingestors/githubIngestor.js` — FetchQueue, 10 rounds, 30 files
- `server/ingestors/websiteIngestor.js` — fetch + extract
- `server/ingestors/folderIngestor.js` — webkitdirectory batch
- `server/ingestors/gitlabIngestor.js` — stub
- `server/ingestors/bitbucketIngestor.js` — stub
- `server/ingestors/codebergIngestor.js` — stub
- `server/agents/contextPackager.js` — heading-depth extraction, token budget, priority
- `server/templateFiller.js` — static + infer blocks, fillManifest, override lookup
- `server/agents/analyst.js` — ContextObject + FetchQueue prompts
- `server/phases/phase1-ingest.js` — orchestrator
- Frontend: setup.js, ingestionMonitor.js, contextExplorer.js, fileTree.js, modelSelector.js
- Prompts: analyst-context.md, analyst-fetchqueue.md
- Templates: 9 context templates

**Tests**: htmlTextExtractor (15+), contextPackager, templateFiller, githubIngestor (node:test + Playwright)

**Verify**: Enter GitHub URL, watch ingestion, context files appear with frontmatter.

### Step 4: Personas

**Files** (4 server + 3 frontend + 8 templates + 7 prompts):

- `server/agents/personaGenerator.js` — sequential CEO → CFA → CFB
- `server/agents/memoryManager.js` — read/write/update memory files
- `server/coderFileGenerator.js` — CLAUDE.md + AGENTS.md generation
- `server/phases/phase2-founding.js` — orchestrator
- Frontend: foundingCouncil.js, personaCard.js, memoryViewer.js
- Prompts: 7 persona prompts
- Templates: 8 persona templates

**Verify**: 3 personas generated, 6 files each, memory viewer works, coder toggle generates files.

### Step 5: Octet + Org Canvas

**Files** (5 server + 4 frontend + 8 prompts + 6 templates):

- `server/agents/octetAgent.js` — pulse loop, consensus, feed, approval
- `server/agents/architect.js` — OrgManifest, vacancy scan
- `server/agents/timelineManager.js` — 3-state model, _timeline.md
- `server/phases/phase3-org.js` — org structure orchestrator
- `server/phases/phase4-growth.js` — vacancy fill orchestrator
- Frontend: orgCanvas.js, timeline.js, timelineItem.js + CSS files
- Prompts: 8 Octet/architect prompts
- Templates: 6 timeline/artifact/scaffold templates

**Verify**: Org structure visible, vacancies detected, timeline shows items, Pulse fires.

### Step 6: Meetings + Attachment

**Files** (3 server + 8 frontend + 10 prompts + 6 templates):

- `server/agents/meetingOrchestrator.js` — context assembly, turns, rolling summary
- `server/scaffoldManager.js` — CLAUDE.md + AGENTS.md at all directory levels
- `server/fileWatcher.js` — fs.watch + debounce + SSE events
- Frontend: meetingBuilder.js, meetingRoom.js, oneOnOne.js, artifactViewer.js, meetingPlanBar.js, contextAttach.js + CSS files
- Prompts: 10 meeting prompts
- Templates: 6 meeting/artifact templates

**Verify**: All 33 Definition of Done items pass. Cold start to Org Canvas under 15 minutes on 7B.

### Post-Build: Website Update

After all 33 DoD items pass:

1. Tag `v0.1.0` in octet-startup-box repo
2. Update `/startup-box` on octetois.com: replace "In Development" → "v0.1 Available", add install block, add "What's in v0.1" section
3. Update landing page product card to reflect available status
4. Deploy website

---

## Critical Files Reference

### Website (modify)

- `public/index.html` — product card (W1.2), nav (W1.3)
- `public/styles.css` — nav styles, product-highlight styles (W1.1-W1.3)
- `public/app.js` — waitlist source detection (W1.4)
- `src/api/waitlist.js` — accept source field (W1.4)
- `public/sitemap.xml` — add /startup-box (W1.5)

### Website (create)

- `public/startup-box.html` — full product page (W1.1)

### Prototype (create — ~160 files total)

- See step breakdowns above for complete file lists

---

## Verification — End-to-End

### Website

- `curl -I https://octetois.com/startup-box` → 200
- Nav link visible on both pages
- Waitlist POST with source `startup-box` records correctly in D1
- Product card on landing page links to `/startup-box`

### Prototype

- All 33 Definition of Done items from spec §29
- `node server.js` starts with zero installs
- Full test suite green: `npm test && npm run test:e2e`
- `git ls-files octet-box-data/` returns 0 files (CI check)
