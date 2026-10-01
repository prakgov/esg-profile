# Company Sustainability Profile: ESG Evaluation and Stock Performance Insights

## Table of Contents
- [Background](#background)
- [Problem Statement](#problem-statement)
- [Objective](#objective)
    - [Traditional Growth Model](#traditional-growth-model-h0)
    - [Modern Growth Model](#modern-growth-model-ha)
    - [Assumptions](#assumptions)
- [Scope & Constraints](#scope--constraints)
    - [Goals](#goals)
    - [Target Users](#target-users)
    - [In Scope](#in-scope)
    - [Out of Scope](#out-of-scope)
    - [Constraints](#constraints)
    - [Open Decisions](#open-decisions)
- [Methodology](#methodology)
    - [Tools and Technologies](#tools--technologies)
    - [Data Acquisition & Ingestion Pipeline](#data-acquisition--ingestion-pipeline)
    - [ESG KPIs Structure](#esg-kpis-structure)
    - [Data Modeling & Backend Architecture](#data-modeling--backend-architecture)
    - [ESG Report Feature Extraction Pipeline](#esg-report-feature-extraction-pipeline)
    - [Analysis & Correlation Workflow](#analysis--correlation-workflow)

- [Notes](#notes)

<hr>

## Background
- **ESG** reporting could be simplified as follows: 
    * **E** for  Environmental 
        * building economic growth within environmental degradation limits, i.e., securing and managing water, soil, air & biodiversity (wildlife)
    * **S** for Social
        * focusing on maintenance and prosperity of the employees and their communities, 
    * **G** for Governance
        * developing ethical standards of administration within the organisation
- Sustainability standards refers to the ESG guidelines outlined by Global Reporting Initiative (GRI) standards. 
- GRI standards is considered one of the most widely used frameworks for ESG reporting among large companies globally, with a majority of the world's largest companies utilizing them to disclose their sustainability impacts.

<hr>

## Problem Statement
Forbes Global 2000 annually publishes top 2000 global public companies which are ranked based on four equally-weighted metrics: sales, assets, market capital and profit. However, the published results do not disclose nor mention any of the companies' sustainability profiles. The questions that aries could be:
- How can a company be claimed as prosperous if it does not provide any reported contributions to any aspects of the Environment, Social or Governance entities within its operational environment?
- How does the company manage its transparency and accountability?
- How does it build stakeholder trust, manage risks, and create long-term value while demonstrating responsible and sustainable business practices? 


## Objective
Using the top 100 companies of the Forbes Global 2000 (2025 list) for 2023-2025 we will try to determine the **Correlation betweeen Company ESG scores (calculated in accordance with ESG KPIs based on GRI standards) and Corporate financial performance (primary) and Stock performance (secondary)**. 

This evaluation will then be visualised using a **dashboard to view company financial performance and stock price movement in relation to calculated ESG-scores**.

### **Traditional Growth Model (H<sub>0</sub>)**
> Value creation & ESG reporting by company **does not correlate with GRI standards** or **ESG reporting by the company is non-existent, and yet leads to company's exponential economic growth**.

### **Modern Growth Model (H<sub>A</sub>)**
> Value creation & ESG Reporting by company **correlates with GRI standards and leads to resilient performance and sustained growth**.
- If any key market vulnerable moments during 2023-2025, will be marked in the analysis to evaluate the company's resilience and performance during such periods.
- Market Vulnerability could be caused due to several factors:
    - economic, 
    - geopolitics, 
    - supply-chain disturbances, 
    - environmental issues, 

    and other major issues which may or may not be related to Environment, Social or Governance aspects within its operational environment.


### Assumptions
1. Company financial performance (sales, profit, assets and market value) defines its economic growth; stock performance is a secondary, market-based indicator.
2. Company Annual sustainability reports are based on GRI standard framework.
3. Market vulnerability (in this context) refers to **environmental issues, supply-chain disturbances and/or economic factors**

[Back to Table of Contents](#table-of-contents)

<br/>

<hr>

## Scope & Constraints

### Goals
- **Primary:** relate company ESG scores to **corporate financial performance** (sales, profit, assets and market value) for 2023-2025.
- **Secondary:** relate ESG scores to **stock price performance** (returns, volatility, drawdowns) over the same period.
- **Deliverable:** a dashboard where each user group can compare a company's financial and stock performance with its ESG score and see the evidence behind it.
- **Success criteria:** report coverage (reports found out of the 300 company-years) and extraction accuracy against a hand-checked sample.

### Target Users
- **Researchers:** need methodology, data lineage, confidence values and robustness results.
- **Long-term retail investors:** need company rankings and ESG scores in the context of financial and stock performance.
- **Institutional portfolio managers:** need peer and industry comparisons, pillar-level detail and disclosure quality.
- **Companies:** need to benchmark their own disclosure and ESG score against peers and see where their disclosure has gaps.

### In Scope
- **Companies:** Top 100 of the Forbes Global 2000 (2025 list).
- **Period:** 2023-2025 (three years) for all data sources.
- **Financial data (primary):** sales, profit, assets and market value for 2023-2025, via the OpenBB MCP Server and yfinance (temporary solution).
- **Stock data (secondary):** daily historical prices for 2023-2025, via the OpenBB MCP Server and yfinance.
- **ESG data:** annual sustainability reports for 2023-2025 (up to 300 PDFs), collected through automated web search. LLM and OCR extraction runs on an Azure AI Foundry student account.
- **ESG framework:** Tekmon's 25 ESG KPIs (10 Environmental, 10 Social, 5 Governance) mapped to GRI standards, scored 0-5 into pillar and overall ESG scores.
- **Analysis:** correlation and regression of ESG scores against financial performance (primary) and returns, volatility, drawdowns and market-vulnerability performance (secondary), with robustness tests.
- **Output:** a Streamlit dashboard showing financial performance and stock movement in relation to ESG scores.

### Out of Scope
- Companies outside the top 100, and years outside 2023-2025.
- GRI standards with no Tekmon counterpart: GRI 207 (Tax), GRI 415 (Public Policy) and GRI 419 (Socioeconomic Compliance).
- Causal claims: results are reported as associations only.
- Investment advice: the dashboard informs analysis and is not a recommendation to buy or sell.
- Independent verification of company-reported figures: external assurance is recorded as a flag, not re-audited.

### Constraints

| Constraint | Impact | Handling in the design |
|---|---|---|
| Small sample (100 companies × 3 years ≈ 300 company-years) with uneven disclosure | Limits statistical power of correlation and regression | Report associations with confidence intervals; repeat with different disclosure-quality thresholds |
| Short window (2023-2025, three years) | Few market-vulnerability events and short trends, which limits conclusions about long-term value creation | Mark the vulnerability events that fall inside 2023-2025; present results as associations over this window |
| Report period differs from financial and stock period | Fiscal-year ends differ between companies; ESG reports cover specific reporting years | Align each report year with the matching fiscal-year financials and stock period before analysis |
| Market data comes from the OpenBB MCP Server and yfinance, an unofficial Yahoo Finance API (temporary solution) | No SLA for yfinance; rate limits and availability of both sources; possible schema changes; personal-use terms | Keep both behind a provider adapter so either can be replaced; throttle and cache requests; store raw responses, source and retrieval timestamps in Bronze; cross-check the two sources and the Forbes 2025 list values |
| LLM and OCR extraction must stay free (Azure AI Foundry student account) | Student credit and quotas cap tokens, request rate and OCR pages | Parse text locally first and OCR only scanned pages; retrieve evidence before prompting so only relevant passages reach the LLM; cache by checksum, KPI and prompt version; throttle to quota; check free-tier OCR page limits and keep a local OCR fallback |
| Companies report in different currencies | Raw sales, profit and assets are not directly comparable | Store the reporting currency; compare using ratios and growth rates, or convert to one currency |
| Reports are heterogeneous PDFs (tables, scans) | Extraction errors and missing values | OCR, evidence validation loop, confidence and lineage stored per value |
| LLM extraction is not fully reliable | Wrong values could distort scores | Validate against source pages, units and KPI definitions; store model version and validation status |
| Missing disclosure is not poor performance | Scores could penalise non-disclosure as bad performance | Disclosure quality and performance stored in separate columns |
| Tekmon KPIs are a GRI-adjacent subset | Not a fully GRI-aligned disclosure set | Documented gaps (GRI 207, 415, 419) |
| Equal initial KPI and pillar weights | Preliminary scores ignore industry materiality | Weights adjustable later by industry materiality or data reliability |
| Storage and compute: DuckDB, Azure Blob or local storage | Single-node analytics; Azure adds cost | Use local storage by default; Azure when sharing is needed |

### Open Decisions
- [X] **Stock window:** 2023-2025, matching the three years of financial data and sustainability reports.
- [ ] **Market-vulnerability events:** which events within 2023-2025 are marked in the analysis.
- [X] **Success criteria:** target report coverage (reports found out of 100) and extraction accuracy against a hand-checked sample.
- [X] **Primary audience:**  researchers, long-term retail investors, or institutional portfolio managers, companies.
- [X] **Human review:** Prakirth Govardhanam.
- [X] **Cost and rate limits:** LLM and OCR extraction stay free using an Azure AI Foundry student account; OpenBB MCP Server and yfinance request limits still apply.
- [ ] **Quota check:** confirm the student credit, model quota and OCR page limits before processing reports.
- [ ] **Source licensing:** terms for storing and processing company PDFs, and the terms of use of OpenBB and yfinance.
- [X] **Financial metrics:** sales, profit, assets and market value are the primary dataset, ingested via the OpenBB MCP Server and yfinance.
- [ ] **Forbes 2025 list source:** the Kaggle dataset referenced earlier is the 2026 list, so a source for the 2025 list is needed.
- [ ] **Ticker mapping:** how Forbes company names are mapped to Yahoo Finance tickers, and how mismatches are checked.

[Back to Table of Contents](#table-of-contents)

<br/>

<hr>

## Methodology
### Tools & Technologies
- Languages: **Python** and **SQL**
    - **Python** - for data acquisition, ingestion, processing, analysis and visualisation
    - **DuckDB** - for relational database management and data storage
- Storage: **Azure Blob Storage** or **Local Storage**
    - **Azure Blob Storage** - for object storage of PDF reports
- Data Sources:
    - **Forbes Global 2000 dataset (2025 list)** - for top 100 companies
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

### Data Acquisition & Ingestion Pipeline
- Using the Forbes Global 2000 (2025 list), the top 100 companies will be collected to keep the data acquisition process straight forward. The source of the 2025 list is to be confirmed (the previously referenced [Kaggle dataset](https://www.kaggle.com/datasets/ellimaaac/forbes-the-global-2000-companies-2026) is the 2026 list).
- For each of the listed company in the top 100 for Forbes Global 2000, annual sustainability reports for 2023-2025 are collected through automated web search and stored in object storage services.

#### Corporate Financial and Stock Price Data
- [OpenBB MCP Server](https://docs.openbb.co/odp/python/quickstart/mcp) will be accessed for historical stock price and corporate financial information.
- [yfinance](https://github.com/ranaroussi/yfinance), an unofficial Yahoo Finance API for Python, is used alongside it as a temporary data source.
- **Corporate financials (primary):** sales, profit, assets and market value for 2023-2025.
- **Stock prices (secondary):** daily historical prices for 2023-2025.
- Raw responses are stored with a retrieval timestamp, and the Forbes 2025 list values are used to cross-check the financials.

#### Data Storage
- Collected data from the reports will be organized into a lakehouse-style architecture: raw data (Bronze), cleaned/enriched data (Silver), and analysis-ready data (Gold).
- Object storage for PDFs: **Azure Blob Storage/Local storage**
    - Store object URL and checksum in the database
- Relational database for structured data: **DuckDB**
    - Store companies, report metadata, extracted ESG KPIs, scores, corporate financials, stock prices, and correlation results


### ESG KPIs Structure
- Company sustainability reports will be processed for recording categorized ESG KPIs into industrially acknowledged impacts by using Tekmon's ESG KPI examples.
- Tekmon's 25 ESG KPIs are a solid, practical starting checklist for a company beginning ESG measurement, and it's broadly consistent with S&P Global ESG Score Industry-weighted  Materiality Matrices approach.
- However, it should be duly noted that few GRI standards have no clear Tekmon counterpart:
    - GRI 207 (Tax), 
    - GRI 415 (Public Policy/lobbying), or 
    - GRI 419 (Socioeconomic Compliance)
- Therefore, Tekmon's list is considered as a practical, GRI-adjacent subset - for general tracking, but not as a fully GRI-aligned disclosure set.
- Refer [ESG KPIs Breakdown](/docs/KPI_Breakdown.md) for category specific KPIs.


### Data Modeling & Backend Architecture
- With reference to the [databricks' blog](https://www.databricks.com/blog/2020/07/10/a-data-driven-approach-to-environmental-social-and-governance.html), advanced NLP techniques or LLM engineering will be performed to understand semantics and context of the sustainability reports.
- Features will be extracted from the reports to verify company behavior with regard to the above listed ESG KPIs and in comparison with its financial performance and stock price movement for 2023-2025.
- Refer [High-Level Architecture overview](./docs/ARCHITECTURE.md).


### ESG Report Feature Extraction pipeline
1. **Ingest reports**: Store PDFs with company, reporting year, source URL, checksum, and metadata.
2. **Extract content**: Parse text, tables, headings, page numbers, and OCR scanned pages where necessary.
3. **Identify relevant evidence**: Retrieve passages and tables using GRI references, KPI names, keywords, and semantic search.
4. **Extract structured features**: Use an LLM to identify KPI values, units, periods, boundaries, targets, trends, and qualitative disclosures.
5. **Validate evidence**: Check extracted values against the source pages, units, calculations, and KPI definitions.
6. **Normalize features**: Convert units, standardize periods, distinguish absolute from intensity metrics, and classify missing or non-disclosed data.
7. **Map to ESG KPIs**: Link each feature to its Environmental, Social, or Governance KPI and associated GRI standard.
8. **Store data lineage**: Save the extracted value, confidence, source quotation, page number, model version, and validation status.
9. **Publish analysis-ready data**: Aggregate validated features into ESG disclosure and performance scores for correlation analysis with financial and stock data.

- Refer [Feature Extraction Pipeline](/docs/ARCHITECTURE.md).


#### ESG Score based on GRI-based ESG KPIs
- Each KPI is assigned a score from 0 to 5 based on reported evidence:
    - **0**: Not disclosed or no relevant evidence
    - **1**: General commitment or policy only
    - **2**: Qualitative disclosure with limited evidence
    - **3**: Quantitative metric disclosed
    - **4**: Quantitative metric with historical trend or target progress
    - **5**: Quantitative, assured, GRI-mapped, and showing positive performance

- A **pillar score** will be calculated, which is the weighted average of the KPI scores within one ESG category (Environmental/Social/Governance):

$$
PillarScore = \frac{\sum (KPI\ Score \times KPI\ Weight)}{\sum KPI\ Weights}
$$

- Initial weights used for preliminary results:
    - **Environmental:** 33.3%
    - **Social:** 33.3%
    - **Governance:** 33.3%
- Within each pillar, weights are assigned equally to its KPIs initially. 
- Later, weights can be adjusted by industry materiality or data reliability.

- Then overall ESG score is calculated:
$$
ESGScore = w_E E + w_S S + w_G G
$$

- Since we  used equal weights for preliminary results:
$$
ESGScore = \frac{E + S + G}{3}
$$

- To evaluate the LLM feature extraction and quality of the ESG report, the relevant metrics are stored in seperate columns within ESG KPI table: 
    - **Confidence:** LLM extraction reliability.
    - **Disclosure quality:** Completeness and clarity of the reported evidence.
    - **GRI alignment:** Correct mapping to the relevant GRI standard.
    - **Assurance:** Whether the reported data is externally assured.
    - **Performance:** The company’s underlying ESG result, not the LLM’s capability.
- This prevents missing disclosure from being confused with poor ESG performance and allows industry-specific weights to be introduced later.


### Analysis & Correlation Workflow
- Dashboard will be rendered using **Streamlit** to visualise key financial metrics such as sales, profit, assets, and market value for the given period in comparison with its deduced ESG score.

1. **Prepare datasets:** Join validated ESG KPI scores with company identifiers, reporting periods, industry, financial data, and stock-price data.
2. **Calculate performance metrics:** Compute year-over-year growth in sales, profit, assets, and market value (primary), then returns, volatility, drawdowns, and performance during selected market-vulnerability periods (secondary).
3. **Aggregate ESG scores:** Calculate Environmental, Social, Governance pillar scores and the overall ESG score.
4. **Control the data:** Handle missing values, normalize by industry, and align ESG reporting periods with stock-performance periods.
5. **Run analysis:** Compare ESG scores with financial performance, stock returns, volatility, and resilience using correlation and regression analysis.
6. **Test robustness:** Repeat using different KPI weights, time windows, and disclosure-quality thresholds.
7. **Visualize results:** Show scatter plots, rankings, time-series comparisons, pillar contributions, and confidence intervals.
8. **Interpret carefully:** Report associations rather than claiming that ESG performance directly causes stock performance.

- Based on the quality of feature extraction and the sustainability reports, we attempt to deduce:
  - **Proxy data** provided by services to companies
  - **Outsourcing ESG activities**, such as purchasing carbon offsets without improvements in Company's Sustainability

- Refer [Analysis Workflow](/docs/ARCHITECTURE.md).

[Back to Table of Contents](#table-of-contents)

<br/>

<hr>
<hr>

## Notes
- General Information:
    - **ESG** - [Environmental, Social and Governance](https://www.investopedia.com/terms/e/environmental-social-and-governance-esg-criteria.asp) - Investopedia
    - **SDG** - [Sustainable Developmental Goals](https://sdgs.un.org/goals) - United Nations

- Dataset:
    - [Forbes Global 2000](https://en.wikipedia.org/wiki/Forbes_Global_2000#2026_list) - Wikipedia

- ESG Reporting Framework:     
    - **GRI** - [Global Reporting Initiative](https://www.globalreporting.org/how-to-use-the-gri-standards/resource-center/)

- Sustainability KPI checklist:
    - [ESG KPIs](https://www.tekmon.com/resources/blog/25-esg-kpi-examples) - Tekmon

- Inspirational blog posts:
    - [Data-driven Approach to Environmental, Social and Governance](https://databricks.com/blog/2020/07/10/a-data-driven-approach-to-environmental-social-and-governance.html) -  databricks
    - [ESG to SDGs: Connected Paths to a Sustainable Future](https://sustainometric.com/esg-to-sdgs-connected-paths-to-a-sustainable-future/) - Sustainometric

<br/>
<hr>

> **Note:** The solution presented in this project is one possible approach. Other methods or solutions may also be used to address the given problem. This is my attempt to focus on one aspect of growth identifiers - **Correlation between a global ESG framework (GRI) and long-term value creation of top 100 global companies**.

<hr>