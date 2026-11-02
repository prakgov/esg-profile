## Overview
- [High-Level Architecture](#high-level-architecture)
- [Data Ingestion Pipeline](#data-ingestion-pipeline)
- [ESG Feature Extraction Pipeline](#esg-feature-extraction-pipeline)
- [Analysis Workflow](#analysis-workflow)
- [Recommended Project Structure](#recommended-project-structure)
- [Project Structure guidelines](#project-structure-guidelines)

<br/>
<hr>

## High-Level Architecture

```mermaid
flowchart TD
    A1["Forbes Global 2000<br/>2025 list, top 100"]
    A2["Company sustainability reports<br/>2023-2025"]
    A3["Corporate financials and historical stock prices<br/>OpenBB MCP Server and yfinance, 2023-2025"]
    B[Data acquisition and ingestion]
    C["PDF storage and metadata<br/>Azure Blob Storage / Local Storage"]
    D[Text, table and OCR extraction]
    E["ESG evidence retrieval<br/>Keywords, GRI references, semantic search"]
    F["LLM-assisted KPI feature extraction<br/>Azure AI Foundry"]
    G[Validation, normalisation and data lineage]
    H["DuckDB<br/>Bronze → Silver → Gold"]
    I[ESG scoring and stock-performance analysis]
    J[Streamlit dashboard]

    A1 --> B
    A2 --> B
    A3 --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
```

## Data Ingestion Pipeline

```mermaid
flowchart TD
    L[("DuckDB")]
    A["Forbes Global 2000 dataset"] --> B["Select top 100 companies"]
    B --> C["Automated web search"]
    C --> D["Collect annual sustainability reports"]
    D --> E["Store reports in object storage"]
    E --> F["Record company, reporting year, source URL, and checksum"]

    B --> G["Access OpenBB MCP Server and yfinance"]
    G --> H["Collect 2023-2025 corporate financials and historical stock prices"]
    H --> L

    F --> I["Bronze layer: raw reports and data"]
    I --> L

    I --> J["Silver layer: cleaned and enriched data"]
    J --> L

    J --> K
    K["Gold layer: analysis-ready data"] --> L


    E --> M[("Azure Blob Storage or Local Storage")]
    L --> N["ESG Feature Extraction Pipeline"]
    M --> N
```
- [Back to top](#overview)

## ESG Feature Extraction Pipeline

```mermaid
flowchart TD
    A["Ingest sustainability reports"] --> B["Extract text, tables, headings, and page numbers"]
    B --> C["OCR scanned pages"]
    C --> D["Identify relevant evidence"]
    D --> E["Retrieve passages using GRI references, KPI names, keywords, and semantic search"]
    E --> F["Extract structured features with an LLM"]
    F --> G["Validate extracted evidence"]
    G --> H{"Evidence and values valid?"}
    H -->|No| I["Review source pages, units, calculations, KPI definitions"]
    I --> F
    H -->|Yes| J["Normalize features"]
    J --> K["Convert units and standardize periods"]
    K --> L["Classify absolute, intensity, missing, and non-disclosed data"]
    L --> M["Map features to ESG KPIs and GRI standards"]
    M --> N["Store data lineage"]
    N --> O["Save confidence, quotation, page, model version, validation status"]
    O --> P["Analysis Workflow"]
```

## Analysis Workflow

```mermaid
flowchart TD
    A["Validated ESG KPI scores"] --> B["Prepare datasets"]
    S["OpenBB and yfinance financials and historical stock data"] --> B

    B --> C["Calculate financial and stock metrics"]
    C --> C0["Sales, profit, assets and market-value growth"]
    C --> C1["Returns"]
    C --> C2["Volatility"]
    C --> C3["Drawdowns"]
    C --> C4["Market-vulnerability performance"]

    B --> D["Aggregate ESG scores"]
    D --> D1["Environmental pillar score"]
    D --> D2["Social pillar score"]
    D --> D3["Governance pillar score"]
    D --> D4["Overall ESG score"]

    C0 --> E["Control and align data"]
    C1 --> E
    C2 --> E
    C3 --> E
    C4 --> E
    D1 --> E
    D2 --> E
    D3 --> E
    D4 --> E

    E --> E1["Handle missing values"]
    E1 --> E2["Normalize by industry"]
    E2 --> E3["Align ESG and stock-performance periods"]

    E3 --> F["Run correlation and regression analysis"]
    F --> G["Test robustness"]
    G --> G1["Vary KPI weights"]
    G --> G2["Vary time windows"]
    G --> G3["Vary disclosure-quality thresholds"]

    G1 --> H["Visualize results"]
    G2 --> H
    G3 --> H

    H --> H1["ESG KPI Heatmaps"]
    H --> H2["Company rankings"]
    H --> H3["Time-series comparisons"]
    H --> H4["Pillar contributions"]
    H --> H5["Confidence intervals"]

    H --> I["Interpret associations carefully"]
    I --> J["Assess proxy data and outsourced ESG activities"]
    I --> K["Avoid causal claims"]
```
- [Back to top](#overview)

<hr>

### Recommended Project Structure

The repository should separate the executable Python package from documentation, configuration, tests, and data products. Within the package, dependencies should flow from domain models and interfaces toward infrastructure adapters and user-facing entry points. This keeps the analysis reproducible and makes it possible to replace yfinance, local file storage, or Azure services without rewriting the scoring and analysis logic.

```text
esg-profile/
├── README.md
├── .gitignore
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── KPI_Breakdown.md
│   ├── DESIGN_DECISIONS.md
│   └── DESIGN_PATTERNS.md
│
├── esg-profile/                       # Python package project
│   ├── pyproject.toml
│   ├── uv.lock
│   ├── .python-version
│   ├── src/
│   │   └── esg_profile/
│   │       ├── __init__.py
│   │       ├── cli.py                  # CLI commands and pipeline orchestration
│   │       ├── config.py               # Environment and typed settings
│   │       │
│   │       ├── domain/                 # Framework-independent business rules
│   │       │   ├── models/             # Company, report, KPI, score, and metric models
│   │       │   ├── enums.py
│   │       │   └── scoring.py           # KPI, pillar, and overall ESG scoring
│   │       │
│   │       ├── ports/                  # Interfaces for replaceable dependencies
│   │       │   ├── market_data.py       # Financials and historical prices
│   │       │   ├── report_sources.py    # Report discovery and retrieval
│   │       │   ├── object_storage.py
│   │       │   └── warehouse.py
│   │       │
│   │       ├── adapters/               # External systems implementing ports
│   │       │   ├── openbb/
│   │       │   ├── yfinance/
│   │       │   ├── azure_blob/
│   │       │   ├── local_storage/
│   │       │   └── duckdb/
│   │       │
│   │       ├── pipelines/
│   │       │   ├── acquisition/        # Forbes, reports, financials, and prices
│   │       │   ├── documents/          # PDF parsing, tables, OCR, and evidence retrieval
│   │       │   ├── extraction/         # LLM extraction, validation, and lineage
│   │       │   ├── normalization/      # Units, periods, currencies, and identifiers
│   │       │   └── scoring/             # ESG scores and disclosure-quality measures
│   │       │
│   │       ├── analysis/
│   │       │   ├── metrics.py           # Returns, volatility, drawdowns, and growth
│   │       │   ├── dataset.py           # Period and industry alignment
│   │       │   ├── statistics.py        # Correlation, regression, and robustness tests
│   │       │   └── vulnerability.py    # Market-vulnerability event definitions
│   │       │
│   │       └── dashboard/
│   │           ├── app.py               # Streamlit entry point
│   │           ├── queries.py           # Gold-layer dashboard queries
│   │           └── components/          # Charts, tables, and evidence views
│   │
│   ├── sql/
│   │   ├── bronze/                     # Raw-ingestion table definitions
│   │   ├── silver/                     # Cleaned and validated transformations
│   │   └── gold/                       # Analysis-ready views and aggregates
│   └── tests/
│       ├── unit/
│       ├── integration/
│       └── fixtures/
│
├── data/                               # Local runtime data; do not commit
│   ├── raw/                            # Downloaded PDFs and source responses
│   ├── bronze/
│   ├── silver/
│   ├── gold/
│   └── warehouse.duckdb
│
├── configs/
│   ├── default.toml
│   ├── kpi_mapping.toml
│   └── companies.csv
├── scripts/                            # Reproducible operational and exploratory entry points
└── artifacts/                          # Reports, plots, and run manifests; do not commit
```

### Project Structure guidelines

- Keep `domain/` independent of Streamlit, DuckDB, Azure SDKs, and provider-specific response formats.
- Keep source-specific code behind `ports/` and `adapters/`; OpenBB and yfinance should implement the same market-data interface.
- Treat `data/raw/` and Bronze records as immutable inputs. Rebuild Silver and Gold outputs from them instead of editing derived data manually.
- Store extraction evidence and lineage with the Silver KPI records, including source checksum, page, quotation, confidence, model version, and validation status.
- Put reusable production logic in `src/`. Exploration and demonstrations are scripted too, through the CLI or `scripts/`; the project uses no notebooks.
- Keep secrets in environment variables or a local secret manager. Commit configuration templates, not credentials.
- Add tests at the same boundaries as the tree: unit-test scoring and normalization, integration-test adapters and DuckDB transformations, and use fixtures for small representative PDFs and API responses.

- [Back to top](#overview)