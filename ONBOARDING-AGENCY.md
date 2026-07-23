# Agency Onboarding — Cowork + Google Ads MCP + the MAA Skill Suite

A setup checklist for an agency adopting this suite in **Claude Cowork**, with
their own Google Ads MCC. No hosted/managed-agent component is involved: the
stack is just (1) the read-only Google Ads MCP as the data source, (2) these
six skills installed as a plugin, and (3) a synced folder for outputs and
client continuity. Budget ~half a day for the one-time setup, then ~10 minutes
per client.

---

## Phase 0 — Credentials (yours, not ours)

Everything authenticates against **your own** Google Ads MCC and Google Cloud
project. Nothing is shared with the publisher of this repo. You need:

1. **Google Ads developer token** — from API Center in your MCC settings.
   Basic access or higher for live client accounts (test tokens only reach
   test accounts).
2. **A Google Cloud project** with the Google Ads API enabled.
3. **OAuth client ID/secret** from that project. Authorize with a user on your
   MCC so every child account is reachable.

⚠️ **Publish the OAuth app to production before going live.** An app left in
"Testing" status issues refresh tokens that expire after 7 days — this is the
single most common cause of "the MCP stopped working," and it presents as
daily reauthorization. See the reauthorization section of
`shared/frameworks/mcp-setup-guide.md`.

## Phase 1 — Stand up the Google Ads MCP

Full detail in `shared/frameworks/mcp-setup-guide.md`. For Cowork specifically:

- **Recommended: deploy to Cloud Run and add it as a custom connector**
  (Settings → Connectors → Add custom connector → your Cloud Run URL). Cowork
  sessions run in the cloud, so a remote URL connector works everywhere — any
  team member, any device, scheduled runs included.
- **Alternative: run it locally** via the desktop app's MCP config. Works, but
  the connector is only available while that desktop is online, which rules
  out unattended scheduled runs.

Verify (5 min): ask Claude *"What customers do I have access to?"* — you
should get your MCC's account list back. Then spot-check one campaign summary
pull against the Google Ads UI (cost arrives in micros; divide by 1,000,000).

## Phase 2 — Install the skill suite

```
/plugin marketplace add Goodrich-Dev/google-ads-maa-skills
/plugin install google-ads-maa-skills
```

Or install the packaged release zip from the repo's Releases page. Confirm the
six skills appear (analyzer, copy-optimizer, change-scripts, lp-auditor,
script, client-view). The suite is MIT-licensed; forking your own copy is fine
and lets you pin versions — just pull upstream updates deliberately.

## Phase 3 — Workspace and output configuration

1. Connect a synced folder to Cowork sessions (Google Drive, Dropbox, or a
   local folder via the desktop app) and tell Claude once per session — or put
   it in a memory/instructions file — where deliverables go. That location is
   `{OUTPUT_DIR}` (see `CONFIGURATION.md`).
2. Pick a continuity mode per `CONFIGURATION.md`: the simple
   `{Client}_MAA-Narrative.md` file (zero structure, fine for most agencies
   starting out) or a structured per-client knowledge base. Start simple; you
   can graduate later.
3. If your agency has house style rules (report header, formatting, naming),
   write them into a single overlay file in your workspace and tell Claude it
   overrides the skill defaults. Don't fork the skills to change style.

## Phase 4 — Per-client onboarding (~10 min each)

1. From the Phase 1 verification list, record the client's **CID** (digits
   only) wherever you keep per-client notes.
2. Run the first MAA: *"Do a first-run MAA for {client}, CID {digits}, via the
   Google Ads MCP."* The analyzer pulls 7D/30D campaign, keyword/QS, search
   term, ad, and conversion data live, and the first report stands on its own.
3. Answer the onboarding gaps the first run will surface — it will ask rather
   than guess: client contact names/roles, target cost per lead, service
   area, and whether competitor-name traffic is wanted or should be negated.
   Feed the answers into the continuity file so week 2 has memory.

No weekly email script is required on the MCP path. Wire the email pipeline
(`WALKTHROUGH.md`, Path B) only if you want a fallback for scheduled runs —
and note the QS column-swap corrector applies **only** to that email path.

## Phase 5 — Weekly cadence

Use Cowork **scheduled tasks** for the weekly run — no managed agent needed.
Each firing starts a fresh session, so the prompt must be standalone:

> Weekly Google Ads MAA for {client}. CID {digits}. Pull data via the Google
> Ads MCP. Read {continuity file/folder path} first. Write the MAA to
> {OUTPUT_DIR}/{client}/, then render the client view. Draft only — do not
> send anything to the client.

Two operating rules worth enforcing from day one:

- **The MCP is read-only and stays that way.** Account changes ship as
  dry-run-first Google Ads Scripts via `google-ads-change-scripts`, reviewed
  by a human before running live.
- **Every deliverable is a draft.** The suite never posts to the client, your
  PM tool, or email on its own.

## Troubleshooting quick hits

| Symptom | Likely cause | Fix |
|---|---|---|
| MCP tools missing at session start | OAuth lapsed | Reauthorize; publish OAuth app to production (Phase 0) |
| "Unrecognized field" GAQL errors | API version drift | Use `get_resource_metadata` for the resource; update `gaql-query-pack.md` |
| Zero-spend clutter in reports | Dormant campaigns included | They should be omitted entirely — see the data contract |
| LSA campaign showing up in the MAA | LSA surfaced by the MCP | Out of scope by rule; omit (separate deliverable if wanted) |
| QS components look wrong | Swap applied to MCP data | Never swap MCP data; the corrector is email-path only |
