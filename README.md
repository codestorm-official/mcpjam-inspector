# Deploy and Host MCPJam Inspector with Railway

MCPJam Inspector is an open-source testing and evaluation platform for MCP server developers. This template gives a team one always-on, HTTPS-accessible Inspector instance so tools, resources, prompts, OAuth flows, and app widgets can be tested without installing a desktop app or opening a local tunnel.

Template marketplace description: `Inspect, debug, and evaluate MCP servers with MCPJam on Railway.`

## About Hosting MCPJam Inspector

The Inspector is a single stateless web service. It serves a browser UI and an API, then connects to the MCP servers you configure from that UI. Access is gated by a private sign-in link printed in the service logs; a stable session token keeps that link valid across restarts.

The deployment runs one pinned official image with no database, cache, queue, or persistent volume. Railway terminates TLS at the edge and forwards traffic to the service. One replica is intentional: the Inspector keeps instance-scoped session state and local tooling, so scaling out changes its behavior.

The Inspector can only reach MCP servers that are network-reachable from its container. Public HTTPS endpoints work directly, and servers deployed in the same Railway project can be reached over the private network.

## How to use the deployment

1. Deploy the template and wait for the health check to pass.
2. Open the service logs and copy the private **Open MCPJam** link containing `#token=...`.
3. Open the whole link in a browser. The token fragment signs the browser in; a bare domain URL only shows instructions.
4. Add an MCP server using an HTTP/S URL or a private-network hostname from the same Railway project.
5. Test tools, resources, prompts, elicitation, OAuth exchanges, and widgets from that single UI.

Share the access link only with people who should have full Inspector access. Rotate the session token if the link leaks.

## What works and what does not

| Scenario | Supported |
| --- | --- |
| Public HTTP/S and SSE MCP servers | ✅ |
| OAuth-protected servers, including DCR and CIMD flows | ✅ |
| Tools, resources, resource templates, prompts, and elicitation | ✅ |
| Full JSON-RPC traces and cross-client chat/evaluation views | ✅ |
| OpenAI Apps SDK and MCP app widget emulator | ✅ |
| MCP servers on the same Railway private network | ✅ |
| STDIO servers from your laptop | ❌ |
| `localhost` MCP servers | ❌ |
| Local filesystem skills and desktop-only integrations | ❌ |

## Comparison with alternatives

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

Sources for this comparison: the [MCPJam repository](https://github.com/MCPJam/inspector) and the [official MCP Inspector repository](https://github.com/modelcontextprotocol/inspector). The hosted app is vendor-operated; local installs require Node.js for the terminal option or a desktop installer.

## Security considerations

- The access link is the credential. Anyone holding it can inspect and call tools on servers configured in that instance.
- Local shell and browser tools are disabled in this deployment. They are not needed for remote MCP inspection and would add unnecessary blast radius on a public URL.
- This is a development and evaluation tool, not a hardened production gateway. Restrict who can reach the domain or place it behind your existing access layer.
- Do not paste production credentials into prompts, traces, or server configurations stored in the browser.
- The upstream image contains telemetry configuration pointing at MCPJam-operated endpoints. Review internal policy before using it with sensitive data.

## Why deploy MCPJam Inspector on Railway?

Railway supplies a stable HTTPS origin, a managed domain, health checks, and logs without a VM. MCP servers in the same project are reachable over the private network, so a test stack can live in one workspace. The pinned image makes the deployment reproducible and rollback is a version change, not a data migration.

## License

MCPJam Inspector is licensed under Apache License 2.0 by the MCPJam authors. This repository contains deployment packaging and documentation only; it is not affiliated with or endorsed by MCPJam.

