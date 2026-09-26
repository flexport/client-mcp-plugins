# Flexport MCP plugin

Flexport's MCP (Model Context Protocol) server lets AI assistants and agentic tools connect to your
[Flexport](https://www.flexport.com) account to act on your behalf: looking up shipments, checking
rates, searching your network, and booking freight, using natural language.

This plugin adds Flexport's official hosted MCP server. It does not install or execute a local
binary. On first connection, your client opens Flexport sign-in in the browser; no API key or
environment variable is required.

## Prerequisites

- An active Flexport account.
- **MCP enabled for your organization by an admin** at
  [app.flexport.com/integrations/mcp-connection](https://app.flexport.com/integrations/mcp-connection).
  If you connect and see an error that MCP has not been enabled, an admin needs to turn it on there.

## Installation

Install **Flexport** from your AI client's plugin marketplace, such as the
[Cursor Marketplace](https://cursor.com/marketplace).

For clients without a Flexport plugin, add the server manually if the client supports remote
(Streamable HTTP) MCP servers:

```json
{
  "mcpServers": {
    "flexport": {
      "url": "https://mcp.flexport.com/mcp"
    }
  }
}
```

Connections are initiated from the agent and tied to your individual Flexport account and user role.
When you're redirected, sign in with your Flexport account to authorize — the whole process takes
about a minute.

## Documentation

- [Overview](https://apidocs.flexport.com/2023-07-01/tag/Overview/)
- [Setup](https://apidocs.flexport.com/2023-07-01/tag/Setup/)
- [Permissions](https://apidocs.flexport.com/2023-07-01/tag/Permissions/)
- [Tools](https://apidocs.flexport.com/2023-07-01/tag/MCP-Tools/)

## Support and resources

- [Flexport](https://www.flexport.com)
- [Flexport Privacy Policy](https://www.flexport.com/privacy/)
- [Flexport Software Visibility Terms and Conditions](https://www.flexport.com/terms-and-conditions/software-visibility-terms-and-conditions/)
- Support: contact your Flexport account team

## Contributing

External pull requests aren't accepted — please open an issue instead.

## License

The contents of this repository are licensed under the [Apache License 2.0](LICENSE). Use of the
hosted Flexport MCP service is governed separately by Flexport's
[Software Visibility Terms and Conditions](https://www.flexport.com/terms-and-conditions/software-visibility-terms-and-conditions/).
