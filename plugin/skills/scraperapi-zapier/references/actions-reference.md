# ScraperAPI Zapier Actions — Field Reference

Authoritative list of every action in the official ScraperAPI Zapier
integration, with the exact field labels users see in the Zap editor, help
text, types, and output shape, reflecting the official ScraperAPI Zapier
integration. Field **labels** are what appear in the UI; the `key` in
parentheses is the internal identifier shown for reference.

The integration provides **actions only — no triggers**. Every Zap needs a
trigger from another app before a ScraperAPI action step.

Auth is a single **API Key** (password field), validated at connect time with
a test request. Get it from `https://dashboard.scraperapi.com/`.

## Table of contents

- [Extract Data From URL](#extract-data-from-url-dataextraction)
- [Get Structured Data](#get-structured-data-getstructureddata)
- [Create Crawler Job](#create-crawler-job-crawlercreatejob)
- [Get Crawler Job Status](#get-crawler-job-status-crawlergetjobstatus)
- [Cancel Crawler Job](#cancel-crawler-job-crawlercanceljob)

---

## Extract Data From URL (`dataExtraction`)

Extract data from a single URL — HTML, Markdown, plain text, or parsed JSON.
Handles proxies, browser rendering, and CAPTCHAs.

### Input fields

| Label | Type | Required | Notes |
|-------|------|----------|-------|
| **URL** | string | ✅ | The URL to extract data from, e.g. `https://example.com`. |
| **Render JavaScript** | boolean | | Render the page with a headless browser before returning HTML. Use for JS-heavy pages. |
| **Country Code** | string | | Lowercase two-letter code (`us`, `uk`, `de`) for geo-targeting. ScraperAPI uses `uk`, not `gb`. Sent lowercased. |
| **Device Type** | choice | | `desktop` (Desktop) or `mobile` (Mobile). |
| **Premium Proxies** | boolean | | Premium residential/mobile proxies. **Mutually exclusive with Ultra Premium.** |
| **Ultra Premium** | boolean | | Most advanced bypass. **Mutually exclusive with Premium.** |
| **Output Format** | choice | | `markdown` / `text` / `csv` / `json`. Defaults to raw HTML. CSV/JSON require an autoparse-supported site. |
| **Auto Parse** | boolean | | Return structured JSON for supported sites instead of HTML. |

Validation performed by the action:
- URL is required.
- Premium + Ultra Premium both `true` → error `InvalidInput` (400).

### Output fields

| Field | Type | Description |
|-------|------|-------------|
| `url` | string | The URL that was scraped (echoed back). |
| `status_code` | integer | HTTP status ScraperAPI returned for the page. |
| `content_type` | string | Content-Type header of the response. |
| `body` | string | **The scraped content** — HTML, Markdown, text, or JSON per Output Format. |

Sample output:

```json
{
  "url": "https://example.com",
  "status_code": 200,
  "content_type": "text/html; charset=UTF-8",
  "body": "<!doctype html><html>…<h1>Example Domain</h1>…</html>"
}
```

Error mapping: 400 → BadRequest, 401 → AuthenticationFailed (bad key), 403 →
Forbidden (out of credits or plan lacks the requested feature), ≥500 →
ServerError.

---

## Get Structured Data (`getStructuredData`)

Retrieve pre-parsed JSON from a supported site category. Uses cascading
dynamic dropdowns.

### Input fields

| Label | Type | Required | Notes |
|-------|------|----------|-------|
| **Category** | choice | ✅ | `ecommerce` (E-commerce) / `search_insights` (Search and Insights) / `real_estate` (Real Estate). Alters the fields below. |
| **Operation** | choice | ✅ | Repopulated from Category. Alters the fields below. |
| *(operation-specific)* | varies | varies | Render dynamically once Operation is chosen — e.g. **ASIN**, **Query**, **Product ID**, **URL**. See `structured-data-operations.md`. |

The Category and Operation fields both have `altersDynamicFields: true`, which
is why the remaining fields only appear after both are selected. Users must
choose Category → Operation before the rest of the form renders.

### Output fields

| Field | Type | Description |
|-------|------|-------------|
| `category` | string | Human-readable category label (e.g. `E-commerce`). |
| `operation` | string | Human-readable operation label (e.g. `Amazon Product details`). |
| `status_code` | integer | HTTP status from ScraperAPI. |
| `data` | object | **The parsed result.** Comes back already parsed; nested fields (e.g. `data.pricing`) are mappable after a test run. |

Sample output:

```json
{
  "category": "E-commerce",
  "operation": "Amazon Product details",
  "status_code": 200,
  "data": {
    "name": "Example Product",
    "product_information": { "ASIN": "B08N5WRWNW" },
    "pricing": "$19.99",
    "average_rating": 4.5,
    "total_reviews": 1234
  }
}
```

`data` is parsed as JSON automatically. When Output Format is CSV (or a Redfin
`raw` option is set), `data` is the raw string instead.

Error mapping: 401 → AuthenticationFailed, 403 → Forbidden (credits), other
≥400 → BadRequest/ServerError with a response snippet.

---

## Create Crawler Job (`crawlerCreateJob`)

Start a one-off crawler job that follows links from a start URL and streams
each page result to a webhook. Returns immediately with a `jobId`.

### Input fields

| Label | Type | Required | Notes |
|-------|------|----------|-------|
| **Start URL** | string | ✅ | Where the crawl begins, e.g. `https://example.com`. |
| **URL Regex (Include)** | string | ✅ | Default `.*`. Links must match to be crawled. Tested against **path + query + fragment**, not the full URL — no scheme/host. e.g. `^/blog/`. |
| **Callback Webhook URL** | string | ✅ | Each page result (success and failure) is POSTed here. Must be a **Catch Hook in a separate Zap**. |
| **Max Depth** | integer | ⚠️ | Max link depth (start URL = depth 0). **Either Max Depth or Crawl Budget is required.** |
| **Crawl Budget** | integer | ⚠️ | Max API credits the crawl may consume. **Either Crawl Budget or Max Depth is required.** |
| **URL Regex (Exclude)** | string | | URLs whose path+query+fragment match are skipped. Same scope as Include. e.g. `/private` or `\?utm_`. |
| **Render JavaScript** | boolean | | Render each page before extracting links/content. |
| **Country Code** | string | | Lowercase two-letter code; `uk` not `gb`. Sent lowercased. |
| **Device Type** | choice | | `desktop` / `mobile`. |
| **Premium Proxies** | boolean | | Mutually exclusive with Ultra Premium. |
| **Ultra Premium** | boolean | | Mutually exclusive with Premium. |
| **Output Format** | choice | | `html` (default) / `markdown` / `text` / `csv` / `json`. CSV/JSON need an autoparse-supported site. |
| **Auto Parse** | boolean | | Structured parsing for supported sites. |

Validation performed by the action:
- Start URL and Callback Webhook URL are required.
- At least one of Max Depth / Crawl Budget is required.
- Max Depth must be a non-negative integer; Crawl Budget a positive integer.
- Premium + Ultra Premium both `true` → error.

### Output fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | e.g. `initiated`. |
| `jobId` | string | The crawler job ID — save it for status/cancel actions. |

Sample output:

```json
{ "status": "initiated", "jobId": "b1f8c8a4-1234-5678-9abc-def012345678" }
```

Crawled pages do **not** appear in this action's output — they arrive at the
Callback Webhook URL. See the two-Zap callback pattern in SKILL.md.

---

## Get Crawler Job Status (`crawlerGetJobStatus`)

Retrieve current status and counters for a crawler job.

### Input fields

| Label | Type | Required | Notes |
|-------|------|----------|-------|
| **Job ID** | string | ✅ | The `jobId` returned by Create Crawler Job. |

### Output fields

| Field | Type | Description |
|-------|------|-------------|
| `crawler_job_id` | string | The job ID. |
| `done` | integer | Pages completed. |
| `failed` | integer | Pages that failed. |
| `active` | integer | Pages currently in progress. |

Sample output:

```json
{ "crawler_job_id": "b1f8c8a4-…", "done": 12, "failed": 0, "active": 3 }
```

An unknown Job ID returns `NotFound` (404).

---

## Cancel Crawler Job (`crawlerCancelJob`)

Cancel an active crawler job by its Job ID. The action first verifies the job
exists (404 if not), then cancels it.

### Input fields

| Label | Type | Required | Notes |
|-------|------|----------|-------|
| **Job ID** | string | ✅ | The `jobId` returned by Create Crawler Job. |

### Output fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `OK` on success. |
| `message` | string | Confirmation, e.g. `Job ID: <id> has been cancelled`. |

Sample output:

```json
{ "status": "OK", "message": "Job ID: b1f8c8a4-… has been cancelled" }
```
