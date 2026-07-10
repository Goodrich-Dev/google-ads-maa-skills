# Google Ads GAQL Query Pack (MCP data pull)

> The reusable queries that reproduce every MAA dataset live from a **Google Ads
> MCP** connector (tool: `search`, GAQL). This is the **primary** data source;
> the `maa-data-automation` email is the fallback. See
> `data-pipeline-contract.md` for the source-priority rules.

## How to call

The MCP `search` tool takes: `customer_id` (CID, digits only, no dashes),
`resource`, `fields` (list), `conditions` (list, AND-combined), optional
`orderings`, `limit`. Record each client's CID once in your per-client config
(see CONFIGURATION.md) so runs don't start with a lookup.

If your MCP server sits under an MCC (manager account), the login customer id is
typically baked into the server config; pass the child account's CID as
`customer_id`.

**The MCP is read-only.** The `search` tool is GAQL reporting only. It cannot
mutate the account — negatives, pauses, bid and budget changes all go through
Google Ads change scripts (`google-ads-change-scripts`), never the MCP. Reading
current state to target a change (e.g. confirming a campaign's bid strategy
before writing the switch script) is fine and encouraged.

## Date windows

- **7D** = the 7 days ending yesterday. **30D** = the 30 days ending yesterday.
- Pass as a condition: `segments.date BETWEEN '{START}' AND '{END}'`
  (YYYY-MM-DD, both required). Run each dataset twice (30D and 7D) where the MAA
  needs both.

## Unit conversions (apply after pulling)

- **Money:** `metrics.cost_micros` and `metrics.cost_per_conversion` are
  **micros** → divide by 1,000,000 for dollars.
- **Rates:** `metrics.ctr` and the impression-share metrics are **fractions
  0–1** → ×100 for a percent.
- **Impression share** metrics exist only on Search campaigns; PMax/LSA return
  them empty.

---

## THE QS COMPONENT MAPPING (why MCP-first kills the swap)

The API fields are correctly named at the source — no inversion, no corrector:

| API field | = MAA component |
|---|---|
| `ad_group_criterion.quality_info.search_predicted_ctr` | **Expected CTR** |
| `ad_group_criterion.quality_info.creative_quality_score` | **Ad Relevance** |
| `ad_group_criterion.quality_info.post_click_quality_score` | **Landing Page Experience** |
| `ad_group_criterion.quality_info.quality_score` | Quality Score (1–10) |

**Read these AS-IS. The Expected-CTR ↔ Ad-Relevance swap and
`correct_qs_columns.py` apply ONLY to the email-fallback branch, never to MCP
data.**

---

## 1. Campaign Summary (run 30D and 7D)

```
resource:   campaign
fields:     campaign.name, campaign.status, campaign.advertising_channel_type,
            campaign.bidding_strategy_type,
            metrics.cost_micros, metrics.impressions, metrics.clicks, metrics.ctr,
            metrics.average_cpc, metrics.conversions, metrics.cost_per_conversion,
            metrics.search_impression_share,
            metrics.search_budget_lost_impression_share,
            metrics.search_rank_lost_impression_share
conditions: segments.date BETWEEN '{START}' AND '{END}'
            metrics.impressions > 0
```

Omit dormant campaigns ($0 in both windows). `advertising_channel_type` flags
SEARCH vs PERFORMANCE_MAX vs LOCAL_SERVICES — note that LSA campaigns surface on
this path even though the email script never carried them; LSA is out of scope
for the MAA (see the data-pipeline contract).

## 2. Keyword Report + Quality Score (run 30D and 7D)

```
resource:   keyword_view
fields:     campaign.name, ad_group.name,
            ad_group_criterion.keyword.text, ad_group_criterion.keyword.match_type,
            ad_group_criterion.quality_info.quality_score,
            ad_group_criterion.quality_info.search_predicted_ctr,
            ad_group_criterion.quality_info.creative_quality_score,
            ad_group_criterion.quality_info.post_click_quality_score,
            metrics.cost_micros, metrics.impressions, metrics.clicks, metrics.ctr,
            metrics.average_cpc, metrics.conversions, metrics.cost_per_conversion
conditions: segments.date BETWEEN '{START}' AND '{END}'
            metrics.impressions > 0
orderings:  metrics.cost_micros DESC
```

QS components are current-state (same in both windows); metrics are windowed.
Read QS per the mapping above (no swap).

## 3. Search Terms (run 30D and 7D)

```
resource:   search_term_view
fields:     search_term_view.search_term, campaign.name, ad_group.name,
            metrics.cost_micros, metrics.impressions, metrics.clicks,
            metrics.conversions, metrics.cost_per_conversion
conditions: segments.date BETWEEN '{START}' AND '{END}'
            metrics.impressions > 0
orderings:  metrics.cost_micros DESC
```

Feeds negative-keyword mining and the n-gram waste read.

## 4. Ad Report — RSA assets + strength (run 30D and 7D)

```
resource:   ad_group_ad
fields:     campaign.name, ad_group.name, ad_group_ad.ad.type, ad_group_ad.ad_strength,
            ad_group_ad.ad.responsive_search_ad.headlines,
            ad_group_ad.ad.responsive_search_ad.descriptions,
            metrics.impressions, metrics.clicks, metrics.ctr, metrics.cost_micros,
            metrics.conversions
conditions: segments.date BETWEEN '{START}' AND '{END}'
            ad_group_ad.status != 'REMOVED'
```

`headlines`/`descriptions` are lists of asset objects (pull the `.text`).

## 5. Conversion Detail — by action name (run 30D and 7D)

```
resource:   campaign
fields:     campaign.name, segments.conversion_action_name, metrics.conversions,
            metrics.conversions_value
conditions: segments.date BETWEEN '{START}' AND '{END}'
            metrics.conversions > 0
```

Breaks conversions out by action (forms vs calls vs booked) for the lead-quality
read.

---

## Mid-cycle pulls: answer your own questions

The MCP makes additional pulls cheap. When the analysis raises a question the
initial datasets can't answer — "is there real demand for long-term rentals?",
"which device is driving the CPA spike?" — pull the extra data inside the same
cycle and turn the answer into a grounded action, instead of writing "keep
watching" and deferring it a week. This applies to Google Ads pulls and to any
other connected data source (keyword research tools, page speed tests, analytics).

## Notes

- If a field name errors, confirm it with the MCP's resource-metadata tool for
  that resource rather than guessing.
- Large hierarchy queries (e.g. `customer_client` across an MCC) can exceed the
  tool's output cap — query a specific `customer_id` and filter/limit.
