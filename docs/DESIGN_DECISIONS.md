# Design Decisions Log

An educational record of the design decisions made on this project — across design, development, testing, refactoring, debugging, deployment, and maintenance — written for learning data engineering, distributed systems, and system design, not just as a changelog.

Each entry explains not only *what* was chosen but the underlying concepts, the alternatives, and why they lost. New entries are added as decisions are made (or revisited) throughout the project's life, not only up front.

## Format for each entry

```
## DD-<NNN>: <short title>
- **Phase:** Design | Development | Testing | Refactoring | Debugging | Deployment | Maintenance
- **Status:** Proposed | Accepted | Superseded by DD-<NNN>
- **Context:** the problem/constraint that forced a decision (Debugging: the symptom observed, not the cause)
- **Root cause:** *(Debugging entries only)* why it actually broke — omit for other phases
- **Decision:** what was chosen (Debugging: the fix; Refactoring: the resulting structure)
- **Alternatives considered:** other options and why each was rejected (Debugging: other fixes that would have only patched the symptom; Refactoring: other restructurings)
- **Concepts:** the data-engineering / distributed-systems / system-design ideas this decision illustrates, explained briefly (Refactoring: name the principle or pattern applied, e.g. extract method, dependency inversion; Testing: name the testing concept, e.g. fixture isolation, test pyramid, flakiness sources)
- **Trade-offs:** what we gained and what we gave up
```

Per-phase guidance:
- **Testing** — log decisions about *what* and *how* to test (unit vs. integration boundary, fixture/mocking strategy, what's deliberately left untested and why), not routine test-writing.
- **Refactoring** — log restructurings driven by a real problem (duplication, a changed requirement, a bottleneck found in Testing/Debugging), not purely stylistic cleanup. Reference the DD(s) that motivated it.
- **Debugging** — only for bugs whose fix involved a real design trade-off or revealed a wrong earlier assumption (link back to the DD that assumption came from, and mark it "Superseded by" if the fix invalidates it) — not routine typo/syntax fixes.

---

## DD-001: Medallion (Bronze/Silver/Gold) architecture for storage

- **Phase:** Design
- **Status:** Accepted
- **Context:** Data flows through several stages — raw PDFs and API responses, cleaned/validated structured records, and analysis-ready aggregates — each needing different reliability and query characteristics.
- **Decision:** Organize the DuckDB warehouse into three layers: Bronze (raw, as-ingested), Silver (cleaned, validated, normalized), Gold (analysis-ready, aggregated).
- **Alternatives considered:**
  - *Single flat schema* — simpler, but conflates "what we received" with "what we trust," making it hard to re-run validation/extraction without losing the original source of truth.
  - *Fully normalized relational schema with no raw layer* — loses the ability to reprocess when extraction logic improves, since the original unprocessed data isn't retained.
- **Concepts:** This is the **medallion/lakehouse architecture** pattern popularized by Databricks. The core idea is *progressive refinement with immutable raw data*: each layer is derived from the one before it, so any layer can be rebuilt by replaying the pipeline, rather than being hand-edited in place. This is closely related to the **ETL vs ELT** distinction — here it's closer to ELT, since raw data lands first (Bronze) and transformation happens inside the warehouse afterward.
- **Trade-offs:** Gains reprocessability and a clear audit trail (useful given LLM extraction confidence/validation requirements). Costs extra storage (data is duplicated across layers) and requires discipline to keep each layer's contract (schema, meaning) stable.

## DD-002: DuckDB as the analytical database

- **Phase:** Design
- **Status:** Accepted
- **Context:** Need a relational store for structured data (companies, KPI scores, stock prices, correlation results) sized for ~100 companies — not a multi-terabyte workload.
- **Decision:** Use DuckDB, an embedded, single-node OLAP database.
- **Alternatives considered:**
  - *PostgreSQL* — a client-server OLTP-oriented database; would need a running server process and is tuned for many small transactional queries rather than analytical scans/aggregations.
  - *Spark/distributed warehouse (Snowflake, BigQuery)* — built for datasets far larger than this project's scale; would add operational and cost overhead with no performance benefit here.
- **Concepts:** This decision hinges on **OLAP vs OLTP**: OLAP (DuckDB) is optimized for column-oriented scans and aggregations over large result sets (e.g., "average ESG score by industry"), while OLTP (Postgres/MySQL) is optimized for many concurrent small reads/writes (e.g., "update one row"). DuckDB is also **embedded** (runs in-process, like SQLite) rather than **client-server**, which removes a whole class of distributed-systems concerns — no network calls, no connection pooling, no concurrent-writer coordination — at the cost of not supporting multiple concurrent writers or remote access out of the box.
- **Trade-offs:** Zero operational overhead (no server to run, back up, or secure) and fast analytical queries at this scale. Doesn't scale to distributed/concurrent-write workloads — acceptable here since ingestion is a batch pipeline, not a multi-user service.

## DD-003: Object storage (Azure Blob / local) separate from the relational database

- **Phase:** Design
- **Status:** Accepted
- **Context:** Sustainability reports are large, unstructured PDF files; DuckDB is a relational engine, not built to store large binary blobs efficiently.
- **Decision:** Store PDFs in object storage (Azure Blob Storage or local filesystem), and store only the object URL and a checksum in DuckDB.
- **Alternatives considered:**
  - *Store PDFs as BLOBs inside the database* — bloats the database file, slows down unrelated queries, and makes the DB harder to version/share.
- **Concepts:** This is the standard **separation of storage by access pattern**: structured, queryable metadata lives in a database; large unstructured payloads live in object storage, linked by reference. The **checksum** (e.g., SHA-256) lets the pipeline detect whether a source file changed since last processed — a basic form of **data integrity verification** / **change detection**, relevant later for deciding when re-extraction is needed.
- **Trade-offs:** Keeps the database small and fast, and files independently cacheable/shareable. Introduces a two-system consistency problem — the DB and the blob store can drift out of sync (e.g., a blob deleted but its row remains) — which the checksum-and-reference pattern only partially mitigates.

## DD-004: LLM-assisted extraction with a validation loop, not direct parsing

- **Phase:** Design
- **Status:** Accepted
- **Context:** ESG KPI values are embedded in heterogeneous, free-form PDF reports (prose, tables, scanned images) with no consistent structure across companies.
- **Decision:** Use an LLM (via LangChain) to extract structured KPI values from retrieved evidence passages, then validate each extraction against its source before accepting it, storing confidence and lineage per value.
- **Alternatives considered:**
  - *Rule-based/regex extraction* — would require bespoke rules per company/report layout; brittle and wouldn't generalize across the top-100 company set.
  - *Trust LLM output directly with no validation* — faster, but risks silently wrong KPI values propagating into the analysis with no way to tell extraction error from genuine non-disclosure.
- **Concepts:** This is a form of **human/automated-in-the-loop data quality control** applied to AI pipelines — treating LLM output as a *candidate* that must be checked against source evidence before being trusted, similar to how a distributed system might treat a replica's data as tentative until acknowledged. It also introduces **data lineage** (tracking where a value came from, with what confidence, validated how) as a first-class concern, not an afterthought — important in any pipeline where downstream consumers need to judge trustworthiness.
- **Trade-offs:** Higher accuracy and traceability, at the cost of extra LLM calls (cost, latency) and pipeline complexity (retry/review loop for failed validations).

## DD-005: Separate "disclosure quality" from "performance" in the data model

- **Phase:** Design
- **Status:** Accepted
- **Context:** A company might not disclose a KPI at all (silence) versus disclosing it and performing poorly — conflating these would unfairly penalize non-disclosure as if it were poor ESG performance.
- **Decision:** Store confidence, disclosure quality, GRI alignment, assurance, and performance as separate columns per KPI value, rather than folding them into a single score.
- **Alternatives considered:**
  - *Single composite score* — simpler schema, but destroys the information needed to distinguish "didn't report" from "reported and did badly," which matters for the project's own hypothesis testing.
- **Concepts:** This is a data-modeling instance of avoiding **premature aggregation** — keeping dimensions that are conceptually independent as separate fields so they can be recombined or re-weighted later, rather than collapsing them early and losing information irreversibly.
- **Trade-offs:** More columns and more complex downstream scoring logic, but preserves analytical flexibility (e.g., re-run with industry-specific weights later) without re-extracting data.

## DD-006: OpenBB MCP Server for stock price data

- **Phase:** Design
- **Status:** Accepted
- **Context:** Need historical stock price data for 100 companies, callable programmatically from the pipeline/agent tooling already in use.
- **Decision:** Access OpenBB via its MCP (Model Context Protocol) server rather than a hand-rolled REST client against a financial data API.
- **Alternatives considered:**
  - *Direct API integration (e.g., yfinance, a paid data vendor API)* — more control, but means writing and maintaining a bespoke client, auth handling, and rate limiting.
- **Concepts:** **MCP** standardizes how an LLM-driven agent/tool discovers and calls external capabilities (here, financial data retrieval) through a uniform protocol, instead of each integration having its own bespoke interface — the same idea as a REST API gateway, but designed for tool-use by AI agents.
- **Trade-offs:** Faster integration and consistency with the rest of the AI-assisted pipeline, at the cost of depending on OpenBB's MCP server's own rate limits and availability rather than a directly-controlled client.

## DD-007: Streamlit for the dashboard

- **Phase:** Design
- **Status:** Accepted
- **Context:** Need to visualize ESG scores against stock performance for a single researcher/small audience, not a production multi-tenant web app.
- **Decision:** Use Streamlit, a Python script-to-webapp framework.
- **Alternatives considered:**
  - *Full web stack (e.g., a frontend framework + REST/GraphQL API backend)* — gives more control over UX and scaling, but is substantial extra engineering. The audience is now defined (researchers, retail investors, portfolio managers, companies; see README "Target Users"), but there is still no concurrency or hosting requirement that would justify it. Revisit if that changes.
- **Concepts:** Illustrates matching **architectural investment to actual audience/scale** — a full client-server web architecture (with its own API design, auth, and deployment topology) is a distributed-systems problem Streamlit sidesteps entirely by running as a single Python process that renders directly from DataFrames.
- **Trade-offs:** Very fast to build and iterate, but harder to scale to many concurrent users or to decouple frontend/backend later if the audience grows beyond the current single-node assumption.

## DD-008: Corporate financial performance as the primary dataset, 2023-2025

- **Phase:** Design
- **Status:** Accepted
- **Context:** The original scope correlated ESG scores with stock price movement for the current year to date. That window is too short to contain a market-vulnerability event, which the alternative hypothesis depends on, and stock price alone is a noisy proxy for economic growth. The Forbes ranking itself is built on sales, assets, market value and profit.
- **Decision:** Make corporate financial performance (sales, profit, assets, market value) the primary dependent variable and stock price performance the secondary one. Use the top 100 companies of the Forbes Global 2000 2025 list, and a three-year window (2023-2025) for financials, stock prices and sustainability reports (up to 300 company-years).
- **Alternatives considered:**
  - *Keep stock price as primary, current year only* — cheapest, but cannot test resilience during market-vulnerability periods and ignores the metrics the Forbes ranking is built on.
  - *Five-year window (2021-2025)* — more events and more statistical power, but about 500 reports to find and extract, which conflicts with the free-tier extraction constraint (DD-010).
- **Concepts:** The unit of analysis changes from *company* (cross-section) to *company-year* (**panel data**). The model needs **time as a first-class dimension** (grain: company × fiscal year), **point-in-time alignment** between report year and fiscal year (fiscal-year ends differ, and reports are published months after year-end), and care with **selection and survivorship bias**: the top 100 are chosen on current size, which correlates with financial performance.
- **Trade-offs:** A stronger test of the hypothesis and 3× more observations (about 300 vs 100), at the cost of 3× more reports to collect and extract, currency and fiscal-year normalization of financials, and a window that is still short for "long-term value creation" claims.

## DD-009: OpenBB MCP Server and yfinance behind a provider adapter

- **Phase:** Design
- **Status:** Accepted
- **Context:** Financials and stock prices are needed for 100 companies over 2023-2025. DD-006 chose the OpenBB MCP Server, but its coverage of fundamentals depends on the underlying provider. yfinance, an unofficial Yahoo Finance API for Python, is free and easy to use, but has no SLA and no stability guarantee.
- **Decision:** Use both the OpenBB MCP Server and yfinance as sources for financials and stock prices. Put them behind a provider adapter (one internal interface, one implementation per source). Treat yfinance as a temporary solution. Store raw responses, the source name and a retrieval timestamp in Bronze, and cross-check values between sources and against the Forbes 2025 list.
- **Alternatives considered:**
  - *yfinance only* — simplest, but drops the MCP integration chosen in DD-006 and leaves a single unofficial source.
  - *OpenBB MCP Server only (DD-006)* — keeps one integration, but may not cover all four financial metrics for every company.
  - *Paid data vendor API* — more reliable and licensed, but conflicts with the free-cost constraint.
- **Concepts:** The **adapter pattern** (a **ports-and-adapters** boundary) isolates pipeline code from volatile external dependencies, so a source can be swapped without touching downstream code. Keeping raw responses in Bronze (DD-001) means a swap can be replayed rather than re-fetched. Cross-checking two independent sources is a basic **data reconciliation** control, and a retrieval timestamp is needed because vendor-side data can be revised (**non-reproducible source** problem).
- **Trade-offs:** Resilience and the ability to detect bad data, at the cost of a second integration, reconciliation logic, and handling of disagreements between sources. yfinance's unofficial status and terms of use remain a risk, tracked in README "Open Decisions".

## DD-010: Free-tier LLM and OCR extraction on an Azure AI Foundry student account

- **Phase:** Design
- **Status:** Accepted
- **Context:** Extracting ESG KPIs from up to 300 long PDF reports needs LLM calls and OCR for scanned pages. The project has no budget, so extraction must stay free. An Azure student account provides limited credit and quotas, not unlimited usage, and OCR free tiers may limit pages processed.
- **Decision:** Run LLM extraction (via LangChain) and OCR on an Azure AI Foundry student account, and design the pipeline to minimise usage: parse text locally first and OCR only pages with no text layer; retrieve evidence passages before prompting so only relevant text reaches the LLM; cache results by checksum, KPI and prompt version; throttle to quota. Keep a local OCR fallback, and check quotas and OCR page limits before processing.
- **Alternatives considered:**
  - *Paid LLM and OCR APIs* — simplest and highest quality, but violates the free constraint.
  - *Local open-source models only* — free and unlimited, but needs capable local hardware and likely lowers extraction quality on long, heterogeneous reports.
  - *Send whole reports to the LLM* — simplest prompt design, but multiplies token use and would exhaust free credit quickly.
- **Concepts:** **Cost as a design constraint**: the retrieval-then-extract design (a **RAG-style** pattern) bounds token use by the evidence size, not the document size. **Idempotent, cached processing** keyed on content hash means reruns and prompt-only changes don't repeat paid work. **Rate limiting and backpressure** keep the pipeline inside quota, and the **graceful degradation** path (a local OCR fallback) avoids a hard dependency on one vendor's free tier.
- **Trade-offs:** Zero marginal cost, at the cost of quota-driven throughput limits, a more complex pipeline (local parsing, retrieval, caching, fallback), and dependence on a student credit that can run out. Extraction quality for scanned reports may be lower with a local OCR fallback.
