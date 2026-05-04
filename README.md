# Architecture Scalpell

**Architecture Scalpell** is a single-file architecture editor for backend systems. It combines:
- a visual canvas (nodes and edges),
- structured module data (APIs, notes, flow steps),
- and documentation export (development briefs + arc42-style architecture snapshots).

The tool is intentionally dependency-light and runs directly in the browser from `index.html`.

## Core Goals

- Keep architecture knowledge close to implementation discussions.
- Capture and review change intent before coding begins.
- Export practical artifacts for AI coding agents and human developers.
- Keep architecture decisions consistent over time.

---

## Feature Overview

## 1) Interactive Architecture Canvas

You can model your system with draggable nodes and directed edges.

### Node Modeling
Each node includes:
- basic metadata (title, subtype/icon),
- detailed notes,
- user flow steps,
- API endpoint specifications.

### Edge Modeling
Edges encode relationships and runtime/data flows between modules.
Different edge styles allow visual distinction for primary, secondary, tertiary, and API-related interactions.

---

## 2) Edit Mode + Change Tracking

`EDIT MODE` activates structural edits. Every meaningful edit is tracked in a `changes` list with metadata such as:
- change type (`add`, `modify`, `delete`),
- target (`node`, `api`, `flow`, `notes`, `tags`, etc.),
- affected node,
- before/after payload,
- and human-readable description.

### Why this matters
This acts as the basis for:
- development brief generation,
- review visibility,
- and later consistency checks.

---

## 3) Development Brief Generator

The **EXPORT BRIEF** modal provides a 3-step flow:
1. Review tracked changes
2. Generate + validate a development brief
3. Export markdown

### LLM-first with local fallback
- Primary path: brief generation via LLM (`callLLM`) **when an LLM proxy/backend is configured**.
- Recommended setup: run a small server-side proxy that holds the Anthropic API key and forwards requests from the browser. This avoids exposing secrets in client-side code and avoids the auth/CORS issues that typically occur when calling the provider directly from a static `index.html` page.
- Fallback path: local deterministic generator (`generateLocalBrief`) if LLM access is unavailable or fails.
- If you open the app directly in the browser without proxy/backend configuration, expect the local generator to be used in practice.

### Validation Behavior
Generated output includes validation metadata:
- `issues` for missing/ambiguous information,
- `notes` for assumptions and confirmations.

This reduces vague implementation requests and surfaces missing context before coding begins.

---

## 4) Full Architecture Snapshot Export

`⊞ EXPORT-ALL-STATES` produces a broad markdown architecture export including:
- goals,
- constraints,
- context,
- solution strategy,
- building blocks,
- runtime view,
- API reference,
- module catalog,
- glossary.

This is useful for architecture handoffs, audits, and baseline documentation.

---

## 5) Auto-Improve (new)

`AUTO-IMPROVE` evaluates a single high-impact, small-to-medium effort architecture improvement.

### Purpose
Automatically propose one strategic improvement that:
- fits the current architecture,
- creates measurable progress toward project goals,
- avoids impulsive low-quality suggestions,
- and includes risks and mitigations.

### Execution Guardrails
Auto-Improve can be triggered only if:
- the current session has **no manual user changes** (`changes.length === 0`), and
- no Auto-Improve was already generated in the same brief session.

This enforces:
- low contradiction risk with parallel user-driven changes,
- one clear strategic focus per brief.

### Output Structure
The proposal includes:
- title,
- rationale,
- impact vs effort,
- architecture fit,
- implementation plan,
- risks,
- mitigations,
- guardrails,
- and a reusable **Dev Brief payload**.

User-confirmed suggestions can be carried as context for downstream implementation work.

---

## Risk Assessment for Auto-Improve

## Key Risks
1. **Strategic mismatch**: suggestion may optimize local architecture but miss business goals.
2. **Context incompleteness**: repository-only signals may miss external constraints.
3. **Overconfidence risk**: generated recommendation appears authoritative despite assumptions.
4. **Workflow lock-in**: one-shot suggestion might bias later decisions.
5. **Validation friction**: overly strict policy could block normal editing flows.

## Mitigation Measures
1. **Strict eligibility gate**: only before manual edits.
2. **One-shot policy**: one auto-improve per brief session.
3. **Mandatory risk/mitigation section** in output.
4. **Explicit guardrails** stored with suggestion.
5. **Human confirmation checkpoint** before operationalizing into a Dev Brief.
6. **Contradiction review** requirement for subsequent architecture changes against accepted suggestion.

## Recommended Future Hardening
- Add severity-ranked contradiction checks for future changes.
- Add a “defer auto-improve” state instead of forced acceptance/rejection.
- Add traceable decision log with timestamps and reviewer identity.
- Add confidence scoring (e.g., data completeness, dependency clarity, rollout complexity).

---

## Data and Persistence

- In-memory state for live editing.
- Browser LocalStorage for quick persistence (`SAVE` / `LOAD`).
- JSON import/export for transfer and versioning.
- HTML self-export for portable frozen snapshots.

---

## Recommended Usage Workflow

1. Load or model current architecture.
2. (Optional) Run **Auto-Improve** early in session (before manual edits).
3. Perform architectural edits in `EDIT MODE`.
4. Review change list.
5. Generate development brief.
6. Validate, copy/download markdown.
7. Export full architecture snapshot for long-form documentation.

---

## Limitations

- No backend persistence by default.
- LLM generation quality depends on available context.
- Validation is helpful but not a substitute for architectural review.
- Browser-only runtime means no built-in CI enforcement.

---

## Quick Start

1. Open `index.html` in a modern browser.
2. Use `EDIT MODE` for structural changes.
3. Use `SETTINGS` to define project metadata.
4. Use `EXPORT BRIEF` for implementation-ready markdown.
5. Use `EXPORT-ALL-STATES` for complete architecture documentation.

---

## Technical Stack

- HTML5
- CSS3
- Vanilla JavaScript
- SVG
- Browser APIs (LocalStorage, Clipboard, File export)

No build process required.
