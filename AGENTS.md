# Agents Guide - Startup in a Box

This file defines how coding agents should operate in this repository. It is intended to work well for Copilot, Codex, Claude Code, and open-weight or self-hosted coding models.

## Purpose

Build the Startup in a Box prototype as a locally hosted organizational intelligence system with:

- zero runtime dependencies
- a raw Node.js HTTP server
- a pure HTML/CSS/vanilla JS frontend
- local-first storage in a sibling data directory
- full observability of inference, orchestration, and file generation

This repo is the prototype application workstream. The companion website workstream lives in `~/dev/octetois/` and must deploy and verify before Workstream 2 begins.

## Source Of Truth

When instructions conflict, use this precedence order:

1. `octet-startup-box-spec.md` - authoritative product and architecture spec
2. `development-plan.md` - implementation sequence, step sizing, and verification plan
3. `CLAUDE.md` - repo execution context and non-negotiable constraints
4. `AGENTS.md` - agent workflow and collaboration rules
5. direct user instructions

Do not improvise around conflicts in the spec. Surface them and resolve them explicitly.

## Project Identity

Startup in a Box is not a chatbot and not a one-shot generator. It is a persistent founder operating system with:

- context ingestion
- persona generation and memory
- Octet orchestration
- meetings and artifacts
- timeline and queue management

Agents should preserve that product identity in naming, UX, and architecture decisions.

The system now includes:

- multi-channel inference pools for `thinking` and `coding`
- a dedicated `/the-board` observation route
- a stable extension layer for organization-specific interfaces

## Non-Negotiable Architecture

These rules come directly from the spec and must not be bypassed:

1. Zero runtime dependencies. `node server.js` must work without installing runtime packages.
2. Use raw `node:http`, `node:fs`, `node:crypto`, and other Node built-ins with the `node:` prefix.
3. Frontend is pure HTML, CSS, and vanilla JS with ES modules. No React, Vue, Svelte, bundlers, or CDNs.
4. All inference calls must go through `server/inferenceAdapter.js` only.
5. All generated file content must go through `server/templateFiller.js` only.
6. All context package assembly must go through `server/agents/contextPackager.js` only.
7. All `state.json` access must go through `server/stateManager.js` only.
8. All runtime writes must go to the sibling `octet-box-data/` directory only.
9. The application source tree is read-only at runtime.
10. `octet-box-data/` must never be committed to git.
11. Every agent call must log a PackageManifest describing included and excluded context.
12. Source configuration, endpoint health, and model discovery must go through the source registry. Do not hardcode a single inference endpoint or a single global model selector.
13. Queue items must be labeled and routed by workload role: `thinking` or `coding`.
14. Organization-specific UI must compose through the documented extension API and adapter boundary, not by patching core app views for one-off workflows.

If a proposed change would bypass one of these seams, redesign the change instead of patching around the rule.

## Delivery Sequence

Build in the planned order. Do not skip ahead unless the user explicitly asks for a spike or design draft.

1. P0: repo setup
2. Step 1: server foundation
3. Step 2: frontend shell
4. Step 3: ingestion plus templates
5. Step 4: personas
6. Step 5: Octet plus Org Canvas
7. Step 6: meetings plus attachment

Respect dependencies between steps. For example:

- do not add meeting orchestration before queue, state, prompts, and context packaging exist
- do not reintroduce a single-endpoint inference assumption after source pools are established
- do not write generated persona or artifact files outside the template pipeline
- do not add cloud-only inference flows because Ollama is required for MVP
- do not implement org-specific custom interfaces by coupling directly to private store or queue internals

## Working Style

### Before Editing

1. Read `CLAUDE.md`, this file, and the relevant spec sections.
2. Read the surrounding modules before changing code.
3. Check the current implementation step so you do not introduce downstream assumptions too early.
4. Prefer understanding existing patterns over introducing a new abstraction.

### While Editing

1. Keep diffs minimal and scoped to one concern.
2. Fix root causes rather than layering temporary workarounds.
3. Avoid speculative refactors unless they are required to satisfy the task.
4. Preserve public contracts and file layout unless the task requires a deliberate change.
5. Use explicit, descriptive names. Optimize for readability by both humans and models.
6. Add comments only when the intent is not obvious from the code.

### After Editing

1. Verify the affected flow with the narrowest meaningful test.
2. State clearly what was validated and what remains unverified.
3. Update docs when behavior, architecture, or operational expectations changed.

## Coding Standards

### Server

- ESM only
- Node built-ins imported with `node:`
- `async/await` preferred over Promise chains
- Dependency injection at testable seams
- No hidden global state when a module boundary can hold it cleanly
- Route handlers should remain thin; business logic belongs in focused modules

### Frontend

- Vanilla JS modules only
- Progressive enhancement mindset
- Prefer simple DOM composition over client-side framework patterns
- Use SSE for live updates where specified in the spec
- Follow the three-column layout and phase-unlocked view model from the spec
- Treat `/the-board` as a first-class product surface with billboard and responsive quick-dashboard behavior

### CSS

- Use the design tokens and semantic colors defined in spec section 5
- No `transition: all`
- Prefer explicit property transitions and predictable state classes
- Preserve the mission-control visual language: dark, precise, alive

### Markdown, Prompts, And Templates

- Markdown is a core system format, not incidental documentation
- Preserve heading hierarchy because it is used for context packaging
- Prompts belong in `.md` files with YAML frontmatter
- Generated files should include required frontmatter and version metadata when applicable
- Do not hardcode large prompt strings inside runtime modules when they belong in prompt files

## Data And File Boundaries

Treat the app repo and the org data repo boundary as a hard contract.

- Application code lives in this repo
- Runtime state and generated organizational data live in sibling `octet-box-data/`
- Use `DATA_DIR` when path configuration is needed
- First-run initialization may create the required data tree if missing
- Never write meetings, personas, prompts overrides, templates overrides, logs, or state into the app source tree during runtime

## Testing And Verification

Use the project's intended test split:

- unit tests: `node:test`
- integration and E2E: Playwright

For each meaningful change, verify the most relevant parts of the Definition of Done instead of relying on generic smoke checks.

High-priority verification targets include:

1. `node server.js` starts cleanly without runtime installs
2. state persists and reloads correctly
3. queue behavior remains observable and durable
4. SSE events still reflect server-side activity accurately
5. generated files land in the data directory, not the app directory
6. core architectural seams are not bypassed
7. pending jobs pick up the latest `contextVersion` after Quick Notes or context updates

If you cannot run a validation step, say so explicitly.

## Review Checklist

When reviewing or proposing a patch, check for these failure modes first:

1. Runtime dependency creep
2. Direct API calls that bypass `inferenceAdapter.js`
3. Direct generated-file writes that bypass `templateFiller.js`
4. Direct `state.json` reads or writes that bypass `stateManager.js`
5. Direct context assembly that bypasses `contextPackager.js`
6. Runtime writes into the app repo instead of `octet-box-data/`
7. Step-order violations that introduce unusable or untestable features early
8. Broken heading structure or missing frontmatter in markdown-driven files
9. Missing persistence, resume, or audit logging behavior
10. UI changes that ignore the spec's phase model, observability, or dark-system visual language
11. Hardcoded inference routing that ignores `thinking` and `coding` workload pools
12. Org-specific custom UI that bypasses the documented extension adapter boundary

## Collaboration Rules For Agents

1. Read before writing.
2. Do not guess when the spec is explicit.
3. Ask for clarification only when a true ambiguity blocks a correct implementation.
4. Prefer small, reviewable increments over broad rewrites.
5. Keep one logical concern per commit or patch.
6. Do not overwrite user changes you did not make.
7. Do not use destructive git operations unless explicitly requested.
8. Distinguish clearly between implemented behavior and future work.

## Commit And PR Guidance

Use conventional commit prefixes:

- `feat:` new user-visible or system capability
- `fix:` correctness or regression fix
- `refactor:` structural improvement without behavior change
- `docs:` documentation only
- `test:` test-only change
- `chore:` tooling or maintenance

PRs and change summaries should include:

1. what changed
2. why it changed
3. how it was verified
4. any remaining risk or follow-up

## Default Agent Behavior

Unless instructed otherwise, agents working in this repo should optimize for:

- correctness over speed
- explicit seams over convenience shortcuts
- observability over hidden automation
- local-first behavior over hosted assumptions
- enterprise-grade maintainability over cleverness

If a change makes the system harder to audit, harder to resume, or easier to bypass architecturally, it is probably the wrong change.