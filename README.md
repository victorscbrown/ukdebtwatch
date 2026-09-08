# UK Debt Watch

Live UK national debt clock — [ukdebtwatch.co.uk](https://ukdebtwatch.co.uk)

Zero-dependency static HTML. No build step, no framework, no package manager.
Netlify publishes the repository root exactly as it is.

## Pages

| File | URL | What it is |
|---|---|---|
| `index.html` | `/` | Live debt counter, FAQ, methodology |
| `your-share/index.html` | `/your-share/` | UK debt calculator — personal share, tax comparison |
| `3-trillion/index.html` | `/3-trillion/` | The £3 trillion milestone |

Also tracked, and **required in every deploy**:

- `og-image.jpg` — social share card
- `google244b72129ae33294.html` — Google Search Console verification
- `robots.txt`, `sitemap.xml`

These last two were previously easy to lose in a manual deploy. Keeping them in
git is the main reason this repo exists.

## The monthly update

The ONS publishes *Public sector finances* around the 21st of each month.
Each release, update the constants block at the top of the `<script>` in
**all three** HTML files. They must stay in sync — every figure on every page
derives from them, so the pages can never contradict each other.

```js
const BASELINE_DATE = new Date('2026-07-31T00:00:00Z');
const BASELINE_DEBT = 2_984_900_000_000;   // PSND ex banks, latest ONS print
const PER_SECOND    = 7_010;               // observed accrual since previous print
const UK_POP        = 68_350_000;          // ONS mid-year estimate
const GDP           = 3_172_000_000_000;   // implied by the ONS debt/GDP ratio
```

Deriving each one:

- **BASELINE_DEBT** — public sector net debt excluding public sector banks, from the
  latest [ONS bulletin](https://www.ons.gov.uk/economy/governmentpublicsectorandtaxes/publicsectorfinance/bulletins/publicsectorfinances/latest).
- **BASELINE_DATE** — the last day of the month that bulletin covers.
- **PER_SECOND** — `(new debt − previous debt) ÷ seconds between the two dates`.
- **GDP** — `new debt ÷ (ONS debt-to-GDP percentage ÷ 100)`.

Also refresh the static tiles that quote a specific month: borrowing financial year
to date, and monthly debt interest. On `/3-trillion/`, the amber ESTIMATE banner has
its confirmed replacement wording in an HTML comment directly above it.

Tax bands in `your-share/index.html` are separate and change each April, not monthly.

## Deploying

Push to the default branch. Netlify builds from git and publishes automatically.

## Analytics

Google Analytics 4 (`G-0KZHW8KQM0`), loaded on all three pages.

Custom events: `affiliate_impression`, `affiliate_click` (with `variant` and
`placement`), `calculator_used`, `calculator_share`, `30s_dwell`.

Each page has a `track(name, props)` helper that fires to `gtag` if present and
to `plausible` if present, so swapping analytics providers is a change to the
`<head>` tag only — the event calls stay as they are.

GA4 sets cookies, so a consent banner is required for UK visitors. There isn't
one yet.

## Affiliate links

Only live, approved partners belong in `AFF_VARIANTS` in `index.html`. Pending
partners sit in a comment block below it with `REPLACE_ME` references — a
placeholder that reaches production is a dead link on every impression it serves.
