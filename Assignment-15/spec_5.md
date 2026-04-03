# Drug Safety Explorer — Project Specification

**Course:** AI Class  
**Tool:** Single-page web application (HTML/CSS/JS)  
**Data Source:** OpenFDA Public API  
**Deployment Target:** GitHub Pages  

---

## Overview

Drug Safety Explorer is an interactive, browser-based tool that allows users to explore drug safety information drawn from the U.S. Food and Drug Administration's publicly available databases. The tool queries live FDA data across three endpoints — adverse events (FAERS), drug recalls (enforcement), and drug labeling — and surfaces meaningful safety signals through visual comparisons, severity breakdowns, and plain-language educational context.

The core design question driving the project was: *what's the most useful way for a non-expert to explore drug safety data without being misled by it?* The answer shaped two primary modes: a **Drug Comparison** mode for investigating specific drugs side-by-side, and a **Drug Class Explorer** mode for comparing multiple medications within the same therapeutic category (e.g. all statins, all SSRIs).

---

## OpenFDA Endpoints Used

All data is fetched live from `https://api.fda.gov` at query time — no data is hardcoded or cached.

### 1. Drug Adverse Events — `/drug/event.json`
The primary endpoint, querying the FDA Adverse Event Reporting System (FAERS).

| Query | Purpose |
|---|---|
| `search=patient.drug.medicinalproduct:"<name>"` | Total adverse event report count |
| `+ AND serious:1` | Count of serious events |
| `+ AND seriousnessdeath:1` | Count of reports with death outcome |
| `+ AND seriousnesshospitalization:1` | Count of hospitalization reports |
| `count=patient.reaction.reactionmeddrapt.exact` | Top reported reactions (MedDRA terms) |
| `count=patient.patientsex` | Sex distribution of reporters |
| `count=patient.reaction.reactionoutcome` | Outcome distribution (recovered, fatal, etc.) |
| Two-drug AND query | Co-administration analysis |

### 2. Drug Enforcement (Recalls) — `/drug/enforcement.json`
| Query | Purpose |
|---|---|
| `search=product_description:"<name>", sort=report_date:desc` | Recent recalls, sorted newest-first |

Returns recall classification (Class I/II/III), reason for recall, recalling firm, and initiation date.

### 3. Drug Labels — `/drug/label.json`
| Query | Purpose |
|---|---|
| `search=openfda.generic_name:"<name>"` | Structured prescribing information |
| `count=openfda.generic_name.exact` | Autocomplete suggestions |
| `count=openfda.brand_name.exact` | Autocomplete suggestions (brand names) |

Returns structured label fields: warnings, indications, contraindications, adverse reactions, and drug interactions.

---

## Features Implemented

### Core Requirements
- ✅ Live API calls to at least one OpenFDA endpoint (queries 3 endpoints across 9+ parallel requests)
- ✅ Single-page web application deployable to GitHub Pages (no build step, no dependencies)
- ✅ Educational use disclaimer prominently displayed
- ✅ Required FDA attribution in the footer

### Ideas to Explore (from assignment brief)

| Feature | Status | Notes |
|---|---|---|
| Side-by-side comparison view with tabs | ✅ Implemented | 7 tabs: Overview, Reactions, Outcomes, Demographics, Recalls, Label Info, Co-Admin |
| Pre-populated example on page load | ✅ Implemented | Warfarin + Ibuprofen loads automatically |
| Visual charts for adverse events | ✅ Implemented | Ranked bar charts in Top Reactions and Overview tabs |
| Recall timeline with classification colors | ✅ Implemented | Class I (red) / II (amber) / III (blue) color coding with dates and reasons |
| Autocomplete drug search | ✅ Implemented | Queries label endpoint as user types with 300ms debounce |
| Co-administration analysis | ✅ Implemented | Dedicated tab; queries FAERS for reports where both drugs appear together |
| Handling edge cases gracefully | ✅ Implemented | Zero-result states, API 404s, missing fields, and network errors all handled with clear messages |

### Stretch Challenge 1 — Help & Educational Popups
Contextual `?` help icons appear throughout the interface. Each opens a modal with plain-language explanation of what the data means and how to interpret it.

**Modals implemented:**
- *How to Read This Data* — general overview, what FAERS can and cannot tell you
- *About FAERS Reports* — voluntary reporting, denominator problem, reporting bias, correlation vs. causation
- *Known Dangerous Drug Pairs* — five classic dangerous combinations with mechanism explanations (Warfarin + NSAIDs, MAO inhibitors + serotonergic drugs, Methotrexate + NSAIDs, Statins + CYP3A4 inhibitors, ACE inhibitors + potassium-sparing diuretics)
- *Understanding Reported Reactions* — MedDRA coding, why "drug ineffective" appears, how to think about counts vs. rates
- *How to Interpret Outcomes* — explanation of each outcome category, why high fatal % can be misleading
- *Understanding Recall Classifications* — definitions of Class I, II, and III with examples
- *What Drug Labels Actually Tell You* — provenance of label data, black box warnings, key sections
- *Analyzing Drug Co-Administration* — what co-admin counts mean, limitations, when to consult clinical sources
- *Exploring Drug Classes* — how to use the class explorer, what rankings represent

Help icons are `<span>` elements (not `<button>`) to avoid invalid HTML nesting inside tab buttons.

### Stretch Challenge 2 — Drug Class Explorer
A second mode (toggled from the header) that lets users compare all drugs within a therapeutic class simultaneously rather than just two drugs at a time.

**8 pre-built drug classes:**

| Class | Example drugs |
|---|---|
| Statins | atorvastatin, simvastatin, rosuvastatin, pravastatin, lovastatin, fluvastatin |
| SSRIs | sertraline, fluoxetine, escitalopram, paroxetine, citalopram, fluvoxamine |
| ACE Inhibitors | lisinopril, enalapril, ramipril, captopril, benazepril, fosinopril |
| NSAIDs | ibuprofen, naproxen, celecoxib, diclofenac, meloxicam, indomethacin |
| Beta Blockers | metoprolol, atenolol, carvedilol, propranolol, bisoprolol, labetalol |
| PPIs | omeprazole, pantoprazole, esomeprazole, lansoprazole, rabeprazole |
| Benzodiazepines | alprazolam, diazepam, lorazepam, clonazepam, temazepam |
| ARBs | losartan, valsartan, irbesartan, olmesartan, telmisartan, candesartan |

**Output for each class comparison:**
- Card grid ranked by total report volume (top-ranked highlighted in amber)
- Safety leaderboard table with green/red best-in-class and worst-in-class highlighting per column
- Per-drug top 5 adverse reactions bar charts displayed side by side

---

## Design Decisions

### Handling Messy, Sparse, and Variable-Quality Data

The FAERS dataset is notoriously noisy. Several design decisions directly address this:

**1. Rates over raw counts everywhere.**  
Every metric that could be misleading as a raw number is expressed as a percentage. A drug with 2 million reports looks alarming; showing that its death rate is 0.3% is more honest than showing 6,000 deaths.

**2. Explicit data quality warnings built into the UI.**  
Rather than hiding limitations in fine print, warnings are embedded inline — next to the metrics that are most prone to misinterpretation. The comparison table includes a note explaining why higher report counts often indicate more widely prescribed drugs, not more dangerous ones.

**3. Graceful zero-state handling.**  
When a drug returns no results (404, empty results, or zero total), the panel shows a clear message explaining possible reasons (wrong name format, rare drug, etc.) rather than silently showing zeros.

**4. Parallel API requests with `Promise.allSettled`.**  
Each drug query fires 9 simultaneous API requests. Using `allSettled` instead of `Promise.all` means a single failing endpoint (e.g. enforcement returning a 404) doesn't break the entire drug panel — it just returns empty data for that section.

**5. The denominator problem is explained, not hidden.**  
FAERS has no denominator — we don't know how many people took a drug. This is acknowledged explicitly in the "About FAERS Reports" modal and referenced in comparison table footnotes.

### Visual Representation Choices

- **Horizontal bar charts** for reaction rankings: easy to scan, preserves label readability for long MedDRA terms
- **Stacked ratio bar** for sex distribution: immediately communicates proportion without requiring the user to read two separate numbers
- **Mini bar chart + breakdown list** for outcomes: the chart gives a quick gestalt view; the list below gives precise percentages
- **Color semantics are consistent throughout**: green = better/lower risk, amber = moderate/warning, red = higher risk/worst. This applies to metric values, comparison table cells, recall badge borders, and class leaderboard highlights
- **Dark clinical aesthetic** (IBM Plex Mono + dark background): deliberate choice to signal that this is a data tool, not a consumer health product. The terminal-style typography reinforces that the data is raw and requires interpretation

### Data Limitations Communicated to Users

| Limitation | How it's addressed |
|---|---|
| Voluntary reporting / underreporting | FAERS modal, inline note on Overview panels |
| No denominator (can't calculate true incidence) | FAERS modal |
| Correlation ≠ causation | About modal, Co-Admin tab inline note |
| Reporting bias (popular drugs get more reports) | Comparison table footnote, FAERS modal |
| Sex reporting bias (women over-represented) | Inline note in Demographics tab |
| Recall search may not match by generic name | Inline note in Recalls tab when no results found |
| Class rankings are not proven safety rankings | Inline note in Class Explorer leaderboard |

---

## Technical Architecture

```
drug-safety-explorer.html
│
├── CSS (embedded)           — ~300 lines, CSS custom properties for theming
│
├── HTML Structure
│   ├── Header + help buttons
│   ├── Educational disclaimer banner
│   ├── Mode switcher (Drug Comparison / Drug Class Explorer)
│   ├── Drug Comparison mode
│   │   ├── Search inputs with autocomplete
│   │   ├── Example chips
│   │   ├── Tab navigation (7 tabs)
│   │   └── Dynamic content panel (#mc)
│   ├── Drug Class Explorer mode
│   │   ├── Class selector grid (8 classes)
│   │   ├── Drug toggle chips
│   │   └── Dynamic content panel (#clsc)
│   ├── Footer with FDA attribution
│   └── 9 modal overlays
│
└── JavaScript (embedded)    — ~350 lines, vanilla ES2017+ (async/await)
    ├── Modal system (openM, closeM, keyboard/overlay dismiss)
    ├── Mode switching (setMode)
    ├── Autocomplete (doAC, pickAC — debounced 300ms)
    ├── Drug Comparison query (runQ → fetchDrug × n → render)
    ├── Drug Class query (runCls → fetchDrug × n → renderCls)
    ├── 7 tab renderers (rOverview, rReactions, rOutcomes, rDemo, rRecalls, rLabel, rCoadmin)
    └── Utility functions (fn, pct, tr, fd, brow)
```

**Key implementation notes:**
- No external JavaScript dependencies — zero npm, zero build step
- All API calls use the native `fetch()` API
- All state is held in module-level variables (`drugData`, `curTab`, `selCls`, etc.)
- The file is fully self-contained and opens directly in any modern browser
- Fonts load from Google Fonts (IBM Plex Mono + IBM Plex Sans); the page degrades to system monospace/sans if offline

---

## Deployment to GitHub Pages

1. Create a new GitHub repository
2. Add `drug-safety-explorer.html` to the repository root, renamed to `index.html`
3. Go to **Settings → Pages**
4. Set source to `main` branch, root directory
5. The site will be live at `https://<username>.github.io/<repo-name>`

No build process, no `package.json`, no environment variables — the OpenFDA API is fully public and requires no API key for the query volumes used by this tool.

---

## Known Limitations

- **Autocomplete reliability**: The label endpoint's count query doesn't always return clean suggestions for all generic names; some drugs may not appear in autocomplete but still return FAERS data when searched directly.
- **Recall matching**: The enforcement endpoint is indexed by product description and brand name, not always generic name. A search for "ibuprofen" may miss recalls filed under a specific brand or NDC number.
- **FAERS name matching**: FAERS reports use the drug name as submitted, which varies — "IBUPROFEN", "Ibuprofen 200mg", "ibu" may all refer to the same drug. The tool searches by `medicinalproduct` field which captures most common spellings but not all variations.
- **Rate limits**: The public OpenFDA API is rate-limited at 1,000 requests/minute per IP without an API key. The tool fires up to 9 parallel requests per drug (up to 6 drugs in class mode = 54 requests). In practice this is well within limits for individual use, but could be an issue in a classroom demo with many simultaneous users.

---

## Data Attribution

> This product uses publicly available data from the U.S. Food and Drug Administration (FDA). FDA is not responsible for this product and does not endorse or recommend this or any other product.

Data sources:
- FDA Adverse Event Reporting System (FAERS) via OpenFDA
- FDA Drug Enforcement Reports (recalls) via OpenFDA  
- FDA Drug Labeling database via OpenFDA
- API documentation: https://open.fda.gov/apis/drug/
