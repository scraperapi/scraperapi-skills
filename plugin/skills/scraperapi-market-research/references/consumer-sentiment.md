# Consumer Sentiment Analysis

**Research question answered:** What do people who publicly discuss this category want, fear, and complain about?

**When to use:** User wants to understand customer needs, pain points, unmet demands, or sentiment within a product category or market segment — not just about one company.

**Before gathering, announce:** *"Starting consumer sentiment research on [category]. Looking for: what the market values, recurring complaints, unmet needs, and switching mentions. Will search G2, Reddit, Capterra, and product review platforms."*

**Data gathering — call these tools, reading results as they return:**

Run these in parallel (first batch):
- Call `google_search` with `query = "[category] review site:g2.com OR site:capterra.com"` — surfaces software review pages
- Call `google_search` with `query = "[category] complaints OR problems OR wish site:reddit.com"` — surfaces Reddit discussions
- Call `google_search` with `query = "best [category] [year] reviews"` — surfaces roundup content

From the URLs returned, select the 2–3 most information-dense sources and fetch them (second batch, run in parallel):
- Call `scrape` with the G2 category grid URL, with `render: true` (G2 builds the grid with JavaScript; add `premium: true` only if the rendered request is still blocked)
- Call `scrape` with the most relevant Reddit thread URL (standard fetch)
- Call `scrape` with the Capterra category URL (standard fetch)

For physical product categories, add (run in parallel with the first batch):
- Call `amazon_search` with `query = "[category]"` — top products reveal what the market buys
- Call `walmart_search` with `query = "[category]"` — the mass-market view of the same category

Then, using IDs from those results:
- Call `amazon_product` with `asin` = the ASIN of the top-ranked product — rating and review summary
- Call `walmart_review` with `productId` = the ID of a top Walmart product, and `sort: "recent"` — cross-platform sentiment, newest first

Then pull a recency pass:
- If a review page you fetched offers a "most recent" sort, `scrape` that view too. Recent and top-rated reviews often diverge, and the difference shows where the market is moving.

**Analysis rules:**

- **Volume threshold:** With fewer than 15 distinct data points, label every finding as directional. A theme drawn from a handful of reviews can be one vocal customer.
- **Thematic coding:** Don't list quotes. Identify recurring themes. A theme earns a place in the report when it appears independently across at least 2–3 sources or is mentioned by a meaningful share of reviewers.
- **Frequency vs. severity:** A complaint mentioned often but described mildly is different from one mentioned rarely but described as a dealbreaker. Note both dimensions.
- **Recency weighting:** Prioritize the past 12 months. If most data is older, say so — it may mean the market has shifted or reviews have stagnated.
- **Verified vs. unverified:** G2 and Capterra mark verified purchasers. Weight these more heavily. A sudden spike in review volume often signals a review campaign, not organic sentiment growth.
- **Complaint categorization:** Separate complaints by root cause — reliability/performance, pricing/value, support quality, documentation/onboarding, missing features. Each category points to a different kind of market gap.
- **Saturation check:** When new sources stop introducing new themes, you have enough data. Note this in the report.
- **Sampling caveat:** Review platforms over-represent customers with strong opinions. The silent majority — satisfied or indifferent — doesn't leave reviews. Flag this limitation in the output.

**Report:** fill in [`assets/consumer-sentiment-report.md`](../assets/consumer-sentiment-report.md).
