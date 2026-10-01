# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository structure

This repo has two distinct parts at different levels of maturity:

- **Root (`/README.md`, `/docs/`)** — the project's design document. It defines the research question, scope, methodology, and planned architecture in detail, but describes a pipeline that is **not yet implemented**.
- **`esg-profile/`** — the actual Python package, currently just a `uv`-managed scaffold (`src/esg_profile/__init__.py` contains only a placeholder `main()`). No dependencies, tests, or lint config exist yet.

When asked to implement something, treat the root docs as the spec and `esg-profile/` as where the implementation should go.

## Commands

All commands run from the `esg-profile/` directory (the package root, with its own `pyproject.toml`), using `uv` (build backend `uv_build`, requires Python >=3.13):

```
uv sync              # install dependencies
uv run esg-profile   # run the CLI entry point (esg_profile:main)
uv build             # build the package
```

No test suite, linter, or formatter is configured yet — `dependencies = []` in `esg-profile/pyproject.toml`. Check that file before assuming any tool (pytest, ruff, etc.) is available.

## Project background and architecture

This project studies the correlation between company ESG (Environmental/Social/Governance) scores and corporate financial performance (primary) and stock performance (secondary) for the top 100 companies in the Forbes Global 2000 (2025 list), for 2023–2025, visualized via a Streamlit dashboard. Full methodology is in `README.md`; diagrams (Mermaid) are in `docs/ARCHITECTURE.md`; the ESG KPI-to-GRI-standard mapping is in `docs/KPI_Breakdown.md`.

### Planned pipeline (per `docs/ARCHITECTURE.md`)

1. **Data acquisition** — Forbes Global 2000 (Kaggle) for the company list; automated web search for annual sustainability report PDFs; the OpenBB MCP Server and yfinance (unofficial Yahoo Finance API, temporary) for corporate financials (primary) and historical stock prices (secondary), 2023–2025. LLM and OCR extraction run on an Azure AI Foundry student account (must stay free).
2. **Storage** — lakehouse-style Bronze (raw) → Silver (cleaned) → Gold (analysis-ready) layers in **DuckDB**; PDFs in Azure Blob Storage or local storage, with checksums recorded in the DB.
3. **ESG feature extraction** — parse text/tables/OCR from reports, retrieve relevant evidence (GRI references, KPI names, semantic search), extract structured KPI values via LLM (LangChain), validate against source evidence, normalize units/periods, and store full data lineage (confidence, source quote, page, model version, validation status) per value.
4. **ESG scoring** — each of Tekmon's 25 KPIs (10 Environmental, 10 Social, 5 Governance; mapped to GRI standards in `docs/KPI_Breakdown.md`) is scored 0–5 based on disclosure evidence. Pillar scores are weighted averages of KPI scores (equal weights initially); overall ESG score is the weighted average of the three pillar scores (currently 33.3% each). Confidence, disclosure quality, GRI alignment, assurance, and performance are stored as separate columns so missing disclosure is never conflated with poor ESG performance.
5. **Analysis** — join ESG scores with stock metrics (returns, volatility, drawdowns, market-vulnerability-period performance), control for missing data/industry/period alignment, run correlation/regression with robustness checks (varying weights, windows, disclosure thresholds). Results are reported as associations, never causal claims.
6. **Dashboard** — Streamlit, showing stock movement against ESG scores plus financial metrics (sales, profit, assets, market value).

### Key constraints to keep in mind

- GRI 207 (Tax), 415 (Public Policy), and 419 (Socioeconomic Compliance) have no Tekmon KPI counterpart — documented as a known gap, not an oversight.
- Report period (ESG scores) and financial/stock period (2023–2025) must be explicitly aligned per company fiscal year before analysis.
- LLM and OCR extraction must stay free (Azure AI Foundry student account): parse text locally first, OCR only scanned pages, cache by checksum/KPI/prompt version, and respect quotas.
- yfinance is unofficial and temporary: keep it, and the OpenBB MCP Server, behind a provider adapter so either can be replaced.
- Some open decisions are still unresolved (see "Open Decisions" in `README.md`): market-vulnerability events to mark, Azure quota/OCR page limits, source licensing, the source of the Forbes 2025 list, and Forbes-to-ticker mapping. Check with the user before making an irreversible design choice on any of these.

## Design patterns

Whenever code introduces or changes a design pattern (e.g. a factory for extractor backends, a strategy for KPI scoring, a repository wrapper around DuckDB), record it in `docs/DESIGN_PATTERNS.md` — name, where it's used, and why. Keep that file in sync with the code as part of the same change, not as a follow-up.

## Design decisions log (educational)

The user is using this project to learn data engineering, distributed systems, and system design. Every design decision — made during design, development, testing, refactoring, debugging, deployment, or maintenance — must be logged in `docs/DESIGN_DECISIONS.md`, following the ADR-style format already in that file (Context, Decision, Alternatives considered, **Concepts** explained for a learner, Trade-offs; Debugging entries also get a **Root cause** field). That file's "Per-phase guidance" section has the bar for each phase (e.g. Debugging entries are only for bugs that involved a real trade-off or invalidated an earlier assumption, not routine fixes). This is distinct from `docs/DESIGN_PATTERNS.md` (which tracks code-level patterns, not decisions/rationale). Add an entry whenever you: choose a technology or library, pick an architecture/data-flow approach, decide a storage or scaling strategy, make a deployment/infra choice, or change an earlier decision (mark the old entry "Superseded by DD-<N>"). Do this as part of the same change, not as a follow-up — and explain the underlying concept, not just the choice.
