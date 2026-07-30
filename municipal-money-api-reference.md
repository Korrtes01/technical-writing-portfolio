# Municipal Money API (National Treasury, South Africa) — Reference Documentation

## Portfolio Summary

Municipal Money is National Treasury's public API for South African municipal financial data — every municipality's income, expenditure, balance sheet, capital spending, audit opinions, and more, sourced from Section 71 submissions under the Municipal Financial Management Act.

The official documentation page is enormous — thousands of lines listing every "cube" (dataset), every line item code, and every dimension for over 20 datasets. That detail is genuinely necessary as a data dictionary. But the actual mechanics of calling the API — the query parameters that let you filter, aggregate, and page through data — are demonstrated only through a couple of dense example URLs, with no dedicated parameter reference. A first-time user has to reverse-engineer the query syntax from example links rather than being told how it works.

This rewrite separates the two concerns: a compact reference for how to query the API, and a pointer to the official docs for the (unavoidably long) list of available datasets and fields.

**Original source:** https://municipaldata.treasury.gov.za/docs

---

## Overview

The Municipal Money API provides South African municipal financial data as structured, queryable datasets ("cubes") — Income and Expenditure, Balance Sheet, Capital Acquisition, Cash Flow, Audit Opinions, and others. No API key is required.

**Base endpoint:** `https://municipaldata.treasury.gov.za/api`

## Core Concept: Cubes

Each dataset is called a "cube" — think of it as a large spreadsheet of financial facts, where every row has both **dimensions** (labels like municipality, year, item) and **measures** (Rand amounts). You query a cube using three main levers: what to filter (`cut`), what to break results down by (`drilldown`), and what to calculate (`aggregates`).

Cube names include: `incexp` (Income and Expenditure), `bsheet` (Balance Sheet), `capital` (Capital Acquisition), `cflow` (Cash Flow), `audit_opinions`, and others. Most have a `_v2` counterpart used for data from the 2019-20 financial year onward (when the reporting standard changed to mSCOA). For the full list of cubes and their available items/dimensions, see the official cube reference at the source link above — that part of the source docs is a genuine data dictionary and is best consulted directly rather than duplicated here.

## Endpoint

`GET /cubes/{cube_name}/aggregate`

## Query Parameters

| Parameter | Format | Description |
|---|---|---|
| `cut` | `dimension.attribute:value` pairs, joined with `\|` | Filters results. Multiple values for the same dimension are separated by `;` inside quotes. |
| `drilldown` | `dimension.attribute` list, joined with `\|` | Breaks the aggregation down by these fields, instead of collapsing them into one total. |
| `aggregates` | e.g. `amount.sum` | The calculation to perform on the measure. |
| `page` | integer | Page number for paginated results. |
| `pagesize` | integer | Results per page. |
| `order` | `field:asc` or `field:desc`, comma-separated for multiple | Sort order. |
| `format` | `json` (default) or `csv` | Response format. |

## Worked Example

Get audited actual income/expenditure figures for Tshwane (demarcation code `TSH`), financial year 2015, broken down by item:

Request: `GET https://municipaldata.treasury.gov.za/api/cubes/incexp/aggregate?drilldown=demarcation.code|demarcation.label|item.code|item.label|item.return_form_structure&cut=financial_year_end.year:2015|amount_type.code:AUDA|financial_period.period:2015|demarcation.code:"TSH"&aggregates=amount.sum`

**Filtering by multiple item codes in one cut** — separate values with `;` inside the quoted value: `&cut=...|item.code:"2800";"5200"`

**CSV export with pagination and sorting**, for offline analysis:

`GET https://municipaldata.treasury.gov.za/api/cubes/incexp_v2/aggregate?drilldown=demarcation.code|demarcation.label|item.code|item.label&cut=financial_year_end.year:2023|amount_type.code:AUDA|demarcation.code:"KZN223"&aggregates=amount.sum&page=1&pagesize=23&order=demarcation.code:asc,item.position_in_return_form:asc&format=csv`

## Non-Aggregatable Dimensions

Some dimensions (e.g. financial year) don't make sense to aggregate across — summing across every year in the dataset produces a meaningless number that changes whenever new data is added. These are marked "non-aggregatable" in the cube reference.

**Rule of thumb:** if a dimension is non-aggregatable, it must appear either as a `cut` (to filter to one value) or a `drilldown` (to break results out by that value) — never left out entirely.

## Amount Types

The `amount_type.code` dimension distinguishes between different kinds of figures for the same line item — common values include:
- `AUDA` — Audited Actual (verified by the Auditor General)
- `ORGB` — Original Budget
- `ADJB` — Adjustments Budget

Always filter or drill down by `amount_type` — comparing unfiltered figures mixes budgeted and actual numbers together.

## Verifying Results

National Treasury also publishes quarterly Section 71 reports as PDFs/Excel. If you need to sanity-check API output, cross-reference against those reports — but be aware the API reflects the current state of the data (including later corrections), while a published report is a snapshot at the time it was generated. A mismatch on one field doesn't necessarily mean an API error; check the same field across other periods or municipalities before assuming something's wrong.

## Getting Help

Support is available at helpdesk@municipalmoney.gov.za. Error responses include an explanation message — check it before escalating.

---
*Rewritten for portfolio purposes. The full dataset/dimension reference is intentionally not duplicated here — consult the official docs at the source link above for the definitive list of cubes, item codes, and dimensions.*
