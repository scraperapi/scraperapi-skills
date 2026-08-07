# AI Parser Tools — Detailed Guide

Six MCP tools for building **reusable AI-generated parsers** that extract
structured JSON from any site that has no dedicated SDE tool. Available on
**both** the remote (hosted) and local server variants.

## When to use AI Parser

- **AI Parser** — repeatable structured extraction across many pages that share
  the same layout, when there is no dedicated SDE tool for the site. Teach a
  parser once from 1–3 example URLs, then apply it to any similar page.
- **`scrape`** — a one-off fetch of a single page.
- **Dedicated SDE tools** (`amazon_*`, `google_*`, `walmart_*`, `ebay_*`,
  `redfin_*`) — a supported marketplace or SERP. Prefer these over AI Parser
  when the site is supported.

## The two-phase, asynchronous workflow

Parser generation is **asynchronous**. Do not try to parse immediately after
creating.

1. **Create** — `ai_parser_create` with 1–3 example URLs (same layout). Returns
   a parser `id` and `version` immediately; the parser is still `GENERATING`.
2. **Poll** — `ai_parser_get_details` until `status` is `"FINISHED"`
   (statuses: `GENERATING` → `FINISHED` or `FAILED`).
3. **Parse** — `ai_parser_parse_url` on any similarly structured URL. Returns
   `{ parser, version, result }`.

Reuse a `FINISHED` parser across as many pages as you like. Editing fields with
`ai_parser_update` may create a new version (see below).

## Credit cost

Per ScraperAPI's public pricing ([docs](https://docs.scraperapi.com/)):

- **Every account gets one free parser.** Generating each *additional* parser
  (`ai_parser_create`, and field-adding/modifying `ai_parser_update`) costs
  **30,000 API credits** — budget for this before creating parsers in a loop.
- **Parsing a URL** (`ai_parser_parse_url`) uses standard scraping costs: **1
  credit** for a plain fetch, more if the `scraperParams` escalate it (e.g.
  `render` = 10, `premium` = 10, `ultraPremium` = 30). This is the recurring
  cost, so reusing one parser across many pages is what keeps it cheap.

Confirm current numbers against the public pricing docs — they can change.

## Parameter naming: remote vs. local

The two server variants use different casing for the same parameters. Both are
listed below; **remote uses camelCase, local uses snake_case**.

| Concept | Remote (hosted) | Local (Python) |
|---------|-----------------|----------------|
| parser id | `parserId` | `parser_id` |
| fetch options | `scraperParams` | `scraper_params` |
| add / modify / rename / remove | `addFields` / `modifyFields` / `renameFields` / `removeFields` | `add_fields` / `modify_fields` / `rename_fields` / `remove_fields` |
| rename target | `newName` | `new_name` |

The `scraperParams` / `scraper_params` object itself: remote keys are
`countryCode`, `sessionNumber`, `keepHeaders`, `deviceType`, `ultraPremium`,
`followRedirect`, `retry404`; local keys are `country_code`, `session_number`,
`keep_headers`, `device_type`, `ultra_premium`, `follow_redirect`, `retry_404`
(`render` and `premium` are the same in both).

Tables below use the **remote (camelCase)** names.

---

## Tool: `ai_parser_create`

Create a reusable parser from example URLs. **Asynchronous** — returns an `id`
and `version` immediately; poll `ai_parser_get_details` until `FINISHED` before
parsing.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | A name for the parser |
| `urls` | string[] (1–3) | Yes | Example URLs that share the same page layout |
| `fields` | Field[] | No | Pre-declared output schema to guide generation. If omitted, the AI infers the fields |
| `scraperParams` | object | No | ScraperAPI fetch options for the example/target pages (see below) |

---

## Tool: `ai_parser_get_details`

Get a parser's generation status, fields, and example results. Poll this after
create or a field-editing update until `status` is `"FINISHED"`.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `parserId` | string | Yes | The parser id returned by `ai_parser_create` |
| `version` | number (≥0) | No | Specific version. Defaults to the latest |

---

## Tool: `ai_parser_parse_url`

Scrape a URL and apply a `FINISHED` parser, returning structured JSON keyed by
the parser's fields (`{ parser, version, result }`). Uses standard scraping
credits (1 for a plain fetch; more with render/premium — see Credit cost above).

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `parserId` | string | Yes | The parser id to run |
| `url` | string | Yes | The URL to scrape and parse |
| `version` | number (≥0) | No | Specific version. Defaults to the latest |
| `scraperParams` | object | No | ScraperAPI fetch options for the target page |

### Best Practices
- Extracted data is under `result` in the response.
- Reuse one parser across many pages — that is the whole point of AI Parser.

---

## Tool: `ai_parser_list`

List the parsers on your account (id, name, status, version, generation time).
Takes **no parameters**. Use it to discover existing parsers before reusing one.

---

## Tool: `ai_parser_delete`

Permanently delete a parser and all its versions. **Irreversible and
destructive** — to change fields instead, use `ai_parser_update`.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `parserId` | string | Yes | The parser id to delete |

---

## Tool: `ai_parser_update`

Edit a parser's fields, creating a new version. **Adding or modifying** fields
triggers asynchronous regeneration (poll `ai_parser_get_details` until
`FINISHED`); **renaming or removing** fields is applied immediately.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `parserId` | string | Yes | The parser id to update |
| `version` | number (≥0) | No | Version to base the update on. Defaults to the latest |
| `addFields` | Field[] | No | Fields to add (async regeneration) |
| `modifyFields` | Field[] | No | Existing fields to modify (async regeneration) |
| `renameFields` | {`name`, `newName`}[] | No | Fields to rename (applied immediately) |
| `removeFields` | string[] | No | Names of fields to remove (applied immediately) |

---

## Sub-objects

### Field object (`fields`, `addFields`, `modifyFields`)

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `name` | string | Yes | Field name. Use dot notation for nesting, e.g. `products.price` |
| `description` | string | Yes | Plain-language description — guides the AI extraction |
| `type` | `"string"` \| `"number"` \| `"array"` | No | Expected type |
| `selector` | string | No | Optional CSS/XPath selector hint |

### scraperParams object (fetch options)

| Key (remote) | Type | Description |
|--------------|------|-------------|
| `render` | boolean | Enable JavaScript rendering |
| `countryCode` | string (2-letter) | Geo-targeting |
| `premium` | boolean | Use premium proxies |
| `ultraPremium` | boolean | Advanced anti-bot bypass (mutually exclusive with `premium`) |
| `sessionNumber` | number | Sticky session id to reuse an IP |
| `keepHeaders` | boolean | Forward your custom headers |
| `deviceType` | `"desktop"` \| `"mobile"` | Emulate device type |
| `followRedirect` | boolean | Follow HTTP redirects |
| `retry404` | boolean | Retry on HTTP 404 |

---

## Typical Workflow

1. **Create**: `ai_parser_create` with `name`, 1–3 example `urls`, and optional
   `fields` to shape the output.
2. **Wait**: poll `ai_parser_get_details` with the returned `parserId` until
   `status` is `"FINISHED"`.
3. **Parse**: `ai_parser_parse_url` with the `parserId` and each target `url`.
4. **Iterate** (optional): `ai_parser_update` to add/rename/remove fields;
   re-poll after add/modify.
5. **Manage**: `ai_parser_list` to find parsers, `ai_parser_delete` to remove one.
