# Company Sustainability Profile: ESG Evaluation and Stock Performance Insights

## Table of Contents
- [Background](#background)
- [Problem Statement](#problem-statement)
- [Objective](#objective)
    - [Traditional Growth Model](#traditional-growth-model-h0)
    - [Modern Growth Model](#modern-growth-model-ha)
    - [Assumptions](#assumptions)
- [Methodology](#methodology)
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
Using the Forbes2000 for the current year we will try to determine the **Correlation betweeen Company ESG scores (calculated in accordance with ESG KPIs based on GRI standards) and Stock performance**. 

This evaluation will then be visualised using a **dashboard to view company stock price movement in relation to calculated ESG-scores**.

### **Traditional Growth Model (H<sub>0</sub>)**
> Value creation & ESG reporting by company **does not correlate with GRI standards** or **ESG reporting by the company is non-existent, and yet leads to company's exponential economic growth**.

### **Modern Growth Model (H<sub>A</sub>)**
> Value creation & ESG Reporting by company **correlates with GRI standards and leads to resilient performance during key market vulnerable moments and sustained growth**.
- Market Vulnerability could be caused due to several factors:
    - economic, 
    - geopolitics, 
    - supply-chain disturbances, 
    - environmental issues, 

    and other major issues which may or may not be related to Environment, Social or Governance aspects within its operational environment.


### Assumptions
1. Company stock performance defines its economic growth.
2. Company Annual sustainability reports are based on GRI standard framework.
3. Market vulnerability (in this context) refers to **environmental issues, supply-chain disturbances and/or economic factors**

[Back to Table of Contents](#table-of-contents)

<br/>

<hr>

## Methodology
### Data Acquisition & Ingestion Pipeline
- Using the [Kaggle dataset](https://www.kaggle.com/datasets/ellimaaac/forbes-the-global-2000-companies-2026), Forbes Global 2000, the top 100 companies will be collected to keep the data acquisition process straight forward.
- For each of the listed company in the top 100 for Forbes Global 2000, annual sustainability reports are collected through automated web search and stored in object storage services.

#### Historical Stock Price Data 
- [OpenBB MCP Server](https://docs.openbb.co/odp/python/quickstart/mcp) will be accessed for historical stock price information.
- Stock price information from the beginning of the current year to till date will be collected for evaluation.

#### Data Storage
- Collected data from the reports will be organized into a lakehouse-style architecture: raw data (Bronze), cleaned/enriched data (Silver), and analysis-ready data (Gold).
- Object storage for PDFs: **Azure Blob Storage/Local storage**
    - Store object URL and checksum in the database
- Relational database for structured data: **DuckDB**
    - Store companies, report metadata, extracted ESG KPIs, scores, stock prices, and correlation results


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
- Features will be extracted from the reports to verify company behavior with regard to the above listed ESG KPIs and in comparison with its stock price movement for the current year.
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
9. **Publish analysis-ready data**: Aggregate validated features into ESG disclosure and performance scores for correlation analysis with stock data.

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

1. **Prepare datasets:** Join validated ESG KPI scores with company identifiers, reporting periods, industry, and stock-price data.
2. **Calculate stock metrics:** Compute returns, volatility, drawdowns, and performance during selected market-vulnerability periods.
3. **Aggregate ESG scores:** Calculate Environmental, Social, Governance pillar scores and the overall ESG score.
4. **Control the data:** Handle missing values, normalize by industry, and align ESG reporting periods with stock-performance periods.
5. **Run analysis:** Compare ESG scores with stock returns, volatility, and resilience using correlation and regression analysis.
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