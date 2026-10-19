# Chat-history and memory exports from each chat app

Research ticket: TomMcIver/New_AI_Slack#4. Researched 2026-10-19.

Scope: what the **owner** can export about past chats and the app's built-in memory from Claude (claude.ai), ChatGPT (chatgpt.com), Grok (grok.com and Grok on X) and Grok Bot (xAI, beta since Aug 2026). Format facts only. No real export content is, or may be, committed to this public repo.

## How to read this note

- **[V]** verified: stated on the vendor's own help centre, docs or legal page (URL given).
- **[U]** unverified: found only in secondary sources, or inferred. Treat as a hypothesis until checked against a real export on the owner's machine.
- Access note: from the research environment, `x.ai`, `docs.x.ai` and `help.x.com` could not be reached directly. Those pages were read through Internet Archive snapshots of the vendor page (snapshot dates 2026-09-27 to 2026-10-17, URL form `web.archive.org/web/<timestamp>/<original-url>`). The original URL is cited; the content is the vendor's text as archived. `help.openai.com/en/articles/7260999` returned HTTP 403 to a plain fetch and was read with a browser user agent. Re-check the exact wording live before relying on it.
- xAI now brands itself **SpaceXSI / SpaceXAI** in its legal pages and docs. "xAI" and "SpaceXAI" below are the same vendor.

## Summary table

| | Claude | ChatGPT | Grok (grok.com / X) | Grok Bot |
|---|---|---|---|---|
| Chat export exists | Yes [V] | Yes [V] | Yes, but undocumented in detail [V that it exists, U on format] | Not documented [V: no export article found] |
| Request path | Settings > Privacy > Export data (web or Claude Desktop, not mobile) [V] | Settings > Data controls > Export data, or the OpenAI Privacy Portal [V] | grok.com or Grok mobile app: Settings > Data Controls (download); or request via x.ai/privacy-portal [V] | None found. Enterprise-only opt-in streaming of prompts/responses via OpenTelemetry [V] |
| Delivery | Email with download link, must be signed in [V] | Email (or SMS if phone-only account) with ZIP link, same account [V] | Not documented (email link claimed by secondary sources) [U] | n/a |
| Link expiry | 24 hours after delivery [V] | 24 hours after receipt [V] | Unknown | n/a |
| Wait time | "small delay" [V] | Up to 7 days [V] | Unknown (minutes to a day per secondary sources) [U] | n/a |
| File format | Not stated by Claude; JSON per a Claude article about moving conversations [V weak]; ZIP with `conversations.json` etc. [U] | ZIP containing `conversations.json`, possibly numbered shards [V]; also `chat.html`, `user.json`, `message_feedback.json` etc. [U] | Not stated by xAI. JSON in a ZIP per secondary sources [U] | n/a |
| Built-in memory in export | Yes: "All memory data is included in data exports" [V]; file name/shape unknown | Not stated [U]; do not assume | Not stated | n/a |
| Memory export feature | Yes, in-app and via a prompt [V] | No dedicated feature found in OpenAI docs [V by absence]; manual "write out what you know" prompt only [U] | No official feature found; "import memory" seen only in secondary sources [U] | Ask the Bot in chat (documented for Team Bots) [V]; no file export |
| Memory import feature | Yes, paste text into Settings > Memory [V] | No dedicated feature found [V by absence] | Unconfirmed [U] | Duplicating a Bot does not copy memory [V] |

## Claude (claude.ai)

### Chat export
- **Request**: Settings > Privacy > "Export data". Works on the web app and Claude Desktop. "It's not possible to run an export from Claude for iOS or Android." [V] https://support.claude.com/en/articles/9450526-how-can-i-export-my-claude-data
- **Who**: Free, Pro and Max users export directly. Team and Enterprise members can export their own chats only after an owner enables "Allow members to export their own data". Team/Enterprise Primary Owners instead export the whole organisation from Organization settings > Data and privacy > Export Data. [V] https://support.claude.com/en/articles/9450526-how-can-i-export-my-claude-data and https://support.claude.com/en/articles/13346720-export-your-organization-s-data
- **Delivery**: download link sent by email to the account address; you must be signed in to download; link expires 24 hours after delivery; repeat the export for a new link. [V] same two URLs.
- **Contents**: "conversation data and the user data for your account". [V] https://support.claude.com/en/articles/9450526-how-can-i-export-my-claude-data
- **Memory is included**: "All memory data is included in data exports." [V] https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- **Not included**: for Team/Enterprise members, "the contents of files you uploaded". Messages, files and projects deleted before the export are not included. Incognito chats are not saved at all for individuals (they are included in Team/Enterprise organisation exports). [V] URLs above.
- **File format**: Claude's export articles do not state the format, size limits or frequency. [V that it is unstated] The only vendor hint is a Claude for Education article that says to "export data from your existing account and upload the conversation data (JSON format)". [V, weak] https://support.claude.com/en/articles/11139144-use-claude-for-education-at-your-university
- **Schema (secondary only)**: a ZIP with `conversations.json` (array of conversations, each with an id, a name, created/updated timestamps, a model field and a `chat_messages` array whose items carry a sender of human/assistant, message text and a UTC timestamp), plus `users.json` and `projects.json`. [U] https://saveai.net/blog/how-to-export-claude-conversations and https://ai-chat-importer.com/blog/how-to-export-claude-conversations . Secondary sources disagree on whether projects and memory are in the ZIP, and say the structure has changed over time. Verify against a real export locally; never commit it.
- **Not documented either way**: projects (names, instructions, attached knowledge), custom instructions / preferences, artifacts, attachments' bytes. Treat as absent until seen. [U]
- **Cannot re-import**: "Exported data can't be imported into another personal Claude account." [V] https://support.claude.com/en/articles/9450526-how-can-i-export-my-claude-data

### Built-in memory export and import (pairs with a Handover prompt)
- **Export**: Settings > Capabilities > "View and edit your memory", or ask Claude in chat to "Write out your memories of me verbatim, exactly as they appear in your memory", then save the text. [V] https://support.claude.com/en/articles/12123587-importing-and-exporting-your-memory
- **Import**: Settings > Memory > "Start import" (legacy: Settings > Capabilities or the home-screen card), paste text, click "Add to memory"; Claude stores individual entries reviewable in the memory panel. Works on web and Claude Desktop, on Free, Pro, Max, Team and Enterprise. [V] same URL.
- The same article supplies a prompt to run in the *other* AI that asks it to list stored memories in one code block, dated where possible, in the user's own words, and to confirm completeness. This is the closest vendor-blessed analogue of a hub Handover prompt. [V] same URL.
- **Caveats**: import is "experimental"; Claude "may not always successfully incorporate imported memories"; memory is "designed to focus on work-related topics", so unrelated personal detail may be dropped; sensitive categories (health, politics, religion) are not saved unless opted in, and some data (government IDs, financial account numbers) is never saved. No numeric size limits are stated. [V] https://support.claude.com/en/articles/12123587-importing-and-exporting-your-memory and https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- Memory is per-project as well as account-wide ("Each project has its own memory"). [V] same 11817273 URL. An export prompt may need running inside each project. [U]
- Legacy-memory users could export legacy memory under Settings > Memory until September 9, 2026; that date has passed. [V] same 11817273 URL.
- Organisation owners cannot view or edit members' memories. [V] same URL.

## ChatGPT (chatgpt.com)

### Chat export
- **Request**: ChatGPT settings > Data controls > "Export data"; or the OpenAI Privacy Portal. The Android app uses the same Data controls > Export Data path, and some app versions open the web to finish. [V] https://help.openai.com/en/articles/7260999 and https://help.openai.com/en/articles/8167885
- **Who**: ChatGPT Free, Go, Plus, Pro and eligible Edu workspaces. Not available for Business, Enterprise or ChatGPT for Healthcare (go through the workspace owner). Not available while logged out. [V] https://help.openai.com/en/articles/7260999 and https://help.openai.com/en/articles/7730893
- **Delivery**: email, or SMS if the account uses only a phone number, when the export is ready; "Exports can take up to 7 days to arrive"; the link "expires 24 hours after you receive it"; download while signed in to the same account that requested it; only one request can be processing at a time. [V] https://help.openai.com/en/articles/7260999 and https://help.openai.com/en/articles/8167885
- **Contents (vendor wording)**: "The downloaded ZIP file includes your chat history and other relevant account data." It does not list files. [V] https://help.openai.com/en/articles/7260999
- **Files named by the vendor**: `conversations.json` is the conversation file; "Larger exports may contain numbered conversation JSON files instead." [V] https://help.openai.com/en/articles/9106926-transfer-exported-conversations-between-chatgpt-accounts
- **Upload limit hint**: the same article advises splitting a too-large conversation JSON before uploading it to ChatGPT, so single-file size can be large enough to matter. [V] same URL. Numeric size limit not stated.
- **Schema (secondary only)** [U] https://ai-chat-importer.com/guides/chatgpt-export-format-explained and https://levelup.gitconnected.com/your-chatgpt-history-is-a-tree-not-a-transcript-2026-5660ceb301d4 :
  - Other files in the ZIP: `chat.html` (some exports), `user.json`, `message_feedback.json`, `model_comparisons.json`, `shared_conversations.json`.
  - `conversations.json` is a JSON array. Each conversation has `id`, `title`, `create_time`, `update_time` (Unix seconds), a `mapping` object and a `current_node`.
  - `mapping` is a graph of nodes, each with `id`, `message`, `parent`, `children`. Edited prompts and regenerated answers create branches, so file order is not conversation order. Rebuild the visible thread by walking `parent` links from `current_node` to the root, then reversing.
  - `message` has `id`, `author.role` (user, assistant, system, tool), `content` with `content_type` and `parts`, and `create_time`. Root and some placeholder nodes have a null `message`. Content types beyond plain text (images, code, tool output, file pointers) exist; attachments are referenced by pointer fields and stored as separate files in the ZIP. The attachment field names were not confirmed from any source here.
- **Not documented**: whether saved memories, custom instructions, projects, GPTs and uploaded-file bytes are in the ZIP. OpenAI's pages say nothing either way. Treat as absent. [U] Uploading an exported conversation file to another account explicitly does **not** transfer custom instructions, memories, GPTs or account settings. [V] https://help.openai.com/en/articles/9106926-transfer-exported-conversations-between-chatgpt-accounts
- **Privacy caveat**: "Downloaded data can include sensitive account information. Keep your download link and exported files secure." [V] https://help.openai.com/en/articles/7260999 . Deleted chats cannot be restored by an export. [V] same URL. Export before a personal workspace is merged into a Business/Enterprise workspace, because the old export link or option may stop working afterwards. [V] same URL.

### Built-in memory export and import
- ChatGPT has two memory styles: "saved memories" (explicit list) and "improved memory" (a broader continually updated summary). The memory summary "may omit details", so it is not a complete record. Saved memories are stored separately from chat history. [V] https://help.openai.com/en/articles/8590148-memory-faq
- Memories are viewed and deleted in ChatGPT settings (path reported as Settings > Personalization > Manage memory [U], from a search-result summary of OpenAI help content). No "export memory" or "import memory" button is described in the OpenAI memory FAQ, the data-controls article or the export article. [V by absence; recheck the live UI]
- The only vendor-documented movement of memory is when migrating a personal workspace into Business/Enterprise: supported cross-chat saved memories are added to the destination workspace; project-scoped memories are not transferred. This is not an export to a file. [V] https://help.openai.com/en/articles/8801890-can-i-migrate-or-merge-my-personal-chatgpt-workspace-over-to-my-chatgpt-business-workspace
- Practical route: a Handover-style prompt asking ChatGPT to print its saved memories and custom instructions as text, which the owner pastes into the hub. Output completeness cannot be guaranteed (the summary may omit items). [U]

## Grok (grok.com, Grok mobile apps, Grok on X)

### Chat export
- xAI's consumer FAQ says: "You are in control to access, download, and delete your data. Please go to your Grok mobile app or Grok.com (Settings/Data Controls) to delete or download your data", and that other requests go to https://x.ai/privacy-portal/. [V] https://x.ai/legal/faq (privacy policy, last updated October 11, 2026: https://x.ai/legal/privacy-policy , repeats that access/correct/delete requests go through the privacy portal).
- The Privacy Portal lists "Access your data: Request a copy of the personal data we hold about you or your organization." It requires access to the email address tied to the request. [V] https://x.ai/privacy-portal
- **Not stated by xAI anywhere found**: delivery method, link expiry, wait time, file names, JSON schema, size limits, whether memory is included. Secondary sources describe a ZIP with a JSON of conversations and disagree on menu names (Settings > Privacy > Export Data vs Settings > Data Controls) and on timing. [U] https://www.llmnesia.com/blog/how-to-export-grok-conversation-history , https://ai-chat-importer.com/blog/how-to-export-grok-conversations . One vendor of a competing export tool claims no native export exists; treat as biased. [U] https://ai-toolbox.co/ai-toolbox-grok-features/export-grok-conversations-txt-markdown-json-pdf-2026
- **Grok on X is governed by X, not xAI**: the xAI privacy policy states it does not apply to Grok on X, which follows the X privacy policy. [V] https://x.ai/legal/privacy-policy . X's own archive article ("Download your data" to a ZIP of HTML and JSON files) lists posts, DMs, media, followers, lists and inferred interests, and does **not** mention Grok conversations. [V that Grok is not listed] https://help.x.com/en/managing-your-account/how-to-download-your-x-archive . X's Grok page documents deleting Grok conversation history (Privacy & Safety > Data sharing and personalization > Grok) but no export. [V] https://help.x.com/en/using-x/about-grok
- **Retention and caveats**: deleted conversations and Private Chats are removed from xAI systems within 30 days unless retained for safety, security or legal reasons. [V] https://x.ai/legal/faq . Because Grok-on-X history lives under the X account, the owner should confirm which surface they used before requesting.
- **ToS**: the owner retains ownership of Input and Output ("User Content"), but grants xAI a broad licence to use it. Beta features are provided "AS IS" and may cause data loss. [V] https://x.ai/legal/terms-of-service (last updated October 12, 2026)

### Built-in memory export and import
- The consumer Grok memory feature (launched April 2025) has no vendor help page found in this research. No official export or import button is documented. [V by absence for export] Secondary reporting says the user can view and delete memories in the chat interface and switch memory off in Data Controls, and that xAI tested an "Import memory from other AI providers" option. [U] https://techcrunch.com/2025/04/16/xai-adds-a-memory-feature-to-grok , https://www.anuma.ai/blog/grok-memory
- Grok **Build** (xAI's developer CLI) has its own memory with `/memory` and `grok memory clear`; this is a different product from the chat app. [V] https://x.ai/news/grok-build-memory (Sep 16, 2026) and https://docs.x.ai/build/modes-and-commands
- Practical route: a Handover-style prompt asking Grok to list what it remembers, pasted by the owner. [U]

## Grok Bot (xAI, beta since Aug 2026)

- Launch date, the "beta" label and the price are from press, not the docs (the docs pages read do not use the word beta): https://www.unite.ai/xai-launches-grok-bot-always-on-ai-teammates-with-their-own-cloud-computers/ [U]. Access per docs: every paid individual Cursor plan, Cursor Teams, or an individual SuperGrok account link. [V] https://docs.x.ai/grok-bot/overview
- **Identity and data settings follow Cursor**: "Grok Bot uses Cursor authentication and account data settings. Training opt-out follows the applicable Cursor account and privacy settings." It "requires cloud data storage". [V] https://docs.x.ai/grok-bot/faq . The Bots run on one persistent cloud computer per user account, shared by all that user's Bots. [V] https://docs.x.ai/grok-bot/overview
- **No chat or memory export is documented** in the Grok Bot docs read (overview, bots, faq, files-and-results, security, teams-and-enterprises, team-bots, mobile). [V by absence] https://docs.x.ai/grok-bot/overview , /bots , /faq , /files-and-results , /security , /teams-and-enterprises , /team-bots , /mobile
- **What memory is**: a Bot "can retain stable working preferences, important facts, and summaries from its work"; conversation and learned role are separate per Bot, while files, browser sessions and group messages move context between Bots. [V] https://docs.x.ai/grok-bot/bots and https://docs.x.ai/grok-bot/faq
- **Only documented way to see memory**: for Team Bots, "To see what the Bot has saved for the team, ask it in chat." The pattern generalises to a personal Bot only by inference. [V for Team Bots, U for personal] https://docs.x.ai/grok-bot/team-bots
- **No memory copy**: duplicating a Bot "does not copy conversation history, learned memory, or chat attachments"; sharing a Bot as a template does not expose the computer, logins or conversation history. Deleting a Bot removes its profile, conversation and routines, but shared computer files and sign-ins may remain; "Backend retention follows the applicable Cursor terms." [V] https://docs.x.ai/grok-bot/bots and https://docs.x.ai/grok-bot/faq
- **Files**: Bots read and write files under `/workspace` on the cloud computer and can return files; attachments are limited to six at a time on desktop, 25 MB each (200 MB video). No bulk download of the workspace is described. [V] https://docs.x.ai/grok-bot/files-and-results
- **Enterprise-only structured export**: Action Recording captures Bot actions as metadata; "Conversation content export" is an opt-in addition to OpenTelemetry Export that streams the text of user prompts and assistant responses (and optionally MCP tool arguments/results). It is off by default, does not backfill earlier messages, and is scrubbed for credentials and emails before export. [V] https://docs.x.ai/grok-bot/teams-and-enterprises and https://docs.x.ai/grok-bot/security (Enterprise only)
- **Verdict**: for an individual owner, Grok Bot exposes **no memory export and no chat-history export** that is documented. Possible channels, all unconfirmed: the xAI privacy-portal data request, Cursor's account data export, or asking the Bot in chat to write out its memory and conversation summary into `/workspace`. [U]

## Implications for the hub import parser

1. **Plan for four different inputs, two of them unknown.** Claude and ChatGPT ZIPs are well enough attested to design parsers. The Grok export format is unverified and Grok Bot has none. Do not write a Grok parser until the owner supplies a real export; keep the parser behind a per-source adapter interface with a "format not recognised" path.
2. **Accept a ZIP or an extracted folder** in the local imports folder, detect the source by file names (`conversations.json` plus `mapping` for ChatGPT; `conversations.json` with `chat_messages` for Claude [U]), and fail clearly on anything else rather than guessing. Remember `conversations.json` is the name for both Claude [U] and ChatGPT [V], so sniff the content, not the name.
3. **ChatGPT is a tree, not a list.** Walk parent links from `current_node`, skip null-message nodes and `system` roles, and treat discarded branches as non-authoritative. Expect numbered shard files [V] and very large single files [V hint], so stream or process per file.
4. **Timestamps differ**: ChatGPT uses Unix seconds [U]; Claude reportedly uses UTC timestamps in an unconfirmed string format [U]. Normalise on read.
5. **Attachments and files are mostly out of scope.** Claude does not export uploaded file contents [V]; ChatGPT's file handling is undocumented [U]. The parser should record that an attachment existed (name only) and never depend on bytes.
6. **Projects, custom instructions and built-in memory are not reliably in the files.** Claude says memory is in its export [V] but no file name or shape is documented; ChatGPT and Grok say nothing. The hub should therefore treat built-in memory as a separate, second import route: the owner pastes the output of a Handover prompt run inside each chat app. Claude has an official prompt-and-paste flow for this [V]; for ChatGPT, Grok and Grok Bot it is a manual prompt [U].
7. **Memory text is lossy and self-reported.** ChatGPT's summary "may omit details" [V]; Claude drops non-work and sensitive topics [V]; Grok Bot memory is summaries and facts [V]. Everything an agent derives from these must enter as a **proposal** with the source and export date, never as ground truth, which fits the existing owner-approval rule in `GLOSSARY.md`.
8. **Privacy and public-repo hygiene.** Exports contain sensitive account data [V for ChatGPT]. The imports folder must be git-ignored and live outside tracked paths; test fixtures must be synthetic and hand-written from the format facts above. Links expire after 24 hours for Claude and ChatGPT [V], so the owner should download promptly.
9. **Freshness.** All export schemas can change without notice (secondary sources report Claude's format changed over time [U]). Version the adapters, store the detected format signature with each import, and re-verify this note when an adapter breaks.
10. **Open items for the owner's first real exports** (do not commit results, only record format facts here): Claude ZIP file list and whether memory/projects are present; ChatGPT ZIP file list, attachment pointer fields, whether memory appears; the actual Grok download (delivery, expiry, file names, schema, whether memory is included); whether xAI privacy-portal or Cursor exports cover Grok Bot.

## Source list

Vendor, read directly:
- https://support.claude.com/en/articles/9450526-how-can-i-export-my-claude-data
- https://support.claude.com/en/articles/13346720-export-your-organization-s-data
- https://support.claude.com/en/articles/12123587-importing-and-exporting-your-memory
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://support.claude.com/en/articles/11139144-use-claude-for-education-at-your-university
- https://help.openai.com/en/articles/7260999 (Exporting your ChatGPT history and data)
- https://help.openai.com/en/articles/8167885 (export from the Android app)
- https://help.openai.com/en/articles/9106926-transfer-exported-conversations-between-chatgpt-accounts
- https://help.openai.com/en/articles/8590148-memory-faq
- https://help.openai.com/en/articles/7730893-data-controls-in-chatgpt
- https://help.openai.com/en/articles/8801890-can-i-migrate-or-merge-my-personal-chatgpt-workspace-over-to-my-chatgpt-business-workspace

Vendor, read via Internet Archive snapshots (direct access blocked in the research environment):
- https://x.ai/legal/faq , https://x.ai/legal/privacy-policy , https://x.ai/legal/terms-of-service , https://x.ai/privacy-portal , https://x.ai/news/grok-build-memory
- https://docs.x.ai/grok-bot/overview , /bots , /faq , /files-and-results , /security , /teams-and-enterprises , /team-bots , /mobile
- https://help.x.com/en/managing-your-account/how-to-download-your-x-archive , https://help.x.com/en/using-x/about-grok

Secondary, used only to flag unverified claims:
- https://saveai.net/blog/how-to-export-claude-conversations
- https://ai-chat-importer.com/blog/how-to-export-claude-conversations
- https://ai-chat-importer.com/guides/chatgpt-export-format-explained
- https://levelup.gitconnected.com/your-chatgpt-history-is-a-tree-not-a-transcript-2026-5660ceb301d4
- https://www.llmnesia.com/blog/how-to-export-grok-conversation-history
- https://ai-chat-importer.com/blog/how-to-export-grok-conversations
- https://ai-toolbox.co/ai-toolbox-grok-features/export-grok-conversations-txt-markdown-json-pdf-2026
- https://techcrunch.com/2025/04/16/xai-adds-a-memory-feature-to-grok
- https://www.anuma.ai/blog/grok-memory
- https://www.unite.ai/xai-launches-grok-bot-always-on-ai-teammates-with-their-own-cloud-computers/
