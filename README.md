# Deploy and Host MCPJam Inspector on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/mcpjam?referralCode=indah&utm_medium=integration&utm_source=template&utm_campaign=generic)

MCPJam Inspector is an open-source testing and evaluation platform for MCP server developers. Inspect tools, resources, prompts, and OAuth exchanges with full JSON-RPC traces, run cross-client evaluations, and preview MCP and ChatGPT app widgets — all from a browser, with no desktop app or local tunnel.

## About Hosting MCPJam Inspector

Hosting turns the Inspector into one always-on web service. It serves a browser UI and API, then connects out to the MCP servers you configure from that UI. Access is gated by a private sign-in link printed in the service logs; a fixed session token keeps that link valid across restarts. The deployment is a single stateless service — no database, cache, or persistent volume — behind Railway-managed HTTPS on one public domain. One replica is intentional: session state and local tooling are instance-scoped. The Inspector only reaches MCP servers that are network-reachable from its container: public HTTPS endpoints directly, and same-project servers over Railway's private network.

## Common Use Cases

- Debug tools, resources, resource templates, prompts, and elicitation flows with full JSON-RPC traces.
- Step through OAuth, DCR, and CIMD exchanges against a remote MCP server.
- Preview OpenAI Apps SDK and MCP app widgets in the built-in emulator.
- Give a distributed team one shared, HTTPS-accessible Inspector URL instead of per-laptop installs.
- Test MCP servers deployed in the same Railway project over the private network.

## Dependencies for MCPJam Inspector Hosting

- Official multi-architecture Docker image `mcpjam/mcp-inspector:v3.24.0`; Node.js and Chromium are bundled, nothing is installed at deploy time.
- One public HTTP service with a Railway-generated domain; no database, cache, queue, object storage, or persistent volume.
- MCP servers to inspect: public HTTP/S endpoints, or services reachable in the same Railway project.

### Implementation Details

Sign-in works through a private link, not a login form. After deployment, copy the full **Open MCPJam** link (including the `#token=...` fragment) from the service logs and open it in a browser; a bare domain URL only shows instructions. Treat that link as a credential and rotate the session token if it leaks.

The Inspector can only reach MCP servers that are network-reachable from its container:

| Scenario | Supported |
| --- | --- |
| Public HTTP/S and SSE MCP servers | ✅ |
| OAuth-protected servers, including DCR and CIMD flows | ✅ |
| Tools, resources, resource templates, prompts, elicitation | ✅ |
| Full JSON-RPC traces and cross-client chat/evaluation views | ✅ |
| OpenAI Apps SDK and MCP app widget emulator | ✅ |
| MCP servers on the same Railway private network | ✅ |
| STDIO servers from your laptop | ❌ |
| `localhost` MCP servers | ❌ |
| Local filesystem skills and desktop-only integrations | ❌ |

## Comparison with Alternatives

✅ = available as a built-in part of the workflow, ❌ = not part of that option's default workflow.

| Capability | This template (Railway) | MCPJam Hosted app | MCPJam local (npx / desktop) | Official MCP Inspector |
| --- | --- | --- | --- | --- |
| Browser-only, no local install | ✅ | ✅ | ❌ | ❌ |
| Always-on public HTTPS URL | ✅ | ✅ | ❌ | ❌ |
| One shareable URL for a team | ✅ | ✅ | ❌ | ❌ |
| Pinned, reproducible version | ✅ | ❌ | ❌ | ❌ |
| Runs inside your own infrastructure boundary | ✅ | ❌ | ✅ | ✅ |
| Reaches private services in the same project/network | ✅ | ❌ | ❌ | ❌ |
| Local STDIO MCP servers | ❌ | ❌ | ✅ | ✅ |
| Cross-client evaluations and widget emulator | ✅ | ✅ | ✅ | ❌ |
| Guided OAuth conformance debugger | ✅ | ✅ | ✅ | ❌ |
| No infrastructure to maintain | ❌ | ✅ | ✅ | ✅ |

Comparison based on the [MCPJam repository](https://github.com/MCPJam/inspector) and the [official MCP Inspector repository](https://github.com/modelcontextprotocol/inspector). The hosted app is vendor-operated; local installs require Node.js for the terminal option or a desktop installer.

## Security Considerations

- The access link is the credential. Anyone holding it can inspect and call tools on servers configured in that instance.
- Local shell and browser tools are disabled in this deployment; they are not needed for remote MCP inspection and would add unnecessary blast radius on a public URL.
- This is a development and evaluation tool, not a hardened production gateway. Restrict who can reach the domain or place it behind your existing access layer.
- Do not paste production credentials into prompts, traces, or server configurations stored in the browser.
- The upstream image contains telemetry configuration pointing at MCPJam-operated endpoints. Review internal policy before using it with sensitive data.

## Why Deploy MCPJam Inspector on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying MCPJam Inspector on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.

MCP servers in the same project are reachable over the private network, so an entire MCP test stack can live in one workspace, and the pinned image makes the deployment reproducible.

## License

MCPJam Inspector is licensed under Apache License 2.0 by the MCPJam authors. This repository contains deployment packaging and documentation only; it is not affiliated with or endorsed by MCPJam.
