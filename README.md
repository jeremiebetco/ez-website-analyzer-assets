# EZ Website Analyzer

A Python CLI tool for **research-oriented web scraping**. It maps a target site, captures rich page artifacts (markdown, HTML, branding, screenshots), parses CSS design tokens, discovers interactive flows on SPAs, and produces structured analysis reports for auditing, competitive research, and technical documentation.

## Usage notes

When scraping third-party sites, follow standard web crawler etiquette: honor `robots.txt`, respect published rate limits, and review the site operator's crawl policies where applicable.

## How it works

The analyzer uses a **two-phase pipeline** focused on capture and structured analysis:

### Phase 1: Raw capture and design analysis

1. **Mapping (Firecrawl Map)** — Discovers the site's internal link network and total page count.
2. **Rich scraping (Firecrawl Scrape)** — For each selected page, captures markdown, HTML, raw HTML, links, images, branding tokens, and a full-page screenshot.
3. **CSS fetch and parse** — Downloads linked external stylesheets from `rawHtml` and extracts design tokens (colors, font families/sizes, spacing, breakpoints, CSS variables).
4. **Design analysis (OpenAI)** — Produces a structured `design-profile.json` from parsed CSS, branding JSON, page content, and an optional vision pass on the screenshot.

### Phase 2: Interaction discovery and content reveal

5. **SPA loading-shell detection** — Detects near-empty HTML / loading spinners and primes the page with a wait + accessibility-tree capture.
6. **Clickable discovery** — Harvests buttons and links from the rendered HTML and accessibility tree (not screenshot OCR).
7. **Derived actions** — Ranks high-value clickables and produces both:
   - deterministic CSS selector actions (merged with per-domain `CUSTOM_ACTIONS` overrides), and
   - fuzzy word labels for interact fallback.
8. **Interaction driving** — Clicks discovered elements (selector first, word-label fallback), extracts revealed content, and merges it into entry-page markdown.

### Final analysis

9. **Run folder export** — Saves all raw artifacts under `runs/` before final synthesis (so scrape credits are not lost if analysis fails).
10. **Multi-pass analysis (OpenAI)** — Runs one structured analysis per page, then synthesizes a site-wide profile into `DESIGN.md`, `components.json`, and `report.md`.

## Crawl depth

Choose at runtime:

| Mode | Max pages | Description |
|------|-----------|-------------|
| Smart selection | 25 | Homepage + high-value paths (about, pricing, contact, etc.) |
| Full crawl | 80 | Site-wide discovery and scrape |

## Run output

Each run creates a timestamped folder:

```
runs/
  20260703-104530_example.com/
    run-metadata.json
    map/links.json
    scrape/{page-slug}/
      page.md, page.html, raw.html, rendered.html
      links.json, images.json, branding.json, css-tokens.json
      clickables.json, a11y.txt
      css/stylesheet_0.css, css/manifest.json
      metadata.json, screenshot.png
    interact/enrichment.md        # when interactions ran
    analysis/
      design-profile.json         # parsed CSS + vision design profile
      site-profile.json
      pages/{page-slug}.json
    DESIGN.md                     # structured site analysis summary
    components.json
    report.md
```

Historical flat reports in `reports/` are from the previous analyzer version and are not updated by new runs.

The `runs/` directory is tracked in git (via `runs/.gitkeep`), but run output inside it is gitignored.

**Source install:** output under project `runs/`. **Binary install:** output under `~/.ez-web/runs/` unless `runs/` exists beside the executable.

## Setup

Install dependencies:

```bash
pip install -r requirements.txt
```

### Configuration & paths

| | Source install (`./analyzer.sh`) | Binary install (release executable) |
|---|---|---|
| **`.env` locations** | Project root `.env` only | Shell env → `.env` beside binary (gap-fill) → `~/.ez-web/.env` (gap-fill; auto-created on first run) |
| **`runs/` output** | `runs/` at project root (or `RUNS_DIR` override) | `~/.ez-web/runs/` by default; use `runs/` beside the binary if you create that folder |
| **`RUNS_DIR` override** | Any explicit value wins in both modes | Same |

Shell / system environment variables always take highest priority. See `.env.example` for all optional keys.

### Source install

**Option A — `.env` file** (recommended for local dev):

```bash
cp .env.example .env
# then edit .env with your keys
```

**Option B — shell environment:**

```bash
export FIRECRAWL_API_KEY=fc-your_actual_key_here
export OPENAI_API_KEY=sk-your_actual_key_here
```

Required variables:

```
FIRECRAWL_API_KEY=fc-your_actual_key_here
OPENAI_API_KEY=sk-your_actual_key_here
```

For the full list of optional settings with defaults and descriptions, see `.env.example`.

Optional configuration (defaults shown in `.env.example`):

```
SMART_MAX_PAGES=25
FULL_MAX_PAGES=80
OPENAI_MODEL=gpt-4o
RUNS_DIR=runs
SMART_PATH_KEYWORDS=about,services,pricing,contact,team,products
EXCLUDE_PATH_PATTERNS=login,cart,privacy,terms,sitemap,feed
INCLUDE_SUBDOMAINS=false
SCRAPE_BATCH_SIZE=3
MAX_INTERACT_STEPS=3
INTERACT_TIMEOUT=60
INTERACT_SCRAPE_MAX_AGE=0
ENABLE_VISION=true
CSS_FETCH_MAX=10
CSS_FETCH_MAX_BYTES=500000
HIGH_VALUE_LABELS=register,continue,agree,next,submit,accept,finish
LOADING_SHELL_MIN_MARKDOWN=300
LOADING_SHELL_MIN_HTML_CHARS=500
SPA_PRIME_WAIT_MS=3000
```

- `SMART_MAX_PAGES` — max pages for smart selection (default: 25)
- `FULL_MAX_PAGES` — max pages for full crawl (default: 80)
- `OPENAI_MODEL` — model for structured site analysis (default: `gpt-4o`)
- `RUNS_DIR` — output root for run folders (default: `runs/` at project root)
- `SCRAPE_BATCH_SIZE` — parallel batch size for batched scraping
- `HIGH_VALUE_LABELS` — keywords used to rank auto-discovered clickables
- `ENABLE_VISION` — optional screenshot vision pass for Phase 1 design analysis (default: `true`)
- `CSS_FETCH_MAX` — max external stylesheets to fetch per page (default: 10)
- `CSS_FETCH_MAX_BYTES` — max bytes per stylesheet fetch (default: 500000)
- `LOADING_SHELL_MIN_MARKDOWN` — markdown length threshold for SPA shell detection
- `LOADING_SHELL_MIN_HTML_CHARS` — rendered HTML size threshold for SPA shell detection
- `SPA_PRIME_WAIT_MS` — wait time when priming SPA pages before re-capture

**Credit note:** Rich scrape requests multiple formats per page (markdown, HTML, branding, screenshot, etc.). Phase 2 may run additional interact sessions on the entry URL. Full crawl at 80 pages uses significantly more Firecrawl credits than markdown-only scraping.

### Binary install

For the release executable (no Python required):

- **API keys:** shell environment variables, then `.env` beside the binary (gap-fill), then `~/.ez-web/.env` (gap-fill; created from the bundled template on first run if missing).
- **Output:** `~/.ez-web/runs/` by default; create `runs/` beside the executable to write output there instead.
- **Download:** see [Releases](#releases) for platform assets and checksum verification.

## Custom scrape actions

`CUSTOM_ACTIONS` in `src/analyzer.py` are **per-domain overrides/seeds**. The pipeline also derives selector actions at runtime from discovered clickables. Static entries are merged with derived actions.

```python
CUSTOM_ACTIONS = {
    "example.com": [
        {"type": "click", "selector": "button#load-more"},
        {"type": "wait", "milliseconds": 2000},
    ],
}
```

Supported step types: `click` (requires `selector`), `wait` (optional `milliseconds`).

## Releases

Every `v*` tag push triggers [`.github/workflows/release-binaries.yml`](.github/workflows/release-binaries.yml), which publishes standalone CLI binaries and `.sha256` checksums to [ez-website-analyzer-assets](https://github.com/jeremiebetco/ez-website-analyzer-assets/releases).

Download the asset for your platform from [GitHub Releases](https://github.com/jeremiebetco/ez-website-analyzer-assets/releases) on the public assets repository.

| Platform | Asset |
|----------|-------|
| Linux (x86_64) | `ez-website-analyzer-linux-x86_64` |
| Windows (x86_64) | `ez-website-analyzer-windows-x86_64.exe` |
| macOS (Apple Silicon) | `ez-website-analyzer-macos-arm64` |

Each binary ships with a matching `.sha256` checksum file.

### Run a release binary

1. Download the binary for your OS from the latest release.
2. Set your API keys — see [Setup → Configuration & paths](#configuration--paths) for `.env` resolution (shell, binary `.env`, `~/.ez-web/.env`).
3. Run the binary from a terminal:

```bash
# macOS / Linux
chmod +x ez-website-analyzer-macos-arm64   # or your platform asset
./ez-website-analyzer-macos-arm64
```

```bat
ez-website-analyzer-windows-x86_64.exe
```

See [Setup → Configuration & paths](#configuration--paths) for `.env` and `runs/` resolution. Steps above cover download and checksum verification.

### Verify checksum

```bash
# macOS / Linux
shasum -a 256 -c ez-website-analyzer-macos-arm64.sha256
```

```powershell
# Windows (PowerShell)
$expected = (Get-Content ez-website-analyzer-windows-x86_64.exe.sha256).Split(' ')[0]
$actual = (Get-FileHash ez-website-analyzer-windows-x86_64.exe -Algorithm SHA256).Hash.ToLower()
$expected -eq $actual
```

### Cut a release (maintainers)

```bash
git tag v0.1.3
git push origin v0.1.3
```

This triggers a binary release (Linux, Windows, macOS Apple Silicon with SHA-256 checksums) on the public [ez-website-analyzer-assets](https://github.com/jeremiebetco/ez-website-analyzer-assets) repository.

## Run

From the project root:

```bash
./analyzer.sh
```

On Windows:

```bat
analyzer.bat
```

Aliases: `./run.sh` and `run.bat` (same behavior).

Override the Python interpreter if needed:

```bash
PYTHON=python ./analyzer.sh    # Windows-friendly
PYTHON=python3 ./analyzer.sh   # default on macOS/Linux
```

Direct invocation (optional):

```bash
python3 src/analyzer.py
```

The launchers `cd` to the project root, run with unbuffered output (`-u`), and preserve interactive prompts and live logs.

The CLI prompts for:

1. **Target URL** — e.g. `https://example.com`
2. **Crawl depth** — `1` for Smart Selection (25 pages), `2` for Full Crawl (80 pages)
3. **Scrape execution** — `1` sequential, `2` batched parallel

Example session:

```
EZ Website Analyzer
===================

Enter target website URL: https://example.com

Select crawl depth:
  [1] Smart Selection (up to 25 high-value pages)
  [2] Full Crawl (up to 80 pages site-wide)

Enter choice (1 or 2): 1

Select scrape execution:
  [1] Sequential (one page at a time)
  [2] Batched (parallel groups)

Enter choice (1 or 2): 2
Mapping site footprint...
Selected 4 page(s) for scraping...
Scraping pages with rich formats...
Phase 1: capturing CSS and analyzing design...
Phase 2: discovering clickables and driving interactions...
Saving raw scrape artifacts...
Running per-page analysis...
Synthesizing site-wide profile...
Writing analysis deliverables...

Run artifacts saved to: /path/to/runs/20260703-104530_example.com
  DESIGN.md       — structured site analysis summary
  components.json — component inventory
  report.md       — human-readable summary
  scrape/         — raw page artifacts
```

## Primary deliverables

| File | Purpose |
|------|---------|
| `DESIGN.md` | Design tokens, parsed CSS profile, interaction map, components, and site structure notes for research and documentation |
| `components.json` | Flattened component inventory with source URLs |
| `analysis/design-profile.json` | Structured design profile from parsed CSS and optional vision |
| `report.md` | Human-readable summary with site metrics and page inventory |
| `scrape/` | Raw evidence: markdown, HTML, parsed CSS, clickables, branding JSON, screenshots |
