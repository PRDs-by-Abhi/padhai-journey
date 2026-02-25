# AGENTS.md

## Cursor Cloud specific instructions

This is a **documentation-only** repository — static HTML, CSS, JS (inline), and Markdown. There are no build steps, no package managers, no backend services, no databases, and no dependencies to install.

### Running the site locally

Serve the repo root with any static HTTP server. Example:

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080/user-journey-flowchart.html` (or `http://localhost:8080/docs/index.html`).

The HTML file is self-contained (inline CSS/JS, Mermaid loaded via CDN) and also works via `file://`, but a local server avoids CORS issues with CDN scripts.

### Lint / Test / Build

- **No linter, test framework, or build system exists** in this repo.
- Validation is limited to opening the HTML in a browser and verifying the interactive flowcharts, navigation tabs, message directory filters, and template library render correctly.

### File structure

| File | Purpose |
|------|---------|
| `user-journey-flowchart.html` | Primary source; edit this file |
| `docs/index.html` | GitHub Pages copy; update via `cp user-journey-flowchart.html docs/index.html` |
| `docs/PRD_TICKETS_REWRITTEN.md` | Detailed PRD tickets for dev tracker |
| `GITHUB_PAGES_SETUP.md` | Instructions for publishing to GitHub Pages |
