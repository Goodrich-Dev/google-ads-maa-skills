---
name: google-ads-client-view
description: >
  Render a finished Google Ads MAA into a client-facing report. Produces a
  markdown client view AND a paste-ready HTML version (built for Basecamp, works
  in any rich-text editor), both led by a one-paragraph summary, two 13-week
  trend charts (leads, cost per lead), emoji-flagged critical items, and a
  separated client to-do, with the full MAA preserved below. It does NO new
  analysis: every line is DERIVED from the source MAA, never invented. Runs as
  the final step after the weekly MAA is written. Trigger on "render the client
  view", "Basecamp version of the MAA", "client report for {client}", or
  "client-facing version of this MAA". Google Ads only.
---

# Google Ads Client View — MAA Render Stage

## What this skill does

Takes a completed Google Ads MAA (from `google-ads-analyzer`) and produces the
client-facing layer that sits on top of it, in two formats:

1. A **markdown client view** (`{Client}_Client-View_{date}.md`), self-contained,
   with the two trend charts embedded as images. Reads cleanly in any markdown
   preview.
2. A **paste-ready HTML version** (`{Client}_Client-View_{date}.html`) whose body
   is rich text (headings, bold, bullets, blockquote, emoji), so it survives a
   copy-paste into a client comms tool. It is tuned for **Basecamp** (the markup
   is restricted to what Basecamp's editor keeps on paste), and the same
   constraints make it safe for most rich-text editors. The two charts are
   dragged in as PNGs.

The full MAA is reproduced **verbatim, in skill bullet format**, below the
summary in both formats, so nothing is lost. The goal is to solve "too dense for
the average owner" by leading with a 30-second read, not by removing depth.

## What this skill does NOT do

- **No analysis.** It never diagnoses, re-weights, or second-guesses the MAA. If
  a finding is not in the MAA, it does not appear in the client view.
- **No edits to the source MAA.** The dated MAA is read-only here. The analysis
  artifact stays pure.
- **No metric invention, no severity reclassification.** See the Cardinal Rule.

## Where it sits in the pipeline

```
google-ads-analyzer  ->  {OUTPUT_DIR}/{Client}/{Client}_MAA_{date}.md
                                        |
                                        v  (final step, every week)
google-ads-client-view  ->  reads that frozen MAA + trend data
                        ->  emits the .md and .html client views + 2 PNG charts
```

It is the **last** step and runs **every** week (unconditionally). Unlike the
copy optimizer or LP auditor, which fire on a detected trigger and add expertise,
this runs always and adds presentation only.

---

## THE CARDINAL RULE: derive, don't invent

Every line in the client view must trace to a specific line in the source MAA. If
you cannot point to where the MAA says it, **delete it.**

This rule exists because the naive version of this layer invents client-friendly
claims the MAA never made (e.g. "phone tracking is solid" when the MAA's CRM
check actively *questions* lead quality), or frames an our-side structural item as
a client-facing crisis. Both are the failure mode of writing the summary as
original prose instead of projecting it from the analysis.

Concrete prohibitions:
- **Never add a metric** not present in the MAA.
- **Never upgrade a hedge into confidence.** If the MAA says "one caution, single
  week of data," the summary may not present that result as proven.
- **Never invent reassurance** ("tracking is solid", "everything's healthy").
- **Never editorialize on charts** ("lower is better", "how many leads the ads
  drove"). Label them with the plain metric name only. Owners know their business.
- **Never reclassify severity.** An our-side action stays our-side; only genuine
  client decisions go in "what we need from you" or the red flags.

---

## Step 1: Load the source MAA (read-only)

Read the latest dated Google Ads MAA for the client:
`{OUTPUT_DIR}/{Client Name}/{Client}_MAA_{YYYY-MM-DD}.md` (see `CONFIGURATION.md`
for `{OUTPUT_DIR}`). That MAA is the **only** source of findings, numbers, and
actions for the client view. If the client is multi-platform and the MAA bundles
several channels, use only the Google Ads section.

If a per-client spec exists (campaign list, top-N metric, target CPA), read it for
the chart target line. If no dated MAA exists for the period, stop — there is
nothing to render, build the MAA first.

## Step 2: Build the 13-week chart data

The charts plot history, so they need the trailing ~13 weeks, not just this week.
Preferred source, in order:

1. **`{OUTPUT_DIR}/{Client Name}/trend.csv`** — a per-client trend file you append
   to each week. Columns: `week,leads,cpl_week,cpl_30d`. This is the deterministic
   path, no prose parsing.
2. **Fallback:** if the trend file is absent, assemble the series by reading the
   `Conversions` and `CPA` lines from the trailing weekly MAAs in the client's
   output folder, oldest to newest, primary campaign. Be consistent about which
   campaign or account total you use, and match whatever the MAA's bottom-line
   paragraph uses so the chart and the summary agree. Write the assembled rows to
   `trend.csv` so the next run is deterministic.

Then run the bundled generator (it does the plotting; do not hand-draw SVG):

```
python3 scripts/render_charts.py \
  --data {trend.csv} \
  --outdir "{OUTPUT_DIR}/{Client Name}/" \
  --client "{Client display name}" \
  --date {YYYY-MM-DD} \
  --target {client target CPA; omit if unknown}
```

It writes `{slug}_chart_leads_{date}.png` and `{slug}_chart_cost-per-lead_{date}.png`.
Blank `cpl_week` / `cpl_30d` cells are skipped (zero-conversion or unreported
weeks). The target line is drawn only if `--target` is given. The script needs
`matplotlib` (`pip install matplotlib`).

**Sanity check:** the chart's latest points must equal the MAA's bottom-line
numbers (this week's CPA, the 30-day average). If they do not match, the trend
data is wrong. Fix it before continuing.

## Step 3: Project the summary (each section maps to the MAA)

Build the client view in this fixed order. The right column is the ONLY place
each section may draw from.

| Client-view section | Source in the MAA |
|---|---|
| Header + status dot (🟢/🟡/🔴) | Overall tone of the MAA. 🟢 on track, 🟡 watch, 🔴 needs attention. Do not overclaim a 🟢 if the MAA hedges. |
| **The bottom line** (1 paragraph) | The MAA's opening / bottom-line paragraph, lightly trimmed. Plain language, no QS/IS/CPA jargon above the fold. |
| **The trend** (2 charts) | The PNGs from Step 2. Caption only if needed; no value judgments. |
| **What's working ✅** | The positive findings already stated in Analysis. Each bullet a real finding (e.g. "new ad drove 10 conversions at $21.49"), not a vibe. |
| **What needs attention 🔴** | Client-facing critical/decision items only, typically the one or two things the client must weigh (budget cap costing leads, broken tracking). Our-side fixes do NOT go here. |
| **What we need from you 👉** | The **client-side** Action items. Keep the ask plus the one-line why. |
| **What we did this week** | The **our-side** Action items (staged dispatches, launches, negatives), phrased as done/in-progress, no action needed. |
| **Full analysis** | The canonical MAA, reproduced **verbatim and in skill bullet format** (see Step 4). |

Severity routing test for each finding: *Is there a decision only the client can
make?* goes to 🔴 / "what we need from you". *Did we already handle or stage it?*
goes to "what we did". *Is it a positive result?* goes to "what's working".

## Step 4: Render both outputs

### 4a. Markdown client view

Use `references/markdown-template.md` as the structural pattern. Embed the charts
as markdown images pointing at the two PNGs (same folder). Keep the section order
from Step 3. The Full analysis section reproduces the MAA verbatim, metrics in
bullet format.

### 4b. Paste-ready HTML version (Basecamp-tuned)

Use `references/basecamp-template.html` as the structural pattern (reuse the
layout, not the sample content). The body that gets pasted must use only what a
restrictive rich-text editor keeps on paste:

- **Keep:** headings, **bold**, *italic*, bulleted and numbered lists,
  blockquote, links, emoji.
- **Drop / never rely on:** inline SVG, `<table>`, CSS-styled boxes, background
  colors, custom font color. Encode criticality with **emoji + bold**, not color.
- **Charts = dragged-in PNGs.** Most editors embed PNG/JPG inline but cannot take
  SVG. Put a clearly marked placeholder where each chart goes, e.g.
  `[CHART] Drag in: {filename}.png`, and tell the user to drag the two PNGs there.
- **Emoji legend:** ✅ working, 🔴 needs attention/decision, 👉 what we need from
  you, 📊 charts, 🟢/🟡/🔴 status dot.
- **Metrics stay in bullet format** in the Full analysis section — the core
  bullets per Search/PMax campaign (Cost, Impressions, Clicks, CTR, CPC,
  Conversions, CPA). Do not flatten metrics into a paragraph.
- Include a short **how-to banner** at the top of the HTML file (outside the paste
  body) telling the user: select the report body, copy, paste into the comms tool,
  then drag the two PNGs into the `[CHART]` spots.

> **Adapting beyond Basecamp:** these constraints are Basecamp's, and they are a
> safe subset for most rich-text editors (email, Notion, Google Docs paste, help
> desks). If your tool accepts richer markup, you can relax them — but the
> emoji-over-color and dragged-in-PNG approach is the most portable default.

**Style:** no em dashes anywhere in either output. Use commas, periods, or
hyphens. Match the client's existing MAA voice.

## Step 5: Output

Write to `{OUTPUT_DIR}/{Client Name}/`:

- `{Client}_Client-View_{YYYY-MM-DD}.md`
- `{Client}_Client-View_{YYYY-MM-DD}.html`
- `{slug}_chart_leads_{YYYY-MM-DD}.png`
- `{slug}_chart_cost-per-lead_{YYYY-MM-DD}.png`

Increment a `_v2`, `_v3` suffix if a file for that date already exists (never
overwrite). Present all four files when done. Do **not** touch the source MAA.

This is a **draft for review before posting**, same as the MAA itself. Never
auto-publish to the client comms tool.

## Step 6: Verify before presenting (required)

Run the trace-check:

1. **Every** summary line maps to a line in the source MAA. List any that do not —
   they must be removed, not "supported later."
2. No invented metrics; no hedge upgraded to confidence; no chart editorializing.
3. The chart endpoints equal the MAA's bottom-line numbers.
4. Metrics in Full analysis are in bullet format, all core bullets per campaign.
5. The HTML paste body contains no `<svg>`, `<table>`, or color-dependent meaning.
6. No em dashes in either output.

If any check fails, fix and re-verify. For high-stakes clients, run the
trace-check as a fresh read of the MAA against the summary (a subagent is ideal).

## Connected

- `google-ads-analyzer` — produces the MAA this stage renders.
- `../../shared/frameworks/methodology.md` — the MAA doctrine (MAA = analysis,
  not a task list).
- `references/markdown-template.md`, `references/basecamp-template.html` — output
  patterns.
- `scripts/render_charts.py` — deterministic chart generator.
