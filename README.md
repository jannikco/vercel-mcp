# Vercel project and deployment workflows for AI assistants

Review deployments and create an approved Vercel project. Read provider data first, then explicitly approve the selected change with operator write opt-in and a unique per-call approval key.

[Start locally](#get-started) · [Connect your assistant](#connect-your-assistant) · [Available tools](#available-tools) · [Hosted PipMCP setup](#use-pipmcp-for-production)

## Built for useful workflows

- **Build a Vercel brief.** List projects. Make that provider response available to your assistant before it proposes a next action.
- **Investigate a chosen Vercel result.** List deployments. Use an account or record returned by your provider rather than guessing an identifier.
- **Add a reviewed Vercel action.** Create project. Review the input first; execution requires operator opt-in and per-call approval.

## Why use this package?

Get a focused Vercel MCP integration without assembling request mappings and a stdio transport yourself. The package provides documented input schemas, provider authentication handling and an example CLI, with the same tools available to your assistant.

Reads are available independently of writes. Approved changes use operator opt-in and a per-call idempotency key; uncertain writes are not silently replayed.

## Get started

**Requirements:** Node.js 22 or newer, npm, and access to the provider account you intend to use.

```sh
git clone https://github.com/jannikco/vercel-mcp.git
cd vercel-mcp
npm ci --ignore-scripts
npm test
npm start
```

`npm start` shows the tool inventory and workflow offline. It does not authenticate or call the provider. The tests use provider fixtures and stdio transport; they do not modify your account.

### Configure your provider account

The current release connects directly to the provider with **your own credentials**. It does not require a PipMCP key for local execution. Configure the environment of the process or MCP client that runs the server; `.env` files are not loaded automatically.

| Environment variable | What to supply |
| --- | --- |
| `VERCEL_TOKEN` | Your own provider API key or access token. |

[Provider credential/reference documentation](https://openapi.vercel.sh). Keep credentials out of commits, issue reports and shared client configuration.

### Run your first provider read

After configuring the environment, run `list_projects`:

```sh
npm start -- --call list_projects '{"limit":"10"}'
```

Replace sample account, record, file and project IDs with values from your authorized account. This command makes a real provider request and returns its result for you to inspect.

## Available tools

| Capability | MCP tool | Effect | Required inputs |
| --- | --- | --- | --- |
| List projects. | `list_projects` | Read | None |
| List deployments. | `list_deployments` | Read | None |
| Create project. | `create_project` | Approved write | `name` |

Tool schemas and bindings are in [connector.json](./connector.json). These are the selected operations this package ships.

### A practical tool sequence

1. Use `list_projects` — List projects.
2. Use `list_deployments` — List deployments.
3. Use `create_project` — Create project.

## Connect your assistant

Add this stdio server to an MCP-compatible client. Replace the absolute path and credential placeholders. Use your client’s secret/environment controls instead of committing this configuration.

```json
{
  "mcpServers": {
    "vercel-mcp": {
      "command": "node",
      "args": [
        "/absolute/path/vercel-mcp/server.mjs"
      ],
      "env": {
        "VERCEL_TOKEN": "YOUR_VERCEL_TOKEN"
      }
    }
  }
}
```

`node server.mjs` starts the MCP server; `npm start` runs the example CLI. The server reserves stdout for MCP messages. If the client does not inherit your shell environment, pass the required variables explicitly.

## Review changes before running writes

The write tool is disabled by default. Enable `CONNECTOR_ALLOW_WRITES=true` in the server process and include this approval object in the **specific** write call after reviewing its arguments:

```json
{
  "_approval": {
    "approved": true,
    "idempotencyKey": "unique-reviewed-request-id"
  }
}
```

This object is added alongside the tool’s own required fields; it is not a complete write request. Use a new idempotency key for a different action. If an outcome is uncertain, inspect the provider before trying again. Creating a record may trigger existing provider automations even when this package does not send a message itself.

## Use PipMCP for production

The local package is useful when you want to run these selected Vercel operations in your own process. PipMCP provides a hosted place to configure provider connections, publish tools, issue endpoint access tokens, apply server-side policy and approvals, and inspect execution/usage records. Provider credentials and hosted execution are configured in that service.

**[Import Vercel into PipMCP](https://pipmcp.com/github/start?integration=openapi-to-mcp&utm_source=github&utm_campaign=github-vercel-mcp&utm_content=readme)**

This provider is not listed as a ready managed template in the current availability snapshot (2026-11-08). The link opens the OpenAPI importer; it does not activate the local tools automatically.

Use [selected-openapi.json](./selected-openapi.json) as a starting point for manual import. Configure the account hostname, authentication and provider-specific requirements, then review and publish the resulting tools in PipMCP.

Local credentials are not uploaded by this repository. A hosted endpoint uses its own PipMCP endpoint access token and published tool inventory; the local and hosted credentials are separate.

## Scope and validation

- The installed package has HTTP request fixtures and MCP stdio `tools/call` tests.
- Fixture success checks client behavior; it does not establish provider authentication, paid-plan access or live account execution.
- No live provider execution is claimed for this package by these tests.
- Hosted tool names, schemas, permissions and availability can differ from this local package. A queued or accepted job/message is not proof of completion or delivery. Provider API plans, account rules and limits apply.
- See [integration.json](./integration.json) for operation-scoped evidence and the provider-specific limits above.

## FAQ

**Do I need a Vercel account?**

You need the provider account, token and permissions required by these operations. Provider charges and API limits remain with that account.

**Does this version require a PipMCP API key?**

This package uses your provider credentials for local execution. To use the hosted service, configure the provider connection in PipMCP and use the endpoint access token issued there.

**Why is a tool missing or a request rejected?**

This repository exposes the selected tools above. Check the required environment variables, provider scopes, account/region requirements and input IDs first. Hosted endpoints have their own published inventory and may exclude tools by role or policy. Do not retry a failed write without checking its outcome.

**Is this an official provider SDK or a hosted-service guarantee?**

This is an independently maintained MCP integration. Its fixtures and source references do not imply provider endorsement, unrestricted API access, or identical hosted behavior.

## Guides and related workflows

- [Vercel integration guide](https://jannikco.github.io/pipmcp-connectors/integrations/vercel/)
- [Vercel workflow and tool sequence](https://jannikco.github.io/pipmcp-connectors/workflows/vercel/)
- [More integration workflows](https://jannikco.github.io/pipmcp-connectors/workflows/)
- [Typed LeadAdapter API client](https://github.com/jannikco/linkedin-typescript-sdk)
- [LeadAdapter Python API client](https://github.com/jannikco/linkedin-python-sdk)
- [LinkedIn API quickstart](https://github.com/jannikco/linkedin-api)
- [LinkedIn people and post search](https://github.com/jannikco/linkedin-search-api)
- [LinkedIn post engagers](https://github.com/jannikco/linkedin-post-engagers)
- [LeadAdapter LinkedIn MCP server](https://github.com/jannikco/linkedin-mcp)
- [OpenAPI to MCP import starter](https://github.com/jannikco/openapi-to-mcp)
- [Swagger to MCP import starter](https://github.com/jannikco/swagger-to-mcp)
- [Vercel API and credential reference](https://openapi.vercel.sh)

Maintained by [jannikco](https://github.com/jannikco). MIT for this repository’s code; dependencies retain their licenses. [Report a reproducible issue](https://github.com/jannikco/vercel-mcp/issues) with redacted input and error details. For credentials or security issues, use [private contact](https://pipmcp.com/contact).
