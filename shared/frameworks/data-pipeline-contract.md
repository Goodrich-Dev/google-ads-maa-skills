# Data Pipeline Contract — Google Ads MAA

This is the agreement between the data-collection Google Ads Script (running
inside a client's Google Ads account) and the `google-ads-analyzer` skill
(running inside Claude). The script writes data into a weekly email; the
analyzer reads it. A stable contract keeps the analyzer from breaking every
time the script changes.

The `google-ads-script` skill in this package generates the collection script.
This document is the canonical reference for what data appears, in what format,
between which markers.

## Email envelope (default convention — configurable)

- **From:** Google Ads Scripts (noreply)
- **To:** the inbox you designate for automation data (set in your script)
- **Subject:** `[MAA Data] {Account Name} — YYYY-MM-DD`
- **Label/filter:** `maa-data-automation` (any label works; the analyzer just
  needs to find the latest email for a given client)
- **Cadence:** weekly
- **Account-name source:** `AdsApp.currentAccount().getName()` — auto-detected,
  no manual config.

## The 10 markers

Each dataset is delimited by paired BEGIN/END markers. The order in the email
body matches the order below.

| # | Marker (begin) | Contents | Columns |
|---|---|---|---|
| 1 | `---BEGIN CAMPAIGN SUMMARY 30D---` | Campaign rollup, 30 days | Campaign, Status, Type, Bid Strategy, Cost, Impressions, Clicks, CTR, Avg CPC, Conversions, Cost/Conv, Search IS, Search IS Lost (Budget), Search IS Lost (Rank) |
| 2 | `---BEGIN CAMPAIGN SUMMARY 7D---` | Campaign rollup, 7 days | (same 14 columns) |
| 3 | `---BEGIN CONVERSION DETAIL 30D---` | Conversions by action name, per campaign, 30 days. Only rows where conversions > 0. | Campaign, Conversion Action, Conversions |
| 4 | `---BEGIN CONVERSION DETAIL 7D---` | Same, 7 days | (same 3 columns) |
| 5 | `---BEGIN KEYWORD REPORT 30D---` | Keyword data, enabled entities only, 30 days | Campaign, Ad Group, Keyword, Match Type, Quality Score, Expected CTR, Landing Page Exp, Ad Relevance, Cost, Impressions, Clicks, CTR, Avg CPC, Conversions, Cost/Conv |
| 6 | `---BEGIN KEYWORD REPORT 7D---` | Same, 7 days | (same 15 columns) |
| 7 | `---BEGIN SEARCH TERMS 30D---` | Search terms, top 100 by cost, 30 days | Campaign, Ad Group, Search Term, Cost, Impressions, Clicks, CTR, Conversions |
| 8 | `---BEGIN SEARCH TERMS 7D---` | Same, 7 days | (same 8 columns) |
| 9 | `---BEGIN AD REPORT 30D---` | Ad data incl. RSA headlines/descriptions (pipe-separated), ad strength, 30 days | Campaign, Ad Group, Ad Type, Headlines, Descriptions, Ad Strength, Impressions, Clicks, CTR, Cost, Conversions, Cost/Conv |
| 10 | `---BEGIN AD REPORT 7D---` | Same, 7 days | (same 12 columns) |

## Parsing notes

- CSV is embedded as text in the email body, not as attachments. Parse between
  marker pairs.
- Quoted fields: campaign/ad-group/keyword/search-term text is double-quoted to
  handle commas in names.
- RSA assets: headlines and descriptions are pipe-separated. Order is the API's
  return order, not the pin order.
- Bid strategy names: the script emits the API enum (e.g. `MAXIMIZE_CONVERSIONS`).
  The analyzer translates to the UI-facing name before any client-facing output.
- Null safety: quality fields emit `N/A` when the API returns null; non-RSA ad
  types emit empty strings for Headlines/Descriptions.
- Conversion Detail emits zero rows when there are no conversions in the window.
  Do not treat an empty section as a parse error.

## Known issue — Quality Score column inversion

Some deployed versions of the collection script have a column-order bug in the
keyword query: the `Expected CTR` and `Ad Relevance` columns are swapped in the
CSV (Landing Page Experience is correct). When parsing a keyword report from an
affected script version:

```
ad_relevance_value  = row["Expected CTR"]     # mislabeled in CSV
expected_ctr_value  = row["Ad Relevance"]     # mislabeled in CSV
lp_experience_value = row["Landing Page Exp"] # correct as labeled
```

Apply the swap before using these values in analysis or dispatch decisions.
Retire the swap once a corrected script is deployed to every account, or gate it
on an email version marker — otherwise it re-inverts correctly-labeled data.
