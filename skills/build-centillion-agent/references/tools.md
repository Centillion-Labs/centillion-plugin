# Centillion MCP tools

Every tool answers a refusal as a tool error whose message says why and what
to do. Long lists page with `cursor`: pass back the `nextCursor` of one answer
to get the next page.

## Reading

| Tool | Arguments | Returns |
| --- | --- | --- |
| `list_agents` | `limit` (1 to 50), `cursor` | Agents, newest first: id, role, model, training status. |
| `get_agent` | `agentId` | Main's published version, never a draft: instructions (clipped at 6,000 characters, with `instructionsTruncated`), model, greeting, guardrails (clipped at 2,000, no flag; a clipped text ends in `…`), workflow steps, actions, tools, knowledge, tests (`id`, `name`, `revision`). An agent with no published version answers with a note saying so. |
| `list_branches` | `agentId` | Branches: `id`, `name`, `isMain`, `revision`, `trafficPercent`, `currentVersionId`, `yourDraftChanges`. |
| `get_workflow` | `agentId`, `branchId` | The whole workflow on the branch's current version, with the branch revision. |
| `get_voice_settings` | `agentId`, `branchId` | The branch's voice settings and revision. `sharedFields` change only in the dashboard. |
| `list_tools` | `limit` (1 to 100), `cursor` | Workspace tools: `id`, `name`, `description`, `kind`, `revision`, `yourAccess`, `agentIds`, `updatedAt`, `headerNames`, and for an `MCP_SERVER` tool `offeredTools` (what discovery found) and `allowedTools` (the enabled ones). Other settings and header values are never returned. |
| `list_batch_calls` | `agentId`, `limit`, `cursor` | Batches of outbound calls: id, name, agent, status, recipient count. |
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
| `save_workflow` | `agentId`, `branchId`, `revision`, `workflow`; optional `message` | `workflow` is the whole workflow in the shape `get_workflow` returns. Quiet branches only. |
| `update_voice_settings` | `agentId`, `branchId`, `revision`, `settings`; optional `message` | `settings` named as `get_voice_settings` names them. Quiet branches only. |
| `create_tool` | `name`, `configuration`; optional `description` | Shared with the caller only. |
| `update_tool` | `toolId`, `revision`; optional `name`, `description`, `configuration` | Leaving `configuration` out keeps every setting. Sent, it replaces all settings, and a field left out takes its default (an omitted `waitForResponse` becomes `false`). No tool reads a tool's settings back, so send a `configuration` only for a tool you created in this conversation and nobody changed since. A header left out is removed; a header sent as `••••••••` keeps its stored value. The mask is refused for a header with no stored value: send that one as `""`. |
| `discover_mcp_tools` | `toolId` | For an `MCP_SERVER` tool: asks its server which tools it offers now, keeps the enabled ones it still offers (an enabled one no longer offered is dropped from the shared tool; published versions keep the copy they were published with), and answers like `list_tools` with the new `revision`. A new `MCP_SERVER` tool has none enabled. |
| `save_test` | `agentId`, `configuration`; optional `branchId` (Main when left out); `testId` and `revision` to replace a test | Writes the test into your draft on that branch and publishes nothing; it ships with the draft through `request_publish`. Answers with the test's `id`, `branchId` and new `revision`. Save tests on the branch you run them on. Replacing a test replaces its whole configuration, which no tool reads back. |
| `run_tests` | `agentId`, `testIds`; optional `branchId` (Main when left out), `repeats` (1 to 10), `allowMessageCredits` | Runs on your own draft on the branch, or on its current version when you have no draft there; never on another person's draft. Answers at once with `queued` and `target`: `{ kind: "your draft", draftFingerprint }` or `{ kind: "current version", versionId }`, plus `branchId` and `branchName`. Queued: `suiteId` and `runIds`, plus `skipped` naming each test left out because a run of it has not finished, with the reason. Not queued: a `note` and the credit `estimate`. A test pinned to a version, or with agent settings of its own, runs as it is set up and is not recorded as a run on the draft. |

A quiet branch is one other than Main with no share of traffic, no phone
number pinned to it and no batch of calls that has not finished.

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
it in the dashboard. `request_publish` also answers with the
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

## Refusals worth knowing

A refusal arrives as the message alone. The common ones:

| Situation | What to do |
| --- | --- |
| The person's role cannot view or change agents. | An owner changes the role. Viewers can only read. |
| The workspace does not let assistants read conversations. | Pass on the link to the setting. Only an owner can turn it on. |
| `save_workflow` or `update_voice_settings` on Main, or on a branch with traffic, a pinned phone number or a batch of calls. | Make the change on a new branch (`create_branch`), then `request_traffic_change`. |
| A voice setting that is shared by every branch (recordings, call limits). | The person changes it in the dashboard. |
| `update_agent_draft` with no setting, or `request_publish` with a draft that has no changes. | Send a change first. |
| Traffic for a branch with no published version, or a split that is already in place. | Publish the branch first, or leave traffic as it is. |
| A test made from a conversation. | The person creates it in the dashboard. |
| The agent has no Main branch yet. | The person opens the agent in the dashboard once, then retry. |
| The branch, test or tool changed since it was read. | Never resend the old payload with the new `revision`. Read it again and redo the change on what you read (`get_workflow`, `get_voice_settings`, `list_branches`). A tool's or test's settings cannot be read back: the person reconciles them in the dashboard. |
