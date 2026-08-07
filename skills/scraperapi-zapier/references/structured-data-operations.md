# Get Structured Data — Operation Matrix

Every operation available under the **Get Structured Data** action, grouped by
Category → Vendor, reflecting the official ScraperAPI Zapier integration.
Operations are surfaced dynamically under the single Get Structured Data
action, so the vendor/operation list can grow without a new action appearing.

**How the form works.** Select **Category** first, then **Operation**; the
operation-specific fields render only after both are chosen (both fields alter
the dynamic form). Field **labels** are what the user sees in the editor. The
"required" fields must be filled; the rest are optional refinements.

Two optional fields are shared by almost every operation:

- **TLD** — top-level domain (e.g. `com`, `co.uk`, `de`).
- **Country Code** — two-letter ISO country code for geo-targeting.
- **Output Format** — `JSON` (default) or `CSV`. (Redfin operations don't
  offer this; they use a **Raw** toggle instead.)

Below, shared optionals are abbreviated as *(+ TLD, Country Code, Output
Format)*. The endpoint path is shown for reference; users never type it — the
action builds it.

---

## Category: E-commerce

### Amazon

**Product details** — `/structured/amazon/product`
- **ASIN** (required) — e.g. `B08N5WRWNW`.
- *(+ TLD, Country Code, Output Format)*

**Search** — `/structured/amazon/search`
- **Query** (required) — e.g. `laptop`.
- **Page** (integer), **Sort** (Amazon `s` param), **Department** (`i` param).
- *(+ TLD, Country Code, Output Format)*

**Offers** — `/structured/amazon/offers`
- **ASIN** (required).
- **Condition**, and boolean filters: **New**, **Used (Good)**, **Used (Like
  New)**, **Used (Very Good)**, **Used (Acceptable)**.
- *(+ TLD, Country Code, Output Format)*

### eBay

**Search** — `/structured/ebay/search/v2`
- **Query** (required).
- **Page** (integer), **Items per Page** (integer), **Seller ID**,
  **Condition**.
- **Buying Format** — Buy It Now / Auction / Accepts Offers.
- **Show Only** — extra filter (e.g. `returns_accepted`, `sold_items`).
- **Sort By** — Best Match / Ending Soonest / Newly Listed / Price — Lowest /
  Price — Highest / Distance — Nearest.
- *(+ TLD, Country Code, Output Format)*

**Product details** — `/structured/ebay/product`
- **Product ID** (required) — eBay item ID.
- *(+ TLD, Country Code, Output Format)*

### Walmart

**Search** — `/structured/walmart/search`
- **Query** (required), **Page** (integer).
- *(+ TLD, Country Code, Output Format)*

**Category browse** — `/structured/walmart/category`
- **Category ID** (required), **Page** (integer).
- *(+ TLD, Country Code, Output Format)*

**Product details** — `/structured/walmart/product`
- **Product ID** (required).
- *(+ TLD, Country Code, Output Format)*

**Reviews** — `/structured/walmart/review`
- **Product ID** (required), **Page** (integer), **Sort**, **Ratings**
  (filter by rating), **Verified Purchase Only** (boolean).
- *(+ TLD, Country Code, Output Format)*

---

## Category: Search and Insights

### Google

**Search** — `/structured/google/search`
- **Query** (required).
- **Time Range** — Hour / Day / Week / Month / Year (`tbs`).
- **Include HTML** (boolean).
- *(+ TLD, Country Code, Output Format)*

**Jobs** — `/structured/google/jobs`
- **Query** (required).
- *(+ TLD, Country Code, Output Format)*

**News** — `/structured/google/news`
- **Query** (required).
- **Time Range** — Hour / Day / Week / Month / Year.
- *(+ TLD, Country Code, Output Format)*

**Shopping** — `/structured/google/shopping`
- **Query** (required).
- **Include HTML** (boolean).
- *(+ TLD, Country Code, Output Format)*

**Maps search** — `/structured/google/mapssearch`
- **Query** (required) — e.g. `pizza near Times Square`.
- **Latitude** (number), **Longitude** (number), **Zoom** (integer),
  **Include HTML** (boolean).
- *(+ TLD, Country Code)* — no Output Format field on this operation.

---

## Category: Real Estate

### Redfin

Redfin operations take a full Redfin **URL** and do not offer an Output Format
choice. For-sale and for-rent add a **Raw** toggle to return raw data instead
of parsed JSON.

**Listings for sale** — `/structured/redfin/forsale`
- **URL** (required) — full Redfin listing or search URL.
- **Raw** (boolean).
- *(+ TLD, Country Code)*

**Listings for rent** — `/structured/redfin/forrent`
- **URL** (required).
- **Raw** (boolean).
- *(+ TLD, Country Code)*

**Search** — `/structured/redfin/search`
- **URL** (required) — full Redfin search URL.
- *(+ TLD, Country Code)*

**Agent profile** — `/structured/redfin/agent`
- **URL** (required) — full Redfin agent URL.
- *(+ TLD, Country Code)*

---

## Quick reference — required field per operation

| Operation | Required input |
|-----------|----------------|
| Amazon Product details / Offers | ASIN |
| Amazon Search | Query |
| eBay Search | Query |
| eBay Product details | Product ID |
| Walmart Search | Query |
| Walmart Category browse | Category ID |
| Walmart Product details / Reviews | Product ID |
| Google Search / Jobs / News / Shopping / Maps search | Query |
| Redfin (all) | URL |
