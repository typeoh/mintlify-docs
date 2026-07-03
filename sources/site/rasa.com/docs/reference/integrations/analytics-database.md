# Source: https://rasa.com/docs/reference/integrations/analytics-database

On this page

## Overview[​](https://rasa.com/docs/reference/integrations/analytics-database/#overview 'Direct link to Overview')

Studio ingests conversation events from a Kafka topic (`rasa-events`) and stores them in a PostgreSQL database. An event ingestion service consumes each message and writes structured records into analytics tables — capturing individual conversation events as well as aggregated daily statistics by channel/version and by flow.

These tables are available as a standard PostgreSQL database, which means you can connect any SQL-compatible BI tool — such as Metabase, Tableau, Looker, Grafana, or Redash — directly to the database and build custom dashboards on top of the data.

## Connecting to the database[​](https://rasa.com/docs/reference/integrations/analytics-database/#connecting-to-the-database 'Direct link to Connecting to the database')

Studio uses a standard PostgreSQL connection. Use the following connection string format:

```
postgresql://<DB_USER>:<DB_PASSWORD>@<DB_HOST>:<DB_PORT>/<DB_NAME>
```

| Parameter | Default value | Description |
| --- | --- | --- |
| `DB_HOST` | — | Hostname or IP of the PostgreSQL server |
| `DB_PORT` | `5432` | PostgreSQL port |
| `DB_NAME` | `studio` | Database name |
| `DB_USER` | `studio_backend` | Database user |
| `DB_PASSWORD` | — | Password for the database user |

**Example (local development):**

```
postgresql://studio_backend:studio_backend@localhost:5432/studio
```

For production deployments, ask your administrator for the host, port, and credentials. It is recommended to create a dedicated read-only database user with `SELECT` privileges on the analytics tables before connecting your BI tool.

## Data model[​](https://rasa.com/docs/reference/integrations/analytics-database/#data-model 'Direct link to Data model')

![Analytics database schema][base64-image]

## Table reference[​](https://rasa.com/docs/reference/integrations/analytics-database/#table-reference 'Direct link to Table reference')

### Conversation[​](https://rasa.com/docs/reference/integrations/analytics-database/#conversation 'Direct link to Conversation')

Each row represents a single conversation session between a user and the assistant.

| Column | Type | Description |
| --- | --- | --- |
| `id` | `string (UUID)` | Primary key. Unique identifier for the conversation. |
| `assistantName` | `string` | Name of the assistant that handled the conversation. |
| `assistantVersion` | `string` | Version of the assistant at the time of the conversation. |
| `totalNumberOfUserMessages` | `integer` | Number of messages sent by the user during the conversation. |
| `startDate` | `datetime` | Timestamp of the first event in the conversation (set on `session_started`). |
| `ingested` | `boolean` | `true` once the conversation has been fully processed (set on `session_ended` or `conversation_inactive`). |
| `reviewState` | `enum` | Review status of the conversation. Values: `UNREVIEWED`, `PARTIALLY_REVIEWED`, `REVIEWED`. |
| `csatRating` | `enum | null` | Customer satisfaction rating, if collected. Values: `SATISFIED`, `UNSATISFIED`, `UNANSWERED`. |
| `lastReviewedAt` | `datetime | null` | Timestamp of the most recent review action. `null` if never reviewed. |
| `escalatedAt` | `datetime | null` | Timestamp when the conversation was escalated to a human agent. `null` if not escalated. |
| `environmentId` | `string (UUID) | null` | Foreign key to the `Environment` table. Identifies which environment (e.g. production, staging) the conversation occurred in. `null` if not set. |

---

### ConversationEvent[​](https://rasa.com/docs/reference/integrations/analytics-database/#conversationevent 'Direct link to ConversationEvent')

Each row represents a single Rasa event within a conversation, such as an action execution, slot update, or flow transition.

| Column | Type | Description |
| --- | --- | --- |
| `id` | `string (UUID)` | Primary key. Unique identifier for the event. |
| `conversationId` | `string (UUID)` | Foreign key to `Conversation.id`. The conversation this event belongs to. |
| `timestamp` | `datetime` | When the event occurred. |
| `modelId` | `string (UUID)` | Foreign key to `MachineLearningModel.id`. The model that generated this event. |
| `type` | `enum` | Type of Rasa event. See values below. |
| `rawData` | `json` | Full raw event payload as received from Rasa. |
| `metadata` | `json` | Additional metadata attached to the event. Defaults to `{}`. |
| `actionText` | `string | null` | Name of the action executed (for `ACTION` events). |
| `name` | `string | null` | Event name, if applicable. |
| `slotValue` | `json | null` | Slot value set by the event (for `SLOT` events). |
| `flowId` | `string | null` | Identifier of the flow associated with the event (for flow-related events). |
| `stepId` | `string | null` | Identifier of the step within the flow. |
| `agentId` | `string | null` | Identifier of the subagent that generated the event, when using multi-agent setups. |

**`type` enum values:**

`ACTION`, `SESSION_STARTED`, `SLOT`, `RESET_SLOTS`, `FLOW_STARTED`, `FLOW_INTERRUPTED`, `FLOW_RESUMED`, `FLOW_COMPLETED`, `FLOW_CANCELLED`, `FORM`, `ACTIVE_LOOP`, `RESTART`, `CONVERSATION_INACTIVE`, `SESSION_ENDED`, `ENTITIES`, `FOLLOWUP`, `ACTION_EXECUTION_REJECTED`, `STACK`, `ROUTING_SESSION_ENDED`, `REMINDER`, `CANCEL_REMINDER`, `REWIND`, `UNDO`, `EXPORT`, `PAUSE`, `RESUME`, `LOOP_INTERRUPTED`, `AGENT_STARTED`, `AGENT_COMPLETED`, `AGENT_INTERRUPTED`, `AGENT_CANCELLED`, `AGENT_RESUMED`

---

### VersionDailyStatistics[​](https://rasa.com/docs/reference/integrations/analytics-database/#versiondailystatistics 'Direct link to VersionDailyStatistics')

Each row contains aggregated daily metrics for a specific channel and assistant version combination. This table is the primary source for time-series dashboards tracking volume, automation rate, latency, and CSAT.

| Column | Type | Description |
| --- | --- | --- |
| `id` | `string (UUID)` | Primary key. |
| `date` | `datetime` | The calendar day this row covers. Combined with `channelName` and `version`, this is unique per row. |
| `channelName` | `string` | Name of the channel (e.g. `rest`, `socketio`). |
| `version` | `string` | Assistant version identifier. |
| `totalSessions` | `integer` | Total number of conversation sessions on this day. |
| `automatedCount` | `integer` | Number of sessions that were fully handled without escalation. |
| `escalatedCount` | `integer` | Number of sessions escalated to a human agent. |
| `averageLatencyMs` | `float` | Mean response latency in milliseconds. |
| `p50LatencyMs` | `float` | 50th percentile (median) response latency in milliseconds. |
| `p95LatencyMs` | `float` | 95th percentile response latency in milliseconds. |
| `p99LatencyMs` | `float` | 99th percentile response latency in milliseconds. |
| `csatSatisfied` | `integer` | Number of sessions rated as satisfied. |
| `csatUnsatisfied` | `integer` | Number of sessions rated as unsatisfied. |
| `csatUnanswered` | `integer` | Number of sessions where CSAT was not answered. |
| `customSuccessCount` | `integer | null` | Number of sessions meeting a custom success criteria, if configured. |
| `customFailureCount` | `integer | null` | Number of sessions failing a custom success criteria, if configured. |
| `createdAt` | `datetime` | When this row was first created. |
| `updatedAt` | `datetime` | When this row was last updated. |
| `successCriteriaId` | `string (UUID) | null` | Foreign key to the `SuccessCriteria` table. References the custom success criteria definition used to compute `customSuccessCount` and `customFailureCount`. `null` if no custom criteria is configured. |

---

### FlowDailyStatistics[​](https://rasa.com/docs/reference/integrations/analytics-database/#flowdailystatistics 'Direct link to FlowDailyStatistics')

Each row contains aggregated daily metrics for a specific flow, allowing you to compare flow-level engagement and escalation rates over time.

| Column | Type | Description |
| --- | --- | --- |
| `id` | `string (UUID)` | Primary key. |
| `date` | `datetime` | The calendar day this row covers. |
| `flowName` | `string` | Name of the flow. |
| `totalSessions` | `integer` | Total number of sessions that entered this flow on this day. |
| `automatedCount` | `integer` | Number of sessions that completed the flow without escalation. |
| `escalatedCount` | `integer` | Number of sessions that were escalated while in this flow. |
| `createdAt` | `datetime` | When this row was first created. |
| `updatedAt` | `datetime` | When this row was last updated. |

---

### SuccessCriteria[​](https://rasa.com/docs/reference/integrations/analytics-database/#successcriteria 'Direct link to SuccessCriteria')

Stores the custom success filter configured per assistant, which defines which flows must (or must not) be executed for a session to be counted as successful. Referenced by `VersionDailyStatistics` to compute `customSuccessCount` and `customFailureCount`.

| Column | Type | Description |
| --- | --- | --- |
| `id` | `string (UUID)` | Primary key. |
| `assistantName` | `string` | Name of the assistant this criteria applies to. |
| `includedFlows` | `string[]` | List of flow names that must be executed for a session to count as successful. |
| `excludedFlows` | `string[]` | List of flow names that must not be executed for a session to count as successful. |
| `createdAt` | `datetime` | When this criteria was created. |
| `updatedAt` | `datetime` | When this criteria was last updated. |

- [Overview](https://rasa.com/docs/reference/integrations/analytics-database/#overview)
- [Connecting to the database](https://rasa.com/docs/reference/integrations/analytics-database/#connecting-to-the-database)
- [Data model](https://rasa.com/docs/reference/integrations/analytics-database/#data-model)
- [Table reference](https://rasa.com/docs/reference/integrations/analytics-database/#table-reference)
 - [Conversation](https://rasa.com/docs/reference/integrations/analytics-database/#conversation)
 - [ConversationEvent](https://rasa.com/docs/reference/integrations/analytics-database/#conversationevent)
 - [VersionDailyStatistics](https://rasa.com/docs/reference/integrations/analytics-database/#versiondailystatistics)
 - [FlowDailyStatistics](https://rasa.com/docs/reference/integrations/analytics-database/#flowdailystatistics)
 - [SuccessCriteria](https://rasa.com/docs/reference/integrations/analytics-database/#successcriteria)