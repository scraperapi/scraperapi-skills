# Price & Category Intelligence

**Research question answered:** How is this market structured on price, and what do customers actually get at each tier?

**When to use:** User wants to understand category pricing before setting or adjusting their own prices, identify underserved price tiers, or map how features correlate with price across a market.

**Important:** This module maps the market's price structure — it's not a competitor pricing comparison. The goal is to understand where the market clusters, where gaps exist, and what the price/value trade-offs are across the category.

**Before gathering, announce:** *"Starting price and category research on [category]. Looking for: price distribution, pricing models in use, feature/price correlation, and hidden costs. Will check product platforms and scrape pricing pages for the top players."*

**Data gathering — call these tools, reading results as they return:**

For physical product categories, run in parallel:
- Call `amazon_search` with `query = "[category]"` — rankings and price spread
- Call `google_shopping` with `query = "[category]"` — price distribution across sellers
- Call `walmart_search` with `query = "[category]"` — mass-market price anchors
- Call `ebay_search` with `query = "[category]"` — secondary market and price floor signals

For SaaS/software categories, first identify the top players:
- Call `google_search` with `query = "[category] pricing"` — find pricing pages for the top 4–6 players

Then scrape their pricing pages in parallel (most pricing pages are static HTML, so start without `render`; 1 credit each):
- Call `scrape` with `url = "[player-a]/pricing"`
- Call `scrape` with `url = "[player-b]/pricing"`
- Call `scrape` with `url = "[player-c]/pricing"`

Then gather third-party pricing commentary:
- Call `google_search` with `query = "[category] pricing review OR hidden costs OR expensive"`
- Call `google_search` with `query = "[category] affordable OR cheap OR free alternative"`

**Analysis rules:**

- **Map the distribution, not just the range:** Where does pricing cluster? A market with most products between $20–40 and a few outliers at $200+ tells a different story than a market evenly spread from $5–$500.
- **Identify the pricing model in use:** Per-seat, usage-based, flat-rate, freemium, enterprise-only. The model signals what the seller believes buyers optimize for.
- **Feature/price correlation:** What do you get at each tier? Which features are table stakes (present at every price point) vs. tier-defining (unlock at a specific price)?
- **Price anchors:** What's the cheapest credible option? What's the most expensive? These anchors define how buyers think about "normal."
- **Hidden cost signals:** Search and review data often surface costs not visible on pricing pages — overage fees, implementation costs, required add-ons. Include these.
- **Note what you can't see:** Enterprise pricing is almost always negotiated. Published prices are a floor, not the actual number. Flag this.

**Report:** fill in [`assets/price-category-report.md`](../assets/price-category-report.md).
