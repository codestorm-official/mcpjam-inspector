# Deploy and Host MCPJam Inspector with Railway

MCPJam Inspector is an open-source platform for testing, inspecting, and evaluating MCP servers, MCP apps, and ChatGPT apps. This template runs a single self-hosted Inspector instance on Railway so a team can debug tools, prompts, resources, OAuth flows, and widget rendering over HTTPS without installing a desktop app or exposing a local port.

Short description (template marketplace): `Inspect, debug, and evaluate MCP servers with MCPJam on Railway.`

## About Hosting MCPJam Inspector

The Inspector is one stateless Node.js web service. It serves a browser UI and a local API on port `6274`, then connects to the MCP servers you configure from that UI. Access is protected by a private sign-in link generated at startup; a stable `MCPJAM_SESSION_TOKEN` keeps that link valid across restarts.

This template uses the official multi-architecture image `mcpjam/mcp-inspector:v3.24.0`. The image already bundles Node.js and the Chromium build needed for widget rendering, so there is no application build step, no custom runtime, and no database. Railway terminates TLS at the edge and forwards HTTP to port `6274`. A volume is not required: the template keeps one replica and relies on the browser session plus a fixed session token rather than local container state.

The Inspector can only reach MCP servers that are network-reachable from its container. Public HTTPS MCP servers work directly; servers in this Railway project should be addressed through their private network hostname. Local STDIO servers from your laptop are not reachable from this deployment.

## Common Use Cases

- Debug tools, resources, prompts, resource templates, and elicitation flows with full JSON-RPC traces.
- Inspect OAuth and DCR/CIMD exchanges step by step against a remote MCP server.
- Preview ChatGPT Apps SDK and MCP app widgets in a browser-based emulator.
- Run cross-client chat and evaluation runs against MCP servers hosted in the same Railway project.
- Give a distributed team one shared, HTTPS-accessible Inspector endpoint instead of per-laptop installs.

## Dependencies for MCPJam Inspector Hosting

- Official Docker image `mcpjam/mcp-inspector:v3.24.0` (amd64 and arm64).
- Node.js runtime and Chromium are included in the image; nothing is installed at deploy time.
- One public HTTP port (`6274`) and one Railway-generated domain.
- No PostgreSQL, Redis, MongoDB, object storage, or persistent volume is used by this template.
- MCP servers you want to inspect: public HTTP/S endpoints or services on the same Railway private network.

### Deployment Dependencies

- [MCPJam Inspector repository](https://github.com/MCPJam/inspector)
- [Official MCPJam Docker image on Docker Hub](https://hub.docker.com/r/mcpjam/mcp-inspector)
- [MCPJam documentation](https://docs.mcpjam.com/)
- [Railway template creation guide](https://docs.railway.com/templates/create)
- [Railway public networking](https://docs.railway.com/networking/public-networking)
- [Railway template variable functions](https://docs.railway.com/templates/create#template-variable-functions)

## Template Configuration

| Setting | Value |
| --- | --- |
| Services | 1 (`MCPJam Inspector`) |
| Source | Docker Image `mcpjam/mcp-inspector:v3.24.0` |
| Application port | `6274` (HTTP) |
| Public networking | Enabled, Railway domain |
| Health check | `GET /health` |
| Replicas | 1 |
| Restart policy | `ON_FAILURE`, max 5 retries |
| Volume | None |
| Template icon | `mcp_jam_2row.png` from the upstream repository (1:1) |

Exact composer steps are in [docs/RAILWAY_SETUP.md](docs/RAILWAY_SETUP.md).

## Environment Variables

| Variable | Required | Template value | Purpose |
| --- | --- | --- | --- |
| `MCPJAM_SESSION_TOKEN` | Yes | `${{secret(32, "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_")}}` | URL-safe secret of at least 24 characters. Keeps the private access link valid across restarts. Mark as secret/sealed. |
| `MCPJAM_ALLOWED_HOSTS` | Yes | `${{RAILWAY_PUBLIC_DOMAIN}}` | Comma-separated hostnames allowed to open the Inspector. Without the Railway hostname, requests are rejected with `HOST_NOT_ALLOWED`. |
| `MCPJAM_INSPECTOR_FRONTEND_URL` | Yes | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | Public browser origin used in printed links and origin validation. |
| `MCPJAM_LOCAL_COMPUTER_ENABLED` | Recommended | `false` | Disables model-driven local shell/computer tools inside the container. They are enabled by default in self-hosted builds. |
| `MCPJAM_LOCAL_BROWSER_ENABLED` | Recommended | `false` | Disables local agent browser tools inside the container. Set to `true` only after testing the features you need. |
| `MCPJAM_ALLOW_WILDCARD_ORIGINS` | Optional | `false` | Only needed if `MCPJAM_ALLOWED_HOSTS` contains a wildcard such as `*.example.com`. |

Do not set `PORT`. The image listens on `6274` by default.

## Networking and Ports

| Path | Exposure | Notes |
| --- | --- | --- |
| `:6274` HTTP | Public via Railway domain | UI, API, WebSocket, and `/health` |
| Railway private network | Internal only | Use `<service>.railway.internal:<port>` for MCP servers in the same project |

Railway's edge provides HTTPS. The container itself serves plain HTTP, which is the expected configuration. A TCP proxy is not needed. If you add a custom domain, add it to `MCPJAM_ALLOWED_HOSTS` as a comma-separated list and point `MCPJAM_INSPECTOR_FRONTEND_URL` at the domain users actually visit.

## Source Options

| Option | Advantages | Disadvantages | Recommended use |
| --- | --- | --- | --- |
| Docker Image `mcpjam/mcp-inspector:v3.24.0` | Fast, reproducible deploy; no build egress; pinned version | Version must be bumped manually; about 950 MB image pull | This template (default) |
| GitHub source using this repository | Image version pinned in code; easy to fork and audit | Requires a Docker build that pulls the same base image | Forks and GitOps workflows |
| GitHub source using upstream `MCPJam/inspector` | Always builds current `main` | Long monorepo build, Chromium download, npm and apt egress, unpinned commit | Upstream development only |

The `Dockerfile` and `railway.json` in this repository exist only for the GitHub-source variant. Railway does not read them when the template service uses the Docker Image source.

## Security Considerations

- The sign-in link is a credential. Treat `#token=...` from `railway logs` as a secret and rotate `MCPJAM_SESSION_TOKEN` if it leaks.
- Keep both local tool switches disabled unless a specific workflow needs them. A remote container has no trusted local machine, and model-driven shell access there is a real blast radius.
- One replica is intentional. Session state and local tools are instance-scoped; scaling out changes behavior and may break in-flight sessions.
- This is a development and evaluation tool, not a hardened production gateway. Restrict who can reach the public domain or put it behind your existing access layer.
- The published image contains upstream telemetry configuration pointing at MCPJam-operated endpoints. Review your organization's policy before using it with sensitive data.
- Never paste production credentials into prompts, traces, or MCP server configuration stored in the browser.

## Why Deploy MCPJam Inspector on Railway?

Railway gives the Inspector a stable HTTPS origin, a managed domain, health checks, and logs without running a VM. MCP servers deployed in the same Railway project are reachable over the free private network, so an entire MCP test stack can live in one project. Template variable functions generate the session secret at deploy time instead of shipping a hardcoded credential.

## Updates and Rollback

1. Check available tags on [Docker Hub](https://hub.docker.com/r/mcpjam/mcp-inspector/tags).
2. In the template composer, change the service image tag and redeploy (or edit the `FROM` line and push if using the GitHub-source variant).
3. Verify `GET /health` returns `200` and open the new private link from the logs.
4. To roll back, set the previous tag and redeploy. No data migration is involved.

## License

MCPJam Inspector is licensed under Apache License 2.0 by the MCPJam authors. This repository contains deployment configuration and documentation only; it is not affiliated with or endorsed by MCPJam.
