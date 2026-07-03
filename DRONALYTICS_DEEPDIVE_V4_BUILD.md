# Dronalytics — Admin Deep Dive Analytics
## Build Context & Continuation Document (v4 — Current)

**Purpose:** Paste this document at the start of a new Claude session to rebuild or continue the Dronalytics admin deep dive analytics screen exactly as designed. Claude should read this fully before writing any code.

**Live URL:** `https://sunnygeorge1202-eng.github.io/dronalytics-design-review/deep-dive/`
**Repo:** `sunnygeorge1202-eng/dronalytics-design-review`
**File to update:** `deep-dive/index.html`
**GitHub token:** split in config — `_a = "ghp_zUW82wwdfzm8Y"` + `_b = "4gKFwrD22D8AasooP3EdUJB"`
**Date:** July 2026

---

## 1. What to say to Claude to start

> "I am continuing the Dronalytics admin deep dive analytics build. Read this document fully before doing anything. The live prototype is at `https://sunnygeorge1202-eng.github.io/dronalytics-design-review/deep-dive/` and the repo is `sunnygeorge1202-eng/dronalytics-design-review`, file `deep-dive/index.html`. When I ask you to make changes, fetch the current file from GitHub first using the token, make the change, and push it back."

---

## 2. Product context

Dronalytics is an AI-powered assessment intelligence platform. The admin managing 6 centres across Kerala uses this screen to analyse cognitive-behavioural learning metrics across learner cohorts and compare centres.

The admin's mental model is always: **centre-first, then batch, then individual learner**. He never needs an aggregate across all centres — he needs to see all 6 centres at once and identify which ones need intervention.

### The six centres

| Centre | Approx. students |
|--------|-----------------|
| GIZ Trivandrum | 58 |
| ILS Kochi | 42 |
| ACL Calicut | 61 |
| YMCA Thrissur | 38 |
| KASE Kozhikode | 35 |
| IHRM Palakkad | 28 |

---

## 3. Non-negotiable design rules

These must never be violated when modifying the screen:

1. **Cross-centre comparison always visible** — every primary chart shows all 6 centres simultaneously. Never collapse to a single-centre view on the primary chart.
2. **No numbers on primary charts** — colour and shape communicate meaning. Numbers only appear at Level 2 of the drill-down.
3. **Labels not scores** — all metric values shown as label strings (Developing, Proficient, Calibrated etc.), never raw numeric scores.
4. **CEFR filter wires through everything** — selecting A2/B1/B2 recomputes all distributions and learner rosters across every chart on every tab.
5. **Drill-down is 3 levels deep** — Level 1: colour-only distribution bar → Level 2: counts + delta from previous assessment → Level 3: learner name table.
6. **Full cohort assumed** — all 15 assessments are complete. No partial cohort gating in the current build except CI Time (requires ≥ 3 sessions, so A1 and A2 cells are always N/A).
7. **scrollIntoView after every L1 click** — drill panel auto-scrolls into view after click with 50ms delay.

---

## 4. Screen structure

```
<filter-bar>      CEFR selector only (All levels / A2 / B1 / B2)
                  Learner count display (updates on filter change)

<tab-bar>         Tab 0: Mastery & Decisions
                  Tab 1: Self-Awareness
                  Tab 2: Autonomy & Tool Use

<tab0> <tab1> <tab2>   Content injected by renderTab functions
```

Each tab div is cleared and re-rendered on tab switch or filter change.

---

## 5. What is on each tab

### Tab 0 — Mastery & Decisions

**BPI (Bloom Proficiency Index)**
- Chart: Heatmap (default) + Trajectory toggle
- Card-level selectors: Skill (Writing/Reading/Listening/Speaking) + Bloom level (Remember/Understand/Apply/Analyse/Evaluate/Create)
- Heatmap: 6 rows (centres) × 15 columns (assessments). Cell colour = dominant BPI label with intensity encoding. Thin data cells (n < 5) get one fixed opacity: hex suffix `55` appended to bg colour. No spectrum of fading — one uniform tint.
- Trajectory: line chart per centre, Y = % Proficient+Strong, X = assessments 1–15, area fill at 6% opacity, end-of-line centre name label
- Drill-down: click any heatmap cell → L1 → L2 → L3

**DCI (Decisive Capability Index)**
- Chart: Snapshot toggle (stacked bars) + Heatmap toggle — **DCI is the only metric with two view modes**
- Snapshot: assessment slider + 6 stacked horizontal bars (one per centre), click any bar → drill-down
- Heatmap: 6 rows × 15 columns, cell = dominant DCI label with intensity encoding, click any cell → drill-down
- 6 labels: Fragile / Emerging / Developing / Competent / Strong / Highly Reliable

**Capability Zone**
- Chart: Waffle grid + assessment slider
- One waffle block per centre (3×2 grid layout). Each square = one learner, coloured by zone.
- 4 zones: Zone 1 Ready Performer / Zone 2 Assisted Performer / Zone 3 Emerging Independent / Zone 4 At Risk
- Click any waffle block → L1 drill-down for that centre at selected assessment

**DQS (Decision Quality Score)**
- Chart: Stacked horizontal bars + assessment slider
- 6 bars (one per centre), proportional segments Left=Low / Mid=Improving / Right=Aligned
- No labels or percentages on bars — colour does the work
- Click any bar → L1 drill-down

---

### Tab 1 — Self-Awareness

**CCG (Confidence Calibration Gap)**
- Chart: Dot plot — all 6 centres × 15 assessments as an SVG
- viewBox: 860×110. Y positions: Calibrated = top (y=1.0), Overestimated = mid (y=0.5), Underestimated = low (y=0.15)
- One dot per centre per assessment, coloured by dominant CCG label. Dashed line connects dots per centre.
- Centre line colour from CLRS_C array (6 distinct colours). Dot fill colour from CCG colour array.
- Click any dot → L1 drill-down
- 3 labels: Underestimated / Calibrated / Overestimated

**PI (Procrastination Index)**
- Chart: Heatmap — 6 rows × 15 columns
- Cell colour = dominant PI label with intensity encoding
- 3 labels: Early (green) / Mid (amber) / Late (red)
- Cluster of red cells = consistently late-starting centre — the pattern the heatmap is designed to surface
- Click any cell → L1 drill-down

**UMS (Uncertainty Mastery Score)**
- Chart: Stat cards (one per centre) + assessment slider
- UMS is only computable when unc ≥ 0.60 for that assessment. Each card shows: computable count + colour bar (label distribution) + N/A count
- Click any colour segment on a card → L1 drill-down then immediately L2 for that segment
- 3 labels: Struggling / Improving / Strong
- Note: N/A count and computable count must both be shown. Never merge them.

**CI Time (Performance Consistency)**
- Chart: Heatmap — 6 rows × 15 columns
- A1 and A2 columns always render as N/A cells (dashed border, no colour) — CI Time requires ≥ 3 sessions
- Columns A3–A15 use intensity encoding
- 3 labels: Inconsistent / Moderately stable / Stable
- Click any cell (A3 onwards) → L1 drill-down

---

### Tab 2 — Autonomy & Tool Use

**Tool Use Pattern**
- Chart: Stacked horizontal bars + assessment slider
- 6 bars (one per centre), 6 pattern segments per bar
- 6 patterns: Independent / Passive user / Moderate user / Strategic user / Dependent shortcutter / Over-reliant thinker
- Click any bar → L1 drill-down

**G10 — Impulsive Decisions**
- Chart: Lollipop chart + assessment slider (shared slider with G11)
- 6 lollipop rows, one per centre
- Stem length = % learners at Fire level (fills from left). Dot colour = threshold reached (Fire/Watch/None)
- Vertical threshold marker at 20% position on track
- 3 labels: Fires (≥3) / Watch (1–2) / None
- Click any track → L1 drill-down for that centre

**G11 — Missed Opportunities**
- Chart: Lollipop chart, same assessment slider as G10
- 3 labels: Fires (≥2) / Watch (1) / None
- Same lollipop structure as G10

---

## 6. Three-level drill-down — full spec

Every metric has three drill panels rendered immediately below its card. Panel IDs: `{metricId}_l1`, `{metricId}_l2`, `{metricId}_l3`.

### Level 1 — colour-only distribution

Triggered by clicking any primary chart element.

- Background: `#EEF4F8`, border: `#c5d8e4`
- Header shows: centre name + assessment number
- Thin data warning (amber, BPI only) if n < 5
- Instruction text: "Colour proportion only — click a segment to see counts and change from previous assessment"
- Horizontal bar, height 32px, border-radius 6px
- Each segment: `width = pct%`, `background = M[id].F[i]`. No text inside segments.
- Label row below bar: label name shown only if segment width ≥ 12%
- Clicking a segment calls L2

### Level 2 — counts and delta

Triggered by clicking a segment in Level 1.

- Background: `#F0F8F4`, border: `#b2d9c4`
- Header shows: centre + assessment + total learner count
- Instruction text: "Click any card to see the learners in that category"
- Grid of count cards (max 4 columns). One card per label.
- Each card: count (large), label name, mini progress bar (% of total), delta from previous assessment
- Delta: `+N% from prev` (green) / `-N% from prev` (red) / `no change` (grey) / `first assessment` (grey)
- Clicking a card calls L3

### Level 3 — learner table

Triggered by clicking a count card in Level 2.

- Background: white, border: `#dde3e8`
- Header shows: centre + assessment + learner count in category
- Table columns: Name | Metric label (as coloured pill) | CEFR | Batch
- Label pill: `background = M[id].F[li]`, `color = M[id].T[li]`
- If no learners: show "No learners in this category" in grey italic
- Learners are assigned to labels proportionally from the distribution

---

## 7. Colour system

### Metric colour arrays (fills F, text T, names N)

```
bpi:     F=["#D85A30","#EF9F27","#1D9E75","#085041"]
         T=["#FAECE7","#412402","#E1F5EE","#9FE1CB"]
         N=["Developing","Emerging","Proficient","Strong"]

dci:     F=["#712B13","#D85A30","#EF9F27","#5DCAA5","#1D9E75","#085041"]
         T=["#FAECE7","#FAECE7","#412402","#04342C","#E1F5EE","#9FE1CB"]
         N=["Fragile","Emerging","Developing","Competent","Strong","Highly Reliable"]

zone:    F=["#085041","#1D9E75","#EF9F27","#D85A30"]
         T=["#9FE1CB","#E1F5EE","#412402","#FAECE7"]
         N=["Zone 1 · Ready Performer","Zone 2 · Assisted Performer","Zone 3 · Emerging Independent","Zone 4 · At Risk"]

dqs:     F=["#D85A30","#EF9F27","#1D9E75"]
         T=["#FAECE7","#412402","#E1F5EE"]
         N=["Low","Improving","Aligned"]

ccg:     F=["#2a78d6","#1D9E75","#D85A30"]
         T=["#fff","#E1F5EE","#FAECE7"]
         N=["Underestimated","Calibrated","Overestimated"]

pi:      F=["#1D9E75","#EF9F27","#D85A30"]
         T=["#E1F5EE","#412402","#FAECE7"]
         N=["Early","Mid","Late"]

ums:     F=["#D85A30","#EF9F27","#1D9E75"]
         T=["#FAECE7","#412402","#E1F5EE"]
         N=["Struggling","Improving","Strong"]

citime:  F=["#D85A30","#EF9F27","#1D9E75"]
         T=["#FAECE7","#412402","#E1F5EE"]
         N=["Inconsistent","Mod. stable","Stable"]

pattern: F=["#085041","#B5D4F4","#EF9F27","#1D9E75","#D85A30","#712B13"]
         T=["#9FE1CB","#0C447C","#412402","#E1F5EE","#FAECE7","#FAECE7"]
         N=["Independent","Passive user","Moderate user","Strategic user","Dependent shortcutter","Over-reliant thinker"]

g10:     F=["#D85A30","#EF9F27","#E8EFF3"]
         T=["#FAECE7","#412402","#555"]
         N=["Fires (≥3)","Watch (1–2)","None"]

g11:     F=["#D85A30","#EF9F27","#E8EFF3"]
         T=["#FAECE7","#412402","#555"]
         N=["Fires (≥2)","Watch (1)","None"]
```

### Centre line colours (for CCG dot plot and BPI trajectory)

```javascript
CLRS_C = ["#2a78d6","#1baf7a","#eda100","#4a3aa7","#e34948","#e87ba4"]
// Index 0=GIZ Trivandrum, 1=ILS Kochi, 2=ACL Calicut, 3=YMCA Thrissur, 4=KASE Kozhikode, 5=IHRM Palakkad
```

### Heatmap intensity encoding

For non-BPI heatmaps, append a hex alpha suffix to the dominant label colour:
- `77` when dominant label % < 45%
- `aa` when 45–65%
- `ff` when > 65%

For BPI heatmap, use a 5-stop ramp per label (lightest to darkest):
```
Developing: ["#FAECE7","#F09070","#D85A30","#993C1D","#712B13"]
Emerging:   ["#FAEEDA","#FAC775","#EF9F27","#BA7517","#854F0B"]
Proficient: ["#E1F5EE","#9FE1CB","#5DCAA5","#1D9E75","#0F6E56"]
Strong:     ["#9FE1CB","#5DCAA5","#1D9E75","#0F6E56","#085041"]
```
Ramp index: pct < 40 → 0, < 55 → 1, < 70 → 2, < 85 → 3, ≥ 85 → 4

BPI thin data (n < 5): append `55` to the computed bg colour. One uniform tint only — no spectrum.

---

## 8. Metric thresholds (engine reference)

| Metric | Labels | Thresholds |
|--------|--------|-----------|
| BPI | Developing / Emerging / Proficient / Strong | < 40 / 40–59 / 60–74 / ≥ 75 |
| DCI | Fragile / Emerging / Developing / Competent / Strong / Highly Reliable | < 30 / 30–44 / 45–59 / 60–74 / 75–89 / ≥ 90 |
| Zone | Zone 1–4 | cap_score ≥ 60 AND avg_cas ≥ 60 → Z1; cap_score ≥ 60 AND avg_cas < 60 → Z2; cap_score < 60 AND avg_cas ≥ 60 → Z3; both < 60 → Z4 |
| DQS | Low / Improving / Aligned | DMS²/100: < 40 / 40–69 / ≥ 70 |
| CCG | Underestimated / Calibrated / Overestimated | abs(perceived_conf − BPI): < 10 = Calibrated; 10–24 = Mild gap; ≥ 25 = Large gap; direction from perceived_conf vs BPI sign |
| PI | Early / Mid / Late | attempt_start/window: < 0.20 / 0.20–0.60 / > 0.60 |
| UMS | N/A / Struggling / Improving / Strong | Only when unc ≥ 0.60: mean ES × 100: < 40 / 40–69 / ≥ 70 |
| CI Time | N/A / Inconsistent / Mod. stable / Stable | Needs ≥ 3 sessions: (1 − pstdev/mean) × 100: < 50 / 50–74 / ≥ 75 |
| Pattern | Independent/Passive/Moderate/Strategic/Dep.shortcutter/Over-reliant | TDR < 0.20 = Independent; TDR 0.20–0.50 + TUI < 40 = Passive; TUI 40–69 = Moderate; TUI ≥ 70 = Strategic; TDR > 0.50 + TUI < 40 = Dependent shortcutter; TDR > 0.50 + TUI ≥ 40 = Over-reliant |
| G10 | None / Watch / Fires | imp_count per assessment: 0 / 1–2 / ≥ 3 |
| G11 | None / Watch / Fires | mo_count per assessment: 0 / 1 / ≥ 2 |

---

## 9. Known bugs fixed in v4

| Bug | Root cause | Fix applied |
|-----|-----------|------------|
| BPI drill-down not working | `getBPI()` returns `{vals, n, thin}` object — L1 was treating it as a plain array, `.reduce()` failed silently | In L1: `const raw = getBPI(...); const arr = raw.vals;` — extract `.vals` explicitly |
| L2 crash on BPI | Same object vs array issue | Added guard: `if(!vals \|\| !Array.isArray(vals)) return;` |
| Self-awareness tab not rendering | JS template string error in tab render | Full tab rewrite |
| CEFR filter not wiring | Data functions ignored `selCEFR` global | All data functions now read `selCEFR` and call `getCentreN()` |
| Drill panels off-screen | No scroll after click | `scrollIntoView({behavior:'smooth', block:'nearest'})` added after every L1 render with 50ms delay |

---

## 10. Data functions — synthetic vs production

Current build uses seeded pseudo-random functions. In production replace with API calls.

### Function signatures

```javascript
// Returns CEFR-filtered learner count for a centre
getCentreN(ci)
// Production: SELECT COUNT(*) FROM learners WHERE centre_id=? AND (cefr_level=? OR ?='all')

// Returns [{id, name, cefr, batch}] for a centre
getLearners(ci)
// Production: SELECT id, name, cefr_level, batch FROM learners WHERE centre_id=? AND (cefr_level=? OR ?='all')

// Returns [count_label_0, count_label_1, ...] for any metric except bpi and ums
getDist(ci, ai, metricKey, customSeed)
// Production: SELECT label_field, COUNT(*) FROM results WHERE centre=? AND assessment_no=? AND cefr=? GROUP BY label_field

// Returns {vals:[dev,eme,pro,str], n:int, thin:bool}
getBPI(ci, si, bi, ai)
// si=skill index, bi=bloom index, ai=assessment index
// thin = (n < 5)
// Production: SELECT bpi_label, COUNT(*) FROM results WHERE centre=? AND assessment_no=? AND skill=? AND bloom_level=? AND cefr=? GROUP BY bpi_label

// Returns {comp:int, vals:[strug,imp,str], na:int}
getUMS(ci, ai)
// comp = learners where unc >= 0.60
// na = learners where unc < 0.60
// Production: two queries — count where unc >= 0.60, then label distribution within that group
```

---

## 11. GitHub push pattern

```bash
# 1. Get SHA of current file
SHA=$(curl -s \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  "https://api.github.com/repos/sunnygeorge1202-eng/dronalytics-design-review/contents/deep-dive/index.html" \
  | python3 -c "import sys,json; print(json.load(sys.stdin).get('sha',''))")

# 2. Base64 encode the updated file
CONTENT=$(base64 -w 0 /path/to/updated.html)

# 3. Push
curl -s -X PUT \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"message\":\"your commit message\",\"content\":\"$CONTENT\",\"sha\":\"$SHA\"}" \
  "https://api.github.com/repos/sunnygeorge1202-eng/dronalytics-design-review/contents/deep-dive/index.html"
```

GitHub Pages rebuilds in 2–3 minutes after each push.

---

## 12. What is still pending

- [ ] Admin welcome screen as GitHub Pages file at `/welcome/` (design agreed in chat, not yet built)
- [ ] Real API wiring — replace all synthetic `getDist()`, `getBPI()`, `getUMS()`, `getLearners()`, `getCentreN()` with real API calls to the Dronalytics analytics service
- [ ] Assessment column highlight in heatmap views when a specific assessment is selected from the CEFR filter bar
- [ ] Student analytics screen — individual learner view, two tabs (Cognitive performance + Behavioural performance), separate GitHub Pages file
- [ ] Update this MD in the repo after each significant change to the build

---

## 13. Other files in the repo

| Path | What it is |
|------|-----------|
| `index.html` | Dronalytics-specific screen design review form (6 roles, async, saves to GitHub + Drive) |
| `generic/index.html` | Generic design review form (any product) |
| `bpi-view2/index.html` | Earlier standalone BPI view (superseded by deep-dive) |
| `config.js` | GitHub token (split) + Apps Script URL |
| `responses/` | JSON submissions from design review forms |
| `DRONALYTICS_DEEPDIVE_ANALYTICS_BUILD.md` | Earlier build context document (v3) |

---

## 14. Related systems built in this project

**Design review system** — six role-specific async review rubrics (FE, BE, AI developer, Python developer, QA, PM). Form at `/dronalytics-design-review/`. Responses save to GitHub and Google Drive. Analysis triggered in this Claude project by saying "Analyse responses for [screen] [version]".

**Google Drive folder** — `Dronalytics Design Reviews` (ID: `1iybszP3uL-qB_-Idevyqqzzv9z2LezBe`). Contains design review responses, analysis skill docs, and build context documents.

**Apps Script receiver** — `https://script.google.com/macros/s/AKfycbxvV2lZbgyLXkEC9s6jZ67uaGB07gCyXsC8riIjZiSYM3GrBA46UK05vDxS2BjuONnz/exec` — receives form submissions and saves JSON to Drive subfolders.
