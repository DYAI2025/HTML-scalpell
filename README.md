# Architecture Scalpell

**Architecture Scalpell** is a lightweight, single-file interactive visualizer and documentation engine for backend architectures. It allows you to map out services (nodes), user flows, and API endpoints, and generate arc42-compliant markdown documentation automatically.

## Features

- **Interactive Canvas**: Drag and drop nodes, draw connections (edges), and organize your system visually.
- **Generic System**: Configurable project metadata (Name, Repo, Goals, Constraints, Glossary).
- **Dynamic Node Types**: Automatically styles nodes based on their type.
- **API Reference**: Document endpoints with methods, paths, descriptions, parameters, and JSON schemas (Request/Response).
- **User Flow Mapping**: Define step-by-step runtime views for each module.
- **arc42 Export**: One-click generation of a comprehensive 9-section documentation snapshot including Mermaid diagrams.
- **Change Tracking**: Records all edits and can generate a "Development Brief" for coding agents.
- **Portable**: Zero dependencies. Works as a single HTML file.

## Usage

1. Open `index.html` in any modern browser.
2. Use **EDIT MODE** to add or modify nodes and connections.
3. Configure your project in the **SETTINGS** modal.
4. Use **EXPORT DATA** to save your work as JSON.
5. Use **EXPORT HTML** to bake the current state into a new, shareable HTML file.
6. Use **EXPORT-ALL-STATES** to generate the full arc42 documentation.

## Technical Snapshot

- **Frontend**: Vanilla HTML5, CSS3 (Glassmorphism), SVG (Connections).
- **Logic**: Vanilla JavaScript.
- **Storage**: Browser LocalStorage + JSON Export/Import.
- **Documentation Format**: Markdown + Mermaid.

---

*Part of the DYAI2025 Toolset.*
