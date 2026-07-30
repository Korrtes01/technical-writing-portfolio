# CelesTrak GP Data API — Reference Documentation

## Portfolio Summary

CelesTrak's official GP data documentation is technically accurate but structured as a 2,000+ word narrative article — part history lesson, part FAQ, part technical spec, all woven together as prose. The actual API contract (which parameters exist, what formats are valid, how errors are handled) is scattered across paragraphs rather than presented as a reference a developer can scan and use. I rewrote it into a standard API reference: quick start, parameter table, format options, worked examples, and error handling — same technical content, restructured for use rather than reading.

**Original source:** https://celestrak.org/NORAD/documentation/gp-data-formats.php

### Before (original, excerpt)

The source opens with several paragraphs of background on how the US government has supplied orbital tracking data since the 1970s, and how the two-line element (TLE) format's two-digit year field created Y2K-style problems. Only well into the page does it explain that queries follow a specific structure, and even then the parameter names, allowed values, and example URLs are introduced one at a time inside running prose rather than in a table. Format options, for instance, are listed as a single unbroken paragraph naming each format in turn, with the note about CSV becoming the default folded into the middle of the sentence rather than called out. There's no parameter reference table, no labeled "quick start," and error-handling guidance (what to do when a request gets blocked) is buried in an FAQ near the bottom of the page.

### After (rewrite)

---

## Overview

The CelesTrak GP (General Perturbations) API provides orbital data — including Two-Line Element sets (TLEs) and newer OMM-based formats — for satellites and other tracked objects in Earth orbit. No API key or authentication is required.

**Base endpoint:**

https://celestrak.org/NORAD/elements/gp.php

## Quick Start

Get the TLE for the International Space Station (catalog number 25544):

GET
https://celestrak.org/NORAD/elements/gp.php?CATNR=25544&FORMAT=TLE

**Response:**
ISS (ZARYA)
1 25544U 98067A   ...
2 25544  51.6400 ...

## Query Parameters

Exactly one of the following selector parameters is required per request.

| Parameter | Type | Description |
|---|---|---|
| `CATNR` | integer (1–9 digits) | Catalog number. Returns data for a single object. |
| `INTDES` | string (`yyyy-nnn`) | International Designator. Returns all objects associated with a specific launch. |
| `GROUP` | string | A named satellite group, as listed on the [CelesTrak Current Data page](https://celestrak.org/NORAD/elements/). Returns all objects in that group. |
| `NAME` | string | Satellite name (partial match supported). Returns all objects whose name contains the given string. |

`FORMAT` (optional): specifies the response format. See below. If omitted, the response defaults to **CSV** (default changed from TLE to CSV as of 2026-05-09 — pin `FORMAT` explicitly rather than relying on the default).

## Format Options

| Value | Description |
|---|---|
| `TLE` / `3LE` | Three-line element set, including the 24-character satellite name on line 0. |
| `2LE` | Two-line element set (no satellite name line). |
| `XML` | CCSDS OMM standard, XML encoding, all mandatory elements. |
| `KVN` | CCSDS OMM standard, key-value notation. |
| `JSON` | OMM keywords, JSON format. |
| `JSON-PRETTY` | Same as `JSON`, pretty-printed. |
| `CSV` | OMM keywords, CSV format. **Default if `FORMAT` is omitted.** |

## Worked Examples

**All objects from a specific launch, JSON pretty-printed:**

GET 
https://celestrak.org/NORAD/elements/gp.php?INTDES=2020-025&FORMAT=JSON-PRETTY

**All objects in a named group, TLE format:**

GET
https://celestrak.org/NORAD/elements/gp.php?GROUP=last-30-days&FORMAT=TLE

**Search by partial satellite name, CSV:**

GET 
https://celestrak.org/NORAD/elements/gp.php?NAME=STARLINK&FORMAT=CSV

## Errors & Rate Limiting

There is no published per-request rate limit, but CelesTrak enforces abuse protection:

- CelesTrak refreshes GP data roughly once every 2 hours. Polling more frequently than that returns the same data and wastes a request.
- If your IP address sends more than 100 HTTP error responses (status `301`, `403`, or `404`) within a 2-hour window, it is temporarily blocked. A `403` response includes a message explaining the block.
- Blocks clear automatically 2 hours after the offending traffic stops.

**Recommended client behavior:**
1. Check for a successful (`200`) response before using returned data.
2. On any non-`200` response, retry at most 2–3 times, then stop and log/alert — don't retry in a tight loop.
3. Cache the last successful response and reuse it if your next scheduled check is less than 2 hours old.
4. Use `https://celestrak.org` (not the deprecated `.com` domain) to avoid an unnecessary `301` redirect on every request.

## Notes

- All mandatory OMM fields are included in `XML`/`KVN`/`JSON`/`CSV` output; fields that are redundant across every record (e.g. `CENTER_NAME`, `REF_FRAME`, `TIME_SYSTEM`) are omitted rather than repeated.
- Objects without 5-digit catalog numbers (9-digit catalog IDs) are not available via `TLE`/`2LE`/`3LE` formats — use `JSON`, `XML`, `KVN`, or `CSV` for full coverage.
- For supplemental (non-SSN-sourced) GP data, a separate endpoint and parameter set applies (`SOURCE` parameter, `sup-gp.php`) — not covered here as it's a distinct product line, not a formatting variant.

---
*Rewritten for portfolio purposes. Verify current parameter behavior against the live CelesTrak documentation before using in production — this API is actively evolving (e.g., the 5→6-digit catalog number transition in progress as of mid-2026).*
