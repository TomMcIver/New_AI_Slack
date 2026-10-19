# Does Composio earn a place in the hub?

Research ticket: GitHub issue #5. Researched 2026-10-19. Primary sources only (Composio's own docs, pricing page, SDK repo, first-party marketing pages). Vocabulary follows `GLOSSARY.md` (hub, owner, agent, chat app, room, ground truth).

Marks: **[V]** verified against the cited first-party page on the date above. **[U]** unverified, ambiguous, internally inconsistent, or taken from a non-docs page. No secondary write-ups (blogs, reviews, aggregator sites) were used as evidence.

## Recommendation

**Skip for v1. Revisit later as an optional adapter.**

- Do not make Composio a dependency. Every Composio path needs a Composio account and API key, and by default Composio holds the owner's OAuth tokens. That conflicts with "anyone can clone the repo and run their own hub" and with the hub keeping its own credentials.
- For the hub's real v1 job (agents read GitHub repos, open issues, draft PRs, never push to main), GitHub's own API is simpler, free, and lets the hub enforce the "never push to main" rule itself (see "What it adds over calling GitHub directly").
- Composio does **not** solve the chat-app problem the owner asked about. It lets chat apps reach **Composio's catalog of apps**, not the hub. The docs say its session MCP endpoint "only exposes Composio's hosted tools" [V], and Custom MCP makes Composio a client of your server, not a gateway in front of it [V]. A chat app reaches the hub only if the hub itself exposes a remote MCP server (or the owner uses the hub's imports). See the chat-app section.
- Keep the door open: if the hub defines a small "tool provider" seam (list tools, execute tool for owner), a Composio-backed adapter can be added later as an optional plugin, off by default, for the long tail of services (Gmail, Slack, Notion) the hub does not want to integrate by hand.

## Answers to the ticket questions

| Question | Answer |
|---|---|
| Custom toolkits or custom tools? | Yes, two forms. In-process custom tools/toolkits (run in your process, not exposed over MCP) and Custom MCP (Composio proxies to your remote MCP server). Both marked experimental. |
| Host an MCP gateway the hub could sit behind or expose through? | No for "sit behind". Composio hosts MCP endpoints for its own catalog only. Custom MCP registers your server as a toolkit inside Composio sessions; Composio is the client. |
| Bridge chat apps (Claude, ChatGPT, Grok Bot) to a custom app/MCP server? | Not as documented. It bridges chat apps to Composio's catalog. Whether a registered `CUSTOM_*` toolkit shows up via Composio Connect to chat apps is not documented [U]. |
| Auth model | Per-end-user (`user_id`) connected accounts, Composio-managed OAuth by default, bring-your-own OAuth app optional. Composio stores tokens. |
| Cost | Free Hobby tier: 100,000 tool calls/month, no credit card. Pro $29/month. |
| Self-hosting | Enterprise arrangement only (Helm/private VPC). Not a free self-host path. SDK is MIT; the backend is not in the public repo. |
| Lock-in | Moderate. Token custody and connected-account state live in Composio's cloud. Tool slugs are Composio-specific. |
| Stranger who clones the repo | Needs a Composio account and `COMPOSIO_API_KEY` for any Composio feature. |

## What it adds over calling GitHub directly

What Composio is: a hosted platform that provides tool discovery, authentication, and execution for 1000+ apps, used from your code via SDK or via a hosted MCP endpoint. Source: https://docs.composio.dev [V], https://github.com/ComposioHQ/composio (README) [V].

Concretely it adds:

1. **One integration surface for many apps.** Toolkits (e.g. `github`) contain tools (e.g. `GITHUB_CREATE_ISSUE`). https://docs.composio.dev [V]
2. **Managed OAuth.** "Composio registers and maintains OAuth apps for popular toolkits such as GitHub, Gmail, and Slack," and handles redirects, token exchange and refresh. https://docs.composio.dev/docs/authentication/custom-app-vs-managed-app.md [V], https://docs.composio.dev/docs/how-composio-works.md [V]
3. **Agent-friendly tool delivery.** By default a session gives the agent a small fixed set of meta tools (search tools, get schemas, multi-execute up to 50 per call, manage connections, remote bash/Python sandbox) rather than hundreds of tool definitions. https://docs.composio.dev/docs/how-composio-works.md [V], https://docs.composio.dev/docs/composio-connect.md [V]
4. **Triggers (webhooks) and a log of every tool execution** (toolkit, action, user_id, timing, and by default inputs/outputs, kept up to one year). https://docs.composio.dev/docs/security/data-retention.md [V]

What it does not add for the hub's v1 needs:

- GitHub read, issue creation and draft PRs are already first-class in the GitHub REST API (`gh` or Octokit). The hub's agents need maybe five operations, so a 1000-app catalog is surplus.
- **The hub must enforce "never push to main" itself.** Over Composio's MCP path, "Tool-call modifiers do not run over MCP. `beforeExecute`/`afterExecute` hooks and `modifySchema` transforms are bypassed, so you cannot intercept, log, or gate calls." https://docs.composio.dev/docs/sessions-via-mcp.md [V]. With a direct client the hub can wrap the call itself (and use a GitHub token limited to issues and PRs, with branch protection on main). Composio's docs do not say how to restrict a GitHub tool to draft PRs only; the available lever is restricting which tools are exposed [V], not constraining arguments.
- Extra hop and extra third party: every call goes hub -> Composio -> GitHub, and tool inputs/outputs (possibly repo contents) are logged by Composio by default. Zero Data Retention is "available on paid plans for a usage-based price and included on Enterprise." https://docs.composio.dev/docs/security/data-retention.md [V]. Composio states it does not train on request/response data "on any plan" [V].

Where it would plausibly earn its keep later: many different SaaS apps per owner, each needing OAuth, where hand-rolling each integration is the cost. Not a v1 concern.

## Custom toolkits and custom tools

Source: https://docs.composio.dev/docs/extending-sessions/custom-tools-and-toolkits.md [V]

- Three patterns: standalone tools, extension tools (`extendsToolkit`, inherit that toolkit's auth), custom toolkits (groups of standalone tools).
- Defined with `experimental_createTool` (TypeScript) or `@composio.experimental.tool()` (Python). The `experimental` naming is the docs' own.
- Execution: "Custom tools execute in-process; remote tools are sent to the backend automatically." Not supported in the sandbox.
- Visibility over MCP: the custom-tools page is silent [U], but the MCP page states explicitly: "The MCP server only exposes Composio's hosted tools, so your local custom tools aren't available over the endpoint." https://docs.composio.dev/docs/sessions-via-mcp.md [V]. So in-process custom tools cannot be given to a chat app through Composio.

## Custom MCP (bring your own MCP server)

Source: https://docs.composio.dev/docs/extending-sessions/custom-mcp.md [V]

- "Composio does not host the server." You deploy a remote MCP server at a public HTTPS URL, register it (`POST /api/v3.1/custom/toolkits/upsert`), and Composio creates a project-scoped `CUSTOM_*` toolkit. Execution "is proxied to your MCP server with the selected connected account's credentials."
- Auth modes: `NO_AUTH`, `API_KEY`, `DCR_OAUTH`. Local/STDIO servers unsupported. Max 500 tools per toolkit. Experimental and "may change". `app_url` and `auth_schemes` are immutable after registration. No dashboard management yet, no continuous sync.
- Direction of data flow: your agent -> Composio session -> your MCP server. That is Composio consuming the hub, not a gateway the hub sits behind. The hub's own agents do not need this; they can call the hub's own functions directly.
- Whether a `CUSTOM_*` toolkit is then reachable by chat apps through Composio Connect is not stated in the docs [U].
- Pricing page lists "Custom tools and MCP" as included on all plans, including Hobby. https://composio.dev/pricing [V]

## Hosting an MCP gateway / bridging chat apps

**Composio Connect** is a hosted MCP server for the Composio catalog, "the shared Composio MCP URL, without creating an SDK session", `https://connect.composio.dev/mcp`, exposing 7 meta-tools. https://docs.composio.dev/docs/composio-connect.md [V]

- Clients listed in the doc: Claude Code, Cursor, Codex, Gemini CLI, VS Code, Claude Desktop ("Claude Cowork"), ChatGPT, Grok, Notion, n8n, OpenAI Agent Builder, any streamable-HTTP MCP client [V].
- Constraints stated in the doc: ChatGPT needs Developer mode (Plus, Pro, Business, Enterprise or Edu plan); Grok "Custom connectors require a paid tier" [V]. The chat-app side therefore has its own paid-plan requirement independent of Composio.
- Connecting a client uses a Composio consumer API key header (`x-consumer-api-key`); apps connect via short-lived OAuth links [V].
- Grok Bot: Composio's own listing describes installing Composio from the Grok Bot plugin marketplace (Add, Authenticate, Authorize). Required plan per that page: "Grok Bot is included with Cursor Ultra, Cursor Premium Teams, and SuperGrok Heavy." https://composio.dev/toolkits/composio/framework/grok-bot.md [U: first-party marketing page, not the docs, and the plan wording is odd, so confirm with xAI before relying on it]. The page does not say you can add your own MCP server to Grok Bot [V as an absence].
- Native agent plugins exist only for Codex and Claude Code (not ChatGPT or Grok). https://docs.composio.dev/docs/agent-plugins.md [V]

Conclusion for the owner's question ("could Composio connect the chat apps to the hub's app?"): Composio connects chat apps to **Composio's catalog of third-party apps**. It is not a documented bridge to an arbitrary custom app. The route that does match the glossary ("joins the hub only through imports or connectors") is the hub exposing its **own** remote MCP server (read ground truth, submit proposals) and the owner adding that URL as a custom connector in each chat app, subject to each chat app's own plan limits above. That needs no Composio. (This last point is a design suggestion, not a verified Composio claim; the chat apps' connector requirements should be checked in their own docs in a separate ticket.)

## Auth model

- **User-scoped.** `user_id` is "an identifier from your app that Composio uses to store connections"; use a stable ID, not an email; using `default` in production "exposes other users' data". A user can have several connected accounts per toolkit. https://docs.composio.dev/docs/how-composio-works.md [V]
- **Managed OAuth by default**, with an option to bring your own OAuth app (own branding, scopes, dedicated quota; for GitHub you register an OAuth app and paste client ID/secret). https://docs.composio.dev/docs/authentication/custom-app-vs-managed-app.md [V]. Managed apps "share quota across all Composio users" [V]. Whether the free tier allows custom OAuth apps: the docs page is silent, but the pricing page says "Using your own OAuth app or API keys gets the full 100K free calls" [V], which implies yes.
- **Token custody.** "In the default Composio Cloud deployment, 'Composio has custody of those credentials.'" Encrypted at rest (AES-256-GCM) and in transit; API responses redact tokens; "Owning the OAuth app does not move the token store into your infrastructure." https://docs.composio.dev/docs/security/token-custody.md [V]
- **Unattended agents** still need a human to complete OAuth consent: "A Composio agent credential does not authorize GitHub, Slack, or another service on someone's behalf." https://docs.composio.dev/docs/agent-setup/unattended-authentication.md [V]. Fine for one owner doing it once; extra setup versus pasting a GitHub token.
- Self-managed credentials: pricing page says "Pass your own access tokens at execution time" on all plans [V], though the details were not read in the docs [U]. That path gets close to a direct integration with extra steps.

## Pricing and limits

Source: https://composio.dev/pricing [V unless noted]; rate limits: https://docs.composio.dev/reference/rate-limits.md [V]

- **Hobby: $0, no credit card.** 100,000 tool calls/month, 50,000 trigger events/month, 3 team members, unlimited connected accounts, 1M sandbox LLM tokens. Hard cap: usage pauses at the limit, no overage billing. Rate limit 2,000 requests/minute per organization.
- **Pro: $29/month**, includes $29 usage credit (no rollover); tool calls $0.0003 each beyond quota; unlimited team members; 10,000 requests/minute.
- **Enterprise:** custom; adds SSO/SCIM, customer-managed keys, custom SLAs.
- Composio-managed apps: only 20K of the 100K free calls apply to managed-OAuth usage; bring-your-own OAuth app gets the full 100K. Over-quota rate for managed apps is stated inconsistently on the same page ($0.0002 per extra call in one place, $0.0005 in another) [U: inconsistent, confirm before budgeting].
- Add-ons on Pro include direct execution (+$0.0001/call), proxy execute (+$0.0002/call), zero data retention (+$0.0001/call) [V].
- Pricing fetch was truncated by the reading tool in places; verify the full page before quoting numbers publicly.

For one owner's hub, volume would sit far below 100K calls/month, so cost is not the objection. The objection is the mandatory account and key.

## Self-hosting and open source

- The public repo `ComposioHQ/composio` is **MIT licensed** and contains the TypeScript/Python SDKs, CLI and provider adapters; the README describes no backend code. https://github.com/ComposioHQ/composio [V] (license via GitHub API; contents via README)
- The platform/backend: the quickstart requires a `COMPOSIO_API_KEY` from the hosted dashboard. https://docs.composio.dev/docs/quickstart [V]
- Self-host exists as an enterprise arrangement: "Private VPC deployment", "Self-hosted deployment: Runs Composio in your environment, with Helm deployment support", and customer-managed keys, "set up with the Composio team". https://docs.composio.dev/docs/security/token-custody.md [V]. A Helm troubleshooting page exists but gives no eligibility, licence or pricing terms. https://docs.composio.dev/kb/guide/platform-self-hosted-helm.md [V]. A cloner cannot count on a free self-host path [U: terms not published].
- "Local sandbox" docs exist for running code execution locally, which is separate from self-hosting the auth/token platform. https://docs.composio.dev/docs/sandbox/local.md [U: not read]

## Lock-in

- Connected accounts, auth configs and (by default) tokens live in Composio's cloud, keyed by `user_id`. Leaving means re-doing OAuth consent per app. Docs describe importing existing connections in, but no documented export: "Reading connected-account state is not a supported token-export workflow." https://docs.composio.dev/docs/security/token-custody.md [V]
- Tool slugs (`GITHUB_CREATE_ISSUE`), session and meta-tool semantics are Composio-specific; custom-tool APIs are `experimental`. The docs also show churn: direct execution is "legacy", sessions recommended, API moved to v3.1. https://docs.composio.dev/docs/how-composio-works.md [V]
- Mitigation if adopted: hide it behind the hub's own tool interface, keep it optional, and never let the ground truth depend on it.

## What a stranger who clones the repo would need

If the hub depended on Composio:

1. A Composio account (free Hobby works) and a `COMPOSIO_API_KEY` from the dashboard. [V] https://docs.composio.dev/docs/quickstart
2. A browser OAuth consent per connected app (Connect Link). [V]
3. If they also want chat apps to use it: Composio Connect consumer key plus a paid tier on the chat app side (ChatGPT Developer mode needs a paid plan; Grok custom connectors need a paid tier). [V] https://docs.composio.dev/docs/composio-connect.md
4. They accept that Composio stores tool inputs/outputs for up to a year unless on a paid plan with ZDR. [V]

Without Composio, a stranger needs only the agent API keys they already must supply plus, optionally, a GitHub token. That matches the repo's clone-and-run goal, hence "skip for v1".

## Open questions (not answered by primary sources)

- Can a registered `CUSTOM_*` toolkit be reached through Composio Connect from Claude/ChatGPT/Grok? Docs silent. Test needs a Composio account; low priority since the hub can expose its own MCP server.
- Does a GitHub toolkit action allow restricting to draft PRs or non-default branches? Not found in the docs read; the safe design enforces this in the hub or via GitHub token scope and branch protection.
- Terms for self-hosted Composio (licence, price, eligibility): ask Composio if ever needed.
- How each chat app (claude.ai, chatgpt.com, Grok Bot) accepts a custom remote MCP connector, including plan requirements: separate ticket, check each vendor's own docs.

## Source list

- https://docs.composio.dev
- https://docs.composio.dev/llms-index.txt
- https://docs.composio.dev/docs/quickstart
- https://docs.composio.dev/docs/how-composio-works.md
- https://docs.composio.dev/docs/composio-connect.md
- https://docs.composio.dev/docs/sessions-via-mcp.md
- https://docs.composio.dev/docs/extending-sessions/custom-tools-and-toolkits.md
- https://docs.composio.dev/docs/extending-sessions/custom-mcp.md
- https://docs.composio.dev/docs/authentication/custom-app-vs-managed-app.md
- https://docs.composio.dev/docs/agent-setup/unattended-authentication.md
- https://docs.composio.dev/docs/agent-plugins.md
- https://docs.composio.dev/docs/security/token-custody.md
- https://docs.composio.dev/docs/security/data-retention.md
- https://docs.composio.dev/reference/rate-limits.md
- https://docs.composio.dev/kb/guide/platform-self-hosted-helm.md
- https://composio.dev/pricing
- https://composio.dev/toolkits/composio/framework/grok-bot.md
- https://github.com/ComposioHQ/composio
