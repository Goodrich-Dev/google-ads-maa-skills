# Changelog

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
