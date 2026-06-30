---
name: google-ads-analyzer
description: >
  Analyze Google Ads (Search, PMax, Video, Display) data using the MAA framework for local service businesses. Trigger when users want to review Google Ads performance, audit campaigns, analyze keywords/search terms, evaluate Quality Score, diagnose wasted spend, assess bid strategies, or get recommendations from Google Ads exports. Also trigger on uploaded Google Ads CSVs, PPC analysis requests, MAA coaching, or phrases like "analyze my Google Ads", "do MAA for [client]", "Google Ads audit", "review this campaign", "help me write an MAA". Google Ads only — not Meta or LSA.
---

# Google Ads Analyzer — MAA Framework for Local Service Businesses

## What This Skill Does

This skill analyzes Google Ads campaign data (Search, PMax, Video, Display) for
local service businesses and delivers a concise MAA (Metrics → Analysis →
Action). The output is a tight, actionable document — not a comprehensive audit
report. Think 2-3 action steps tied directly to what the data shows, starting
with the highest-impact foundational fixes before moving to optimization.

**Campaign type depth:** Search and PMax campaigns are the primary focus and get
full diagnostic treatment. Video and Display campaigns get a lighter touch —
condensed metrics and a brief note in Analysis — since they typically serve
awareness goals rather than direct lead generation.

The skill operates in two modes:

- **Analysis Mode**: Ingest data, diagnose, deliver a concise MAA
- **Coaching Mode**: Review a user-written MAA, teach transferable principles

## Core Principles

These principles are baked into every analysis. They come from Dennis Yu's
BlitzMetrics methodology.

### MAA: Metrics → Analysis → Action

Never jump to tactics before diagnosing. Every recommendation cites the metric
it moves and the analysis that justifies it. The MAA is a single cohesive
narrative, not three disconnected sections.

### Balanced Metrics

Never evaluate a metric in isolation. Metrics always travel in pairs:
Conversions ↔ CPA, Revenue ↔ ROAS, CTR ↔ CVR, Impression Share ↔ Marginal
CPA/ROAS, Volume ↔ Profit.

The classic trap: CPA drops from $50 to $15, everyone celebrates — but lead
volume collapsed from 80 to 12 because someone turned off everything except
brand. Always check the balancing metric.

### Top-N Focus

Sort by spend, focus on the top 5-10 items. These account for ~60% of total
spend. Don't analyze the long tail unless hunting for waste. Actionable insight
quickly, not a 20-page report.

**Omit dormant campaigns entirely.** Campaigns with $0 spend (or negligible
spend like <$1) in both the 7D and 30D windows do not appear anywhere in the
MAA — not in Metrics, not in Analysis, not in Action. Don't waste the reader's
attention on campaigns that aren't running. If every campaign in the account is
dormant, say so and skip straight to Action.

### "We" Tone

Write as an embedded team member. "We're seeing strong CTR on the emergency
plumbing keywords" — not "Your CTR is strong."

## Step 1: Quick Context

Before analyzing, get enough context to interpret the numbers. A $200 CPA might
be excellent for roofing and terrible for drain cleaning.

Ask only what's needed — adapt based on what you already know. Don't force all
five questions if the user is ready to go:

1. **Business & Goals**: What service type? Primary conversion goal (calls,
   forms, bookings)? Target CPA or monthly budget?
2. **Conversion Setup**: Where do conversions happen? Do you trust the tracking?
3. **Service Area & Season**: Geography? Peak or slow season?
4. **Benchmarks**: What does "good" look like for this account?
5. **Known Issues**: Anything already worrying you?

If the user says "just look at the data," work with what you have and flag
context gaps as they become relevant in the analysis.

## Step 1.5: Read the Narrative Document (if it exists)

Before analyzing data, check whether a narrative document exists for this
client. Narrative documents live in the client's output folder and follow the
naming pattern `{Client}_MAA-Narrative.md`. They contain:

- **Account Context**: Static info about the business, campaigns, targets
- **Client Contacts**: Name(s) and role(s) of the people who receive the MAA
  or act on its recommendations (e.g., "Sam — account owner, makes strategic
  decisions" or "Nicholas — manages the website, handles LP changes"). These
  names are used throughout the MAA when action items require their input.
- **Running Themes**: Persistent issues being tracked across weeks
- **Weekly Log**: Dated entries with what was shared with the client, actions
  taken, and client feedback

**If a narrative exists, read it before writing the MAA.** This is critical
for continuity. Use it to:

- **Follow up on open threads**: If last week flagged "watch QS on keyword X,"
  explicitly report on it this week. If an action was recommended (e.g., "add
  negatives for competitor terms"), look in this week's data for evidence of
  whether it was implemented.
- **Avoid repeating yourself**: Don't re-explain context the narrative already
  captures. Instead, reference it and build on it: "The DKI test we flagged
  two weeks ago is now showing QS movement on geo keywords."
- **Update running themes**: If a theme is resolved, note it. If a new pattern
  emerges, add it.

**If no narrative exists**, proceed normally — the MAA will stand on its own.
After delivering the MAA, create a new narrative document seeded with the
account context and first weekly log entry.

**If the user is running an ad-hoc analysis** (not a weekly cycle), still read
the narrative for context but don't update it — the user will update it with
whatever they actually share with the client.

## Step 2: Get the Data

### Email Pipeline Mode (Primary)

Data arrives automatically via Gmail from a Google Ads Script that runs weekly.
Search Gmail for label `maa-data-automation` or subject prefix `[MAA Data]` to
find the latest report for a given client.

Each email contains ten datasets embedded in the body between markers:
- `---BEGIN CAMPAIGN SUMMARY 30D---` / `---END CAMPAIGN SUMMARY 30D---`
- `---BEGIN CAMPAIGN SUMMARY 7D---` / `---END CAMPAIGN SUMMARY 7D---`
- `---BEGIN CONVERSION DETAIL 30D---` / `---END CONVERSION DETAIL 30D---`
- `---BEGIN CONVERSION DETAIL 7D---` / `---END CONVERSION DETAIL 7D---`
- `---BEGIN KEYWORD REPORT 30D---` / `---END KEYWORD REPORT 30D---`
- `---BEGIN KEYWORD REPORT 7D---` / `---END KEYWORD REPORT 7D---`
- `---BEGIN SEARCH TERMS 30D---` / `---END SEARCH TERMS 30D---`
- `---BEGIN SEARCH TERMS 7D---` / `---END SEARCH TERMS 7D---`
- `---BEGIN AD REPORT 30D---` / `---END AD REPORT 30D---`
- `---BEGIN AD REPORT 7D---` / `---END AD REPORT 7D---`

The **conversion detail** section breaks out conversions by action name per
campaign (e.g., "Phone Calls: 2, Lead Forms: 3"). Use this to populate the
conversion breakdown in the Metrics section's Conversions bullet. If this
section is missing (older script version), show the total conversion count
from the campaign summary without a breakdown.

The ad report contains: Campaign, Ad Group, Ad Type, Headlines (pipe-separated),
Descriptions (pipe-separated), Ad Strength, Impressions, Clicks, CTR, Cost,
Conversions, and Cost/Conv.

Parse the CSV data between these markers. The client name is in the email
subject: `[MAA Data] Client Name — YYYY-MM-DD`.

### Quality Score column inversion — MANDATORY swap on Keyword Report parse

The deployed Google Ads Script (`maa-data-script-updated.js` in every client's
account) has a known bug (discovered 2026-06-05): in `runKeywordQuery`, the GAQL
select order for the three QS components doesn't match the CSV header order, which
inverts the **Expected CTR** and **Ad Relevance** columns. Landing Page
Experience is in the middle position and is correct.

**What this means at parse time:** the column labeled `Expected CTR` actually
contains the **Ad Relevance** value, and the column labeled `Ad Relevance`
actually contains the **Expected CTR** value. Swap them before using them in any
analysis or dispatch decision.

```
For each row in KEYWORD REPORT 30D and KEYWORD REPORT 7D:
  ad_relevance_value   = row["Expected CTR"]     # mislabeled in CSV
  expected_ctr_value   = row["Ad Relevance"]      # mislabeled in CSV
  lp_experience_value  = row["Landing Page Exp"]  # correct as labeled
```

After the swap, use the *swapped* values in QS component callouts in Analysis,
any per-keyword QS narrative, and the dispatch triggers (Below Average Ad
Relevance vs. Expected CTR route to different copy fixes — getting the component
right is the whole point). The swap is **mandatory** for every Keyword Report
parse.

**Why this lives in the skill body, not just the framework:** this rule
previously lived only in `Data-Pipeline-Contract`, and a heavier analyzer prompt
(the v2 challenger) read the columns at face value and mislabeled a component on
a live client MAA. Restating it in the body makes it un-missable.

**When this rule will be retired:** if/when the corrected script is deployed to
every active client account, this swap becomes a bug (it would re-invert
correctly-labeled data) — remove it then, or gate it on an email version marker.
See `../../shared/frameworks/data-pipeline-contract.md` for the canonical column
mapping and the conditions under which this swap applies.

### Bid Strategy Label Mapping

The GAQL API returns internal bid strategy names that differ from what AMs see
in the Google Ads UI. Always translate to the UI-facing name:

- `TARGET_SPEND` → **Maximize Clicks**
- `MAXIMIZE_CONVERSIONS` → **Maximize Conversions**
- `MAXIMIZE_CONVERSION_VALUE` → **Maximize Conversion Value**
- `TARGET_CPA` → **Target CPA**
- `TARGET_ROAS` → **Target ROAS**
- `MANUAL_CPC` → **Manual CPC**
- `MANUAL_CPV` → **Manual CPV**
- `TARGET_CPM` → **Target CPM**
- `TARGET_CPV` → **Target CPV**

Never use the API label in the MAA output.

### Dual Date Range Analysis

Every MAA uses two time windows:
- **7-day data** is the primary view — the current MAA cycle. This is what the
  Metrics section displays.
- **30-day data** is the trend baseline — used as a reference in the Analysis
  section to identify momentum and whether previous actions had impact. The 30D
  data is NOT displayed in the Metrics section.

When writing the MAA:
- **Metrics section**: Show **7D data only**. Each campaign gets individual
  bullet points for each metric. The 30D data is context you use internally
  but don't display here.
- **Analysis section**: This is where 30D context appears. Use 30D as a
  reference point to support broader observations — not as a headline. Lead
  with the business implication, then cite the trend: "The campaign is
  struggling to compete in auctions — CTR is down to 0.36% this week from
  0.71% over the 30-day window." Don't lead with the number-to-number
  comparison itself.
- **Action section**: Prioritize based on what the 7D data reveals. If 30D
  shows a problem but 7D shows it's already improving, note that and deprioritize.
  If 7D shows a new problem not visible in 30D, escalate it.

### When the user uploads data manually

Auto-detect what they provide. Tell them what you found and what's missing.

### When you need to request data

Keep it short. The AM needs a clear, simple ask — not a training manual on how
Google Ads exports work. Say something like:

> "To run this MAA, I need three exports from the Google Ads account (last 30
> days): **Campaign summary**, **Keyword report with Quality Score**, and
> **Search terms report**. Want me to walk you through pulling those, or can
> you grab them?"

If they need the step-by-step, THEN provide export instructions from
`references/gaql-queries.md`. Don't front-load the full column-by-column
specs unless asked.

## Step 3: Diagnose

The following checklist is your internal reasoning framework for prioritizing
what to recommend. It exists to help YOU think through the account
systematically. It is NOT something you share with the user.

Never write the words "Tier 1", "Tier 2", "Tier 3", "Tier 4", "diagnostic
ladder", "foundations tier", "hygiene tier", or any similar framework language
in your response to the user. The user sees a clean MAA with numbered action
steps. They don't need to know how you arrived at the priority order — they
just need the recommendations to be in the right order, which this checklist
ensures.

Work through these checks in order. If earlier checks reveal critical issues,
focus your action steps there first.

### Patience Principle — Don't Recommend Changes Too Quickly

This skill's output needs to be safe for people without deep Google Ads
experience to act on. That means resisting the urge to recommend changes
based on a single week of data. Small-budget local service accounts produce
noisy data — one bad week on a relevant keyword doesn't mean the keyword is
broken. True learning takes time, and changing things too frequently prevents
the algorithm from stabilizing.

**Before recommending a change, ask: how long has this been running?**
- Changes implemented last week (new ads, new ad groups, bid strategy
  switches, restructures) should almost never be flagged as problems this
  week. They haven't had time to produce meaningful data.
- A keyword or ad group needs at least 2-3 weeks of consistent
  underperformance before it warrants an action item — unless there's a
  dramatic, obvious signal (e.g., spending $200 with zero clicks, or a
  keyword that's clearly off-intent).
- When in doubt, note the observation in Analysis ("worth watching") rather
  than escalating to Action. An observation in Analysis costs nothing. A
  premature change in Action can set the account back.

The goal of Action is 2-3 solid things we can do this week to improve
performance — not a list of everything that could theoretically be better.

### Check A — Is the foundation working?

- **Conversion Tracking**: Set up? Firing? Signs of breakage (strong CTR + zero
  conversions, sudden drops)? Cross-reference with CRM if possible.
  **Evaluate conversion accuracy at the campaign level only.** If a campaign's
  conversion count looks reasonable for its spend and matches backend data,
  treat it as accurate and evaluate performance against that real number. If
  account-level totals are inflated beyond what the campaigns show, that's
  likely activity from other campaign types (YouTube, LSA, etc.) outside this
  export — flag it briefly and move on. Don't diagnose account-level gaps.
  Focus on whether the campaigns in this export are counting the right things.
- **Bid Strategy**: Manual CPC or maximize clicks → conversion-based strategy is
  often the single biggest win. Needs 15+ conversions/30 days for smart bidding.
  Below that, consolidate or use shared budgets/portfolio bidding.
- **Geographic Targeting**: Tight to service area? "Presence" vs "Presence or
  Interest" setting? Leaking budget to wrong areas?
- **Brand vs. Non-Brand**: Mixed together? Brand inflates CTR, deflates CPA,
  masks non-brand underperformance.
- **Landing Page Relevance**: Dedicated service pages or just the homepage?

### Check B — Is budget leaking?

- **Negative Keywords**: Sort search terms by cost, look for irrelevant queries
  (jobs, DIY, wrong services, wrong geos). Common local service waste: job
  seekers, DIY queries, geographic mismatches.
- **Match Type Guardrails**: Broad match without negatives bleeds budget.
  Graduated proven queries to exact?
- **Ad Copy & Quality Score**: Break QS into its three components and report
  each one explicitly in the Analysis: **Expected CTR** (are the ads compelling
  enough to earn clicks?), **Ad Relevance** (does the copy match search intent?),
  and **Landing Page Experience** (does the LP deliver on the ad's promise?).
  Below Average on any = fixable problem. Always name which component is below
  average — "QS is low" without the component breakdown isn't actionable.
  Ad copy mentions service + city + clear CTA? Call/location extensions present?
- **Ad Copy Relevance**: Review RSA headlines and descriptions against the
  top-performing keywords and search terms. Are the ads speaking to what people
  are actually searching? Look for: generic headlines that don't mention the
  core service, missing geographic relevance (no city/area name), weak CTAs,
  or headlines that don't align with the landing page promise.
- **Ad Strength**: Google rates RSAs as Poor/Average/Good/Excellent. Poor or
  Average usually means not enough headline/description variants or not enough
  thematic diversity. Flag any ad with Poor or Average strength — adding more
  unique headlines is a quick win.
- **Ad Performance Splits**: If an ad group has multiple ads, compare their
  metrics. One ad dramatically outperforming another suggests the weaker one
  should be paused or rewritten.
- **Budget Allocation**: IS lost to budget on good campaigns while bad campaigns
  have leftover? Rebalance.

### Check C — Can we improve performance?

- **Ad Group Coherence**: Tightly themed? Mixed intents in one ad group?
  Restructuring doesn't trigger learning periods (Google 2026 guidance).
- **Device & Time-of-Day**: Mobile-heavy (common for local service calls)?
  Time patterns (emergency = after hours, remodeling = lunch breaks)?
- **Audience Layering**: Observation audiences for home services, homeowners?
- **Landing Page CVR**: Conversion rate variance across pages? Mobile experience?
- **Ad Testing**: 12+ headlines, 4+ descriptions in RSAs? Testing themes?

### Check D — Advanced (rarely needed for local service)

Read `references/playbook-frameworks.md` only when the account is mature and
Checks A-C are clean. Covers: incrementality, profit modeling, cohort/causal
analysis, metric trade-off curves, experimentation protocols.

## Step 4: Deliver the MAA

The output should be concise and directly useful. Not an audit report — a working
document the AM can act on this week.

### Output Format

Save the MAA as a markdown file in the client's output folder. The base output
location is configurable — see `CONFIGURATION.md` at the repo root. The default
convention is:

`{OUTPUT_DIR}/{Client Name}/{Client}_MAA_{YYYY-MM-DD}.md`

Example: `output/Summit Dumpster Rental/SummitDumpster_MAA_2026-04-24.md`

`{OUTPUT_DIR}` defaults to `./output` but can point anywhere — a project folder,
a synced drive, or (for BlitzBase users) the client's vault folder. If the client
folder doesn't exist, create it. The file should contain the full MAA (Metrics,
Analysis, Action) as formatted markdown. Present the finished file to the user
when complete.

### Narrative Update (weekly cycle only)

After delivering the MAA during a weekly analysis cycle, update the client's
narrative document (`{Client}_MAA-Narrative.md`):

1. **Add a new Weekly Log entry** at the top of the log (most recent first)
   with today's date. Include: key findings, actions recommended, and open
   questions for next week.
2. **Update Running Themes** if anything shifted — a resolved issue gets
   marked resolved, a new persistent pattern gets added.
3. **Fill in Account Context** if it's still a placeholder.
4. **Fill in Client Contacts** if missing. Note the name(s) and role(s) of
   anyone who receives the MAA, manages the website, or makes strategic
   decisions. This enables the MAA to address action items to the right
   person by name.

**Do NOT update the narrative for ad-hoc analyses.** The user will update it
with whatever they actually share with the client, plus any client feedback.
The narrative reflects what was communicated, not every draft.

### Metrics

**Campaign-level is the highest level of analysis.** Do not analyze or report
account-level aggregate statistics. Account-level totals will appear in exports
but they may include activity from campaign types outside this analysis (LSA,
etc.) whose goals and conversions are not visible in the export. Analyzing
account-level data risks conflating unrelated activity and producing misleading
conclusions.

If account-level totals are visibly higher than the sum of the campaigns in the
export (e.g., account shows 209 conversions but campaign-level data only shows
1), add a brief callout: "The account-level numbers include activity from
campaigns outside this export — we should verify which conversion actions are
counted at the campaign level to make sure we aren't inflating our counts."
Then move on. Do not try to diagnose or explain the account-level gap.

Break out metrics **by campaign** using bullet points. Each campaign gets a bold
header showing campaign name, campaign type, and bid strategy. Then list
individual bullet points for each core metric, with the **metric name bolded**
as the leading portion of the bullet. Show **7D data only** in this section.

Top-N focus — only include campaigns with meaningful spend.

**Required metrics for every campaign** (always include these seven):
- **Cost**
- **Impressions**
- **Clicks**
- **CTR**
- **CPC**
- **Conversions** — break out by conversion action name when the data includes
  it: "**Conversions**: 4 (2 Phone Calls, 2 Lead Forms)". This tells the reader
  what kind of business activity the ads are driving, not just a count.
- **CPA** (or note "N/A — zero conversions" if applicable)

**Do NOT include an "Other metrics of note" line.** Metrics like search
impression share, IS lost to budget/rank, Quality Score components, and
conversion rate are diagnostic — they belong in the Analysis section where
they can be explained in context. The Metrics section is the "what happened"
overview. Keep it clean: the seven core bullets per campaign, nothing more.

**Example format:**

> **Emergency Plumbing - Search** (Search | Maximize Conversions)
>
> - **Cost**: $2,450
> - **Impressions**: 38,200
> - **Clicks**: 847
> - **CTR**: 2.22%
> - **CPC**: $2.89
> - **Conversions**: 34 (22 Phone Calls, 12 Lead Forms)
> - **CPA**: $72.06
>
> **General Plumbing - PMax** (Performance Max | Maximize Conversions)
>
> - **Cost**: $1,200
> - **Impressions**: 15,400
> - **Clicks**: 312
> - **CTR**: 2.03%
> - **CPC**: $3.85
> - **Conversions**: 8 (5 Phone Calls, 3 Lead Forms)
> - **CPA**: $150.00

**Video and Display campaigns get a condensed format.** Instead of the full
seven-metric breakdown, show them in a compact block: campaign name, cost,
impressions, clicks, and the metric most relevant to their goal (e.g., video
views and cost-per-view for Video campaigns, or CTR for Display). One or two
lines, not a full bullet list. Example:

> **Home Service Contractors Video** (Video | Target CPV) — $105.52 spent,
> 21.5K impressions, 3,280 views at $0.03 CPV. Solid engagement for a
> $10/day awareness budget.

After all campaigns, include a **Data gaps** note if relevant — flag missing
data (e.g., empty keyword reports from PMax, no visible search terms). Keep
it brief.

### Business Metrics Trend

Close the Metrics section with a short paragraph (2-3 sentences max) on the
most important business-level question: are we converting? This is a
high-level pulse check on conversion trends — week over week and against the
30-day average. It states what is happening, not why.

Examples:
- "Conversions are trending up — 5 this week vs. 3 last week, with a 30D
  average of 3.4/week. CPA is holding steady at $165."
- "Zero conversions for the second straight week despite 76 clicks. The 30D
  window shows the same pattern — 153 clicks, zero conversions."
- "Steady week — 3 conversions at $82 CPA, in line with the 30D average of
  3.2/week at $79."

If things are going well, this is just a quick confirmation and doesn't need
to feed the later sections. If conversions are declining or absent, this
paragraph naturally sets up what Analysis needs to explore.

---

### Analysis

A narrative explaining what the data means in context — the "why" behind
the numbers. Write in "we" language. Keep to 2-4 focused paragraphs, with
each paragraph no longer than 3-4 sentences. Every paragraph should build
toward the action items, advancing the story of what's happening and why the
recommended actions follow logically.

**Structure the Analysis as a narrative arc, not a topic list.** Organize by
storyline: what happened → why it happened → what that means for the decision.
Don't organize by category (waste analysis, then QS analysis, then strategy
analysis). Instead, each paragraph should build on the last so that by the
time the reader reaches the Action section, the reasoning is already built
and the recommendations feel inevitable.

**Open with the most important diagnostic finding.** Lead with the insight
that drives the biggest action item. Don't open with a recap of last week's
activities — that's backward-looking. The Business Metrics Trend paragraph
in the Metrics section already covers the week-over-week conversion picture.
Analysis opens with *why* things are the way they are.

**Previous efforts can be mentioned briefly, but keep them subordinate.**
If negatives from last week are working or a QS change is showing movement,
weave that into the narrative in a clause or sentence — don't give it a full
paragraph. Example: "The competitor negatives from two weeks ago are holding
— no branded bleed this week — but a new waste cluster appeared around
navigational queries." The prior effort is context, not the headline.

**This is where diagnostic metrics live.** Metrics like impression share, IS
lost to budget/rank, Quality Score components, conversion rate, and search
term waste percentages belong here — accompanied by the explanation of what
they mean and why they matter. These metrics were intentionally excluded from
the Metrics section so they could appear here with context rather than as
raw numbers.

Key behaviors:
- **Keep paragraphs to 3-4 sentences.** If a paragraph is running longer, it's
  trying to cover too much. Split it or cut. The Analysis section should be
  tight and scannable — a busy AM reads this between calls.
- If foundational issues exist, lead with that plainly: "Before we optimize
  anything, we need to make sure conversion tracking is solid."
- Always use balanced metrics language: "CPA improved to $35, but volume dropped
  from 45 to 28 leads — we need to understand why."
- Translate to business impact: "47 irrelevant search terms cost $830 last
  month — roughly 4 leads at our current CPA."
- Be tactful: "still gathering data" not "failing."
- **Routine items get brief treatment.** Search term cleanup, standard negative
  keyword additions, and ongoing optimization work are expected week to week.
  Don't over-explain them unless there's a decision to make. "We found another
  $40 in waste on navigational queries — negatives are ready to run" is enough.
  A full paragraph dissecting each waste term is too much unless the pattern
  reveals something new or requires a strategic choice.
- **Narrative continuity**: If a narrative document exists, reference prior
  findings briefly where relevant. Don't repeat context the narrative already
  captures — build on it.
- **Always end the Analysis section with a standing CRM quality prompt.** This
  is a brief, consistent checkpoint — not an action item. Frame it around lead
  quality, not tracking verification. Examples:
  - When conversions exist: *"CRM check: We recorded 23 conversions this week
    — how's the quality of those leads looking?"*
  - When zero conversions: *"CRM check: Google shows zero conversions this
    week — are we seeing any calls or form fills that aren't being tracked?"*
  This stays in Analysis every week as a standing prompt to the AM. It only
  escalates to an Action item if the data itself suggests something is broken
  (e.g., strong CTR with zero conversions over multiple weeks, or a sudden
  drop from a known baseline).
- **Stay at the campaign level.** If account-level totals don't match campaign
  totals, note it briefly and recommend verifying conversion actions — but don't
  try to explain the gap or build analysis around account-level numbers.
- Don't restate what's already visible in the Metrics bullets — interpret and
  connect the dots instead.
- **Use 30D trends as supporting evidence, not headlines.** When comparing 7D to
  30D, weave the trend into a broader observation rather than leading with it.
  Instead of "CTR halved from 0.71% to 0.33%" as the opening, lead with the
  business implication and use the trend as backup: "The campaign is struggling
  to compete in auctions — CTR has been sliding and is down to 0.33% this week
  from 0.71% over the 30-day window."
- **Video/Display gets a brief note, not deep analysis.** One or two sentences
  on whether the awareness campaign is delivering as expected. Don't force lead
  gen diagnostics onto awareness campaigns. If a Video or Display campaign has a
  notable finding (e.g., irrelevant placements, surprising engagement), call it
  out — otherwise, a quick "running as expected" is sufficient.
- **Weave ad copy findings into the narrative.** The ad report data (headlines,
  descriptions, ad strength, per-ad performance) should inform Analysis
  naturally — not as a separate section. When discussing a campaign's
  performance, note if ad copy is a contributing factor. Examples:
  - Poor CTR + generic headlines → "Part of the CTR problem may be the ad copy
    — the headlines don't mention veneers specifically, which is what people
    are searching for."
  - Ad strength rated Poor/Average → "Google is rating this RSA as Average,
    which usually means it needs more headline variety to find winning
    combinations."
  - One ad dominating impressions over another → "One ad in this ad group is
    pulling nearly all impressions at a much higher CTR — the other may be
    dragging performance down."
  If the ads look healthy and well-aligned, there's no need to force ad copy
  commentary. Only surface it when it's a meaningful factor.

### Action

2-3 specific, forward-facing action steps — things we can do this week or
next to improve ad performance. Prioritized by the diagnostic ladder (without
mentioning the ladder). Each action step should:

- State what to do in plain language
- State why, linked to something in the Analysis
- Note expected impact where possible

**Every action item must be forward-facing.** The test: does this item
describe something we're going to *do*, or something we already *did*?
Confirming that last week's negatives are holding, noting that the new ads
haven't moved QS yet, or checking that a previously excluded keyword stayed
out — those are observations, and they belong in Analysis. Action is
exclusively about what happens next.

Watching or monitoring something *can* be an action item if it's flagging a
potential future problem: "Watch 'demolition bin rental' — if it continues
spending without converting over the next 1-2 weeks, consider pausing." That's
forward-facing because it describes what we might need to do. But "We checked
and keyword A didn't show up this week" is backward-facing and belongs in
Analysis.

The ordering naturally starts with the biggest-impact fixes first (broken
tracking, wrong bid strategy) before leakage fixes (negatives, QS) before
optimization work (creative testing, audiences, ad copy improvements). The user
just sees a numbered list of smart recommendations — your internal reasoning is
invisible.

### Distinguish "things we did" from "things the client needs to do"

Action items fall into two categories, and they should read differently:

**Our-side items** (negatives added, scripts run, ad copy changes made):
Use completion language. If already done, mark with ✅ and state what happened.
If pending but within our control, state what we'll do and when.

**Client-side items** (landing page changes, CRM access, strategic decisions):
Use collaborative language. Address the client contact by name (from the
narrative document). Frame as a partnership, not a task assignment:
- "Nicholas, we noticed the interior page headline still references winter
  services — would you like us to take a pass at updating it? We'd need
  Elementor access."
- "Sam, do you have a sense of what types of consumers are specifically
  searching for 'roll off dumpster'? The answer would help guide which route
  we pursue."

The pattern: name the person → describe the situation → ask a specific
question or make a specific offer with a specific gate (what you need from
them to proceed).

### When the skill presents strategic options, end with a question

When an action item involves a strategic choice (restructure vs. rewrite,
pause vs. hold, switch bid strategy vs. wait), present the options and then
ask the client a question that helps inform the decision. The skill should
inform the decision, not make it. The question positions the client as a
decision-maker with expertise the data alone can't provide.

**Example:** "From our perspective, two options: (a) write dedicated
headlines, or (b) break the variants into their own ad group. Sam, do you
have a sense of what types of consumers search for 'roll off dumpster'? The
answer would help guide which route we pursue."

### Split completed and open items when density is high

When more than half of the action items are already completed, the reader has
to wade through done items to find what still needs their attention. In these
cases, lead the Action section with a brief note on what was completed this
week, then list the open items in detail:

> **This week's priority order: stop the bleeding first, then fix the structure.**
>
> 1. ✅ **Negatives script run (4/17).** 27 negatives added. Estimated
>    $140–160/week in waste eliminated.
> 2. ✅ **AI Max turned off (4/17).** Confirmed with Google rep.
>
> 3. **Hold Maximize Conversions through 5/1 — decision point extended.**
>    [detailed rationale...]
> 4. **Apply English-only language targeting — still pending.** [details...]

This keeps the completed items visible for the record without burying the
open items.

**Ad copy actions belong in the optimization tier** — they're important but
rarely the first thing to fix. Typical ad copy action items include: adding
more headline/description variants to improve ad strength, rewriting headlines
to better match high-value search terms, pausing underperforming ads, or
adding geographic or service-specific language. When recommending ad copy
changes, be specific — cite the actual headlines that need work and suggest
concrete alternatives based on what the search term data shows people are
looking for.

**Merge related actions into single steps.** If two actions are naturally done
together (e.g., building a new campaign + pre-loading negatives for that
campaign), combine them into one action step. Don't pad the list — fewer,
meatier steps are better than more granular ones.

### Quarterback Dispatch — Chaining Downstream Skills

The analyzer is the quarterback of the Google Ads skill pipeline. When
diagnosis reveals a problem that a downstream skill can solve, don't just
mention the skill — dispatch it. This means including enough context in the
action item that the downstream skill can execute immediately when the user
(or a scheduled task) triggers the chain.

**Dispatch triggers and what to hand off:**

| Condition Detected | Dispatch To | Hand Off |
|---|---|---|
| Below Average Ad Relevance or Expected CTR on keywords with >$50/week spend | **google-ads-copy-optimizer** | Ad group name, current headlines/descriptions, top search terms for that ad group, QS component breakdown |
| Waste terms identified totaling >10% of campaign spend | **google-ads-change-scripts** | List of terms to negate, match types, campaign name, estimated weekly savings |
| Below Average Landing Page Experience on keywords with meaningful spend, OR strong CTR + zero conversions for 2+ weeks | **google-ads-lp-auditor** | Landing page URL, current ad copy (headlines + descriptions), QS component data, campaign context (business, target CPA, conversion action, audience) |
| Ad group restructure recommended (e.g., keyword needs its own ad group) | **google-ads-change-scripts** | Keywords to move, source/destination ad group names, match types |
| Ad rated POOR or AVERAGE strength | **google-ads-copy-optimizer** | Current headlines/descriptions, ad strength rating, ad group's top search terms |

**How to write a dispatch in the Action section:**

Don't write: "The google-ads-copy-optimizer skill can generate headlines."
(This is a suggestion to the air.)

Do write: "**Dedicated RSA created for 'Roll Off Dumpster' ad group.** 15
headlines covering core service, geo, urgency, CTAs, sizes, and
differentiators. 'Roll Off Dumpster Rentals' pinned to H1."
(This is what happens when the optimizer was actually dispatched.)

Or, if the action is pending and needs the user to trigger it: "**Next step:
generate dedicated ad copy for the new ad group.** The copy optimizer has
what it needs — the top search terms, the QS breakdown, and the current
headlines. Ready to run when you are."

The goal is that each MAA action item either (a) was already handled by a
downstream skill during this session, or (b) is staged with enough context
that a single command triggers the downstream skill. No action item should
require the user to re-explain the problem to another skill.

### The "Start here" close

End the Action section with a close that does one of these — not a formulaic
restatement of the top action item:

1. **Name a person and make a specific ask**: "Nicholas, we need to get going
   on the interior page copy fixes. We're happy to help if you'd like — we'd
   just need website access."
2. **Pose the most important open question for next week**: "The key question
   going into next week: does the cleaned-up traffic mix produce the account's
   first conversion under Maximize Conversions?"
3. **Both**: A named ask followed by the open question.

If the top action item is already completed (e.g., negatives were run during
this session), the close should point to what's next, not what's done.

**Example of the right level of conciseness for an Action section:**

> 1. ✅ **Added 12 negative keywords** — $640/month in waste on job-seeker
>    and DIY queries, stopped today.
> 2. **Switch the main campaign from Maximize Clicks to Maximize Conversions** —
>    we're optimizing for traffic volume, not leads. With 18 conversions last
>    month, we have enough data for smart bidding.
> 3. **Create a separate brand campaign** — brand and non-brand are mixed,
>    which makes our $45 CPA look better than it really is on non-brand.
>
> **Start here:** Sarah, the bid strategy switch is the biggest lever we
> haven't pulled yet. Do you want us to make the change this week, or would
> you prefer to wait until the negatives have a week of clean data?

## Coaching Mode

When a user submits their own MAA for critique or wants to learn the process,
read `references/coaching-guide.md` for the full coaching framework. The short
version: be specific about gaps, teach transferable principles (balanced metrics,
the "because" bridge, action-to-analysis linkage), and use a constructive tone.

Important: even in coaching mode, don't teach the internal diagnostic checklist
labels (Check A/B/C/D or any tier numbering). Instead, teach the underlying
principle: "fix what's broken before optimizing what works" and "verify your
foundation before scaling." The sequencing concept is valuable to teach — the
internal labels are not.

## PMax-Specific Notes

- Asset Groups replace ad groups — evaluate by asset group theme
- Search term visibility is limited — use insights tab and categories
- Brand exclusions are critical — without them, PMax claims brand conversions
- URL expansion controls matter — is PMax going to intended pages?
- Watch for PMax/Search cannibalization (balanced metrics: PMax CPA looks good
  but total conversions didn't grow)

## Google's 2026 Structural Guidance

- **Consolidation** over granularity — fewer campaigns with more data density
- **15 conversions/30 days** threshold for smart bidding
- **AI Max** uses LLM-based intent matching — be curious about unexpected queries
- **Restructuring doesn't trigger learning periods** — bidding models recognize
  assets regardless of structure
- **Brand and geo controls** are the modern segmentation levers
