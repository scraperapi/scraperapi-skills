# Demand & Trend Signals

**Research question answered:** Is this market growing or shrinking, and where is consumer attention shifting?

**When to use:** User wants to understand market momentum, identify emerging sub-categories, time a product launch or market entry, or spot demand shifts before they're obvious.

**Before gathering, announce:** *"Starting demand and trend research on [category]. Looking for: funding and investment activity, content volume and recency, product market movement, and hiring patterns as a leading indicator."*

**Data gathering — call these tools, reading results as they return:**

Run these in parallel (first batch):
- Call `google_search` with `query = "[category] [year]"` — checks recency of discussion
- Call `google_search` with `query = "[category] growing OR declining OR trend"` — surfaces explicit trend claims
- Call `google_search` with `query = "[emerging sub-category or related term]"` — tests adjacent demand

Run these in parallel (second batch):
- Call `google_news` with `query = "[category] funding OR launch OR acquisition"` — money movement signals
- Call `google_news` with `query = "[category] trend OR forecast OR growth"` — analyst and press signals

Run these in parallel (third batch):
- Call `amazon_search` with `query = "[category]"` — bestseller rankings reveal current demand shape
- Call `google_shopping` with `query = "[category]"` — new entrants and price movement
- Call `google_jobs` with `query = "[category] [key role]"` — hiring as a leading indicator of where companies are investing

**Analysis rules:**

- **Leading vs. lagging indicators:** News coverage of a trend is a lagging indicator — it confirms what's already happening. Hiring patterns and new product launches are leading indicators — they suggest where investment is going before outcomes are visible. Label them accordingly.
- **Signal triangulation:** A trend claim needs at least two independent signals. One news article is not a trend. Rising Amazon bestseller rank + hiring surge + funding news = a pattern.
- **Confidence levels:** Use explicit language — "strong signal" (3+ independent sources), "emerging signal" (2 sources), "single data point" (1 source, treat as directional only).
- **Sub-category emergence:** Look for adjacent queries gaining traction that didn't exist 12–24 months ago. These are often where market white space lives.
- **Uncertainty is honest:** Demand signals from public web data are probabilistic. Present them as such — "the data suggests momentum" not "the market is growing."

**Report:** fill in [`assets/demand-trends-report.md`](../assets/demand-trends-report.md).
