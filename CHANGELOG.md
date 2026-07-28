# Changelog

## 1.3.0 — 2026-07-23

Fit-and-finish release driven by a cold-start review: a fresh session with no
prior context attempted to guide setup from the docs alone. Everything that
tripped it is fixed here.

Docs — blocking fixes:
- `README.md`: install command had a literal `<your-github-username>`
  placeholder; now reads `Goodrich-Dev/google-ads-maa-skills`. Added a
  Requirements line (Python 3.x + matplotlib for charts, pipx for local stdio
  MCP) and updated the continuity paragraph to describe both auto-detected
  modes.

Docs — setup depth (`shared/frameworks/mcp-setup-guide.md`):
- Prerequisites expanded into a "Before you start" walkthrough: where the
  developer token lives, a dedicated test-vs-Basic access callout (apply on
  day one; manual approval lag), enabling the Google Ads API in GCP, OAuth
  client type per deployment (Desktop for stdio, Web for Cloud Run), and
  publishing the OAuth consent screen to Production moved up from "Known
  limitation" into setup — doing it up front prevents the 7-day refresh-token
  expiry instead of treating it as a later fix.
- Cloud Run option now includes the concrete `gcloud` sequence, with the
  two-pass deploy called out explicitly (`GOOGLE_ADS_MCP_BASE_URL` is assigned
  by Cloud Run after the first deploy, so deploy → read URL → update).

Troubleshooting (`WALKTHROUGH.md`):
- New first entry: what a test-access token failing against a live account
  looks like, and the fix (request Basic access).
- Reauthorization entry now distinguishes lapsed authorization from
  misconfiguration, including the Testing-status OAuth app masquerading as
  constant lapses.

Hygiene:
- Added `.gitignore` (output/, credentials patterns, Python and OS noise) —
  `{OUTPUT_DIR}` defaults to `./output`, which previously landed client MAAs in
  the repo working tree with nothing preventing a commit.
- Added `requirements.txt` (matplotlib, optional).
- Added `shared/scripts/README.md` documenting `correct_qs_columns.py` usage
  and the never-on-MCP-data warning.

## 1.2.0 — 2026-07-23

Continuity system upgraded from a single narrative file to an optional
structured knowledge base, based on a live first-run test against a brand-new
client account (no prior setup) that surfaced where the skills and a structured
vault disagree.

Continuity:
- `CONFIGURATION.md`: "BlitzBase / vault continuity" section rewritten as
  **two auto-detected continuity modes** — simple mode (the existing
  `{Client}_MAA-Narrative.md`, unchanged, still the zero-setup default) and
  **knowledge-base mode** (a `compiled/` wiki of living status docs:
  `maa-metric-spec.md`, `paid-status.md`, `active-issues.md`,
  `data-sources.md`, `trend.csv`, plus `raw/maa-reports/` as the delta and
  voice source). When `compiled/` exists the analyzer reads and maintains it
  instead of the narrative file.
- First-run bootstrap rule: if `maa-metric-spec.md` is missing in
  knowledge-base mode, the analyzer creates it before writing the report —
  from the most recent archived MAA, or from the first run's own structure for
  a brand-new client.
- Overlay contract documented: a vault-level
  `clients/_shared/maa-skills-binding.md` file may override any generic skill
  convention (naming, section ordering, house style). Overlay wins over skill
  defaults. This lets one public skill codebase serve differently-structured
  private vaults without forking.

Analyzer:
- Narrative-file instructions scoped to simple mode only; knowledge-base mode
  reads `client-knowledge-base.md` → `compiled/` → last archived MAA, and the
  wrap-up updates `paid-status.md` / `active-issues.md` / `trend.csv` instead
  of appending a narrative log entry.

Notes from the first-run test that motivated this release:
- A cold account with no prior setup produced a structurally sound MAA from
  the skill alone (MCP path, QS components correctly read without the swap,
  LSA correctly excluded), confirming the skills stand alone.
- Everything that broke was continuity-layer, not analysis-layer: file naming,
  narrative vs. wiki, house style rules the skill had no way to know. Hence
  the overlay contract rather than more skill body text.

Packaging & distribution:
- Restored the missing `.claude-plugin/` folder (`plugin.json` +
  `marketplace.json`) — the documented `/plugin marketplace add` install path
  did not work without it.
- Added `ONBOARDING-AGENCY.md` — a Cowork-focused setup checklist for a
  third-party agency adopting the suite with their own MCC, credentials, and
  Google Ads MCP deployment (Cloud Run connector recommended; no
  managed-agent component). Includes weekly scheduled-run prompt template and
  a troubleshooting table.


## 1.1.0 — 2026-07-10

Synced from the live production version. The operating system these skills
mirror runs weekly on real client accounts; improvements found in those runs
flow here. Headline change: **the Google Ads MCP is now the primary data
source**, with the email pipeline as the fallback.

Data pipeline:
- Added `shared/frameworks/mcp-setup-guide.md` — full setup guide for the
  official Google Ads MCP server (prerequisites, local + Cloud Run deployment,
  verification, reauthorization caveat).
- Added `shared/frameworks/gaql-query-pack.md` — the GAQL queries that
  reproduce every MAA dataset live from the API, with unit conversions and the
  QS component field mapping.
- `data-pipeline-contract.md` rewritten around source priority: MCP first
  (read-only, correctly-labeled QS components, no swap), email fallback second,
  manual exports third.
- Added `shared/scripts/correct_qs_columns.py` — deterministic corrector for
  the email path's QS column inversion. The swap is now mechanical, scoped to
  the email path only, with a per-client anchor sanity check.
- LSA campaigns explicitly out of scope (they surface on the MCP path; omit
  them like dormant campaigns).

Analyzer:
- New diagnostic gates from live runs: Structure Before Tactics (dependency
  ordering of recommendations), Verify Structure From the Data (never infer
  targeting from names), Label Inferences as Inferences.
- Action section: one list with inline owner tags (no per-person sub-sections),
  no padding, no non-actions, no method notes in the report body.
- Mid-cycle data pulls: when the analysis raises a question, pull the extra
  data in the same cycle and turn it into a grounded action.

Client view:
- Confident bottom line: the above-the-fold summary never ends on a caveat;
  hedges move to the 🟡 section.
- 🟡 vs 🔴 split formalized; arrangement-type watch items framed as the shared
  plan, not our deficiency.
- "Render" always routes to this skill's templates, never an ad-hoc mockup.
- "What we need from you" carries only genuine asks; resolved asks drop off.

Docs:
- README, WALKTHROUGH (Path A = MCP, Path B = email), and CONFIGURATION updated
  for the two-path data setup; roadmap section added (persistent MCP
  authorization is the known blocker to fully hands-off scheduled runs).

## 1.0.0 — 2026-06-30

First public release. Packaged six production Google Ads skills for
distribution:

- google-ads-analyzer (the quarterback: MAA framework)
- google-ads-copy-optimizer
- google-ads-change-scripts
- google-ads-lp-auditor
- google-ads-script
- google-ads-client-view (final render step: client-facing report + trend charts)

Changes made for distribution:
- Removed personal/workspace-specific output paths; output location is now
  configurable (see CONFIGURATION.md).
- Removed references to private vault/knowledge-base paths. The MAA methodology
  and data-pipeline contract are bundled under `shared/frameworks/` so the
  skills work standalone.
- Added a full anonymized worked example under `shared/examples/`: one client
  (Summit Dumpster Rental, week of 2026-06-26) with a real MAA and its real
  production client-view render (markdown + Basecamp HTML + two trend charts +
  the trend.csv input), plus sample ad-copy and change-script outputs. Client,
  people, locations, account IDs, and competitor names changed; numbers and
  reasoning preserved.
- Internal vault-integrated test variants of these skills are intentionally
  excluded from this package.
