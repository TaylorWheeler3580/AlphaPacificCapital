# Alpha Pacific Capital — Media Performance Dashboard

Single-file dashboard for three Las Vegas apartment communities, built from the
pulseMax report covering **19 August – 8 October 2026**.

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

`index.html` carries the Meta lead roster: 152 names, email addresses and phone
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

## The lead workbook has four different column layouts

`scripts/parse_leads.py` reads columns A–E across three sheets. The layout varies
both between sheets and within one sheet:

| Sheet | Layout | Rows |
|---|---|---|
| Onyx | Date / First / Last / Email / Phone | 36 |
| Onyx | Date / fb\|ig / First / Last / Email / Phone | 107 |
| Camino Al Norte | Date / First / Last / Email / Phone | 4 |
| Camino Al Norte | Date / fb\|ig / First / Last / Email — **no phone column** | 3 |
| Elysian at St. Rose | Date / First / Last / Email / Phone | 2 |

Reading by position puts `fb` in the first-name column, and on the Camino form
that has no phone column it prints "Just me" as a phone number. So the parser
finds the email cell, reads the names back from it, and accepts the following
cell as a phone only if it parses as one. One Onyx number was stored by Excel as
a float (`7024703217.0`), where a naive digit strip turns the trailing `.0` into
an eleventh digit — numeric cells are coerced to int first.

Sheet names are matched by regex, not exact title: "Camino" became "Camino Al
Norte" between exports.

A shape check prints on every run — malformed rows must be zero.

The platform column is captured where present (74 Facebook, 36 Instagram) but not
displayed, since only A–E were asked for.

**The export and the report do not cover the same window**, and the gap differs
by property:

| | Export | Meta reports | |
|---|---|---|---|
| The Onyx | 143 (Aug 25 – Sep 30) | 155 | 12 missing |
| Camino Al Norte | 7 (Oct 8 – 9) | 5 | 2 extra, both dated Oct 9 |
| Elysian at St. Rose | 2 (Oct 8) | 4 | 2 missing |

October reconciles exactly at account level: 9 in the export, 9 in the deck. The
Leads panel shows both figures side by side wherever the deck reports at that
scope, and offers no comparison for a single property in a single month, because
the deck does not report that. Seven Onyx rows repeat an email address, so 152
submissions are 145 distinct people.

## Data notes

**"Search & Intent Media" is skipped.** It repeats the SEM section verbatim —
identical impressions, clicks and conversions — so counting it would double SEM.

## Periods

The deck carries a **Monthly Performance** table for each tactic, and those tables
are the only month split in the report. They are account level: no tactic breaks a
month out by property. They reconcile exactly to the flight totals —
Display 40,911 + 120,533 + 8,356 = 169,800; Social 10,736 + 34,135 + 4,636 = 49,507;
SEM 8,926 + 20,378 + 1,154 = 30,458.

So on **All three** the period pills are exact, and on a property tab they are not:
Delivery and Tactics fall back to the full flight and the page says so. The one
exception is the Leads panel, where dates come from the Meta export and the month
really is that property's.

**Meta used to be the exception and no longer is.** Until 5 October it ran a single
campaign, for The Onyx, so its monthly figures were that property's in full. Camino
Al Norte launched 5 October and Elysian 6 October, so the monthly Social table now
mixes all three and that shortcut has been removed from the template.

The months are 13, 30 and 8 days long, so the totals are not comparable on their
own. The impressions tile shows a per-day rate and a per-day change against the
month before.

**September was restated.** The previous deck, cut at 29 September, reported
116,509 Display and 33,292 Social impressions for the month; this one reports
120,533 and 34,135. One extra day plus pulseMax finalising the numbers.

Everything below Tactics — campaigns, creative, ad groups, keywords, fences, zips,
geography — is flight level in the source and carries a **Full flight** chip when a
month is selected.

## October pacing

Every tactic fell sharply in the first eight days of October, on a per-day basis:

| Tactic | Sep / day | Oct / day | Change |
|---|---|---|---|
| Display | 4,018 | 1,045 | −74% |
| Social | 1,138 | 580 | −49% |
| SEM | 679 | 144 | −79% |

Search also recorded no conversions at all in October, against 13 in September.
Eight days is a short window and the last days of a pulseMax pull can still be
settling, so this is flagged in the dashboard rather than interpreted. Worth
checking budget pacing before reading it as a performance drop.

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
