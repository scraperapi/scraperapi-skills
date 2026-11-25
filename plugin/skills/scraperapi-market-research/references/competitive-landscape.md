# Competitive Landscape

**Research question answered:** Who are the significant players in this market, how do they position themselves, and where are the gaps?

**When to use:** User wants to map the market before making a positioning, entry, or investment decision. Run this module *after* Consumer Sentiment and Price & Category Intelligence when possible — positioning analysis is more useful when you already know what the market cares about and how it's priced.

**Before gathering, announce:** *"Starting competitive landscape research on [category]. Looking for: who the significant players are, how they're tiered, what they claim vs. what customers say, and where the white space is."*

**Data gathering — call these tools, reading results as they return:**

Run these in parallel (first batch — discover who's in the market):
- Call `google_search` with `query = "best [category] tools [year]"` — surfaces comparison roundups
- Call `google_search` with `query = "[category] alternatives OR competitors"` — surfaces head-to-head comparisons
- Call `google_search` with `query = "site:g2.com [category] grid OR leaders"` — surfaces analyst-style market maps
- Call `google_news` with `query = "[category] funding OR acquisition OR launch"` — surfaces recent moves

From the results, identify the 6–8 most significant players. Then fetch their homepages in parallel (standard fetch — limit to top players):
- Call `scrape` with `url = "[player-1-homepage]"`
- Call `scrape` with `url = "[player-2-homepage]"`
- Call `scrape` with `url = "[player-3-homepage]"`

Then validate stated positioning against customer perception:
- Call `google_search` with `query = "[player-1] review OR alternative OR vs"`
- Call `google_search` with `query = "[player-2] review OR alternative OR vs"`

**Analysis rules:**

- **Tier players by market position, not by your familiarity with them.** Enterprise players (complex sales, high ACVs, broad feature sets), mid-market (self-serve or inside sales, mid-range pricing), SMB/startup (low friction, low price, narrow scope), open-source/free (community-led, no direct revenue model).
- **Distinguish stated positioning from perceived positioning.** What a company says on its homepage vs. what customers say in reviews are often different. Both matter. Note the gap when it exists.
- **Build a positioning map, not just a list.** Choose two dimensions that the market actually competes on (from Consumer Sentiment data if available) and place players on them.
- **Flag recent moves.** A company that raised a round six months ago or just launched a new product line is in a different position than its current homepage suggests.
- **Note what's absent.** White space — positions nobody occupies — is often more strategically valuable than knowing who's where.

**Report:** fill in [`assets/competitive-landscape-report.md`](../assets/competitive-landscape-report.md).
