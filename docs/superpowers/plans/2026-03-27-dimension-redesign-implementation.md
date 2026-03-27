# Dimension Redesign Implementation Plan

> **For Claude:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace PPP with Capital Mobilization, State Capacity with Execution Capacity, rebuild Momentum as Transition Velocity (±), and expand Resource Security to include critical minerals — across all 85 countries, the visualization, analysis, and narratives.

**Architecture:** Four new/rebuilt dimensions require: (1) research and scoring CSVs with sourcing, (2) updated JS data arrays and computation logic in the monolithic index.html, (3) rewritten analysis and narrative text. No new files beyond CSVs — everything lives in index.html.

**Tech Stack:** Vanilla JS, Three.js r128, single-file HTML app. No build step. CSV data files for research backing.

**Spec:** `docs/superpowers/specs/2026-03-27-dimension-redesign.md`

---

## Chunk 1: Research & Data Collection

Four parallel research tasks — one per changed dimension. Each produces a CSV with 85 country scores, sub-component breakdowns, data sources, and evidence notes.

### Task 1: Research Execution Capacity scores (85 countries)

**Files:**
- Create: `execution_capacity_scores.csv`

**Methodology:** Hybrid scoring per spec. Each country gets four sub-scores weighted 35/25/20/20:
- Infrastructure delivery track record (35%) — WEF GCI infrastructure pillar as anchor, evidence-based delivery speed/mega-project assessment
- Current infrastructure quality (25%) — WEF infrastructure pillar, World Bank LPI, SAIDI/SAIFI grid reliability
- Public investment deployment (20%) — OECD/IMF infrastructure investment as % GDP, evidence-based deployment ratio
- Continuity & corruption risk (20%) — CPI (Transparency International), FSI continuity-relevant components, political violence indicators

**CSV format:**
```
Country,Execution_Capacity_Score,Delivery_Track_Record,Infrastructure_Quality,Public_Investment_Deployment,Continuity_Risk,Anchoring_Sources,Evidence_Notes
```

- [ ] **Step 1: Web-search quantitative anchoring data**
  - WEF Global Competitiveness infrastructure pillar scores (85 countries)
  - World Bank Logistics Performance Index 2023 (85 countries)
  - OECD infrastructure investment as % GDP (OECD members + available non-OECD)
  - Transparency International CPI 2024 (85 countries)
  - Grid reliability SAIDI/SAIFI data where available

- [ ] **Step 2: Build evidence-based scores for delivery track record**
  - Use existing `infrastructural_adaptability_research.csv` as starting foundation (62 countries already scored)
  - Research remaining ~23 countries using same methodology
  - Focus on: km of rail/road/grid built per decade, mega-project completion rates, construction speed benchmarks
  - Recalibrate existing scores: strip out democratic governance bias, reward pure execution (China should score 85-95 on delivery, not 90 which includes governance)

- [ ] **Step 3: Score all 85 countries**
  - Compute weighted composite: (delivery × 0.35) + (quality × 0.25) + (investment × 0.20) + (continuity × 0.20)
  - Normalize to 0-100 scale
  - Cross-check against spec expectations: China 75-85, UAE high, US drops, Rwanda rises
  - Flag which sub-scores are data-anchored vs evidence-based in Evidence_Notes column

- [ ] **Step 4: Write CSV**
  - All 85 countries, sorted alphabetically
  - Every cell must have a value (no blanks)
  - Evidence_Notes column cites specific examples for evidence-based scores

- [ ] **Step 5: Commit**
  ```bash
  git add execution_capacity_scores.csv
  git commit -m "Add Execution Capacity research data for 85 countries"
  ```

### Task 2: Research Capital Mobilization scores (85 countries)

**Files:**
- Create: `capital_mobilization_scores.csv`

**Methodology:** Hybrid scoring per spec. Four sub-scores weighted 30/25/25/20:
- Infrastructure investment as % GDP (30%) — OECD/IMF data, national budget data for non-OECD
- Green/transition finance capacity (25%) — CPI global flows, IRENA renewable investment data
- Development bank & sovereign fund capacity (25%) — SWF Institute AUM, development bank capitalization
- FDI attraction in transition sectors (20%) — UNCTAD FDI data, fDi Markets

**CSV format:**
```
Country,Capital_Mobilization_Score,Infra_Investment_Pct_GDP,Green_Finance_Capacity,Dev_Bank_Sovereign_Fund,FDI_Transition,Anchoring_Sources,Evidence_Notes
```

- [ ] **Step 1: Web-search quantitative anchoring data**
  - OECD infrastructure investment data (public + private)
  - IMF public investment data for non-OECD
  - Climate Policy Initiative Global Landscape of Climate Finance (country breakdown)
  - IRENA renewable energy investment by country
  - SWF Institute sovereign wealth fund rankings
  - UNCTAD World Investment Report 2024 (FDI flows)

- [ ] **Step 2: Build evidence-based scores for deployment mechanisms**
  - Score development bank capacity: Temasek/GIC (Singapore), CDB/AIIB (China), KfW (Germany), KDB (South Korea), BNDES (Brazil), etc.
  - Score sovereign fund deployment into transition: Norway's GPFG, UAE's Mubadala/ADIA, Saudi PIF
  - For countries with limited capital markets: assess access to multilateral development bank funding (World Bank, ADB, AfDB)
  - Score FDI attraction evidence where quantitative data is incomplete

- [ ] **Step 3: Score all 85 countries**
  - Compute weighted composite: (infra_invest × 0.30) + (green_finance × 0.25) + (dev_bank × 0.25) + (fdi × 0.20)
  - Normalize to 0-100 scale
  - Cross-check against spec expectations: China very high, South Korea jumps, Nordics moderate, US moderate
  - Flag data-anchored vs evidence-based

- [ ] **Step 4: Write CSV**

- [ ] **Step 5: Commit**
  ```bash
  git add capital_mobilization_scores.csv
  git commit -m "Add Capital Mobilization research data for 85 countries"
  ```

### Task 3: Research Transition Velocity scores (85 countries)

**Files:**
- Create: `transition_velocity_scores.csv`

**Methodology:** Hybrid scoring per spec. Four sub-scores weighted 30/25/25/20. **This dimension allows negative values** — a country whose infrastructure is decaying or transition stalling scores negative.

Sub-components:
- Renewable deployment rate (30%) — IRENA MW added/year normalized by GDP or population
- Electrification trajectory (25%) — IEA EV adoption slope, heat pump adoption, grid electrification % change
- Infrastructure condition delta (25%) — is built environment improving or deteriorating?
- Clean investment trajectory (20%) — BloombergNEF/CPI YoY investment change

**CSV format:**
```
Country,Transition_Velocity_Score,Renewable_Deployment_Rate,Electrification_Trajectory,Infrastructure_Condition_Delta,Clean_Investment_Trajectory,Anchoring_Sources,Evidence_Notes
```

- [ ] **Step 1: Web-search quantitative anchoring data**
  - IRENA Renewable Capacity Statistics 2025 (MW added per year, all 85 countries)
  - IEA Global EV Outlook 2025 (EV sales share by country)
  - IEA heat pump adoption data (primarily OECD)
  - BloombergNEF annual clean energy investment by country
  - Use existing `clean_energy_share_10yr_change.csv` and `clean_energy_share_20yr_change.csv` as inputs for renewable trajectory

- [ ] **Step 2: Build evidence-based scores for infrastructure condition delta**
  - US: negative — ASCE 2021 Report Card grade C-, 43% roads poor/mediocre, 7.5% bridges structurally deficient, grid reliability declining (SAIDI increasing), California HSR 16 years no service
  - China: strongly positive — 48,000 km HSR, grid expansion, 278 GW renewable added 2024
  - UK: negative to flat — HS2 descoped, rail delays increasing, infrastructure quality declining per WEF
  - Gulf states: positive — massive new-build programs
  - Score each country on trajectory: -50 (severe decay) to +100 (rapid improvement)
  - External conflict absorbed here: Ukraine deeply negative post-2022, Syria/Libya/Yemen deeply negative

- [ ] **Step 3: Score all 85 countries on raw bipolar scale**
  - Raw scores can range approximately -30 to +80
  - Compute weighted composite of sub-scores
  - Do NOT normalize to 0-100 yet — keep raw bipolar scores in CSV
  - Normalization happens in index.html using observed min/max

- [ ] **Step 4: Write CSV**

- [ ] **Step 5: Commit**
  ```bash
  git add transition_velocity_scores.csv
  git commit -m "Add Transition Velocity research data for 85 countries (bipolar scale)"
  ```

### Task 4: Research Resource & Material Security scores (85 countries)

**Files:**
- Modify: expand existing `resource_security_data.csv` or create new `resource_material_security_scores.csv`

**Methodology:** Hybrid scoring per spec. Five sub-scores weighted 25/20/25/15/15:
- Food self-sufficiency (25%) — existing data from `resource_security_data.csv`
- Water security (20%) — existing data from `resource_security_data.csv`
- Critical mineral reserves (25%) — USGS Mineral Commodity Summaries 2025, BGS
- Mineral refining & processing capacity (15%) — USGS, IEA Critical Minerals 2024
- Supply chain diversification (15%) — trade agreement diversity, source country count, sanctions exposure

**CSV format:**
```
Country,Resource_Material_Security_Score,Food_Self_Sufficiency,Water_Security,Critical_Mineral_Reserves,Mineral_Refining_Capacity,Supply_Chain_Diversification,Anchoring_Sources,Evidence_Notes
```

- [ ] **Step 1: Extract existing food/water scores from resource_security_data.csv**
  - Map existing Food_Import_Dep_Pct and Water_Stress_Level to 0-100 sub-scores
  - These become the food (25%) and water (20%) sub-components directly

- [ ] **Step 2: Web-search critical minerals data**
  - USGS Mineral Commodity Summaries 2025: reserves of lithium, cobalt, rare earths, copper, nickel, graphite, silicon by country
  - IEA Critical Minerals report 2024: refining/processing share by country
  - Key data points: Australia (lithium #1, rare earths #2), Chile (copper #1, lithium #2), DRC (cobalt #1), China (rare earth refining 60%+, lithium refining 65%+, cobalt refining 70%+), Indonesia (nickel #1)

- [ ] **Step 3: Score critical mineral reserves (25%)**
  - Score based on: number of critical minerals with significant reserves, global share of each
  - Minerals weighted by transition importance: lithium, cobalt, copper, nickel, rare earths, graphite, silicon, manganese
  - Countries with diverse, large reserves score highest (Australia, Canada, Brazil, China, US)
  - Countries with zero reserves score 0 (Singapore, Qatar, Kuwait, most small nations)

- [ ] **Step 4: Score mineral refining capacity (15%)**
  - China dominates: 60-80% of most critical mineral refining
  - Other significant: Indonesia (nickel smelting), Chile (lithium processing), Japan (specialty processing)
  - Score based on share of global processing capacity across key minerals

- [ ] **Step 5: Score supply chain diversification (15%)**
  - Number of bilateral/multilateral mineral supply agreements
  - Diversity of import sources (vs single-source dependency on China)
  - Sanctions exposure risk (Russia, DRC, etc.)
  - Evidence-based for most countries

- [ ] **Step 6: Compute composite and write CSV**
  - Weighted composite: (food × 0.25) + (water × 0.20) + (reserves × 0.25) + (refining × 0.15) + (diversification × 0.15)
  - Normalize to 0-100
  - Cross-check: Australia jumps, Chile jumps, Japan/SK/Taiwan drop, China complex

- [ ] **Step 7: Commit**
  ```bash
  git add resource_material_security_scores.csv
  git commit -m "Add Resource & Material Security research data for 85 countries"
  ```

---

## Chunk 2: Update index.html — Data Arrays & Computation

### Task 5: Add clean energy quality flags to raw data

**Files:**
- Modify: `index.html` — lines ~1759-1906 (raw data array)

- [ ] **Step 1: Identify legacy-hydro countries**
  - Countries where >50% of clean energy is legacy hydropower: Brazil, Norway, Iceland, Peru, Colombia, Paraguay, Costa Rica, New Zealand, Austria, Switzerland, Canada, Venezuela
  - Countries with modern buildout: Denmark, Germany, Netherlands, UK, Chile, Australia, Ireland, Portugal, Finland, Sweden (wind/solar dominant)

- [ ] **Step 2: Add `hydroLegacy` boolean flag to each raw data entry**
  - Add `hy:true` or `hy:false` to each country object in the `raw` array
  - This flag is informational only (does not affect score per spec)
  - Will be displayed in tooltips and detail panel

- [ ] **Step 3: Commit**
  ```bash
  git add index.html
  git commit -m "Add hydro legacy flags to country data"
  ```

### Task 6: Replace data fields and normalization for 4 changed dimensions

**Files:**
- Modify: `index.html`
  - Raw data array (~line 1759): replace `ppp` → `cm`, `sc` → `ex`, `mo` → `tv`, `rs` → `rm`
  - Normalization constants (~line 1804): replace PPP/MO constants with new ranges
  - Normalization functions (~line 1914): replace `nPPP()` with `nCM()`, add `nTV()` for bipolar normalization
  - Composite calculation (~line 2004): update field references

- [ ] **Step 1: Replace raw data field names and values**
  - For each of 85 country entries in `const raw=[...]`:
    - `ppp: <old_value>` → `cm: <capital_mobilization_score>` (from CSV Task 2)
    - `sc: <old_value>` → `ex: <execution_capacity_score>` (from CSV Task 1)
    - `mo: <old_value>` → `tv: <transition_velocity_score>` (from CSV Task 3, raw bipolar)
    - `rs: <old_value>` → `rm: <resource_material_security_score>` (from CSV Task 4)
  - Keep `ce`, `ct`, `co`, `eci`, `cx`, `pop`, `desc` unchanged for now (desc updated in Task 11)

- [ ] **Step 2: Update normalization constants**
  Replace at ~line 1804:
  ```javascript
  // OLD:
  const LN_MIN=Math.log(1150),LN_MAX=Math.log(150508),LN_R=LN_MAX-LN_MIN;
  const MO_MIN=23.5,MO_MAX=77.6,MO_R=MO_MAX-MO_MIN;

  // NEW:
  // Capital Mobilization: 0-100 scale, no special normalization needed
  // Transition Velocity: bipolar scale, needs min/max from data
  const TV_MIN=-30,TV_MAX=80,TV_R=TV_MAX-TV_MIN;  // adjust after data is in
  ```
  Remove `LN_MIN/LN_MAX/LN_R` (PPP log normalization no longer needed).
  Keep `ECI_MIN/ECI_MAX/ECI_R` unchanged.

- [ ] **Step 3: Update normalization functions**
  Replace at ~line 1914:
  ```javascript
  // OLD:
  function nPPP(v){return Math.max(0,Math.min(1,(Math.log(v)-LN_MIN)/LN_R));}
  function nMO(v){return Math.max(0,Math.min(1,(v-MO_MIN)/MO_R));}

  // NEW:
  function nTV(v){return Math.max(0,Math.min(1,(v-TV_MIN)/TV_R));}
  ```
  Remove `nPPP()` and `nMO()`. Capital Mobilization (0-100) and Execution Capacity (0-100) use simple `v/100` normalization inline.

- [ ] **Step 4: Update compute() function**
  At ~line 2034, update the data mapping:
  ```javascript
  // OLD:
  const ceN=r.ce/100,ppN=nPPP(r.ppp),ctN=Math.max(0,r.ct)/100,coN=r.co/100,ecN=nECI(r.eci),scN=r.sc/100,moN=nMO(r.mo);
  const rsN=r.rs/100;

  // NEW:
  const ceN=r.ce/100,ctN=Math.max(0,r.ct)/100,coN=r.co/100,ecN=nECI(r.eci);
  const exN=r.ex/100,cmN=r.cm/100,tvN=nTV(r.tv);
  const rmN=r.rm/100;
  ```

  Update composite formula:
  ```javascript
  // OLD:
  const comp=((w1*ceN+w2*ppN+w3*ctN+w4*coN+w5*ecN+w6*scN+w7*moN+w8*rsN+w9*(1-cxN))/wT)*100;

  // NEW:
  const comp=((w1*ceN+w2*ctN+w3*coN+w4*ecN+w5*exN+w6*cmN+w7*tvN+w8*rmN+w9*(1-cxN))/wT)*100;
  ```

  Update eSov/ePow axis calculations:
  ```javascript
  // OLD:
  const xW=w1+w3+w4+w5||1;
  const yW=w2+w6+w7||1;
  const eSov=(w1*ceN+w3*ctN+w4*coN+w5*ecN)/xW*100;
  const ePow=(w2*ppN+w6*scN+w7*moN)/yW*100;

  // NEW:
  const xW=w1+w2+w3+w4||1;
  const yW=w5+w6+w7||1;
  const eSov=(w1*ceN+w2*ctN+w3*coN+w4*ecN)/xW*100;
  const ePow=(w5*exN+w6*cmN+w7*tvN)/yW*100;
  ```

  Update the spread object to use new field names:
  ```javascript
  // OLD:
  ceS:ceN*100,ppS:ppN*100,ctS:ctN*100,coS:coN*100,ecS:ecN*100,scS:scN*100,moS:moN*100,rsS:rsN*100,cxS:cxN*100

  // NEW:
  ceS:ceN*100,ctS:ctN*100,coS:coN*100,ecS:ecN*100,exS:exN*100,cmS:cmN*100,tvS:tvN*100,rmS:rmN*100,cxS:cxN*100
  ```

- [ ] **Step 5: Update prof() function and 3D Z-axis position calculation**
  Replace references to `ppN`, `scN`, `moN`, `rsN` with `cmN`, `exN`, `tvN`, `rmN` in:
  - The profile/breadth calculation (`prof()` ~line 1918)
  - The 3D Z-axis sphere positioning (~line 2479): `c.rsN` → `c.rmN` in the `resilienceN` calculation:
    ```javascript
    // OLD:
    const resilienceN = (w9*(1-c.cxN) + w8*c.rsN) / (w8+w9 || 1);
    // NEW:
    const resilienceN = (w9*(1-c.cxN) + w8*c.rmN) / (w8+w9 || 1);
    ```

- [ ] **Step 6: Verify compute() produces valid scores**
  - Open index.html in browser
  - Check that composite scores render for all 85 countries
  - Check that no NaN or Infinity values appear
  - Check that rankings list populates

- [ ] **Step 7: Commit**
  ```bash
  git add index.html
  git commit -m "Replace PPP/StateCapacity/Momentum/ResourceSec with new dimensions in compute logic"
  ```

### Task 7: Update slider labels, weight IDs, and axis descriptions

**Files:**
- Modify: `index.html`
  - Slider HTML (~lines 1478-1535)
  - Axis group labels
  - CSS color variables if dimension colors change

- [ ] **Step 1: Update weight slider labels in HTML**
  The sliders need new labels. The weight IDs (`w1`-`w9`) can stay the same but their labels and ordering must match the new dimensions:

  ```
  w1 → "Energy" (unchanged)
  w2 → "Clean Tech" (was PPP — relabel)
  w3 → "Compute & AI" (unchanged label, was w4 position)
  w4 → "Complexity" (unchanged label, was w5 position)
  w5 → "Execution Cap." (new — replaces State Cap.)
  w6 → "Capital Mob." (new — replaces PPP)
  w7 → "Velocity (±)" (new — replaces Momentum)
  w8 → "Materials" (new — replaces Resource Sec.)
  w9 → "Climate Exp." (unchanged)
  ```

  **IMPORTANT:** The weight numbering (w1-w9) must match the composite formula in Task 6. Map carefully:
  - w1 = Clean Energy (ceN) — X axis
  - w2 = Clean Tech (ctN) — X axis
  - w3 = Compute & AI (coN) — X axis
  - w4 = ECI (ecN) — X axis
  - w5 = Execution Capacity (exN) — Y axis
  - w6 = Capital Mobilization (cmN) — Y axis
  - w7 = Transition Velocity (tvN) — Y axis
  - w8 = Resource & Material Security (rmN) — Z axis
  - w9 = Climate Exposure (cxN) — Z axis

- [ ] **Step 2: Update axis group headers**
  - X axis header: "X — Transition Readiness (What you're building?)" (was "What you have?")
  - Y axis header: "Y — Power to Execute (Can you build it?)" (was "What you've proven?")
  - Z axis header: "Z — Compound Exposure (What you're up against?)" (was "How resilient?")

- [ ] **Step 3: Update CSS color variable names**
  If dimension CSS variables reference old names (e.g., `--ppp`, `--state`, `--momentum`, `--resource`), rename to `--cleantech`, `--execution`, `--capital`, `--velocity`, `--materials`. Check the `:root` CSS block (~line 10-40) and all references.

- [ ] **Step 4: Commit**
  ```bash
  git add index.html
  git commit -m "Update slider labels, axis descriptions, and CSS variables for new dimensions"
  ```

### Task 8: Update tooltip, detail panel, and comparison rendering

**Files:**
- Modify: `index.html`
  - Tooltip rendering (~lines 2650-2684)
  - `openDetail()` function (~lines 2087-2146)
  - `renderComparison()` function (~lines 2147+)
  - Mini-bar legend in rankings panel

- [ ] **Step 1: Update tooltip labels (BOTH renderers)**
  There are **two** tooltip renderers — one for 2D SVG (~line 2526-2540) and one for 3D canvas (~line 2660-2673). Both must be updated.
  Replace in BOTH tooltip HTML generators:
  ```javascript
  // OLD labels and field references:
  'PPP' → c.ppS
  'State Cap.' → c.scS
  'Momentum' → c.moS
  'Resource Sec.' → c.rsS

  // NEW:
  'Exec. Cap.' → c.exS
  'Capital Mob.' → c.cmS
  'Velocity' → c.tvS
  'Materials' → c.rmS
  ```
  Also update CSS color vars in both: `--ppp` → `--capital`, `--state` → `--execution`, `--momentum` → `--velocity`, `--resource` → `--materials`.
  Also add hydro legacy indicator: if `c.hy`, append " (hydro)" after Energy value.

- [ ] **Step 2: Update detail panel dimension bars**
  In `openDetail()` (~line 2120), update the `dims` array:
  ```javascript
  const dims=[
    {key:'energy',label:'Energy',val:c.ceS},
    {key:'cleantech',label:'Clean Tech',val:c.ctS},
    {key:'compute',label:'Compute & AI',val:c.coS},
    {key:'eci',label:'ECI',val:c.ecS},
    {key:'execution',label:'Exec. Capacity',val:c.exS},
    {key:'capital',label:'Capital Mob.',val:c.cmS},
    {key:'velocity',label:'Velocity',val:c.tvS},
    {key:'materials',label:'Materials',val:c.rmS},
    {key:'climate',label:'Climate Exp.',val:c.cxS},
  ];
  ```

- [ ] **Step 3: Update renderComparison() dimension bars**
  In `renderComparison()` (~line 2148), update the `dims` array to match Step 2, using `primary.exS/compare.exS`, `primary.cmS/compare.cmS`, etc.

- [ ] **Step 4: Update mini-bar legend colors in rankings panel**
  The ranking entries show mini colored bars for each dimension. Update labels and field references to match new dimension names.

- [ ] **Step 5: Verify in browser**
  - Hover over countries: tooltip shows new dimension names and correct values
  - Click a country: detail panel shows correct 9 bars with new labels
  - Click a second country: comparison panel renders correctly

- [ ] **Step 6: Commit**
  ```bash
  git add index.html
  git commit -m "Update tooltips, detail panel, and comparison for new dimensions"
  ```

### Task 9: Update 2D/3D axis labels

**Files:**
- Modify: `index.html`
  - 2D SVG axis labels (~line 2629-2630)
  - 3D Three.js axis labels (~line 2371-2378)
  - Z-axis label in 3D view

- [ ] **Step 1: Update 2D SVG axis labels**
  ```javascript
  // X-axis label — no text change needed, "TRANSITION READINESS →" is still correct
  // Y-axis label — no text change needed, "↑ POWER TO EXECUTE" is still correct
  ```

- [ ] **Step 2: Update 3D axis labels**
  Z-axis label change:
  ```javascript
  // OLD:
  const zLabel=makeTextSprite('CLIMATE RESILIENCE →', ...);

  // NEW:
  const zLabel=makeTextSprite('COMPOUND EXPOSURE →', ...);
  ```

- [ ] **Step 3: Commit**
  ```bash
  git add index.html
  git commit -m "Update 3D Z-axis label to Compound Exposure"
  ```

---

## Chunk 3: Update Cluster Logic & Archetypes

### Task 10: Rewrite cluster classification function

**Files:**
- Modify: `index.html` — `cluster()` function (~line 1949)

The cluster function must use new field names and may need threshold adjustments since Execution Capacity and Capital Mobilization will have different distributions than the old State Capacity and PPP.

- [ ] **Step 1: Update field references in cluster()**
  ```javascript
  // OLD:
  function cluster(c) {
    if (c.scN > 0.80 && c.ceN > 0.35 && c.rsN >= 0.50) return 'leader';
    if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
    if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35) return 'latent';
    if (c.ceN < 0.05 && c.ppN > 0.6) return 'petro';
    if (c.ceN > 0.20 && c.scN < 0.60 && c.ecN < 0.50) return 'fragile';
    if (c.cxN > 0.5 && c.scN < 0.60) return 'crisis';
    return 'default';
  }

  // NEW — initial pass (thresholds will be tuned after data review):
  function cluster(c) {
    if (c.exN > 0.75 && c.ceN > 0.35 && c.rmN >= 0.50) return 'leader';
    if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
    if (c.ecN > 0.75 && c.exN > 0.60 && c.ceN < 0.35) return 'latent';
    if (c.ceN < 0.05 && c.cmN > 0.5) return 'petro';
    if (c.ceN > 0.20 && c.exN < 0.55 && c.ecN < 0.50) return 'fragile';
    if (c.cxN > 0.5 && c.exN < 0.55) return 'crisis';
    return 'default';
  }
  ```

- [ ] **Step 2: Review cluster assignments against all 85 countries**
  - Load in browser, check which countries fall into each cluster
  - Verify the 6 archetype groups make sense with new scores
  - Key checks:
    - Leaders: Nordics + Switzerland + Singapore should still qualify
    - Power: US + China should still qualify
    - Latent: Japan, South Korea, Germany, Taiwan should still qualify
    - Petro: Gulf states should still qualify (using Capital Mobilization instead of PPP)
    - Fragile: Brazil, Peru, Colombia — check if mineral security changes their Z position
    - Crisis: Bangladesh, Philippines, Pakistan, Nigeria

- [ ] **Step 3: Tune thresholds if needed**
  - Adjust threshold values based on actual score distributions
  - The Execution Capacity distribution will likely be wider than old State Capacity (China jumps, US drops) so thresholds may need adjustment
  - Petro detection now uses Capital Mobilization instead of PPP — petrostates with sovereign funds may score differently

- [ ] **Step 4: Update clusterTags if archetype names change**
  ```javascript
  // Consider if any tag names should change — spec suggests same 6 archetypes
  // but the membership may shift
  ```

- [ ] **Step 5: Commit**
  ```bash
  git add index.html
  git commit -m "Update cluster classification for new dimension fields and thresholds"
  ```

---

## Chunk 4: Update Country Descriptions & Analysis

### Task 11: Update all 85 country description strings

**Files:**
- Modify: `index.html` — `desc` field in each raw data entry (~lines 1759-1906)

Each country's `desc` string currently references PPP, State Capacity, and Momentum. These must be rewritten to reference Execution Capacity, Capital Mobilization, and Transition Velocity.

- [ ] **Step 1: Identify all desc strings that reference old dimensions**
  - Search for: "PPP", "state capacity", "governance", "WGI", "FSI", "SPI", "momentum", "resource security" in desc strings
  - List all countries whose descriptions need updating

- [ ] **Step 2: Rewrite descriptions for top-20 countries**
  - These are the most-viewed descriptions, highest priority
  - Reference new dimension names and scores
  - Highlight execution capacity examples (infrastructure delivery, construction speed)
  - Highlight capital mobilization mechanisms (sovereign funds, development banks)
  - Highlight transition velocity direction (accelerating, decaying, stagnant)
  - Add hydro legacy note where applicable

- [ ] **Step 3: Rewrite descriptions for remaining 65 countries**
  - Same approach, can be briefer for lower-ranked countries
  - Ensure every description references at least the most notable new dimension scores

- [ ] **Step 4: Commit**
  ```bash
  git add index.html
  git commit -m "Rewrite country descriptions for new dimension framework"
  ```

### Task 12: Rewrite ANALYSIS.md

**Files:**
- Modify: `ANALYSIS.md`

The entire analysis must be rewritten against the new dimensions. The structure can stay similar but all findings, examples, and weight scenarios need updating.

- [ ] **Step 1: Rewrite Section 1 — "No Country Dominates"**
  - Update dimension references throughout
  - China's execution capacity now scores high (not held back by governance metrics)
  - US drops on execution capacity despite high compute
  - Singapore's story changes: still perfect execution, but now explicitly rewarded for it

- [ ] **Step 2: Rewrite Section 2 — "Country Archetypes"**
  - Balanced Leaders: defined by execution capacity + clean energy + material security (not governance)
  - Latent Electrostates: unchanged thesis but reference execution capacity instead of state capacity
  - Single-Dimension Powers: add note about US negative transition velocity
  - Clean-but-Fragile: Brazil's material security may change (mineral reserves)
  - Petrostates: now detected by capital mobilization instead of PPP, reference sovereign fund deployment
  - Add any new archetype observations from the reshuffling

- [ ] **Step 3: Rewrite Section 3 — "Weight Configurations"**
  - Replace "Weight only PPP" → "Weight only Capital Mobilization"
  - Replace "Weight only State Capacity" → "Weight only Execution Capacity"
  - Replace "Weight only Momentum" → "Weight only Transition Velocity"
  - Run each scenario and report new top-5 rankings
  - Add new scenario: "Weight only Transition Velocity" — reveals who's accelerating vs decaying

- [ ] **Step 4: Rewrite Section 4 — "Momentum Changes Everything"**
  - Now "Transition Velocity Changes Everything"
  - Highlight countries with negative velocity (US infrastructure decay, UK, Canada)
  - Highlight countries with strong positive velocity (China, UAE, Chile)
  - This section becomes more powerful with the negative-velocity capability

- [ ] **Step 5: Rewrite Section 5 — "Five Structural Findings"**
  - Finding 1 (compound problem): unchanged thesis
  - Finding 2 (8-12 countries lead): recheck membership
  - Finding 3 (resource security): expand to include material security
  - Finding 4 (compound exposure): unchanged
  - Finding 5 (state capacity as multiplier): becomes "execution capacity as multiplier" — now cleaner because it measures what actually matters

- [ ] **Step 6: Rewrite Section 6 — "What the Index Cannot See"**
  - Some old blind spots may be partially addressed (political will partially captured in execution capacity track record)
  - Update veto players section (US execution capacity now captures this better)
  - Add any new blind spots introduced by the redesign
  - Keep supply chain vulnerability section, update with material security context

- [ ] **Step 7: Update data sources citation at bottom**

- [ ] **Step 8: Commit**
  ```bash
  git add ANALYSIS.md
  git commit -m "Rewrite ANALYSIS.md for new dimension framework"
  ```

---

## Chunk 5: Update Narratives & Story Presets

### Task 13: Update narrative intro slides

**Files:**
- Modify: `index.html` — narrative slides (~lines 1737-1809)

- [ ] **Step 1: Update slide text**
  - Slide 0: "The 21st century belongs to electrostates" — keep, but update description paragraph to reference execution capacity and capital mobilization instead of governance
  - Slide 1: "Only 8-12 countries" — update list of required capabilities to match new dimension names
  - Slide 2: "Countries most exposed" — unchanged (climate exposure didn't change)
  - Slide 3: "Wealth is not resilience" — perfect opportunity to reference Capital Mobilization vs PPP distinction. Qatar has high capital but narrow deployment. Reference transition velocity: "Canada is going backwards" → "The United States' infrastructure is actively decaying"
  - Slide 4: "85 nations. 9 dimensions. 3 axes." — update axis descriptions to match new framework. Z axis is now "Compound Exposure" not "Climate Resilience"

- [ ] **Step 2: Commit**
  ```bash
  git add index.html
  git commit -m "Update narrative intro slides for new dimension framework"
  ```

### Task 14: Update story presets

**Files:**
- Modify: `index.html` — `storyPresets` object (~lines 2701-2746)

- [ ] **Step 1: Update preset weight mappings**
  The weight IDs (`w1`-`w9`) now map to different dimensions. Every preset must be rechecked:

  ```javascript
  // NEW weight mapping:
  // w1=Energy, w2=CleanTech, w3=Compute, w4=ECI, w5=ExecCap, w6=CapMob, w7=Velocity, w8=Materials, w9=Climate

  reset: { weights: {w1:11,w2:11,w3:11,w4:11,w5:11,w6:11,w7:11,w8:11,w9:11} },
  // reset unchanged

  petroClock: {
    // OLD: w1:0,w2:5,w3:0,w4:0,w5:0,w6:5,w7:0,w8:30,w9:30
    // w2 was PPP (now Clean Tech), w6 was State Cap (now Capital Mob), w8 was Resource Sec (now Materials)
    // Intent: reveal petrostate compound vulnerability via resource/climate
    weights: {w1:0,w2:0,w3:0,w4:0,w5:5,w6:5,w7:0,w8:30,w9:30},
    // Keep exec cap and capital mob at small weight, heavy on materials + climate
  },

  usGamble: {
    // OLD: w4:50 (Compute) — but w4 is now ECI
    // Intent: weight compute heavily to show US dominance
    // Compute is now w3
    weights: {w1:5,w2:5,w3:50,w4:5,w5:5,w6:5,w7:5,w8:5,w9:5},
  },

  moving: {
    // OLD: w7:70 (Momentum) — w7 is still Velocity, same position
    // Intent: weight transition velocity to show who's moving
    weights: {w1:0,w2:0,w3:0,w4:0,w5:0,w6:0,w7:70,w8:0,w9:0},
    // Update body text to reference velocity (±) and infrastructure decay
  },

  breadth: {
    // equal weights — unchanged
    weights: {w1:11,w2:11,w3:11,w4:11,w5:11,w6:11,w7:11,w8:11,w9:11},
  },

  crisis: {
    // OLD: w6:20 (State Cap → now Capital Mob), w8:20 (Resource Sec → now Materials), w9:40 (Climate)
    // Intent: reveal crisis states via exposure + low capacity
    // Should use Execution Capacity (w5) instead of Capital Mobilization (w6)
    weights: {w1:0,w2:0,w3:0,w4:0,w5:20,w6:0,w7:0,w8:20,w9:40},
  },
  ```

- [ ] **Step 2: Update preset body text**
  - `petroClock.body`: Reference material security (minerals) alongside food/water. Reference capital mobilization instead of PPP.
  - `usGamble.body`: Add note about negative transition velocity, infrastructure decay. Update to reference execution capacity gap.
  - `moving.body`: Completely rewrite — now "Transition Velocity" not "Momentum." Highlight that negative values reveal decay. US infrastructure decline becomes the counter-story. Update country examples with new velocity scores.
  - `moving.countries`: Update with new velocity score values.
  - `breadth.body`: Update dimension names in text.
  - `crisis.body`: Reference execution capacity instead of state capacity.

- [ ] **Step 3: Update preset highlightFn and countries strings**
  - `moving.highlightFn`: change from `c.moS > 65` to `c.tvS > 65` (or appropriate threshold for new velocity scores)
  - `usGamble.countries`: Update score references
  - `petroClock.countries`: Update to include material security scores
  - `crisis.countries`: Update with execution capacity scores instead of state capacity

- [ ] **Step 4: Consider adding new preset: "Infrastructure Decay"**
  ```javascript
  decay: {
    weights: {w1:0,w2:0,w3:0,w4:0,w5:30,w6:0,w7:50,w8:0,w9:0},
    title: 'Infrastructure Decay',
    body: 'Weight execution capacity and transition velocity to reveal the gap between countries that build and countries that crumble. The United States, United Kingdom, and Canada have high capability on paper but their infrastructure is actively deteriorating. China, UAE, and the Nordics are accelerating. Wealth without deployment is decay.',
    highlight: null,
    highlightFn: function(c){ return c.tvS < 30; },
    countries: 'US (velocity: negative, ASCE grade C-) · UK (HS2 cancelled, rail declining) · Canada (energy transition stalling) — vs China (velocity: 80+, 48k km HSR) · UAE (velocity: 60+, fastest petrostate pivot)'
  },
  ```

- [ ] **Step 5: Commit**
  ```bash
  git add index.html
  git commit -m "Update story presets and add Infrastructure Decay preset for new dimensions"
  ```

---

## Chunk 6: Update Analysis Deck in HTML & Final Verification

### Task 15: Update the analysis deck text in index.html

**Files:**
- Modify: `index.html` — analysis deck HTML (~lines 1619-1735)

The analysis deck is the in-page prose (separate from ANALYSIS.md). It contains the same findings but in shorter form, rendered directly in the UI.

- [ ] **Step 1: Update "No Country Dominates" section**
  - Replace "PPP" references with "capital mobilization"
  - Replace "state capacity" with "execution capacity"
  - Replace "momentum" with "transition velocity"
  - Update China's narrative: now scores high on execution capacity
  - Update US narrative: reference infrastructure decay, negative velocity

- [ ] **Step 2: Update "Six Country Archetypes" section**
  - Update archetype descriptions to reference new dimensions
  - "Balanced Leaders" defined by execution capacity, not governance quality
  - "Latent Electrostates" reference execution capacity
  - Add infrastructure decay angle to US description

- [ ] **Step 3: Update "Five Findings That Survive Any Weighting" section**
  - Finding 3: expand resource security to material security
  - Finding 5: "Execution capacity remains the multiplier" (not state capacity)
  - Update petrostate tautology acknowledgment to reference material security

- [ ] **Step 4: Update "What the Index Cannot See" section**
  - Some blind spots are now partially addressed
  - Update with new limitations introduced by the redesign

- [ ] **Step 5: Commit**
  ```bash
  git add index.html
  git commit -m "Update analysis deck text for new dimension framework"
  ```

### Task 16: Final verification pass

**Files:**
- Read: `index.html` (full)

- [ ] **Step 1: Search for stale references**
  - Grep index.html for: "PPP", "ppp", "ppN", "ppS", "State Cap", "scN", "scS", "Momentum", "moN", "moS", "Resource Sec", "rsN", "rsS"
  - Every match is a bug — all old dimension references must be gone
  - Exception: comments or attribution text that intentionally references the old framework

- [ ] **Step 2: Verify in browser — full functional test**
  - Load index.html in browser
  - Narrative intro plays correctly with new text
  - All 85 countries render in 2D and 3D views
  - Tooltips show new dimension names and correct values
  - Detail panel shows correct bars with new labels
  - Comparison mode works between two countries
  - All 6 (or 7) story presets activate correctly
  - Weight sliders relabel correctly and composite recalculates
  - Rankings panel sorts correctly
  - No console errors

- [ ] **Step 3: Verify data consistency**
  - Spot-check 5 countries: compare CSV source data to rendered values
  - Check China: execution capacity should be 75-85 (per spec expectation)
  - Check US: execution capacity should have dropped significantly
  - Check Australia: resource & material security should have jumped
  - Check transition velocity: at least one country should show negative

- [ ] **Step 4: Commit final verification**
  ```bash
  git add index.html
  git commit -m "fix: verification pass — remove stale dimension references"
  ```

---

## Execution Notes

**Task dependencies:**
- Tasks 1-4 (research) are **fully parallel** — no dependencies between them
- Task 5 (hydro flags) has no dependency on research tasks
- Task 6 (data replacement) depends on Tasks 1-4 completing
- Tasks 7-9 (UI labels/tooltips/axes) can run parallel with Task 6 but reference same file
- Task 10 (cluster) depends on Task 6 (needs new field names in data)
- Task 11 (descriptions) depends on Task 6 (needs final scores to write accurate descriptions)
- Task 12 (ANALYSIS.md) depends on Task 10 (needs final archetype assignments)
- Tasks 13-14 (narratives/presets) depend on Task 7 (needs correct weight mappings)
- Task 15 (analysis deck) depends on Task 12 (should be consistent with ANALYSIS.md)
- Task 16 (verification) is last — depends on everything

**Parallelization plan:**
```
Phase 1 (parallel):  Tasks 1, 2, 3, 4, 5  — independent CSV files, no conflicts
Phase 2 (sequential): Task 6              — core data/compute changes in index.html
Phase 3 (sequential): Tasks 7, 8, 9       — all modify index.html, must be sequential
Phase 4 (sequential): Task 10             — cluster logic in index.html
Phase 5 (parallel):  Tasks 11, 12         — Task 11 = index.html, Task 12 = ANALYSIS.md (different files)
Phase 6 (sequential): Tasks 13, 14, 15    — all modify index.html, must be sequential
Phase 7 (sequential): Task 16             — final verification
```

**NOTE:** Phases 3 and 6 are sequential because Tasks 7-9 and 13-15 all edit `index.html`. Parallel subagents on the same file will produce merge conflicts. Only Phase 1 (separate CSVs) and Phase 5 (different files) are genuinely parallel.

**Critical risk:** Transition Velocity normalization. Since this is bipolar (negative to positive), the TV_MIN and TV_MAX constants in Task 6 Step 2 must be set from the actual data produced in Task 3. Do not hardcode until Task 3 is complete.
