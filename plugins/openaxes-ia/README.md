# OpenAxes IA Plugin

Integrates Claude with OpenAxes Investigation Assistant via MCP, enabling investigation management, legal hold operations, document search and review, and report generation — all from Cowork.

## Components

### MCP Server

Connects to the OpenAxes IA MCP endpoint over HTTP with token authentication.

### Skills

- **Investigation Management** — List, view, create, close, and reopen investigations
- **Legal Hold** — Manage custodians, send hold notifications, apply and release endpoint holds
- **Search and Review** — Search investigation documents and apply review tags
- **Reports** — List, execute, and schedule reports; combine with Cowork document skills to create formatted deliverables

## Setup

The MCP token is configured in `.mcp.json`. To rotate the token:

1. Generate a new MCP token from your IA profile at `/user/profile/mcp-tokens`
2. Update the `x-mcp-token` header value in `.mcp.json`
3. Revoke the old token

## Usage

Ask Claude things like:

- "Show me my active investigations"
- "Create a new investigation from the standard template"
- "List custodians on investigation X"
- "Search for documents mentioning 'contract renewal' in investigation X"
- "Run the investigation summary report and create a Word doc from it"
- "Schedule the compliance report to run every Monday at 9 AM"
