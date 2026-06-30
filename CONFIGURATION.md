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

The analyzer expects Google Ads data in the format described in
`shared/frameworks/data-pipeline-contract.md`. The two normal ways to feed it:

1. **Automated weekly email** (recommended). Use `google-ads-script` to build a
   collection script that runs in the account and emails the 10 datasets each
   week. Point the analyzer at whatever inbox/label receives it.
2. **Manual exports.** Paste or upload CSV exports (campaign summary, keyword
   report with Quality Score, search terms). The analyzer auto-detects what you
   provide and tells you what's missing.

If your collection script has the known Quality Score column-swap bug, the
analyzer corrects it automatically — see the data-pipeline contract for detail.

## Optional: BlitzBase / vault continuity

If you keep a per-client knowledge base, create a narrative file named
`{Client}_MAA-Narrative.md` in the client's output folder. When it exists, the
analyzer reads it before writing (to follow up on open threads and avoid
repeating context) and appends a dated log entry after each weekly MAA. Without
it, every MAA simply stands on its own. No setup required either way.
