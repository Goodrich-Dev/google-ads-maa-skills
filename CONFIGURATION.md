# Configuration

These skills run with zero configuration. The two things you may want to set are
the **output location** and the **data source**.

## Output location (`{OUTPUT_DIR}`)

The analyzer and copy optimizer save deliverables to:

```
{OUTPUT_DIR}/{Client Name}/{Client}_MAA_{YYYY-MM-DD}.md
{OUTPUT_DIR}/{Client Name}/{Client}_AdCopy_{YYYY-MM-DD}.md
```

`{OUTPUT_DIR}` defaults to `./output`. To change it, just tell Claude where you
want files saved (e.g. "save MAAs under my Clients folder"), or set it once at
the top of a session. It can point at:

- a local project folder,
- a synced drive folder,
- a per-client folder in a knowledge base / vault (see below).

The per-client subfolder is created automatically if it doesn't exist.

## Data source

Three ways to feed the analyzer, in priority order (full rules in
`shared/frameworks/data-pipeline-contract.md`):

1. **Google Ads MCP** (recommended). Connect a Google Ads MCP server and record
   each client's CID. The analyzer pulls every dataset live via GAQL
   (`shared/frameworks/gaql-query-pack.md`) — correctly-labeled data, no email
   plumbing, and mid-analysis follow-up pulls. The MCP is read-only; account
   changes still go through change scripts.
2. **Automated weekly email** (fallback / backup for scheduled runs). Use
   `google-ads-script` to build a collection script that runs in the account and
   emails the 10 datasets each week. Point the analyzer at whatever inbox/label
   receives it.
3. **Manual exports.** Paste or upload CSV exports (campaign summary, keyword
   report with Quality Score, search terms). The analyzer auto-detects what you
   provide and tells you what's missing.

If your collection script has the known Quality Score column-swap bug (email
path only), run `shared/scripts/correct_qs_columns.py` on the raw data at
ingest — see the data-pipeline contract for detail. Never apply the swap to MCP
data.

## Optional: client continuity

Without any setup, every MAA stands on its own. To give the analyzer memory
across weeks, pick one of two modes — it auto-detects which one you're using by
what exists in the client's folder.

### Simple mode: a narrative file

Create `{Client}_MAA-Narrative.md` in the client's output folder. When it
exists, the analyzer reads it before writing (to follow up on open threads and
avoid repeating context) and appends a dated log entry after each weekly MAA.
One file, zero structure to maintain. Best for a single operator tracking a
handful of accounts.

### Knowledge-base mode: a compiled wiki

If you run a structured per-client knowledge base (BlitzBase-style or your own),
replace the single narrative file with living status docs the analyzer reads
and maintains. The convention:

```
clients/{client}/
├── client-knowledge-base.md      # index: who they are, contacts, engagement
├── compiled/                     # the living wiki (analyzer reads AND updates)
│   ├── maa-metric-spec.md        # this client's section order and metric names
│   ├── paid-status.md            # live QS state, incl. the per-client QS anchor
│   ├── active-issues.md          # open problems, feeds Analysis and Action
│   ├── data-sources.md           # CID, data path, corrected-script status
│   └── trend.csv                 # week,leads,cpl_week,cpl_30d (client-view input)
├── deliverables/                 # dated outputs (MAAs, client views, scripts)
└── raw/                          # append-only source material
    ├── google-ads/               # raw pulls / email exports
    └── maa-reports/              # archived past MAAs (delta + voice source)
```

When a `compiled/` folder exists, the analyzer treats it as the source of truth
and does NOT create or update a narrative file: it reads the knowledge base and
last archived MAA before writing, and its wrap-up updates `paid-status.md`,
`active-issues.md`, and `trend.csv` instead of appending a narrative log entry.
If `maa-metric-spec.md` is missing, the analyzer creates it on the first run
from the most recent archived MAA (or from the first run's own structure for a
brand-new client) before writing the report.

A vault-level overlay file (e.g. `clients/_shared/maa-skills-binding.md`) may
override any generic convention in these skills — file naming, section ordering,
house style. If the client's knowledge base contains one, it wins over this
document and over the skill defaults.
