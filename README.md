# Company Sustainability Profile: ESG Evaluation and Stock Performance Insights

## Overview
This project evaluates the sustainability profiles of the world's largest public companies (Forbes Global 2000) and tests whether their ESG (Environmental, Social and Governance) scores are associated with **corporate financial performance** (primary: sales, profit, assets and market value) and **stock performance** (secondary: returns, volatility and drawdowns) over 2023-2025.

ESG scores are built from Tekmon's 25 ESG KPIs, mapped to the GRI Standards and extracted from company sustainability reports with an LLM-assisted, evidence-validated pipeline. The results are shown in a Streamlit dashboard.

**Research question:** Does value creation that is reported in line with GRI-based ESG KPIs go together with resilient performance and sustained growth (H<sub>A</sub>), or does economic growth occur regardless of ESG reporting (H<sub>0</sub>)? Results are reported as associations, not causal claims. The full problem statement, hypotheses, scope and methodology are in [RESEARCH.md](docs/RESEARCH.md).

- [Project Status](#project-status)
- [Documentation](#documentation)
- [Quickstart](#quickstart)
- [Repository Layout](#repository-layout)
- [Tools and Technologies](#tools--technologies)
- [License](#license)

<hr>

## Project Status
**Pilot in progress.** The pilot is a thin end-to-end slice that runs every pipeline stage once on real data before scaling to the full top 100:

- **Companies:** the top 3 of the Forbes Global 2000 2026 list, selected at runtime. No company names or tickers are hard-coded; only the number of companies and the year window are configurable.
- **Period:** fiscal years 2023-2025 for sustainability reports, financials and stock prices.
- **Pipeline:** Forbes list → ticker and website resolution → financials and stock prices → sustainability report discovery → KPI extraction and validation → ESG scoring → analysis-ready tables → dashboard.
- **Purpose:** to prove the pipeline works. Three companies are too few for statistical conclusions, so the pilot dashboard shows data, not correlation results.

The work plan and its issues are in [PILOT_PLAN.md](docs/PILOT_PLAN.md). The decisions behind the pilot are logged in [DESIGN_DECISIONS.md](docs/DESIGN_DECISIONS.md), and decisions still open are tracked in [RESEARCH.md "Open Decisions"](docs/RESEARCH.md#open-decisions).

<hr>

## Documentation

| Document | What it covers |
|---|---|
| [PILOT_PLAN.md](docs/PILOT_PLAN.md) | Pilot work plan: pipeline, key design points, dependencies, the GitHub issues for each step, and verification |
| [RESEARCH.md](docs/RESEARCH.md) | Background, problem statement, hypotheses, scope and constraints, open decisions, and the full methodology (data acquisition, KPI structure, extraction, ESG scoring, analysis workflow) |
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | Mermaid diagrams of the high-level architecture and each pipeline, plus the recommended project structure and its guidelines |
| [DESIGN_DECISIONS.md](docs/DESIGN_DECISIONS.md) | Educational, ADR-style log of every design decision: context, alternatives, concepts and trade-offs |
| [DESIGN_PATTERNS.md](docs/DESIGN_PATTERNS.md) | Design patterns used in the code, where and why |
| [KPI_Breakdown.md](docs/KPI_Breakdown.md) | Tekmon's 25 ESG KPIs mapped to GRI Standards |

<hr>

## Quickstart
> The pipeline is not implemented yet. These steps will be completed as the pilot is built; only the commands below work today.

Requirements: Python 3.13+ and [uv](https://docs.astral.sh/uv/).

```bash
cd esg-profile
uv sync              # install dependencies
uv run esg-profile   # run the CLI entry point (currently a placeholder)
```

Planned for the pilot: API keys in a local `.env` file (a committed `.env.example` will list the required variables), `uv run esg-profile run all` to run the pipeline, and a Streamlit command to open the dashboard.

<hr>

## Repository Layout

```text
esg-profile/
├── README.md          # Project overview and usage (this file)
├── docs/              # Research, architecture, design decisions and patterns, KPI mapping
└── esg-profile/       # Python package (uv project): src/esg_profile/
```

The full target structure, and the reasoning behind it, are in [ARCHITECTURE.md "Recommended Project Structure"](docs/ARCHITECTURE.md#recommended-project-structure) and [DD-011](docs/DESIGN_DECISIONS.md#dd-011-layered-python-package-with-ports-and-adapters).

<hr>

## Tools & Technologies
- Languages: **Python** and **SQL**
    - **Python** - for data acquisition, ingestion, processing, analysis and visualisation
    - **DuckDB** - for relational database management and data storage
- Storage: **Azure Blob Storage** or **Local Storage**
    - **Azure Blob Storage** - for object storage of PDF reports
- Data Sources:
    - **Forbes Global 2000 dataset (2026 list, via Kaggle)** - for top 100 companies (top 3 in the pilot)
    - **SerpApi** - web search API for finding sustainability report PDFs
    - **OpenBB MCP Server** - for historical stock price data and corporate financials
    - **yfinance** - unofficial Yahoo Finance API for Python, for corporate financials and historical stock prices (temporary solution)
    - **Company sustainability reports** - PDFs collected through automated web search
- Visualization:
    - **Streamlit** - for dashboard visualisation
- AI/ML:
    - **Azure AI Foundry (student account)** - for free LLM and OCR extraction
    - **LangChain** - for LLM-assisted feature extraction and validation
    - Feature Extraction - PDF Parsing, OCR, Semantic Search
- Diagramming:
    - **Mermaid** - for architecture and workflow visualisation
- ESG Reporting Framework:
    - **GRI Standards** - for ESG reporting framework and KPI mapping

See [DESIGN_DECISIONS.md](docs/DESIGN_DECISIONS.md) for why each tool was chosen over its alternatives.

<hr>

## License
Licensed under the GNU General Public License v3.0. See [LICENSE](LICENSE).
