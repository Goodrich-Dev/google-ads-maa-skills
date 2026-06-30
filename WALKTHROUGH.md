# Walkthrough — Running the Weekly MAA System End to End

This is the full operating loop: from a cold install to a client-ready report.
You do the **one-time setup** (steps 1-4) once per client account. After that the
**weekly loop** (steps 5-7) is what you repeat.

If you would rather not automate data collection, skip to "Manual mode" at the
bottom — you can run the whole thing from CSV exports.

---

## One-time setup

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

It will: find the latest data email, parse the ten datasets, diagnose the account
(the MAA framework), and where the data warrants it, dispatch the downstream
skills — `google-ads-copy-optimizer` for ad-copy fixes, `google-ads-change-scripts`
for negatives/pauses/restructures, `google-ads-lp-auditor` for landing-page
issues. It saves the MAA to `{OUTPUT_DIR}/{Client}/{Client}_MAA_{date}.md`.

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
| Once per account | Build + install + schedule the data script, set the Gmail label | `google-ads-script` |
| Weekly (auto) | Data email arrives | the deployed script |
| Weekly | Run the MAA | `google-ads-analyzer` (+ dispatched specialists) |
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

- **Analyzer can't find the data.** Confirm the feed email arrived and carries the
  `maa-data-automation` label (Step 3). Check the subject starts with `[MAA Data]`.
- **Quality Score components look swapped.** Some script versions invert the
  Expected CTR and Ad Relevance columns; the analyzer corrects this automatically.
  See the data-pipeline contract for detail.
- **Charts don't match the MAA's headline numbers.** The trend data is off. The
  render step's sanity check catches this — fix `trend.csv` so the latest points
  equal this week's CPA and the 30-day average.
- **HTML loses its formatting on paste.** Paste only the white report card, and
  encode urgency with emoji and bold (not color). Drag the PNG charts in after.
