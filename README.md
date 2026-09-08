# client-mcp-plugins

Plugin manifests and skills that let Claude and OpenAI clients use the Flexport MCP server ([`client-mcp`](https://github.com/flexport/client-mcp), private) for shipment tracking and freight rate/booking workflows.

This repo contains only manifests and workflow documentation ("skills") — the MCP server implementation lives in the private `client-mcp` repo and is not affected by this repo's visibility.

## Structure

- `server.json` — generic MCP server registration (per the [MCP server schema](https://static.modelcontextprotocol.io/schemas/2025-09-29/server.schema.json)).
- `.mcp.json` — Claude Desktop/Code local server registration.
- `.claude-plugin/plugin.json` — Claude Code plugin manifest.
- `skills/` — standalone-value capabilities: shipment tracking, instant price quoting, quote-status lookup.
- `workflow-skills/` — the write operations that require a specific multi-step call sequence: requesting a custom rate, booking an instant rate, and booking without a rate.

Skills only cover tools and tool clusters that provide value on their own — pure lookup helpers (port/address/HS-code/company-entity search) are documented as steps inside the relevant workflow-skill instead of getting a standalone skill.

## Contributing

Changes go through a pull request; only `emerald` and `commerce-foundation` team members can merge to `main`. External pull requests aren't accepted — please open an issue instead.
