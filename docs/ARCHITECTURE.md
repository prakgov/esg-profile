## Overview
- [High-Level Architecture](#high-level-architecture)
- [Data Ingestion Pipeline](#data-ingestion-pipeline)
- [ESG Feature Extraction Pipeline](#esg-feature-extraction-pipeline)
- [Analysis Workflow](#analysis-workflow)

<br/>
<hr>

## High-Level Architecture

```mermaid
flowchart TD
    A1[Forbes Global 2000]
    A2[Company sustainability reports]
    A3[Historical stock prices]
    B[Data acquisition and ingestion]
    C["PDF storage and metadata<br/>Azure Blob Storage / Local Storage"]
    D[Text, table and OCR extraction]
    E["ESG evidence retrieval<br/>Keywords, GRI references, semantic search"]
    F[LLM-assisted KPI feature extraction]
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

    B --> G["Access OpenBB MCP Server"]
    G --> H["Collect current-year historical stock prices"]
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
    S["Open BB Historical Stock Data"] --> B

    B --> C["Calculate stock metrics"]
    C --> C1["Returns"]
    C --> C2["Volatility"]
    C --> C3["Drawdowns"]
    C --> C4["Market-vulnerability performance"]

    B --> D["Aggregate ESG scores"]
    D --> D1["Environmental pillar score"]
    D --> D2["Social pillar score"]
    D --> D3["Governance pillar score"]
    D --> D4["Overall ESG score"]

    C1 --> E["Control and align data"]
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