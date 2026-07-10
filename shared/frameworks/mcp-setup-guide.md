# Google Ads MCP Setup Guide

How to connect the analyzer's primary data source: the official **Google Ads
MCP server** (`googleads/google-ads-mcp`). Once connected, the analyzer pulls
every MAA dataset live via GAQL (`gaql-query-pack.md`) — correctly-labeled data,
no email plumbing, and mid-analysis follow-up pulls.

**What you get:** a read-only bridge to the Google Ads API with three tools:

- `list_accessible_customers` — the customer IDs and account names the
  authenticated user can reach
- `search` — executes GAQL queries (this is what the analyzer uses)
- `get_resource_metadata` — field/structure reference for a resource type

**Read-only is enforced by the server itself** (current release). It cannot
modify bids, pause campaigns, or create assets — which is exactly the boundary
this skill set wants. Account changes go through `google-ads-change-scripts`.

Official reference: Google's developer integration guide at
developers.google.com/google-ads/api/docs/developer-toolkit/mcp-server and the
github.com/googleads/google-ads-mcp repository.

---

## Prerequisites (one-time, per organization)

You need three credentials before configuring anything. All come from the
Google Ads API onboarding flow (developers.google.com/google-ads/api → Get
started):

1. **Developer token** — the 22-character string from your Google Ads MCC
   (API Center in the MCC settings). A test-access token works for test
   accounts; you need Basic access or higher for live client accounts.
2. **Google Cloud project ID** — any GCP project you control, with the Google
   Ads API enabled.
3. **OAuth credentials** — an OAuth2 Client ID/secret pair from that GCP
   project, or application default credentials. The Google account you
   authorize with must have access to the accounts you'll query (for an agency,
   that usually means a user on the MCC).

**Agency/MCC note:** authorize with a user on the MCC and every client account
under it becomes reachable. The analyzer passes the child account's CID as
`customer_id` on each query — record each client's CID (digits only, no dashes)
in your per-client notes so runs don't start with a lookup.

---

## Option 1 — Run it locally (stdio)

Best for a single operator on one machine. Add this to your MCP host's config
(Claude Code: `.mcp.json` in the project or `~/.claude.json`; other hosts:
consult their docs):

```json
{
  "mcpServers": {
    "google-ads-mcp": {
      "command": "pipx",
      "args": [
        "run",
        "--spec",
        "git+https://github.com/googleads/google-ads-mcp.git",
        "google-ads-mcp"
      ],
      "env": {
        "GOOGLE_PROJECT_ID": "YOUR_PROJECT_ID",
        "GOOGLE_ADS_DEVELOPER_TOKEN": "YOUR_DEVELOPER_TOKEN"
      }
    }
  }
}
```

Requires `pipx` installed. On first run it walks you through the OAuth consent
flow in a browser.

## Option 2 — Deploy to Cloud Run (shared / remote)

Best when several people or a Cowork/desktop environment need the same server,
or you want it reachable as a URL connector. Summary (full commands in Google's
guide):

1. Create an Artifact Registry repo and build the image with Cloud Build from
   the `google-ads-mcp` source.
2. Deploy to Cloud Run with these environment variables:
   `GOOGLE_PROJECT_ID`, `GOOGLE_ADS_DEVELOPER_TOKEN`,
   `GOOGLE_ADS_MCP_OAUTH_CLIENT_ID`, `GOOGLE_ADS_MCP_OAUTH_CLIENT_SECRET`,
   `GOOGLE_ADS_MCP_BASE_URL` (assigned by Cloud Run after first deploy),
   `FASTMCP_HOST=0.0.0.0`.
3. Point your MCP client at the deployed URL:

```json
{
  "mcpServers": {
    "google-ads-mcp": {
      "httpUrl": "https://your-cloud-run-url.a.run.app/mcp"
    }
  }
}
```

In Claude Cowork / desktop, add it as a custom connector with that URL and
complete the OAuth flow when prompted.

---

## Verify the connection (5 minutes)

1. Ask: *"What customers do I have access to?"* — should return your account
   list via `list_accessible_customers`.
2. Record each client's CID from that list into your per-client config.
3. Ask: *"Pull the 7-day campaign summary for {client} via the Google Ads MCP"*
   — the analyzer runs the campaign query from `gaql-query-pack.md`. Spot-check
   the numbers against the Google Ads UI. Remember cost comes back in micros
   (divide by 1,000,000) and rates as 0–1 fractions.

If a field name errors, use `get_resource_metadata` for that resource instead
of guessing.

---

## Known limitation: reauthorization

Depending on your OAuth app's publishing status and token handling, the
connection may require frequent — even daily — reauthorization. In practice
this is the main blocker to fully hands-off scheduled runs: if the `search`
tool is missing when a run starts, the authorization has lapsed.

Mitigations, in order of preference:

1. **Publish the OAuth app to production** (an app left in "Testing" status
   issues refresh tokens that expire after 7 days) and verify refresh-token
   persistence in your deployment.
2. **Keep the email pipeline wired as the fallback** (`WALKTHROUGH.md`, Path B)
   so a scheduled run degrades to the email data instead of stalling.
3. **Reauthorize before scheduled runs** if neither of the above is in place
   yet.

---

## What the analyzer expects once connected

- Queries per `gaql-query-pack.md`: campaign summary, keyword report + QS
  components, search terms, ad report, conversion detail — each in 7D and 30D
  windows.
- **QS components read AS-IS** on this path. The column swap in the
  data-pipeline contract is for the email fallback only.
- LSA campaigns will appear in pulls (`advertising_channel_type =
  LOCAL_SERVICES`); they are out of scope for the MAA — omit them.
- The MCP is never used to change the account. Changes ship as dry-run-first
  scripts via `google-ads-change-scripts`.
