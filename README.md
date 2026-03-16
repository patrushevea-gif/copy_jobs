# AI Exposure of the US Job Market

Analyzing how susceptible every occupation in the US economy is to AI and automation, using data from the Bureau of Labor Statistics [Occupational Outlook Handbook](https://www.bls.gov/ooh/) (OOH).

**Live demo: [joshkale.github.io/jobs](https://joshkale.github.io/jobs/)**

![AI Exposure Treemap](jobs.png)

## What's here

The BLS OOH covers **342 occupations** spanning every sector of the US economy, with detailed data on job duties, work environment, education requirements, pay, and employment projections. We scraped all of it, scored each occupation's AI exposure using an LLM, and built an interactive treemap visualization.

## Data pipeline

1. **Scrape** (`scrape.py`) — Playwright (non-headless, BLS blocks bots) downloads raw HTML for all 342 occupation pages into `html/`.
2. **Parse** (`parse_detail.py`, `process.py`) — BeautifulSoup converts raw HTML into clean Markdown files in `pages/`.
3. **Tabulate** (`make_csv.py`) — Extracts structured fields (pay, education, job count, growth outlook, SOC code) into `occupations.csv`.
4. **Score** (`score.py`) — Sends each occupation's Markdown description to an LLM (Gemini Flash via OpenRouter) with a scoring rubric. Each occupation gets an AI Exposure score from 0-10 with a rationale. Results saved to `scores.json`.
5. **Build site data** (`build_site_data.py`) — Merges CSV stats and AI exposure scores into a compact `site/data.json` for the frontend.
6. **Website** (`site/index.html`) — Interactive treemap visualization where area = employment and color = AI exposure (green to red).

## Key files

| File | Description |
|------|-------------|
| `occupations.json` | Master list of 342 occupations with title, URL, category, slug |
| `occupations.csv` | Summary stats: pay, education, job count, growth projections |
| `scores.json` | AI exposure scores (0-10) with rationales for all 342 occupations |
| `html/` | Raw HTML pages from BLS (source of truth, ~40MB) |
| `pages/` | Clean Markdown versions of each occupation page |
| `site/` | Static website (treemap visualization) |

## AI exposure scoring

Each occupation is scored on a single **AI Exposure** axis from 0 to 10, measuring how much AI will reshape that occupation. The score considers both direct automation (AI doing the work) and indirect effects (AI making workers so productive that fewer are needed).

A key signal is whether the job's work product is fundamentally digital — if the job can be done entirely from a home office on a computer, AI exposure is inherently high. Conversely, jobs requiring physical presence, manual skill, or real-time human interaction have a natural barrier.

**Calibration examples from the dataset:**

| Score | Meaning | Examples |
|-------|---------|---------|
| 0-1 | Minimal | Roofers, janitors, construction laborers |
| 2-3 | Low | Electricians, plumbers, nurses aides, firefighters |
| 4-5 | Moderate | Registered nurses, retail workers, physicians |
| 6-7 | High | Teachers, managers, accountants, engineers |
| 8-9 | Very high | Software developers, paralegals, data analysts, editors |
| 10 | Maximum | Medical transcriptionists |

Average exposure across all 342 occupations: **5.3/10**.

## Visualization

The main visualization is an interactive **treemap** where:
- **Area** of each rectangle is proportional to employment (number of jobs)
- **Color** indicates AI exposure on a green (safe) to red (exposed) scale
- **Layout** groups occupations by BLS category
- **Hover** shows detailed tooltip with pay, jobs, outlook, education, exposure score, and LLM rationale

## Deploy to GitHub Pages (standard branch deploy)

Yes — after these changes you can deploy and open the site in a browser as a static site, without running Python/uv on the server.

- The repository has a GitHub Actions workflow that deploys the `site/` folder to GitHub Pages.
- Push your branch to `master` or `main` and GitHub will publish it.
- For custom branches, open **Actions → Deploy static content to Pages → Run workflow**.

After deployment, open the Pages URL from **Settings → Pages** (or from the workflow output).

If Pages is configured to serve repository root (instead of GitHub Actions artifact), opening the root URL can show README. This repo now includes a root `index.html` that redirects to `site/`, so the interactive app opens by default.

## Setup

### Standard Python deployment (without uv / virtualenv)

```bash
python3 -m pip install -r requirements.txt
```

### Alternative: uv workflow

```bash
uv sync
```

Requires an OpenRouter API key in `.env` only if you run scoring:
```
OPENROUTER_API_KEY=your_key_here
```

## Quick start (recommended after clone)

If you only want to open the project as-is (with current committed data), no scraping/scoring is required:

```bash
cd site
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

## Full pipeline

### Standard Python commands (no uv)

```bash
# Generate Markdown from HTML
python3 process.py

# Generate CSV summary
python3 make_csv.py

# Build website data
python3 build_site_data.py

# Optional: score AI exposure (requires OPENROUTER_API_KEY)
python3 score.py
python3 build_site_data.py

# Serve the site locally
cd site && python3 -m http.server 8000
```

### Same pipeline with uv

```bash
uv run python process.py
uv run python make_csv.py
uv run python build_site_data.py

# Optional: score AI exposure (requires OPENROUTER_API_KEY)
uv run python score.py
uv run python build_site_data.py

cd site && python -m http.server 8000
```

### Optional: scrape data from BLS again

Scraping is optional because `html/` is already committed. If you still want fresh raw pages:

```bash
# standard python
python3 -m playwright install chromium
python3 scrape.py

# or uv
uv run playwright install chromium
uv run python scrape.py
```
