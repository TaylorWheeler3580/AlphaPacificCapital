# Alpha Pacific Capital — Media Performance Dashboard

Single-file dashboard for three Las Vegas apartment communities, built from the
pulseMax report covering **19 August – 9 September 2026**.

- **Camino Al Norte** — caminoalnorteapts.com
- **The Onyx** — theonyxapartments.com
- **Elysian at St. Rose** — elysianatstrose.com

Open `index.html` in a browser. No build step, no server, no dependencies.

## Rebuilding

```bash
pip install python-pptx
python3 scripts/parse.py           # deck -> data/model.json
python3 scripts/parse_keywords.py  # Google Ads exports -> data/keywords.json
python3 scripts/build.py    # template + data -> index.html
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
| SEM | ad group name — `..._Camino Al Norte Apts_Townhome rentals` |

Anything without a property in its name stays at account level and is labelled as
such, rather than split by guesswork.

## Data notes

**"Search & Intent Media" is skipped.** It repeats the SEM section verbatim —
identical impressions, clicks and conversions — so counting it would double SEM.

**860 of 14,300 search impressions are unattributed.** Ad groups cover 13,440;
the rest ran in campaigns the report doesn't break out by ad group. Flagged in the
Search panel rather than silently absorbed.

**Keywords come from Google Ads exports, not the deck.** `scripts/parse_keywords.py`
reads the three tab-separated exports in `source-reports/keyword-exports/` and attributes
every keyword by its ad group name, which carries the property. Column sets differ between
files (the Onyx export has an extra Currency code column), so fields are looked up by
header name rather than position.

**The export window is labelled 13 Aug – 9 Sep**, but the campaign did not launch until
19 August, so nothing served in the earlier days. The period is deliberately not carried
into the dashboard — surfacing it would imply a discrepancy that doesn't exist.

**Quality Score is parsed but not shown.** Too many keywords have none for the column to
be worth its width.

**Cost is parsed but not displayed**, matching the Platte Valley dashboard. Restoring it
is a template change only.

**Geography and target fences are account level.** The report doesn't break them out
by property, so those panels show the same figures on every tab and say so.

**Call tracking is parsed but not displayed.** `scripts/parse.py` still extracts the call
log, recordings and platform summaries into `data/model.json`; the panel was removed at
the client's request. Restoring it is a template change only, no re-parse needed.

**No foot-traffic visits yet.** Geofencing attribution usually needs several weeks
of delivery before visits register, and this campaign launched 19 August.

**Age breakdown is deliberately excluded.** Housing is a Fair Housing sensitive
category, so the Meta age table is parsed out rather than displayed. Gender is still
shown — say the word if that should come out too.

**Meta is Onyx only.** The other two properties show no Social row because none ran,
not because data is missing.
