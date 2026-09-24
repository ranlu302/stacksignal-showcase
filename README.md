# StackSignal

### Australian Data Engineer Job Market Intelligence

StackSignal brings job-posting signals and official occupation trends into a focused
dashboard for exploring skills, locations and advertised salaries across Australia.

[Open the application](https://stacksignal.duckdns.org)

## What you can explore

- **Skills:** compare how often tracked skills appear in the captured job sample.
- **Locations:** explore where observed opportunities are concentrated.
- **Salaries:** compare eligible advertised annual salaries, with missing data and
  insufficient sample sizes made explicit.
- **Occupation trends:** view broader demand patterns from the Jobs and Skills
  Australia Internet Vacancy Index.
- **Job explorer:** browse a read-only selection of posting facts and follow links
  to the original advertisements.

## Project focus

StackSignal is a practical data-engineering project focused on reliable processing,
clear data contracts, reproducible analysis and useful presentation. It separates
data collection, validation, analytical calculations and the dashboard.

Engineering priorities include preserving captured inputs, handling interrupted
processing, testing metric definitions and keeping public information separate from
personal application records.

## Interpreting the data

The hosted application uses a restricted sample of real posting facts from selected
employer and government sources. It is not a comprehensive or representative survey
of the Australian job market.

Skill percentages describe jobs with usable descriptions. Salary comparisons use
eligible annual values in one currency and are withheld when the sample is too small.
Missing salaries remain missing; hourly and daily rates are not annualized.

Occupation statistics provide a broader perspective and should not be interpreted
as an exact count of Data Engineer vacancies.

## Technology

Python · Polars · Parquet · Pydantic · Streamlit · SQLite · Docker · pytest

## Repository scope

This repository contains this README only. Application source code, implementation
files, tests, databases, provider captures and personal records remain private.
No source-code usage or redistribution permission is granted by this showcase.

## Attribution

Occupation-demand data: [Jobs and Skills Australia — Internet Vacancy Index](https://www.jobsandskills.gov.au/data/internet-vacancy-index).
© Commonwealth of Australia, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Built by [Ran Lu](https://github.com/ranlu302).
