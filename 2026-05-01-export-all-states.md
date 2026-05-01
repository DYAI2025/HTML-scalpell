# Export-All-States Button Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add an `EXPORT-ALL-STATES` button to the Bazodiac architecture sheet that generates a complete, standardized arc42 + OpenAPI-style markdown snapshot of every node, route, request/response schema, and edge — readable by humans and machines, comparable across snapshots.

**Architecture:** Single-file HTML artifact (`bazodiac-userflow.html`). All node and edge data already lives in two JS arrays (`nodes`, `edges`). The new feature is one button in the header, one async generator function, and reuse of the existing export modal and progress-step UI from `exportChanges()`.

**Tech Stack:** Vanilla HTML + CSS + JS, no build step. The output format follows the **arc42 lite** structure (sections 1–9) with an embedded **OpenAPI-style endpoint reference** for machine readability.

**Working file:** `/mnt/user-data/outputs/bazodiac-userflow.html` (v2 from earlier turn — has working `exportChanges()` and the modal scaffolding)

---

## Format Decision (read first)

The output uses **arc42 lite + OpenAPI-style endpoint table**:
- **arc42 lite** because it is the standard for software architecture documentation, has 9 numbered sections, and is widely understood by both humans and tooling (templates, linters, AI agents).
- **OpenAPI-style endpoint table** because route/method/schema is the universal contract format and copy-pasteable into Swagger/Redoc/Postman.

Every snapshot will have the **same section order, same heading levels, same table columns** so two snapshots can be diffed mechanically.

---

## Section Order (immutable contract)

1. Header block (project name, generated-at, repo, domain, counts)
2. Table of Contents
3. Introduction & Goals + Quality Goals table
4. Constraints (technical + design)
5. Context & Scope (mermaid `graph LR`)
6. Solution Strategy
7. Building Block View (mermaid `flowchart TD` with subgraphs by node type)
8. Runtime View — User Flows (one numbered list per node)
9. API Reference
   - 9a. Endpoint Index (single table: method, path, module, description)
   - 9b. Endpoint Details (per-module sections with parameters table + JSON schemas)
10. Module Catalogue (per-node fact table + notes)
11. Glossary

This order is the contract. Do not reorder.

---

## Task 1: Add the button to the header

**Files:**
- Modify: `bazodiac-userflow.html` — header right-side button group

**Step 1: Locate the header**

Find the `<div class="header-right">` block. It already contains the `EXPORT CHANGES` button.

**Step 2: Insert the new button before `EXPORT CHANGES`**

```html
<button class="export-btn alt" id="export-arch-btn" onclick="exportFullArchitecture()">
  ⊞ EXPORT-ALL-STATES
</button>
```

**Step 3: Verify the `.export-btn.alt` style exists in CSS**

Search for `.export-btn.alt`. If absent, add:

```css
.export-btn.alt {
  background: rgba(100,140,200,0.08);
  border-color: rgba(140,170,210,0.4);
  color: #a8c0e0;
}
.export-btn.alt:hover {
  background: rgba(100,140,200,0.18);
  color: #d8e4f4;
  border-color: rgba(160,190,230,0.7);
}
```

**Step 4: Open the file in a browser, verify button appears, has blue tint, hover works**

Expected: Button visible left of `EXPORT CHANGES`, no console errors, click does nothing yet (function not defined — that's fine for now).

**Step 5: Stage**

This is a single-file artifact, no git. Save and move on.

---

## Task 2: Add the `exportFullArchitecture()` entry point

**Files:**
- Modify: `bazodiac-userflow.html` — `<script>` block, after `exportChanges()` definition

**Step 1: Add the function**

```javascript
async function exportFullArchitecture() {
  document.getElementById('export-modal-title').textContent = 'Full Architecture State';
  document.getElementById('export-modal-subtitle').textContent = 'Complete documentation snapshot';

  const totalEndpoints = nodes.reduce((s,n) => s + (n.detail.apis?.length || 0), 0);
  document.getElementById('changes-summary').innerHTML =
    `<strong>${nodes.length}</strong> nodes · <strong>${totalEndpoints}</strong> endpoints · <strong>${edges.length}</strong> edges`;

  document.getElementById('export-modal').classList.add('open');
  document.getElementById('export-loading').style.display = 'flex';
  document.getElementById('export-md').style.display = 'none';

  const steps = [
    'enumerating modules',
    'collecting API routes',
    'extracting request/response schemas',
    'mapping user-flow edges',
    'building module index',
    'rendering arc42-aligned markdown',
    'finalizing snapshot'
  ];
  document.getElementById('check-list').innerHTML =
    steps.map(s => `<div class="check-list-item">${s}</div>`).join('');

  await runWithProgress(steps);

  const md = generateFullArchitectureMd();
  document.getElementById('export-loading').style.display = 'none';
  const ta = document.getElementById('export-md');
  ta.style.display = 'block';
  ta.value = md;
  document.getElementById('export-meta').textContent =
    `${md.split('\n').length} lines · ${md.length} chars · arc42 + OpenAPI-style · ${new Date().toLocaleString()}`;
  currentExportFilename = `bazodiac-architecture-${new Date().toISOString().slice(0,10)}.md`;
}
```

**Step 2: Add a temporary stub for `generateFullArchitectureMd`**

```javascript
function generateFullArchitectureMd() {
  return '# placeholder\n\nFilling in next tasks.\n';
}
```

**Step 3: Verify**

Click the button. Expected: modal opens, progress runs through 7 steps, then placeholder markdown shows. Copy/download should already work (they're shared with `exportChanges`).

---

## Task 3: Implement section 1 — Header block

**Files:**
- Modify: `bazodiac-userflow.html` — replace `generateFullArchitectureMd` body

**Step 1: Replace the stub**

```javascript
function generateFullArchitectureMd() {
  const ts = new Date().toISOString();
  const totalEndpoints = nodes.reduce((s,n) => s + (n.detail.apis?.length || 0), 0);

  let md = '';
  md += `# Architecture Documentation — ${PROJECT.name}\n\n`;
  md += `> **Snapshot:** ${ts}  \n`;
  md += `> **Repository:** \`${PROJECT.repo}\`  \n`;
  md += `> **Domain:** \`${PROJECT.domain}\`  \n`;
  md += `> **Format:** arc42 (lite) + OpenAPI-style endpoint reference  \n`;
  md += `> **Modules:** ${nodes.length} · **Endpoints:** ${totalEndpoints} · **Edges:** ${edges.length}\n\n`;

  return md;
}
```

**Step 2: Click the button, verify the header renders correctly**

Expected: ISO timestamp, repo/domain interpolated from `PROJECT` constant, counts match the data arrays.

---

## Task 4: Add section 2 — Table of Contents

**Step 1: Append after the header in `generateFullArchitectureMd`**

```javascript
md += `## Table of Contents\n\n`;
md += `1. [Introduction & Goals](#1-introduction--goals)\n`;
md += `2. [Constraints](#2-constraints)\n`;
md += `3. [Context & Scope](#3-context--scope)\n`;
md += `4. [Solution Strategy](#4-solution-strategy)\n`;
md += `5. [Building Block View](#5-building-block-view)\n`;
md += `6. [Runtime View — User Flows](#6-runtime-view--user-flows)\n`;
md += `7. [API Reference](#7-api-reference)\n`;
md += `8. [Module Catalogue](#8-module-catalogue)\n`;
md += `9. [Glossary](#9-glossary)\n\n`;
```

**Step 2: Verify**

Click button, confirm 9 numbered TOC entries appear.

---

## Task 5: Add section 3 — Introduction & Goals

**Step 1: Append**

```javascript
md += `## 1. Introduction & Goals\n\n`;
md += `${PROJECT.description}\n\n`;
md += `### Quality Goals\n\n`;
md += `| Priority | Quality | Concrete Scenario |\n`;
md += `|----------|---------|-------------------|\n`;
md += `| 1 | Precision | Astronomical calculations accurate to 0.001° via Swiss Ephemeris. |\n`;
md += `| 2 | Reproducibility | Same birth data + date always produces the same Signatur V3. |\n`;
md += `| 3 | Aesthetic Coherence | Dark base, gold accents, Cormorant Garamond — no generic AI styling. |\n`;
md += `| 4 | Ethical Safety | Sensitive content has consent flows, transparency, de-escalation. |\n\n`;
```

**Step 2: Verify table renders, 4 rows, columns aligned**

---

## Task 6: Add section 4 — Constraints

**Step 1: Append**

```javascript
md += `## 2. Constraints\n\n`;
md += `### Technical Constraints\n\n`;
PROJECT.stack.forEach(s => md += `- ${s}\n`);
md += `\n### Design Constraints\n\n`;
md += `- Less words, more space, more mystery.\n`;
md += `- Invitation over reassurance, tension over certainty.\n`;
md += `- Palette: dark base (navy, forest, brown) + gold/cream accents.\n`;
md += `- Typography: Cormorant Garamond display, DM Mono / sans-serif body.\n\n`;
```

**Step 2: Verify both subsections present**

---

## Task 7: Add section 5 — Context & Scope (mermaid)

**Step 1: Append**

```javascript
md += `## 3. Context & Scope\n\n`;
md += '```mermaid\n';
md += `graph LR\n`;
md += `  user((User))\n`;
md += `  app[Bazodiac Frontend]\n`;
md += `  api[API Layer]\n`;
md += `  ephem[Swiss Ephemeris<br/>WASM]\n`;
md += `  bafe[BAFE BaZi Engine<br/>Docker]\n`;
md += `  llm[LLM Provider]\n`;
md += `  user --> app\n  app --> api\n  api --> ephem\n  api --> bafe\n  api --> llm\n`;
md += '```\n\n';
```

**Step 2: Paste output into a mermaid renderer (mermaid.live), confirm it renders**

Expected: Valid mermaid, no syntax errors.

---

## Task 8: Add section 6 — Solution Strategy

**Step 1: Append**

```javascript
md += `## 4. Solution Strategy\n\n`;
md += `- **Fusion model:** Western astrology + BaZi + Wu Xing aggregated into a 5-dimensional Signatur (V3).\n`;
md += `- **Quiz ecosystem:** \`scoreQuiz()\` pipeline emits \`ContributionEvent\`s that update the Signatur via \`AFFINITY_MAP\`.\n`;
md += `- **Two engines:** Swiss Ephemeris for Western astro; BAFE for BaZi pillars and luck cycles.\n`;
md += `- **Visual core:** Fusion Ring with dual gravitational system (outer global field + line-based sector forces).\n\n`;
```

---

## Task 9: Add section 7 — Building Block View (auto from `nodes` + `edges`)

**Step 1: Append**

```javascript
md += `## 5. Building Block View\n\n`;
md += '```mermaid\n';
md += `flowchart TD\n`;
const groups = {
  'Entry':    nodes.filter(n => n.type === 'entry'),
  'Astro':    nodes.filter(n => n.type === 'astro'),
  'BaZi':     nodes.filter(n => n.type === 'bazi'),
  'Quiz':     nodes.filter(n => n.type === 'quiz'),
  'Signatur': nodes.filter(n => n.type === 'sig'),
  'Fusion':   nodes.filter(n => n.type === 'fusion'),
  'Services': nodes.filter(n => n.type === 'api'),
};
Object.entries(groups).forEach(([gname, gnodes]) => {
  if(gnodes.length === 0) return;
  md += `  subgraph ${gname.replace(/\s/g,'_')}["${gname}"]\n`;
  gnodes.forEach(n => md += `    ${n.id}["${n.label}"]\n`);
  md += `  end\n`;
});
edges.forEach(e => md += `  ${e.from} --> ${e.to}\n`);
md += '```\n\n';
```

**Step 2: Verify**

- Every node from `nodes` appears in exactly one subgraph
- Every edge from `edges` appears as a `-->` line
- Mermaid is syntactically valid

---

## Task 10: Add section 8 — Runtime View (user flows)

**Step 1: Append**

```javascript
md += `## 6. Runtime View — User Flows\n\n`;
nodes.forEach(n => {
  if(!n.detail.flow || n.detail.flow.length === 0) return;
  md += `### ${n.label}\n\n`;
  md += `*${n.detail.subtitle}*\n\n`;
  n.detail.flow.forEach((step, i) => md += `${i+1}. ${step}\n`);
  md += `\n`;
});
```

**Step 2: Verify**

Every node with a `detail.flow` array becomes a subsection with numbered steps. Nodes without flows are skipped (e.g., the Landing page if it's empty). Confirm count by hand-spot-checking 3 nodes.

---

## Task 11: Add section 9a — Endpoint Index

This is the machine-readable contract surface. Single table, one row per endpoint, sorted by module appearance.

**Step 1: Append**

```javascript
md += `## 7. API Reference\n\n`;
md += `Total endpoints: **${totalEndpoints}**\n\n`;
md += `### Endpoint Index\n\n`;
md += `| Method | Path | Module | Description |\n`;
md += `|--------|------|--------|-------------|\n`;
nodes.forEach(n => {
  (n.detail.apis||[]).forEach(a => {
    md += `| \`${a.method}\` | \`${a.path}\` | ${n.label} | ${a.desc} |\n`;
  });
});
md += `\n`;
```

**Step 2: Verify row count = `totalEndpoints`**

---

## Task 12: Add section 9b — Endpoint Details (per-module, with schemas)

**Step 1: Append**

```javascript
md += `### Endpoint Details\n\n`;
nodes.forEach(n => {
  if(!n.detail.apis || n.detail.apis.length === 0) return;
  md += `#### Module: ${n.label}\n\n`;
  n.detail.apis.forEach(a => {
    md += `##### \`${a.method} ${a.path}\`\n\n`;
    md += `${a.desc}\n\n`;
    md += `**Parameters:**\n\n`;
    if(a.params && a.params.length) {
      md += `| Name | Description |\n|------|-------------|\n`;
      a.params.forEach(p => {
        const m = p.match(/^([^:]+):\s*(.+)$/);
        if(m) md += `| \`${m[1].trim()}\` | ${m[2].trim()} |\n`;
        else  md += `| \`${p}\` | — |\n`;
      });
    } else {
      md += `_None._\n`;
    }
    md += `\n`;
    if(a.requestSchema) {
      md += `**Request schema:**\n\n\`\`\`json\n${a.requestSchema}\n\`\`\`\n\n`;
    }
    if(a.responseSchema) {
      md += `**Response schema:**\n\n\`\`\`json\n${a.responseSchema}\n\`\`\`\n\n`;
    }
  });
});
```

**Step 2: Verify**

- Parameters with `name: description` syntax split into two table columns
- Parameters without `:` show `—` as description
- Endpoints with no params show `_None._`
- Schemas wrapped in fenced JSON blocks

---

## Task 13: Add section 10 — Module Catalogue

**Step 1: Append**

```javascript
md += `## 8. Module Catalogue\n\n`;
nodes.forEach(n => {
  md += `### ${n.label}\n\n`;
  md += `| Field | Value |\n|-------|-------|\n`;
  md += `| **ID** | \`${n.id}\` |\n`;
  md += `| **Type** | \`${n.type}\` |\n`;
  md += `| **Title** | ${n.detail.title} |\n`;
  md += `| **Subtitle** | ${n.detail.subtitle} |\n`;
  md += `| **Tags** | ${(n.detail.tags||[]).map(t=>'`'+t+'`').join(', ') || '—'} |\n`;
  md += `| **Endpoints** | ${(n.detail.apis||[]).length} |\n`;
  md += `| **Inbound edges** | ${edges.filter(e=>e.to===n.id).map(e=>'`'+e.from+'`').join(', ') || '—'} |\n`;
  md += `| **Outbound edges** | ${edges.filter(e=>e.from===n.id).map(e=>'`'+e.to+'`').join(', ') || '—'} |\n\n`;
  if(n.detail.notes) md += `**Notes:** ${n.detail.notes}\n\n`;
});
```

**Step 2: Verify**

Every node appears once, fact table has 8 fields, inbound/outbound counts match by spot-check on `dashboard` (should have many outbound, one inbound from `onboarding`).

---

## Task 14: Add section 11 — Glossary + closing

**Step 1: Append**

```javascript
md += `## 9. Glossary\n\n`;
md += `| Term | Definition |\n|------|------------|\n`;
md += `| **Signatur V3** | 5-dimensional master signal: Resonanz, Schatten, Antrieb, Körper, Beziehung. |\n`;
md += `| **AFFINITY_MAP** | Mapping table connecting quiz answer dimensions to Signatur dimensions. |\n`;
md += `| **ContributionEvent** | An event emitted by \`scoreQuiz()\` that updates a user's Signatur. |\n`;
md += `| **BaZi / 八字** | Chinese four-pillars system: Year, Month, Day, Hour as Heavenly Stem + Earthly Branch. |\n`;
md += `| **Wu Xing / 五行** | Five-elements model: Wood, Fire, Earth, Metal, Water. |\n`;
md += `| **Day Master** | The Heavenly Stem of the Day pillar; identity anchor of a BaZi reading. |\n`;
md += `| **Luck Pillars / 大運** | Ten-year cycles overlaying the natal BaZi chart. |\n`;
md += `| **Fusion Ring** | Core visualization combining Western and BaZi data with dual-gravity dissonance. |\n`;
md += `| **Light / Heavy Dissonance** | Two distinct conflict modes in the Fusion Ring (subtle waves vs collision pulses). |\n`;
md += `| **Swiss Ephemeris** | High-precision astronomical library used for planetary positions. |\n`;
md += `| **BAFE** | BaZi Analysis Framework Engine, deployed via Docker. |\n\n`;
md += `---\n\n`;
md += `*Snapshot generated from the interactive architecture sheet. Re-export after structural changes to keep documentation aligned with implementation.*\n`;

return md;
```

**Step 2: Verify final closing line and `return md` at the end of the function**

---

## Task 15: End-to-end verification

**Step 1: Click `EXPORT-ALL-STATES` button**

Expected: Modal opens → 7 progress steps run → markdown appears.

**Step 2: Visual check the markdown structure**

- [ ] All 9 numbered sections present in correct order
- [ ] Header has timestamp, repo, domain, counts
- [ ] Both mermaid blocks have valid syntax
- [ ] Endpoint index row count = sum of `n.detail.apis.length` for all nodes
- [ ] Module Catalogue has one entry per node
- [ ] Glossary has 11 rows

**Step 3: Copy markdown → paste into a markdown viewer (e.g. typora, vscode preview, github gist)**

Expected: Tables render, mermaid renders if extension enabled, headings collapse correctly, all anchors in TOC resolve.

**Step 4: Download `.md` file**

Expected: Filename pattern `bazodiac-architecture-YYYY-MM-DD.md`. Open in editor, verify byte content matches what was shown.

**Step 5: Diff two snapshots**

After making one trivial edit (e.g. add a route via the edit modal — but DO NOT export changes — just leave it staged), re-export. Diff the two snapshots. Should differ in only:
- Snapshot timestamp line
- Counts in header
- The new endpoint's row + detail block

This proves the format is stable and diffable.

---

## Cross-Cutting Concerns

- [ ] Function `runWithProgress` already exists from `exportChanges` — verify reuse, no duplication.
- [ ] `currentExportFilename` is shared — verify last-clicked button wins (this is intentional, both export functions write to same modal).
- [ ] `escapeHtml` is NOT used in markdown generation — markdown does not need HTML escaping; backticks handle code, pipes need escaping in tables (no current data has pipes — flag if added later).
- [ ] If a future endpoint contains a `|` character in its description, the table will break. Consider replacing `|` with `\|` in the description before insertion. Add this to "remaining risks" but not required for current data.

---

## Output Contract Summary (for Verification)

After Task 15:

### phase
Implements `EXPORT-ALL-STATES` button + `generateFullArchitectureMd` function with 9 arc42 sections + OpenAPI endpoint reference.

### verification
- Button renders: visual check
- Function returns markdown: visual check in modal
- Markdown structure: section spot-check + table row count
- Mermaid blocks: paste into mermaid.live
- Round-trip diff stability: two snapshots, only expected differences

### remaining risks
- Pipe character in any future description/notes will break tables — escape if added.
- Mermaid renderers vary in label parsing — labels with `[`, `]`, `(`, `)` need testing if added.

### confidence
- High for static data (current state)
- Medium for forward-compatibility with arbitrary future content (escaping)

---

*Plan complete.*
