# octet-box — "Startup in a Box"
## Project Specification v0.8
**Repo:** `octet-startup-box` (new, public, MIT)
**Owner:** octet:ois / Ben McNulty
**Target:** Locally-served pure HTML/CSS/JS web app + zero-dep Node.js server
**Status:** Pre-development — agent planning input

---

## Table of Contents

1. [Vision](#1-vision)
2. [Core Principles](#2-core-principles)
3. [System Architecture](#3-system-architecture)
4. [Application Phases](#4-application-phases)
5. [Visual Design Language](#5-visual-design-language)
6. [Layout & Navigation](#6-layout--navigation)
7. [Setup Panel — Phase 0](#7-setup-panel--phase-0)
8. [Context Ingestion — Phase 1](#8-context-ingestion--phase-1)
9. [Context Packaging Architecture](#9-context-packaging-architecture)
10. [Persona System](#10-persona-system)
11. [Persona Memory System](#11-persona-memory-system)
12. [Octet Orchestration Agent](#12-octet-orchestration-agent)
13. [Org Canvas — Departments, Teams & Roles](#13-org-canvas--departments-teams--roles)
14. [Unified Meeting Builder](#14-unified-meeting-builder)
15. [Meeting Room](#15-meeting-room)
16. [One-on-One Interface](#16-one-on-one-interface)
17. [Organization Timeline](#17-organization-timeline)
18. [Inference Queue Manager](#18-inference-queue-manager)
19. [Universal Context Attachment System](#19-universal-context-attachment-system)
20. [File Segregation & Update Architecture](#20-file-segregation--update-architecture)
21. [Inference Source Management](#21-inference-source-management)
22. [Template System](#22-template-system)
23. [Agent Integration Scaffolding](#23-agent-integration-scaffolding)
24. [AppState Schema](#24-appstate-schema)
25. [Server Architecture](#25-server-architecture)
26. [Prompt Architecture](#26-prompt-architecture)
27. [Repo Structure](#27-repo-structure)
28. [MVP Scope](#28-mvp-scope)
29. [Definition of Done](#29-definition-of-done)
30. [Dev Planning Agent Handoff](#30-dev-planning-agent-handoff)

---

## 1. Vision

**Startup in a Box** is a fully self-contained, locally-hosted organizational intelligence environment. It transforms raw startup context into a living system of autonomous AI personas that plan, meet, remember, and evolve — powered entirely by local Ollama inference, with zero external dependencies.

The application is not a generator and not a chatbot. It is a **founder's operating system**: a persistent, self-sustaining organizational simulation where AI personas hold domain expertise, accumulate memory, participate in structured meetings, and autonomously advance organizational work — all under the founder's full observability and control.

**The self-sustaining loop:**
```
Context → Org Structure → Persona Memory → Meetings → Artifacts → Timeline
    ↑                                                                  │
    └──────────────── Octet Pulse (continuous synthesis) ─────────────┘
```

Everything is local. Everything is editable. Everything is observable in real time.

---

## 2. Core Principles

- **Local-first, zero cloud**: All inference via Ollama. All storage on local filesystem.
- **Pure web stack**: HTML/CSS/vanilla JS frontend. No React, no Vue, no bundler.
- **Zero server deps**: Raw `node:http`. No Express, no npm packages.
- **Token efficiency as architecture**: Every agent call uses the minimum necessary context. Context is never dumped wholesale — it is targeted, structured, and declared.
- **Markdown as the universal format**: All context, memory, artifacts, and timeline entries are `.md` files. Heading structure IS the organizational structure.
- **Progressive interface**: The UI grows with the organization. Each phase unlocks new panels. Nothing is hidden — it just hasn't been born yet.
- **Unified attachment everywhere**: Any context document or persona can be added to a meeting plan from any view with one click.
- **Full observability + editability**: Every autonomous action is visible, logged, and reversible. The system can be edited as it runs.
- **Self-sustaining with human override**: Autonomous operation is the goal; human judgment is always one toggle away.

---

## 3. System Architecture

### 3.1 Component Map

```
┌──────────────────────────────────────────────────────────────┐
│                  Browser (localhost:3000)                     │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  TOPBAR: Logo | Org | Phase | Pulse | Autonomy | Queue │  │
│  ├───────────────┬──────────────────────────┬─────────────┤  │
│  │  LEFT PANEL   │      MAIN CANVAS         │ RIGHT PANEL │  │
│  │               │                          │             │  │
│  │  Phase Track  │  [Switchable Views]       │  Octet Feed │  │
│  │  Context Tree │  Setup / Ingestion /      │  Inf. Queue │  │
│  │  Personas     │  Context Explorer /       │  Timeline   │  │
│  │  Org Outline  │  Org Canvas /             │  Sidebar    │  │
│  │  Timeline     │  Meeting Room /           │             │  │
│  │               │  1:1 Interface /          │             │  │
│  │               │  Artifact Viewer          │             │  │
│  └───────────────┴──────────────────────────┴─────────────┘  │
│                                                              │
│  [+ Meeting Plan]  floating action — visible on all views    │
└──────────────────────────────────────────────────────────────┘
                        │ fetch / SSE
┌───────────────────────▼──────────────────────────────────────┐
│              Node.js Local Server (port 3000)                 │
│  Static serving | Ollama proxy | Filesystem API | SSE stream  │
└───────────────────────┬──────────────────────────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────────┐
│  /octet-box-data/                                             │
│    state.json          AppState                              │
│    context/            Organized ingestion files (.md/.txt)  │
│    personas/           Per-persona memory + knowledge files   │
│    meetings/           Transcripts, agendas, action items     │
│    timeline/           Three-state org timeline              │
│    artifacts/          Generated org documents               │
│    queue/
    config/
      sources.json               # API keys + source config (never in git)
    prompts-override/              # user-customized prompts
    templates-override/            # user-customized templates
    coders/                        # CLAUDE.md + AGENTS.md per coder persona              Inference queue + run logs            │
└───────────────────────┬──────────────────────────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────────┐
│  Ollama (localhost:11434)                                     │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Application Phases

Phases persist in `state.json`. App resumes phase on reload. Each phase unlocks new UI panels.

```
PHASE 0 — SETUP
  Ollama auto-detected. Model selected. Context inputs provided.

PHASE 1 — INGESTION
  Context sources ingested → organized /context/ file tree.
  ANALYST agent synthesizes → ContextObject + _index.md.
  User reviews and edits context files. Re-ingestion available.

PHASE 2 — FOUNDING COUNCIL
  CEO + 2 Co-Founders generated (context-derived).
  User reviews, edits, approves each persona.
  Each persona's memory system initialized.
  Octet activated with org ground truth.

PHASE 3 — ORGANIZATION DRAFT
  Octet + triumvirate draft OrgManifest: departments, teams, roles.
  Vacancy detection. Timeline initialized (PLANNED state).
  First Sprint created.

PHASE 4 — ORGANIZATIONAL GROWTH
  New personas fill vacancies (autonomous or manual).
  Departments and teams expanded.
  Org Canvas fully interactive.

PHASE 5 — ONGOING OPERATIONS (steady state)
  Meeting Room + 1:1 active. Octet Pulse running.
  Inference Queue managed. Timeline advancing.
  Self-sustaining loop with full observability.
```

---

## 5. Visual Design Language

**Aesthetic:** Mission control meets founder's war room. Dark, precise, alive. System state is communicated through color and motion — not text labels alone.

```css
/* Core palette */
--color-bg:              #0a0b0e;
--color-surface:         #111318;
--color-surface-raised:  #161820;
--color-border:          #1e2129;
--color-border-active:   #2e3340;

/* Semantic state colors */
--color-octet:           #00d4ff;   /* Octet agent — electric cyan */
--color-pulse:           #00ff88;   /* active/alive */
--color-planned:         #7c6fff;   /* planned state — purple */
--color-active:          #ffaa00;   /* active/in-progress — amber */
--color-completed:       #00cc66;   /* completed — green */
--color-vacancy:         #ff6b35;   /* gap/missing — orange */
--color-danger:          #ff4444;   /* error/blocked */
--color-user:            #a78bfa;   /* user voice — lavender */

/* Text */
--color-text-primary:    #e8eaf0;
--color-text-dim:        #6b7280;
--color-text-faint:      #374151;

/* Accent */
--color-accent:          #7c6fff;
```

**Typography:**
- Interface chrome: `IBM Plex Mono` — technical authority
- Agent speech + content: `Inter` — readable, neutral
- Persona names: `DM Mono` — distinct identity
- Meeting chat: monospace, 0.9rem

**Motion guidelines:**
- Phase transitions: 300ms slide-in from right, ease-out
- Pulse indicator: CSS `@keyframes` radial glow, 2s loop
- New persona card: scale(0.95)→scale(1) + fade-in, 250ms
- Inference active: skeleton shimmer on affected component
- Timeline state change: color wash transition, 400ms
- Typing indicator: 3-dot bounce while agent inference runs

---

## 6. Layout & Navigation

### 6.1 Topbar (always visible)

```
[◈ octet:box]  [Org Name]  Phase: Org Draft  |  [●  Octet Pulse]  [Autonomy ○──●]  [Thinking: 2]  [Coding: 1]  [Queue: 3 ▼]  [Board ↗]
```

- **Org Name**: editable inline on click
- **Phase label**: clicking opens Phase Progress panel
- **Octet Pulse indicator**: animated state (see §12.2)
- **Autonomy Toggle**: ON/OFF for autonomous Pulse actions
- **Thinking badge**: number of healthy `thinking` source lanes and their active model assignments; clicking opens Sources config
- **Coding badge**: number of healthy `coding` source lanes and their active model assignments; clicking opens Sources config
- **Queue badge**: count of pending inference jobs; clicking opens Inference Queue panel (see §18)
- **Board link**: opens `/the-board` in a dedicated observation layout (see §6.6)

### 6.2 Left Sidebar

Always visible. Sections collapse/expand.

```
LEFT SIDEBAR
─────────────────────────────
▼ Phase Progress
  ● Setup         ✓
  ● Ingestion     ✓
  ● Founding      ✓
  ● Org Draft     → (active)
  ● Growth        …
  ● Operations    …

▼ Context
  [browsable file tree]
  [+ Add to Meeting ⊕]  ← on hover over any file

▼ Active Personas
  ◉ Sarah Chen — CEO
  ◉ Marcus Webb — CTO
  ◉ Layla Torres — CPO
  + 2 more
  [+ Add to Meeting ⊕]  ← on hover over any persona

▼ Org Structure
  [Department outline]
  [+ New Department]

▼ Timeline
  [mini timeline — latest 5 items]
  [View Full Timeline →]
─────────────────────────────
```

### 6.3 Main Canvas Views

| View | Key | Unlock Phase |
|---|---|---|
| Setup | `setup` | 0 |
| Ingestion Monitor | `ingest` | 1 |
| Context Explorer | `context` | 1+ |
| Founding Council | `founding` | 2 |
| Org Canvas | `org` | 3+ |
| Meeting Builder | `meeting-builder` | 2+ |
| Meeting Room | `meeting` | 2+ |
| One-on-One | `1on1` | 2+ |
| The Board | `board` | 2+ |
| Timeline | `timeline` | 3+ |
| Artifact Viewer | `artifacts` | 1+ |

### 6.4 Right Panel

Always visible. Two sub-tabs:

```
RIGHT PANEL
─────────────────────────────
[◈ Octet Feed]  [⏱ Queue]
─────────────────────────────
(content of active tab — see §12.3 and §18)
```

### 6.5 Floating Action — Active Meeting Plan

A persistent floating bar appears at the bottom of every view once a meeting plan is open:

```
┌──────────────────────────────────────────────────────────────┐
│  ⊕ Active Meeting Plan: "Tech Stack Review"                  │
│  Personas: Sarah, Marcus  |  Docs: stack.md, risks.md  |  2+ │
│  [Open Builder →]  [Clear]                                   │
└──────────────────────────────────────────────────────────────┘
```

This bar is the visual anchor of the Universal Context Attachment System (§19).

### 6.6 Board Endpoint — `/the-board`

`/the-board` is a dedicated observation surface for founders, board members, and operator stakeholders. It is not a second application. It is a specialized route over the same live state, queue, meetings, and timeline data.

Core requirements:

- fullscreen billboard mode for wall displays and large monitors
- responsive quick-dashboard mode for phones, tablets, and laptops
- zoom levels: `wall`, `room`, and `detail`
- clickable drill-down tiles for workstreams, meetings, queue lanes, personas, and timeline items
- live grouping of who is working on what by department, team, role, and active queue assignment
- persistent **[Quick Note]** action that opens an overlay modal and routes the note to Octet immediately

Representative board tiles:

- Org workload map
- Development workflows in progress
- Live meeting wall
- Queue lane utilization
- Strategic tensions and approvals
- Timeline now / next / blocked

Each tile can open a focused billboard view without leaving `/the-board`. Example: clicking a live meeting tile opens a meeting-first display showing the current agenda item, speaking persona, consensus state, and linked artifacts.

---

## 7. Setup Panel — Phase 0

### 7.1 Inference Source Auto-Detection

On page load: `GET /api/sources`.

First-run defaults:
- **Coding default**: local Ollama at `http://localhost:11434`
- **Thinking default**: a user-configurable networked Ollama endpoint, persisted once saved

Every source card exposes **[Test ↺]**. For Ollama-compatible sources, the status check calls `GET /api/tags` against the configured host and stores the runtime response:
- connection status
- discovered model list for dropdown selection
- model count
- response time and last-checked timestamp
- currently selected model for that source record

States:
- **Connected**: green pulse dot, discovered models populate the source dropdown
- **Needs Configuration**: amber dot — host missing or not yet saved
- **Unreachable**: amber dot — last test failed, last successful model list remains visible but stale-badged
- **Error**: red dot — invalid auth, malformed endpoint, or non-Ollama response

Local sources auto-retry every 5 seconds during setup. Networked and cloud sources retry on demand and whenever the user saves the source record.

### 7.2 Input Fields

All fields optional. System adapts to any combination.

| Field | Type | Notes |
|---|---|---|
| **Organization Name** | Text | Defaults to "My Startup" |
| **Brief Description** | Textarea | What, who, why |
| **Website URL** | URL | Any public URL — HtmlTextExtractor |
| **Code Repository URL** | URL | Auto-detects GitHub / GitLab / Bitbucket / Codeberg |
| **Local Folder** | `webkitdirectory` picker | Top-level project folder |
| **Model** | Select | From Ollama; persisted |

### 7.3 Validation & Launch

At least one of: Description, Website, Repo, or Folder required.

**[Initialize Startup in a Box →]** → writes `state.json` → emits `phase:start:1` via SSE → transitions to Ingestion Monitor.

---

## 8. Context Ingestion — Phase 1

### 8.1 Pipeline

```
1. Fetch website URL → HtmlTextExtractor → raw text block
2. Fetch repo URL → RepoIngestor (auto-detected type) → file tree + key files
   └── ANALYST-driven iterative FetchQueue (max 10 rounds / 30 files)
3. Read local folder → FolderIngestor → prioritized file excerpts
4. Merge → IngestBuffer (cap: 32,000 chars)
5. ANALYST agent pass → ContextObject JSON
6. Write /octet-box-data/context/ file tree (see §8.3)
7. SSE: ingest:complete → Context Explorer unlocked
```

### 8.2 Zero-Dep HtmlTextExtractor

Pure function `(rawHtml: string) => PlainTextBlock`. No external libraries. Regex-based tag stripping, entity decode lookup table, 16,000 char cap.

```javascript
// /server/ingestors/htmlTextExtractor.js
export function extractText(rawHtml) { ... }
// Returns: { title, description, headings[], paragraphs[], ctaText[], listItems[], rawCharCount }
```

### 8.3 Repo Source Detection

| Source | Pattern | API |
|---|---|---|
| GitHub | `github.com/{owner}/{repo}` | GitHub REST v3 |
| GitLab | `gitlab.com/{user}/{repo}` | GitLab REST v4 |
| Bitbucket | `bitbucket.org/{user}/{repo}` | Bitbucket REST v2 |
| Codeberg | `codeberg.org/{user}/{repo}` | Gitea API |
| Generic | any `.git` URL | Raw URL conventions |

**ANALYST FetchQueue loop** (GitHub and APIs that support it):
```javascript
// FetchQueue emitted by ANALYST after each seed/round
{ items: [{ path: string, reason: string }], done?: boolean }
```
Orchestrator resolves each path, appends to IngestBuffer, re-calls ANALYST until `done: true` or hard stop.

### 8.4 Context File Tree

```
/octet-box-data/context/
  _index.md                  ← TOC-structured master summary
  org/
    identity.md
    audience.md
    risks.md
  tech/
    stack.md
    architecture.md
    repo-tree.md
  market/
    positioning.md
    competitors.md
  raw/
    website.txt
    readme.txt
    local-files.txt
```

**Every file has frontmatter:**
```markdown
---
octet:context-type: org/identity
octet:section-tags: [identity, name, description, value-prop]
octet:last-updated: 2026-03-28T12:00:00Z
octet:source: website-ingestion
octet:token-estimate: 420
---
# Organization Identity
...heading-structured content...
```

The `section-tags` and `token-estimate` fields enable the Context Packaging system (§9) to select and assemble targeted context without reading full files.

---

## 9. Context Packaging Architecture

**The most critical architectural system for autonomous operation on limited hardware.**

All agent calls in octet-box use **Context Packages** — not raw file dumps. A Context Package is a structured, token-budgeted assembly of only the content a specific agent needs for a specific task.

### 9.1 Markdown Heading Structure as Organizational Contract

Every context file, persona memory file, timeline entry, and meeting artifact uses consistent heading levels with defined semantic roles:

```markdown
# [Document Title]          ← H1: identity / always included when file referenced
## [Major Section]          ← H2: primary domain areas — selectively included
### [Subsection]            ← H3: detail — included only when task requires depth
#### [Item / Entry]         ← H4: individual records — included only for targeted retrieval
```

The orchestrator can include a file at H1-only (identity, ~50 tokens), H1+H2 (overview, ~200 tokens), or full depth (complete, variable tokens) — declared per prompt in frontmatter.

### 9.2 Context Package Assembly

Every prompt file declares its context requirements in frontmatter:

```yaml
---
prompt: meeting-participant
context-depth:
  context/org/identity.md: h2        # include through H2 sections
  context/org/risks.md: full         # full file
  context/tech/stack.md: h1          # title only
  persona/{personaId}/memory.md: h3  # include through H3
  persona/{personaId}/expertise.md: full
token-budget: 6000                   # hard cap for this prompt's context block
priority-order: [memory, identity, risks, stack]  # drop lowest priority if over budget
---
```

The `ContextPackager` module assembles these declarations into a single context string:

```javascript
// /server/agents/contextPackager.js
export async function assemble(promptMeta, variables) {
  // 1. Load each declared file
  // 2. Extract content to declared heading depth using heading-boundary regex
  // 3. Prepend file path as H1 anchor comment: <!-- context: org/identity -->
  // 4. Concatenate in priority order
  // 5. If total > token-budget: truncate lowest priority items first
  // 6. Return assembled context string + PackageManifest (what was included/excluded)
  return { contextString, packageManifest }
}
```

The `PackageManifest` is logged to the run log for every agent call — full auditability of what each agent saw.

### 9.3 Heading-Based TOC Index (`_index.md`)

`_index.md` is a structured table of contents maintained by Octet after any context update:

```markdown
---
octet:context-type: index
octet:token-estimate: 180
---
# Org Context Index

## Organization
- org/identity.md — name, domain, value prop, stage
- org/audience.md — target users, use cases, personas
- org/risks.md — risks, blockers, open questions

## Technology
- tech/stack.md — languages, frameworks, infra
- tech/architecture.md — system design, patterns
- tech/repo-tree.md — directory structure

## Market
- market/positioning.md — competitive position
- market/competitors.md — known competitors

## Raw Sources
- raw/website.txt — extracted landing page text
- raw/readme.txt — repository README
```

Agents receive `_index.md` in every call as a navigation map — letting them "know what to know about" before the ContextPackager assembles their specific context.

### 9.4 Section-Level Retrieval

For large files or deep memory stores, the orchestrator can retrieve a single heading section without reading the whole file:

```javascript
// /server/agents/contextPackager.js
export function extractSection(markdownContent, headingPath) {
  // headingPath: "## Expertise > ### Domain Knowledge"
  // Returns: content between matching heading and next same-level heading
}
```

This enables surgical context inclusion: "give me just the 'Decision Style' section of Sarah's memory file" without loading her full 3,000-token memory document.

---

## 10. Persona System

### 10.1 Founding Council (Phase 2)

Three sequential Ollama calls after context review:
1. **CEO** — visionary, derives from org identity + strategy context
2. **Co-Founder A** — technical domain (inferred from tech stack)
3. **Co-Founder B** — business/market domain (inferred from audience context)

Domain for co-founders is context-derived. Each persona includes:

```javascript
// PersonaObject (in state.json)
{
  id: string,                  // "ceo", "cofounder-cto", etc.
  role: string,
  name: string,
  background: string,
  expertise: string[],
  personality: string,
  decisionStyle: string,
  systemPromptPath: string,    // path to /personas/{id}/system-prompt.md
  memoryPath: string,          // path to /personas/{id}/memory.md
  knowledgePath: string,       // path to /personas/{id}/knowledge.md
  modelId: string,
  department: string | null,
  reportsTo: string | null,    // persona id
  isActive: boolean,
  avatarSeed: string,
  createdAt: string,
  editedAt: string | null
}
```

### 10.2 Persona File Structure

Each persona has a dedicated directory in `/octet-box-data/personas/{id}/`:

```
/personas/{id}/
  profile.md           ← bio, background, expertise (displayed in card)
  system-prompt.md     ← full Ollama system prompt (editable)
  memory.md            ← accumulated experience log (see §11)
  knowledge.md         ← domain expertise knowledge base (see §11)
  relationships.md     ← org map from this persona's perspective
  context-map.md       ← which context files this persona considers relevant
```

### 10.3 Persona Processing States

A persona can be in one of four states, visually indicated on their card:

| State | Indicator | Behavior |
|---|---|---|
| `ready` | `◉` green dot | Fully available for meetings, queue, and 1:1 |
| `processing` | `⟳` spinning cyan arc | Being generated or updated — unavailable |
| `queued` | `◎` dim pulsing ring | Waiting in inference queue — unavailable |
| `inactive` | `○` grey dot | Manually deactivated — excluded from all workflows |

When a persona enters `processing` or `queued` state:
- They are excluded from any new meeting assignments and Pulse actions
- Any queued jobs that depend on this persona are paused until they return to `ready`
- The queue processor inserts their prerequisite jobs (generation, memory init, knowledge init) at Priority 1 ahead of dependent tasks
- The persona card shows a shimmer animation and a status label: "Generating memory…" / "Initializing knowledge…"
- On completion, the card snaps to `ready` with a brief pulse flash, and the paused queue resumes

This ensures the system never calls a persona with incomplete or stale context.

### 10.4 Persona Card UI

```
┌─────────────────────────────────────────┐
│  ◉  Sarah Chen                    [CEO] │
│     Chief Executive Officer        [🖥] │  ← coder badge if set
│  ───────────────────────────────────    │
│  "Pragmatic product strategist          │
│   with 12 years in B2B SaaS..."         │
│                                         │
│  Expertise: Product • GTM • Ops         │
│  Style: Direct, data-driven             │
│  Memory: 12 entries  │  Model: llama3.2 │
│  ───────────────────────────────────    │
│  [✏ Edit] [🧠 Memory] [⚙ Prompt]        │
│  [💬 1:1]  [⊕ Add to Meeting]           │
└─────────────────────────────────────────┘

[PROCESSING STATE — shimmer overlay]
┌─────────────────────────────────────────┐
│  ⟳  Marcus Webb                   [CTO] │  ← spinning arc
│     Chief Technology Officer            │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │  ← shimmer lines
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─                  │
│                                         │
│  Initializing domain knowledge…         │
│  [████████░░░░░░]  Step 2 of 4          │
│  ───────────────────────────────────    │
│  [View Queue] — unavailable until ready │
└─────────────────────────────────────────┘
```

### 10.5 Persona Edit Interface

Inline panel, no modals:
- Editable: Name, Role, Background, Expertise (tags), Personality, Decision Style, Model
- **[⚙ System Prompt]** accordion: full prompt text editable + [Regenerate from profile ↺]
- **[Save Changes]** → updates `state.json` + `profile.md` → immediately live
- **[Regenerate Persona ↺]** → re-runs generation (confirmation required)

### 10.6 Persona Manager — Org-Wide

Accessible from sidebar "Active Personas" and Org Canvas:

| Action | Trigger | Behavior |
|---|---|---|
| Add Manual | Form | Hand-write full persona |
| Add Generated | Role title input | System generates contextually |
| Fill Vacancy | Vacancy card [Fill] | Role pre-filled, one click |
| Deactivate | Card toggle | Excluded from meetings/pulse |
| Clone | Card menu | Duplicate as variant |
| Assign Department | Card menu | Moves in org tree |
| Add to Meeting | `⊕` on any card | See §19 |

### 10.7 Coder Persona Type

Any persona can be designated as a **Coder** — a persona whose identity, memory, and org context are materialized as filesystem files that coding agents (Claude Code, Aider, Cursor, etc.) will automatically load from their working directory.

**Enabling Coder mode:**
A toggle in the persona edit interface: `[🖥 Coder Persona  ○──●]`

When enabled and the persona is saved, two additional files are generated into the persona's directory and into a dedicated `/coders/{personaId}/` directory at the project root:

```
/octet-box-data/personas/{id}/
  CLAUDE.md       ← Claude Code persona file
  AGENTS.md       ← generic agents file (Aider, Codex, etc.)
```

These files are also written to `/coders/{personaId}/CLAUDE.md` and `/coders/{personaId}/AGENTS.md` — structured so that symlinking or copying them into any project directory gives the coding agent the persona's full context.

**CLAUDE.md template structure:**
```markdown
# {{persona.name}} — {{persona.role}}

## Identity
{{persona.background}}

## Coding Philosophy & Style
{{generated: from persona.decisionStyle + persona.expertise + org tech context}}

## Organization Context
{{generated: from org/identity.md H2 + tech/stack.md H2 + tech/architecture.md H2}}

## Current Priorities
{{generated: from persona.memory.md "Open Items" section}}

## Team & Relationships
{{generated: from persona.relationships.md H2 — relevant coding collaborators}}

## Domain Knowledge Relevant to Code
{{generated: from persona.knowledge.md sections tagged with tech/code keywords}}

## Coding Standards for This Project
{{generated: from tech/stack.md full + any /context/tech/*.md files}}
```

**AGENTS.md template structure:**
```markdown
# Agent Context — {{persona.name}} ({{persona.role}})
# Organization: {{orgName}}
# Generated: {{timestamp}}

## Persona
{{persona.background}} {{persona.personality}}

## Technical Authority
{{persona.expertise filtered to technical domains}}

## Project Tech Stack
{{tech/stack.md H2}}

## Architecture Overview
{{tech/architecture.md H1+H2}}

## Active Work & Priorities
{{persona.memory.md "Open Items" H4 entries}}

## Behavioral Guidelines
- Always maintain consistency with the {{orgName}} technical standards
- Reference organization context before proposing architectural changes
- Flag decisions that conflict with known team agreements
```

**Generation process (queued as Priority 1 after persona save):**
1. Load persona files: `profile.md`, `memory.md`, `knowledge.md`, `relationships.md`
2. Load org context: `tech/stack.md`, `tech/architecture.md`, `org/identity.md`
3. Call inference: fill all `{{generated: ...}}` template sections using the declared context sources
4. Write output files: `personas/{id}/CLAUDE.md`, `personas/{id}/AGENTS.md`, `coders/{id}/CLAUDE.md`, `coders/{id}/AGENTS.md`
5. SSE: `persona:coder-files-ready` → card updates to show `[🖥]` badge + [Open Coder Files ↗] button

**Re-generation triggers:**
- Persona profile edited + saved (if coder mode on)
- Memory file updated (debounced — regenerates after 60s of no further updates)
- [Regenerate Coder Files ↺] manual button on persona card
- Org tech context files updated (Octet queues re-gen for all coder personas)

**Coder files are app-managed but org-data-resident** — they live in `/octet-box-data/`, not in the application source. They move with the organization, not the application.

---

## 11. Persona Memory System

Each persona maintains two structured knowledge files. These are the only files referenced when a persona speaks — not the full org context.

### 11.1 Memory File (`memory.md`)

An append-only log of the persona's organizational experiences, updated after every meeting, 1:1, and autonomous action in which they participated.

```markdown
---
octet:context-type: persona-memory
octet:persona-id: ceo
octet:token-estimate: 1240
octet:last-updated: 2026-03-28T15:00:00Z
---
# Sarah Chen — Memory Log

## Decisions Made
### 2026-03-28 — Tech Stack Retained
Decision to maintain TypeScript for API layer following Tech Stack Review meeting.
Context: Marcus presented feasibility analysis. Risk of migration outweighed gains.
Outcome: Marcus tasked with Python ML module prototype.

### 2026-03-27 — Co-founder Alignment
Agreed on 6-month runway target during Founding Session. Layla to own GTM plan.

## Meetings Attended
### 2026-03-28 — Tech Stack Review (Board)
Role: Decision authority. Deferred to CTO on technical specifics.
Key insight: Team alignment on TypeScript is strong — do not disrupt.

## Open Items
### Awaiting: Python ML prototype from Marcus (due: EOW)
### Pending review: GTM plan draft from Layla

## Relationships
### Marcus Webb (CTO): High trust. Defer technical decisions.
### Layla Torres (CPO): Building. Needs more face time.
```

### 11.2 Knowledge File (`knowledge.md`)

The persona's curated domain expertise base — generated initially from context + role, updated as the org accumulates artifacts.

```markdown
---
octet:context-type: persona-knowledge
octet:persona-id: ceo
octet:token-estimate: 980
octet:last-updated: 2026-03-28T12:00:00Z
---
# Sarah Chen — Domain Knowledge

## Product Strategy
### Core Frameworks
Outcome-driven roadmapping, Jobs-to-be-Done, continuous discovery.

### Org Context Application
Product is dev tooling for AI builders. Adoption driven by DX and trust.
Key constraint: small team must deliver visible value fast.

## GTM & Revenue
### Positioning
Open-source core → paid hosted tier. Developer love → enterprise expand.

## Leadership & Operations
### Decision Style
Bias to action. Validate with data when available, move without when not.
Delegate execution, own outcomes.
```

### 11.3 Relationships File (`relationships.md`)

Updated by Octet after any inter-persona interaction. Provides each persona a unique perspective on the org.

```markdown
---
octet:context-type: persona-relationships
octet:persona-id: ceo
---
# Sarah Chen — Org Relationships

## Direct Reports
### Marcus Webb (CTO)
Interaction history: 3 meetings. Trust level: High.
Known preferences: Data before decisions. Hates scope creep.

### Layla Torres (CPO)
Interaction history: 1 meeting. Trust level: Building.
Known preferences: User-centric framing. Proactive communicator.

## Organization Context
Current active team size: 3 personas.
My role in org: Final decision authority. Cultural north star.
```

### 11.4 Memory Review Interface

Accessed via **[🧠 Memory]** on any persona card. Full canvas view:

```
Sarah Chen — Memory & Knowledge
─────────────────────────────────────────────────────────────
[Memory Log]  [Domain Knowledge]  [Relationships]  [Context Map]
─────────────────────────────────────────────────────────────
[file content rendered as editable markdown]

[+ Add Entry]  [Regenerate Section ↺]  [Save]

Memory stats: 12 entries | ~1,240 tokens | Last updated: 3 min ago
```

All four tabs correspond to the four persona files. Every file is fully editable. [Regenerate Section ↺] re-runs the generation prompt for the active section using current org context.

### 11.5 Memory Update Triggers

Memory is updated automatically after:
- Any meeting the persona participated in (post-meeting Octet synthesis pass)
- Any autonomous task completed by the persona
- Manual edit by user
- [Update All Memories ↺] in persona manager (Octet re-synthesizes all)

Memory updates are queued to the Inference Queue (§18) to avoid blocking the main interface.

---

## 12. Octet Orchestration Agent

### 12.1 Identity & Role

Octet is the permanent meta-agent — not a persona, not a participant. It orchestrates, narrates, synthesizes, and maintains ground truth. Identified in UI by `◈` symbol and `--color-octet` (electric cyan) on every surface it touches.

Octet has access to:
- Full `_index.md` (always)
- Current `state.json` phase + persona roster
- Timeline (all three states)
- Inference Queue status
- All meeting transcripts (H1+H2 depth by default, full on demand)

**Octet system prompt** (`/prompts/defaults/octet-system.md`):
```
You are Octet, the orchestration intelligence for {{orgName}}.
You are the organizational operating system — not a persona, not a participant.
You maintain ground truth, facilitate consensus, and advance organizational progress.

Your core operating principles:
1. You speak concisely, factually, and with procedural authority.
2. You never advocate for a position — you surface positions, test alignment, and build consensus.
3. You always structure your outputs as markdown with consistent heading hierarchy.
4. Every decision you facilitate is traceable: you record who agreed, who dissented, and why.
5. You advance the organization toward outcomes — you do not allow process to become an end in itself.

Current org context: {{contextSummary}}
Active personas: {{personaRoster}}
Current phase: {{phase}}
Timeline state: {{timelineSummary}}
```

---

### 12.2 Consensus Framework — Octet's Facilitation Model

This is the core behavioral specification for how Octet operates in all meeting, planning, and orchestration contexts. It is not a prompt — it is the architectural contract that governs Octet's facilitation logic across every workflow in the application.

#### 12.2.1 Foundational Philosophy

Octet operates on the premise that the best organizational decisions emerge from **structured deliberation** — not from the loudest voice, the most senior role, or the fastest response. Its job is to create the conditions where every relevant perspective is heard, weighed, and integrated before a decision is committed.

This is not soft consensus-seeking. Octet drives toward **decisions and outcomes**. It uses process as a tool, not a shield. When genuine alignment exists, Octet moves fast. When it doesn't, Octet surfaces the disagreement explicitly, names the tension, and proposes a resolution path — rather than papering over it.

#### 12.2.2 Consensus Modes

Octet selects a consensus mode based on the decision type, urgency, and number of stakeholders involved. The mode is declared at the start of each decision-requiring meeting exchange:

| Mode | When Used | Process |
|---|---|---|
| **Ratification** | Low-stakes, clear domain ownership | Domain owner proposes → others can object within defined window → silence = consent |
| **Consultative** | Medium-stakes, cross-domain impact | Domain owner decides after input from affected parties → decision logged with dissents |
| **Deliberative** | High-stakes, no clear owner | Structured multi-round exchange → Octet synthesizes → explicit vote or recorded alignment |
| **Directive** | Time-critical or deadlocked | Octet escalates to highest authority persona → decision made unilaterally → logged for review |
| **Deferral** | Insufficient information | Decision formally deferred → Octet creates a research/investigation action item → re-queues |

The current consensus mode is always visible in the meeting canvas during any decision point:
```
◈ Octet  [Deliberative Mode]  Seeking alignment on: Python ML layer decision
```

#### 12.2.3 Structured Deliberation Pattern

When Octet enters deliberative mode, it runs a consistent multi-step pattern:

```
Step 1 — FRAME
  Octet states the decision to be made in a single sentence.
  Octet identifies the decision criteria (speed, cost, risk, reversibility, alignment).
  Octet names which personas hold relevant expertise and stakes.

Step 2 — SURFACE
  Each relevant persona is invited to state their position and primary rationale.
  Octet enforces speaking order and time discipline.
  Octet does not editorialize during surfacing — only records.

Step 3 — MAP
  Octet produces a ConsensusMap: a structured summary of positions.
  Identifies: areas of agreement, areas of tension, unstated assumptions.
  ConsensusMap is rendered in the meeting canvas and logged to the transcript.

Step 4 — TEST
  Octet proposes a synthesized position that attempts to honor all constraints.
  Invites targeted responses: "Does this proposal address your concern, {{persona}}?"
  Registers explicit agreement, conditional agreement, or objection per persona.

Step 5 — COMMIT or ITERATE
  If alignment threshold met (configurable, default: no unresolved objections):
    → Decision committed. Octet writes DecisionRecord (see §12.2.4).
    → Timeline item and action items updated.
  If not:
    → Octet names the remaining blocker explicitly.
    → Switches to Directive mode (if urgency warrants) or Deferral mode.
    → Never cycles more than 2 iterations without escalating or deferring.
```

#### 12.2.4 DecisionRecord

Every decision committed through Octet produces a `DecisionRecord` — written to the meeting's `decisions.md` file and to the organization timeline:

```markdown
#### Decision: Retain TypeScript for API Layer
Date: 2026-03-28  Mode: Deliberative  Meeting: Tech Stack Review

**Decision:** Maintain TypeScript as the primary language for the API layer.
Python ML module to be developed as an isolated, independently deployable service.

**Alignment:**
- Sarah Chen (CEO): ✓ Agreed — cost of migration outweighs near-term gains
- Marcus Webb (CTO): ✓ Agreed — team velocity is the primary constraint
- Layla Torres (CPO): ✓ Agreed — no user-facing impact from language choice

**Dissents / Conditions:** None recorded.

**Rationale Summary:** Team has deep TypeScript fluency. Migration risk is high relative
to ML layer requirements, which can be isolated. Decision is reversible at 6-month review.

**Review Trigger:** Python ML prototype completion (owner: Marcus Webb)
**Committed by:** ◈ Octet
```

DecisionRecords are referenced by persona memory updates, timeline entries, and future meeting agendas. They are the organizational record of *why* things are the way they are.

#### 12.2.5 Dissent Protocol

Octet treats dissent as organizationally valuable, not as an obstacle to route around.

When a persona registers an objection:
1. Octet names the objection explicitly in the ConsensusMap and DecisionRecord — it is not softened or omitted
2. If the decision proceeds over the objection, the dissent is flagged as a `ReviewTrigger` — Octet will resurface it at the next relevant milestone
3. The dissenting persona's memory is updated with the outcome and their recorded position
4. Repeated unresolved dissents from the same persona on related topics surface as a `StrategicTension` item in the Octet Feed — prompting the founder to address the underlying disagreement

#### 12.2.6 Productivity Guardrails

Octet enforces process discipline to prevent unproductive loops:

| Guardrail | Trigger | Action |
|---|---|---|
| **Loop detection** | Same topic discussed >2 rounds without new information | Octet names the loop, forces a decision branch |
| **Scope creep** | Discussion strays >2 levels from agenda item | Octet flags and parks the tangent as a new agenda item |
| **Dominance** | One persona has spoken >60% of last 5 turns | Octet explicitly invites silent personas |
| **Stalemate** | No movement toward alignment after deliberation | Octet escalates to Directive mode or Deferral |
| **Drift** | Decision under discussion conflicts with prior DecisionRecord | Octet surfaces the conflict: "This appears to contradict the decision on [date]. Shall we formally revisit?" |

These guardrails apply in both Autonomous and Board Meeting modes. In Board Meeting mode, they also apply to the user's contributions — Octet will gently flag if the user's input is pushing the meeting off-track, and offer to create a separate agenda item.

#### 12.2.7 Consensus State in the Meeting Canvas

During any decision-active meeting exchange, the meeting canvas renders a live **Consensus Meter** below the active agenda item:

```
─── Agenda Item 2: Python ML Layer Decision ──────────────────────
Consensus Mode: Deliberative  │  Step: 3 of 5 — MAP

ConsensusMap:
  Sarah Chen   ████████████  Aligned — cost concern
  Marcus Webb  ████████░░░░  Conditional — needs prototype first
  Layla Torres ████████████  Aligned — no product impact

Alignment: 66% (1 conditional)  [View Full Map]
─────────────────────────────────────────────────────────────────
```

The Consensus Meter is a live view — it updates after each persona contribution without requiring a page interaction.

#### 12.2.8 Post-Meeting Consensus Synthesis

After every meeting, Octet runs a post-meeting synthesis pass (queued as Priority 2) that:

1. Reviews all `DecisionRecord` entries from the meeting
2. Checks each against the current `_index.md` and `org/risks.md` for consistency
3. Flags any decisions that introduce new risks not previously captured
4. Updates `_timeline.md` with decision summaries
5. Updates each participating persona's `memory.md` with decisions relevant to their domain
6. Generates a `StrategicAlignment` section in the meeting's `summary.md`:

```markdown
## Strategic Alignment Assessment
◈ Octet post-meeting synthesis

### Decisions Consistent with Established Direction
- TypeScript retention: consistent with tech/stack.md positioning

### New Strategic Implications
- Python ML module isolation creates a new service boundary — architecture.md should be updated

### Tensions to Monitor
- Marcus's conditional alignment on Python suggests unresolved uncertainty about ML scope
  → Recommended: 1:1 with Marcus before Architecture Review meeting
```

This synthesis is surfaced in the Octet Feed and appended to the relevant timeline items.

---

### 12.3 Octet Pulse — Heartbeat Loop

**Toggle:** Topbar `[Octet Pulse ○──●]`

```
Pulse Tick (every N seconds, configurable 30–300s):
  1. Assemble PulseContext package:
     - _index.md (H1+H2)
     - state.json summary (phase, persona count, queue depth)
     - timeline/_active.md (H2 only — current sprint/meeting titles)
     - Any items flagged: needsReview: true in state
  2. Call Octet: "Given current org state, what is the highest priority action?"
  3. Octet returns PulseAction:
     {
       action: "update-memory" | "fill-vacancy" | "queue-meeting" | 
               "update-timeline" | "synthesize-artifact" | "flag-for-review" | "idle",
       target: string,
       reason: string,
       requiresApproval: boolean,
       estimatedTokens: number
     }
  4. if requiresApproval → Octet Feed notification [Approve] / [Skip] / [Edit]
  5. if !requiresApproval && autonomy ON → add to Inference Queue immediately
  6. Log to /octet-box-data/queue/pulse-log.jsonl
```

**Pulse indicator states (topbar):**
- `OFF` — grey static dot
- `ON / idle` — slow cyan glow, 2s pulse
- `ON / working` — fast cyan pulse + spinner arc
- `ON / awaiting approval` — amber pulse + badge count
- `ON / queue full` — amber static (queue at capacity, pulse paused)

### 12.4 Octet Feed (Right Panel Tab 1)

```
◈ Octet Feed                                    [Clear] [Export]
──────────────────────────────────────────────────────────────
14:22  ◈ Pulse tick #47
       → Vacancy detected: No legal coverage
       Priority: High
       [Generate Persona: General Counsel]  [Skip]  [Edit]

14:18  ◈ Memory updated
       Sarah Chen — 1 new decision entry
       [View Memory →]

14:10  Autonomous meeting completed
       Q1 Roadmap Planning  •  47 min
       3 action items extracted
       [View Transcript →]  [Review Action Items →]

13:55  ◈ Timeline updated
       "Q1 Roadmap Planning" → COMPLETED
       [View Timeline →]
──────────────────────────────────────────────────────────────
```

Each entry is interactive. Approve/skip/view without leaving the Feed.

---

## 13. Org Canvas — Departments, Teams & Roles

### 13.1 Structure

The org is structured as: **Departments → Teams → Roles → Personas**

```javascript
// OrgManifest (in state.json)
{
  departments: [
    {
      id: string,
      name: string,
      description: string,
      head: string | null,         // persona id
      teams: [
        {
          id: string,
          name: string,
          description: string,
          lead: string | null,
          roles: [
            {
              id: string,
              title: string,
              domain: string,
              mandate: string,
              filledBy: string | null,    // persona id, null = vacancy
              isOpen: boolean,
              priority: "critical" | "high" | "normal" | "future"
            }
          ]
        }
      ]
    }
  ]
}
```

### 13.2 Org Canvas Views

```
[Tree View]  [List View]  [Vacancies Only]
[+ Department]  [+ Team]  [+ Role]  [Run Vacancy Scan ↺]
─────────────────────────────────────────────────────────
◈ Octet (Orchestrator)
    │
    ├── 🏢 Product & Engineering
    │       Lead: Marcus Webb (CTO)
    │       ├── 🔧 Core Platform Team
    │       │       ├── Marcus Webb — CTO [◉ active] [edit]
    │       │       └── ┄ ┄ Backend Engineer  [VACANCY: high] [Fill →]
    │       └── 🎨 Product Team
    │               ├── Layla Torres — CPO [◉ active] [edit]
    │               └── ┄ ┄ Product Designer [VACANCY: normal] [Fill →]
    │
    └── 🏢 Operations & Strategy
            Lead: Sarah Chen (CEO)
            └── 🎯 Executive
                    ├── Sarah Chen — CEO [◉ active] [edit]
                    └── ┄ ┄ General Counsel [VACANCY: critical] [Fill →]
```

### 13.3 Department / Team / Role Management

**Add Department:**
- [+ Department] → inline form: Name, Description, Head (select from personas or leave vacant)
- [Generate with Octet ↺] → Octet proposes department structure from context

**Add Team:**
- [+ Team] within department → Name, Description, Lead
- Teams inherit department context for agent prompts

**Add Role:**
- [+ Role] within team → Title, Domain, Mandate, Priority
- [Generate Role Description ↺] → Octet writes mandate from title + team context
- Role created as vacancy unless persona assigned

**Fill Role (one click):**
- From vacancy card: [Fill with Existing →] → persona selector dropdown
- From vacancy card: [Generate New Persona →] → queues persona generation with role context pre-filled

**Bulk operations:**
- [Run Vacancy Scan ↺] → Octet reviews full org + context → emits VacancyReport → highlights missing roles
- [Auto-fill All Critical Vacancies] → queues persona generation for all `priority: critical` vacancies (requires approval if autonomy off)

### 13.4 Role Card (Vacancy State)

```
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
  ⚠ VACANCY — Critical
  General Counsel
  Legal / Compliance
  "Oversee legal risk, contracts, and regulatory compliance..."
                                                  
  [Fill with Existing →]  [Generate Persona →]  [Edit Role]
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

---

## 14. Unified Meeting Builder

A single interface for planning any type of meeting. Accessible from: topbar [+ Meeting], any persona card [⊕ Add to Meeting], any context file [⊕ Add to Meeting], sidebar [+ New Meeting], or Timeline planned item.

### 14.1 Meeting Types & Autonomous Toggle

**Meeting Type** determines Octet's facilitation style and participant auto-suggestions. It is independent of who is driving the meeting.

| Type | Facilitation Style | Auto-Suggest Participants |
|---|---|---|
| `kickoff` | Vision alignment, ground rules | All active personas |
| `planning` | Goal-setting, task breakdown | Relevant department leads |
| `review` | Progress vs. plan, blockers | Assigned role owners |
| `retrospective` | What worked, what didn't | Sprint participants |
| `strategy` | Direction-setting, tradeoffs | C-Suite + relevant leads |
| `working-session` | Focused task execution | Domain expert(s) |
| `all-hands` | Broadcast + Q&A | All active personas |
| `custom` | Free-form | Manual selection only |

**Autonomous Toggle** — a separate binary control on the meeting plan, distinct from type:

```
Mode: [● Autonomous]  [○ Board Meeting]
```

| Mode | Behavior |
|---|---|
| **Autonomous** | All turns driven by agents. User is an observer. [Inject Note] available at any time for the facilitator to interpret and incorporate without halting the meeting. |
| **Board Meeting** | User participates as a voice in the group chat alongside agents. Octet facilitates. User messages are attributed separately (`👤 You`). |

The `1on1` type always implies Board Meeting mode and does not show the toggle (see §16).

**Injecting into an Autonomous meeting:**
The [Inject Note] button is always visible during any autonomous meeting. Notes are delivered to Octet as a facilitator aside, not as a participant message. Octet decides when and how to weave the note into the meeting flow — it may redirect discussion, add an agenda item, or note it for post-meeting action items. The note is logged in the transcript with `[INJECTED NOTE]` attribution.

### 14.2 Meeting Builder Interface

```
┌──────────────────────────────────────────────────────────────┐
│  ◈ New Meeting Plan                                    [×]   │
│  ──────────────────────────────────────────────────────────  │
│  Meeting Name: [Tech Stack Review              ]             │
│                                                              │
│  Type: [▼ Working Session        ]                           │
│                                                              │
│  Participants                               [⊕ Add Persona]  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ ◉ Sarah Chen — CEO          [×]                      │   │
│  │ ◉ Marcus Webb — CTO         [×]                      │   │
│  │ [+ You (Board Mode)]                                 │   │
│  └──────────────────────────────────────────────────────┘   │
│  Octet suggests: ◉ Layla Torres — CPO  [Add]                │
│                                                              │
│  Context Documents                         [⊕ Add Document]  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ 📄 tech/stack.md               [×]                   │   │
│  │ 📄 tech/architecture.md        [×]                   │   │
│  │ 📄 org/risks.md                [×]                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Agenda Draft (rough — Octet will structure):                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ should we switch to python for the ml layer?         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  [◈ Generate Agenda with Octet ↺]                           │
│                                                              │
│  Generated Agenda:                                           │
│  ── 1. Stack audit: current state (Marcus, 10m)             │
│  ── 2. Python ML layer: feasibility + tradeoffs (All, 15m)  │
│  ── 3. Decision + action items (All, 5m)                    │
│                                                              │
│  Mode: [● Autonomous]  [○ Board Meeting]                 │
│                                                              │
│  [Save to Queue]  [Start Now →]                              │
└──────────────────────────────────────────────────────────────┘
```

### 14.3 Octet History Review for Meeting Planning

When [◈ Generate Agenda with Octet ↺] is triggered, Octet receives:
- Meeting type + name + rough agenda text
- Participant personas (profile.md H1+H2 + memory.md H2 only)
- Attached context documents (H2 depth)
- Timeline: relevant recent items (H1 only — titles + status)
- Any prior meetings with overlapping participants or topics (H1+H2)

This gives Octet the context to produce an agenda grounded in organizational history — not a generic template.

### 14.4 Participant Auto-Suggestion

After meeting type is selected, Octet emits `MeetingSuggestion`:
```javascript
{
  suggestedParticipants: [
    { personaId: "cto", reason: "Domain expert for tech stack decision" },
    { personaId: "cpo", reason: "Product impact of stack choice" }
  ],
  suggestedDocuments: [
    { path: "tech/stack.md", reason: "Core context for decision" },
    { path: "org/risks.md", reason: "Risk items relevant to migration" }
  ]
}
```
Suggestions appear inline in the builder as [Add] chips — one click to accept.

---

## 15. Meeting Room

### 15.1 Pre-Meeting Context Package Assembly

Before the meeting starts, Octet assembles a `MeetingContextPackage`:

```javascript
{
  agenda: string,                   // structured agenda text
  participantContexts: {            // per-persona context bundle
    [personaId]: {
      systemPrompt: string,         // from system-prompt.md
      memoryExcerpt: string,        // memory.md H2+H3 relevant to agenda
      knowledgeExcerpt: string      // knowledge.md sections relevant to topic
    }
  },
  sharedDocuments: string,          // attached context docs at declared depth
  orgSummary: string,               // _index.md H1+H2 + relationships H1
  priorMeetingExcerpts: string,     // relevant past transcripts H1 only
  tokenBudget: number,              // total for this meeting
  packageManifest: PackageManifest  // logged for auditability
}
```

The total token budget is calculated as: `(model context window × 0.6) - response reservation`.

### 15.2 Autonomous Meeting Canvas

```
◈ Autonomous Meeting — Tech Stack Review              [⏸] [⏹]
────────────────────────────────────────────────────────────────
Duration: 00:04:12   Mode: Autonomous   Participants: Sarah, Marcus

Progress: ████████░░░░░░  Agenda Item 2/3

─── Item 2: Python ML layer — feasibility + tradeoffs ──────────

◈ Octet   14:22   Marcus, please present your feasibility
                  analysis for a Python ML module. Sarah,
                  hold your response until Marcus completes.

⚡ Marcus  14:22   [typing...]
   CTO

────────────────────────────────────────────────────────────────
[Transcript ▾]  [Context Package ▾]  [Inject Note]  [↓ Live]
```

**[Inject Note]** — user can push a note into the meeting context mid-session without taking the floor. Octet decides when/how to integrate it.

**[Context Package ▾]** — expandable panel showing exactly what context each persona has for this meeting. Full observability of the token package.

Token streaming: each Ollama `/api/chat` response is streamed via SSE token by token and appended to the transcript in real time.

### 15.3 Board Meeting Canvas

Identical layout, plus user input at bottom:

```
─── Item 2: Python ML layer ─────────────────────────────────────

◈ Octet   14:22   Marcus, present your analysis.

⚡ Marcus  14:23   Looking at the codebase, our TypeScript core
   CTO             handles 94% of business logic...

👤 You    14:24   What about wrapping existing Rust components?

⚡ Marcus  14:25   [typing...]

────────────────────────────────────────────────────────────────
[You] ┌─────────────────────────────────────────────────────┐
      │                                                     │
      └─────────────────────────────────────────────────────┘
      [Send ↵]  [◈ Let Octet draft for me ↺]  [@ Mention]
```

**[@ Mention]** — direct a message to a specific persona: `@marcus what's the risk?`

### 15.4 Progressive Context Management (Long Meetings)

For meetings that extend beyond the model context window:

```
ContextRollingManager:
  - After every N exchanges (configurable, default: 10):
    - Octet synthesizes a MeetingProgressSummary:
      { decisionsToDate, openItems, currentAgendaItem }
    - Summary replaces raw transcript in active context
    - Raw transcript preserved in /meetings/{id}/transcript-full.md
    - Summary written to /meetings/{id}/transcript-rolling.md
  - Each persona's context for subsequent exchanges:
    [ system-prompt ] + [ rolling summary ] + [ current exchange ]
  - Token budget recalculated after each summary pass
```

This ensures coherence across long meetings without context overflow.

### 15.5 Meeting Output

```
/octet-box-data/meetings/{id}/
  agenda.md                ← structured agenda
  transcript-full.md       ← complete verbatim record
  transcript-rolling.md    ← Octet-synthesized running summary
  action-items.md          ← extracted action items with owners
  decisions.md             ← key decisions reached
  summary.md               ← Octet post-meeting synthesis
```

**`summary.md` structure:**
```markdown
# Tech Stack Review — Meeting Summary

## Key Decisions
### Retain TypeScript for API layer
...

## Action Items
### Marcus Webb — Python ML Prototype
Due: EOW. Deliverable: feasibility doc with benchmarks.

## Open Questions
### Rust wrapper feasibility
Needs follow-up research before next meeting.

## Recommended Next Steps
### Schedule: Architecture Review
Suggested participants: Marcus, Layla
```

Action items from `action-items.md` are automatically appended to the Org Canvas action items feed and to each assigned persona's `memory.md`.

---

## 16. One-on-One Interface

A lightweight, conversational interface for working directly with a single persona — without the ceremony of a full meeting setup.

### 16.1 Launch

From any persona card: **[💬 1:1]**

Or from sidebar: any persona → **[💬 1:1]**

The 1:1 is immediately available — no agenda required. It opens in the main canvas.

### 16.2 1:1 Canvas

```
1:1 with Marcus Webb — CTO
────────────────────────────────────────────────────────────────
Context: [tech/stack.md ×] [tech/architecture.md ×]  [⊕ Add Doc]
Topic:   [Help me plan the Python ML investigation spec   ]

⚡ Marcus   Hi. Based on what I know about our stack, here's
   CTO      how I'd approach scoping the Python ML module...

👤 You      What's the minimum viable prototype scope?

⚡ Marcus   [typing...]
   CTO

────────────────────────────────────────────────────────────────
[You] ┌─────────────────────────────────────────────────────┐
      │                                                     │
      └─────────────────────────────────────────────────────┘
      [Send ↵]  [Save as Artifact ▾]  [→ Escalate to Meeting]
```

### 16.3 1:1 Context

The persona receives:
- Their own `system-prompt.md`
- Their `memory.md` (full — this is their domain)
- Their `knowledge.md` (full)
- Their `relationships.md` (H2 only)
- Attached context documents (declared depth)
- `_index.md` (H1 only — navigation map)

No other personas' contexts. This is the persona thinking for themselves.

### 16.4 Outputs from 1:1

- **[Save as Artifact ▾]** — saves conversation or selected exchange as `.md` in `/artifacts/`
- **[→ Escalate to Meeting]** — pushes current conversation summary + topic into Meeting Builder pre-filled for a follow-up with additional personas
- 1:1 transcript appended to persona's `memory.md` after session ends

---

## 17. Organization Timeline

### 17.1 Three-State Model

Every trackable organizational item lives in one of three states:

| State | Color | Meaning |
|---|---|---|
| `PLANNED` | `--color-planned` (purple) | Defined, not yet started |
| `ACTIVE` | `--color-active` (amber) | In progress |
| `COMPLETED` | `--color-completed` (green) | Done, artifacts finalized |

Octet manages state transitions as part of the Pulse loop.

### 17.2 Timeline Item Types

```javascript
// TimelineItem
{
  id: string,
  type: "phase" | "sprint" | "meeting" | "decision" | "artifact" | "milestone" | "review",
  title: string,
  state: "planned" | "active" | "completed",
  startDate: string | null,
  endDate: string | null,
  description: string,
  artifactPaths: string[],    // linked files
  linkedPersonas: string[],   // persona ids involved
  parentId: string | null,    // e.g. sprint owns meetings
  tags: string[],
  createdAt: string,
  updatedAt: string
}
```

### 17.3 Timeline File Structure

```
/octet-box-data/timeline/
  _timeline.md             ← master TOC (Octet-maintained)
  phases/
    phase-0-setup.md
    phase-1-ingestion.md
    ...
  sprints/
    sprint-1.md
    sprint-2.md
  meetings/
    → symlinks or refs to /meetings/{id}/summary.md
  milestones/
    mvp-launch.md
    ...
```

### 17.4 `_timeline.md` — The Org Chronicle

This is the single source of truth for all organizational history. Structured for both human reading and agent traversal.

```markdown
---
octet:context-type: timeline
octet:token-estimate: 640
octet:last-updated: 2026-03-28T15:30:00Z
---
# Org Timeline — My Startup

## COMPLETED

### Phase 1 — Ingestion
Date: 2026-03-28  Duration: 4 min
Context files created: 12  Sources: website, github, local folder

### Phase 2 — Founding Council
Date: 2026-03-28
Personas created: Sarah Chen (CEO), Marcus Webb (CTO), Layla Torres (CPO)

### Meeting — Tech Stack Review
Date: 2026-03-28  Type: Working Session  Duration: 47 min
Participants: Sarah, Marcus  Mode: Autonomous
Decision: Retain TypeScript. Python ML module to be prototyped.
→ artifact: meetings/mtg-001/summary.md

## ACTIVE

### Sprint 1 — Foundation
Started: 2026-03-28
Goal: Complete org structure draft + first domain plans
Open items: 4  Completed: 2

### Phase 3 — Org Draft
Started: 2026-03-28
OrgManifest: In progress. 2 vacancies identified (critical).

## PLANNED

### Meeting — Architecture Review
Planned by: Octet Pulse (2026-03-28)
Participants: Marcus, Layla
Trigger: Python ML prototype completion

### Milestone — MVP Definition
Target: End of Sprint 1
Owner: Sarah Chen (CEO)
```

### 17.5 Timeline Interface

```
Organization Timeline                    [+ Add Item]  [◈ Auto-Advance ↺]
──────────────────────────────────────────────────────────────────────────
[All]  [Phases]  [Sprints]  [Meetings]  [Decisions]  [Milestones]
──────────────────────────────────────────────────────────────────────────

● COMPLETED ──────────────────────────────────────────────────────────────

  ✓ Phase 1 — Ingestion              2026-03-28    [View →]
  ✓ Phase 2 — Founding Council       2026-03-28    [View →]
  ✓ Meeting — Tech Stack Review      2026-03-28    47m  [View →]

▶ ACTIVE ─────────────────────────────────────────────────────────────────

  → Sprint 1 — Foundation            Started 3/28  4 open  [View →]
  → Phase 3 — Org Draft              In progress          [View →]

○ PLANNED ────────────────────────────────────────────────────────────────

  ○ Meeting — Architecture Review    Trigger: On prototype  [Edit →] [Start]
  ○ Milestone — MVP Definition       End of Sprint 1        [Edit →] [Start]
──────────────────────────────────────────────────────────────────────────
[◈ Octet: Advance Timeline]  generates next planned items from current state
```

**[◈ Octet: Advance Timeline]** — Octet reviews org state and proposes new PLANNED items (meetings, milestones, reviews) based on open action items, phase goals, and organizational context.

### 17.6 Octet Timeline State Transitions

Octet advances timeline items during Pulse ticks:
- `planned → active`: when a meeting starts, sprint kicks off, or user clicks [Start]
- `active → completed`: when a meeting ends, sprint closes, or [Mark Complete]
- Octet writes a completion summary to the item's `.md` file
- `_timeline.md` regenerated after every state change

---

## 18. Inference Queue Manager

### 18.1 Purpose

Inference is managed through a workload-aware queue with dynamic source lanes. Jobs are classified as either `thinking` or `coding`, routed to healthy configured sources for that workload, and executed with full visibility into what is running, where it is running, and which agent or persona owns it.

Default first-run topology:
- one `coding` lane backed by the local Ollama endpoint
- one `thinking` lane backed by a networked Ollama endpoint
- additional lanes appear automatically as more healthy sources are configured and enabled

Meeting turns remain sequential inside a single meeting context even when multiple source lanes exist.

### 18.2 Queue Item Structure

```javascript
// QueueItem
{
  id: string,
  type: "ingest-analyst" | "persona-generate" | "persona-memory-update" |
        "meeting-agenda" | "meeting-turn" | "pulse-tick" | "vacancy-scan" |
      "artifact-generate" | "timeline-advance" | "1on1-turn" |
      "quick-note-route" | "coder-file-generate" | "workflow-snapshot",
  label: string,               // human-readable description
  agentId: string,             // "octet", "analyst", persona id, etc.
    workload: "thinking" | "coding",
    sourceRole: "thinking" | "coding",
    requestedSourceId: string | null,
    assignedSourceId: string | null,
    assignedModel: string | null,
  estimatedTokens: number,
  priority: 1 | 2 | 3,        // 1 = immediate (user action), 2 = normal, 3 = background
  status: "pending" | "running" | "completed" | "failed" | "skipped",
    sequential: boolean,
    contextRecipeId: string | null,   // assemble fresh context at execution time
    contextVersion: number | null,
  enqueuedAt: string,
  startedAt: string | null,
  completedAt: string | null,
  requiresApproval: boolean,
  approvedAt: string | null,
  result: string | null,       // path to output file or inline result
  error: string | null
}
```

### 18.3 Queue Panel (Right Panel Tab 2)

```
⏱ Inference Queue  [Thinking: 2] [Coding: 1]      [Pause All] [Clear Completed]
────────────────────────────────────────────────────────────────────────────────
▶ RUNNING — THINKING LANES
  T1  ⟳  Meeting Turn — Marcus Webb (CTO)
      Source: think-net-01  •  Model: qwen3:32b  •  Est. 340 tokens
      [████████░░░░░░░░] 58%

  T2  ⟳  Pulse Tick #48
      Source: think-net-02  •  Model: llama3.3:70b  •  Est. 200 tokens

▶ RUNNING — CODING LANES
  C1  ⟳  Coder File Generate — Marcus Webb
      Source: code-local-01  •  Model: qwen2.5-coder:14b  •  Est. 420 tokens

── PENDING — THINKING (2) ─────────────────────────────────────────
  1  Quick Note Route — Board Observation          [Skip]
     Priority: 1  •  Octet routing  •  Source policy: thinking-default

  2  Memory Update — Sarah Chen                    [Skip] [↑]
     Post-meeting synthesis  •  Priority: 2  •  Context: v18
     ⚠ Requires Approval  [Approve]

── PENDING — CODING (1) ───────────────────────────────────────────
  3  Workflow Snapshot — Sprint 1 Dev Board        [Skip]
     Priority: 2  •  Source policy: coding-default

── COMPLETED (12) ─────────────────────────────────────────────────
  ✓  Persona Generate — Layla Torres (CPO)    14:01  [View]
  ✓  Meeting Agenda — Tech Stack Review       13:58  [View]
  ... (collapsible)
────────────────────────────────────────────────────────────────────────────────
Est. queue completion: ~4 min  |  Healthy lanes: 3  |  Jobs: 3/∞
```

### 18.4 Queue Behavior

- **Priority 1 (immediate)**: User-triggered actions — inserted at front of queue
- **Priority 2 (normal)**: Octet-directed operational tasks
- **Priority 3 (background)**: Pulse-driven autonomous maintenance

- **Workload routing**: planning, meetings, Pulse, memory synthesis, and strategic reasoning default to `thinking`; coder personas, development workflow views, scaffold enhancement, and implementation-oriented jobs default to `coding`
- **Dynamic scaling**: each healthy enabled source contributes one or more worker lanes based on `maxConcurrentJobs`; the queue opens and closes lanes as source health changes
- **Source assignment happens at start time**: pending items hold a context recipe, not a frozen context package; `ContextPackager` assembles the final package immediately before execution
- **Quick Note refresh behavior**: every Quick Note enqueues a Priority 1 `quick-note-route` job, Octet updates `context/org/board-notes.md`, increments `contextVersion`, and all pending jobs recompute their package before they start

If a meeting is running, meeting turns are all Priority 1 within the meeting lane and still execute in strict turn order.

**[Pause All]** — halts queue processing after current job. All pending items remain. Resume with [Resume].

**User can reorder** Priority 2/3 items by drag or [↑] button within their workload grouping.

### 18.5 Queue Persistence

Queue state written to `/octet-box-data/queue/queue.json` after every job state change. Source health and model selections persist in `/octet-box-data/config/sources.json`. On server restart, pending jobs resume after the source registry rehydrates healthy lanes and meeting context jobs are re-initialized from transcript.

---

## 19. Universal Context Attachment System

**The unifying UX pattern of octet-box.** From any view, any surface, any context — one click attaches a document or persona to the active meeting plan.

### 19.1 The `⊕` Attach Button

Every context file in the Context Explorer, sidebar file tree, and Artifact Viewer has an `⊕` icon that appears on hover. Every persona card has an `⊕` button. Clicking either:

1. If no active meeting plan exists: creates a new unnamed plan ("Untitled Meeting")
2. Adds the item (document or persona) to the plan's attachment list
3. A floating Meeting Plan bar appears/updates at the bottom of the screen
4. A subtle toast: "Added: stack.md to meeting plan"

### 19.2 Floating Meeting Plan Bar

Persistent across all views once a plan is open:

```
┌──────────────────────────────────────────────────────────────────┐
│  ⊕ Meeting Plan: "Tech Stack Review"                             │
│  👤 Sarah  👤 Marcus  │  📄 stack.md  📄 risks.md  │  +2 more   │
│  [Open Builder →]  [Start Now →]  [Save to Queue]  [Clear ×]    │
└──────────────────────────────────────────────────────────────────┘
```

[Open Builder →] expands the full Meeting Builder (§14.2) pre-populated with all attached items.

### 19.3 Context Tree `⊕` Integration

Sidebar context tree:

```
▼ Context
  ├── _index.md                        [⊕]
  ├── org/
  │   ├── identity.md                  [⊕]
  │   ├── audience.md                  [⊕]
  │   └── risks.md                     [⊕] ← hover shows
  └── tech/
      ├── stack.md                     [⊕]
      └── architecture.md              [⊕]
```

Clicking `[⊕]` on a folder attaches all files within it.

### 19.4 Persona Quick-Attach

Sidebar "Active Personas" — `⊕` on hover:

```
▼ Active Personas
  ◉ Sarah Chen — CEO              [💬] [⊕]
  ◉ Marcus Webb — CTO             [💬] [⊕]
  ◉ Layla Torres — CPO            [💬] [⊕]
```

### 19.5 Multi-Plan Support

Multiple meeting plans can exist simultaneously (e.g. planning next week's meetings). The active plan (shown in the floating bar) can be switched from the Meeting Builder. Plans are stored in `state.json` under `meetingPlans[]`.

---

## 20. File Segregation & Update Architecture

### 20.1 Hard Boundary

The application and organization data must be fully and cleanly separated. This is a first-class architectural constraint — not an afterthought.

```
octet-box/                         ← APPLICATION ROOT (version-controlled, updatable)
  server.js
  server/
  public/
  prompts/
  package.json
  README.md
  LICENSE
  .gitignore
  .env.example

octet-box-data/                    ← ORGANIZATION DATA ROOT (gitignored, never touched by updates)
  state.json
  context/
  personas/
  meetings/
  timeline/
  artifacts/
  coders/
  notes/
  interface-modules/
  queue/
```

**The contract:** No application file ever writes to a path inside `octet-box/` at runtime. No update to `octet-box/` ever touches `octet-box-data/`. These two directory trees never overlap.

### 20.2 `octet-box-data/` Location

By default, `octet-box-data/` is created as a sibling to the application root — not inside it:

```
/Users/ben/projects/
  octet-box/          ← app (git clone, pull to update)
  octet-box-data/     ← org data (never in git)
```

Configurable via `.env`: `DATA_DIR=/path/to/custom/data/location`

This means a user can `git pull` the latest application code without any risk to their organization data, even if they have been running the system autonomously for months.

### 20.3 State Schema Versioning & Migration

`state.json` includes a `version` field. When the server starts:

1. Read `state.json` version
2. Compare to current application schema version (in `server/stateManager.js` constant)
3. If mismatch: run migration chain (`server/migrations/v0.4→v0.5.js`, etc.)
4. Migrations are additive only — they add new fields with defaults, never delete or rename existing fields without a compatibility shim
5. Original `state.json` is backed up to `state.json.bak.{version}` before migration

```javascript
// server/migrations/index.js
export const migrations = {
  "0.4": migrateV04toV05,
  "0.5": migrateV05toV06,
  // ...
}
```

### 20.4 Prompt & Template Versioning

Prompts in `/prompts/` can evolve with application updates. User-customized prompts must be preserved.

**Pattern:**
- Application ships prompts in `/prompts/defaults/` — read-only source of truth
- User customizations go to `octet-box-data/prompts-override/` — same filename, takes precedence
- `promptLoader.js` checks override directory first, falls back to defaults
- On update: defaults update cleanly, user overrides persist untouched
- UI shows `[customized]` badge on any prompt that has a user override

### 20.5 Template System (see §22)

All generated files (CLAUDE.md, AGENTS.md, persona files, meeting templates, timeline entries) use a versioned template system. Templates live in `/templates/` within the app root. Generated output goes to `octet-box-data/`. Updates to templates do not retroactively alter existing generated files — only new generations use updated templates. A [Regenerate from Updated Template ↺] option is exposed per-file in the UI.

### 20.6 Update Workflow for Users

```
# Update the application
cd octet-box/
git pull

# Start — data is untouched, migrations run automatically on first start
node server.js
```

No manual migration steps. No data loss. Organization resumes exactly where it left off.

---

## 21. Inference Source Management

### 21.1 Source Pools And Workload Roles

Inference sources are organized into two logical workload pools:

| Pool | Purpose | Default Topology | Typical Jobs |
|---|---|---|---|
| **Thinking** | strategy, synthesis, meetings, Pulse, memory, planning | networked Ollama on first run | Octet, ANALYST, meeting facilitation, memory updates |
| **Coding** | implementation-oriented, repo-aware, tool-ready tasks | local Ollama on first run | coder persona files, development workflow snapshots, future code agents |

The system supports multiple configured sources per pool. A source record belongs to one pool, but identical provider types may appear multiple times with different hosts, models, or auth settings.

### 21.2 Supported Providers

| Provider | Deployment | Auth | Model Discovery | Default Concurrency |
|---|---|---|---|---|
| **Local Ollama** | local process | None | `GET /api/tags` | 1 |
| **Networked Ollama** | LAN/remote HTTP | Optional bearer token | `GET /api/tags` | 1 |
| **OpenRouter** | Cloud API | API key | provider list / configured catalog | configurable |
| **OpenAI** | Cloud API | API key | configured catalog | configurable |
| **Anthropic** | Cloud API | API key | configured catalog | configurable |

### 21.3 Source Record Configuration UI

Accessible from: Topbar source badges → **[⚙ Configure Sources]**

```
Inference Sources
────────────────────────────────────────────────────────────────
[Thinking Sources]                              [+ Add Thinking]

  think-net-01  ● Connected  9 models discovered  42ms
  Label: [Networked Ollama - Strategy           ]
  Host:  [http://192.168.1.50:11434             ]  [Test ↺]
  Model: [▼ qwen3:32b                           ]
  Max Concurrent Jobs: [1]

[Coding Sources]                                [+ Add Coding]

  code-local-01  ● Connected  6 models discovered  12ms
  Label: [Local Ollama - Coding                 ]
  Host:  [http://localhost:11434                ]  [Test ↺]
  Model: [▼ qwen2.5-coder:14b                   ]
  Tooling Profile: [▼ coding                    ]
  Max Concurrent Jobs: [1]

[Save Sources]
────────────────────────────────────────────────────────────────
Routing Summary:
  Thinking default → think-net-01
  Coding default   → code-local-01
```

Each source record maintains its own persisted selected model and discovered-model dropdown. The UI does not assume a single global model selector.

### 21.4 Runtime Health Checks And Model Discovery

Status checks are runtime calls to the configured inference endpoint. For Ollama-compatible providers the check is `GET /api/tags`, which returns both liveness and the current model list.

The stored status payload for each source includes:

```javascript
{
  id: string,
  label: string,
  role: "thinking" | "coding",
  provider: "ollama-local" | "ollama-network" | "openrouter" | "openai" | "anthropic",
  host: string,
  enabled: boolean,
  selectedModel: string | null,
  discoveredModels: string[],
  lastStatus: "connected" | "unreachable" | "error" | "needs-config",
  lastLatencyMs: number | null,
  lastCheckedAt: string | null,
  maxConcurrentJobs: number
}
```

This payload is persisted to `octet-box-data/config/sources.json`. Status is refreshed on startup, on manual test, and whenever a lane fails and needs reassessment.

### 21.5 Workload Routing

Queue items declare their workload role up front. Routing policy is then applied in this order:

1. explicit `requestedSourceId`, if present and healthy
2. workload pool default (`thinking` or `coding`)
3. next healthy source in the same pool with available capacity
4. optional fallback source in the same pool

Queue labels always surface the selected workload and assigned source so the operator can see whether a job is being handled as `thinking` or `coding`.

### 21.6 Per-Persona And Per-Workflow Model Assignment

Each persona, workflow, and board view can target a source record rather than a provider class:

```
Persona Edit — Model
  Workload: [▼ coding                           ]
  Source:   [▼ code-local-01                    ]
  Model:    [▼ qwen2.5-coder:14b                ]
  Fallback: [▼ code-local-02                    ]
```

Octet defaults to the `thinking` pool unless a task explicitly requires `coding` behavior or tool-ready execution.

### 21.7 Unified API Adapter (`inferenceAdapter.js`)

A single adapter module normalizes all sources to a common interface. All agent code calls the adapter — never a source directly.

```javascript
// server/inferenceAdapter.js
export async function chat(config) {
  // config: { sourceId, role, model, messages, tools, stream, onToken }
  // Returns: { text, tokensUsed, sourceId, role, model }
}
```

Streaming is normalized: all sources emit tokens via the same `onToken(token)` callback, which feeds the SSE stream regardless of provider. Coding-designated sources may also expose a tool-capable execution profile through the same adapter contract.

### 21.8 Dynamic Queue Scaling And Telemetry

The queue derives its worker lanes from the enabled healthy source set. One source can contribute one lane or many depending on `maxConcurrentJobs` and provider capability.

Parallelizable job groups include:
- persona memory updates after a meeting
- coder file regeneration jobs
- independent board snapshot jobs
- Pulse sub-tasks with no data dependency

Non-parallelizable jobs include:
- meeting turns inside a meeting
- phase-dependent generation chains
- any queue item marked `sequential: true`

For cloud sources, token and cost metrics are stored per job. For local and networked Ollama sources, latency and throughput are recorded instead so operators can compare real runtime behavior across lanes.

---

## 22. Template System

### 22.1 Purpose

All files generated by octet-box — persona files, coder files, meeting artifacts, timeline entries — are produced by filling versioned templates with inference-generated content. This makes every generated file:

- **Predictable in structure**: agents can always find sections by heading
- **Patchable**: template updates propagate cleanly via [Regenerate ↺]
- **Auditable**: each file records which template version produced it
- **Consistent**: no ad-hoc formatting differences between runs or versions

### 22.2 Template Directory

Templates live in `/templates/` in the application root (version-controlled):

```
/templates/
  personas/
    profile.md.tmpl
    memory.md.tmpl
    knowledge.md.tmpl
    relationships.md.tmpl
    context-map.md.tmpl
    system-prompt.md.tmpl
    CLAUDE.md.tmpl
    AGENTS.md.tmpl
  meetings/
    agenda.md.tmpl
    summary.md.tmpl
    action-items.md.tmpl
    decisions.md.tmpl
  timeline/
    sprint.md.tmpl
    milestone.md.tmpl
    phase.md.tmpl
  context/
    identity.md.tmpl
    _index.md.tmpl
  artifacts/
    domain-plan.md.tmpl
    executive-plan.md.tmpl
```

### 22.3 Template Syntax

Templates use two types of insertions:

**Static interpolation** `{{variable}}` — filled by the template engine from known data:
```
{{persona.name}}, {{persona.role}}, {{orgName}}, {{timestamp}}, {{templateVersion}}
```

**Inference fill** `{{infer: context-spec | instruction}}` — filled by an Ollama/API call:
```
{{infer: persona.expertise + tech/stack.md H2 | Write this persona's coding philosophy in 3 sentences}}
{{infer: persona.memory.md "Open Items" | List current priorities as bullet points}}
```

Each `{{infer: ...}}` block becomes one inference call, batched by the `TemplateFiller` module.

### 22.4 TemplateFiller Module

```javascript
// server/templateFiller.js
export async function fill(templatePath, variables, inferenceSource) {
  // 1. Load template from /templates/ (or user override in octet-box-data/templates-override/)
  // 2. Parse: separate static {{}} from infer {{infer:}} blocks
  // 3. Fill all static variables immediately
  // 4. For each infer block:
  //    a. Assemble context from declared context-spec via ContextPackager
  //    b. Enqueue inference job (batched if parallel mode)
  //    c. Insert result at block location
  // 5. Write frontmatter: template version, generated timestamp, source persona/meeting id
  // 6. Return filled content string
  return { content, fillManifest }  // fillManifest: what was inferred vs. static
}
```

The `fillManifest` is appended to the generated file's frontmatter:
```yaml
---
octet:template: personas/CLAUDE.md.tmpl
octet:template-version: "0.5"
octet:generated-at: 2026-03-28T15:00:00Z
octet:fill-manifest: [profile:static, coding-philosophy:inferred, org-context:inferred]
---
```

### 22.5 User Template Overrides

Users can customize any template:
- Copy template from `/templates/` to `octet-box-data/templates-override/` (same relative path)
- `TemplateFiller` checks override directory first
- UI shows `[custom template]` badge when override is active
- [Reset to Default Template] removes override

Application updates to `/templates/` never overwrite user overrides. Users see a notification in the UI when a default template they've overridden has been updated: "Default template updated — [View Diff] [Apply Update] [Keep Custom]"

### 22.6 Template Version in Generated Files

Every generated file's frontmatter records `octet:template-version`. When a template is updated, the UI surfaces stale files:
- Context Explorer: files with outdated templates show a `[↑ template updated]` badge
- Persona memory viewer: same badge on stale sections
- [Regenerate from Updated Template ↺] re-runs fill for that file only
- [Bulk Regenerate Stale Files] queues regeneration for all outdated files (Octet Pulse can do this autonomously if approved)

---

---

## 23. Agent Integration Scaffolding

### 23.1 Purpose & Philosophy

Every directory in `octet-box-data/` and every generated artifact is a potential working context for a frontier model. The autonomous local inference system generates well-structured drafts — but users will inevitably want to bring those drafts to Claude, GPT-4o, Gemini, or their coding agent of choice for deeper enhancement, refinement, or extension.

This section specifies the **Agent Integration Scaffolding** system: a standardized, automatically maintained layer of `CLAUDE.md` and `AGENTS.md` files at every meaningful folder level and alongside every artifact. These files give any external agent or coding tool:

1. **Orientation** — what this folder/artifact is, where it fits in the org, and what it's for
2. **Directives** — how to enhance or modify the contents while preserving integration compatibility
3. **Schema contracts** — the heading structure, frontmatter fields, and formatting rules the application depends on
4. **Re-integration guidance** — how to get enhanced content back into the system seamlessly

The pattern is consistent and predictable: any agent, in any directory, can read `CLAUDE.md` and know exactly what it's working with and what the rules are.

---

### 23.2 Scaffold Coverage Map

`CLAUDE.md` and `AGENTS.md` are generated at every level that has independent semantic meaning:

```
octet-box-data/
  CLAUDE.md                      ← org root: overview of entire data structure
  AGENTS.md
  context/
    CLAUDE.md                    ← context system overview + enhancement guidance
    AGENTS.md
    org/
      CLAUDE.md                  ← org identity context files
      AGENTS.md
    tech/
      CLAUDE.md                  ← tech context files
      AGENTS.md
    market/
      CLAUDE.md                  ← market context files
      AGENTS.md
    raw/
      CLAUDE.md                  ← raw ingested source files (read-only guidance)
      AGENTS.md
  personas/
    CLAUDE.md                    ← persona system overview
    AGENTS.md
    {personaId}/
      CLAUDE.md                  ← this persona: full identity + memory context
      AGENTS.md                  ← (enhanced version for coding agents, see §10.7)
  meetings/
    CLAUDE.md                    ← meeting artifact system overview
    AGENTS.md
    {meetingId}/
      CLAUDE.md                  ← this meeting: context, decisions, follow-up guidance
      AGENTS.md
  timeline/
    CLAUDE.md                    ← timeline structure + three-state model guidance
    AGENTS.md
  artifacts/
    CLAUDE.md                    ← org artifact library overview
    AGENTS.md
    {artifactName}/
      CLAUDE.md                  ← this artifact: purpose, schema, enhancement guidance
      AGENTS.md
  coders/
    CLAUDE.md                    ← coder persona files overview
    AGENTS.md
```

**Root-level `octet-box-data/CLAUDE.md`** is the master orientation document — the first file any external agent should read. It maps the entire data structure and routes to the relevant subdirectory for any task.

---

### 23.3 Scaffold File Types

#### CLAUDE.md

Optimized for Claude Code and similar instruction-following coding agents. Prioritizes:
- Precise behavioral directives in imperative language
- Schema enforcement rules
- Frontmatter field requirements
- What NOT to change (integration-critical structure)

#### AGENTS.md

A more portable format designed for agents that don't specifically recognize `CLAUDE.md` (Aider, Codex, Cursor, custom agents). Same semantic content, slightly more explanatory in tone, less Claude-specific in phrasing.

Both files are generated from the same template pair — they share data, differ in framing.

---

### 23.4 Scaffold Templates

Templates in `/templates/scaffolding/`:

```
/templates/scaffolding/
  root-data-dir.CLAUDE.md.tmpl       ← octet-box-data/ root
  root-data-dir.AGENTS.md.tmpl
  context-dir.CLAUDE.md.tmpl         ← any context/ subdirectory
  context-dir.AGENTS.md.tmpl
  persona-dir.CLAUDE.md.tmpl         ← personas/{id}/ (general)
  persona-dir.AGENTS.md.tmpl
  coder-persona.CLAUDE.md.tmpl       ← personas/{id}/ when coder=true (§10.7)
  coder-persona.AGENTS.md.tmpl
  meeting-dir.CLAUDE.md.tmpl         ← meetings/{id}/
  meeting-dir.AGENTS.md.tmpl
  artifact-dir.CLAUDE.md.tmpl        ← artifacts/{name}/
  artifact-dir.AGENTS.md.tmpl
  timeline-dir.CLAUDE.md.tmpl        ← timeline/
  timeline-dir.AGENTS.md.tmpl
  generic-dir.CLAUDE.md.tmpl         ← fallback for any unrecognized directory
  generic-dir.AGENTS.md.tmpl
```

---

### 23.5 Template Content Structure

Each scaffold template follows a consistent heading structure. The `CLAUDE.md` template for a context subdirectory (`context-dir.CLAUDE.md.tmpl`) is the canonical example:

```markdown
---
octet:scaffold-type: context-dir
octet:template-version: "{{templateVersion}}"
octet:generated-at: {{timestamp}}
octet:scope: {{dirPath}}
octet:org: {{orgName}}
---
# {{dirLabel}} — Context Directory

> This directory is part of the **{{orgName}}** organization context managed by octet-box.
> Read this file before modifying any contents of this directory.

## What This Directory Contains
{{infer: context/_index.md H2 section matching dirPath | Describe in 2-3 sentences what context files in this directory represent and their role in the organization}}

## Files in This Directory
{{infer: list of files in dirPath + their frontmatter | Generate a table: filename | content-type | token-estimate | last-updated}}

## How to Enhance These Files

### What You Should Do
- Expand content within existing `##` and `###` sections
- Add new `###` subsections under established `##` headings
- Improve specificity, accuracy, and depth of existing content
- Correct factual errors or outdated information
- Add examples, evidence, or supporting detail

### What You Must Not Change
- The `---` frontmatter block at the top of each file (the application reads these fields)
- The `# H1` document title
- The `## H2` section headings (these are the application's structural anchors)
- The `octet:context-type` frontmatter value
- File names (the application's context index references files by path)

### Frontmatter You Should Update After Edits
- `octet:last-updated` → set to current ISO timestamp
- `octet:token-estimate` → update to reflect new content length (approximate: chars ÷ 4)

## Re-Integration
After enhancing files in this directory:
1. Save files in place — the application reads directly from this filesystem path
2. In the octet-box interface: **Context Explorer → [Re-run ANALYST ↺]** to re-synthesize `_index.md`
3. If you changed `org/identity.md` or `org/risks.md`: trigger **[Update All Memories ↺]** to propagate changes to persona memory files
4. No import step required — the application will pick up changes on next read

## Organization Context
{{infer: context/org/identity.md H1+H2 | One paragraph summary of the organization for agent orientation}}
```

---

### 23.6 Artifact-Level Scaffolding

Every artifact generated by octet-box — domain plans, executive plans, meeting summaries, sprint reports — gets a sibling `CLAUDE.md` in its directory. For a domain plan:

```
/artifacts/domain-plan-cto/
  domain-plan.md          ← the artifact
  CLAUDE.md               ← enhancement guidance for this specific artifact
  AGENTS.md
```

The artifact `CLAUDE.md` is more specific than a directory-level scaffold. It includes:

```markdown
# CTO Domain Plan — Enhancement Guide

## Artifact Purpose
This is Marcus Webb's (CTO) founding domain plan for {{orgName}}.
It defines the technical strategy, priorities, and first-90-day roadmap for the engineering function.

## Current Quality Assessment
{{infer: domain-plan.md full | Rate completeness 1-10 per section. Identify the 3 weakest sections by specificity.}}

## Recommended Enhancements
{{infer: domain-plan.md + tech/stack.md H2 + tech/architecture.md H2 | List 5 specific ways to strengthen this plan with concrete, org-specific content}}

## Schema Contract
The application reads specific sections of this file by heading name.
These headings must be preserved exactly:
- `## Situation Analysis`
- `## Priorities`
- `## First Week Actions`
- `## Key Hires`
- `## Tools and Infrastructure`
- `## Risks and Mitigations`
- `## Success Metrics`
- `## Dependencies`

## Re-Integration
After enhancing `domain-plan.md`:
- Save in place — no import needed
- In octet-box: the next Pulse tick will detect the updated file and offer to re-sync
  Marcus Webb's memory with the updated plan content
- To force immediate sync: **Personas → Marcus Webb → [🧠 Memory] → [Regenerate Section ↺]**
```

The "Current Quality Assessment" and "Recommended Enhancements" sections are inference-filled at generation time — giving the user a personalized briefing on where the frontier model's effort will be highest value.

---

### 23.7 ScaffoldManager Module

```javascript
// server/scaffoldManager.js

// Called whenever a new directory or artifact is created in octet-box-data/
export async function scaffoldDirectory(dirPath, scaffoldType, context) {
  // 1. Select template pair from /templates/scaffolding/ based on scaffoldType
  // 2. Assemble context package for template fill (ContextPackager)
  // 3. Call TemplateFiller for CLAUDE.md and AGENTS.md
  // 4. Write both files to dirPath/
  // 5. Enqueue re-generation of parent directory scaffold (parent CLAUDE.md lists child dirs)
  // 6. SSE: scaffold:created { path: dirPath }
}

export async function scaffoldArtifact(artifactPath, artifactType, context) {
  // Same as above but for individual artifact files
  // Writes CLAUDE.md + AGENTS.md to artifact's parent directory
}

export async function regenerateAll(reason) {
  // Triggered by: org context update, template version change, [Regenerate All Scaffolds ↺]
  // Walks octet-box-data/ tree, re-generates all scaffold files
  // Queued as Priority 3 background jobs, batched
}

export async function getRootManifest() {
  // Returns parsed content of octet-box-data/CLAUDE.md for UI display
}
```

---

### 23.8 Generation Triggers

Scaffold files are generated or regenerated automatically in the following circumstances:

| Trigger | Scope | Priority |
|---|---|---|
| New directory created (any phase) | That directory only | 2 |
| New artifact file written | Artifact dir + parent dir | 2 |
| New persona created | `personas/{id}/` + `personas/` root | 1 (blocks persona ready state) |
| Persona profile or memory updated | `personas/{id}/` only | 3 (debounced 60s) |
| Context file updated (Context Explorer) | Affected context subdirectory | 3 |
| Org context files updated (identity, risks, stack) | All scaffold files referencing those sources | 3 (batched) |
| Meeting completed | `meetings/{id}/` | 2 |
| Template version updated (app update) | All scaffold files using that template | 3 (Pulse-driven, requires approval) |
| [Regenerate All Scaffolds ↺] (manual) | All directories | 3 (batched) |

The root `octet-box-data/CLAUDE.md` is regenerated whenever any subdirectory scaffold changes — it always reflects the current state of the data tree.

---

### 23.9 Scaffold Viewer in UI

Accessible from: Context Explorer sidebar → any directory → **[◈ View Agent Guide]** button

```
Agent Guide — context/tech/
────────────────────────────────────────────────────────────────
[CLAUDE.md]  [AGENTS.md]                [Regenerate ↺]  [Copy ↗]

# Tech Context Directory — Enhancement Guide

> This directory is part of the [Org Name] organization context...

## What This Directory Contains
Technical context files describing the organization's engineering stack,
architectural patterns, and repository structure...

## Files in This Directory
| File | Content Type | Tokens | Last Updated |
|---|---|---|---|
| stack.md | tech/stack | 340 | 3 min ago |
| architecture.md | tech/architecture | 520 | 1 hr ago |
| repo-tree.md | tech/repo-tree | 180 | 1 hr ago |

...

────────────────────────────────────────────────────────────────
[Open in Explorer]  [Copy Path]  [Export as .zip with contents ↗]
```

**[Export as .zip with contents ↗]** — packages the directory contents + `CLAUDE.md` + `AGENTS.md` as a zip for dropping into an IDE or external agent context. This is the primary handoff mechanism for taking octet-box generated content to a frontier model.

---

### 23.10 Re-Integration Contract

When a user enhances content externally and brings it back, the system needs to absorb the changes cleanly. The re-integration contract is:

**What the user does:**
1. Export directory or artifact (zip or direct file edit)
2. Enhance with frontier model in their IDE or tool of choice
3. Save enhanced files back to the same path in `octet-box-data/`

**What the system does automatically:**
- File watcher on `octet-box-data/` detects changes (`fs.watch`)
- Changed files compared against last-known state in context index (`contextIndex` in AppState)
- For context files: Octet Feed notification — "External changes detected in `tech/stack.md`" → [Re-sync ↺] [View Diff]
- For persona files: memory update queued (Priority 2)
- For artifact files: timeline item flagged as "externally updated" — Pulse will synthesize a changelog entry
- Schema validation: frontmatter fields checked — if required fields missing or heading structure broken, warning surfaced in UI with specific repair guidance

**What never changes:**
- File paths — the context index is path-keyed; moving files breaks references
- `## H2` section headings in context and artifact files — these are structural anchors
- Frontmatter `octet:context-type` values — these route files to the right agent context slots

The `CLAUDE.md` in every directory states these constraints explicitly — so the frontier model working in that context has them in its own working memory.

---

### 23.11 Integration with Coder Persona Files (§10.7)

The persona-level `CLAUDE.md` and `AGENTS.md` specified in §10.7 are a specialization of this system — generated by the same `ScaffoldManager` using the `coder-persona` template pair when a persona has `isCoder: true`. They are richer and more persona-specific than a standard directory scaffold, but follow the same structural contract and re-integration rules.

The `/coders/{personaId}/` directory at the data root contains symlink-ready copies of the persona's coder files, organized so that a user can point any coding agent's working directory there and get the full persona + org context immediately.

### 23.12 Organization Interface Extension Kit

The application supports organization-specific interface development without modifying the core application shell. Custom surfaces compose through a stable extension layer rather than patching internal store logic or core view modules directly.

Principles:

- core routes, queue logic, source routing, meetings, and state remain application-owned
- custom interfaces consume versioned adapters, not internal modules
- built-in components and workflow modules provide reference implementations for common startup operating patterns
- extension surfaces must remain backwards compatible through an abstraction filter that shields organization-specific implementations from internal refactors

Shipped reference modules include:

- development workflow board
- project and milestone board
- live meeting wall
- queue lane monitor
- persona workload map

In-app documentation is required:

- `/docs/style-guide` — visual tokens, layout patterns, animation rules, responsiveness guidance
- `/docs/interface-api` — board adapters, action contracts, manifest schema, compatibility rules

Extension contract example:

```javascript
// public/js/extensions/registry.js
registerBoardView({
  id: "dev-workflow",
  title: "Development Workflow",
  supportsZoom: true,
  minPhase: 2,
  render(adapter, mountNode) {}
})
```

Organization-specific modules live in `octet-box-data/interface-modules/` and are loaded through a versioned manifest. They may read from a stable `BoardDataAdapter` and emit typed actions only. They must not import the core store, queue internals, or private component implementations directly.


## 24. AppState Schema

Full `state.json` schema. Single source of truth.

```javascript
{
  version: "0.5",
  orgName: string,
  description: string,
  phase: 0 | 1 | 2 | 3 | 4 | 5,
  autonomyEnabled: boolean,
  contextVersion: number,            // increments whenever pending jobs must recompose context
  pulseIntervalSeconds: number,       // default: 60

  inferenceRouting: {
    sourcePools: {
      thinking: InferenceSourceRecord[],
      coding: InferenceSourceRecord[]
    },
    defaultSourceIds: {
      thinking: string | null,
      coding: string | null
    }
  },

  inputs: {
    websiteUrl?: string,
    repoUrl?: string,
    repoType?: "github" | "gitlab" | "bitbucket" | "codeberg" | "generic",
    localFolderName?: string
  },

  contextIndex: {                     // indexed summary of all context files
    [path: string]: {
      contextType: string,
      sectionTags: string[],
      tokenEstimate: number,
      lastUpdated: string
    }
  },

  personas: PersonaObject[],

  orgManifest: {
    departments: Department[]
  } | null,

  vacancies: VacancyReport | null,

  meetingPlans: {                     // pending/draft meeting plans
    id: string,
    name: string,
    type: string,
    participants: string[],
    documents: string[],
    agendaDraft: string,
    generatedAgenda: string | null,
    isActive: boolean                 // shown in floating bar
  }[],

  meetings: {
    id: string,
    name: string,
    type: string,
    mode: "autonomous" | "board" | "1on1",
    participants: string[],
    state: "planned" | "active" | "completed",
    startedAt: string | null,
    completedAt: string | null,
    meetingDir: string              // path to /meetings/{id}/
  }[],

  timeline: {
    items: TimelineItem[]
  },

  inferenceQueue: QueueItem[],

  strategicTensions: {             // persistent unresolved dissents surfaced by Octet
    id: string,
    personaId: string,
    topic: string,
    firstRaisedAt: string,
    relatedDecisionIds: string[],
    occurrences: number,
    resolved: boolean
  }[],

  decisionRecords: {               // index of all committed decisions
    id: string,
    title: string,
    meetingId: string,
    date: string,
    alignmentSummary: string,
    hasDissent: boolean,
    reviewTrigger: string | null
  }[],

  actionItems: {
    id: string,
    text: string,
    assignedTo: string,
    sourceId: string,
    completed: boolean,
    dueDate: string | null
  }[],

  quickNotes: {
    id: string,
    text: string,
    authorLabel: string,
    source: "the-board" | "meeting" | "manual",
    status: "received" | "routed" | "applied" | "archived",
    appliedContextVersion: number | null,
    createdAt: string,
    routedAt: string | null
  }[],

  boardState: {
    activeView: string,
    focusEntityId: string | null,
    zoom: "wall" | "room" | "detail",
    lastViewedAt: string | null
  },

  interfaceExtensions: {
    manifestVersion: string,
    modules: {
      id: string,
      enabled: boolean,
      compatibilityRange: string,
      lastLoadedAt: string | null
    }[]
  },

  createdAt: string,
  updatedAt: string
}
```

---

## 25. Server Architecture

### 25.1 Tech Stack

| Layer | Choice | Rationale |
|---|---|---|
| Runtime | Node.js 20+ ESM | No framework needed |
| HTTP | Raw `node:http` | Zero deps |
| Static serving | `node:fs` + MIME map | ~30 lines |
| SSE | `node:http` response streaming | Native |
| Filesystem | `node:fs/promises` | All state + context I/O |
| Ollama | Native `fetch` | Proxy + streaming |
| Repo APIs | Native `fetch` | GitHub, GitLab, etc. |
| HTML parse | `HtmlTextExtractor` | Zero-dep pure function |
| Queue | In-memory + `queue.json` + source lanes | Workload-aware multi-lane processor |

**Zero npm dependencies. `node server.js` only.**

### 25.2 API Routes

```
GET  /                          serve index.html
GET  /the-board                 serve board observation route
GET  /api/ollama/models         proxy GET /api/tags
POST /api/ollama/chat           proxy POST /api/chat (streaming forwarded)
GET  /api/sources               read configured inference sources + health
PUT  /api/sources               write source configuration
POST /api/sources/test          test a source and refresh its model list
GET  /api/sources/:id/models    read persisted discovered model list for a source
POST /api/ingest                trigger ingestion pipeline
GET  /api/context               list context file tree
GET  /api/context/:path         read context file
PUT  /api/context/:path         write context file
GET  /api/personas              list all personas
GET  /api/personas/:id          read persona files (profile, memory, etc.)
PUT  /api/personas/:id/:file    write persona file
GET  /api/state                 read AppState
PUT  /api/state                 write AppState (partial patch supported)
GET  /api/timeline              read timeline items
PUT  /api/timeline/:id          update timeline item state
GET  /api/queue                 read inference queue
POST /api/queue                 enqueue job
PUT  /api/queue/:id             update job (approve, skip, reorder)
GET  /api/board                 read aggregated board state
POST /api/notes/quick           submit a Quick Note to Octet
GET  /api/meetings/:id          read meeting directory listing
GET  /api/meetings/:id/:file    read meeting file
GET  /api/events                SSE stream
POST /api/folder-read           process webkitdirectory file batch
```

### 25.3 SSE Event Catalogue

```javascript
// Phase & state
{ type: "phase:change",         data: { phase, label } }
{ type: "state:updated",        data: { updatedAt } }

// Ingestion
{ type: "ingest:source",        data: { source, status } }
{ type: "ingest:file",          data: { path, status } }
{ type: "ingest:complete",      data: {} }

// Agent activity
{ type: "agent:start",          data: { agentId, task, queueId } }
{ type: "agent:token",          data: { agentId, token, queueId } }
{ type: "agent:complete",       data: { agentId, queueId } }
{ type: "agent:error",          data: { agentId, error, queueId } }

// Personas
{ type: "persona:created",      data: { personaId } }
{ type: "persona:updated",      data: { personaId, file } }
{ type: "memory:updated",       data: { personaId } }

// Meetings
{ type: "meeting:start",        data: { meetingId } }
{ type: "meeting:message",      data: { meetingId, personaId, text, final } }
{ type: "meeting:agenda-item",  data: { meetingId, item, index } }
{ type: "meeting:complete",     data: { meetingId } }

// Timeline
{ type: "timeline:state",       data: { itemId, newState } }
{ type: "timeline:item-added",  data: { item } }

// Queue
{ type: "queue:enqueued",       data: { job } }
{ type: "queue:started",        data: { jobId } }
{ type: "queue:progress",       data: { jobId, percent } }
{ type: "queue:completed",      data: { jobId } }
{ type: "queue:failed",         data: { jobId, error } }
{ type: "queue:approval",       data: { jobId, label } }
{ type: "queue:assigned",       data: { jobId, sourceId, role, model } }

// Sources
{ type: "source:status",        data: { sourceId, status, latencyMs } }
{ type: "source:models",        data: { sourceId, models } }

// Board + Quick Note
{ type: "quick-note:received",  data: { noteId } }
{ type: "quick-note:applied",   data: { noteId, contextVersion } }
{ type: "board:focus",          data: { viewId, entityId } }

// Octet
{ type: "octet:feed",           data: { entry } }
{ type: "pulse:tick",           data: { action, requiresApproval } }
{ type: "vacancy:detected",     data: VacancyReport }
```

---

## 26. Prompt Architecture

All prompts in `/prompts/` as `.md` with YAML frontmatter. Loaded at runtime. `{{variable}}` template interpolation.

```
/prompts/
  # Ingestion
  analyst-context.md            IngestBuffer → ContextObject + context files
  analyst-fetchqueue.md         Seed data → FetchQueue of additional files

  # Persona generation
  persona-ceo.md                CEO from org context
  persona-cofounder.md          Co-founder (domain-parameterized)
  persona-generic.md            Any role, context + role title
  persona-memory-init.md        Initialize memory.md after generation
  persona-knowledge-init.md     Initialize knowledge.md after generation
  persona-memory-update.md      Post-meeting memory synthesis
  persona-relationships-update.md  Update relationships after interaction

  # Org structure
  architect-org.md              OrgManifest from triumvirate + context
  vacancy-scan.md               Identify org coverage gaps
  role-mandate-generate.md      Write role mandate from title + team

  # Octet
  octet-system.md               Octet meta-agent base prompt
  octet-pulse.md                Pulse tick: next priority action
  octet-timeline-advance.md     Propose new PLANNED timeline items
  octet-timeline-transition.md  Write completion summary for state change
  octet-context-index.md        Regenerate _index.md after context update

  # Meetings
  meeting-agenda.md             Structure rough draft → formal agenda
  meeting-suggest.md            Suggest participants + documents for type
  meeting-facilitator.md        Octet in-meeting facilitation turns
  meeting-participant.md        Per-persona meeting participation
  meeting-rolling-summary.md    Rolling context compression
  meeting-post-synthesis.md     Post-meeting summary + action items
  meeting-action-extract.md     Extract structured action items

  # 1:1
  1on1-participant.md           Single-persona 1:1 conversation

  # Artifacts
  artifact-domain-plan.md       Persona domain plan generation
  artifact-executive-plan.md    CEO synthesis: roadmap + exec summary
```

**Context declaration per prompt (frontmatter):**
```yaml
---
prompt: meeting-participant
context-depth:
  context/_index.md: h1
  persona/{{personaId}}/system-prompt.md: full
  persona/{{personaId}}/memory.md: h3
  persona/{{personaId}}/knowledge.md: h2
  persona/{{personaId}}/relationships.md: h2
  meeting/{{meetingId}}/agenda.md: full
token-budget: 5000
priority-order: [system-prompt, memory, knowledge, agenda, index, relationships]
---
```

---

## 27. Repo Structure

```
octet-box/
  server.js                          # entry point: starts HTTP server
  server/
    router.js                        # request dispatch
    sseEmitter.js                    # SSE broadcast to all clients
    stateManager.js                  # read/write state.json
    ollamaClient.js                  # Ollama API wrapper (streaming-capable)
    promptLoader.js                  # load + interpolate prompts
    sourceRegistry.js                # persisted source config + health + model discovery
    inferenceQueue.js                # workload-aware queue + dynamic source lanes
    runLogger.js                     # JSONL audit log
    ingestors/
      htmlTextExtractor.js           # zero-dep pure function
      websiteIngestor.js
      githubIngestor.js              # iterative FetchQueue traversal
      gitlabIngestor.js
      bitbucketIngestor.js
      codebergIngestor.js
      folderIngestor.js              # webkitdirectory handler
      repoDetector.js
    phases/
      phase1-ingest.js
      phase2-founding.js
      phase3-org.js
      phase4-growth.js
    agents/
      contextPackager.js             # context package assembly (§9)
      analyst.js
      architect.js
      personaGenerator.js
      memoryManager.js               # memory read/write/update
      octetAgent.js                  # Octet + pulse loop
      meetingOrchestrator.js         # meeting turn management
      timelineManager.js             # state transitions + synthesis
    inferenceAdapter.js              # unified API wrapper (all sources)
    inferenceAdapters/
      ollamaAdapter.js
      openRouterAdapter.js
      openAIAdapter.js
      anthropicAdapter.js
    templateFiller.js                # template engine: static + infer blocks
    migrations/
      index.js                       # migration chain runner
      v0.4-v0.5.js
    coderFileGenerator.js            # persona CLAUDE.md + AGENTS.md via ScaffoldManager
    scaffoldManager.js               # directory + artifact scaffold generation (§23)
    fileWatcher.js                   # fs.watch wrapper → re-integration detection
  public/
    index.html
    the-board.html
    docs/
      style-guide.html               # design tokens + layout usage
      interface-api.html             # extension API + compatibility guide
    css/
      main.css                       # CSS vars, reset, layout grid
      components.css                 # cards, panels, forms, tags
      animations.css                 # pulse, transitions, typing, shimmer
      board.css                      # billboard and focus-board layouts
      org-canvas.css                 # tree layout, vacancy cards
      meeting-room.css               # chat layout, message bubbles
      timeline.css                   # three-state timeline
      queue.css                      # queue panel
    js/
      app.js                         # init, SSE listener, view router
      extensions/
        adapter.js                   # stable abstraction filter for custom modules
        registry.js                  # built-in + org module registration
      views/
        setup.js
        ingestionMonitor.js
        contextExplorer.js
        foundingCouncil.js
        orgCanvas.js
        meetingBuilder.js
        meetingRoom.js
        oneOnOne.js
        theBoard.js
        boardFocus.js
        timeline.js
        artifactViewer.js
      components/
        personaCard.js
        fileTree.js
        octetFeed.js
        phaseProgress.js
        modelSelector.js
        queuePanel.js
        quickNoteModal.js
        meetingPlanBar.js            # floating attachment bar
        contextAttach.js             # ⊕ attach button + toast
        memoryViewer.js              # persona memory tabs
        timelineItem.js
      state/
        store.js                     # client-side state mirror
        actions.js
        sseHandler.js                # maps SSE events → store updates
  prompts/
    defaults/                        # application-shipped prompts (see §25)
  templates/                         # versioned file templates (see §22, §23)
    personas/
    meetings/
    timeline/
    context/
    artifacts/
    scaffolding/                     # CLAUDE.md + AGENTS.md templates per dir type (§23.4)
  octet-box-data/                    # org data root — NEVER in git, NEVER touched by app updates
    state.json
    context/
    personas/
    meetings/
    timeline/
    artifacts/
    notes/
    queue/
    config/
      sources.json               # API keys + source config (never in git)
    prompts-override/              # user-customized prompts
    templates-override/            # user-customized templates
    coders/                        # CLAUDE.md + AGENTS.md per coder persona
    interface-modules/             # org-specific UI modules behind adapter boundary
  .gitignore
  .env.example                       # DATA_DIR, PORT, PULSE_INTERVAL (no API keys — those go in sources.json)
  package.json                       # { "type": "module" }, scripts only, no deps
  README.md
  LICENSE
```

---

## 28. MVP Scope

### In Scope (v0.1 Prototype)

**New in v0.8:**
- [x] Agent Integration Scaffolding: CLAUDE.md + AGENTS.md at every data directory level
- [x] Artifact-level scaffold files with quality assessment + enhancement guidance
- [x] ScaffoldManager module: auto-generates on directory/artifact creation
- [x] File watcher: detects external changes, queues re-sync, validates schema
- [x] Scaffold viewer in Context Explorer with [Export as .zip ↗]
- [x] Root `octet-box-data/CLAUDE.md` as master orientation document
- [x] Re-integration contract: frontmatter validation + heading structure enforcement

**New in v0.5/v0.6:**
- [x] Autonomous/Board Meeting toggle (not a meeting type)
- [x] [Inject Note] available in all autonomous meetings
- [x] File segregation: `octet-box/` vs `octet-box-data/` hard boundary
- [x] State schema versioning + migration chain
- [x] Prompt user-override system (`prompts-override/`)
- [x] Inference source management: Local Ollama, Networked Ollama, OpenRouter, OpenAI, Anthropic
- [x] Multiple `thinking` and `coding` source records with persisted model selection
- [x] Per-persona model + source assignment with fallback
- [x] `inferenceAdapter.js` unified API wrapper
- [x] Workload-aware queue with dynamic source lanes
- [x] Cost tracking for cloud API sources
- [x] Persona processing states: `ready`, `processing`, `queued`, `inactive`
- [x] Processing animation + prerequisite queue gating
- [x] Coder persona toggle + CLAUDE.md + AGENTS.md generation
- [x] Coder file re-generation on memory/profile update (debounced)
- [x] Template system: versioned `.tmpl` files, `TemplateFiller` module
- [x] Template user overrides (`templates-override/`)
- [x] Template version tracking in generated file frontmatter
- [x] Stale template detection + bulk regeneration
- [x] `/the-board` billboard route with drill-down views
- [x] Quick Note overlay routed to Octet and applied to pending job context
- [x] Interface extension kit with in-app style guide and API docs


**Infrastructure**
- [x] Zero-dep Node.js HTTP server
- [x] Pure HTML/CSS/JS frontend, ESM modules
- [x] Thinking/coding source auto-detection + per-source model discovery + auto-retry
- [x] SSE real-time event stream
- [x] AppState persistence + resume on reload

**Ingestion**
- [x] All 5 input fields + validation
- [x] HtmlTextExtractor (zero-dep)
- [x] GitHub ingestion (iterative ANALYST FetchQueue)
- [x] Local folder ingestion (`webkitdirectory`)
- [x] ANALYST synthesis → structured context file tree
- [x] Context Explorer (browse + edit + re-ingest)

**Context Packaging**
- [x] Markdown heading-based content extraction
- [x] ContextPackager (frontmatter-declared, token-budgeted)
- [x] `_index.md` TOC generation and maintenance
- [x] PackageManifest logging per agent call

**Personas**
- [x] Founding Council generation (CEO + 2 co-founders)
- [x] Full persona file structure (`/personas/{id}/`)
- [x] Persona card UI with all action buttons
- [x] Persona edit interface
- [x] Persona add (manual + generated) + deactivate + clone

**Persona Memory**
- [x] Memory/knowledge/relationships init on persona creation
- [x] Memory viewer (all 4 tabs, editable)
- [x] Memory update after meetings
- [x] Memory inclusion in agent context packages

**Octet**
- [x] Octet meta-agent with org ground truth
- [x] Octet Pulse heartbeat loop
- [x] Autonomy Toggle
- [x] Octet Feed (right panel)

**Org Canvas**
- [x] Department / Team / Role structure
- [x] Org Canvas (tree view + list view)
- [x] Vacancy cards + fill flow (existing or generate)
- [x] Vacancy scan (Octet-driven)

**Meeting System**
- [x] Unified Meeting Builder
- [x] Meeting type selector + Octet participant/doc suggestions
- [x] Agenda generation from rough draft
- [x] Meeting Room — Board Meeting mode
- [x] Meeting Room — Autonomous mode
- [x] Progressive context management (rolling summary)
- [x] Meeting output files (transcript, summary, action items)

**1:1 Interface**
- [x] 1:1 launch from persona card
- [x] Persona-context-only package
- [x] Escalate to meeting builder
- [x] 1:1 transcript → persona memory

**Timeline**
- [x] Three-state timeline model
- [x] `_timeline.md` auto-maintained by Octet
- [x] Timeline interface (all views + filtering)
- [x] Octet state transitions

**Inference Queue**
- [x] Multi-lane queue processor grouped by `thinking` and `coding`
- [x] Queue panel (right panel)
- [x] Priority system (1/2/3)
- [x] Pause/resume + approve/skip controls
- [x] Queue persistence (`queue.json`)

**Board Observability**
- [x] `/the-board` route with fullscreen and quick-dashboard modes
- [x] Work grouped by team, role, and active task
- [x] Click-through focus views for meetings, workstreams, and queue lanes
- [x] Quick Note overlay modal routed through Octet

**Interface Extension System**
- [x] Built-in workflow reference modules for common startup operations
- [x] Stable adapter boundary for org-specific interface modules
- [x] In-app style guide and interface API documentation

**Universal Attachment**
- [x] `⊕` attach button on all context files
- [x] `⊕` attach button on all persona cards
- [x] Floating Meeting Plan bar (all views)
- [x] Multiple meeting plans

### Out of Scope for v0.1
- GitLab / Bitbucket / Codeberg (stubbed, GitHub only)
- Full authoring parity for every core workspace on mobile
- Multi-user / shared sessions
- Cloud-only operation with no Ollama-capable source available
- Export to external tools
- Custom Octet system prompt editing (v0.2)

---

## 29. Definition of Done

1. `node server.js` starts on port 3000, zero npm installs, zero errors
2. Ollama auto-detected on page load; models list populates
3. GitHub URL only → complete ingestion → structured context file tree with frontmatter
4. Context Explorer: files browsable, editable, saveable, re-ingestable
5. Founding Council: 3 context-appropriate personas with all 6 files initialized
6. Memory viewer shows all 4 tabs with generated content per persona
7. Org Canvas shows department/team/role tree with at least 1 vacancy
8. Meeting Builder: rough text → structured agenda via Octet; participant suggestions appear
9. Board Meeting (2 personas + user) → transcript + action items written to disk
10. Autonomous Meeting (2 personas, no user input) → completes, memory updated
11. 1:1 with any persona → conversation grounded in their memory/knowledge
12. Timeline shows completed/active/planned items; Octet advances state after meeting
13. Inference Queue shows all jobs; pause/resume works; approval gate works
14. `⊕` button on any context file or persona → appears in floating Meeting Plan bar
15. Full cold-start to active Org Canvas in under 15 minutes on a 7B model
16. Page reload resumes all state correctly from `state.json`
17. `octet-box-data/` is a sibling to `octet-box/`; `git pull` on app dir does not alter org data
18. State schema migration runs automatically on version mismatch; backup created
19. Inference source config supports multiple `thinking` and `coding` source records; selected models and auth are stored only in `sources.json` (data dir)
20. Queue scales across healthy source lanes; queue panel shows assigned source, workload role, and model per running job
21. Saving an edited persona transitions card to `processing` state; queue gates dependent jobs
22. Enabling Coder on a persona generates `CLAUDE.md` + `AGENTS.md` in both persona dir and `/coders/{id}/`
23. Template version recorded in every generated file's frontmatter; stale badge shown in UI when template updated
24. Meeting canvas shows Consensus Mode label and live Consensus Meter during decision-active exchanges
25. DecisionRecord written to `decisions.md` after every committed decision; referenced in timeline and persona memory
26. Dissent recorded in DecisionRecord and flagged as ReviewTrigger; surfaces in Octet Feed at next relevant milestone
27. Post-meeting strategic alignment synthesis queued and run after every meeting; output appended to `summary.md`
28. StrategicTension items accumulate in AppState and surface in Octet Feed when threshold reached
29. `octet-box-data/CLAUDE.md` generated on first run; accurately describes full data tree structure
30. Every persona directory contains `CLAUDE.md` + `AGENTS.md`; content reflects current memory and org context
31. Every artifact directory contains `CLAUDE.md` + `AGENTS.md`; quality assessment section is inference-filled
32. File watcher detects external edits to `octet-box-data/`; Octet Feed shows "External changes detected" with re-sync option
33. Scaffold viewer in Context Explorer shows [Export as .zip ↗]; exported zip opens in IDE with CLAUDE.md readable by agent
34. First-run source setup provisions a local `coding` Ollama source and a configurable networked `thinking` Ollama source; each source test fetches and persists a live model list
35. Operators can add multiple `thinking` and `coding` sources, each with its own selected model, health state, and concurrency setting
36. Every queue item is labeled `thinking` or `coding` and assigned to a healthy source lane before execution
37. A Quick Note submitted from `/the-board` creates a Priority 1 Octet routing job, updates `context/org/board-notes.md`, and bumps `contextVersion`
38. Pending jobs that have not started recompose their context package against the latest `contextVersion` before execution
39. `/the-board` renders correctly in fullscreen billboard mode and responsive quick-dashboard mode across phone, tablet, laptop, and large-monitor sizes
40. `/the-board` shows who is working on what grouped by workstream, team, role, and active meeting or queue assignment, with clickable drill-down views
41. In-app style guide and interface API docs are available, and org-specific interface modules load only through the documented adapter boundary

---

## 30. Dev Planning Agent Handoff

> You are a senior Node.js architect and vanilla JavaScript frontend engineer building **octet-box** — a locally-hosted startup organization simulator. Full specification is in this document.
>
> Stack: zero-dep Node.js HTTP server (raw `node:http`), pure HTML/CSS/vanilla JS frontend (ES2022 ESM), Ollama local inference, filesystem-based state.
>
> Build sequence — confirm completion and await instruction before each step:
>
> **Step 1 — Server foundation**
> `server.js` (raw http, static serving, MIME map), `router.js`, `sseEmitter.js`, `stateManager.js` (AppState schema §24, partial patch, schema version migration chain), `sourceRegistry.js` (persisted source pools, health checks, model discovery), `inferenceAdapter.js` (unified adapter across all configured sources — see §21.7), `inferenceQueue.js` (workload-aware lanes, priority levels, prerequisite gating, contextVersion refresh, persist to queue.json). Establish `octet-box-data/` sibling directory structure from §20.2 on first run.
>
> **Step 2 — Frontend shell**
> `public/index.html`: full layout from §6 (topbar, left sidebar, main canvas, right panel), CSS variables from §5, CSS grid layout, no content yet. `public/the-board.html`: dedicated observation shell for `/the-board`. `app.js`: SSE listener mapped to store, view router, phase change handler. `state/store.js` + `state/sseHandler.js`.
>
> **Step 3 — Ingestion pipeline + template system**
> `htmlTextExtractor.js`, `repoDetector.js`, `githubIngestor.js` (FetchQueue), `websiteIngestor.js`, `folderIngestor.js`. `contextPackager.js`. `promptLoader.js` (defaults + override lookup). `templateFiller.js` (static + infer block resolution, fillManifest, frontmatter version tagging). Setup view + Ingestion Monitor view.
>
> **Step 4 — Persona system**
> `personaGenerator.js`, `memoryManager.js`, `coderFileGenerator.js`. Phase 2 founding council. Persona processing states (ready/processing/queued/inactive) with shimmer animation and prerequisite queue gating. Persona card + edit interface + memory viewer. Coder persona toggle + CLAUDE.md/AGENTS.md generation via `templateFiller.js`. Founding Council view.
>
> **Step 5 — Octet + Org Canvas**
> `octetAgent.js` (pulse loop, feed entries, approval gating, Quick Note routing). `architect.js` (OrgManifest, vacancy scan). Org Canvas view (tree + list, department/team/role structure, vacancy cards). Timeline foundation (`timelineManager.js`, `_timeline.md` generation, Timeline view). Initial `/the-board` workload and workstream tiles.
>
> **Step 6 — Meeting system**
> `meetingOrchestrator.js` (context package assembly, turn sequencing, rolling summary). Meeting Builder view. Meeting Room view (board + autonomous). 1:1 view. Universal attachment (`contextAttach.js`, `meetingPlanBar.js`). Inference Queue panel. Complete `/the-board` with live meeting wall, queue drill-downs, Quick Note overlay, and interface extension documentation.
>
> Constraints:
> - All server: ESM, `node:` prefix, zero npm deps
> - All frontend: vanilla JS ESM, no frameworks, no CDN
> - All inference calls: through `inferenceAdapter.js` only — never direct fetch to any API
> - All file generation: through `templateFiller.js` only — no ad-hoc string building for generated files
> - All data writes: to `octet-box-data/` only — never to `octet-box/` at runtime
> - All context assembly: through `contextPackager.js` only
> - All state writes: through `stateManager.js` only
> - SOLID principles throughout
> - Every agent call logs a PackageManifest to run log
>
> Begin with Step 1.

---

*Spec v0.8 — Multi-Channel Inference + Board Observability | 2026-03-28 | octet:ois / Ben McNulty*
