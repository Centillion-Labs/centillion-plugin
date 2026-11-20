# Centillion MCP tools

Generated from the server's tool list. Every tool answers a refusal as a
tool error whose message says why and what to do. Long lists page with
`cursor`: pass back the `nextCursor` of one answer to get the next page.
The same tools are commands of the `centillion` CLI:
`centillion <tool> --json '{...}'`.

## Reading

These tools change nothing.

### `get_workspace` — Get workspace

Show the Centillion workspace this assistant is connected to: id, name, plan, this month's credits as the account menu shows them, your role in it and whether it lets you change agents, whether assistants may read conversations, and the dashboard's address.

No arguments.

### `list_models` — List models

List the models an agent can use, as the agent editor's model picker lists them. models are the ids create_agent and update_agent_draft take as model, and update_voice_settings takes as model in the Custom voice mode; only an available one can be chosen. realtimeModels are the ids update_voice_settings takes as realtimeModel in the Realtime voice mode.

No arguments.

### `list_agents` — List agents

List the AI agents in the connected Centillion workspace, newest first, with their role, model and training status. Pass an agent's id to get_agent, list_conversations or get_test_results.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_agent` — Get agent

Show how one agent is set up in its published version: instructions, model, greeting, guardrails, the workflow's steps, its actions and tools, knowledge sources and tests. Long instructions and guardrails are cut short (instructionsTruncated); read_agent_text returns them whole, and your draft. Tool settings and secrets are never returned; get_tool reads one tool with its secrets masked.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `read_agent_text` — Read agent text

Read the whole of an agent's instructions, guardrails or greeting on a branch: your own draft when you have one, else the branch's current version (source says which). Long texts come in pages: call again with offset set to nextOffset until it is null. Answers with the branch revision and a fingerprint of the whole text, which edit_agent_text and append_agent_text take.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |
| `field` | `instructions` \| `guardrails` \| `greeting` | yes | Which text: instructions, guardrails or greeting. |
| `offset` | integer (min 0) |  | Where to start, in characters: nextOffset of the last page. |
| `limit` | integer (min 1, max 50,000) |  | How many characters to return, 1 to 50000. |

### `get_agent_link` — Get agent link

Links for a person to try an agent. previewUrl opens its Preview in the dashboard, for members of the workspace: it runs the draft of whoever opens it on that branch, or the branch's current version when they have none, so your draft when you open it yourself. publicPage is the agent's own web page that anyone can chat on, with its published version, when its Help page channel is on; otherwise it says where a person turns it on. Nothing is turned on here.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch the Preview opens on. Main when left out. |

### `get_embed_code` — Get embed code

The code that puts an agent on a website, as its Channels page shows it under Install: the chat bubble widget's script and the inline iframe. Both answer only while the agent's Chat bubble channel is on (chatBubbleOn), and only on the domains in allowedDomains when the agent restricts them. Conversations get the published version. Nothing is turned on here; when something is off, the answer says where a person changes it.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `list_conversations` — List conversations

List one agent's website chat, API and phone conversations, newest first, optionally between two times. Messaging-channel threads are not included yet. Returns ids for get_conversation, not transcripts. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `startDate` | string |  | Only conversations started at or after this ISO 8601 time. |
| `endDate` | string |  | Only conversations started at or before this ISO 8601 time. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_conversation` — Get conversation

Read one conversation: its latest messages, a page at a time going back, and its analysis when it has one. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `conversationId` | string (min length 1, max length 128) | yes | The conversation's id, from list_conversations. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_test_results` — Get test results

Show an agent's latest test runs, newest first: which test, what it ran on (target: your draft of a branch with its fingerprint, or a branch's version), how many repeats passed, failed or gave no verdict, and the judge's reason for each. Simulated transcripts are left out. For a test made from a customer conversation, the reasons are left out (reasonsHidden) unless the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `testId` | string (min length 1, max length 128) |  | Only the runs of this test. |
| `limit` | integer (min 1, max 20) |  | How many to return, 1 to 20. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_branches` — List branches

List an agent's branches: id, name, revision, share of conversations (trafficPercent), current version, and how many changes your own draft on each one holds (yourDraftChanges). That is the draft request_publish publishes. The dashboard's unpublished-changes count reads what is open in its editor, so the two differ while someone edits there. Writes to a branch take its revision.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `get_workflow` — Get workflow

Read the workflow on a branch's current version, whole, with the branch revision. Edit it and send it back with save_workflow.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |

### `get_voice_settings` — Get voice settings

Read the voice settings a branch's calls run with, and the branch revision. sharedFields are shared by every branch and change only in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |

### `list_tools` — List tools

List the workspace tools you can see: id, name, kind, revision, your access and the agents using each one. Their settings, which can hold secrets, are not listed; get_tool reads one tool's settings with its secrets masked.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_tool` — Get tool

Read one workspace tool you can see, whole: what list_tools returns and its configuration, in the shape update_tool takes. Header values and an OAuth client secret read as ••••••••, which update_tool takes back to keep the stored value; secret headers name workspace secrets by id. Fixed parameter values are returned as set.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `toolId` | string (min length 1, max length 128) | yes | The tool's id, from list_tools. |

### `get_test` — Get test

Read one of an agent's tests whole, in the shape save_test takes it, with its id and revision: from your draft of a branch when you have one, else from the branch's current version (source says which). For a test made from a customer conversation, the replayed messages, the agent's original reply and the conversation's variables are left out (conversationDetailsHidden) unless the workspace allows AI assistants to read conversations; such a test is changed in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `testId` | string (min length 1, max length 128) | yes | The test's id, from get_agent or save_test. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |

### `list_secrets` — List secrets

List the workspace's secrets, newest first: id, name, and the tools and published agents that send each one. Their values are never returned. A tool sends a secret as a header when its configuration's secretHeaders maps the header's name to the secret's id. People add secrets in Workspace settings, General, Secrets.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `search` | string (max length 100) |  | Only secrets whose name starts with this, ignoring case. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_auth_connections` — List auth connections

List the workspace's auth connections, newest first: id, name, type (oauth2_client_credentials, basic, bearer or oauth2_jwt), provider, and the tools and published agents that sign in with each one. Their secrets are never returned. An HTTP_TOOL signs in with a connection when its configuration's authConnectionId is the connection's id, and a BROWSER_FORM's link request when page.authConnectionId is; the connection then sends the Authorization header, so the tool sets neither an Authorization header of its own nor auth. People add connections in Workspace settings, General, Auth connections.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `search` | string (max length 100) |  | Only connections whose name starts with this, ignoring case. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_batch_calls` — List batch calls

List the workspace's batches of outbound calls, newest first: id, name, agent, status and how many recipients. Recipients themselves are not returned.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's batches. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_campaigns` — List campaigns

List the workspace's Campaigns page campaigns, newest first: id, name, agent, status (active, paused, needs_attention, cancelling, cancelled, completed or failed), channel, whether it is kept open for new leads, and its lead counts. Batch calls are listed by list_batch_calls.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's campaigns. |
| `status` | `active` \| `paused` \| `needs_attention` \| `cancelling` \| `cancelled` \| `completed` \| `failed` |  | `active` covers a campaign calling or waiting for its start. `needs_attention` is a campaign that paused itself, such as after repeated call failures; `lastError` says why. `cancelling` is a cancelled campaign whose calls already placed are still finishing. `failed` is a completed campaign none of whose leads could be processed. |
| `search` | string (max length 200) |  | Only campaigns whose name contains this text. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_campaign` — Get campaign

Read one campaign with its lead policy (calling windows, retry limits, follow-up switches) and its progress: leads waiting, on a call, being settled and finished by disposition, the calls waiting by type, and when the next call is due.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) | yes | The campaign's id, from list_campaigns. |

### `list_campaign_leads` — List campaign leads

List a campaign's leads, newest first: id, your own recipientId, state (queued, in_progress or finished), disposition, the next call and the call counters. Numbers, names and variables are included only in a workspace that lets AI assistants read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) | yes | The campaign's id, from list_campaigns. |
| `state` | `in_progress` \| `finished` |  | in_progress for leads that may still be called, finished for the others. |
| `dispositions` | list of `completed` \| `removed` \| `do_not_call` \| `campaign_ended` \| `cancelled` \| `attempt_limit_reached` \| `callback_limit_reached` \| `redirect_limit_reached` \| `permission_not_granted` \| `permission_declined` \| `permission_request_blocked` \| `permission_expired` \| `invalid_callback_time` \| `invalid_redirect_number` \| `could_not_process` (max items 15) |  | Only leads that finished with one of these. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_campaign_attention` — List campaign attention items

List what needs a person across the workspace's campaigns, or one campaign's, newest first: leads that could not be processed, gave a callback time or a number that could not be used, or were set aside for review, and the pauses campaigns put themselves in. Each has its campaign, a reasonCode, what went wrong and whether a retry can help. Resolve or retry them in the dashboard or through the Campaigns API.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) |  | Only this campaign's items; every campaign's when left out. |
| `includeResolved` | boolean |  | Also list items already resolved. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_phone_numbers` — List phone numbers

List the workspace's phone numbers, newest first: id, number, label, whether it was bought from Centillion, the agent it answers for (null when it answers for none), and whether it follows that agent's traffic split or is pinned to a branch. Pass an id to request_phone_assignment. Carrier credentials are never returned.

No arguments.

### `get_message_settings` — Get message settings

Read the workspace's message settings, as the Integrations page shows them: its message templates (the text a Messages · Send action sends, with placeholders such as {link}, {first_name} and {opt_out}), its opt-out lines, whether an SMS or email sender is turned on, and its e-signature connections by id and name. No credential is returned.

No arguments.

### `list_knowledge_sources` — List knowledge sources

List an agent's knowledge sources as its Knowledge page in the dashboard editor shows them, newest first: id, type, name, url, import status and progress, the last error, and whether the agent is trained on the source's current text. Removed sources are listed with removed true. Their text is not returned; get_agent shows the sources of the published version.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `search_docs` — Search Centillion docs

Search Centillion's help pages for how the product works: agents, channels, workflows, tests, the API and plans. Returns the matching sections with links.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string (min length 2, max length 200) | yes | What to look for. |

## Building

These tools change drafts, quiet branches, tools, tests, message templates, knowledge sources and email addresses. None of them reaches a conversation on its own.

### `create_agent` — Create agent

Create an agent in the connected workspace. Its instructions, model, greeting, guardrails and default language go into its first version on Main. It answers only through the API until someone connects a channel in the dashboard, so a person can try it at once with get_agent_link.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string (min length 1, max length 80) | yes | The agent's name. |
| `role` | `customer-support` \| `sales-agent` \| `shopping-assistant` |  | What the agent is for. customer-support when left out. |
| `instructions` | string (min length 20, max length 100,000) |  | Its instructions. The role's template when left out. |
| `model` | string (min length 1, max length 200) |  | A model id from list_models that is available. The default when left out. |
| `greeting` | string (min length 1, max length 500) |  | The message a chat opens with. The default when left out. |
| `guardrails` | string (max length 20,000) |  | Rules the agent always keeps. The default when left out. |
| `defaultLanguage` | string or null |  | A language code such as en or es-MX. None when left out. |

### `update_agent_draft` — Update agent draft

Change your draft of an agent's settings on a branch: instructions, guardrails, greeting, model, temperature, default language. Only the settings you send change, and a text you send replaces the old one whole; to change part of a text, use edit_agent_text or append_agent_text. The draft is yours and reaches no conversation until it is published with request_publish.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `instructions` | string (min length 20, max length 100,000) |  |  |
| `guardrails` | string (max length 20,000) |  |  |
| `greeting` | string (min length 1, max length 500) |  |  |
| `model` | string (min length 1, max length 200) |  |  |
| `temperature` | number (min 0, max 1) |  |  |
| `defaultLanguage` | string or null |  | A language code such as en or es-MX, or null for none. |

### `edit_agent_text` — Edit agent text

Replace an exact passage in your draft of an agent's instructions, guardrails or greeting on a branch (Main when left out), leaving the rest as it is. Read the text first with read_agent_text and send its revision and fingerprint: the edit is refused if the text or the branch changed since, or if find does not appear exactly expectedOccurrences times. Saved in your draft, like update_agent_draft: nothing reaches a conversation until request_publish is approved.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |
| `revision` | integer (min 0) | yes | The branch revision read_agent_text answered with. |
| `field` | `instructions` \| `guardrails` \| `greeting` | yes | Which text: instructions, guardrails or greeting. |
| `fingerprint` | string (min length 1, max length 64) | yes | The fingerprint read_agent_text answered with for the whole text. |
| `find` | string (min length 1, max length 100,000) | yes | The exact passage to replace, as read_agent_text returned it: same case, spaces and line breaks. |
| `replace` | string (max length 100,000) | yes | What goes in its place. Empty to remove it. |
| `expectedOccurrences` | integer (min 1, max 1,000) |  | How many times find appears; every one is replaced. 1 when left out. |

### `append_agent_text` — Append agent text

Add a passage at the end of your draft of an agent's instructions, guardrails or greeting on a branch (Main when left out). Read the text first with read_agent_text and send its revision and fingerprint: the append is refused if the text or the branch changed since. Saved in your draft, like update_agent_draft: nothing reaches a conversation until request_publish is approved.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |
| `revision` | integer (min 0) | yes | The branch revision read_agent_text answered with. |
| `field` | `instructions` \| `guardrails` \| `greeting` | yes | Which text: instructions, guardrails or greeting. |
| `fingerprint` | string (min length 1, max length 64) | yes | The fingerprint read_agent_text answered with for the whole text. |
| `text` | string (min length 1, max length 100,000) | yes | The passage to add at the end. |
| `separator` | string (max length 10) |  | What goes between the current text and yours, when there is one. A blank line when left out. |

### `create_branch` — Create branch

Start a branch of an agent from Main or another branch. A new branch answers no conversations, so its workflow and voice settings can be changed before asking for traffic.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `name` | string (min length 1, max length 140) | yes | The new branch's name. |
| `description` | string (max length 4,000) |  |  |
| `fromBranchId` | string (min length 1, max length 128) |  | The branch to start from. Main when left out. |

### `save_workflow` — Save workflow

Replace a branch's workflow, checked as the dashboard checks it. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) it becomes the branch's next version at once, and earlier versions stay available. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft) and reaches nobody until request_publish is approved. Refused while a launched batch of calls on the branch has not finished.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version. |
| `workflow` | object | yes | The whole workflow, in the shape get_workflow returns. |

### `update_voice_settings` — Update voice settings

Change some of a branch's voice settings, checked as the dashboard's Voice card checks them. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) they become the branch's next version at once. On Main or a branch something answers from, they go into your draft on that branch (savedAs draft) and reach nobody until request_publish is approved. Refused while a launched batch of calls on the branch has not finished. Recordings and call limits are shared by every branch: ask for them with request_voice_shared_settings.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version. |
| `settings` | object | yes | The settings to change, named as get_voice_settings names them. |

### `create_tool` — Create tool

Create a workspace tool, shared with you only, checked as the dashboard checks it. Agents use it once it is added to a workflow step or a branch.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string (min length 1, max length 140) | yes |  |
| `description` | string (max length 8,000) |  |  |
| `configuration` | object | yes | The tool's settings, with its kind, as the developer API's tools endpoints take them. Header and parameter values and an OAuth client secret are stored and never read back. An HTTP_TOOL or MCP_SERVER sends a workspace secret as a header with secretHeaders, { headerName: secretId }, the id from list_secrets; a header is either in headers or in secretHeaders, not both. An HTTP_TOOL can sign in with OAuth 2.0 client credentials in auth (type oauth2_client_credentials, tokenUrl, clientId, clientSecret, an optional space-separated scope, and clientAuthentication basic or body), and then has no Authorization header of its own. Instead of auth, an HTTP_TOOL, or a BROWSER_FORM's link request in page, can sign in with a workspace auth connection: authConnectionId is the connection's id from list_auth_connections, and the request then has neither auth nor an Authorization header of its own. A BROWSER_FORM fills a web form in a browser: page is { from: url, url } or { from: request, method, url, headers, auth, linkPath } (a request whose JSON answer holds the form's link at linkPath), allowedHosts lists the hosts it may open, fields are { label, parameter, type text, checkbox or select, required }, and before and submit are steps { action click or check, target { by role, role, name } or { by label, label } or { by css, selector } }. |

### `update_tool` — Update tool

Change a workspace tool's name, description or settings. Published versions keep the copy they were published with, so agents change only when a new version is published.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `toolId` | string (min length 1, max length 128) | yes | The tool's id, from list_tools. |
| `revision` | integer (min 0) | yes | The tool revision you read, from list_tools. |
| `name` | string (min length 1, max length 140) |  |  |
| `description` | string (max length 8,000) |  |  |
| `configuration` | object |  | Replaces all of the tool's settings. A header left out is removed, and so are OAuth sign-in and an auth connection. To keep a header's stored value, send its name (headerNames in list_tools) with the value ••••••••; to keep secret headers, send secretHeaders from list_tools. To keep OAuth sign-in, send auth from list_tools with clientSecret ••••••••; a new token URL or server origin needs the real secret. To keep an auth connection, send authConnectionId from list_tools. A BROWSER_FORM keeps both in page, its link request. |

### `discover_mcp_tools` — Discover MCP server tools

Ask an MCP_SERVER tool's server which tools it offers now, and keep the enabled ones it still offers. Returns the offered tool names.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `toolId` | string (min length 1, max length 128) | yes | The tool's id, from list_tools. |

### `save_test` — Save test

Create an agent test, or replace one with testId and its revision, in your draft of a branch (Main when left out). Nothing is published: run it on the draft with run_tests, and it ships with the draft through request_publish.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft gets the test. Main when left out. |
| `testId` | string (min length 1, max length 128) |  | The test to change, from get_agent or get_test. Left out, a new test. |
| `revision` | integer (min 0) |  | The test's revision from get_test or get_agent, to change a test. |
| `configuration` | object | yes | The whole test, as the developer API's agent tests endpoints take it. Required: name, maxTurns (1 to 50), expectedActions (the tool ids the agent must call; [] for none) and actionResponses (the replies those tools give in the test; [] for none). By testType (simulation when left out): simulation also needs simulatedUser and successCriterion; next_reply needs messages ending with a user message and successCriterion; tool_invocation needs messages ending with a user message. |

### `run_tests` — Run tests

Run some of an agent's tests on your draft of a branch (Main when left out), so a change is tested before you ask to publish it. Without a draft they run on the branch's current version. It answers at once with what was tested, and names in skipped each test left out because a run of it has not finished; read the results later with get_test_results.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch to test: your draft on it when you have one, else its current version. Main when left out. |
| `testIds` | list of string (max items 100) | yes | The tests to run, from get_agent. |
| `repeats` | integer (min 1, max 10) |  |  |
| `allowMessageCredits` | boolean |  | Run even when it uses message credits. The first answer says when it would. |

### `save_message_template` — Save message template

Create a message template, or replace one whole with templateId, as the Integrations page saves it. It needs the text for SMS, email or both. A send reads its template when it runs, so a template that a connector action or any published version of an agent already names is changed only in the dashboard. smsOptOut in the answer says whether the SMS text tells the reader how to opt out.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string (min length 1, max length 140) | yes | The name a Messages · Send action picks the template by. |
| `language` | string (min length 2, max length 16) |  | A language code such as en or es. en when left out. A send uses the template in the conversation's language, else the English one. |
| `smsBody` | string (max length 1,600) |  | The SMS text, up to 1600 characters. Empty when left out. Place the opt-out line with {opt_out}. |
| `emailSubject` | string (max length 200) |  | The email subject. Empty when left out. |
| `emailBody` | string (max length 20,000) |  | The email text. Empty when left out. |
| `templateId` | string (min length 1, max length 128) |  | The template to replace, from get_message_settings. Left out, a new template. |

### `create_connector_action` — Create connector action

Create a connector action tool (kind INTEGRATION_ACTION), shared with you only, checked as the dashboard checks it: a typed operation such as Messages · Send that a workflow's Tool step runs. Its credentials stay on the workspace's connection, never on the tool. It runs only once a workflow step uses it and that version is published; save_workflow and request_publish do that. Change or archive it in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string (min length 1, max length 140) |  | The tool's name. The action's own name, such as Messages · Send, when left out. |
| `description` | string (max length 4,000) |  |  |
| `action` | object | yes | The connector action, by type. messaging.send sends a template from get_message_settings (templateName, language) by sms, email or the channel the step names (input); recipient contact goes through the opt-out, do-not-call and consent checks, and recipient staff goes only to staffAddress, the workspace's own number or address. The esign.* types take connectionId, an e-signature connection from get_message_settings. The hubspot type takes connectionId, a HubSpot connection from list_integrations, and tool, one of the HubSpot tools such as find_contact_by_email or log_call. |

### `add_knowledge_text` — Add knowledge text

Add a text snippet to an agent's knowledge, as its Knowledge page adds one, and queue the agent's training. It joins the agent's knowledge in the dashboard editor, which trains on it, and conversations use it once the agent is next published from the dashboard editor; request_publish publishes your own draft, which does not hold it.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `name` | string (min length 1, max length 250) | yes | The source's name. |
| `content` | string (min length 1, max length 1,000,000) | yes | The text the agent learns, up to 1 MB. |

### `add_knowledge_website` — Add knowledge website

Import one public web page into an agent's knowledge, as its Knowledge page imports a single page, and queue the agent's training once it is read. The import runs in the background: list_knowledge_sources shows its progress. To crawl a whole site, add it in the dashboard. It joins the agent's knowledge in the dashboard editor, which trains on it, and conversations use it once the agent is next published from the dashboard editor; request_publish publishes your own draft, which does not hold it.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `url` | string (min length 1, max length 2,048) | yes | The public page to import, such as example.com/pricing. |
| `name` | string (min length 1, max length 250) |  | The source's name. The page's address when left out. |

### `create_email_sender` — Create email sender

Give an agent its generated email address, as its Email settings do: Helpdesk for the mail it receives, then the address, with its replies off. An agent that has an address keeps it. A Messages · Send action sends email only from an address whose replies are on (status ACTIVE), and turning them on reaches customers, so ask for it with request_channel_activation, which a person approves; until then the send answers no_sender.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

## Running campaigns

A campaign calls people, so `create_campaign` sets one up and returns an approval link. Once a person launches it, the others steer it.

### `create_campaign` — Create campaign

Set up a Campaigns page campaign with its leads, as the Campaigns API's create request takes them: the agent, a phone number or WhatsApp account, up to 1,000 recipients, the start, the lead policy groups to use instead of the workspace's, and keepOpen for one that takes leads later. Nobody is called yet: the campaign is saved as a draft and this returns a link where a person approves its launch, agreeing to the outbound terms as they do. A WhatsApp template is checked with Meta now.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string (min length 1, max length 120) | yes |  |
| `agentId` | string (min length 1, max length 128) | yes |  |
| `channel` | object | yes |  |
| `recipients` | list of object (max items 1,000) |  | Up to 1,000 leads. Send more, or send them over time, to a campaign created with `keepOpen` through the leads endpoint. |
| `startsAt` | string or null |  | When calling may start; left out, at once. Calls still wait for a calling window. |
| `timezone` | string (min length 1, max length 64) |  | The IANA timezone the calling windows are read in, instead of the policy's. |
| `branchId` | string or null |  | A published branch of the agent to call with; left out, the agent's traffic split decides. |
| `concurrencyLimit` | integer or null |  |  |
| `callsPerMinute` | integer (min 1, max 20) |  |  |
| `allowedOverrides` | list of `prompt` \| `first_message` \| `language` \| `voice_id` (max items 4) |  |  |
| `variant` | `simple` \| `advanced` |  | Left out, `advanced`, or `simple` for a WhatsApp text campaign, which sends each lead one message. |
| `policy` | object |  | Policy groups to use instead of the workspace's: each group sent replaces that whole group of the policy the workspace settings give the agent. |
| `keepOpen` | boolean |  | Keep the campaign open for leads added after it starts: it does not complete when every lead is finished, only when it is cancelled. Only an open campaign takes new leads. |
| `idempotencyKey` | string (min length 1, max length 128) |  | Makes the request safe to retry, as the `Idempotency-Key` header does; send one or the other. Without either, a request repeating an earlier one's body exactly returns that campaign. |

### `add_campaign_leads` — Add campaign leads

Add leads to a campaign created with keepOpen that has not finished. A person approved that campaign's launch knowing it takes leads later, so new leads are called like the others, inside the calling windows. Only an open campaign takes leads.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) | yes | The campaign's id, from list_campaigns. |
| `leads` | list of object (max items 200) | yes | Up to 200 leads, as the create request's recipients. A lead whose recipientId, or number when it has none, is already in the campaign is returned as it is. |

### `pause_campaign` — Pause campaign

Pause a running or scheduled campaign: it places no new calls until resumed, and calls already under way carry on. A paused campaign answers as it is.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) | yes | The campaign's id, from list_campaigns. |

### `resume_campaign` — Resume campaign

Resume a paused campaign, including one that paused itself once its cause is fixed: it calls again inside its calling windows, or waits for a start still ahead.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) | yes | The campaign's id, from list_campaigns. |

### `cancel_campaign` — Cancel campaign

Cancel a campaign that has not finished: it calls nobody else, every waiting lead finishes as cancelled, and a call under way finishes on its own. This can't be undone.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `campaignId` | string (min length 1, max length 128) | yes | The campaign's id, from list_campaigns. |

## Asking for approval

Each request tool checks the request the way the dashboard would, then returns a link where an Owner or Member approves or declines it. Nothing changes until then; requests expire after 24 hours.

### `request_publish` — Request publish

Ask for your draft on a branch to be published as its next version. Nothing is published yet: this returns a link where a person approves or declines it, and shows them the changes and the test runs made on this exact draft with run_tests.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch to publish your draft on. Main when left out. |
| `message` | string (max length 1,000) |  | A short note for the new version. |

### `request_traffic_change` — Request traffic change

Ask for an agent's conversations to be split between its branches. Nothing changes yet: this returns a link where a person approves or declines it. Each branch keeps the version it has now; if one gets a new version first, the request can no longer run.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `allocations` | list of object (max items 30) | yes | Each branch's share of conversations, adding up to 100. Branches left out get none. |

### `request_batch_launch` — Request batch launch

Ask for a batch of outbound calls to be launched as it is set up now. Nothing is called yet: this returns a link where a person approves or declines it. If the batch is edited before then, the request can no longer run.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `batchId` | string (min length 1, max length 128) | yes | The batch's id, from list_batch_calls. |

### `request_archive` — Request archive

Ask for a branch or a workspace tool to be archived. Nothing is archived yet: this returns a link where a person approves or declines it.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `target` | object | yes |  |

### `request_phone_number` — Request phone number

Ask to buy a US local phone number from Centillion for an agent, at the monthly price it is offered at, paid from the workspace's prepaid credits. Nothing is bought yet: this returns a link where a person approves or declines it, and the number and its price. A bought number answers calls once request_phone_assignment assigns it. Numbers from another carrier are imported in the dashboard, since that takes the carrier's credentials.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `areaCode` | string |  | A three-digit US area code to choose the number in. |
| `e164` | string |  | The number to buy, from otherNumbers in an earlier answer. The first number offered when left out. |
| `label` | string (max length 64) |  | A friendly name for the number, such as Support line. |
| `autoRenew` | boolean |  | Renew it every month from prepaid credits. false when left out, as in the dashboard. |

### `request_phone_assignment` — Request phone assignment

Ask for a workspace phone number to answer for an agent, and optionally for one of its branches. Nothing changes yet: this returns a link where a person approves or declines it. If the number is moved, or the pinned branch gets a new version, before then, the request can no longer run.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `phoneLineId` | string (min length 1, max length 128) | yes | The number's id, from list_phone_numbers. |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | Pin the number to this published branch, so every call to it answers there. Left out, the number follows the agent's traffic split. |

### `request_voice_shared_settings` — Request shared voice settings

Ask to change an agent's recording retention or call limits, which every branch shares and which take effect at once, outside any version. Nothing changes yet: this returns a link where a person approves or declines it. If any of these settings is changed before then, the request can no longer run.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `settings` | object | yes | The settings to change, named as get_voice_settings names them under sharedFields: retainRecordingsForever, recordingRetentionDays (1 to 3650 days, or null), dailyCallLimit and maxConcurrentSessions (a positive whole number, or null for no limit). Recordings are kept forever while retainRecordingsForever is true, so send it false with recordingRetentionDays to delete them after that many days. |

### `request_channel_activation` — Request channel activation

Ask to turn on automatic replies from an agent's paused email sender, so the agent starts answering the emails it receives. Nothing is sent yet: this returns a link where a person approves or declines it.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `connectionId` | string (min length 1, max length 128) |  | The email sender's connection id. Left out, the agent's only paused email sender. |

