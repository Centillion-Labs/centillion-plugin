# Centillion MCP tools

Every tool answers a refusal as a tool error whose message says why and what
to do. Long lists page with `cursor`: pass back the `nextCursor` of one answer
to get the next page.

## Reading

| Tool | Arguments | Returns |
| --- | --- | --- |
| `list_agents` | `limit` (1 to 50), `cursor` | Agents, newest first: id, role, model, training status. |
| `get_agent` | `agentId` | Main's published version, never a draft: instructions (clipped at 6,000 characters, with `instructionsTruncated`), model, greeting, guardrails (clipped at 2,000, no flag; a clipped text ends in `…`), workflow steps, actions, tools, knowledge, tests (`id`, `name`, `revision`). An agent with no published version answers with a note saying so. |
| `list_branches` | `agentId` | Branches: `id`, `name`, `isMain`, `revision`, `trafficPercent`, `currentVersionId`, `yourDraftChanges`, `versionsAheadOfMain`, `versionsBehindMain`. |
| `get_workflow` | `agentId`, `branchId` | The whole workflow on the branch's current version, with the branch revision. |
| `get_voice_settings` | `agentId`, `branchId` | The branch's voice settings and revision. `sharedFields` (recordings and call limits) are shared by every branch and change only through `request_voice_shared_settings`. |
| `list_tools` | `limit` (1 to 100), `cursor` | Workspace tools: `id`, `name`, `description`, `kind`, `revision`, `yourAccess`, `agentIds`, `updatedAt`, `headerNames`, `auth` for a tool that signs in with OAuth (`type`, `tokenUrl`, `clientId`, `scope`, never the client secret), and for an `MCP_SERVER` tool `offeredTools` (what discovery found) and `allowedTools` (the enabled ones). Other settings and header values are never returned. |
| `list_batch_calls` | optional `agentId`, `limit` (1 to 50), `cursor` | Batches of outbound calls, newest first: id, name, agent, status, recipient count, and `total`. Recipients are never returned. |
| `list_phone_numbers` | none | The workspace's numbers, newest first: `id`, `e164`, `label`, `boughtFromCentillion`, `agent` (`null` when it answers for none) and `routing`: the agent's traffic split, or the branch it is pinned to. Carrier credentials are never returned. |
| `get_message_settings` | none | `templates` (the text a Messages · Send action sends, with placeholders such as `{link}`, `{first_name}` and `{opt_out}`), `optOutLines`, `activeSenders` (whether an SMS or email sender is on) and `eSignatureConnections` (`id`, `name`). No credential is returned. |
| `list_knowledge_sources` | `agentId` | The sources on the agent's Knowledge page in the dashboard editor, newest first: `id`, `type`, `name`, `url`, `status`, `importing`, `pageCount`, `progress`, `lastError`, `trained`, `trainedAt`, `removed`, `updatedAt`. Never the text. `get_agent` shows the published version's sources. |
| `list_conversations` | `agentId`, `startDate`, `endDate`, `limit`, `cursor` | Conversation ids, newest first. Needs the conversation setting. |
| `get_conversation` | `agentId`, `conversationId`, `limit`, `cursor` | Messages, newest page first, and the analysis. Needs the conversation setting. |
| `get_test_results` | `agentId`, `testId`, `limit` (1 to 20), `cursor` | Test runs: `testName`, `versionId`, `target`, `repeats`, `passed`, `failed`, `noVerdict`, `running`, `reasonsHidden`, and per repeat `status`, `reason`, `missingActions`, `unexpectedActions`, `error`. `target` is `{ kind: "draft", branchId, yourDraft, draftFingerprint }`, `{ kind: "branch", branchId, versionId }`, or `null` for a run started outside an assistant or one that did not run on the queued draft. |
| `search_docs` | `query` | Help page sections that match, with links. |

## Building

| Tool | Arguments | Notes |
| --- | --- | --- |
| `create_agent` | `name`; optional `role` (`customer-support`, `sales-agent`, `shopping-assistant`), `instructions` (20 characters or more), `model` | Starts with a Main branch. Answers only through the API until a channel is connected in the dashboard. |
| `update_agent_draft` | `agentId`, `revision`; optional `branchId` (Main when left out), `instructions`, `guardrails`, `greeting`, `model`, `temperature` (0 to 1), `defaultLanguage` (a code such as `en`, or `null`) | Only the settings sent change, and a text setting is replaced whole. Answers with the new `revision` and the draft, its instructions clipped at 2,000 characters and guardrails at 1,000. |
| `create_branch` | `agentId`, `name`; optional `description`, `fromBranchId` (Main when left out) | A new branch answers no conversations. |
| `save_workflow` | `agentId`, `branchId`, `revision`, `workflow`; optional `message` | `workflow` is the whole workflow in the shape `get_workflow` returns, checked as the dashboard's workflow builder checks it. Answers `savedAs: "version"` with the `versionId`, or `savedAs: "draft"`. |
| `update_voice_settings` | `agentId`, `branchId`, `revision`, `settings`; optional `message` | Only the settings sent change, named as `get_voice_settings` names them. Recordings and call limits go through `request_voice_shared_settings`. Answers like `save_workflow`. |
| `create_tool` | `name`, `configuration`; optional `description` | Shared with the caller only. |
| `update_tool` | `toolId`, `revision`; optional `name`, `description`, `configuration` | Leaving `configuration` out keeps every setting. Sent, it replaces all settings, and a field left out takes its default (an omitted `waitForResponse` becomes `false`). No tool reads a tool's settings back, so send a `configuration` only for a tool you created in this conversation and nobody changed since. A header left out is removed; a header sent as `••••••••` keeps its stored value. The mask is refused for a header with no stored value: send that one as `""`. |
| `discover_mcp_tools` | `toolId` | For an `MCP_SERVER` tool: asks its server which tools it offers now, keeps the enabled ones it still offers (an enabled one no longer offered is dropped from the shared tool; published versions keep the copy they were published with), and answers like `list_tools` with the new `revision`. A new `MCP_SERVER` tool has none enabled. |
| `save_test` | `agentId`, `configuration`; optional `branchId` (Main when left out); `testId` and `revision` to replace a test | Writes the test into your draft on that branch and publishes nothing; it ships with the draft through `request_publish`. Answers with the test's `id`, `branchId` and new `revision`. Save tests on the branch you run them on. Replacing a test replaces its whole configuration, which no tool reads back. |
| `run_tests` | `agentId`, `testIds`; optional `branchId` (Main when left out), `repeats` (1 to 10), `allowMessageCredits` | Runs on your own draft on the branch, or on its current version when you have no draft there; never on another person's draft. Answers at once with `queued` and `target`: `{ kind: "your draft", draftFingerprint }` or `{ kind: "current version", versionId }`, plus `branchId` and `branchName`. Queued: `suiteId` and `runIds`, plus `skipped` naming each test left out because a run of it has not finished, with the reason. Not queued: a `note` and the credit `estimate`. A test pinned to a version, or with agent settings of its own, runs as it is set up and is not recorded as a run on the draft. |
| `save_message_template` | `name`, optional `templateId` (to replace one), `language` (`en` when left out), `smsBody` (up to 1,600 characters), `emailSubject`, `emailBody` | Needs SMS text, email text or both. Replaces a template whole. Refused for a template that a connector action, or any published version of any agent, already sends: a send reads the template when it runs, so changing it would reach customers without a publish. `smsOptOut` in the answer says whether the SMS text tells the reader how to opt out. |
| `create_connector_action` | `action`; optional `name`, `description` | `action.type` is `messaging.send` (a `templateName` and `language` from `get_message_settings`, sent by SMS, email or the step's channel) or one of the `esign.*` types with a `connectionId` from `get_message_settings`. Shared with the caller only. It runs once a workflow Tool step uses it and that version is published. Change or archive it in the dashboard. |
| `add_knowledge_text` | `agentId`, `name`, `content` (up to 1 MB) | Adds a text source and queues training. |
| `add_knowledge_website` | `agentId`, `url`; optional `name` | Imports one public page in the background; `list_knowledge_sources` shows its progress. A whole site is crawled from the dashboard. |
| `create_email_sender` | `agentId` | Gives the agent its generated email address, with Helpdesk for the mail it receives and its replies off. An agent that has an address keeps it. Answers with `address`, `status` and `settingsUrl`. |

`save_workflow` and `update_voice_settings` become the branch's next version
at once on a quiet branch: one other than Main with no share of traffic, no
phone number pinned to it and no batch of calls. On Main, on a branch with
traffic or a pinned number, and on a branch set up for a batch not launched
yet, they go into your draft on that branch and answer `savedAs: "draft"`;
the draft ships through `request_publish`. Both refuse while a launched batch
on the branch has not finished.

Knowledge added with `add_knowledge_text` or `add_knowledge_website` joins the
agent's knowledge in the dashboard editor, not your draft. Conversations use
it once someone next publishes the agent from the dashboard editor;
`request_publish` does not carry it.

### Tool configuration

`configuration.kind` is one of `HTTP_TOOL`, `CLIENT_TOOL`, `MCP_SERVER` or
`CODE_TOOL`. Knowledge is not a tool kind here, and integration actions come
from an integration connected in the dashboard.

- `HTTP_TOOL`: `url` (HTTPS, no credentials in it), `method` (`GET`, `POST`, `PUT`, `PATCH`,
  `DELETE`), `headers`, `parameters`, and optional `timeoutMs`,
  `bodyEncoding` (`json` or `form`), `responseFields`.
- `CLIENT_TOOL`: `parameters`, `waitForResponse`, `timeoutMs`.
- `MCP_SERVER`: `url`, `headers`, `allowedTools`.
- `CODE_TOOL`: `code`, `parameters`, `timeoutMs` (50 to 5000).

Each parameter has a `name`, a `type` (`string`, `number`, `boolean`,
`integer`, `object`, `array`), a `description`, and optional `location`
(`path`, `query`, `body`), `required`, `enumValues`, `properties` and `items`.

Give a header that carries a credential an empty value (`""`) and let the
person enter the real value in the dashboard. Never write a credential into
`headers` yourself.

### Test configuration

Every test needs `name`, `maxTurns` (1 to 50), `expectedActions` (the ids of
the actions the agent must call, `[]` for none) and `actionResponses` (`[]`
for none). `testType` is `simulation` when left out:

- `simulation`: also needs `simulatedUser` (who the simulated customer is and
  what they want) and `successCriterion` (what the judge checks).
- `next_reply`: needs `messages` ending with a `user` message, and a
  `successCriterion`.
- `tool_invocation`: needs `messages` ending with a `user` message, and checks
  the actions in `expectedActions`.

An action id the agent no longer has is refused. A configuration with
`sourceConversation` is refused: those tests are made in the dashboard.

## Asking for approval

Each of these checks the request the way the dashboard would, then answers
with `status: "waiting_for_approval"`, a `summary`, an `approvalUrl` and an
`expiresAt` 24 hours later. Nothing changes until an Owner or Member approves
it in the dashboard, and a request whose subject changed after it was made can
no longer be approved. `request_publish` also answers with the
`draftFingerprint` and `testsOnThisDraft`: for each test, the latest run on
that exact draft (`testId`, `testName`, `ranAt`, `passed`, `failed`,
`noVerdict`, `running`), or `"not tested"`. The approval page shows the same
results.

| Tool | Arguments | Asks for |
| --- | --- | --- |
| `request_publish` | `agentId`; optional `branchId` (Main when left out), `message` | Publishing the caller's draft on the branch as its next version. |
| `request_traffic_change` | `agentId`, `allocations` (`branchId` and `percent` each, adding up to 100) | Splitting conversations between branches. Branches left out get none. |
| `request_batch_launch` | `batchId` | Launching a batch of outbound calls as it is set up now. |
| `request_archive` | `target`: `{ kind: "branch", agentId, branchId }` or `{ kind: "tool", toolId }` | Archiving a branch or a workspace tool. |
| `request_phone_number` | `agentId`; optional `areaCode` (three US digits), `e164` (from `otherNumbers` in an earlier answer), `label`, `autoRenew` (`false` when left out) | Buying a US local number from Centillion, paid monthly from prepaid credits. Answers also with `number` (`e164`, `place`, `monthlyPrice`) and up to five `otherNumbers`. A bought number answers calls once `request_phone_assignment` assigns it. |
| `request_phone_assignment` | `phoneLineId` (from `list_phone_numbers`), `agentId`; optional `branchId` | Having the number answer for the agent: following its traffic split, or pinned to the published branch given. |
| `request_voice_shared_settings` | `agentId`, `settings`: any of `retainRecordingsForever`, `recordingRetentionDays` (1 to 3650, or `null`), `dailyCallLimit`, `maxConcurrentSessions` (a positive whole number, or `null` for no limit) | Changing settings every branch shares. They apply at once, outside any version. To delete recordings after some days, send `retainRecordingsForever: false` with `recordingRetentionDays`. |
| `request_channel_activation` | `agentId`; optional `connectionId` (the agent's only paused email sender when left out) | Turning on automatic replies from the agent's email sender. Until then a Messages · Send action by email answers `no_sender`. |

## Refusals worth knowing

A refusal arrives as the message alone. The common ones:

| Situation | What to do |
| --- | --- |
| The person's role cannot view or change agents. | An owner changes the role. Viewers can only read. |
| The workspace does not let assistants read conversations. | Pass on the link to the setting. Only an owner can turn it on. |
| `save_workflow` or `update_voice_settings` on a branch whose launched batch of calls has not finished. | Wait for the batch to finish, or make the change on a new branch (`create_branch`). |
| A voice setting that is shared by every branch (recordings, call limits) sent to `update_voice_settings`. | Ask for it with `request_voice_shared_settings`. |
| `save_message_template` for a template a connector action or a published version already sends. | The person changes it under **Integrations** in the dashboard. |
| `update_agent_draft` with no setting, or `request_publish` with a draft that has no changes. | Send a change first. |
| Traffic for a branch with no published version, or a split that is already in place. | Publish the branch first, or leave traffic as it is. |
| A test made from a conversation. | The person creates it in the dashboard. |
| The agent has no Main branch yet. | The person opens the agent in the dashboard once, then retry. |
| The branch, test or tool changed since it was read. | Never resend the old payload with the new `revision`. Read it again and redo the change on what you read (`get_workflow`, `get_voice_settings`, `list_branches`). A tool's or test's settings cannot be read back: the person reconciles them in the dashboard. |
