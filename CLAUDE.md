# Startup in a Box — Agent Context

## Project Identity

**Startup in a Box** is a fully self-contained, locally-hosted organizational intelligence environment. It transforms raw startup context into a living system of autonomous AI personas that plan, meet, remember, and evolve — powered by local Ollama inference with optional cloud sources.

- **Spec**: `octet-startup-box-spec.md` (v0.8) — authoritative for all implementation decisions
- **Repo**: `github.com/benmcnulty/octet-startup-box` (public, MIT)
- **Parent org**: Octet OIS (`octetois.com`)

## Two-Repo Working Context

This project spans two independent repositories:

- **`~/dev/octetois/`** — live website (Cloudflare Workers). Workstream 1: `/startup-box` page + landing page update.
- **`~/dev/octet-startup-box/`** — this repo. Workstream 2: the prototype application.

Workstream 1 (website) must deploy and verify before Workstream 2 begins.

## Tech Stack

- **Server**: Zero-dep Node.js 20+ (`node:http`, `node:fs`, `node:crypto`). No Express, no npm packages.
- **Frontend**: Pure HTML/CSS/vanilla JS. ES2022 ESM. No React, no bundler, no CDN.
- **Inference**: Multi-channel source pools with Ollama local/networked first, plus optional OpenRouter/OpenAI/Anthropic via a unified adapter and workload-aware routing
- **State**: `state.json` in sibling `octet-box-data/` directory. Markdown files for all context, memory, artifacts.
- **Testing**: `node:test` (unit), `@playwright/test` (integration + E2E). Dev deps only.

## Architectural Constraints (Non-Negotiable)

| Constraint | Rule |
| --- | --- |
| Zero runtime deps | `node server.js` must work with nothing installed |
| Inference routing | All inference through `server/inferenceAdapter.js` exclusively |
| Source management | All endpoint health, model discovery, and pool assignment go through `server/sourceRegistry.js` |
| File generation | All generated content through `server/templateFiller.js` exclusively |
| Data writes | All writes to `octet-box-data/` only. App directory is read-only at runtime |
| Context assembly | All context packages through `server/agents/contextPackager.js` only |
| State writes | All `state.json` access through `server/stateManager.js` only |
| Custom interfaces | Org-specific UI must go through the documented extension layer, not by patching core app views |
| PackageManifest | Every agent call logs what context was included/excluded |
| Data never in git | `octet-box-data/` must never be committed |

## Implementation Sequence

Six steps, each producing a committed, testable increment:

1. **Server Foundation** — server.js, router, SSE, stateManager, sourceRegistry, inferenceAdapter, inferenceQueue, promptLoader
2. **Frontend Shell** — 3-column layout, `/the-board` shell, CSS design system, SSE client, view router, state store
3. **Ingestion + Templates** — htmlTextExtractor, githubIngestor, contextPackager, templateFiller, Setup + Ingestion views
4. **Personas** — personaGenerator, memoryManager, coderFileGenerator, Founding Council, persona cards
5. **Octet + Org Canvas** — octetAgent (Pulse, consensus, Quick Note routing), architect (OrgManifest), timeline, Org Canvas view, board projections
6. **Meetings + Attachment** — meetingOrchestrator, Meeting Room (board + autonomous), 1:1, universal attachment, `/the-board`, queue panel, interface extension docs

## Code Standards

- ESM throughout, `node:` prefix for all builtins
- `const` preferred, `async/await` over Promise chains
- Single responsibility per module, dependencies injected at testable seams
- No `transition: all` in CSS — explicit property lists
- All CSS custom properties from spec §5 (dark palette, semantic state colors)

## Development

```bash
node server.js          # Start (zero deps required)
node --watch server.js  # Dev mode with file watching
node --test tests/unit/**/*.test.js    # Unit tests
npx playwright test     # E2E tests
```
