# CLAUDE.md — MLC Network HUB

AI assistant guide for understanding and working with this codebase.

---

## Project Overview

**mlc-hub** is a static, single-page web application for radio network management and market intelligence. It serves as an internal dashboard for MLC Network, tracking US Spanish-language radio stations, show coverage, CRM contacts, and market dominance data.

**Key characteristics:**
- Zero backend — entirely client-side, no server or database
- Zero build step — raw HTML files served directly
- Zero external dependencies — only CDN-loaded resources (Google Fonts, Leaflet.js)
- All data is hardcoded in JavaScript `const` arrays within the HTML files
- Interface language: Spanish (`lang="es"`)

---

## Repository Structure

```
mlc-hub/
├── index.html               # Entry point — redirects to the main dashboard
├── MLC_Network_HUB_v6.html  # Primary dashboard (current production version)
├── MLC_Radio_Map.html       # Standalone radio coverage map interface
└── README.md                # One-line project description
```

There are no subdirectories, no `src/` folders, no `package.json`, no build artifacts.

---

## Technology Stack

| Layer      | Technology                       |
|------------|----------------------------------|
| Markup     | HTML5 (`<!DOCTYPE html>`)        |
| Styles     | CSS3 with custom properties (`:root` vars) |
| Logic      | Vanilla JavaScript (ES6+)        |
| Maps       | Inline SVG + Leaflet.js 1.9.4 (CDN) |
| Fonts      | Google Fonts — Inter (CDN)       |
| Hosting    | Any static web server or `file://` |

---

## Main File: `MLC_Network_HUB_v6.html`

This is the production file. All application logic, styles, and data live here (~815 lines, ~4.4 MB due to inline SVG map paths).

### Sections (Hash-based routing)

| Section ID    | Label       | Description                                 |
|---------------|-------------|---------------------------------------------|
| `overview`    | Overview    | KPI cards + US map with market dominance    |
| `coverage`    | Cobertura   | Coverage maps (Shows, Services, News, DOM)  |
| `shows`       | Shows       | Per-show station tables (Jimena/Myriam/Chiqui/Alex) |
| `services`    | Servicios   | MLC service catalog (Imagen, Spots)         |
| `contacts`    | Contactos   | CRM table with search and filters           |

Navigation is handled by `onclick="go('section-id')"` buttons in the topbar.

### Data Constants

All data is defined as `const` arrays at the top of the `<script>` block:

| Variable       | Contents                                      |
|----------------|-----------------------------------------------|
| `jimenaData`   | Stations carrying the Jimena show             |
| `myriamData`   | Stations carrying the Myriam show             |
| `chiquiData`   | Stations carrying the Chiqui show             |
| `alexData`     | Stations carrying the Alex show               |
| `contactsData` | Full CRM: directors, call signs, markets, formats |
| `spotsData`    | Advertising spots/services                    |
| `imagenData`   | Branding/image services                       |
| `dotsData`     | Geo-coordinates for market dominance map dots |

Each station object follows this shape:
```javascript
{
  call: "KBFW",
  freq: "87.9",
  band: "FM",
  name: "La Pantera 87.9 FM",
  market: "Dallas",
  state: "TX",
  director: "Gerardo Lopez",
  owner: "RadioGL",
  phone: "+1 (817) 228-2205",
  email: "gerardo@example.com",
  website: "https://...",
  is_mlc: true,   // true = MLC partner, false = lead
  format: "Regional Mexican",
  notes: ""
}
```

### Key JavaScript Functions

| Function | Purpose |
|---|---|
| `go(id)` | Navigate to a main section |
| `showTab(id, btn)` | Switch show subtabs (Jimena/Myriam/etc.) |
| `covTab(id, btn)` | Switch coverage map panes |
| `buildShowTbl(tbodyId, stations, color, isAlex)` | Render a station table from a data array |
| `buildDots(groupId, dots, colorFn, sizeFn, onClick)` | Render SVG dots on the US map |
| `domColor(p)` | Returns color string based on market dominance percentage |
| `showPanel(d)` | Populate the side info panel with market details |
| `filterCRM()` | Live search/filter the contacts table |
| `setCRMFilter(f, btn)` | Apply preset filters (mlc / lead / phone / email) |

### CSS Design System

All colors and spacing use CSS custom properties defined in `:root`:

```css
--bg: #0f172a        /* page background (dark navy) */
--card: #1e293b      /* card background */
--border: #2d3f5a    /* card borders */
--blue: #3b82f6      /* primary accent */
--green: #22c55e     /* MLC partner / success */
--yellow: #f59e0b    /* lead / warm prospect */
--red: #ef4444       /* alert / warning */
--purple: #a78bfa    /* secondary accent */
--text: #e5e7eb      /* primary text */
--text2: #94a3b8     /* secondary text */
--text3: #64748b     /* muted text */
--radius: 10px       /* standard border radius */
```

Color-coded chips:
- `.chip-mlc` — blue, for confirmed MLC partner stations
- `.chip-lead` — yellow, for prospective stations
- `.chip-ok` — green, for active/verified
- `.chip-soon` — gray, for pending

---

## Development Conventions

### How to Add or Update Data

Data updates are made by editing the JavaScript `const` arrays directly inside the `<script>` block of `MLC_Network_HUB_v6.html`. There is no external data file or API call.

Steps:
1. Open `MLC_Network_HUB_v6.html` in a text editor
2. Find the relevant `const` array (e.g., `contactsData`)
3. Add or modify objects following the existing shape
4. Save and open in a browser to verify

### How to Add a New Section

1. Add a nav `<button class="tab" onclick="go('new-id')">Label</button>` in the topbar
2. Add `<section id="new-id" class="section">...</section>` in `<main>`
3. The `go()` function handles toggling `.active` class

### Versioning Convention

Files are named with a version suffix: `v3`, `v4`, `v5`, `v6`. When making a major update, duplicate the current file and increment the version. Update `index.html` to redirect to the new version.

### Responsive Design

A media query at `@media(max-width:680px)` handles mobile layout. The topbar nav collapses, KPI grid reduces to 2 columns, and map panels stack vertically. Maintain these breakpoints when adding new grid layouts.

---

## No Testing Infrastructure

There are no unit tests, integration tests, or CI/CD pipelines. Testing is manual:
1. Open the HTML file in a browser
2. Navigate through all sections and tabs
3. Verify tables render, maps display dots, search/filter works
4. Check mobile layout at ≤680px viewport

---

## Deployment

No build step is required.

**Local:**
```bash
# Any of these work:
open MLC_Network_HUB_v6.html
python3 -m http.server 8080
npx serve .
```

**Production:** Copy the HTML files to any web server's document root (nginx, Apache, GitHub Pages, Cloudflare Pages, etc.).

The `index.html` redirects to `MLC_Network_HUB_v6.html` via a meta refresh or JS redirect — update this pointer when releasing a new version.

---

## Git Workflow

- Primary branch: `main` (or `master`)
- Feature/AI branches: `claude/<description>-<session-id>`
- Commits typically follow "Add files via upload" or descriptive messages like "Dashboard v6 - ..."
- No hooks, linters, or pre-commit checks are configured

---

## Constraints and Gotchas

- **File size:** The HTML files are large (~4.4 MB) because SVG US map paths are inlined. Avoid adding more inline SVG unless necessary.
- **No persistence:** Any data a user types (search filters, etc.) is lost on page reload. Do not assume localStorage or cookies are available.
- **CDN dependency:** The app requires an internet connection for Google Fonts and Leaflet. It will still render but look degraded if CDNs are unreachable.
- **Spanish UI:** All user-visible text should remain in Spanish. Variable names and code comments may be in English.
- **No module system:** There is no `import`/`export`. All functions and data are global within the single `<script>` block.
- **Inline styles:** Prefer CSS classes over inline `style=""` attributes to maintain consistency with the design system.
