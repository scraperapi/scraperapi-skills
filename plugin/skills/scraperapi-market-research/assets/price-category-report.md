# Price & Category Intelligence: [Category/Market]
*Data collected on [YYYY-MM-DD]*

## Research question
[What pricing decision does this inform?]

## Raw data collected
*Unfiltered — prices, product listings, and page content exactly as returned.*

| Source | URL | Product / Plan | Price (as shown) | Raw detail |
|--------|-----|---------------|-----------------|-----------|
| [Amazon / Google Shopping / Pricing page] | [url] | [product or plan name] | [$X] | [limits, features, or notes as shown] |

---
*Analysis below is Claude's interpretation of the above data.*

---

## Market price structure

### Where the market clusters
[Describe the distribution — e.g., "the mass market sits between $X–$Y, with a premium tier above $Z"]

### Price anchors
- **Floor** (cheapest credible option): [price + what you get + source]
- **Midmarket median**: [price range + typical features at this tier]
- **Premium ceiling**: [price + what justifies it + source]

## Pricing model breakdown
| Player/Tier | Price | Model | Key features included | Notable limits |
|-------------|-------|-------|----------------------|---------------|
| [Name] | [$X/mo] | [usage/seat/flat] | [features] | [limits] |
| [Name] | [$X/mo] | [usage/seat/flat] | [features] | [limits] |
| [Name] | [Custom] | [enterprise] | [features] | [limits] |

*Sources: [URLs for each pricing page]*

## Table stakes vs. tier-defining features
- **Table stakes** (present at all price points): [features]
- **Mid-tier unlocks** (appear at $X+): [features]
- **Premium/enterprise only**: [features]

## Hidden and variable costs
[Overage fees, required add-ons, implementation costs, annual vs. monthly delta — sourced from reviews and search]

## Pricing model signals
[What the dominant model tells you about how buyers in this market make decisions]

## Gaps and opportunities
[Underserved price tiers, missing feature/price combinations, positioning gaps]

## Data confidence notes
[Enterprise pricing is negotiated and not visible here. Published prices are a floor. Flag any paywalled or incomplete pricing pages.]

## Implications
**Immediate:** [Pricing changes or tests warranted now]
**Near-term:** [Tier or packaging decisions for this quarter]
**Strategic:** [Long-term pricing model positioning]

## Gap report
| Gap | Why it wasn't filled | Suggested follow-up |
|-----|---------------------|-------------------|
| [e.g. Pricing page returned empty] | [e.g. Page appears JS-rendered] | [e.g. Retry `scrape` with `render: true` (add `premium: true` only if it's still blocked), or check `[player] pricing site:g2.com`] |
| [e.g. Enterprise pricing not visible] | [e.g. Expected — enterprise is negotiated] | [e.g. Search `[player] pricing enterprise OR custom quote reviews` for buyer estimates] |
