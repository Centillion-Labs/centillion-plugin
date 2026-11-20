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

### `get_agent_configuration` — Get agent configuration

Read an agent's whole configuration on a branch, in the shape save_draft_configuration takes: settings, workflow and display settings, actions, tools, evaluation criteria, tests, voice settings, knowledge sources and specialists. It is your own draft when you have one, else the branch's current version (source says which), with the branch revision and a fingerprint of what was read; with versionId, that version. Secrets read as masks: a tool's header value or OAuth client secret as ••••••••, an action's header, parameter or webhook secret as __centillion_action_secret_saved__; a save takes them back to keep the stored value. Knowledge sources are listed without their text (get_knowledge_source reads one). For a test made from a customer conversation, its replayed messages, original reply and variables are left out unless the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |
| `versionId` | string (min length 1, max length 128) |  | A version to read instead, such as currentVersionId from list_branches. A version is read as it was published and cannot be saved to. |

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

### `list_channels` — List channels

Where agents are live, newest agent first: each agent's website and app channels that are on (such as chat-bubble, agent-page and api) and its connected messaging accounts (WhatsApp, email, SMS and others) by provider and name, with each account's status and last error. Phone numbers are listed by list_phone_numbers. Nothing is turned on here; channelsUrl is where a person connects or turns one on. No credential is returned.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's channels. Every agent's when left out. |
| `limit` | integer (min 1, max 50) |  | How many agents to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

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

### `search_conversations` — Search conversations

Find logged website chat, API and phone conversations from the last N days, newest first, by words anywhere in them, by channel or by sentiment, with an excerpt around where each matched and a link to it in the dashboard. Each page reads up to 500 conversations: scanned and leftToScan say how many, and nextCursor reads further back. Pass a conversation's agentId and conversationId to get_conversation. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's conversations. |
| `days` | integer (min 1, max 180) |  | How many days back from now, 1 to 180. |
| `search` | string (max length 200) |  | Words that must all appear in a conversation, ignoring case. Left out, every conversation matches. |
| `channel` | string (min length 1, max length 60) |  | Only this channel, such as phone, chat-bubble or api. |
| `sentiment` | `Positive` \| `Neutral` \| `Negative` |  | Only conversations the analysis found this sentiment in. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `semantic_search_conversations` — Search conversations by meaning

Find conversations about something by meaning rather than exact words, as the Conversations page's Smart search does: the closest passages first, each with its conversation, an excerpt of up to 800 characters and a link. It searches only conversations the analysis indexed, which needs Smart search on in an agent's analysis settings. Your query's words, not a conversation's, are sent to the embedding model. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string (min length 2, max length 200) | yes | What the conversations are about, in your own words. |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's conversations. |
| `branchId` | string (min length 1, max length 128) |  | Only conversations this branch answered. Needs agentId. |
| `range` | `today` \| `7d` \| `30d` \| `90d` \| `all` |  | How far back: today, 7d, 30d, 90d or all. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `resolve_conversation_link` — Resolve conversation link

Turn a dashboard link to a conversation, such as one a teammate pasted or an alert's Join link, into its agent and the conversationId get_conversation reads. Links to another workspace are refused. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `link` | string (min length 1, max length 2,000) | yes | A dashboard link to a conversation, such as one from its Conversations page or a Slack or push alert's Join link. |

### `get_conversation_recording` — Get call recording

A link that plays one phone call's recording for 10 minutes and then stops working (expiresAt), or until the recording's retention ends if that is sooner. Share it only with someone who may hear the call. Refused when the conversation is not a call or its call has no recording that can be played. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `conversationId` | string (min length 1, max length 128) | yes | The call's conversation id, from list_conversations or search_conversations. |

### `count_conversations` — Count conversations

Count an agent's logged website chat, API and phone conversations over the last N days: the total, per day, per channel and per sentiment. Counts only, with no conversation's words, so it works whether or not the workspace allows AI assistants to read conversations. Messaging-channel threads are not counted.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `days` | integer (min 1, max 180) |  | How many days back from now, 1 to 180. |

### `workspace_conversation_stats` — Workspace conversation stats

Compare the workspace's agents over the last N days: each agent's logged conversations, how many of the newest 2,000 it could not answer and at what rate (a heuristic that finds fallbacks and handoffs worded the usual ways), and the credits each used by model, with the plan. Counts work either way; the questions each agent missed most are customers' own words, so they are included only where the workspace allows AI assistants to read conversations (questionsHidden).

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `days` | integer (min 1, max 180) |  | How many days back from now, 1 to 180. |

### `query_analytics` — Query analytics

Read the dashboards' metrics for a period, for the workspace, one agent or one campaign: each metric's value over the period, as the dashboards total it (a sum, an average or a rate), optionally per interval and split by agent or channel, and conversations broken down by channel, country, agent, criterion, tool, model or language. Counts and rates only, with no conversation's words, so it works whether or not the workspace allows AI assistants to read conversations. Sentiment metrics need advanced analytics; without analytics at all, only conversation and message totals are shown.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent. |
| `campaignId` | string (min length 1, max length 128) |  | Only the conversations this campaign opened. |
| `from` | string | yes | The period's start, an ISO 8601 time. |
| `to` | string | yes | The period's end, an ISO 8601 time, up to a year later. |
| `interval` | `minute` \| `hour` \| `day` \| `week` |  | The series' step: minute for up to a day, hour for up to 31 days, day or week. |
| `metrics` | list of `conversations` \| `duration` \| `totalDuration` \| `credits` \| `averageCredits` \| `llmCost` \| `averageLlmCost` \| `success` \| `csat` \| `responseTime` \| `sentiment` \| `transfers` \| `messages` \| `thumbsUp` \| `thumbsDown` \| `positive` \| `neutral` \| `negative` \| `unanalyzed` \| `generationLatency` \| `turnLatency` \| `interruptions` \| `queued` \| `queueTime` \| `abandoned` \| `abandonmentRate` \| `timeoutRate` \| `rejectionRate` \| `toolCalls` \| `toolErrors` \| `toolErrorRate` \| `toolLatency` \| `llmRequests` \| `llmTtft` \| `llmFallbackRate` \| `inputTokens` \| `outputTokens` \| `documentReferences` \| `retrievalLatency` \| `guardrails` \| `leads` \| `collectedData` \| `workflowTransitions` \| `inbound` \| `outbound` \| `missed` \| `failed` \| `voicemail` (max items 12) |  | The metrics, named as the dashboards name them. |
| `series` | boolean |  | Also each metric's value per interval. |
| `groupBy` | `none` \| `agent` \| `channel` |  | Split each series by agent or by channel. |
| `breakdowns` | list of `channel` \| `country` \| `agent` \| `evaluation` \| `tool` \| `model` \| `language` \| `agent-csat` \| `agent-transfers` (max items 9) |  | Conversations counted by channel, country, agent, evaluation criterion, tool, model or language. Topics are in get_topics. |

### `get_topics` — Get topics

What a period's conversations were about, as the Topics dashboard shows it: each topic's name with how many conversations had it, how many of those were positive, negative and analyzed, and their evaluation successes, most common first. Topic names come from the agent's topic rules and its conversation analysis, not from quotes, so the list works whether or not the workspace allows AI assistants to read conversations. With a topic, its conversations instead, newest first, 20 a page, which needs that setting. Needs a plan with advanced analytics.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent. |
| `campaignId` | string (min length 1, max length 128) |  | Only the conversations this campaign opened. |
| `from` | string | yes | The period's start, an ISO 8601 time. |
| `to` | string | yes | The period's end, an ISO 8601 time, up to a year later. |
| `topic` | string (min length 1, max length 100) |  | One topic's name, to list its conversations instead. Needs the workspace to allow AI assistants to read conversations. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_insights` — Get insights

What an agent's conversations show: each evaluation criterion's successes, failures, unknown and pending verdicts with its success rate and its numeric scores' average as a percentage of their maximum, for a period (needs a plan with analytics), and why customers are leaving: the churn reasons with how many customers gave each, and how many of the last 30 days' analyzed conversations showed an intent to leave. Counts only, so this works whether or not the workspace allows AI assistants to read conversations. With churnReason, also the customers who gave that reason, with their contact details and their words summarized, which needs that setting.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `from` | string |  | The evaluation period's start. 30 days ago when left out. |
| `to` | string |  | The evaluation period's end. Now when left out. |
| `churnReason` | string |  | One churn reason's slug, to list the customers who gave it, 25 a page. Needs the workspace to allow AI assistants to read conversations. |
| `page` | integer (min 1, max 1,000) |  |  |

### `list_issues` — List issues

List the workspace's issues, or one agent's, newest first unless sort says otherwise: id, title, status, priority, who raised it and how, who is assigned and how often it was reported. Filter by status, priority, source, assignee, who raised it, a conversation or the issue they were merged into. Searching their words, and the excerpt where a search matched, need the workspace to allow AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's issues. |
| `status` | `PENDING` \| `IN_PROGRESS` \| `RESOLVED` \| `CANCELLED` |  | One status. Left out, every status but MERGED. |
| `priorities` | list of `URGENT` \| `HIGH` \| `MEDIUM` \| `LOW` (max items 4) |  |  |
| `sources` | list of `AGENT` \| `CONVERSATION_REVIEW` \| `MANUAL` \| `ANALYSIS` (max items 4) |  | Who raised them: the agent, a review of a conversation, a teammate by hand, or conversation analysis. |
| `assigneeUserId` | string (min length 1, max length 200) |  | A teammate's user id from list_issue_assignees, or unassigned for issues nobody is assigned to. |
| `ownerUserId` | string (min length 1, max length 200) |  | Who raised them: a teammate's user id, or agent for the agent. |
| `conversationId` | string (min length 1, max length 250) |  | Issues about one conversation, by the id list_conversations gives it, or a channel conversation's id. |
| `mergedIntoIssueId` | string (min length 1, max length 128) |  | The issues merged into this one. Don't combine with status. |
| `search` | string (min length 1, max length 200) |  | Words in the title, description or comments. Needs the workspace to allow AI assistants to read conversations, since comments can quote customers. |
| `sort` | `newest` \| `priority` \| `last_reported` |  |  |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_issue` — Get issue

Read one issue: its state, the conversations it is about, its activity (comments, comments on single messages, status and priority changes, merges), what was merged into it and the merge proposal that fixes it. Where the workspace allows AI assistants to read conversations, also its description, every comment's words and the transcript of its first conversation with each message's turnIndex; otherwise those are left out (textHidden), since they can quote the customer.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string (min length 1, max length 128) | yes | The issue's id, from list_issues. |

### `list_issue_assignees` — List issue assignees

List the teammates an agent's issues can be assigned to: workspace members who can view agents, so not Support Associates, with their user id, name and email.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `get_test_results` — Get test results

Show an agent's latest test runs, newest first: which test, what it ran on (target: your draft of a branch with its fingerprint, or a branch's version), how many repeats passed, failed or gave no verdict, and the judge's reason for each. Simulated transcripts are left out. For a test made from a customer conversation, the reasons are left out (reasonsHidden) unless the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `testId` | string (min length 1, max length 128) |  | Only the runs of this test. |
| `limit` | integer (min 1, max 20) |  | How many to return, 1 to 20. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_test_runs` — List test runs

List an agent's test runs, newest first, one row per test run: which test, when, what it ran on (target, for runs an assistant started), and how many repeats passed, failed, gave no verdict or are still running, without the judge's reasons. suiteId names the run_tests call it was part of: get_test_run reads that whole, with reasons, and get_test_transcript reads one run's simulated conversation.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `testId` | string (min length 1, max length 128) |  | Only the runs of this test. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_test_run` — Get test run

Read one run of an agent's tests, a run_tests call or a run started in the dashboard: the branch, who started it, how many tests passed and failed, and for each test its repeats with the judge's reason and the tools that were missing or unexpected. failedOnly keeps the tests that need a look. For a test made from a customer conversation, the reasons are left out (reasonsHidden) unless the workspace allows AI assistants to read conversations. get_test_transcript reads a test's simulated conversation.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `suiteId` | string (min length 1, max length 128) | yes | The run, from run_tests (suiteId) or list_test_runs (suiteId). |
| `failedOnly` | boolean |  | Only the tests with a repeat that failed or gave no verdict. Every test when left out. |

### `get_test_transcript` — Get test transcript

Read the simulated conversation of one test's run, each repeat with its verdict, the judge's reason and the tools the agent called, long messages clipped. A test made from a customer conversation replays it, so its transcript is read only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `runId` | string (min length 1, max length 128) | yes | One test's run: id from list_test_runs or get_test_run, or runIds from run_tests. |
| `repeat` | integer (min 1, max 10) |  | Only this repeat, counting from 1. Every repeat when left out. |

### `list_branches` — List branches

List an agent's branches: id, name, revision, share of conversations (trafficPercent), current version, and how many changes your own draft on each one holds (yourDraftChanges). That is the draft request_publish publishes. The dashboard's unpublished-changes count reads what is open in its editor, so the two differ while someone edits there. Writes to a branch take its revision.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `get_branch` — Get branch

Read one of an agent's branches: name, description, whether it is Main, protected, follows Main or is archived, its revision, share of conversations and current version, and its versions newest first, a page at a time, with each version's number, message, kind and who made it. The whole configuration is in get_agent_configuration.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_branch_status` — Get branch status

Where each of an agent's branches stands: versions ahead of and behind Main, conversations in the last 7 days, share of traffic, phone numbers pinned to it and batches of calls set up on it, whether it is quiet (nothing answers from it, so rebase_branch can run on it) or what answers from it, who holds a draft on it and how many changes yours has, and the open merge proposals.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `get_traffic_history` — Get traffic history

How an agent's conversations are split between its branches now, and each change to the split, newest first: when, who made it and every branch's share after it, for the last 100 changes. Change the split with request_traffic_change.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `preview_merge` — Preview merge

Show what merging one branch into another would change on the target, without merging: each setting that would change with its value before and after, and the settings both branches changed differently (conflictPaths), with which side they would take (conflictsTake). request_merge asks a person to merge. Values of a test made from a customer conversation are left out unless the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch whose changes would be merged. |
| `targetBranchId` | string (min length 1, max length 128) | yes | The branch they would be merged into, such as Main. |
| `force` | boolean |  | Let the merged branch win every conflict. Left out, a conflicting setting keeps the value of the branch changed last. |

### `preview_rebase` — Preview rebase

Show what pulling Main's changes into a branch would change on it, without changing anything: each setting with its value before and after, and the settings both changed (conflictPaths), which keep the branch's value. Values of a test made from a customer conversation are left out unless the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |

### `list_merge_proposals` — List merge proposals

List an agent's merge proposals, newest first, from its latest 200: id, number, title, status (OPEN, MERGED or CLOSED), the branch it merges and the branch it merges into, who opened it and each reviewer's state (PENDING, APPROVED or CHANGES_REQUESTED), with the link to its page. get_merge_proposal reads one with its comments and changes.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `status` | `OPEN` \| `MERGED` \| `CLOSED` |  | Only proposals with this status. Every status when left out. |
| `branchId` | string (min length 1, max length 128) |  | Only proposals that merge this branch or merge into it. |
| `limit` | integer (min 1, max 50) |  | How many to return, 1 to 50. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `get_merge_proposal` — Get merge proposal

Read one merge proposal: its title, description and status, the two branches, whether it can merge now and why not (mergeable), each reviewer's state, its activity (comments, approvals, change requests, closes, reopens and the merge) and, while it is open, each setting a merge would change on the target with its value before and after and the settings both branches changed (conflictPaths); once merged, what the merge changed. Secrets read as masks, and values of a test made from a customer conversation are left out unless the workspace allows AI assistants to read conversations. Nothing is merged here.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `proposalId` | string (min length 1, max length 128) | yes | The proposal's id, from list_merge_proposals or get_branch_status. |

### `list_reviewers` — List reviewers

List the teammates who can review an agent's merge proposals, as the dashboard's reviewer picker offers them: workspace Owners and Members, by user id, name and email. Viewers and Support Associates can't review. open_merge_proposal, request_merge_review and update_merge_proposal take these user ids.

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

### `list_procedures` — List procedures

List an agent's procedures on a branch, newest change first: your own draft when you have one, else the branch's current version (source says which). Each has its id, name, type (freeform or deterministic), status (active, draft or inactive; only active ones run), trigger and folder. Also the procedure folders, and how many stored procedures can't be read (unreadable), which the editor shows for repair. get_procedure reads one whole.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |

### `get_procedure` — Get procedure

Read one of an agent's procedures whole, as save_procedure takes it, from your own draft on a branch or, without one, the branch's current version. changedInDraft says whether your draft's copy differs from the current version's, which discard_procedure_changes undoes; inCurrentVersion says whether the current version has it at all. Send updatedAt back to save_procedure to change it only if nobody did since.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |
| `procedureId` | string (min length 1, max length 128) | yes | The procedure's id, from list_procedures. |

### `search_procedures` — Search procedures

Find an agent's procedures whose name, trigger or content holds some text, ignoring case, in your own draft on a branch or its current version. Each match says where it matched and shows a short passage around it.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |
| `query` | string (min length 1, max length 200) | yes | Text to look for in names, triggers and contents. |

### `get_procedure_schema` — Get procedure schema

The JSON Schemas of the two kinds of procedure save_procedure takes: freeform, whose content is instructions the agent follows, naming a tool as [tool id="…"], a knowledge source as [kb id="…"], another procedure as [procedure id="…"] and a built-in tool as [system_tool id="end_call"]; and deterministic, whose content.steps are structured steps: ask, tell, say, tool_call, branch, sub_procedure and system_tool. The id, timestamps and counts are set by the save.

No arguments.

### `list_pronunciation_dictionaries` — List pronunciation dictionaries

List the pronunciation dictionaries an agent's calls use on a branch, in the order they apply: your own draft when you have one, else the branch's current version (source says which). Each has a name, an id and a version. The rules themselves are kept by the voice provider and are not returned. An agent has up to three.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |

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

### `get_tool_executions` — Get tool executions

Read a workspace tool's calls, newest first, 25 a page, as its Logs tab shows them: when it ran, its status, whether it succeeded, how long it took, the agent and branch, where it came from (conversation, playground, test or dashboard) and the conversation's id; with the tool's total calls and average time. With executionId, one call's arguments, request, answer and error, with secrets masked. What a call carried can hold a customer's words, so it needs the workspace to allow AI assistants to read conversations, except for a call made from the dashboard's tool test. A tool you can't see in list_tools can't be read.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `toolId` | string (min length 1, max length 128) | yes | The tool's id, from list_tools. |
| `executionId` | string (min length 1, max length 128) |  | One call's id from an earlier page, to read what it carried: its arguments, the request it sent, its answer and its error. |
| `status` | `all` \| `success` \| `failed` |  | Only calls that succeeded or failed. All when left out. |
| `from` | string |  | Only calls at or after this ISO 8601 time. |
| `to` | string |  | Only calls at or before this ISO 8601 time. |
| `cursor` | string (min length 1, max length 400) |  | The nextCursor of the previous page, for the next one. |

### `get_test` — Get test

Read one of an agent's tests whole, in the shape save_test takes it, with its id and revision: from your draft of a branch when you have one, else from the branch's current version (source says which). For a test made from a customer conversation, the replayed messages, the agent's original reply and the conversation's variables are left out (conversationDetailsHidden) unless the workspace allows AI assistants to read conversations; such a test is changed in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `testId` | string (min length 1, max length 128) | yes | The test's id, from get_agent or save_test. |
| `branchId` | string (min length 1, max length 128) |  | The branch: your draft on it when you have one, else its current version. Main when left out. |

### `list_workspace_tests` — List workspace tests

List the workspace's test library, newest first, as the Tests page shows it: id, name, type, folder, revision, whether you made it and whether it was made from a customer conversation, with the library's folders. Library tests are shared by every agent; attach_tests adds them to an agent's draft, and duplicate_test copies one.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `search` | string (max length 200) |  | Only tests whose name contains this, ignoring case. |
| `folderId` | string (min length 1, max length 128) |  | Only the tests in this folder. |
| `createdBy` | `all` \| `created-by-me` \| `created-by-others` |  | Whose tests: everyone's when left out. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_secrets` — List secrets

List the workspace's secrets, newest first: id, name, and the tools and published agents that send each one. Their values are never returned. A tool sends a secret as a header when its configuration's secretHeaders maps the header's name to the secret's id. People add secrets in Workspace settings, General, Secrets.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `search` | string (max length 100) |  | Only secrets whose name starts with this, ignoring case. |
| `limit` | integer (min 1, max 100) |  | How many to return, 1 to 100. |
| `cursor` | string (min length 1, max length 1,000) |  | The nextCursor of the previous page, for the next one. |

### `list_auth_connections` — List auth connections

List the workspace's auth connections, newest first: id, name, type (oauth2_client_credentials, basic, bearer or oauth2_jwt), provider, and the tools and published agents that sign in with each one. Their secrets are never returned. An HTTP_TOOL or an MCP_SERVER signs in with a connection when its configuration's authConnectionId is the connection's id, and a BROWSER_FORM's link request when page.authConnectionId is; the connection then sends the Authorization header (an MCP_SERVER's discovery and every call to it included), so the tool sets neither an Authorization header of its own nor auth. People add connections in Workspace settings, General, Auth connections.

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

### `get_batch_call` — Get batch call

Read one batch of outbound calls: its status, its settings (calling number or WhatsApp account, branch, ringing timeout, voicemail detection, concurrency, start, rate and allowed overrides), whether its consent was confirmed, how many recipients have each status and outcome (results), and its recipients newest first, a page at a time, each with its status, outcome, last error and the call it placed. A recipient's number, name, variables and overrides are included only in a workspace that lets AI assistants read conversations (contactDetails); the counts are always.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `batchId` | string (min length 1, max length 128) | yes | The batch's id, from list_batch_calls. |
| `recipientStatus` | `QUEUED` \| `WAITING_FOR_VOICE` \| `SENDING` \| `SENT` \| `FAILED` \| `SKIPPED` \| `NEEDS_REVIEW` |  | Only the recipients with this status. |
| `limit` | integer (min 1, max 100) |  | How many recipients to return, 1 to 100. |
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

### `list_whatsapp_accounts` — List WhatsApp accounts

List the workspace's WhatsApp accounts, newest first, as the WhatsApp page does: id (what create_batch_call takes as connectionId), Meta's phone number ID, the number's display name and number, the business account's name and ID, the agent that answers it (null for none), its status and its Settings switches (messaging, audioReplies, typingIndicator). status is ACTIVE when its agent answers it, PAUSED when its replies are paused, and ERROR when it lost its connection to WhatsApp (lastError says why). No access token or other credential is returned.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Only the accounts this agent answers. Every account's when left out. |

### `get_whatsapp_account` — Get WhatsApp account

Read one WhatsApp account as its sheet on the WhatsApp page shows it: the number and business account, the agent that answers it, its status and its Settings switches: messaging (whether the agent answers messages at all), audioReplies (whether a voice note is answered with one) and typingIndicator (whether messages are marked read with a typing indicator while the agent replies). status is ACTIVE when its agent answers it, PAUSED when its replies are paused, and ERROR when it lost its connection to WhatsApp (lastError says why). With the links to its sheet, where a person assigns it or changes a switch, and to WhatsApp Manager. No access token or other credential is returned.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `accountId` | string (min length 1, max length 128) | yes | The account's id or its phone number ID, from list_whatsapp_accounts. |

### `list_webhooks` — List webhooks

List the workspace's webhook endpoints, newest first, as Workspace settings, Webhooks shows them: id, URL, description, whether it is on, the events it receives, whether a rotated secret still signs beside the new one, and its latest deliveries with each one's event, status, attempts, response status and last error. An endpoint's signing secret is shown once, in the dashboard, when the endpoint is made or its secret rotated, and is never returned here. A person adds, changes or deletes endpoints in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `deliveries` | integer (min 0, max 50) |  | How many of each endpoint's latest deliveries to return, from the last 30 days, 0 to 50. |

### `get_workspace_settings` — Get workspace settings

Read how the workspace's conversations start and what it sends out, as Workspace settings, Webhooks shows it: the conversation initiation webhook a conversation's variables and overrides come from (mode disabled or custom, its URL, the names of the headers it sends and which overrides it may set), how many webhook endpoints there are and how many are on (list_webhooks lists them), and whether AI assistants may read conversations. With an agentId, also that agent's own setting: disabled, custom, or workspace for the workspace's webhook. Header values hold credentials and are never returned. Nothing changes here: a person changes these in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | Also this agent's own initiation webhook setting, from its Settings, Security. |

### `get_credit_usage` — Get credit usage

The credits the workspace's agents used, as the Usage page counts them: the total, each day's credits from the first day to the last (up to 366 days), each agent's total with deleted agents named apart, and the credits Preview, tests and test calls used, counted apart. Counts only, no conversation. get_workspace shows this month's balance against the plan, and workspace_conversation_stats the credits by model.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string |  | The first day, YYYY-MM-DD in UTC. The 1st of this month when left out. |
| `to` | string |  | The last day, YYYY-MM-DD in UTC, up to today. Today when left out. |
| `agentId` | string (min length 1, max length 128) |  | Only this agent's credits. |

### `get_message_settings` — Get message settings

Read the workspace's message settings, as the Integrations page shows them: its message templates (the text a Messages · Send action sends, with placeholders such as {link}, {first_name} and {opt_out}), its opt-out lines, whether an SMS or email sender is turned on, and its e-signature connections by id and name. No credential is returned.

No arguments.

### `list_knowledge_sources` — List knowledge sources

List an agent's knowledge sources as its Knowledge page in the dashboard editor shows them, newest first: id, type, name, url, import status and progress, the last error, and whether the agent is trained on the source's current text. Removed sources are listed with removed true. Without agentId, the workspace's knowledge base, with how many agents use each source (usedByAgents), for attach_knowledge_sources. Their text is not returned; get_knowledge_source reads one, and get_agent shows the sources of the published version.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) |  | The agent whose sources to list. Without it, the workspace's knowledge base, every source any agent can attach. |

### `get_knowledge_source` — Get knowledge source

Read one of an agent's knowledge sources, as its Knowledge page in the dashboard editor shows it: type, name, address, import status, revision, whether the agent is trained on it, how many agents use it, a Q&A's questions, and its text a page at a time (call again with offset set to nextOffset until it is null). A text or Q&A source's text is the one update_knowledge_source changes; other sources show the text read from them, and a website the pages it imports. A support-tickets source holds customers' conversations, so its text is returned only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `sourceId` | string (min length 1, max length 128) | yes | The source's id, from list_knowledge_sources. |
| `offset` | integer (min 0) |  | Where to start, in characters: nextOffset of the last page. |
| `limit` | integer (min 1, max 50,000) |  | How many characters to return, 1 to 50000. |

### `get_knowledge_summary` — Get knowledge summary

Sum up an agent's knowledge: in the dashboard editor, how many sources of each type and their size against the agent's storage limit, how many are importing or not trained yet, the training's state, and the sources other agents use too; and how many sources the version on Main that conversations use was published with.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `search_agent_knowledge` — Search agent knowledge

Look up a question in an agent's trained knowledge in the dashboard editor, the way a conversation looks up the customer's question: a Q&A answer when one matches exactly, and the passages that match best, with their sources. Run train_agent after a change first. It counts nothing as asked. Passages from a support-tickets source quote customers' conversations, so they are left out (ticketPassagesHidden) unless the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `query` | string (min length 2, max length 500) | yes | A customer's question, or the words to look for. |

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

### `duplicate_agent` — Duplicate agent

Copy an agent into a new agent of the workspace, as the developer API's copy does: every branch's current version, with its workflow, procedures, tools, tests, evaluation criteria and knowledge sources. The copy answers only through the API until someone turns a channel on in the dashboard: the original's phone numbers, messaging accounts, email address, website widget and integrations stay with the original. Its knowledge trains again in the background, at no credit cost. Refused when the workspace has reached its agent limit. The original does not change.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `name` | string (min length 1, max length 80) |  | The copy's name. The original's with (copy) when left out. |

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

### `save_draft_configuration` — Save draft configuration

Replace your draft of an agent's configuration on a branch (Main when left out) with the whole configuration get_agent_configuration returned, changed. Send its revision and fingerprint: the save is refused if the branch or your configuration changed since you read it. It is checked as the developer API checks a draft: every action, tool, test, criterion and source must belong to this agent or its workspace. Keep masks such as •••••••• as they were read to keep the stored secrets; a new secret value is never sent here, so add one in the dashboard. Knowledge sources keep the text they were saved with and can be left out, not added. A test made from a customer conversation is kept as it is, or left out. Nothing is published: it reaches no conversation until request_publish is approved.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to replace. Main when left out. |
| `revision` | integer (min 0) | yes | The branch revision get_agent_configuration answered with. |
| `fingerprint` | string (min length 1, max length 64) | yes | The fingerprint get_agent_configuration answered with. |
| `configuration` | object | yes | The whole configuration get_agent_configuration returned, with your changes. Masks stay as they were read to keep each stored secret. |

### `discard_draft` — Discard draft

Throw away your own draft of an agent on a branch (Main when left out), so what you read and test there is the branch's current version again. Only your draft goes: teammates' drafts, the branch's versions and what conversations use stay as they are. A discarded draft cannot be brought back.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to discard. Main when left out. |

### `set_agent_tools` — Set agent tools

Give an agent workspace tools, or take them away, in your draft of a branch (Main when left out). An added tool is copied into the draft as it is set up now, with its secrets kept; a tool you can't see in list_tools can't be added. A removed tool leaves the draft's copy, so adding it back copies its current setup. A tool a specialist uses stays for that specialist. Nothing is published: the agent uses the change once request_publish is approved.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `add` | list of string (max items 100) |  | Workspace tools to give the agent, by id from list_tools. |
| `remove` | list of string (max items 100) |  | Tools to take away from the agent, by id. |

### `create_branch` — Create branch

Start a branch of an agent from Main or another branch. A new branch answers no conversations, so its workflow and voice settings can be changed before asking for traffic.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `name` | string (min length 1, max length 140) | yes | The new branch's name. |
| `description` | string (max length 4,000) |  |  |
| `fromBranchId` | string (min length 1, max length 128) |  | The branch to start from. Main when left out. |

### `update_branch` — Update branch

Rename a branch or change its description, which change nothing a customer sees, also on a branch that answers customers. Main keeps its name. Anything else is refused: a branch's share of traffic changes with request_traffic_change, and whether it follows Main or is protected changes in the dashboard.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `name` | string (min length 1, max length 140) |  |  |
| `description` | string (max length 4,000) |  |  |

### `rebase_branch` — Rebase branch

Pull Main's changes into a quiet branch as its next version: one other than Main with no share of traffic, no phone number pinned to it and no batch of calls. A setting both changed keeps the branch's value; preview it with preview_rebase. Earlier versions stay. A branch something answers from is refused: a person rebases or merges it in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from get_branch_status. |

### `open_merge_proposal` — Open merge proposal

Open a merge proposal, as you, to merge one of an agent's branches into another, optionally with reviewers from list_reviewers, as Create proposal on the agent's Proposals tab does. It is refused when the branch has nothing new for the target, or when an open proposal already merges the same two branches. Nothing is merged: Approving and requesting changes stay with a person, on the proposal's page in the dashboard (url); request_merge asks a person to merge it. preview_merge shows what a merge would change. Nobody is notified, by email or in the dashboard: the proposal, its reviewers and its comments show on the agent's Proposals tab, so tell the people it concerns.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `sourceBranchId` | string (min length 1, max length 128) | yes | The branch whose changes the proposal would merge. |
| `targetBranchId` | string (min length 1, max length 128) | yes | The branch it would merge them into, such as Main. |
| `title` | string (min length 1, max length 200) | yes |  |
| `description` | string (max length 20,000) |  | What the change is and why, in Markdown. |
| `reviewerIds` | list of string (max items 20) |  | Teammates' user ids, from list_reviewers. |

### `update_merge_proposal` — Update merge proposal

Close a merge proposal without merging it, reopen a closed one, or set who reviews it, as the proposal's page does. Removing a reviewer takes their review away, an approval included. Approving and requesting changes stay with a person, on the proposal's page in the dashboard (url); request_merge asks a person to merge it. Nobody is notified, by email or in the dashboard: the proposal, its reviewers and its comments show on the agent's Proposals tab, so tell the people it concerns.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `proposalId` | string (min length 1, max length 128) | yes | The proposal's id, from list_merge_proposals or get_branch_status. |
| `status` | `OPEN` \| `CLOSED` |  | CLOSED closes it without merging; OPEN reopens a closed one. A merged proposal can't change. |
| `reviewerIds` | list of string (max items 20) |  | Who reviews it, from list_reviewers, replacing the reviewers it has: one left out is removed with their review. A reviewer who requested changes can't be removed. |

### `comment_on_merge_proposal` — Comment on merge proposal

Add a comment to a merge proposal's activity, as you. A comment is not a verdict: approving or requesting changes is a person's review on the proposal's page. Nobody is notified, by email or in the dashboard: the proposal, its reviewers and its comments show on the agent's Proposals tab, so tell the people it concerns.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `proposalId` | string (min length 1, max length 128) | yes | The proposal's id, from list_merge_proposals or get_branch_status. |
| `comment` | string (min length 1, max length 10,000) | yes | In Markdown. |

### `request_merge_review` — Request merge review

Ask teammates from list_reviewers to review an open merge proposal: they are added to its reviewers, and the reviewers it has stay, each with the review they gave. Only a person reviews: they approve or request changes on the proposal's page. Nobody is notified, by email or in the dashboard: the proposal, its reviewers and its comments show on the agent's Proposals tab, so tell the people it concerns.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `proposalId` | string (min length 1, max length 128) | yes | The proposal's id, from list_merge_proposals or get_branch_status. |
| `reviewerIds` | list of string (max items 20) | yes | Teammates' user ids, from list_reviewers. |

### `save_workflow` — Save workflow

Replace a branch's workflow, checked as the dashboard checks it. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) it becomes the branch's next version at once, and earlier versions stay available. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft) and reaches nobody until request_publish is approved. Refused while a launched batch of calls on the branch has not finished.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version. |
| `workflow` | object | yes | The whole workflow, in the shape get_workflow returns. |

### `add_workflow_step` — Add workflow step

Add one step to a branch's workflow, as the workflow builder adds it, optionally connected from an existing step, checked as the dashboard checks a workflow. The answer has the new step and its id. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) the change is the branch's next version at once. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft), which reaches nobody until request_publish is approved; get_agent_configuration reads your draft's workflow. Refused while a launched batch of calls on the branch has not finished.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version, on a branch nothing answers from. |
| `type` | `subagent` \| `say` \| `update_state` \| `agent_transfer` \| `phone_transfer` \| `tool` \| `switch` \| `wait_event` \| `end` | yes | The step: subagent (talks with its own prompt and tools), say, update_state, agent_transfer, phone_transfer, tool, switch, wait_event or end. |
| `label` | string (max length 140) |  | The step's name. |
| `settings` | object |  | The step's settings, named as the workflow's steps name them, such as prompt and toolIds on a subagent, message on a say step, or toolId on a tool step. Left out, the builder's defaults. |
| `parentId` | string (min length 1, max length 128) |  | A group step to add it inside. Outside any group when left out. |
| `connectFrom` | object |  | An existing step to connect to the new one. |

### `connect_workflow_steps` — Connect workflow steps

Connect two steps of a branch's workflow, as dragging between them in the builder does, with the condition that moves the conversation. A step's connections are tried in the order they were added. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) the change is the branch's next version at once. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft), which reaches nobody until request_publish is approved; get_agent_configuration reads your draft's workflow. Refused while a launched batch of calls on the branch has not finished.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version, on a branch nothing answers from. |
| `sourceId` | string (min length 1, max length 128) | yes | The step the conversation leaves. |
| `targetId` | string (min length 1, max length 128) | yes | The step it moves to. |
| `forward` | object or null |  | When the conversation moves from source to target. The builder's default when left out: none out of Start, the uncovered result out of a tool, otherwise then cases out of a switch; a step whose default is a condition for the model to judge needs one. null for a connection that only leads back. |
| `backward` | object |  | When the conversation comes back from target to source. |

### `update_workflow_step` — Update workflow step

Change some of one workflow step's settings on a branch, as its panel in the builder does; a setting you send replaces the old one whole. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) the change is the branch's next version at once. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft), which reaches nobody until request_publish is approved; get_agent_configuration reads your draft's workflow. Refused while a launched batch of calls on the branch has not finished.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version, on a branch nothing answers from. |
| `stepId` | string (min length 1, max length 128) | yes | The step's id, from the workflow in get_agent_configuration (settings.displaySettings.workflow.nodes) or get_workflow. |
| `settings` | object | yes | The settings to change, named as the step names them, such as label, prompt, toolIds or position; the ones left out stay. A step's id, type, group and connection order don't change here. |

### `update_workflow_edge` — Update workflow edge

Change one connection of a branch's workflow: the steps it joins or its forward and backward conditions, as its panel in the builder does. A connection keeps at least one of the two. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) the change is the branch's next version at once. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft), which reaches nobody until request_publish is approved; get_agent_configuration reads your draft's workflow. Refused while a launched batch of calls on the branch has not finished.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version, on a branch nothing answers from. |
| `edgeId` | string (min length 1, max length 128) | yes | The connection's id, from the workflow's edges in get_agent_configuration or get_workflow. |
| `sourceId` | string (min length 1, max length 128) |  | A new step for it to leave from. |
| `targetId` | string (min length 1, max length 128) |  | A new step for it to lead to. |
| `forward` | object or null |  | A new forward condition, or null for none. |
| `backward` | object or null |  | A new backward condition, or null for none. |

### `delete_workflow_element` — Delete workflow element

Remove one step, with its connections, or one connection from a branch's workflow, as deleting it in the builder does. A workflow's Start stays. Earlier versions keep the workflow as they had it. On a branch other than Main that nothing answers from yet (no traffic, no phone number pinned to it, no batch of calls) the change is the branch's next version at once. On Main or a branch something answers from, it goes into your draft on that branch (savedAs draft), which reaches nobody until request_publish is approved; get_agent_configuration reads your draft's workflow. Refused while a launched batch of calls on the branch has not finished.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) | yes | The branch's id, from list_branches. |
| `revision` | integer (min 0) | yes | The branch revision you read, from list_branches. |
| `message` | string (max length 4,000) |  | A short note for the new version, on a branch nothing answers from. |
| `stepId` | string (min length 1, max length 128) |  | A step to remove, with its connections; a group goes with the steps inside it. |
| `edgeId` | string (min length 1, max length 128) |  | A connection to remove. |

### `save_procedure` — Save procedure

Create one of an agent's procedures, or replace one whole with procedureId, in your draft of a branch (Main when left out), checked as the editor checks it and as a publish would: a structured procedure's steps, the tools and procedures it names, and no procedure calling itself in a circle. An agent has up to 100. Saved in your draft, which reaches no conversation until request_publish is approved. Test it with run_tests.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `procedureId` | string (min length 1, max length 128) |  | The procedure to replace. Left out, a new procedure. |
| `updatedAt` | string (max length 40) |  | The procedure's updatedAt from get_procedure: the save is refused if it changed since. |
| `procedure` | object | yes | The whole procedure, as get_procedure returns it or get_procedure_schema describes it: name, type (freeform or deterministic, which can't change later), status (active, draft or inactive), trigger, triggerEnabled, content, and folderId from list_procedures' folders. A freeform one can also take system_tools and guardrails. |

### `delete_procedure` — Delete procedure

Take a procedure out of your draft of an agent on a branch (Main when left out). Published versions keep it until your draft is published; one your draft added is gone. Refused while another active procedure calls it. Saved in your draft, which reaches no conversation until request_publish is approved. Test it with run_tests.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `procedureId` | string (min length 1, max length 128) | yes | The procedure's id, from list_procedures. |

### `discard_procedure_changes` — Discard procedure changes

Undo your draft's changes to one procedure on a branch (Main when left out): it goes back to how the branch's current version has it, or, when your draft added it, out of the draft. The rest of your draft stays. A discarded change can't be brought back.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `procedureId` | string (min length 1, max length 128) | yes | The procedure's id, from list_procedures. |

### `update_system_tool` — Update system tool

Turn one of an agent's built-in tools on or off, or change its settings, in your draft of a branch (Main when left out), checked as the agent's Tools panel checks it: a transfer tool needs at least one rule with a condition and an agent of this workspace or a number to call, Update state needs at least one update, and a new voicemail message is refused where the workspace handles regulated data. get_agent_configuration shows the current ones in settings.displaySettings.systemTools. Saved in your draft, which reaches no conversation until request_publish is approved. Test it with run_tests.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `tool` | `end_call` \| `language_detection` \| `skip_turn` \| `update_state` \| `transfer_to_agent` \| `transfer_to_number` \| `play_keypad_touch_tone` \| `voicemail_detection` \| `flag_for_review` | yes | The built-in tool: end_call, language_detection, skip_turn, update_state, transfer_to_agent, transfer_to_number, play_keypad_touch_tone, voicemail_detection or flag_for_review. |
| `settings` | object | yes | The settings to change; the ones left out stay as they are. Every tool takes enabled, description (what the agent is told about it, up to 8000 characters) and interruptions (allow, execution or whole_turn). language_detection also takes onlyAtStart; skip_turn takes reprompt and repromptWaitSeconds (1 to 600); update_state takes updates, [{ variable, value }] with value an expression; transfer_to_agent takes rules, [{ id, condition, targetAgentId, transferMessage, delayMs, targetSpeaksFirst, keepVoice, keepVoiceOverrides }]; transfer_to_number takes rules, [{ id, condition, transferType, destinationType, destination, playRingingTone, postDialDigits, transferMessage, sipHeaders }]; play_keypad_touch_tone takes suppressTurn and useOutOfBandDtmf; voicemail_detection takes message, what the agent leaves on a voicemail (up to 2000 characters; empty hangs up). A list you send replaces the old one whole. |

### `update_agent_behavior` — Update agent behavior

Change how an agent behaves in your draft of a branch (Main when left out), as its Behavior and voice delivery settings save it: tone, the default manner, timezone, extra languages, per-channel reply length and format, and on calls spoken numbers, pronunciation hints, expressive delivery and its tags, language presets with their openings, how soon it answers, and background sound. Only what you send changes. Language presets and pronunciation dictionaries apply only with a compatible Custom speech voice. Saved in your draft, which reaches no conversation until request_publish is approved. Test it with run_tests.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `tone` | string (max length 2,000) |  | How the agent sounds, up to 2000 characters. |
| `defaultPersonality` | boolean |  | Keep the default helpful, polite manner. |
| `timezone` | string (max length 80) |  | An IANA timezone such as Europe/Rome, which tells the agent the date and time; empty for none. |
| `additionalLanguages` | list of string (max items 20) |  | Language codes the agent may also speak, such as fr or es-MX. |
| `normalizeSpeech` | boolean |  | On calls, say numbers, dates and currency as words. |
| `pronunciation` | list of object (max items 50) |  | How to say particular words on calls, [{ word, pronunciation }], up to 50. Replaces the list. |
| `expressiveMode` | boolean |  | On calls, adapt the delivery to the conversation. |
| `audioTags` | list of object (max items 20) |  | Delivery tags for expressive speech, [{ tag, description }], up to 20. Replaces the list. |
| `channels` | object |  | Per channel: verbosity (auto, concise or thorough), format (plain or markdown) and responseTime (auto, 15s, 30s or 45s). Only the channels and fields you send change. |
| `languagePresets` | list of object (max items 32) |  | The language presets a call's starting language picks its opening from. Replaces the list: a preset for a language that had one keeps its voice, and a new one speaks with the agent's voice. A preset's voice is chosen in the dashboard. |
| `turnEagerness` | `normal` \| `patient` \| `eager` |  | How soon the agent answers on a call. |
| `backgroundSound` | object |  | The audio behind the agent on a call: background, volume (0.01 to 1), crossfade, and filter (phone sounds like a phone line). Only the fields you send change. |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |

### `save_agent_variables` — Save agent variables

Set the test values of an agent's variables, the {{name}} placeholders in its texts, tools and workflow, in your draft of a branch (Main when left out), as the dashboard's Vars saves them. Preview, tests and test calls fill placeholders with them; no conversation with a customer ever falls back to them, so keep real customer data and secrets out (a name starting with secret__ is refused). The answer lists every variable the draft uses with its test value. Saved in your draft, which reaches no conversation until request_publish is approved. Test it with run_tests.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `values` | object |  | Test values to set, by variable name, such as { first_name: 'Ana' }. A name starts with a letter or an underscore. |
| `remove` | list of string (max items 100) |  | Variables whose test value to take away. |

### `create_pronunciation_dictionary` — Create pronunciation dictionary

Make a pronunciation dictionary from rules, as importing a lexicon in the agent's voice settings does, and add it to your draft of the agent on a branch (Main when left out), after the ones it has. The voice provider keeps the rules. An agent has up to three dictionaries; it needs a plan with voice, and a workspace that handles regulated data cannot make one. They apply only with a compatible Custom speech voice. Saved in your draft, which reaches no conversation until request_publish is approved. Test it with run_tests.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `name` | string (min length 1, max length 100) | yes | The dictionary's name. |
| `rules` | list of object (max items 500) | yes | Up to 500 rules, each { kind, word, value }: kind alias says word as the text in value; kind ipa or cmu says it as the phonemes in value, which need expressive speech. Rules match case exactly and apply in order. |

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
| `configuration` | object | yes | The tool's settings, with its kind, as the developer API's tools endpoints take them. Header and parameter values and an OAuth client secret are stored and never read back. An HTTP_TOOL or MCP_SERVER sends a workspace secret as a header with secretHeaders, { headerName: secretId }, the id from list_secrets; a header is either in headers or in secretHeaders, not both. An HTTP_TOOL can sign in with OAuth 2.0 client credentials in auth (type oauth2_client_credentials, tokenUrl, clientId, clientSecret, an optional space-separated scope, and clientAuthentication basic or body), and then has no Authorization header of its own. Instead of auth, an HTTP_TOOL, or a BROWSER_FORM's link request in page, can sign in with a workspace auth connection: authConnectionId is the connection's id from list_auth_connections, and the request then has neither auth nor an Authorization header of its own. An MCP_SERVER signs in the same way with authConnectionId, on discovery and on every call, and then has no Authorization header of its own. A BROWSER_FORM fills a web form in a browser: page is { from: url, url } or { from: request, method, url, headers, auth, linkPath } (a request whose JSON answer holds the form's link at linkPath), allowedHosts lists the hosts it may open, fields are { label, parameter, type text, checkbox or select, required }, and before and submit are steps { action click or check, target { by role, role, name } or { by label, label } or { by css, selector } }. |

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

### `attach_tests` — Attach tests

Add tests from the workspace's test library (list_workspace_tests) to your draft of an agent on a branch (Main when left out), checked as the dashboard checks them: each tool a test expects must be one the agent has. Each becomes one of the agent's tests, linked to its library test, and answers with the ids run_tests and get_test take. A library test already in the draft stays there once. Nothing is published.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `testIds` | list of string (max items 100) | yes | Tests from the workspace's test library, by id. |

### `detach_test` — Detach test

Take a test out of your draft of an agent on a branch (Main when left out). Nothing is deleted: the test, its runs and its library test stay, and the branch's published versions keep it until your draft is published.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `testId` | string (min length 1, max length 128) | yes | The agent's test, from get_agent or get_test. |

### `duplicate_test` — Duplicate test

Copy one of an agent's tests, or a test from the workspace library, into your draft of the agent on a branch (Main when left out) as a new test of its own, to change with save_test without touching the original. Name testId or libraryTestId. A test made from a customer conversation is copied only where the workspace allows AI assistants to read conversations. Nothing is published.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `testId` | string (min length 1, max length 128) |  | The agent's test to copy, from get_agent or get_test: from your draft when you have one, else the branch's current version. |
| `libraryTestId` | string (min length 1, max length 128) |  | Or a library test to copy, from list_workspace_tests. |
| `name` | string (min length 1, max length 150) |  | The copy's name. The test's name and (copy) when left out. |

### `create_test_from_conversation` — Create test from conversation

Make a test that replays one of an agent's conversations up to a message, as Create test on the conversation does in the dashboard: the messages, the tools the agent ran and the conversation's variables, with the agent's original reply kept to compare with. You give its name, type and what should happen. It goes into your draft of the agent on a branch (Main when left out); nothing is published. Works only where the workspace allows AI assistants to read conversations.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `branchId` | string (min length 1, max length 128) |  | The branch whose draft to change. Main when left out. |
| `conversationId` | string (min length 1, max length 128) | yes | The conversation's id, from list_conversations. |
| `messageId` | string (min length 1, max length 128) |  | Replay up to this message, from get_conversation; choosing one of the agent's replies tests the reply to the customer's message before it. The last customer message when left out. |
| `name` | string (min length 1, max length 150) | yes | The test's name. |
| `testType` | `next_reply` \| `tool_invocation` \| `simulation` |  | next_reply judges the agent's next reply against successCriterion; tool_invocation checks it calls the tools in expectedActions; simulation lets simulatedUser carry the conversation on. next_reply when left out. |
| `successCriterion` | string (max length 10,000) |  | What the agent should do, which the judge checks. Needed except for tool_invocation. |
| `simulatedUser` | string (max length 10,000) |  | For simulation: who the simulated customer is and what they want. |
| `expectedActions` | list of string (max items 100) |  | The tool ids the agent must call. None when left out. |

### `create_test_folder` — Create test folder

Create a folder in the workspace's test library, as the Tests page does, to group library tests with move_tests.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string (min length 1, max length 150) | yes | The folder's name. |

### `move_tests` — Move tests

Move library tests into a folder of the workspace's test library, or out of every folder with folderId null. Only where they are listed changes; agents' tests stay as they are.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `testIds` | list of string (max items 100) | yes | Tests from the workspace's test library, by id. |
| `folderId` | string or null | yes | The folder, from list_workspace_tests, or null for none. |

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

### `add_knowledge_crawl` — Add knowledge crawl

Read a public website's pages into an agent's knowledge, as its Knowledge page imports a website: it looks for the site's pages by crawling its links or reading its sitemap, imports the first ones found, at most 50 as the dashboard allows per source, and queues the agent's training once they are read. The import runs in the background: list_knowledge_sources shows its progress. A larger site goes in as several sources, with excludePaths. It joins the agent's knowledge in the dashboard editor, which trains on it, and conversations use it once the agent is next published from the dashboard editor; request_publish publishes your own draft, which does not hold it.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `url` | string (min length 1, max length 2,048) | yes | The public website to read, such as example.com. |
| `mode` | `crawl` \| `sitemap` |  | crawl follows the site's links from url; sitemap reads the pages its sitemap lists. crawl when left out. |
| `name` | string (min length 1, max length 250) |  | The source's name. The site's address when left out. |
| `excludePaths` | list of object (max items 30) |  | Pages to leave out, each { match: starts, ends, contains, exact or wildcard, path }. |
| `maxPages` | integer (min 1, max 50) |  | How many of the pages found to import, 1 to 50. |

### `update_knowledge_source` — Update knowledge source

Change a text or Q&A knowledge source's name, text or questions, as its Knowledge page saves it, and queue the training of every agent that uses it. Send the revision get_knowledge_source answered with: the change is refused if the source changed since. A text you send replaces the old one whole. Websites, files, Notion pages and support tickets change in the dashboard. It joins the agent's knowledge in the dashboard editor, which trains on it, and conversations use it once the agent is next published from the dashboard editor; request_publish publishes your own draft, which does not hold it.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `sourceId` | string (min length 1, max length 128) | yes | The source's id, from list_knowledge_sources. |
| `revision` | integer (min 0) | yes | The source's revision, from get_knowledge_source. |
| `name` | string (min length 1, max length 250) |  |  |
| `content` | string (min length 1, max length 1,000,000) |  | The whole new text, up to 1 MB, replacing the old one: a Q&A's answer. |
| `questions` | list of string (max items 50) |  | For a Q&A source, every question it answers, replacing them. |

### `attach_knowledge_sources` — Attach knowledge sources

Give an agent sources from the workspace's knowledge base, as its Knowledge page attaches them, within its knowledge storage limit, and queue its training. A source stays one source, shared with the agents that use it. A source removed from this agent with detach_knowledge_source comes back. It joins the agent's knowledge in the dashboard editor, which trains on it, and conversations use it once the agent is next published from the dashboard editor; request_publish publishes your own draft, which does not hold it.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `sourceIds` | list of string (max items 100) | yes | Workspace sources, by id from list_knowledge_sources: without agentId it lists the workspace's knowledge base, with another agent's id that agent's sources. |

### `detach_knowledge_source` — Detach knowledge source

Remove a source from an agent's knowledge in the dashboard editor, as its Knowledge page removes one, and queue its training. Nothing is deleted: the source stays in the workspace and with the other agents that use it, attach_knowledge_sources brings it back, and published versions keep it until the agent is next published from the dashboard editor.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `sourceId` | string (min length 1, max length 128) | yes | The source's id, from list_knowledge_sources. |

### `train_agent` — Train agent

Train an agent on its knowledge in the dashboard editor again, as Retry training on its Knowledge page does: the training runs in the background, after any source still importing, and costs no credits. get_knowledge_summary shows when it is done. Conversations use the trained knowledge once the agent is next published from the dashboard editor.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `create_email_sender` — Create email sender

Give an agent its generated email address, as its Email settings do: Helpdesk for the mail it receives, then the address, with its replies off. An agent that has an address keeps it. A Messages · Send action sends email only from an address whose replies are on (status ACTIVE), and turning them on reaches customers, so ask for it with request_channel_activation, which a person approves; until then the send answers no_sender.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |

### `flag_conversation` — Flag conversation

Flag one of an agent's conversations for review, as Flag issue on a transcript does: a new issue about it, with a comment on the whole conversation, comments on single messages, or both. When the conversation already has a pending or in progress issue, the comments go to that one instead (addedToOpenIssue) and a more urgent priority is raised on it. Nobody is notified: the issue shows up on the Issues page in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `conversationId` | string (min length 1, max length 250) | yes | The conversation's id from list_conversations, or a channel conversation's id. |
| `title` | string (min length 1, max length 120) |  | Up to 120 characters. Left out, the comment's first line. |
| `comment` | string (min length 1, max length 10,000) |  | About the whole conversation, in Markdown. |
| `turnComments` | list of object (max items 50) |  | Comments on single messages. |
| `priority` | `URGENT` \| `HIGH` \| `MEDIUM` \| `LOW` |  |  |

### `create_issue` — Create issue

Raise an issue on an agent by hand, as you: a follow-up or a problem seen anywhere, optionally assigned to a teammate who can view agents. Nobody is notified: the issue shows up on the Issues page in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `title` | string (min length 1, max length 120) |  | Up to 120 characters. Left out, the description's first line. |
| `description` | string (min length 1, max length 10,000) | yes | What is wrong, in Markdown. |
| `priority` | `URGENT` \| `HIGH` \| `MEDIUM` \| `LOW` |  |  |
| `status` | `PENDING` \| `IN_PROGRESS` \| `RESOLVED` \| `CANCELLED` |  |  |
| `assigneeUserId` | string (min length 1, max length 200) |  | A teammate's user id from list_issue_assignees. |

### `update_issue` — Update issue

Change an issue's title, description, status, priority or assignee. Only what you send changes; status and priority changes are recorded in its activity. Nobody is notified: the issue shows up on the Issues page in the dashboard.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string (min length 1, max length 128) | yes | The issue's id, from list_issues. |
| `title` | string (min length 1, max length 120) |  |  |
| `description` | string or null |  | Null clears it. |
| `status` | `PENDING` \| `IN_PROGRESS` \| `RESOLVED` \| `CANCELLED` |  | Any status but MERGED, which only merging sets. A merged issue keeps its status. |
| `priority` | `URGENT` \| `HIGH` \| `MEDIUM` \| `LOW` or null |  | Null clears it. |
| `assigneeUserId` | string or null |  | A teammate's user id from list_issue_assignees, or null to unassign. |

### `comment_on_issue` — Comment on issue

Add a comment to an issue, as you, and optionally resolve it. Nobody is notified: the issue shows up on the Issues page in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string (min length 1, max length 128) | yes | The issue's id, from list_issues. |
| `comment` | string (min length 1, max length 10,000) | yes | In Markdown. |
| `resolve` | boolean |  | Also resolve the issue, when it is pending or in progress. |

### `comment_on_issue_turn` — Comment on issue message

Comment on one message of a conversation an issue is about, as you, the way a comment on a turn of its transcript does. Nobody is notified: the issue shows up on the Issues page in the dashboard.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string (min length 1, max length 128) | yes | The issue's id, from list_issues. |
| `turnIndex` | integer (min 0) | yes | The message's position in the conversation's transcript, from 0, oldest first, the customer's and the agent's messages alike: get_issue's transcript numbers them. |
| `comment` | string (min length 1, max length 10,000) | yes | In Markdown. |
| `conversationId` | string (min length 1, max length 250) |  | One of the issue's conversations[].id. Left out, its first conversation with a transcript. |

### `create_batch_call` — Create batch call

Set up a batch of outbound calls as a draft that calls nobody, as the batch call API's create request does without launch: the agent, how the calls are placed, the start and up to 1,000 recipients, with the API's checks on the plan, the calling number or WhatsApp account, the branch and the recipients. Consent stays unconfirmed: the person who approves the launch agrees to the outbound terms. The answer says whether it could launch as it is (launch) and how to ask for that with request_batch_launch. The draft shows on the agent's Outbound page, and a branch it calls with counts as answering from it, so workflow and voice changes there go into your draft.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `name` | string (min length 1, max length 120) | yes |  |
| `settings` | object | yes | How the calls are placed. For phone calls: phoneLineId, a number from list_phone_numbers that can call out; for WhatsApp calls: channel whatsapp and connectionId, an account from list_whatsapp_accounts assigned to this agent. Also branchId, a published branch to call with, or null for the agent's traffic split; ringingTimeoutSeconds, 5 to 120 (60 in the dashboard); answeringMachineDetection, on when left out; concurrencyLimit, or null for automatic; allowedOverrides, which of prompt, first_message, language and voice_id a recipient's overrides may set ([] for none). |
| `timezone` | string (max length 64) |  | The IANA timezone the batch's times are in. |
| `startsAt` | string or null |  | When calling starts, in ISO 8601, or null for as soon as it is launched. |
| `sendRatePerMinute` | integer (min 1, max 20) |  | How many calls start per minute, 1 to 20. |
| `recipients` | list of object (max items 1,000) | yes | Up to 1,000 recipients, each with phone (its country code first, such as +12025550101; for WhatsApp, the person's WhatsApp number), name, variables the agent's placeholders read, and overrides ({} for none). Each number once, and never the batch's own calling number. |

### `cancel_batch_call` — Cancel batch call

Cancel a running, scheduled or paused batch of outbound calls, as Cancel on its page does. Recipients still waiting are skipped and nobody else is called. A call already dialing, ringing or connected is not hung up: it carries on to its end. This can't be undone; a person can copy the batch into a new draft in the dashboard. A batch already cancelled answers as it is; a draft or a finished batch is refused, and a Campaigns page campaign is cancelled with cancel_campaign. Nobody in the workspace is notified: the batch's page shows it cancelled.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `batchId` | string (min length 1, max length 128) | yes | The batch's id, from list_batch_calls. |

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

### `request_merge` — Request merge

Ask for one of an agent's branches to be merged into another, or for an open merge proposal to be merged. The merge is the target's next version: on a branch customers reach, they get it at once. Nothing is merged yet: this returns a link where a person approves or declines it, and shows them what preview_merge shows. If either branch gets a new version before then, or the settings both changed are no longer the ones shown, the request can no longer run.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `merge` | object | yes | A branch into another, or an open merge proposal, which merges its branch into its target. |
| `archiveSource` | boolean |  | Archive the merged branch afterwards, as the dashboard does unless told not to. Main is never archived. |

### `request_delete` — Request delete

Ask for something to be deleted for good: an agent with its conversations, a test on an agent's Main (as a new version of Main), a library test (agents keep their copies), a knowledge source of the workspace (taken out of every agent that has it), or an issue. Nothing is deleted yet: this returns a link where a person approves or declines it, with what goes with it and what uses it. An agent customers still reach through a channel, a phone number or a running campaign is refused until those are turned off, moved or cancelled. If what is deleted changes before then, the request can no longer run.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `target` | object | yes |  |

### `request_outbound_call` — Request outbound call

Ask for one outbound test call from a workspace number to a number you name, as the Phone page's Test call places it: the agent calls, using call credits. Nothing is dialed yet: this returns a link where a person approves or declines it; approving takes their agreement to the outbound communications terms. If the branch it answers on gets a new version before then, the request can no longer run.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `phoneLineId` | string (min length 1, max length 128) | yes | The calling number's id, from list_phone_numbers. |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `toE164` | string (min length 7, max length 32) | yes | The number to call, in international format, such as +14155550123. |
| `branchId` | string (min length 1, max length 128) |  | Answer with this published branch. Left out, the call follows the agent's traffic split. |
| `variables` | object |  | Values for the agent's variables on this call, as Vars sets them. |
| `answeringMachineDetection` | boolean |  | Hang up on voicemail, as the Test call's switch does. |

### `request_batch_retry` — Request batch retry

Ask for a finished or cancelled batch's failed and unanswered recipients to be called again, as Retry on the batch does: a new batch of them, launched when approved. Recipients who opted out or whose calls need review are left out. Nothing is called yet: this returns a link where a person approves or declines it, and how many are called again; approving takes their agreement to the outbound terms. If the batch changes, or would call other recipients, before then, the request can no longer run. The recipients' numbers are listed only where the workspace lets AI assistants read conversations.

*Reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `batchId` | string (min length 1, max length 128) | yes | The finished or cancelled batch's id, from list_batch_calls. |

### `request_whatsapp_assignment` — Request WhatsApp assignment

Ask for a WhatsApp account to be answered by an agent, or by none, as the account's sheet on the WhatsApp page assigns it. The agent answers new messages at once; the agent it replaces keeps its conversations. Nothing changes yet: this returns a link where a person approves or declines it. If the account is assigned to another agent before then, the request can no longer run.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `connectionId` | string (min length 1, max length 128) | yes | The WhatsApp account's id, from list_whatsapp_accounts. |
| `agentId` | string or null | yes | The agent to answer the account, or null for none. |

### `request_webhook_change` — Request webhook change

Ask to add a workspace webhook endpoint, change its URL, description, events or whether it is on, or delete it with its delivery log, as Workspace settings → Webhooks does. An endpoint that is on receives customers' conversation data at its URL. Nothing changes yet: this returns a link where a person approves or declines it. A new endpoint's signing secret is made when it is approved and shown once to the approver, never to you. If the endpoint is changed before then, the request can no longer run.

*Overwrites, reaches outside the workspace.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `change` | object | yes |  |

### `request_live_settings` — Request live settings

Ask to change a setting that reaches customers at once, outside any version: the agent's name, its chat widget's look, its Topics settings, an evaluation criterion or a shared analysis library item on the analysis its editor has open, or how its conversations start (the initiation webhook, by header names only; a header's value is a secret a person enters in the dashboard). Nothing changes yet: this returns a link where a person approves or declines it, with the setting before and after. If the setting is changed before then, the request can no longer run.

*Overwrites.*

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `agentId` | string (min length 1, max length 128) | yes | The agent's id, from list_agents. |
| `setting` | object | yes |  |

