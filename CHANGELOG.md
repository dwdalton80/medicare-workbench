# Changelog

Dated history of substantive changes to the Medicare Options Workbench.

## 2026-09-12 — Book of business bulk CSV check

Added a "Book of business — bulk CSV check" card so a whole client list can be screened in one pass instead of one client at a time:

- CSV upload with a runtime column-mapper: the tool reads the file's header row, guesses which column is which (name, state, date of birth, marital status, monthly income, resources, current coverage type, months without Part B/Part D), and lets the broker correct any guess or mark a field "Not in this file." This avoids hardcoding assumptions about any one CRM's export format (built with Spark Advisors exports in mind, but works with any CSV).
- Each row is scored using the exact same constants and calculation logic already used by the single-client form (`R`, `MEDIGAP`, `bandStatus()`) — the MSP two-pass tier match, Extra Help/Medicaid/SSI band logic, and late-enrollment penalty formulas are shared, not reimplemented, so bulk results can't drift out of sync with a one-off screen.
- New: an Oklahoma Medigap birthday-rule window calculator that works from an actual date of birth (month + day), computing whether a client is currently inside their 60-day window (30 days before their birthday through 60 days after), how many days remain, or when the next window opens. Only flagged for Oklahoma, and only for clients recorded as currently holding a Medigap plan (or with coverage type left unmapped, in which case the result carries a "confirm on Medigap" caveat).
- Income and resources are treated as optional per row — rows missing either show "Needs income" for the affected programs rather than a false "does not qualify," since most CRM exports (Spark Advisors included) don't carry income/asset data.
- Results table defaults to "show only clients with an action flag" so a large book collapses to the handful worth a call; a "Download results as CSV" button exports the flagged (or full) list.
- Verified with Playwright: auto-column-detection against a representative sample file, an in-window and an out-of-window Oklahoma birthday-rule case, a Texas row (correctly N/A for the birthday rule), rows with missing DOB/income, LEP math on a penalty row, CSV export round-trip, re-uploading a second file with a minimal column set, and no horizontal scroll or console errors at 360/390/430px.

## Prior to 2026-09-12 — CMS Landscape plan lookup, and earlier feature rounds

Before this entry, the tool had already grown a "CMS Landscape plan lookup" (Section 3 of the form): a county selector plus an embedded CY2026 CMS Medicare Advantage & Part D Landscape File dataset, letting a broker search and pull a real plan's premium, MOOP, deductible, and star rating directly into Option B or C with one click. It had also already accumulated: an IRMAA bracket lookup (auto-fills the Part B premium override and Part D surcharge from filing status + MAGI, including the married-filing-separately-lived-apart exception), a late-enrollment penalty calculator (Part B and Part D, matching Medicare.gov's own worked examples), a mobile-responsive layout pass, and the initial accuracy audit that fixed the Medicare Savings Program tier-matching logic and a borderline-income status bug. See the in-app "2026 program thresholds" panel for the current sourcing on every dollar figure in use.

## 2026 figures reference

| Figure | 2026 value | Source |
|---|---|---|
| Part B premium | $202.90/mo | CMS |
| Part B deductible | $283 | CMS |
| Part A deductible | $1,736 | CMS |
| Part D deductible (max) | $615 | Medicare.gov |
| Part D true OOP cap | $2,100 | Medicare.gov |
| Part D national base beneficiary premium | $38.99 | CMS CY2026 Parts C&D Announcement |
| Extra Help income limit, individual | $23,940/yr | SSA |
| Extra Help income limit, married | $32,460/yr | SSA |
| Extra Help resource limit, individual | $18,090 | SSA |
| Extra Help resource limit, married | $36,100 | SSA |
| QMB income limit, individual | $1,350/mo | Medicare.gov |
| SLMB income limit, individual | $1,616/mo | Medicare.gov |
| QI income limit, individual | $1,816/mo | Medicare.gov |
| MSP resource limit, individual | $9,950 | Medicare.gov |
| MSP resource limit, married | $14,910 | Medicare.gov |
| SSI federal benefit rate, individual | $994/mo | SSA |
| SSI federal benefit rate, married | $1,491/mo | SSA |
| SSI resource limit, individual | $2,000 | SSA |
| SSI resource limit, married | $3,000 | SSA |
| Oklahoma ABD Medicaid income limit, individual | $1,350/mo | OAC 317:35-7-38 |
| Texas ABD Medicaid income limit, individual | $994/mo | Texas HHS |
| IRMAA tier 1 threshold (single / MFJ) | $109,000 / $218,000 | CMS |
| IRMAA tier 1 Part B / Part D | $284.10 / $14.50 | CMS |
| IRMAA top tier Part B / Part D | $689.90 / $91.00 | CMS |

**Caveats:** Medicaid ABD, MSP, and SSI figures are simplified single-pathway screens — real state Medicaid programs have more routes to eligibility (spend-downs, medically-needy pathways) than a single income/resource line captures. IRMAA is based on MAGI from **two years prior** to the plan year. All figures current as of the last update noted above; reverify against CMS/SSA/state sources each fall.
