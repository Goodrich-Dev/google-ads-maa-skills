# Changelog

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
