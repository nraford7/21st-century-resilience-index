# 21CRI Dimension Redesign Spec

**Date:** 2026-03-27
**Status:** Approved (brainstorm complete)

## Thesis

The 21st century belongs to electrostates — nations that transition from fossil fuel dependency to electrified infrastructure, cheap compute, and AI-amplified productivity. The index measures who has the compound capability to execute that transition and thrive through it.

**Causal chain:** Clean energy → electrified infrastructure → cheap compute & AI → productivity gains across manufacturing, healthcare, knowledge work, services → digital governance, smart cities, smart finance → 21st century resilience.

**Prerequisites:** execution capacity for large-scale infrastructure, consistent industrial policy, capital mobilization, political continuity, material security, low climate exposure.

## The Reshaped 9 Dimensions

### X-Axis: Transition Readiness (what you're building)

#### 1. Clean Energy % (refined)
- **What it measures:** Current clean energy share of primary energy
- **Data source:** Ember Global Electricity Review 2025, OWID, Energy Institute
- **Change from v1:** Add a quality flag distinguishing legacy hydro from new-build renewables. Brazil's 50.6% hydro-inherited clean energy is structurally different from Denmark's wind/solar buildout. The flag doesn't change the score but is visible in tooltips and analysis.

#### 2. Clean Tech Production (unchanged)
- **What it measures:** Share of global clean tech manufacturing — solar, batteries, EVs, wind
- **Data source:** IEA, BloombergNEF, UNCTAD 2024
- **Change from v1:** None

#### 3. Compute & AI (unchanged)
- **What it measures:** Blended score of hyperscale data center capacity (60%) + AI readiness (40%)
- **Data source:** Synergy Research, TOP500, SIA 2024, Oxford Insights GARI 2025, Tortoise GAI 2024
- **Change from v1:** None

#### 4. Economic Complexity Index (unchanged)
- **What it measures:** Sophistication of manufacturing exports — ability to build complex systems
- **Data source:** Harvard Growth Lab 2024
- **Normalization:** (ECI + 2.0) / 4.0 × 100
- **Change from v1:** None

### Y-Axis: Power to Execute (can you actually build it)

#### 5. Execution Capacity (replaces State Capacity)
- **What it measures:** Demonstrated ability to deliver large-scale infrastructure, not governance quality or democratic legitimacy
- **Why the change:** The old State Capacity metric (WGI + FSI + SPI) rewarded democratic norms and penalized authoritarian states that are phenomenal at infrastructure delivery. China scored 52.3 despite building 48,000 km of HSR in 15 years. The thesis doesn't care about legitimacy — it cares about whether you can build.

**Sub-components:**

| Sub-component | Weight | Anchoring data | Evidence fills |
|---------------|--------|----------------|----------------|
| Infrastructure delivery track record | 35% | WEF Global Competitiveness infrastructure pillar, World Bank LPI | Custom evidence scoring: km of rail/road/grid built per decade, mega-project completion rate, construction speed benchmarks |
| Current infrastructure quality | 25% | WEF infrastructure pillar, grid reliability (SAIDI/SAIFI where available), World Bank LPI | Evidence-based quality assessment for countries outside OECD data coverage |
| Public investment deployment | 20% | OECD infrastructure investment as % GDP, IMF public investment data | Evidence-based for non-OECD: ratio of budgeted to actually deployed infrastructure spend |
| Continuity & corruption risk | 20% | Fragile States Index (continuity-relevant components), Corruption Perceptions Index, political violence indicators | Evidence-based assessment of elite capture, succession risk, institutional continuity |

**Why continuity/corruption is embedded here (not a separate dimension):** Internal instability already shows up in execution outcomes — if a country is in crisis, it can't deliver infrastructure. A separate stability dimension would double-count. But stability is a *leading indicator* that execution metrics (which are lagging) can miss. Weighting it at 20% of Execution Capacity flags risk without dominating the score. This captures pre-collapse vulnerability (pre-Arab Spring Tunisia, Sri Lanka 2022) without penalizing authoritarian stability.

**Expected reshuffling:**
- China: jumps significantly (from 52.3 → likely 75-85)
- UAE/Singapore: remain high or increase
- US: drops hard (wealth without delivery — can't permit a transmission line, California HSR on year 16)
- Rwanda: rises (remarkable execution for income level)
- UK: drops (HS2 disaster, declining rail/infrastructure quality)

#### 6. Capital Mobilization Capacity (replaces PPP)
- **What it measures:** Ability to fund and deploy capital into transition infrastructure at scale — not current wealth
- **Why the change:** PPP per capita measures how rich your citizens are. Luxembourg tops PPP but isn't building an electrostate. South Korea is mid-pack on PPP but mobilizes capital ferociously. The thesis needs investment capacity, not GDP.

**Sub-components:**

| Sub-component | Weight | Anchoring data | Evidence fills |
|---------------|--------|----------------|----------------|
| Infrastructure investment as % GDP | 30% | OECD/IMF public + private infrastructure investment data | Evidence-based for non-OECD using national budget data, World Bank project pipelines |
| Green/transition finance capacity | 25% | Climate Policy Initiative global flows, Bloomberg green bond data, IRENA renewable investment | Evidence-based for countries with limited capital markets — sovereign fund capacity, development bank access |
| Development bank & sovereign fund capacity | 25% | Sovereign fund AUM (SWF Institute), national/regional development bank capitalization | Evidence-based assessment of deployment mechanisms — Temasek, CDB, KfW, BNDES vs. countries with no institutional mechanism |
| FDI attraction in transition sectors | 20% | fDi Markets (FT), UNCTAD FDI data filtered to energy/infrastructure/tech | Evidence-based for countries where FDI data is incomplete |

**Expected reshuffling:**
- Saudi Arabia/UAE: mixed — massive sovereign funds but narrow deployment into transition
- South Korea: jumps (KDB, aggressive industrial investment, high infra spend)
- China: very high (CDB, policy banks, massive public investment as % GDP)
- US: moderate (deep capital markets but politically unable to deploy consistently)
- Nordics: moderate (smaller economies but efficient deployment)

#### 7. Transition Velocity (replaces Momentum — allows negative)
- **What it measures:** Speed and direction of electrostate transition. Explicitly allows negative values for countries whose infrastructure is decaying or transition is stalling.
- **Why the change:** Old Momentum was a grab-bag — internet penetration (saturated for developed nations), generic GDP growth, ECI movement. None of these directly measure transition speed. The new version measures only transition-relevant acceleration, and critically, it captures **decay**.

**Sub-components:**

| Sub-component | Weight | Anchoring data | Evidence fills |
|---------------|--------|----------------|----------------|
| Renewable deployment rate | 30% | IRENA Renewable Capacity Statistics 2025, IEA Renewables report — MW added/year normalized by population or GDP | N/A — data coverage is good |
| Electrification trajectory | 25% | IEA Global EV Outlook (EV adoption slope), heat pump adoption rates, grid electrification % change | Evidence-based for non-automotive electrification — rail electrification, industrial process electrification |
| Infrastructure condition delta | 25% | OECD infrastructure quality trend data, ASCE-equivalent ratings where available | Custom evidence scoring: is the built environment improving or deteriorating? Bridge condition, grid reliability trend, transport quality trajectory. US: negative. China: strongly positive. |
| Clean investment trajectory | 20% | BloombergNEF annual investment data YoY, CPI Global Landscape of Climate Finance | Evidence-based for countries outside BNEF coverage |

**Why external/regional conflict is embedded here (not a separate dimension):** Active conflict or severe regional instability depresses the velocity trajectory directly. Ukraine had positive velocity pre-2022, now massively negative — that shows up in the investment and deployment data without needing a separate conflict score. War IS the velocity change. If sanctions, conflict, or regional instability are draining resources from transition investment, the trajectory captures it.

**Expected reshuffling:**
- US: low or negative (crumbling infrastructure, inconsistent deployment, political oscillation on climate policy)
- China: very high positive (fastest renewable buildout in history, massive grid expansion, HSR)
- Vietnam: clarified — high GDP growth but coal is outpacing renewables, so velocity is lower than old Momentum suggested
- Gulf states (UAE especially): positive (starting from zero but accelerating)
- Western Europe: moderate positive (deploying but slowly, permitting bottlenecks)

### Z-Axis: Compound Exposure (what you're up against)

#### 8. Climate Exposure 2050 (unchanged)
- **What it measures:** Projected climate vulnerability under IPCC scenarios
- **Data source:** WRI 2025, ND-GAIN 2023, IPCC AR6, Swiss Re GDP-at-risk
- **Sub-components:** Sea level rise (25%), heat/wet-bulb (25%), water stress (20%), agricultural/GDP impact (15%), extreme weather (15%)
- **Scoring:** 0-100 where higher = worse. Inverted in composite formula (1 - normalized score).
- **Change from v1:** None

#### 9. Resource & Material Security (expands Resource Security)
- **What it measures:** Food, water, AND critical mineral security. The electrostate depends on materials as the petrostate depended on oil.
- **Why the change:** Old Resource Security only measured food/water self-sufficiency. The electrostate transition requires massive material inputs — lithium, cobalt, rare earths, copper, nickel, silicon. A country that can feed itself but can't source battery materials is not resource-secure for the transition.

**Sub-components:**

| Sub-component | Weight | Anchoring data | Evidence fills |
|---------------|--------|----------------|----------------|
| Food self-sufficiency | 25% | FAO food import dependency %, agricultural output data | No change from v1 |
| Water security | 20% | WRI Aqueduct water stress, per-capita renewable water resources | No change from v1 |
| Critical mineral reserves | 25% | USGS Mineral Commodity Summaries 2025, British Geological Survey, national geological survey data | Evidence-based for countries where reserves are known but incompletely surveyed |
| Mineral refining & processing capacity | 15% | USGS, IEA Critical Minerals report 2024, IRENA | Evidence-based for emerging processing capacity (Indonesia nickel, Chile lithium) |
| Supply chain diversification | 15% | Bilateral trade agreements, number of source countries for critical inputs, sanctions exposure | Evidence-based assessment of supply chain resilience vs. single-source dependency |

**Key insight:** China dominates mineral refining (80%+ of rare earth processing, 65%+ of lithium refining, 70%+ of cobalt refining) but imports raw materials. This mirrors the petrostate problem in reverse — processing dominance built on extraction dependency. The index should capture both sides.

**Expected reshuffling:**
- Australia: jumps significantly (lithium, cobalt, rare earths + food/water secure)
- Chile: jumps (world's largest copper reserves, lithium triangle)
- Canada: remains high (food/water + mining + nickel/uranium)
- DRC: interesting tension (cobalt giant but low on everything else — the "resource curse" electrostate edition)
- Japan/South Korea/Taiwan: drop (near-total material import dependency, same structural vulnerability as petrostates but for different inputs)
- China: complex (refining dominance but raw material import dependency)

## Axis Summary

| Axis | Dim | Old | New | Change |
|------|-----|-----|-----|--------|
| X | 1 | Clean Energy % | Clean Energy % (+ quality flag) | Refined |
| X | 2 | Clean Tech | Clean Tech | Unchanged |
| X | 3 | Compute & AI | Compute & AI | Unchanged |
| X | 4 | ECI | ECI | Unchanged |
| Y | 5 | State Capacity | **Execution Capacity** | Replaced |
| Y | 6 | PPP per capita | **Capital Mobilization** | Replaced |
| Y | 7 | Momentum (20yr) | **Transition Velocity (±)** | Rebuilt |
| Z | 8 | Climate Exposure 2050 | Climate Exposure 2050 | Unchanged |
| Z | 9 | Resource Security | **Resource & Material Security** | Expanded |

## Stability Treatment (documented)

Internal and external stability are **not** separate dimensions. They are embedded as sub-components to avoid double-counting:

- **Internal stability** (protests, corruption, elite capture, succession risk) → 20% of Execution Capacity as "Continuity & Corruption Risk." This is a leading indicator that execution track record (lagging) can miss. It flags countries that look good on delivery history but are structurally fragile.

- **External stability** (war, regional conflict, sanctions) → Absorbed into Transition Velocity as a natural drag. Active conflict depresses investment flows, deployment rates, and infrastructure condition. Ukraine's velocity went from positive to deeply negative in 2022 — the data captures the effect without needing a separate conflict dimension.

**Rationale:** If instability prevents execution, it shows up in Execution Capacity. If conflict drains resources, it shows up in Transition Velocity. A separate stability dimension would penalize the same root cause 3-4 times. Embedding it ensures the signal is present without distorting the composite.

## Data Sourcing Approach

**Hybrid (Option C):**
- Anchor on quantitative data from established sources where available (WEF, World Bank, OECD, USGS, IEA, IRENA, BloombergNEF)
- Fill gaps with custom evidence-based scoring using the same methodology as the existing Infrastructural Adaptability research
- Be transparent about which scores are data-anchored vs. evidence-based — flag this in tooltips and methodology notes
- All evidence-based scores must cite specific examples and sources

## Composite Formula

No structural change to the formula:

```
Composite = [(w1×ceN + w2×ctN + w3×coN + w4×ecN + w5×exN + w6×cmN + w7×tvN + w8×rmN + w9×(1-cxN)) / wT] × 100
```

**Variable key:**
- `ceN` = Clean Energy % (normalized 0-1)
- `ctN` = Clean Tech Production (normalized 0-1)
- `coN` = Compute & AI (normalized 0-1)
- `ecN` = Economic Complexity Index (normalized 0-1)
- `exN` = Execution Capacity (replaces State Capacity)
- `cmN` = Capital Mobilization (replaces PPP)
- `tvN` = Transition Velocity (replaces Momentum — allows negative, bipolar normalization)
- `rmN` = Resource & Material Security (expanded from Resource Security)
- `cxN` = Climate Exposure 2050 (inverted: 1 - cxN)

**Dimension weights (`w1`-`w9`):** User-adjustable via sliders, default equal weight (11 each, wT = 99). Same interactive weighting system as v1 — the index deliberately avoids prescribing fixed weights, letting users test how assumptions drive conclusions.

**Normalization note:** Transition Velocity allows negative values. Normalization maps the full observed range (e.g., -30 to +80) onto 0-1, where 0 = maximum decay, 1 = maximum acceleration. The zero-point (no change) sits at ~0.27, not 0.5 — stagnation is closer to failure than success.

## Implementation Sequence

1. **Research & data collection** — gather quantitative anchoring data for all new sub-components across 85 countries
2. **Evidence-based scoring** — fill gaps using Infrastructural Adaptability methodology
3. **Normalize & validate** — ensure new dimensions don't distort composite distribution
4. **Update index.html** — new data arrays, slider labels, tooltip descriptions, analysis text
5. **Update ANALYSIS.md** — rewrite findings against new dimensions
6. **Narrative pass** — update story presets and archetype definitions for new framework

## Open Questions

- Should the clean energy quality flag (legacy hydro vs. new build) affect the score or remain informational only?
- Archetype definitions will shift significantly — "Clean but Fragile" countries may reshuffle if resource/material security changes their Z-axis position. Recluster after data is in.
