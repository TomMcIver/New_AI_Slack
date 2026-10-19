# Which connectors can each chat app use to reach the hub?

Research for issue #3. Researched 2026-10-19. Vocabulary follows `GLOSSARY.md` (hub, owner, chat app, agent, ground truth, proposal).

## Answer in one paragraph

All three chat apps can call a custom remote MCP server today, so the hub can expose one MCP endpoint and every chat app uses it as a connector. One endpoint is enough if it is **Streamable HTTP**, at a **public HTTPS URL**, with **OAuth 2.1 (PKCE S256) supporting both Client ID Metadata Documents (CIMD) and Dynamic Client Registration (DCR)**. OAuth is the only auth scheme all of them accept (Grok Bot reportedly accepts nothing else). A hub on the owner's PC works through a **named** Cloudflare Tunnel or ngrok. The tunnel only needs to be reachable from the vendors' clouds, because every call originates from the vendor's servers, not from the owner's device. The riskiest unknowns are ChatGPT plan eligibility for an individual owner and the current Grok Bot behaviour. See "Open questions and risks".

## Evidence legend

| Tag | Meaning |
|---|---|
| **V** | Verified: I read the vendor's own page directly in this session. |
| **V-excerpt** | Vendor-owned page (docs.x.ai, x.ai, help.openai.com, cursor.com), but direct fetch was blocked from my environment (TLS error or HTTP 403). I saw its text only through a search-engine excerpt. Treat as probable; re-check by hand. |
| **U** | Unverified: only in secondary material (forum posts, blogs, guides by competing MCP vendors). Not relied on for conclusions. |

## Per-app summary

| | Claude (claude.ai) | ChatGPT (chatgpt.com) | grok.com | Grok Bot (beta) |
|---|---|---|---|---|
| Can call a custom remote MCP server? | Yes (V) | Yes, on eligible plans (V + V-excerpt) | Yes, "Bring Your Own MCP" (V-excerpt) | Yes in principle, via Cursor's MCP stack; details thin (V-excerpt + U) |
| Plan needed | Free (1 custom connector), Pro, Max, Team, Enterprise (V) | Business, Enterprise/Edu clearly; Pro limited to read/fetch; Plus not mentioned (V-excerpt, older and newer pages conflict) | Docs say connectors are "available to all Grok users" (V-excerpt); admin adds it on Business/Enterprise | Individual Cursor plan, Cursor Teams, or linked SuperGrok tier (V-excerpt) |
| Transport | Streamable HTTP; legacy HTTP+SSE still works, being deprecated (V) | Streamable HTTP and SSE (V) | Streamable HTTP and SSE per the API page (V-excerpt); consumer page not explicit | Cursor cloud agents: HTTP and stdio, not SSE (V-excerpt for Cursor; U that Grok Bot inherits it) |
| Auth | OAuth 2.0 (CIMD or DCR), no-auth, or static header (beta, limited orgs) (V) | OAuth (CIMD/DCR), no-auth, or mixed. No static bearer header documented (V) | "complete any required authentication"; built-in connectors use OAuth; header support not stated (V-excerpt) | OAuth only, per user reports; no way to enter a static header (U) |
| Read vs write tools | Both; user approves tool calls; per-tool "Blocked" setting (V) | Tools without `readOnlyHint` are treated as writes and need confirmation each conversation (V) | Not documented in what I could read (gap) | Not documented (gap) |
| Public URL needed | Yes, reachable from Anthropic egress `160.79.104.0/21` (V) | Yes, public HTTPS, or OpenAI's Secure MCP Tunnel on workspace plans (V) | Yes; localhost and private addresses rejected (V-excerpt) | Yes; connection runs from Cursor's cloud, not the owner's machine (U) |
| Reaches a hub on the owner's PC via tunnel | Yes; Anthropic's testing page recommends Cloudflare Tunnel or ngrok (V) | Yes for testing ("development tunnel"); docs imply a stable public endpoint for real use (V) | Yes; docs name ngrok and cloudflared; quick tunnels break SSE (V-excerpt) | Probably, if public HTTPS and OAuth work (U) |
| Known limits | Tool result about 150,000 characters; 240 s per tool call; OAuth endpoints must answer in 10 s; max 4 request headers (V) | Secure MCP Tunnel: 600 req/min, 10 MiB, 120 s (V). Public-endpoint tool limits not stated | Not stated | Not stated |

## Details and sources

### Claude (claude.ai)

- Custom connectors by URL work on Free, Pro, Max, Team and Enterprise; Free is limited to one custom connector (V). https://claude.com/docs/connectors/custom/remote-mcp
- On Team and Enterprise an Owner adds the connector for the organisation and members then sign in themselves (V). Same page.
- Connections come from Anthropic's cloud, so the server must be publicly reachable. Outbound traffic comes from `160.79.104.0/21` (V). https://platform.claude.com/docs/en/api/ip-addresses and https://claude.com/docs/connectors/building/authentication ("Network reference")
- Transport: Streamable HTTP is recommended; legacy HTTP+SSE is supported but being deprecated. A URL ending in `/sse` selects the old transport (V). https://claude.com/docs/connectors/building and the remote-mcp page above.
- Auth (V, https://claude.com/docs/connectors/building/authentication):
  - OAuth 2.0 with DCR or CIMD supported by default; follows MCP authorization specs 2025-03-26, 2025-06-18 and 2025-11-25.
  - Redirect URI to allow: `https://claude.ai/api/mcp/auth_callback`.
  - A `401` with `WWW-Authenticate: Bearer resource_metadata="..."` is required to start sign-in; Claude ignores that header on a `200`. The protected-resource `resource` value must equal the URL the user typed, and only the first `authorization_servers` entry is used.
  - CIMD is used only if the authorization server advertises `client_id_metadata_document_supported: true` and `"none"` in `token_endpoint_auth_methods_supported`; otherwise DCR.
  - PKCE S256 on every request; `/token` must accept `application/x-www-form-urlencoded`; return `invalid_grant` for dead refresh tokens; discovery/registration/token endpoints must answer within 10 s (refresh 30 s).
  - "No authentication" is supported by default. Static header (API key or bearer) auth is **beta, limited organisations** only, so a personal Claude account should not be assumed to have it. https://claude.com/docs/connectors/custom/remote-mcp
- Tools can read or create/modify/delete data; Claude asks for approval per call, and the user can choose "Always allow" or block individual tools (V). https://claude.com/docs/connectors/custom/remote-mcp ("Security and privacy")
- Limits: about 150,000 characters per tool result and 240 s per tool call on claude.ai (V). https://claude.com/docs/connectors/building
- Tunnel: Anthropic says a local server needs a public URL and recommends Cloudflare Tunnel or ngrok. If the server uses the TypeScript SDK's `createMcpExpressApp()`, add the tunnel hostname to `allowedHosts` or requests fail with `403 Invalid Host`. Keep auth on while tunnelling (V). https://claude.com/docs/connectors/building/testing
- Mobile: the building guide says Claude connects from claude.ai, Claude Desktop, Claude mobile, Cowork and Claude Code with the same connector infrastructure (V). Adding a connector is done in the web/desktop settings UI.

### ChatGPT (chatgpt.com)

- Adding a server: Plugins, plus button, "Add custom MCP server", enter name, description and URL "including the `/mcp` path", set authentication, accept the risk warning, create as a plugin (V). https://developers.openai.com/plugins/deploy/connect-chatgpt
- Reachability: public HTTPS endpoint, or Secure MCP Tunnel for private servers (V). Same page. A "development tunnel or another HTTPS forwarding service" can provide an endpoint for local testing, but those testing options do not replace the public HTTPS endpoint required for public distribution.
- Transport: SSE and streamable HTTP (V). https://developers.openai.com/api/docs/guides/custom-mcp-server
- Auth: OAuth, no authentication, or mixed (initialize and tool-list unauthenticated, tools per-tool). For OAuth, ChatGPT uses CIMD or DCR depending on server support (V). Same page; also https://developers.openai.com/api/docs/mcp (recommends OAuth with CIMD). No static bearer-token field is described for the ChatGPT UI.
- Read vs write: `readOnlyHint` marks a tool read-only; tools without it are treated as write actions; write actions require confirmation by default, remembered only within one conversation (V). https://developers.openai.com/api/docs/guides/custom-mcp-server. OpenAI also warns that a write can happen even if a server tags a tool read-only (V, https://developers.openai.com/api/docs/mcp).
- `search` and `fetch` tools are required only for deep research and company knowledge; "Custom MCP servers do not require `search`/`fetch` tools" (V). Both pages above. Format if implemented: `search(query)` returns results with `id`, `title`, `url`; `fetch(id)` returns `id`, `title`, `text`, `url`, optional `metadata`; citations need a non-empty `url` (V, https://developers.openai.com/api/docs/mcp).
- Server instructions: keep the first 512 characters self-contained (V, custom-mcp-server page).
- Use ChatGPT on the web; workspace permissions and Lockdown apply (V, custom-mcp-server page). Mobile support is not stated.
- **Plans (V-excerpt, conflicting)**: the help article "Connecting custom MCP servers in ChatGPT" (https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt) is described in search excerpts as: custom MCP servers available on ChatGPT web for Business and Enterprise/Edu; Pro users can connect MCP servers with read/fetch permissions; full MCP with write actions limited to Business and Enterprise/Edu. Plus is not mentioned for custom servers. An older version required developer mode. Secondary sources (U) disagree widely, including claims that Plus works. Direct fetch of help.openai.com returned 403, so I could not read the current wording. **Whether an individual owner on Plus or Pro can both read and write is not settled.**
- Secure MCP Tunnel (V, https://developers.openai.com/api/docs/guides/secure-mcp-tunnels): an outbound-only `tunnel-client` run inside the network, tunnel ID plus runtime API key, and the tunnel must be associated with the ChatGPT workspace. Limits: 600 requests/min, 10 MiB per message, 120 s deadline. It is aimed at workspace accounts and is not needed if a public tunnel is used.

### grok.com (consumer Grok connectors)

All items here are V-excerpt: docs.x.ai refused my connection (TLS error), so I saw search excerpts of the pages named.

- grok.com and the Grok apps have connectors, and "Bring Your Own MCP" was announced in May 2026. https://x.ai/news/grok-connectors and https://docs.x.ai/grok/connectors
- Setup: grok.com/connectors, New Connector, Custom, enter the MCP server URL, complete any required authentication; Grok then discovers the tools.
- Reachability: public internet only; localhost and private-network addresses are rejected. For local servers use ngrok or cloudflared; Cloudflare quick tunnels do not support SSE, and free tunnel URLs change on restart so the connector must be re-added. https://docs.x.ai/grok/connectors and https://docs.x.ai/grok/connectors/custom-mcp-tunneling
- Business and Enterprise: an admin adds the connector in the console first; members connect their own accounts. https://docs.x.ai/grok/connector-management
- Gaps: transport on the consumer page, plan floor for custom connectors, whether a static header is accepted, and read/write approval behaviour are not stated in what I could see.

### Grok Bot (xAI, beta since Aug 2026)

- What it is (V-excerpt): a persistent agent running on a cloud computer in Cursor's cloud, with desktop and mobile apps as clients. Included with individual Cursor plans and Cursor Teams, and can be linked to a SuperGrok subscription. https://docs.x.ai/grok-bot/overview and https://x.ai/news/introducing-grok-bot
- MCP (V-excerpt): there is no separate Grok Bot connector list. Connectors appear as plugins and obey the team's Cursor MCP policy. The MCP allowlist is Enterprise-only. Grok Bot 101 says it supports the same MCP servers, plugins and skills as Cursor. https://docs.x.ai/grok-bot/teams-and-enterprises and https://x.ai/bot/guides/grok-bot-101
- Cursor's own docs (V-excerpt via search; cursor.com also refused direct fetch): remote MCP supports headers or OAuth; cloud agents accept HTTP and stdio but not SSE; OAuth is per user. https://cursor.com/docs/mcp and https://cursor.com/docs/cloud-agent/capabilities
- **U (Cursor community forum bug reports, user-written; I could not open them either):** custom remote MCP in Grok Bot is OAuth-only in the UI; OAuth registration failed for some servers with "redirect_uri not allowed"; the connection and OAuth discovery run from Cursor's cloud, so private/VPN servers fail; the iOS browser flow did not return to the app. A Cursor/Grok Bot representative reportedly acknowledged some of these. Threads: https://forum.cursor.com/t/grok-bot-custom-mcp-oauth-fails-before-sign-in-redirect-uri-not-allowed/171877 , https://forum.cursor.com/t/grok-bot-custom-remote-mcp-oauth-never-starts-fetch-failed-same-url-works-in-cursor-ide/168188 , https://forum.cursor.com/t/grok-bot-ios-mcp-oauth-browser-flow-does-not-return-to-the-app/168330
- Secondary guides (Composio, Blotato and similar) describe three conflicting setup flows. Ignored because they sell competing products and disagree with each other (U).
- Conclusion for Grok Bot: treat support as "should work with a standards-compliant Streamable HTTP + OAuth endpoint", but test it before promising it.

### xAI API remote MCP tools (matters for the live Grok agent)

- The API supports remote MCP tools: pass `server_url` (required), `server_label` (required), optional `allowed_tools`, `authorization` (sent as the Authorization header) and `headers`. Only Streamable HTTP and SSE transports. The OpenAI-style `require_approval` and `connector_id` parameters are **not supported** (V-excerpt). https://docs.x.ai/developers/tools/remote-mcp
- xAI's servers make the calls, so the endpoint must be publicly reachable here too (V-excerpt, inferred from the docs saying xAI manages the connection on your behalf).
- Relevance: the live Grok agent is an agent, and agents already reach the hub through its API. Remote MCP matters only if we want xAI to run the tool loop itself against the hub. If so, the same endpoint works, and a static bearer in `authorization` is allowed here (unlike chat apps). Because there is no approval prompt, only expose proposal-style write tools to it (see recommendation).
- OpenAI's Responses API has an equivalent remote-MCP feature for OpenAI agents: Streamable HTTP or SSE, public server or `tunnel_id`, `authorization` field, `require_approval` default on (V, https://developers.openai.com/api/docs/guides/tools-connectors-mcp). Those are agent-side, not chat-app-side, so out of scope beyond noting that one endpoint serves them too.

### Tunnel facts

- Cloudflare quick tunnels: no SSE support, 200 in-flight requests, no uptime guarantee, hostname changes each run (V). https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/ . Use a **named** tunnel with a stable hostname, or ngrok with a reserved domain.
- MCP Streamable HTTP rules that matter behind a tunnel: single endpoint supporting POST (and GET or 405); server must validate `Origin` and should authenticate; `MCP-Session-Id` and SSE streaming are optional, so a stateless JSON-response server is valid (V). https://modelcontextprotocol.io/specification/2025-11-25/basic/transports

## Which apps can reach a hub on the owner's PC through a tunnel

All of them can, under the same condition: the tunnel presents a stable public HTTPS hostname and the hub is running. Every vendor calls from its own cloud (Anthropic from `160.79.104.0/21`; Cursor's cloud for Grok Bot per user reports), so nothing needs to be reachable from the owner's phone or laptop. Consequences:

- The hub is reachable by the chat apps only while the PC and tunnel are up. A sleeping PC means "sync with my hub" fails.
- Do not use quick tunnels (SSE unsupported; URL changes).
- Allow the tunnel hostname in any Host/Origin allow-list the MCP SDK enforces.
- The public URL means anyone on the internet can hit the endpoint, so OAuth must actually authenticate the owner (see below), and rate limiting is advised.

## Recommendation: minimal hub MCP endpoint that works with all of them

1. **Transport**: one Streamable HTTP endpoint at `https://<stable-host>/mcp`. Stateless mode, JSON responses (no SSE needed), so it survives tunnels and Grok Bot's HTTP-only cloud agents. Skip legacy `/sse`.
2. **Discovery and auth**: OAuth 2.1 with PKCE S256.
   - Return `401` with `WWW-Authenticate: Bearer resource_metadata="..."` for unauthenticated calls.
   - Serve RFC 9728 protected-resource metadata whose `resource` equals the exact MCP URL, and RFC 8414 authorization-server metadata with a single issuer.
   - Advertise **both** CIMD (`client_id_metadata_document_supported`, `"none"` in token auth methods) and a DCR `registration_endpoint`: Claude and ChatGPT use CIMD when offered, and DCR is the fallback.
   - Redirect URIs: Claude's is `https://claude.ai/api/mcp/auth_callback`. I did not find ChatGPT's or Grok's published redirect URIs in vendor docs, so let DCR/CIMD supply them rather than hard-coding.
   - Token endpoint accepts form-encoded bodies, issues refresh tokens, rotates them, and responds in under 10 s.
   - The authorization step must require the **owner** to log in (for example a passkey or a long passphrase). Do not auto-approve DCR clients; a public tunnel plus open DCR otherwise lets anyone connect.
   - Optional extra for agents only: accept a static bearer on the same endpoint for xAI/OpenAI API tool calls, where an `authorization` field exists.
3. **Tools** (keep the set small; Claude and ChatGPT read tool names and descriptions to decide when to call):
   - `search(query)` and `fetch(id)` over the ground truth, shaped as OpenAI specifies (`id`, `title`, `url`, `text`), annotated `readOnlyHint: true`. This also makes ChatGPT deep research and company knowledge work.
   - `get_ground_truth(subject)` (or equivalent): read-only, returns claims for a subject.
   - `propose_claim(subject, text, source)`: the only write tool. It creates a **proposal**, never a claim, which matches the glossary rule that only the owner approves entries into ground truth. Chat apps still show their normal approval prompt, and the xAI API (no approval parameter) cannot silently change ground truth.
   - Put a short self-contained description in the server instructions (the first 512 characters matter for ChatGPT).
4. **Sizes and timing**: keep results well under 150,000 characters (Claude) and tool calls well under 120 s (the lowest documented limit that could apply); paginate large subjects.
5. **Hosting**: named Cloudflare Tunnel or ngrok reserved domain; keep authentication on; add the tunnel host to SDK host allow-lists; validate `Origin`.
6. **Not needed for v1**: SSE, sessions, resources, prompts, sampling, resource subscriptions (Claude does not support the last two), MCP Apps UI.

## Open questions and risks

1. **ChatGPT plan eligibility** for a single owner: Plus is not named anywhere I could verify, and Pro may be read-only. If the owner is on Plus, ChatGPT may not be able to use the hub at all, and the fallback is the existing import path. Needs a manual check of help.openai.com or a trial add of the connector.
2. **Grok Bot** is new; the only MCP documentation I could see is "same as Cursor". OAuth-only behaviour and redirect-URI problems come from forum reports (U). Needs a hands-on test with the real endpoint.
3. **grok.com plan floor** and write-approval behaviour for custom connectors are not stated in the excerpts I saw.
4. **Blocked sources**: docs.x.ai, x.ai, cursor.com, forum.cursor.com and help.openai.com could not be fetched from the research environment (TLS error or 403), so every claim tagged V-excerpt rests on a search-engine excerpt of the vendor page. Re-reading those pages in a normal browser would upgrade them to V.
5. Vendor docs are changing quickly (OpenAI's move to "plugins", xAI/SpaceXAI naming, Anthropic's header-auth beta). Re-check before building ticket #10.
