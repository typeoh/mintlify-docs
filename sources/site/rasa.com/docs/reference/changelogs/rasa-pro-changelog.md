# Source: https://rasa.com/docs/reference/changelogs/rasa-pro-changelog

On this page

All notable changes to Rasa Pro will be documented in this page. This product adheres to [Semantic Versioning](https://semver.org/) starting with version 3.3 (initial version).

Rasa Pro consists of two deployable artifacts: Rasa Pro and Rasa Pro Services. You can read the change log for both artifacts below.

## \[3.16.13\] - 2026-06-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31613---2026-06-04 'Direct link to [3.16.13] - 2026-06-04')

Rasa Pro 3.16.13 (2026-06-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes 'Direct link to Bugfixes')

- Fixed a spurious "Fatal error on SSL transport" traceback printed to stderr on Windows after `rasa test e2e` completes by explicitly closing litellm's shared httpx sessions before the event loop shuts down.
 
- Fixed a crash in rasa-tools where training an assistant with a sub-agent MCP server configured caused a `RuntimeError: Attempted to exit cancel scope in a different task than it was entered in`, crashing the stdio server and making the next tool call fail with `MCP error -32000: Connection closed`.
 
- Fix two bugs causing random crashes and resource leaks when running Rasa with --workers 2+: a pre-fork event loop left open during JWT setup, and litellm async HTTP clients not being closed on worker shutdown.
 
 Root cause — Sanic uses fork-based multi-worker mode (legacy=True). create\_app() was calling asyncio.new\_event\_loop() + set\_event\_loop() in the main process to satisfy sanic-jwt's Initialize(), then leaving the loop open. Forked workers inherited its kqueue/epoll file descriptors, conflicting with each worker's own loop (~50% crash at startup). Separately, close\_resources() never called litellm.close\_litellm\_async\_clients(), leaving cached async HTTP clients open when workers exit.
 
- An explicitly configured `api_key` in the `rasa` provider model config now takes precedence over the Rasa license, allowing custom endpoints to authenticate with their own credentials.
 
- Transient tracker store connection errors (e.g. Postgres restart) no longer flood logs with full stack traces on each retry attempt; the error message is preserved on one line.
 
- Fix `resume_flow()` marking the wrong `AgentStackFrame` as resumed when a flow contains multiple sub-agent frames, which prevented `run_agent()` from taking its resume branch.
 
- Upgrading from 3.15.x to 3.16.x no longer causes every subsequent message to fail with `InvalidFlowStepIdException` when a tracker was mid-flow at upgrade time and a flow step ID was renamed or removed. The flow executor now detects stale step IDs, logs a structured warning, removes the stale frame, and continues processing — rather than propagating the exception as a `GraphComponentException`.
 

## \[3.16.12\] - 2026-06-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31612---2026-06-02 'Direct link to [3.16.12] - 2026-06-02')

Rasa Pro 3.16.12 (2026-06-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-1 'Direct link to Bugfixes')

- Fixed a performance regression in `SQLTrackerStore.retrieve()` introduced in v3.16, where every retrieve loaded the full conversation history before replay-safe session slicing. Retrieve now fetches a minimum event prefix by trying the most recent session boundaries first (up to three). When reconstruction fails with a dialogue-stack patch error, the store retries from the last stack event in that prefix whose patch applies to an empty stack. If all 3 attempts fail, the store falls back to loading the full conversation history as before. This greatly reduces database I/O and memory use for long-lived conversations while preserving replay-safe behaviour.
- Fixed graph schema validation on Python 3.13+ when components use PEP 604 union type hints (e.g. `Domain | None`), which previously raised a false `GraphSchemaValidationException`.
- Fixed MCP flow tool tracing metrics when `_execute_mcp_tool_call` mutates the shared `initial_events` list in place by capturing the pre-mutation event count for output slicing, and by detecting failures from `InternalErrorPatternFlowStackFrame` on the dialogue stack rather than from unchanged event list length.
- Updated `idna`, `pip` and `urllib3` to address security vulnerabilities.

## \[3.16.11\] - 2026-06-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31611---2026-06-01 'Direct link to [3.16.11] - 2026-06-01')

Rasa Pro 3.16.11 (2026-06-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-2 'Direct link to Bugfixes')

- Fixed DIETClassifier training failures on GPU with TensorFlow 2.19 and Keras 3, where training could fail with an `InvalidArgumentError` about unsupported `SparseReshape` operations on XLA GPU JIT devices. `RasaModel` now inherits from `tensorflow.keras.Model` instead of standalone Keras 3's `Model`, so it uses the same Keras stack as the rest of Rasa's TensorFlow code and respects `TF_USE_LEGACY_KERAS=1`.

## \[3.16.10\] - 2026-05-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31610---2026-05-22 'Direct link to [3.16.10] - 2026-05-22')

Rasa Pro 3.16.10 (2026-05-22)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-3 'Direct link to Bugfixes')

- Fixed a `KeyError` in `ContinueInterruptedPatternFlowStackFrame.from_dict` that occurred when loading tracker events stored before the multi-interrupt fields were introduced. This caused repeated tracker store retries that blocked the server event loop, leading to liveness probe failures and pod restarts.

## \[3.16.9\] - 2026-05-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3169---2026-05-20 'Direct link to [3.16.9] - 2026-05-20')

Rasa Pro 3.16.9 (2026-05-20)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-4 'Direct link to Bugfixes')

- Fix an interrupted sub-agent not being properly resumed after the user answers "Yes" to `pattern_continue_interrupted`. This is fixed by introducing a transient `RESUMING` agent state that sits between `INTERRUPTED` and `WAITING_FOR_INPUT`: `resume_flow()` now sets the agent frame to `RESUMING` instead of directly to `WAITING_FOR_INPUT`; `run_agent()` detects this and re-invokes the agent with `resumed_after_interruption=True`.

## \[3.16.8\] - 2026-05-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3168---2026-05-19 'Direct link to [3.16.8] - 2026-05-19')

Rasa Pro 3.16.8 (2026-05-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-5 'Direct link to Bugfixes')

- Fixed `InvalidFlowIdException` and repeated error logs that occurred when a conversation tracker contained historical events referencing flows or slots that were subsequently removed from the model.

## \[3.16.7\] - 2026-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3167---2026-05-15 'Direct link to [3.16.7] - 2026-05-15')

Rasa Pro 3.16.7 (2026-05-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-6 'Direct link to Bugfixes')

- Updated security-flagged dependencies:
 - `litellm`: `1.82.4` → `1.84.0`
 - `langchain`: `~0.3.27` → `~1.2.16`
 - `langchain-community`: `~0.3.29` → `~0.4.1`
 - `langchain-core`: `~0.3.81` → `~1.3.2`
 - `langsmith`: `~0.6.3` → `~0.7.31`
 - `langchain-qdrant`: `~0.2.1` → `~1.1.0`
 - `langchain-openai`: transitive → `~1.2.1` (new direct pin)
 - `langchain-text-splitters`: transitive → `~1.1.2` (new direct pin)
- The `no_text_normalization` parameter is now correctly passed to the Rime TTS WebSocket connection, allowing users to skip text normalization to reduce latency.

## \[3.16.6\] - 2026-05-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3166---2026-05-12 'Direct link to [3.16.6] - 2026-05-12')

Rasa Pro 3.16.6 (2026-05-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-7 'Direct link to Bugfixes')

- Fix bug when user's message during bot's speech is queued after bot is done speaking. When interruptions are DISABLED, if the bot is speaking and the user talks over it, the bot should ignore that speech entirely. When interruptions are ENABLED, the bot should only stop and listen if the user says enough words to qualify as an interruption (based on the min\_words setting).
 
- When interruptions are ENABLED, user message should be queued for processing only if it passes interruption criteria.
 
 If interruptions are disabled, user message should be queued only if user speaks during collect step utterance.
 
 Updated signature of following methods:
 
 - send\_start\_marker: `send_start_marker(self, recipient_id: str)` => `send_start_marker(self, marker_input: MarkerInput)`
 - send\_intermediate\_marker: `send_intermediate_marker(self, recipient_id: str)` => `send_intermediate_marker(self, marker_input: MarkerInput)`
 - send\_end\_marker: `send_end_marker(self, recipient_id: str)` => `send_end_marker(self, marker_input: MarkerInput)`
 - send\_marker\_message: `send_marker_message(self, recipient_id: str)` => `send_marker_message(self, marker_input: MarkerInput)`
 - create\_marker\_message: create\_marker\_message(self, recipient\_id: str) => `create_marker_message(self, marker_input: MarkerInput) -> MarkerMessageOutput`
 
 where MarkerInput is a Pydantic-based class:
 
 ```
    class MarkerInput(BaseModel):    recipient_id: str    marker_type: MarkerType    step_type: Optional[StepType] = None
    ```
 
- Remove the "hand over" command from the default GPT-5.1 prompt template. The command had already been removed from all other prompt templates in the 3.16.0 release.
 
- Fixed a bug where sub-agent flows would end immediately after interruption resumption instead of re-invoking the agent.
 
 When a flow containing a sub-agent call was interrupted (e.g., by a user digression) and then resumed via `pattern_continue_interrupted`, the agent frame's state remained `INTERRUPTED`. This caused the flow executor to skip the agent call and advance to END, triggering `pattern_completed` prematurely.
 
 The agent frame state is now correctly reset to `WAITING_FOR_INPUT` during flow resumption, ensuring the sub-agent is re-invoked with full conversation history as expected.
 
- Fixed MCP tracing attribute extraction for router-based model groups so tracing no longer crashes with `KeyError('provider')` when instrumenting ReAct sub-agent `send_message` calls.
 
- When interruptions are enabled and bot is not speaking, user utterances should not be checked if they are passing min\_words threshold.
 

## \[3.16.5\] - 2026-04-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3165---2026-04-24 'Direct link to [3.16.5] - 2026-04-24')

Rasa Pro 3.16.5 (2026-04-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-8 'Direct link to Bugfixes')

- Fixed a bug where the validator was mutating the allowed values list of categorical slots in-place, causing `None` entries to accumulate in slots used across multiple flows via subflows. This corrupted the flow retrieval FAISS embeddings during training.
- Fixed multi-agent flows looping back to an agent call when the top stack frame was **interrupted** instead of waiting for input, which could leave multiple agents active and fail graph execution. The loop-back shortcut now applies only when the agent frame is waiting for user input.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.16.4\] - 2026-04-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3164---2026-04-17 'Direct link to [3.16.4] - 2026-04-17')

Rasa Pro 3.16.4 (2026-04-17)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-9 'Direct link to Bugfixes')

- Fixed tracing span attributes for LLM components that use model groups. Spans now carry the correct model group ID and, for router groups, the representative model's attributes (preferring OpenAI when present for prompt token counting). The `llm_is_router_group` attribute is also set on spans when a router group is active.
- Update `cryptography` to version 46.0.7 to fix a security vulnerability. For more details, see CVE-2026-39892.

## \[3.16.3\] - 2026-04-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3163---2026-04-09 'Direct link to [3.16.3] - 2026-04-09')

Rasa Pro 3.16.3 (2026-04-09)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements 'Direct link to Improvements')

- Replaced the generic "Start the server from your IDE" message after `rasa tools init` with IDE-specific instructions for Claude Code, Cursor, VS Code, and JetBrains.
- Added `rasa tools status` command that displays current project configuration and runtime readiness.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-10 'Direct link to Bugfixes')

- Fixed validation incorrectly rejecting custom ContextualResponseRephraser subclasses configured as nlg.type in endpoints.yml. The validator now resolves the class and checks inheritance instead of only accepting the literal string "rephrase".
- Updated `PyJWT`, `werkzeug`, `aiohttp`, `orjson`, `pyasn1`, `requests`, and `ujson`to resolve security vulnerabilities.

## \[3.16.2\] - 2026-04-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3162---2026-04-02 'Direct link to [3.16.2] - 2026-04-02')

Rasa Pro 3.16.2 (2026-04-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-11 'Direct link to Bugfixes')

- Fixed correction reset when a user answers the active collect slot and corrects another slot in the same turn.
 
 `CorrectSlotsCommand.create_correction_frame` no longer chooses the reset step only from slots in the correction payload. It also considers the current collect-information step and picks the earlier collect step in flow order, so the stack resets to the question currently being asked instead of jumping ahead to a later slot’s collect step (which skipped validation for the new value on the active slot).
 

## \[3.16.1\] - 2026-04-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3161---2026-04-01 'Direct link to [3.16.1] - 2026-04-01')

Rasa Pro 3.16.1 (2026-04-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-12 'Direct link to Bugfixes')

- Fixed three bugs in `ConcurrentRedisLockStore` that caused an `IndexError: deque index out of range` crash in the PII anonymization and deletion cron jobs under multi-replica deployments.
 
 - `get_lock()` now returns `None` when no ticket keys exist in Redis (all tickets had expired), instead of returning an empty `ConcurrentTicketLock` object that caused downstream callers to crash.
 - `save_lock()` now skips the Redis write when the ticket has already expired (TTL ≤ 0), preventing a `ResponseError` from Redis.
 - `save_lock()` previously passed the absolute epoch expiry timestamp as the Redis `EX` TTL, resulting in ticket keys that effectively never expired (~55 years). The TTL is now correctly computed as a relative duration in seconds.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-1 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.16.0\] - 2026-03-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3160---2026-03-26 'Direct link to [3.16.0] - 2026-03-26')

Rasa Pro 3.16.0 (2026-03-26)

### Breaking changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#breaking-changes 'Direct link to Breaking changes')

Upgrading may require config edits, code updates, or retraining. All incompatible changes for this release are listed below in one place.

- Default language models used by built-in LLM features have changed. If you need the same behavior as before, pin the previous model names in `endpoints.yml`.
 
 | Component | New default model |
 | --- | --- |
 | `LLMBasedCommandGenerator` | `gpt-5.1-2025-11-13` |
 | `CompactLLMCommandGenerator` | `gpt-5.1-2025-11-13` |
 | `SearchReadyLLMCommandGenerator` | `gpt-5.1-2025-11-13` |
 | `ContextualResponseRephraser` | `gpt-5.1-2025-11-13` |
 | `IntentlessPolicy` | `gpt-5-mini-2025-08-07` |
 | `MCPBaseAgent` | `gpt-5.1-2025-11-13` |
 | `EnterpriseSearchPolicy` | `gpt-5-mini-2025-08-07` |
 | `LLMBasedRouter` | `gpt-5-mini-2025-08-07` |
 | `ConversationRephraser` | `gpt-5-mini-2025-08-07` |
 
- Default prompts for command generation changed. If you already set a custom `prompt_template` in `config.yml`, that file is still used.
 
- Default prompts for ReAct agents changed (task vs general assistant behavior and `task_completed` guidance). If you override templates, compare your custom files against the latest templates in `rasa/agents/templates/`.
 

, [#4851](https://github.com/rasahq/rasa-private/issues/4851), [#4867](https://github.com/rasahq/rasa-private/issues/4867): Agent interface changed.

1. `process_output` was removed from the shared agent protocol. Use `process_agent_output` for `A2AAgent` implementations and `process_tool_output` for ReAct/MCP agent implementations:
 
 ```
    # A2A agentsasync def process_agent_output(self, output: AgentOutput) -> AgentOutput# ReAct/MCP agentsasync def process_tool_output(    self,    current_iteration_tool_results: Dict[str, AgentToolResult],    cumulative_tool_results: Dict[str, AgentToolResult],    output_channel: Optional[OutputChannel] = None,) -> List[Event]
    ```
 
2. `run` signature changed with optional cooperative cancellation support:
 
 ```
    async def run(    self,    input: "AgentInput",    output_channel: Optional[OutputChannel] = None,    cancellation_token: Optional["CancellationToken"] = None,) -> "AgentOutput"
    ```
 
3. Custom tool callable signature changed:
 
 - Before: `Callable[[Dict[str, Any]], AgentToolResult]`
 - After: `Callable[[Dict[str, Any], AgentToolContext], AgentToolResult]`
 - Import `AgentToolContext` via `from rasa.agents.schemas import AgentToolContext`.

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals 'Direct link to Deprecations and Removals')

- Remove `HumanHandoff` command from the codebase and default prompt templates.
- Remove generic `process_output` method from `AgentProtocol`. In ReAct / MCP agents, use `process_tool_output` for tool-result-based post-processing. In `A2AAgent`, use `process_agent_output` method.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features 'Direct link to Features')

- Added optional `reasoning_effort` on LLM model configuration for providers and models that support it (for example OpenAI and Azure OpenAI reasoning-capable models). By default, Rasa tries to apply the lowest appropriate `reasoning_effort` for the resolved model name (using provider metadata where available). We recommend using the lowest supported value unless you explicitly need deeper reasoning. Any value you set explicitly is left unchanged. Models that do not support this parameter do not receive a default. For Azure deployments configured without a concrete `model` name, Rasa can resolve the backing model and set `reasoning_effort` after the first probe response.
 
 Example configuration in `endpoints.yml`:
 
 ```
    model_groups:  - id: openai-reasoning    models:      - provider: openai        model: gpt-5-mini-2025-08-07        reasoning_effort: "minimal"
    ```
 
- Added `max_polling_time` and `polling_initial_delay` to A2A agent configuration so polling parameters can be set per agent. Defaults remain 60 seconds and 0.5 seconds respectively when not set.
 
 To override polling for an A2A agent, set the options in the agent's configuration:
 
 ```
    agent:  name: my_agent  protocol: A2A  # ...configuration:  agent_card: "https://example.com/agent-card.json"  max_polling_time: 120        # max total polling time in seconds (default: 60)  polling_initial_delay: 1.0   # initial delay before first poll in seconds (default: 0.5)  # ... other configuration (timeout, max_retries, etc.)
    ```
 
- Added filler message support for ReAct agents via a new `enable_filler_messages` config flag. Filler messages are **enabled by default** and are streamed to the user before tool execution, with streaming capability determined by a new `supports_streaming` property on `OutputChannel`.
 
 To disable filler messages for an agent, set `enable_filler_messages: false` in the agent's configuration:
 
 ```
    agent:  name: my_agent  # ...configuration:  enable_filler_messages: false  # ... other configuration (llm, timeout, etc.)
    ```
 
- Implements user-scoped tracker querying and durable conversation metadata across all tracker stores. Persists `user_id` and `conversation_started_timestamp` in the tracker object to use for querying and ordering:
 
 - serialisation now omits null user\_id
 - deserialisation handles `JSONDecodeError`
 - Ensures `conversation_started_timestamp` backfilled on save/update for backward compatibility
 
 Summary of tracker store changes for user-scoped retrieval of conversation trackers:
 
 - SQL: introduces users table (sender\_id→user\_id, timestamp) with upsert per dialect; JOIN-based retrieval and DB-level ordering/pagination; cleanup on delete
 - Redis: maintains secondary index `user_trackers:\{user_id\}` (Sorted Set) for O(1) lookup; batch mget, robust deserialisation, index cleanup
 - Mongo: creates indices on user\_id and (conversation\_started\_timestamp, sender\_id); aggregation pipeline for ordering/pagination; restores user\_id
 - Dynamo: saves user\_id and conversation\_started\_timestamp (Decimal) in update\_item; GSI support (user\_id-index with sort on conversation\_started\_timestamp) with full-scan fallback
- Added optional `user_id` parameter to `DialogueStateTracker` to enable associating multiple conversations with a single end user.
 
- Implemented conversation lifecycle management with `ConversationInactive` and enhanced `SessionEnded` events. Conversations can now be marked as inactive (resumable via user message or `ConversationResumed` event) or terminated (permanently ended, blocking all further updates).
 
 Added a new `session_config` option `start_session_after_expiry` that controls what happens when a session expires (based on `session_expiration_time`).
 
 - When `true` (default), Rasa emits a `SessionStarted` event and starts a new session, ensuring backward compatibility.
 - When `false`, expired sessions simply continue without automatically starting a new session.
- Add REST API endpoint `/users/{user_id}/trackers` to retrieve all conversations for a given user.
 
 This admin endpoint provides user-scoped tracker querying with:
 
 - **Authentication**: Requires token or JWT authentication
 - **Pagination**: Supports `limit` and `offset` query parameters for efficient pagination
 - **Event verbosity**: Configurable via `include_events` query parameter (NONE, APPLIED, AFTER\_RESTART, ALL)
 - **Error handling**: Proper validation and HTTP status codes for invalid inputs
 - **Performance**: Database-level sorting and pagination for efficient queries with 1000+ conversations
 
 **Important**: This is an **admin endpoint** that should be used as a proxy or in a backend service to manage what data is exposed to the client. Direct client access should be avoided to maintain security and control over sensitive user data.
 
 Example usage:
 
 ```
    GET /users/{user_id}/trackers?limit=50&offset=0
    ```
 
 Response format:
 
 ```
    {  "conversations": [...],  "limit": 50,  "offset": 0}
    ```
 
- Implemented Multilingual Voice AI Agents. The configuration of Automatic Speech Recognition (ASR) and Text to Speech (TTS) changes through the conversation based on the value of built-in `language` slot. For each ASR and TTS Engine, users can configure the language, model or voice to be used for each value of `language` slot. In case of ASR Engines, reconfiguration requires a websocket reconnect. Rasa manages that on the fly to ensure that no speech is lost. The signature of certain methods in classes `ASR_Engine` and `TTS_Engine` have also changed. These methods are, **init** and from\_config\_dict.
 
- Implemented session-scoped conversations. Each conversation session now has a unique `session_id` (UUID v4) that is automatically generated and attached to all events within that session. This enables better tracking and analytics of user sessions. Session IDs are automatically generated on session start or resume events and propagated to event metadata.
 
 Added `current_session_id` field to DialogueStateTracker. The session ID is automatically:
 
 - Generated as a UUID4 when a new session starts or resumes. Generation triggers include:
 - Brand new conversations (first event on a tracker)
 - SlotSet for session metadata slot
 - `action_session_start` or `SessionStarted` events
 - `ConversationResumed` after inactivity
 - `UserUttered` reactivating an inactive conversation
 
 Added `session_id` to event metadata. The generated ID is automatically included in the metadata of all events created during a conversation session.
 
- Add `timer_store` configuration to `endpoints.yml` to enable configurable background session timer storage.
 
- Add MCP `_meta` parameter support so you can pass user-specific data to MCP servers without exposing it to the LLM.
 
 - **Configuration:** Optional `meta_map` per MCP server in `endpoints.yml` with:
 - **`static`**: Fixed key-value pairs always sent in `_meta`
 - **`from_slots`**: Slot-to-mcp key mappings; slot values are sent under the given param in `_meta`
 - **Execution:** Both flow-based and agent-based MCP tool calls build and send `_meta` when `meta_map` is configured. A compatibility layer supports current MCP SDK versions that do not expose `meta` on `call_tool`.
 - **Agent-based calls:** For `from_slots`, values are read from agent input only when the slot is present; slots that have been removed from agent input will not appear in `_meta`.
 
 Example `endpoints.yml`:
 
 ```
    mcp_servers:  - name: internal_api_server    url: http://internal-api:8000/mcp/    type: http    meta_map:      static:        api_version: "v2"        source: "rasa_agent"      from_slots:        - slot: user_id          param: user_id        - slot: role          param: user_role
    ```
 
 This replaces the need to override private Rasa methods (e.g. `execute_mcp_tool`, `send_message`) to inject auth or context into MCP requests.
 
- Implemented Session Timer System for automatic session expiration. Conversations are now automatically marked as inactive after a configurable timeout period, supporting both single-instance and horizontally-scaled deployments.
 
 - Timers are scheduled when a session starts and reset on each user message
 - When a timer expires, a `ConversationInactive` event is emitted
 - Race condition handling ensures correct behavior in multi-pod deployments
 - Redis-backed storage enables timer persistence across restarts
 - Automatic recovery of expired timers on server startup
 - Fallback to in-memory storage during Redis outages
 - `SessionTimerStore` / `SessionTimerManager` abstract base classes
 - `InMemorySessionTimerStore` / `InMemorySessionTimerManager` for single-instance deployments
 - `RedisSessionTimerStore` / `RedisSessionTimerManager` for distributed deployments
- Implemented out-of-the-box collection of customer satisfaction.
 
 - Added built-in `pattern_customer_satisfaction` for CSAT collection.
 - The pattern is triggered automatically via a link from `pattern_completed` when users decline to continue the conversation.
 - It collects a `csat_score` slot (categorical: `satisfied`/`unsatisfied`) and responds with appropriate thank-you messages.
 - The pattern, its responses (`utter_ask_csat_score`, `utter_csat_thank_you_satisfied` and `utter_csat_thank_you_unsatisfied`) and when the pattern is triggered are all fully customizable.
- Handle `SessionEnded` events returned from custom actions gracefully by stopping the prediction loop and flow executor when the conversation is terminated.
 
- Added the system action `action_handoff_metric` to mark conversations as "not contained" for containment metrics. Any conversation that includes this action emits an `ActionExecuted` Kafka event and is counted as not contained. The action is a no-op; no user-visible behavior or migration is required.
 
 Updated the default `pattern_human_handoff` flow to run `action_handoff_metric` before `utter_human_handoff_not_available`, so existing assistants automatically emit the metric when the human handoff pattern is used. If you have overridden `pattern_human_handoff` in your flows, you can add a step `action: action_handoff_metric` at the start of that flow to include those conversations in containment metrics.
 
- Add support for multiple Audio Formats in Rasa. These are G.711 μ-law (legacy) along with Linear PCM 24kHz and Linear PCM 48kHz.
 
- Extended support for Audio Formats, Updated Azure and Deepgram Automatic Speech Recognition (ASR) to support multiple audio formats. G.711 μ-law, Linear PCM 24kHz and Linear 48kHz. Updated Azure, Deepgram, and Cartesia Text to Speech (TTS) to support the same audio formats. Rime TTS only supports G.711 μ-law and Linear PCM 24kHz. Updated Jambonz and Audiocodes Stream Channel to use Linear PCM 24kHz audio when sending or receiving audio from the platform.
 
- Add prompt templates and the corresponding model mapping for GPT-5.1 and GPT-5.2.
 
- Update default models for LLM-based components: `CompactLLMCommandGenerator`, `SearchReadyLLMCommandGenerator`, `LLMBasedCommandGenerator`, `ContextualResponseRephraser` and `MCPBaseAgent` will use `gpt-5.1-2025-11-13`; `EnterpriseSearchPolicy`, `LLMBasedRouter`, `ConversationRephraser` and `IntentlessPolicy` will use `gpt-5-mini-2025-08-07`.
 
- Browser Audio channel uses Linear PCM 48kHz audio with ASR and TTS components. The sample rate of this encoding can be configured with `sample_rate` property of the channel. Supported values in kHz are 8000, 24000 and 48000 Voice Inspector has been updated to receive and play Linear PCM at any of the supported sample rates. The sample rate is sent to the Voice Inspector in a handshake message.
 
- ReAct agent now uses the streaming LLM API and consumes the response incrementally instead of buffering the full stream. Content tokens are delivered to the user in real time when the channel supports streaming. Tool calls are executed after the stream segment completes and their results are sent in one shot. This reduces perceived latency.
 
- Voice channels enforce two type of minimum gaps between bot audio: a shorter gap for regular back-to-back utterances, and a longer gap following filler messages (e.g., when streaming assistant text alongside tool calls from ReAct-style agents). After each message is sent, when the next message arrives, the channel checks the time since the last message was sent against the configured minimum delay. If more time is needed, generates silence audio for that duration, sends the silence to the stream, and then sends the next message. Consecutive turns are therefore naturally spaced.
 
- Add prompt templates and the corresponding model mapping for Claude Sonnet 4.5 and Claude Sonnet 4.6.
 
- Add public method `process_tool_output()` in `MCPBaseAgent` for implementing custom post-processing logic based on the tool results in each iteration.
 
- Support metadata pass-through in A2AAgent so that specialized data can be passed to the agent's backend via `AgentInput.metadata` without being processed by the LLM. The metadata is forwarded to the `Message` sent to the A2A server.
 
- - Add graceful cancellation of long-running A2A agent operations (both polling and streaming). When `ConversationInactive` event is added to the tracker after `session_expiration_time` minutes while an A2A task is in progress, the operation is now cancelled promptly instead of running until timeout. **Note**: This is a model breaking change. Please retrain your model.
 - Introduce a public method `Agent.cancel_background_tasks(sender_id)` that can be used to explicitly trigger the cancellation of the background A2A processing (both streaming and polling) from other components (e.g. from custom channels).
 - Add a new POST `/cancel_background_tasks/<sender_id>` endpoint in the REST channel that allows frontends and external systems to explicitly cancel background processing for a conversation.
 - Closing the Inspector browser tab now automatically cancels any in-flight A2A operations for that conversation.
- `AgentInput.metadata` is now propagated to custom tool execution. Tool executors receive `(args, context)`, where `args` contains the LLM-generated tool parameters and `context` is an `AgentToolContext`. The request metadata is available via `context.metadata` and it can be set by the agent in the `process_input` method using the `AgentInput.metadata` attribute.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-1 'Direct link to Improvements')

- The continue-interrupted pattern is now triggered after a knowledge (e.g. EnterpriseSearch) answer, so users are asked if they want to resume previously interrupted flows when returning from a search.
 
- When an agent was interrupted and then resumed, it now gets reinvoked with the current context and a system instruction that it was interrupted and may use the new context or ask again. Previously, the agent only repeated its last message and did not run again.
 
- Add `SessionPaused` event which purpose is to be triggered by external systems when a session is paused. It has the following properties:
 
 - `event` - which is always set to `session_paused`
 - `reason` - which describes why the session was paused
 - `timestamp` - (optional) Unix timestamp, which indicates when the session was paused
 
 If `timestamp` is not provided, the current Unix timestamp of the Rasa Pro server will be used.
 
 This event can be used to notify pipelines bout the pause state of a session, allowing them to take appropriate actions based on the session's status.
 
 To use this event, external systems should trigger it when a session is paused, via `POST` `conversations/<sender id>/tracker/events`.
 
 Example of the payload for the POST request:
 
 ```
    {"event": "session_paused", "timestamp": 1761232628.512342, "reason": "Widget closed"}
    ```
 
- Refactor Copilot response handling to support structured streaming from the agent SDK copilot and improve text content processing. The new implementation properly segments text content parts into start, delta, and end events, handles controlled predictions, and fixes buffer handling issues in the legacy copilot handler.
 
- Events streamed to the event broker now include `user_id` in the message payload alongside `sender_id`. This allows downstream consumers to identify the end user associated with each conversation event.
 
- Add support to stream audio chunks from Azure Speech Service via websocket. Rasa will use streaming via websocket when response does not contain any SSML tags. If response contains any SSML tag, Rasa will default to use HTTPS to create audio chunks from Azure Speech Services. Improve interruption handling by:
 
 - stopping stream from Azure Speech Services when interruption happens
 - stopping to send audio chunks from Rasa to external voice service when interruption happens
- Improve `action_session_start` execution to ensure it runs only once (coordinating calls from the `MessageProcessor` and the `FlowPolicy` when it triggers `pattern_session_start`).
 
- Update eligibility and session-splitting logic for privacy cron jobs in line with the new session management improvements.
 
 - when USER\_CHAT\_INACTIVITY\_IN\_MINUTES is unset, “new” trackers are processed based on ConversationInactive/SessionEnded events and session\_id grouping (with a grace period of min\_after\_session\_end), while “legacy” trackers without session\_id fall back to a fixed 30min + min\_after\_session\_end threshold.
 - when the env var is set, time-based behavior remains but is now deprecated and applied per session.
 
 SQL tracker store: `update()` now supports a content-only path when the timestamp-based delete removes no rows (e.g. anonymization: same timestamps, content changed). In that case, the store either updates existing event rows in place where content differs or performs a full replace if event counts differ, so anonymized content is persisted correctly in a single atomic transaction.
 
 Anonymization cron job: the privacy manager now uses `update(updated_tracker)` for anonymization instead of delete-then-save. SQL tracker store behavior is aligned with MongoDB, Redis and DynamoDB (overwrite semantics), and anonymization with SQL completes in one atomic operation without a separate delete/save sequence.
 
- Cancel timer unconditionally when a `SessionEnded` event is appended via the REST API, regardless of whether `execute_side_effects` is requested.
 
- Add `model_name` to event metadata.
 
- Add `tool_timeout` for MCP/ReAct agents configuration to configure tool-call timeout in seconds.
 
- Enabled deletion of a Studio assistant during upload. Introduced a new `--dangerously-delete-existing` flag to `rasa studio upload`. When set, any existing Studio assistant with the same name is deleted before the upload proceeds, with no interactive confirmation.
 
- Added logic for signaling interruptions in tts streaming. Sendind a variation of clear command to stop streaming audio and clear the buffer to prepare for the next response.
 
- Added a warning log when `carry_over_slots_to_new_session` and `start_session_after_expiry` are both set to `false` in the domain session config, as `carry_over_slots_to_new_session` has no effect when no session boundary is created on expiry.
 
- Use the prompt template for `gpt-5.1-2025-11-13` as new default prompt template in the `CompactLLMCommandGenerator` and `SearchReadyLLMCommandGenerator`.
 
- SQL and Mongo tracker stores now slice the latest conversation session using the same boundary as other tracker stores: `ActionExecuted(action_session_start)`. That action is always executed by the message processor when a new session starts, while `SessionStarted` may be omitted by a custom action-session-start action. Incremental save offsets and SQL partial-fetch queries were updated to match this boundary.
 
- Add normalization of `AgentOutput.events` so entries from ReAct/A2A customizations may be either Rasa `Event` instances or serialized event dicts; dicts are parsed into proper events before flow handling (e.g. stack metadata attachment), and invalid entries are dropped with structured error logs.
 
- Declared `safetensors` and `keras` as optional NLU extras (`pip install 'rasa-pro[nlu]'` or `full`), so minimal installs no longer pull them in directly. `regex` is also declared optional and listed under those extras for consistency with `WhitespaceTokenizer`, but it remains installed on most minimal installs because the base dependency `tiktoken` (used by the LLM stack) depends on `regex`.
 
 The affected code paths lazy-import `safetensors` and `regex`; if either package is not installed, they raise `MissingDependencyException` with install instructions. `keras` is only pulled in for the TensorFlow model stack (`rasa.utils.tensorflow.models`), so it is optional for the same minimal-install story as `safetensors`.
 
- Improved sub-agent call steps that use `exit_if`: when the same ReAct agent runs again in one conversation, slots named in `exit_if` are cleared so a new run does not reuse values from the last finished run. Ongoing multi-turn agent sessions (waiting for the next user message) are unchanged.
 
- Two improvements to `DynamoTrackerStore`:
 
 - **Idempotent `save()`**: concurrent or retried saves no longer produce duplicate events. New conversations are written via `put_item` with `attribute_not_exists(sender_id)`, ensuring the full tracker is stored exactly once. Subsequent saves append only new last-turn events via a conditional `update_item` guarded by `last_event_timestamp`, which atomically rejects any append whose events are already stored — without requiring a read-before-write.
 - **Reused boto3 resource**: `boto3.resource("dynamodb")` is now instantiated once in `__init__` as `self._dynamo` and shared across all methods, eliminating repeated resource construction per call. The connection pool size is configurable via the `max_pool_connections` constructor argument (default: 50) to prevent urllib3 pool saturation under concurrent load.
- The `GET /users/{user_id}/trackers` endpoint no longer replays conversation history through `DialogueStateTracker.from_dict()` and `current_state()` on every request. Each tracker store now implements `get_serialized_trackers_by_user_id()`, which returns serialized event dicts directly from storage without event replay.
 
 Additional store-specific improvements included with this change:
 
 - **SQL**: replaced an N+1 query loop (one `retrieve_full_tracker` call per conversation) with two bulk queries — one paginated `users` table query and one `events IN (...)` fetch — eliminating query count growth with conversation volume.
 - **Redis**: orphaned sorted-set index entries are now removed in a single batched `ZREM` call instead of one call per orphan.
 - **All stores**: `current_session_id` is now derived consistently from the last event's metadata rather than from stored state, correctly returning `None` when the last event is `ConversationInactive`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-13 'Direct link to Bugfixes')

- Restarted task-oriented ReAct agents no longer exit immediately. When an agent is restarted, the previous run's user/assistant messages are now marked in the conversation so the model does not set slots from them; the system prompt instructs the agent to treat the current interaction as a fresh start for slot collection.
 
- Fix potential Tensor shape mismatch error in `TEDPolicy` and `DIETClassifier`.
 
- Fix bug with the missing `language_data` module by installing `langcodes` with the `data` extra dependency.
 
- Fixed `language` parameter in RimeTTS configuration to be passed correctly to the Rime API. Temporarily changed RimeTTS to non-streaming input mode as a workaround for a Rime API bug that impacts conversation experience; audio is now generated from the complete response at once.
 
- Update `mcp` version to `~1.23.0` to address security vulnerability CVE-2025-66416.
 
- Fix bug in `CommandPayloadReader` where regex matching did not account for list slots, leading to incorrect parsing of slot keys and values. Now, slot names and values are correctly extracted even if a list is provided.
 
- Previously, `DialogueStateTracker.has_coexistence_routing_slot` could incorrectly return `True` when the tracker was created without domain slots (i.e., using `AnySlotDict`), because `AnySlotDict` pretends all slots exist. Now, the property returns False in that case, so the routing slot is only considered present if it is actually defined in the domain.
 
- Fix LLM prompt template loading to raise an error instead of just printing a warning when a custom prompt file is missing or cannot be read.
 
- Fix `AttributeError: 'str' object has no attribute 'pop'` when using `KnowledgeAnswerCommand` with tracing enabled.
 
- Fix `UnboundLocalError` in `E2ETestRunner._handle_fail_diff` that caused e2e tests to crash when processing `slot_was_not_set` assertion failures.
 
- Fixed button payloads with nested parentheses (e.g., `/SetSlots(slot=value)`) not being parsed correctly in `rasa shell`. The payload was incorrectly stripped of its `/SetSlots` prefix, causing it to be treated as text instead of a slot-setting command.
 
- Add missing deprecation warning for legacy `RASA_PRO_LICENSE` environment variable. Update license validation error messages to reference the actual environment variable used.
 
- Include response button titles in conversation history in prompts of LLM based components.
 
- Fixed e2e test coverage report to correctly track coverage per flow. Each flow now appears as a separate entry with accurate coverage percentages. Additionally, `call`/`link` steps and `collect` steps with prefilled slots are now properly marked as visited.
 
- Fix DUT fails with AttributeError when running with custom CommandGenerator
 
- Fix OAuth2AuthStrategy to use URL-encoded parameters with Basic Authentication to fetch access token
 
- Remove config file content from endpoint read success logs to prevent sensitive data exposure.
 
- The MCP task agent now includes slot changes in the agent output when the state is `INPUT_REQUIRED` or when max iterations is reached. Previously, slots set via set\_slot tools were updated in memory but not forwarded in the output, so they were not persisted until exit conditions were met.
 
- Fix race conditions in PII cron jobs by requiring a LockStore for BackgroundPrivacyManager and acquiring a per-sender\_id lock while anonymization/deletion jobs read-modify-write trackers.
 
- Rasa needs to check if it is in listening mode before it triggers the silence timeout watcher. Silence timeout watcher will be triggered if all are true:
 
 - User is not speaking
 - Bot is not speaking
 - Rasa is in listening mode
 
 If any of the conditions above are not true, Rasa will cancel the silence timeout watcher.
 
- Channel specific silence\_timeout can now be set in credentials file. Fixes the type error. If silence\_timeout is not set the channel will use global value.
 
- Stop silence timeout immediately when interruption is detected.
 
- When an A2A task has status `input_required`, Rasa now populates `AgentOutput.structured_results` from both `task.artifacts` and DataParts in `task.status.message.parts`. Previously only the text message was returned and structured data was omitted, so clients can now process artifact data when the agent is waiting for user input.
 
- Fix PII redaction of user or bot messages when slot values do not match the original message text. This can occur when the original messages are multi-line or TTS-style bot read-backs.
 
- Update setuptools to 80.10.2 to address vulnerability [https://osv.dev/vulnerability/DEBIAN-CVE-2026-23949](https://osv.dev/vulnerability/DEBIAN-CVE-2026-23949). Replaced randomname with duoname (no dependencies) library.
 
- Fixed the backend tracing wrapper (e.g., Jaeger or OTLP) around send\_message to correctly forward the output\_channel argument to the underlying implementation.
 
- When an agent or MCP tool call returns a fatal error (e.g. internal server error), the user no longer receives two bot messages. Previously both the internal error message ("Sorry, I am having trouble with that...") and the flow-cancelled message ("Okay, stopping...") were shown. Now only the internal error message is shown; the cancel pattern is not pushed for system-initiated failures, so the flow is still marked ended for tracking but the confusing cancel utterance is omitted.
 
- Add rephrase endpoint validation by treating defaults-only misconfiguration as warning and user-defined rephrase misconfiguration as error.
 
- Issue a warning when ASR and TTS `language_map` is missing languages from `additional_language` property read from `config.yml`.
 
- Fixed replay-safe latest-session handling for all built-in tracker stores. Tracker store `retrieve` method uses `get_latest_replay_safe_session_tracker`, widening the replay prefix when stack patches cannot be applied on the latest-session slice alone.
 
- Fixed `default_language` raising `RasaException("No default language configured")` even when `language:` was properly set in `config.yml`. It happened when the `langauge` slot was reset (set to `None`).
 
- Improved OpenTelemetry histograms for MCP agent LLM usage: `mcp_agent_llm_prompt_token_usage` and `mcp_agent_llm_response_duration` now record a consistent, call-scoped set of attributes (`agent_name`, `execution_context`, and `protocol_type`), making metrics easier to aggregate.
 
- Fixes Implemented:
 
 1. Both default and customized `action_session_start` now raise a Rasa exception when the session is already started for the current message. This exception is caught and handled by `MessageProcessor` which appends `ActionExecutionRejected` event and continues processing the message. This ensures that the message processing flow is not interrupted by the exception and allows for proper handling of the session start action. This change was required to avoid generating new session ids when this action is executed multiple times in the same time-bound tracker session.
 
 2. Fix conversation stalling when a flow contains an explicit `action_session_start` step and the session is already started for the current message.
 
 
 - The root cause explanation: The attempt to execute `action_session_start` a 2nd time for an existing session is rejected. The action execution rejection propagates to the DefaultPolicyPredictionEnsemble, which zeroes out action\_session\_start's confidence across all policy predictions. Since FlowPolicy only predicted that one action (with confidence 1.0), zeroing it leaves only the default action\_listen at 0.0.
 - The fix: detect this condition in FlowPolicy and skip the step immediately, letting FlowPolicy continue the loop and return the actual next flow step's action.
- Fixed empty assistant bubbles in Socket.IO channels (including Inspector) when the model returned whitespace-only or paragraph-only text. Non-streaming LLM responses are now stripped before send and blank segments from `\n\n` splitting are no longer emitted.
 
- Fixed three bugs in `DynamoTrackerStore`:
 
 - **`JsonPatchConflict` on tracker retrieval**: duplicate events appended by concurrent or retried `save()` calls caused `jsonpatch` to attempt operations on already-mutated dialogue stack fields (e.g. removing `frame_type` a second time), raising an unhandled exception. Duplicate events are now removed on load via `_deduplicate_events()`, which deduplicates by full event content to safely handle multiple legitimate events sharing the same timestamp.
 - **Event accumulation across trackers in `get_trackers_by_user_id`**: `events_with_floats` was declared outside the per-item loop in `_process_dynamodb_items`, causing each reconstructed tracker to accumulate all preceding trackers' events. Each tracker now gets its own isolated events list.
 - **Pre-`UserUttered` events lost on first save**: `save()` only appended last-turn events (from the latest `UserUttered` timestamp onwards), so events emitted before the first user message — such as `action_session_start`, `session_started`, and initial `slot` events — were never persisted for new conversations. First saves now use `put_item` to store the complete tracker state.
- Upgrade `langsmith` dependency to 0.6.3 to address CVE-2026-25528.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-2 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.15.27\] - 2026-06-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31527---2026-06-04 'Direct link to [3.15.27] - 2026-06-04')

Rasa Pro 3.15.27 (2026-06-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-14 'Direct link to Bugfixes')

- Fixed a spurious "Fatal error on SSL transport" traceback printed to stderr on Windows after `rasa test e2e` completes by explicitly closing litellm's shared httpx sessions before the event loop shuts down.
 
- Fixed a crash in rasa-tools where training an assistant with a sub-agent MCP server configured caused a `RuntimeError: Attempted to exit cancel scope in a different task than it was entered in`, crashing the stdio server and making the next tool call fail with `MCP error -32000: Connection closed`.
 
- Fix two bugs causing random crashes and resource leaks when running Rasa with --workers 2+: a pre-fork event loop left open during JWT setup, and litellm async HTTP clients not being closed on worker shutdown.
 
 Root cause — Sanic uses fork-based multi-worker mode (legacy=True). create\_app() was calling asyncio.new\_event\_loop() + set\_event\_loop() in the main process to satisfy sanic-jwt's Initialize(), then leaving the loop open. Forked workers inherited its kqueue/epoll file descriptors, conflicting with each worker's own loop (~50% crash at startup). Separately, close\_resources() never called litellm.close\_litellm\_async\_clients(), leaving cached async HTTP clients open when workers exit.
 
- An explicitly configured `api_key` in the `rasa` provider model config now takes precedence over the Rasa license, allowing custom endpoints to authenticate with their own credentials.
 
- Transient tracker store connection errors (e.g. Postgres restart) no longer flood logs with full stack traces on each retry attempt; the error message is preserved on one line.
 
- Fix `resume_flow()` marking the wrong `AgentStackFrame` as resumed when a flow contains multiple sub-agent frames, which prevented `run_agent()` from taking its resume branch.
 
- Fixed multiple Langfuse session IDs being created for a single Dialogue Understanding Test (DUT) run by using the original test-case sender ID as the session ID for all per-step LLM traces.
 
- Upgrading from 3.15.x to 3.16.x no longer causes every subsequent message to fail with `InvalidFlowStepIdException` when a tracker was mid-flow at upgrade time and a flow step ID was renamed or removed. The flow executor now detects stale step IDs, logs a structured warning, removes the stale frame, and continues processing — rather than propagating the exception as a `GraphComponentException`.
 

## \[3.15.26\] - 2026-06-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31526---2026-06-02 'Direct link to [3.15.26] - 2026-06-02')

Rasa Pro 3.15.26 (2026-06-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-15 'Direct link to Bugfixes')

- Fixed graph schema validation on Python 3.13+ when components use PEP 604 union type hints (e.g. `Domain | None`), which previously raised a false `GraphSchemaValidationException`.
- Fixed MCP flow tool tracing metrics when `_execute_mcp_tool_call` mutates the shared `initial_events` list in place by capturing the pre-mutation event count for output slicing, and by detecting failures from `InternalErrorPatternFlowStackFrame` on the dialogue stack rather than from unchanged event list length.
- Updated `idna`, `pip` and `urllib3` to address security vulnerabilities.

## \[3.15.25\] - 2026-06-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31525---2026-06-01 'Direct link to [3.15.25] - 2026-06-01')

Rasa Pro 3.15.25 (2026-06-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-16 'Direct link to Bugfixes')

- Fixed DIETClassifier training failures on GPU with TensorFlow 2.19 and Keras 3, where training could fail with an `InvalidArgumentError` about unsupported `SparseReshape` operations on XLA GPU JIT devices. `RasaModel` now inherits from `tensorflow.keras.Model` instead of standalone Keras 3's `Model`, so it uses the same Keras stack as the rest of Rasa's TensorFlow code and respects `TF_USE_LEGACY_KERAS=1`.

## \[3.15.24\] - 2026-05-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31524---2026-05-22 'Direct link to [3.15.24] - 2026-05-22')

Rasa Pro 3.15.24 (2026-05-22)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-17 'Direct link to Bugfixes')

- Fix an interrupted sub-agent not being properly resumed after the user answers "Yes" to `pattern_continue_interrupted`. This is fixed by introducing a transient `RESUMING` agent state that sits between `INTERRUPTED` and `WAITING_FOR_INPUT`: `resume_flow()` now sets the agent frame to `RESUMING` instead of directly to `WAITING_FOR_INPUT`; `run_agent()` detects this and dispatches to `_handle_resume_interrupted_agent`, which replays the agent's last message and transitions the frame back to `WAITING_FOR_INPUT`.
- Fixed a `KeyError` in `ContinueInterruptedPatternFlowStackFrame.from_dict` that occurred when loading tracker events stored before the multi-interrupt fields were introduced. This caused repeated tracker store retries that blocked the server event loop, leading to liveness probe failures and pod restarts.

## \[3.15.23\] - 2026-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31523---2026-05-15 'Direct link to [3.15.23] - 2026-05-15')

Rasa Pro 3.15.23 (2026-05-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-18 'Direct link to Bugfixes')

- Updated security-flagged dependencies:
 - `litellm`: `1.82.4` → `1.84.0`
 - `langchain`: `~0.3.27` → `~1.2.16`
 - `langchain-community`: `~0.3.29` → `~0.4.1`
 - `langchain-core`: `~0.3.81` → `~1.3.2`
 - `langsmith`: `~0.6.3` → `~0.7.31`
 - `langchain-qdrant`: `~0.2.1` → `~1.1.0`
 - `langchain-openai`: transitive → `~1.2.1` (new direct pin)
 - `langchain-text-splitters`: transitive → `~1.1.2` (new direct pin)
- The `no_text_normalization` parameter is now correctly passed to the Rime TTS WebSocket connection, allowing users to skip text normalization to reduce latency.

## \[3.15.22\] - 2026-05-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31522---2026-05-06 'Direct link to [3.15.22] - 2026-05-06')

Rasa Pro 3.15.22 (2026-05-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-19 'Direct link to Bugfixes')

- Fixed a bug where sub-agent flows would end immediately after interruption resumption instead of re-invoking the agent.
 
 When a flow containing a sub-agent call was interrupted (e.g., by a user digression) and then resumed via `pattern_continue_interrupted`, the agent frame's state remained `INTERRUPTED`. This caused the flow executor to skip the agent call and advance to END, triggering `pattern_completed` prematurely.
 
 The agent frame state is now correctly reset to `WAITING_FOR_INPUT` during flow resumption, ensuring the sub-agent is re-invoked with full conversation history as expected.
 
- Fixed MCP tracing attribute extraction for router-based model groups so tracing no longer crashes with `KeyError('provider')` when instrumenting ReAct sub-agent `send_message` calls.
 
- When interruptions are enabled and bot is not speaking, user utterances should not be checked if they are passing min\_words threshold.
 

## \[3.15.21\] - 2026-04-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31521---2026-04-24 'Direct link to [3.15.21] - 2026-04-24')

Rasa Pro 3.15.21 (2026-04-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-20 'Direct link to Bugfixes')

- Fixed a bug where the validator was mutating the allowed values list of categorical slots in-place, causing `None` entries to accumulate in slots used across multiple flows via subflows. This corrupted the flow retrieval FAISS embeddings during training.
- Fixed multi-agent flows looping back to an agent call when the top stack frame was **interrupted** instead of waiting for input, which could leave multiple agents active and fail graph execution. The loop-back shortcut now applies only when the agent frame is waiting for user input.
- Fixed tracing span attributes for LLM components that use model groups. Spans now carry the correct model group ID and, for router groups, the representative model's attributes (preferring OpenAI when present for prompt token counting). The `llm_is_router_group` attribute is also set on spans when a router group is active.

## \[3.15.20\] - 2026-04-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31520---2026-04-10 'Direct link to [3.15.20] - 2026-04-10')

Rasa Pro 3.15.20 (2026-04-10)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-21 'Direct link to Bugfixes')

- When interruptions are ENABLED, user message should be queued for processing only if it passes interruption criteria.
 
- If interruptions are disabled, user message should be queued only if user speaks during collect step utterance.
 
 Updated signature of following methods:
 
 - send\_start\_marker: `send_start_marker(self, recipient_id: str)` => `send_start_marker(self, marker_input: MarkerInput)`
 - send\_intermediate\_marker: `send_intermediate_marker(self, recipient_id: str)` => `send_intermediate_marker(self, marker_input: MarkerInput)`
 - send\_end\_marker: `send_end_marker(self, recipient_id: str)` => `send_end_marker(self, marker_input: MarkerInput)`
 - send\_marker\_message: `send_marker_message(self, recipient_id: str)` => `send_marker_message(self, marker_input: MarkerInput)`
 - create\_marker\_message: create\_marker\_message(self, recipient\_id: str) => `create_marker_message(self, marker_input: MarkerInput) -> MarkerMessageOutput`
 
 where MarkerInput is a Pydantic-based class:
 
 ```
    class MarkerInput(BaseModel):    recipient_id: str    marker_type: MarkerType    step_type: Optional[StepType] = None
    ```
 
- Fixed validation incorrectly rejecting custom ContextualResponseRephraser subclasses configured as nlg.type in endpoints.yml. The validator now resolves the class and checks inheritance instead of only accepting the literal string "rephrase".
 
- Updated `PyJWT`, `werkzeug`, `aiohttp`, `orjson`, `pyasn1`, `requests`, and `ujson`to resolve security vulnerabilities.
 

## \[3.15.19\] - 2026-04-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31519---2026-04-02 'Direct link to [3.15.19] - 2026-04-02')

Rasa Pro 3.15.19 (2026-04-02)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-2 'Direct link to Improvements')

- Declared `safetensors` and `keras` as optional NLU extras (`pip install 'rasa-pro[nlu]'` or `full`), so minimal installs no longer pull them in directly. `regex` is also declared optional and listed under those extras for consistency with `WhitespaceTokenizer`, but it remains installed on most minimal installs because the base dependency `tiktoken` (used by the LLM stack) depends on `regex`.
 
 The affected code paths lazy-import `safetensors` and `regex`; if either package is not installed, they raise `MissingDependencyException` with install instructions. `keras` is only pulled in for the TensorFlow model stack (`rasa.utils.tensorflow.models`), so it is optional for the same minimal-install story as `safetensors`.
 
- Improved sub-agent call steps that use `exit_if`: when the same ReAct agent runs again in one conversation, slots named in `exit_if` are cleared so a new run does not reuse values from the last finished run. Ongoing multi-turn agent sessions (waiting for the next user message) are unchanged.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-22 'Direct link to Bugfixes')

- When an agent or MCP tool call returns a fatal error (e.g. internal server error), the user no longer receives two bot messages. Previously both the internal error message ("Sorry, I am having trouble with that...") and the flow-cancelled message ("Okay, stopping...") were shown. Now only the internal error message is shown; the cancel pattern is not pushed for system-initiated failures, so the flow is still marked ended for tracking but the confusing cancel utterance is omitted.
 
- Add rephrase endpoint validation by treating defaults-only misconfiguration as warning and user-defined rephrase misconfiguration as error.
 
- Improved OpenTelemetry histograms for MCP agent LLM usage: `mcp_agent_llm_prompt_token_usage` and `mcp_agent_llm_response_duration` now record a consistent, call-scoped set of attributes (`agent_name`, `execution_context`, and `protocol_type`), making metrics easier to aggregate.
 
- Upgrade `langsmith` dependency to 0.6.3 to address CVE-2026-25528.
 
- Fixed three bugs in `ConcurrentRedisLockStore` that caused an `IndexError: deque index out of range` crash in the PII anonymization and deletion cron jobs under multi-replica deployments.
 
 - `get_lock()` now returns `None` when no ticket keys exist in Redis (all tickets had expired), instead of returning an empty `ConcurrentTicketLock` object that caused downstream callers to crash.
 - `save_lock()` now skips the Redis write when the ticket has already expired (TTL ≤ 0), preventing a `ResponseError` from Redis.
 - `save_lock()` previously passed the absolute epoch expiry timestamp as the Redis `EX` TTL, resulting in ticket keys that effectively never expired (~55 years). The TTL is now correctly computed as a relative duration in seconds.
- Fixed correction reset when a user answers the active collect slot and corrects another slot in the same turn.
 
 `CorrectSlotsCommand.create_correction_frame` no longer chooses the reset step only from slots in the correction payload. It also considers the current collect-information step and picks the earlier collect step in flow order, so the stack resets to the question currently being asked instead of jumping ahead to a later slot’s collect step (which skipped validation for the new value on the active slot).
 
- When interruptions are ENABLED, user message should be queued for processing only if it passes interruption criteria.
 

## \[3.15.18\] - 2026-03-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31518---2026-03-16 'Direct link to [3.15.18] - 2026-03-16')

Rasa Pro 3.15.18 (2026-03-16)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-23 'Direct link to Bugfixes')

- Restarted task-oriented ReAct agents no longer exit immediately. When an agent is restarted, the previous run's user/assistant messages are now marked in the conversation so the model does not set slots from them; the system prompt instructs the agent to treat the current interaction as a fresh start for slot collection.
- When an A2A task has status `input_required`, Rasa now populates `AgentOutput.structured_results` from both `task.artifacts` and DataParts in `task.status.message.parts`. Previously only the text message was returned and structured data was omitted, so clients can now process artifact data when the agent is waiting for user input.
- Fix PII redaction of user or bot messages when slot values do not match the original message text. This can occur when the original messages are multi-line or TTS-style bot read-backs.
- Update setuptools to 80.10.2 to address vulnerability [https://osv.dev/vulnerability/DEBIAN-CVE-2026-23949](https://osv.dev/vulnerability/DEBIAN-CVE-2026-23949). Replaced randomname with duoname (no dependencies) library.
- Fixed the backend tracing wrapper (e.g., Jaeger or OTLP) around send\_message to correctly forward the output\_channel argument to the underlying implementation.

## \[3.15.17\] - 2026-03-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31517---2026-03-11 'Direct link to [3.15.17] - 2026-03-11')

Rasa Pro 3.15.17 (2026-03-11)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-1 'Direct link to Deprecations and Removals')

- Removed an explicit deprecation warning for the license varible `RASA_PRO_LICENSE` to provide clarity that we will continue to support both this and the newer `RASA_LICENSE` variable for the forseeable future.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-24 'Direct link to Bugfixes')

- Fixes MCP tool result handling to now process results that contain only `structuredContent` (and no or empty `content`). Previously, an empty `content` field caused the result to be skipped and slots were not set from `structuredContent`.
 
- Downgraded the A2A polling "waiting to poll again" log from ERROR to INFO so normal polling no longer triggers false alerts.
 
- SQL tracker store: `update()` now supports a content-only path when the timestamp-based delete removes no rows (e.g. anonymization: same timestamps, content changed). In that case, the store either updates existing event rows in place where content differs or performs a full replace if event counts differ, so anonymized content is persisted correctly in a single atomic transaction.
 
 Anonymization cron job: the privacy manager now uses `update(updated_tracker)` for anonymization instead of delete-then-save. SQL tracker store behavior is aligned with MongoDB, Redis and DynamoDB (overwrite semantics), and anonymization with SQL completes in one atomic operation without a separate delete/save sequence.
 
- Prevent `JsonPatchConflict` exceptions raised when tracker is split in sub-sessions during PII anonymization cron jobs. Replace usage of a utility function that was recreating tracker objects using sub-sessions with an approach retrieving list of events for each sub-session instead.
 

## \[3.15.16\] - 2026-03-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31516---2026-03-06 'Direct link to [3.15.16] - 2026-03-06')

Rasa Pro 3.15.16 (2026-03-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-25 'Direct link to Bugfixes')

- When a flow had Agent A → flow steps → Agent B and the user restarted Agent A while Agent B was already started (then interrupted), execution after the restarted Agent A completed would jump back to Agent B and skip the flow steps between A and B. Flow steps between agents are now executed correctly, and the previously interrupted Agent B is resumed instead of started from scratch.
- Channel specific silence\_timeout can now be set in credentials file. Fixes the type error. If silence\_timeout is not set the channel will use global value.

## \[3.15.15\] - 2026-03-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31515---2026-03-03 'Direct link to [3.15.15] - 2026-03-03')

Rasa Pro 3.15.15 (2026-03-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-26 'Direct link to Bugfixes')

- - **Kafka event broker background polling thread caused test hangs and didn't allow the process to exit:** The background poll thread is now a daemon thread, so the process can terminate when the broker is not closed explicitly (e.g. in tests or after an abrupt exit). The `close()` method uses a 5-second join timeout so it does not block forever if the poll thread is stuck (e.g. when the broker is unreachable).
 - **Background polling:** The poll loop now runs for the whole lifetime of the broker, even before the producer is created. When the producer is not ready, the loop sleeps briefly instead of exiting. This allows the IAM OAuth callback to be triggered when using SASL\_SSL with AWS IAM (MSK), so tokens can be refreshed as the library calls `poll()`.
 - **Connection keepalive (optional):** New broker options `socket_keepalive_enable`, `reconnect_backoff_ms`, `reconnect_backoff_max_ms`, and `topic_metadata_refresh_interval_ms` let you reduce idle connection drops. When `socket_keepalive_enable` is true, these are passed to the underlying producer.
- Fixes race conditions in PII cron jobs by requiring a LockStore for BackgroundPrivacyManager and acquiring a per-sender\_id lock while anonymization/deletion jobs read-modify-write trackers.
 
 Implemented measures to defend against JSON patch issues and ensure correct event order.
 
 - **Deletion cron job:** When reconstructing the tracker from retained events after deleting old sessions, the deletion job now catches `JsonPatchException` and `JsonPointerException` (e.g. from invalid or inconsistent dialogue stack updates). On failure, the tracker is left unchanged and an error is logged instead of overwriting or deleting data, so no data loss occurs. Users can resolve stack-related issues by appending a `Restarted` event to the tracker if needed.
 - **Anonymization cron job:** Events from already-anonymized, processed, and ineligible sessions are now sorted by timestamp before rebuilding the tracker, with original index used as a tie-breaker when timestamps are identical. This keeps replay order chronologically consistent and ensures dialogue stack updates and other order-dependent events are applied in the correct sequence.
- ReAct agents now filter conversation events to user and bot utterances before building assistant and user role messages, and the default context limit is 10 utterance messages. Previously, the last 20 events were taken first and then filtered, so fewer user and assistant messages were included in the context.
 

## \[3.15.14\] - 2026-02-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31514---2026-02-26 'Direct link to [3.15.14] - 2026-02-26')

Rasa Pro 3.15.14 (2026-02-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-27 'Direct link to Bugfixes')

- The MCP task agent now includes slot changes in the agent output when the state is `INPUT_REQUIRED` or when max iterations is reached. Previously, slots set via set\_slot tools were updated in memory but not forwarded in the output, so they were not persisted until exit conditions were met.

## \[3.15.13\] - 2026-02-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31513---2026-02-25 'Direct link to [3.15.13] - 2026-02-25')

Rasa Pro 3.15.13 (2026-02-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-28 'Direct link to Bugfixes')

- Upgraded `litellm` to 1.81.15 so that `strict: True` can be passed to Bedrock and tool calling has the same guarantees as with OpenAI. Upgraded `openai` to >=2.8.0 to satisfy litellm's dependency.
- Unquoted environment variable references for MCP and A2A OAuth credentials (`client_id`, `client_secret`) in `endpoints.yml` are now accepted. Previously, training failed unless these values were wrapped in double quotes (e.g. `"${MCP_CLIENT_SECRET}"`).
- Move MCP task agent exit condition evaluation to after `process_output`, so that `SlotSet` events added by custom `process_output` implementations are considered when checking `exit_if` conditions.

## \[3.15.12\] - 2026-02-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31512---2026-02-23 'Direct link to [3.15.12] - 2026-02-23')

Rasa Pro 3.15.12 (2026-02-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-29 'Direct link to Bugfixes')

- **E2E fixture resolution and conftest hierarchy:**
 
 Fixture resolution now uses a conftest-style hierarchy, with local overrides taking precedence over global fixtures. This is a change from the previous behavior where all fixtures were merged into a single list, which could lead to silent data loss if duplicate fixture names were used. Now, every test case has its own resolved set of fixtures, and duplicate fixture names are allowed when the intent is "override".
 
 Details:
 
 - **Conftest-style hierarchy:** Fixtures can be defined at root or folder level (e.g. `conftest.yml` / `conftest.yaml`); all e2e tests under that path see them.
 - **Single-file run parity:** Running one test file resolves fixtures the same way as when that file is run as part of the full suite (global/conftest fixtures are loaded for that file’s path).
 - **Runtime namespace:** Each test case has its own resolved set of fixtures; there is no shared mutable fixture state between tests. Duplicate fixture names in **different** files are allowed when the intent is “override” (no need to remove or rename for full-suite runs).
 - **Override semantics:** Local (file-level) fixtures override folder-level; folder-level overrides root. Within a single file, duplicate fixture names remain an error.
- Update cryptography dependency to 46.0.5 to address CVE-2026-26007. Update pillow dependency to 12.1.1 to address CVE-2026-25990. Update pyasn1 dependency to 0.6.2 to address CVE-2026-23490. Update python-multipart dependency to 0.0.22 to address CVE-2026-24486.
 

## \[3.15.11\] - 2026-02-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31511---2026-02-18 'Direct link to [3.15.11] - 2026-02-18')

Rasa Pro 3.15.11 (2026-02-18)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-30 'Direct link to Bugfixes')

- Upgrade `litellm` to 1.80.0.
- Remove config file content from endpoint read success logs to prevent sensitive data exposure.
- Update `protobuf` to v5.29.6 to address CVE-2026-0994. Update `wheel` to v0.46.3 to address CVE-2026-24049.

## \[3.15.10\] - 2026-02-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31510---2026-02-10 'Direct link to [3.15.10] - 2026-02-10')

Rasa Pro 3.15.10 (2026-02-10)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-3 'Direct link to Improvements')

- Flows can now link to `pattern_search` (e.g. to trigger RAG or branch on knowledge-based search).

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-31 'Direct link to Bugfixes')

- Added interruption handling for final transcripts. Moved interruption check to eliminate interruption handling delay.
- Fix Enterprise Search Policy triggering `utter_ask_rephrase` instead of `utter_no_relevant_answer_found` when the vector store search returns no matching documents. The `pattern_cannot_handle` now correctly receives the `cannot_handle_no_relevant_answer` as a `context.reason` in all cases where no relevant answer is found, not only when the optional relevancy check rejects the answer.

## \[3.15.9\] - 2026-02-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3159---2026-02-05 'Direct link to [3.15.9] - 2026-02-05')

Rasa Pro 3.15.9 (2026-02-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-32 'Direct link to Bugfixes')

- Upgrade Keras to 3.12.1 to address CVE-2026-0897.
- Fixed `language` parameter in RimeTTS configuration to be passed correctly to the Rime API. Temporarily changed RimeTTS to non-streaming input mode as a workaround for a Rime API bug that impacts conversation experience; audio is now generated from the complete response at once.

## \[3.15.8\] - 2026-01-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3158---2026-01-26 'Direct link to [3.15.8] - 2026-01-26')

Rasa Pro 3.15.8 (2026-01-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-33 'Direct link to Bugfixes')

- Update azure-core to 1.38.0 to address CVE-2026-21226.

## \[3.15.7\] - 2026-01-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3157---2026-01-21 'Direct link to [3.15.7] - 2026-01-21')

Rasa Pro 3.15.7 (2026-01-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-34 'Direct link to Bugfixes')

- Fixed e2e test coverage report to correctly track coverage per flow. Each flow now appears as a separate entry with accurate coverage percentages. Additionally, `call`/`link` steps and `collect` steps with prefilled slots are now properly marked as visited.
- Fix OAuth2AuthStrategy to use URL-encoded parameters with Basic Authentication to fetch access token

## \[3.15.6\] - 2026-01-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3156---2026-01-15 'Direct link to [3.15.6] - 2026-01-15')

Rasa Pro 3.15.6 (2026-01-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-35 'Direct link to Bugfixes')

- Fixed an issue where storing documents in FAISS individually could hit the token limit. Now, documents are stored in batches to avoid exceeding the token limit.
- Throw error when duplicate fixtures are found in the same file or across files.
- `SessionEnded` does not reset the tracker anymore.
- Updated `werkzeug`, `aiohttp`, `filelock`, `fonttools`, `marshmallow`, `urllib3` and `langchain-core`to resolve security vulnerabilities.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-3 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.15.5\] - 2025-12-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3155---2025-12-29 'Direct link to [3.15.5] - 2025-12-29')

Rasa Pro 3.15.5 (2025-12-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-36 'Direct link to Bugfixes')

- Include response button titles in conversation history in prompts of LLM based components.
- Fix OpenTelemetry exporter `Invalid type <class 'NoneType'> of value None` error when recording the request duration metric for requests made from Rasa server to the custom action server. This error occurred because the url parameter was not being passed to the method that records the request duration metric, resulting in a NoneType value.
- Fix DUT fails with AttributeError when running with custom CommandGenerator

## \[3.15.4\] - 2025-12-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3154---2025-12-19 'Direct link to [3.15.4] - 2025-12-19')

Rasa Pro 3.15.4 (2025-12-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-37 'Direct link to Bugfixes')

- Fixed token expiration validation failing on servers running in non-UTC timezones. Token expiration checks now use timezone-aware UTC datetimes consistently, preventing premature "access token expired" errors on systems in timezones like UTC+8.

## \[3.15.3\] - 2025-12-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3153---2025-12-11 'Direct link to [3.15.3] - 2025-12-11')

Rasa Pro 3.15.3 (2025-12-11)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-38 'Direct link to Bugfixes')

- Fix potential Tensor shape mismatch error in `TEDPolicy` and `DIETClassifier`.
- Update `mcp` version to `~1.23.0` to address security vulnerability CVE-2025-66416.
- Fix bug in `CommandPayloadReader` where regex matching did not account for list slots, leading to incorrect parsing of slot keys and values. Now, slot names and values are correctly extracted even if a list is provided.
- Previously, `DialogueStateTracker.has_coexistence_routing_slot` could incorrectly return `True` when the tracker was created without domain slots (i.e., using `AnySlotDict`), because `AnySlotDict` pretends all slots exist. Now, the property returns False in that case, so the routing slot is only considered present if it is actually defined in the domain.
- Fix LLM prompt template loading to raise an error instead of just printing a warning when a custom prompt file is missing or cannot be read.
- Fix `AttributeError: 'str' object has no attribute 'pop'` when using `KnowledgeAnswerCommand` with tracing enabled.
- Fix `UnboundLocalError` in `E2ETestRunner._handle_fail_diff` that caused e2e tests to crash when processing `slot_was_not_set` assertion failures.
- Fixed button payloads with nested parentheses (e.g., `/SetSlots(slot=value)`) not being parsed correctly in `rasa shell`. The payload was incorrectly stripped of its `/SetSlots` prefix, causing it to be treated as text instead of a slot-setting command.
- Add missing deprecation warning for legacy `RASA_PRO_LICENSE` environment variable. Update license validation error messages to reference the actual environment variable used.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-4 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.15.2\] - 2025-12-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3152---2025-12-04 'Direct link to [3.15.2] - 2025-12-04')

Rasa Pro 3.15.2 (2025-12-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-39 'Direct link to Bugfixes')

- Fix agentic prompt template to access slots directly.
- Disable LLM health check for Enterprise Search Policy when generative search is disabled (i.e. use\_generative\_llm is False).
- Fix deduplication of collect steps in cases where the same slot was called in different flows.
- Cancel active flow before triggering internal pattern error during a custom action failure.
- Fix bug with the missing `language_data` module by installing `langcodes` with the `data` extra dependency.

## \[3.15.1\] - 2025-12-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3151---2025-12-02 'Direct link to [3.15.1] - 2025-12-02')

Rasa Pro 3.15.1 (2025-12-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-40 'Direct link to Bugfixes')

- Raise validation error when duplicate slot definitions are found across domains.
- Raise validation error when a slot with an initial value set is collected by a flow collect step which sets `asks_before_filling` to `true` without having a corresponding collect utterance or custom action.
- Update `pip` to fix security vulnerability.
- Fix AgentToolSchema.\_ensure\_property\_types() correctly preserves structural keywords ($ref, anyOf, oneOf, etc.)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-5 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.15.0\] - 2025-11-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3150---2025-11-26 'Direct link to [3.15.0] - 2025-11-26')

Rasa Pro 3.15.0 (2025-11-26)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-1 'Direct link to Features')

- Added Langfuse integration for tracing LLM and embedding calls. All LLM-based components (Command Generators, Rephraser, Enterprise Search Policy, ReAct Sub Agent) and embedding operations are automatically traced when Langfuse is configured. Each trace includes, among other things, input/output, latency, token usage, cost, session ID, and component name.
 
 To get started, configure Langfuse by adding a `langfuse` entry to the `tracing` section in `endpoints.yml`.
 
 Example:
 
 ```
    tracing:  - type: langfuse    public_key: ${LANGFUSE_PUBLIC_KEY}    private_key: ${LANGFUSE_PRIVATE_KEY}    host: https://cloud.langfuse.com
    ```
 
 Custom components can override `get_llm_tracing_metadata()` to customize metadata.
 
- Added support for triggering clarification when multiple `StartFlow` commands are generated with no active flow. To enable this feature, set the `CLARIFY_ON_MULTIPLE_START_FLOWS` environment variable to `True`.
 
- Adds DTMF (Dual-Tone Multi-Frequency) input support for collect steps in flows. Voice channels can now configure DTMF collection with options for digit length, finish key, and audio input control. The feature gracefully handles non-voice channels by skipping DTMF configuration when call\_state is not initialized.
 
- Added new CLI option `-f, --e2e-failed-tests` to export failed e2e tests to a file that can be directly used to re-run only those tests. It accepts an optional filename or directory path, with automatic timestamping to prevent overwriting previous test runs.
 
- Added support for including current datetime information by default in LLM prompts. This feature enables LLM-based components to include date and time context in their prompts, helping the model understand temporal references and provide time-aware responses.
 
 **Components Updated:**
 
 - `CompactLLMCommandGenerator`
 - `SearchReadyLLMCommandGenerator`
 - `EnterpriseSearchPolicy`
 - `MCPOpenAgent` (with timezone support)
 - `MCPTaskAgent` (with timezone support)
 
 **Prompt Template Updates:** When `include_date_time` is enabled, prompts include a "Date & Time Context" section showing:
 
 - Current date (formatted as "DD Month, YYYY")
 - Current time (formatted as "HH:MM:SS" with timezone)
 - Current day of the week
 
 **E2E Testing Support:** For deterministic testing, you can mock the datetime using the `mocked_datetime` slot in e2e test fixtures. The mocked datetime value must be provided in ISO 8601 format (e.g., `"2024-01-15T10:30:00+00:00"`). The e2e test runner validates the format and raises a `ValidationError` if the value is invalid.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-4 'Direct link to Improvements')

- Updated `pattern_completed` to check wether the user wants to continue the conversation or not by adding a `collect` step to the pattern.
 
- The `OutputChannel` is now provided to policies through the graph inputs. This enables any policy to access the output channel and send messages directly to the user, which is particularly useful for sending intermediate or filler messages.
 
- Allow LLM responses to be streamed to the output channel for any generative responses. This currently only applies to Enterprise Search Policy and Rephraser. Output channels that support streaming should handle duplicate message prevention when a response has already been streamed. Voice output channels (Browser Audio, Genesys, Audiocodes Stream, and Jambonz Stream) have been updated to use the streaming tokens to prepare the Text-to-Speech audio stream instead of waiting for the complete response to be generated. These changes are backward compatible; any existing channel does NOT need any additional changes. However, it can add new methods to make use of the streaming responses whenever they are available.
 
- Enhanced the clarification pattern with a configurable limit on the number of flows in the clarification options. Added a new slot `max_clarification_options` to the `default_flows_for_patterns.yml` with an initial value of `3` which can be changed by overriding the `initial_value` of the slot `max_clarification_options` to customize the maximum number of flows displayed during clarification.
 
- Enhanced the clarification pattern to handle empty clarification options. Added `utter_clarification_no_options_rasa` response to `default_flows_for_patterns.yml`, triggered when `pattern_clarification` receives a `ClarifyCommand` with no options.
 
- Allow e2e test results CLI flag `-o`, `--e2e-results` to be specified with a custom path. If the flag is used without a path, the default path `tests/` is used.
 
- Enable generative responses dispatched by custom actions to be evaluated by `generative_response_is_grounded` and `generative_response_is_relevant` assertions in E2E testing. In order for the E2E testing framework to evaluate generative responses dispatched by custom actions, you must add the custom action dispatching the generative bot response to the `utter_source` key of the appopriate assertion in the E2E test case. For example, if you have a custom action `action_generate_summary` that generates a summary using a generative model, you can add the following assertion to your E2E test case:
 
 ```
    test_cases:  - test_case: Generate summary for user query    steps:      - user: |          Can you provide a summary of the latest news on climate change?        assertions:          - generative_response_is_grounded:              threshold: 0.8              utter_source: action_generate_summary              ground_truth: <insert_ground_truth_summary_here>          - generative_response_is_relevant:              threshold: 0.9              utter_source: action_generate_summary
    ```
 
- Allow `flow_started` and `pattern_clarification_contains` E2E testing assertions to define the operator i.e. `all`, `any`, for matching multiple flow\_id values. Previously, `flow_started` only supported a single flow\_id value, while `pattern_clarification_contains` defaulted to `all`. Introduce new format to specify the operator explicitly:
 
 ```
    test_cases:  - test_case: user is asked to clarify contact-related message    steps:      - user: contacts        assertions:          - pattern_clarification_contains:              operator: "any"              flow_ids:                - remove_contact                - list_contacts                - add_contact                - update_contact  - test_case: user removes a missing contact    steps:      - user: i want to remove a contact        assertions:          - flow_started:              operator: "any"              flow_ids:                - remove_contact                - update_contact
    ```
 
 The `operator` field can take values `all` or `any`, determining whether all specified flow\_ids must match or if any one of them is sufficient.
 
 The previous format for these two assertions remains unchanged for backward compatibility, however it has been deprecated and will be removed in a future major release.
 
- Updated the default model and voice of Cartesia TTS, using `sonic-3` and voice ID American English Katie.
 
- Move commit messages to top of the SSE
 
- Allow trigerring empty `Clarify Command` without flow options.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-41 'Direct link to Bugfixes')

- Run action\_session\_start if tracker ends with SessionEnded event.
- Fixed PostgreSQL `UniqueViolation` error when running an assistant with multiple Sanic workers.
- Fix Kafka producer creation failing when SASL mechanism is specified in lowercase. The SASL mechanism is now case-insensitive in the Kafka producer configuration.
- Fix issue where the validation of the assistant files continued even when the provided domain was invalid and was being loaded as empty. The training or validation command didn't exit because the final merged domain contained only the default implementations for patterns, slots and responses and therefore passed the check for being non-empty.
- Trigger pattern\_internal\_error in a CALM assistant when a custom action fails during execution.
- Create AWS Bedrock / Sagemaker client only if the LLM healthcheck environment variable is set. If the environment variable is not set, validate that required credentials are present.
- Update `langchain-core` version to `~0.3.80` to address security vulnerability CVE-2025-65106.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-6 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.25\] - 2026-06-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31425---2026-06-04 'Direct link to [3.14.25] - 2026-06-04')

Rasa Pro 3.14.25 (2026-06-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-42 'Direct link to Bugfixes')

- Fixed a spurious "Fatal error on SSL transport" traceback printed to stderr on Windows after `rasa test e2e` completes by explicitly closing litellm's shared httpx sessions before the event loop shuts down.
 
- Fixed a crash in rasa-tools where training an assistant with a sub-agent MCP server configured caused a `RuntimeError: Attempted to exit cancel scope in a different task than it was entered in`, crashing the stdio server and making the next tool call fail with `MCP error -32000: Connection closed`.
 
- Fix two bugs causing random crashes and resource leaks when running Rasa with --workers 2+: a pre-fork event loop left open during JWT setup, and litellm async HTTP clients not being closed on worker shutdown.
 
 Root cause — Sanic uses fork-based multi-worker mode (legacy=True). create\_app() was calling asyncio.new\_event\_loop() + set\_event\_loop() in the main process to satisfy sanic-jwt's Initialize(), then leaving the loop open. Forked workers inherited its kqueue/epoll file descriptors, conflicting with each worker's own loop (~50% crash at startup). Separately, close\_resources() never called litellm.close\_litellm\_async\_clients(), leaving cached async HTTP clients open when workers exit.
 
- An explicitly configured `api_key` in the `rasa` provider model config now takes precedence over the Rasa license, allowing custom endpoints to authenticate with their own credentials.
 
- Fixed graph schema validation on Python 3.13+ when components use PEP 604 union type hints (e.g. `Domain | None`), which previously raised a false `GraphSchemaValidationException`.
 
- Transient tracker store connection errors (e.g. Postgres restart) no longer flood logs with full stack traces on each retry attempt; the error message is preserved on one line.
 
- Fix `resume_flow()` marking the wrong `AgentStackFrame` as resumed when a flow contains multiple sub-agent frames, which prevented `run_agent()` from taking its resume branch.
 
- Fixed multiple Langfuse session IDs being created for a single Dialogue Understanding Test (DUT) run by using the original test-case sender ID as the session ID for all per-step LLM traces.
 
- Fixed MCP flow tool tracing metrics when `_execute_mcp_tool_call` mutates the shared `initial_events` list in place by capturing the pre-mutation event count for output slicing, and by detecting failures from `InternalErrorPatternFlowStackFrame` on the dialogue stack rather than from unchanged event list length.
 
- Updated `idna`, `pip` and `urllib3` to address security vulnerabilities.
 
- Update Cartesia to use the model `sonic-latest` from `sonic-english` which has been sunsetted. Also updated the API version to `2026-03-01` from `2024-06-10`
 
- Upgrading from 3.15.x to 3.16.x no longer causes every subsequent message to fail with `InvalidFlowStepIdException` when a tracker was mid-flow at upgrade time and a flow step ID was renamed or removed. The flow executor now detects stale step IDs, logs a structured warning, removes the stale frame, and continues processing — rather than propagating the exception as a `GraphComponentException`.
 

## \[3.14.24\] - 2026-06-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31424---2026-06-01 'Direct link to [3.14.24] - 2026-06-01')

Rasa Pro 3.14.24 (2026-06-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-43 'Direct link to Bugfixes')

- Fixed DIETClassifier training failures on GPU with TensorFlow 2.19 and Keras 3, where training could fail with an `InvalidArgumentError` about unsupported `SparseReshape` operations on XLA GPU JIT devices. `RasaModel` now inherits from `tensorflow.keras.Model` instead of standalone Keras 3's `Model`, so it uses the same Keras stack as the rest of Rasa's TensorFlow code and respects `TF_USE_LEGACY_KERAS=1`.

## \[3.14.23\] - 2026-05-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31423---2026-05-22 'Direct link to [3.14.23] - 2026-05-22')

Rasa Pro 3.14.23 (2026-05-22)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-44 'Direct link to Bugfixes')

- Fix an interrupted sub-agent not being properly resumed after the user answers "Yes" to `pattern_continue_interrupted`. This is fixed by introducing a transient `RESUMING` agent state that sits between `INTERRUPTED` and `WAITING_FOR_INPUT`: `resume_flow()` now sets the agent frame to `RESUMING` instead of directly to `WAITING_FOR_INPUT`; `run_agent()` detects this and dispatches to `_handle_resume_interrupted_agent`, which replays the agent's last message and transitions the frame back to `WAITING_FOR_INPUT`.
- Fixed a `KeyError` in `ContinueInterruptedPatternFlowStackFrame.from_dict` that occurred when loading tracker events stored before the multi-interrupt fields were introduced. This caused repeated tracker store retries that blocked the server event loop, leading to liveness probe failures and pod restarts.

## \[3.14.22\] - 2026-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31422---2026-05-15 'Direct link to [3.14.22] - 2026-05-15')

Rasa Pro 3.14.22 (2026-05-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-45 'Direct link to Bugfixes')

- Updated security-flagged dependencies:
 
 - `litellm`: `~1.81.15` → `1.84.0`
 - `langchain`: `~0.3.27` → `~1.2.16`
 - `langchain-community`: `~0.3.29` → `~0.4.1`
 - `langchain-core`: `~0.3.81` → `~1.3.2`
 - `langsmith`: `~0.6.3` → `~0.7.31`
 - `qdrant-client`: `~1.9.1` → `~1.17.1`
 - `langchain-qdrant`: transitive → `~1.1.0` (new direct pin)
 - `langchain-openai`: transitive → `~1.2.1` (new direct pin)
 - `langchain-text-splitters`: transitive → `~1.1.2` (new direct pin)
- Fixed a bug where sub-agent flows would end immediately after interruption resumption instead of re-invoking the agent.
 
 When a flow containing a sub-agent call was interrupted (e.g., by a user digression) and then resumed via `pattern_continue_interrupted`, the agent frame's state remained `INTERRUPTED`. This caused the flow executor to skip the agent call and advance to END, triggering `pattern_completed` prematurely.
 
 The agent frame state is now correctly reset to `WAITING_FOR_INPUT` during flow resumption, ensuring the sub-agent is re-invoked with full conversation history as expected.
 
- Fixed MCP tracing attribute extraction for router-based model groups so tracing no longer crashes with `KeyError('provider')` when instrumenting ReAct sub-agent `send_message` calls.
 

## \[3.14.21\] - 2026-04-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31421---2026-04-24 'Direct link to [3.14.21] - 2026-04-24')

Rasa Pro 3.14.21 (2026-04-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-46 'Direct link to Bugfixes')

- Fixed a bug where the validator was mutating the allowed values list of categorical slots in-place, causing `None` entries to accumulate in slots used across multiple flows via subflows. This corrupted the flow retrieval FAISS embeddings during training.
- Fixed multi-agent flows looping back to an agent call when the top stack frame was **interrupted** instead of waiting for input, which could leave multiple agents active and fail graph execution. The loop-back shortcut now applies only when the agent frame is waiting for user input.
- Fixed validation incorrectly rejecting custom ContextualResponseRephraser subclasses configured as nlg.type in endpoints.yml. The validator now resolves the class and checks inheritance instead of only accepting the literal string "rephrase".
- Updated `PyJWT`, `werkzeug`, `aiohttp`, `orjson`, `pyasn1`, `requests`, and `ujson`to resolve security vulnerabilities.
- Fixed tracing span attributes for LLM components that use model groups. Spans now carry the correct model group ID and, for router groups, the representative model's attributes (preferring OpenAI when present for prompt token counting). The `llm_is_router_group` attribute is also set on spans when a router group is active.

## \[3.14.20\] - 2026-04-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31420---2026-04-02 'Direct link to [3.14.20] - 2026-04-02')

Rasa Pro 3.14.20 (2026-04-02)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-5 'Direct link to Improvements')

- Declared `safetensors` and `keras` as optional NLU extras (`pip install 'rasa-pro[nlu]'` or `full`), so minimal installs no longer pull them in directly. `regex` is also declared optional and listed under those extras for consistency with `WhitespaceTokenizer`, but it remains installed on most minimal installs because the base dependency `tiktoken` (used by the LLM stack) depends on `regex`.
 
 The affected code paths lazy-import `safetensors` and `regex`; if either package is not installed, they raise `MissingDependencyException` with install instructions. `keras` is only pulled in for the TensorFlow model stack (`rasa.utils.tensorflow.models`), so it is optional for the same minimal-install story as `safetensors`.
 
- Improved sub-agent call steps that use `exit_if`: when the same ReAct agent runs again in one conversation, slots named in `exit_if` are cleared so a new run does not reuse values from the last finished run. Ongoing multi-turn agent sessions (waiting for the next user message) are unchanged.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-47 'Direct link to Bugfixes')

- When an agent or MCP tool call returns a fatal error (e.g. internal server error), the user no longer receives two bot messages. Previously both the internal error message ("Sorry, I am having trouble with that...") and the flow-cancelled message ("Okay, stopping...") were shown. Now only the internal error message is shown; the cancel pattern is not pushed for system-initiated failures, so the flow is still marked ended for tracking but the confusing cancel utterance is omitted.
 
- Add rephrase endpoint validation by treating defaults-only misconfiguration as warning and user-defined rephrase misconfiguration as error.
 
- Upgrade `langsmith` dependency to 0.6.3 to address CVE-2026-25528.
 
- Fixed three bugs in `ConcurrentRedisLockStore` that caused an `IndexError: deque index out of range` crash in the PII anonymization and deletion cron jobs under multi-replica deployments.
 
 - `get_lock()` now returns `None` when no ticket keys exist in Redis (all tickets had expired), instead of returning an empty `ConcurrentTicketLock` object that caused downstream callers to crash.
 - `save_lock()` now skips the Redis write when the ticket has already expired (TTL ≤ 0), preventing a `ResponseError` from Redis.
 - `save_lock()` previously passed the absolute epoch expiry timestamp as the Redis `EX` TTL, resulting in ticket keys that effectively never expired (~55 years). The TTL is now correctly computed as a relative duration in seconds.
- Fixed correction reset when a user answers the active collect slot and corrects another slot in the same turn.
 
 `CorrectSlotsCommand.create_correction_frame` no longer chooses the reset step only from slots in the correction payload. It also considers the current collect-information step and picks the earlier collect step in flow order, so the stack resets to the question currently being asked instead of jumping ahead to a later slot’s collect step (which skipped validation for the new value on the active slot).
 

## \[3.14.19\] - 2026-03-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31419---2026-03-16 'Direct link to [3.14.19] - 2026-03-16')

Rasa Pro 3.14.19 (2026-03-16)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-48 'Direct link to Bugfixes')

- Restarted task-oriented ReAct agents no longer exit immediately. When an agent is restarted, the previous run's user/assistant messages are now marked in the conversation so the model does not set slots from them; the system prompt instructs the agent to treat the current interaction as a fresh start for slot collection.
- When an A2A task has status `input_required`, Rasa now populates `AgentOutput.structured_results` from both `task.artifacts` and DataParts in `task.status.message.parts`. Previously only the text message was returned and structured data was omitted, so clients can now process artifact data when the agent is waiting for user input.
- Fix PII redaction of user or bot messages when slot values do not match the original message text. This can occur when the original messages are multi-line or TTS-style bot read-backs.
- Update setuptools to 80.10.2 to address vulnerability [https://osv.dev/vulnerability/DEBIAN-CVE-2026-23949](https://osv.dev/vulnerability/DEBIAN-CVE-2026-23949). Replaced randomname with duoname (no dependencies) library.

## \[3.14.18\] - 2026-03-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31418---2026-03-11 'Direct link to [3.14.18] - 2026-03-11')

Rasa Pro 3.14.18 (2026-03-11)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-2 'Direct link to Deprecations and Removals')

- Removed an explicit deprecation warning for the license varible `RASA_PRO_LICENSE` to provide clarity that we will continue to support both this and the newer `RASA_LICENSE` variable for the forseeable future.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-49 'Direct link to Bugfixes')

- When a flow had Agent A → flow steps → Agent B and the user restarted Agent A while Agent B was already started (then interrupted), execution after the restarted Agent A completed would jump back to Agent B and skip the flow steps between A and B. Flow steps between agents are now executed correctly, and the previously interrupted Agent B is resumed instead of started from scratch.
 
- Fixes MCP tool result handling to now process results that contain only `structuredContent` (and no or empty `content`). Previously, an empty `content` field caused the result to be skipped and slots were not set from `structuredContent`.
 
- Downgraded the A2A polling "waiting to poll again" log from ERROR to INFO so normal polling no longer triggers false alerts.
 
- SQL tracker store: `update()` now supports a content-only path when the timestamp-based delete removes no rows (e.g. anonymization: same timestamps, content changed). In that case, the store either updates existing event rows in place where content differs or performs a full replace if event counts differ, so anonymized content is persisted correctly in a single atomic transaction.
 
 Anonymization cron job: the privacy manager now uses `update(updated_tracker)` for anonymization instead of delete-then-save. SQL tracker store behavior is aligned with MongoDB, Redis and DynamoDB (overwrite semantics), and anonymization with SQL completes in one atomic operation without a separate delete/save sequence.
 
- Prevent `JsonPatchConflict` exceptions raised when tracker is split in sub-sessions during PII anonymization cron jobs. Replace usage of a utility function that was recreating tracker objects using sub-sessions with an approach retrieving list of events for each sub-session instead.
 

## \[3.14.17\] - 2026-03-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31417---2026-03-03 'Direct link to [3.14.17] - 2026-03-03')

Rasa Pro 3.14.17 (2026-03-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-50 'Direct link to Bugfixes')

- - **Kafka event broker background polling thread caused test hangs and didn't allow the process to exit:** The background poll thread is now a daemon thread, so the process can terminate when the broker is not closed explicitly (e.g. in tests or after an abrupt exit). The `close()` method uses a 5-second join timeout so it does not block forever if the poll thread is stuck (e.g. when the broker is unreachable).
 - **Background polling:** The poll loop now runs for the whole lifetime of the broker, even before the producer is created. When the producer is not ready, the loop sleeps briefly instead of exiting. This allows the IAM OAuth callback to be triggered when using SASL\_SSL with AWS IAM (MSK), so tokens can be refreshed as the library calls `poll()`.
 - **Connection keepalive (optional):** New broker options `socket_keepalive_enable`, `reconnect_backoff_ms`, `reconnect_backoff_max_ms`, and `topic_metadata_refresh_interval_ms` let you reduce idle connection drops. When `socket_keepalive_enable` is true, these are passed to the underlying producer.
- Fixes race conditions in PII cron jobs by requiring a LockStore for BackgroundPrivacyManager and acquiring a per-sender\_id lock while anonymization/deletion jobs read-modify-write trackers.
 
 Implemented measures to defend against JSON patch issues and ensure correct event order.
 
 - **Deletion cron job:** When reconstructing the tracker from retained events after deleting old sessions, the deletion job now catches `JsonPatchException` and `JsonPointerException` (e.g. from invalid or inconsistent dialogue stack updates). On failure, the tracker is left unchanged and an error is logged instead of overwriting or deleting data, so no data loss occurs. Users can resolve stack-related issues by appending a `Restarted` event to the tracker if needed.
 - **Anonymization cron job:** Events from already-anonymized, processed, and ineligible sessions are now sorted by timestamp before rebuilding the tracker, with original index used as a tie-breaker when timestamps are identical. This keeps replay order chronologically consistent and ensures dialogue stack updates and other order-dependent events are applied in the correct sequence.
- ReAct agents now filter conversation events to user and bot utterances before building assistant and user role messages, and the default context limit is 10 utterance messages. Previously, the last 20 events were taken first and then filtered, so fewer user and assistant messages were included in the context.
 

## \[3.14.16\] - 2026-02-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31416---2026-02-26 'Direct link to [3.14.16] - 2026-02-26')

Rasa Pro 3.14.16 (2026-02-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-51 'Direct link to Bugfixes')

- Upgraded `litellm` to 1.81.15 so that `strict: True` can be passed to Bedrock and tool calling has the same guarantees as with OpenAI. Upgraded `openai` to >=2.8.0 to satisfy litellm's dependency.
- Unquoted environment variable references for MCP and A2A OAuth credentials (`client_id`, `client_secret`) in `endpoints.yml` are now accepted. Previously, training failed unless these values were wrapped in double quotes (e.g. `"${MCP_CLIENT_SECRET}"`).
- Move MCP task agent exit condition evaluation to after `process_output`, so that `SlotSet` events added by custom `process_output` implementations are considered when checking `exit_if` conditions.
- The MCP task agent now includes slot changes in the agent output when the state is `INPUT_REQUIRED` or when max iterations is reached. Previously, slots set via set\_slot tools were updated in memory but not forwarded in the output, so they were not persisted until exit conditions were met.

## \[3.14.15\] - 2026-02-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31415---2026-02-23 'Direct link to [3.14.15] - 2026-02-23')

Rasa Pro 3.14.15 (2026-02-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-52 'Direct link to Bugfixes')

- Remove config file content from endpoint read success logs to prevent sensitive data exposure.
 
- Update `protobuf` to v5.29.6 to address CVE-2026-0994. Update `wheel` to v0.46.3 to address CVE-2026-24049.
 
- **E2E fixture resolution and conftest hierarchy:**
 
 Fixture resolution now uses a conftest-style hierarchy, with local overrides taking precedence over global fixtures. This is a change from the previous behavior where all fixtures were merged into a single list, which could lead to silent data loss if duplicate fixture names were used. Now, every test case has its own resolved set of fixtures, and duplicate fixture names are allowed when the intent is "override".
 
 Details:
 
 - **Conftest-style hierarchy:** Fixtures can be defined at root or folder level (e.g. `conftest.yml` / `conftest.yaml`); all e2e tests under that path see them.
 - **Single-file run parity:** Running one test file resolves fixtures the same way as when that file is run as part of the full suite (global/conftest fixtures are loaded for that file’s path).
 - **Runtime namespace:** Each test case has its own resolved set of fixtures; there is no shared mutable fixture state between tests. Duplicate fixture names in **different** files are allowed when the intent is “override” (no need to remove or rename for full-suite runs).
 - **Override semantics:** Local (file-level) fixtures override folder-level; folder-level overrides root. Within a single file, duplicate fixture names remain an error.

## \[3.14.14\] - 2026-02-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31414---2026-02-12 'Direct link to [3.14.14] - 2026-02-12')

Rasa Pro 3.14.14 (2026-02-12)

- Upgrade `litellm` to 1.80.0.
- Fix Enterprise Search Policy triggering `utter_ask_rephrase` instead of `utter_no_relevant_answer_found` when the vector store search returns no matching documents. The `pattern_cannot_handle` now correctly receives the `cannot_handle_no_relevant_answer` as a `context.reason` in all cases where no relevant answer is found, not only when the optional relevancy check rejects the answer.

## \[3.14.13\] - 2026-02-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31413---2026-02-05 'Direct link to [3.14.13] - 2026-02-05')

Rasa Pro 3.14.13 (2026-02-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-53 'Direct link to Bugfixes')

- Upgrade Keras to 3.12.1 to address CVE-2026-0897.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-7 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.12\] - 2026-01-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31412---2026-01-26 'Direct link to [3.14.12] - 2026-01-26')

Rasa Pro 3.14.12 (2026-01-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-54 'Direct link to Bugfixes')

- Update azure-core to 1.38.0 to address CVE-2026-21226.

## \[3.14.11\] - 2026-01-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31411---2026-01-21 'Direct link to [3.14.11] - 2026-01-21')

Rasa Pro 3.14.11 (2026-01-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-55 'Direct link to Bugfixes')

- Fixed e2e test coverage report to correctly track coverage per flow. Each flow now appears as a separate entry with accurate coverage percentages. Additionally, `call`/`link` steps and `collect` steps with prefilled slots are now properly marked as visited.
- Throw error when duplicate fixtures are found in the same file or across files.
- Fix OAuth2AuthStrategy to use URL-encoded parameters with Basic Authentication to fetch access token

## \[3.14.10\] - 2026-01-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31410---2026-01-15 'Direct link to [3.14.10] - 2026-01-15')

Rasa Pro 3.14.10 (2026-01-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-56 'Direct link to Bugfixes')

- Fixed an issue where storing documents in FAISS individually could hit the token limit. Now, documents are stored in batches to avoid exceeding the token limit.
- `SessionEnded` does not reset the tracker anymore.
- Updated `werkzeug`, `aiohttp`, `filelock`, `fonttools`, `marshmallow`, `urllib3` and `langchain-core`to resolve security vulnerabilities.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-8 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.9\] - 2025-12-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3149---2025-12-29 'Direct link to [3.14.9] - 2025-12-29')

Rasa Pro 3.14.9 (2025-12-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-57 'Direct link to Bugfixes')

- Include response button titles in conversation history in prompts of LLM based components.
- Fix OpenTelemetry exporter `Invalid type <class 'NoneType'> of value None` error when recording the request duration metric for requests made from Rasa server to the custom action server. This error occurred because the url parameter was not being passed to the method that records the request duration metric, resulting in a NoneType value.
- Fix DUT fails with AttributeError when running with custom CommandGenerator

## \[3.14.8\] - 2025-12-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3148---2025-12-19 'Direct link to [3.14.8] - 2025-12-19')

Rasa Pro 3.14.8 (2025-12-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-58 'Direct link to Bugfixes')

- Fixed token expiration validation failing on servers running in non-UTC timezones. Token expiration checks now use timezone-aware UTC datetimes consistently, preventing premature "access token expired" errors on systems in timezones like UTC+8.

## \[3.14.7\] - 2025-12-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3147---2025-12-11 'Direct link to [3.14.7] - 2025-12-11')

Rasa Pro 3.14.7 (2025-12-11)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-59 'Direct link to Bugfixes')

- Fix potential Tensor shape mismatch error in `TEDPolicy` and `DIETClassifier`.
- Update `mcp` version to `~1.23.0` to address security vulnerability CVE-2025-66416.
- Fix bug in `CommandPayloadReader` where regex matching did not account for list slots, leading to incorrect parsing of slot keys and values. Now, slot names and values are correctly extracted even if a list is provided.
- Previously, `DialogueStateTracker.has_coexistence_routing_slot` could incorrectly return `True` when the tracker was created without domain slots (i.e., using `AnySlotDict`), because `AnySlotDict` pretends all slots exist. Now, the property returns False in that case, so the routing slot is only considered present if it is actually defined in the domain.
- Fix LLM prompt template loading to raise an error instead of just printing a warning when a custom prompt file is missing or cannot be read.
- Fix `AttributeError: 'str' object has no attribute 'pop'` when using `KnowledgeAnswerCommand` with tracing enabled.
- Fix `UnboundLocalError` in `E2ETestRunner._handle_fail_diff` that caused e2e tests to crash when processing `slot_was_not_set` assertion failures.
- Fixed button payloads with nested parentheses (e.g., `/SetSlots(slot=value)`) not being parsed correctly in `rasa shell`. The payload was incorrectly stripped of its `/SetSlots` prefix, causing it to be treated as text instead of a slot-setting command.
- Add missing deprecation warning for legacy `RASA_PRO_LICENSE` environment variable. Update license validation error messages to reference the actual environment variable used.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-9 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.6\] - 2025-12-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3146---2025-12-05 'Direct link to [3.14.6] - 2025-12-05')

Rasa Pro 3.14.6 (2025-12-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-60 'Direct link to Bugfixes')

- Fix agentic prompt template to access slots directly.
- Disable LLM health check for Enterprise Search Policy when generative search is disabled (i.e. use\_generative\_llm is False).
- Fix deduplication of collect steps in cases where the same slot was called in different flows.
- Cancel active flow before triggering internal pattern error during a custom action failure.
- Fix bug with the missing `language_data` module by installing `langcodes` with the `data` extra dependency.

## \[3.14.5\] - 2025-12-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3145---2025-12-02 'Direct link to [3.14.5] - 2025-12-02')

Rasa Pro 3.14.5 (2025-12-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-61 'Direct link to Bugfixes')

- Update `pip` to fix security vulnerability.
- Fix AgentToolSchema.\_ensure\_property\_types() correctly preserves structural keywords ($ref, anyOf, oneOf, etc.)

## \[3.14.4\] - 2025-11-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3144---2025-11-27 'Direct link to [3.14.4] - 2025-11-27')

Rasa Pro 3.14.4 (2025-11-27)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-62 'Direct link to Bugfixes')

- Fixed `patten-continue-interrupted` running out of order before linked flows.
- Fix `rasa studio upload` timeouts by enabling TCP keep-alive with platform-specific socket options to maintain stable connections.
- Remove `action_metadata` tracing span attribute from `EnterpriseSearchPolicy` instrumentation to prevent PII leakages. Add new environment variable `RASA_TRACING_DEBUGGING_ENABLED` to enable adding `action_metadata` to `EnterpriseSearchPolicy` spans for debugging purposes. By default, this variable is set to `false` to ensure PII is not logged in production environments.
- Fixed PostgreSQL `UniqueViolation` error when running an assistant with multiple Sanic workers.
- Fix Kafka producer creation failing when SASL mechanism is specified in lowercase. The SASL mechanism is now case-insensitive in the Kafka producer configuration.
- Fix issue where the validation of the assistant files continued even when the provided domain was invalid and was being loaded as empty. The training or validation command didn't exit because the final merged domain contained only the default implementations for patterns, slots and responses and therefore passed the check for being non-empty.
- Trigger pattern\_internal\_error in a CALM assistant when a custom action fails during execution.
- Create AWS Bedrock / Sagemaker client only if the LLM healthcheck environment variable is set. If the environment variable is not set, validate that required credentials are present.
- Update `langchain-core` version to `~0.3.80` to address security vulnerability CVE-2025-65106.
- Raise validation error when duplicate slot definitions are found across domains.
- Raise validation error when a slot with an initial value set is collected by a flow collect step which sets `asks_before_filling` to `true` without having a corresponding collect utterance or custom action.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-10 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.3\] - 2025-11-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3143---2025-11-13 'Direct link to [3.14.3] - 2025-11-13')

Rasa Pro 3.14.3 (2025-11-13)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-11 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.2\] - 2025-10-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3142---2025-10-30 'Direct link to [3.14.2] - 2025-10-30')

Rasa Pro 3.14.2 (2025-10-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-6 'Direct link to Improvements')

- Add new environment variable `LOG_LEVEL_PYMONGO` to control the logging level of PyMongo dependency of Rasa. This can be useful to reduce the verbosity of logs. Default value is `INFO`.
 
 The logging level of PyMongo can also be set via the `LOG_LEVEL_LIBRARIES` environment variable, which provides the default logging level for a selection of third-party libraries used by Rasa. If both variables are set, `LOG_LEVEL_PYMONGO` takes precedence.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-63 'Direct link to Bugfixes')

- Clean up duplicated `ChitChatAnswerCommands` in command processor.
 
 Ensure that one `CannotHandleCommand` remains if only multiple `CannotHandleCommands` were present.
 
- LLM request timeouts are now enforced at the event loop level using `asyncio.wait_for`.
 
 Previously, timeout values configured in `endpoints.yml` could be overridden by the HTTP client's internal timeout behavior. This fix ensures that when a specific timeout value is configured for LLM requests, the request respects that exact timing regardless of the underlying HTTP client implementation.
 
- Fixed an issue where duplicate collect steps in a flow could cause rendering problems, such as exceeding token limits. This occurred when a flow called another flow multiple times. Now, each collect step is listed only once when retrieving all collect steps for a flow.
 
- If a FloatSlot does not have min and max values set in the domain, the slot will not be validated against any range. If the slot defines an initial value, as well as min and max values, the initial value will be validated against the range and an error will be raised if the initial value is out of range. Enhance run-time validation for new values assigned to FloatSlot instances, ensuring they fall within the defined min and max range if these are set. If min and max are not set, no range validation is performed.
 
- Fixed bug preventing `deployment` parameter from being used in generative response LLM judge configuration.
 
- Fixed bug where `persisted_slots` defined in called flows were incorrectly reset when the parent flow ended, unless they were also explicitly defined in the parent flow. Persisted slots now only need to be defined in the called flow to remain persisted after the parent flow ends.
 
- Fixes the prompt template resolution logic in the CompactLLMCommandGenerator and SearchReadyLLMCommandGenerator classes.
 
- When `action_clean_stack` is used in a user flow, allow `pattern_completed` to be triggered even if there are pattern flows at the bottom of the dialogue stack below the top user flow frame. Previously, `pattern_completed` would only trigger if there were no other frames below the top user flow frame. This assumes that the `action_clean_stack` is the last action called in the user flow.
 
- Update `pip` version used in Dockerfile to `23.*` to address security vulnerability CVE-2023-5752. Update `python-socketio` version to `5.14.0` to address security vulnerability CVE-2025-61765.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-12 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.1\] - 2025-10-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3141---2025-10-10 'Direct link to [3.14.1] - 2025-10-10')

Rasa Pro 3.14.1 (2025-10-10)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-13 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.14.0\] - 2025-10-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3140---2025-10-09 'Direct link to [3.14.0] - 2025-10-09')

Rasa Pro 3.14.0 (2025-10-09)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-2 'Direct link to Features')

- Added the possibility to call an agent from within a flow via a `call` step. We support two types of ReAct style agents:
 
 1. An open ended exploratory agent - Here the user’s goal is loosely defined, for example researching about stocks to invest in. There is no pre-defined criteria which can be used to know when the goal is reached. Hence the agent itself needs to figure out when it has helped the user to the best of its capabilities.
 
 ```
        flows:  stock_investment_research:    description: helps research and analyze stock investment options    steps:      - call: stock_explorer  # runs until agent signals completion
        ```
 
 2. A task specific agent - Here we can come up with conditions, which when true, signals the completion of a user goal. For example, helping the user find an appointment slot based on their preferences - you know the agent can exit once all user’s preferences have been met and they have agreed to a slot.
 
 ```
        flows:  book_appointment:    description: helps users book appointments    steps:      - call: ticket_agent        exit_if:          - slots.chosen_appointment is not null          - slots.booking_completed is True**      - collect: user_confirmation
        ```
 
 
 To configure the agents you need to create a folder structure like this:
 
 ```
    your_project/├── config.yml├── domain/├── data/flows/└── sub_agents/    └── stock_explorer/        ├── config.yml  # mandatory        └── prompt_template.jinja2  # optional
    ```
 
 The `config.yml` file for each agent is mandatory and should look, for example, like this:
 
 ```
    # Basic agent informationagent:  name: stock_explorer # name of the agent  protocol: RASA # protocol for its connections either RASA or A2A  description: "Agent that helps users research and analyze stock options" # a brief description used by the command generator to continue / stop the agent and the agent during its execution# Core configurationconfiguration:  llm:  # (optional) Same format as other Rasa LLM configs    type: openai    model: gpt-4  prompt_template: sub_agents/stock_explorer/prompt_template.jinja2  # (optional) prompt template file containing a customized prompt  timeout: 30  # (optional) seconds before timing out  max_retries: 3  # (optional) connection retry attempts# MCP server connectionsconnections:  mcp_servers:    - name: trade_server      include_tools:  # optional: specify which tools to include        - find_symbol        - get_company_news        - apply_technical_analysis        - fetch_live_price      exclude_tools:  # optional: tools to exclude; helpful to not allow the agent to execute critical tools.        - place_buy_order        - view_positions
    ```
 
- Added the possibility to call an MCP tool directly from a flow step via a `call` step.
 
 Example:
 
 ```
      - call: place_buy_order  # MCP tool name    mcp_server: trade_server # MCP server where tool is available    mapping:      input:      - param: ticker_symbol    # tool parameter name        slot: stock_name     # slot to send as value      - param: quantity        slot: order_quantity      output:      - slot: order_status    # slot to store results        result_key: result.structuredContent.order_status.success
    ```
 
 The MCP server needs to be defined in the `endpoints.yml` file
 
 ```
    mcp_servers:  - name: trade_server    url: http://localhost:8080    type: http
    ```
 
- We updated the inspector
 
 - to highlight when a call step is calling an agent or an MCP tool using a little icon in the flow view.
 - and added an additional agent widget that shows the available agents and their corresponding status in the conversation.
- Whenever agents are configured, the `SearchReadyLLMCommandGenerator` and the `CompactLLMCommandGenerator` will use new default prompt templates that contain new agent specific commands (`RestartAgent` and `ContinueAgent`) and instructions.
 
- Adds Interruption Handling Support (beta) in Voice Stream Channels, namely Browser Audio, Twilio Media Streams and Jambonz Stream. Interruptions are identified from partial transcripts sent by the Automatic Speech Recogintion (ASR) Engine. Interruption Handling is disabled by defaulat and can be configured using the optional `interruptions` key in `credentials.yml`.
 
 ```
    browser_audio:  server_url: localhost  interruptions:    enabled: True    min_words: 3  asr:    name: "deepgram"  tts:    name: cartesia
    ```
 
 The configuration property `min_words` (default 3) requires the partial transcript to have at least 3 words to interrupt the bot. This is a rudimentary way to filter backchannels (brief responses like "hmm", "yeah", "ok").
 
 Interruptions can be disabled for certain responses using the domain response property `allow_interruptions` (default true). Example,
 
 ```
    responses:  utter_current_balance:    - text: You still have {current_balance} in your account.      allow_interruptions: false
    ```
 
- Added the following new events to capture the state of any agent used within a conversation:
 
 - `AgentStarted`
 - `AgentCompleted`
 - `AgentCancelled`
 - `AgentResumed`
 - `AgentInterrupted`
- Added support for Redis Cluster mode in the tracker store for horizontal scaling and cloud compatibility.
 
 Added support for Redis Sentinel mode in the tracker store for high availability with automatic failover.
 
 Added 3 new configuration parameters to `RedisTrackerStore`:
 
 - `deployment_mode`: String value specifying Redis deployment type ("standard", "cluster", or "sentinel"). Defaults to "standard".
 - `endpoints`: List of strings in "host:port" format defining cluster nodes or sentinel instances.
 - `sentinel_service`: String value specifying the sentinel service name to connect to. Only used in sentinel mode, defaults to "mymaster".
- Add support for IAM authentication to AWS RDS SQL tracker store. To enable this feature:
 
 - set the `IAM_CLOUD_PROVIDER` environment variable to `aws`.
 - set the `AWS_DEFAULT_REGION` environment variable to your desired AWS region.
 - if you want to enforce SSL verification, you can set the `SQL_TRACKER_STORE_SSL_MODE` environment variable (for example, to `verify-full` or `verify-ca`) and provide the path to the CA certificate using the `SQL_TRACKER_STORE_SSL_ROOT_CERTIFICATE` environment variable.
- Add support for IAM authentication to AWS Managed Streaming for Kafka event broker. This allows secure authentication to an AWS MSK cluster without the need for static credentials. To enable this feature:
 
 - set the `IAM_CLOUD_PROVIDER` environment variable to `aws`.
 - set the `AWS_DEFAULT_REGION` environment variable to your desired AWS region.
 - ensure that your application has the necessary IAM permissions to access the AWS Managed Service Kafka cluster.
 - ensure the topic is created on the MSK cluster.
 - use the `SASL_SSL` security protocol and `OAUTHBEARER` SASL mechanism in your Kafka configuration in `endpoints.yml`.
 - download the root certificate from Amazon Trust Services: [https://www.amazontrust.com/repository/](https://www.amazontrust.com/repository/) and provide its path to the `ssl_cafile` event broker yaml property.
- Added support for Redis Cluster mode in `RedisLockStore` for horizontal scaling and cloud compatibility.
 
 Added support for Redis Sentinel mode in `RedisLockStore` for high availability with automatic failover.
 
 Added 3 new configuration parameters to `RedisLockStore`:
 
 - `deployment_mode`: String value specifying Redis deployment type ("standard", "cluster", or "sentinel"). Defaults to "standard".
 - `endpoints`: List of strings in "host:port" format defining cluster nodes or sentinel instances.
 - `sentinel_service`: String value specifying the sentinel service name to connect to. Only used in sentinel mode, defaults to "mymaster".
- Added support for Redis Cluster mode in `ConcurrentRedisLockStore` for horizontal scaling and cloud compatibility.
 
 Added support for Redis Sentinel mode in `ConcurrentRedisLockStore` for high availability with automatic failover.
 
 Added 3 new configuration parameters to `ConcurrentRedisLockStore`:
 
 - `deployment_mode`: String value specifying Redis deployment type ("standard", "cluster", or "sentinel"). Defaults to "standard".
 - `endpoints`: List of strings in "host:port" format defining cluster nodes or sentinel instances.
 - `sentinel_service`: String value specifying the sentinel service name to connect to. Only used in sentinel mode, defaults to "mymaster".
- Add support for IAM authentication to AWS ElastiCache for Redis lock store. This allows secure authentication to an AWS ElastiCache without the need for static credentials. To enable this feature:
 
 - set the `IAM_CLOUD_PROVIDER` environment variable to `aws`.
 - set the `AWS_DEFAULT_REGION` environment variable to your desired AWS region.
 - download the root certificate from Amazon Trust Services and provide its path to the `ssl_ca_certs` lock store yaml property.
 - if enabling cluster mode, set the `AWS_ELASTICACHE_CLUSTER_NAME` environment variable to your ElastiCache cluster name.
 - ensure that your application has the necessary IAM permissions to access the AWS ElastiCache for Redis cluster.
- Default silence timeout is now configurable per channel. Default silence timeout is read from channel configuration in credentials file. Example:
 
 ```
    audiocodes_stream:    server_url: "humble-arguably-stork.ngrok-free.app"    asr:        name: deepgram    tts:        name: cartesia    silence_timeout: 5
    ```
 
 Silence timeout at collect step is also configurable per channel:
 
 ```
    flows:    - name: example_flow      steps:        - collect:            silence_timeout:                - audiocodes_stream: 5
    ```
 
 If not configured, default silence timeout is 7 seconds for each channel.
 
- Adds support for Deepgram TTS Adds `browser_audio` channel to the credentials.yml in `tutorial` template
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-7 'Direct link to Improvements')

- In case a `StartFlowCommand` for a flow, that is already on the stack, is predicted, we now resume the flow and interrupt the current active flow instead of ignoring the `StartFlowCommand`.
 
- Updated `pattern_continue_interrupted` to ask the user if they want to proceed with any of the interrupted flows before resuming that particular flow.
 
- We have isolated the dependencies that are only required for NLU components. These include:
 
 - `transformers` (version: `"~4.38.2"`)
 - `tensorflow` (version: `"^2.19.0`, only available for `"python_version < '3.12'`)
 - `tensorflow-text` (version `"^2.19.0"`, only available for `"python_version < '3.12'`)
 - `tensorflow-hub` (version: `"^0.13.0"`, only available for `"python_version < '3.12'`)
 - `tensorflow-io-gcs-filesystem` (version: `"==0.31"` for `sys_platform == 'win32'`, version: `"==0.34"` for `sys_platform == 'linux'`, version: `"==0.34"` for `sys_platform == 'linux'` for `"sys_platform == 'darwin' and platform_machine != 'arm64'`; all only available for `"python_version < '3.12'`)
 - `tensorflow-metal` (version: `"^1.2.0"`, only available for `"python_version < '3.12'`)
 - `tf-keras` (version: `"^2.15.0"`, only available for `"python_version < '3.12'`)
 - `spacy` (version: `"^3.5.4"`)
 - `sentencepiece` (version: `"~0.1.99"`, only available for `"python_version < '3.12'`)
 - `skops` (version: `"~0.13.0"`)
 - `mitie` (version: `"^0.7.36"`, added for consistency)
 - `jieba` (version: `">=0.42.1, <0.43"`)
 - `sklearn-crfsuite` (version: `"~0.5.0"`)
 
 The dependencies for those components are now in their own extra, called `nlu` and can be installed via: `poetry install --extras nlu`, `pip install 'rasa-pro[nlu]'` or `uv pip install 'rasa-pro[nlu]'`.
 
- We added dependencies for agent orchestration, `mcp` (`"~1.12.0"`) and `a2a-sdk` (`"~0.3.4"`), to the list of default dependencies. As a result, they get installed via: `poetry install`, `pip install 'rasa-pro'` or `uv pip install 'rasa-pro'`.
 
 To ensure compatibility between `a2a-sdk` and `tensorflow`, `tensorflow` and its related dependencies were updated. These include:
 
 - `tensorflow` (from `"2.14.1"` to `"^2.19.0"`)
 - `tensorflow-text` (from `"2.14.0"` to `^2.19.0`)
 - `tensorflow-metal` (from `"1.1.0"` to `"^1.2.0"`)
 - instead of `keras` (`"2.14.0"`), we now use `tf-keras` (`"^2.15.0"`)
 
 Additionally, tensorflow dependencies that are incompatible with the upgraded `tensorflow` version were removed. These include: `tensorflow-intel`, `tensorflow-cpu-aws`, `tensorflow-macos`.
 
 Because tensorflow and related packages were upgraded, tensorflow code was also updated. As a consequence, the incremental training feature is not supported anymore. This is because `tf-keras` (`"^2.15.0"`) does not allow a compiled model to be updated.
 
- We updated the supported Python versions:
 
 - Dropped support for Python 3.9.
 - Added support for Python 3.12 and Python 3.13. So, whilst we previously supported `python = ">=3.9.2,<3.12"`, we now support `python = ">=3.10.0,<3.14"`.
 
 To ensure compatibility with the newly supported Python versions, existing dependencies had to be updated. These include:
 
 - `protobuf` (from `"~4.25.8"` to `"~5.29.5"`)
 - `importlib-resources` (from `"6.1.3"` to `"^6.5.2"`)
 - `opentelemetry-sdk` (from `"~1.16.0"` to `"~1.33.0"`)
 - `opentelemetry-exporter-jaeger` (`"~1.16.0"`) - has been removed
 - `opentelemetry-exporter-otlp` (from `"~1.16.0"` to `"~1.33.0"`)
 - `opentelemetry-api` (from `"~1.16.0"` to `"~1.33.0"`)
 - `pep440-version-utils` - only available for `"python_version < '3.13'"`
 - `pymilvus` (from `">=2.4.1,<2.4.2"` to `"^2.6.1"`)
 - `numpy` (from `"~1.26.4"` to `"~2.1.3"`)
 - `scipy` (`"~1.13.1"` for `"python_version < '3.12'"`; `"~1.14.0"` for `"python_version >= '3.12'"`)
 - `tensorflow` (version: `"^2.19.0`, only available for `"python_version < '3.12'`)
 - `tensorflow-text` (version `"^2.19.0"`, only available for `"python_version < '3.12'`)
 - `tensorflow-hub` (version: `"^0.13.0"`, only available for `"python_version < '3.12'`)
 - `tensorflow-io-gcs-filesystem` (version: `"==0.31"` for `sys_platform == 'win32'`, version: `"==0.34"` for `sys_platform == 'linux'`, version: `"==0.34"` for `sys_platform == 'linux'` for `"sys_platform == 'darwin' and platform_machine != 'arm64'`; all only available for `"python_version < '3.12'`)
 - `tensorflow-metal` (version: `"^1.2.0"`, only available for `"python_version < '3.12'`)
 - `tf-keras` (version: `"^2.15.0"`, only available for `"python_version < '3.12'`)
 - `sentencepiece` (version: `"~0.1.99"`, only available for `"python_version < '3.12'`)
 
 Note that `tensorflow` and related dependencies are only supported for Python < 3.12. As a result, components requiring those dependencies are not available for Python >= 3.12. These are: `DIETClassifier` , `TEDPolicy` , `UnexpecTEDIntentPolicy` , `ResponseSelector` , `ConveRTFeaturizer` and `LanguageModelFeaturizer` .
 
- We have isolated the dependencies required for channel connectors. These include:
 
 - `fbmessenger` (version: `"~6.0.0"`)
 - `twilio` (version: `"~9.7.2"`)
 - `webexteamssdk` (version: `">=1.6.1,<1.7.0"`)
 - `mattermostwrapper` (version: `"~2.2"`)
 - `rocketchat_API` (version: `">=1.32.0,<1.33.0"`)
 - `aiogram` (version: `"~3.22.0"`)
 - `slack-sdk` (version: `"~3.36.0"`)
 - `cvg-python-sdk` (version: `"^0.5.1"`)
 
 The dependencies for those channels are now in their own extra, called `channels` and can be installed via: `poetry install --extras channels`, `pip install 'rasa-pro[channels]'` or `uv pip install 'rasa-pro[channels]'`. Note that the dependencies for the channels `browser_audio`, `studio_chat`, `socketIO` and `rest` are not included and remain in the main list of dependencies.
 
- Validation errors now raise meaningful exceptions instead of calling sys.exit(1), producing clearer and more actionable log messages while allowing custom error handling.
 
- Engine-related modules now raise structured exceptions instead of calling `sys.exit(1)` or `print_error_and_exit`, providing clearer, more actionable log messages and enabling custom error handling.
 
- Enable connection to AWS services for LLMs (e.g. Bedrock, Sagemaker) via multiple additional methods to environment variables, for example by using IAM roles or AWS credentials file.
 
- Relocate `latency display` in Inspector
 
- Implement Redis connection factory class to handle different Redis deployment modes. This class abstracts Redis connection creation and automatically selects the appropriate redis-py client (Redis, RedisCluster, or Sentinel) based on configuration.
 
 - In `standard` mode, connects to a single Redis instance.
 - In `cluster` mode, connects to a Redis Cluster using the provided endpoints.
 - In `sentinel` mode, connects to a Redis Sentinel setup for high availability.
- Metadata of `action_listen` event in the tracker contains the execution times for Command Processor and Prediction Loop for the previous turn. This information is present in the key `execution_times` in the metadata and the times are in milliseconds.
 
- Rasa Voice Inspector (browser\_audio channel) used to require a Rasa License with a specific voice scope. This scope requirement has been dropped.
 
- Add new environment variables for each AWS service integration (RDS, ElastiCache for Redis, MSK) that indicates whether to use IAM authentication when connecting to the service:
 
 - `KAFKA_MSK_AWS_IAM_ENABLED` - set to `true` to enable IAM authentication for MSK connections.
 - `RDS_SQL_DB_AWS_IAM_ENABLED` - set to `true` to enable IAM authentication for RDS connections.
 - `ELASTICACHE_REDIS_AWS_IAM_ENABLED` - set to `true` to enable IAM authentication for ElastiCache for Redis connections.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-64 'Direct link to Bugfixes')

- Pass all flows to `find_updated_flows` to avoid creating a `HandleCodeChangeCommand` in situations where flows were not updated.
- The .devcontainer docker-compose file now uses the new `docker.io/bitnamilegacy` repository for images, rather than the old `docker.io/bitnami`. This change was made in response to Bitnami's announcement that they will be putting all of their public Helm Chart container images behind a paywall starting August 28th, 2025. Existing public images are moved to the new legacy repository - `bitnamilegacy` - which is only intended for short-term migration purposes.
- In case an agents fails with an fatal error or returns an unkown state, we now cancel the current active flow next to triggering `pattern_internal_error`.
- Bugfix for Jambonz Stream channel Websocket URL path which resulted in failed Websocket Connections
- Fixed the contextual response rephraser to use and update the correct translated response text for the current language, instead of falling back to the default response.
- Refactored `validate_argument_paths` to accept list values and aggregate all missing paths before exiting.
- Fix expansion of referenced environment variables in `endpoints.yml` during Bedrock model config validation.
- Enabled the data argument to support both string and list inputs, normalizing to a list for consistent handling.
- Fixed model loading failures on Windows systems running recent Python versions (3.9.23+, 3.10.18+, 3.11.13+) due to tarfile security fix incompatibility with Windows long path prefix.
- Fixes the silence handling bug in Audiocodes Stream Channel where consecutive bot responses could trip silence timeout pattern while the bot is speaking. Audiocodes Stream channel now forecefully cancels the silence timeout watcher whenever it sends the bot response audio.
- Use correct formatting method for messages in `completion` and `acompletion` functions of `LiteLLMRouterLLMClient`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-14 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.24\] - 2026-02-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31324---2026-02-23 'Direct link to [3.13.24] - 2026-02-23')

Rasa Pro 3.13.24 (2026-02-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-65 'Direct link to Bugfixes')

- **E2E fixture resolution and conftest hierarchy:**
 
 Fixture resolution now uses a conftest-style hierarchy, with local overrides taking precedence over global fixtures. This is a change from the previous behavior where all fixtures were merged into a single list, which could lead to silent data loss if duplicate fixture names were used. Now, every test case has its own resolved set of fixtures, and duplicate fixture names are allowed when the intent is "override".
 
 Details:
 
 - **Conftest-style hierarchy:** Fixtures can be defined at root or folder level (e.g. `conftest.yml` / `conftest.yaml`); all e2e tests under that path see them.
 - **Single-file run parity:** Running one test file resolves fixtures the same way as when that file is run as part of the full suite (global/conftest fixtures are loaded for that file’s path).
 - **Runtime namespace:** Each test case has its own resolved set of fixtures; there is no shared mutable fixture state between tests. Duplicate fixture names in **different** files are allowed when the intent is “override” (no need to remove or rename for full-suite runs).
 - **Override semantics:** Local (file-level) fixtures override folder-level; folder-level overrides root. Within a single file, duplicate fixture names remain an error.

## \[3.13.23\] - 2026-02-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31323---2026-02-12 'Direct link to [3.13.23] - 2026-02-12')

Rasa Pro 3.13.23 (2026-02-12)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-8 'Direct link to Improvements')

- Flows can now link to `pattern_search` (e.g. to trigger RAG or branch on knowledge-based search).

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-66 'Direct link to Bugfixes')

- Fixed e2e test coverage report to correctly track coverage per flow. Each flow now appears as a separate entry with accurate coverage percentages. Additionally, `call`/`link` steps and `collect` steps with prefilled slots are now properly marked as visited.
- Upgrade `litellm` to 1.80.0.
- Fix Enterprise Search Policy triggering `utter_ask_rephrase` instead of `utter_no_relevant_answer_found` when the vector store search returns no matching documents. The `pattern_cannot_handle` now correctly receives the `cannot_handle_no_relevant_answer` as a `context.reason` in all cases where no relevant answer is found, not only when the optional relevancy check rejects the answer.

## \[3.13.22\] - 2026-01-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31322---2026-01-15 'Direct link to [3.13.22] - 2026-01-15')

Rasa Pro 3.13.22 (2026-01-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-67 'Direct link to Bugfixes')

- Fixed an issue where storing documents in FAISS individually could hit the token limit. Now, documents are stored in batches to avoid exceeding the token limit.
- Throw error when duplicate fixtures are found in the same file or across files.
- `SessionEnded` does not reset the tracker anymore.
- Updated `werkzeug`, `filelock`, `fonttools`, `marshmallow`, `urllib3` and `langchain-core`to resolve security vulnerabilities.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-15 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.21\] - 2025-12-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31321---2025-12-29 'Direct link to [3.13.21] - 2025-12-29')

Rasa Pro 3.13.21 (2025-12-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-68 'Direct link to Bugfixes')

- Include response button titles in conversation history in prompts of LLM based components.
- Fix OpenTelemetry exporter `Invalid type <class 'NoneType'> of value None` error when recording the request duration metric for requests made from Rasa server to the custom action server. This error occurred because the url parameter was not being passed to the method that records the request duration metric, resulting in a NoneType value.
- Fix DUT fails with AttributeError when running with custom CommandGenerator

## \[3.13.20\] - 2025-12-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31320---2025-12-19 'Direct link to [3.13.20] - 2025-12-19')

Rasa Pro 3.13.20 (2025-12-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-69 'Direct link to Bugfixes')

- Fixed token expiration validation failing on servers running in non-UTC timezones. Token expiration checks now use timezone-aware UTC datetimes consistently, preventing premature "access token expired" errors on systems in timezones like UTC+8.

## \[3.13.19\] - 2025-12-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31319---2025-12-11 'Direct link to [3.13.19] - 2025-12-11')

Rasa Pro 3.13.19 (2025-12-11)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-70 'Direct link to Bugfixes')

- Fix bug in `CommandPayloadReader` where regex matching did not account for list slots, leading to incorrect parsing of slot keys and values. Now, slot names and values are correctly extracted even if a list is provided.
- Previously, `DialogueStateTracker.has_coexistence_routing_slot` could incorrectly return `True` when the tracker was created without domain slots (i.e., using `AnySlotDict`), because `AnySlotDict` pretends all slots exist. Now, the property returns False in that case, so the routing slot is only considered present if it is actually defined in the domain.
- Fix LLM prompt template loading to raise an error instead of just printing a warning when a custom prompt file is missing or cannot be read.
- Fix `AttributeError: 'str' object has no attribute 'pop'` when using `KnowledgeAnswerCommand` with tracing enabled.
- Fix `UnboundLocalError` in `E2ETestRunner._handle_fail_diff` that caused e2e tests to crash when processing `slot_was_not_set` assertion failures.
- Fixed button payloads with nested parentheses (e.g., `/SetSlots(slot=value)`) not being parsed correctly in `rasa shell`. The payload was incorrectly stripped of its `/SetSlots` prefix, causing it to be treated as text instead of a slot-setting command.
- Add missing deprecation warning for legacy `RASA_PRO_LICENSE` environment variable. Update license validation error messages to reference the actual environment variable used.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-16 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.18\] - 2025-12-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31318---2025-12-04 'Direct link to [3.13.18] - 2025-12-04')

Rasa Pro 3.13.18 (2025-12-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-71 'Direct link to Bugfixes')

- Update `pip` to fix security vulnerability.
- Disable LLM health check for Enterprise Search Policy when generative search is disabled (i.e. use\_generative\_llm is False).
- Fix deduplication of collect steps in cases where the same slot was called in different flows.
- Cancel active flow before triggering internal pattern error during a custom action failure.
- Fix bug with the missing `language_data` module by installing `langcodes` with the `data` extra dependency.

## \[3.13.17\] - 2025-11-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31317---2025-11-27 'Direct link to [3.13.17] - 2025-11-27')

Rasa Pro 3.13.17 (2025-11-27)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-72 'Direct link to Bugfixes')

- Fixed PostgreSQL `UniqueViolation` error when running an assistant with multiple Sanic workers.
- Fix Kafka producer creation failing when SASL mechanism is specified in lowercase. The SASL mechanism is now case-insensitive in the Kafka producer configuration.
- Fix issue where the validation of the assistant files continued even when the provided domain was invalid and was being loaded as empty. The training or validation command didn't exit because the final merged domain contained only the default implementations for patterns, slots and responses and therefore passed the check for being non-empty.
- Trigger pattern\_internal\_error in a CALM assistant when a custom action fails during execution.
- Create AWS Bedrock / Sagemaker client only if the LLM healthcheck environment variable is set. If the environment variable is not set, validate that required credentials are present.
- Update `langchain-core` version to `~0.3.80` to address security vulnerability CVE-2025-65106.
- Raise validation error when duplicate slot definitions are found across domains.
- Raise validation error when a slot with an initial value set is collected by a flow collect step which sets `asks_before_filling` to `true` without having a corresponding collect utterance or custom action.

## \[3.13.16\] - 2025-11-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31316---2025-11-21 'Direct link to [3.13.16] - 2025-11-21')

Rasa Pro 3.13.16 (2025-11-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-73 'Direct link to Bugfixes')

- Fixed `patten-continue-interrupted` running out of order before linked flows.
- Fix `rasa studio upload` timeouts by enabling TCP keep-alive with platform-specific socket options to maintain stable connections.
- Remove `action_metadata` tracing span attribute from `EnterpriseSearchPolicy` instrumentation to prevent PII leakages. Add new environment variable `RASA_TRACING_DEBUGGING_ENABLED` to enable adding `action_metadata` to `EnterpriseSearchPolicy` spans for debugging purposes. By default, this variable is set to `false` to ensure PII is not logged in production environments.

## \[3.13.15\] - 2025-11-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31315---2025-11-13 'Direct link to [3.13.15] - 2025-11-13')

Rasa Pro 3.13.15 (2025-11-13)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-17 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.14\] - 2025-10-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31314---2025-10-30 'Direct link to [3.13.14] - 2025-10-30')

Rasa Pro 3.13.14 (2025-10-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-9 'Direct link to Improvements')

- Add new environment variable `LOG_LEVEL_PYMONGO` to control the logging level of PyMongo dependency of Rasa. This can be useful to reduce the verbosity of logs. Default value is `INFO`.
 
 The logging level of PyMongo can also be set via the `LOG_LEVEL_LIBRARIES` environment variable, which provides the default logging level for a selection of third-party libraries used by Rasa. If both variables are set, `LOG_LEVEL_PYMONGO` takes precedence.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-74 'Direct link to Bugfixes')

- Clean up duplicated `ChitChatAnswerCommands` in command processor.
 
 Ensure that one `CannotHandleCommand` remains if only multiple `CannotHandleCommands` were present.
 
- LLM request timeouts are now enforced at the event loop level using `asyncio.wait_for`.
 
 Previously, timeout values configured in `endpoints.yml` could be overridden by the HTTP client's internal timeout behavior. This fix ensures that when a specific timeout value is configured for LLM requests, the request respects that exact timing regardless of the underlying HTTP client implementation.
 
- Fixed an issue where duplicate collect steps in a flow could cause rendering problems, such as exceeding token limits. This occurred when a flow called another flow multiple times. Now, each collect step is listed only once when retrieving all collect steps for a flow.
 
- If a FloatSlot does not have min and max values set in the domain, the slot will not be validated against any range. If the slot defines an initial value, as well as min and max values, the initial value will be validated against the range and an error will be raised if the initial value is out of range. Enhance run-time validation for new values assigned to FloatSlot instances, ensuring they fall within the defined min and max range if these are set. If min and max are not set, no range validation is performed.
 
- Fixed bug preventing `deployment` parameter from being used in generative response LLM judge configuration.
 
- Fixed bug where `persisted_slots` defined in called flows were incorrectly reset when the parent flow ended, unless they were also explicitly defined in the parent flow. Persisted slots now only need to be defined in the called flow to remain persisted after the parent flow ends.
 
- Fixes the prompt template resolution logic in the CompactLLMCommandGenerator and SearchReadyLLMCommandGenerator classes.
 
- When `action_clean_stack` is used in a user flow, allow `pattern_completed` to be triggered even if there are pattern flows at the bottom of the dialogue stack below the top user flow frame. Previously, `pattern_completed` would only trigger if there were no other frames below the top user flow frame. This assumes that the `action_clean_stack` is the last action called in the user flow.
 
- Update `pip` version used in Dockerfile to `23.*` to address security vulnerability CVE-2023-5752. Update `python-socketio` version to `5.14.0` to address security vulnerability CVE-2025-61765.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-18 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.13\] - 2025-10-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31313---2025-10-13 'Direct link to [3.13.13] - 2025-10-13')

Rasa Pro 3.13.13 (2025-10-13)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-75 'Direct link to Bugfixes')

- Use correct formatting method for messages in `completion` and `acompletion` functions of `LiteLLMRouterLLMClient`.

## \[3.13.12\] - 2025-09-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31312---2025-09-17 'Direct link to [3.13.12] - 2025-09-17')

Rasa Pro 3.13.12 (2025-09-17)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-10 'Direct link to Improvements')

- Accept either `RASA_PRO_LICENSE` or `RASA_LICENSE` as a valid Environment Variable for providing the Rasa License. Also deprecates `RASA_PRO_LICENSE` Environment Variable.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-76 'Direct link to Bugfixes')

- Fixed conversation hanging when using direct action execution with invalid actions module by deferring module validation until action execution time, allowing proper exception handling in the processor.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-19 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.11\] - 2025-09-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31311---2025-09-15 'Direct link to [3.13.11] - 2025-09-15')

Rasa Pro 3.13.11 (2025-09-15)

No significant changes.

## \[3.13.10\] - 2025-09-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31310---2025-09-12 'Direct link to [3.13.10] - 2025-09-12')

Rasa Pro 3.13.10 (2025-09-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-77 'Direct link to Bugfixes')

- Fixes the silence handling bug in Audiocodes Stream Channel where consecutive bot responses could trip silence timeout pattern while the bot is speaking. Audiocodes Stream channel now forcefully cancels the silence timeout watcher whenever it sends the bot response audio.
- Update `langchain` to `0.3.27` and `langchain-community` to `0.3.29` to fix CVE-2025-6984 vulnerability in `langchain-community` version `0.2.19`.

## \[3.13.9\] - 2025-09-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3139---2025-09-03 'Direct link to [3.13.9] - 2025-09-03')

Rasa Pro 3.13.9 (2025-09-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-78 'Direct link to Bugfixes')

- Upgrade `skops` to version `0.13.0` and `requests` to version `2.32.5` to fix vulnerabilities.
- Fixed flow retrieval to skip vector store population when there are no flows to embed.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-20 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.13.8\] - 2025-08-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3138---2025-08-26 'Direct link to [3.13.8] - 2025-08-26')

Rasa Pro 3.13.8 (2025-08-26)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-11 'Direct link to Improvements')

- Validation errors now raise meaningful exceptions instead of calling sys.exit(1), producing clearer and more actionable log messages while allowing custom error handling.
- Enable connection to AWS services for LLMs (e.g. Bedrock, Sagemaker) via multiple additional methods to environment variables, for example by using IAM roles or AWS credentials file.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-79 'Direct link to Bugfixes')

- The .devcontainer docker-compose file now uses the new `docker.io/bitnamilegacy` repository for images, rather than the old `docker.io/bitnami`. This change was made in response to Bitnami's announcement that they will be putting all of their public Helm Chart container images behind a paywall starting August 28th, 2025. Existing public images are moved to the new legacy repository - `bitnamilegacy` - which is only intended for short-term migration purposes.
- Fix expansion of referenced environment variables in `endpoints.yml` during Bedrock model config validation.
- Fixed model loading failures on Windows systems running recent Python versions (3.9.23+, 3.10.18+, 3.11.13+) due to tarfile security fix incompatibility with Windows long path prefix.

## \[3.13.7\] - 2025-08-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3137---2025-08-15 'Direct link to [3.13.7] - 2025-08-15')

Rasa Pro 3.13.7 (2025-08-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-80 'Direct link to Bugfixes')

- Don't tigger a slot correction for slots that are currently set to `None` as `None` counts as empty value.
- Enabled the data argument to support both string and list inputs, normalizing to a list for consistent handling.

## \[3.13.6\] - 2025-08-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3136---2025-08-08 'Direct link to [3.13.6] - 2025-08-08')

Rasa Pro 3.13.6 (2025-08-08)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-81 'Direct link to Bugfixes')

- Fixed the contextual response rephraser to use and update the correct translated response text for the current language, instead of falling back to the default response.
- Refactored `validate_argument_paths` to accept list values and aggregate all missing paths before exiting.

## \[3.13.5\] - 2025-07-31[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3135---2025-07-31 'Direct link to [3.13.5] - 2025-07-31')

Rasa Pro 3.13.5 (2025-07-31)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-82 'Direct link to Bugfixes')

- Fix correction of slots:
 
 - Slots can only be corrected in case they belong to any flow on the stack and the slot to be corrected is part of a collect step in any of those flows.
 - A correction of a slot should be applied if the flow that is about to start is using this slot.
- Upgrade `axios` to fix security vulnerability.
 
- Fix issue where called flows could not set slots of their parent flow.
 

## \[3.13.4\] - 2025-07-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3134---2025-07-23 'Direct link to [3.13.4] - 2025-07-23')

Rasa Pro 3.13.4 (2025-07-23)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-12 'Direct link to Improvements')

- Engine-related modules now raise structured exceptions instead of calling `sys.exit(1)` or `print_error_and_exit`, providing clearer, more actionable log messages and enabling custom error handling.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-83 'Direct link to Bugfixes')

- Fix validation of the FAISS documents folder to ensure it correctly discovers files in a recursive directory structure.
- Updated `Inspector` dependent packages (`vite`, and `@adobe/css-tools`) to address security vulnerabilities.
- Fixed bug preventing `model_group` from being used with `embeddings` in generative response LLM judge configuration.

## \[3.13.3\] - 2025-07-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3133---2025-07-17 'Direct link to [3.13.3] - 2025-07-17')

Rasa Pro 3.13.3 (2025-07-17)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-84 'Direct link to Bugfixes')

- Allowed NLG servers to return None for the text property and ensured custom response data is properly retained.

## \[3.13.2\] - 2025-07-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3132---2025-07-16 'Direct link to [3.13.2] - 2025-07-16')

Rasa Pro 3.13.2 (2025-07-16)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-85 'Direct link to Bugfixes')

- Pass all flows to `find_updated_flows` to avoid creating a `HandleCodeChangeCommand` in situations where flows were not updated.
- Bugfix for Jambonz Stream channel Websocket URL path which resulted in failed Websocket Connections

## \[3.13.1\] - 2025-07-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3131---2025-07-14 'Direct link to [3.13.1] - 2025-07-14')

Rasa Pro 3.13.1 (2025-07-14)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-86 'Direct link to Bugfixes')

- Fix issues with bot not giving feedback for slot corrections.
- Fixed bot speaking state management in Audiocodes Stream channel. This made the assistant unable to handle user silences Added periodic keepAlive messages to deepgram, this interval can be configured with `keep_alive_interval` parameter

## \[3.13.0\] - 2025-07-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3130---2025-07-07 'Direct link to [3.13.0] - 2025-07-07')

Rasa Pro 3.13.0 (2025-07-07)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-3 'Direct link to Deprecations and Removals')

- Deprecate IntentlessPolicy and schedule for removal in Rasa `4.0.0`.
- Removed `monitor_silence` parameter from Voice Channel configuration. Silence Monitoring is now enabled by default. It can be configured by changing the value of Global Silence Timeout
- Remove pre-CALM PII management capability originally released in Rasa Plus 3.6.0. Remove `presidio`, `faker` and `pycountry` dependencies from Rasa Pro.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-3 'Direct link to Features')

- Introducing the `SearchReadyLLMCommandGenerator` component, an enhancement over the `CompactLLMCommandGenerator`. This new component improves the triggering accuracy of `KnowledgeAnswerCommand` and should be used when the `EnterpriseSearchPolicy` is added to the pipeline. By default, this new component does not trigger the `ChitChatAnswerCommand` and `HumanHandoffCommand`. Handling small talk and chit-chat conversations is now delegated to the `EnterpriseSearchPolicy`.
 
 To incorporate the `SearchReadyLLMCommandGenerator` into your pipeline, simply add the following:
 
 pipeline:
 
 ```
    ...- name: SearchReadyLLMCommandGenerator...
    ```
 
- Implement Privacy Filter capability to detect PII in 3 supported events (SlotSet, BotUttered, and UserUttered) and anonymise PII from the event data.
 
 The PII detection takes a tiered approach:
 
 - the first tier represents a slot-based approach: the sensitive data is stored in a slot whose name is defined in the privacy YAML configuration.
 - the second optional tier uses a default GliNer model to detect PII that is not captured by the slot-based approach.
 
 The anonymisation of PII is done by redacting or replacing the sensitive data with a placeholder. These anonymisation rules are also defined in the privacy YAML configuration.
 
- `EnterpriseSearchPolicy` can now assess the relevancy of the generated answer. By default, the policy does not check the relevancy of the generated answer. But you enable the relevancy check by setting `check_relevancy` to `true` in the policy configuration.
 
 ```
      policies:    - name: FlowPolicy    - name: EnterpriseSearchPolicy      ...      check_relevancy: true # by default this is false
    ```
 
 If the relevancy check is enabled, the policy will evaluate the generated answer and determine if it is relevant to the user's query. In case the answer is relevant, the generated answer will be returned to the user. If the answer is not relevant, the policy will fallback to `pattern_cannot_handle` to handle the situation.
 
 ```
    pattern_cannot_handle:  description: |    Conversation repair flow for addressing failed command generation scenarios  name: pattern cannot handle  steps:    - noop: true      next:          ... # other cases        # Fallback to handle cases where the generated answer is not relevant            - if: "'{{context.reason}}' = 'cannot_handle_no_relevant_answer'"            then:              - action: utter_no_relevant_answer_found                next: "END"
    ```
 
- Implement privacy manager class which manages privacy-related tasks in the background.
 
 This class handles the anonymization and deletion of sensitive information in dialogue state trackers stored in the tracker store, as well as the streaming of anonymized events to event brokers. It uses background schedulers to periodically run these tasks and processes trackers from a queue to ensure that sensitive information is handled in a timely manner.
 
- Add support for `jambonz_stream` voice-stream channel. Log level of `websockets` library is set to `ERROR` by default, it can be changed by the environment variable `LOG_LEVEL_LIBRARIES`.
 
- Remove the beta feature flag check from the `RepeatBotMessagesCommand`. The `RASA_PRO_BETA_REPEAT_COMMAND` environment variable is no longer needed as the feature is GA in 3.13.0.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-13 'Direct link to Improvements')

- Coverage report feature can now be used independently of RASA\_PRO\_BETA\_FINE\_TUNING\_RECIPE feature flag.
 
- Update default Embedding model from `text-embedding-ada-002` to `text-embedding-3-large`. Update `LLMBasedRouter`, `IntentlessPolicy` and `ContextualResponseRephraser` default models to use `gpt-4o-2024-11-20` instead of `gpt-3.5-turbo`. Update the `EnterpriseSearchPolicy` to use `gpt-4.1-mini-2025-04-14` instead of `gpt-3.5-turbo`.
 
- Update `MultistepCommandGenerator` to use `gpt-4o`, specifically `gpt-4o-2024-11-20`, instead of `gpt-3.5-turbo` as `gpt-3.5-turbo` will be deprecated on July 16, 2025. Add deprecation warning for `SingleStepCommandGenerator`; leave current default model as `gpt-4-0613`. Adapt tests for CommandGenerators to use `gpt-4o` instead of `gpt-3.5-turbo` due to upcoming model deprecation. Update `LLMJudgeModel` and `Fine tuning Conversation Rephraser` to use `gpt-4.1-mini`, specifically `gpt-4.1-mini-2025-04-14`, instead of `gpt-4o-mini`, as `gpt-4o-mini` will be deprecated on September 15, 2025.
 
- Make `CALM` template the default for `rasa init`.
 
- Redis lock store now accepts `host` property from endpoints config. Property `url` is marked as deprecated for Redis lock store.
 
- Added support for basic authentication in Twilio channels (Voice Ready and Voice Streaming). This allows users to authenticate their Twilio channels using basic authentication credentials, enhancing security and access control for voice communication. To use this feature, set `username` and `password` in the Twilio channel configuration.
 
 credentials.yaml
 
 ```
    twilio_voice:    username: your_username    password: your_password    ...twilio_media_streams:    username: your_username    password: your_password    ...
    ```
 
 At Twilio, configure the webhook URL to include the basic authentication credentials:
 
 ```
    # twilio voice webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_voice/webhook# twilio media streams webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_media_streams/webhook
    ```
 
- Added two endpoints to the model service for retrieving the assistant’s default configuration and the default project template used by the assistant.
 
- All conversations on Twilio Media Streams, Audiocodes Stream, Genesys channel ends with the message `/session_end`. This applies to both the conversation ended by user and the assistant.
 
- Refactor Voice Inspector to use AudioWorklet Web API
 
- Added new CLI commands to support project-level linking and granular push/pull workflows for Rasa Studio assistants:
 
 - Introduced `rasa studio link` to associate a local project with a specific Studio assistant.
 - Added `rasa studio pull` and `rasa studio push`, with subcommands for granular resource updates (`config`, `endpoints`) between local and Studio assistants.
- Implement `delete` method for `InMemoryTrackerStore`,`AuthRetryTrackerStore`, `AwaitableTrackerStore` and `FailSafeTrackerStore` subclasses.
 
- Implement `delete` method for `MongoTrackerStore`.
 
- Implement `delete` method for SQLTrackerStore: this method accepts sender\_id and deletes the corresponding tracker from the store if it exists.
 
- Implement `delete` method for DynamoTrackerStore: this method accepts sender\_id and deletes the corresponding tracker from the store if it exists.
 
- Implement `delete` method for `RedisTrackerStore`.
 
- Update `KafkaEventBroker` YAML config to accept 2 new parameters:
 
 - `stream_pii`: boolean (default: true). If set to `false`, the broker will not publish un-anonymised events to the configured topic.
 - `anonymization_topics`: list of strings (default: \[\]). If set, the broker will publish anonymised events to the configured topics when the PII management capability is enabled.
- Add 2 new PII configuration parameters to `PikaEventBroker`:
 
 - `stream_pii`: Boolean flag to control whether or not to publish un-anonymised events to the configured `queues`. If set to False, un-anonymised events won't be published to RabbitMQ. Defaults to `True` for backwards compatibility.
 - `anonymization_queues`: List of queue names that should receive anonymized events with PII removed. Defaults to an empty list (`[]`).
- Add a new `anonymized_at` timestamp field to the Rasa Pro events supported by the PII management capability:
 
 - `user`
 - `slot`
 - `bot` This field indicates when the PII data in the event was anonymized. The field is set to `null` if the PII data has not been anonymized.
- Add support for reading `privacy` configuration key from endpoints file to enable PII management capability.
 
- Add new `DELETE /conversations/<conversation_id>/tracker` endpoint that allows deletion of tracker data for a specific conversation.
 
- Cleaned up potential sources of PII from `info`, `exception` and `error` logs.
 
- Allow default patterns to link to `pattern_chitchat`.
 
- Cleaned up potential sources of PII from `warning` logs.
 
- Add a warning log to alert users when exporting tracker data that contains unanonymized events.
 
- Remove beta feature flag for pypred predicate usage in conditional response variations. Mark this functionality for general availability (GA).
 
- Updated the default behavior of `pattern_chitchat`. With the deprecation of `IntentlessPolicy`, `pattern_chitchat` now defaults to responding with `utter_cannot_handle` instead of triggering `action_trigger_chitchat`.
 
- PII deletion job now performs a single database transaction in the case of trackers with multiple sessions, where only some sessions are eligible for deletion. The deletion job overwrites the serialized tracker record with the retained events only. This ensures that the tracker store is updated atomically, preventing any potential tracker data loss in case of Rasa Pro server crashes or other issues during the deletion job execution.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-87 'Direct link to Bugfixes')

- Fixes a bug where fine-tuning data generation raises a FineTuningDataPreparationException for test cases without assertions that end with one/multiple user utterance/s, as opposed to one/multiple bot utterance/s.
 
 For these types of test cases, the fine-tuning data generation transcript now contains all test case utterances up to but excluding the last user utterance(s) as the test case does not specify the corresponding, expected bot response(s).
 
- allowed for usage of litellm model prefixes that do not end in '<provider>/'
 
- Fixes a bug in Inspector that raised a TypeError when serialising `numpy.float64` from Tracker
 
- - Fixes an issue with prompt rendering where minified JSON structures were displayed without properly escaping newlines, tabs, and quotes.
 - Introduced a new Jinja `filter to_json_encoded_string` that escapes newlines (`\n`), tabs (`\t`), and quotes (`\"`) for safe JSON rendering. `to_json_encoded_string` filter preserves other special characters (e.g., umlauts) without encoding them.
 - Updated the default prompts for `gpt-4o` and `claude-sonnet-3.5`
- Fix intermittent crashes on Inspector app when Enterprise Search or Chitchat Policies were triggered
 
- Add channel name to UserMessage created by the Audiocodes channel.
 
- Reduced redundant log entries when reading prompt templates by contextualizing them. Added source component and method metadata to logs, and changed prompt-loading logs triggered from `fingerprint_addon` to use `DEBUG` level.
 
- Prompts for rephrased messages and conversations are now rendered using agent, eliminating errors that occurred when string replacements broke after prompt updates.
 
- Fix retrieval of latest tracker session from DynamoTrackerStore.
 
- Files generated by the finetuning data generation pipeline are now encoded in UTF-8, allowing characters such as German umlauts (ä, ö, ü) to render correctly.
 
- Fix an issue where the SetSlot and Clarify command value was parsed incorrectly if a newline character immediately followed the value argument in the LLM output.
 
- Fixed the repeat action to include all messages of the last turn in collect steps.
 
- - Fix an issue where running `rasa inspect` would always set the `route_session_to_calm` slot to `True`, even when no user-triggered commands were present. This caused incorrect routing to CALM, bypassing the logic of the router.
 - Fix a regression in non-sticky routing where sessions intended for the NLU were incorrectly routed to CALM when the router predicted `NoopCommand()`.
- Fix validation and improve error messages for the documents source directory when FAISS vector store is configured for the Enterprise Search Policy.
 
- Prevent slot correction when the slot is already set to the desired value.
 
- Fallback to `CannotHandle` command when the slot predicted by the LLM is not defined in the domain.
 
- Fix tracker store PII background jobs not updating the `InMemoryTrackerStore` reference stored by the Agent.
 
- Fix potential `KeyError` in `EnterpriseSearchPolicy` citation post-processing when the source does not have the correct citation. Improve the citation post-processing logic to handle additional edge cases.
 
- Fix issue with PrivacyManager unable to retrieve trackers from Redis tracker store during its deletion background job, because the Redis tracker store prefix was added twice during retrieval.
 
- Fixed anonymization failing for `FloatSlots` when user-provided integer values don't match the stored float representation during text replacement.
 
- Fix Redis tracker store saving tracker with prefix added twice during the anonymization background job.
 
- Fix DynamoDB tracker store overwriting tracker with the latest session in case of multiple sessions saved prior.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-21 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.42\] - 2025-12-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31242---2025-12-11 'Direct link to [3.12.42] - 2025-12-11')

Rasa Pro 3.12.42 (2025-12-11)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-88 'Direct link to Bugfixes')

- Fix bug in `CommandPayloadReader` where regex matching did not account for list slots, leading to incorrect parsing of slot keys and values. Now, slot names and values are correctly extracted even if a list is provided.
- Previously, `DialogueStateTracker.has_coexistence_routing_slot` could incorrectly return `True` when the tracker was created without domain slots (i.e., using `AnySlotDict`), because `AnySlotDict` pretends all slots exist. Now, the property returns False in that case, so the routing slot is only considered present if it is actually defined in the domain.
- Fix LLM prompt template loading to raise an error instead of just printing a warning when a custom prompt file is missing or cannot be read.
- Fix `AttributeError: 'str' object has no attribute 'pop'` when using `KnowledgeAnswerCommand` with tracing enabled.
- Fix `UnboundLocalError` in `E2ETestRunner._handle_fail_diff` that caused e2e tests to crash when processing `slot_was_not_set` assertion failures.
- Fixed button payloads with nested parentheses (e.g., `/SetSlots(slot=value)`) not being parsed correctly in `rasa shell`. The payload was incorrectly stripped of its `/SetSlots` prefix, causing it to be treated as text instead of a slot-setting command.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-22 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.41\] - 2025-12-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31241---2025-12-04 'Direct link to [3.12.41] - 2025-12-04')

Rasa Pro 3.12.41 (2025-12-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-89 'Direct link to Bugfixes')

- Update `pip` to fix security vulnerability.
- Disable LLM health check for Enterprise Search Policy when generative search is disabled (i.e. use\_generative\_llm is False).
- Fix deduplication of collect steps in cases where the same slot was called in different flows.
- Cancel active flow before triggering internal pattern error during a custom action failure.
- Fix bug with the missing `language_data` module by installing `langcodes` with the `data` extra dependency.

## \[3.12.40\] - 2025-11-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31240---2025-11-27 'Direct link to [3.12.40] - 2025-11-27')

Rasa Pro 3.12.40 (2025-11-27)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-90 'Direct link to Bugfixes')

- Fixed PostgreSQL `UniqueViolation` error when running an assistant with multiple Sanic workers.
- Fix issue where the validation of the assistant files continued even when the provided domain was invalid and was being loaded as empty. The training or validation command didn't exit because the final merged domain contained only the default implementations for patterns, slots and responses and therefore passed the check for being non-empty.
- Trigger pattern\_internal\_error in a CALM assistant when a custom action fails during execution.
- Update `langchain-core` version to `~0.3.80` to address security vulnerability CVE-2025-65106.
- Raise validation error when duplicate slot definitions are found across domains.
- Raise validation error when a slot with an initial value set is collected by a flow collect step which sets `asks_before_filling` to `true` without having a corresponding collect utterance or custom action.

## \[3.12.39\] - 2025-11-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31239---2025-11-21 'Direct link to [3.12.39] - 2025-11-21')

Rasa Pro 3.12.39 (2025-11-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-91 'Direct link to Bugfixes')

- Fixed `patten-continue-interrupted` running out of order before linked flows.
- Remove `action_metadata` tracing span attribute from `EnterpriseSearchPolicy` instrumentation to prevent PII leakages. Add new environment variable `RASA_TRACING_DEBUGGING_ENABLED` to enable adding `action_metadata` to `EnterpriseSearchPolicy` spans for debugging purposes. By default, this variable is set to `false` to ensure PII is not logged in production environments.
- Fix Kafka producer creation failing when SASL mechanism is specified in lowercase. The SASL mechanism is now case-insensitive in the Kafka producer configuration.

## \[3.12.38\] - 2025-11-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31238---2025-11-13 'Direct link to [3.12.38] - 2025-11-13')

Rasa Pro 3.12.38 (2025-11-13)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-23 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.37\] - 2025-10-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31237---2025-10-30 'Direct link to [3.12.37] - 2025-10-30')

Rasa Pro 3.12.37 (2025-10-30)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-92 'Direct link to Bugfixes')

- Clean up duplicated `ChitChatAnswerCommands` in command processor.
 
 Ensure that one `CannotHandleCommand` remains if only multiple `CannotHandleCommands` were present.
 
- LLM request timeouts are now enforced at the event loop level using `asyncio.wait_for`.
 
 Previously, timeout values configured in `endpoints.yml` could be overridden by the HTTP client's internal timeout behavior. This fix ensures that when a specific timeout value is configured for LLM requests, the request respects that exact timing regardless of the underlying HTTP client implementation.
 
- Fixed an issue where duplicate collect steps in a flow could cause rendering problems, such as exceeding token limits. This occurred when a flow called another flow multiple times. Now, each collect step is listed only once when retrieving all collect steps for a flow.
 
- Fixes the prompt template resolution logic in the CompactLLMCommandGenerator.
 
- When `action_clean_stack` is used in a user flow, allow `pattern_completed` to be triggered even if there are pattern flows at the bottom of the dialogue stack below the top user flow frame. Previously, `pattern_completed` would only trigger if there were no other frames below the top user flow frame. This assumes that the `action_clean_stack` is the last action called in the user flow.
 
- Update `pip` version used in Dockerfile to `23.*` to address security vulnerability CVE-2023-5752.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-24 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.36\] - 2025-10-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31236---2025-10-27 'Direct link to [3.12.36] - 2025-10-27')

Rasa Pro 3.12.36 (2025-10-27)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-25 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.35\] - 2025-10-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31235---2025-10-21 'Direct link to [3.12.35] - 2025-10-21')

Rasa Pro 3.12.35 (2025-10-21)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-14 'Direct link to Improvements')

- Add new environment variable `LOG_LEVEL_PYMONGO` to control the logging level of PyMongo dependency of Rasa. This can be useful to reduce the verbosity of logs. Default value is `INFO`.
 
 The logging level of PyMongo can also be set via the `LOG_LEVEL_LIBRARIES` environment variable, which provides the default logging level for a selection of third-party libraries used by Rasa. If both variables are set, `LOG_LEVEL_PYMONGO` takes precedence.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-93 'Direct link to Bugfixes')

- If a FloatSlot does not have min and max values set in the domain, the slot will not be validated against any range. If the slot defines an initial value, as well as min and max values, the initial value will be validated against the range and an error will be raised if the initial value is out of range. Enhance run-time validation for new values assigned to FloatSlot instances, ensuring they fall within the defined min and max range if these are set. If min and max are not set, no range validation is performed.
- Fixed bug preventing `deployment` parameter from being used in generative response LLM judge configuration.
- Fixed bug where `persisted_slots` defined in called flows were incorrectly reset when the parent flow ended, unless they were also explicitly defined in the parent flow. Persisted slots now only need to be defined in the called flow to remain persisted after the parent flow ends.

## \[3.12.34\] - 2025-10-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31234---2025-10-13 'Direct link to [3.12.34] - 2025-10-13')

Rasa Pro 3.12.34 (2025-10-13)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-94 'Direct link to Bugfixes')

- Fixed conversation hanging when using direct action execution with invalid actions module by deferring module validation until action execution time, allowing proper exception handling in the processor.
- Use correct formatting method for messages in `completion` and `acompletion` functions of `LiteLLMRouterLLMClient`.

## \[3.12.33\] - 2025-09-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31233---2025-09-12 'Direct link to [3.12.33] - 2025-09-12')

Rasa Pro 3.12.33 (2025-09-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-95 'Direct link to Bugfixes')

- Fixes the silence handling bug in Audiocodes Stream Channel where consecutive bot responses could trip silence timeout pattern while the bot is speaking. Audiocodes Stream channel now forcefully cancels the silence timeout watcher whenever it sends the bot response audio.
- Fixed flow retrieval to skip vector store population when there are no flows to embed.

## \[3.12.32\] - 2025-09-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31232---2025-09-03 'Direct link to [3.12.32] - 2025-09-03')

Rasa Pro 3.12.32 (2025-09-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-96 'Direct link to Bugfixes')

- Upgrade `skops` to version `0.13.0` and `requests` to version `2.32.5` to fix vulnerabilities.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-26 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.31\] - 2025-08-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31231---2025-08-26 'Direct link to [3.12.31] - 2025-08-26')

Rasa Pro 3.12.31 (2025-08-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-97 'Direct link to Bugfixes')

- The .devcontainer docker-compose file now uses the new `docker.io/bitnamilegacy` repository for images, rather than the old `docker.io/bitnami`. This change was made in response to Bitnami's announcement that they will be putting all of their public Helm Chart container images behind a paywall starting August 28th, 2025. Existing public images are moved to the new legacy repository - `bitnamilegacy` - which is only intended for short-term migration purposes.
- Fixed model loading failures on Windows systems running recent Python versions (3.9.23+, 3.10.18+, 3.11.13+) due to tarfile security fix incompatibility with Windows long path prefix.

## \[3.12.30\] - 2025-08-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31230---2025-08-19 'Direct link to [3.12.30] - 2025-08-19')

Rasa Pro 3.12.30 (2025-08-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-98 'Direct link to Bugfixes')

- Don't tigger a slot correction for slots that are currently set to `None` as `None` counts as empty value.

## \[3.12.29\] - 2025-07-31[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31229---2025-07-31 'Direct link to [3.12.29] - 2025-07-31')

Rasa Pro 3.12.29 (2025-07-31)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-99 'Direct link to Bugfixes')

- Fix correction of slots:
 
 - Slots can only be corrected in case they belong to any flow on the stack and the slot to be corrected is part of a collect step in any of those flows.
 - A correction of a slot should be applied if the flow that is about to start is using this slot.

## \[3.12.28\] - 2025-07-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31228---2025-07-29 'Direct link to [3.12.28] - 2025-07-29')

Rasa Pro 3.12.28 (2025-07-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-100 'Direct link to Bugfixes')

- Upgrade `axios` to fix security vulnerability.
- Fix issue where called flows could not set slots of their parent flow.

## \[3.12.27\] - 2025-07-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31227---2025-07-23 'Direct link to [3.12.27] - 2025-07-23')

Rasa Pro 3.12.27 (2025-07-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-101 'Direct link to Bugfixes')

- Fix validation of the FAISS documents folder to ensure it correctly discovers files in a recursive directory structure.
- Updated `Inspector` dependent packages (`vite`, and `@adobe/css-tools`) to address security vulnerabilities.
- Fixed bug preventing `model_group` from being used with `embeddings` in generative response LLM judge configuration.

## \[3.12.26\] - 2025-07-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31226---2025-07-17 'Direct link to [3.12.26] - 2025-07-17')

Rasa Pro 3.12.26 (2025-07-17)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-102 'Direct link to Bugfixes')

- Allowed NLG servers to return None for the text property and ensured custom response data is properly retained.

## \[3.12.25\] - 2025-07-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31225---2025-07-16 'Direct link to [3.12.25] - 2025-07-16')

Rasa Pro 3.12.25 (2025-07-16)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-103 'Direct link to Bugfixes')

- Pass all flows to `find_updated_flows` to avoid creating a `HandleCodeChangeCommand` in situations where flows were not updated.

## \[3.12.24\] - 2025-07-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31224---2025-07-14 'Direct link to [3.12.24] - 2025-07-14')

Rasa Pro 3.12.24 (2025-07-14)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-104 'Direct link to Bugfixes')

- Fixed the repeat action to include all messages of the last turn in collect steps.
- Fix issues with bot not giving feedback for slot corrections.
- Fixed bot speaking state management in Audiocodes Stream channel. This made the assistant unable to handle user silences Added periodic keepAlive messages to deepgram, this interval can be configured with `keep_alive_interval` parameter

## \[3.12.23\] - 2025-07-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31223---2025-07-08 'Direct link to [3.12.23] - 2025-07-08')

Rasa Pro 3.12.23 (2025-07-08)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-105 'Direct link to Bugfixes')

- Reverted fix for custom multilingual output payloads being overwritten.

## \[3.12.22\] - 2025-07-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31222---2025-07-07 'Direct link to [3.12.22] - 2025-07-07')

Rasa Pro 3.12.22 (2025-07-07)

No significant changes.

## \[3.12.21\] - 2025-07-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31221---2025-07-03 'Direct link to [3.12.21] - 2025-07-03')

Rasa Pro 3.12.21 (2025-07-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-106 'Direct link to Bugfixes')

- Fixes a bug where fine-tuning data generation raises a FineTuningDataPreparationException for test cases without assertions that end with one/multiple user utterance/s, as opposed to one/multiple bot utterance/s.
 
 For these types of test cases, the fine-tuning data generation transcript now contains all test case utterances up to but excluding the last user utterance(s) as the test case does not specify the corresponding, expected bot response(s).
 
- Fix validation and improve error messages for the documents source directory when FAISS vector store is configured for the Enterprise Search Policy.
 
- Prevent slot correction when the slot is already set to the desired value.
 
- Fallback to `CannotHandle` command when the slot predicted by the LLM is not defined in the domain.
 
- Fix potential `KeyError` in `EnterpriseSearchPolicy` citation post-processing when the source does not have the correct citation. Improve the citation post-processing logic to handle additional edge cases.
 

## \[3.12.20\] - 2025-06-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31220---2025-06-24 'Direct link to [3.12.20] - 2025-06-24')

Rasa Pro 3.12.20 (2025-06-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-107 'Direct link to Bugfixes')

- Flows now traverse called and linked flows, including nested and branching called / linked flows. As a result, E2E coverage reports include any linked and called flows triggered by the flow being tested.
- Fixed a bug in Genesys and Audiocodes Stream channels where conversations used a placeholder value `default` as the Sender ID. Added a new method `VoiceInputChannel.get_sender_id(call_parameters)` that returns the platform's `call_id` as the Sender ID. This ensures each conversation has a unique identifier based on the call ID from the respective platform. Channels can override this method to customize Sender ID generation.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-27 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.19\] - 2025-06-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31219---2025-06-18 'Direct link to [3.12.19] - 2025-06-18')

Rasa Pro 3.12.19 (2025-06-18)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-108 'Direct link to Bugfixes')

- Fix issues where linked flows could not be cancelled and slots collected within linked flows could not be prefilled.
- Fix `InvalidFlowStepIdException` thrown when a bot is restarted with a retrained model which contains an update to the step order in a given active flow. Once retrained and restarted, the bot will now correctly handle the updated step order by triggering `pattern_code_change`.

## \[3.12.18\] - 2025-06-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31218---2025-06-12 'Direct link to [3.12.18] - 2025-06-12')

Rasa Pro 3.12.18 (2025-06-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-109 'Direct link to Bugfixes')

- Ensure that old step ID formats (without the flow ID prefix) can be loaded without raising an `InvalidFlowStepIdException` in newer Rasa versions that expect the flow ID prefix.
- - Fix an issue where running `rasa inspect` would always set the `route_session_to_calm` slot to `True`, even when no user-triggered commands were present. This caused incorrect routing to CALM, bypassing the logic of the router.
 - Fix a regression in non-sticky routing where sessions intended for the NLU were incorrectly routed to CALM when the router predicted `NoopCommand()`.
- Enable slot prefilling in patterns.

## \[3.12.17\] - 2025-06-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31217---2025-06-05 'Direct link to [3.12.17] - 2025-06-05')

Rasa Pro 3.12.17 (2025-06-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-110 'Direct link to Bugfixes')

- Fix an issue where the SetSlot and Clarify command value was parsed incorrectly if a newline character immediately followed the value argument in the LLM output.

## \[3.12.16\] - 2025-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31216---2025-06-03 'Direct link to [3.12.16] - 2025-06-03')

Rasa Pro 3.12.16 (2025-06-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-111 'Direct link to Bugfixes')

- Make `domain` an optional argument in `CommandProcessorComponent`. This change addresses a potential `TypeError` that could occur when loading a model trained without providing domain as a required argument. By making domain optional, models trained with older configurations or without a domain component will now load correctly without errors.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-28 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.15\] - 2025-06-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31215---2025-06-02 'Direct link to [3.12.15] - 2025-06-02')

Rasa Pro 3.12.15 (2025-06-02)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-15 'Direct link to Improvements')

:

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-112 'Direct link to Bugfixes')

- Add turn\_wrapper to count multiple utterances by bot/user as single turn rather than individual turns. Always include last user utterance in rephraser prompt's conversation history.
- Remove `StoryGraphProvider` from the prediction graph in case `IntentlessPolicy` is not present to reduce the loading time of the bot in cases where the `IntentlessPolicy` is not used and a lot of stories are present.

## \[3.12.14\] - 2025-05-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31214---2025-05-28 'Direct link to [3.12.14] - 2025-05-28')

Rasa Pro 3.12.14 (2025-05-28)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-16 'Direct link to Improvements')

- - Updates the parameter name from `max_tokens`, which is deprecated by OpenAI, to `max_completion_tokens`. The old `max_tokens` is not supported for the `o` models.
 - Exposes LiteLLM's `drop_params` parameter for LLM configurations.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-113 'Direct link to Bugfixes')

- - Fixes default config initialization for the `IntentlessPolicy`.
- Prompts for rephrased messages and conversations are now rendered using agent, eliminating errors that occurred when string replacements broke after prompt updates.
- Fix parsing of escape characters in translation of LLM prediction to SetSlotCommand.
- Files generated by the finetuning data generation pipeline are now encoded in UTF-8, allowing characters such as German umlauts (ä, ö, ü) to render correctly.

## \[3.12.13\] - 2025-05-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31213---2025-05-19 'Direct link to [3.12.13] - 2025-05-19')

Rasa Pro 3.12.13 (2025-05-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-114 'Direct link to Bugfixes')

- The `Clarify` (syntax used by `SingleStepLLMCommandGenerator`) / `disambiguate flows` (syntax used by `CompactLLMCommandGenerator`) command will now parse flow names with dashes.
- Fix remote model download when models are stored in a path, not in the root of the remote storage. Add new training CLI param `--remote-root-only` that can be used by the model service to store the model in the root of the remote storage. Propagate this parameter to the persistor's `persist` method. Simplify persistor code when retrieving models by downloading the model to the target path directly rather than copying the downloaded model to the target path. This also improved testability.
- When inspector is not used, root server path should output: `Hello from Rasa: <version>.`. When inspector is used, root server path should output HTML page with a link to the path on which inspector can be reached.
- Change the default value of `minimize_num_calls` in the config of LLM-based command generators to `True`.

## \[3.12.12\] - 2025-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31212---2025-05-15 'Direct link to [3.12.12] - 2025-05-15')

Rasa Pro 3.12.12 (2025-05-15)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-17 'Direct link to Improvements')

- Improved `CRFEntityExtractor` persistence and loading methods to improve model loading times.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-115 'Direct link to Bugfixes')

- Bumps aiohttp to 3.10.x, sentry-sdk to 2.8.x. Also bumps the locked versions for urllib3 and h11

## \[3.12.11\] - 2025-05-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31211---2025-05-14 'Direct link to [3.12.11] - 2025-05-14')

Rasa Pro 3.12.11 (2025-05-14)

No significant changes.

## \[3.12.10\] - 2025-05-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31210---2025-05-08 'Direct link to [3.12.10] - 2025-05-08')

Rasa Pro 3.12.10 (2025-05-08)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-116 'Direct link to Bugfixes')

- Fix issues in Audiocodes Channel Connector. The values in `user_phone` and `bot_phone` available in session\_started\_metadata are swapped to correctly map to `calller` and `callee` respectively. Fixed evening handling where events without the key "parameters" raised an exception.
- Filtered out only `None` values from the response payload to avoid removing valid empty values.

## \[3.12.9\] - 2025-05-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3129---2025-05-06 'Direct link to [3.12.9] - 2025-05-06')

Rasa Pro 3.12.9 (2025-05-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-117 'Direct link to Bugfixes')

- Implemented backtracking and corrected the recursion logic in the all-paths generation for a flow. Removed the need to deepcopy the `step_ids_visited` set on each branch within `_handle_links`, which prevents the coverage report from freezing due to hitting the recursion limit.
- Upgrade openai and litellm dependencies to fix found vulnerabilities in litellm.

## \[3.12.8\] - 2025-04-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3128---2025-04-30 'Direct link to [3.12.8] - 2025-04-30')

Rasa Pro 3.12.8 (2025-04-30)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-118 'Direct link to Bugfixes')

- Reduced redundant log entries when reading prompt templates by contextualizing them. Added source component and method metadata to logs, and changed prompt-loading logs triggered from `fingerprint_addon` to use `DEBUG` level.
- Add support for `auth_token` to Rasa Inspector.
- Fixed issue where a slot mapping referencing a form in the `active_loop` slot mapping conditions that wouldn't list this slot in the form's `required_slots` caused training to fail. Now, this is only logged as a validation warning without causing the training to fail.

## \[3.12.7\] - 2025-04-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3127---2025-04-28 'Direct link to [3.12.7] - 2025-04-28')

Rasa Pro 3.12.7 (2025-04-28)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-18 'Direct link to Improvements')

- Adds two optional properties on Jambonz channel connector, `username` and `password` which can be used to enable Basic Access Authentication
 
- Added support for basic authentication in Twilio channels (Voice Ready and Voice Streaming). This allows users to authenticate their Twilio channels using basic authentication credentials, enhancing security and access control for voice communication. To use this feature, set `username` and `password` in the Twilio channel configuration.
 
 credentials.yaml
 
 ```
    twilio_voice:    username: your_username    password: your_password    ...twilio_media_streams:    username: your_username    password: your_password    ...
    ```
 
 At Twilio, configure the webhook URL to include the basic authentication credentials:
 
 ```
    # twilio voice webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_voice/webhook# twilio media streams webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_media_streams/webhook
    ```
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-119 'Direct link to Bugfixes')

- Fail rasa commands (`rasa run`, `rasa inspect`, `rasa shell`) when model file path doesn't exist instead of defaulting to the latest model file from the default directory `/models`.
- Fix the behaviour of `action_hangup` on Voice Inspector (browser\_audio channel). Display that the session has ended
- Display a helpful error message in case of invalid API Key with Azure TTS
- Fixes Audiocodes Channel's event to intent mapping. All audiocode events are now mapped to an intent in the format `vaig_event_<event>`. That is, if Audiocodes sends an event `noUserInput`, the assistant will receive the intent `/vaig_event_noUserInput`

## \[3.12.6\] - 2025-04-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3126---2025-04-15 'Direct link to [3.12.6] - 2025-04-15')

Rasa Pro 3.12.6 (2025-04-15)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-4 'Direct link to Deprecations and Removals')

- Remove the behaviour handling digressions as eligible flows that can be started while handling an active collect step. These properties have been removed from the flow and collect step:
 - `ask_confirm_digressions`
 - `block_digressions` Remove the new pattern `pattern_handle_digressions`.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-4 'Direct link to Features')

- Introduce a new boolean property `force_slot_filling` for the `collect` flow step. This property allows you to suppress incorrect predictions of the command generator or user digressions that are not relevant to the current slot filling. To enable this behavior, you should set the `force_slot_filling` property to `True` in the `collect` step of your flow configuration.
 
 ```
    flows:    order_pizza:      name: order pizza      description: user asks for a pizza      steps:      - collect: pizza_type      - collect: quantity      - collect: address        force_slot_filling: true
    ```
 
 When `force_slot_filling` is set to `True`, the command generator will only process the `SetSlot` command for the specified slot. By default, the property is set to `False`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-120 'Direct link to Bugfixes')

- Security patch for Audiocodes and Genesys Channel connector Adds `api_key` (required) and `client_secret` (optional) properties to Genesys channel configuration Adds `token` (optional) property to Audiocodes-Stream channel
- Fix execution of custom validation action `action_validate_slot_mappings` by passing the extracted slot events to the tracker used when running this action.
- Make sure training always fail when domain is invalid.
- Add channel name to UserMessage created by the Audiocodes channel.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-29 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.5\] - 2025-04-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3125---2025-04-07 'Direct link to [3.12.5] - 2025-04-07')

Rasa Pro 3.12.5 (2025-04-07)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-121 'Direct link to Bugfixes')

- Fix `ChitChatAnswerCommand` command replaced with `CannotHandleCommand` if there are no e2e stories defined. Improve validation that `IntentlessPolicy` has applicable responses: either responses in the domain that are not part of any flow, or if there are e2e stories. This validation performed during the training time and during cleanup of the `ChitChatAnswerCommand` command.
- - Fixes an issue with prompt rendering where minified JSON structures were displayed without properly escaping newlines, tabs, and quotes.
 - Introduced a new Jinja `filter to_json_encoded_string` that escapes newlines (`\n`), tabs (`\t`), and quotes (`\"`) for safe JSON rendering. `to_json_encoded_string` filter preserves other special characters (e.g., umlauts) without encoding them.
 - Updated the default prompts for `gpt-4o` and `claude-sonnet-3.5`

## \[3.12.4\] - 2025-04-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3124---2025-04-01 'Direct link to [3.12.4] - 2025-04-01')

Rasa Pro 3.12.4 (2025-04-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-122 'Direct link to Bugfixes')

- Send error event to the Kafka broker, when the original message size is too large, above the configured broker limit. This error handling mechanism was added to prevent Rasa-Pro server crashes.
- Update the following dependencies to versions that contain the latest patches for security vulnerabilities:
 - `jinja2`
 - `werkzeug`
 - `cryptography`
 - `pyarrow`
 - `langchain`
 - `langchain-community`
- Fix intermittent crashes on Inspector app when Enterprise Search or Chitchat Policies were triggered

## \[3.12.3\] - 2025-03-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3123---2025-03-26 'Direct link to [3.12.3] - 2025-03-26')

Rasa Pro 3.12.3 (2025-03-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-123 'Direct link to Bugfixes')

- Fixes a bug in Inspector that raised a TypeError when serialising `numpy.float64` from Tracker

## \[3.12.2\] - 2025-03-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3122---2025-03-25 'Direct link to [3.12.2] - 2025-03-25')

Rasa Pro 3.12.2 (2025-03-25)

No significant changes.

## \[3.12.1\] - 2025-03-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3121---2025-03-21 'Direct link to [3.12.1] - 2025-03-21')

Rasa Pro 3.12.1 (2025-03-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-124 'Direct link to Bugfixes')

- Fix filtering of StartFlow commands from LLM-based command generators during the overlap check with prior commands. When prior commands do not contain any StartFlow or HandleDigression commands, we should not filter out the StartFlow command from the LLM-based command generator.
- Remove:
 - cancel command when digression handling is defined
 - duplicate digression handling commands during command processing

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-30 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.12.0\] - 2025-03-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3120---2025-03-19 'Direct link to [3.12.0] - 2025-03-19')

Rasa Pro 3.12.0 (2025-03-19)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-5 'Direct link to Deprecations and Removals')

- Deprecate MultiStepLLMCommandGenerator and schedule for removal in Rasa `4.0.0`.
- Remove the beta feature flag check from the e2e testing with assertions feature. The `RASA_PRO_BETA_E2E_ASSERTIONS` environment variable is no longer needed as the feature is GA in 3.12.0.
- Deprecate the `custom` slot mapping type and its `action` slot mapping property which has been replaced with `run_action_every_turn` property name to retain backwards-compatible behavior.
- Deprecate the former list of dictionaries format for the `condition` key in a conditional response variation.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-5 'Direct link to Features')

- Add capability to use OAuth over Azure Entra ID for OpenAI instances deployed on Azure.
 
- Added Voice Stream Channel Connector for Genesys Cloud (AudioConnector Integration)
 
- Prevent unwanted digressions at collect flow steps by using one of the following new attributes available at both flow and collect step level:
 
 - `ask_confirm_digressions`: Asks the user to confirm if to continue with the current flow. Can be set to `true` or to a list of flow ids for which this behaviour should be activated.
 - `block_digressions`: Blocks any digression from the current flow and informs the user that they will return to the digression once the current flow is completed. Can be set to `true` or to a list of flow ids for which this behaviour should be activated.
 
 The above-mentioned behaviour is governed by a new pattern `pattern_handle_digressions` which is triggered only when the above attributes are used.
 
- Implement real-time validation of slot values.
 
 Add an optional property `validation` to slots to configure validations that should be run immediately. This property expects a list of `rejections` and a `refill_utter` which will be used to prompt users to provide a new value when validation fails.
 
 These validations are limited to common, reusable and universal checks that work independently of conversation context. For more complex validations that depend on conversation state, business logic, or external data, implement them in your flow definitions or custom actions instead.
 
- Introducing the `CompactLLMCommandGenerator` component, an enhancement over the `SingleStepLLMCommandGenerator`. This new component utilizes the highest-performing prompts for the models `gpt-4o-2024-11-20` and `claude-3-5-sonnet-20240620`.
 
 To incorporate the `CompactLLMCommandGenerator` into your pipeline, simply add the following:
 
 ```
    pipeline:  ...  - name: CompactLLMCommandGenerator  ...
    ```
 
- Multi-language support was implemented to enable the assistant to deliver localized responses and flow names that dynamically adjust to the user's language preference. In particular:
 
 - The default language is defined using the `language` key in `config.yml`, while additional supported languages are specified under `additional_languages`.
 - A `translation` section was introduced for responses to provide language-specific versions of the response text.
 - A `translation` section was added for flows to define localized flow names.
 - The rephraser prompt now accommodates the selected language.
 - Validation mechanisms were implemented to ensure proper use of translations; the CLI command `rasa validate data translations` is available for verification.
 - A new slot type, `StrictCategoricalSlot`, was developed to restrict its values to a predefined set.
 - A built-in `language` slot of the `StrictCategoricalSlot` type was added for managing translations effectively.
- \[beta\] Added Voice Stream channel connector to Audiocodes (audiocodes\_stream)
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-19 'Direct link to Improvements')

- Added validation to issue warnings for non-existent fixture/metadata names referenced in end-to-end tests.
 
- Implemented a fail fast mechanism that fails the end-to-end test case on the first failure, whether in user/bot turns or while using the assertions, to provide faster feedback.
 
- Added `utterance_end_ms` configuration to deepgram asr to handle noisy environments better
 
- Replace the optional dependency `mlflow` leveraged in the beta release of E2E testing with assertions when evaluating generative answers with custom prompts for each of the two generative metrics: `generative_response_is_relevant` and `generative_response_is_grounded`. This change now enables the usage of different LLM model providers and allows for a more flexible evaluation of generative components.
 
 Additionally, these generative assertions can make use of a new property `utter_source` (i.e. Enterprise Search, Contextual Rephraser or Intentless). This enables the assertion to be applied to a specific bot message source. It also for example prevents phrases such as `Is there anything else i can help you with?` triggered by `pattern_completed` to be checked for groundedness when the assertion should not be applied to it. Remove applying the same generative assertion to multiple bot messages in the same turn, however one bot message can be evaluated by multiple generative assertions in the same turn.
 
- Remove unnecessary deepcopy to improve performance in `undo_fallback_prediction` method of `FallbackClassifier`
 
- Add capability to control whether `pattern_completed` should execute when flow completes its execution. To control this behavior, a new parameter `run_pattern_completed` is added to the flow definition. By default this parameter is set to `True` which means `pattern_completed` will be executed when flow completes its execution (backward compatible). If this parameter is set to `False`, `pattern_completed` will not be executed when flow completes its execution.
 
- Allow slots to be filled by different slot extraction mechanisms (e.g. from\_llm, predefined NLU-based mappings, custom actions etc.). Add new `from_llm` slot mapping boolean property `allow_nlu_correction`(by default set to `False`), which gives permission for LLM-issued SetSlot commands to correct slots previously filled via NLU-based mechanisms.
 
 Allow LLM-based command generators to issue other commands after `NLUCommandAdapter` has issued commands. Introduce a new LLM-based command generator config property `minimize_num_calls` (by default set to `False`) which maintains backwards compatibility with previous behaviour where LLM-based command generators were blocked from invoking the LLM after `NLUCommandAdapter` had issued commands.
 
 Update the default utterance `utter_corrected_previous_input` to use a new context property `new_slot_values`.
 
- Set the default priority for StartFlow commands issued by different command generator types i.e. NLUCommandAdapter or LLM-based command generator: When the different command generators issue StartFlow commands for different flows in the same user turn, the NLUCommandAdapter will always take priority while the LLM-based start flow command will be discarded.
 
 Remove the limitation that the NLUCommandAdapter must always precede the LLM-based command generator in the config pipeline.
 
- Introduce a new slot mapping type `controlled` that can be assigned to slots that are set via button payloads, `set_slots` flow steps or custom actions.
 
 Slots that solely use the new `controlled` slot mapping will not be available to be filled probabilistically by the NLU or LLM components. Note that this slot mapping can still be used alongside the other slot mapping types, however this comes with the risk of the slot being filled by the NLU or LLM components in a probabilistic manner.
 
- Slots with mappings of type `controlled` (formerly the now deprecated `custom` mapping type) can be set at every turn without its custom action having to be called explicitly by the user flow.
 
 If you are building a coexistence assistant where different `controlled` slots are set by custom actions in different subsystems, you must indicate which coexistence system is allowed to fill the slot. This is done by setting the `coexistence_system` property in the slot mapping configuration. This property is a string that must match one of the available categorical values: `NLU`, `CALM`, `SHARED` (when either system can set the slot).
 
- Support user to send a preformatted message to the `invoke_llm` method. This let's the user switch between the `user` and `system` roles when invoking the LLM model.
 
- Add support for [`pypred`](https://github.com/armon/pypred) predicates in conditional response variations, similar to the usage of predicates in flows. The `condition` key in a response variation can now also be a string predicate that supports only the `slots` namespace. One of many logical operators supported is `not`.
 
- Make current slot type and its allowed values available for the prompt template rendering in `SingleStepLLMCommandGenerator` and `CompactLLMCommandGenerator` classes. Now you can use `{{ current_slot_type }}` and `{{ current_slot_allowed_values }}` placeholders in your custom prompt template.
 
- Made azure ASR `endpoint` and `host`, azure tts `endpoint` and cartesia tts `endpoint` configurable.
 
- Support usage of custom commands in the fine-tuning recipe.
 
- add support for `mstts` markups on azure TTS for improved SSML usage
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-125 'Direct link to Bugfixes')

- Add the possibility to pass a `transform` callable parameter when writing yaml. This allows passing a custom function to transform endpoints before uploading to Studio. This was required to fix the issue where yaml wraps in quotes any string that doesn't start with an alphabetic character such as unexpanded environment variables in the endpoints yml file.
- Fixed the accuracy calculation to prevent 100% assertion reporting when a test case fails before any assertions are reached.
- Fixed regression on training time for projects with a lot of YAML files.
- Fix AvailableEndpoints to read from the default `endpoints.yaml`, if no endpoint is specified.
- Update domain yaml schema for conditional response condition `type` key to specify valid enum type as `slot` only.
- - Fixed an issue where the `pattern_continue_interrupted` was not correctly triggered when the flow digressed to a step containing a link.
- Add the flow ID as a prefix to step ID to ensure uniqueness. This resolves a rare bug where steps in a child flow with a structure similar to those in a parent flow (using a "call" step) could result in duplicate step IDs. In this case duplicates previously caused incorrect next step selection.
- Enable default action `action_extract_slots` to set slots that should be shared for coexistence in a NLU-based system, when the same slot can be requested and filled by a flow in the CALM system too.
- Fixed conversation stalling in AudioCodes channel by handling activities in background tasks. Previously, activities were processed synchronously which blocked responses to AudioCodes, causing request timeouts and activity retries. These retries would cancel ongoing processing and get rejected as duplicates. Now activities are processed asynchronously while responding immediately to AudioCodes requests.
- Improved error handling for Deepgram and Cartesia connection failures to display more meaningful error messages when authentication fails or other connection issues occur.
- Modify Enterprise Search Citation Prompt Template to use `doc.text`
- Fixes ClarifyCommand syntax in the fine-tuning recipe.
- Fixed a bug that lead to the response to silence timeouts being cut off
- Fixed a bug in Voice Inspector where the tracker (hence the conversation transcript) was only updated after the prediction loop was complete. The bug resulted in a perceived delay in case of slow custom actions where transcript was rendered after processing the complete conversation turn. Now the tracker is sent to the Inspector app after every iteration of prediction loop, which conveys a more accurate conversation state and transcript on the inspector app
- Fix passing the incorrect input type (user question text instead of bot answer text) to the prompt used by the `generative_response_is_relevant` assertion. Add instructions to both relevance and groundedness prompts to not add any more explanations to the LLM output apart from the expected json output to prevent parsing errors.
- Make real-time validation work with all slot types. Fixes bug where the same `ValidateSlotPatternFlowStackFrame` was being triggered multiple times.
- Handle multiple duplicate digressing flows occurring within the same flow:
 - if `action_block_digressions` runs for a found duplicate digressing flow already on the stack, it will not add it again
 - if `action_continue_digressions` runs for a found duplicate digressing flow already on the stack, it first removes it from the stack before pushing it to the top of the stack.
- Do not push the clarification pattern when the top user frame is an interruption frame.
- Consider linked and called flows as active flows when processing `StartFlow` commands.
- Fixed slot value injection in translated responses by updating response keys to interpolate.
- Updated language code parsing to enforce BCP 47 standard.
- Fix validation check that ensures that the slot used in the response condition is defined in the domain file.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-31 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.11.19\] - 2025-08-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31119---2025-08-19 'Direct link to [3.11.19] - 2025-08-19')

Rasa Pro 3.11.19 (2025-08-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-126 'Direct link to Bugfixes')

- Fix correction of slots:
 
 - Slots can only be corrected in case they belong to any flow on the stack and the slot to be corrected is part of a collect step in any of those flows.
 - A correction of a slot should be applied if the flow that is about to start is using this slot.
- Don't tigger a slot correction for slots that are currently set to `None` as `None` counts as empty value.
 
- Fix issue where called flows could not set slots of their parent flow.
 

## \[3.11.18\] - 2025-07-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31118---2025-07-24 'Direct link to [3.11.18] - 2025-07-24')

Rasa Pro 3.11.18 (2025-07-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-127 'Direct link to Bugfixes')

- Fix issues with bot not giving feedback for slot corrections.
- Fix validation of the FAISS documents folder to ensure it correctly discovers files in a recursive directory structure.
- Updated `Inspector` dependent packages (`vite`, and `@adobe/css-tools`) to address security vulnerabilities.
- Upgrade `axios` to fix security vulnerability.
- Upgrade `litellm` to fix security vulnerability.

## \[3.11.17\] - 2025-07-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31117---2025-07-03 'Direct link to [3.11.17] - 2025-07-03')

Rasa Pro 3.11.17 (2025-07-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-128 'Direct link to Bugfixes')

- Flows now traverse called and linked flows, including nested and branching called / linked flows. As a result, E2E coverage reports include any linked and called flows triggered by the flow being tested.
- Fix issues where linked flows could not be cancelled and slots collected within linked flows could not be prefilled.
- Fix validation and improve error messages for the documents source directory when FAISS vector store is configured for the Enterprise Search Policy.
- Prevent slot correction when the slot is already set to the desired value.
- Fallback to `CannotHandle` command when the slot predicted by the LLM is not defined in the domain.
- Fix potential `KeyError` in `EnterpriseSearchPolicy` citation post-processing when the source does not have the correct citation. Improve the citation post-processing logic to handle additional edge cases.

## \[3.11.16\] - 2025-06-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31116---2025-06-12 'Direct link to [3.11.16] - 2025-06-12')

Rasa Pro 3.11.16 (2025-06-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-129 'Direct link to Bugfixes')

- Ensure that old step ID formats (without the flow ID prefix) can be loaded without raising an `InvalidFlowStepIdException` in newer Rasa versions that expect the flow ID prefix.
- Fix an issue where running `rasa inspect` would always set the `route_session_to_calm` slot to `True`, even when no user-triggered commands were present. This caused incorrect routing to CALM, bypassing the logic of the router.
- Enable slot prefilling in patterns.

## \[3.11.15\] - 2025-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31115---2025-06-03 'Direct link to [3.11.15] - 2025-06-03')

Rasa Pro 3.11.15 (2025-06-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-130 'Direct link to Bugfixes')

- Make `domain` an optional argument in `CommandProcessorComponent`. This change addresses a potential `TypeError` that could occur when loading a model trained without providing domain as a required argument. By making domain optional, models trained with older configurations or without a domain component will now load correctly without errors.

## \[3.11.14\] - 2025-06-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31114---2025-06-02 'Direct link to [3.11.14] - 2025-06-02')

Rasa Pro 3.11.14 (2025-06-02)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-20 'Direct link to Improvements')

- - Updates the parameter name from `max_tokens`, which is deprecated by OpenAI, to `max_completion_tokens`. The old `max_tokens` is not supported for the `o` models.
 - Exposes LiteLLM's `drop_params` parameter for LLM configurations. :

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-131 'Direct link to Bugfixes')

- The `Clarify` command now parses flow names with dashes.
- Add turn\_wrapper to count multiple utterances by bot/user as single turn rather than individual turns. Always include last user utterance in rephraser prompt's conversation history.
- Remove `StoryGraphProvider` from the prediction graph in case `IntentlessPolicy` is not present to reduce the loading time of the bot in cases where the `IntentlessPolicy` is not used and a lot of stories are present.
- - Fixes default config initialization for the `IntentlessPolicy`.
- Fix remote model download when models are stored in a path, not in the root of the remote storage. Add new training CLI param `--remote-root-only` that can be used by the model service to store the model in the root of the remote storage. Propagate this parameter to the persistor's `persist` method. Simplify persistor code when retrieving models by downloading the model to the target path directly rather than copying the downloaded model to the target path. This also improved testability.
- When inspector is not used, root server path should output: `Hello from Rasa: <version>.`. When inspector is used, root server path should output HTML page with a link to the path on which inspector can be reached.

## \[3.11.13\] - 2025-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31113---2025-05-15 'Direct link to [3.11.13] - 2025-05-15')

Rasa Pro 3.11.13 (2025-05-15)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-21 'Direct link to Improvements')

- Improved `CRFEntityExtractor` persistence and loading methods to improve model loading times.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-132 'Direct link to Bugfixes')

- Bumps aiohttp to 3.10.x, sentry-sdk to 2.8.x. Also bumps the locked versions for urllib3 and h11

## \[3.11.12\] - 2025-05-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31112---2025-05-14 'Direct link to [3.11.12] - 2025-05-14')

Rasa Pro 3.11.12 (2025-05-14)

No significant changes.

## \[3.11.11\] - 2025-05-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31111---2025-05-08 'Direct link to [3.11.11] - 2025-05-08')

Rasa Pro 3.11.11 (2025-05-08)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-133 'Direct link to Bugfixes')

- Fix issues in Audiocodes Channel Connector. The values in `user_phone` and `bot_phone` available in session\_started\_metadata are swapped to correctly map to `calller` and `callee` respectively. Fixed evening handling where events without the key "parameters" raised an exception.

## \[3.11.10\] - 2025-05-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31110---2025-05-06 'Direct link to [3.11.10] - 2025-05-06')

Rasa Pro 3.11.10 (2025-05-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-134 'Direct link to Bugfixes')

- Implemented backtracking and corrected the recursion logic in the all-paths generation for a flow. Removed the need to deepcopy the `step_ids_visited` set on each branch within `_handle_links`, which prevents the coverage report from freezing due to hitting the recursion limit.

## \[3.11.9\] - 2025-04-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3119---2025-04-30 'Direct link to [3.11.9] - 2025-04-30')

Rasa Pro 3.11.9 (2025-04-30)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-135 'Direct link to Bugfixes')

- Upgrade openai and litellm dependencies to fix found vulnerabilities in litellm.
- Add support for `auth_token` to Rasa Inspector.
- Fixed issue where a slot mapping referencing a form in the `active_loop` slot mapping conditions that wouldn't list this slot in the form's `required_slots` caused training to fail. Now, this is only logged as a validation warning without causing the training to fail.

## \[3.11.8\] - 2025-04-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3118---2025-04-28 'Direct link to [3.11.8] - 2025-04-28')

Rasa Pro 3.11.8 (2025-04-28)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-22 'Direct link to Improvements')

- Adds two optional properties on Jambonz channel connector, `username` and `password` which can be used to enable Basic Access Authentication
 
- Added support for basic authentication in Twilio channels (Voice Ready and Voice Streaming). This allows users to authenticate their Twilio channels using basic authentication credentials, enhancing security and access control for voice communication. To use this feature, set `username` and `password` in the Twilio channel configuration.
 
 credentials.yaml
 
 ```
    twilio_voice:    username: your_username    password: your_password    ...twilio_media_streams:    username: your_username    password: your_password    ...
    ```
 
 At Twilio, configure the webhook URL to include the basic authentication credentials:
 
 ```
    # twilio voice webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_voice/webhook# twilio media streams webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_media_streams/webhook
    ```
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-136 'Direct link to Bugfixes')

- Fixed a bug that lead to the response to silence timeouts being cut off
- Fail rasa commands (`rasa run`, `rasa inspect`, `rasa shell`) when model file path doesn't exist instead of defaulting to the latest model file from the default directory `/models`.
- Fix the behaviour of `action_hangup` on Voice Inspector (browser\_audio channel). Display that the session has ended
- Display a helpful error message in case of invalid API Key with Azure TTS
- Fixes Audiocodes Channel's event to intent mapping. All audiocode events are now mapped to an intent in the format `vaig_event_<event>`. That is, if Audiocodes sends an event `noUserInput`, the assistant will receive the intent `/vaig_event_noUserInput`

## \[3.11.7\] - 2025-04-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3117---2025-04-14 'Direct link to [3.11.7] - 2025-04-14')

Rasa Pro 3.11.7 (2025-04-14)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-137 'Direct link to Bugfixes')

- Fix `ChitChatAnswerCommand` command replaced with `CannotHandleCommand` if there are no e2e stories defined. Improve validation that `IntentlessPolicy` has applicable responses: either responses in the domain that are not part of any flow, or if there are e2e stories. This validation performed during the training time and during cleanup of the `ChitChatAnswerCommand` command.
- Security patch for Audiocodes channel connector. Audiocodes channel was only checking for the existence of authentication token. It now does a constant-time string comparison of the authentication token in connection request with that provided in channel configruation.
- Make sure training always fail when domain is invalid.
- Add channel name to UserMessage created by the Audiocodes channel.

## \[3.11.6\] - 2025-04-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3116---2025-04-02 'Direct link to [3.11.6] - 2025-04-02')

Rasa Pro 3.11.6 (2025-04-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-138 'Direct link to Bugfixes')

- Improved error handling for Deepgram and Cartesia connection failures to display more meaningful error messages when authentication fails or other connection issues occur.
- Modify Enterprise Search Citation Prompt Template to use `doc.text`
- Fixes ClarifyCommand syntax in the fine-tuning recipe.
- Send error event to the Kafka broker, when the original message size is too large, above the configured broker limit. This error handling mechanism was added to prevent Rasa-Pro server crashes.
- Update the following dependencies to versions that contain the latest patches for security vulnerabilities:
 - `jinja2`
 - `werkzeug`
 - `requests`
 - `cryptography`
 - `pyarrow`
 - `langchain`
 - `langchain-community`

## \[3.11.5\] - 2025-02-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3115---2025-02-18 'Direct link to [3.11.5] - 2025-02-18')

Rasa Pro 3.11.5 (2025-02-18)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-139 'Direct link to Bugfixes')

- Updated `Inspector` dependent packages (cross-spawn, mermaid, dom-purify, vite, braces, ws, axios and rollup) to address security vulnerabilities.
- Enable default action `action_extract_slots` to set slots that should be shared for coexistence in a NLU-based system, when the same slot can be requested and filled by a flow in the CALM system too.
- Fixed conversation stalling in AudioCodes channel by handling activities in background tasks. Previously, activities were processed synchronously which blocked responses to AudioCodes, causing request timeouts and activity retries. These retries would cancel ongoing processing and get rejected as duplicates. Now activities are processed asynchronously while responding immediately to AudioCodes requests.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-32 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.11.4\] - 2025-01-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3114---2025-01-30 'Direct link to [3.11.4] - 2025-01-30')

Rasa Pro 3.11.4 (2025-01-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-23 'Direct link to Improvements')

- Remove unnecessary deepcopy to improve performance in `undo_fallback_prediction` method of `FallbackClassifier`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-140 'Direct link to Bugfixes')

- - Fixed an issue where the `pattern_continue_interrupted` was not correctly triggered when the flow digressed to a step containing a link.
- Add the flow ID as a prefix to step ID to ensure uniqueness. This resolves a rare bug where steps in a child flow with a structure similar to those in a parent flow (using a "call" step) could result in duplicate step IDs. In this case duplicates previously caused incorrect next step selection.
- Optimized the `DirectCustomActionExecutor` by registering custom actions only once.
- Updated `cryptography` and `anyio` to resolve security vulnerabilities.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-33 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.11.3\] - 2025-01-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3113---2025-01-14 'Direct link to [3.11.3] - 2025-01-14')

Rasa Pro 3.11.3 (2025-01-14)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-24 'Direct link to Improvements')

- Enhances YAML parser to validate environment variable resolution for sensitive keys.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-141 'Direct link to Bugfixes')

- Add flow yaml validation when using the HTTP API `/model/train` endpoint. An invalid flow yaml will return a 400 response status code with a message describing the error.
- Make `pattern_session_start` work with `rasa inspector` to allow the assistant proactively start the conversation with a user.
- Fix writing the test cases obtained via the e2e test case conversion command to file, where `test_cases` key was written as a list item, instead of a dict key. This caused running the test cases to fail because it didn't comply with the e2e test schema. This PR fixes the issue by writing the test cases as a dict key.
- Fixed Inspector's Tracker State view not updating in real-time by moving story fetch logic into WebSocket message handler. Previously, story updates were only triggered on session ID changes, causing stale tracker state after the first conversation turn.
- Add pre-training custom validation to the domain responses that would raise a Rasa Pro validation error when a domain response is an empty sequence.
- Fixes a critical security vulnerability with `jsonpickle` dependency by upgrading to the patched version.
- Updated `pymilvus` and `minio` to address security vulnerability.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-34 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.11.2\] - 2024-12-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3112---2024-12-19 'Direct link to [3.11.2] - 2024-12-19')

Rasa Pro 3.11.2 (2024-12-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-142 'Direct link to Bugfixes')

- Validate that `api_type` key is only used for supported providers (Azure and OpenAI).
- Enable asserting events returned by `action_session_start` when running end-to-end testing with assertions format. The following assertions can be used:
 - `slot_was_set`
 - `slot_was_not_set`
 - `bot_uttered`
 - `bot_did_not_utter`
 - `action_executed`
- Fixed voice inspector to work with any URL by dynamically constructing WebSocket URL from current domain. This enables voice testing in GitHub Codespaces and other remote environments.
- - Fixed an error in `rasa llm finetune prepare-data` when using a subclass of `SingleStepLLMCommandGenerator`.
 - Resolved an issue where `rasa llm finetune prepare-data` did not support model groups.
- Fix AvailableEndpoints to read from the default `endpoints.yaml`, if no endpoint is specified.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-35 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.11.1\] - 2024-12-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3111---2024-12-13 'Direct link to [3.11.1] - 2024-12-13')

Rasa Pro 3.11.1 (2024-12-13)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-143 'Direct link to Bugfixes')

- Add the possibility to pass a `transform` callable parameter when writing yaml. This allows passing a custom function to transform endpoints before uploading to Studio. This was required to fix the issue where yaml wraps in quotes any string that doesn't start with an alphabetic character such as unexpanded environment variables in the endpoints yml file.
- Pass flow human-readable name instead of flow id when the cancel pattern stack frame is pushed during flow policy validation checks of collect steps.
- Fixed the accuracy calculation to prevent 100% assertion reporting when a test case fails before any assertions are reached.
- Fixed regression on training time for projects with a lot of YAML files.

## \[3.11.0\] - 2024-12-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3110---2024-12-11 'Direct link to [3.11.0] - 2024-12-11')

Rasa Pro 3.11.0 (2024-12-11)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-6 'Direct link to Deprecations and Removals')

- Removed `UnexpecTEDIntentPolicy` from the default config.yml. It is an experimental policy and not suitable for default configuration
- The `reset_after_flow_ends` property of collect steps is now deprecated and will be removed in Rasa Pro 4.0.0. Please use the `persisted_slots` property at the flow level instead.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-6 'Direct link to Features')

- Added Twilio Media Streams channel which can be configured to use arbitrary Text-To-Speech and Speech-To-Text services. Added Voice Stream Channel Interface which makes it easier to add voice channels that directly integrate with audio streams. Added support for Deepgram Speech-To-Text and Azure Text-To-Speech in Voice Stream Channels.
 
- Added default action `action_hangup` it can be used to hang up a phone call from a flow. Added `SessionEnded` event and `SessionEndCommand` command Updated Audiocodes, Jambonz and Twilio Voice channels to send `/session_end` if the phone call is disconnected by user.
 
- Added support for Cartesia Text-To-Speech in Voice Stream Channels.
 
- Implement Rasa Pro native model service that takes care of training and running an assistant model in Studio. To find out more about this service, read more in the Studio [documentation](https://rasa.com/docs/studio/deployment/architecture#studio-model-service-container).
 
- Added a feature to be able to use voice to interact with the bot in the inspector.
 
- Multi-LLM Routing:
 
 1. **Decoupled LLM Configuration from Components**
 
 - The previous integration of LLMs within CALM is closely tied to the components where they are used. However, this is no longer necessary, as we no longer perform training within the individual components that interact with external LLM endpoints.
 - As a result, LLM and embedding client configurations have been moved to `endpoints.yml`. To define LLM configurations in `endpoints.yml`, use the `model_groups` as shown below:
 
 ```
            model_groups:  - id: gpt-4-direct    models:      - provider: openai        model: gpt-4        timeout: 7        temperature: 0.0  - id: text-embedding-3-small-direct    models:      - provider: openai        model: text-embedding-3-small
            ```
 
 - These `model_groups` can then be referenced in `config.yml` as follows:
 
 ```
            pipeline:  ...  - name: SingleStepLLMCommandGenerator    llm:      model_group: gpt-4-direct    flow_retrieval:      embeddings:        model_group: text-embedding-3-smal-direct  ...
            ```
 
 2. **Support for Multiple Subscription Deployments**
 
 - Allows customers to use deployments from different subscriptions for the same provider.
 - Resolved the limitation of API key configuration being tied exclusively to a single environment variable.
 
 Example configuration in `endpoints.yml` for Azure deployments:
 
 ```
        model_groups:  - id: azure-gpt-model-eu    models:       - provider: azure         deployment: azure-eu-deployment         api_base: https://api.azure-europe.example.com         api_version: 2024-08-01-preview         api_key: ${AZURE_API_KEY_EU}         timeout: 7         temperature: 0.0         ...  - id: azure-gpt-model-us    models:       - provider: azure         deployment: azure-us-deployment         api_base: https://api.azure-us.example.com         api_version: 2024-08-01-preview         api_key: ${AZURE_API_KEY_US}         timeout: 7         temperature: 0.0         ...     ...
        ```
 
 3. **Seamless Model Configuration Across Environments Without Retraining**
 
 - Added support for using different model configurations in different environments, such as `dev`, `staging`, and `prod`, without requiring the bot to be retrained for each environment.
 - Extended the `${...}` syntax to `deployment`, `api_base`, and `api_version` in `model_groups`, allowing these values to change dynamically based on the environment.
 
 ```
        model_groups:- id: azure-gpt-4  models:    - provider: azure      deployment: ${AZURE_DEPLOYMENT_GPT4}      api_base: ${AZURE_API_BASE_GPT4}      api_key: ${AZURE_API_KEY_GPT4}      ...- id: azure-text-embeddings-3-small  models:    - provider: azure      deployment: ${AZURE_DEPLOYMENT_EMBEDDINGS_3_SMALL}      api_base: ${AZURE_API_BASE_EMBEDDINGS_3_SMALL}      api_key: ${AZURE_API_EMBEDDINGS_3_SMALL}      ...
        ```
 
 4. **Supporting Multiple Deployments for Load Balancing**
 
 - Enabled targeting of multiple LLM deployments for a single Rasa component.
 - Implemented the routing feature that supports load balancing to handle rate limits and improve scalability. When multiple models are defined within a model group, you can specify the `router` key with a `routing_strategy` to control how requests are distributed among the models.
 
 Example configuration in `endpoints.yml` for Azure deployments with load balancing:
 
 ```
        model_groups:  - id: azure-gpt-models     models:       - provider: azure         deployment: azure-eu-deployment         api_base: https://api.azure-europe.example.com         api_version: 2024-08-01-preview         api_key: ${AZURE_API_KEY_EU}         timeout: 7         temperature: 0.0         ...       - provider: azure         deployment: azure-us-deployment         api_base: https://api.azure-us.example.com         api_version: 2024-08-01-preview         api_key: ${AZURE_API_KEY_US}         timeout: 7         temperature: 0.0         ...     router:       routing_strategy: least-busy     ...
        ```
 
 Example of usage in `config.yml`:
 
 ```
        pipeline:...  - name: SingleStepLLMCommandGenerator    llm:      model_group: azure-gpt-models      ...
        ```
 
 5. **Backward Compatibility**
 
 - Existing configurations that couple LLMs to specific Rasa components remain unaffected by this change.
 - However, this configuration method is now deprecated and scheduled for removal in version 4.0.0.
- Added support for Azure Speech-To-Text in Voice Stream Channels.
 
- Added `UserSilenceCommand` and `pattern_user_silence` which is triggered by Voice Stream channels when the user is silent for more than a silence timeout. These values are configurable with the newly added slots `silence_timeout` and `consecutive_silence_timeouts`. Silence Monitoring is disabled by default and can be enabled using the configuration `monitor_silence: true` in the relevant Voice Stream Channel configuration.
 
- The inspector is not its own input / output channel anymore. Rather, it can be attached to other channels. This way, it isn't limited to conversations going through the socketio channel anymore, but can be used with other text channels or voice channels.
 
 You can attach it to any channel(s) configured in your credentials.yml by adding a flag to rasa run: rasa run --inspect.
 
 In addition to that, the conenience cli command rasa inspect is retained, which starts the inspector with the socketio channel as usual.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-25 'Direct link to Improvements')

- In Audiocodes channel, `/vaig_event_start` is replaced by `/session_start`. This intent marks the beginning of conversation and it is sent when the phone call is connected.
 
- Introduced the environment variable `MAX_NUMBER_OF_PREDICTIONS_CALM` to configure the CALM-specific limit for the number of predictions. This variable defaults to 1000, providing a higher prediction limit compared to the default value of 10 for nlu-based assistants.
 
- In Audiocodes and Twilio Voice channel connector, the call metadata received from the providers can be accessed in the slot `session_started_metadata`. The call metadata parameter names have been standardised with CallParameters dataclass Twilio Voice Channel Connector sends `/session_start` intent at the beginning of conversation and the channel parameter `initial_prompt` has been removed
 
- Enable configurability of Vault secret manager's mount point property in the endpoints yaml file or as an environment variable.
 
- In Twilio Media Streams channel connector, call metadata is availble in `session_start_metadata` slot. It also supports default action `action_hangup`
 
- Catch API connection errors, and validate the correctness of the values present in model configuration at model training time by making a test API request. This feature is enabled by default and can be disabled by setting the environment variable `LLM_API_HEALTH_CHECK` to `False`.
 
- `Socketio` channel connector now sends the websocket messages `tracker_state` and `rasa_events` with each bot response. `tracker_state` contains the tracker store state at that point in conversation and includes slots, events, stack, latest message and latest action. `rasa_events` contains a list of new events that have happened since the last message.
 
- Speech-To-Text and Text-To-Speech Services can be configured for Voice Stream Channel Connectors Added tests for voice components and redefined code structure
 
- Add support for Python 3.11
 
- Removed JSON response validation except when HTTP protocol and E2E Stub is used for Custom Action execution.
 
- Optimized JSON response validation by initializing the `Draft202012Validator` once and caching it.
 
- Add an optional property `persisted_slots` at the flow level. This property configures whether slots collected or set across any of the flow steps should be persisted after the flow ends. This property expects a list of slot names.
 
- Added support for custom Automatic Speech Recognition (ASR) or Text To Speech (TTS) providers to a Rasa Assistant. This allows developers to bring their own speech providers to Rasa by subclassing classes `ASREngine` and `TTSEngine`
 
- If flow retrieval is disabled, a warning is raised only if the number of user flows exceed 20.
 
- Added validation to the `TestCase` class to issue a warning when duplicate user messages lack metadata or have incorrect metadata. This enhancement provides clear guidance to users on the issue and how to resolve it.
 
- Fixed global `should-hangup` variable in Voice Stream Channels by moving to a context variable CallState that stores the session variables
 
- Run Rasa Pro data validation before uploading to Studio. This is to avoid uploading invalid assistant data that would raise errors during Rasa Pro model training in Studio.
 
- Added `vector_name` to Qdrant's configuration to enable customization of the vector field name for storing embeddings.
 
- Enhanced `YamlValidationException` error messages to include the line number and a relevant YAML snippet showing where the validation error occurred. Line numbers start from 1 (1-based indexing).
 
 The error-handling behavior has been modified so that only one validation error is displayed. This exception is raised when the YAML content does not comply with the defined YAML schema.
 
- Added a new assertion type `bot_did_not_utter` to allow testing that the bot does not utter specific messages or include certain buttons during conversations.
 
- Ensure that the model service fails properly if the minimum disk space requirement is not met.
 
- Do not expand environment variables when reading yaml files during `rasa studio upload` execution.
 
- Stream model files to Studio rather than providing full files. Provide a HEAD endpoint for Studio to check if a model is available and what its size is. Add an environment variable to set the port of the model service. This makes the development with Studio easier, previously the port was hard coded making it harder to use a separately deployed model service now that Studio includes that in its development deployment.
 
- Add flag `--skip-yaml-validation` to skip YAML validation during Rasa run. User can use it to skip domain YAML validation during Rasa run. Do not instantiate multiple instances of TrainingDataImporter class for validation and training.
 
- Introduced a `summarize_history` flag for the contextual response rephraser, defaulting to `True`. When set to `False`, the conversation transcript instead of the summary is included in the prompt of the contextual response rephraser. This saves a separate summarization call to an LLM. The number of conversation turns to be used when `summarize_history` is set to `False` can be set via `max_historical_turns`. By default this value is set to 5.
 
 Example:
 
 ```
    nlg:  - type: rephrase    summarize_history: False    max_historical_turns: 5
    ```
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-144 'Direct link to Bugfixes')

- Fix OpenAI LLM client ignoring API base and API version arguments if set.
 
- Fix `AttributeError` with the instrumentation of the `run` method of the `CustomActionExecutor` class.
 
- Throw DuplicatedFlowIdException during `rasa data validate` and `rasa train` if there are duplicate flows defined.
 
- Replace `pickle` and `joblib` with safer alternatives, e.g. `json`, `safetensors`, and `skops`, for serializing components.
 
 **Note**: This is a model breaking change. Please retrain your model.
 
 If you have a custom component that inherits from one of the components listed below and modified the `persist` or `load` method, make sure to update your code. Please contact us in case you encounter any problems.
 
 Affected components:
 
 - `CountVectorFeaturizer`
 - `LexicalSyntacticFeaturizer`
 - `LogisticRegressionClassifier`
 - `SklearnIntentClassifier`
 - `DIETClassifier`
 - `CRFEntityExtractor`
 - `TrackerFeaturizer`
 - `TEDPolicy`
 - `UnexpectedIntentTEDPolicy`
- Avoid filling slots that have `ask_before_filling = True` and utilize a `from_text` slot mapping during other steps in the flow. Ensure that the `NLUCommandAdapter` only fills these types of slots when the flow reaches the designated collection step.
 
- Check for the metadata's `step_id` and `active_flow` keys when adding the `ActionExecuted` event to the flows paths stack.
 
- Fixed a bug on Windows where flow files with names starting with 'u' would fail to load due to improper path escaping in YAML content processing
 
- Fixes OpenAIException - AsyncClient.**init**() got an unexpected keyword argument 'proxies'
 
- Fix retrieval of model file stored in the cloud storage by the model service. This change consisted in uploading only the model file instead of the full model path during training when `--remote-storage` CLI flag is used.
 
- Fix issue in e2e testing when customising `action_session_start` would lead to AttributeError, because the `output_channel` was not set. This is now fixed by setting the `output_channel` to `CollectingOutputChannel()`.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-36 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.27\] - 2025-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31027---2025-06-03 'Direct link to [3.10.27] - 2025-06-03')

Rasa Pro 3.10.27 (2025-06-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-145 'Direct link to Bugfixes')

- Make `domain` an optional argument in `CommandProcessorComponent`. This change addresses a potential `TypeError` that could occur when loading a model trained without providing domain as a required argument. By making domain optional, models trained with older configurations or without a domain component will now load correctly without errors.

## \[3.10.26\] - 2025-06-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31026---2025-06-02 'Direct link to [3.10.26] - 2025-06-02')

Rasa Pro 3.10.26 (2025-06-02)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-26 'Direct link to Improvements')

- - Updates the parameter name from `max_tokens`, which is deprecated by OpenAI, to `max_completion_tokens`. The old `max_tokens` is not supported for the `o` models.
 - Exposes LiteLLM's `drop_params` parameter for LLM configurations.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-146 'Direct link to Bugfixes')

- The `Clarify` command now parses flow names with dashes.
- Add turn\_wrapper to count multiple utterances by bot/user as single turn rather than individual turns. Always include last user utterance in rephraser prompt's conversation history.
- Remove `StoryGraphProvider` from the prediction graph in case `IntentlessPolicy` is not present to reduce the loading time of the bot in cases where the `IntentlessPolicy` is not used and a lot of stories are present.
- - Fixes default config initialization for the `IntentlessPolicy`.

## \[3.10.25\] - 2025-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31025---2025-05-15 'Direct link to [3.10.25] - 2025-05-15')

Rasa Pro 3.10.25 (2025-05-15)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-27 'Direct link to Improvements')

- Improved `CRFEntityExtractor` persistence and loading methods to improve model loading times.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-147 'Direct link to Bugfixes')

- Bumps aiohttp to 3.10.x, sentry-sdk to 2.8.x. Also bumps the locked versions for urllib3 and h11

## \[3.10.24\] - 2025-05-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31024---2025-05-14 'Direct link to [3.10.24] - 2025-05-14')

Rasa Pro 3.10.24 (2025-05-14)

No significant changes.

## \[3.10.23\] - 2025-05-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31023---2025-05-06 'Direct link to [3.10.23] - 2025-05-06')

Rasa Pro 3.10.23 (2025-05-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-148 'Direct link to Bugfixes')

- Implemented backtracking and corrected the recursion logic in the all-paths generation for a flow. Removed the need to deepcopy the `step_ids_visited` set on each branch within `_handle_links`, which prevents the coverage report from freezing due to hitting the recursion limit.

## \[3.10.22\] - 2025-04-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31022---2025-04-30 'Direct link to [3.10.22] - 2025-04-30')

Rasa Pro 3.10.22 (2025-04-30)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-149 'Direct link to Bugfixes')

- Add support for `auth_token` to Rasa Inspector.

## \[3.10.21\] - 2025-04-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31021---2025-04-29 'Direct link to [3.10.21] - 2025-04-29')

Rasa Pro 3.10.21 (2025-04-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-150 'Direct link to Bugfixes')

- Fixed issue where a slot mapping referencing a form in the `active_loop` slot mapping conditions that wouldn't list this slot in the form's `required_slots` caused training to fail. Now, this is only logged as a validation warning without causing the training to fail.

## \[3.10.20\] - 2025-04-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31020---2025-04-28 'Direct link to [3.10.20] - 2025-04-28')

Rasa Pro 3.10.20 (2025-04-28)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-28 'Direct link to Improvements')

- Added support for basic authentication in Twilio voice channel. This allows users to authenticate their Twilio voice channel using basic authentication credentials, enhancing security and access control for voice communication. To use this feature, set `username` and `password` in the Twilio channel configuration.
 
 credentials.yaml
 
 ```
    twilio_voice:    username: your_username    password: your_password    ...
    ```
 
 At Twilio, configure the webhook URL to include the basic authentication credentials:
 
 ```
    # twilio voice webhookhttps://<username>:<password>@yourdomain.com/webhooks/twilio_voice/webhook
    ```
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-151 'Direct link to Bugfixes')

- Fail rasa commands (`rasa run`, `rasa inspect`, `rasa shell`) when model file path doesn't exist instead of defaulting to the latest model file from the default directory `/models`.
- Upgrade openai and litellm dependencies to fix found vulnerabilities in litellm.

## \[3.10.19\] - 2025-04-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31019---2025-04-15 'Direct link to [3.10.19] - 2025-04-15')

Rasa Pro 3.10.19 (2025-04-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-152 'Direct link to Bugfixes')

- Fix `ChitChatAnswerCommand` command replaced with `CannotHandleCommand` if there are no e2e stories defined. Improve validation that `IntentlessPolicy` has applicable responses: either responses in the domain that are not part of any flow, or if there are e2e stories. This validation performed during the training time and during cleanup of the `ChitChatAnswerCommand` command.
- Security patch for Audiocodes channel connector. Audiocodes channel was only checking for the existence of authentication token. It now does a constant-time string comparison of the authentication token in connection request with that provided in channel configruation.
- Make sure training always fail when domain is invalid.
- Add channel name to UserMessage created by the Audiocodes channel.

## \[3.10.18\] - 2025-04-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31018---2025-04-02 'Direct link to [3.10.18] - 2025-04-02')

Rasa Pro 3.10.18 (2025-04-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-153 'Direct link to Bugfixes')

- Updated `Inspector` dependent packages (cross-spawn, mermaid, dom-purify, vite, braces, ws, axios and rollup) to address security vulnerabilities.
- Modify Enterprise Search Citation Prompt Template to use `doc.text`
- Fixes ClarifyCommand syntax in the fine-tuning recipe.
- Send error event to the Kafka broker, when the original message size is too large, above the configured broker limit. This error handling mechanism was added to prevent Rasa-Pro server crashes.
- Update the following dependencies to versions that contain the latest patches for security vulnerabilities:
 - `jinja2`
 - `werkzeug`
 - `requests`
 - `cryptography`
 - `pyarrow`
 - `langchain`
 - `langchain-community`

## \[3.10.17\] - 2025-01-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31017---2025-01-30 'Direct link to [3.10.17] - 2025-01-30')

Rasa Pro 3.10.17 (2025-01-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-29 'Direct link to Improvements')

- Remove unnecessary deepcopy to improve performance in `undo_fallback_prediction` method of `FallbackClassifier`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-154 'Direct link to Bugfixes')

- - Fixed an issue where the `pattern_continue_interrupted` was not correctly triggered when the flow digressed to a step containing a link.
- Add the flow ID as a prefix to step ID to ensure uniqueness. This resolves a rare bug where steps in a child flow with a structure similar to those in a parent flow (using a "call" step) could result in duplicate step IDs. In this case duplicates previously caused incorrect next step selection.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-37 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.16\] - 2025-01-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31016---2025-01-15 'Direct link to [3.10.16] - 2025-01-15')

Rasa Pro 3.10.16 (2025-01-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-155 'Direct link to Bugfixes')

- - Fixed an error in `rasa llm finetune prepare-data` when using a subclass of `SingleStepLLMCommandGenerator`.
- Make `pattern_session_start` work with `rasa inspector` to allow the assistant proactively start the conversation with a user.
- Fix writing the test cases obtained via the e2e test case conversion command to file, where `test_cases` key was written as a list item, instead of a dict key. This caused running the test cases to fail because it didn't comply with the e2e test schema. This PR fixes the issue by writing the test cases as a dict key.
- Add pre-training custom validation to the domain responses that would raise a Rasa Pro validation error when a domain response is an empty sequence.
- Fixes a critical security vulnerability with `jsonpickle` dependency by upgrading to the patched version.
- Updated `pymilvus` and `minio` to address security vulnerability.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-38 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.15\] - 2024-12-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31015---2024-12-18 'Direct link to [3.10.15] - 2024-12-18')

Rasa Pro 3.10.15 (2024-12-18)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-156 'Direct link to Bugfixes')

- Validate that `api_type` key is only used for supported providers (Azure and OpenAI).
- Fix issue in e2e testing when customising `action_session_start` would lead to AttributeError, because the `output_channel` was not set. This is now fixed by setting the `output_channel` to `CollectingOutputChannel()`.
- Fixed the accuracy calculation to prevent 100% assertion reporting when a test case fails before any assertions are reached.
- Pass flow human-readable name instead of flow id when the cancel pattern stack frame is pushed during flow policy validation checks of collect steps.
- Try to instantiate LLM/embeddings client when loading component to validate environment variables.
- Enable asserting events returned by `action_session_start` when running end-to-end testing with assertions format. The following assertions can be used:
 - `slot_was_set`
 - `slot_was_not_set`
 - `bot_uttered`
 - `action_executed`

## \[3.10.14\] - 2024-12-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31014---2024-12-04 'Direct link to [3.10.14] - 2024-12-04')

Rasa Pro 3.10.14 (2024-12-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-157 'Direct link to Bugfixes')

- Avoid filling slots that have `ask_before_filling = True` and utilize a `from_text` slot mapping during other steps in the flow. Ensure that the `NLUCommandAdapter` only fills these types of slots when the flow reaches the designated collection step.
- Fixes OpenAIException - AsyncClient.**init**() got an unexpected keyword argument 'proxies'
- Fix validation for LLM/Embedding clients when the api\_base is configured in the config itself but not as an environment variable.

## \[3.10.13\] - 2024-11-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31013---2024-11-29 'Direct link to [3.10.13] - 2024-11-29')

Rasa Pro 3.10.13 (2024-11-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-158 'Direct link to Bugfixes')

- Implement `eq` and `hash` functions for `ChangeFlowCommand` to fix `error=unhashable type: 'ChangeFlowCommand'` error in `MultiStepCommandGenerator`.
- Fixed an issue on Windows where flow files with names starting with 'u' would fail to load due to improper path escaping in YAML content processing
- Store the value of the `--disable-verify` CLI flag in the `disable_verify` attribute of the `StudioConfig` object, so it can be reused across other studio commands.

## \[3.10.12\] - 2024-11-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31012---2024-11-25 'Direct link to [3.10.12] - 2024-11-25')

Rasa Pro 3.10.12 (2024-11-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-159 'Direct link to Bugfixes')

- Replace `pickle` and `joblib` with safer alternatives, e.g. `json`, `safetensors`, and `skops`, for serializing components.
 
 **Note**: This is a model breaking change. Please retrain your model.
 
 If you have a custom component that inherits from one of the components listed below and modified the `persist` or `load` method, make sure to update your code. Please contact us in case you encounter any problems.
 
 Affected components:
 
 - `CountVectorFeaturizer`
 - `LexicalSyntacticFeaturizer`
 - `LogisticRegressionClassifier`
 - `SklearnIntentClassifier`
 - `DIETClassifier`
 - `CRFEntityExtractor`
 - `TrackerFeaturizer`
 - `TEDPolicy`
 - `UnexpectedIntentTEDPolicy`

## \[3.10.11\] - 2024-11-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31011---2024-11-20 'Direct link to [3.10.11] - 2024-11-20')

Rasa Pro 3.10.11 (2024-11-20)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-160 'Direct link to Bugfixes')

- Fix parsing of commands in case the LLM response surrounds flow names, slot names, or slot values with single or double quotes.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-39 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.10\] - 2024-11-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31010---2024-11-14 'Direct link to [3.10.10] - 2024-11-14')

Rasa Pro 3.10.10 (2024-11-14)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-161 'Direct link to Bugfixes')

- Check for the metadata's `step_id` and `active_flow` keys when adding the `ActionExecuted` event to the flows paths stack.

## \[3.10.9\] - 2024-11-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3109---2024-11-13 'Direct link to [3.10.9] - 2024-11-13')

Rasa Pro 3.10.9 (2024-11-13)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-162 'Direct link to Bugfixes')

- Introduced the environment variable `MAX_NUMBER_OF_PREDICTIONS_CALM` to configure the CALM-specific limit for the number of predictions. This variable defaults to 1000, providing a higher prediction limit compared to the default value of 10 for nlu-based assistants.
- Filter out comments from e2e test input files when writing e2e results to file.
- Specified UTF-8 encoding to correctly read test cases on Windows.

## \[3.10.8\] - 2024-10-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3108---2024-10-24 'Direct link to [3.10.8] - 2024-10-24')

Rasa Pro 3.10.8 (2024-10-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-163 'Direct link to Bugfixes')

- The user message "/restart" is now restarting the session again after adding a proper implementation (stack frame and command) for `pattern_restart`.
- Only infer and set the provider to `azure` for our LLM clients in case NO `provider` is specified, but the `deployment` key is set.
- Fix OPENAI\_API\_KEY authentication error when using self-hosted provider.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-40 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.7\] - 2024-10-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3107---2024-10-17 'Direct link to [3.10.7] - 2024-10-17')

Rasa Pro 3.10.7 (2024-10-17)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-30 'Direct link to Improvements')

- Change default response of `utter_free_chitchat_response` from `"placeholder_this_utterance_needs_the_rephraser"` to `"Sorry, I'm not able to answer that right now."`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-164 'Direct link to Bugfixes')

- Disallow using the command payload syntax to set slots not filled by any of the active or startable flow(s) `collect` steps.
- Add flow name to error message `validator.verify_flows_steps_against_domain.collect_step`.
- Update e2e test results output files on each test run so that, for example, when all tests pass on subsequent runs after failing previously, the failed results output file is emptied.
- Disable strict SSL verification to the Rasa Studio authentication server via the `--disable-verify` or `-x` CLI argument added to the `rasa studio config` command.
- Upgrade `zipp` dependency version to fix a security vulnerability: [CVE-2024-5569](https://github.com/advisories/GHSA-jfmj-5v4g-7637).

## \[3.10.6\] - 2024-10-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3106---2024-10-04 'Direct link to [3.10.6] - 2024-10-04')

Rasa Pro 3.10.6 (2024-10-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-165 'Direct link to Bugfixes')

- Fix cleanup of `SetSlot` commands issued by the LLM-based command generator for slots that define a slot mapping other than the `from_llm` slot mapping. The command processor now correctly removes the SetSlot command in these scenarios and instead adds a `CannotHandleCommand`.
- Fix `UnicodeDecodeError` while reading Windows path from yaml files.
- Fix model loading from remote storage by correcting the handling of remote storage enum during the creation of the persistor object.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-41 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.5\] - 2024-10-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3105---2024-10-01 'Direct link to [3.10.5] - 2024-10-01')

Rasa Pro 3.10.5 (2024-10-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-166 'Direct link to Bugfixes')

- Fix the case where IntentlessPolicy is triggered while no e2e stories were written to guide it. In this situation a CannotHandleCommand will be issued.
 
- Update litellm to version 1.45.0 to fix security vulnerability (CVE-2024-6587). Update gitpython to version 3.1.41 to fix security vulnerability (CVE-2024-22190). Update certifi to version 2024.07.04 to fix security vulnerability (CVE-2024-39689).
 
- Prevent invalid domain with incorrectly defined intent from throwing stack trace. Throw InvalidDomain exception and send message to the user instead. The message looks like this:
 
 ```
    Detected invalid intent definition: {'intent': 'ask_help'}.Please make sure all intent definitions are valid.
    ```
 
- Support text completions endpoint when using self hosted models.
 
 The `use_chat_completions_endpoint` parameter is now supported when using self-hosted models. This parameter is used to enable the use of the chat completions endpoint when using a self-hosted model. This parameter is set to `True` by default. To use the text completions endpoint, set `use_chat_completions_endpoint` to `False` in the `llm` section of the component.
 
 Usage:
 
 ```
    llm:    provider: self-hosted    model: meta-llama/Meta-Llama-3-8B    api_base: "https://my-endpoint/v1"    use_chat_completions_endpoint: false
    ```
 
- Fixes an issue where the `CountVectorsFeaturizer` and `LogisticRegressionClassifier` would throw error during inference when no NLU training data is provided.
 
- Added tracing explicitly to `GRPCCustomActionExecutor.run` in order to pass the tracing context to the action server.
 

## \[3.10.4\] - 2024-09-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3104---2024-09-25 'Direct link to [3.10.4] - 2024-09-25')

Rasa Pro 3.10.4 (2024-09-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-167 'Direct link to Bugfixes')

- Fix failing validation of categorical slots when slot values contain Apostrophe.

## \[3.10.3\] - 2024-09-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3103---2024-09-20 'Direct link to [3.10.3] - 2024-09-20')

Rasa Pro 3.10.3 (2024-09-20)

No significant changes.

## \[3.10.2\] - 2024-09-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3102---2024-09-19 'Direct link to [3.10.2] - 2024-09-19')

Rasa Pro 3.10.2 (2024-09-19)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-7 'Direct link to Deprecations and Removals')

- Dropped support for Python 3.8 ahead of [Python 3.8 End of Life in October 2024](https://devguide.python.org/versions/#supported-versions). In Rasa Pro versions 3.10.0, 3.9.11 and 3.8.13, we needed to pin the TensorFlow library version to 2.13.0rc1 in order to remove critical vulnerabilities; this resulted in poor user experience when installing these versions of Rasa Pro with `uv pip`. Removing support for Python 3.8 will make it possible to upgrade to a stabler version of TensorFlow.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-31 'Direct link to Improvements')

- Update Keras and Tensorflow to version 2.14. This will eliminate the need to use the `--prerelease allow` flag when installing Rasa Pro using `uv pip` tool.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-168 'Direct link to Bugfixes')

- Revert the old behavior when loading trained model by supplying a path to the model on the remote storage by using the model path (`-m`) argument when `REMOTE_STORAGE_PATH` environment variable is not set. Resulting path on the remote storage will be the same as the model path (`-m`) argument.
 
 Additionally, entire model path (`-m`) argument wil be used when trained model is being uploaded to the remote storage with `REMOTE_STORAGE_PATH` environment variable not set. Resulting path on the remote storage will be the same as the model path (`-m`) argument.
 
 If `REMOTE_STORAGE_PATH` environment variable is set, only the file name part of the model path (`-m`) argument is used in both loading and storage from/to the remote storage. Resulting path on the remote storage will be: `REMOTE_STORAGE_PATH` + file name part of the model path (`-m`) argument.
 
- Fixed UnexpecTEDIntentlessPolicy training errors that resulted from a change to batching behavior. Changed the batching behavior back to the original for all components. Made the changed batching behavior accessible in DietClassifier using `drop_small_last_batch: True`.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-42 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.10.1\] - 2024-09-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3101---2024-09-11 'Direct link to [3.10.1] - 2024-09-11')

Rasa Pro 3.10.1 (2024-09-11)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-169 'Direct link to Bugfixes')

- Fix OpenAI LLM client ignoring API base and API version arguments if set.
- Fix `FileNotFound` error when running `rasa studio` commands and no pre-existing local assistant project exists.
- Fixed telemetry collection for the components Rephraser, LLM Intent Classifier, Intentless Policy and Enterprise Search Policy to ensure that the telemetry data is only collected when it is enabled
- Update the default config for E2E test conversion to use the `provider` key instead of `api_type`.
- Fix inconsistent recording of telemetry events for llm-based command generators.
- Throw deprecation warning when REQUESTS\_CA\_BUNDLE env var is used.

## \[3.10.0\] - 2024-09-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3100---2024-09-04 'Direct link to [3.10.0] - 2024-09-04')

Rasa Pro 3.10.0 (2024-09-04)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-8 'Direct link to Deprecations and Removals')

- Remove experimental `LLMIntentClassifier`. Use Rasa CALM instead.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-7 'Direct link to Features')

- Implement the shell output of [accuracy rate by assertion type](https://rasa.com/docs/rasa-pro/testing/e2e-testing-assertions/assertions-how-to-guide#test-results-analysis) as a table when running end-to-end testing with assertions.
 
- Implement E2E testing assertions that measure metrics such as grounded-ness and answer relevance of generative responses issued by either Enterprise Search or the Contextual Response Rephraser.
 
 You must specify a threshold which must be reached for the generative evaluation assertion to pass. In addition, you can also specify `ground_truth` if you prefer providing this in the E2E test rather than relying on the retrieved context from the vector store (in the case of Enterprise Search) or from the domain (in the case of Contextual Response Rephraser) that is stored in the bot utterance event metadata. For rephrased answers, you must specify `utter_name` to run the assertion.
 
 These assertions can be specified for user steps only and cannot be used alongside the former E2E test format. You can learn more about this new feature in the documentation sections for [grounded](https://rasa.com/docs/rasa-pro/testing/e2e-testing-assertions/assertions-fundamentals#generative-response-is-grounded-assertion) and [relevant](https://rasa.com/docs/rasa-pro/testing/e2e-testing-assertions/assertions-fundamentals#generative-response-is-relevant-assertion) assertion types.
 
 To enable this feature, please set the environment variable `RASA_PRO_BETA_E2E_ASSERTIONS` to `true`.
 
 ```
    export RASA_PRO_BETA_E2E_ASSERTIONS=true
    ```
 
- You can now produce a coverage report of your e2e tests via the following command:
 
 ```
    rasa test e2e <e2e-test-folder> --coverage-report [--coverage-output-path <output-folder>]
    ```
 
 The coverage report contains the number of steps and the number of tested steps per flow. Untested steps are referenced by line numbers.
 
 ```
    Flow Name Coverage  Num Steps  Missing Steps  Line Numbers   flow_1    0.00%          1              1       [10-10]   flow_2  100.00%          4              0            []    Total   80.00%          5              1
    ```
 
 Additionally, we also create a histogram of command coverage showing how many and what commands are produced in your e2e tests.
 
 To enable this feature, please set the environment variable `RASA_PRO_BETA_FINETUNING_RECIPE` to `true`.
 
 ```
    export RASA_PRO_BETA_FINETUNING_RECIPE=true
    ```
 
 More information can be found on the [documentation](https://rasa.com/docs/rasa-pro/production/testing-your-assistant#e2e-test-coverage-report) of the feature.
 
- Create a self-hosted LLM client compatible with OpenAI format. Users can connect to their own self-hosted LLM server that is compatible with OpenAI format.
 
 Sample basic usage:
 
 ```
        llm:        provider: self-hosted        model: <deployment_name>        api_base: <deployment_url>        api_type: openai [Optional]
    ```
 
- Add a new [CLI command](https://rasa.com/docs/rasa-pro/command-line-interface#rasa-llm-finetune-prepare-data) `rasa llm finetune prepare-data` to create a dataset from e2e tests that can be used to fine-tune a base model for the task of command generation.
 
 To enable this feature, please set the environment variable `RASA_PRO_BETA_FINETUNING_RECIPE` to `true`.
 
 ```
    export RASA_PRO_BETA_FINETUNING_RECIPE=true
    ```
 
- It is now allowed to link to `pattern_human_handoff` from any pattern and user flow.
 
- Allow links from all patterns to user flows except for `pattern_internal_error`.
 
- - **LiteLLM Integration & Reduced LangChain Reliance:**
 - Introduced `LLMClient` and `EmbeddingClient` protocols for standardized client interfaces.
 - Created lightweight client wrappers for LiteLLM to streamline model instantiation, management, and inference.
 - Updated `llm_factory` and `embedder_factory` to utilize these LiteLLM client wrappers.
 - Added dedicated clients for Azure OpenAI and OpenAI to support both LLMs and embedding models.
 - Added a HuggingFace client to compute embeddings using locally stored transformer models via the `sentence-transformers` package.
 - **LangChain Update:** Upgraded to the latest version (0.2.x) for improved compatibility and features. To understand the implications on your assistant, please refer to the [feature documentation](https://rasa.com/docs/rasa-pro/concepts/components/llm-configuration) and the [migration guide](https://rasa.com/docs/rasa-pro/migration-guide#rasa-pro-39-to-rasa-pro-310).
- Implement as part of E2E testing a new type of evaluation specifically designed to increase confidence in CALM. This evaluation runs assertions on the assistant's actual events and generative responses. New assertions include the ability to check for the presence of specific events, such as:
 
 - flow started, flow completed or flow cancelled events
 - whether `pattern_clarification` was triggered for specific flows
 - whether buttons rendered well as part of the bot uttered event
 - whether slots were set correctly or not
 - whether the bot text response matches a provided regex pattern
 - whether the bot response matches a provided domain response name
 
 These assertions can be specified for user steps only and cannot be used alongside the former E2E test format. You can learn more about this new feature in the [documentation](https://rasa.com/docs/rasa-pro/testing/e2e-testing-assertions/assertions-introduction).
 
 To enable this feature, please set the environment variable `RASA_PRO_BETA_E2E_ASSERTIONS` to `true`.
 
 ```
    export RASA_PRO_BETA_E2E_ASSERTIONS=true
    ```
 
- Configure [LLM-as-Judge settings](https://rasa.com/docs/rasa-pro/testing/e2e-testing-assertions/assertions-installation#generative-response-llm-judge-configuration) in the `llm_as_judge` section of the `conftest.yml` file. These settings will be used to evaluate the groundedness and relevance of generated bot responses. The `conftest.yml` is discoverable as long as it is in the root directory of the assistant project, at the same level as the `config.yml` file.
 
 If the `conftest.yml` file is not present in the root directory, the default LLM judge settings will be used.
 
- Implement automatic E2E test case conversion from sample conversation data.
 
 This feature includes:
 
 - A CLI command to convert sample conversation data (CSV, XLSX) into executable E2E test cases.
 - Conversion of sample data using an LLM to generate YAML formatted test cases.
 - Export of generated test cases into a specified YAML file.
 
 Usage:
 
 ```
    rasa data convert e2e <path>
    ```
 
 To enable this feature, please set the environment variable `RASA_PRO_BETA_E2E_CONVERSION` to `true`.
 
 ```
    export RASA_PRO_BETA_E2E_CONVERSION=true
    ```
 
 For more details, please refer to this [documentation page](https://rasa.com/docs/rasa-pro/testing/e2e-test-conversion).
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-32 'Direct link to Improvements')

- Implemented custom action stubbing for E2E test cases. To define custom action stubs, add `stub_custom_actions` to the test case file.
 
 Stubs can be defined in two ways:
 
 - Test file level: Define each action by its name (`action_name`).
 - Test case level: Define the stub using the test case ID as a prefix (`test_case_id::action_name`).
 
 To learn more about this feature, please refer to the [documentation](https://rasa.com/docs/rasa-pro/production/testing-your-assistant#stubbing-custom-actions).
 
 To enable this feature, set the environment variable `RASA_PRO_BETA_STUB_CUSTOM_ACTION` to `true`:
 
 ```
    export RASA_PRO_BETA_STUB_CUSTOM_ACTION=true
    ```
 
- Add `max_messages_in_query` parameter to Enterprise Search Policy, it allows controlling the number of past messages that are used in the search query for retrieval
 
- Configure [LLM E2E test converter settings](https://rasa.com/docs/rasa-pro/testing/e2e-test-conversion#llm-configuration) in the `llm_e2e_test_conversion` section of the `conftest.yml` file.
 
 These settings will be used to configure the LLM used to convert sample conversation data into E2E test cases.
 
 The `conftest.yml` is discoverable as long as it is in the root directory of the tests output path.
 
 If the `conftest.yml` file is not present in the root directory, the default LLM settings will be used.
 
- Add the datetime of Rasa Pro license expiry to `rasa --version` command Add `/license` API endpoint that also returns the same information
 
- Suppress LiteLLM info and debug log messages in the console.
 
- Cache llm\_factory and embedder\_factory methods to avoid client instantiation and validation for every user utterance.
 
- Added E2E Test Conversion Completed telemetry event with file type and test case count properties.
 
- Separate writing of failed and passed e2e test results to distinct file paths.
 
- Implement support for evaluating IntentlessPolicy responses with generative response assertions.
 
- Use direct custom action execution in tutorial and CALM templates. Skip action server health check in e2e testing if direct custom action execution is configured.
 
- Modified the type of flows which are included into the import CLI (previously only user flows were enabled, now patterns are included). Use case: This is needed for Studio 1.7, since that release is enabling modification and management of patterns inside Studio, and needs the ability to import patterns from yaml files.
 
- Improve events and responses sub-schemas used by the `stub_custom_actions` sub-schema of end-to-end testing. The events sub-schema only allows the usage of events which are supported by the `rasa-sdk`. These are documented in the [action server API documentation](https://rasa.com/docs/reference/api/pro/action-server-api/).
 
- Change default model of conversation rephraser to 'gpt-4o-mini'.
 
- Add `file_path` to `Flow` so that we can show the full name, e.g. `path/to/flow.py::flow name` in the e2e test coverage report.
 
- Introduced remote storage to upload trained model to persistors(AWS, GCP, Azure)
 
- Add ability to download training data from remote storage(gcs, aws, azure)
 
- Allow saving models to and retrieving from sub folders in cloud storage.
 
- Introduced `DirectCustomActionExecutor` for executing custom actions directly through the assistant.
 
 Introduced `actions_module` variable under `action_endpoint` in `endpoints.yml` to explicitly specify the path to custom actions module.
 
 If `actions_module` is set, custom actions will be executed directly through the assistant.
 
- Add validation for the values against which categorical and boolean slots are checked in the if conditional steps. An error will be thrown when a slot is compared to an invalid/non-existent value for boolean and categorical slots.
 
- Add user query and retrieved document results to the metadata of `action_send_text` predicted by EnterpriseSearchPolicy. In addition, add domain ground truth responses to the `BotUttered` event metadata when rephrasing is enabled. These changes were required to allow evaluations of generative responses against the ground truth stored in the metadata of `BotUttered` events.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-170 'Direct link to Bugfixes')

- Fix problem with custom action invocation when model is loaded from remote storage.
 
- Ensure certificates for openai based clients.
 
- Mark the first slot event as seen when the user turn in a E2E test case contains multiple slot events for the same slot. This fixes the issue when the `assertion_order_enabled` is set to `true` and the user step in a test case contained multiple `slot_was_set` assertions for the same slot, the last slot event was marked as seen when the first assertion was running. This caused the test to fail for subsequent `slot_was_set` assertions for the same slot with error `Slot <slot_name> was not set`.
 
- Validate the LLM configuration during training for the following components:
 
 - `Contextual Response Rephraser`
 - `Enterprise Search Policy`
 - `Intentless Policy`
 - `LLM Based Command Generator`
 - `LLM Based Router`
 
 Additionally, update the `get_provider_from_config` method to retrieve the provider using both the `model` and `model_name` configuration parameters.
 
- Fixes throwing the deprecation warning if the setting for Azure OpenAI Embedding Client was not set through the deprecated environment variable.
 
- Fix execution of stub custom actions when they contain test case name and the separator in its provided stub name. Test runner will now correctly execute the correct stub implementation for the same custom action dependent on the test name.
 
- Add validation to conversation rephraser.
 
- Ensure YAML files with datetime-formatted strings are read as plain strings instead of being converted to datetime objects.
 
- Deprecate 'request\_timeout' for OpenAI and Azure OpenAI clients in favor of 'timeout'
 
- Forbid `stream` and `n` parameters for clients. Having these parameters within `llm` and `embeddings` configuration will result in error.
 
- Raise deprecation warning if `api_type` is set to `huggingface` instead of `huggingface_local` for HuggingFace local embeddings.
 
- Fix resolving aliases for deprecated keys when instantiating LLM and embedding clients.
 
- Fix detection of conftest file which contained custom LLM judge configuration.
 
- Fix issue with Rasa Pro Studio download command exporting default flows which had not been customized by the Studio user. Rasa Pro Studio download command only exports user defined flows, customized patterns and user defined domain locally from the Studio instance.
 
 Similarly, fix issue with Rasa Pro Studio upload command importing default flows which had not been customized to Studio. Rasa Pro Studio upload command only imports user defined flows, customized patterns and user defined domain to the Studio instance.
 
- Disable auto-inferring provider from the config. Ensure the provider is explicitly read from the `provider` key.
 
- Fix writing e2e test cases to disk. `slot_was_set` and `slot_was_not_set` are now written down correctly.
 
- The rephraser of the `rasa llm finetune data-prepare` command now compares the original user message and the user message returned in the LLM output case-insensitive.
 
- \[rasa llm finetune prepare-data\] Do not rephrase user messages that come from a button payload.
 
- Separate commands in the expected LLM output by newlines.
 
- Fix TypeError in PatternClarificationContainsAssertion hash function by converting sets to lists for successful JSON serialization.
 
- Fix validation in case a link to `pattern_human_handoff` is used.
 
- \[`rasa llm finetune prepare-data`\] Skip paraphrasing module in case `num-rephrases` is set to 0.
 
- Update the handling of incorrect use of slash syntax. Messages with undefined intents do not automatically trigger `pattern_cannot_handle`; instead, they are sanitized (prepended slash(es) are removed) and passed through the graph.
 
- Allow suitable patterns to be properly started using nlu triggers
 
- Fix API connection error for bedrock embedding endpoint.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-43 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.20\] - 2025-04-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3920---2025-04-14 'Direct link to [3.9.20] - 2025-04-14')

Rasa Pro 3.9.20 (2025-04-14)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-171 'Direct link to Bugfixes')

- Updated `Inspector` dependent packages (cross-spawn, mermaid, dom-purify, vite, braces, ws, axios and rollup) to address security vulnerabilities.
- Modify Enterprise Search Citation Prompt Template to use `doc.text`
- Security patch for Audiocodes channel connector. Audiocodes channel was only checking for the existence of authentication token. It now does a constant-time string comparison of the authentication token in connection request with that provided in channel configruation.

## \[3.9.19\] - 2025-01-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3919---2025-01-30 'Direct link to [3.9.19] - 2025-01-30')

Rasa Pro 3.9.19 (2025-01-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-33 'Direct link to Improvements')

- Remove unnecessary deepcopy to improve performance in `undo_fallback_prediction` method of `FallbackClassifier`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-172 'Direct link to Bugfixes')

- - Fixed an issue where the `pattern_continue_interrupted` was not correctly triggered when the flow digressed to a step containing a link.
- Add the flow ID as a prefix to step ID to ensure uniqueness. This resolves a rare bug where steps in a child flow with a structure similar to those in a parent flow (using a "call" step) could result in duplicate step IDs. In this case duplicates previously caused incorrect next step selection.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-44 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.18\] - 2025-01-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3918---2025-01-15 'Direct link to [3.9.18] - 2025-01-15')

Rasa Pro 3.9.18 (2025-01-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-173 'Direct link to Bugfixes')

- Fix issue in e2e testing when customising `action_session_start` would lead to AttributeError, because the `output_channel` was not set. This is now fixed by setting the `output_channel` to `CollectingOutputChannel()`.
- Pass flow human-readable name instead of flow id when the cancel pattern stack frame is pushed during flow policy validation checks of collect steps.
- Make `pattern_session_start` work with `rasa inspector` to allow the assistant proactively start the conversation with a user.
- Fixes a critical security vulnerability with `jsonpickle` dependency by upgrading to the patched version.
- Updated `minio` to address security vulnerability.

## \[3.9.17\] - 2024-12-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3917---2024-12-05 'Direct link to [3.9.17] - 2024-12-05')

Rasa Pro 3.9.17 (2024-12-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-174 'Direct link to Bugfixes')

- Implement `eq` and `hash` functions for `ChangeFlowCommand` to fix `error=unhashable type: 'ChangeFlowCommand'` error in `MultiStepCommandGenerator`.

## \[3.9.16\] - 2024-11-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3916---2024-11-26 'Direct link to [3.9.16] - 2024-11-26')

Rasa Pro 3.9.16 (2024-11-26)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-175 'Direct link to Bugfixes')

- Replace `pickle` and `joblib` with safer alternatives, e.g. `json`, `safetensors`, and `skops`, for serializing components.
 
 **Note**: This is a model breaking change. Please retrain your model.
 
 If you have a custom component that inherits from one of the components listed below and modified the `persist` or `load` method, make sure to update your code. Please contact us in case you encounter any problems.
 
 Affected components:
 
 - `CountVectorFeaturizer`
 - `LexicalSyntacticFeaturizer`
 - `LogisticRegressionClassifier`
 - `SklearnIntentClassifier`
 - `DIETClassifier`
 - `CRFEntityExtractor`
 - `TrackerFeaturizer`
 - `TEDPolicy`
 - `UnexpectedIntentTEDPolicy`

## \[3.9.15\] - 2024-10-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3915---2024-10-18 'Direct link to [3.9.15] - 2024-10-18')

Rasa Pro 3.9.15 (2024-10-18)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-34 'Direct link to Improvements')

- Change default response of `utter_free_chitchat_response` from `"placeholder_this_utterance_needs_the_rephraser"` to `"Sorry, I'm not able to answer that right now."`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-176 'Direct link to Bugfixes')

- Fix cleanup of `SetSlot` commands issued by the LLM-based command generator for slots that define a slot mapping other than the `from_llm` slot mapping. The command processor now correctly removes the SetSlot command in these scenarios and instead adds a `CannotHandleCommand`.
- Disallow using the command payload syntax to set slots not filled by any of the active or startable flow(s) `collect` steps.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-45 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.14\] - 2024-10-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3914---2024-10-02 'Direct link to [3.9.14] - 2024-10-02')

Rasa Pro 3.9.14 (2024-10-02)

No significant changes.

## \[3.9.13\] - 2024-10-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3913---2024-10-01 'Direct link to [3.9.13] - 2024-10-01')

Rasa Pro 3.9.13 (2024-10-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-177 'Direct link to Bugfixes')

- Fix inconsistent recording of telemetry events for llm-based command generators.
- Added tracing explicitly to `GRPCCustomActionExecutor.run` in order to pass the tracing context to the action server.
- Fixes an issue where the `CountVectorsFeaturizer` and `LogisticRegressionClassifier` would throw error during inference when no NLU training data is provided.

## \[3.9.12\] - 2024-09-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3912---2024-09-20 'Direct link to [3.9.12] - 2024-09-20')

Rasa Pro 3.9.12 (2024-09-20)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-9 'Direct link to Deprecations and Removals')

- Dropped support for Python 3.8 ahead of [Python 3.8 End of Life in October 2024](https://devguide.python.org/versions/#supported-versions). In Rasa Pro versions 3.10.0, 3.9.11 and 3.8.13, we needed to pin the TensorFlow library version to 2.13.0rc1 in order to remove critical vulnerabilities; this resulted in poor user experience when installing these versions of Rasa Pro with `uv pip`. Removing support for Python 3.8 will make it possible to upgrade to a stabler version of TensorFlow.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-35 'Direct link to Improvements')

- Update Keras and Tensorflow to version 2.14. This will eliminate the need to use the `--prerelease allow` flag when installing Rasa Pro using `uv pip` tool.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-178 'Direct link to Bugfixes')

- Fix `AttributeError` with the instrumentation of the `run` method of the `CustomActionExecutor` class.
- Fixed UnexpecTEDIntentlessPolicy training errors that resulted from a change to batching behavior. Changed the batching behavior back to the original for all components. Made the changed batching behavior accessible in DietClassifier using `drop_small_last_batch: True`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-46 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.11\] - 2024-09-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3911---2024-09-13 'Direct link to [3.9.11] - 2024-09-13')

Rasa Pro 3.9.11 (2024-09-13)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-179 'Direct link to Bugfixes')

- Update Keras to 2.13.1 and Tensorflow to 2.13.0rc0 to fix critical vulnerability (CVE-2024-3660).

## \[3.9.10\] - 2024-09-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3910---2024-09-12 'Direct link to [3.9.10] - 2024-09-12')

Rasa Pro 3.9.10 (2024-09-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-180 'Direct link to Bugfixes')

- Fix `FileNotFound` error when running `rasa studio` commands and no pre-existing local assistant project exists.
- Fixed telemetry collection for the components Rephraser, LLM Intent Classifier, Intentless Policy and Enterprise Search Policy to ensure that the telemetry data is only collected when it is enabled

## \[3.9.9\] - 2024-08-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#399---2024-08-23 'Direct link to [3.9.9] - 2024-08-23')

Rasa Pro 3.9.9 (2024-08-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-181 'Direct link to Bugfixes')

- Updated behaviour of policies in coexistence:
 
 - CALM policies run in case the routing slot is set to `True` (routing to CALM).
 - Policies of the nlu-based system run in case the routing slot is set to `False` (routing to NLU-based system) or `None` (non-sticky routing).
- Don't create an instance of `FlowRetrieval` in the command generators in case no flows exists.
 
- Patterns do not count as active flows in `MultiStepLLMCommandGenerator` anymore.
 
- Make sure that all e2e test cases in rasa inspector are valid.
 
- Downloading of CALM Assistants from Studio improved:
 
 - Downloading CALM assistants from Studio now includes `config` and `endpoints` files
 - Downloading CALM assistants from Studio now doesn't require `config.yml` and `data` folder to exist

## \[3.9.8\] - 2024-08-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#398---2024-08-21 'Direct link to [3.9.8] - 2024-08-21')

Rasa Pro 3.9.8 (2024-08-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-182 'Direct link to Bugfixes')

- Fix problem with custom action invocation when model is loaded from remote storage.

## \[3.9.7\] - 2024-08-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#397---2024-08-15 'Direct link to [3.9.7] - 2024-08-15')

Rasa Pro 3.9.7 (2024-08-15)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-183 'Direct link to Bugfixes')

- Fix extraction of tracing context from the request headers and injection into the Rasa server tracing context.
- `YamlValidationException` will correctly return line number of the element where the error occurred when line number of that element is not returned by `ruamel.yaml` (for elements of primitive types, e.g. `str`, `int`, etc.), instead of returning the line number of the parent element.
- Updated `setuptools` to fix security vulnerability.
- Fix tracing context propagation to work for all external service calls.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-47 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.6\] - 2024-08-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#396---2024-08-07 'Direct link to [3.9.6] - 2024-08-07')

Rasa Pro 3.9.6 (2024-08-07)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-48 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.5\] - 2024-08-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#395---2024-08-01 'Direct link to [3.9.5] - 2024-08-01')

Rasa Pro 3.9.5 (2024-08-01)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-36 'Direct link to Improvements')

- Enabled generative chitchat in the `tutorial` template with instructions on how to turn it off added to the documentation.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-184 'Direct link to Bugfixes')

- Update the usage of `time.process_time_ns` with `time.perf_counter_ns` to fix the inconsistencies between duration metrics and trace spans duration.

## \[3.9.4\] - 2024-07-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#394---2024-07-25 'Direct link to [3.9.4] - 2024-07-25')

Rasa Pro 3.9.4 (2024-07-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-185 'Direct link to Bugfixes')

- Fix instrumentation not accounting for `kwargs` that are passed to `NLUCommandAdapter.predict_commands`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-49 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.9.3\] - 2024-07-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#393---2024-07-18 'Direct link to [3.9.3] - 2024-07-18')

Rasa Pro 3.9.3 (2024-07-18)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-186 'Direct link to Bugfixes')

- Refactor the supported remote storage (AWS, GCS, Azure) verification check before downloading Rasa model by fixing the initial implementation which attempted to create the object storage to check existence.
- Fix `TypeError: InformationRetrieval.search() got an unexpected keyword argument` when tracing is enabled with `EnterpriseSearchPolicy`.
- Change `warning` log level to `error` log level for `Validator` methods that verify that forms and actions used in stories and rules are present in the domain.

## \[3.9.2\] - 2024-07-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#392---2024-07-09 'Direct link to [3.9.2] - 2024-07-09')

Rasa Pro 3.9.2 (2024-07-09)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-187 'Direct link to Bugfixes')

- Add key-word arguments in the predict\_commands method of LLM-based CommandGenerator class to ensure custom components are not impacted by changes to the signature of the base classes.

## \[3.9.1\] - 2024-07-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#391---2024-07-04 'Direct link to [3.9.1] - 2024-07-04')

Rasa Pro 3.9.1 (2024-07-04)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-188 'Direct link to Bugfixes')

- Modify the validation to throw an error for a missing associated action/utterance in a collect step only if the slot does not have a defined initial value.
- Modify the collect step validation in flow executor to trigger `pattern_internal_error` for a missing associated action/utterance in a collect step only if the slot does not have a defined initial value.

## \[3.9.0\] - 2024-07-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#390---2024-07-03 'Direct link to [3.9.0] - 2024-07-03')

Rasa Pro 3.9.0 (2024-07-03)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-8 'Direct link to Features')

- Introduce a new response [button payload format](https://rasa.com/docs/rasa-pro/concepts/responses#payload-syntax) that runs set slot CALM commands directly by skipping the user message processing pipeline.
 
- Added support for Information Retrieval custom components. It allows Enterprise Search Policy to be used with arbitrary search systems. Custom Information Retrievals can be implemented as a subclass of `rasa.core.information_retrieval.InformationRetrieval`
 
- Enable slot filling in a CALM assistant to be configurable:
 
 - either use NLU-based predefined slot mappings that instructs `NLUCommandAdapter` to issue SetSlot commands with values extracted from the user input via an entity extractor or intent classifier
 - or use the new predefined slot mapping `from_llm` which enables LLM-based command generators to issue SetSlot commands If no slot mapping is defined, the default behavior is to use the `from_llm` slot mapping.
 
 In case you had been using `custom` slot mapping type for slots set with the prediction of the LLM-based command generator, you need to update your assistant configuration to use the new `from_llm` slot mapping type. Note that even if you have written custom slot validation actions (following the `validate_<slot_name>` convention) for slots set by the LLM-based command generator, you need to update your assistant configuration to use the new `from_llm` slot mapping type.
 
 For [slots that are set only via the custom action](https://rasa.com/docs/reference/primitives/slots/#custom-slot-mappings) e.g. slots set by external sources only, you need to add the action name to the slot mapping:
 
 ```
    slots:  slot_name:    type: text    mappings:      - type: custom        action: custom_action_name
    ```
 
- Skip `SetSlot` commands issued by LLM based command generators for slots with NLU-based predefined slot mappings. Instead, the command processor component will issue `CannotHandle` command to trigger `pattern_cannot_handle` if no other valid command is found.
 
- Rasa now supports gRPC protocol for custom actions. This allows users to use gRPC to invoke custom actions. To connect to the action server using gRPC, specify:
 
 endpoints.yml
 
 ```
    action_endpoint:    url: "grpc://<rasa-grpc-action-server>:<port>"
    ```
 
 Users can use secure (TLS) and insecure connections to communicate over gRPC. To use TLS specify the following in `endpoints.yml`:
 
 endpoints.yml
 
 ```
    action_endpoint:    url: "grpc://<rasa-grpc-action-server>:<port>"    cafile: "<ca_file>"
    ```
 
- Add `MultiStepLLMCommandGenerator` as an alternative LLM based command generator. `MultiStepLLMCommandGenerator` breaks down the task of dialogue understanding into two steps: handling the flows and filling the slots. The component was designed to enable cheaper and smaller LLMs, such as `gpt-3.5-turbo`, as viable alternatives to costlier but more powerful models such as `gpt-4`. To use the `MultiStepLLMCommandGenerator` add it to your pipeline:
 
 ```
    pipeline:  ...  - name: MultiStepLLMCommandGenerator  ...
    ```
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-37 'Direct link to Improvements')

- Improve diagram display in the inspector by adding an horizontal scroll and an auto scroll to the active step.
 
- Create a separate default prompt for Enterprise Search with source citation enabled and revert the default Enterprise Search prompt to that of `3.7.x`.
 
- Refactored `RemoteAction` to utilize a new `CustomActionExecutor` interface by implementing `HTTPCustomActionExecutor` to handle HTTP requests for custom actions.
 
- Implemented an optimization to reduce payload size by ensuring the Assistant sends the domain dictionary to the Action Server only once, which the server then stores. If the Action Server responds with a 449 status code indicating a missing domain context, the Assistant will repeat the API request including the domain dictionary in the payload, ensuring the server properly saves this data.
 
- Integrate the capability of testing scenarios that reflect actual operational environments where conversations can be influenced by real-time external data. This is done by injecting metadata when running end-to-end tests.
 
- Introduced LRU caching for reading and parsing YAML files to enhance performance by avoiding multiple reads of the same file. Added `READ_YAML_FILE_CACHE_MAXSIZE` environment variable with a default value of 256 to configure the cache size.
 
- Add validations for flow ID to allow only alphanumeric characters, underscores, and hyphens except for the first character.
 
- The `LLMCommandGenerator` component has been renamed to `SingleStepLLMCommandGenerator`. There is no change to the functionality.
 
 Using the `LLMCommandGenerator` as the name of the component results in a deprecation warning as it will be permanently renamed to `SingleStepLLMCommandGenerator` in 4.0.0. Please modify the assistant’s configuration to use the `SingleStepLLMCommandGenerator` instead of the `LLMCommandGenerator` to avoid seeing the deprecation warning.
 
- Make improvements to `rasa data validate` that check if the usage of slot mappings in a CALM assistant is valid:
 
 - a slot cannot have both a `from_llm` mapping and either a nlu-predefined mapping or a custom slot mapping
 - a slot collected in a flow by a custom action has an associated `action_ask_` defined in the domain
 - a CALM assistant with slots that have nlu-based predefined mappings include `NLUCommandAdapter` in the config pipeline
 - a NLU-based assistant cannot have slots that have a `from_llm` mapping
- Modify post processing of commands - Clarify command with single option is converted into a StartFlow command.
 
- Improve debug logging for predicate evaluation.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-189 'Direct link to Bugfixes')

- Properly handle projects where `rasa studio download` is run in a project with no NLU data.
- Tracing is supported for actions called over gRPC protocol.
- Fix the hash function of ClarifyCommand to return a hashed list of options.
- Raise an error if action\_reset\_routing is used without the defined ROUTE\_TO\_CALM\_SLOT / router.
- Add a few bugfixes to the CALM slot mappings feature:
 - Coexistence bot should ignore `NoOpCommand` when checking if the processed message contains commands.
 - Update condition under which FlowPolicy triggers `pattern_internal_error` for slots with custom slot mappings.
- Remove invalid warnings during collect step.
- - Fixed issue where messages with invalid intent triggers ('/<intent>') were not handled correctly. Now triggering the `pattern_cannot_handle`.
 - Introduced a new reason `cannot_handle_invalid_intent` for use in the pattern\_cannot\_handle switch mechanism to improve error handling.
- Validates that a collect step in a flow either has an action or an utterance defined in the domain to avoid the bot being silent.
- Slots that are set via response buttons should not trigger `pattern_cannot_handle` regardless of the slots' mapping type.
- Coerce "None", "null" or "undefined" slot values set via response buttons to be of type `NoneType` instead of `str`.
- Avoid raising a `UserWarning` during validation of response buttons which contain double curly braces.
- Do not run NLUCommandAdapter during message parsing when receiving a `/SetSlots` button payload. This is because the NLUCommandAdapter run during message parsing (when the graph is skipped) is meant to handle intent button payloads only.
- Exclude slots that are not collected in any flow from being set by the NLUCommandAdapter in a coexistence assistant.
- Default action `action_extract_slots` should not run custom actions specified in custom slot mappings for slots that are set by custom actions in the flows/CALM system of a coexistence assistant.
- Fix pattern flows being unavailable during input preparation and template rendering in `MultiStepLLMCommandGenerator`.
- Skip command cleaning when no commands are present in NLUCommandAdapter. Fix get active flows to return the correct active flows, including all the nested parent flows if present.
- If FlowPolicy tries to collect a slot with a custom slot mapping without the `action` key or `action_ask` specified in the domain, it will trigger `pattern_cancel_flow` first, then `pattern_internal_error`.
- Cancel user flow in progress and invoke pattern\_internal\_error if the flow reached a collect step which does not have an associated utter\_ask response or action\_ask action defined in the domain.
- IntentlessPolicy abstains from making a prediction during coexistence when it's the turn of the NLU-based system.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-50 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.8.17\] - 2024-10-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3817---2024-10-18 'Direct link to [3.8.17] - 2024-10-18')

Rasa Pro 3.8.17 (2024-10-18)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-38 'Direct link to Improvements')

- Change default response of `utter_free_chitchat_response` from `"placeholder_this_utterance_needs_the_rephraser"` to `"Sorry, I'm not able to answer that right now."`.

## \[3.8.16\] - 2024-10-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3816---2024-10-02 'Direct link to [3.8.16] - 2024-10-02')

Rasa Pro 3.8.16 (2024-10-02)

No significant changes.

## \[3.8.15\] - 2024-10-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3815---2024-10-01 'Direct link to [3.8.15] - 2024-10-01')

Rasa Pro 3.8.15 (2024-10-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-190 'Direct link to Bugfixes')

- Fixes an issue where the `CountVectorsFeaturizer` and `LogisticRegressionClassifier` would throw error during inference when no NLU training data is provided.

## \[3.8.14\] - 2024-09-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3814---2024-09-20 'Direct link to [3.8.14] - 2024-09-20')

Rasa Pro 3.8.14 (2024-09-20)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-10 'Direct link to Deprecations and Removals')

- Dropped support for Python 3.8 ahead of [Python 3.8 End of Life in October 2024](https://devguide.python.org/versions/#supported-versions). In Rasa Pro versions 3.10.0, 3.9.11 and 3.8.13, we needed to pin the TensorFlow library version to 2.13.0rc1 in order to remove critical vulnerabilities; this resulted in poor user experience when installing these versions of Rasa Pro with `uv pip`. Removing support for Python 3.8 will make it possible to upgrade to a stabler version of TensorFlow.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-39 'Direct link to Improvements')

- Update Keras and Tensorflow to version 2.14. This will eliminate the need to use the `--prerelease allow` flag when installing Rasa Pro using `uv pip` tool.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-191 'Direct link to Bugfixes')

- Fixed UnexpecTEDIntentlessPolicy training errors that resulted from a change to batching behavior. Changed the batching behavior back to the original for all components. Made the changed batching behavior accessible in DietClassifier using `drop_small_last_batch: True`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-51 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.8.13\] - 2024-09-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3813---2024-09-12 'Direct link to [3.8.13] - 2024-09-12')

Rasa Pro 3.8.13 (2024-09-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-192 'Direct link to Bugfixes')

- Fixed telemetry collection for the components Rephraser, LLM Intent Classifier, Intentless Policy and Enterprise Search Policy to ensure that the telemetry data is only collected when it is enabled
- Update Keras to 2.13.1 and Tensorflow to 2.13.0rc0 to fix critical vulnerability (CVE-2024-3660).

## \[3.8.12\] - 2024-08-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3812---2024-08-12 'Direct link to [3.8.12] - 2024-08-12')

Rasa Pro 3.8.12 (2024-08-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-193 'Direct link to Bugfixes')

- Fix `TypeError: InformationRetrieval.search() got an unexpected keyword argument` when tracing is enabled with `EnterpriseSearchPolicy`.
- Fix extraction of tracing context from the request headers and injection into the Rasa server tracing context.
- Update the usage of `time.process_time_ns` with `time.perf_counter_ns` to fix the inconsistencies between duration metrics and trace spans duration.
- `YamlValidationException` will correctly return line number of the element where the error occurred when line number of that element is not returned by `ruamel.yaml` (for elements of primitive types, e.g. `str`, `int`, etc.), instead of returning the line number of the parent element.
- Updated `setuptools` to fix security vulnerability.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-52 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.8.11\] - 2024-07-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3811---2024-07-04 'Direct link to [3.8.11] - 2024-07-04')

Rasa Pro 3.8.11 (2024-07-04)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-40 'Direct link to Improvements')

- Improve debug logging for predicate evaluation.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-194 'Direct link to Bugfixes')

- Raise an error if action\_reset\_routing is used without the defined ROUTE\_TO\_CALM\_SLOT / router.
- Remove invalid warnings during collect step.
- - Fixed issue where messages with invalid intent triggers ("/intent\_name") were not handled correctly. Now triggering the `pattern_cannot_handle`.
 - Introduced a new reason `cannot_handle_invalid_intent` for use in the pattern\_cannot\_handle switch mechanism to improve error handling.
- Validates that a collect step in a flow either has an action or an utterance defined in the domain to avoid the bot being silent.
- Skip command cleaning when no commands are present in NLUCommandAdapter. Fix get active flows to return the correct active flows, including all the nested parent flows if present.
- Update the handling of incorrect use of slash syntax. Messages with undefined intents do not automatically trigger `pattern_cannot_handle`; instead, they are sanitized (prepended slash(es) are removed) and passed through the graph.
- Modify the validation to throw an error for a missing associated action/utterance in a collect step only if the slot does not have a defined initial value.

## \[3.8.10\] - 2024-06-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3810---2024-06-19 'Direct link to [3.8.10] - 2024-06-19')

Rasa Pro 3.8.10 (2024-06-19)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-41 'Direct link to Improvements')

- Added NLG validation to the rasa model training process.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-195 'Direct link to Bugfixes')

- Fixes Clarify command being dropped by command processor due to presence of coexistence slot - `route_session_to_calm`
- Fix validation for LLMBasedRouter to check only for calm\_entry.sticky

## \[3.8.9\] - 2024-06-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#389---2024-06-14 'Direct link to [3.8.9] - 2024-06-14')

Rasa Pro 3.8.9 (2024-06-14)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-42 'Direct link to Improvements')

- Add validations for flow ID to allow only alphanumeric characters, underscores, and hyphens except for the first character.

## \[3.8.8\] - 2024-06-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#388---2024-06-07 'Direct link to [3.8.8] - 2024-06-07')

Rasa Pro 3.8.8 (2024-06-07)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-196 'Direct link to Bugfixes')

- Add wrappers around openai clients that can set the self-signed certs via `REQUESTS_CA_BUNDLE` env variable.

## \[3.8.7\] - 2024-05-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#387---2024-05-29 'Direct link to [3.8.7] - 2024-05-29')

Rasa Pro 3.8.7 (2024-05-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-197 'Direct link to Bugfixes')

- Add support for domain entities in CALM import
- Download NLU data when running `rasa studio download` for a modern assistant with NLU triggers. Previously, this data was not downloaded, leading to a partial assistant.

## \[3.8.6\] - 2024-05-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#386---2024-05-27 'Direct link to [3.8.6] - 2024-05-27')

Rasa Pro 3.8.6 (2024-05-27)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-43 'Direct link to Improvements')

- Adds `tracker_state` attribute to `OutputChannel`. It simplifies the access of tracker state for custom channel connector with `CollectingOutputChannel.tracker_state`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-198 'Direct link to Bugfixes')

- If a button in a response does not have a payload, socketio channel will use the title as payload by default rather than throwing an exception.

## \[3.8.5\] - 2024-05-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#385---2024-05-03 'Direct link to [3.8.5] - 2024-05-03')

Rasa Pro 3.8.5 (2024-05-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-199 'Direct link to Bugfixes')

- Trigger `pattern_internal_error` if collection does not exist in a Qdrant vector store.

## \[3.8.4\] - 2024-04-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#384---2024-04-30 'Direct link to [3.8.4] - 2024-04-30')

Rasa Pro 3.8.4 (2024-04-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-44 'Direct link to Improvements')

- Added support for NLU Triggers by supporting uploading the NLU files for CALM Assistants

## \[3.8.3\] - 2024-04-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#383---2024-04-26 'Direct link to [3.8.3] - 2024-04-26')

Rasa Pro 3.8.3 (2024-04-26)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-45 'Direct link to Improvements')

- - Throw validation error and exit when duplicate responses are found across domains. This is a breaking change, as it will cause training to fail if duplicate responses are found. If you have duplicate responses in your training data, you will need to remove them before training.
 - Update domain importing to ignore the warnings about duplicates when merging with the default flow domain

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-200 'Direct link to Bugfixes')

- Use AzureChatOpenAI class instead of AzureOpenAI class to instantiate openai models deployed in Azure. This fixes the usage of gpt-3.5-turbo model in Azure.
- Fixes validation to catch empty placeholders in response that dumps entire context.
- Fix security vulnerabilities by updating poetry environment: fonttools, CVE-2023-45139, from 4.40.0 to 4.43.0 aiohttp, CVE-2024-27306, from 3.9.3 to 3.9.4 dnspython, CVE-2023-29483, from 2.3.0 to 2.6.1 pymongo, CVE-2024-21506, from 4.3.3 to 4.6.3
- Numbers that are part of the body of the LLM answer in EnterpriseSearch should not be matched as citation references in the postprocessing method.
- Errors from the Flow Retrieval API are now both logged and thrown. When such errors occur, an ErrorCommand is emitted by the Command Generator.

## \[3.8.2\] - 2024-04-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#382---2024-04-25 'Direct link to [3.8.2] - 2024-04-25')

Rasa Pro 3.8.2 (2024-04-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-201 'Direct link to Bugfixes')

- Add the currently active flow as well as the called flow (if present) to the list of available flows for the `LLMCommandGenerator`.
- Fix custom prompt not read from the model resource path for LLMCommandGenerator.

## \[3.8.1\] - 2024-04-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#381---2024-04-17 'Direct link to [3.8.1] - 2024-04-17')

Rasa Pro 3.8.1 (2024-04-17)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-46 'Direct link to Improvements')

- Adjusted chat widget behavior to remain open when clicking outside the chat box area.
- Improve debug logs to include information about evaluation of `if-else` conditions in flows at runtime.
- Remove the `ContextualResponseRephraser` from the tutorial template to keep it simple as it is not needed anymore.
- Update poetry package manager version to `1.8.2`. Check the [migration guide](https://rasa.com/docs/rasa-pro/migration-guide#rasa-pro-380-to-rasa-pro-381) for instructions on how to update your environment.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-202 'Direct link to Bugfixes')

- Introduced support for numbered Markdown lists.
- Added support for uploading assistants with default domain directory.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-53 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.8.0\] - 2024-04-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#380---2024-04-03 'Direct link to [3.8.0] - 2024-04-03')

Rasa Pro 3.8.0 (2024-04-03)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-9 'Direct link to Features')

- Introduces **semantic retrieval of flows** at runtime to reduce the size of the prompt sent to the LLM by utilizing similarity between vector embeddings. It enables the assistant to scale to a large number of flows.
 
 Flow retrieval is **enabled by default**. To configure it, you can modify the settings under the `flow_retrieval` property of `LLMCommandGenerator` component. For detailed configuration options, refer to our [documentation](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-flow-retrieval).
 
 Introduces `always_include_in_prompt` field to the [flow definition](https://rasa.com/docs/rasa-pro/concepts/flows/#flow-properties). If field is set to `true` and the [flow guard](https://rasa.com/docs/rasa-pro/concepts/starting-flows/#flow-guards) defined in the `if` field evaluates to `true`, the flow will be included in the prompt.
 
- Introduction of coexistence between CALM and NLU-based assistants. Coexistence allows you to use policies from both CALM and NLU-based assistants in a single assistant. This allows migrating from NLU-based paradigm to CALM in an iterative fashion.
 
- Introduction of `call` step. You can use a `call` step to embed another flow. When the execution reaches a `call` step, Rasa starts the called flow. Once the called flow is complete, the execution continues with the calling flow.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-47 'Direct link to Improvements')

- Instrument the `command_processor` module, in particular the following functions:
 
 - `execute_commands`
 - `clean_up_commands`
 - `validate_state_of_commands`
 - `remove_duplicated_set_slots`
- Improve the instrumentation of `LLMCommandGenerator`:
 
 - extract more LLM configuration parameters, e.g. `type`, `temperature`, `request-timeout`, `engine` and `deployment` (the latter 2 being only for the Azure OpenAI service).
 - instrument the private method `_check_commands_against_startable_flows` to track the commands with which the LLM responded, as well as the startable flow ids.
- Instrument `flow_executor.py` module, in particular these functions:
 
 - `advance_flows()`: extract `available_actions` tracing tag
 - `advance_flows_until_next_action()`: extract action name and score, metadata and prediction events as tracing tags from the returned prediction value
 - `run_step()`: extract step custom id, description and current flow id.
- Instrument `Policy._prediction()` method for each of the policy subclasses.
 
- Instrument `IntentlessPolicy` methods such as:
 
 - `find_closest_response`: extract the `response` and `score` from the returned tuple;
 - `select_response_examples`: extract the `ai_response_examples` from returned value;
 - `select_few_shot_conversations`: extract the `conversation_samples` from returned value;
 - `extract_ai_responses`: extract the `ai_responses` from returned value;
 - `generate_answer`: extract the `llm_response` from returned value.
- 1. Instrument `InformationRetrieval.search` method for supported vector stores: extract query and document metadata tracing attributes.
 2. Instrument `EnterpriseSearchPolicy._generate_llm_answer` method: extract LLM config tracing attributes.
 3. Extract dialogue stack current context in the following functions:
 
 - `rasa.dialogue_understanding.processor.command_processor.clean_up_commands`
 - `rasa.core.policies.flows.flow_executor.advance_flows`
 - `rasa.core.policies.flows.flow_executor.run_step`
- 1. Instrument `NLUCommandAdapter.predict_commands` method and extract the `commands` from the returned value, as well as the user message `intent`.
 2. Improve LLM config tracing attribute extraction for `ContextualResponseRephraser`.
- Add new config boolean property `trace_prompt_tokens` that would enable the tracing of the length of the prompt tokens for the following components:
 
 - `LLMCommandGenerator`
 - `EnterpriseSearchPolicy`
 - `IntentlessPolicy`
 - `ContextualResponseRephraser`
- Enable execution of single E2E tests by including the test case name in the path to test cases, like so: `path/to/test_cases.yml::test_case_name` or `path/to/folder_containing_test_cases::test_case_name`.
 
- Implement `MetricInstrumentProvider` interface whose role is to:
 
 - register instruments during metrics configuration
 - retrieve the appropriate instrument to record measurements in the relevant instrumentation code section
- Enabled the setting of a minimum similarity score threshold for retrieved documents in Enterprise Search's `vector_store` with the addition of the `threshold` property. If no documents are retrieved, it triggers Pattern Cannot Handle. This feature is supported in Milvus and Qdrant vector stores.
 
- Record measurements for the following metrics in the instrumentation code:
 
 - CPU usage of the `LLMCommandGenerator`
 - memory usage of `LLMCommandGenerator`
 - prompt token usage of `LLMCommandGenerator`
 - method call duration for LLM specific calls (in `LLMCommandGenerator`, `EnterpriseSearchPolicy`, `IntentlessPolicy`, `ContextualResponseRephraser`)
 - rasa client request duration
 - rasa client request body size
 
 Instrument `EndpointConfig.request()` method call in order to measure the client request metrics.
 
- Improvements around default behaviour of `ChitChatAnswerCommand()`:
 
 - The command processor will issue `CannotHandleCommand()` instead of the `ChitChatCommand()` when `pattern_chitchat` uses an action step `action_trigger_chitchat` without the `IntentlessPolicy` being configured. During training a warning is raised.
 - Changed the default pattern\_chitchat to:
 
 ```
    pattern_chitchat:  description: handle interactions with the user that are not task-oriented  name: pattern chitchat  steps:    - action: action_trigger_chitchat
    ```
 
 - Default rasa init template for CALM comes with `IntentlessPolicy` added to pipeline.
- Add support for OTLP Collector as metrics receiver which can forward metrics to the chosen metrics backend, e.g. Prometheus.
 
- Enable document source citation for Enterprise Search knowledge answers by setting the boolean `citation_enabled: true` property in the `config.yml` file:
 
 ```
    policies:  - name: EnterpriseSearchPolicy    citation_enabled: true
    ```
 
- Add telemetry events for flow retrieval and call step
 
- Tighten python dependency constraints in `pyproject.toml`, hence reducing the installation time to around 20 minutes with `pip` (and no caching enabled).
 
- Improved tracing clarity of the Contextual Response Rephraser by adding the `_create_history` method span, including its LLM configuration attributes.
 
- Users now have enhanced control over the debugging process of LLM-driven components. This update introduces a fine-grained, customizable logging that can be controlled through specific environment variables.
 
 For example, set the `LOG_LEVEL_LLM` environment variable to enable detailed logging at the desired level for all the LLM components or specify the component you are debugging:
 
 ## Example configuration[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#example-configuration 'Direct link to Example configuration')
 
 ```
    export LOG_LEVEL_LLM=DEBUGexport LOG_LEVEL_LLM_COMMAND_GENERATOR=INFOexport LOG_LEVEL_LLM_ENTERPRISE_SEARCH=INFOexport LOG_LEVEL_LLM_INTENTLESS_POLICY=DEBUGexport LOG_LEVEL_LLM_REPHRASER=DEBUG
    ```
 
- If the user wants to chat with the assistant at the end of `rasa init`, we are now calling `rasa inspect` instead of `rasa shell`.
 
- A slot can now be collected via an action `action_ask_<slot-name>` instead of the utterance `utter_ask_<slot-name>` in a collect step. You can either define an utterance or an action for the collect step in your flow. Make sure to add your custom action `action_ask_<slot-name>` to the domain file.
 
- Validate the configuration of the coexistence router before the actual training starts.
 
- Improved error handling in Enterprise Search Policy, changed the prompt to improve formatting of documents and ensured empty slots are not added to the prompt.
 
- Implement asynchronous graph execution. CALM assistants rely on a lot of I/O calls (e.g. to a LLM service), which impaired performances. With this change, we've improved the response time performance by 10x. All policies and components now support async calling.
 
- Merge `rasa` and `rasa-plus` packages into one. As a result, we renamed the Python package to `rasa-pro` and the Docker image to `rasa-pro`. Please head over to the migration guide [here](https://rasa.com/docs/rasa-pro/migration-guide#installation) for installation, and [here](https://rasa.com/docs/rasa-pro/migration-guide#component-yaml-configuration-changes) for the necessary configuration updates.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-203 'Direct link to Bugfixes')

- Updated pillow and jinja2 packages to address security vulnerabilities.
 
- Fix OpenTelemetry `Invalid type NoneType for attribute value` warning.
 
- Add support for `metadata_payload_key` for Qdrant Vector Store with an error message if `content_payload_key` or `metadata_payload_key` are incorrect
 
- Changed the ordering of returned events to order by ID (previously timestamp) in SQL Tracker Store
 
- Improved the end-to-end test comparison mechanism to accurately handle and strip trailing newline characters from expected bot responses, preventing false negatives due to formatting mismatches.
 
- Fixed a bug that caused inaccurate search results in Enterprise Search when a bot message appeared before the last user message.
 
- Fixes flow guards pypredicate evaluatation bug: pypredicate was evaluated with `Slot` instances instead of slot values
 
- Post-process source citations in Enterprise Search Policy responses so that they are enumerated in the correct order.
 
- Resolves issue causing the `FlowRetrieval.populate` to always use default embeddings.
 
- Fix the bug with the validation of routing setup crashing when the pipeline is not specified (null)
 
- Remove conversation turns prior to a restart when creating a conversation transcript for an LLM call.
 
 This helps in cases where the prior conversation is not relevant for the current session. Information which should be carried to the next session should explicitly be stored in slots.
 
- Add tracker back to the LLMCommandGenerator.parse\_command to ensure compatibility with custom command generator built with 3.7.
 
- Move coexistence routing setup validation from `rasa.validator.Validator` to `rasa.engine.validation`. This gave access to graph schema which allowed for validation checks of subclassed routers.
 
- Fixes a bug in determining the name of the model based on provided parameters.
 
- `LogisticRegressionClassifier` checks if training examples are present during training and logs a warning in case no training examples are provided.
 
- Fixes the bug that resulted in an infinite loop on a collect step in a flow with a flow guard set to `if: False`.
 
- Fix training the enterprise search policy multiple times with a different source folder name than the default name "docs".
 
- Log message `llm_command_generator.predict_commands.finished` is set to debug log by default. To enable logging of the `LLMCommandGenerator` set `LOG_LEVEL_LLM_COMMAND_GENERATOR` to `INFO`.
 
- Improvements and fixes to cleaning up commands:
 
 - Clean up predicted `StartFlow` commands from the `LLMCommandGenerator` if the flow, that should be started, is already active.
 - Clean up predicted SetSlot commands from the `LLMCommandGenerator` if the value of the slot is already set on the tracker.
 - Use string comparison for slot values to make sure to capture cases when the `LLMCommandGenerator` predicted a string value but the value set on the tracker is, for example, an integer value.
- Remove `context` from list of `restricted` slots
 
- Improved handling of categorical slots with text values when using CALM.
 
 Slot values extracted by the command generator (LLM) will be stored in the same casing as the casing used to define the categorical slot values in the domain. E.g. A categorical slot defined to store the values \["A", "B"\] will store "A" if the LLM predicts the slot to be filled with "a". Previously, this would have stored "a".
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-54 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.7.9\] - 2024-03-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#379---2024-03-26 'Direct link to [3.7.9] - 2024-03-26')

Rasa Pro 3.7.9 (2024-03-26)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-48 'Direct link to Improvements')

- Add validations for flow ID to allow only alphanumeric characters, underscores, and hyphens except for the first character.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-204 'Direct link to Bugfixes')

- Changed the ordering of returned events to order by ID (previously timestamp) in SQL Tracker Store
 
- Fixes flow guards pypredicate evaluatation bug: pypredicate was evaluated with `Slot` instances instead of slot values
 
- Improved handling of categorical slots with text values when using CALM.
 
 Slot values extracted by the command generator (LLM) will be stored in the same casing as the casing used to define the categorical slot values in the domain. E.g. A categorical slot defined to store the values \["A", "B"\] will store "A" if the LLM predicts the slot to be filled with "a". Previously, this would have stored "a".
 
- Log message `llm_command_generator.predict_commands.finished` is set to debug log by default. To enable logging of the `LLMCommandGenerator` set `LOG_LEVEL_LLM_COMMAND_GENERATOR` to `INFO`.
 
- Improvements and fixes to cleaning up commands:
 
 - Clean up predicted `StartFlow` commands from the `LLMCommandGenerator` if the flow, that should be started, is already active.
 - Clean up predicted SetSlot commands from the `LLMCommandGenerator` if the value of the slot is already set on the tracker.
 - Use string comparison for slot values to make sure to capture cases when the `LLMCommandGenerator` predicted a string value but the value set on the tracker is, for example, an integer value.

## \[3.7.8\] - 2024-02-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#378---2024-02-28 'Direct link to [3.7.8] - 2024-02-28')

Rasa Pro 3.7.8 (2024-02-28)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-49 'Direct link to Improvements')

- Improved UX around ClarifyCommand by checking options for existence and ordering them. Also, now dropping Clarify commands if there are any other commands to prevent two questions or statements to be uttered at the same time.
- LLMCommandGenerator returns CannotHandle() command when is encountered with scenarios where it is unable to predict a valid command.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-205 'Direct link to Bugfixes')

- Replace categorical slot values in a predicate with lower case replacements. This fixes the case sensitive slot comparisons in flow guards, branches in flows and slot rejections.
- Modify flows YAML schema to make next step mandatory to noop step.
- Flush messages when Kafka producer is closed. This is to ensure that all messages in the producer's internal queue are sent to the broker. Ensure to import all pattern stack frame subclasses of `DialogueStackFrame` when retrieving tracker from the tracker store, a required step during `rasa export`.
- Add support for `metadata_payload_key` for Qdrant Vector Store with an error message if `content_payload_key` or `metadata_payload_key` are incorrect

## \[3.7.7\] - 2024-02-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#377---2024-02-06 'Direct link to [3.7.7] - 2024-02-06')

Rasa Pro 3.7.7 (2024-02-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-206 'Direct link to Bugfixes')

- Updated pillow and jinja2 packages to address security vulnerabilities.

## \[3.7.6\] - 2024-02-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#376---2024-02-01 'Direct link to [3.7.6] - 2024-02-01')

Rasa Pro 3.7.6 (2024-02-01)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-207 'Direct link to Bugfixes')

- Fix reported issue, e.g. [https://github.com/RasaHQ/rasa/issues/5461](https://github.com/RasaHQ/rasa/issues/5461) in Rasa Pro: Do not unpack json payload if `data` key is not present in the response custom output payloads when using socketio channel. This allows assistants which use custom output payloads to work with the Rasa Inspector debugging tool.
- Make flow description a required property in the flow json schema.
- Fix training the enterprise search policy multiple times with a different source folder name than the default name "docs".

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-55 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.7.5\] - 2024-01-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#375---2024-01-24 'Direct link to [3.7.5] - 2024-01-24')

Rasa Pro 3.7.5 (2024-01-24)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-50 'Direct link to Improvements')

- Add new embedding types: `huggingface` and `huggingface_bge`. These new types import the `HuggingFaceEmbeddings` and `HuggingFaceBgeEmbeddings` embedding classes from Langchain.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-208 'Direct link to Bugfixes')

- Fixes a bug that caused the `full_retrieval_intent_name` key to be missing in the published event. Rasa Analytics makes use of this key to get the Retrieval Intent Name
- Pin `grpcio` indirect dependency to `1.56.2` to address [CVE-2023-33953](https://www.cve.org/CVERecord?id=CVE-2023-33953) Pin `aiohttp` to version `3.9.0` to address [CVE-2023-49081](https://www.cve.org/CVERecord?id=CVE-2023-49081)
- Fixes the bug that resulted in an infinite loop on a collect step in a flow with a flow guard set to `if: False`.
- Changed the parameters request timeout to 10 seconds and maximum number of retries to 1 for the default LLM used by Enterprise Search Policy. Any error during vector search or LLM API calls should now trigger the pattern `pattern_internal_error`. Updated the default enterprise search policy prompt to respond more succinctly to queries.

## \[3.7.4\] - 2024-01-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#374---2024-01-03 'Direct link to [3.7.4] - 2024-01-03')

Rasa Pro 3.7.4 (2024-01-03)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-51 'Direct link to Improvements')

- Add embeddings type `azure` to simplify azure configurations, particularly when using Enterprise Search Policy

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-209 'Direct link to Bugfixes')

- Add a validation in `rasa data validate` to check the LinkFlowStep refers to a valid flow ID

## \[3.7.3\] - 2023-12-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#373---2023-12-21 'Direct link to [3.7.3] - 2023-12-21')

Rasa Pro 3.7.3 (2023-12-21)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-52 'Direct link to Improvements')

- Persist prompt as part of the model and reread prompt from the model storage instead of original file path during loading. Impacts LLMCommandGenerator.
- Replaced soon to be depracted text-davinci-003 model with gpt-3.5-turbo. Affects components - LLM Intent Classifier and Contextual Response Rephraser.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-210 'Direct link to Bugfixes')

- Fix stale cache of local knowledge base used by EnterpriseSearchPolicy by implementing the `fingerprint_addon` class method.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-56 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.7.2\] - 2023-12-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#372---2023-12-07 'Direct link to [3.7.2] - 2023-12-07')

Rasa Pro 3.7.2 (2023-12-07)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-211 'Direct link to Bugfixes')

- Fix propagation of context across rasa spans when running `rasa run --enable-api` in the case when no additional tracing context is passed to rasa.
- Fixed a bug in policy invocation that made Enterprise Search Policy and `action_trigger_search` behaved strangely when used with rules and stories
- Updated aiohttp, cryptography and langchain to address security vulnerabilities.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-57 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.7.1\] - 2023-12-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#371---2023-12-01 'Direct link to [3.7.1] - 2023-12-01')

Rasa Pro 3.7.1 (2023-12-01)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-53 'Direct link to Improvements')

- Improved error handling in Enterprise Search Policy, changed the prompt to improve formatting of documents and ensured empty slots are not added to the prompt

## \[3.7.0\] - 2023-11-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#370---2023-11-22 'Direct link to [3.7.0] - 2023-11-22')

Rasa Pro 3.7.0 (2023-11-22)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-10 'Direct link to Features')

- Added Enterprise Search Policy that uses an LLM with conversation context and relevant knowledge base documents to generate rephrased responses. The LLM is prompted to answer the user questions given the chat transcript, documents retrived from a document search and the slot values so far. This policy supports an in-memory Faiss vector store and connecting to instances of Milvus or Qdrant vector store.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-54 'Direct link to Improvements')

- Skip executing the pipeline when the user message is of the form /intent or /intent + entities.
 
- Remove tensorflow-addons from dependencies as it is now deprecated.
 
- Add building multi-platform Docker image (amd64/arm64)
 
- Switch struct log to `FilteringBoundLogger` in order to retain log level set in the config.
 
- Added metadata as an additional argument as an additional parameter to an `Action`s `run` method.
 
 Added an additional default action called `action_send_text` which allows a policy to respond with a text. The text is passed to the action using the metadata, e.g. `metadata={"message": {"text": "Hello"}}`.
 
 Added LLM utility functions.
 
- Passed request headers from REST channel.
 
- Added additional method `fingerprint_addon` to the `GraphComponent` interface to allow inclusion of external data into the fingerprint calculation of a component
 
- Added Schema file and schema validation for flows.
 
- Added environment variables to configure JWT and auth token. For JWT the following environment variables are available:
 
 - JWT\_SECRET
 - JWT\_METHOD
 - JWT\_PRIVATE\_KEY
 
 For auth token the following environment variable is available:
 
 - AUTH\_TOKEN
- Add skip question command
 
- Update the CALM starter template by:
 
 - adding the following flows from the financial chatbot:
 - add\_contact.yml
 - remove\_contact.yml
 - list\_contacts.yml
 - using multiple modules (in the form of yml files) to segregate the flows (a good model to be followed)
 - adding e2e tests:
 - happy paths
 - cancelations
 - corrections
- - Enhanced the Rasa error pattern for accommodating various error types.
 - Upgraded the LLMCommandGenerator for processing the new 'user\_input' configuration section. This update includes handling of messages that surpass the defined character limit.
 
 **Configuration Update:**
 
 The LLMCommandGenerator now supports a user-defined character limit via the 'user\_input' configuration:
 
 ```
      - name: LLMCommandGenerator    llm:      ...    user_input:      max_characters: 500
    ```
 
 **Default Behavior:**
 
 In the absence of a specified limit, it defaults to a 420-character cap. To bypass the limit entirely, set the 'max\_characters' value to -1.
 
- - Bot now returns a default message in response to an empty user message. This improves user experience by providing feedback even when no input is detected.
 - `LLMCommandGenerator` behavior updated. It now returns an `ErrorCommand` for empty user messages.
 - Updated default error pattern and added the default utterance in `default_flows_for_patterns.yml`
- Add support for Vault namespaces. To use namespace set either:
 
 - `VAULT_NAMESPACE` environment variable
 - `namespace` property in `secrets_manager` section at `endpoints.yaml`
- Added Rasa Labs LLM components. Added components are:
 
 - `LLMIntentClassifier`
 - `IntentlessPolicy`
 - `ContextualResponseRephraser`
- Made it possible for the Rasa REST channel to accept OpenTelemetry tracing context.
 
- Improved the naming of trace spans and added more trace tags.
 
- Add `slot_was_not_set` to E2E testing for asserting that a slot was not set and that a slot was not set with a specific value.
 
- Introduced the rasa studio download command, enabling data retrieval from the studio. Implemented the option to refresh the Keycloak token. Expanded the functionality of RasaPrimitiveStorageMapper with the addition of flows. Added flows support to `rasa studio train`.
 
- Instrument `LLMCommandGenerator._generate_action_list_using_llm` and `Command.run_command_on_tracker` methods.
 
- Added the default values for the number of tokens generated by the LLM (`max_tokens`)
 
- Make the instrumentation of `Command.run_command_on_tracker` method applicable to all subclasses of the `Command` class\`
    
-   Instrument `ContextualResponseRephraser._generate_llm_response` and `ContextualResponseRephraser.generate` methods.
    
-   Extract commands as tracing attributes from message input when previous node was the `LLMCommandGenerator`.
    
-   Rename `rasa chat` command to `rasa inspect` and rename channel name to `inspector`.
    
-   Extract `events` and `optional_events` when `GraphNode` is `FlowPolicy`.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-212 'Direct link to Bugfixes')

-   uvloop is disabled by default on apple silicon machines
    
-   Add `rasa_events` to the list of anonymizable structlog keys and rename structlog keys.
    
-   Introduce a validation step in `rasa data validate` and `rasa train` commands to identify non-existent paths and empty domains.
    
-   Rich responses containing buttons with parentheses characters are now correctly parsed. Previously any characters found between the first identified pair of `()` in response button took precedence.
    
-   Resolve dependency incompatibility: Pin version of `dnspython` to ==2.3.0.
    
-   Fixed `KeyError` which resulted when `domain_responses` doesn't exist as a keyword argument while using a custom action dispatcher with nlg server.
    
-   Fixed incompatibility with latest python-socketio release.
    
    The python-socketio released a backwards incompatible change on their minor release. This fix addresses this and makes the code compatible with prior and the new python-socketio version.
    
    [https://github.com/miguelgrinberg/python-socketio/blob/main/CHANGES.md](https://github.com/miguelgrinberg/python-socketio/blob/main/CHANGES.md)
    
-   Fixed the `404 Not Found` Github actions error while removing packages.
    
-   Corrected E2E diff behavior to prevent it from going out of sync when more than one turn difference exists between actual and expected events. Fixed E2E tests from propagating errors when events and test steps did not have the same length. Fixed the issue where E2E tests couldn't locate slot events that were not arranged chronologically. Resolved the problem where E2E tests were incorrectly diffing user utter events when they were not in the correct order.
    
-   Fixed E2E runner wrongly selecting the first available bot utterance when generating the test fail diff.
    
-   Updated werkzeug and urllib3 to address security vulnerabilities.
    
-   Fix cases when E2E test runner crashes when there is no response from the bot.
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation 'Direct link to Improved Documentation')

-   Update wording in Rasa Pro installation page.
-   Updated docs on sending Conversation Events to Multiple DBs.
-   Corrected [action server api](https://rasa.com/docs/reference/api/pro/action-server-api/) sample in docs.
-   Document support for Vault namespaces.
-   Updated tracing documentation to include tracing in the action server and the REST Channel.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-58 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.6.13\] - 2023-10-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3613---2023-10-23 'Direct link to [3.6.13] - 2023-10-23')

Rasa Pro 3.6.13 (2023-10-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-213 'Direct link to Bugfixes')

-   Fix wrong conflicts that occur when rasa validate stories is run with slots that have active\_loop set to null in mapping conditions.

## \[3.6.12\] - 2023-10-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3612---2023-10-10 'Direct link to [3.6.12] - 2023-10-10')

Rasa Pro 3.6.12 (2023-10-10)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-55 'Direct link to Improvements')

-   Added `username` to the connection parameters for `ConcurrentRedisLockStore`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-214 'Direct link to Bugfixes')

-   Refresh headers used in requests (e.g. action server requests) made by `EndpointConfig` using its `headers` attribute.
-   Upgrade `pillow` to `10.0.1` to address security vulnerability CVE-2023-4863 found in `10.0.0` version.
-   Fix setuptools security vulnerability CVE-2022-40897 in Docker build by updating setuptools in poetry's environment.

## \[3.6.11\] - 2023-10-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3611---2023-10-05 'Direct link to [3.6.11] - 2023-10-05')

Rasa Pro 3.6.11 (2023-10-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-215 'Direct link to Bugfixes')

-   Intent names will not be falsely abbreviated in interactive training (fixes OSS-413).
    
    This will also fix a bug where forced user utterances (using the regex matcher) will be reverted even though they are present in the domain.
    
-   Cache `EndpointConfig` session object using `cached_property` decorator instead of recreating this object on every request. Initialize these connection pools for action server and model server endpoints as part of the Sanic `after_server_start` listener. Also close connection pools during Sanic `after_server_stop` listener.
    

## \[3.6.10\] - 2023-09-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3610---2023-09-26 'Direct link to [3.6.10] - 2023-09-26')

Rasa Pro 3.6.10 (2023-09-26)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-56 'Direct link to Improvements')

-   Improved handling of last batch during DIET and TED training. The last batch is discarded if it contains less than half a batch size of data.
-   Added `username` to the connection parameters for `RedisLockStore` and `RedisTrackerStore`
-   Telemetry data is only send for licensed users.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-1 'Direct link to Improved Documentation')

-   Remove the Playground from docs.

## \[3.6.9\] - 2023-09-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#369---2023-09-15 'Direct link to [3.6.9] - 2023-09-15')

Rasa Pro 3.6.9 (2023-09-15)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-57 'Direct link to Improvements')

-   Added additional method `fingerprint_addon` to the `GraphComponent` interface to allow inclusion of external data into the fingerprint calculation of a component

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-216 'Direct link to Bugfixes')

-   Fixed `KeyError` which resulted when `domain_responses` doesn't exist as a keyword argument while using a custom action dispatcher with nlg server.

## \[3.6.8\] - 2023-08-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#368---2023-08-30 'Direct link to [3.6.8] - 2023-08-30')

Rasa Pro 3.6.8 (2023-08-30)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-217 'Direct link to Bugfixes')

-   Fix E2E testing diff algorithm to support the following use cases:
    -   asserting a slot was not set under a `slot_was_set` block
    -   asserting multiple slot names and/or values under a `slot_was_set` block Additionally, the diff algorithm has been improved to show a higher fidelity result.

## \[3.6.7\] - 2023-08-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#367---2023-08-29 'Direct link to [3.6.7] - 2023-08-29')

Rasa Pro 3.6.7 (2023-08-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-218 'Direct link to Bugfixes')

-   Updated certifi, cryptography, and scipy packages to address security vulnerabilities.
-   Updated setuptools and wheel to address security vulnerabilities.

## \[3.6.6\] - 2023-08-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#366---2023-08-23 'Direct link to [3.6.6] - 2023-08-23')

Rasa Pro 3.6.6 (2023-08-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-219 'Direct link to Bugfixes')

-   Updated setuptools and wheel to address security vulnerabilities.

## \[3.6.5\] - 2023-08-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#365---2023-08-17 'Direct link to [3.6.5] - 2023-08-17')

Rasa Pro 3.6.5 (2023-08-17)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-58 'Direct link to Improvements')

-   Use the same session across requests in `RasaNLUHttpInterpreter`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-220 'Direct link to Bugfixes')

-   Resolve dependency incompatibility: Pin version of `dnspython` to ==2.3.0.
-   Fix the issue in `rasa test e2e` where test diff inaccurately displayed actual event transcripts, leading to the duplication of `BotUtter`` or` UserUtter ``events. This occurred specifically when `SetSlot`` events took place that were not explicitly defined in the Test Cases.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-2 'Direct link to Improved Documentation')

-   Updated PII docs with new section on how to use Rasa X/Enterprise with PII management solution, and a new note on debug logs being displayed after the bot message with `rasa shell`.

## \[3.6.4\] - 2023-07-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#364---2023-07-21 'Direct link to [3.6.4] - 2023-07-21')

Rasa Pro 3.6.4 (2023-07-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-221 'Direct link to Bugfixes')

-   Extract conditional response variation and channel variation filtering logic into a separate component. Enable usage of this component in the NaturalLanguageGenerator subclasses (e.g. CallbackNaturalLanguageGenerator, TemplatedNaturalLanguageGenerator). Amend nlg\_request\_format to include a single response ID string field, instead of a list of IDs.
-   Added details to the logs of successful and failed cases of running the markers upload command.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-3 'Direct link to Improved Documentation')

-   Updated commands with square brackets e.g (`pip install rasa[spacy]`) to use quotes (`pip install 'rasa[spacy]'`) for compatibility with zsh in docs.

## \[3.6.3\] - 2023-07-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#363---2023-07-20 'Direct link to [3.6.3] - 2023-07-20')

Rasa Pro 3.6.3 (2023-07-20)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-59 'Direct link to Improvements')

-   Added a human readable component to structlog using the `event_info` key and made it the default rendered key if present.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-222 'Direct link to Bugfixes')

-   Fix the issue with the most recent model not being selected if the owner or permissions where modified on the model file.
-   Fixed `BlockingIOError` which occured as a result of too large data passed to strulogs.
-   Fixed the error handling mechanism in `rasa test e2e` to quickly detect and communicate errors when the action server, defined in endpoints.yaml, is not available.
-   Allow hyphens `-` to be present in e2e test slot names.
-   Resolved issues in `rasa test e2e` where errors occurred when the bot concluded the conversation with `SetSlot` events while there were remaining steps in the test case. Corrected the misleading error message '- No slot set' to '- Slot types do not match' in `rasa test e2e` when a type mismatch occurred during testing.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-4 'Direct link to Improved Documentation')

-   Update action server documentation with new capability to extend Sanic features by using plugins. Update rasa-sdk dependency to version 3.6.1.
-   Updated commands with square brackets e.g (`pip install rasa[spacy]`) to use quotes (`pip install 'rasa[spacy]'`) for compatibility with zsh in docs.

## \[3.6.2\] - 2023-07-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#362---2023-07-06 'Direct link to [3.6.2] - 2023-07-06')

Rasa Pro 3.6.2 (2023-07-06)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-60 'Direct link to Improvements')

-   Add building Docker container for arm64 (e.g. to allow running Rasa inside docker on M1/M2).
    
    Bumped the version of OpenTelemetry to meet the requirement of protobuf 4.x.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-223 'Direct link to Bugfixes')

-   Resolves the issue of importing TensorFlow on Docker for ARM64 architecture.

## \[3.6.1\] - 2023-07-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#361---2023-07-03 'Direct link to [3.6.1] - 2023-07-03')

Rasa Pro 3.6.1 (2023-07-03)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-61 'Direct link to Improvements')

-   Add building multi-platform Docker image (amd64/arm64)
-   Switch struct log to `FilteringBoundLogger` in order to retain log level set in the config.
-   Add new anonymizable structlog keys.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-224 'Direct link to Bugfixes')

-   Add `rasa_events` to the list of anonymizable structlog keys and rename structlog keys.
-   Introduce a validation step in `rasa data validate` and `rasa train` commands to identify non-existent paths and empty domains.
-   Rich responses containing buttons with parentheses characters are now correctly parsed. Previously any characters found between the first identified pair of `()` in response button took precedence.
-   Add PII bugfixes (e.g. handling None values and casting data types to string before being passed to the anonymizer) after testing manually with Audiocodes channel.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-5 'Direct link to Improved Documentation')

-   Update wording in Rasa Pro installation page.
-   Document new PII Management section.
-   Added Documentation for Realtime Markers Section.
-   Add "Rasa Pro Change Log" to documentation.
-   Document new Load Testing Guidelines section.
-   Changes the formatting of realtime markers documentation page

## \[3.6.0\] - 2023-06-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#360---2023-06-14 'Direct link to [3.6.0] - 2023-06-14')

Rasa Pro 3.6.0 (2023-06-14)

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-11 'Direct link to Deprecations and Removals')

-   Removed Python 3.7 support as [it reaches its end of life in June 2023](https://devguide.python.org/versions/)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-11 'Direct link to Features')

-   Implemented PII (Personally Idenfiable Information) management using Microsoft Presidio as the entity analyzer and anonymization engine. The feature covers the following:
    
    -   anonymization of Rasa events (`UserUttered`, `BotUttered`, `SlotSet`, `EntitiesAdded`) before they are streamed to Kafka event broker anonymization topics specified in `endpoints.yml`.
    -   anonymization of Rasa logs that expose PII data
    
    The main components of the feature are:
    
    -   anonymization rules that define in `endpoints.yml` the PII entities to be anonymized and the anonymization method to be used
    -   anonymization executor that executes the anonymization rules on a given text
    -   anonymization orchestrator that orchestrates the execution of the anonymization rules and publishes the anonymized event to the matched Kafka topic.
    -   anonymization pipeline that contains a list of orchestrators and is registered to a singleton provider component, which gets invoked in hook calls in Rasa Pro when the pipeline must be retrieved for anonymizing events and logs.
    
    Please read through the PII Management section in the official documentation to learn how to get started.
    
-   Implemented support for real time evaluation of Markers with the Analytics Data Pipeline, enabling you to gain valuable insights and enhance the performance of your Rasa Assistant.
    
    For this feature, we've added support for `rasa markers upload` command. Running this command validates the marker configuration file against the domain file and uploads the configuration to Analytics Data Pipeline.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-62 'Direct link to Improvements')

-   Add optional property `ids` to the nlg server request body. IDs will be transmitted to the NLG server and can be used to identify the response variation that should be used.
    
-   Add building Docker container for arm64 (e.g. to allow running Rasa inside docker on M1/M2).
    
-   Add support for Location data from Whatsapp on Twilio Channel
    
-   Add validation to `rasa train` to align validation expectations with `rasa data validate`. Add `--skip-validation` flag to disable validation and `--fail-on-validation-warnings`, `--validation-max-history` to `rasa train` to have the same options as `rasa data validate`.
    
-   Updated tensorflow to version 2.11.1 for all platforms except Apple Silicon which stays on 2.11.0 as 2.11.1 is not available yet
    
-   Slot mapping conditions accept `active_loop` specified as `null` in those cases when slots with this mapping condition should be filled only outside form contexts.
    
-   Add an optional `description` key to the Markers Configuration format. This can be used to add documentation and context about marker's usage. For example, a `markers.yml` can look like
    
    ```
    marker_name_provided:  description: “Name slot has been set”  slot_was_set: namemarker_mood_expressed:  description: “Unhappy or Great Mood was expressed”  or:    - intent: mood_unhappy    - intent: mood_great
    ```
    
-   Add `rasa marker upload` command to upload markers to the Rasa Pro Services. Usage: `rasa marker upload --config=<path-to-config-file> -d=<path-to-domain-file> -rasa-pro-services-url=<url>`.
    
-   Enhance the validation of the `anonymization` key in `endpoints.yaml` by introducing checks for required fields and duplicate IDs.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-225 'Direct link to Bugfixes')

-   Fix running custom form validation to update required slots at form activation when prefilled slots consist only of slots that are not requested by the form.
-   Anonymize `rasa_events` structlog key.
-   Fixes issue with uploading locally trained model to a cloud rasa-plus instance where the conversation does not go as expected because slots don't get set correctly, e.g. an error is logged `Tried to set non existent slot 'placeholder_slot_name'. Make sure you added all your slots to your domain file.`. This is because the updated domain during the cloud upload did not get passed to the wrapped tracker store of the `AuthRetryTrackerStore` rasa-plus component. The fix was to add domain property and setter methods to the `AuthRetryTrackerStore` component.
-   When using `rasa studio upload`, if no specific `intents` or `entities` are specified by the user, the update will now include all available `intents` or `entities`.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-6 'Direct link to Improved Documentation')

-   Explicitly set Node.js version to 12.x in order to run Docusaurus.
-   Update obselete commands in docs README.
-   Correct docker image name for `deploy-rasa-pro-services` in docs.
-   Update Compatibility Matrix.
-   Implement `rasa data split stories` to split stories data into train/test parts.
-   Updated [knowledge base action docs](https://rasa.com/docs/rasa-pro/action-server/knowledge-bases) to reflect the improvements made in `knowledge base actions` in Rasa Pro 3.6 version. This enhancement now allows users to query for the `object` attribute without the need for users to request a list of `objects` of a particular `object type` beforehand. The docs update mentions this under `:::info New in 3.6` section.
-   Fix dead link in Analytics documentation.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-59 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.5.12\] - 2023-06-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3512---2023-06-23 'Direct link to [3.5.12] - 2023-06-23')

Rasa Pro 3.5.12 (2023-06-23)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-226 'Direct link to Bugfixes')

-   Rich responses containing buttons with parentheses characters are now correctly parsed. Previously any characters found between the first identified pair of `()` in response button took precedence.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-60 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.5.11\] - 2023-06-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3511---2023-06-08 'Direct link to [3.5.11] - 2023-06-08')

Rasa Pro 3.5.11 (2023-06-08)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-227 'Direct link to Bugfixes')

-   Fix running custom form validation to update required slots at form activation when prefilled slots consist only of slots that are not requested by the form.

## \[3.5.10\] - 2023-05-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3510---2023-05-23 'Direct link to [3.5.10] - 2023-05-23')

Rasa Pro 3.5.10 (2023-05-23)

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-7 'Direct link to Improved Documentation')

-   Added documentation for spaces alpha

## \[3.5.9\] - 2023-05-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#359---2023-05-19 'Direct link to [3.5.9] - 2023-05-19')

Rasa Pro 3.5.9 (2023-05-19)

No significant changes.

## \[3.5.8\] - 2023-05-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#358---2023-05-12 'Direct link to [3.5.8] - 2023-05-12')

Rasa Pro 3.5.8 (2023-05-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-228 'Direct link to Bugfixes')

-   Explicitly handled `BufferError exception - Local: Queue full` in Kafka producer.

## \[3.5.7\] - 2023-05-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#357---2023-05-09 'Direct link to [3.5.7] - 2023-05-09')

Rasa Pro 3.5.7 (2023-05-09)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-229 'Direct link to Bugfixes')

-   `SlotSet` events will be emitted when the value set by the custom action is the same as the existing value of the slot. This was fixed for `AugmentedMemoizationPolicy` to work properly with truncated trackers.
    
    To restore the previous behaviour, the custom action can return a SlotSet only if the slot value has changed. For example,
    
    ```
    class CustomAction(Action):    def name(self) -> Text:        return "custom_action"    def run(self, dispatcher: CollectingDispatcher,            tracker: Tracker,            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:        # current value of the slot        slot_value = tracker.get_slot('my_slot')        # value of the entity        # this is parsed from the user utterance        entity_value = next(tracker.get_latest_entity_values("entity_name"), None)        if slot_value != entity_value:          return[SlotSet("my_slot", entity_value)]
    ```
    

## \[3.5.6\] - 2023-04-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#356---2023-04-28 'Direct link to [3.5.6] - 2023-04-28')

Rasa Pro 3.5.6 (2023-04-28)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-230 'Direct link to Bugfixes')

-   Addresses Regular Expression Denial of Service vulnerability in slack connector ([https://owasp.org/www-community/attacks/Regular\_expression\_Denial\_of\_Service\_-\_ReDoS](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS))
-   Fix parsing of RabbitMQ URL provided in `endpoints.yml` file to include vhost path and query parameters. Re-allows inclusion of credentials in the URL as a regression fix (this was supported in 2.x).

## \[3.5.5\] - 2023-04-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#355---2023-04-20 'Direct link to [3.5.5] - 2023-04-20')

Rasa Pro 3.5.5 (2023-04-20)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-231 'Direct link to Bugfixes')

-   Allow slot mapping parameter `intent` to accept a list of intent names (as strings), in addition to accepting an intent name as a single string.
-   Fix `BlockingIOError` when running `rasa shell` on utterances with more than 5KB of text.
-   Use `ruamel.yaml` round-trip loader in order to preserve all comments after appending `assistant_id` to `config.yml`.
-   Fix `AttributeError: 'NoneType' object has no attribute 'send_response'` caused by retrieving tracker via `GET /conversations/\{conversation_id\}/tracker` endpoint when `action_session_start` is customized in a custom action. This was addressed by passing an instance of `CollectingOutputChannel` to the method retrieving the tracker from the `MessageProcessor`.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-8 'Direct link to Improved Documentation')

-   Updated AWS model loading documentation to indicate what should `AWS_ENDPOINT_URL` environment variable be set to. Added integration test for AWS model loading.
-   Updated Rasa Pro Services documentation to add `KAFKA_SSL_CA_LOCATION` environment variable. Allows connections over SSL to Kafka
-   Added note to CLI documentation to address encoding and color issues on certain Windows terminals

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-61 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.5.4\] - 2023-04-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#354---2023-04-05 'Direct link to [3.5.4] - 2023-04-05')

Rasa Pro 3.5.4 (2023-04-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-232 'Direct link to Bugfixes')

-   Fix issue with failures while publishing events to RabbitMQ after a RabbitMQ restart. The fix consists of pinning `aio-pika` dependency to `8.2.3`, since this issue was introduced in `aio-pika` v`8.2.4`.
-   Patch redis Race Conditiion vulnerability.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-62 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.5.3\] - 2023-03-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#353---2023-03-30 'Direct link to [3.5.3] - 2023-03-30')

Rasa Pro 3.5.3 (2023-03-30)

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-9 'Direct link to Improved Documentation')

-   Add new Rasa Pro page in docs, together with minimal content changes.

## \[3.5.2\] - 2023-03-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#352---2023-03-30 'Direct link to [3.5.2] - 2023-03-30')

Rasa Pro 3.5.2 (2023-03-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-63 'Direct link to Improvements')

-   Add a self-reference of the synonym in the EntitySynonymMapper to handle entities extracted in a casing different to synonym case. (For example if a synonym `austria` is added, entities extracted with any alternate casing of the synonym will also be mapped to `austria`). It addresses ATO-616

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-233 'Direct link to Bugfixes')

-   Make custom actions inheriting from rasa-sdk `FormValidationAction` parent class an exception of the `selective_domain` rule and always send them domain.
-   Fix 2 issues detected with the HTTP API:
    -   The `GET /conversations/\{conversation_id\}/tracker` endpoint was not returning the tracker with all sessions when `include_events` query parameter was set to `ALL`. The fix constituted in using `TrackerStore.retrieve_full_tracker` method instead of `TrackerStore.retrieve` method in the function handling the `GET /conversations/\{conversation_id\}/tracker` endpoint. Implemented or updated this method across all tracker store subclasses.
    -   The `GET /conversations/\{conversation_id\}/story` endpoint was not returning all the stories for all sessions when `all_sessions` query parameter was set to `true`. The fix constituted in using all events of the tracker to be converted in stories instead of only the `applied_events`.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-10 'Direct link to Improved Documentation')

-   Add documentation for secrets managers.

## \[3.5.1\] - 2023-03-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#351---2023-03-24 'Direct link to [3.5.1] - 2023-03-24')

Rasa Pro 3.5.1 (2023-03-24)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-234 'Direct link to Bugfixes')

-   Fixes training `DIETCLassifier` on the GPU.
    
    A deterministic GPU implementation of SparseTensorDenseMatmulOp is not currently available
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-11 'Direct link to Improved Documentation')

-   Updated `Test your assistant` section to describe the new end-to-end testing feature. Also updated CLI and telemetry reference docs.
-   Update Compatibility Matrix.

## \[3.5.0\] - 2023-03-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#350---2023-03-21 'Direct link to [3.5.0] - 2023-03-21')

Rasa Pro 3.5.0 (2023-03-21)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-12 'Direct link to Features')

-   Add a new required key (`assistant_id`) to `config.yml` to uniquely identify assistants in deployment. The assistant identifier is extracted from the model metadata and added to the metadata of all dialogue events. Re-training will be required to include the assistant id in the event metadata.
    
    If the assistant identifier is missing from the `config.yml` or the default identifier value is not replaced, a random identifier is generated during each training.
    
    An assistant running without an identifier will issue a warning that dialogue events without identifier metadata will be streamed to the event broker.
    
-   End-to-end testing is an enhanced and comprehensive CLI-based testing tool that allows you to test conversation scenarios with different pre-configured contexts, execute custom actions, verify response texts or names, and assert when slots are filled. It is available ysing the new `rasa test e2e` command.
    
-   You can now store your assistant's secrets in an external credentials manager. In this release, Rasa Pro currently supports credentials manager for the Tracker Store with HashiCorp Vault.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-64 'Direct link to Improvements')

-   Add capability to send compressed body in HTTP request to action server. Use COMPRESS\_ACTION\_SERVER\_REQUEST=True to turn the feature on.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-235 'Direct link to Bugfixes')

-   Address potentially missing events with Pika consumer due to weak references on asynchronous tasks, as specifcied in [Python official documentation](https://docs.python.org/3/library/asyncio-task.html#asyncio.create_task).
-   Sets a global seed for numpy, TensorFlow, keras, Python and CuDNN, to ensure consistent random number generation.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-12 'Direct link to Improved Documentation')

-   Clarify in the docs, how rules are designed and how to use this behaviour to abort a rule

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-63 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.4.14\] - 2023-06-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3414---2023-06-08 'Direct link to [3.4.14] - 2023-06-08')

Rasa Pro 3.4.14 (2023-06-08)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-236 'Direct link to Bugfixes')

-   Fix running custom form validation to update required slots at form activation when prefilled slots consist only of slots that are not requested by the form.

## \[3.4.13\] - 2023-05-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3413---2023-05-19 'Direct link to [3.4.13] - 2023-05-19')

Rasa Pro 3.4.13 (2023-05-19)

No significant changes.

## \[3.4.12\] - 2023-05-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3412---2023-05-12 'Direct link to [3.4.12] - 2023-05-12')

Rasa Pro 3.4.12 (2023-05-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-237 'Direct link to Bugfixes')

-   Explicitly handled `BufferError exception - Local: Queue full` in Kafka producer.

## \[3.4.11\] - 2023-05-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3411---2023-05-09 'Direct link to [3.4.11] - 2023-05-09')

Rasa Pro 3.4.11 (2023-05-09)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-238 'Direct link to Bugfixes')

-   Fix parsing of RabbitMQ URL provided in `endpoints.yml` file to include vhost path and query parameters. Re-allows inclusion of credentials in the URL as a regression fix (this was supported in 2.x).
    
-   `SlotSet` events will be emitted when the value set by the custom action is the same as the existing value of the slot. This was fixed for `AugmentedMemoizationPolicy` to work properly with truncated trackers.
    
    To restore the previous behaviour, the custom action can return a SlotSet only if the slot value has changed. For example,
    
    ```
    class CustomAction(Action):    def name(self) -> Text:        return "custom_action"    def run(self, dispatcher: CollectingDispatcher,            tracker: Tracker,            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:        # current value of the slot        slot_value = tracker.get_slot('my_slot')        # value of the entity        # this is parsed from the user utterance        entity_value = next(tracker.get_latest_entity_values("entity_name"), None)        if slot_value != entity_value:          return[SlotSet("my_slot", entity_value)]
    ```
    

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-64 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.4.10\] - 2023-04-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3410---2023-04-17 'Direct link to [3.4.10] - 2023-04-17')

Rasa Pro 3.4.10 (2023-04-17)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-65 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.4.9\] - 2023-04-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#349---2023-04-05 'Direct link to [3.4.9] - 2023-04-05')

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-66 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.4.8\] - 2023-04-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#348---2023-04-03 'Direct link to [3.4.8] - 2023-04-03')

Rasa Pro 3.4.8 (2023-04-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-239 'Direct link to Bugfixes')

-   Fix issue with failures while publishing events to RabbitMQ after a RabbitMQ restart. The fix consists of pinning `aio-pika` dependency to `8.2.3`, since this issue was introduced in `aio-pika` v`8.2.4`.

## \[3.4.7\] - 2023-03-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#347---2023-03-30 'Direct link to [3.4.7] - 2023-03-30')

Rasa Pro 3.4.7 (2023-03-30)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-65 'Direct link to Improvements')

-   Add a self-reference of the synonym in the EntitySynonymMapper to handle entities extracted in a casing different to synonym case. (For example if a synonym `austria` is added, entities extracted with any alternate casing of the synonym will also be mapped to `austria`). It addresses ATO-616

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-240 'Direct link to Bugfixes')

-   Fix 2 issues detected with the HTTP API:
    -   The `GET /conversations/\{conversation_id\}/tracker` endpoint was not returning the tracker with all sessions when `include_events` query parameter was set to `ALL`. The fix constituted in using `TrackerStore.retrieve_full_tracker` method instead of `TrackerStore.retrieve` method in the function handling the `GET /conversations/\{conversation_id\}/tracker` endpoint. Implemented or updated this method across all tracker store subclasses.
    -   The `GET /conversations/\{conversation_id\}/story` endpoint was not returning all the stories for all sessions when `all_sessions` query parameter was set to `true`. The fix constituted in using all events of the tracker to be converted in stories instead of only the `applied_events`.
-   Make custom actions inheriting from rasa-sdk `FormValidationAction` parent class an exception of the `selective_domain` rule and always send them domain.

## \[3.4.6\] - 2023-03-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#346---2023-03-16 'Direct link to [3.4.6] - 2023-03-16')

Rasa Pro 3.4.6 (2023-03-16)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-241 'Direct link to Bugfixes')

-   Fixes CountVectorFeaturizer to train when min\_df != 1.

## \[3.4.5\] - 2023-03-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#345---2023-03-09 'Direct link to [3.4.5] - 2023-03-09')

Rasa Pro 3.4.5 (2023-03-09)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-242 'Direct link to Bugfixes')

-   Check unresolved slots before initiating model training.
-   Fixes the bug when a slot (with `from_intent` mapping which contains no input for `intent` parameter) will no longer fill for any intent that is not under the `not_intent` parameter.
-   Fix validation metrics calculation when batch\_size is dynamic.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-13 'Direct link to Improved Documentation')

-   Add a link to an existing docs section on how to test the audio channel on `localhost`.

## \[3.4.4\] - 2023-02-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#344---2023-02-17 'Direct link to [3.4.4] - 2023-02-17')

Rasa Pro 3.4.4 (2023-02-17)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-66 'Direct link to Improvements')

-   Add capability to send compressed body in HTTP request to action server. Use COMPRESS\_ACTION\_SERVER\_REQUEST=True to turn the feature on.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-243 'Direct link to Bugfixes')

-   Fix the error which resulted during merging multiple domain files where at least one of them contains custom actions that explicitly need `send_domain` set as True in the domain.

## \[3.4.3\] - 2023-02-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#343---2023-02-14 'Direct link to [3.4.3] - 2023-02-14')

Rasa Pro 3.4.3 (2023-02-14)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-67 'Direct link to Improvements')

-   Add support for custom RulePolicy.
-   Add capability to select which custom actions should receive domain when they are invoked.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-244 'Direct link to Bugfixes')

-   Fix calling the form validation action twice for the same user message triggering a form.
-   Fix conditional response does not check other conditions if first condition matches.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-14 'Direct link to Improved Documentation')

-   Add section in tracker store docs to document the fallback tracker store mechanism.

## \[3.4.2\] - 2023-01-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#342---2023-01-27 'Direct link to [3.4.2] - 2023-01-27')

Rasa Pro 3.4.2 (2023-01-27)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-245 'Direct link to Bugfixes')

-   Decision to publish docs should not consider next major and minor alpha release versions.
    
-   Exit training/running Rasa model when SpaCy runtime version is not compatible with the specified SpaCy model version.
    
-   The new custom logging feature was not working due to small syntax issue in the argparse level.
    
    Previously, the file name passed using the argument --logging-config-file was never retrieved so it never creates the new custom config file with the desired formatting.
    

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-67 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.4.1\] - 2023-01-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#341---2023-01-19 'Direct link to [3.4.1] - 2023-01-19')

Rasa Pro 3.4.1 (2023-01-19)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-246 'Direct link to Bugfixes')

-   Changed categorical slot comparison to be case insensitive.
-   Exit training when transformer\_size is not divisible by the number\_of\_attention\_heads parameter and update the transformer documentations.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-15 'Direct link to Improved Documentation')

-   Update compatibility matrix between Rasa-plus and Rasa Pro services.

## \[3.4.0\] - 2022-12-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#340---2022-12-14 'Direct link to [3.4.0] - 2022-12-14')

Rasa Pro 3.4.0 (2022-12-14)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-13 'Direct link to Features')

-   Add metadata to Websocket channel. Messages can now include a `metadata` object which will be included as metadata to Rasa. The metadata can be supplied on a user configurable key with the `metadata_key` setting in the `socketio` section of the `credentials.yml`.
-   Use a new IVR Channel to connect your assistant to AudioCodes VoiceAI Connect.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-68 'Direct link to Improvements')

-   Added `./docker/Dockerfile_pretrained_embeddings_spacy_it` to include Spacy's Italian pre-trained model `it_core_news_md`.
-   Replace `kafka-python` dependency with `confluent-kafka` async Producer API.
-   Add support for Python 3.10 version.
-   Added CLI option `--logging-config-file` to enable configuration of custom logs formatting.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-247 'Direct link to Bugfixes')

-   Implements a new CLI option `--jwt-private-key` required to have complete support for asymmetric algorithms as specified originally in the docs.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-16 'Direct link to Improved Documentation')

-   Clarify in the documentation how to write testing stories if a user presses a button with payload.
-   Clarify prioritisation of used slot asking option in forms in documentation.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-68 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.3.8\] - 2023-04-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#338---2023-04-06 'Direct link to [3.3.8] - 2023-04-06')

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-69 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.3.7\] - 2023-03-31[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#337---2023-03-31 'Direct link to [3.3.7] - 2023-03-31')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-69 'Direct link to Improvements')

-   Add a self-reference of the synonym in the EntitySynonymMapper to handle entities extracted in a casing different to synonym case. (For example if a synonym `austria` is added, entities extracted with any alternate casing of the synonym will also be mapped to `austria`). It addresses ATO-616

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-248 'Direct link to Bugfixes')

-   Fix issue with failures while publishing events to RabbitMQ after a RabbitMQ restart. The fix consists of pinning `aio-pika` dependency to `8.2.3`, since this issue was introduced in `aio-pika` v`8.2.4`.
-   Fix 2 issues detected with the HTTP API:
    -   The `GET /conversations/\{conversation_id\}/tracker` endpoint was not returning the tracker with all sessions when `include_events` query parameter was set to `ALL`. The fix constituted in using `TrackerStore.retrieve_full_tracker` method instead of `TrackerStore.retrieve` method in the function handling the `GET /conversations/\{conversation_id\}/tracker` endpoint. Implemented or updated this method across all tracker store subclasses.
    -   The `GET /conversations/\{conversation_id\}/story` endpoint was not returning all the stories for all sessions when `all_sessions` query parameter was set to `true`. The fix constituted in using all events of the tracker to be converted in stories instead of only the `applied_events`.

## \[3.3.6\] - 2023-03-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#336---2023-03-09 'Direct link to [3.3.6] - 2023-03-09')

Rasa Pro 3.3.6 (2023-03-09)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-249 'Direct link to Bugfixes')

-   Fixes the bug when a slot (with `from_intent` mapping which contains no input for `intent` parameter) will no longer fill for any intent that is not under the `not_intent` parameter.
-   Fix validation metrics calculation when batch\_size is dynamic.

## \[3.3.5\] - 2023-02-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#335---2023-02-21 'Direct link to [3.3.5] - 2023-02-21')

No significant changes.

## \[3.3.4\] - 2023-02-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#334---2023-02-14 'Direct link to [3.3.4] - 2023-02-14')

Rasa Pro 3.3.4 (2023-02-14)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-70 'Direct link to Improvements')

-   Add capability to send compressed body in HTTP request to action server. Use COMPRESS\_ACTION\_SERVER\_REQUEST=True to turn the feature on.
-   Add support for custom RulePolicy.

## \[3.3.3\] - 2022-12-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#333---2022-12-01 'Direct link to [3.3.3] - 2022-12-01')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-250 'Direct link to Bugfixes')

-   Bypass Windows path length restrictions upon saving and reading a model archive in `rasa.engine.storage.LocalModelStorage`.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-71 'Direct link to Improvements')

-   Updated tensorflow to 2.8.4.

## \[3.3.2\] - 2022-11-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#332---2022-11-30 'Direct link to [3.3.2] - 2022-11-30')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-72 'Direct link to Improvements')

-   Added support for camembert french bert model

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-251 'Direct link to Bugfixes')

-   Fixes `RuntimeWarning: coroutine 'Bot.set_webhook' was never awaited` issue encountered when starting the rasa server, which caused the Telegram bot to be unresponsive.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-17 'Direct link to Improved Documentation')

-   The documentation was updated for Buttons using messages that start with '/'. Previously, it wrongly stated that messages with '/' bypass NLU, which is not the case.
-   Add documentation for Rasa Pro Services upgrades.

## \[3.3.1\] - 2022-11-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#331---2022-11-09 'Direct link to [3.3.1] - 2022-11-09')

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-18 'Direct link to Improved Documentation')

-   Add docs on how to set up additional data lakes for Rasa Pro analytics pipeline.
-   Update migration guide and form docs with prescriptive recommendation on how to implement dynamic forms with custom slot mappings.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-73 'Direct link to Improvements')

-   Updated numpy and scikit learn version to fix vulnerabilities of those dependencies

## \[3.3.0\] - 2022-10-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#330---2022-10-24 'Direct link to [3.3.0] - 2022-10-24')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-14 'Direct link to Features')

-   Tracing capabilities for your Rasa Pro assistant. Distributed tracing tracks requests as they flow through a distributed system (in this case: a Rasa assistant), sending data about the requests to a tracing backend which collects all trace data and enables inspecting it. With this version of the Tracing feature, Rasa Pro supports OpenTelemetry.
-   `ConcurrentRedisLockStore` is a new lock store that uses Redis as a persistence layer and is safe for use with multiple Rasa server replicas.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-74 'Direct link to Improvements')

-   Added option `--offset-timestamps-by-seconds` to offset the timestamp of events when using `rasa export`
-   Rasa supports native installations on Apple Silicon (M1 / M2). Please follow the installation instructions and take a look at the limitations.
-   Caching `Message` and `Features` fingerprints unless they are altered, saving up to 2/3 of fingerprinting time in our tests.
-   Added package versions of component dependencies as an additional part of fingerprinting calculation. Upgrading an dependency will thus lead to a retraining of the component in the future. Also, by changing fingerprint calculation, the next training after this change will be a complete retraining.
-   Export events continuously rather than loading all events in memory first when using `rasa export`. Events will be streamed right from the start rather than loading all events first and pushing them to the broker afterwards.
-   Adds new dependency pluggy, with which it was possible to implement new plugin functionality. This plugin manager enables the extension and/or enhancement of the Rasa command line interface with functionality made available in the rasa-plus package.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-252 'Direct link to Bugfixes')

-   Fix `BlockingIOError` when running `rasa interactive`, after the upgrade of `prompt-toolkit` dependency.
-   Fixes a bug that lead to initial slot values being incorporated into all rules by default, thus breaking most rules when the slot value changed from its initial value
-   Made logistic regression classifier output a proper intent ranking and made ranking length configurable

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-12 'Direct link to Deprecations and Removals')

-   Remove code related to Rasa X local mode as it is deprecated and scheduled for removal.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-70 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.2.13\] - 2023-03-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3213---2023-03-09 'Direct link to [3.2.13] - 2023-03-09')

Rasa 3.2.13 (2023-03-09)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-253 'Direct link to Bugfixes')

-   Fix validation metrics calculation when batch\_size is dynamic.
-   Fixes the bug when a slot (with `from_intent` mapping which contains no input for `intent` parameter) will no longer fill for any intent that is not under the `not_intent` parameter.

## \[3.2.12\] - 2023-02-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3212---2023-02-21 'Direct link to [3.2.12] - 2023-02-21')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-15 'Direct link to Features')

-   Add metadata to Websocket channel. Messages can now include a `metadata` object which will be included as metadata to Rasa. The metadata can be supplied on a user configurable key with the `metadata_key` setting in the `socketio` section of the `credentials.yml`.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-75 'Direct link to Improvements')

-   Add capability to send compressed body in HTTP request to action server. Use COMPRESS\_ACTION\_SERVER\_REQUEST=True to turn the feature on.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-254 'Direct link to Bugfixes')

-   Decision to publish docs should not consider next major and minor alpha release versions.

## \[3.2.11\] - 2022-12-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3211---2022-12-05 'Direct link to [3.2.11] - 2022-12-05')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-76 'Direct link to Improvements')

-   Caching `Message` and `Features` fingerprints unless they are altered, saving up to 2/3 of fingerprinting time in our tests.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-255 'Direct link to Bugfixes')

-   Implements a new CLI option `--jwt-private-key` required to have complete support for asymmetric algorithms as specified originally in the docs.

## \[3.2.10\] - 2022-09-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3210---2022-09-29 'Direct link to [3.2.10] - 2022-09-29')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-256 'Direct link to Bugfixes')

-   Fixes scenarios in which a slot with `from_trigger_intent` mapping that specifies an `active_loop` condition was being filled despite that `active_loop` not being activated. In addition, fixes scenario in which a slot with `from_trigger_intent` mapping without a specified `active_loop` mapping condition is only filled if the form gets activated. Removes unnecessary validation warning that a slot with `from_trigger_intent` and a mapping condition should be included in the form's required\_slots.
-   Fixed a bug where `DIETClassier` crashed during training when both masked language modelling and evaluation during training were used.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-19 'Direct link to Improved Documentation')

-   Rasa SDK documentation lives now in Rasa Open Source documentation under the _Rasa SDK_ category.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-71 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.2.9\] - 2022-09-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#329---2022-09-09 'Direct link to [3.2.9] - 2022-09-09')

Yanked.

## \[3.2.8\] - 2022-09-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#328---2022-09-08 'Direct link to [3.2.8] - 2022-09-08')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-257 'Direct link to Bugfixes')

-   Fix bug where `KeywordIntentClassifier` overrides preceding intent classifiers' predictions although the `KeyWordIntentClassifier` was not matching any keywords.

## \[3.2.7\] - 2022-08-31[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#327---2022-08-31 'Direct link to [3.2.7] - 2022-08-31')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-77 'Direct link to Improvements')

-   Improve `rasa data validate` command so that it uses custom importers when they are defined in config file.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-258 'Direct link to Bugfixes')

-   Re-instates the REST channel metadata feature. Metadata can be provided on the `metadata` key.

## \[3.2.6\] - 2022-08-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#326---2022-08-12 'Direct link to [3.2.6] - 2022-08-12')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-259 'Direct link to Bugfixes')

-   This fix makes sure that when a Domain object is loaded from multiple files where one file specifies a custom session config and the rest do not, the default session configuration does not override the custom session config.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-72 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.2.5\] - 2022-08-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#325---2022-08-05 'Direct link to [3.2.5] - 2022-08-05')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-260 'Direct link to Bugfixes')

-   Fix `KeyError` which resulted when `action_two_stage_fallback` got executed in a project whose domain contained slot mappings.
-   Fixes regression in which slot mappings were prioritized according to reverse order as they were listed in the domain, instead of in order from first to last, as was implicitly expected in `2.x`. Clarifies this implicit priority order in the docs.
-   Enables the dispatching of bot messages returned as events by slot validation actions.

## \[3.2.4\] - 2022-07-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#324---2022-07-21 'Direct link to [3.2.4] - 2022-07-21')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-261 'Direct link to Bugfixes')

-   Added session\_config key as valid domain key during domain loading from directory containing a separate domain file with session configuration.
-   Run default action `action_extract_slots` after a custom action returns a `UserUttered` event to fill any applicable slots.
-   Handle the case when an `EndpointConfig` object is given as parameter to the `AwaitableTrackerStore.create()` method.

## \[3.2.3\] - 2022-07-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#323---2022-07-18 'Direct link to [3.2.3] - 2022-07-18')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-262 'Direct link to Bugfixes')

-   -   Fixed error in creating response when slack sends retry messages. Assigning `None` to `response.text` caused `TypeError: Bad body type. Expected str, got NoneType`.
    -   Fixed Slack triggering timeout after 3 seconds if the action execution is too slow. Running `on_new_message` as an asyncio background task instead of a blocking await fixes this by immediately returning a response with code 200.
-   Revert change in #10295 that removed running the form validation action on activation of the form before the loop is active.
    
-   `SlotSet` events will be emitted when the value set by the current user turn is the same as the existing value.
    
    Previously, `ActionExtractSlots` would not emit any `SlotSet` events if the new value was the same as the existing one. This caused the augmented memoization policy to lose these slot values when truncating the tracker.
    

## \[3.2.2\] - 2022-07-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#322---2022-07-05 'Direct link to [3.2.2] - 2022-07-05')

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-20 'Direct link to Improved Documentation')

-   Update documentation for customizable classes such as tracker stores, event brokers and lock stores.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-73 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.2.1\] - 2022-06-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#321---2022-06-17 'Direct link to [3.2.1] - 2022-06-17')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-263 'Direct link to Bugfixes')

-   Fix failed check in `rasa data validate` that verifies forms in rules or stories are consistent with the domain when the rule or story contains a default action as `active_loop` step.

## \[3.2.0\] - 2022-06-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#320---2022-06-14 'Direct link to [3.2.0] - 2022-06-14')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-13 'Direct link to Deprecations and Removals')

-   [NLU training data](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-training-data) in JSON format is deprecated and will be removed in Rasa Open Source 4.0. Please use `rasa data convert nlu -f yaml --data <path to NLU data>` to convert your NLU JSON data to YAML format before support for NLU JSON data is removed.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-78 'Direct link to Improvements')

-   Make `TrackerStore` interface methods asynchronous and supply an `AwaitableTrackerstore` wrapper for custom tracker stores which do not implement the methods as asynchronous.
-   Added flag `use_gpu` to `TEDPolicy` and `UnexpecTEDIntentPolicy` that can be used to enable training on CPU even when a GPU is available.
-   Add `--endpoints` command line parameter to `rasa train` parser.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-264 'Direct link to Bugfixes')

-   The azure botframework channel now validates the incoming JSON Web Tokens (including signature).
    
    Previously, JWTs were not validated at all.
    
-   `rasa shell` now outputs custom json unicode characters instead of `\uxxxx` codes
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-21 'Direct link to Improved Documentation')

-   Clarify aspects of the API spec GET /status endpoint: Correct response schema for model\_id - a string, not an object.
    
    GET /conversations/{conversation\_id}/tracker: Describe each of the enum options for include\_events query parameter
    
    POST & PUT /conversations/{conversation\_id}/tracker/eventss: Events schema added for each event type
    
    GET /conversations/{conversation\_id}/story: Clarified the all\_sessions query parameter and default behaviour.
    
    POST /model/test/intents : Remove JSON payload option since it is not supported
    
    POST /model/parse: Explain what emulation\_mode is and how it affects response results
    

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-74 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.1.7\] - 2022-08-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#317---2022-08-30 'Direct link to [3.1.7] - 2022-08-30')

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-75 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.1.6\] - 2022-07-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#316---2022-07-20 'Direct link to [3.1.6] - 2022-07-20')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-265 'Direct link to Bugfixes')

-   Run default action `action_extract_slots` after a custom action returns a `UserUttered` event to fill any applicable slots.

## \[3.1.5\] - 2022-07-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#315---2022-07-15 'Direct link to [3.1.5] - 2022-07-15')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-266 'Direct link to Bugfixes')

-   `SlotSet` events will be emitted when the value set by the current user turn is the same as the existing value.
    
    Previously, `ActionExtractSlots` would not emit any `SlotSet` events if the new value was the same as the existing one. This caused the augmented memoization policy to lose these slot values when truncating the tracker.
    

## \[3.1.4\] - 2022-06-21 No significant changes.[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#314---2022-06-21--------------------------no-significant-changes 'Direct link to [3.1.4] - 2022-06-21                          No significant changes.')

Upgrade dependent libraries with security vulnerabilities (Pillow, TensorFlow, ujson).

## \[3.1.3\] - 2022-06-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#313---2022-06-17 'Direct link to [3.1.3] - 2022-06-17')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-267 'Direct link to Bugfixes')

-   The azure botframework channel now validates the incoming JSON Web Tokens (including signature).
    
    Previously, JWTs were not validated at all.
    
-   Backports fix for failed check in `rasa data validate` that verifies forms in rules or stories are consistent with the domain when the rule or story contains a default action as `active_loop` step.
    

## \[3.1.2\] - 2022-06-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#312---2022-06-08 'Direct link to [3.1.2] - 2022-06-08')

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-76 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.1.1\] - 2022-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#311---2022-06-03 'Direct link to [3.1.1] - 2022-06-03')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-268 'Direct link to Bugfixes')

-   Remove warning for Rasa X localmode not being supported when the `--production` flag is present.
-   Pin requirement for `scipy<1.8.0` since `scipy>=1.8.0` is not backward compatible with `scipy<1.8.0` and additionally requires Python>=3.8, while Rasa supports Python 3.7 as well.
-   Fix the extraction of values for slots with mapping conditions from trigger intents that activate a form, which was possible in `2.x`.

## \[3.1.0\] - 2022-03-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#310---2022-03-25 'Direct link to [3.1.0] - 2022-03-25')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-16 'Direct link to Features')

-   Add configuration options (via env variables) for library logging.
    
-   Support other recipe types.
    
    This pull request also adds support for graph recipes, see details at [https://rasa.com/docs/rasa/model-configuration](https://rasa.com/docs/rasa/model-configuration) and check Graph Recipe page.
    
    Graph recipe is a raw format for specifying executed graph directly. This is useful if you need a more powerful way to specify your model creation.
    
-   Added optional `ssl_keyfile`, `ssl_certfile`, and `ssl_ca_certs` parameters to the Redis tracker store.
    
-   Added `LogisticRegressionClassifier` to the NLU classifiers.
    
    This model is lightweight and might help in early prototyping. The training times typically decrease substantially, but the accuracy might be a bit lower too.
    
-   Added support for Python 3.9.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-79 'Direct link to Improvements')

-   Bump TensorFlow version to 2.7.
    
    caution
    
    We can't guarantee the exact same output and hence model performance if your configuration uses `LanguageModelFeaturizer`. This applies to the case where the model is re-trained with the new rasa open source version without changing the configuration, random seeds, and data as well as to the case where a model trained with a previous version of rasa open source is loaded with this new version for inference.
    
    We suggest training a new model if you are upgrading to this version of Rasa Open Source.
    
-   Make `rasa data validate` check for duplicated intents, forms, responses and slots when using domains split between multiple files.
    
-   Add an `influence_conversation` flag to entites to provide a shorthand for ignoring an entity for all intents.
    
-   Add `--request-timeout` command line argument to `rasa shell`, allowing users to configure the time a request can take before it's terminated.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-269 'Direct link to Bugfixes')

-   Validate regular expressions in nlu training data configuration.
    
-   Unset the default values for `num_threads` and `finetuning_epoch_fraction` to `None` in order to fix cases when CLI defaults override the data from config.
    
-   Update `rasa data validate` to not fail when `active_loop` is `null`
    
-   Fixes Domain loading when domain config uses multiple yml files.
    
    Previously not all configures attributes were necessarily known when merging Domains, and in the case of `entities` were not being properly assigned to `intents`.
    
-   Fix `max_history` truncation in `AugmentedMemoizationPolicy` to preserve the most recent `UserUttered` event. Previously, `AugmentedMemoizationPolicy` failed to predict next action after long sequences of actions (longer than `max_history`) because the policy did not have access to the most recent user message.
    
-   Add `RASA_ENVIRONMENT` header in Kafka only if the environmental variable is set.
    
-   Merge domain entities as lists of dicts, not lists of lists to support entity roles and groups across multiple domains.
    
-   Add an option to specify `--domain` for `rasa test nlu` CLI command.
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-22 'Direct link to Improved Documentation')

-   Fixed an over-indent in the Tokenizers section of the Components page of the docs.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-77 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.10\] - 2022-03-15## \[3.0.10\] - 2022-03-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3010---2022-03-15-3010---2022-03-15 'Direct link to [3.0.10] - 2022-03-15## [3.0.10] - 2022-03-15')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-270 'Direct link to Bugfixes')

-   Fix broken conversion from Rasa JSON NLU data to Rasa YAML NLU data.

## \[3.0.9\] - 2022-03-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#309---2022-03-11 'Direct link to [3.0.9] - 2022-03-11')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-271 'Direct link to Bugfixes')

-   Fix Socket IO connection issues by upgrading sanic to v21.12.
    
    The bug is caused by [an invalid function signature](https://github.com/sanic-org/sanic/issues/2272) and is fixed in [v21.12](https://sanic.readthedocs.io/en/v21.12.1/sanic/changelog.html#version-21-12-0).
    
    This update brings some deprecations in `sanic`:
    
    -   Sanic and Blueprint may no longer have arbitrary properties attached to them
        -   Fixed this by moving user defined properties to the `instance.ctx` object
    -   Sanic and Blueprint forced to have compliant names
        -   Fixed this by using string literal names instead of the module's name via \_\_name\_\_
    -   `sanic.exceptions.abort` is Deprecated
        -   Fixed by replacing it with `sanic.exceptions.SanicException`
    -   `sanic.response.StreamingHTTPResponse` is deprecated
        -   Fixed by replacing it with sanic.response.ResponseStream
-   Update `rasa data validate` to not fail when `active_loop` is `null`
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-23 'Direct link to Improved Documentation')

-   Updated the `model_confidence` parameter in `TEDPolicy` and `DIETClassifier`. The `linear_norm` is removed as it is no longer supported.
-   Added an additional step to `Receiving Messages` section in slack.mdx documentation. After a slack update this additional step is needed to allow direct messages to the bot.
-   Backport the updated deployment docs to 3.0.x.

## \[3.0.8\] - 2022-02-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#308---2022-02-11 'Direct link to [3.0.8] - 2022-02-11')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-80 'Direct link to Improvements')

-   Allow single tokens in rasa end-to-end test files to be annotated with multiple entities.
    
    Some entity extractors (s.a. `RegexEntityExtractor`) can generate multiple entities from a single expression. Before this change, `rasa test` would fail in this case, because test stories could not be annotated correctly. New annotation option is
    
    ```
    stories:  - story: Some story    steps:      - user: |          cancel my [iphone][{"entity":"iphone", "value":"iphone"},{"entity":"smartphone", "value":"true"}{"entity":"mobile_service", "value":"true"}]        intent: cancel_contract
    ```
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-272 'Direct link to Bugfixes')

-   Fixed a bug where the `POST /conversations/<conversation_id>/tracker/events` endpoint repeated session start events when appending events to a new tracker.

## \[3.0.7\] - 2022-02-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#307---2022-02-09 'Direct link to [3.0.7] - 2022-02-09')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-273 'Direct link to Bugfixes')

-   Checkpoint weights were never loaded before. Implements overwriting checkpoint weights to the final model weights after training of `DIETClassifier`, `ResponseSelector` and `TEDPolicy`.
-   Allow arbitrary keys under each slot in the domain to allow for custom slot types.
-   Fix issue with missing running event loop in `MainThread` when starting Rasa Open Source for Rasa X with JWT secrets.

## \[3.0.6\] - 2022-01-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#306---2022-01-28 'Direct link to [3.0.6] - 2022-01-28')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-14 'Direct link to Deprecations and Removals')

-   Removed CompositionView.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-274 'Direct link to Bugfixes')

-   Fixes a bug which was caused by `DIETClassifier` (`ResponseSelector`, `SklearnIntentClassifier` and `CRFEntityExtractor` have the same issue) trying to process message which didn't have required features. Implements removing unfeaturized messages for the above-mentioned components before training and prediction.
-   Enable slots with `from_entity` mapping that are not part of a form's required slots to be set during active loop.
-   Catch `ValueError` for any port values that cannot be cast to integer and re-raise as `RasaException` during the initialisation of `SQLTrackerStore`.
-   Use `tf.function` for model prediction to improve inference speed.
-   Tie prompt-toolkit to ^2.0 to fix `rasa-shell`.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-24 'Direct link to Improved Documentation')

-   Update dynamic form behaviour docs section with an example on how to override `required_slots` in case of removal of a form required slot.

## \[3.0.5\] - 2022-01-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#305---2022-01-19 'Direct link to [3.0.5] - 2022-01-19')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-275 'Direct link to Bugfixes')

-   Corrects `transformer_size` parameter value (`None` by default) with a default size during loading in case `ResponseSelector` contains transformer layers.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-78 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.4\] - 2021-12-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#304---2021-12-22 'Direct link to [3.0.4] - 2021-12-22')

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-79 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.3\] - 2021-12-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#303---2021-12-16 'Direct link to [3.0.3] - 2021-12-16')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-276 'Direct link to Bugfixes')

-   Copy lookup tables to train and test folds in cross validation. Before, the generated folds did not have a copy of the lookup tables from the original NLU data, so that `RegexEntityExtractor` could not recognize any entities during the evaluation.
-   Do not print warning when subintent actions have response.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-80 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.2\] - 2021-12-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#302---2021-12-09 'Direct link to [3.0.2] - 2021-12-09')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-277 'Direct link to Bugfixes')

-   Update SQLAlchemy version to a compatible one in case other dependencies force a lower version.
-   Fix overriding of default config with custom config containing nested dictionaries. Before, the keys of a nested dictionary in the default config that were not specified in the custom config got lost.
-   Add `UserWarning` to alert users trying to run `rasa x` CLI command with rasa version 3.0 or higher that rasa-x currently doesn't support rasa 3.x.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-25 'Direct link to Improved Documentation')

-   Added note to the slot mappings section of the migration guide to recommend checking dynamic form behavior on migrated assistants.

## \[3.0.1\] - 2021-12-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#301---2021-12-02 'Direct link to [3.0.1] - 2021-12-02')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-278 'Direct link to Bugfixes')

-   Fix previous slots getting filled after a restart. Previously events were searched from oldest to newest which meant we would find first occurrence of a message and use slots from thereafter. Now we use the last utterance or the restart event.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-81 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.0\] - 2021-11-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#300---2021-11-23 'Direct link to [3.0.0] - 2021-11-23')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-15 'Direct link to Deprecations and Removals')

-   Remove backwards compatibility code with Rasa Open Source 1.x, Rasa Enterprise 0.35, and other outdated backwards compatibility code in `rasa.cli.x`, `rasa.core.utils`, `rasa.model_testing`, `rasa.model_training` and `rasa.shared.core.events`.
    
-   Removed Python 3.6 support as [it reaches its end of life in December 2021](https://www.python.org/dev/peps/pep-0494/#lifespan).
    
-   Follow through on removing deprecation warnings for synchronous `EventBroker` methods.
    
-   Follow through on deprecation warnings for policies and policy ensembles.
    
-   Follow through on deprecation warnings for `rasa.shared.data`.
    
-   Follow through on deprecation warnings for the `Domain`. Most importantly this will enforce the schema of the [`forms` section](https://legacy-docs-oss.rasa.com/docs/rasa/forms) in the domain file. This further includes the removal of the `UnfeaturizedSlot` type.
    
-   Remove deprecated `change_form_to` and `set_form_validation` methods from `DialogueStateTracker`.
    
-   Remove the support of Markdown training data format. This includes:
    
    -   reading and writing of story files in Markdown format
    -   reading and writing of NLU data in Markdown format
    -   reading and writing of retrieval intent data in Markdown format
    -   all the Markdown examples and tests that use Markdown
-   Removed automatic renaming of deprecated action `action_deactivate_form` to `action_deactivate_loop`. `action_deactivate_form` will just be treated like other non-existing actions from now on.
    
-   Remove deprecated `sorted_intent_examples` method from `TrainingData`.
    
-   Raising `RasaException` instead of deprecation warning when using `class_from_module_path` for loading types other than classes.
    
-   Specifying the `retrieve_events_from_previous_conversation_sessions` kwarg for the any `TrackerStore` was deprecated and has now been removed. Please use the `retrieve_full_tracker()` method instead.
    
    Deserialization of pickled trackers was deprecated and has now been removed. Rasa will perform any future save operations of trackers using json serialisation.
    
    Removed catch for missing (deprecated) `session_date` when saving trackers in `DynamoTrackerStore`.
    
-   Removed the deprecated dialogue policy state featurizers: `BinarySingleStateFeature` and `LabelTokenizerSingleStateFeaturizer`.
    
    Removed the deprecated method `encode_all_actions` of `SingleStateFeaturizer`. Use `encode_all_labels` instead.
    
-   Follow through with removing deprecated policies: `FormPolicy`, `MappingPolicy`, `FallbackPolicy`, `TwoStageFallbackPolicy`, and `SklearnPolicy`.
    
    Remove warning about default value of `max_history` in MemoizationPolicy. The default value is now `None`.
    
-   Follow through on deprecation warnings and remove code, tests, and docs for `ConveRTTokenizer`, `LanguageModelTokenizer` and `HFTransformersNLP`.
    
-   `rasa.shared.nlu.training_data.message.Message` method `get_combined_intent_response_key` has been removed. `get_full_intent` should now be used in its place.
    
-   Intent IDs sent with events (to kafka and elsewhere) have been removed, intent names can be used instead (or if numerical values are needed for backwards compatibility, one can also hash the names to get previous ID values, ie. `hash(intent_name)` is the old ID values). Intent IDs have been removed because they were providing no extra value and integers that large were problematic for some event broker implementations.
    
-   Remove `loop` argument from `train` method in `rasa`. This argument became redundant when Python 3.6 support was dropped as `asyncio.run` became available in Python 3.7.
    
-   Remove `template_variables` and `e2e` arguments from `get_stories` method of `TrainingDataImporter`. This argument was used in Markdown data format and became redundant once Markdown was removed.
    
-   `weight_sparsity` has been removed. Developers should replace it with `connection_density` in the following way: `connection_density` = 1-`weight_sparsity`.
    
    `softmax` is not available as a `loss_type` anymore.
    
    The `linear_norm` option has been removed as possible value for `model_confidence`. Please, use `softmax` instead.
    
    `minibatch` has been removed as a value for `tensorboard_log_level`, use `batch` instead.
    
    Removed deprecation warnings related to the removed component config values.
    
-   Follow through on removing deprecation warnings raised in these modules:
    
    -   `rasa/server.py`
        
    -   `rasa/core/agent.py`
        
    -   `rasa/core/actions/action.py`
        
    -   `rasa/core/channels/mattermost.py`
        
    -   `rasa/core/nlg/generator.py`
        
    -   `rasa/nlu/registry.py`
        
-   Remove deprecation warnings associated with the `"number_additional_patterns"` parameter of `rasa.nlu.featurizers.sparse_featurizer.regex_featurizer.RegexFeaturizer`. This parameter is no longer needed for incremental training.
    
    Remove deprecation warnings associated with the `"additional_vocabulary_size"` parameter of `rasa.nlu.featurizers.sparse_featurizer.count_vectors_featurizer.CountVectorsFeaturizer`. This parameter is no longer needed for incremental training.
    
    Remove deprecated functions `training_states_actions_and_entities` and `training_states_and_actions` from `rasa.core.featurizers.tracker_featurizers.TrackerFeaturizer`. Use `training_states_labels_and_entities` and `training_states_and_labels` instead.
    
-   Follow through on deprecation warning for `NGramFeaturizer`
    
-   The CLI commands `rasa data convert config` and `rasa data convert responses` which converted from the Rasa Open Source 1 to the Rasa Open Source 2 formats were removed. Please use a Rasa Open Source 2 installation to convert your training data before moving to Rasa Open Source 3.
    
-   `rasa.core.agent.Agent.visualize` was removed. Please use `rasa visualize` or `rasa.core.visualize.visualize` instead.
    
-   Removed slot auto-fill functionality, making the key invalid to use in the domain file. The `auto_fill` parameter was also removed from the constructor of the `Slot` class. In order to continue filling slots with entities of the same name, you now have to define a `from_entity` mapping in the `slots` section of the domain. To learn more about how to migrate your 2.0 assistant, please read the migration guide.
    

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-17 'Direct link to Features')

-   Training data version upgraded from `2.0` to `3.0` due to breaking changes to format in Rasa Open Source 3.0
    
-   A new experimental feature called `Markers` has been added. `Markers` allow you to define points of interest in conversations as a set of conditions that need to be met. A new command `rasa evaluate markers` allows you to apply these conditions to your existing tracker stores and outputs the points at which the conditions were satisfied.
    
-   Rasa Open Source now uses the [model configuration](https://legacy-docs-oss.rasa.com/docs/rasa/model-configuration) to build a
    
    [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph). This graph describes the dependencies between the items in your model configuration and how data flows between them. This has two major benefits:
    
    -   Rasa Open Source can use the computational graph to optimize the execution of your model. Examples for this are efficient caching of training steps or executing independent steps in parallel.
    -   Rasa Open Source can represent different model architectures flexibly. As long as the graph remains acyclic Rasa Open Source can in theory pass any data to any graph component based on the model configuration without having to tie the underlying software architecture to the used model architecture.
    
    This change required changes to custom policies and custom NLU components. See the documentation for a detailed [migration guide](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide#custom-policies-and-custom-components).
    
-   Added explicit mechanism for slot filling that allows slots to be set and/or updated throughout the conversation. This mechanism is enabled by defining global slot mappings in the `slots` section of the domain file.
    
    In order to support this new functionality, implemented a new default action: `action_extract_slots`. This new action runs after each user turn and checks if any slots can be filled with information extracted from the last user message based on defined slot mappings.
    
    Since slot mappings were moved away from the `forms` section of the domain file, converted the form's `required_slots` to a list of slot names. In order to restrict certain mappings to a form, you can now use the `conditions` key in the mapping to define the applicable `active_loop`, like so:
    
    ```
    slots:  location:    type: text    influence_conversation: false    mappings:    - type: from_entity      entity: city      conditions:      - active_loop: booking_form
    ```
    
    To learn more about how to migrate your 2.0 assistant, please read the migration guide.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-81 'Direct link to Improvements')

-   Updated the `/status` endpoint response payload, and relevant documentation, to return/reflect the updated 3.0 keys/values.
    
-   Bump TensorFlow version to 2.6.
    
    This update brings some security benefits (see TensorFlow [release notes](https://github.com/tensorflow/tensorflow/releases/tag/v2.6.0) for details). However, internal experiments suggest that it is also associated with increased train and inference time, as well as increased memory usage.
    
    You can read more about why we decided to update TensorFlow, and what the expected impact is [here](https://rasa.com/blog/let-s-talk-about-tensorflow-2-6/).
    
    If you experience a significant increase in train time, inference time, and/or memory usage, please let us know in the [forum](https://forum.rasa.com/t/feedback-upgrading-to-tensorflow-2-6/48331).
    
    Users can no longer set `TF_DETERMINISTIC_OPS=1` if they are using GPU(s) because a `tf.errors.UnimplementedError` will be thrown by TensorFlow (read more [here](https://github.com/tensorflow/tensorflow/releases/tag/v2.6.0)).
    
    caution
    
    This **breaks backward compatibility of previously trained models**. It is not possible to load models trained with previous versions of Rasa Open Source. Please re-train your assistant before trying to use this version.
    
-   Added authentication support for connecting to external RabbitMQ servers. Currently user has to hardcode a username and a password in a URL in order to connect to an external RabbitMQ server.
    
-   1.  Failed test stories will display full retrieval intents.
        
    2.  Retrieval intents will be extracted during action prediction in test stories so that we won't have unnecessary mismatches anymore.
        
    
    Let's take this example story:
    
    ```
    - story: test story  steps:  - user: |      what is your name?    intent: chitchat/ask_name  - action: utter_chitchat/ask_name  - intent: bye  - action: utter_bye
    ```
    
    Before:
    
    ```
      steps:  - intent: chitchat   # 1) intent is not displayed in it's original form  - action: utter_chitchat/ask_name  # predicted: utter_chitchat                  # 2) retrieval intent is not extracted during action prediction and we have a mismatch  - intent: bye  # some other fail  - action: utter_bye # some other fail
    ```
    
    Both 1) and 2) problems are solved.
    
    Now:
    
    ```
      steps:  - intent: chitchat/ask_name  - action: utter_chitchat/ask_name  - intent: bye  # some other fail  - action: utter_bye # some other fail
    ```
    
-   Added `-i` command line option to make RASA listen on a specific ip-address instead of any network interface
    
-   `rasa data validate` now checks that forms referenced in `active_loop` directives are defined in the domain
    
-   Every conversation event now includes in its metadata the ID of the model which loaded at the time it was created.
    
-   Send indices of user message tokens along with the `UserUttered` event through the event broker to Rasa X.
    
-   Added optional flag to convert intent ID hashes from integer to string in the `KafkaEventBroker`.
    
-   Make it possible to use `or` functionality for `slot_was_set` events.
    
-   Upgraded the spaCy dependency from version 3.0 to 3.1.
    
-   Implemented `fingerprint` methods in these classes:
    
    -   `Event`
    -   `Slot`
    -   `DialogueStateTracker`
-   Added debug message that logs when a response condition is used.
    
-   The naming scheme for trained models was changed. Unless you provide a `--fixed-model-name` to `rasa train`, Rasa Open Source will now generate a new model name using the schema `<timestamp>-<random name>.tar.gz`, e.g.
    
    -   `20211018-094821-composite-pita.tar.gz` (for a model containing a trained NLU and dialogue model)
    -   `nlu-20211018-094821-composite-pita.tar.gz` (for a model containing only a trained NLU model but not a dialogue model)
    -   `core-20211018-094821-composite-pita.tar.gz` (for a model containing only a trained dialogue model but no NLU model)
-   Due to changes in the model architecture the behavior of `rasa train --dry-run` changed. The exit codes now have the following meaning:
    
    -   `0` means that the model does not require an expensive retraining. However, the responses might still require updating by running `rasa train`
    -   `1` means that one or multiple components require to be retrained.
    -   `8` means that the `--force` flag was used and hence any cached results are ignored and the entire model is retrained.
-   Machine learning components like `DIETClassifier`, `ResponseSelector` and `TEDPolicy` using a `ranking_length` parameter will no longer report renormalised confidences for the top predictions by default.
    
    A new parameter `renormalize_confidences` is added to these components which if set to `True`, renormalizes the confidences of top `ranking_length` number of predictions to sum up to 1. The default value is `False`, which means no renormalization will be applied by default. It is advised to leave it to `False` but if you are trying to reproduce the results from previous versions of Rasa Open Source, you can set it to `True`.
    
    Renormalization will only be applied if `model_confidence=softmax` is used.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-279 'Direct link to Bugfixes')

-   Fixed validation behavior and logging output around unused intents and utterances.
-   `rasa test nlu --cross-validation` uses autoconfiguration when no pipeline is defined instead of failing
-   Update DynamoDb tracker store to correctly retrieve all `sender_ids` from a DynamoDb table.
-   Fix for `failed_test_stories.yml` not printing the correct message when the extracted entity specified in a test story is incorrect.
-   Fix CVE-2021-41127

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-26 'Direct link to Improved Documentation')

-   Added new docs for Markers.
-   Update pip in same command which installs rasa and clarify supported version in docs.
-   Update `pika` consumer code in Event Brokers documentation.
-   Adds documentation on how to use `CRFEntityExtractor` with features from a dense featurizer (e.g. `LanguageModelFeaturizer`).
-   Updated docs (Domain, Forms, Default Actions, Migration Guide, CLI) to provide more detail over the new slot mappings changes.
-   Updated documentation publishing mechanisms to build one version of [the documentation](https://rasa.com/docs/rasa) for each major version of Rasa Open Source, starting from 2.x upwards. Previously, we were building one version of the documentation for each minor version of Rasa Open Source, resulting in a poor user experience and high maintenance costs.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-82 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.8.16\] - 2021-12-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2816---2021-12-09 'Direct link to [2.8.16] - 2021-12-09')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-82 'Direct link to Improvements')

-   The value of the `RASA_ENVIRONMENT` environmental variable is sent as a header in messages logged by `KafkaEventBroker`. This value was previously only made available by `PikaEventConsumer`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-280 'Direct link to Bugfixes')

-   Make `action_metadata` json serializable and make it available on the tracker. This is a backport of a fix in 3.0.0.

## \[2.8.15\] - 2021-11-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2815---2021-11-25 'Direct link to [2.8.15] - 2021-11-25')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-281 'Direct link to Bugfixes')

-   Validate regular expressions in nlu training data configuration.

## \[2.8.14\] - 2021-11-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2814---2021-11-18 'Direct link to [2.8.14] - 2021-11-18')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-282 'Direct link to Bugfixes')

-   Bump TensorFlow version to 2.6.2. _We have plans to port this change to 3.x (see [this issue](https://github.com/RasaHQ/rasa/issues/10378))_.
-   Downgrade google-auth to <2.

## \[2.8.13\] - 2021-11-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2813---2021-11-11 'Direct link to [2.8.13] - 2021-11-11')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-283 'Direct link to Bugfixes')

-   Fixed new intent creation in `rasa interactive` command. Previously, this failed with 500 from the server due to `UnexpecTEDIntentPolicy` trying to predict with the new intent not in domain.
-   Install mitie library when preparing test runs. This step was missing before and tests were thus failing as we have many tests which rely on mitie library. Previously, `make install-full` was required.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-83 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.8.12\] - 2021-10-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2812---2021-10-21 'Direct link to [2.8.12] - 2021-10-21')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-284 'Direct link to Bugfixes')

-   Fixed a bug where `rasa test --fail-on-prediction-errors` would raise a `WrongPredictionException` for entities which were actually predicted correctly.
    
    This happened in two ways:
    
    1.  if for a user message some entities were extracted multiple times (by multiple entity extractors) but listed only once in the test story,
    2.  if the order in which entities from a message were extracted didn't match the order in which they were listed in the test story.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-27 'Direct link to Improved Documentation')

-   Improve the documentation for training `TEDPolicy` with data augmentation.

## \[2.8.11\] - 2021-10-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2811---2021-10-20 'Direct link to [2.8.11] - 2021-10-20')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-285 'Direct link to Bugfixes')

-   Updates dependency on `sanic-jwt` (1.5.0 -> ">=1.6.0, <1.7.0")
    
    This removes the need to pin the version of `pyjwt` as the newer version of `sanic-jwt` manages this properly.
    

## \[2.8.10\] - 2021-10-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2810---2021-10-14 'Direct link to [2.8.10] - 2021-10-14')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-286 'Direct link to Bugfixes')

-   Add List handling in the `send_custom_json` method on `channels/facebook.py`. Bellow are some examples that could cause en error before.
    
    Example 1: when the whole json is a List
    
    ```
    [    {        "blocks": {            "type": "progression_bar",            "text": {"text": "progression 1", "level": "1"},        }    },    {"sender": {"id": "example_id"}},]
    ```
    
    Example 2: instead of being a Dict, _blocks_ is a List when there are 2 _type_ keys under it
    
    ```
    {    "blocks": [        {"type": "title", "text": {"text": "Conversation progress"}},        {            "type": "progression_bar",            "text": {"text": "Look how far we are...", "level": "1"},        },    ]}
    ```
    
-   Fixed bug when using wit.ai training data to train. Training failed with an error similarly to this:
    
    ```
      File "./venv/lib/python3.8/site-packages/rasa/nlu/classifiers/diet_classifier.py", line 803, in train    self.check_correct_entity_annotations(training_data)  File "./venv/lib/python3.8/site-packages/rasa/nlu/extractors/extractor.py", line 418, in check_correct_entity_annotations    entities_repr = [  File "./venv/lib/python3.8/site-packages/rasa/nlu/extractors/extractor.py", line 422, in \<listcomp\>    entity[ENTITY_ATTRIBUTE_VALUE],KeyError: 'value'
    ```
    
-   Fix CVE-2021-41127
    

## \[2.8.9\] - 2021-10-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#289---2021-10-08 'Direct link to [2.8.9] - 2021-10-08')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-83 'Direct link to Improvements')

-   Bump TensorFlow version to 2.6.
    
    This update brings some security benefits (see TensorFlow [release notes](https://github.com/tensorflow/tensorflow/releases/tag/v2.6.0) for details). However, internal experiments suggest that it is also associated with increased train and inference time, as well as increased memory usage.
    
    You can read more about why we decided to update TensorFlow, and what the expected impact is [here](https://rasa.com/blog/let-s-talk-about-tensorflow-2-6/).
    
    If you experience a significant increase in train time, inference time, and/or memory usage, please let us know in the [forum](https://forum.rasa.com/t/feedback-upgrading-to-tensorflow-2-6/48331).
    
    Users can no longer set `TF_DETERMINISTIC_OPS=1` if they are using GPU(s) because a `tf.errors.UnimplementedError` will be thrown by TensorFlow (read more [here](https://github.com/tensorflow/tensorflow/releases/tag/v2.6.0)).
    
    caution
    
    This **breaks backward compatibility of previously trained models**. It is not possible to load models trained with previous versions of Rasa Open Source. Please re-train your assistant before trying to use this version.
    

## \[2.8.8\] - 2021-10-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#288---2021-10-06 'Direct link to [2.8.8] - 2021-10-06')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-84 'Direct link to Improvements')

-   Added a function to display the actual text of a Token when inspecting a Message in a pipeline, making it easier to debug.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-28 'Direct link to Improved Documentation')

-   Removing the experimental feature warning for `conditional response variations` from the Rasa docs. The behaviour of the feature remains unchanged.
-   Updates [quick install documentation](https://rasa.com/docs/rasa-pro/installation/python/installation) with optional venv step, better pip install instructions, & M1 warning

## \[2.8.7\] - 2021-09-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#287---2021-09-20 'Direct link to [2.8.7] - 2021-09-20')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-287 'Direct link to Bugfixes')

-   Explicitly set the upper limit for currently compatible TensorFlow versions.

## \[2.8.6\] - 2021-09-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#286---2021-09-09 'Direct link to [2.8.6] - 2021-09-09')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-288 'Direct link to Bugfixes')

-   Fix rules not being applied when a featurised categorical slot has as one of its allowed values `none`, `NoNe`, `None` or a similar value.

## \[2.8.5\] - 2021-09-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#285---2021-09-06 'Direct link to [2.8.5] - 2021-09-06')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-289 'Direct link to Bugfixes')

-   AugmentedMemoizationPolicy is accelerated for large trackers
-   Bump tensorflow to 2.3.4 to address security vulnerabilities

## \[2.8.4\] - 2021-09-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#284---2021-09-02 'Direct link to [2.8.4] - 2021-09-02')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-85 'Direct link to Improvements')

-   Increase speed of augmented lookup for `AugmentedMemoizationPolicy`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-290 'Direct link to Bugfixes')

-   Fix `--data` being treated as if non-optional on sub-commands of `rasa data convert`
-   Fixes bug where `hide_rule_turn` was defaulting to `None` when ActionExecuted was deserialised.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-84 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.8.3\] - 2021-08-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#283---2021-08-19 'Direct link to [2.8.3] - 2021-08-19')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-291 'Direct link to Bugfixes')

-   Ignore checking that intent is in domain for E2E story utterances when running `rasa data validate`. Previously data validation would fail on E2E stories.

## \[2.8.2\] - 2021-08-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#282---2021-08-04 'Direct link to [2.8.2] - 2021-08-04')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-292 'Direct link to Bugfixes')

-   Fixes a bug which caused training of `UnexpecTEDIntentPolicy` to crash when end-to-end training stories were included in the training data.
    
    Stories with end-to-end training data will now be skipped for the training of `UnexpecTEDIntentPolicy`.
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-29 'Direct link to Improved Documentation')

-   Removing the experimental feature warning for the `story validation` tool from the rasa docs. The behaviour of the feature remains unchanged.
-   Removing the experimental feature warning for `entity roles and groups` from the rasa docs, and from the code where it previously appeared as a print statement. The behaviour of the feature remains otherwise unchanged.

## \[2.8.1\] - 2021-07-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#281---2021-07-22 'Direct link to [2.8.1] - 2021-07-22')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-86 'Direct link to Improvements')

-   Add support for `cafile` parameter in `endpoints.yaml`. This will load a custom local certificate file and use it when making requests to that endpoint.
    
    For example:
    
    ```
    action_endpoint:  url: https://localhost:5055/webhook  cafile: ./cert.pem
    ```
    
    This means that requests to the action server `localhost:5055` will use the certificate `cert.pem` located in the current working directory.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-293 'Direct link to Bugfixes')

-   Fixes wrong overriding of `epochs` parameter when `TEDPolicy` or `UnexpecTEDIntentPolicy` is not loaded in finetune mode.

## \[2.8.0\] - 2021-07-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#280---2021-07-12 'Direct link to [2.8.0] - 2021-07-12')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-16 'Direct link to Deprecations and Removals')

-   The option `model_confidence=linear_norm` is deprecated and will be removed in Rasa Open Source `3.0.0`.
    
    Rasa Open Source `2.3.0` introduced `linear_norm` as a possible value for `model_confidence` parameter in machine learning components such as `DIETClassifier`, `ResponseSelector` and `TEDPolicy`. Based on user feedback, we have identified multiple problems with this option. Therefore, `model_confidence=linear_norm` is now deprecated and will be removed in Rasa Open Source `3.0.0`. If you were using `model_confidence=linear_norm` for any of the mentioned components, we recommend to revert it back to `model_confidence=softmax` and re-train the assistant. After re-training, we also recommend to [re-tune the thresholds for fallback components](https://legacy-docs-oss.rasa.com/docs/rasa/fallback-handoff/#fallbacks).
    
-   The fallback mechanism for spaCy models has now been removed in Rasa `3.0.0`.
    
    Rasa Open Source `2.5.0` introduced support for spaCy 3.0. This introduced a breaking feature because models would no longer be manually linked. To make the transition smooth Rasa would rely on the `language` parameter in the `config.yml` to fallback to a medium spaCy model if no model was configured for the `SpacyNLP` component. In Rasa Open Source `3.0.0` and onwards the `SpacyNLP` component will require the model name (like `"en_core_web_md"`) to be passed explicitly.
    

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-18 'Direct link to Features')

-   Added `sasl_mechanism` as an optional configurable parameters for the [Kafka Producer](https://rasa.com/docs/rasa-pro/production/event-brokers#kafka-event-broker).
    
-   Introduces a new policy called [`UnexpecTEDIntentPolicy`](https://legacy-docs-oss.rasa.com/docs/rasa/policies#unexpected-intent-policy).
    
    `UnexpecTEDIntentPolicy` helps you review conversations and also allows your bot to react to unexpected user turns in conversations. It is an auxiliary policy that should only be used in conjunction with at least one other policy, as the only action that it can trigger is the special and newly introduced [`action_unlikely_intent`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#action_unlikely_intent) action.
    
    The auto-configuration will include `UnexpecTEDIntentPolicy` in your configuration automatically, but you can also include it yourself in the `policies` section of the configuration:
    
    ```
    policies:  - name: UnexpecTEDIntentPolicy    epochs: 200    max_history: 5
    ```
    
    As part of the feature, it also introduces:
    
    -   [`IntentMaxHistoryTrackerFeaturizer`](https://legacy-docs-oss.rasa.com/docs/rasa/policies#3-intent-max-history) to featurize the trackers for `UnexpecTEDIntentPolicy`.
    -   `MultiLabelDotProductLoss` to support `UnexpecTEDIntentPolicy`'s multi-label training objective.
    -   A new default action called [`action_unlikely_intent`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#action_unlikely_intent).
    
    `rasa test` command has also been adapted to `UnexpecTEDIntentPolicy`:
    
    -   If a test story contains `action_unlikely_intent` and the policy ensemble does not trigger it, this leads to a test error (wrongly predicted action) and the corresponding story will be logged in `failed_test_stories.yml`.
    -   If the story does not contain `action_unlikely_intent` and Rasa Open Source does predict it then the prediction of `action_unlikely_intent` will be ignored for the evaluation (and hence not lead to a prediction error) but the story will be logged in a file called `stories_with_warnings.yml`.
    
    The `rasa data validate` command will warn if `action_unlikely_intent` is included in the training stories. Accordingly, `YAMLStoryWriter` and `MarkdownStoryWriter` have been updated to not dump `action_unlikely_intent` when writing stories to a file.
    
    caution
    
    The introduction of a new default action **breaks backward compatibility of previously trained models**. It is not possible to load models trained with previous versions of Rasa Open Source. Please re-train your assistant before trying to use this version.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-87 'Direct link to Improvements')

-   Added detailed json schema validation for `UserUttered`, `SlotSet`, `ActionExecuted` and `EntitiesAdded` events both sent and received from the action server, as well as covered at high-level the validation of the rest of the 20 events. In case the events are invalid, a `ValidationError` will be raised.
    
-   Users don't need to specify an additional buffer size for sparse featurizers anymore during incremental training.
    
    Space for new sparse features are created dynamically inside the downstream machine learning models - `DIETClassifier`, `ResponseSelector`. In other words, no extra buffer is created in advance for additional vocabulary items and space will be dynamically allocated for them inside the model.
    
    This means there's no need to specify `additional_vocabulary_size` for [`CountVectorsFeaturizer`](https://legacy-docs-oss.rasa.com/docs/rasa/components#countvectorsfeaturizer) or `number_additional_patterns` for [`RegexFeaturizer`](https://legacy-docs-oss.rasa.com/docs/rasa/components#regexfeaturizer). These parameters are now deprecated.
    
    **Before**
    
    ```
    pipeline:  - name: "WhitespaceTokenizer"  - name: "RegexFeaturizer"    number_additional_patterns: 100  - name: "CountVectorsFeaturizer"    additional_vocabulary_size: {text: 100, response: 20}
    ```
    
    **Now**
    
    ```
    pipeline:  - name: "WhitespaceTokenizer"  - name: "RegexFeaturizer"  - name: "CountVectorsFeaturizer"
    ```
    
    Also, all custom layers specifically built for machine learning models - `RasaSequenceLayer`, `RasaFeatureCombiningLayer` and `ConcatenateSparseDenseFeatures` now inherit from `RasaCustomLayer` so that they support flexible incremental training out of the box.
    
-   Speed up the contradiction check of the [`RulePolicy`](https://legacy-docs-oss.rasa.com/docs/rasa/policies#rule-policy) by a factor of 3.
    
-   Change the confidence score assigned by [`FallbackClassifier`](https://legacy-docs-oss.rasa.com/docs/rasa/components#fallbackclassifier) to fallback intent to be the same as the fallback threshold.
    
-   Issue a UserWarning if a specified **domain folder** contains files that look like YML files but cannot be parsed successfully. Only invoked if user specifies a folder path in `--domain` paramater. Previously those invalid files in the specified folder were silently ignored. **Does not apply** to individually specified domain YAML files, e.g. `--domain /some/path/domain.yml`, those being invalid will still raise an exception.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-294 'Direct link to Bugfixes')

-   Fix for unnecessary retrain and duplication of folders in the model

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-85 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.7.2\] - 2021-08-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#272---2021-08-09 'Direct link to [2.7.2] - 2021-08-09')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-295 'Direct link to Bugfixes')

-   Ignore checking that intent is in domain for E2E story utterances when running `rasa data validate`. Previously data validation would fail on E2E stories.
-   Fix for unnecessary retrain and duplication of folders in the model

## \[2.7.1\] - 2021-06-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#271---2021-06-16 'Direct link to [2.7.1] - 2021-06-16')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-296 'Direct link to Bugfixes')

-   Best model checkpoint allows for metrics to be equal to previous best if at least one metric improves, rather than strict improvement for each metric.
    
-   Fixes a bug where multiple plots overlap each other and are rendered incorrectly when comparing performance across multiple NLU pipelines.
    
-   Don't evaluate entities if no entities present in test data.
    
    Also, catch exception in `plot_paired_histogram` when data is empty.
    

## \[2.7.0\] - 2021-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#270---2021-06-03 'Direct link to [2.7.0] - 2021-06-03')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-88 'Direct link to Improvements')

-   Changed the default config to train the `RulePolicy` before the `TEDPolicy`. This means that conflicting rule/stories will be identified before a potentially slow training of the `TEDPolicy`.
-   Updated validator used by `rasa data validate` to verify that actions used in stories and rules are present in the domain and that form slots match domain slots.
-   Rename `plot_histogram` to `plot_paired_histogram` and fix missing bars in the plot.
-   Changed --data option type in the \`\`rasa data validate\`\`\` command to allow more than one path to be passed.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-297 'Direct link to Bugfixes')

- The file `failed_test_stories.yml` (generated by `rasa test`) now also includes the wrongly predicted entity as a comment next to the entity of a user utterance. Additionally, the comment printed next to the intent of a user utterance is printed only if the intent was wrongly predicted (irrelevantly if there was a wrongly predicted entity or not in the specific user utterance).
- Added check in PikaEventBroker constructor: if port cannot be cast to integer, raise RasaException
- Fixed bug where missing intent warnings appear when running `rasa test`
- Update `should_retrain` function to return the correct fingerprint comparison result even when there is a problem with model unpacking.
- Handle correctly Telegram edited message.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-86 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.6.3\] - 2021-05-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#263---2021-05-28 'Direct link to [2.6.3] - 2021-05-28')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-298 'Direct link to Bugfixes')

- `ResponseSelector` can now be trained with the transformer enabled (i.e. when a positive `number_of_transformer_layers` is provided) even if one doesn't specify the transformer's size. Previously, not specifying `transformer_size` led to an error.
- Return `EntityEvaluationResult` during evaluation of test stories only if `parsed_message` is not `None`.
- Ignore `OSError` in Sentry reporting.
- Replaced `ValueError` with `RasaException` in TED model `_check_data` method.
- Changed import to fix agent creation in Jupyter.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-87 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.6.2\] - 2021-05-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#262---2021-05-18 'Direct link to [2.6.2] - 2021-05-18')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-299 'Direct link to Bugfixes')

- Fixed a bug where [`ListSlot`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#list-slot)s were filled with single items in case only one matching entity was extracted for this slot.
 
 Values applied to [`ListSlot`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#list-slot)s will be converted to a `List` in case they aren't one.
 
- Fix bug with false rule conflicts
 
 This essentially reverts [PR 8446](https://github.com/RasaHQ/rasa/pull/8446/files), except for the tests. The PR is redundant due to [PR 8646](https://github.com/RasaHQ/rasa/pull/8646/files).
 
- Handle `AttributeError` thrown by empty slot mappings in domain form through refactoring.
 
- Fixed incorrect `The action 'utter_\<response selector intent\>' is used in the stories, but is not a valid utterance action` error when running `rasa data validate` with response selector responses in the domain file.
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-30 'Direct link to Improved Documentation')

- Added a note to clarify best practice for resetting all slots after form deactivation.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-88 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.6.1\] - 2021-05-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#261---2021-05-11 'Direct link to [2.6.1] - 2021-05-11')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-300 'Direct link to Bugfixes')

- Made `SchemaError` message available to validator so that the reason why reason schema validation fails during `rasa data validate` is displayed when response `text` value is `null`. Added warning message when deprecated MappingPolicy format is used in the domain.
 
- When there are multiple entities in a user message, they will get sorted when creating a representation of the current dialogue state.
 
 Previously, the ordering was random, leading to inconsistent state representations. This would sometimes lead to memoization policies failing to recall a memorised action.
 

## \[2.6.0\] - 2021-05-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#260---2021-05-06 'Direct link to [2.6.0] - 2021-05-06')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-17 'Direct link to Deprecations and Removals')

- In forms, the keyword `required_slots` should always precede the definition of slot mappings and the lack of it is deprecated. Please see the [migration guide](https://rasa.com/docs/rasa-pro/migration-guide) for more information.
- `rasa.data.get_test_directory`, `rasa.data.get_core_nlu_directories`, and `rasa.shared.nlu.training_data.training_data.TrainingData::get_core_nlu_directories` are deprecated and will be removed in Rasa Open Source 3.0.0.
- Update the minimum compatible model version to "2.6.0". This means all models trained with an earlier version will have to be retrained.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-19 'Direct link to Features')

- Feature enhancement enabling JWT authentication for the Socket.IO channel. Users can define `jwt_key` and `jwt_method` as parameters in their credentials file for authentication.
 
- Allows a Rasa bot to be connected to a Twilio Voice channel. More details in the [Twilio Voice docs](https://rasa.com/docs/rasa-pro/connectors/twilio-voice)
 
- Conditional response variations are supported in the `domain.yml` without requiring users to write custom actions code.
 
 A condition can be a list of slot-value mapping constraints.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-89 'Direct link to Improvements')

- Added an optional `ignored_intents` parameter in forms.
 
 - To use it, add the `ignored_intents` parameter in your `domain.yml` file after the forms name and provide a list of intents to ignore. Please see [Forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) for more information.
 - This can be used in case the user never wants to fill any slots of a form with the specified intent, e.g. chitchat.
- Add function to carry `max_history` to featurizer
 
- Improved the machine learning models' codebase by factoring out shared feature-processing logic into three custom layer classes:
 
 - `ConcatenateSparseDenseFeatures` combines multiple sparse and dense feature tensors into one.
 - `RasaFeatureCombiningLayer` additionally combines sequence-level and sentence-level features.
 - `RasaSequenceLayer` is used for attributes with sequence-level features; it additionally embeds the combined features with a transformer and facilitates masked language modeling.
- Added the following usability improvements with respect to entities getting extracted multiple times:
 
 - Added warnings for competing entity extractors at training time and for overlapping entities at inference time
 - Improved docs to help users handle overlapping entity problems.
- Replace `weight_sparsity` with `connection_density` in all transformer-based models and add guarantees about internal layers.
 
 We rename `DenseWithSparseWeights` into `RandomlyConnectedDense`, and guarantee that even at density zero the output is dense and every input is connected to at least one output. The former `weight_sparsity` parameter of DIET, TED, and the ResponseSelector, is now roughly equivalent to `1 - connection_density`, except at very low densities (high sparsities).
 
 All layers and components that used to have a `sparsity` argument (`Ffnn`, `TransformerRasaModel`, `MultiHeadAttention`, `TransformerEncoderLayer`, `TransformerEncoder`) now have a `density` argument instead.
 
- Rasa test now prints a warning if the test stories contain bot utterances that are not part of the domain.
 
- Updated `asyncio.Task.all_tasks` to `asyncio.all_tasks`, with a fallback for python 3.6, which raises an AttributeError for `asyncio.all_tasks`. This removes the deprecation warning for the `Task.all_tasks` usage.
 
- Change variable name from `i` to `array_2D`
 
- Implement a new interface `run_inference` inside `RasaModel` which performs batch inferencing through tensorflow models.
 
 `rasa_predict` inside `RasaModel` has been made a private method now by changing it to `_rasa_predict`.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-301 'Direct link to Bugfixes')

- Fixed a bug for plotting trackers with non-ascii texts during interactive training by enforcing utf-8 encoding
- Fix masked language modeling in DIET to only apply masking to token-level (sequence-level) features. Previously, masking was applied to both token-level and sentence-level features.
- Make it possible to use `null` entities in stories.
- Introduce a `skip_validation` flag in order to speed up reading YAML files that were already validated.
- Fixed a bug in interactive training that lead to crashes for long Chinese, Japanese, or Korean user or bot utterances.

## \[2.5.2\] - 2021-06-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#252---2021-06-16 'Direct link to [2.5.2] - 2021-06-16')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-20 'Direct link to Features')

- Added `sasl_mechanism` as an optional configurable parameters for the [Kafka Producer](https://rasa.com/docs/rasa-pro/production/event-brokers#kafka-event-broker).

## \[2.5.1\] - 2021-04-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#251---2021-04-28 'Direct link to [2.5.1] - 2021-04-28')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-302 'Direct link to Bugfixes')

- Fixed prediction for rules with multiple entities.
- Mitigated Matplotlib backend issue using lazy configuration and added a more explicit error message to guide users.

## \[2.5.0\] - 2021-04-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#250---2021-04-12 'Direct link to [2.5.0] - 2021-04-12')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-18 'Direct link to Deprecations and Removals')

- The following import abbreviations were removed:
 - `rasa.core.train`: Please use `rasa.core.train.train` instead.
 - `rasa.core.visualize`: Please use `rasa.core.visualize.visualize` instead.
 - `rasa.nlu.train`: Please use `rasa.nlu.train.train` instead.
 - `rasa.nlu.test`: Please use `rasa.nlu.test.run_evaluation` instead.
 - `rasa.nlu.cross_validate`: Please use `rasa.nlu.test.cross_validate` instead.

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-21 'Direct link to Features')

- Upgraded Rasa to be compatible with spaCy 3.0.
 
 This means that we can support more features for more languages but there are also a few changes.
 
 SpaCy 3.0 deprecated the `spacy link \<language model\>` command so that means that from now on [the full model name](https://spacy.io/models) needs to be used in the `config.yml` file.
 
 **Before**
 
 Before you could run `spacy link en en_core_web_md` and then we would be able to pick up the correct model from the `language` parameter.
 
 ```
    language: enpipeline:   - name: SpacyNLP
    ```
 
 **Now**
 
 This behavior will be deprecated and instead you'll want to be explicit in `config.yml`.
 
 ```
    language: enpipeline:   - name: SpacyNLP     model: en_core_web_md
    ```
 
 **Fallback**
 
 To make the transition easier, Rasa will try to fall back to a medium spaCy model when-ever a compatible language is configured for the entire pipeline in `config.yml` even if you don't specify a `model`. This fallback behavior is temporary and will be deprecated in Rasa 3.0.0.
 
 We've updated our docs to reflect these changes. All examples now show a direct link to the correct spaCy model. We've also added a warning to the [SpaCyNLP](https://legacy-docs-oss.rasa.com/docs/rasa/components#spacynlp) docs that explains the fallback behavior.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-90 'Direct link to Improvements')

- Improved CLI startup time.
 
- Add `augmentation` and `num_threads` arguments to API `POST /model/train`
 
 Fix boolean casting issue for `force_training` and `save_to_default_model_directory` arguments
 
- Add minimum compatible version to --version command
 
- Updated warning for unexpected slot events during prediction time to Rasa Open Source 2.0 YAML training data format.
 
- Hide dialogue turns predicted by `RulePolicy` in the tracker states for ML-only policies like `TEDPolicy` if those dialogue turns only appear as rules in the training data and do not appear in stories.
 
 Add `set_shared_policy_states(...)` method to all policies. This method sets `_rule_only_data` dict with keys:
 
 - `rule_only_slots`: Slot names, which only occur in rules but not in stories.
 - `rule_only_loops`: Loop names, which only occur in rules but not in stories.
 
 This information is needed for correct featurization to hide dialogue turns that appear only in rules.
 
- Faster reading of YAML NLU training data files.
 
- Added partition\_by\_sender flag to [Kafka Producer](https://rasa.com/docs/rasa-pro/production/event-brokers#kafka-event-broker) to optionally associate events with Kafka partition based on sender\_id.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-303 'Direct link to Bugfixes')

- Fixed the 'loading model' message which was logged twice when using `rasa run`.
 
- Change training data validation to only count nlu training examples.
 
- Rule tracker states no longer include the initial value of slots. Rules now only require slot values when explicitly stated in the rule.
 
- `rasa test`, `rasa test core` and `rasa test nlu` no longer show temporary paths in case there are issues in the test files.
 
- Resolved memory problems with dense features and `CRFEntityExtractor`
 
- Handle empty intent and entity mapping in the `domain`.
 
 There is now an InvalidDomain exception raised if in the `domain.yml` file there are empty intent or entity mappings. An example of empty intent and entity mappings is the following :
 
 ```
    intents:  - greet:  - goodbye:entities:  - cuisine:  - number:
    ```
 
- Fixed a bug in a form where slot mapping doesn't work if the predicted intent name is substring for another intent name.
 
- Fixes bug where stories could not be retrieved if entities had no start or end.
 
- Catch ChannelNotFoundEntity exception coming from the pika broker and raise as ConnectionException.
 
- Fix bug with NoReturn throwing an exception in Python 3.7.0 when running `rasa train`
 
- Throw `RasaException` instead of `ValueError` in situations when environment variables specified in YAML cannot be expanded.
 
- Updated python-engineio version for compatibility with python-socketio
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-89 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.4.3\] - 2021-03-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#243---2021-03-26 'Direct link to [2.4.3] - 2021-03-26')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-304 'Direct link to Bugfixes')

- Fixes bug where stories could not be retrieved if entities had no start or end.

## \[2.4.2\] - 2021-03-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#242---2021-03-25 'Direct link to [2.4.2] - 2021-03-25')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-305 'Direct link to Bugfixes')

- Fix `UnicodeException` in `is_key_in_yaml`.
- Fixed the bug that events from previous conversation sessions would be re-saved in the [`SQLTrackerStore`](https://rasa.com/docs/rasa-pro/production/tracker-stores#sqltrackerstore) or [`MongoTrackerStore`](https://rasa.com/docs/rasa-pro/production/tracker-stores#mongotrackerstore) when `retrieve_events_from_previous_conversation_sessions` was true.

## \[2.4.1\] - 2021-03-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#241---2021-03-23 'Direct link to [2.4.1] - 2021-03-23')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-306 'Direct link to Bugfixes')

- Fix `TEDPolicy` training e2e entities when no entities are present in the stories but there are entities in the domain.
- Fixed missing model configuration file validation.
- In Rasa 2.4.0, support for using `template` in `utter_message` when handling a custom action was wrongly deprecated. Both `template` and `response` are now supported, though note that `template` will be deprecated at Rasa 3.0.0.

## \[2.4.0\] - 2021-03-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#240---2021-03-11 'Direct link to [2.4.0] - 2021-03-11')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-19 'Direct link to Deprecations and Removals')

- NLG Server
 
 - Changed request format to send `response` as well as `template` as a field. The `template` field will be removed in Rasa Open Source 3.0.0.
 
 `rasa.core.agent`
 
 - The terminology `template` is deprecated and replaced by `response`. Support for `template` from the NLG response will be removed in Rasa Open Source 3.0.0. Please see [here](https://rasa.com/docs/rasa-pro/production/nlg) for more details.
 
 `rasa.core.nlg.generator`
 
 - `generate()` now takes in `utter_action` as a parameter.
 - The terminology `template` is deprecated and replaced by `response`. Support for `template` in the `NaturalLanguageGenerator` will be removed in Rasa Open Source 3.0.0.
 
 `rasa.shared.core.domain`
 
 - The property `templates` is deprecated. Use `responses` instead. It will be removed in Rasa Open Source 3.0.0.
 - `retrieval_intent_templates` will be removed in Rasa Open Source 3.0.0. Please use `retrieval_intent_responses` instead.
 - `is_retrieval_intent_template` will be removed in Rasa Open Source 3.0.0. Please use `is_retrieval_intent_response` instead.
 - `check_missing_templates` will be removed in Rasa Open Source 3.0.0. Please use `check_missing_responses` instead.
 
 Response Selector
 
 - The field `template_name` will be deprecated in Rasa Open Source 3.0.0. Please use `utter_action` instead. Please see [here](https://legacy-docs-oss.rasa.com/docs/rasa/components#selectors) for more details.
 - The field `response_templates` will be deprecated in Rasa Open Source 3.0.0. Please use `responses` instead. Please see [here](https://legacy-docs-oss.rasa.com/docs/rasa/components#selectors) for more details.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-91 'Direct link to Improvements')

- The following endpoints now require the existence of the conversation for the specified conversation ID, raising an exception and returning a 404 status code.
 
 - `GET /conversations/\<conversation_id:path\>/story`
 
 - `POST /conversations/\<conversation_id:path\>/execute`
 
 - `POST /conversations/\<conversation_id:path\>/predict`
 
- Simplify our training by overwriting `train_step` instead of `fit` for our custom models.
 
 This allows us to use the build-in callbacks from Keras, such as the [Tensorboard Callback](https://www.tensorflow.org/api_docs/python/tf/keras/callbacks/TensorBoard), which offers more functionality compared to what we had before.
 
 warning
 
 If you want to use Tensorboard for `DIETClassifier`, `ResponseSelector`, or `TEDPolicy` and log metrics after every (mini)batch, please use 'batch' instead of 'minibatch' as 'tensorboard\_log\_level'.
 
- When `TED` is configured to extract entities `rasa test` now evaluates them against the labels in the test stories. Results are saved in `/results` along with the results for the NLU components that extract entities.
 
- We're now running integration tests for Rasa Open Source, with initial coverage for `SQLTrackerStore` (with PostgreSQL), `RedisLockStore` (with Redis) and `PikaEventBroker` (with RabbitMQ). The integration tests are now part of our CI, and can also be ran locally using `make test-integration` (see [Rasa Open Source README](https://github.com/RasaHQ/rasa#running-the-integration-tests) for more information).
 
- Allow tests to be located anywhere, not just in `tests` directory.
 
- Model configuration files are now validated whether they match the expected schema.
 
- Speed up `YAMLStoryReader.is_key_in_yaml` function by making it to check if key is in YAML without actually parsing the text file.
 
- Speed up YAML parsing by reusing parsers, making the process of environment variable interpolation optional, and by not adding duplicating implicit resolvers and YAML constructors to `ruamel.yaml`
 
- Drastically improved finger printing time for large story graphs
 
- Remove console logging of conversation level F1-score and precision since these calculations were not meaningful.
 
 Add conversation level accuracy to core policy results logged to file in `story_report.json` after running `rasa test core` or `rasa test`.
 
- Improved the [lock store](https://rasa.com/docs/rasa-pro/production/lock-stores) debug log message when the process has to queue because other messages have to be processed before this item.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-307 'Direct link to Bugfixes')

- Fixed the bug that OR statements in stories would break the check whether a model needs to be retrained
 
- Update the spec of `POST /model/test/intents` and add tests for cases when JSON is provided.
 
 Fix the incorrect temporary file extension for the data that gets extracted from the payload provided in the body of `POST /model/test/intents` request.
 
- Fix for the cli command `rasa data convert config` when migrating Mapping Policy and no rules.
 
 Making `rasa data convert config` migrate correctly the Mapping Policy when no rules are available. It updates the `config.yml` file by removing the `MappingPolicy` and adding the `RulePolicy` instead. Also, it creates the `data/rules.yml` file even if empty in the case of no available rules.
 
- Allow to have slots with values that result to a dictionary under the key `slot_was_set` (in `stories.yml` file).
 
 An example would be to have the following story step in `stories.yml`:
 
 ```
    - slot_was_set:    - some_slot:        some_key: 'some_value'        other_key: 'other_value'
    ```
 
 This would be allowed if the `some_slot` is also set accordingly in the `domain.yml` with type `any`.
 
- Update the fingerprinting function to recognize changes in lookup files.
 
- Fixed a bug when interpolating environment variables in YAML files which included `$` in their value. This led to the following stack trace:
 
 ```
    ValueError: Error when trying to expand the environment variables in '${PASSWORD}'. Please make sure to also set these environment variables: '['$qwerty']'.(13 additional frame(s) were not displayed)...  File "rasa/utils/endpoints.py", line 26, in read_endpoint_config    content = rasa.shared.utils.io.read_config_file(filename)  File "rasa/shared/utils/io.py", line 527, in read_config_file    content = read_yaml_file(filename)  File "rasa/shared/utils/io.py", line 368, in read_yaml_file    return read_yaml(read_file(filename, DEFAULT_ENCODING))  File "rasa/shared/utils/io.py", line 349, in read_yaml    return yaml_parser.load(content) or {}  File "rasa/shared/utils/io.py", line 314, in env_var_constructor    " variables: '{}'.".format(value, not_expanded)
    ```
 
- The REQUESTED\_SLOT always belongs to the currently active form.
 
 Previously it was possible that after form switching, the REQUESTED\_SLOT was for the previous form.
 
- Update the `LanguageModelFeaturizer` tests to reflect new default model weights for `bert`, and skip all `bert` tests with default model weights on CI, run `bert` tests with `bert-base-uncased` on CI instead.
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-31 'Direct link to Improved Documentation')

- Update links to Sanic docs in the documentation.
- Update Rasa Playground to correctly use `tracking_id` when calling API methods.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-90 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.3.5\] - 2021-06-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#235---2021-06-16 'Direct link to [2.3.5] - 2021-06-16')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-22 'Direct link to Features')

- Added `sasl_mechanism` as an optional configurable parameters for the [Kafka Producer](https://rasa.com/docs/rasa-pro/production/event-brokers#kafka-event-broker).

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-92 'Direct link to Improvements')

- Drastically improved finger printing time for large story graphs
- Improved the [lock store](https://rasa.com/docs/rasa-pro/production/lock-stores) debug log message when the process has to queue because other messages have to be processed before this item.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-308 'Direct link to Bugfixes')

- Fixed the bug that OR statements in stories would break the check whether a model needs to be retrained
- Updated `python-engineio` dependency version for compatibility with `python-socketio`.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-32 'Direct link to Improved Documentation')

- Update links to Sanic docs in the documentation.

## \[2.3.4\] - 2021-02-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#234---2021-02-26 'Direct link to [2.3.4] - 2021-02-26')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-309 'Direct link to Bugfixes')

- Setting `model_confidence=cosine` in `DIETClassifier`, `ResponseSelector` and `TEDPolicy` is deprecated and will no longer be available. This was introduced in Rasa Open Source version `2.3.0` but post-release experiments suggest that using cosine similarity as model's confidences can change the ranking of predicted labels which is wrong.
 
 `model_confidence=inner` is deprecated and is replaced by `model_confidence=linear_norm` as the former produced an unbounded range of confidences which broke the logic of assistants in various other places.
 
 We encourage you to try `model_confidence=linear_norm` which will produce a linearly normalized version of dot product similarities with each value in the range `[0,1]`. This can be done with the following config:
 
 ```
    - name: DIETClassifier  model_confidence: linear_norm  constrain_similarities: True
    ```
 
 This should ease up [tuning fallback thresholds](https://legacy-docs-oss.rasa.com/docs/rasa/fallback-handoff/#fallbacks) as confidences for wrong predictions are better distributed across the range `[0, 1]`.
 
 If you trained a model with `model_confidence=cosine` or `model_confidence=inner` setting using previous versions of Rasa Open Source, please re-train by either removing the `model_confidence` option from the configuration or setting it to `linear_norm`.
 
 `model_confidence=cosine` is removed from the configuration generated by [auto-configuration](https://legacy-docs-oss.rasa.com/docs/rasa/model-configuration#suggested-config).
 

## \[2.3.3\] - 2021-02-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#233---2021-02-25 'Direct link to [2.3.3] - 2021-02-25')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-310 'Direct link to Bugfixes')

- Fixed bug where the conversation does not lock before handling a reminder event.

## \[2.3.2\] - 2021-02-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#232---2021-02-22 'Direct link to [2.3.2] - 2021-02-22')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-311 'Direct link to Bugfixes')

- Fix a bug where, if a user injects an intent using the HTTP API, slot auto-filling is not performed on the entities provided.

## \[2.3.1\] - 2021-02-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#231---2021-02-17 'Direct link to [2.3.1] - 2021-02-17')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-312 'Direct link to Bugfixes')

- Fixed a YAML validation error which happened when executing multiple validations concurrently. This could e.g. happen when sending concurrent requests to server endpoints which process YAML training data.

## \[2.3.0\] - 2021-02-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#230---2021-02-11 'Direct link to [2.3.0] - 2021-02-11')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-93 'Direct link to Improvements')

- Expose diagnostic data for action and NLU predictions.
 
 Add `diagnostic_data` field to the Message and Prediction objects, which contain information about attention weights and other intermediate results of the inference computation. This information can be used for debugging and fine-tuning, e.g. with [RasaLit](https://github.com/RasaHQ/rasalit).
 
 For examples of how to access the diagnostic data, see [here](https://gist.github.com/JEM-Mosig/c6e15b81ee70561cb72e361aff310d7e).
 
- Using the `TrainingDataImporter` interface to load the data in `rasa test core`.
 
 Failed test stories are now referenced by their absolute path instead of the relative path.
 
- Improve error handling and Sentry tracking:
 
 - Raise `MarkdownException` when training data in Markdown format cannot be read.
 - Raise `InvalidEntityFormatException` error instead of `json.JSONDecodeError` when entity format is in valid in training data.
 - Gracefully handle empty sections in endpoint config files.
 - Introduce `ConnectionException` error and raise it when `TrackerStore` and `EventBroker` cannot connect to 3rd party services, instead of raising exceptions from 3rd party libraries.
 - Improve `rasa.shared.utils.common.class_from_module_path` function by making sure it always returns a class. The function currently raises a deprecation warning if it detects an anomaly.
 - Ignore `MemoryError` and `asyncio.CancelledError` in Sentry.
 - `rasa.shared.utils.validation.validate_training_data` now raises a `SchemaValidationError` when validation fails (this error inherits `jsonschema.ValidationError`, ensuring backwards compatibility).
- Allow `PolicyEnsemble` in cases where calling individual policy's `load` method returns `None`.
 
- User message metadata can now be accessed via the default slot `session_started_metadata` during the execution of a [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization).
 
 ```
    from typing import Any, Text, Dict, Listfrom rasa_sdk import Action, Trackerfrom rasa_sdk.events import SlotSet, SessionStarted, ActionExecuted, EventTypeclass ActionSessionStart(Action):    def name(self) -> Text:        return "action_session_start"    async def run(      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]    ) -> List[Dict[Text, Any]]:        metadata = tracker.get_slot("session_started_metadata")        # Do something with the metadata        print(metadata)        # the session should begin with a `session_started` event and an `action_listen`        # as a user message follows        return [SessionStarted(), ActionExecuted("action_listen")]
    ```
 
- Add BILOU tagging schema for entity extraction in end-to-end TEDPolicy.
 
- Added two new parameters `constrain_similarities` and `model_confidence` to machine learning (ML) components - [DIETClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier), [ResponseSelector](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier) and [TEDPolicy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#ted-policy).
 
 Setting `constrain_similarities=True` adds a sigmoid cross-entropy loss on all similarity values to restrict them to an approximate range in `DotProductLoss`. This should help the models to perform better on real world test sets. By default, the parameter is set to `False` to preserve the old behaviour, but users are encouraged to set it to `True` and re-train their assistants as it will be set to `True` by default from Rasa Open Source 3.0.0 onwards.
 
 Parameter `model_confidence` affects how model's confidence for each label is computed during inference. It can take three values:
 
 1. `softmax` - Similarities between input and label embeddings are post-processed with a softmax function, as a result of which confidence for all labels sum up to 1.
 2. `cosine` - Cosine similarity between input label embeddings. Confidence for each label will be in the range `[-1,1]`.
 3. `inner` - Dot product similarity between input and label embeddings. Confidence for each label will be in an unbounded range.
 
 Setting `model_confidence=cosine` should help users tune the fallback thresholds of their assistant better. The default value is `softmax` to preserve the old behaviour, but we recommend using `cosine` as that will be the new default value from Rasa Open Source 3.0.0 onwards. The value of this option does not affect how confidences are computed for entity predictions in `DIETClassifier` and `TEDPolicy`.
 
 With both the above recommendations, users should configure their ML component, e.g. `DIETClassifier`, as
 
 ```
    - name: DIETClassifier  model_confidence: cosine  constrain_similarities: True  ...
    ```
 
 Once the assistant is re-trained with the above configuration, users should also tune fallback confidence thresholds.
 
 Configuration option `loss_type=softmax` is now deprecated and will be removed in Rasa Open Source 3.0.0 . Use `loss_type=cross_entropy` instead.
 
 The default [auto-configuration](https://legacy-docs-oss.rasa.com/docs/rasa/model-configuration#suggested-config) is changed to use `constrain_similarities=True` and `model_confidence=cosine` in ML components so that new users start with the recommended configuration.
 
 **EDIT**: Some post-release experiments revealed that using `model_confidence=cosine` is wrong as it can change the order of predicted labels. That's why this option was removed in Rasa Open Source version `2.3.3`. `model_confidence=inner` is deprecated as it produces an unbounded range of confidences which can break the logic of assistants in various other places. Please use `model_confidence=linear_norm` which will produce a linearly normalized version of dot product similarities with each value in the range `[0,1]`. Please read more about this change under the notes for release `2.3.4`.
 
- Use simple random uniform distribution of integers in negative sampling, because negative sampling with `tf.while_loop` and random shuffle inside creates a memory leak.
 
- Added support to configure `exchange_name` for [pika event broker](https://rasa.com/docs/rasa-pro/production/event-brokers#pika-event-broker).
 
- If `MaxHistoryTrackerFeaturizer` is used, invert the dialogue sequence before passing it to the transformer so that the last dialogue input becomes the first one and therefore always have the same positional encoding.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-313 'Direct link to Bugfixes')

- Fixed an error when using the endpoint `GET /conversations/\<conversation_id:path\>/story` with a tracker which contained slots.
 
- Add the option to configure whether extracted entities should be split by comma (`","`) or not to TEDPolicy. Fixes crash when this parameter is accessed during extraction.
 
- When switching forms, the next form will always correctly ask for the first required slot.
 
 Before, the next form did not ask for the slot if it was the same slot as the requested slot of the previous form.
 
- Fix the bug when `RulePolicy` handling loop predictions are overwritten by e2e `TEDPolicy`.
 
- When switching forms, the next form is cleanly activated.
 
 Before, the next form was correctly activated, but the previous form had wrongly uttered the response that asked for the requested slot when slot validation for that slot had failed.
 
- Fix a bug in incremental training when passing a specific model path with the `--finetune` argument.
 
- Fix the role of `unidirectional_encoder` in TED. This parameter is only applied to transformers for `text`, `action_text` and `label_action_text`.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-91 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.2.10\] - 2021-02-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2210---2021-02-08 'Direct link to [2.2.10] - 2021-02-08')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-94 'Direct link to Improvements')

- Updated error message when using incompatible model versions.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-314 'Direct link to Bugfixes')

- Limit `numpy` version to `< 1.2` as `tensorflow` is not compatible with `numpy` versions `>= 1.2`. `pip` versions `<= 20.2` don't resolve dependencies conflicts correctly which could result in an incompatible `numpy` version and the following error:
 
 ```
    NotImplementedError: Cannot convert a symbolic Tensor (strided_slice_6:0) to a numpy array. This error may indicate that you're trying to pass a Tensor to a NumPy call, which is not supported
    ```
 

## \[2.2.9\] - 2021-02-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#229---2021-02-02 'Direct link to [2.2.9] - 2021-02-02')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-315 'Direct link to Bugfixes')

- Correctly include the `confused_with` field in the test report for the [`POST /model/test/intents`](https://rasa.com/docs/reference/api/pro/http-api/) endpoint.

## \[2.2.8\] - 2021-01-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#228---2021-01-28 'Direct link to [2.2.8] - 2021-01-28')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-316 'Direct link to Bugfixes')

- Fixes a bug in [forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) where the next slot asked was not consistent after returning to a form from an unhappy path.

## \[2.2.7\] - 2021-01-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#227---2021-01-25 'Direct link to [2.2.7] - 2021-01-25')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-95 'Direct link to Improvements')

- Add support for in `RasaYAMLWriter` for writing intent and example metadata back into NLU YAML files.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-317 'Direct link to Bugfixes')

- Fixed a bug with `Domain.is_domain_file()` that could raise an Exception in case the potential domain file is not a valid YAML.

## \[2.2.6\] - 2021-01-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#226---2021-01-21 'Direct link to [2.2.6] - 2021-01-21')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-318 'Direct link to Bugfixes')

- Fix wrong warning `The method 'EventBroker.close' was changed to be asynchronous` when the `EventBroker.close` was actually asynchronous.
 
- Fix incremental training for cases when training data does not contain entities but `DIETClassifier` is configured to perform entity recognition also.
 
 Now, the instance of `RasaModelData` inside `DIETClassifier` does not contain `entities` as a feature for training if there is no training data present for entity recognition.
 

## \[2.2.5\] - 2021-01-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#225---2021-01-12 'Direct link to [2.2.5] - 2021-01-12')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-319 'Direct link to Bugfixes')

- Fixed key-error bug on `rasa data validate stories`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-92 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.2.4\] - 2021-01-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#224---2021-01-08 'Direct link to [2.2.4] - 2021-01-08')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-96 'Direct link to Improvements')

- Improve the warning in case the [RulePolicy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#rule-policy) or the deprecated [MappingPolicy](https://rasa.com/docs/rasa/2.x/policies#mapping-policy) are missing from the model's `policies` configuration. Changed the info log to a warning as one of this policies should be added to the model configuration.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-320 'Direct link to Bugfixes')

- Explicitly specify the `crypto` extra dependency of `pyjwt` to ensure that the `cryptography` dependency is installed. `cryptography` is strictly required to be able to be able to verify JWT tokens.

## \[2.2.3\] - 2021-01-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#223---2021-01-06 'Direct link to [2.2.3] - 2021-01-06')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-321 'Direct link to Bugfixes')

- Correctly retrieve intent ranking from `UserUttered` even during default affirmation action implementation.
 
- Fixed a problem when using the `POST /model/test/intents` endpoint together with a [model server](https://rasa.com/docs/rasa-pro/production/model-storage#load-model-from-server). The error looked as follows:
 
 ```
    ERROR    rasa.core.agent:agent.py:327 Could not load model due to Detected inconsistent loop usage. Trying to schedule a task on a new event loop, but scheduler was created with a different event loop. Make sure there is only one event loop in use and that the scheduler is running on that one.
    ```
 
 This also fixes a problem where testing a model from a model server would change the production model.
 

## \[2.2.2\] - 2020-12-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#222---2020-12-21 'Direct link to [2.2.2] - 2020-12-21')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-322 'Direct link to Bugfixes')

- Fixed incompatibility between Rasa Open Source 2.2.x and Rasa X < 0.35.

## \[2.2.1\] - 2020-12-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#221---2020-12-17 'Direct link to [2.2.1] - 2020-12-17')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-323 'Direct link to Bugfixes')

- Fixed a problem where a [form](https://legacy-docs-oss.rasa.com/docs/rasa/forms) wouldn't reject when the `FormValidationAction` re-implemented `required_slots`.
 
- Fixed an error when using the [SQLTrackerStore](https://rasa.com/docs/rasa-pro/production/tracker-stores#sqltrackerstore) with a Postgres database and the parameter `login_db` specified.
 
 The error was:
 
 ```
    psycopg2.errors.SyntaxError: syntax error at end of inputrasa-production_1  | LINE 1: SELECT 1 FROM pg_catalog.pg_database WHERE datname = ?
    ```
 

## \[2.2.0\] - 2020-12-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#220---2020-12-16 'Direct link to [2.2.0] - 2020-12-16')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-20 'Direct link to Deprecations and Removals')

- `Domain.random_template_for` is deprecated and will be removed in Rasa Open Source 3.0.0. You can alternatively use the `TemplatedNaturalLanguageGenerator`.
 
 `Domain.action_names` is deprecated and will be removed in Rasa Open Source 3.0.0. Please use `Domain.action_names_or_texts` instead.
 
- Interfaces for `Policy.__init__` and `Policy.load` have changed. See [migration guide](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide/#rasa-21-to-rasa-22) for details.
 
- Deprecate training and test data in Markdown format. This includes:
 
 - reading and writing of story files in Markdown format
 - reading and writing of NLU data in Markdown format
 - reading and writing of retrieval intent data in Markdown format
 
 Support for Markdown data will be removed entirely in Rasa Open Source 3.0.0.
 
 Please convert your existing Markdown data by using the commands from the [migration guide](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide/#rasa-21-to-rasa-22):
 
 ```
    rasa data convert nlu -f yaml --data={SOURCE_DIR} --out={TARGET_DIR}rasa data convert nlg -f yaml --data={SOURCE_DIR} --out={TARGET_DIR}rasa data convert core -f yaml --data={SOURCE_DIR} --out={TARGET_DIR}
    ```
 
- `Domain.add_categorical_slot_default_value`, `Domain.add_requested_slot` and `Domain.add_knowledge_base_slots` are deprecated and will be removed in Rasa Open Source 3.0.0. Their internal versions are now called during the Domain creation. Calling them manually is no longer required.
 

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-23 'Direct link to Features')

- Incremental training of models in a pipeline is now supported.
 
 If you have added new NLU training examples or new stories/rules for dialogue manager, you don't need to train the pipeline from scratch. Instead, you can initialize the pipeline with a previously trained model and continue finetuning the model on the complete dataset consisting of new training examples. To do so, use `rasa train --finetune`. For more detailed explanation of the command, check out the docs on [incremental training](https://legacy-docs-oss.rasa.com/docs/rasa/command-line-interface/#incremental-training).
 
 Added a configuration parameter `additional_vocabulary_size` to [`CountVectorsFeaturizer`](https://legacy-docs-oss.rasa.com/docs/rasa/components#countvectorsfeaturizer) and `number_additional_patterns` to [`RegexFeaturizer`](https://legacy-docs-oss.rasa.com/docs/rasa/components#regexfeaturizer). These parameters are useful to configure when using incremental training for your pipelines.
 
- Add the option to use cross-validation to the [`POST /model/test/intents`](https://rasa.com/docs/reference/api/pro/http-api/) endpoint. To use cross-validation specify the query parameter `cross_validation_folds` in addition to the training data in YAML format.
 
 Add option to run NLU evaluation ([`POST /model/test/intents`](https://rasa.com/docs/reference/api/pro/http-api/)) and model training ([`POST /model/train`](https://rasa.com/docs/reference/api/pro/http-api/)) asynchronously. To trigger asynchronous processing specify a callback URL in the query parameter `callback_url` which Rasa Open Source should send the results to. This URL will also be called in case of errors.
 
- Make [TED Policy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#ted-policy) an end-to-end policy. Namely, make it possible to train TED on stories that contain intent and entities or user text and bot actions or bot text. If you don't have text in your stories, TED will behave the same way as before. Add possibility to predict entities using TED.
 
 Here's an example of a dialogue in the Rasa story format:
 
 ```
    stories:- story: collect restaurant booking info  # name of the story - just for debugging  steps:  - intent: greet                          # user message with no entities  - action: utter_ask_howcanhelp           # action that the bot should execute  - intent: inform                         # user message with entities    entities:    - location: "rome"    - price: "cheap"  - bot: On it                             # actual text that bot can output  - action: utter_ask_cuisine  - user: I would like [spanish](cuisine). # actual text that user input  - action: utter_ask_num_people
    ```
 
 Some model options for `TEDPolicy` got renamed. Please update your configuration files using the following mapping:
 
 | Old model option | New model option |
 | --- | --- |
 | transformer\_size | dictionary “transformer\_size” with keys |
 | | “text”, “action\_text”, “label\_action\_text”, “dialogue” |
 | number\_of\_transformer\_layers | dictionary “number\_of\_transformer\_layers” with keys |
 | | “text”, “action\_text”, “label\_action\_text”, “dialogue” |
 | dense\_dimension | dictionary “dense\_dimension” with keys |
 | | “text”, “action\_text”, “label\_action\_text”, “intent”, |
 | | “action\_name”, “label\_action\_name”, “entities”, “slots”, |
 | | “active\_loop” |
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-97 'Direct link to Improvements')

- Added a message showing the location where the failed stories file was saved.
 
- Add support for the top-level response keys `quick_replies`, `attachment` and `elements` refered to in `rasa.core.channels.OutputChannel.send_reponse`, as well as `metadata`.
 
- Changed the format of the histogram of confidence values for both correct and incorrect predictions produced by running `rasa test`.
 
- Run [`bandit`](https://bandit.readthedocs.io/en/latest/) checks on pull requests. Introduce `make static-checks` command to run all static checks locally.
 
- Add `rasa train --dry-run` command that allows to check if training needs to be performed and what exactly needs to be retrained.
 
- [`POST /model/test/intents`](https://rasa.com/docs/reference/api/pro/http-api/) now returns the `report` field for `intent_evaluation`, `entity_evaluation` and `response_selection_evaluation` as machine-readable JSON payload instead of string.
 
- Make `rasa data validate stories` work for end-to-end.
 
 The `rasa data validate stories` function now considers the tokenized user text instead of the plain text that is part of a state. This is closer to what Rasa Core actually uses to distinguish states and thus captures more story structure problems.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-324 'Direct link to Bugfixes')

- Rename `language_list` to `supported_language_list` for `JiebaTokenizer`.
- A `float` slot returns unambiguous values - `[1.0, <value>]` if successfully converted, `[0.0, 0.0]` if not. This makes it possible to distinguish an empty float slot from a slot set to `0.0`.
 
 caution
 
 This change is model-breaking. Please retrain your models.
 
- Fix an erroneous attribute for Redis key prefix in `rasa.core.tracker_store.RedisTrackerStore`: 'RedisTrackerStore' object has no attribute 'prefix'.
- Remove token when its text (for example, whitespace) can't be tokenized by LM tokenizer (from `LanguageModelFeaturizer`).
- Temporary directories which were created during requests to the [HTTP API](https://rasa.com/docs/rasa-pro/production/http-api) are now cleaned up correctly once the request was processed.
- Add option `use_word_boundaries` for `RegexFeaturizer` and `RegexEntityExtractor`. To correctly process languages such as Chinese that don't use whitespace for word separation, the user needs to add the `use_word_boundaries: False` option to those two components.
- Correctly fingerprint the default domain slots. Previously this led to the issue that `rasa train core` would always retrain the model even if the training data hasn't changed.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-33 'Direct link to Improved Documentation')

- Return the "Migrate from" entry to the docs sidebar.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-93 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.1.3\] - 2020-12-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#213---2020-12-04 'Direct link to [2.1.3] - 2020-12-04')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-98 'Direct link to Improvements')

- Removed `multidict` from the project dependencies. `multidict` continues to be a second order dependency of Rasa Open Source but will be determined by the dependencies which use it instead of by Rasa Open Source directly.
 
 This resolves issues like the following:
 
 ```
    sanic 20.9.1 has requirement multidict==5.0.0, but you'll have multidict 4.6.0 which is incompatible.
    ```
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-325 'Direct link to Bugfixes')

- `SingleStateFeaturizer` checks whether it was trained with `RegexInterpreter` as nlu interpreter. If that is the case, `RegexInterpreter` is used during prediction.
 
- Make sure the `responses` are synced between NLU training data and the Domain even if there're no retrieval intents in the NLU training data.
 
- Categorical slots will have a default value set when just updating nlg data in the domain.
 
 Previously this resulted in `InvalidDomain` being thrown.
 
- - Preserve `domain` slot ordering while dumping it back to the file.
 - Preserve multiline `text` examples of `responses` defined in `domain` and `NLU` training data.

## \[2.1.2\] - 2020-11-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#212---2020-11-27 'Direct link to [2.1.2] - 2020-11-27')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-326 'Direct link to Bugfixes')

- Slots that use `initial_value` won't cause rule contradiction errors when `conversation_start: true` is used. Previously, two rules that differed only in their use of `conversation_start` would be flagged as contradicting when a slot used `initial_value`.
 
 In checking for incomplete rules, an action will be required to have set _only_ those slots that the same action has set in another rule. Previously, an action was expected to have set also slots which, despite being present after this action in another rule, were not actually set by this action.
 
- Fixed Rasa Open Source not being able to fetch models from certain URLs.
 

## \[2.1.1\] - 2020-11-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#211---2020-11-23 'Direct link to [2.1.1] - 2020-11-23')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-327 'Direct link to Bugfixes')

- Sender ID is correctly set when copying the tracker and sending it to the action server (instead of sending the `default` value). This fixes a problem where the action server would only retrieve trackers with a `sender_id` `default`.

## \[2.1.0\] - 2020-11-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#210---2020-11-17 'Direct link to [2.1.0] - 2020-11-17')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-21 'Direct link to Deprecations and Removals')

- The [`Policy`](https://legacy-docs-oss.rasa.com/docs/rasa/policies) interface was changed to return a `PolicyPrediction` object when `predict_action_probabilities` is called. Returning a list of probabilities directly is deprecated and support for this will be removed in Rasa Open Source 3.0.
 
 You can adapt your custom policy by wrapping your probabilities in a `PolicyPrediction` object:
 
 ```
    from rasa.core.policies.policy import Policy, PolicyPrediction# ... other importsdef predict_action_probabilities(        self,        tracker: DialogueStateTracker,        domain: Domain,        interpreter: NaturalLanguageInterpreter,        **kwargs: Any,    ) -> PolicyPrediction:    probabilities = ... # an action prediction of your policy    return PolicyPrediction(probabilities, "policy_name", policy_priority=self.priority)
    ```
 
 The same change was applied to the `PolicyEnsemble` interface. Instead of returning a tuple of action probabilities and policy name, it is now returning a `PolicyPrediction` object. Support for the old `PolicyEnsemble` interface will be removed in Rasa Open Source 3.0.
 
 caution
 
 This change is model-breaking. Please retrain your models.
 
- The [Pika Event Broker](https://rasa.com/docs/rasa-pro/production/event-brokers#pika-event-broker) no longer supports the environment variables `RABBITMQ_SSL_CA_FILE` and `RABBITMQ_SSL_KEY_PASSWORD`. You can alternatively specify `RABBITMQ_SSL_CA_FILE` in the RabbitMQ connection URL as described in the [RabbitMQ documentation](https://www.rabbitmq.com/uri-query-parameters.html).
 
 ```
    event_broker: type: pika url: "amqps://user:password@host?cacertfile=path_to_ca_cert&password=private_key_password" queues: - my_queue
    ```
 
 Support for `RABBITMQ_SSL_KEY_PASSWORD` was removed entirely.
 
 The method [`Event Broker.close`](https://rasa.com/docs/rasa-pro/production/event-brokers) was changed to be asynchronous. Support for synchronous implementations will be removed in Rasa Open Source 3.0.0. To adapt your implementation add the `async` keyword:
 
 ```
    from rasa.core.brokers.broker import EventBrokerclass MyEventBroker(EventBroker):    async def close(self) -> None:        # clean up event broker resources
    ```
 

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-24 'Direct link to Features')

- [Policies](https://legacy-docs-oss.rasa.com/docs/rasa/policies) can now return obligatory and optional events as part of their prediction. Obligatory events are always applied to the current conversation tracker. Optional events are only applied to the conversation tracker in case the policy wins.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-99 'Direct link to Improvements')

- Changed `Agent.load` method to support `pathlib` paths.
 
- If you are using the feature [Entity Roles and Groups](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-training-data#entities-roles-and-groups), you should now also list the roles and groups in your domain file if you want roles and groups to influence your conversations. For example:
 
 ```
    entities:  - city:      roles:        - from        - to  - name  - topping:      groups:        - 1        - 2  - size:      groups:        - 1        - 2
    ```
 
 Entity roles and groups can now influence dialogue predictions. For more information see the section [Entity Roles and Groups influencing dialogue predictions](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-training-data#entity-roles-and-groups-influencing-dialogue-predictions).
 
- Predictions of the [`FallbackClassifier`](https://legacy-docs-oss.rasa.com/docs/rasa/components#fallbackclassifier) are ignored when [evaluating the NLU model](https://rasa.com/docs/rasa-pro/nlu-based-assistants/testing-your-assistant#evaluating-an-nlu-model) Note that the `FallbackClassifier` predictions still apply to [test stories](https://rasa.com/docs/rasa-pro/nlu-based-assistants/testing-your-assistant#writing-test-stories).
 
- Adapt the training data reader and emulator for wit.ai to their latest format. Update the instructions in the migrate from wit.ai documentation to run Rasa Open Source in wit.ai emulation mode.
 
- Adding configurable prefixes to Redis [Tracker](https://rasa.com/docs/rasa-pro/production/tracker-stores) and [Lock Stores](https://rasa.com/docs/rasa-pro/production/lock-stores) so that a single Redis instance (and logical DB) can support multiple conversation trackers and locks. By default, conversations will be prefixed with `tracker:...` and all locks prefixed with `lock:...`. Additionally, you can add an alphanumeric-only `prefix: value` in `endpoints.yml` such that keys in redis will take the form `value:tracker:...` and `value:lock:...` respectively.
 
- Log the model's relative path when using CLI commands.
 
- Adds the option to configure whether extracted entities should be split by comma (`","`) or not. The default behaviour is `True` - i.e. split any list of extracted entities by comma. This makes sense for a list of ingredients in a recipie, for example `"avocado, tofu, cauliflower"`, however doesn't make sense for an address such as `"Schönhauser Allee 175, 10119 Berlin, Germany"`.
 
 In the latter case, add a new option to your config, e.g. if you are using the `DIETClassifier` this becomes:
 
 ```
    ...- name: DIETClassifier  split_entities_by_comma: False...
    ```
 
 in which case, none of the extracted entities will be split by comma. To switch it on/off for specific entity types you can use:
 
 ```
    ...- name: DIETClassifier  split_entities_by_comma:    address: True    ingredient: False...
    ```
 
 where both `address` and `ingredient` are two entity types.
 
 This feature is also available for `CRFEntityExtractor`.
 
- Fetching test stories from the HTTP API endpoint `GET /conversations/<conversation_id>/story` no longer triggers an update of the [conversation session](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#session-configuration).
 
 Added a new boolean query parameter `all_sessions` (default: `false`) to the [HTTP API](https://rasa.com/docs/rasa-pro/production/http-api) endpoint for fetching test stories (`GET /conversations/<conversation_id>/story`).
 
 When setting `?all_sessions=true`, the endpoint returns test stories for all conversation sessions for `conversation_id`. When setting `?all_sessions=all_sessions`, or when omitting the `all_sessions` parameter, a single test story is returned for `conversation_id`. In cases where multiple conversation sessions exist, only the last story is returned.
 
 Specifying the `retrieve_events_from_previous_conversation_sessions` kwarg for the [Tracker Store](https://rasa.com/docs/rasa-pro/production/tracker-stores) class is deprecated and will be removed in Rasa Open Source 3.0. Please use the `retrieve_full_tracker()` method instead.
 
- Improve the `rasa data convert nlg` command and introduce the `rasa data convert responses` command to simplify the migration from pre-2.0 response selector format to the new format.
 
- Added warning for when an option is provided for a [component](https://legacy-docs-oss.rasa.com/docs/rasa/components) that is not listed as a key in the defaults for that component.
 
- [Forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) no longer reject their execution before a potential custom action for validating / extracting slots was executed. Forms continue to reject in two cases automatically:
 
 - A slot was requested to be filled, but no slot mapping applied to the latest user message and there was no custom action for potentially extracting other slots.
 - A slot was requested to be filled, but the custom action for validating / extracting slots didn't return any slot event.
 
 Additionally you can also reject the form execution manually by returning a `ActionExecutionRejected` event within your custom action for validating / extracting slots.
 
- Remove dependency between `ConveRTTokenizer` and `ConveRTFeaturizer`. The `ConveRTTokenizer` is now deprecated, and the `ConveRTFeaturizer` can be used with any other `Tokenizer`.
 
 Remove dependency between `HFTransformersNLP`, `LanguageModelTokenizer`, and `LanguageModelFeaturizer`. Both `HFTransformersNLP` and `LanguageModelTokenizer` are now deprecated. `LanguageModelFeaturizer` implements the behavior of the stack and can be used with any other `Tokenizer`.
 
- Gray out "Download" button in Rasa Playground when the project is not yet ready to be downloaded.
 
- Slot mappings for [Forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) in the domain are now optional. If you do not provide any slot mappings as part of the domain, you need to provide [custom slot mappings](https://legacy-docs-oss.rasa.com/docs/rasa/forms#custom-slot-mappings) through a custom action. A form without slot mappings is specified as follows:
 
 ```
    forms:  my_form:    # no mappings
    ```
 
 The action for [forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) can now be overridden by defining a custom action with the same name as the form. This can be used to keep using the deprecated Rasa Open Source `FormAction` which is implemented within the Rasa SDK. Note that it is **not** recommended to override the form action for anything else than using the deprecated Rasa SDK `FormAction`.
 
- Changed the default model weights loaded for `HFTransformersNLP` component.
 
 Use a [language agnostic sentence embedding model](https://tfhub.dev/google/LaBSE/1) as the default model. These model weights should help improve performance on intent classification and response selection.
 
- Add validations for [slot mappings](https://legacy-docs-oss.rasa.com/docs/rasa/forms#slot-mappings). If a slot mapping is not valid, an `InvalidDomain` error is raised.
 
- Adapt the training data reader and emulator for LUIS to their v3 format and add support for roles. Update the instructions in the "Migrate from LUIS" documentation page to reflect the recent changes made to the UI of LUIS.
 
- Adapt the training data reader and emulator for DialogFlow to [their latest format](https://cloud.google.com/dialogflow/es/docs/reference/rest/v2/DetectIntentResponse) and add support for regex entities.
 
- The [Pika Event Broker](https://rasa.com/docs/rasa-pro/production/event-brokers#pika-event-broker) was reimplemented with the `[aio-pika` library\[([https://docs.aio-pika.com/](https://docs.aio-pika.com/)). Messages will now be published to RabbitMQ asynchronously which improves the prediction performance.
 
- The confidence of the [`FallbackClassifier`](https://legacy-docs-oss.rasa.com/docs/rasa/components#fallbackclassifier) predictions is set to `1 - top intent confidence`.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-328 'Direct link to Bugfixes')

- `ActionRestart` will now trigger `ActionSessionStart` as a followup action.
 
- Fixed a bug with `rasa data split nlu` which caused the resulting train / test ratio to sometimes differ from the ratio specified by the user or by default.
 
 The splitting algorithm ensures that every intent and response class appears in both the training and the test set. This means that each split must contain at least as many examples as there are classes, which for small datasets can contradict the requested training fraction. When this happens, the command issues a warning to the user that the requested training fraction can't be satisfied.
 
- Fixed bug where slots with `influence_conversation=false` affected the action prediction if they were set manually using the `POST /conversations/<conversation_id/tracker/events` endpoint in the [HTTP API](https://rasa.com/docs/rasa-pro/production/http-api).
 
- Update Pika event broker to be a separate process and make it use a `multiprocessing.Queue` to send and process messages. This change should help avoid situations when events stop being sent after a while.
 
- Ignore rules when validating stories
 
- - Updated Slack Connector for new Slack Events API
- Update Rasa Playground "Download" button to work correctly depending on the current chat state.
 
- Test stories can now contain both: normal intents and retrieval intents. The `failed_test_stories.yml`, generated by `rasa test`, also specifies the full retrieval intent now. Previously `rasa test` would fail on test stories that specified retrieval intents.
 
- The converter tool is now able to convert test stories that contain a number as entity type.
 
- The converter tool now converts test stories and stories that contain full retrieval intents correctly. Previously the response keys were deleted during conversion to YAML.
 
- The slack connector requires a configuration for `slack_signing_secret` to make the connector more secure. The configuration value needs to be added to your `credentials.yml` if you are using the slack connector.
 
- Fixed model fingerprinting - it should avoid some more unecessary retrainings now.
 
- Fixed a problem when slots of type `text` or `list` were referenced by name only in the training data and this was treated as an empty value. This means that the two following stories are equivalent in case the slot type is `text`:
 
 ```
    stories:- story: Story referencing slot by name  steps:  - intent: greet  - slot_was_set:    - name- story: Story referencing slot with name and value  steps:  - intent: greet  - slot_was_set:    - name: "some name"
    ```
 
 Note that you still need to specify values for all other slot types as only `text` and `list` slots are featurized in a binary fashion.
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-34 'Direct link to Improved Documentation')

- Correct data validation docs

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-94 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.0.8\] - 2020-11-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#208---2020-11-26 'Direct link to [2.0.8] - 2020-11-26')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-329 'Direct link to Bugfixes')

- Slots that use `initial_value` won't cause rule contradiction errors when `conversation_start: true` is used. Previously, two rules that differed only in their use of `conversation_start` would be flagged as contradicting when a slot used `initial_value`.
 
 In checking for incomplete rules, an action will be required to have set _only_ those slots that the same action has set in another rule. Previously, an action was expected to have set also slots which, despite being present after this action in another rule, were not actually set by this action.
 

## \[2.0.7\] - 2020-11-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#207---2020-11-24 'Direct link to [2.0.7] - 2020-11-24')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-330 'Direct link to Bugfixes')

- `ActionRestart` will now trigger `ActionSessionStart` as a followup action.
 
- Fixed Rasa Open Source not being able to fetch models from certain URLs.
 
 This addresses an issue introduced in 2.0.3 where `rasa-production` could not use the models from `rasa-x` in Rasa X server mode.
 
- `SingleStateFeaturizer` checks whether it was trained with `RegexInterpreter` as NLU interpreter. If that is the case, `RegexInterpreter` is used during prediction.
 

## \[2.0.6\] - 2020-11-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#206---2020-11-10 'Direct link to [2.0.6] - 2020-11-10')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-331 'Direct link to Bugfixes')

- Fixed a bug that occurred when setting multiple Sanic workers in combination with a custom [Lock Store](https://rasa.com/docs/rasa-pro/production/lock-stores). Previously, if the number was set higher than 1 and you were using a custom lock store, it would reject because of a strict check to use a [Redis Lock Store](https://rasa.com/docs/rasa-pro/production/lock-stores#redislockstore).
- Fixed a bug in the [`TwoStageFallback`](https://legacy-docs-oss.rasa.com/docs/rasa/fallback-handoff/#two-stage-fallback) action which reverted too many events after the user successfully rephrased.

## \[2.0.5\] - 2020-11-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#205---2020-11-10 'Direct link to [2.0.5] - 2020-11-10')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-332 'Direct link to Bugfixes')

- Fix a bug because of which only one retrieval intent was present in `all_retrieval_intent` key of the output of `ResponseSelector` even if there were multiple retrieval intents present in the training data.

## \[2.0.4\] - 2020-11-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#204---2020-11-08 'Direct link to [2.0.4] - 2020-11-08')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-333 'Direct link to Bugfixes')

- Fixed error when starting Rasa X locally without a proper git setup.
- Properly validate incoming webhook requests for the Slack connector to be authentic.

## \[2.0.3\] - 2020-10-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#203---2020-10-29 'Direct link to [2.0.3] - 2020-10-29')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-334 'Direct link to Bugfixes')

- Fix [ConveRTTokenizer](https://rasa.com/docs/rasa/2.x/components#converttokenizer) failing because of wrong model URL by making the `model_url` parameter of `ConveRTTokenizer` mandatory.
 
 Since the ConveRT model was taken [offline](https://github.com/RasaHQ/rasa/issues/6806), we can no longer use the earlier public URL of the model. Additionally, since the licence for the model is unknown, we cannot host it ourselves. Users can still use the component by setting `model_url` to a community/self-hosted model URL or path to a local directory containing model files. For example:
 
 ```
    pipeline:    - name: ConveRTTokenizer      model_url: <remote/local path to model>
    ```
 
- Update example formbot to use `FormValidationAction` for slot validation
 

## \[2.0.2\] - 2020-10-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#202---2020-10-22 'Direct link to [2.0.2] - 2020-10-22')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-335 'Direct link to Bugfixes')

- Fix description of previous event in output of `rasa data validate stories`
- Fixed command line coloring for windows command lines running an encoding other than `utf-8`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-95 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[2.0.1\] - 2020-10-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#201---2020-10-20 'Direct link to [2.0.1] - 2020-10-20')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-336 'Direct link to Bugfixes')

- Create correct `KafkaProducer` for `PLAINTEXT` and `SASL_SSL` security protocols.
- - Fix `YAMLStoryReader` not being able to represent [`OR` statements](https://legacy-docs-oss.rasa.com/docs/rasa/stories#or-statements) in conversion mode.
 - Fix `MarkdownStoryWriter` not being able to write stories with `OR` statements (when loaded in conversion mode).

## \[2.0.0\] - 2020-10-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#200---2020-10-07 'Direct link to [2.0.0] - 2020-10-07')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-22 'Direct link to Deprecations and Removals')

- Removed previously deprecated packages `rasa_nlu` and `rasa_core`.
 
 Use imports from `rasa.core` and `rasa.nlu` instead.
 
- Removed previously deprecated classes:
 
 - event brokers (`EventChannel` and `FileProducer`, `KafkaProducer`, `PikaProducer`, `SQLProducer`)
 - intent classifier `EmbeddingIntentClassifier`
 - policy `KerasPolicy`
 
 Removed previously deprecated methods:
 
 - `Agent.handle_channels`
 - `TrackerStore.create_tracker_store`
 
 Removed support for pipeline templates in `config.yml`
 
 Removed deprecated training data keys `entity_examples` and `intent_examples` from json training data format.
 
- Removed `restaurantbot` example as it was confusing and not a great way to build a bot.
 
- `LabelTokenizerSingleStateFeaturizer` is deprecated. To replicate `LabelTokenizerSingleStateFeaturizer` functionality, add a `Tokenizer` with `intent_tokenization_flag: True` and `CountVectorsFeaturizer` to the NLU pipeline. An example of elements to be added to the pipeline is shown in the improvement changelog 6296\`.
    
    `BinarySingleStateFeaturizer` is deprecated and will be removed in the future. We recommend to switch to `SingleStateFeaturizer`.
    
-   Specifying the parameters `force` and `save_to_default_model_directory` as part of the JSON payload when training a model using `POST /model/train` is now deprecated. Please use the query parameters `force_training` and `save_to_default_model_directory` instead. See the [API documentation](https://rasa.com/docs/reference/api/pro/http-api/) for more information.
    
-   The conversation event `form` was renamed to `active_loop`. Rasa Open Source will continue to be able to read and process old `form` events. Note that serialized trackers will no longer have the `active_form` field. Instead the `active_loop` field will contain the same information. Story representations in Markdown and YAML will use `active_loop` instead of `form` to represent the event.
    
-   Removed support for `queue` argument in `PikaEventBroker` (use `queues` instead).
    
    Domain file:
    
    -   Removed support for `templates` key (use `responses` instead).
    -   Removed support for string `responses` (use dictionaries instead).
    
    NLU `Component`:
    
    -   Removed support for `provides` attribute, it's not needed anymore.
    -   Removed support for `requires` attribute (use `required_components()` instead).
    
    Removed `_guess_format()` utils method from `rasa.nlu.training_data.loading` (use `guess_format` instead).
    
    Removed several config options for [TED Policy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#ted-policy), [DIETClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier) and [ResponseSelector](https://legacy-docs-oss.rasa.com/docs/rasa/components#responseselector):
    
    -   `hidden_layers_sizes_pre_dial`
    -   `hidden_layers_sizes_bot`
    -   `droprate`
    -   `droprate_a`
    -   `droprate_b`
    -   `hidden_layers_sizes_a`
    -   `hidden_layers_sizes_b`
    -   `num_transformer_layers`
    -   `num_heads`
    -   `dense_dim`
    -   `embed_dim`
    -   `num_neg`
    -   `mu_pos`
    -   `mu_neg`
    -   `use_max_sim_neg`
    -   `C2`
    -   `C_emb`
    -   `evaluate_every_num_epochs`
    -   `evaluate_on_num_examples`
    
    Please check the documentation for more information.
    
-   The conversation event `form_validation` was renamed to `loop_interrupted`. Rasa Open Source will continue to be able to read and process old `form_validation` events.
    
-   `SklearnPolicy` was deprecated. `TEDPolicy` is the preferred machine-learning policy for dialogue models.
    
-   [Slots](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#slots) of type `unfeaturized` are now deprecated and will be removed in Rasa Open Source 3.0. Instead you should use the property `influence_conversation: false` for every slot type as described in the [migration guide](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide/#unfeaturized-slots).
    
-   [Conversation sessions](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#session-configuration) are now enabled by default if your [Domain](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain) does not contain a session configuration. Previously a missing session configuration was treated as if conversation sessions were disabled. You can explicitly disable conversation sessions using the following snippet:
    
    domain.yml
    
    ```
    session_config:  # A session expiration time of `0`  # disables conversation sessions  session_expiration_time: 0
    ```
    
-   Using the [default action](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions) `action_deactivate_form` to deactivate the currently active loop / [Form](https://legacy-docs-oss.rasa.com/docs/rasa/forms) is deprecated. Please use `action_deactivate_loop` instead.
    

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-25 'Direct link to Features')

-   Added template name to the metadata of bot utterance events.
    
    `BotUttered` event contains a `template_name` property in its metadata for any new bot message.
    
-   Added a `--num-threads` CLI argument that can be passed to `rasa train` and will be used to train NLU components.
    
-   You can now define what kind of features should be used by what component (see [Choosing a Pipeline](https://legacy-docs-oss.rasa.com/docs/rasa/tuning-your-model/)).
    
    You can set an alias via the option `alias` for every featurizer in your pipeline. The `alias` can be anything, by default it is set to the full featurizer class name. You can then specify, for example, on the [DIETClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier) what features from which featurizers should be used. If you don't set the option `featurizers` all available features will be used. This is also the default behavior. Check components to see what components have the option `featurizers` available.
    
    Here is an example pipeline that shows the new option. We define an alias for all featurizers in the pipeline. All features will be used in the `DIETClassifier`. However, the `ResponseSelector` only takes the features from the `ConveRTFeaturizer` and the `CountVectorsFeaturizer` (word level).
    
    ```
    pipeline:- name: ConveRTTokenizer- name: ConveRTFeaturizer  alias: "convert"- name: CountVectorsFeaturizer  alias: "cvf_word"- name: CountVectorsFeaturizer  alias: "cvf_char"  analyzer: char_wb  min_ngram: 1  max_ngram: 4- name: RegexFeaturizer  alias: "regex"- name: LexicalSyntacticFeaturizer  alias: "lsf"- name: DIETClassifier:- name: ResponseSelector  epochs: 50  featurizers: ["convert", "cvf_word"]- name: EntitySynonymMapper
    ```
    
    caution
    
    This change is model-breaking. Please retrain your models.
    
-   Added `--port` commandline argument to the interactive learning mode to allow changing the port for the Rasa server running in the background.
    
-   Add new entity extractor `RegexEntityExtractor`. The entity extractor extracts entities using the lookup tables and regexes defined in the training data. For more information see [RegexEntityExtractor](https://legacy-docs-oss.rasa.com/docs/rasa/components#regexentityextractor).
    
-   Introduced a new `YAML` format for Core training data and implemented a parser for it. Rasa Open Source can now read stories in both `Markdown` and `YAML` format.
    
-   You can now enable threaded message responses from Rasa through the Slack connector. This option is enabled using an optional configuration in the credentials.yml file
    
    ```
        slack:      slack_token:      slack_channel:      use_threads: True
    ```
    
    Button support has also been added in the Slack connector.
    
-   Add support for [rules](https://legacy-docs-oss.rasa.com/docs/rasa/rules) data and [forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) in YAML format.
    
-   The NLU `interpreter` is now passed to the [Policies](https://legacy-docs-oss.rasa.com/docs/rasa/policies) during training and inference time. Note that this requires an additional parameter `interpreter` in the method `predict_action_probabilities` of the `Policy` interface. In case a custom `Policy` implementation doesn't provide this parameter Rasa Open Source will print a warning and omit passing the `interpreter`.
    
-   Added the new dialogue policy RulePolicy which will replace the old “rule-like” policies [Mapping Policy](https://rasa.com/docs/rasa/2.x/policies#mapping-policy), [Fallback Policy](https://rasa.com/docs/rasa/2.x/policies#fallback-policy), [Two-Stage Fallback Policy](https://rasa.com/docs/rasa/2.x/policies#two-stage-fallback-policy), and [Form Policy](https://rasa.com/docs/rasa/2.x/policies#form-policy). These policies are now deprecated and will be removed in the future. Please see the [rules documentation](https://legacy-docs-oss.rasa.com/docs/rasa/rules) for more information.
    
    Added new NLU component [FallbackClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#fallbackclassifier) which predicts an intent `nlu_fallback` in case the confidence was below a given threshold. The intent `nlu_fallback` may then be used to write stories / rules to handle the fallback in case of low NLU confidence.
    
    ```
    pipeline:- # Other NLU components ...- name: FallbackClassifier  # If the highest ranked intent has a confidence lower than the threshold then  # the NLU pipeline predicts an intent `nlu_fallback` which you can then be used in  # stories / rules to implement an appropriate fallback.  threshold: 0.5
    ```
    
-   Added possibility to split the domain into separate files. All YAML files under the path specified with `--domain` will be scanned for domain information (e.g. intents, actions, etc) and then combined into a single domain.
    
    The default value for `--domain` is still `domain.yml`.
    
-   Add optional metadata argument to `NaturalLanguageInterpreter`'s parse method.
    
-   The Rasa Open Source API endpoint `POST /model/train` now supports training data in YAML format. Please specify the header `Content-Type: application/yaml` when training a model using YAML training data. See the [API documentation](https://rasa.com/docs/reference/api/pro/http-api/) for more information.
    
-   Added a YAML schema and a writer for 2.0 Training Core data.
    
-   Users can now use the `rasa data convert \{nlu|core\} -f yaml` command to convert training data from Markdown format to YAML format.
    
-   Add option `use_lemma` to `CountVectorsFeaturizer`. By default it is set to `True`.
    
    `use_lemma` indicates whether the featurizer should use the lemma of a word for counting (if available) or not. If this option is set to `False` it will use the word as it is.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-100 'Direct link to Improvements')

-   Add support for Python 3.8.
    
-   Changed the project structure for Rasa projects initialized with the [CLI](https://rasa.com/docs/rasa-pro/command-line-interface) (using the `rasa init` command): `actions.py` -> `actions/actions.py`. `actions` is now a Python package (it contains a file `actions/__init__.py`). In addition, the `__init__.py` at the root of the project has been removed.
    
-   `DIETClassifier` now also assigns a confidence value to entity predictions.
    
-   Added behavior to the `rasa --version` command. It will now also list information about the operating system, python version and `rasa-sdk`. This will make it easier for users to file bug reports.
    
-   Support for additional training metadata.
    
    Training data messages now to support kwargs and the Rasa JSON data reader includes all fields when instantiating a training data instance.
    
-   Standardize testing output. The following test output can be produced for intents, responses, entities and stories:
    
    -   report: a detailed report with testing metrics per label (e.g. precision, recall, accuracy, etc.)
    -   errors: a file that contains incorrect predictions
    -   successes: a file that contains correct predictions
    -   confusion matrix: plot of confusion matrix
    -   histogram: plot of confidence distribution (not available for stories)
-   To avoid the problem of our entity extractors predicting entity labels for just a part of the words, we introduced a cleaning method after the prediction was done. We should avoid the incorrect prediction in the first place. To achieve this we will not tokenize words into sub-words anymore. We take the mean feature vectors of the sub-words as the feature vector of the word.
    
    caution
    
    This change is model breaking. Please, retrain your models.
    
-   Move option `case_sensitive` from the tokenizers to the featurizers.
    
    -   Remove the option from the `WhitespaceTokenizer` and `ConveRTTokenizer`.
    -   Add option `case_sensitive` to the `RegexFeaturizer`.
-   If a user sends a voice message to the bot using Facebook, users messages was set to the attachments URL. The same is now also done for the rest of attachment types (image, video, and file).
    
-   Creating a `Domain` using `Domain.fromDict` can no longer alter the input dictionary. Previously, there could be problems when the input dictionary was re-used for other things after creating the `Domain` from it.
    
-   The debug-level logs when instantiating an [SQLTrackerStore](https://rasa.com/docs/rasa-pro/production/tracker-stores#sqltrackerstore) no longer show the password in plain text. Now, the URL is displayed with the password hidden, e.g. `postgresql://username:***@localhost:5432`.
    
-   Shorten the information in tqdm during training ML algorithms based on the log level. If you train your model in debug mode, all available metrics will be shown during training, otherwise, the information is shorten.
    
-   Ignore conversation test directory `tests/` when importing a project using `MultiProjectImporter` and `use_e2e` is `False`. Previously, any story data found in a project subdirectory would be imported as training data.
    
-   Implemented model checkpointing for DIET (including the response selector) and TED. The best model during training will be stored instead of just the last model. The model is evaluated on the basis of `evaluate_every_number_of_epochs` and `evaluate_on_number_of_examples`.
    
    Checkpointing is enabled iff the following is set for the models in the `config.yml` file:
    
    -   `checkpoint_model: True`
    -   `evaluate_on_number_of_examples > 0`
    
    The model is stored to whatever location has been specified with the `--out` parameter when calling `rasa train nlu/core ...`.
    
-   `rasa data split nlu` now makes sure that there is at least one example per intent and response in the test data.
    
-   The method `ensure_consistent_bilou_tagging` now also considers the confidence values of the predicted tags when updating the BILOU tags.
    
-   We updated the way how we save and use features in our NLU pipeline.
    
    The message object now has a dedicated field, called `features`, to store the features that are generated in the NLU pipeline. We adapted all our featurizers in a way that sequence and sentence features are stored independently. This allows us to keep different kind of features for the sequence and the sentence. For example, the `LexicalSyntacticFeaturizer` does not produce any sentence features anymore as our experiments showed that those did not bring any performance gain just quite a lot of additional values to store.
    
    We also modified the DIET architecture to process the sequence and sentence features independently at first. The features are concatenated just before the transformer.
    
    We also removed the `__CLS__` token again. Our Tokenizers will not add this token anymore.
    
    caution
    
    This change is model-breaking. Please retrain your models.
    
-   Add endpoint kwarg to `rasa.jupyter.chat` to enable using a custom action server while chatting with a model in a jupyter notebook.
    
-   Support for rasa conversation id with special characters on the server side - necessary for some channels (e.g. Viber)
    
-   Add support for proxy use in [slack](https://rasa.com/docs/rasa-pro/connectors/slack) input channel.
    
-   Log the number of examples per intent during training. Logging can be enabled using `rasa train --debug`.
    
-   Support for other remote storages can be achieved by using an external library.
    
-   Add `output_channel` query param to `/conversations/<conversation_id>/tracker/events` route, along with boolean `execute_side_effects` to optionally schedule/cancel reminders, and forward bot messages to output channel.
    
-   Allow Rasa to boot when model loading exception occurs. Forward HTTP Error responses to standard log output.
    
-   Rename `DucklingHTTPExtractor` to `DucklingEntityExtractor`.
    
-   -   Modified functionality of `SingleStateFeaturizer`.
        
        `SingleStateFeaturizer` uses trained NLU `Interpreter` to featurize intents and action names. This modified `SingleStateFeaturizer` can replicate `LabelTokenizerSingleStateFeaturizer` functionality. This component is deprecated from now on. To replicate `LabelTokenizerSingleStateFeaturizer` functionality, add a `Tokenizer` with `intent_tokenization_flag: True` and `CountVectorsFeaturizer` to the NLU pipeline. Please update your configuration file.
        
        For example:
        
        ```
        language: enpipeline:  - name: WhitespaceTokenizer    intent_tokenization_flag: True  - name: CountVectorsFeaturizer
        ```
        
        Please train both NLU and Core (using `rasa train`) to use a trained tokenizer and featurizer for core featurization.
        
        The new `SingleStateFeaturizer` stores slots, entities and forms in sparse features for more lightweight storage.
        
        `BinarySingleStateFeaturizer` is deprecated and will be removed in the future. We recommend to switch to `SingleStateFeaturizer`.
        
    -   Modified `TEDPolicy` to handle sparse features. As a result, `TEDPolicy` may require more epochs than before to converge.
        
    -   Default TEDPolicy featurizer changed to `MaxHistoryTrackerFeaturizer` with infinite max history (takes all dialogue turns into account).
        
    -   Default batch size for TED increased from \[8,32\] to \[64, 256\]
        
-   [Response selector templates](https://legacy-docs-oss.rasa.com/docs/rasa/components#responseselector) now support all features that domain utterances do. They use the yaml format instead of markdown now. This means you can now use buttons, images, ... in your FAQ or chitchat responses (assuming they are using the response selector).
    
    As a consequence, training data form in markdown has to have the file suffix `.md` from now on to allow proper file type detection-
    
-   Support for test stories written in yaml format.
    
-   [Response Selectors](https://legacy-docs-oss.rasa.com/docs/rasa/components#responseselector) are now trained on retrieval intent labels by default instead of the actual response text. For most models, this should improve training time and accuracy of the `ResponseSelector`.
    
    If you want to revert to the pre-2.0 default behavior, add the `use_text_as_label=true` parameter to your `ResponseSelector` component.
    
    You can now also have multiple response templates for a single sub-intent of a retrieval intent. The first response template containing the text attribute is picked for training(if `use_text_as_label=True`) and a random template is picked for bot's utterance just as how other `utter_` templates are picked.
    
    All response selector related evaluation artifacts - `report.json, successes.json, errors.json, confusion_matrix.png` now use the sub-intent of the retrieval intent as the target and predicted labels instead of the actual response text.
    
    The output schema of `ResponseSelector` has changed - `full_retrieval_intent` and `name` have been deprecated in favour of `intent_response_key` and `response_templates` respectively. Additionally a key `all_retrieval_intents` is added to the response selector output which will hold a list of all retrieval intents(faq,chitchat, etc.) that are present in the training data.An example output looks like this -
    
    ```
    "response_selector": {    "all_retrieval_intents": ["faq"],    "default": {      "response": {        "id": 1388783286124361986, "confidence": 1.0, "intent_response_key": "faq/is_legit",        "response_templates": [          {            "text": "absolutely",            "image": "https://i.imgur.com/nGF1K8f.jpg"          },          {            "text": "I think so."          }        ],      },      "ranking": [        {          "id": 1388783286124361986,          "confidence": 1.0,          "intent_response_key": "faq/is_legit"        },      ]
    ```
    
    An example bot demonstrating how to use the `ResponseSelector` is added to the `examples` folder.
    
-   Do not modify conversation tracker's `latest_input_channel` property when using `POST /trigger_intent` or `ReminderScheduled`.
    
-   Do not set the output dimension of the `sparse-to-dense` layers to the same dimension as the dense features.
    
    Update default value of `dense_dimension` and `concat_dimension` for `text` in `DIETClassifier` to 128.
    
-   Retrieval actions with `respond_` prefix are now replaced with usual utterance actions with `utter_` prefix.
    
    If you were using retrieval actions before, rename all of them to start with `utter_` prefix. For example, `respond_chitchat` becomes `utter_chitchat`. Also, in order to keep the response templates more consistent, you should now add the `utter_` prefix to all response templates defined for retrieval intents. For example, a response template `chitchat/ask_name` becomes `utter_chitchat/ask_name`. Note that the NLU examples for this will still be under `chitchat/ask_name` intent. The example `responseselectorbot` should help clarify these changes further.
    
-   Added telemetry reporting. Rasa uses telemetry to report anonymous usage information. This information is essential to help improve Rasa Open Source for all users. Reporting will be opt-out. More information can be found in our [telemetry documentation](https://rasa.com/docs/rasa-pro/telemetry/telemetry).
    
-   Update `extract_other_slots` method inside `FormAction` to fill a slot from an entity with a different name if corresponding slot mapping of `from_entity` type is unique.
    
-   [Slots](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#slots) of any type can now be ignored during a conversation. To do so, specify the property `influence_conversation: false` for the slot.
    
    ```
    slot:  a_slot:    type: text    influence_conversation: false
    ```
    
    The property `influence_conversation` is set to `true` by default. See the [documentation for slots](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#slots) for more information.
    
    A new slot type [`any`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#any-slot) was added. Slots of this type can store any value. Slots of type `any` are always ignored during conversations.
    
-   Improved exception handling within Rasa Open Source.
    
    All exceptions that are somewhat expected (e.g. errors in file formats like configurations or training data) will share a common base class `RasaException`.
    
    ::warning Backwards Incompatibility Base class for the exception raised when an action can not be found has been changed from a `NameError` to a `ValueError`. ::
    
    Some other exceptions have also slightly changed:
    
    -   raise `YamlSyntaxException` instead of YAMLError (from ruamel) when failing to load a yaml file with information about the line where loading failed
    -   introduced `MissingDependencyException` as an exception raised if packages need to be installed
-   Debug logs from `matplotlib` libraries are now hidden by default and are configurable with the `LOG_LEVEL_LIBRARIES` environment variable.
    
-   Update `KafkaEventBroker` to support `SASL_SSL` and `PLAINTEXT` protocols.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-337 'Direct link to Bugfixes')

-   Fixed issue where temporary model directories were not removed after pulling from a model server.
    
    If the model pulled from the server was invalid, this could lead to large amounts of local storage usage.
    
-   Fixed a bug in the `CountVectorsFeaturizer` which resulted in the very first message after loading a model to be processed incorrectly due to the vocabulary not being loaded yet.
    
-   Fixed Rasa shell skipping button messages if buttons are attached to a message previous to the latest.
    
-   Stack level for `FutureWarning` updated to level 2.
    
-   If custom utter message contains no value or integer value, then it fails returning custom utter message. Fixed by converting the template to type string.
    
-   Don't create TensorBoard log files during prediction.
    
-   Fixed DIET breaking with empty spaCy model.
    
-   Pinned the library version for the Azure [Cloud Storage](https://rasa.com/docs/rasa-pro/production/model-storage#load-model-from-cloud) to 2.1.0 since the persistor is currently not compatible with later versions of the azure-storage-blob library.
    
-   Remove `clean_up_entities` from extractors that extract pre-defined entities. Just keep the clean up method for entity extractors that extract custom entities.
    
-   Fixed issue where the `DucklingHTTPExtractor` component would not work if its `url` contained a trailing slash.
    
-   Changed to variable `CERT_URI` in `hangouts.py` to a string type
    
-   Slots will be correctly interpolated for `button` responses.
    
    Previously this resulted in no interpolation due to a bug.
    
-   Remove option `token_pattern` from `CountVectorsFeaturizer`. Instead all tokenizers now have the option `token_pattern`. If a regular expression is set, the tokenizer will apply the token pattern.
    
-   Allow user to retry failed file exports in interactive training.
    
-   Fixed a bug when custom metadata passed with the utterance always restarted the session.
    
-   `WhitespaceTokenizer` does not remove vowel signs in Hindi anymore.
    
-   Convert entity values coming from `DucklingHTTPExtractor` to string during evaluation to avoid mismatches due to different types.
    
-   Update `FeatureSignature` to store just the feature dimension instead of the complete shape. This change fixes the usage of the option `share_hidden_layers` in the `DIETClassifier`.
    
-   Unescape the `\n, \t, \r, \f, \b` tokens on reading nlu data from markdown files.
    
    On converting json files into markdown, the tokens mentioned above are espaced. These tokens need to be unescaped on loading the data from markdown to ensure that the data is treated in the same way.
    
-   Fix the way training data is generated in rasa test nlu when using the `-P` flag. Each percentage of the training dataset used to be formed as a part of the last sampled training dataset and not as a sample from the original training dataset.
    
-   Prevent `WhitespaceTokenizer` from outputting empty list of tokens.
    
-   Add `EntityExtractor` as a required component for `EntitySynonymMapper` in a pipeline.
    
-   Better handling of input sequences longer than the maximum sequence length that the `HFTransformersNLP` models can handle.
    
    During training, messages with longer sequence length should result in an error, whereas during inference they are gracefully handled but a debug message is logged. Ideally, passing messages longer than the acceptable maximum sequence lengths of each model should be avoided.
    
-   When using the `DynamoTrackerStore`, if there are more than 100 DynamoDB tables, the tracker could attempt to re-create an existing table if that table was not among the first 100 listed by the dynamo API.
    
-   Fixed a deprication warning that pops up due to changes in numpy
    
-   Update `rasabaster` to fix an issue with syntax highlighting on "Prototype an Assistant" page.
    
    Update default stories and rules on "Prototype an Assistant" page.
    
-   Fixed a bug in the `serialise` method of the `EvaluationStore` class which resulted in a wrong end-to-end evaluation of the predicted entities.
    
-   [Forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms) with slot mappings defined in `domain.yml` must now be a dictionary (with form names as keys). The previous syntax where `forms` was simply a list of form names is still supported.
    
-   Remove BILOU tag prefix from role and group labels when creating entities.
    
-   Fixed a bug in the featurization of the boolean slot type. Previously, to set a slot value to "true", you had to set it to "1", which is in conflict with the documentation. In older versions `true` (without quotes) was also possible, but now raised an error during yaml validation.
    
-   Fixed a bug in rasa interactive. Now it exports the stories and nlu training data as yml file.
    
-   Fixed slots not being featurized before first user utterance.
    
    Fixed AugmentedMemoizationPolicy to forget the first action on the first going back
    
-   Fixed the remote URL of ConveRT model as it was recently updated by its authors.
    
-   Treat the length of OOV token as 1 to fix token align issue when OOV occurred.
    
-   Fixed the bug when entity was extracted even if it had a role or group but roles or groups were not expected.
    
-   Fixed the bug that caused `supported_language_list` of `Component` to not work correctly.
    
    To avoid confusion, only one of `supported_language_list` and `not_supported_language_list` can be set to not `None` now
    
-   Fixed issue where responses including `text: ""` and no `custom` key would incorrectly fail domain validation.
    
-   Fixed issue where extra keys other than `title` and `payload` inside of `buttons` made a response fail domain validation.
    
-   Do not filter training data in model.py but on component side.
    
-   Check if a model was provided when executing `rasa test core`. If not, print a useful error message and stop.
    
-   Transfer only response templates for retrieval intents from domain to NLU Training Data.
    
    This avoids retraining the NLU model if one of the non retrieval intent response templates are edited.
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-35 'Direct link to Improved Documentation')

-   Added documentation on `ambiguity_threshold` parameter in Fallback Actions page.
-   Remove outdated whitespace tokenizer warning in Testing Your Assistant documentation.
-   Updated Facebook Messenger channel docs with supported attachment information
-   Update `rasa shell` documentation to explain how to recreate external channel session behavior.
-   Event brokers documentation should say `url` instead of `host`.
-   Update `rasa init` documentation to include `tests/conversation_tests.md` in the resulting directory tree.
-   Update ["Validating Form Input" section](https://legacy-docs-oss.rasa.com/docs/rasa/forms#validating-form-input) to include details about how `FormValidationAction` class makes it easier to validate form slots in custom actions and how to use it.
-   Update the examples in the API docs to use YAML instead of Markdown

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-96 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.10.26\] - 2021-06-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11026---2021-06-17 'Direct link to [1.10.26] - 2021-06-17')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-26 'Direct link to Features')

-   Added `sasl_mechanism` as an optional configurable parameter for the [Kafka Producer](https://rasa.com/docs/rasa-pro/production/event-brokers#kafka-event-broker).

## \[1.10.25\] - 2021-04-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11025---2021-04-14 'Direct link to [1.10.25] - 2021-04-14')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-27 'Direct link to Features')

-   Added `partition_by_sender` flag to Kafka Producer to optionally associate events with Kafka partition based on sender\_id.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-101 'Direct link to Improvements')

-   Improved the [lock store](https://rasa.com/docs/rasa-pro/production/lock-stores) debug log message when the process has to queue because other messages have to be processed before this item.

## \[1.10.24\] - 2021-03-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11024---2021-03-29 'Direct link to [1.10.24] - 2021-03-29')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-338 'Direct link to Bugfixes')

-   Added `group_id` parameter back to `KafkaEventBroker` to fix error when instantiating event broker with a config containing the `group_id` parameter which is only relevant to the event consumer

## \[1.10.23\] - 2021-02-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11023---2021-02-22 'Direct link to [1.10.23] - 2021-02-22')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-339 'Direct link to Bugfixes')

-   Fixed bug where the conversation does not lock before handling a reminder event.

## \[1.10.22\] - 2021-02-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11022---2021-02-05 'Direct link to [1.10.22] - 2021-02-05')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-340 'Direct link to Bugfixes')

-   Backported the Rasa Open Source 2 `PikaEventBroker` implementation to address problems when using it with multiple Sanic workers.

## \[1.10.21\] - 2021-02-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11021---2021-02-01 'Direct link to [1.10.21] - 2021-02-01')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-102 'Direct link to Improvements')

-   The `url` option now supports a list of servers `url: ['10.0.0.158:32803','10.0.0.158:32804']`. Removed `group_id` because it is not a valid Kafka producer parameter.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-341 'Direct link to Bugfixes')

-   Fixed a bug that occurred when setting multiple Sanic workers in combination with a custom [Lock Store](https://rasa.com/docs/rasa-pro/production/lock-stores). Previously, if the number was set higher than 1 and you were using a custom lock store, it would reject because of a strict check to use a [Redis Lock Store](https://rasa.com/docs/rasa-pro/production/lock-stores#redislockstore).
-   Fix a bug where, if a user injects an intent using the HTTP API, slot auto-filling is not performed on the entities provided.

## \[1.10.20\] - 2020-12-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11020---2020-12-18 'Direct link to [1.10.20] - 2020-12-18')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-342 'Direct link to Bugfixes')

-   Fix scikit-learn crashing during evaluation of `ResponseSelector` predictions.

## \[1.10.19\] - 2020-12-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11019---2020-12-17 'Direct link to [1.10.19] - 2020-12-17')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-103 'Direct link to Improvements')

-   Kafka Producer connection now remains active across sends. Added support for group and client id. The Kafka producer also adds support for the `PLAINTEXT` and `SASL_SSL` protocols.
    
    DynamoDB table exists check fixed bug when more than 100 tables exist.
    
-   Replace use of `python-telegram-bot` package with `pyTelegramBotAPI`
    
-   Use response selector keys (sub-intents) as labels for plotting the confusion matrix during NLU evaluation to improve readability.
    

## \[1.10.18\] - 2020-11-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11018---2020-11-26 'Direct link to [1.10.18] - 2020-11-26')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-343 'Direct link to Bugfixes')

-   [#7340](https://github.com/rasahq/rasa/issues/7340%3E): Fixed an issues with the DynamoDB TrackerStore creating a new table entry/object for each TrackerStore update. The column `session_date` has been deprecated and should be removed manually in existing DynamoDB tables.

## \[1.10.17\] - 2020-11-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11017---2020-11-12 'Direct link to [1.10.17] - 2020-11-12')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-344 'Direct link to Bugfixes')

-   Prevent the message handling process in `PikaEventBroker` from being terminated.

## \[1.10.16\] - 2020-10-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11016---2020-10-15 'Direct link to [1.10.16] - 2020-10-15')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-345 'Direct link to Bugfixes')

-   Update Pika event broker to be a separate process and make it use a `multiprocessing.Queue` to send and process messages. This change should help avoid situations when events stop being sent after a while.

## \[1.10.15\] - 2020-10-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11015---2020-10-09 'Direct link to [1.10.15] - 2020-10-09')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-346 'Direct link to Bugfixes')

-   Fixed issue where temporary model directories were not removed after pulling from a model server. If the model pulled from the server was invalid, this could lead to large amounts of local storage usage.
-   Treat the length of OOV token as 1 to fix token align issue when OOV occurred.
-   Fixed `MappingPolicy` not predicting `action_listen` after the mapped action while running `rasa test`.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-104 'Direct link to Improvements')

-   Debug logs from `matplotlib` libraries are now hidden by default and are configurable with the `LOG_LEVEL_LIBRARIES` environment variable.

## \[1.10.14\] - 2020-09-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11014---2020-09-23 'Direct link to [1.10.14] - 2020-09-23')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-347 'Direct link to Bugfixes')

-   Fixed the remote URL of ConveRT model as it was recently updated by its authors. Also made the remote URL configurable at runtime in the corresponding tokenizer's and featurizer's configuration.

## \[1.10.13\] - 2020-09-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11013---2020-09-22 'Direct link to [1.10.13] - 2020-09-22')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-348 'Direct link to Bugfixes')

-   Remove BILOU tag prefix from role and group labels when creating entities.

## \[1.10.12\] - 2020-09-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11012---2020-09-03 'Direct link to [1.10.12] - 2020-09-03')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-349 'Direct link to Bugfixes')

-   Fix slow training of `CRFEntityExtractor` when using Entity Roles and Groups.

## \[1.10.11\] - 2020-08-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11011---2020-08-21 'Direct link to [1.10.11] - 2020-08-21')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-105 'Direct link to Improvements')

-   Do not deepcopy slots when instantiating trackers. This leads to a significant speedup when training on domains with a large number of slots.
    
-   Added more debugging logs to the [Lock Stores](https://rasa.com/docs/rasa-pro/production/lock-stores) to simplify debugging in case of
    
    connection problems.
    
    Added a new parameter `socket_timeout` to the `RedisLockStore`. If Redis doesn't answer within `socket_timeout` seconds to requests from Rasa Open Source, an error is raised. This avoids seemingly infinitely blocking connections and exposes connection problems early.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-350 'Direct link to Bugfixes')

-   Fixed a bug where domain fields such as `store_entities_as_slots` were overridden with defaults and therefore ignored.
-   If two entities are separated by a comma (or any other symbol), extract them as two separate entities.
-   If two entities are separated by a single space and uses BILOU tagging, extract them as two separate entities based on their BILOU tags.

## \[1.10.10\] - 2020-08-04[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11010---2020-08-04 'Direct link to [1.10.10] - 2020-08-04')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-351 'Direct link to Bugfixes')

-   Fixed `TypeError: expected string or bytes-like object` issue caused by integer, boolean, and null values in templates.

## \[1.10.9\] - 2020-07-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1109---2020-07-29 'Direct link to [1.10.9] - 2020-07-29')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-106 'Direct link to Improvements')

-   Rasa Open Source will no longer add `responses` to the `actions` section of the domain when persisting the domain as a file. This addresses related problems in Rasa X when Integrated Version Control introduced big diffs due to the added utterances in the `actions` section.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-352 'Direct link to Bugfixes')

-   Consider entity roles/groups during interactive learning.

## \[1.10.8\] - 2020-07-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1108---2020-07-15 'Direct link to [1.10.8] - 2020-07-15')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-353 'Direct link to Bugfixes')

-   Add 'Access-Control-Expose-Headers' for 'filename' header
-   Fixed a bug where an invalid language variable prevents rasa from finding training examples when importing Dialogflow data.

## \[1.10.7\] - 2020-07-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1107---2020-07-07 'Direct link to [1.10.7] - 2020-07-07')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-28 'Direct link to Features')

-   Add `not_supported_language_list` to component to be able to define languages that a component can NOT handle.
    
    `WhitespaceTokenizer` is not able to process languages which are not separated by whitespace. `WhitespaceTokenizer` will throw an error if it is used with Chinese, Japanese, and Thai.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-354 'Direct link to Bugfixes')

-   `WhitespaceTokenizer` only removes emoji if complete token matches emoji regex.

## \[1.10.6\] - 2020-07-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1106---2020-07-06 'Direct link to [1.10.6] - 2020-07-06')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-355 'Direct link to Bugfixes')

-   Prevent `WhitespaceTokenizer` from outputting empty list of tokens.

## \[1.10.5\] - 2020-07-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1105---2020-07-02 'Direct link to [1.10.5] - 2020-07-02')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-356 'Direct link to Bugfixes')

-   Explicitly remove all emojis which appear as unicode characters from the output of `regex.sub` inside `WhitespaceTokenizer`.

## \[1.10.4\] - 2020-07-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1104---2020-07-01 'Direct link to [1.10.4] - 2020-07-01')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-357 'Direct link to Bugfixes')

-   `WhitespaceTokenizer` does not remove vowel signs in Hindi anymore.
    
-   Previously, specifying a lock store in the endpoint configuration with a type other than `redis` or `in_memory` would lead to an `AttributeError: 'str' object has no attribute 'type'`. This bug is fixed now.
    
-   Fix `Interpreter parsed an intent ...` warning when using the `/model/parse` endpoint with an NLU-only model.
    
-   Convert entity values coming from any entity extractor to string during evaluation to avoid mismatches due to different types.
    
-   The assistant will respond through the webex channel to any user (room) communicating to it. Before the bot responded only to a fixed `roomId` set in the `credentials.yml` config file.
    

## \[1.10.3\] - 2020-06-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1103---2020-06-12 'Direct link to [1.10.3] - 2020-06-12')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-107 'Direct link to Improvements')

-   Reduced duplicate logs and warnings when running `rasa train`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-358 'Direct link to Bugfixes')

-   Remove the `clean_up_entities` method from the `DIETClassifier` and `CRFEntityExtractor` as it let to incorrect entity predictions.
    
-   Fix server crashes that occurred when Rasa Open Source pulls a model from a [model server](https://rasa.com/docs/rasa-pro/production/model-storage#load-model-from-server) and an exception was thrown during model loading (such as a domain with invalid YAML).
    

## \[1.10.2\] - 2020-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1102---2020-06-03 'Direct link to [1.10.2] - 2020-06-03')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-359 'Direct link to Bugfixes')

-   Responses used in ResponseSelector now support new lines with explicitly adding `\\n` between them.
    
-   Fixed a bug in [`rasa export`](https://rasa.com/docs/rasa-pro/command-line-interface#rasa-export)) which caused Rasa Open Source to only migrate conversation events from the last [Session configuration](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#session-configuration).
    

## \[1.10.1\] - 2020-05-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1101---2020-05-15 'Direct link to [1.10.1] - 2020-05-15')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-108 'Direct link to Improvements')

-   Creating a `Domain` using `Domain.fromDict` can no longer alter the input dictionary. Previously, there could be problems when the input dictionary was re-used for other things after creating the `Domain` from it.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-360 'Direct link to Bugfixes')

-   Don't create TensorBoard log files during prediction.
    
-   Fix: DIET breaks with empty spaCy model
    
-   Remove `clean_up_entities` from extractors that extract pre-defined entities. Just keep the clean up method for entity extractors that extract custom entities.
    
-   Fixed issue where the `DucklingHTTPExtractor` component would not work if its url contained a trailing slash.
    
-   Fix list index out of range error in `ensure_consistent_bilou_tagging`.
    

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-97 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.10.0\] - 2020-04-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1100---2020-04-28 'Direct link to [1.10.0] - 2020-04-28')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-29 'Direct link to Features')

-   Add support for entities with roles and grouping of entities in Rasa NLU.
    
    You can now define a role and/or group label in addition to the entity type for entities. Use the role label if an entity can play different roles in your assistant. For example, a city can be a destination or a departure city. The group label can be used to group multiple entities together. For example, you could group different pizza orders, so that you know what toppings goes with which pizza and what size which pizza has. For more details see [Entities Roles and Groups](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-training-data#entities-roles-and-groups).
    
    To fill slots from entities with a specific role/group, you need to either use forms or use a custom action. We updated the tracker method `get_latest_entity_values` to take an optional role/group label. If you want to use a form, you can add the specific role/group label of interest to the slot mapping function `from_entity` (see [Forms](https://legacy-docs-oss.rasa.com/docs/rasa/forms)).
    
    note
    
    Composite entities are currently just supported by the [DIETClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier) and [CRFEntityExtractor](https://legacy-docs-oss.rasa.com/docs/rasa/components#crfentityextractor).
    
-   Update training data format for NLU to support entities with a role or group label.
    
    You can now specify synonyms, roles, and groups of entities using the following data format: Markdown:
    
    ```
    [LA]{"entity": "location", "role": "city", "group": "CA", "value": "Los Angeles"}
    ```
    
    JSON:
    
    ```
    "entities": [    {        "start": 10,        "end": 12,        "value": "Los Angeles",        "entity": "location",        "role": "city",        "group": "CA",    }]
    ```
    
    The markdown format `[LA](location:Los Angeles)` is deprecated. To update your training data file just execute the following command on the terminal of your choice: `sed -i -E 's/\\[([^)]+)\\]\\(([^)]+):([^)]+)\\)/[\\1]{"entity": "\\2", "value": "\\3"}/g' nlu.md`
    
    For more information about the new data format see [Training Data Format](https://legacy-docs-oss.rasa.com/docs/rasa/training-data-format).
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-109 'Direct link to Improvements')

-   Suppressed `pika` logs when establishing the connection. These log messages mostly happened when Rasa X and RabbitMQ were started at the same time. Since RabbitMQ can take a few seconds to initialize, Rasa X has to re-try until the connection is established. In case you suspect a different problem (such as failing authentication) you can re-enable the `pika` logs by setting the log level to `DEBUG`. To run Rasa Open Source in debug mode, use the `--debug` flag. To run Rasa X in debug mode, set the environment variable `DEBUG_MODE` to `true`.
    
-   Include the source filename of a story in the failed stories
    
    Include the source filename of a story in the failed stories to make it easier to identify the file which contains the failed story.
    
-   Add confusion matrix and “confused\_with” to response selection evaluation
    
    If you are using ResponseSelectors, they now produce similiar outputs during NLU evaluation. Misclassfied responses are listed in a “confused\_with” attribute in the evaluation report. Similiarily, a confusion matrix of all responses is plotted.
    
-   Added `socketio` to the compatible channels for [Reminders and External Events](https://legacy-docs-oss.rasa.com/docs/rasa/reaching-out-to-user).
    
-   Update `POST /model/train` endpoint to accept retrieval action responses at the `responses` key of the JSON payload.
    
-   All Rasa Open Source images are now using Python 3.7 instead of Python 3.6.
    
-   Update dependencies based on the `dependabot` check.
    
-   Add dropout between `FFNN` and `DenseForSparse` layers in `DIETClassifier`, `ResponseSelector` and `EmbeddingIntentClassifier` controlled by `use_dense_input_dropout` config parameter.
    
-   `DIETClassifier` only counts as extractor in `rasa test` if it was actually trained for entity recognition.
    
-   Remove regularization gradient for variables that don't have prediction gradient.
    
-   Raise a warning in `CRFEntityExtractor` and `DIETClassifier` if entities are not correctly annotated in the training data, e.g. their start and end values do not match any start and end values of tokens.
    
-   Add `full_retrieval_intent` property to `ResponseSelector` rankings
    
-   Change default values for hyper-parameters in `EmbeddingIntentClassifier` and `DIETClassifier`
    
    Use `scale_loss=False` in `DIETClassifier`. Reduce the number of dense dimensions for sparse features of text from 512 to 256 in `EmbeddingIntentClassifier`.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-361 'Direct link to Bugfixes')

-   Fixed issue where posting to certain callback channel URLs would return a 500 error on successful posts due to invalid response format.
    
-   One word can just have one entity label.
    
    If you are using, for example, `ConveRTTokenizer` words can be split into multiple tokens. Our entity extractors assign entity labels per token. So, it might happen, that a word, that was split into two tokens, got assigned two different entity labels. This is now fixed. One word can just have one entity label at a time.
    
-   An entity label should always cover a complete word.
    
    If you are using, for example, `ConveRTTokenizer` words can be split into multiple tokens. Our entity extractors assign entity labels per token. So, it might happen, that just a part of a word has an entity label. This is now fixed. An entity label always covers a complete word.
    
-   Fixed an issue that happened when metadata is passed in a new session.
    
    Now the metadata is correctly passed to the ActionSessionStart.
    
-   Updated Python dependency `ruamel.yaml` to `>=0.16`. We recommend to use at least `0.16.10` due to the security issue [CVE-2019-20478](https://nvd.nist.gov/vuln/detail/CVE-2019-20478) which is present in in prior versions.
    

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-98 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.9.7\] - 2020-04-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#197---2020-04-23 'Direct link to [1.9.7] - 2020-04-23')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-110 'Direct link to Improvements')

-   The stream reading timeout for `rasa shell\` is now configurable by using the environment variable \`\`RASA\_SHELL\_STREAM\_READING\_TIMEOUT\_IN\_SECONDS`. This can help to fix problems when using` rasa shell\` with custom actions which run 10 seconds or longer.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-362 'Direct link to Bugfixes')

- Reverted changes in 1.9.6 that led to model incompatibility. Upgrade to 1.9.7 to fix `self.sequence_lengths_for(tf_batch_data[TEXT_SEQ_LENGTH][0]) IndexError: list index out of range` error without needing to retrain earlier 1.9 models.
 
 Therefore, all 1.9 models except for 1.9.6 will be compatible; a model trained on 1.9.6 will need to be retrained on 1.9.7.
 

## \[1.9.6\] - 2020-04-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#196---2020-04-15 'Direct link to [1.9.6] - 2020-04-15')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-363 'Direct link to Bugfixes')

- Fix rasa test nlu plotting when using multiple runs.
 
- Fixed issue where `max_number_of_predictions` was not considered when running end-to-end testing.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-99 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.9.5\] - 2020-04-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#195---2020-04-01 'Direct link to [1.9.5] - 2020-04-01')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-111 'Direct link to Improvements')

- Support for [PostgreSQL schemas](https://www.postgresql.org/docs/11/ddl-schemas.html) in [SQLTrackerStore](https://rasa.com/docs/rasa-pro/production/tracker-stores#sqltrackerstore). The `SQLTrackerStore` accesses schemas defined by the `POSTGRESQL_SCHEMA` environment variable if connected to a PostgreSQL database.
 
 The schema is added to the connection string option's `-csearch_path` key, e.g. `-options=-csearch_path=<SCHEMA_NAME>` (see the [PostgreSQL docs](https://www.postgresql.org/docs/11/contrib-dblink-connect.html) for more details). As before, if no `POSTGRESQL_SCHEMA` is defined, Rasa uses the database's default schema (`public`).
 
 The schema has to exist in the database before connecting, i.e. it needs to have been created with
 
 ```
    CREATE SCHEMA schema_name;
    ```
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-364 'Direct link to Bugfixes')

- Fixed ambiguous logging in `DIETClassifier` by adding the name of the calling class to the log message.

## \[1.9.4\] - 2020-03-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#194---2020-03-30 'Direct link to [1.9.4] - 2020-03-30')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-365 'Direct link to Bugfixes')

- Fix memory leak problem on increasing number of calls to `/model/parse` endpoint.

## \[1.9.3\] - 2020-03-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#193---2020-03-27 'Direct link to [1.9.3] - 2020-03-27')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-366 'Direct link to Bugfixes')

- Set default value for `weight_sparsity` in `ResponseSelector` to `0`. This fixes a bug in the default behavior of `ResponseSelector` which was accidentally introduced in `rasa==1.8.0`. Users should update to this version and re-train their models if `ResponseSelector` was used in their pipeline.

## \[1.9.2\] - 2020-03-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#192---2020-03-26 'Direct link to [1.9.2] - 2020-03-26')

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-36 'Direct link to Improved Documentation')

- Fix documentation to bring back Sara.

## \[1.9.1\] - 2020-03-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#191---2020-03-25 'Direct link to [1.9.1] - 2020-03-25')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-367 'Direct link to Bugfixes')

- Fix an issue where the deprecated `queue` parameter for the [Pika Event Broker](https://rasa.com/docs/rasa-pro/production/event-brokers#pika-event-broker) was ignored and Rasa Open Source published the events to the `rasa_core_events` queue instead. Note that this does not change the fact that the `queue` argument is deprecated in favor of the `queues` argument.

## \[1.9.0\] - 2020-03-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#190---2020-03-24 'Direct link to [1.9.0] - 2020-03-24')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-30 'Direct link to Features')

- Channel `hangouts` for Rasa integration with Google Hangouts Chat is now supported out-of-the-box.
 
- Add an optional path to a specific directory to download and cache the pre-trained model weights for [HFTransformersNLP](https://rasa.com/docs/rasa/2.x/components#hftransformersnlp).
 
- Add options `tensorboard_log_directory` and `tensorboard_log_level` to `EmbeddingIntentClassifier`, `DIETClasifier`, `ResponseSelector`, `EmbeddingPolicy` and `TEDPolicy`.
 
 By default `tensorboard_log_directory` is `None`. If a valid directory is provided, metrics are written during training. After the model is trained you can take a look at the training metrics in tensorboard. Execute `tensorboard --logdir <path-to-given-directory>`.
 
 Metrics can either be written after every epoch (default) or for every training step. You can specify when to write metrics using the variable `tensorboard_log_level`. Valid values are 'epoch' and 'minibatch'.
 
 We also write down a model summary, i.e. layers with inputs and types, to the given directory.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-112 'Direct link to Improvements')

- Make response timeout configurable. `rasa run`, `rasa shell` and `rasa x` can now be started with `--response-timeout <int>` to configure a response timeout of `<int>` seconds.
 
- Add full retrieval intent name to message data `ResponseSelector` will now add the full retrieval intent name e.g. `faq/which_version` to the prediction, making it accessible from the tracker.
 
- Added `PikaEventBroker` ([Pika Event Broker](https://rasa.com/docs/rasa-pro/production/event-brokers#pika-event-broker)) support for publishing to multiple queues. Messages are now published to a `fanout` exchange with name `rasa-exchange` (see [exchange-fanout](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchange-fanout) for more information on `fanout` exchanges).
 
 The former `queue` key is deprecated. Queues should now be specified as a list in the `endpoints.yml` event broker config under a new key `queues`. Example config:
 
 ```
    event_broker:  type: pika  url: localhost  username: username  password: password  queues:    - queue-1    - queue-2    - queue-3
    ```
 
- Change `rasa init` to include `tests/conversation_tests.md` file by default.
 
- The endpoint `PUT /conversations/<conversation_id>/tracker/events` no longer adds session start events (to learn more about conversation sessions, please see [Session configuration](https://rasa.com/docs/rasa-pro/nlu-based-assistants/domain#session-configuration)) in addition to the events which were sent in the request payload. To achieve the old behavior send a `GET /conversations/<conversation_id>/tracker` request before appending events.
 
- Make `scale_loss` for intents behave the same way as in versions below `1.8`, but only scale if some of the examples in a batch has probability of the golden label more than `0.5`. Introduce `scale_loss` for entities in `DIETClassifier`.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-368 'Direct link to Bugfixes')

- Fixed the bug when FormPolicy was overwriting MappingPolicy prediction (e.g. `/restart`). Priorities for [Mapping Policy](https://rasa.com/docs/rasa/2.x/policies#mapping-policy) and [Form Policy](https://rasa.com/docs/rasa/2.x/policies#form-policy) are no longer linear: `FormPolicy` priority is 5, but its prediction is ignored if `MappingPolicy` is used for prediction.
 
- Fixed issue related to storing Python `float` values as `decimal.Decimal` objects in DynamoDB tracker stores. All `decimal.Decimal` objects are now converted to `float` on tracker retrieval.
 
 Added a new docs section on [DynamoTrackerStore](https://rasa.com/docs/rasa-pro/production/tracker-stores#dynamotrackerstore).
 
- Fixed bug where `FallbackPolicy` would always fall back if the fallback action is `action_listen`.
 
- Fixed bug where starting or ending a response with `\\n\\n` led to one of the responses returned being empty.
 
- Fixes issue where model always gets retrained if multiple NLU/story files are in a directory, by sorting the list of files.
 
- Fixed ambiguous logging in DIETClassifier by adding the name of the calling class to the log message.
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-37 'Direct link to Improved Documentation')

- Restructure the “Evaluating models” documentation page and rename this page to [Testing Your Assistant](https://rasa.com/docs/rasa-pro/nlu-based-assistants/testing-your-assistant).
 
- Improved documentation on how to build and deploy an action server image for use on other servers such as Rasa X deployments.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-100 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.8.3\] - 2020-03-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#183---2020-03-27 'Direct link to [1.8.3] - 2020-03-27')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-369 'Direct link to Bugfixes')

- Fixes issue where model always gets retrained if multiple NLU/story files are in a directory, by sorting the list of files.
 
- Fixed ambiguous logging in DIETClassifier by adding the name of the calling class to the log message.
 
- Set default value for `weight_sparsity` in `ResponseSelector` to `0`. This fixes a bug in the default behavior of `ResponseSelector` which was accidentally introduced in `rasa==1.8.0`. Users should update to this version or `rasa>=1.9.3` and re-train their models if `ResponseSelector` was used in their pipeline.
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-38 'Direct link to Improved Documentation')

- Improved documentation on how to build and deploy an action server image for use on other servers such as Rasa X deployments.

## \[1.8.2\] - 2020-03-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#182---2020-03-19 'Direct link to [1.8.2] - 2020-03-19')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-370 'Direct link to Bugfixes')

- Fixed bug when installing rasa with `poetry`.
 
- Fixed bug with `EmbeddingIntentClassifier`, where results weren't the same as in 1.7.x. Fixed by setting weight sparsity to 0.
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-39 'Direct link to Improved Documentation')

- Explain how to run commands as `root` user in Rasa SDK Docker images since version `1.8.0`. Since version `1.8.0` the Rasa SDK Docker images does not longer run as `root` user by default. For commands which require `root` user usage, you have to switch back to the `root` user in your Docker image as described in [Building an Action Server Image](https://legacy-docs-oss.rasa.com/docs/rasa/action-server/deploy-action-server#building-an-action-server-image).
 
- Made improvements to Building Assistants tutorial
 

## \[1.8.1\] - 2020-03-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#181---2020-03-06 'Direct link to [1.8.1] - 2020-03-06')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-371 'Direct link to Bugfixes')

- Fixed issue with using language models like `xlnet` along with `entity_recognition` set to `True` inside `DIETClassifier`.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-101 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.8.0\] - 2020-02-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#180---2020-02-26 'Direct link to [1.8.0] - 2020-02-26')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-23 'Direct link to Deprecations and Removals')

- Removed `Agent.continue_training` and the `dump_flattened_stories` parameter from `Agent.persist`.
 
- Properties `Component.provides` and `Component.requires` are deprecated. Use `Component.required_components()` instead.
 

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-31 'Direct link to Features')

- Add default value `__other__` to `values` of a `CategoricalSlot`.
 
 All values not mentioned in the list of values of a `CategoricalSlot` will be mapped to `__other__` for featurization.
 
- Add story structure validation functionality (e.g. rasa data validate stories –max-history 5).
 
- Add [LexicalSyntacticFeaturizer](https://legacy-docs-oss.rasa.com/docs/rasa/components#lexicalsyntacticfeaturizer) to sparse featurizers.
 
 `LexicalSyntacticFeaturizer` does the same featurization as the `CRFEntityExtractor`. We extracted the featurization into a separate component so that the features can be reused and featurization is independent from the entity extraction.
 
- Integrate language models from HuggingFace's [Transformers](https://github.com/huggingface/transformers) Library.
 
 Add a new NLP component [HFTransformersNLP](https://rasa.com/docs/rasa/2.x/components#hftransformersnlp) which tokenizes and featurizes incoming messages using a specified pre-trained model with the Transformers library as the backend. Add [LanguageModelTokenizer](https://rasa.com/docs/rasa/2.x/components#languagemodeltokenizer) and [LanguageModelFeaturizer](https://legacy-docs-oss.rasa.com/docs/rasa/components#languagemodelfeaturizer) which use the information from [HFTransformersNLP](https://rasa.com/docs/rasa/2.x/components#hftransformersnlp) and sets them correctly for message object. Language models currently supported: BERT, OpenAIGPT, GPT-2, XLNet, DistilBert, RoBERTa.
 
- Added a new CLI command `rasa export` to publish tracker events from a persistent tracker store using an event broker. See [Export Conversations to an Event Broker](https://rasa.com/docs/rasa-pro/command-line-interface#rasa-export), [Tracker Stores](https://rasa.com/docs/rasa-pro/production/tracker-stores) and [Event Brokers](https://rasa.com/docs/rasa-pro/production/event-brokers) for more details.
 
- Refactor how GPU and CPU environments are configured for TensorFlow 2.0.
 
 Please refer to the documentation on [Configuring TensorFlow](https://legacy-docs-oss.rasa.com/docs/rasa/tuning-your-model/#configuring-tensorflow) to understand which environment variables to set in what scenarios. A couple of examples are shown below as well:
 
 ```
    # This specifies to use 1024 MB of memory from GPU with logical ID 0 and 2048 MB of memory from GPU with logical ID 1TF_GPU_MEMORY_ALLOC="0:1024, 1:2048"# Specifies that at most 3 CPU threads can be used to parallelize multiple non-blocking operationsTF_INTER_OP_PARALLELISM_THREADS="3"# Specifies that at most 2 CPU threads can be used to parallelize a particular operation.TF_INTRA_OP_PARALLELISM_THREADS="2"
    ```
 
- Added a new NLU component [DIETClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier) and a new policy [TEDPolicy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#ted-policy).
 
 DIET (Dual Intent and Entity Transformer) is a multi-task architecture for intent classification and entity recognition. You can read more about this component in the [DIETClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/components#dietclassifier) documentation. The new component will replace the `EmbeddingIntentClassifier` and the [CRFEntityExtractor](https://legacy-docs-oss.rasa.com/docs/rasa/components#crfentityextractor) in the future. Those two components are deprecated from now on. See [migration guide](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide/#rasa-17-to-rasa-18) for details on how to switch to the new component.
 
 [TEDPolicy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#ted-policy) is the new name for EmbeddingPolicy. `EmbeddingPolicy` is deprecated from now on. The functionality of `TEDPolicy` and `EmbeddingPolicy` is the same. Please update your configuration file to use the new name for the policy.
 
- The sentence vector of the `SpacyFeaturizer` and `MitieFeaturizer` can be calculated using max or mean pooling.
 
 To specify the pooling operation, set the option `pooling` for the `SpacyFeaturizer` or the `MitieFeaturizer` in your configuration file. The default pooling operation is `mean`. The mean pooling operation also does not take into account words, that do not have a word vector.
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-113 'Direct link to Improvements')

- Added command line argument `--conversation-id` to `rasa interactive`. If the argument is not given, `conversation_id` defaults to a random uuid.
 
- Added a new command-line argument `--init-dir` to command `rasa init` to specify the directory in which the project is initialised.
 
- Added support to send images with the twilio output channel.
 
- Part of Slack sanitization: Multiple garbled URL's in a string coming from slack will be converted into actual strings. `Example: health check of <http://eemdb.net|eemdb.net> and <http://eemdb1.net|eemdb1.net> to health check of eemdb.net and eemdb1.net`
 
- New command-line argument –conversation-id will be added and wiil give the ability to set specific conversation ID for each shell session, if not passed will be random.
 
- Messages sent to the [Pika Event Broker](https://rasa.com/docs/rasa-pro/production/event-brokers#pika-event-broker) are now persisted. This guarantees the RabbitMQ will re-send previously received messages after a crash. Note that this does not help for the case where messages are sent to an unavailable RabbitMQ instance.
 
- Added support for mattermost connector to use bot accounts.
 
- We updated our code to TensorFlow 2.
 
- Events exported using `rasa export` receive a message header if published through a `PikaEventBroker`. The header is added to the message's `BasicProperties.headers` under the `rasa-export-process-id` key (`rasa.core.constants.RASA_EXPORT_PROCESS_ID_HEADER_NAME`). The value is a UUID4 generated at each call of `rasa export`. The resulting header is a key-value pair that looks as follows:
 
 ```
    'rasa-export-process-id': 'd3b3d3ffe2bd4f379ccf21214ccfb261'
    ```
 
- Added `followlinks=True` to os.walk calls, to allow the use of symlinks in training, NLU and domain data.
 
- Support invoking a `SlackBot` by direct messaging or `@<app name>` mentions.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-372 'Direct link to Bugfixes')

- Fixed timestamp parsing warning when using DucklingHTTPExtractor
 
- Fixed issue with `action_restart` getting overridden by `action_listen` when the [Mapping Policy](https://rasa.com/docs/rasa/2.x/policies#mapping-policy) and the [Two-Stage Fallback Policy](https://rasa.com/docs/rasa/2.x/policies#two-stage-fallback-policy) are used together.
 
- Fixed incorrectly raised Error encountered in pipelines with a `ResponseSelector` and NLG.
 
 When NLU training data is split before NLU pipeline comparison, NLG responses were not also persisted and therefore training for a pipeline including the `ResponseSelector` would fail.
 
 NLG responses are now persisted along with NLU data to a `/train` directory in the `run_x/xx%_exclusion` folder.
 
- Fixed sending custom json with Twilio channel
 

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-40 'Direct link to Improved Documentation')

- Updated the documentation to properly suggest not to explicitly add utterance actions to the domain.
 
- Added user guide for reminders and external events, including `reminderbot` demo.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-102 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.7.4\] - 2020-02-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#174---2020-02-24 'Direct link to [1.7.4] - 2020-02-24')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-373 'Direct link to Bugfixes')

- Tracker stores supporting conversation sessions (`SQLTrackerStore` and `MongoTrackerStore`) do not save the tracker state to database immediately after starting a new conversation session. This leads to the number of events being saved in addition to the already-existing ones to be calculated correctly.
 
 This fixes `action_listen` events being saved twice at the beginning of conversation sessions.
 

## \[1.7.3\] - 2020-02-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#173---2020-02-21 'Direct link to [1.7.3] - 2020-02-21')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-374 'Direct link to Bugfixes')

- Fix segmentation fault when running `rasa train` or `rasa shell`.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-41 'Direct link to Improved Documentation')

- Fix doc links on “Deploying your Assistant” page

## \[1.7.2\] - 2020-02-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#172---2020-02-13 'Direct link to [1.7.2] - 2020-02-13')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-375 'Direct link to Bugfixes')

- Fixed incompatibility of Oracle with the [SQLTrackerStore](https://rasa.com/docs/rasa-pro/production/tracker-stores#sqltrackerstore), by using a `Sequence` for the primary key columns. This does not change anything for SQL databases other than Oracle. If you are using Oracle, please create a sequence with the instructions in the [SQLTrackerStore](https://rasa.com/docs/rasa-pro/production/tracker-stores#sqltrackerstore) docs.

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-42 'Direct link to Improved Documentation')

- Added section on setting up the SQLTrackerStore with Oracle
 
- Renamed “Running the Server” page to “Configuring the HTTP API”
 

## \[1.7.1\] - 2020-02-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#171---2020-02-11 'Direct link to [1.7.1] - 2020-02-11')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-376 'Direct link to Bugfixes')

- Fixed file loading of non proper UTF-8 story files, failing properly when checking for story files.
 
- Fix problem with multi-intents. Training with multi-intents using the `CountVectorsFeaturizer` together with `EmbeddingIntentClassifier` is working again.
 
- Fix bug `ValueError: Cannot concatenate sparse features as sequence dimension does not match`.
 
 When training a Rasa model that contains responses for just some of the intents, training was failing. Fixed the featurizers to return a consistent feature vector in case no response was given for a specific message.
 
- If no text features are present in `EmbeddingIntentClassifier` return the intent `None`.
 
- Resolve version conflicts: Pin version of cloudpickle to ~=1.2.0.
 

## \[1.7.0\] - 2020-01-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#170---2020-01-29 'Direct link to [1.7.0] - 2020-01-29')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-24 'Direct link to Deprecations and Removals')

- The endpoint `/conversations/<conversation_id>/execute` is now deprecated. Instead, users should use the `/conversations/<conversation_id>/trigger_intent` endpoint and thus trigger intents instead of actions.
 
- Remove option `use_cls_token` from tokenizers and option `return_sequence` from featurizers.
 
 By default all tokenizer add a special token (`__CLS__`) to the end of the list of tokens. This token will be used to capture the features of the whole utterance.
 
 The featurizers will return a matrix of size (number-of-tokens x feature-dimension) by default. This allows to train sequence models. However, the feature vector of the `__CLS__` token can be used to train non-sequence models. The corresponding classifier can decide what kind of features to use.
 

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-32 'Direct link to Features')

- Rename `templates` key in domain to `responses`.
 
 `templates` key will still work for backwards compatibility but will raise a future warning.
 
- Added a new configuration parameter, `ranking_length` to the `EmbeddingPolicy`, `EmbeddingIntentClassifier`, and `ResponseSelector` classes.
 
- External events and reminders now trigger intents (and entities) instead of actions.
 
 Add new endpoint `/conversations/<conversation_id>/trigger_intent`, which lets the user specify an intent and a list of entities that is injected into the conversation in place of a user message. The bot then predicts and executes a response action.
 
- Add `ConveRTTokenizer`.
 
 The tokenizer should be used whenever the `ConveRTFeaturizer` is used.
 
 Every tokenizer now supports the following configuration options: `intent_tokenization_flag`: Flag to check whether to split intents (default `False`). `intent_split_symbol`: Symbol on which intent should be split (default `_`)
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-114 'Direct link to Improvements')

- Remove the need of specifying utter actions in the `actions` section explicitly if these actions are already listed in the `templates` section.
 
- Entity examples that have been extracted using an external extractor are excluded from Markdown dumping in `MarkdownWriter.dumps()`. The excluded external extractors are `DucklingHTTPExtractor` and `SpacyEntityExtractor`.
 
- The `EmbeddingPolicy`, `EmbeddingIntentClassifier`, and `ResponseSelector` now by default normalize confidence levels over the top 10 results. See [Rasa 1.6 to Rasa 1.7](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide/#rasa-16-to-rasa-17) for more details.
 
- `ReminderCancelled` can now cancel multiple reminders if no name is given. It still cancels a single reminder if the reminder's name is specified.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-377 'Direct link to Bugfixes')

- Requests to `/model/train` do not longer block other requests to the Rasa server.
 
- Fixed default behavior of `rasa test core --evaluate-model-directory` when called without `--model`. Previously, the latest model file was used as `--model`. Now the default model directory is used instead.
 
 New behavior of `rasa test core --evaluate-model-directory` when given an existing file as argument for `--model`: Previously, this led to an error. Now a warning is displayed and the directory containing the given file is used as `--model`.
 
- Updated the dependency `networkx` from 2.3.0 to 2.4.0. The old version created incompatibilities when using pip.
 
 There is an imcompatibility between Rasa dependecy requests 2.22.0 and the own depedency from Rasa for networkx raising errors upon pip install. There is also a bug corrected in `requirements.txt` which used `~=` instead of `==`. All of these are fixed using networkx 2.4.0.
 
- Fixed compatibility issue with Microsoft Bot Framework Emulator if `service_url` lacked a trailing `/`.
 
- DynamoDB tracker store decimal values will now be rounded on save. Previously values exceeding 38 digits caused an unhandled error.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-103 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[1.6.2\] - 2020-01-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#162---2020-01-28 'Direct link to [1.6.2] - 2020-01-28')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-115 'Direct link to Improvements')

- Switching back to a TensorFlow release which only includes CPU support to reduce the size of the dependencies. If you want to use the TensorFlow package with GPU support, please run `pip install tensorflow-gpu==1.15.0`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-378 'Direct link to Bugfixes')

- Fixes `Exception 'Loop' object has no attribute '_ready'` error when running `rasa init`.
 
- Updated the end-to-end ValueError you recieve when you have a invalid story format to point to the updated doc link.
 

## \[1.6.1\] - 2020-01-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#161---2020-01-07 'Direct link to [1.6.1] - 2020-01-07')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-379 'Direct link to Bugfixes')

- Use an empty domain in case a model is loaded which has no domain (avoids errors when accessing `agent.doman.<some attribute>`).
 
- Replace error message with warning in tokenizers and featurizers if default parameter not set.
 
- Pin sanic patch version instead of minor version. Fixes sanic `_run_request_middleware()` error.
 
- Fix wrong calculation of additional conversation events when saving the conversation. This led to conversation events not being saved.
 
- Fix wrong order of conversation events when pushing events to conversations via `POST /conversations/<conversation_id>/tracker/events`.
 

## \[1.6.0\] - 2019-12-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#160---2019-12-18 'Direct link to [1.6.0] - 2019-12-18')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-25 'Direct link to Deprecations and Removals')

- Removed `ner_features` as a feature name from `CRFEntityExtractor`, use `text_dense_features` instead.
 
 The following settings match the previous `NGramFeaturizer`:
 
 ```
    pipeline:- name: 'CountVectorsFeaturizer'  analyzer: 'char_wb'  min_ngram: 3  max_ngram: 17  max_features: 10  min_df: 5
    ```
 
- To [use custom features in the `CRFEntityExtractor`](https://legacy-docs-oss.rasa.com/docs/rasa/components#crfentityextractor) use `text_dense_features` instead of `ner_features`. If `text_dense_features` are present in the feature set, the `CRFEntityExtractor` will automatically make use of them. Just make sure to add a dense featurizer in front of the `CRFEntityExtractor` in your pipeline and set the flag `return_sequence` to `True` for that featurizer.
 
- Deprecated `Agent.continue_training`. Instead, a model should be retrained.
 
- Specifying lookup tables directly in the NLU file is now deprecated. Please specify them in an external file.
 

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-33 'Direct link to Features')

- Replaced the warnings about missing templates, intents etc. in validator.py by debug messages.
 
- Added conversation sessions to trackers.
 
 A conversation session represents the dialog between the assistant and a user. Conversation sessions can begin in three ways: 1. the user begins the conversation with the assistant, 2. the user sends their first message after a configurable period of inactivity, or 3. a manual session start is triggered with the `/session_start` intent message. The period of inactivity after which a new conversation session is triggered is defined in the domain using the `session_expiration_time` key in the `session_config` section. The introduction of conversation sessions comprises the following changes:
 
 - Added a new event `SessionStarted` that marks the beginning of a new conversation session.
 
 - Added a new default action `ActionSessionStart`. This action takes all `SlotSet` events from the previous session and applies it to the next session.
 
 - Added a new default intent `session_start` which triggers the start of a new conversation session.
 
 - `SQLTrackerStore` and `MongoTrackerStore` only retrieve events from the last session from the database.
 
 
 note
 
 The session behavior is disabled for existing projects, i.e. existing domains without session config section.
 
- Preparation for an upcoming change in the `EmbeddingIntentClassifier`:
 
 Add option `use_cls_token` to all tokenizers. If it is set to `True`, the token `__CLS__` will be added to the end of the list of tokens. Default is set to `False`. No need to change the default value for now.
 
 Add option `return_sequence` to all featurizers. By default all featurizers return a matrix of size (1 x feature-dimension). If the option `return_sequence` is set to `True`, the corresponding featurizer will return a matrix of size (token-length x feature-dimension). See [Text Featurizers](https://legacy-docs-oss.rasa.com/docs/rasa/components#featurizers). Default value is set to `False`. However, you might want to set it to `True` if you want to use custom features in the `CRFEntityExtractor`. See [passing custom features to the `CRFEntityExtractor`](https://legacy-docs-oss.rasa.com/docs/rasa/components#crfentityextractor)
 
 Changed some featurizers to use sparse features, which should reduce memory usage with large amounts of training data significantly. Read more: [Text Featurizers](https://legacy-docs-oss.rasa.com/docs/rasa/components#featurizers) .
 
 caution
 
 These changes break model compatibility. You will need to retrain your old models!
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-116 'Direct link to Improvements')

- Added `--no-plot` option for `rasa test` command, which disables rendering of confusion matrix and histogram. By default plots will be rendered.
 
- If matplotlib couldn't set up a default backend, it will be set automatically to TkAgg/Agg one
 
- Add the option `\`random\_seed\``to the`\`rasa data split nlu\`\` command to generate reproducible train/test splits.
    
-   Changed `url` `__init__()` arguments for custom tracker stores to `host` to reflect the `__init__` arguments of currently supported tracker stores. Note that in `endpoints.yml`, these are still declared as `url`.
    
-   The `kafka-python` dependency has become as an “extra” dependency. To use the `KafkaEventConsumer`, `rasa` has to be installed with the `[kafka]` option, i.e.
    
    ```
    $ pip install rasa[kafka]
    ```
    
-   Allow creation of natural language interpreter and generator by classname reference in `endpoints.yml`.
    
-   Made it explicit that interactive learning does not work with NLU-only models.
    
    Interactive learning no longer trains NLU-only models if no model is provided and no core data is provided.
    
-   The `intent_report.json` created by `rasa test` now creates an extra field `confused_with` for each intent. This is a dictionary containing the names of the most common false positives when this intent should be predicted, and the number of such false positives.
    
-   `rasa test nlu --cross-validation` now also includes an evaluation of the response selector. As a result, the train and test F1-score, accuracy and precision is logged for the response selector. A report is also generated in the `results` folder by the name `response_selection_report.json`
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-380 'Direct link to Bugfixes')

-   If a `wait_time_between_pulls` is configured for the model server in `endpoints.yml`, this will be used instead of the default one when running Rasa X.
    
-   Training Luis data with `luis_schema_version` higher than 4.x.x will show a warning instead of throwing an exception.
    
-   Running `rasa interactive` with no NLU data now works, with the functionality of `rasa interactive core`.
    
-   When loading models from S3, namespaces (folders within a bucket) are now respected. Previously, this would result in an error upon loading the model.
    
-   “rasa init” will ask if user wants to train a model
    
-   Pin `multidict` dependency to 4.6.1 to prevent sanic from breaking, see [the Sanic GitHub issue](https://github.com/huge-success/sanic/issues/1729 'Sanic Github Issue #1729 about Multidict update breaking Sanic') for more info.
    
-   Fix errors during training and testing of `ResponseSelector`.
    

## \[1.5.3\] - 2019-12-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#153---2019-12-11 'Direct link to [1.5.3] - 2019-12-11')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-117 'Direct link to Improvements')

-   Improved error message that appears when an incorrect parameter is passed to a policy.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-381 'Direct link to Bugfixes')

-   Added `rasa/nlu/schemas/config.yml` to wheel package
    
-   Pin `multidict` dependency to 4.6.1 to prevent sanic from breaking, see [the Sanic GitHub issue](https://github.com/huge-success/sanic/issues/1729 'Sanic Github Issue #1729 about Multidict update breaking Sanic')
    

## \[1.5.2\] - 2019-12-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#152---2019-12-09 'Direct link to [1.5.2] - 2019-12-09')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-118 'Direct link to Improvements')

-   `rasa interactive` will skip the story visualization of training stories in case there are more than 200 stories. Stories created during interactive learning will be visualized as before.
    
-   The log level for SocketIO loggers, including `websockets.protocol`, `engineio.server`, and `socketio.server`, is now handled by the `LOG_LEVEL_LIBRARIES` environment variable, where the default log level is `ERROR`.
    
-   Updated all example bots and documentation to use the updated `dispatcher.utter_message()` method from rasa-sdk==1.5.0.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-382 'Direct link to Bugfixes')

-   `rasa interactive` will not load training stories in case the visualization is skipped.
    
-   Fixed error where spacy models where not found in the docker images.
    
-   Fixed unnecessary `kwargs` unpacking in `rasa.test.test_core` call in `rasa.test.test` function.
    
-   Training data files now get loaded in the same order (especially relevant to subdirectories) each time to ensure training consistency when using a random seed.
    
-   Locks for tickets in `LockStore` are immediately issued without a redundant check for their availability.
    

### Improved Documentation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-43 'Direct link to Improved Documentation')

-   Added `towncrier` to automatically collect changelog entries.
    
-   Document the pipeline for `pretrained_embeddings_convert` in the pre-configured pipelines section.
    
-   `Proactively Reaching Out to the User Using Actions` now correctly links to the endpoint specification.
    

## \[1.5.1\] - 2019-11-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#151---2019-11-27 'Direct link to [1.5.1] - 2019-11-27')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-119 'Direct link to Improvements')

-   When NLU training data is dumped as Markdown file the intents are not longer ordered alphabetically, but in the original order of given training data

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-383 'Direct link to Bugfixes')

-   End to end stories now support literal payloads which specify entities, e.g. `greet: /greet{"name": "John"}`
    
-   Slots will be correctly interpolated if there are lists in custom response templates.
    
-   Fixed compatibility issues with `rasa-sdk` `1.5`
    
-   Updated `/status` endpoint to show correct path to model archive
    

## \[1.5.0\] - 2019-11-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#150---2019-11-26 'Direct link to [1.5.0] - 2019-11-26')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-34 'Direct link to Features')

-   Added data validator that checks if domain object returned is empty. If so, exit early from the command `rasa data validate`.
    
-   Added the KeywordIntentClassifier.
    
-   Added documentation for `AugmentedMemoizationPolicy`.
    
-   Fall back to `InMemoryTrackerStore` in case there is any problem with the current tracker store.
    
-   Arbitrary metadata can now be attached to any `Event` subclass. The data must be stored under the `metadata` key when reading the event from a JSON object or dictionary.
    
-   Add command line argument `rasa x --config CONFIG`, to specify path to the policy and NLU pipeline configuration of your bot (default: `config.yml`).
    
-   Added a new NLU featurizer - `ConveRTFeaturizer` based on [ConveRT](https://github.com/PolyAI-LDN/polyai-models) model released by PolyAI.
    
-   Added a new preconfigured pipeline - `pretrained_embeddings_convert`.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-120 'Direct link to Improvements')

-   Do not retrain the entire Core model if only the `templates` section of the domain is changed.
    
-   Upgraded `jsonschema` version.
    

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-26 'Direct link to Deprecations and Removals')

-   Remove duplicate messages when creating training data (issues/1446).

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-384 'Direct link to Bugfixes')

-   `MultiProjectImporter` now imports files in the order of the import statements
    
-   Fixed server hanging forever on leaving `rasa shell` before first message
    
-   Fixed rasa init showing traceback error when user does Keyboard Interrupt before choosing a project path
    
-   `CountVectorsFeaturizer` featurizes intents only if its analyzer is set to `word`
    
-   Fixed bug where facebooks generic template was not rendered when buttons were `None`
    
-   Fixed default intents unnecessarily raising undefined parsing error
    

## \[1.4.6\] - 2019-11-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#146---2019-11-22 'Direct link to [1.4.6] - 2019-11-22')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-385 'Direct link to Bugfixes')

-   Fixed Rasa X not working when any tracker store was configured for Rasa.
    
-   Use the matplotlib backend `agg` in case the `tkinter` package is not installed.
    

## \[1.4.5\] - 2019-11-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#145---2019-11-14 'Direct link to [1.4.5] - 2019-11-14')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-386 'Direct link to Bugfixes')

-   NLU-only models no longer throw warnings about parsing features not defined in the domain
    
-   Fixed bug that stopped Dockerfiles from building version 1.4.4.
    
-   Fixed format guessing for e2e stories with intent restated as `/intent`
    

## \[1.4.4\] - 2019-11-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#144---2019-11-13 'Direct link to [1.4.4] - 2019-11-13')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-35 'Direct link to Features')

-   `PikaEventProducer` adds the RabbitMQ `App ID` message property to published messages with the value of the `RASA_ENVIRONMENT` environment variable. The message property will not be assigned if this environment variable isn't set.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-121 'Direct link to Improvements')

-   Updated Mattermost connector documentation to be more clear.
    
-   Updated format strings to f-strings where appropriate.
    
-   Updated tensorflow requirement to `1.15.0`
    
-   Dump domain using UTF-8 (to avoid `\\UXXXX` sequences in the dumped files)
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-387 'Direct link to Bugfixes')

-   Fixed exporting NLU training data in `json` format from `rasa interactive`
    
-   Fixed numpy deprecation warnings
    

## \[1.4.3\] - 2019-10-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#143---2019-10-29 'Direct link to [1.4.3] - 2019-10-29')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-388 'Direct link to Bugfixes')

-   Fixed `Connection reset by peer` errors and bot response delays when using the RabbitMQ event broker.

## \[1.4.2\] - 2019-10-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#142---2019-10-28 'Direct link to [1.4.2] - 2019-10-28')

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-27 'Direct link to Deprecations and Removals')

-   TensorFlow deprecation warnings are no longer shown when running `rasa x`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-389 'Direct link to Bugfixes')

-   Fixed `'Namespace' object has no attribute 'persist_nlu_data'` error during interactive learning
    
-   Pinned networkx~=2.3.0 to fix visualization in rasa interactive and Rasa X
    
-   Fixed `No model found` error when using `rasa run actions` with “actions” as a directory.
    

## \[1.4.1\] - 2019-10-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#141---2019-10-22 'Direct link to [1.4.1] - 2019-10-22')

Regression: changes from `1.2.12` were missing from `1.4.0`, readded them

## \[1.4.0\] - 2019-10-19[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#140---2019-10-19 'Direct link to [1.4.0] - 2019-10-19')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-36 'Direct link to Features')

-   add flag to CLI to persist NLU training data if needed
    
-   log a warning if the `Interpreter` picks up an intent or an entity that does not exist in the domain file.
    
-   added `DynamoTrackerStore` to support persistence of agents running on AWS
    
-   added docstrings for `TrackerStore` classes
    
-   added buttons and images to mattermost.
    
-   `CRFEntityExtractor` updated to accept arbitrary token-level features like word vectors (issues/4214)
    
-   `SpacyFeaturizer` updated to add `ner_features` for `CRFEntityExtractor`
    
-   Sanitizing incoming messages from slack to remove slack formatting like `<mailto:xyz@rasa.com|xyz@rasa.com>` or `<http://url.com|url.com>` and substitute it with original content
    
-   Added the ability to configure the number of Sanic worker processes in the HTTP server (`rasa.server`) and input channel server (`rasa.core.agent.handle_channels()`). The number of workers can be set using the environment variable `SANIC_WORKERS` (default: 1). A value of >1 is allowed only in combination with `RedisLockStore` as the lock store.
    
-   Botframework channel can handle uploaded files in `UserMessage` metadata.
    
-   Added data validator that checks there is no duplicated example data across multiples intents
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-122 'Direct link to Improvements')

-   Unknown sections in markdown format (NLU data) are not ignored anymore, but instead an error is raised.
    
-   It is now easier to add metadata to a `UserMessage` in existing channels. You can do so by overwriting the method `get_metadata`. The return value of this method will be passed to the `UserMessage` object.
    
-   Tests can now be run in parallel
    
-   Serialise `DialogueStateTracker` as json instead of pickle. **DEPRECATION warning**: Deserialisation of pickled trackers will be deprecated in version 2.0. For now, trackers are still loaded from pickle but will be dumped as json in any subsequent save operations.
    
-   Event brokers are now also passed to custom tracker stores (using the `event_broker` parameter)
    
-   Don't run the Rasa Docker image as `root`.
    
-   Use multi-stage builds to reduce the size of the Rasa Docker image.
    
-   Updated the `/status` api route to use the actual model file location instead of the `tmp` location.
    

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-28 'Direct link to Deprecations and Removals')

-   **Removed Python 3.5 support**

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-390 'Direct link to Bugfixes')

-   fixed missing `tkinter` dependency for running tests on Ubuntu
    
-   fixed issue with `conversation` JSON serialization
    
-   fixed the hanging HTTP call with `ner_duckling_http` pipeline
    
-   fixed Interactive Learning intent payload messages saving in nlu files
    
-   fixed DucklingHTTPExtractor dimensions by actually applying to the request
    

## \[1.3.10\] - 2019-10-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1310---2019-10-18 'Direct link to [1.3.10] - 2019-10-18')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-37 'Direct link to Features')

-   Can now pass a package as an argument to the `--actions` parameter of the `rasa run actions` command.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-391 'Direct link to Bugfixes')

-   Fixed visualization of stories with entities which led to a failing visualization in Rasa X

## \[1.3.9\] - 2019-10-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#139---2019-10-10 'Direct link to [1.3.9] - 2019-10-10')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-38 'Direct link to Features')

-   Port of 1.2.10 (support for RabbitMQ TLS authentication and `port` key in event broker endpoint config).
    
-   Port of 1.2.11 (support for passing a CA file for SSL certificate verification via the –ssl-ca-file flag).
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-392 'Direct link to Bugfixes')

-   Fixed the hanging HTTP call with `ner_duckling_http` pipeline.
    
-   Fixed text processing of `intent` attribute inside `CountVectorFeaturizer`.
    
-   Fixed `argument of type 'NoneType' is not iterable` when using `rasa shell`, `rasa interactive` / `rasa run`
    

## \[1.3.8\] - 2019-10-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#138---2019-10-08 'Direct link to [1.3.8] - 2019-10-08')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-123 'Direct link to Improvements')

-   Policies now only get imported if they are actually used. This removes TensorFlow warnings when starting Rasa X

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-393 'Direct link to Bugfixes')

-   Fixed error `Object of type 'MaxHistoryTrackerFeaturizer' is not JSON serializable` when running `rasa train core`
    
-   Default channel `send_` methods no longer support kwargs as they caused issues in incompatible channels
    

## \[1.3.7\] - 2019-09-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#137---2019-09-27 'Direct link to [1.3.7] - 2019-09-27')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-394 'Direct link to Bugfixes')

-   re-added TLS, SRV dependencies for PyMongo
    
-   socketio can now be run without turning on the `--enable-api` flag
    
-   MappingPolicy no longer fails when the latest action doesn't have a policy
    

## \[1.3.6\] - 2019-09-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#136---2019-09-21 'Direct link to [1.3.6] - 2019-09-21')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-39 'Direct link to Features')

-   Added the ability for users to specify a conversation id to send a message to when using the `RasaChat` input channel.

## \[1.3.5\] - 2019-09-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#135---2019-09-20 'Direct link to [1.3.5] - 2019-09-20')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-395 'Direct link to Bugfixes')

-   Fixed issue where `rasa init` would fail without spaCy being installed

## \[1.3.4\] - 2019-09-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#134---2019-09-20 'Direct link to [1.3.4] - 2019-09-20')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-40 'Direct link to Features')

-   Added the ability to set the `backlog` parameter in Sanics `run()` method using the `SANIC_BACKLOG` environment variable. This parameter sets the number of unaccepted connections the server allows before refusing new connections. A default value of 100 is used if the variable is not set.
    
-   Status endpoint (`/status`) now also returns the number of training processes currently running
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-396 'Direct link to Bugfixes')

-   Added the ability to properly deal with spaCy `Doc`\-objects created on empty strings as discussed in [issue #4445](https://github.com/RasaHQ/rasa/issues/4445 'Rasa issue #4445: Handling spaCy objects on empty strings'). Only training samples that actually bear content are sent to `self.nlp.pipe` for every given attribute. Non-content-bearing samples are converted to empty `Doc`\-objects. The resulting lists are merged with their preserved order and properly returned.
    
-   asyncio warnings are now only printed if the callback takes more than 100ms (up from 1ms).
    
-   `agent.load_model_from_server` no longer affects logging.
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-124 'Direct link to Improvements')

-   The endpoint `POST /model/train` no longer supports specifying an output directory for the trained model using the field `out`. Instead you can choose whether you want to save the trained model in the default model directory (`models`) (default behavior) or in a temporary directory by specifying the `save_to_default_model_directory` field in the training request.

## \[1.3.3\] - 2019-09-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#133---2019-09-13 'Direct link to [1.3.3] - 2019-09-13')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-397 'Direct link to Bugfixes')

-   Added a check to avoid training `CountVectorizer` for a particular attribute of a message if no text is provided for that attribute across the training data.
    
-   Default one-hot representation for label featurization inside `EmbeddingIntentClassifier` if label features don't exist.
    
-   Policy ensemble no longer incorrectly wrings “missing mapping policy” when mapping policy is present.
    
-   “text” from `utter_custom_json` now correctly saved to tracker when using telegram channel
    

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-29 'Direct link to Deprecations and Removals')

-   Removed computation of `intent_spacy_doc`. As a result, none of the spacy components process intents now.

## \[1.3.2\] - 2019-09-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#132---2019-09-10 'Direct link to [1.3.2] - 2019-09-10')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-398 'Direct link to Bugfixes')

-   SQL tracker events are retrieved ordered by timestamps. This fixes interactive learning events being shown in the wrong order.

## \[1.3.1\] - 2019-09-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#131---2019-09-09 'Direct link to [1.3.1] - 2019-09-09')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-125 'Direct link to Improvements')

-   Pin gast to == 0.2.2

## \[1.3.0\] - 2019-09-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#130---2019-09-05 'Direct link to [1.3.0] - 2019-09-05')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-41 'Direct link to Features')

-   Added option to persist nlu training data (default: False)
    
-   option to save stories in e2e format for interactive learning
    
-   bot messages contain the `timestamp` of the `BotUttered` event, which can be used in channels
    
-   `FallbackPolicy` can now be configured to trigger when the difference between confidences of two predicted intents is too narrow
    
-   experimental training data importer which supports training with data of multiple sub bots. Please see the docs for more information.
    
-   throw error during training when triggers are defined in the domain without `MappingPolicy` being present in the policy ensemble
    
-   The tracker is now available within the interpreter's `parse` method, giving the ability to create interpreter classes that use the tracker state (eg. slot values) during the parsing of the message. More details on motivation of this change see issues/3015.
    
-   add example bot `knowledgebasebot` to showcase the usage of `ActionQueryKnowledgeBase`
    
-   `softmax` starspace loss for both `EmbeddingPolicy` and `EmbeddingIntentClassifier`
    
-   `balanced` batching strategy for both `EmbeddingPolicy` and `EmbeddingIntentClassifier`
    
-   `max_history` parameter for `EmbeddingPolicy`
    
-   Successful predictions of the NER are written to a file if `--successes` is set when running `rasa test nlu`
    
-   Incorrect predictions of the NER are written to a file by default. You can disable it via `--no-errors`.
    
-   New NLU component `ResponseSelector` added for the task of response selection
    
-   Message data attribute can contain two more keys - `response_key`, `response` depending on the training data
    
-   New action type implemented by `ActionRetrieveResponse` class and identified with `response_` prefix
    
-   Vocabulary sharing inside `CountVectorsFeaturizer` with `use_shared_vocab` flag. If set to True, vocabulary of corpus is shared between text, intent and response attributes of message
    
-   Added an option to share the hidden layer weights of text input and label input inside `EmbeddingIntentClassifier` using the flag `share_hidden_layers`
    
-   New type of training data file in NLU which stores response phrases for response selection task.
    
-   Add flag `intent_split_symbol` and `intent_tokenization_flag` to all `WhitespaceTokenizer`, `JiebaTokenizer` and `SpacyTokenizer`
    
-   Added evaluation for response selector. Creates a report `response_selection_report.json` inside `--out` directory.
    
-   argument `--config-endpoint` to specify the URL from which `rasa x` pulls the runtime configuration (endpoints and credentials)
    
-   `LockStore` class storing instances of `TicketLock` for every `conversation_id`
    
-   environment variables `SQL_POOL_SIZE` (default: 50) and `SQL_MAX_OVERFLOW` (default: 100) can be set to control the pool size and maximum pool overflow for `SQLTrackerStore` when used with the `postgresql` dialect
    
-   Add a bot\_challenge intent and a utter\_iamabot action to all example projects and the rasa init bot.
    
-   Allow sending attachments when using the socketio channel
    
-   `rasa data validate` will fail with a non-zero exit code if validation fails
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-126 'Direct link to Improvements')

-   added character-level `CountVectorsFeaturizer` with empirically found parameters into the `supervised_embeddings` NLU pipeline template
    
-   NLU evaluations now also stores its output in the output directory like the core evaluation
    
-   show warning in case a default path is used instead of a provided, invalid path
    
-   compare mode of `rasa train core` allows the whole core config comparison, naming style of models trained for comparison is changed (this is a breaking change)
    
-   pika keeps a single connection open, instead of open and closing on each incoming event
    
-   `RasaChatInput` fetches the public key from the Rasa X API. The key is used to decode the bearer token containing the conversation ID. This requires `rasa-x>=0.20.2`.
    
-   more specific exception message when loading custom components depending on whether component's path or class name is invalid or can't be found in the global namespace
    
-   change priorities so that the `MemoizationPolicy` has higher priority than the `MappingPolicy`
    
-   substitute LSTM with Transformer in `EmbeddingPolicy`
    
-   `EmbeddingPolicy` can now use `MaxHistoryTrackerFeaturizer`
    
-   non zero `evaluate_on_num_examples` in `EmbeddingPolicy` and `EmbeddingIntentClassifier` is the size of hold out validation set that is excluded from training data
    
-   defaults parameters and architectures for both `EmbeddingPolicy` and `EmbeddingIntentClassifier` are changed (this is a breaking change)
    
-   evaluation of NER does not include 'no-entity' anymore
    
-   `--successes` for `rasa test nlu` is now boolean values. If set incorrect/successful predictions are saved in a file.
    
-   `--errors` is renamed to `--no-errors` and is now a boolean value. By default incorrect predictions are saved in a file. If `--no-errors` is set predictions are not written to a file.
    
-   Remove `label_tokenization_flag` and `label_split_symbol` from `EmbeddingIntentClassifier`. Instead move these parameters to `Tokenizers`.
    
-   Process features of all attributes of a message, i.e. - text, intent and response inside the respective component itself. For e.g. - intent of a message is now tokenized inside the tokenizer itself.
    
-   Deprecate `as_markdown` and `as_json` in favour of `nlu_as_markdown` and `nlu_as_json` respectively.
    
-   pin python-engineio >= 3.9.3
    
-   update python-socketio req to >= 4.3.1
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-399 'Direct link to Bugfixes')

-   `rasa test nlu` with a folder of configuration files
    
-   `MappingPolicy` standard featurizer is set to `None`
    
-   Removed `text` parameter from send\_attachment function in slack.py to avoid duplication of text output to slackbot
    
-   server `/status` endpoint reports status when an NLU-only model is loaded
    

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-30 'Direct link to Deprecations and Removals')

-   Removed `--report` argument from `rasa test nlu`. All output files are stored in the `--out` directory.

## \[1.2.12\] - 2019-10-16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1212---2019-10-16 'Direct link to [1.2.12] - 2019-10-16')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-42 'Direct link to Features')

-   Support for transit encryption with Redis via `use_ssl: True` in the tracker store config in endpoints.yml

## \[1.2.11\] - 2019-10-09[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1211---2019-10-09 'Direct link to [1.2.11] - 2019-10-09')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-43 'Direct link to Features')

-   Support for passing a CA file for SSL certificate verification via the –ssl-ca-file flag

## \[1.2.10\] - 2019-10-08[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1210---2019-10-08 'Direct link to [1.2.10] - 2019-10-08')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-44 'Direct link to Features')

-   Added support for RabbitMQ TLS authentication. The following environment variables need to be set: `RABBITMQ_SSL_CLIENT_CERTIFICATE` - path to the SSL client certificate (required) `RABBITMQ_SSL_CLIENT_KEY` - path to the SSL client key (required) `RABBITMQ_SSL_CA_FILE` - path to the SSL CA file (optional, for certificate verification) `RABBITMQ_SSL_KEY_PASSWORD` - SSL private key password (optional)
    
-   Added ability to define the RabbitMQ port using the `port` key in the `event_broker` endpoint config.
    

## \[1.2.9\] - 2019-09-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#129---2019-09-17 'Direct link to [1.2.9] - 2019-09-17')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-400 'Direct link to Bugfixes')

-   Correctly pass SSL flag values to x CLI command (backport of

## \[1.2.8\] - 2019-09-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#128---2019-09-10 'Direct link to [1.2.8] - 2019-09-10')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-401 'Direct link to Bugfixes')

-   SQL tracker events are retrieved ordered by timestamps. This fixes interactive learning events being shown in the wrong order. Backport of `1.3.2` patch (PR #4427).

## \[1.2.7\] - 2019-09-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#127---2019-09-02 'Direct link to [1.2.7] - 2019-09-02')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-402 'Direct link to Bugfixes')

-   Added `query` dictionary argument to `SQLTrackerStore` which will be appended to the SQL connection URL as query parameters.

## \[1.2.6\] - 2019-09-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#126---2019-09-02 'Direct link to [1.2.6] - 2019-09-02')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-403 'Direct link to Bugfixes')

-   fixed bug that occurred when sending template `elements` through a channel that doesn't support them

## \[1.2.5\] - 2019-08-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#125---2019-08-26 'Direct link to [1.2.5] - 2019-08-26')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-45 'Direct link to Features')

-   SSL support for `rasa run` command. Certificate can be specified using `--ssl-certificate` and `--ssl-keyfile`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-404 'Direct link to Bugfixes')

-   made default augmentation value consistent across repo
    
-   `'/restart'` will now also restart the bot if the tracker is paused
    

## \[1.2.4\] - 2019-08-23[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#124---2019-08-23 'Direct link to [1.2.4] - 2019-08-23')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-405 'Direct link to Bugfixes')

-   the `SocketIO` input channel now allows accesses from other origins (fixes `SocketIO` channel on Rasa X)

## \[1.2.3\] - 2019-08-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#123---2019-08-15 'Direct link to [1.2.3] - 2019-08-15')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-127 'Direct link to Improvements')

-   messages with multiple entities are now handled properly with e2e evaluation
    
-   `data/test_evaluations/end_to_end_story.md` was re-written in the restaurantbot domain
    

## \[1.2.3\] - 2019-08-15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#123---2019-08-15-1 'Direct link to [1.2.3] - 2019-08-15')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-128 'Direct link to Improvements')

-   messages with multiple entities are now handled properly with e2e evaluation
    
-   `data/test_evaluations/end_to_end_story.md` was re-written in the restaurantbot domain
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-406 'Direct link to Bugfixes')

-   Free text input was not allowed in the Rasa shell when the response template contained buttons, which has now been fixed.

## \[1.2.2\] - 2019-08-07[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#122---2019-08-07 'Direct link to [1.2.2] - 2019-08-07')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-407 'Direct link to Bugfixes')

-   `UserUttered` events always got the same timestamp

## \[1.2.1\] - 2019-08-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#121---2019-08-06 'Direct link to [1.2.1] - 2019-08-06')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-46 'Direct link to Features')

-   Docs now have an `EDIT THIS PAGE` button

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-408 'Direct link to Bugfixes')

-   `Flood control exceeded` error in Telegram connector which happened because the webhook was set twice

## \[1.2.0\] - 2019-08-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#120---2019-08-01 'Direct link to [1.2.0] - 2019-08-01')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-47 'Direct link to Features')

-   add root route to server started without `--enable-api` parameter
    
-   add `--evaluate-model-directory` to `rasa test core` to evaluate models from `rasa train core -c <config-1> <config-2>`
    
-   option to send messages to the user by calling `POST /conversations/\{conversation_id\}/execute`
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-129 'Direct link to Improvements')

-   `Agent.update_model()` and `Agent.handle_message()` now work without needing to set a domain or a policy ensemble
    
-   Update pytype to `2019.7.11`
    
-   new event broker class: `SQLProducer`. This event broker is now used when running locally with Rasa X
    
-   API requests are not longer logged to `rasa_core.log` by default in order to avoid problems when running on OpenShift (use `--log-file rasa_core.log` to retain the old behavior)
    
-   `metadata` attribute added to `UserMessage`
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-409 'Direct link to Bugfixes')

-   `rasa test core` can handle compressed model files
    
-   rasa can handle story files containing multi line comments
    
-   template will retain { if escaped with {. e.g. {{“foo”: {bar}}} will result in {“foo”: “replaced value”}
    

## \[1.1.8\] - 2019-07-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#118---2019-07-25 'Direct link to [1.1.8] - 2019-07-25')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-48 'Direct link to Features')

-   `TrainingFileImporter` interface to support customizing the process of loading training data
    
-   fill slots for custom templates
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-130 'Direct link to Improvements')

-   `Agent.update_model()` and `Agent.handle_message()` now work without needing to set a domain or a policy ensemble
    
-   update pytype to `2019.7.11`
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-410 'Direct link to Bugfixes')

-   interactive learning bug where reverted user utterances were dumped to training data
    
-   added timeout to terminal input channel to avoid freezing input in case of server errors
    
-   fill slots for image, buttons, quick\_replies and attachments in templates
    
-   `rasa train core` in comparison mode stores the model files compressed (`tar.gz` files)
    
-   slot setting in interactive learning with the TwoStageFallbackPolicy
    

## \[1.1.7\] - 2019-07-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#117---2019-07-18 'Direct link to [1.1.7] - 2019-07-18')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-49 'Direct link to Features')

-   added optional pymongo dependencies `[tls, srv]` to `requirements.txt` for better mongodb support
    
-   `case_sensitive` option added to `WhiteSpaceTokenizer` with `true` as default.
    

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-411 'Direct link to Bugfixes')

-   validation no longer throws an error during interactive learning
    
-   fixed wrong cleaning of `use_entities` in case it was a list and not `True`
    
-   updated the server endpoint `/model/parse` to handle also messages with the intent prefix
    
-   fixed bug where “No model found” message appeared after successfully running the bot
    
-   debug logs now print to `rasa_core.log` when running `rasa x -vv` or `rasa run -vv`
    

## \[1.1.6\] - 2019-07-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#116---2019-07-12 'Direct link to [1.1.6] - 2019-07-12')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-50 'Direct link to Features')

-   rest channel supports setting a message's input\_channel through a field `input_channel` in the request body

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-131 'Direct link to Improvements')

-   recommended syntax for empty `use_entities` and `ignore_entities` in the domain file has been updated from `False` or `None` to an empty list (`[]`)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-412 'Direct link to Bugfixes')

-   `rasa run` without `--enable-api` does not require a local model anymore
    
-   using `rasa run` with `--enable-api` to run a server now prints “running Rasa server” instead of “running Rasa Core server”
    
-   actions, intents, and utterances created in `rasa interactive` can no longer be empty
    

## \[1.1.5\] - 2019-07-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#115---2019-07-10 'Direct link to [1.1.5] - 2019-07-10')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-51 'Direct link to Features')

-   debug logging now tells you which tracker store is connected
    
-   the response of `/model/train` now includes a response header for the trained model filename
    
-   `Validator` class to help developing by checking if the files have any errors
    
-   project's code is now linted using flake8
    
-   `info` log when credentials were provided for multiple channels and channel in `--connector` argument was specified at the same time
    
-   validate export paths in interactive learning
    

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-132 'Direct link to Improvements')

-   deprecate `rasa.core.agent.handle_channels(...)\`. Please use \`\`rasa.run(...)`or`rasa.core.run.configure\_app\` instead.
 
- `Agent.load()` also accepts `tar.gz` model file
 

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-31 'Direct link to Deprecations and Removals')

- revert the stripping of trailing slashes in endpoint URLs since this can lead to problems in case the trailing slash is actually wanted
 
- starter packs were removed from Github and are therefore no longer tested by Travis script
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-413 'Direct link to Bugfixes')

- all temporal model files are now deleted after stopping the Rasa server
 
- `rasa shell nlu` now outputs unicode characters instead of `\\uxxxx` codes
 
- fixed PUT /model with model\_server by deserializing the model\_server to EndpointConfig.
 
- `x in AnySlotDict` is now `True` for any `x`, which fixes empty slot warnings in interactive learning
 
- `rasa train` now also includes NLU files in other formats than the Rasa format
 
- `rasa train core` no longer crashes without a `--domain` arg
 
- `rasa interactive` now looks for endpoints in `endpoints.yml` if no `--endpoints` arg is passed
 
- custom files, e.g. custom components and channels, load correctly when using the command line interface
 
- `MappingPolicy` now works correctly when used as part of a PolicyEnsemble
 

## \[1.1.4\] - 2019-06-18[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#114---2019-06-18 'Direct link to [1.1.4] - 2019-06-18')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-52 'Direct link to Features')

- unfeaturize single entities
 
- added agent readiness check to the `/status` resource
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-133 'Direct link to Improvements')

- removed leading underscore from name of '\_create\_initial\_project' function.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-414 'Direct link to Bugfixes')

- fixed bug where facebook quick replies were not rendering
 
- take FB quick reply payload rather than text as input
 
- fixed bug where training\_data path in metadata.json was an absolute path
 

## \[1.1.3\] - 2019-06-14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#113---2019-06-14 'Direct link to [1.1.3] - 2019-06-14')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-415 'Direct link to Bugfixes')

- fixed any inconsistent type annotations in code and some bugs revealed by type checker

## \[1.1.2\] - 2019-06-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#112---2019-06-13 'Direct link to [1.1.2] - 2019-06-13')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-416 'Direct link to Bugfixes')

- fixed duplicate events appearing in tracker when using a PostgreSQL tracker store

## \[1.1.1\] - 2019-06-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#111---2019-06-13 'Direct link to [1.1.1] - 2019-06-13')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-417 'Direct link to Bugfixes')

- fixed compatibility with Rasa SDK
 
- bot responses can contain `custom` messages besides other message types
 

## \[1.1.0\] - 2019-06-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#110---2019-06-13 'Direct link to [1.1.0] - 2019-06-13')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-53 'Direct link to Features')

- nlu configs can now be directly compared for performance on a dataset in `rasa test nlu`

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-134 'Direct link to Improvements')

- update the tracker in interactive learning through reverting and appending events instead of replacing the tracker
 
- `POST /conversations/\{conversation_id\}/tracker/events` supports a list of events
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-418 'Direct link to Bugfixes')

- fixed creation of `RasaNLUHttpInterpreter`
 
- form actions are included in domain warnings
 
- default actions, which are overriden by custom actions and are listed in the domain are excluded from domain warnings
 
- SQL `data` column type to `Text` for compatibility with MySQL
 
- non-featurizer training parameters don't break SklearnPolicy anymore
 

## \[1.0.9\] - 2019-06-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#109---2019-06-10 'Direct link to [1.0.9] - 2019-06-10')

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-135 'Direct link to Improvements')

- revert PR #3739 (as this is a breaking change): set `PikaProducer` and `KafkaProducer` default queues back to `rasa_core_events`

## \[1.0.8\] - 2019-06-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#108---2019-06-10 'Direct link to [1.0.8] - 2019-06-10')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-54 'Direct link to Features')

- support for specifying full database urls in the `SQLTrackerStore` configuration
 
- maximum number of predictions can be set via the environment variable `MAX_NUMBER_OF_PREDICTIONS` (default is 10)
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-136 'Direct link to Improvements')

- default `PikaProducer` and `KafkaProducer` queues to `rasa_production_events`
 
- exclude unfeaturized slots from domain warnings
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-419 'Direct link to Bugfixes')

- loading of additional training data with the `SkillSelector`
 
- strip trailing slashes in endpoint URLs
 

## \[1.0.7\] - 2019-06-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#107---2019-06-06 'Direct link to [1.0.7] - 2019-06-06')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-55 'Direct link to Features')

- added argument `--rasa-x-port` to specify the port of Rasa X when running Rasa X locally via `rasa x`

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-420 'Direct link to Bugfixes')

- slack notifications from bots correctly render text
 
- fixed usage of `--log-file` argument for `rasa run` and `rasa shell`
 
- check if correct tracker store is configured in local mode
 

## \[1.0.6\] - 2019-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#106---2019-06-03 'Direct link to [1.0.6] - 2019-06-03')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-421 'Direct link to Bugfixes')

- fixed backwards incompatible utils changes

## \[1.0.5\] - 2019-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#105---2019-06-03 'Direct link to [1.0.5] - 2019-06-03')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-422 'Direct link to Bugfixes')

- fixed spacy being a required dependency (regression)

## \[1.0.4\] - 2019-06-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#104---2019-06-03 'Direct link to [1.0.4] - 2019-06-03')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-56 'Direct link to Features')

- automatic creation of index on the `sender_id` column when using an SQL tracker store. If you have an existing data and you are running into performance issues, please make sure to add an index manually using `CREATE INDEX event_idx_sender_id ON events (sender_id);`.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-137 'Direct link to Improvements')

- NLU evaluation in cross-validation mode now also provides intent/entity reports, confusion matrix, etc.

## \[1.0.3\] - 2019-05-30[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#103---2019-05-30 'Direct link to [1.0.3] - 2019-05-30')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-423 'Direct link to Bugfixes')

- non-ascii characters render correctly in stories generated from interactive learning
 
- validate domain file before usage, e.g. print proper error messages if domain file is invalid instead of raising errors
 

## \[1.0.2\] - 2019-05-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#102---2019-05-29 'Direct link to [1.0.2] - 2019-05-29')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-57 'Direct link to Features')

- added `domain_warnings()` method to `Domain` which returns a dict containing the diff between supplied `actions`, `intents`, `entities`, `slots` and what's contained in the domain

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-424 'Direct link to Bugfixes')

- fix lookup table files failed to load issues/3622
 
- buttons can now be properly selected during cmdline chat or when in interactive learning
 
- set slots correctly when events are added through the API
 
- mapping policy no longer ignores NLU threshold
 
- mapping policy priority is correctly persisted
 

## \[1.0.1\] - 2019-05-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#101---2019-05-21 'Direct link to [1.0.1] - 2019-05-21')

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-425 'Direct link to Bugfixes')

- updated installation command in docs for Rasa X

## \[1.0.0\] - 2019-05-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#100---2019-05-21 'Direct link to [1.0.0] - 2019-05-21')

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-58 'Direct link to Features')

- added arguments to set the file paths for interactive training
 
- added quick reply representation for command-line output
 
- added option to specify custom button type for Facebook buttons
 
- added tracker store persisting trackers into a SQL database (`SQLTrackerStore`)
 
- added rasa command line interface and API
 
- Rasa HTTP training endpoint at `POST /jobs`. This endpoint will train a combined Rasa Core and NLU model
 
- `ReminderCancelled(action_name)` event to cancel given action\_name reminder for current user
 
- Rasa HTTP intent evaluation endpoint at `POST /intentEvaluation`. This endpoints performs an intent evaluation of a Rasa model
 
- option to create template for new utterance action in `interactive learning`
 
- you can now choose actions previously created in the same session in `interactive learning`
 
- add formatter 'black'
 
- channel-specific utterances via the `- "channel":` key in utterance templates
 
- arbitrary json messages via the `- "custom":` key in utterance templates and via `utter_custom_json()` method in custom actions
 
- support to load sub skills (domain, stories, nlu data)
 
- support to select which sub skills to load through `import` section in `config.yml`
 
- support for spaCy 2.1
 
- a model for an agent can now also be loaded from a remote storage
 
- log level can be set via environment variable `LOG_LEVEL`
 
- add `--store-uncompressed` to train command to not compress Rasa model
 
- log level of libraries, such as tensorflow, can be set via environment variable `LOG_LEVEL_LIBRARIES`
 
- if no spaCy model is linked upon building a spaCy pipeline, an appropriate error message is now raised with instructions for linking one
 

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-138 'Direct link to Improvements')

- renamed all CLI parameters containing any `_` to use dashes `-` instead (GNU standard)
 
- renamed `rasa_core` package to `rasa.core`
 
- for interactive learning only include manually annotated and ner\_crf entities in nlu export
 
- made `message_id` an additional argument to `interpreter.parse`
 
- changed removing punctuation logic in `WhitespaceTokenizer`
 
- `training_processes` in the Rasa NLU data router have been renamed to `worker_processes`
 
- created a common utils package `rasa.utils` for nlu and core, common methods like `read_yaml` moved there
 
- removed `--num_threads` from run command (server will be asynchronous but running in a single thread)
 
- the `_check_token()` method in `RasaChat` now authenticates against `/auth/verify` instead of `/user`
 
- removed `--pre_load` from run command (Rasa NLU server will just have a maximum of one model and that model will be loaded by default)
 
- changed file format of a stored trained model from the Rasa NLU server to `tar.gz`
 
- train command uses fallback config if an invalid config is given
 
- test command now compares multiple models if a list of model files is provided for the argument `--model`
 
- Merged rasa.core and rasa.nlu server into a single server. See swagger file in `docs/_static/spec/server.yaml` for available endpoints.
 
- `utter_custom_message()` method in rasa\_core\_sdk has been renamed to `utter_elements()`
 
- updated dependencies. as part of this, models for spacy need to be reinstalled for 2.1 (from 2.0)
 
- make sure all command line arguments for `rasa test` and `rasa interactive` are actually used, removed arguments that were not used at all (e.g. `--core` for `rasa test`)
 

### Deprecations and Removals[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-32 'Direct link to Deprecations and Removals')

- removed possibility to execute `python -m rasa_core.train` etc. (e.g. scripts in `rasa.core` and `rasa.nlu`). Use the CLI for rasa instead, e.g. `rasa train core`.
 
- removed `_sklearn_numpy_warning_fix` from the `SklearnIntentClassifier`
 
- removed `Dispatcher` class from core
 
- removed projects: the Rasa NLU server now has a maximum of one model at a time loaded.
 

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-426 'Direct link to Bugfixes')

- evaluating core stories with two stage fallback gave an error, trying to handle None for a policy
 
- the `/evaluate` route for the Rasa NLU server now runs evaluation in a parallel process, which prevents the currently loaded model unloading
 
- added missing implementation of the `keys()` function for the Redis Tracker Store
 
- in interactive learning: only updates entity values if user changes annotation
 
- log options from the command line interface are applied (they overwrite the environment variable)
 
- all message arguments (kwargs in dispatcher.utter methods, as well as template args) are now sent through to output channels
 
- utterance templates defined in actions are checked for existence upon training a new agent, and a warning is thrown before training if one is missing
 

- [\[3.16.13\] - 2026-06-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31613---2026-06-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes)
- [\[3.16.12\] - 2026-06-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31612---2026-06-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-1)
- [\[3.16.11\] - 2026-06-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31611---2026-06-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-2)
- [\[3.16.10\] - 2026-05-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31610---2026-05-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-3)
- [\[3.16.9\] - 2026-05-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3169---2026-05-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-4)
- [\[3.16.8\] - 2026-05-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3168---2026-05-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-5)
- [\[3.16.7\] - 2026-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3167---2026-05-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-6)
- [\[3.16.6\] - 2026-05-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3166---2026-05-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-7)
- [\[3.16.5\] - 2026-04-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3165---2026-04-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-8)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes)
- [\[3.16.4\] - 2026-04-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3164---2026-04-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-9)
- [\[3.16.3\] - 2026-04-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3163---2026-04-09)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-10)
- [\[3.16.2\] - 2026-04-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3162---2026-04-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-11)
- [\[3.16.1\] - 2026-04-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3161---2026-04-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-12)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-1)
- [\[3.16.0\] - 2026-03-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3160---2026-03-26)
 - [Breaking changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#breaking-changes)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-1)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-13)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-2)
- [\[3.15.27\] - 2026-06-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31527---2026-06-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-14)
- [\[3.15.26\] - 2026-06-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31526---2026-06-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-15)
- [\[3.15.25\] - 2026-06-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31525---2026-06-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-16)
- [\[3.15.24\] - 2026-05-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31524---2026-05-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-17)
- [\[3.15.23\] - 2026-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31523---2026-05-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-18)
- [\[3.15.22\] - 2026-05-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31522---2026-05-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-19)
- [\[3.15.21\] - 2026-04-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31521---2026-04-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-20)
- [\[3.15.20\] - 2026-04-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31520---2026-04-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-21)
- [\[3.15.19\] - 2026-04-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31519---2026-04-02)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-2)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-22)
- [\[3.15.18\] - 2026-03-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31518---2026-03-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-23)
- [\[3.15.17\] - 2026-03-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31517---2026-03-11)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-1)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-24)
- [\[3.15.16\] - 2026-03-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31516---2026-03-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-25)
- [\[3.15.15\] - 2026-03-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31515---2026-03-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-26)
- [\[3.15.14\] - 2026-02-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31514---2026-02-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-27)
- [\[3.15.13\] - 2026-02-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31513---2026-02-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-28)
- [\[3.15.12\] - 2026-02-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31512---2026-02-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-29)
- [\[3.15.11\] - 2026-02-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31511---2026-02-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-30)
- [\[3.15.10\] - 2026-02-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31510---2026-02-10)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-3)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-31)
- [\[3.15.9\] - 2026-02-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3159---2026-02-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-32)
- [\[3.15.8\] - 2026-01-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3158---2026-01-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-33)
- [\[3.15.7\] - 2026-01-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3157---2026-01-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-34)
- [\[3.15.6\] - 2026-01-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3156---2026-01-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-35)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-3)
- [\[3.15.5\] - 2025-12-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3155---2025-12-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-36)
- [\[3.15.4\] - 2025-12-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3154---2025-12-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-37)
- [\[3.15.3\] - 2025-12-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3153---2025-12-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-38)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-4)
- [\[3.15.2\] - 2025-12-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3152---2025-12-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-39)
- [\[3.15.1\] - 2025-12-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3151---2025-12-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-40)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-5)
- [\[3.15.0\] - 2025-11-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3150---2025-11-26)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-1)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-4)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-41)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-6)
- [\[3.14.25\] - 2026-06-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31425---2026-06-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-42)
- [\[3.14.24\] - 2026-06-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31424---2026-06-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-43)
- [\[3.14.23\] - 2026-05-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31423---2026-05-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-44)
- [\[3.14.22\] - 2026-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31422---2026-05-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-45)
- [\[3.14.21\] - 2026-04-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31421---2026-04-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-46)
- [\[3.14.20\] - 2026-04-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31420---2026-04-02)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-5)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-47)
- [\[3.14.19\] - 2026-03-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31419---2026-03-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-48)
- [\[3.14.18\] - 2026-03-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31418---2026-03-11)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-2)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-49)
- [\[3.14.17\] - 2026-03-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31417---2026-03-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-50)
- [\[3.14.16\] - 2026-02-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31416---2026-02-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-51)
- [\[3.14.15\] - 2026-02-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31415---2026-02-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-52)
- [\[3.14.14\] - 2026-02-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31414---2026-02-12)
- [\[3.14.13\] - 2026-02-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31413---2026-02-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-53)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-7)
- [\[3.14.12\] - 2026-01-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31412---2026-01-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-54)
- [\[3.14.11\] - 2026-01-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31411---2026-01-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-55)
- [\[3.14.10\] - 2026-01-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31410---2026-01-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-56)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-8)
- [\[3.14.9\] - 2025-12-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3149---2025-12-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-57)
- [\[3.14.8\] - 2025-12-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3148---2025-12-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-58)
- [\[3.14.7\] - 2025-12-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3147---2025-12-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-59)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-9)
- [\[3.14.6\] - 2025-12-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3146---2025-12-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-60)
- [\[3.14.5\] - 2025-12-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3145---2025-12-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-61)
- [\[3.14.4\] - 2025-11-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3144---2025-11-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-62)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-10)
- [\[3.14.3\] - 2025-11-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3143---2025-11-13)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-11)
- [\[3.14.2\] - 2025-10-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3142---2025-10-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-6)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-63)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-12)
- [\[3.14.1\] - 2025-10-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3141---2025-10-10)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-13)
- [\[3.14.0\] - 2025-10-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3140---2025-10-09)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-2)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-7)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-64)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-14)
- [\[3.13.24\] - 2026-02-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31324---2026-02-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-65)
- [\[3.13.23\] - 2026-02-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31323---2026-02-12)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-8)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-66)
- [\[3.13.22\] - 2026-01-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31322---2026-01-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-67)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-15)
- [\[3.13.21\] - 2025-12-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31321---2025-12-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-68)
- [\[3.13.20\] - 2025-12-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31320---2025-12-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-69)
- [\[3.13.19\] - 2025-12-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31319---2025-12-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-70)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-16)
- [\[3.13.18\] - 2025-12-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31318---2025-12-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-71)
- [\[3.13.17\] - 2025-11-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31317---2025-11-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-72)
- [\[3.13.16\] - 2025-11-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31316---2025-11-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-73)
- [\[3.13.15\] - 2025-11-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31315---2025-11-13)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-17)
- [\[3.13.14\] - 2025-10-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31314---2025-10-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-9)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-74)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-18)
- [\[3.13.13\] - 2025-10-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31313---2025-10-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-75)
- [\[3.13.12\] - 2025-09-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31312---2025-09-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-76)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-19)
- [\[3.13.11\] - 2025-09-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31311---2025-09-15)
- [\[3.13.10\] - 2025-09-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31310---2025-09-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-77)
- [\[3.13.9\] - 2025-09-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3139---2025-09-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-78)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-20)
- [\[3.13.8\] - 2025-08-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3138---2025-08-26)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-79)
- [\[3.13.7\] - 2025-08-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3137---2025-08-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-80)
- [\[3.13.6\] - 2025-08-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3136---2025-08-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-81)
- [\[3.13.5\] - 2025-07-31](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3135---2025-07-31)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-82)
- [\[3.13.4\] - 2025-07-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3134---2025-07-23)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-83)
- [\[3.13.3\] - 2025-07-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3133---2025-07-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-84)
- [\[3.13.2\] - 2025-07-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3132---2025-07-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-85)
- [\[3.13.1\] - 2025-07-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3131---2025-07-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-86)
- [\[3.13.0\] - 2025-07-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3130---2025-07-07)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-3)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-3)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-87)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-21)
- [\[3.12.42\] - 2025-12-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31242---2025-12-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-88)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-22)
- [\[3.12.41\] - 2025-12-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31241---2025-12-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-89)
- [\[3.12.40\] - 2025-11-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31240---2025-11-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-90)
- [\[3.12.39\] - 2025-11-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31239---2025-11-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-91)
- [\[3.12.38\] - 2025-11-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31238---2025-11-13)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-23)
- [\[3.12.37\] - 2025-10-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31237---2025-10-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-92)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-24)
- [\[3.12.36\] - 2025-10-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31236---2025-10-27)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-25)
- [\[3.12.35\] - 2025-10-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31235---2025-10-21)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-93)
- [\[3.12.34\] - 2025-10-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31234---2025-10-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-94)
- [\[3.12.33\] - 2025-09-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31233---2025-09-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-95)
- [\[3.12.32\] - 2025-09-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31232---2025-09-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-96)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-26)
- [\[3.12.31\] - 2025-08-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31231---2025-08-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-97)
- [\[3.12.30\] - 2025-08-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31230---2025-08-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-98)
- [\[3.12.29\] - 2025-07-31](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31229---2025-07-31)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-99)
- [\[3.12.28\] - 2025-07-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31228---2025-07-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-100)
- [\[3.12.27\] - 2025-07-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31227---2025-07-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-101)
- [\[3.12.26\] - 2025-07-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31226---2025-07-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-102)
- [\[3.12.25\] - 2025-07-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31225---2025-07-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-103)
- [\[3.12.24\] - 2025-07-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31224---2025-07-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-104)
- [\[3.12.23\] - 2025-07-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31223---2025-07-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-105)
- [\[3.12.22\] - 2025-07-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31222---2025-07-07)
- [\[3.12.21\] - 2025-07-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31221---2025-07-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-106)
- [\[3.12.20\] - 2025-06-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31220---2025-06-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-107)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-27)
- [\[3.12.19\] - 2025-06-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31219---2025-06-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-108)
- [\[3.12.18\] - 2025-06-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31218---2025-06-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-109)
- [\[3.12.17\] - 2025-06-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31217---2025-06-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-110)
- [\[3.12.16\] - 2025-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31216---2025-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-111)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-28)
- [\[3.12.15\] - 2025-06-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31215---2025-06-02)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-112)
- [\[3.12.14\] - 2025-05-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31214---2025-05-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-113)
- [\[3.12.13\] - 2025-05-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31213---2025-05-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-114)
- [\[3.12.12\] - 2025-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31212---2025-05-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-115)
- [\[3.12.11\] - 2025-05-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31211---2025-05-14)
- [\[3.12.10\] - 2025-05-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31210---2025-05-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-116)
- [\[3.12.9\] - 2025-05-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3129---2025-05-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-117)
- [\[3.12.8\] - 2025-04-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3128---2025-04-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-118)
- [\[3.12.7\] - 2025-04-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3127---2025-04-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-119)
- [\[3.12.6\] - 2025-04-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3126---2025-04-15)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-4)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-4)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-120)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-29)
- [\[3.12.5\] - 2025-04-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3125---2025-04-07)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-121)
- [\[3.12.4\] - 2025-04-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3124---2025-04-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-122)
- [\[3.12.3\] - 2025-03-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3123---2025-03-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-123)
- [\[3.12.2\] - 2025-03-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3122---2025-03-25)
- [\[3.12.1\] - 2025-03-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3121---2025-03-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-124)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-30)
- [\[3.12.0\] - 2025-03-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3120---2025-03-19)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-5)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-5)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-125)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-31)
- [\[3.11.19\] - 2025-08-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31119---2025-08-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-126)
- [\[3.11.18\] - 2025-07-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31118---2025-07-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-127)
- [\[3.11.17\] - 2025-07-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31117---2025-07-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-128)
- [\[3.11.16\] - 2025-06-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31116---2025-06-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-129)
- [\[3.11.15\] - 2025-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31115---2025-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-130)
- [\[3.11.14\] - 2025-06-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31114---2025-06-02)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-131)
- [\[3.11.13\] - 2025-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31113---2025-05-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-132)
- [\[3.11.12\] - 2025-05-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31112---2025-05-14)
- [\[3.11.11\] - 2025-05-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31111---2025-05-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-133)
- [\[3.11.10\] - 2025-05-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31110---2025-05-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-134)
- [\[3.11.9\] - 2025-04-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3119---2025-04-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-135)
- [\[3.11.8\] - 2025-04-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3118---2025-04-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-136)
- [\[3.11.7\] - 2025-04-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3117---2025-04-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-137)
- [\[3.11.6\] - 2025-04-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3116---2025-04-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-138)
- [\[3.11.5\] - 2025-02-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3115---2025-02-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-139)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-32)
- [\[3.11.4\] - 2025-01-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3114---2025-01-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-140)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-33)
- [\[3.11.3\] - 2025-01-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3113---2025-01-14)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-141)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-34)
- [\[3.11.2\] - 2024-12-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3112---2024-12-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-142)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-35)
- [\[3.11.1\] - 2024-12-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3111---2024-12-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-143)
- [\[3.11.0\] - 2024-12-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3110---2024-12-11)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-6)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-6)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-144)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-36)
- [\[3.10.27\] - 2025-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31027---2025-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-145)
- [\[3.10.26\] - 2025-06-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31026---2025-06-02)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-146)
- [\[3.10.25\] - 2025-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31025---2025-05-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-147)
- [\[3.10.24\] - 2025-05-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31024---2025-05-14)
- [\[3.10.23\] - 2025-05-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31023---2025-05-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-148)
- [\[3.10.22\] - 2025-04-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31022---2025-04-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-149)
- [\[3.10.21\] - 2025-04-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31021---2025-04-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-150)
- [\[3.10.20\] - 2025-04-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31020---2025-04-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-151)
- [\[3.10.19\] - 2025-04-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31019---2025-04-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-152)
- [\[3.10.18\] - 2025-04-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31018---2025-04-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-153)
- [\[3.10.17\] - 2025-01-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31017---2025-01-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-154)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-37)
- [\[3.10.16\] - 2025-01-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31016---2025-01-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-155)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-38)
- [\[3.10.15\] - 2024-12-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31015---2024-12-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-156)
- [\[3.10.14\] - 2024-12-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31014---2024-12-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-157)
- [\[3.10.13\] - 2024-11-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31013---2024-11-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-158)
- [\[3.10.12\] - 2024-11-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31012---2024-11-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-159)
- [\[3.10.11\] - 2024-11-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31011---2024-11-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-160)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-39)
- [\[3.10.10\] - 2024-11-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#31010---2024-11-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-161)
- [\[3.10.9\] - 2024-11-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3109---2024-11-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-162)
- [\[3.10.8\] - 2024-10-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3108---2024-10-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-163)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-40)
- [\[3.10.7\] - 2024-10-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3107---2024-10-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-164)
- [\[3.10.6\] - 2024-10-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3106---2024-10-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-165)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-41)
- [\[3.10.5\] - 2024-10-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3105---2024-10-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-166)
- [\[3.10.4\] - 2024-09-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3104---2024-09-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-167)
- [\[3.10.3\] - 2024-09-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3103---2024-09-20)
- [\[3.10.2\] - 2024-09-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3102---2024-09-19)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-7)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-31)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-168)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-42)
- [\[3.10.1\] - 2024-09-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3101---2024-09-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-169)
- [\[3.10.0\] - 2024-09-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3100---2024-09-04)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-8)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-7)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-32)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-170)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-43)
- [\[3.9.20\] - 2025-04-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3920---2025-04-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-171)
- [\[3.9.19\] - 2025-01-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3919---2025-01-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-33)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-172)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-44)
- [\[3.9.18\] - 2025-01-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3918---2025-01-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-173)
- [\[3.9.17\] - 2024-12-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3917---2024-12-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-174)
- [\[3.9.16\] - 2024-11-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3916---2024-11-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-175)
- [\[3.9.15\] - 2024-10-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3915---2024-10-18)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-34)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-176)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-45)
- [\[3.9.14\] - 2024-10-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3914---2024-10-02)
- [\[3.9.13\] - 2024-10-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3913---2024-10-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-177)
- [\[3.9.12\] - 2024-09-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3912---2024-09-20)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-9)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-35)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-178)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-46)
- [\[3.9.11\] - 2024-09-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3911---2024-09-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-179)
- [\[3.9.10\] - 2024-09-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3910---2024-09-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-180)
- [\[3.9.9\] - 2024-08-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#399---2024-08-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-181)
- [\[3.9.8\] - 2024-08-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#398---2024-08-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-182)
- [\[3.9.7\] - 2024-08-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#397---2024-08-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-183)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-47)
- [\[3.9.6\] - 2024-08-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#396---2024-08-07)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-48)
- [\[3.9.5\] - 2024-08-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#395---2024-08-01)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-36)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-184)
- [\[3.9.4\] - 2024-07-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#394---2024-07-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-185)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-49)
- [\[3.9.3\] - 2024-07-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#393---2024-07-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-186)
- [\[3.9.2\] - 2024-07-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#392---2024-07-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-187)
- [\[3.9.1\] - 2024-07-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#391---2024-07-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-188)
- [\[3.9.0\] - 2024-07-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#390---2024-07-03)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-8)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-37)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-189)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-50)
- [\[3.8.17\] - 2024-10-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3817---2024-10-18)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-38)
- [\[3.8.16\] - 2024-10-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3816---2024-10-02)
- [\[3.8.15\] - 2024-10-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3815---2024-10-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-190)
- [\[3.8.14\] - 2024-09-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3814---2024-09-20)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-10)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-39)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-191)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-51)
- [\[3.8.13\] - 2024-09-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3813---2024-09-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-192)
- [\[3.8.12\] - 2024-08-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3812---2024-08-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-193)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-52)
- [\[3.8.11\] - 2024-07-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3811---2024-07-04)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-40)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-194)
- [\[3.8.10\] - 2024-06-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3810---2024-06-19)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-41)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-195)
- [\[3.8.9\] - 2024-06-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#389---2024-06-14)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-42)
- [\[3.8.8\] - 2024-06-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#388---2024-06-07)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-196)
- [\[3.8.7\] - 2024-05-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#387---2024-05-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-197)
- [\[3.8.6\] - 2024-05-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#386---2024-05-27)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-43)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-198)
- [\[3.8.5\] - 2024-05-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#385---2024-05-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-199)
- [\[3.8.4\] - 2024-04-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#384---2024-04-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-44)
- [\[3.8.3\] - 2024-04-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#383---2024-04-26)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-45)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-200)
- [\[3.8.2\] - 2024-04-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#382---2024-04-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-201)
- [\[3.8.1\] - 2024-04-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#381---2024-04-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-46)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-202)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-53)
- [\[3.8.0\] - 2024-04-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#380---2024-04-03)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-9)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-47)
- [Example configuration](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#example-configuration)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-203)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-54)
- [\[3.7.9\] - 2024-03-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#379---2024-03-26)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-48)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-204)
- [\[3.7.8\] - 2024-02-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#378---2024-02-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-49)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-205)
- [\[3.7.7\] - 2024-02-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#377---2024-02-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-206)
- [\[3.7.6\] - 2024-02-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#376---2024-02-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-207)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-55)
- [\[3.7.5\] - 2024-01-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#375---2024-01-24)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-50)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-208)
- [\[3.7.4\] - 2024-01-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#374---2024-01-03)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-51)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-209)
- [\[3.7.3\] - 2023-12-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#373---2023-12-21)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-52)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-210)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-56)
- [\[3.7.2\] - 2023-12-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#372---2023-12-07)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-211)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-57)
- [\[3.7.1\] - 2023-12-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#371---2023-12-01)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-53)
- [\[3.7.0\] - 2023-11-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#370---2023-11-22)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-10)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-54)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-212)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-58)
- [\[3.6.13\] - 2023-10-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3613---2023-10-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-213)
- [\[3.6.12\] - 2023-10-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3612---2023-10-10)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-55)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-214)
- [\[3.6.11\] - 2023-10-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3611---2023-10-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-215)
- [\[3.6.10\] - 2023-09-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3610---2023-09-26)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-56)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-1)
- [\[3.6.9\] - 2023-09-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#369---2023-09-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-57)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-216)
- [\[3.6.8\] - 2023-08-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#368---2023-08-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-217)
- [\[3.6.7\] - 2023-08-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#367---2023-08-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-218)
- [\[3.6.6\] - 2023-08-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#366---2023-08-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-219)
- [\[3.6.5\] - 2023-08-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#365---2023-08-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-58)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-220)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-2)
- [\[3.6.4\] - 2023-07-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#364---2023-07-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-221)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-3)
- [\[3.6.3\] - 2023-07-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#363---2023-07-20)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-59)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-222)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-4)
- [\[3.6.2\] - 2023-07-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#362---2023-07-06)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-60)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-223)
- [\[3.6.1\] - 2023-07-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#361---2023-07-03)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-61)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-224)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-5)
- [\[3.6.0\] - 2023-06-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#360---2023-06-14)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-11)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-11)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-62)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-225)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-6)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-59)
- [\[3.5.12\] - 2023-06-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3512---2023-06-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-226)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-60)
- [\[3.5.11\] - 2023-06-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3511---2023-06-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-227)
- [\[3.5.10\] - 2023-05-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3510---2023-05-23)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-7)
- [\[3.5.9\] - 2023-05-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#359---2023-05-19)
- [\[3.5.8\] - 2023-05-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#358---2023-05-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-228)
- [\[3.5.7\] - 2023-05-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#357---2023-05-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-229)
- [\[3.5.6\] - 2023-04-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#356---2023-04-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-230)
- [\[3.5.5\] - 2023-04-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#355---2023-04-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-231)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-8)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-61)
- [\[3.5.4\] - 2023-04-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#354---2023-04-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-232)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-62)
- [\[3.5.3\] - 2023-03-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#353---2023-03-30)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-9)
- [\[3.5.2\] - 2023-03-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#352---2023-03-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-63)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-233)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-10)
- [\[3.5.1\] - 2023-03-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#351---2023-03-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-234)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-11)
- [\[3.5.0\] - 2023-03-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#350---2023-03-21)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-12)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-64)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-235)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-12)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-63)
- [\[3.4.14\] - 2023-06-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3414---2023-06-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-236)
- [\[3.4.13\] - 2023-05-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3413---2023-05-19)
- [\[3.4.12\] - 2023-05-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3412---2023-05-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-237)
- [\[3.4.11\] - 2023-05-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3411---2023-05-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-238)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-64)
- [\[3.4.10\] - 2023-04-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3410---2023-04-17)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-65)
- [\[3.4.9\] - 2023-04-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#349---2023-04-05)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-66)
- [\[3.4.8\] - 2023-04-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#348---2023-04-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-239)
- [\[3.4.7\] - 2023-03-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#347---2023-03-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-65)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-240)
- [\[3.4.6\] - 2023-03-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#346---2023-03-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-241)
- [\[3.4.5\] - 2023-03-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#345---2023-03-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-242)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-13)
- [\[3.4.4\] - 2023-02-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#344---2023-02-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-66)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-243)
- [\[3.4.3\] - 2023-02-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#343---2023-02-14)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-67)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-244)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-14)
- [\[3.4.2\] - 2023-01-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#342---2023-01-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-245)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-67)
- [\[3.4.1\] - 2023-01-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#341---2023-01-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-246)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-15)
- [\[3.4.0\] - 2022-12-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#340---2022-12-14)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-13)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-68)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-247)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-16)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-68)
- [\[3.3.8\] - 2023-04-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#338---2023-04-06)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-69)
- [\[3.3.7\] - 2023-03-31](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#337---2023-03-31)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-69)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-248)
- [\[3.3.6\] - 2023-03-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#336---2023-03-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-249)
- [\[3.3.5\] - 2023-02-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#335---2023-02-21)
- [\[3.3.4\] - 2023-02-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#334---2023-02-14)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-70)
- [\[3.3.3\] - 2022-12-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#333---2022-12-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-250)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-71)
- [\[3.3.2\] - 2022-11-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#332---2022-11-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-72)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-251)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-17)
- [\[3.3.1\] - 2022-11-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#331---2022-11-09)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-18)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-73)
- [\[3.3.0\] - 2022-10-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#330---2022-10-24)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-14)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-74)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-252)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-12)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-70)
- [\[3.2.13\] - 2023-03-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3213---2023-03-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-253)
- [\[3.2.12\] - 2023-02-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3212---2023-02-21)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-75)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-254)
- [\[3.2.11\] - 2022-12-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3211---2022-12-05)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-76)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-255)
- [\[3.2.10\] - 2022-09-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3210---2022-09-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-256)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-19)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-71)
- [\[3.2.9\] - 2022-09-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#329---2022-09-09)
- [\[3.2.8\] - 2022-09-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#328---2022-09-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-257)
- [\[3.2.7\] - 2022-08-31](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#327---2022-08-31)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-77)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-258)
- [\[3.2.6\] - 2022-08-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#326---2022-08-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-259)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-72)
- [\[3.2.5\] - 2022-08-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#325---2022-08-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-260)
- [\[3.2.4\] - 2022-07-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#324---2022-07-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-261)
- [\[3.2.3\] - 2022-07-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#323---2022-07-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-262)
- [\[3.2.2\] - 2022-07-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#322---2022-07-05)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-20)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-73)
- [\[3.2.1\] - 2022-06-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#321---2022-06-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-263)
- [\[3.2.0\] - 2022-06-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#320---2022-06-14)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-13)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-78)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-264)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-21)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-74)
- [\[3.1.7\] - 2022-08-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#317---2022-08-30)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-75)
- [\[3.1.6\] - 2022-07-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#316---2022-07-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-265)
- [\[3.1.5\] - 2022-07-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#315---2022-07-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-266)
- [\[3.1.4\] - 2022-06-21 No significant changes.](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#314---2022-06-21--------------------------no-significant-changes)
- [\[3.1.3\] - 2022-06-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#313---2022-06-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-267)
- [\[3.1.2\] - 2022-06-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#312---2022-06-08)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-76)
- [\[3.1.1\] - 2022-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#311---2022-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-268)
- [\[3.1.0\] - 2022-03-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#310---2022-03-25)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-16)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-79)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-269)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-22)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-77)
- [\[3.0.10\] - 2022-03-15## \[3.0.10\] - 2022-03-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#3010---2022-03-15-3010---2022-03-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-270)
- [\[3.0.9\] - 2022-03-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#309---2022-03-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-271)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-23)
- [\[3.0.8\] - 2022-02-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#308---2022-02-11)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-80)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-272)
- [\[3.0.7\] - 2022-02-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#307---2022-02-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-273)
- [\[3.0.6\] - 2022-01-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#306---2022-01-28)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-274)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-24)
- [\[3.0.5\] - 2022-01-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#305---2022-01-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-275)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-78)
- [\[3.0.4\] - 2021-12-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#304---2021-12-22)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-79)
- [\[3.0.3\] - 2021-12-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#303---2021-12-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-276)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-80)
- [\[3.0.2\] - 2021-12-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#302---2021-12-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-277)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-25)
- [\[3.0.1\] - 2021-12-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#301---2021-12-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-278)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-81)
- [\[3.0.0\] - 2021-11-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#300---2021-11-23)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-15)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-81)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-279)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-26)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-82)
- [\[2.8.16\] - 2021-12-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2816---2021-12-09)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-82)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-280)
- [\[2.8.15\] - 2021-11-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2815---2021-11-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-281)
- [\[2.8.14\] - 2021-11-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2814---2021-11-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-282)
- [\[2.8.13\] - 2021-11-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2813---2021-11-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-283)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-83)
- [\[2.8.12\] - 2021-10-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2812---2021-10-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-284)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-27)
- [\[2.8.11\] - 2021-10-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2811---2021-10-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-285)
- [\[2.8.10\] - 2021-10-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2810---2021-10-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-286)
- [\[2.8.9\] - 2021-10-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#289---2021-10-08)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-83)
- [\[2.8.8\] - 2021-10-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#288---2021-10-06)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-84)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-28)
- [\[2.8.7\] - 2021-09-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#287---2021-09-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-287)
- [\[2.8.6\] - 2021-09-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#286---2021-09-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-288)
- [\[2.8.5\] - 2021-09-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#285---2021-09-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-289)
- [\[2.8.4\] - 2021-09-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#284---2021-09-02)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-85)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-290)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-84)
- [\[2.8.3\] - 2021-08-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#283---2021-08-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-291)
- [\[2.8.2\] - 2021-08-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#282---2021-08-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-292)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-29)
- [\[2.8.1\] - 2021-07-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#281---2021-07-22)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-86)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-293)
- [\[2.8.0\] - 2021-07-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#280---2021-07-12)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-16)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-18)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-87)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-294)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-85)
- [\[2.7.2\] - 2021-08-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#272---2021-08-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-295)
- [\[2.7.1\] - 2021-06-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#271---2021-06-16)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-296)
- [\[2.7.0\] - 2021-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#270---2021-06-03)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-88)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-297)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-86)
- [\[2.6.3\] - 2021-05-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#263---2021-05-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-298)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-87)
- [\[2.6.2\] - 2021-05-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#262---2021-05-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-299)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-30)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-88)
- [\[2.6.1\] - 2021-05-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#261---2021-05-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-300)
- [\[2.6.0\] - 2021-05-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#260---2021-05-06)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-17)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-19)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-89)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-301)
- [\[2.5.2\] - 2021-06-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#252---2021-06-16)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-20)
- [\[2.5.1\] - 2021-04-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#251---2021-04-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-302)
- [\[2.5.0\] - 2021-04-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#250---2021-04-12)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-18)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-21)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-90)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-303)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-89)
- [\[2.4.3\] - 2021-03-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#243---2021-03-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-304)
- [\[2.4.2\] - 2021-03-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#242---2021-03-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-305)
- [\[2.4.1\] - 2021-03-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#241---2021-03-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-306)
- [\[2.4.0\] - 2021-03-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#240---2021-03-11)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-19)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-91)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-307)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-31)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-90)
- [\[2.3.5\] - 2021-06-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#235---2021-06-16)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-22)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-92)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-308)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-32)
- [\[2.3.4\] - 2021-02-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#234---2021-02-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-309)
- [\[2.3.3\] - 2021-02-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#233---2021-02-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-310)
- [\[2.3.2\] - 2021-02-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#232---2021-02-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-311)
- [\[2.3.1\] - 2021-02-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#231---2021-02-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-312)
- [\[2.3.0\] - 2021-02-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#230---2021-02-11)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-93)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-313)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-91)
- [\[2.2.10\] - 2021-02-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#2210---2021-02-08)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-94)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-314)
- [\[2.2.9\] - 2021-02-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#229---2021-02-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-315)
- [\[2.2.8\] - 2021-01-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#228---2021-01-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-316)
- [\[2.2.7\] - 2021-01-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#227---2021-01-25)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-95)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-317)
- [\[2.2.6\] - 2021-01-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#226---2021-01-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-318)
- [\[2.2.5\] - 2021-01-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#225---2021-01-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-319)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-92)
- [\[2.2.4\] - 2021-01-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#224---2021-01-08)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-96)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-320)
- [\[2.2.3\] - 2021-01-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#223---2021-01-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-321)
- [\[2.2.2\] - 2020-12-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#222---2020-12-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-322)
- [\[2.2.1\] - 2020-12-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#221---2020-12-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-323)
- [\[2.2.0\] - 2020-12-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#220---2020-12-16)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-20)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-23)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-97)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-324)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-33)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-93)
- [\[2.1.3\] - 2020-12-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#213---2020-12-04)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-98)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-325)
- [\[2.1.2\] - 2020-11-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#212---2020-11-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-326)
- [\[2.1.1\] - 2020-11-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#211---2020-11-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-327)
- [\[2.1.0\] - 2020-11-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#210---2020-11-17)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-21)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-24)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-99)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-328)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-34)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-94)
- [\[2.0.8\] - 2020-11-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#208---2020-11-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-329)
- [\[2.0.7\] - 2020-11-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#207---2020-11-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-330)
- [\[2.0.6\] - 2020-11-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#206---2020-11-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-331)
- [\[2.0.5\] - 2020-11-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#205---2020-11-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-332)
- [\[2.0.4\] - 2020-11-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#204---2020-11-08)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-333)
- [\[2.0.3\] - 2020-10-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#203---2020-10-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-334)
- [\[2.0.2\] - 2020-10-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#202---2020-10-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-335)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-95)
- [\[2.0.1\] - 2020-10-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#201---2020-10-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-336)
- [\[2.0.0\] - 2020-10-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#200---2020-10-07)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-22)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-25)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-100)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-337)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-35)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-96)
- [\[1.10.26\] - 2021-06-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11026---2021-06-17)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-26)
- [\[1.10.25\] - 2021-04-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11025---2021-04-14)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-27)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-101)
- [\[1.10.24\] - 2021-03-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11024---2021-03-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-338)
- [\[1.10.23\] - 2021-02-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11023---2021-02-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-339)
- [\[1.10.22\] - 2021-02-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11022---2021-02-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-340)
- [\[1.10.21\] - 2021-02-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11021---2021-02-01)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-102)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-341)
- [\[1.10.20\] - 2020-12-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11020---2020-12-18)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-342)
- [\[1.10.19\] - 2020-12-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11019---2020-12-17)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-103)
- [\[1.10.18\] - 2020-11-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11018---2020-11-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-343)
- [\[1.10.17\] - 2020-11-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11017---2020-11-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-344)
- [\[1.10.16\] - 2020-10-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11016---2020-10-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-345)
- [\[1.10.15\] - 2020-10-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11015---2020-10-09)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-346)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-104)
- [\[1.10.14\] - 2020-09-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11014---2020-09-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-347)
- [\[1.10.13\] - 2020-09-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11013---2020-09-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-348)
- [\[1.10.12\] - 2020-09-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11012---2020-09-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-349)
- [\[1.10.11\] - 2020-08-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11011---2020-08-21)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-105)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-350)
- [\[1.10.10\] - 2020-08-04](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#11010---2020-08-04)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-351)
- [\[1.10.9\] - 2020-07-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1109---2020-07-29)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-106)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-352)
- [\[1.10.8\] - 2020-07-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1108---2020-07-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-353)
- [\[1.10.7\] - 2020-07-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1107---2020-07-07)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-354)
- [\[1.10.6\] - 2020-07-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1106---2020-07-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-355)
- [\[1.10.5\] - 2020-07-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1105---2020-07-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-356)
- [\[1.10.4\] - 2020-07-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1104---2020-07-01)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-357)
- [\[1.10.3\] - 2020-06-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1103---2020-06-12)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-107)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-358)
- [\[1.10.2\] - 2020-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1102---2020-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-359)
- [\[1.10.1\] - 2020-05-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1101---2020-05-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-108)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-360)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-97)
- [\[1.10.0\] - 2020-04-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1100---2020-04-28)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-29)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-109)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-361)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-98)
- [\[1.9.7\] - 2020-04-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#197---2020-04-23)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-110)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-362)
- [\[1.9.6\] - 2020-04-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#196---2020-04-15)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-363)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-99)
- [\[1.9.5\] - 2020-04-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#195---2020-04-01)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-111)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-364)
- [\[1.9.4\] - 2020-03-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#194---2020-03-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-365)
- [\[1.9.3\] - 2020-03-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#193---2020-03-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-366)
- [\[1.9.2\] - 2020-03-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#192---2020-03-26)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-36)
- [\[1.9.1\] - 2020-03-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#191---2020-03-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-367)
- [\[1.9.0\] - 2020-03-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#190---2020-03-24)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-30)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-112)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-368)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-37)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-100)
- [\[1.8.3\] - 2020-03-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#183---2020-03-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-369)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-38)
- [\[1.8.2\] - 2020-03-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#182---2020-03-19)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-370)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-39)
- [\[1.8.1\] - 2020-03-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#181---2020-03-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-371)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-101)
- [\[1.8.0\] - 2020-02-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#180---2020-02-26)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-23)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-31)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-113)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-372)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-40)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-102)
- [\[1.7.4\] - 2020-02-24](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#174---2020-02-24)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-373)
- [\[1.7.3\] - 2020-02-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#173---2020-02-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-374)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-41)
- [\[1.7.2\] - 2020-02-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#172---2020-02-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-375)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-42)
- [\[1.7.1\] - 2020-02-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#171---2020-02-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-376)
- [\[1.7.0\] - 2020-01-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#170---2020-01-29)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-24)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-32)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-114)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-377)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#miscellaneous-internal-changes-103)
- [\[1.6.2\] - 2020-01-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#162---2020-01-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-115)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-378)
- [\[1.6.1\] - 2020-01-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#161---2020-01-07)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-379)
- [\[1.6.0\] - 2019-12-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#160---2019-12-18)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-25)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-33)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-116)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-380)
- [\[1.5.3\] - 2019-12-11](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#153---2019-12-11)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-117)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-381)
- [\[1.5.2\] - 2019-12-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#152---2019-12-09)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-118)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-382)
 - [Improved Documentation](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improved-documentation-43)
- [\[1.5.1\] - 2019-11-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#151---2019-11-27)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-119)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-383)
- [\[1.5.0\] - 2019-11-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#150---2019-11-26)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-34)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-120)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-26)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-384)
- [\[1.4.6\] - 2019-11-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#146---2019-11-22)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-385)
- [\[1.4.5\] - 2019-11-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#145---2019-11-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-386)
- [\[1.4.4\] - 2019-11-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#144---2019-11-13)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-35)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-121)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-387)
- [\[1.4.3\] - 2019-10-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#143---2019-10-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-388)
- [\[1.4.2\] - 2019-10-28](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#142---2019-10-28)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-389)
- [\[1.4.1\] - 2019-10-22](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#141---2019-10-22)
- [\[1.4.0\] - 2019-10-19](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#140---2019-10-19)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-36)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-122)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-28)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-390)
- [\[1.3.10\] - 2019-10-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1310---2019-10-18)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-37)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-391)
- [\[1.3.9\] - 2019-10-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#139---2019-10-10)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-38)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-392)
- [\[1.3.8\] - 2019-10-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#138---2019-10-08)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-123)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-393)
- [\[1.3.7\] - 2019-09-27](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#137---2019-09-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-394)
- [\[1.3.6\] - 2019-09-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#136---2019-09-21)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-39)
- [\[1.3.5\] - 2019-09-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#135---2019-09-20)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-395)
- [\[1.3.4\] - 2019-09-20](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#134---2019-09-20)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-40)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-396)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-124)
- [\[1.3.3\] - 2019-09-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#133---2019-09-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-397)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-29)
- [\[1.3.2\] - 2019-09-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#132---2019-09-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-398)
- [\[1.3.1\] - 2019-09-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#131---2019-09-09)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-125)
- [\[1.3.0\] - 2019-09-05](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#130---2019-09-05)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-41)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-126)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-399)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-30)
- [\[1.2.12\] - 2019-10-16](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1212---2019-10-16)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-42)
- [\[1.2.11\] - 2019-10-09](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1211---2019-10-09)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-43)
- [\[1.2.10\] - 2019-10-08](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#1210---2019-10-08)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-44)
- [\[1.2.9\] - 2019-09-17](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#129---2019-09-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-400)
- [\[1.2.8\] - 2019-09-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#128---2019-09-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-401)
- [\[1.2.7\] - 2019-09-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#127---2019-09-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-402)
- [\[1.2.6\] - 2019-09-02](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#126---2019-09-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-403)
- [\[1.2.5\] - 2019-08-26](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#125---2019-08-26)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-45)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-404)
- [\[1.2.4\] - 2019-08-23](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#124---2019-08-23)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-405)
- [\[1.2.3\] - 2019-08-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#123---2019-08-15)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-127)
- [\[1.2.3\] - 2019-08-15](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#123---2019-08-15-1)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-128)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-406)
- [\[1.2.2\] - 2019-08-07](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#122---2019-08-07)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-407)
- [\[1.2.1\] - 2019-08-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#121---2019-08-06)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-46)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-408)
- [\[1.2.0\] - 2019-08-01](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#120---2019-08-01)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-47)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-129)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-409)
- [\[1.1.8\] - 2019-07-25](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#118---2019-07-25)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-48)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-130)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-410)
- [\[1.1.7\] - 2019-07-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#117---2019-07-18)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-49)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-411)
- [\[1.1.6\] - 2019-07-12](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#116---2019-07-12)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-50)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-131)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-412)
- [\[1.1.5\] - 2019-07-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#115---2019-07-10)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-51)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-132)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-31)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-413)
- [\[1.1.4\] - 2019-06-18](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#114---2019-06-18)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-52)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-133)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-414)
- [\[1.1.3\] - 2019-06-14](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#113---2019-06-14)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-415)
- [\[1.1.2\] - 2019-06-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#112---2019-06-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-416)
- [\[1.1.1\] - 2019-06-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#111---2019-06-13)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-417)
- [\[1.1.0\] - 2019-06-13](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#110---2019-06-13)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-53)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-134)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-418)
- [\[1.0.9\] - 2019-06-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#109---2019-06-10)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-135)
- [\[1.0.8\] - 2019-06-10](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#108---2019-06-10)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-54)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-136)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-419)
- [\[1.0.7\] - 2019-06-06](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#107---2019-06-06)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-55)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-420)
- [\[1.0.6\] - 2019-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#106---2019-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-421)
- [\[1.0.5\] - 2019-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#105---2019-06-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-422)
- [\[1.0.4\] - 2019-06-03](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#104---2019-06-03)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-56)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-137)
- [\[1.0.3\] - 2019-05-30](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#103---2019-05-30)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-423)
- [\[1.0.2\] - 2019-05-29](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#102---2019-05-29)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-57)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-424)
- [\[1.0.1\] - 2019-05-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#101---2019-05-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-425)
- [\[1.0.0\] - 2019-05-21](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#100---2019-05-21)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#features-58)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#improvements-138)
 - [Deprecations and Removals](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#deprecations-and-removals-32)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/#bugfixes-426)