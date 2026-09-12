# Medicare Options Workbench

A single-page, no-install client screening tool for Oklahoma and Texas Medicare brokers. Enter a household's age, state, income, resources, and current coverage, plus up to two alternative plans, and it will:

- Estimate and compare total annual cost across current coverage and up to two alternatives (Part B premium, plan/Part D premium, drug costs, and MOOP-capped medical out-of-pocket).
- Look up real Medicare Advantage and stand-alone Part D plans for the client's county straight from CMS's own CY2026 Landscape File, and pull a plan's premium/MOOP/deductible/star rating into Option B or C with one click.
- Flag network, formulary, and star-rating fit alongside the cost comparison, so a cheaper plan with a dropped doctor or drug doesn't read as automatically "better."
- Screen the household against Extra Help (Part D Low-Income Subsidy), Medicare Savings Programs (QMB / SLMB / QI), state ABD Medicaid (SoonerCare or Texas Medicaid), and SSI — with direct sign-up links when a program looks like a fit.
- Check whether a move to a cheaper or better-fitting plan is actually guaranteed issue, or requires medical underwriting, under Oklahoma or Texas rules — including the state Medigap "birthday rule" (Oklahoma has one, Texas doesn't), under-65 disability/ESRD Medigap access, and handling for clients who aren't enrolled in Medicare yet.
- Look up a client's IRMAA bracket from their filing status and MAGI from two years prior, and apply the resulting Part B premium and Part D surcharge to every option in the cost comparison with one click.
- Estimate Part B and Part D late-enrollment penalties from the number of months a client went without coverage.
- **Upload a whole book of business as a CSV** and re-run the Oklahoma birthday-rule, late-enrollment-penalty, and income-based assistance checks across every client in one pass — see [Book of business bulk check](#book-of-business-bulk-check) below.

It is built for **D&D Insurance Group's internal use**, scoped specifically to Oklahoma and Texas law. It is a screening aid, not a substitute for SSA's, a state Medicaid office's, or CMS's own determinations — see [Limitations](#limitations-and-disclaimers) below.

## Quick start

This is a single self-contained HTML file (`index.html`) with no build step, no server, and no external dependencies beyond two Google Fonts loaded over the network. Three ways to run it:

**1. Open it directly.** Download or clone the repo and open `index.html` in any modern browser (Chrome, Edge, Safari, Firefox). Everything runs client-side — no data is sent anywhere, no backend, no database.

**2. Host it for free with GitHub Pages:**

1. In this repository, go to **Settings → Pages**.
2. Under "Build and deployment," set **Source** to "Deploy from a branch."
3. Set **Branch** to `main` and folder to `/ (root)`.
4. Save — GitHub publishes the site within a minute or two. If this repo is private, note that private-repo Pages requires a paid GitHub plan; otherwise drop `index.html` into any other static host (Netlify, Vercel, Cloudflare Pages, an S3 bucket) — it works the same way anywhere.

**3. Host it anywhere else.** Since it's one static HTML file, it can be dropped into any static web host, an internal file share, or emailed as an attachment and opened locally.

## Book of business bulk check

The "Book of business — bulk CSV check" card near the top of the page lets you screen a whole client list at once instead of one client at a time:

1. **Choose CSV file** — export your client list from your CRM (e.g. Spark Advisors) or any spreadsheet as a CSV and select it. Nothing leaves the browser tab; there's no upload to a server.
2. **Match columns** — the tool reads your file's header row and guesses which column is which (name, state, date of birth, marital status, income, resources, current coverage type, and months without Part B/Part D coverage). Fix any guess that's wrong, and leave a field as "Not in this file" if your export doesn't have it — checks that need that data are simply skipped for those rows rather than assumed to fail. A default state applies to any row where the state column is missing or unrecognized.
3. **Run check** — every row is scored with the exact same math as the single-client form below (same `R` and `MEDIGAP` constants, same MSP/Medicaid/SSI tier logic), so results never drift out of sync with a one-off screen.
4. **Review flagged clients** — the results table shows, per client: whether they're in (or approaching) their Oklahoma Medigap birthday-rule window, Extra Help/MSP/state-Medicaid/SSI status, and any late-enrollment penalty math from months-without-coverage columns. "Show only clients with an action flag" is on by default so a book of a few hundred collapses to the handful worth a call.
5. **Download results as CSV** to save or share the flagged list.

Notes on this feature:
- **Date of birth drives the birthday-rule check.** An age-only column isn't enough to compute an actual window (it needs the month and day), so rows without a parseable DOB show "Add DOB to check" instead of a guess.
- **Income and resources are optional per row.** Most CRM exports (Spark Advisors included) don't carry income data — rows missing it show "Needs income" for the four income-based programs rather than a false negative. If you keep income/resources in a separate file, the simplest path is adding those as extra columns to your CRM export before uploading.
- **The birthday-rule flag assumes the client is currently on a Medigap plan** unless your file's coverage-type column says otherwise (the birthday rule only grants a guaranteed-issue right to switch Medigap-to-Medigap, not from Medicare Advantage).
- Accepts standard comma-separated CSV with quoted fields; date of birth can be `MM/DD/YYYY` or any format `Date.parse` understands (e.g. `YYYY-MM-DD`).

## How it's organized

```
medicare-workbench/
├── index.html      # the entire tool — markup, styles, and logic in one file
├── README.md       # this file
└── CHANGELOG.md    # dated history of substantive changes
```

Everything lives in `index.html`: a `<style>` block (respects the visitor's light/dark OS setting), the form markup, and a single `<script>` block with the constants (`R`, `MEDIGAP`, `SIGNUP`, `IRMAA`), the embedded CMS Landscape File plan data (`PLAN_DATA`), the cost/eligibility/underwriting calculation functions, the bulk CSV check, and the rendering code that keeps the results live as fields change.

There is no framework, bundler, or package manager — plain HTML/CSS/JavaScript, intentionally, so it stays this easy to host and maintain.

## Updating the tool each year

Medicare's dollar thresholds change every fall for the following plan year. All of them live in one place — the `R` constants object near the top of the `<script>` block in `index.html`:

```js
var R = {
  year: 2026,
  partB: { premium: 202.90, deductible: 283 },
  partA: { deductible: 1736 },
  partD: { deductible: 615, moopCap: 2100, nbbp: 38.99 }, // nbbp drives the Part D late-enrollment penalty calc
  lis: { incI: 23940, incM: 32460, resI: 18090, resM: 36100 },
  msp: {
    qmb:  { incI: 1350, incM: 1824 },
    slmb: { incI: 1616, incM: 2184 },
    qi:   { incI: 1816, incM: 2455 },
    resI: 9950, resM: 14910
  },
  ssi: { fbrI: 994, fbrM: 1491, resI: 2000, resM: 3000, genExclusion: 20 },
  medicaidABD: {
    OK: { name: "SoonerCare (Oklahoma Medicaid)", incI: 1350, incM: 1824, resI: 9950, resM: 14910 },
    TX: { name: "Texas Medicaid", incI: 994, incM: 1491, resI: 2000, resM: 3000 }
  }
};
```

Each fall (figures are usually announced by CMS/SSA in September–November for the coming plan year):

1. Update `year` and every dollar figure in `R` against the current sources listed in the in-app "2026 program thresholds" panel — primarily CMS.gov, Medicare.gov, and SSA.gov.
2. Update the `IRMAA` constants object (just below `R`) against CMS's annual Part B/Part D premium fact sheet.
3. Replace the embedded `PLAN_DATA` blob with the new plan year's CMS Landscape File release once it's out (this is the largest annual update — the file is refreshed monthly by CMS during AEP, so it's worth re-pulling more than once each fall).
4. Update the source links inside the `<details class="thresholds">` block if any of them change URLs.
5. Spot-check the `MEDIGAP` object (Oklahoma birthday rule, Texas's lack of one, under-65 access) — state law changes far less often than federal dollar amounts, but it's worth a periodic recheck.
6. Test a few known scenarios in a browser before relying on it with a real client — see [Testing](#testing-changes) below.
7. Bump the date note in [CHANGELOG.md](CHANGELOG.md) and republish.

## Testing changes

There's no automated test suite (it's a static form-driven tool), so after any change to `index.html`, manually verify in a browser:

- The page loads with no console errors, in both Oklahoma and Texas.
- The example client data produces sensible cost, eligibility, and underwriting results.
- At least one deliberately borderline income scenario for the Medicare Savings Program and state Medicaid checks.
- The IRMAA lookup and late-enrollment penalty check at a few known values against their own worked examples.
- The plan lookup returns results for at least one county in each state, and "→ B" / "→ C" correctly fills Option B/C.
- **Book of business CSV check:** upload a small test CSV with a mix of complete and partial rows (missing DOB, missing income, an out-of-window Oklahoma birthday, an in-window one, a Texas row) and confirm the auto-detected column mapping is reasonable, results match what you'd expect from running each row through the single-client form by hand, and "Download results as CSV" produces a readable file.
- No horizontal scroll or console errors at a phone-width viewport (~390px).

## Limitations and disclaimers

- **This is a screening tool, not a determination.** Extra Help, Medicare Savings Program, Medicaid, and SSI results are simplified estimates — SSA and the relevant state Medicaid office make the actual determination.
- **Not covered:** QDWI (a narrow Part-A-only MSP for people under 65), and non-Medicare programs like SNAP or LIHEAP.
- **Underwriting guidance is a simplification** of Oklahoma and Texas Medigap rules plus federal guaranteed-issue triggers — always confirm current carrier and state rules before telling a client they can or can't switch without underwriting.
- **Plan lookup data is a snapshot.** The embedded CMS Landscape File data is only as current as the last time it was refreshed in this file (see the date noted in the "CMS Landscape plan lookup" panel) — always confirm final numbers in Sunfire or CMS Plan Finder before quoting.
- **The late-enrollment penalty check is informational only** — it does not automatically add to the cost comparison, to avoid double-counting a penalty already reflected in an entered premium.
- **The IRMAA lookup assumes the broker has correctly determined MAGI and filing status.**
- **The book of business CSV check is a bulk screen, not a bulk determination** — it applies the same simplified math as the single-client form to every row, so the same caveats apply per client, multiplied by the size of the file. Treat the flagged list as a call list to verify, not a finished eligibility determination.
- **Nothing here replaces Scope of Appointment / TPMO compliance requirements** before an enrollment conversation.
- All figures are for **plan year 2026** as of the last update — see [CHANGELOG.md](CHANGELOG.md) for sourcing and audit history.

## License / usage

This tool was built for D&D Insurance Group's internal use. There's no license file because it isn't intended for public redistribution — treat the repository as private/internal unless that changes.
