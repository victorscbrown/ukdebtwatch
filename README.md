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
| `uk-national-debt-by-year/index.html` | `/uk-national-debt-by-year/` | Debt by year chart and table |
| `help/index.html` | `/help/` | Mortgage arrears & repossession help (linked from social videos) |

Also tracked, and **required in every deploy**:

- `og-image.jpg` — social share card
- `google244b72129ae33294.html` — Google Search Console verification
- `robots.txt`, `sitemap.xml`

These last two were previously easy to lose in a manual deploy. Keeping them in
git is the main reason this repo exists.

## The monthly update

The ONS publishes *Public sector finances* around the 21st of each month — but
the exact day moves (21 Aug, 22 Sep, 21 Oct, 20 Nov 2026), so do not rely on a
fixed date.

Each release, update the constants block at the top of the `<script>` in **every
page that carries it**. Do not work from a remembered list of files — pages get
added, and a page left on an old baseline contradicts the rest of the site:

```sh
grep -rln "BASELINE_DEBT" --include=*.html .
```

As of October 2026 that is `index.html`, `your-share/index.html`,
`3-trillion/index.html` and `uk-national-debt-by-year/index.html`. They must
stay byte-identical — every figure on every page derives from them.

```js
const BASELINE_DATE = new Date('2026-09-30T00:00:00Z');
const BASELINE_DEBT = 3_005_000_000_000;   // PSND ex banks, latest ONS print
const PER_SECOND    = 2_907;               // 12-month rise / 31,536,000
const UK_POP        = 68_350_000;          // ONS mid-year estimate
const GDP           = 3_196_800_000_000;   // implied by the ONS debt/GDP ratio
```

Deriving each one:

- **BASELINE_DEBT** — public sector net debt excluding public sector banks, from the
  latest [ONS bulletin](https://www.ons.gov.uk/economy/governmentpublicsectorandtaxes/publicsectorfinance/bulletins/publicsectorfinances/latest).
- **BASELINE_DATE** — the last day of the month that bulletin covers.
- **PER_SECOND** — `(the twelve-month rise in net debt) ÷ 31,536,000`, using the
  year-on-year figure the bulletin states ("£X billion more than a year
  earlier"). **Not** the one-month change. Single months are far too volatile
  and the ONS revises them heavily: July 2026 was revised down £2.9bn and
  August by a further £4.4bn, which would have made a one-month rate absurd.
  This constant also drives the calculator's pension-age projection, so a noisy
  rate does visible damage.
- **GDP** — `new debt ÷ (ONS debt-to-GDP percentage ÷ 100)`.

Also refresh the static tiles that quote a specific month: borrowing financial year
to date, and monthly debt interest.

Then grep for the **previous** month's figures as literal strings (debt, percentage,
per-second rate, interest, borrowing, and the month's name) across every HTML file and
update each one. Stale duplicates in prose are the usual failure mode — these numbers
are repeated in page copy, meta descriptions, Open Graph and Twitter tags, FAQ answers,
methodology blocks, JSON-LD, tickers and screen-reader summaries. Update `dateModified`
in any JSON-LD and `<lastmod>` in `sitemap.xml`, but only for pages this release
actually changed.

`/3-trillion/` passed its milestone in September 2026, confirmed by the ONS on
21 October 2026, and its banner is now in the green confirmed state. The ONS
publishes a monthly snapshot rather than a running total, so there is no official
crossing date *inside* September — the page states the month and must not assert a day.

If a release contradicts something the site asserts, make the pages accurate to what
the ONS actually published rather than forcing the update to fit. That happened twice
running: the September release was expected to confirm £3 trillion and did not, and the
October release confirmed it a month earlier than the page had projected.

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

## Shared header, footer and cookie consent (added 11 Nov 2026)

Every page carries the same site menu, footer (About · Contact · Privacy · Sources · Articles · Help · Cookie settings) and cookie banner. They are generated by `_tools/chrome.py`: run `python3 _tools/chrome.py` from the repo root after adding a page or changing the menu. It is safe to re-run; it replaces the previous chrome.

- Google Analytics loads **only after the visitor clicks Accept** (the `udw-ga` script in each page's head). Do not paste the standard GA snippet into new pages; run the script instead.
- The homepage's `id="header-right"` status line lives inside the new header and is still filled in by the homepage JavaScript from `BASELINE_DATE` and `NEXT_RELEASE`.
- New pages (for example monthly reports) should start from an existing page so they inherit the chrome, then run the script.
- Minimum text size is 12px everywhere.
