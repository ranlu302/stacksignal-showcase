<div align="center">

# StackSignal

### See the market. Focus your next move.

Explore the skills employers seek, where Data Engineer opportunities appear,
and how advertised salaries compare across Australia.

[**Open live demo →**](https://stacksignal.duckdns.org) · [What you can explore](#what-you-can-explore) · [How it works](#how-it-works) · [Under the hood](#under-the-hood)

</div>

## What you can explore

### Which skills are employers asking for?

See the skills mentioned most often in the captured job advertisements. Compare their share of the sample to understand which capabilities recur across opportunities.

### Where are the opportunities—and what do they pay?

Explore observed vacancies by Australian city and compare eligible advertised annual salaries. Missing salaries stay visible as a gap; small samples are clearly marked rather than turned into misleading comparisons.

### How is broader demand changing?

Follow monthly occupation trends from Jobs and Skills Australia. This provides a wider context alongside individual job advertisements, using broader occupation groups rather than an exact count of Data Engineer roles.

### Which postings can I inspect?

The public Job explorer shows a read-only selection of real posting facts and links to the original advertisements. Personal application tracking stays in the private workspace; application stages, scores, notes and employer advertisement text are excluded from the public export.

## How it works

StackSignal brings scattered information into a consistent view: it collects source records, checks and organizes them, then calculates the comparisons shown in the dashboard. Original captures are retained so results can be reproduced and investigated.

Two sources provide different perspectives: **job advertisements** show individual opportunities, while **official occupation statistics** show broader trends. They are processed independently and presented together.

**Clear limits build trust.** Results describe the captured sample, not the entire Australian job market. The hosted application uses an allowlisted sample of real posting facts, not a representative market survey. Personal application records and provider captures are not published in this showcase.

---

## Under the hood

The sections below explain the implementation, its reliability guarantees and the work still to be verified.

### System architecture

Two independent analytical pipelines support the application: individual job postings and official occupation-demand trends. Personal application tracking is a separate private workspace.

- **Source adapters** isolate provider retrieval and mapping from processing.
- **Data pipelines** validate records and publish datasets for their respective analytics paths.
- **Metric functions** keep business rules separate from dashboard presentation.
- **The career store** serves the application tracker separately from market datasets.

## Engineering decisions

### 1. Preserve progress at the ingestion boundary

Each successful postings page is persisted before the next request begins. A later request failure leaves earlier captures available for processing. Reprocessing those captures requires no new provider request and produces the same logical records for the same inputs.

**Tradeoff:** replay is supported; resuming a mutable provider search from an exact saved cursor is not guaranteed.

### 2. Publish a consistent dataset generation

Postings, extracted skills and processing state are written into a new generation. An atomic pointer switch publishes that generation only after all writes succeed. Readers resolve the pointer once and use that generation throughout a read, preventing mixed-version results. A non-blocking process lock serializes postings writers.

**Failure behaviour:** failed publication leaves the previous generation selected. Corrupt raw JSON aborts publication instead of silently reducing the dataset.

**Scope:** these guarantees address process failures on supported local POSIX filesystems. Power-loss durability, distributed coordination and the separate occupation-trend publisher require different guarantees.

### 3. Treat metric definitions as contracts

A useful dashboard must make its denominators, exclusions and observation window clear.

- **Skill demand:** jobs mentioning a skill divided by jobs with usable descriptions; both counts use the same cohort.
- **Salary percentiles:** one configured currency, explicit annual salaries and at least 10 eligible observations. Estimated salaries are excluded by default.
- **Missing data:** missing values remain missing; hourly and daily rates are not annualized.
- **Skill extraction:** a deterministic taxonomy with word-boundary matching makes results inspectable and repeatable.
- **Coverage:** posting, retrieval and first/last observation dates describe the captured sample, not complete labour-market coverage.

### 4. Keep public facts separate from personal data

The hosted application reads a restricted export of posting facts from allowlisted employer ATS and government hosts. Advertisement text, provider captures and personal application records are excluded. The public Job explorer has no application-writing controls. Docker build checks reject private workspace files from the deployment image.

This public repository contains this README only. Application source code, tests, deployment scripts, provider captures, databases and personal application records are not distributed here.

## Verification

The private application's offline test suite covers the following behaviours. This is a summary of internal verification, not a publicly executable test suite.

- **Ingestion recovery:** interrupted pagination preserves previously captured pages.
- **Publication safety:** failures during dataset, state and pointer publication preserve the previous selected generation.
- **Concurrency:** overlapping postings writers are rejected; readers use a single generation.
- **Replay and input integrity:** repeated processing preserves logical results, while corrupt raw input prevents publication.
- **Metric boundaries:** skill-demand cohort consistency and the salary sample threshold are explicitly tested.

The private application uses automated linting, tests and fixture-based smoke checks. These checks establish behaviour on test datasets; production-scale throughput has not been benchmarked. No workflows or test artifacts are included in this showcase.

## Project objective

StackSignal is a focused project for refreshing data engineering and software engineering skills through implementation, testing and operational exercises. The goal is to build a system whose design decisions and failure behaviour can be explained convincingly in a technical interview, with evidence from the implementation.

The work prioritizes four areas:

- **Practical reliability:** preserve captured inputs, publish consistent datasets and demonstrate recovery from interrupted processing.
- **Automated testing:** turn data contracts, business rules and failure scenarios into repeatable checks, with clear limits on what each check proves.
- **Delivery:** build hands-on experience with Docker, CI and deployment, including release verification and rollback practice.
- **Operational reasoning:** diagnose stale data, invalid inputs and failed runs; document how to identify the failure, recover and verify the result.

Each milestone should produce an explainable engineering decision, a reproducible demonstration and an honest account of the remaining limitations. The public application is available. The deployment runbook records a completed automated release and an isolated failed-release exercise on 23 September 2026. That exercise covered health-check failure and recovery; it does not establish zero downtime or recovery from every deployment failure. Implementation and operational records remain private.

## Technology

- **Data processing:** Python, Polars, PyArrow / Parquet, Pydantic
- **Application:** Streamlit, SQLite
- **Delivery and quality:** Docker, pytest, Ruff, GitHub Actions

Local batch processing keeps the current execution and recovery model inspectable. A public demo is available over HTTPS. Distributed processing and international market expansion are outside the implemented scope.

## Data sources and interpretation

Provider adapters exist for TheirStack and Adzuna. Live ingestion and public display depend on the applicable permissions; Adzuna remains evaluation-only without the required written licence. The hosted public export contains selected real posting facts; its role mix reflects a personal search and its host allowlist, not complete market coverage.

The occupation-demand series is independent of the postings sample. Selected occupation groups serve as a broader demand proxy, rather than an exact count of Data Engineer vacancies.

**Attribution:** [Jobs and Skills Australia — Internet Vacancy Index](https://www.jobsandskills.gov.au/data/internet-vacancy-index). © Commonwealth of Australia, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

---

Built by [Ran Lu](https://github.com/ranlu302) · **StackSignal — Data Engineering Job Market Intelligence**
