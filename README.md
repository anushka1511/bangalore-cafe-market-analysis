# Grounds for Expansion: Where Should Bangalore's Next Specialty Café Open?
**A data-driven site-selection analysis, validated against real expansion and closure decisions by Blue Tokai and Third Wave Coffee.**

---

## Table of Contents
- [Motivation](#motivation)
- [Business Problem](#business-problem)
- [Data Source](#data-source)
- [Methodology](#methodology)
- [Tech Stack](#tech-stack)
- [How to Run](#how-to-run)
- [Repository Structure](#repository-structure)
- [Findings](#findings)
- [Business Implications](#business-implications)
- [Limitations](#limitations)
- [Future Work](#future-work)

---

## Motivation
India's specialty coffee category is in the middle of a real, fast-moving expansion race. Blue Tokai (founded 2013) and Third Wave Coffee (founded 2016) have both scaled aggressively in Bangalore specifically — Third Wave alone runs 75 outlets in the city, Blue Tokai close behind — while also showing visible signs of strain: Blue Tokai's losses grew roughly 3.5x between FY22 and FY23 even as revenue climbed, and at least one of its own outlets (Koramangala 6th Block) has since closed. That combination — rapid expansion alongside real financial and operational pressure — makes site selection a genuinely high-stakes decision for these companies, not an academic question. I wanted to find out whether a purely data-driven method, built from public restaurant-listing data with zero access to either company's internal information, could independently arrive at defensible site-selection insight — and then check that method against what these companies actually did.

## Business Problem
**If a specialty café brand were deciding where to open its next Bangalore outlet, which localities show genuine unmet demand — strong existing customer engagement with cafés in the area, but no entrenched premium-tier competitor — versus which are already contested or saturated?**

This is framed as a real expansion decision, not an open-ended data exploration, because that's the only way the output is actually checkable: a real site-selection recommendation can be validated against what companies with far better information than this dataset actually chose to do.

## Data Source
**Zomato Bangalore Restaurants** (Kaggle, ~51,700 listings, 17 columns), supplemented with manual cross-referencing against Blue Tokai's and Third Wave Coffee's official store locators as of October 2026.

Relevant columns used: `name`, `location`, `rest_type`, `cuisines`, `rate`, `votes`, `approx_cost(for two people)`.

**Known limitation up front:** this dataset was collected several years before this analysis (not real-time), so it reflects Bangalore's café landscape as it stood at collection time, not the present day. This matters for interpretation — see [Limitations](#limitations) — but locality-level structural patterns (which neighborhoods are dense café/nightlife corridors) are relatively slow-moving over this kind of timeframe, which is part of why cross-checking against *current* store locations (a 2026 ground truth) against *older* demand data (the dataset) turned out to still produce a checkable, informative result rather than a meaningless one.

## Methodology

### 1. Filtering to café-type listings
Filtered the full listing set to rows where `cuisines` or `rest_type` contains "Cafe" (case-insensitive string match). This is an imperfect but reasonable proxy — sanity-checked by confirming the resulting count was in a plausible range (low thousands) relative to Bangalore's total listing volume.

### 2. Cleaning
Real scraped data is messy by default: `approx_cost(for two people)` arrives as text with comma separators (e.g. "1,200"), and `rate` arrives as a string like "4.1/5" rather than a usable number. Both were parsed into clean numeric fields; rows missing any of the core fields needed for scoring were dropped.

### 3. Defining the premium tier
The top quartile (75th percentile) of `approx_cost(for two people)` among café listings was used as the cutoff for "premium." This is a defensible, standard way to define a relative top tier without hand-picking an arbitrary rupee figure.

### 4. Building the locality-level opportunity score
For every locality with at least 5 café listings (a minimum sample-size threshold to avoid single-café noise dominating a locality's numbers), I computed:
- **`demand_intensity`** — average vote count per café in that locality, normalized 0-1 against the highest-demand locality in the dataset. Votes were used rather than count of cafés alone, because café *count* measures supply, not demand — a locality could have many mediocre cafés or few highly-engaged ones, and only the latter signals genuine unmet customer interest.
- **`pct_premium`** — the share of a locality's cafés already in the premium tier, i.e. how saturated the top price segment already is.
- **`opportunity_score = demand_intensity × (1 − pct_premium)`** — high when a locality has strong demonstrated engagement with cafés *and* a thin premium offering relative to that demand. A locality with zero cafés and zero votes scores near zero by this formula, deliberately — absence of competition alone is not treated as evidence of opportunity, only genuine demand with an underserved premium tier is.

### 5. Real-world validation
The top-ranked localities were manually cross-checked against Blue Tokai's and Third Wave Coffee's official store locator pages (current as of this analysis) to see whether the data-driven ranking lined up with, diverged from, or predicted real-world outcomes — including one case (Koramangala 6th Block) where a real outlet had since closed, giving a genuine "did the model flag the weak location correctly" test case rather than only a "did it find an open one" test.

## Tech Stack
- **Python** (pandas for data manipulation, matplotlib for visualization)
- **Google Colab** (execution environment)
- Manual secondary research (official brand store locators) for validation

## How to Run
1. Download the Zomato Bangalore Restaurants dataset from Kaggle and extract the CSV.
2. Open `bangalore_cafe_analysis.ipynb` in Google Colab.
3. Run cells top to bottom — Step 0 handles getting the CSV into the Colab session (either direct upload or Google Drive mount, both documented in the notebook).
4. Outputs: a ranked opportunity table, `top10_opportunity.png`, and `opportunity_map.png`.

## Repository Structure
```
├── README.md                     this file
├── bangalore_cafe_analysis.ipynb  full analysis notebook
├── top10_opportunity.png          bar chart of top-ranked localities
├── opportunity_map.png            scatter plot of the full demand/premium-gap space
└── market-memo.md                 companion strategy memo: café-led vs. at-home business models (Blue Tokai / Sleepy Owl / Third Wave)
```

## Findings

**Top 5 localities by opportunity score:**

| Rank | Locality | Score | Café count | Avg. votes |
|---|---|---|---|---|
| 1 | Koramangala 4th Block | 0.49 | 169 | 1,738 |
| 2 | Cunningham Road | 0.36 | 112 | 1,849 |
| 3 | Koramangala 1st Block | 0.35 | 81 | 1,131 |
| 4 | Church Street | 0.32 | 124 | 1,280 |
| 5 | Koramangala 5th Block | 0.26 | 458 | 1,357 |

**Cross-checked against official store locators (Blue Tokai, Third Wave), October 2026:**

| Locality | Blue Tokai | Third Wave | Interpretation |
|---|---|---|---|
| Koramangala 4th Block | Not present | Present (4.4★, 5,000+ reviews) | Already a proven, successful location for one brand |
| Cunningham Road | Not present | Present | Same pattern |
| **Koramangala 1st Block** | **Not present** | **Not present** | **Confirmed white space, directly adjacent to both brands' nearby blocks** |
| Church Street | Not present | Present (4.4★, 1,100+ reviews) | Already proven for one brand |
| Koramangala 5th Block | Present | Not present | Already proven for one brand |
| Koramangala 6th Block (lower-ranked) | Previously present, now closed | Not present | Model correctly ranked this below its neighbors; real-world outcome confirms it as the weaker location |

No locality in the top 5 has both brands present simultaneously — the two chains appear to be making distinct block-level bets rather than directly competing head-to-head within the same block.

*Methodological transparency note: an earlier pass of this validation, based on a secondary (Zomato cross-reference) source, incorrectly concluded Blue Tokai also operated in Koramangala 4th Block. Re-checking against the brand's own official store locator corrected this. The final table above reflects the corrected, primary-source validation.*

## Business Implications
The opportunity score — built entirely from public engagement and pricing data, with no access to either company's internal information — independently reproduced three distinct real-world outcomes: a confirmed gap (Koramangala 1st Block), two confirmed successes (4th and 5th Block), and one confirmed failure (6th Block). That is a stronger claim than simply "identified an open location" — it demonstrates the scoring method has genuine discriminative validity across success, failure, and unexploited opportunity, using only data any analyst could access before a real decision is made. For a new entrant evaluating Bangalore, Koramangala 1st Block is the single most defensible recommendation this analysis produces: demonstrated nearby demand, zero incumbent presence from either major chain.

## Limitations
- The underlying dataset predates the real-time validation step by several years; the analysis should be read as a test of the scoring method's predictive signal, not a live, current-day expansion recommendation.
- Votes and ratings are engagement proxies, not actual footfall, revenue, or profitability data — a locality can show high engagement without the underlying unit economics necessarily working for a new entrant.
- The 75th-percentile premium threshold is one reasonable definition among several possible ones; results were not stress-tested against alternative threshold choices in this version.
- "Locality" granularity (Zomato's own locality labels) may not perfectly align with how a real estate/expansion team would define a catchment area.

## Future Work
- Stress-test the opportunity formula against alternative weightings to check how sensitive the ranked list is to the specific formula chosen.
- Extend the same method to a less-saturated Tier-2 Indian city, where genuine white space is more plausible at scale.
- Incorporate rent/real-estate cost data, where available, to move from a pure-demand opportunity score toward a fuller unit-economics-informed recommendation.
