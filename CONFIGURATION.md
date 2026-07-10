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

## Optional: BlitzBase / vault continuity

If you keep a per-client knowledge base, create a narrative file named
`{Client}_MAA-Narrative.md` in the client's output folder. When it exists, the
analyzer reads it before writing (to follow up on open threads and avoid
repeating context) and appends a dated log entry after each weekly MAA. Without
it, every MAA simply stands on its own. No setup required either way.
