# OpenAxes IA Plugin

Integrates Claude with OpenAxes Investigation Assistant via MCP, enabling investigation management, legal hold operations, document search and review, and report generation — all from Cowork.

## Components

### MCP Server

Connects to the OpenAxes IA MCP endpoint at `https://ia.openaxes.com/mcp` over HTTPS.
Authentication uses OAuth 2.1 with PKCE — the first time Claude invokes a tool, your
browser opens to the OpenAxes IA consent page. Approve it once; Claude caches the
bearer token and uses it for all subsequent calls.

### Skills

- **Investigation Management** — List, view, create, close, and reopen investigations
- **Legal Hold** — Manage custodians, send hold notifications, apply and release endpoint holds
- **Search and Review** — Search investigation documents and apply review tags
- **Reports** — List, execute, and schedule reports; combine with Cowork document skills to create formatted deliverables

## Setup

Install the plugin via the OpenAxes Claude marketplace:

```
/plugin marketplace add OpenAxes/OpenAxes.Claude.Marketplace
/plugin install openaxes-ia@openaxes-plugins
```

No token configuration is required — Claude handles the OAuth flow automatically
when you first invoke a tool.

### Revoking access

To disconnect Claude from OpenAxes IA, revoke the MCP session token from your
OpenAxes IA profile at `/user/profile/mcp-tokens`. Claude will re-prompt for
authorization on the next tool call.

### Manual / air-gapped installs (advanced)

For environments where OAuth is not practical, a static token header can be
configured instead. Generate a token at `/user/profile/mcp-tokens` and set up
an `.mcp.json` with an `x-mcp-token` header. This is not the recommended path
for marketplace-installed plugins.

## Usage

Ask Claude things like:

- "Show me my active investigations"
- "Create a new investigation from the standard template"
- "List custodians on investigation X"
- "Search for documents mentioning 'contract renewal' in investigation X"
- "Run the investigation summary report and create a Word doc from it"
- "Schedule the compliance report to run every Monday at 9 AM"
