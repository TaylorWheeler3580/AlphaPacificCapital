# Alpha Pacific Capital — Media Performance Dashboard

Single-file dashboard for three Las Vegas apartment communities, built from the
pulseMax report covering **19 August – 29 September 2026**.

- **Camino Al Norte** — caminoalnorteapts.com
- **The Onyx** — theonyxapartments.com
- **Elysian at St. Rose** — elysianatstrose.com

Open `index.html` in a browser. No build step, no server, no dependencies.

## Rebuilding

```bash
pip install python-pptx
python3 scripts/parse.py           # deck -> data/model.json
python3 scripts/parse_keywords.py  # Google Ads exports -> data/keywords.json
python3 scripts/parse_leads.py     # Meta lead workbook -> data/leads.json
python3 scripts/build.py           # template + data -> index.html
python3 scripts/build.py --public  # same page, roster removed -> index-public.html
```

Drop a newer deck in `source-reports/` and re-run. Edit `scripts/template.html`,
never `index.html` — the latter is generated.

## How properties are separated

The report is organised by channel, not by property. Property names appear inside
campaign, ad-group and creative strings, so attribution matches on those:

| Tactic | Attribution source |
|---|---|
| Display (geofencing) | campaign name — `..._The Onyx Apartments_GFT` |
| Display creative | file name — `The Onyx CR31328.gif` |
| Social (Meta) | campaign name — Onyx only; the other two aren't running Meta |
| SEM | campaign name — `..._Elysian at St. Rose_SEM-SEARCH` |

Anything without a property in its name stays at account level and is labelled as
such, rather than split by guesswork.

## Two builds, and why

`index.html` carries the Meta lead roster: 143 names, email addresses and phone
numbers belonging to people who enquired about an apartment. It is the working
copy for the leasing team.

`scripts/build.py --public` writes `index-public.html`, identical but with every lead
row dropped — the Leads panel keeps its counts and says the roster was held back.
Use that for anything shared outside the leasing team, and for GitHub Pages.

Two further things are stripped from **both** builds by `build.py`:

- the CallRail log — caller names, phone numbers and recording URLs, parsed into
  `data/model.json` but never inlined, since the call panel was removed
- nothing else; everything else on the page is aggregate

`data/model.json` and `data/leads.json` are both gitignored. If the repository is
public, only `index-public.html` belongs in it.

## The lead workbook is two exports stacked

`scripts/parse_leads.py` reads columns A–E. 36 rows are `Date / First / Last /
Email / Phone`; the other 107 carry an extra platform column at B that shifts
everything one to the right. Reading by position puts `fb` in the first-name
column and slides the phone number out of the table, so the parser finds the
email cell and reads the rest relative to it. It prints a shape check on every
run — that count must be zero.

The platform column is captured where present (71 Facebook, 36 Instagram) but
not displayed, since only A–E were asked for.

**The export and the report do not cover the same window.** The export runs
25 August – 30 September; the report runs 19 August – 29 September. Meta reports
153 leads for the flight where the export lists 143, and the export includes
three leads dated 30 September, after the report's cut. The Leads panel shows
both figures side by side and says so rather than picking one. Seven rows repeat
an email address, so 143 submissions are 136 distinct people.

## Data notes

**"Search & Intent Media" is skipped.** It repeats the SEM section verbatim —
identical impressions, clicks and conversions — so counting it would double SEM.

## Periods

The deck carries a **Monthly Performance** table for each tactic, and those tables
are the only month split in the report. They are account level: no tactic breaks a
month out by property. They reconcile exactly to the flight totals —
Display 40,911 + 116,509 = 157,420, Social 10,736 + 33,292 = 44,028,
SEM 8,926 + 20,120 = 29,046.

The period pills therefore behave differently by tab, and the dashboard says which
case you are in:

| Tab | August / September |
|---|---|
| All three | exact — delivery and tactics come straight from the monthly tables |
| The Onyx | exact for Social only; Onyx runs the account's single Meta campaign |
| Camino Al Norte, Elysian | not reported; the page shows the full flight and says so |

August covers 19–31 (13 days) and September 1–29 (29 days), so the totals are not
comparable on their own. The impressions tile shows a per-day rate, and September
carries a per-day change against August.

Everything below Tactics — campaigns, creative, ad groups, keywords, fences, zips,
geography — is flight level in the source and carries a **Full flight** chip when a
month is selected.

## SEM is attributed at campaign level

Earlier builds summed the deck's **Ad Group Performance** table. That table prints
only the top five ad groups — 27,044 of 29,046 impressions — and understated every
property, Elysian worst of all (1 conversion instead of 5). The **Campaign
Performance** table has one campaign per property and reconciles exactly
(14,081 + 8,111 + 6,854 = 29,046; conversions 3 + 2 + 5 = 10), so that is the
attribution source. Ad groups are still parsed and shown as detail.

**Keywords come from Google Ads exports, not the deck.** `scripts/parse_keywords.py`
reads the three tab-separated exports in `source-reports/keyword-exports/` and attributes
every keyword by its ad group name, which carries the property. Column sets differ between
files (the Onyx export has an extra Currency code column), so fields are looked up by
header name rather than position.

**The keyword export is older than the deck.** The exports on file cover the opening
flight and account for 15,935 of the 29,046 search impressions in this report — about
55%. The Search panel's four headline figures come from the deck, and a banner states
the coverage. The banner is computed from the ratio, so it disappears on its own once a
refreshed export lands above 90%. No date is printed: the export window starts before
launch and printing it would imply a discrepancy that doesn't exist.

**Quality Score is parsed but not shown.** Too many keywords have none for the column to
be worth its width.

**Cost is parsed but not displayed**, matching the Platte Valley dashboard. Restoring it
is a template change only.

**Zip-code performance is new in this deck** and account level, like the rest of
geography. Only the top five zips are printed.

**Geography and target fences are account level.** The report doesn't break them out
by property, so those panels show the same figures on every tab and say so.

**Call tracking is parsed but not displayed.** `scripts/parse.py` still extracts the call
log, recordings and platform summaries into `data/model.json`; the panel was removed at
the client's request. Restoring it is a template change only, no re-parse needed.

**Foot traffic has started registering.** Six visits are now matched to the property
addresses — 3 Onyx, 2 Camino Al Norte, 1 Elysian — all view-through rather than
click-through, and all in September. Attribution lags delivery, so the figure keeps
moving after a report is cut.

**Every search conversion so far is a phone call.** The deck's conversion-action table
splits the ten into First Time Phone Call (6), Calls from ads (5) and Repeat Phone Call
(4); those overlap, which is why they sum past ten. No form fills or site actions have
been tracked against search.

**Age breakdown is deliberately excluded.** Housing is a Fair Housing sensitive
category, so the Meta age table is parsed out rather than displayed. Gender is still
shown — say the word if that should come out too.

**Meta is Onyx only.** The other two properties show no Social row because none ran,
not because data is missing.
