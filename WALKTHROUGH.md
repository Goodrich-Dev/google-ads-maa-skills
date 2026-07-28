# Walkthrough — Running the Weekly MAA System End to End

This is the full operating loop: from a cold install to a client-ready report.
You do the **one-time setup** once per client account. After that the
**weekly loop** (steps 5-7) is what you repeat.

There are two ways to feed the analyzer data. **Path A (MCP, recommended)**
pulls live from the Google Ads API — correctly-labeled data, no email plumbing,
and the analyzer can pull follow-up data mid-analysis. **Path B (email
pipeline)** is the original automation and remains the fallback when no MCP is
connected. If you would rather not automate at all, skip to "Manual mode" at the
bottom — you can run the whole thing from CSV exports.

---

## One-time setup — Path A: Google Ads MCP (recommended)

### Step A1 — Connect a Google Ads MCP

Set up the official Google Ads MCP server and connect it to your Claude
environment (Cowork connector or Claude Code MCP config), authorized against
the Google Ads account (or the MCC that manages it). Full instructions —
prerequisites, local and Cloud Run deployment, verification, and the
reauthorization caveat — are in **`shared/frameworks/mcp-setup-guide.md`**.

### Step A2 — Record the client's CID

Note the client's customer ID (digits only, no dashes) somewhere the analyzer
reads — a per-client config or notes file. Runs shouldn't start with a lookup.

### Step A3 — Verify with one pull

Ask: *"Pull the 7-day campaign summary for {client} via the Google Ads MCP."*
The queries live in `shared/frameworks/gaql-query-pack.md`. Confirm the numbers
match the Google Ads UI. That's setup done — no script, no Gmail label.

**Known limitation:** some Google Ads MCP setups require reauthorization as
often as daily. If the `search` tool is missing at the start of a run,
reauthorize the connector, or let the run fall back to the email path (keep
Path B wired as a backup if you schedule hands-off runs).

---

## One-time setup — Path B: the email pipeline (fallback / backup)

### Step 1 — Build the data-collection script

Ask the script builder to generate the weekly data feed:

> "Build a Google Ads script that emails me the weekly MAA data."

The `google-ads-script` skill produces a single JavaScript file with a CONFIG
block at the top. Edit two things in CONFIG before you deploy it:

- the **recipient email** (where the weekly data lands — use the inbox you will
  point the analyzer at),
- the **error email** (`ERROR_EMAIL`) so failures alert you.

The script emails ten datasets per run, wrapped in `---BEGIN ...---` /
`---END ...---` markers, with the subject `[MAA Data] {Account Name} — {date}`.
The exact contract is in `shared/frameworks/data-pipeline-contract.md`.

### Step 2 — Install the script in Google Ads

1. In the Google Ads account: **Tools → Bulk actions → Scripts → +** (new script).
2. Paste the script in.
3. **Authorize** it when prompted (it needs permission to read the account and
   send mail).
4. Click **Preview**, then **Run** once. Confirm the email arrives.

### Step 3 — Set up the Gmail label so the analyzer can find the data

The analyzer locates the latest data email by label or subject. Create a filter
so every feed email is tagged consistently:

1. In Gmail: **Settings → Filters and Blocked Addresses → Create a new filter.**
2. Match on subject `[MAA Data]` (or on the sender, Google Ads Scripts).
3. **Apply the label** `maa-data-automation` (create it if needed).

Now "find the latest MAA data for {client}" always resolves to the right email.

### Step 4 — Schedule the script to run weekly

Back in the Google Ads Scripts screen, set the script's **Frequency** to
**Weekly**, on the day/time you want the data ready (Friday morning is a common
choice so it is waiting for a Monday review). Save.

That is setup done. Data now arrives on its own every week.

---

## The weekly loop

### Step 5 — Run the analysis

When you are ready for the week's review, prompt the analyzer:

> "Do an MAA for {client}."

It will: pull the datasets live via the Google Ads MCP (or, on the fallback
path, find the latest data email and parse the ten datasets), diagnose the
account (the MAA framework), and where the data warrants it, dispatch the
downstream skills — `google-ads-copy-optimizer` for ad-copy fixes,
`google-ads-change-scripts` for negatives/pauses/restructures,
`google-ads-lp-auditor` for landing-page issues. On the MCP path it can also
pull follow-up data mid-analysis when its own findings raise a question, instead
of deferring to next week. It saves the MAA to
`{OUTPUT_DIR}/{Client}/{Client}_MAA_{date}.md`.

**Want this to run automatically?** Schedule it as a recurring task ("every Monday
at 9am, run the MAA for {client}"). The analysis still produces a draft for you to
review — it does not act on the account by itself.

### Step 6 — Review and adjust

Read the MAA. This is a draft, not an auto-send:

- Sanity-check the numbers against what you know about the account.
- For any generated change script (negatives, pauses, RSA edits), **run it in
  dry-run first** (the change scripts default to dry-run), confirm the preview,
  then run for real.
- Tweak wording or priorities. If you correct something, the analyzer learns the
  account's context for next week if you keep a narrative file (optional, see
  `CONFIGURATION.md`).

### Step 7 — Render the client view

Once the MAA reads the way you want, prompt the render step:

> "Render the client view for {client}."

`google-ads-client-view` produces a plain-language client report in two formats —
a clean markdown version and a paste-ready, Basecamp-tuned HTML — led by a
one-paragraph summary and two 13-week trend charts (leads, cost per lead), with
the full MAA preserved below. It adds presentation only; it never changes the
analysis.

**For the trend charts:** keep a `trend.csv` in the client's output folder
(`week,leads,cpl_week,cpl_30d`), appending one row per week. If it is missing, the
render step will reconstruct the history from your prior MAAs, but maintaining the
file keeps the charts exact. The chart generator needs `matplotlib`
(`pip install matplotlib`).

Deliver the HTML by pasting it into Basecamp (or any rich-text tool) and dragging
in the two chart PNGs where marked.

---

## The cadence at a glance

| Frequency | What | Which skill |
|---|---|---|
| Once per account | Connect the Google Ads MCP + record the CID (Path A), or build + schedule the data script and Gmail label (Path B) | connector / `google-ads-script` |
| Weekly (auto, Path B) | Data email arrives | the deployed script |
| Weekly | Run the MAA (MCP pull, or parse the email) | `google-ads-analyzer` (+ dispatched specialists) |
| Weekly | Review, dry-run and apply changes | you + `google-ads-change-scripts` |
| Weekly | Render and deliver the client report | `google-ads-client-view` |

---

## Manual mode (no automation)

You can run everything without the script or scheduling. Export three reports from
the Google Ads account (campaign summary, keyword report with Quality Score,
search terms), then:

> "Do an MAA for {client}" — and attach the exports.

The analyzer auto-detects what you provided and tells you what is missing. Steps
6 and 7 (review, render) are identical.

---

## Troubleshooting

- **Live-account queries fail but the connection "works."** If
  `list_accessible_customers` succeeds but every `search` against a real client
  CID errors with an authorization/permission denial, your developer token is
  almost certainly still **test access** — test tokens can only query test
  accounts, and the error text does not say so. Fix: request **Basic access**
  in your MCC's API Center (manual approval, 1–3 business days). See "Before
  you start" in `shared/frameworks/mcp-setup-guide.md`.
- **MCP `search` tool missing / unauthorized.** Some Google Ads MCP setups
  require frequent (even daily) reauthorization. Reauthorize the connector and
  retry, or let the run fall back to the email path. If you schedule hands-off
  runs, keep Path B wired as the backup until your connection persists.
  How to tell **lapsed authorization** from **misconfiguration**: lapsed auth
  worked before and now the tools are simply absent or prompt for OAuth again —
  reauthorizing fixes it in one click. Misconfiguration never worked in the
  first place, or fails with explicit credential/permission errors after
  authorizing — recheck the env vars and the OAuth app's publishing status
  (a "Testing"-status app expires refresh tokens after 7 days, which
  masquerades as constant lapses).
- **Analyzer can't find the data (email path).** Confirm the feed email arrived
  and carries the `maa-data-automation` label (Step 3). Check the subject starts
  with `[MAA Data]`.
- **Quality Score components look swapped (email path only).** Affected script
  versions invert the Expected CTR and Ad Relevance columns; run
  `shared/scripts/correct_qs_columns.py` on the raw email data, then read the
  corrected file as-labeled. Never apply the swap to MCP data — the API returns
  correctly-labeled components. See the data-pipeline contract for detail.
- **Charts don't match the MAA's headline numbers.** The trend data is off. The
  render step's sanity check catches this — fix `trend.csv` so the latest points
  equal this week's CPA and the 30-day average.
- **HTML loses its formatting on paste.** Paste only the white report card, and
  encode urgency with emoji and bold (not color). Drag the PNG charts in after.
