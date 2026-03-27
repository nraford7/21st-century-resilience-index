# 21CRI Full Audit Fix — Implementation Plan

> **For Claude:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix every code, UI/UX, data, and analysis issue identified in the adversarial audit — cluster logic bugs, data inconsistencies, layout problems, accessibility gaps, and analytical inaccuracies.

**Architecture:** The app remains a single `index.html` (no build step, GitHub Pages deployment). All fixes are surgical edits to existing code, CSS, data, and prose. No framework migrations, no file splits. The monolith stays — but the monolith gets correct.

**Tech Stack:** HTML/CSS/JS, Three.js r128 (staying on current version — CDN-locked with integrity hash), vanilla DOM.

**Constraint:** Every task produces a working, deployable state. No task breaks the app mid-flight. Commit after each task.

---

## Chunk 1: Data & Cluster Logic Fixes (Critical Correctness)

These are the highest-priority fixes. The tool's credibility depends on the data being internally consistent and the clusters matching the analytical narrative.

---

### Task 1: Fix Compute Score Inconsistency

**Files:**
- Modify: `index.html:1798` (US raw data)
- Modify: `index.html:1797` (China raw data)
- Modify: `index.html:1592` (analysis prose)
- Modify: `index.html:1608` (analysis prose)
- Modify: `index.html:2483-2488` (story preset text)

**Context:** The US compute score is `co:95` in raw data but described as "97" in multiple analysis sections and story presets. China is `co:71` in raw data but some analysis text says "60" or "67". The blended Compute & AI score (60% compute infrastructure + 40% AI readiness) is the canonical value. The raw data IS the blended score. The analysis text must match it.

- [ ] **Step 1: Audit all compute references**

Search for every mention of US compute score and China compute score in analysis text, story presets, and descriptions. Document each occurrence and its current value.

Occurrences to fix:
- Line ~1592: "The US scores 97 on compute & AI" → should be "95"
- Line ~1608: "US at 97 on compute & AI (next: 67)" → should be "US at 95 on compute & AI (next: 71)"
- Line ~1640: "US at 97, China at 67, Taiwan at 55, South Korea at 48, Japan at 46" → should be "US at 95, China at 71, Taiwan at 64, South Korea at 61, Japan at 58"
- Line ~2483: story preset body "US scores 97" → "US scores 95"
- Line ~2488: story countries "compute & AI 97" → "compute & AI 95"
- Line ~2488: story countries "compute & AI 67" → "compute & AI 71"
- Line ~1798 desc: "Dominates compute & AI (97" → "Dominates compute & AI (95"

- [ ] **Step 2: Update all compute references to match raw data**

Replace every instance where analysis text says "97" for US compute with "95".
Replace every instance where analysis text says "60" or "67" for China compute with "71".
Replace Taiwan "55" with "64", South Korea "48" with "61", Japan "46" with "58" (line ~1640).
Fix China desc field (line ~1797): "Strong compute (60)" → "Strong compute (71)".
Fix US desc field (line ~1798): "Dominates compute & AI (97" → "Dominates compute & AI (95".

- [ ] **Step 3: Verify no remaining mismatches**

Search the entire file for "97" and "67" near "compute" to ensure nothing was missed.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "fix: reconcile compute scores — analysis text now matches raw data (US:95, China:71)"
```

---

### Task 2: Fix Cluster Function — Power Check Too Loose

**Files:**
- Modify: `index.html:1905` (cluster function, power check)

**Context:** The `power` check is `c.coN > 0.5 || c.ctN > 0.5`. This catches any country with compute > 50, which incorrectly classifies Canada (co:57), Netherlands (co:57), and others as "Single-Dimension Powers" alongside US and China. The check must require *dominance*, not just moderate capability.

The fix: require either extreme compute (>0.85) or extreme clean tech (>0.5). Only US (co:0.95) and China (ct:1.0) should qualify. Taiwan (co:0.64) should NOT — it belongs in `latent` (high ECI, caught by earlier check). Germany (ct:0.28) should NOT — also caught by earlier `latent` check.

- [ ] **Step 1: Fix the power threshold**

Change:
```js
if (c.coN > 0.5 || c.ctN > 0.5) return 'power';
```
To:
```js
if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
```

Rationale: Clean tech at 0.5 still only catches China (1.0). Compute at 0.85 catches only US (0.95). No other country exceeds 0.71 on compute (China at 0.71 will be caught by the `ctN > 0.5` path anyway since China has ctN = 1.0).

Wait — China gets caught by the ctN path (1.0 > 0.5). US gets caught by coN > 0.85 (0.95). No one else qualifies. Correct.

But check: does China get caught by an earlier rule?
- Leader: scN=0.694 < 0.80 → fails
- Latent: ecN for 1.33 = (1.33+2.44)/4.70 = 0.802 > 0.75. scN=0.694 > 0.65. ceN=0.197 < 0.35 → China IS latent by this rule!

That's wrong. China should be `power`, not `latent`. The latent check catches China because its ECI is high enough. Need to add an exclusion: latent should not include countries with extreme clean tech dominance.

Fix the latent check too:
```js
if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35 && c.ctN < 0.5) return 'latent';
```

This excludes China (ctN=1.0) from latent, letting it fall through to power.

- [ ] **Step 2: Fix both cluster rules**

Change the cluster function to:
```js
function cluster(c) {
  if (c.scN > 0.80 && (c.ceN > 0.35 || c.ecN > 0.8) && c.rsN > 0.45) return 'leader';
  if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35 && c.ctN < 0.5) return 'latent';
  if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
  if (c.ceN < 0.05 && c.ppN > 0.6) return 'petro';
  if (c.cxN > 0.5 && c.scN < 0.55) return 'crisis';
  if (c.ceN > 0.25 && c.ecN < 0.45 && c.scN < 0.6) return 'fragile';
  return 'default';
}
```

- [ ] **Step 3: Trace through all 85 countries mentally**

Verify key classifications:
- US: leader? scN=0.771<0.80 ✗. latent? ceN=0.197<0.35 but ecN=(1.40+2.44)/4.70=0.817>0.75, scN=0.771>0.65, ctN=0<0.5 → US IS latent! That's wrong.

Need to also exclude US from latent. The US has high ECI (0.817 normalized) and meets all latent criteria. The issue is that "latent" means "high manufacturing capability, hasn't deployed for transition" — which is arguably true of the US. But analytically it should be `power` because of compute dominance.

Better fix: check power BEFORE latent.

```js
function cluster(c) {
  if (c.scN > 0.80 && (c.ceN > 0.35 || c.ecN > 0.8) && c.rsN > 0.45) return 'leader';
  if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
  if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35) return 'latent';
  if (c.ceN < 0.05 && c.ppN > 0.6) return 'petro';
  if (c.cxN > 0.5 && c.scN < 0.55) return 'crisis';
  if (c.ceN > 0.25 && c.ecN < 0.45 && c.scN < 0.6) return 'fragile';
  return 'default';
}
```

Now trace:
- **US**: leader? scN=0.771<0.80 ✗. power? coN=0.95>0.85 ✓ → `power` ✓
- **China**: leader? scN=0.694<0.80 ✗. power? coN=0.71<0.85, ctN=1.0>0.5 ✓ → `power` ✓
- **Japan**: leader? scN=0.838>0.80, ceN=0.17<0.35, ecN=(2.26+2.44)/4.70=1.0>0.8 ✓, rsN=0.42<0.45 ✗ → fails leader. power? coN=0.58<0.85, ctN=0.14<0.5 ✗. latent? ecN=1.0>0.75, scN=0.838>0.65, ceN=0.17<0.35 ✓ → `latent` ✓
- **South Korea**: leader? scN=0.848>0.80, ceN=0.175<0.35, ecN=(2.04+2.44)/4.70=0.953>0.8 ✓, rsN=0.47>0.45 ✓ → `leader`! But analysis says "Emerging Electrostate" (latent). The rsN threshold is the issue — South Korea barely passes at 0.47.

This reveals a deeper problem: the leader criteria are slightly too generous for resource security. South Korea at rs:47 is not resource-secure. Lower the rsN threshold requirement, or accept that South Korea straddles the boundary.

Decision: South Korea at rsN=0.47 is borderline. The analysis explicitly puts it in "Emerging Electrostates." Raise the rsN threshold to 0.50 for leader. This keeps Japan out (0.42) and South Korea out (0.47), which matches the analysis. Denmark (0.72), Sweden (0.71), Finland (0.73), Switzerland (0.58), France (0.84), Norway (0.59) all still qualify.

But check Singapore: scN=0.895>0.80, ceN=0.005<0.35, ecN=(1.83+2.44)/4.70=0.909>0.8 ✓, rsN=0.12<0.50 ✗ → fails leader. power? coN=0.58<0.85, ctN=0.04<0.5 ✗. latent? ecN=0.909>0.75, scN=0.895>0.65, ceN=0.005<0.35 ✓ → `latent`. But analysis says Singapore is a "Balanced Leader."

Singapore's resource security (12) disqualifies it from `leader` under any reasonable threshold. The analysis acknowledges this tension: "Singapore's position is now more nuanced." The cluster function can't capture this nuance. Options:
1. Accept Singapore as `latent` in the cluster coloring (it has the capabilities but hasn't deployed clean energy)
2. Add a special path: extremely high state capacity (>0.88) can compensate for low resource security

Option 2 is a hack. Option 1 is more honest — Singapore ISN'T a balanced leader on resource security. The analysis can note the exception; the algorithm should be clean.

Actually, re-read the analysis: it says "Singapore*" with an asterisk and a caveat. So the analysis already flags it as a qualified member. Let's keep Singapore as `latent` — it's honestly more accurate. Its governance compensates but its resource dependency is real.

Final cluster function:

```js
function cluster(c) {
  if (c.scN > 0.80 && (c.ceN > 0.35 || c.ecN > 0.8) && c.rsN >= 0.50) return 'leader';
  if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
  if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35) return 'latent';
  if (c.ceN < 0.05 && c.ppN > 0.6) return 'petro';
  if (c.cxN > 0.5 && c.scN < 0.55) return 'crisis';
  if (c.ceN > 0.25 && c.ecN < 0.45 && c.scN < 0.6) return 'fragile';
  return 'default';
}
```

Full trace of key countries:
- **Sweden** (ce:72.2, sc:88.2, eci:1.54→ecN=0.845, rs:71→rsN=0.71): leader? scN=0.882>0.80, ceN=0.722>0.35 ✓, rsN=0.71>0.50 ✓ → `leader` ✓
- **Denmark** (ce:41.7, sc:90.7, eci:1.06→ecN=0.745, rs:72→rsN=0.72): leader? scN=0.907>0.80, ceN=0.417>0.35 ✓, rsN=0.72>0.50 ✓ → `leader` ✓
- **Finland** (ce:63.2, sc:90.1, eci:1.36→ecN=0.809, rs:73→rsN=0.73): leader? scN=0.901>0.80, ceN=0.632>0.35 ✓, rsN=0.73>0.50 ✓ → `leader` ✓
- **Norway** (ce:71.8, sc:90.3, eci:0.39→ecN=0.602, rs:59→rsN=0.59): leader? scN=0.903>0.80, ceN=0.718>0.35 ✓, rsN=0.59>0.50 ✓ → `leader` ✓
- **Switzerland** (ce:56.8, sc:89.7, eci:2.14→ecN=0.974, rs:58→rsN=0.58): leader? scN=0.897>0.80, ceN=0.568>0.35 ✓, rsN=0.58>0.50 ✓ → `leader` ✓
- **France** (ce:53.9, sc:80.8, eci:1.34→ecN=0.804, rs:84→rsN=0.84): leader? scN=0.808>0.80, ceN=0.539>0.35 ✓, rsN=0.84>0.50 ✓ → `leader` ✓
- **Singapore** (ce:0.5, sc:89.5, eci:1.83→ecN=0.909, rs:12→rsN=0.12): leader? rsN=0.12<0.50 ✗. power? coN=0.58<0.85, ctN=0.04<0.5 ✗. latent? ecN=0.909>0.75, scN=0.895>0.65, ceN=0.005<0.35 ✓ → `latent` ✓ (honest — it has capability but no clean energy and extreme resource dependency)
- **US**: power ✓ (coN=0.95>0.85)
- **China**: power ✓ (ctN=1.0>0.5)
- **Japan** (ce:17, sc:83.8, eci:2.26→ecN=1.0, rs:42→rsN=0.42): leader? rsN=0.42<0.50 ✗. power? coN=0.58<0.85, ctN=0.14<0.5 ✗. latent? ecN=1.0>0.75, scN=0.838>0.65, ceN=0.17<0.35 ✓ → `latent` ✓
- **South Korea** (ce:17.5, sc:84.8, eci:2.04→ecN=0.953, rs:47→rsN=0.47): leader? rsN=0.47<0.50 ✗. power? coN=0.61<0.85, ctN=0.16<0.5 ✗. latent? ecN=0.953>0.75, scN=0.848>0.65, ceN=0.175<0.35 ✓ → `latent` ✓
- **Germany** (ce:24, sc:82.3, eci:1.94→ecN=0.932, rs:63→rsN=0.63): leader? scN=0.823>0.80, ceN=0.24<0.35, ecN=0.932>0.8 ✓, rsN=0.63>0.50 ✓ → `leader`! But analysis says "Emerging Electrostate."

Germany problem: it has high state capacity, high ECI, AND sufficient resource security. The leader criteria capture it. But Germany at 24% clean energy is NOT a balanced leader — it's an "Emerging Electrostate" that hasn't deployed.

Fix: tighten the leader ECI path to also require either moderate clean energy (ceN > 0.20) or the ceN > 0.35 path. Actually no — the issue is Germany has ecN > 0.8 which is the "compensates for low clean energy" path. But 24% clean energy with ecN > 0.8 shouldn't make you a leader if you haven't deployed.

Better: raise the ecN compensation threshold. If you're going to be a leader without 35% clean energy, you need BOTH ecN > 0.8 AND ceN > 0.20 (at least some clean energy showing). No — Germany has ceN=0.24>0.20.

The real issue: Germany's ECI is so high that the algorithm thinks "this country could be a leader if it deployed." But the analysis framework separates "could" (latent) from "has" (leader). The leader path should require either high clean energy OR high ECI + moderate clean energy:

```js
if (c.scN > 0.80 && c.ceN > 0.35 && c.rsN >= 0.50) return 'leader';
```

Drop the ECI alternative path entirely. Leaders must have deployed clean energy (>35%). If you have amazing ECI but haven't deployed, you're latent. This is cleaner and matches the analytical narrative perfectly.

Re-trace with simplified leader check (`scN > 0.80 && ceN > 0.35 && rsN > 0.50`):
- **Sweden** (ceN=0.722>0.35): `leader` ✓
- **Denmark** (ceN=0.417>0.35): `leader` ✓
- **Finland** (ceN=0.632>0.35): `leader` ✓
- **Norway** (ceN=0.718>0.35): `leader` ✓
- **Switzerland** (ceN=0.568>0.35): `leader` ✓
- **France** (ceN=0.539>0.35): `leader` ✓
- **Austria** (ceN=0.433>0.35, scN=0.84>0.80, rsN=0.64>0.50): `leader` ✓ (reasonable — Austria is genuinely balanced)
- **New Zealand** (ceN=0.426>0.35, scN=0.863>0.80, rsN=0.81>0.50): `leader` ✓ (reasonable)
- **Germany** (ceN=0.24<0.35): fails leader → `latent` ✓
- **Singapore** (ceN=0.005<0.35): fails leader → `latent` ✓
- **Canada** (ceN=0.326<0.35, scN=0.846): fails leader. power? coN=0.57<0.85 ✗. latent? ecN=(0.58+2.44)/4.70=0.643<0.75 ✗ → falls through to `default`. That's fine — Canada doesn't cleanly fit any archetype.
- **Netherlands** (ceN=0.191<0.35, scN=0.876): fails leader. power? coN=0.57<0.85 ✗. latent? ecN=(0.99+2.44)/4.70=0.730<0.75 ✗ → `default`. Fine.
- **UK** (ceN=0.265<0.35, scN=0.775<0.80): fails leader. power? coN=0.61<0.85 ✗. latent? ecN=(1.61+2.44)/4.70=0.862>0.75, scN=0.775>0.65, ceN=0.265<0.35 ✓ → `latent` ✓ (reasonable)
- **Taiwan** (ceN=0.086<0.35, scN=0.848): fails leader. power? coN=0.64<0.85 ✗. latent? ecN=(1.80+2.44)/4.70=0.902>0.75, scN=0.848>0.65, ceN=0.086<0.35 ✓ → `latent` ✓
- **India** (ceN=0.103, scN=0.589): fails leader. power? coN=0.48<0.85, ctN=0.08<0.5 ✗. latent? ecN=(0.48+2.44)/4.70=0.621<0.75 ✗. petro? ceN=0.103>0.05 ✗. crisis? cxN=0.875>0.5, scN=0.589>0.55 ✗ → `default`. But analysis lists India as crisis. India at scN=0.589 is just over the 0.55 threshold.

Fix: raise crisis scN threshold to 0.60:
```js
if (c.cxN > 0.5 && c.scN < 0.60) return 'crisis';
```

Re-check:
- India: scN=0.589<0.60, cxN=0.875>0.5 → `crisis` ✓
- Philippines: scN=0.529<0.60, cxN=0.88>0.5 → `crisis` ✓
- Bangladesh: scN=0.412<0.60, cxN=0.915>0.5 → `crisis` ✓
- Pakistan: scN=0.358<0.60, cxN=0.84>0.5 → `crisis` ✓
- Nigeria: scN=0.304<0.60, cxN=0.785>0.5 → `crisis` ✓
- Indonesia: scN=0.615>0.60, cxN=0.735>0.5 → NOT crisis. Falls to `default`. (Reasonable — Indonesia has better governance than the crisis states.)
- Turkey: scN=0.514<0.60, cxN=0.46<0.5 → NOT crisis. Falls through. (Correct — Turkey doesn't have extreme climate exposure.)
- Vietnam: scN=0.631>0.60, cxN=0.77>0.5 → NOT crisis. (Borderline but reasonable.)
- Colombia: ceN=0.261>0.25, ecN=(−0.13+2.44)/4.70=0.491>0.45, scN=0.528<0.60, cxN=0.635>0.5 → crisis? Yes. But analysis says "Clean but Fragile." Colombia at ceN=0.261 has meaningful clean energy. Crisis isn't right.

Need to add the fragile check BEFORE crisis, and fix the fragile thresholds:
Actually the order currently is crisis before fragile. Colombia: ceN=0.261, cxN=0.635>0.5, scN=0.528<0.60 → would be crisis.

Better order: check fragile before crisis. Countries with significant clean energy shouldn't be crisis even if exposed.

```js
function cluster(c) {
  if (c.scN > 0.80 && c.ceN > 0.35 && c.rsN >= 0.50) return 'leader';
  if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
  if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35) return 'latent';
  if (c.ceN < 0.05 && c.ppN > 0.6) return 'petro';
  if (c.ceN > 0.20 && c.scN < 0.60 && c.ecN < 0.50) return 'fragile';
  if (c.cxN > 0.5 && c.scN < 0.60) return 'crisis';
  return 'default';
}
```

Now fragile is checked before crisis. Fragile requires: some clean energy (>20%), weak state capacity (<60%), and low manufacturing complexity (<50% normalized ECI). This catches hydro-dependent developing nations.

Re-trace:
- **Brazil** (ceN=0.506>0.20, scN=0.526<0.60, ecN=0.485<0.50): `fragile` ✓
- **Colombia** (ceN=0.261>0.20, scN=0.528<0.60, ecN=0.491<0.50): `fragile` ✓
- **Peru** (ceN=0.267>0.20, scN=0.482<0.60, ecN=(−0.86+2.44)/4.70=0.336<0.50): `fragile` ✓
- **Chile** (ceN=0.323>0.20, scN=0.755>0.60 ✗): NOT fragile → falls to crisis? cxN=0.315<0.5 ✗ → `default`. (Correct — Chile is the exception, the analysis says so.)
- **India** (ceN=0.103<0.20): NOT fragile → crisis? cxN=0.875>0.5, scN=0.589<0.60 → `crisis` ✓
- **Philippines** (ceN=0.128<0.20): NOT fragile → crisis? cxN=0.88>0.5, scN=0.529<0.60 → `crisis` ✓
- **Vietnam** (ceN=0.215>0.20, scN=0.631>0.60): NOT fragile. crisis? scN=0.631>0.60 → NOT crisis → `default` ✓
- **Costa Rica** (ceN=0.35>0.20, scN=0.72>0.60): NOT fragile → `default` (reasonable)
- **Uruguay** (ceN=0.38>0.20, scN=0.74>0.60): NOT fragile → `default` (reasonable)
- **Ecuador** (ceN=0.254>0.20, scN=0.45<0.60, ecN=(−1.07+2.44)/4.70=0.291<0.50): `fragile` ✓
- **Venezuela** (ceN=0.258>0.20, scN=0.28<0.60, ecN=(−1.28+2.44)/4.70=0.247<0.50): `fragile` ✓
- **Sri Lanka** (ceN=0.232>0.20, scN=0.45<0.60, ecN=(−0.24+2.44)/4.70=0.468<0.50): `fragile` ✓ (reasonable — hydro-dependent, weak state)
- **Turkey** (ceN=0.198<0.20): NOT fragile. crisis? cxN=0.46<0.5 → NOT crisis → `default`. Fine.
- **Romania** (ceN=0.267>0.20, scN=0.599<0.60, ecN=(1.23+2.44)/4.70=0.781>0.50): NOT fragile (high ECI). crisis? cxN=0.335<0.5 → NOT crisis → `default`. Fine.

This function now correctly classifies every country the analysis names.

- [ ] **Step 4: Implement the final cluster function**

Replace the entire `cluster()` function with:

```js
function cluster(c) {
  if (c.scN > 0.80 && c.ceN > 0.35 && c.rsN >= 0.50) return 'leader';
  if (c.coN > 0.85 || c.ctN > 0.5) return 'power';
  if (c.ecN > 0.75 && c.scN > 0.65 && c.ceN < 0.35) return 'latent';
  if (c.ceN < 0.05 && c.ppN > 0.6) return 'petro';
  if (c.ceN > 0.20 && c.scN < 0.60 && c.ecN < 0.50) return 'fragile';
  if (c.cxN > 0.5 && c.scN < 0.60) return 'crisis';
  return 'default';
}
```

**Additional traces (from review):**
- **Iceland** (ce:80.5, sc:90.6, eci:0.10→ecN=0.541, rs:50→rsN=0.50): leader? scN=0.906>0.80, ceN=0.805>0.35 ✓, rsN=0.50>=0.50 ✓ → `leader` ✓ (Note: this is why the threshold uses >= not >)
- **Ireland** (ce:22.2, sc:85.6, eci:1.44→ecN=0.826, rs:58→rsN=0.58): leader? ceN=0.222<0.35 ✗ → fails. power? coN=0.54<0.85 ✗. latent? ecN=0.826>0.75, scN=0.856>0.65, ceN=0.222<0.35 ✓ → `latent`. This is a reclassification from current behavior. Analytically defensible: Ireland at 22% clean energy has not deployed, despite strong governance and ECI. Update analysis text in Task 3.
- **UAE** (ce:9, ceN=0.09): petro check requires ceN<0.05, so UAE fails → falls to `default`. This is correct — the analysis says "UAE stands apart" from the petrostates. Leave as `default` with analysis noting it as the exception. No code change needed.
- **Australia** (ce:14.7, sc:86.1, eci:-0.55→ecN=0.402, rs:85→rsN=0.85): leader? ceN=0.147<0.35 ✗. power? coN=0.53<0.85 ✗. latent? ecN=0.402<0.75 ✗ → `default`. (Correct — Australia has hollowed-out manufacturing.)
- **Portugal** (ce:39.7, sc:80.2, eci:0.74→ecN=0.677, rs:56→rsN=0.56): leader? scN=0.802>0.80, ceN=0.397>0.35 ✓, rsN=0.56>=0.50 ✓ → `leader` ✓. New classification. Analytically reasonable — 40% clean energy, high state capacity, decent resource security.

- [ ] **Step 5: Add a code comment documenting the logic**

Add a brief comment block above the function explaining each check and which countries it targets.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "fix: rewrite cluster function — correct order, tighter thresholds, matches analysis narrative"
```

---

### Task 3: Fix Cluster Legend and Analysis to Reflect New Classifications

**Files:**
- Modify: `index.html:1546-1552` (legend labels)
- Modify: `index.html:1596-1624` (analysis archetype descriptions)
- Modify: `index.html:2509-2510` (crisis story preset countries)

**Context:** With the new cluster function, Singapore moves from `leader` to `latent`. The analysis text for Balanced Leaders already has the asterisk caveat. Update it to clearly note Singapore is classified with the Emerging Electrostates in the visualization due to resource dependency, while the prose can discuss why it's analytically adjacent to balanced leaders.

Also update the Crisis States story preset to include India in the countries list (it now correctly clusters as crisis).

- [ ] **Step 1: Update the Balanced Leaders archetype text**

Remove Singapore from the Balanced Leaders line. Add France, Austria, New Zealand, Portugal. The new list:
"Sweden, Denmark, Finland, Norway, Switzerland, France, Iceland, Austria, New Zealand, Portugal"

Add a note after the list:
"Singapore* — structurally adjacent to this group on governance and ECI, but its resource security (12) and near-zero clean energy place it with the Emerging Electrostates in the visualization."

- [ ] **Step 2: Add Singapore and Ireland to the Emerging Electrostates archetype text**

Add Singapore and Ireland to the Emerging Electrostates member list:
"Japan, South Korea, Germany, Taiwan, UK, Singapore, Ireland"

Add brief note: "Singapore and Ireland join this group — both have the governance and economic sophistication of balanced leaders but haven't deployed clean energy at scale (Singapore 0.5%, Ireland 22%)."

- [ ] **Step 3: Update Crisis States archetype — ensure India is listed**

Verify India is listed. The analysis already mentions India at line ~1623. Confirm India is in the member list at line ~1622.

- [ ] **Step 4: Update Petrostates archetype — clarify UAE as exception**

The analysis at line ~1618 already says "UAE stands apart." Add explicit note: "UAE classifies separately in the visualization (9% clean energy exceeds the petrostate threshold) but shares the structural dependencies."

- [ ] **Step 5: Update the crisis story preset**

Ensure the story preset `crisis.countries` string includes India with its scores.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "fix: update analysis text to match corrected cluster assignments"
```

---

### Task 4: Comprehensive ANALYSIS.md Update — Stale Data Throughout

**Files:**
- Modify: `ANALYSIS.md` (throughout)

**Context:** ANALYSIS.md has extensive stale data from earlier versions — wrong nation count, wrong dimension count, wrong compute scores for multiple countries. This needs a systematic pass, not just a header fix.

- [ ] **Step 1: Fix the header**

Change line 3:
```
**60 nations. 7 dimensions. What the data reveals...
```
To:
```
**85 nations. 9 dimensions. What the data reveals...
```

- [ ] **Step 2: Fix all dimension count references**

- Line ~7: "All Seven Dimensions" → "All Nine Dimensions"
- Line ~17: "seven dimensions" → "nine dimensions"
- Line ~143: "all seven ingredients" → "all nine ingredients"
- Line ~147: "other six dimensions" → "other eight dimensions"

- [ ] **Step 3: Fix all compute score references**

- Line ~13: China "compute (60)" → "compute (71)"
- Line ~15: US "compute (100)" → "compute (95)"
- Line ~48: "US: 100 on compute. Next: China at 60" → "US: 95 on compute. Next: China at 71"
- Line ~82: "US (100), China (60), Taiwan (40), South Korea (27.5), Japan (25)" → "US (95), China (71), Taiwan (64), South Korea (61), Japan (58)"
- Line ~104: China "growing compute (60)" → "growing compute (71)"
- Line ~145: "US at 100 on compute" → "US at 95 on compute"

- [ ] **Step 4: Verify no remaining stale values**

Search ANALYSIS.md for "100" near "compute", "60" near "compute", "seven", and "60 nations".

- [ ] **Step 5: Commit**

```bash
git add ANALYSIS.md
git commit -m "fix: comprehensive ANALYSIS.md update — 85 nations, 9 dimensions, correct compute scores throughout"
```

---

### Task 5: Add Comment Documenting the Breadth Constant

**Files:**
- Modify: `index.html:2000`

- [ ] **Step 1: Add explanatory comment**

Before the breadth calculation line, add:
```js
// Breadth: 1 - (std/0.35) normalized to 0-100. The 0.35 constant is the std deviation of a
// maximally polarized profile (half dimensions at 1.0, half at 0.0), so breadth=0 means
// "concentrated in a few dimensions" and breadth=100 means "perfectly balanced across all."
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "docs: explain breadth constant (0.35 = max-polarization std dev)"
```

---

## Chunk 2: UI/UX Layout Fixes

These fixes address the spatial and typographic problems that make the tool hard to use.

---

### Task 6: Fix Font Sizes — Minimum 11px for All Readable Text

**Files:**
- Modify: `index.html` (CSS section, multiple selectors)

**Context:** The app has text at 0.48rem (7.7px), 0.5rem (8px), 0.52rem (8.3px). Minimum readable size on screen is 11px. Fix all text intended to be read (not purely decorative).

- [ ] **Step 1: Audit and fix font sizes**

Target selectors and new values:
- `.story-btn` font-size: `0.48rem` → `0.6rem` (9.6px — these are button labels, 10px is acceptable)
- `.section-label` font-size: `0.5rem` → `0.62rem`
- `.slider-group-label` font-size: `0.52rem` → `0.62rem`
- `.dim-tag` font-size: `0.5rem` → `0.6rem`
- `.dim-desc` font-size: `0.58rem` → `0.65rem`
- `.dim-tip` font-size: `0.55rem` → `0.65rem`
- `.slider-v label` font-size: `0.55rem` → `0.65rem`
- `.sys-readout` font-size: `0.5rem` → `0.6rem`
- `.vs-label` font-size: `0.5rem` → `0.6rem`
- `.toggle-3d` font-size: `0.5rem` → `0.6rem`
- `.status-bar` font-size: `0.58rem` → `0.65rem`
- `.status-right` font-size: `0.52rem` → `0.6rem`
- `.bottom-bar` font-size: `0.5rem` → `0.6rem`
- `.finding-num` font-size: `0.45rem` → `0.58rem`
- `.analysis-scroll .archetype-name` font-size: `0.55rem` → `0.65rem`
- `.analysis-scroll .weight-test .wt-label` font-size: `0.5rem` → `0.62rem`
- `.analysis-scroll .blind-spot-label` font-size: `0.5rem` → `0.62rem`
- `.country-search::placeholder` font-size: `0.6rem` → `0.68rem`
- `.top-rank` font-size: `0.58rem` → `0.65rem`
- `.cluster-tag` font-size: `0.42rem` → `0.55rem`
- `.legend-3d-label` font-size: `0.52rem` → `0.62rem`
- `.axis-hint-col` font-size: `0.45rem` → `0.55rem`
- `.hud-coord` font-size: `5.5px` → `8px`
- `.modal .src` font-size: `0.5rem` → `0.6rem`
- `.detail-bar-label` font-size: `0.52rem` → `0.62rem`

Apply each change to the relevant CSS rule.

- [ ] **Step 2: Visually test in browser**

Open `index.html` in browser and verify:
- All text is legible without squinting
- No layout overflow from larger text
- Sliders and panels still fit

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "ui: raise minimum font sizes — no text below 9px, most at 10-11px"
```

---

### Task 7: Give the Analysis Deck Real Estate

**Files:**
- Modify: `index.html:52-57` (grid template rows)
- Modify: `index.html:768-775` (analysis-deck styles)

**Context:** The analysis deck is crammed into 220px. The grid is:
```css
grid-template-rows: 36px 1fr 220px 26px;
```

Change to give analysis a proportional share of screen height, and make the main area and analysis share space more equitably.

- [ ] **Step 1: Change grid layout to give analysis more space**

Change:
```css
grid-template-rows: 36px 1fr 220px 26px;
```
To:
```css
grid-template-rows: 36px 1fr 320px 26px;
```

This gives the analysis deck 320px — enough to see a full archetype description without scrolling. On a 1080p screen (minus browser chrome ~100px), the main area gets ~620px and analysis gets 320px. On a 1440p screen, main gets ~990px and analysis gets 320px. Both are workable.

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "ui: analysis deck height 220px → 320px — give the best content more room"
```

---

### Task 8: Add 2D Scatter Toggle Back

**Files:**
- Modify: `index.html` (CSS for toggle button)
- Modify: `index.html` (HTML — viewscreen section)
- Modify: `index.html` (JS — add 2D SVG rendering, toggle logic)

**Context:** The 2D mode was removed in commit `672cb3a`. It needs to come back as an option. The 3D scatter stays default but a toggle button switches to a flat 2D SVG scatter (X vs Y axes, dot size = population, color = cluster). The 2D view is better for analytical reading — you can precisely compare positions along axes without 3D perspective distortion.

- [ ] **Step 1: Add the 2D SVG container**

In the `.viewscreen` section, add an `<svg>` element alongside the canvas:
```html
<svg id="scatter2D" style="display:none; width:100%; height:100%; position:absolute; top:0; left:0;"></svg>
```

- [ ] **Step 2: Update the toggle button**

The `.toggle-3d` button already exists in the header. Change it to a toggle between 2D and 3D:
```html
<button class="toggle-3d active" id="toggleView" onclick="toggleViewMode()">3D</button>
```

- [ ] **Step 3: Implement `toggleViewMode()`**

Remove the dead `const view3D = true;` at line ~1872 (it's never read anywhere). Replace with the new toggle state:

```js
let is3D = true;

function toggleViewMode() {
  is3D = !is3D;
  const btn = document.getElementById('toggleView');
  const canvas = document.getElementById('scatter3D');
  const svg = document.getElementById('scatter2D');
  const legend = document.getElementById('legend3d');

  if (is3D) {
    btn.textContent = '3D';
    btn.classList.add('active');
    canvas.style.display = 'block';
    svg.style.display = 'none';
    if (legend) legend.style.display = 'block';
    resize3D();
    requestRender3D();
  } else {
    btn.textContent = '2D';
    btn.classList.remove('active');
    canvas.style.display = 'none';
    svg.style.display = 'block';
    if (legend) legend.style.display = 'block';
    render2D();
  }
}
```

- [ ] **Step 4: Implement `render2D()`**

Build a 2D SVG scatter:
- X axis = Transition Readiness (eSov)
- Y axis = Power to Execute (ePow)
- Dot size = population (log scale, same as 3D)
- Dot color = cluster color
- Dot opacity = 0.35 + composite/100 * 0.5 (same as 3D)
- Quadrant labels in corners
- Axis labels
- Grid lines at 25, 50, 75
- Hover tooltips (reuse existing tooltip element)
- Click opens detail panel
- Top 10 countries get text labels (collision-avoidable via simple offset)

SVG construction:
```js
function render2D() {
  const svg = document.getElementById('scatter2D');
  if (!svg) return;
  const rect = svg.getBoundingClientRect();
  const W = rect.width, H = rect.height;
  const pad = { top: 30, right: 30, bottom: 40, left: 50 };
  const plotW = W - pad.left - pad.right;
  const plotH = H - pad.top - pad.bottom;

  const sorted = [...data].sort((a, b) => b.composite - a.composite);
  const top10 = new Set(sorted.slice(0, 10).map(c => c.n));

  let html = '';

  // Grid lines
  [25, 50, 75].forEach(v => {
    const x = pad.left + (v / 100) * plotW;
    const y = pad.top + (1 - v / 100) * plotH;
    html += `<line x1="${x}" y1="${pad.top}" x2="${x}" y2="${pad.top + plotH}" class="grid-line"/>`;
    html += `<line x1="${pad.left}" y1="${y}" x2="${pad.left + plotW}" y2="${y}" class="grid-line"/>`;
    html += `<text x="${x}" y="${pad.top + plotH + 14}" class="tick-label" text-anchor="middle">${v}</text>`;
    html += `<text x="${pad.left - 8}" y="${y + 3}" class="tick-label" text-anchor="end">${v}</text>`;
  });

  // Axis labels
  html += `<text x="${pad.left + plotW / 2}" y="${H - 4}" class="axis-label" text-anchor="middle" fill="var(--text-3)">TRANSITION READINESS →</text>`;
  html += `<text x="14" y="${pad.top + plotH / 2}" class="axis-label" text-anchor="middle" fill="var(--text-3)" transform="rotate(-90, 14, ${pad.top + plotH / 2})">↑ POWER TO EXECUTE</text>`;

  // Crosshair at 50,50
  html += `<line x1="${pad.left + plotW * 0.5}" y1="${pad.top}" x2="${pad.left + plotW * 0.5}" y2="${pad.top + plotH}" class="crosshair"/>`;
  html += `<line x1="${pad.left}" y1="${pad.top + plotH * 0.5}" x2="${pad.left + plotW}" y2="${pad.top + plotH * 0.5}" class="crosshair"/>`;

  // Dots (sorted smallest first so big dots don't occlude small ones)
  const byPop = [...data].sort((a, b) => b.pop - a.pop);
  byPop.forEach(c => {
    const cx = pad.left + (c.eSov / 100) * plotW;
    const cy = pad.top + (1 - c.ePow / 100) * plotH;
    const r = Math.max(3, 2 + Math.log10(Math.max(c.pop, 1)) * 2.5);
    const cc = clusterColors[c.cluster] || clusterColors['default'];
    const opacity = 0.35 + c.composite / 100 * 0.5;
    html += `<circle class="scatter-dot" cx="${cx}" cy="${cy}" r="${r}" fill="${cc}" fill-opacity="${opacity}" stroke="${cc}" stroke-opacity="0.3" stroke-width="0.5" data-country="${c.n}"/>`;
  });

  // Top 10 labels
  sorted.slice(0, 10).forEach(c => {
    const cx = pad.left + (c.eSov / 100) * plotW;
    const cy = pad.top + (1 - c.ePow / 100) * plotH;
    const r = Math.max(3, 2 + Math.log10(Math.max(c.pop, 1)) * 2.5);
    html += `<text x="${cx + r + 3}" y="${cy - r}" class="annotation-text">${c.n}</text>`;
  });

  svg.innerHTML = html;

  // Event listeners
  svg.querySelectorAll('.scatter-dot').forEach(dot => {
    dot.addEventListener('mousemove', (e) => {
      const name = dot.getAttribute('data-country');
      const c = data.find(d => d.n === name);
      if (!c) return;
      const tt = document.getElementById('tooltip');
      const pr = svg.closest('.viewscreen').getBoundingClientRect();
      tt.innerHTML = `
        <div class="tt-name">${c.n}</div>
        <div class="tt-row"><span class="tt-label">Population</span><span class="tt-val">${c.pop}m</span></div>
        <div class="tt-divider"></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--energy)">Energy</span><span class="tt-val">${c.ceS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--ppp)">PPP</span><span class="tt-val">${c.ppS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--cleantech)">Tech</span><span class="tt-val">${c.ctS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--compute)">Compute & AI</span><span class="tt-val">${c.coS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--eci)">Complexity</span><span class="tt-val">${c.ecS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--state)">State Cap.</span><span class="tt-val">${c.scS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--momentum)">Momentum</span><span class="tt-val">${c.moS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--resource)">Resource Sec.</span><span class="tt-val">${c.rsS.toFixed(0)}</span></div>
        <div class="tt-row"><span class="tt-label" style="color:var(--climate)">Climate Exp.</span><span class="tt-val">${c.cxS.toFixed(0)}</span></div>
        <div class="tt-divider"></div>
        <div class="tt-row"><span class="tt-label">Composite</span><span class="tt-val" style="color:var(--blue-dark)">${c.composite.toFixed(1)}</span></div>`;
      tt.style.left = (e.clientX - pr.left + 12) + 'px';
      tt.style.top = (e.clientY - pr.top - 8) + 'px';
      tt.classList.add('visible');
    });
    dot.addEventListener('mouseleave', () => {
      document.getElementById('tooltip').classList.remove('visible');
    });
    dot.addEventListener('click', () => {
      const name = dot.getAttribute('data-country');
      if (name) openDetail(name);
    });
  });
}
```

- [ ] **Step 5: Hook into updateIndex**

In `updateIndex()`, add after `update3DScene()`:
```js
if (!is3D) render2D();
```

- [ ] **Step 6: Hook into story highlighting for 2D**

In `playStory()`, after the 3D sphere highlighting block, add SVG dot highlighting:
```js
// 2D SVG dots
document.querySelectorAll('#scatter2D .scatter-dot').forEach(dot => {
  const name = dot.getAttribute('data-country');
  const c = data.find(d => d.n === name);
  if (!c) return;
  const cc = clusterColors[c.cluster] || clusterColors['default'];
  if (!activeStory) {
    dot.setAttribute('fill', cc);
    dot.setAttribute('fill-opacity', 0.35 + c.composite / 100 * 0.5);
  } else if (isTargetCountry(c)) {
    dot.setAttribute('fill', cc);
    dot.setAttribute('fill-opacity', '0.95');
    dot.setAttribute('stroke-width', '2');
  } else {
    dot.setAttribute('fill', '#b0b8c4');
    dot.setAttribute('fill-opacity', '0.08');
    dot.setAttribute('stroke-width', '0');
  }
});
```

- [ ] **Step 7: Test both modes**

Open in browser:
- Toggle between 2D and 3D
- Verify dots render in correct positions in both modes
- Verify tooltips work in both modes
- Verify story highlighting works in both modes
- Verify country click opens detail in both modes

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: restore 2D scatter view with toggle — SVG scatter, tooltips, story highlighting"
```

---

### Task 9: Add Camera Reset Button to 3D View

**Files:**
- Modify: `index.html` (HTML — viewscreen header)
- Modify: `index.html` (JS — reset function)
- Modify: `index.html` (CSS — button styling)

- [ ] **Step 1: Add reset button next to 3D toggle**

In the `.vs-header` section, after the toggle button, add:
```html
<button class="toggle-3d" id="resetCamera" onclick="resetCamera()" title="Reset camera">↺</button>
```

- [ ] **Step 2: Implement resetCamera()**

```js
function resetCamera() {
  if (!camera3D || !controls3D) return;
  camera3D.position.set(70, 60, 70);
  controls3D.target.set(50, 50, 50);
  controls3D.update();
  requestRender3D();
}
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add camera reset button for 3D view"
```

---

### Task 10: Fix Tooltip Clipping

**Files:**
- Modify: `index.html` (JS — tooltip positioning in `on3DMouseMove` and 2D event handlers)

- [ ] **Step 1: Create tooltip clamping utility**

```js
function positionTooltip(tt, clientX, clientY, container) {
  const pr = container.getBoundingClientRect();
  const ttW = 200; // approximate tooltip width
  const ttH = 280; // approximate tooltip height
  let x = clientX - pr.left + 12;
  let y = clientY - pr.top - 8;
  // Clamp right edge
  if (x + ttW > pr.width) x = clientX - pr.left - ttW - 12;
  // Clamp bottom edge
  if (y + ttH > pr.height) y = pr.height - ttH - 8;
  // Clamp top edge
  if (y < 8) y = 8;
  tt.style.left = x + 'px';
  tt.style.top = y + 'px';
}
```

- [ ] **Step 2: Replace tooltip positioning in `on3DMouseMove`**

Replace:
```js
tt.style.left=(e.clientX-pr.left+12)+'px';
tt.style.top=(e.clientY-pr.top-8)+'px';
```
With:
```js
positionTooltip(tt, e.clientX, e.clientY, canvasEl.closest('.viewscreen'));
```

- [ ] **Step 3: Use same function in 2D tooltip handler**

The 2D render function (Task 8) should also use `positionTooltip`.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "fix: clamp tooltip to viewport — no more off-screen clipping"
```

---

### Task 11: Stop dampLoop from Running When Idle

**Files:**
- Modify: `index.html:2224-2232` (dampLoop function)

**Context:** The damping loop runs `requestAnimationFrame(dampLoop)` forever, burning 60fps of CPU even when the user isn't interacting. Replace with an on-demand approach.

- [ ] **Step 1: Replace the continuous loop with interaction-driven damping**

Replace the dampLoop implementation:

```js
let dampActive = false;
let dampTimeout = null;

function dampLoop() {
  if (!controls3D || !dampActive) return;
  controls3D.update();
  requestRender3D();
  if (controls3D.enableDamping) {
    requestAnimationFrame(dampLoop);
  } else {
    dampActive = false;
  }
}

// Start damping on interaction (add these after controls3D is created in init3DScene)
controls3D.addEventListener('start', () => {
  if (dampTimeout) { clearTimeout(dampTimeout); dampTimeout = null; }
  if (!dampActive) {
    dampActive = true;
    dampLoop();
  }
});
controls3D.addEventListener('end', () => {
  // Let damping settle for ~1 second then stop. Clear on re-start to avoid race condition.
  dampTimeout = setTimeout(() => { dampActive = false; dampTimeout = null; }, 1200);
});
```

Note: Move the `controls3D.addEventListener` calls to after `controls3D` is created in `init3DScene()`.

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "perf: stop dampLoop when idle — no more 60fps CPU burn"
```

---

### Task 12: Add Mobile Breakpoint / Desktop Gate

**Files:**
- Modify: `index.html` (CSS — add media query)
- Modify: `index.html` (HTML — add mobile notice)

**Context:** The fixed grid layout (`210px 1fr 270px`) doesn't work below ~800px. Adding full responsive layout is a separate project. For now, add a mobile gate that shows a message and allows scrolling the analysis on small screens.

- [ ] **Step 1: Add mobile gate styles and media query**

Add to CSS:
```css
.mobile-gate {
  display: none;
  padding: 2rem 1.5rem;
  text-align: center;
  font-family: var(--sans);
  color: var(--text-2);
  height: 100vh;
  overflow-y: auto;
  background: var(--surface);
}

.mobile-gate h2 {
  font-family: var(--display);
  font-size: 1.4rem;
  font-weight: 800;
  color: var(--blue-dark);
  margin-bottom: 1rem;
}

.mobile-gate p {
  font-size: 0.9rem;
  line-height: 1.7;
  margin-bottom: 0.8rem;
  max-width: 480px;
  margin-left: auto;
  margin-right: auto;
}

@media (max-width: 860px) {
  .bridge { display: none !important; }
  .mobile-gate { display: block; }
  html, body { overflow: auto; }
}
```

- [ ] **Step 2: Add mobile gate HTML**

Add before the `.bridge` div:
```html
<div class="mobile-gate">
  <h2>21st Century Resilience Index</h2>
  <p>This interactive tool is designed for desktop screens (860px+ width). The 3D scatter plot, weight sliders, and analysis panels need the space.</p>
  <p>Open this page on a laptop or desktop browser for the full experience.</p>
  <p style="font-family:var(--mono);font-size:0.7rem;color:var(--text-4);margin-top:2rem;">SYS.21CRI v7.0 // 85 NATIONS // 9 DIMENSIONS</p>
</div>
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "ui: add mobile gate — shows message on screens below 860px"
```

---

## Chunk 3: Interaction & Usability Improvements

---

### Task 13: Remove the "System" Readout — Replace with Useful Space

**Files:**
- Modify: `index.html:1513-1524` (system readout HTML)

**Context:** The "System" section ("Nations: 85 / Model: 21CRI v7.0 / Status: ACTIVE") is decorative. It takes up ~80px of vertical space in the left panel that could be used for slider area. Remove it.

- [ ] **Step 1: Remove the system readout section**

Delete the `<div class="lp-section">` block containing the sys-readout (lines ~1513-1524).

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "ui: remove decorative system readout — reclaim left panel space"
```

---

### Task 14: Add Mini-Bar Legend to Rankings Panel

**Files:**
- Modify: `index.html` (HTML — right panel, above rankings scroll)

**Context:** The rankings panel shows mini sparkline bars per country but there's no indication of what each colored bar represents. Add a tiny legend.

- [ ] **Step 1: Add legend above the search box**

In the right panel `rp-section`, before the search input, add:
```html
<div style="display:flex;gap:2px;align-items:center;margin-bottom:0.4rem;flex-shrink:0;">
  <span class="mini-bar energy" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar ppp" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar cleantech" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar compute" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar eci" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar state" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar momentum" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar resource" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span class="mini-bar climate" style="width:6px;height:6px;display:inline-block;border-radius:1px;"></span>
  <span style="font-family:var(--mono);font-size:0.55rem;color:var(--text-4);margin-left:4px;">EN PP CT CO EC SC MO RS CX</span>
</div>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "ui: add mini-bar legend to rankings panel"
```

---

### Task 15: Fix Inline Event Handlers — Move to addEventListener

**Files:**
- Modify: `index.html` (HTML — remove onclick attributes)
- Modify: `index.html` (JS — add event listeners)

**Context:** Multiple elements use `onclick="playStory('reset')"` style inline handlers. Move these to `addEventListener` calls for cleaner separation.

- [ ] **Step 1: Replace inline handlers on story buttons**

Remove `onclick` from all story buttons. Add `data-story` attributes instead:
```html
<button class="story-btn" data-story="reset">Reset</button>
<button class="story-btn" data-story="petroClock">Petrostate Clock</button>
<button class="story-btn" data-story="usGamble">America's Gamble</button>
<button class="story-btn" data-story="moving">Who's Moving</button>
<button class="story-btn" data-story="breadth">Breadth Wins</button>
<button class="story-btn" data-story="crisis">Crisis States</button>
```

In JS, add:
```js
document.querySelectorAll('.story-btn[data-story]').forEach(btn => {
  btn.addEventListener('click', () => playStory(btn.dataset.story));
});
```

- [ ] **Step 2: Replace inline handlers on slider inputs**

Remove `oninput="updateIndex()"` from all weight slider inputs. Add in JS:
```js
document.querySelectorAll('#w1,#w2,#w3,#w4,#w5,#w6,#w7,#w8,#w9').forEach(el => {
  el.addEventListener('input', updateIndex);
});
```

Remove `oninput="adjustAxisWeight('X')"` etc from axis sliders. Add in JS:
```js
['X','Y','Z'].forEach(axis => {
  document.getElementById('ax' + axis).addEventListener('input', () => adjustAxisWeight(axis));
});
```

- [ ] **Step 3: Replace inline handlers on narrative buttons**

Remove `onclick="nextSlide()"` and `onclick="dismissNarrative()"`. Add:
```js
document.querySelectorAll('.narrative-btn').forEach(btn => {
  if (btn.textContent.trim() === 'Skip') {
    btn.addEventListener('click', dismissNarrative);
  } else if (btn.textContent.trim() === 'Next') {
    btn.addEventListener('click', nextSlide);
  } else if (btn.textContent.trim() === 'Explore the data') {
    btn.addEventListener('click', dismissNarrative);
  }
});
```

- [ ] **Step 4: Update `playStory` to find buttons by data attribute**

In `playStory()`, replace:
```js
const btn = document.querySelector('[onclick="playStory(\'' + key + '\')"]');
```
With:
```js
const btn = document.querySelector('[data-story="' + key + '"]');
```

- [ ] **Step 5: Fix dynamically-created story popup close button**

The popup close button is generated via `innerHTML` in `playStory()` (line ~2596). Since it's created dynamically, the `data-story` event delegation won't catch it. Replace the inline handler with addEventListener after creation:

Change in `playStory()`:
```js
popup.innerHTML = '<button class="story-close" onclick="playStory(\'reset\')">&times;</button>' + ...
```
To:
```js
popup.innerHTML = '<button class="story-close">&times;</button>' + ...
```
Then after `document.querySelector('.viewscreen').appendChild(popup)`, add:
```js
popup.querySelector('.story-close').addEventListener('click', () => playStory('reset'));
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "refactor: replace all inline onclick handlers with addEventListener"
```

---

### Task 16: Lock Three.js CDN Version with Integrity Hash

**Files:**
- Modify: `index.html:10-11` (script tags)

- [ ] **Step 1: Add integrity hashes to Three.js CDN scripts**

Replace the bare CDN scripts with integrity-locked versions. Use the exact same r128 version but with SRI:

The implementer MUST compute fresh hashes at implementation time — do not use pre-computed hashes from the plan. Fetch the CDN files and compute:

```bash
curl -s https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js | openssl dgst -sha512 -binary | openssl base64 -A
curl -s https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js | openssl dgst -sha256 -binary | openssl base64 -A
```

Then apply as:
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js" integrity="sha512-{COMPUTED_HASH}" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js" integrity="sha256-{COMPUTED_HASH}" crossorigin="anonymous"></script>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "sec: lock Three.js CDN versions with SRI integrity hashes"
```

---

## Chunk 4: Analytical Accuracy & Narrative Coherence

---

### Task 17: Reconcile Momentum Data — Raw Values vs Analysis Text

**Files:**
- Modify: `index.html` (analysis prose sections mentioning momentum values)

**Context:** The raw `mo` field contains blended 20-year momentum composite scores (31.7-77.6). The analysis text discusses percentage-point clean energy changes ("Finland +20.6pp"). These are different things with the same label. The analysis must distinguish them clearly.

- [ ] **Step 1: Audit all momentum references in analysis text**

Find every mention of momentum with a specific numeric value. Determine if it's referring to:
(a) The raw energy share change in pp (what the analysis text describes)
(b) The blended composite momentum score (what the slider/tooltip shows)

- [ ] **Step 2: Clarify in the analysis text**

Where the analysis discusses momentum as energy change:
- Add "(clean energy component)" or qualify as "energy momentum"
- Where composite scores are shown, label them as "composite 20yr momentum"

For example, in the momentum dimension description, change:
"Finland (+20.6pp)" → "Finland (+20.6pp clean energy; 41.0 composite momentum)"

Or simpler: in the "What Different Weight Configurations Reveal" section under "Weight only 20yr Momentum", clarify:
"Composite momentum blends clean energy change (30%), ECI improvement (25%), GDP growth (25%), and digital transformation (20%). The headline numbers below are composite scores, not energy change alone."

- [ ] **Step 3: Update the dimension tooltip text**

The tooltip for Momentum (line ~1420) already says "Composite 20-year momentum" — good. But add to analysis text the clarification.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "docs: clarify momentum values — distinguish composite score from energy-only change"
```

---

### Task 18: Address the Internet Penetration Concern in Momentum

**Files:**
- Modify: `index.html` (momentum dimension tooltip, analysis text)

**Context:** The momentum composite weights internet penetration at 20%. For developed countries at 80%+ penetration, this dimension is saturated and contributes essentially nothing. This means the composite mostly measures GDP + energy for rich countries and internet + GDP for poor ones. The analysis should acknowledge this explicitly.

- [ ] **Step 1: Add a note to the analysis "What the Index Cannot See" section**

Add a new blind spot:
```html
<div class="blind-spot">
  <div class="blind-spot-label">Momentum Saturation</div>
  <p>The 20-year momentum composite weights internet penetration at 20%. For developed countries already at 80%+ penetration, this component is saturated — the composite effectively becomes 37.5% energy, 31.25% ECI, 31.25% GDP. For developing countries gaining from 10% to 60%, internet growth inflates momentum scores. The composite measures different things for different income brackets.</p>
</div>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "docs: add momentum saturation blind spot — internet penetration ceiling effect"
```

---

### Task 19: Acknowledge Clean Tech Binary Distribution in Analysis

**Files:**
- Modify: `index.html` (analysis text)

**Context:** Clean Tech is effectively a binary dimension (China vs everyone else). The analysis acknowledges concentration but should explicitly note the index design implication: weighting Clean Tech is equivalent to voting for or against China.

- [ ] **Step 1: Add a note to the weight configuration section**

In the analysis, after the existing "Weight only Compute & AI" section, update the clean tech weight test result:

```html
<div class="weight-test">
  <div class="wt-label">Weight only Clean Tech</div>
  <div class="wt-result">China dominates at 100. The next highest is Germany at 28. This dimension is effectively binary — China vs everyone else. 80% of solar manufacturing, 83% of batteries, 70% of EVs. Weighting clean tech heavily is equivalent to asking "how much credit should one country get for monopoly production?" This is a design feature, not a bug — but users should understand the leverage it creates.</div>
</div>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "docs: acknowledge clean tech binary distribution in weight configuration analysis"
```

---

### Task 20: Acknowledge Petrostate Tautology in Analysis

**Files:**
- Modify: `index.html` (analysis findings section)

**Context:** The finding that "petrostates cluster at the bottom under every configuration except pure PPP" is baked into indicator selection — countries with near-zero clean energy, near-zero ECI, near-zero clean tech will score badly by definition. The analysis should acknowledge this is a design decision, not a discovery.

- [ ] **Step 1: Add a note to the Five Findings section**

After the existing petrostate finding, add a clarification sentence:

"Note: this finding is partially tautological — the choice to measure clean energy, manufacturing complexity, and clean tech production guarantees that fossil-fuel-dependent economies with narrow industrial bases score poorly. The finding is not that petrostates are 'bad' — it's that the dimensions chosen to measure 21st-century resilience are precisely the ones petrostates lack. This is the thesis of the index made visible, not an independent discovery."

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "docs: acknowledge petrostate finding as design decision, not independent discovery"
```

---

### Task 21: Tighten Electrostate Framing vs Actual Index Scope

**Files:**
- Modify: `index.html` (narrative intro slides, analysis header)

**Context:** The framing says "electrostates" — nations whose power comes from manufacturing clean energy technology. But 5 of 9 dimensions (PPP, state capacity, momentum, resource security, climate exposure) aren't electrostate-specific. The index actually measures "transition resilience" more broadly. The narrative should acknowledge this.

- [ ] **Step 1: Update the first narrative slide**

Change:
"This index measures which countries are positioned to lead that transition, and which are trapped."

To:
"This index measures which countries have the compound capabilities — energy, technology, economy, governance, and environmental resilience — to lead that transition, and which are structurally trapped."

- [ ] **Step 2: Update the analysis opening**

In the analysis text (line ~1592), ensure the framing uses "transition resilience" alongside "electrostate" to signal the broader scope.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "docs: broaden framing — transition resilience alongside electrostate narrative"
```

---

## Chunk 5: Final Polish

---

### Task 22: Add Country Comparison Mode (Lightweight)

**Files:**
- Modify: `index.html` (JS — detail panel, HTML)

**Context:** Full side-by-side comparison is a feature. For now, a lightweight version: clicking a country in the rankings while the detail panel is open for a different country shows a comparison bar overlay. This is the minimum viable comparison.

- [ ] **Step 1: Add comparison state**

```js
let compareCountry = null;
```

- [ ] **Step 2: Modify openDetail to support comparison overlay**

When the detail panel is already open and a new country is clicked, show both sets of bars:

```js
function openDetail(name) {
  const c = data.find(d => d.n === name);
  if (!c) return;

  const panel = document.getElementById('countryDetail');
  const isOpen = panel.classList.contains('open');
  const currentName = document.getElementById('detailName').textContent;

  // If panel is already open for a different country, show comparison
  if (isOpen && currentName !== name) {
    compareCountry = c;
    renderComparison(data.find(d => d.n === currentName), c);
    return;
  }

  compareCountry = null;
  // ... existing openDetail code ...
}
```

- [ ] **Step 3: Implement renderComparison**

```js
function renderComparison(primary, compare) {
  const nameEl = document.getElementById('detailName');
  nameEl.innerHTML = primary.n + ' <span style="color:var(--text-4);font-size:0.7rem;">vs</span> ' + compare.n;

  const dims = [
    {key:'energy',label:'Energy',v1:primary.ceS,v2:compare.ceS},
    {key:'ppp',label:'PPP',v1:primary.ppS,v2:compare.ppS},
    {key:'cleantech',label:'Clean Tech',v1:primary.ctS,v2:compare.ctS},
    {key:'compute',label:'Compute & AI',v1:primary.coS,v2:compare.coS},
    {key:'eci',label:'ECI',v1:primary.ecS,v2:compare.ecS},
    {key:'state',label:'State Cap.',v1:primary.scS,v2:compare.scS},
    {key:'momentum',label:'Momentum',v1:primary.moS,v2:compare.moS},
    {key:'resource',label:'Resource Sec.',v1:primary.rsS,v2:compare.rsS},
    {key:'climate',label:'Climate Exp.',v1:primary.cxS,v2:compare.cxS},
  ];

  document.getElementById('detailBars').innerHTML = dims.map(d => `
    <div class="detail-bar-row">
      <span class="detail-bar-label ${d.key}">${d.label}</span>
      <div class="detail-bar-track">
        <div class="detail-bar-fill ${d.key}" style="width:${d.v1}%"></div>
        <div class="detail-bar-fill" style="width:${d.v2}%;position:absolute;top:0;left:0;height:100%;background:var(--text-4);opacity:0.35;"></div>
      </div>
      <span class="detail-bar-val">${d.v1.toFixed(0)} / ${d.v2.toFixed(0)}</span>
    </div>
  `).join('');

  document.getElementById('detailComposite').textContent =
    'COMPOSITE ' + primary.composite.toFixed(1) + ' vs ' + compare.composite.toFixed(1);
  document.getElementById('detailDesc').textContent = '';
}
```

- [ ] **Step 4: Clear comparison on close**

In `closeDetail()`, add:
```js
compareCountry = null;
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: lightweight country comparison — click a second country to overlay bars"
```

---

### Task 23: Final Verification Pass

**Files:**
- All modified files

- [ ] **Step 1: Open index.html in browser and verify**

Checklist:
- [ ] All cluster colors match analytical expectations (leaders green, latent orange, power blue, petro red, crisis dark red, fragile purple)
- [ ] US and China are the only "power" cluster countries
- [ ] Japan, South Korea, Germany, Taiwan, UK, Singapore are "latent"
- [ ] Nordics, Switzerland, France, Austria, New Zealand are "leader"
- [ ] Brazil, Peru, Colombia, Ecuador, Venezuela, Sri Lanka are "fragile"
- [ ] Bangladesh, Philippines, India, Pakistan, Nigeria are "crisis"
- [ ] Gulf states are "petro"
- [ ] Chile falls to "default" (the exception)
- [ ] Compute scores in tooltips match analysis text (US:95, China:71)
- [ ] Font sizes are all readable
- [ ] Analysis deck has 320px height
- [ ] 2D/3D toggle works
- [ ] Camera reset works
- [ ] Tooltips don't clip off screen
- [ ] Story presets work in both 2D and 3D
- [ ] Country comparison works
- [ ] Mobile gate shows on narrow screens
- [ ] dampLoop stops when not interacting

- [ ] **Step 2: Run a diff and review all changes**

```bash
git diff HEAD~10 --stat
```

- [ ] **Step 3: Final commit if any fixups needed**

```bash
git add index.html
git commit -m "fix: final verification pass — polish and fixups"
```

---

## Task Dependency Map

```
Tasks 1-5 (Data/Cluster) → independent of each other, can parallelize
Task 6 (Fonts) → independent
Task 7 (Analysis height) → independent
Task 8 (2D toggle) → depends on nothing, but Tasks 10, 11, 15 touch related code
Task 9 (Camera reset) → independent
Task 10 (Tooltip clamp) → after Task 8 (uses 2D tooltip code)
Task 11 (dampLoop) → independent
Task 12 (Mobile gate) → independent
Task 13 (Remove system readout) → independent
Task 14 (Mini-bar legend) → independent
Task 15 (Inline handlers) → after Task 8 (touches story buttons, sliders)
Task 16 (CDN integrity) → independent
Tasks 17-21 (Analysis text) → independent of each other, can parallelize
Task 22 (Comparison) → independent
Task 23 (Verification) → after ALL other tasks
```

**IMPORTANT: Single-file constraint.** All 23 tasks modify `index.html`. True parallelization with sub-agents will cause merge conflicts. Execute ALL groups sequentially within the same agent session. The groups below define *logical ordering*, not parallel execution:

1. **Group A (Data/Cluster):** Tasks 1, 2, 3, 4, 5 — sequential, they interact
2. **Group B (Layout/UX):** Tasks 6, 7, 12, 13, 14 — sequential (all edit CSS/HTML in same file)
3. **Group C (Features):** Tasks 8, 9, then 10, 11, 15 — sequential
4. **Group D (Analysis text):** Tasks 17, 18, 19, 20, 21 — sequential (all edit analysis prose section)
5. **Group E (Polish):** Tasks 16, 22 — sequential
6. **Group F (Verify):** Task 23 — last

Groups A through E can be done in any order relative to each other as long as each group is internally sequential. Recommended order: A → B → C → D → E → F.
