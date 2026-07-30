# Frankfurter Currency Exchange API — Reference Documentation

## Portfolio Summary

This sample was written from scratch, not rewritten from an existing source. Frankfurter's own docs are genuinely well-organized, so this isn't a before/after — it's here to show a different skill: structuring complete API reference documentation from raw endpoint behavior, including verifying every example against the live API rather than copying claims from elsewhere.

Every request and response shown below was tested live against the production API before being included.

---

## Overview

Frankfurter provides free, open-source currency exchange rate data sourced from the European Central Bank (ECB) and other central banks. It covers 30+ major currencies with daily rate updates and historical data back to 1999. No API key or account is required.

**Base endpoint:**

https://api.frankfurter.dev/v1

## Quick Start

Get the latest exchange rate from USD to all supported currencies:

GET 
https://api.frankfurter.dev/v1/latest?base=USD

**Response (verified live, 2026-07-29):**
```json
{
  "amount": 1.0,
  "base": "USD",
  "date": "2026-07-29",
  "rates": {
    "AUD": 1.4409,
    "EUR": 0.87873,
    "GBP": 0.7525,
    "ZAR": 16.7499,
    "...": "..."
  }
}

Authentication
None. All endpoints are public and unauthenticated.
Endpoints
Latest Rates
Fetch the most recent working day's rates (updated daily, roughly 16:00 CET).

GET /latest

Parameter
Type
Required
Description
base
string (currency code)
No
Base currency to convert from. Defaults to EUR.
symbols
string (comma-separated codes)
No
Restrict the response to specific target currencies.

Example — latest rate, ZAR base, limited to GBP and USD:
GET https://api.frankfurter.dev/v1/latest?base=ZAR&symbols=GBP,USD

Historical Rates (single date)
Fetch rates for one specific past date.

GET /{date}
{date} must be in YYYY-MM-DD format.
Example:
GET https://api.frankfurter.dev/v1/1999-01-04?base=USD&symbols=EUR

Note: Dates are stored in UTC. If your local timezone differs from UTC, a query near midnight may return data for a different calendar day than expected. Data returned for the current day is not final and can change as new rates are published.

Fetch daily rates across a period.
GET /{start_date}..{end_date}

Both dates use YYYY-MM-DD format. Omit the end date to fetch up to the present.

Example — open-ended range (up to today):
GET https://api.frankfurter.dev/v1/2025-01-01..2025-12-31?symbols=ZAR

Example — open-ended range (up to today):
GET https://api.frankfurter.dev/v1/2026-01-01..?symbols=USD
Tip: Always pass symbols on time series requests. Without it, the response includes every supported currency for every date in the range, which gets large fast.

Available Currencies
Get the list of supported currency codes and their full names.
GET /currencies

Example:
GET https://api.frankfurter.dev/v1/currencies

Response (abbreviated):
{
  "AUD": "Australian Dollar",
  "GBP": "British Pound",
  "USD": "United States Dollar",
  "ZAR": "South African Rand"
}

Currency Conversion (client-side)
There is no dedicated conversion endpoint. Fetch the rate, then multiply in your own code:
function convert(from, to, amount) {
  return fetch(`https://api.frankfurter.dev/v1/latest?base=${from}&symbols=${to}`)
    .then(res => res.json())
    .then(data => (amount * data.rates[to]).toFixed(2));
}

convert("USD", "ZAR", 100).then(result =>
  console.log(`100 USD = ${result} ZAR`)
);
Errors
Standard HTTP status codes apply:
Status
Meaning
400
Invalid parameter or malformed request.
404
Currency, date, or resource not found.
422
Request understood but cannot be processed (e.g. a currency with no data for a requested date).

Rate Limits
There is no published request quota, but requests are rate-limited to prevent abuse. For high-volume or production use:
Cache responses where possible — rates only update once per working day.
Consider self-hosting (Frankfurter is open source and Docker-deployable) if you need guaranteed high throughput.
Notes
ECB rates are published on business days only. Weekends and EU holidays return the most recently published rate — this is expected behavior, not a bug or missing data.
The base currency defaults to EUR if omitted — always pass base explicitly if you're working with non-EUR figures, to avoid silently getting EUR-denominated data.
This is the v1 API. A v2 exists with additional features (multi-provider blending, CSV/NDJSON output, provider attribution) — v1 remains fully supported and is the simpler choice for straightforward conversion needs.

Written for portfolio purposes. All example requests and responses were tested against the live API. Verify current behavior against Frankfurter's official documentation before production use.
