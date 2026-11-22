# Railway Template Setup: MCPJam Inspector

Railway templates are created in the Railway UI, not from a manifest file in the repository. This guide records the exact composer settings to enter.

## Source used by this template

This template uses **this repository as the GitHub source**. The repository contains a one-line `Dockerfile` that pins the official image `mcpjam/mcp-inspector:v3.24.0`, plus a `railway.json` that selects the Dockerfile builder and applies the deploy settings.

Railway reads `railway.json` and `Dockerfile` only when the service source is GitHub. Alternatively, the same image can be entered directly in the **Docker Image** source field — that path skips this repository entirely. Use **one** of them, never both. This guide follows the GitHub-source path.

## 1. Push the repository and create the template

1. Push this folder to GitHub, for example `https://github.com/<your-org>/mcpjam-inspector-railway`. The repository can be public or private if Railway has GitHub access to it.
2. Open [Railway Templates](https://railway.com/workspace/templates) in your workspace.
3. Click **New Template** → **Start from scratch**.
4. Name: `MCPJam Inspector`.
5. Icon URL (1:1 PNG from the upstream repository):

   ```text
   https://raw.githubusercontent.com/MCPJam/inspector/main/mcpjam-inspector/client/public/mcp_jam_2row.png
   ```

6. Description (under 75 characters):

   ```text
   Inspect, debug, and evaluate MCP servers with MCPJam on Railway.
   ```

## 2. Add the service from GitHub

1. Click **Add New** → **GitHub Repo**.
2. Select `mcpjam-inspector-railway` and authorize the GitHub scope if asked.
3. Branch: `main` (or the branch you pushed).
4. Root directory: `/` (repository root).
5. Service name: `MCPJam Inspector`.
6. Leave the build command and start command empty. `railway.json` sets `builder: DOCKERFILE` and `dockerfilePath: Dockerfile`; the image entrypoint is `node bin/start.js`.

## 3. Variables tab

Paste the following values exactly. Descriptions are shown to template users; keep them.

| Variable | Value | Description to enter | Mark as secret |
| --- | --- | --- | --- |
| `MCPJAM_SESSION_TOKEN` | `${{secret(32, "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_")}}` | URL-safe session secret (32 chars). Keeps the private access link stable across restarts. Rotate if exposed. | Yes |
| `MCPJAM_ALLOWED_HOSTS` | `${{RAILWAY_PUBLIC_DOMAIN}}` | Hostname allowed to open the Inspector. Add custom domains as a comma-separated list. | No |
| `MCPJAM_INSPECTOR_FRONTEND_URL` | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | Public browser origin used for access links and origin checks. | No |
| `MCPJAM_LOCAL_COMPUTER_ENABLED` | `false` | Disables model-driven shell and computer tools inside the container. | No |
| `MCPJAM_LOCAL_BROWSER_ENABLED` | `false` | Disables local agent browser tools inside the container. | No |
| `MCPJAM_ALLOW_WILDCARD_ORIGINS` | `false` | Only enable if the allowlist uses a wildcard domain. | No |

Do not add `PORT`. The image defaults to `6274`.

## 4. Settings tab

| Setting | Value |
| --- | --- |
| Public networking | Enabled, HTTP, auto-generated Railway domain |
| Port | `6274` |
| Healthcheck path | `/health` |
| Healthcheck timeout | `300` seconds (allows for a cold image pull) |
| Start command | Leave empty (image entrypoint is `node bin/start.js`) |
| Root directory | `/` (repository root) |
| Replicas | `1` |
| Restart policy | `ON_FAILURE`, max retries `5` |
| Volume | None |

## 5. Publish

1. Set **Category** to `AI / ML`. Railway has no separate Developer Tools category; MCP tooling is listed under AI.
2. Click **Create Template**.
3. Deploy the template once from its template page and wait for the health check to pass.
4. Open the service logs and copy the private **Open MCPJam** link containing `#token=...`.
5. Open that link in a browser to sign in. A bare domain URL only shows instructions.
6. Optionally verify readiness:

   ```bash
   curl -sS -o /dev/null -w '%{http_code}\n' https://YOUR-DOMAIN/health
   ```

7. Share the template URL, and publish it from the template page when ready for the marketplace.

## Custom domain

Railway's `${{RAILWAY_PUBLIC_DOMAIN}}` covers the generated domain only. If users visit a custom domain:

- `MCPJAM_ALLOWED_HOSTS`: `generated-domain,custom-domain` (comma-separated hostnames, no scheme or port).
- `MCPJAM_INSPECTOR_FRONTEND_URL`: `https://custom-domain`.

## Alternative: Docker Image source (no repository)

If you do not want to push this repository, skip steps 1 and 2 above and add the service with **Add New → Docker Image**, image `mcpjam/mcp-inspector:v3.24.0`. Steps 3–5 are unchanged, except root directory does not apply. Railway will not read `railway.json` or `Dockerfile` in that case, so set the deploy settings in step 4 manually.

Do not configure both sources for the same service. This guide uses the GitHub-source path.

## Troubleshooting

| Symptom | Likely cause | Action |
| --- | --- | --- |
| `403 HOST_NOT_ALLOWED` in the browser | Hostname missing from the allowlist | Add the exact hostname to `MCPJAM_ALLOWED_HOSTS` |
| Page loads but API calls fail | Wrong `Origin` or frontend URL | Set `MCPJAM_INSPECTOR_FRONTEND_URL` to the exact HTTPS origin |
| Access link stops working after restart | Session token was not fixed | Check `MCPJAM_SESSION_TOKEN` is set with a template secret function |
| Deploy is slow | Cold pull of the ~950 MB image | Expected on first deploy; subsequent deploys reuse the layer |
| MCP server not visible | STDIO server or unreachable address | Use an HTTP/S endpoint reachable from the container |
