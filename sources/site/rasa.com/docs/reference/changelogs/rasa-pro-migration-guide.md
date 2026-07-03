# Source: https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide

On this page

This page contains information about changes between major versions and how you can migrate from one version to another.

## Rasa Pro 3.15 to Rasa Pro 3.16[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-315-to-rasa-pro-316 'Direct link to Rasa Pro 3.15 to Rasa Pro 3.16')

### Default Models for LLM-based Components[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#default-models-for-llm-based-components 'Direct link to Default Models for LLM-based Components')

**Migration Impact**: If you rely on previous implicit defaults for built-in LLM-based components, pin model names explicitly in `endpoints.yml` to preserve prior behavior.

The default model mapping changed in `3.16.0`:

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

### Changes to Default Command-Generator Prompt Templates[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-command-generator-prompt-templates 'Direct link to Changes to Default Command-Generator Prompt Templates')

**Migration Impact**: If you use custom `prompt_template` files, compare them with the updated shipped templates and include the relevant changes.

The shipped defaults for command generators changed:

- `CompactLLMCommandGenerator` defaults now use:
 - `command_prompt_v2_gpt_5_1_2025_11_13_template.jinja2`
 - `agent_command_prompt_v2_gpt_5_1_2025_11_13_template.jinja2`
- `SearchReadyLLMCommandGenerator` defaults now use:
 - `command_prompt_v3_gpt_5_1_2025_11_13_template.jinja2`
 - `agent_command_prompt_v3_gpt_5_1_2025_11_13_template.jinja2`

See the full shipped templates in the reference docs:

- [CompactLLMCommandGenerator prompt templates](https://rasa.com/docs/reference/config/components/llm-command-generators/#compactllmcommandgenerator)
- [SearchReadyLLMCommandGenerator prompt templates](https://rasa.com/docs/reference/config/components/llm-command-generators/#searchreadyllmcommandgenerator)

If you already provide a custom `prompt_template`, that custom file is still used.

### Changes to Default ReAct Sub-agent Prompt Templates[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-react-sub-agent-prompt-templates 'Direct link to Changes to Default ReAct Sub-agent Prompt Templates')

**Migration Impact**: If you use `configuration.prompt_template` for ReAct sub agents or custom completion behavior, align your customizations with the updated defaults.

The shipped ReAct defaults changed for:

- Task-specific prompt template: `mcp_task_agent_prompt_template.jinja2`
- General-purpose prompt template: `mcp_open_agent_prompt_template.jinja2`
- Built-in `task_completed` completion guidance/tool instructions

See the reference sections:

- [Default ReAct prompt templates](https://rasa.com/docs/reference/config/agents/react-sub-agents/#default-prompt-templates)
- [`task_completed` tool behavior](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-task_completed-tool)

### Changes to Agent Protocol and Custom Tool Interface[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-agent-protocol-and-custom-tool-interface 'Direct link to Changes to Agent Protocol and Custom Tool Interface')

**Migration Impact**: If you maintain custom A2A/ReAct agent implementations or custom MCP tools, update method signatures and hook usage as described below.

#### `process_output` removal and replacement hooks[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#process_output-removal-and-replacement-hooks 'Direct link to process_output-removal-and-replacement-hooks')

`process_output` was removed from the shared agent protocol. Use protocol-specific hooks:

```
# A2A agentsasync def process_agent_output(self, output: AgentOutput) -> AgentOutput# ReAct / MCP agentsasync def process_tool_output(    self,    current_iteration_tool_results: Dict[str, AgentToolResult],    cumulative_tool_results: Dict[str, AgentToolResult],    output_channel: Optional[OutputChannel] = None,) -> List[Event]
```

#### `run` signature changed[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#run-signature-changed 'Direct link to run-signature-changed')

`run` now supports cooperative cancellation via an optional `cancellation_token`:

```
async def run(    self,    input: "AgentInput",    output_channel: Optional[OutputChannel] = None,    cancellation_token: Optional["CancellationToken"] = None,) -> "AgentOutput"
```

#### Custom tool callable signature changed[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#custom-tool-callable-signature-changed 'Direct link to Custom tool callable signature changed')

- Before: `Callable[[Dict[str, Any]], AgentToolResult]`
- After: `Callable[[Dict[str, Any], AgentToolContext], AgentToolResult]`

Import `AgentToolContext` with:

```
from rasa.agents.schemas import AgentToolContext
```

### Support for `user_id` in `DialogueStateTracker`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#support-for-user_id-in-dialoguestatetracker 'Direct link to support-for-user_id-in-dialoguestatetracker')

As part of the changes to support session management and user tracking across multiple conversation sessions, the `DialogueStateTracker` now includes an optional `user_id` field. This field is automatically handled by Rasa's built-in tracker stores, but **custom tracker store implementations must be updated** to support it. If you have a custom tracker store implementation, you need to:

#### 1\. Update `retrieve()` Methods[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#1-update-retrieve-methods 'Direct link to 1-update-retrieve-methods')

Ensure your `retrieve()` and `retrieve_full_tracker()` methods pass `user_id` to `DialogueStateTracker.from_dict()`:

```
async def retrieve(self, sender_id: Text) -> Optional[DialogueStateTracker]:    # ... retrieve events and user_id from storage ...        return DialogueStateTracker.from_dict(        sender_id,         events,         slots,        user_id=user_id  # Add this parameter    )
```

#### 2\. Update `save()` and `update()` Methods[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#2-update-save-and-update-methods 'Direct link to 2-update-save-and-update-methods')

If your custom tracker store uses the mixin inheritance pattern with `SerializedTrackerAsText` or `SerializedTrackerAsDict`, then no changes are needed. The mixin classes have been updated to include `user_id` in serialization automatically.

However, if your tracker store implements its own `serialise_tracker()` method without using the mixins, you must manually include `user_id`:

```
class MyCustomTrackerStore(TrackerStore):    def serialise_tracker(self, tracker: DialogueStateTracker) -> Any:        # You must manually include user_id in your serialization        return {            "sender_id": tracker.sender_id,            "user_id": tracker.user_id,  # Add this            "events": [e.as_dict() for e in tracker.events],            # ... other fields        }
```

Both `save()` and `update()` methods preserve `user_id`:

- `save()` uses `serialise_tracker()` which automatically includes `user_id` (via mixins)
- `update()` typically receives `tracker.current_state()` which includes `user_id`

#### 3\. Update Deserialization (If Custom)[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#3-update-deserialization-if-custom 'Direct link to 3. Update Deserialization (If Custom)')

If you override `deserialise_tracker()`, ensure you extract and pass `user_id`:

```
def deserialise_tracker(    self, sender_id: Text, serialised_tracker: Any) -> Optional[DialogueStateTracker]:    # Extract user_id from your serialized format    user_id = serialised_tracker.get("user_id")        tracker = self.init_tracker(sender_id, user_id=user_id)    # ... recreate tracker state ...    return tracker
```

#### 4\. Implement `get_serialized_trackers_by_user_id()` Method[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#4-implement-get_serialized_trackers_by_user_id-method 'Direct link to 4-implement-get_serialized_trackers_by_user_id-method')

The `GET /users/{user_id}/trackers` endpoint now calls `get_serialized_trackers_by_user_id()` to return conversation data directly from storage, without replaying events through `DialogueStateTracker.from_dict()` and `current_state()`. You must implement this method in your custom tracker store to power the endpoint efficiently.

```
async def get_serialized_trackers_by_user_id(    self,    user_id: str,    limit: Optional[int] = None,    skip: Optional[int] = None,) -> List[Dict[str, Any]]:    """Retrieve serialized trackers for the given user_id.    Args:        user_id: User ID to retrieve trackers for.        limit: Optional limit on the number of trackers to return.        skip: Optional offset for pagination.    Returns:        List of serialized tracker dicts for the given user_id, ordered by        conversation_started_timestamp and sender_id, without replaying        events through DialogueStateTracker.    """    # Fetch raw serialized tracker dicts from your storage    serialized_trackers = []    # ... your implementation to load serialized data by user_id ...    # Return directly — no DialogueStateTracker instantiation needed    return serialized_trackers
```

Each returned dict must include at minimum the `sender_id`, `events`, and any top-level metadata fields (such as `user_id` and `current_session_id`) expected by the API response schema. The `current_session_id` field should be derived from the metadata of the last event in the tracker's event list, and must be `null` when the last event is `ConversationInactive`.

### Removal of the `HumanHandoff` / `hand over` command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#removal-of-the-humanhandoff--hand-over-command 'Direct link to removal-of-the-humanhandoff--hand-over-command')

The `HumanHandoffCommand` class was removed from the codebase, and the corresponding `hand over` command was removed from all default prompt templates. If you're using a custom prompt template that includes the `hand over` command, please make sure to remove it. If you want to continue using human handoff functionality, you can implement a custom command for that. Please see customization guide for more details: [Customizing the Command Set](https://rasa.com/docs/pro/customize/command-generator/#important-note-on-customizing-the-command-set).

## Rasa Pro 3.14 to Rasa Pro 3.15[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-314-to-rasa-pro-315 'Direct link to Rasa Pro 3.14 to Rasa Pro 3.15')

### Changes to `invoke_llm` Method Signature[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-invoke_llm-method-signature 'Direct link to changes-to-invoke_llm-method-signature')

As part of the Langfuse integration for tracing LLM calls, the `invoke_llm` method signature has been updated to use `LLMInput` instead of a plain prompt string. This change enables passing metadata (such as session ID, component name, and model ID) to LLM calls for better observability.

This breaking change affects any custom components that override the `invoke_llm` method in the following components:

- `CompactLLMCommandGenerator`
- `SearchReadyLLMCommandGenerator`
- `ContextualResponseRephraser`
- `EnterpriseSearchPolicy`
- `IntentlessPolicy`
- `LLMBasedRouter`

#### Migration Guide[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-guide 'Direct link to Migration Guide')

If you have custom components that override `invoke_llm`, you need to update the method signature and how you call the LLM:

- Rasa Pro 3.14
- Rasa Pro 3.15

```
from rasa.shared.utils.llm import llm_factory, DEFAULT_LLM_CONFIGfrom rasa.shared.constants import LLM_CONFIG_KEYasync def invoke_llm(self, prompt: str) -> Optional[str]:    """Use LLM to generate a response.    Args:        prompt: The prompt to send to the LLM.    Returns:        The generated text.    """    llm = llm_factory(self.config.get(LLM_CONFIG_KEY), DEFAULT_LLM_CONFIG)    try:        llm_response = await llm.acompletion(prompt)        return llm_response.choices[0]    except Exception as e:        structlogger.error("component.llm.error", error=e)        raise ProviderClientAPIException(            message="LLM call exception", original_exception=e        )
```

```
from rasa.shared.utils.llm import llm_factory, DEFAULT_LLM_CONFIG, LLMInputfrom rasa.shared.providers.llm.llm_response import LLMResponsefrom rasa.shared.constants import LLM_CONFIG_KEYasync def invoke_llm(self, llm_input: LLMInput) -> Optional[LLMResponse]:    """Use LLM to generate a response.    Args:        llm_input: The LLM input containing the prompt and metadata.    Returns:        The LLM response object.    """    llm = llm_factory(self.config.get(LLM_CONFIG_KEY), DEFAULT_LLM_CONFIG)    try:        llm_response = await llm.acompletion(            llm_input.prompt, metadata=llm_input.metadata        )        return llm_response    except Exception as e:        structlogger.error("component.llm.error", error=e)        raise ProviderClientAPIException(            message="LLM call exception", original_exception=e        )
```

#### Creating LLMInput[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#creating-llminput 'Direct link to Creating LLMInput')

When calling `invoke_llm`, you now need to create an `LLMInput` object that includes both the prompt and metadata:

```
from rasa.shared.utils.llm import LLMInput# Create LLMInput with prompt and metadatallm_input = LLMInput(    prompt=your_prompt,    metadata=self.get_llm_tracing_metadata(tracker))# Call invoke_llm with LLMInputresponse = await self.invoke_llm(llm_input)
```

The `get_llm_tracing_metadata()` method is available in all LLM-based components and returns a dictionary with session ID, tags, and custom metadata.

### Changes to Pattern Clarification[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-pattern-clarification 'Direct link to Changes to Pattern Clarification')

We modified the `pattern_clarification` to handle empty clarification options. Here is a comparison between the old and new implementations:

- Rasa Pro 3.14
- Rasa Pro 3.15

```
pattern_clarification:    description: Conversation repair flow for handling ambiguous requests that could match multiple flows    name: pattern clarification    steps:      - action: action_clarify_flows      - action: utter_clarification_options_rasa
```

```
pattern_clarification:    description: Conversation repair flow for handling ambiguous requests that could match multiple flows    name: pattern clarification    steps:      - noop: true        next:          - if: context.names            then: action_clarify_flows          - else:            - action: utter_clarification_no_options_rasa              next: END      - id: action_clarify_flows        action: action_clarify_flows      - action: utter_clarification_options_rasa
```

### Changes to `AgentInput` and `AgentOutput`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-agentinput-and-agentoutput 'Direct link to changes-to-agentinput-and-agentoutput')

**Migration Impact**: These changes enable intermediate message support for sub agents. If you have custom agent implementations, you may need to update your code to handle the new fields.

To enable sending intermediate messages from sub agents, we updated the `AgentInput` and `AgentOutput` schemas:

**`AgentInput` Changes:**

- Added `recipient_id: Optional[str] = None` field to ensure intermediate messages are sent to the correct recipient

**`AgentOutput` Changes:**

- Changed the `events` field from `Optional[List[SlotSet]]` to `Optional[List[Event]]` to support bot uttered events for intermediate messages sent to users

### Changes to Agent Protocol[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-agent-protocol 'Direct link to Changes to Agent Protocol')

**Migration Impact**: If you have custom agent protocol implementations, you need to update the `run` method signature to accept the optional `output_channel` parameter.

We added an `output_channel` parameter to the agent protocol's `run` method signature to allow sending intermediate messages directly to users via the output channel:

```
async def run(    self, input: "AgentInput", output_channel: Optional[OutputChannel] = None) -> "AgentOutput":    """Send a message to Agent/server and return response.    Args:        input: The agent input containing user message and context        output_channel: Optional output channel for sending intermediate messages    Returns:        The agent output containing response and status    """
```

### Updated Default Prompts to Support Current Date Time in Prompts[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#updated-default-prompts-to-support-current-date-time-in-prompts 'Direct link to Updated Default Prompts to Support Current Date Time in Prompts')

The default prompt templates for all components that support including current date and time information have been updated to include a "Date & Time Context" section. By default, `include_date_time` is enabled, so prompts will automatically include date and time context unless explicitly disabled. This change affects the following components:

- `CompactLLMCommandGenerator`
- `SearchReadyLLMCommandGenerator`
- `EnterpriseSearchPolicy`
- `MCPOpenAgent` (with timezone support)
- `MCPTaskAgent` (with timezone support)

#### What Changed[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#what-changed 'Direct link to What Changed')

By default, the prompts now automatically include a "Date & Time Context" section that displays:

- Current date (formatted as "DD Month, YYYY")
- Current time (formatted as "HH:MM:SS" with timezone)
- Current day of the week

This context is added to help LLMs understand temporal references and provide time-aware responses.

#### Impact[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#impact 'Direct link to Impact')

If you have custom prompt templates that override the default templates, you may want to review them to ensure they are compatible with the new DateTime context format. The DateTime context is conditionally included using Jinja2 template syntax:

```
{% if current_datetime %}---### Date & Time Context- Current date: {{ current_datetime.strftime("%d %B, %Y") }}   (DD Month, YYYY)- Current time: {{ current_datetime.strftime("%H:%M:%S") }} ({{ current_datetime.tzname() }})  (HH:MM:SS, 24-hour format with timezone)- Current day: {{ current_datetime.strftime("%A") }}     (Day of week){% endif %}
```

#### Migration Steps[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-steps 'Direct link to Migration Steps')

1. **Review your custom prompt templates** (if any) to ensure they work with the new DateTime context format. The DateTime context is included by default, so your templates should handle the `current_datetime` variable.
2. **Test your components** to verify the prompts render correctly with the date and time context included.
3. **If you want to disable date/time context**, you can set `include_date_time: false` in your component configuration:
 
 config.yml
 
 ```
    pipeline:  - name: CompactLLMCommandGenerator    include_date_time: false
    ```
 

If you're using the default prompt templates, no action is required, the DateTime context will be automatically included by default.

## Rasa Pro 3.13 to Rasa Pro 3.14[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-313-to-rasa-pro-314 'Direct link to Rasa Pro 3.13 to Rasa Pro 3.14')

### Dependencies[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#dependencies 'Direct link to Dependencies')

info

To avoid any conflicts we strongly recommend using a fresh environment when installing Rasa >=3.14.0.

The default `pip` package for `rasa-pro` now supports Python versions `3.12` and `3.13`, and drops support for Python version `3.9`.

The package will exclude the following dependency categories:

1. **`nlu`** - All dependencies required to run NLU/coexistence bots, including: `transformers`, `tensorflow` (and related packages: `tensorflow-text`, `tensorflow-hub`, `tensorflow-gcs-filesystem`, `tensorflow-metal`, `tf-keras`), `spacy`, `sentencepiece`, `skops`, `mitie`, `jieba`, `sklearn-crfsuite`.
 
2. **`channels`** - All dependencies required to connect to channel connectors, including: `fbmessenger`, `twilio`, `webexteamssdk`, `mattermostwrapper`, `rocketchat_API`, `aiogram`, `slack-sdk`, `cvg-python-sdk`. _Note_: The following channels are **NOT** included in the channels extra: `browser_audio`, `studio_chat`, `socketIO`, and `rest` (used by inspector for text and voice).
 

Optional dependency categories are available to install the relevant packages. Use `pip install rasa-pro[nlu]` if you have an agent with NLU components. Similarly use `pip install rasa-pro[channels]` if you have an agent making use of channel connectors.

Docker images will continue to have the same dependencies as previously, except for the additional packages `mcp` and `a2a-sdk` which now form part of the core dependencies.

**Important**: `tensorflow` and its related dependencies are only supported for `"python_version < '3.12'"`, so components requiring TensorFlow are not available for Python ≥ 3.12. These components are: `DIETClassifier`, `TEDPolicy`, `UnexpecTEDIntentPolicy`, `ResponseSelector`, `ConveRTFeaturizer`, and `LanguageModelFeaturizer`.

### Pattern Continue Interrupted[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#pattern-continue-interrupted 'Direct link to Pattern Continue Interrupted')

We modified the `pattern_continue_interrupted` to ask for confirmation before returning to an interrupted flow. Here is an example conversation comparing the old and new implementations:

- Old
- New

```
user: I want to transfer moneybot: how much do you want to transfer?user: wait what's my balance?bot: you have 4200 in your accountbot: how much do you want to transfer? # immediately returns to the interrupted flow
```

```
user: I want to transfer moneybot: how much do you want to transfer?user: wait what's my balance?bot: you have 4200 in your accountbot: Would you like to go back to transferring money? # asks for confirmation before continuing
```

**Rationale for this change:**

1. **Improved UX**: Immediately returning to interrupted flows often creates an unnatural conversational experience.
2. **Error Correction**: Sometimes the command generator incorrectly identifies a cancellation + start flow as a digression. This change allows users to guide the assistant in correcting these mistakes.
3. **Agent Integration**: Sub agents sometimes don't have a reliable way to signal completion. When these agents are wrapped in flows and a digression occurs, we need user input to determine if the agent's task is complete.

Here is the comparison between the old and the new `pattern_continue_interrupted`.

- Old
- New

```
pattern_continue_interrupted:  description: Conversation repair flow for managing when users switch between different flows  name: pattern continue interrupted  steps:    - action: utter_flow_continue_interrupted
```

```
pattern_continue_interrupted:  description: Conversation repair flow for managing when users switch between different flows  name: pattern continue interrupted  steps:    - noop: true      next:        - if: context.multiple_flows_interrupted          then: collect_interrupted_flow_to_continue        - else: collect_continue_interrupted_flow_confirmation    - id: collect_interrupted_flow_to_continue      collect: interrupted_flow_to_continue      description: "Fill this slot with the name of the flow the user wants to continue. If the user does not want to continue any of the interrupted flows, fill this slot with 'none'."      next:        - if: slots.interrupted_flow_to_continue is not "none"          then:            - action: action_continue_interrupted_flow              next: END        - else:            - action: action_cancel_interrupted_flows              next: END    - id: collect_continue_interrupted_flow_confirmation      collect: continue_interrupted_flow_confirmation      description: "If the user wants to continue the interrupted flow, fill this slot with true. If the user does not want to continue the interrupted flow, fill this slot with false."      next:        - if: slots.continue_interrupted_flow_confirmation          then:            - action: action_continue_interrupted_flow              next: END        - else:            - action: action_cancel_interrupted_flows              next: END
```

### Command Generator[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#command-generator 'Direct link to Command Generator')

#### Prompt Template[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#prompt-template 'Direct link to Prompt Template')

**No Action Required**: If you don't use sub agents, your prompt templates remain unchanged.

**Migration Required**: When sub agents are used, Rasa automatically switches to new default prompts that include agent support. If you have customized prompt templates for the `CompactLLMCommandGenerator` or `SearchReadyLLMCommandGenerator`, you must update your custom prompts to include the new agent-related commands and functionality.

**Prompt Templates of the `CompactLLMCommandGenerator`**

- GPT-4o with Agent Support
- Claude 3.5 Sonnet with Agent Support

The prompt template for the `gpt-4o-2024-11-20` model with agent support is as follows:

```
## Task DescriptionYour task is to analyze the current conversation context and generate a list of actions to start new business processes that we call flows, to extract slots, or respond to small talk and knowledge requests.---## Available Flows and SlotsUse the following structured data:```json{"flows":[{% for flow in available_flows %}{"name":"{{ flow.name }}","description":{{ flow.description | to_json_escaped_string }}{% if flow.agent_info %},"sub-agents":[{% for agent in flow.agent_info %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}{% if flow.slots %},"slots":[{% for slot in flow.slots %}{"name":"{{ slot.name }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":{{ slot.allowed_values }}{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}```---## Available Actions:* `start flow flow_name`: Starting a flow. For example, `start flow transfer_money` or `start flow list_contacts`.* `set slot slot_name slot_value`: Slot setting. For example, `set slot transfer_money_recipient Freddy`. Can be used to correct and change previously set values. ONLY use slots that are explicitly defined in the flow's slot list.* `cancel flow`: Cancelling the current flow.* `disambiguate flows flow_name1 flow_name2 ... flow_name_n`: Disambiguate which flow should be started when user input is ambiguous by listing the potential flows as options. For example, `disambiguate flows list_contacts add_contact remove_contact ...` if the user just wrote "contacts".* `provide info`: Responding to the user's questions by supplying relevant information, such as answering FAQs or explaining services.* `offtopic reply`: Responding to casual or social user messages that are unrelated to any flows, engaging in friendly conversation and addressing off-topic remarks.* `hand over`: Handing over to a human, in case the user seems frustrated or explicitly asks to speak to one.* `repeat message`: Repeating the last bot message.{% if active_agent %}* `continue agent`: Continue the currently active agent {{ active_agent.name }}. This has HIGHEST PRIORITY when an agent is active and the user is responding to agent questions.{% endif %}{% if completed_agents %}* `restart agent agent_name`: Restart the agent with the given name, in case the user wants to change some answer to a previous question asked by the agent. For example, `restart agent car_research_agent` if the user changed his mind about the car he wants to buy. ONLY use agents that are listed in the `completed_agents` section.{% endif %}---## General Tips* Do not fill slots with abstract values or placeholders.* For categorical slots try to match the user message with allowed slot values. Use "other" if you cannot match it.* Set the boolean slots based on the user response. Map positive responses to `True`, and negative to `False`.* Extract text slot values exactly as provided by the user. Avoid assumptions, format changes, or partial extractions.* ONLY use `set slot` with slots that are explicitly defined in the current flow's slot list. Do NOT create or assume slots that don't exist.* Only use information provided by the user.* Use clarification in ambiguous cases.* Multiple flows can be started. If a user wants to digress into a second flow, you do not need to cancel the current flow.* Do not cancel the flow unless the user explicitly requests it.* Strictly adhere to the provided action format.* ONLY use the exact actions listed above. Do NOT invent new actions like "respond <message>" or any other variations.* Focus on the last message and take it one step at a time.* Use the previous conversation steps only to aid understanding.{% if active_agent %}* When an agent is active, ALWAYS prioritize `continue agent` over `provide info` or `offtopic reply` unless the user is clearly asking something unrelated to the agent's task.{% endif %}{% if completed_agents %}* ONLY use `restart agent` with agents that are listed in the `completed_agents` section. Do NOT restart non-existent agents.{% endif %}{% if active_agent or completed_agents %}* If you're unsure about agent names, refer to the structured data provided in the `Current State` section.{% endif %}---## Current State{% if current_flow != None %}Use the following structured data:```json{"active_flow":{"name":"{{ current_flow }}","current_step":{"requested_slot":"{{ current_slot }}","requested_slot_description":{{ current_slot_description | to_json_escaped_string }}},"slots":[{% for slot in flow_slots %}{"name":"{{ slot.name }}","value":"{{ slot.value }}","type":"{{ slot.type }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":"{{ slot.allowed_values }}"{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}{% if active_agent %},"active_agent":{"name":"{{ active_agent.name }}","description":{{ active_agent.description | to_json_escaped_string }}}{% endif %}{% if completed_agents %},"completed_agents":[{% for agent in completed_agents %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}```{% else %}You are currently not inside any flow.{% endif %}---## Conversation History{{ current_conversation }}---## TaskCreate an action list with one action per line in response to the user's last message: """{{ user_message }}""".Your action list:
```

The prompt template for the `claude-3-5-sonnet-20240620` / `anthropic.claude-3-5-sonnet-20240620-v1:0` model with agent support is as follows:

```
## Task DescriptionYour task is to analyze the current conversation context and generate a list of actions to start new business processes that we call flows, to extract slots, or respond to small talk and knowledge requests.--## Available Actions:* `start flow flow_name`: Starting a flow. For example, `start flow transfer_money` or `start flow list_contacts`.* `set slot slot_name slot_value`: Slot setting. For example, `set slot transfer_money_recipient Freddy`. Can be used to correct and change previously set values. ONLY use slots that are explicitly defined in the flow's slot list.* `cancel flow`: Cancelling the current flow.* `disambiguate flows flow_name1 flow_name2 ... flow_name_n`: Disambiguate which flow should be started when user input is ambiguous by listing the potential flows as options. For example, `disambiguate flows list_contacts add_contact remove_contact ...` if the user just wrote "contacts".* `provide info`: Responding to the user's questions by supplying relevant information, such as answering FAQs or explaining services.* `offtopic reply`: Responding to casual or social user messages that are unrelated to any flows, engaging in friendly conversation and addressing off-topic remarks.* `hand over`: Handing over to a human, in case the user seems frustrated or explicitly asks to speak to one.* `repeat message`: Repeating the last bot message.{% if active_agent %}* `continue agent`: Continue the currently active agent {{ active_agent.name }}. This has HIGHEST PRIORITY when an agent is active and the user is responding to agent questions.{% endif %}{% if completed_agents %}* `restart agent agent_name`: Restart the agent with the given name, in case the user wants to change some answer to a previous question asked by the agent. For example, `restart agent car_research_agent` if the user changed his mind about the car he wants to buy. ONLY use agents that are listed in the `completed_agents` section.{% endif %}--## General Tips* Do not fill slots with abstract values or placeholders.* For categorical slots try to match the user message with allowed slot values. Use "other" if you cannot match it.* Set the boolean slots based on the user response. Map positive responses to `True`, and negative to `False`.* Always refer to the slot description to determine what information should be extracted and how it should be formatted.* For text slots, extract values exactly as provided by the user unless the slot description specifies otherwise. Preserve formatting and avoid rewording, truncation, or making assumptions.* ONLY use `set slot` with slots that are explicitly defined in the current flow's slot list. Do NOT create or assume slots that don't exist.* Only use information provided by the user.* Use clarification in ambiguous cases.* Multiple flows can be started. If a user wants to digress into a second flow, you do not need to cancel the current flow.* Do not cancel the flow unless the user explicitly requests it.* Strictly adhere to the provided action format.* ONLY use the exact actions listed above. Do NOT invent new actions like "respond <message>" or any other variations.* Focus on the last message and take it one step at a time.* Use the previous conversation steps only to aid understanding.{% if active_agent %}* When an agent is active, ALWAYS prioritize `continue agent` over `provide info` or `offtopic reply` unless the user is clearly asking something unrelated to the agent's task.{% endif %}{% if completed_agents %}* ONLY use `restart agent` with agents that are listed in the `completed_agents` section. Do NOT restart non-existent agents.{% endif %}{% if active_agent or completed_agents %}* If you're unsure about agent names, refer to the structured data provided in the `Current State` section.{% endif %}--## Available Flows and SlotsUse the following structured data:```json{"flows":[{% for flow in available_flows %}{"name":"{{ flow.name }}","description":{{ flow.description | to_json_escaped_string }}{% if flow.agent_info %},"sub-agents":[{% for agent in flow.agent_info %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}{% if flow.slots %},"slots":[{% for slot in flow.slots %}{"name":"{{ slot.name }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":{{ slot.allowed_values }}{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}```--## Current State{% if current_flow != None %}Use the following structured data:```json{"active_flow":{"name":"{{ current_flow }}","current_step":{"requested_slot":"{{ current_slot }}","requested_slot_description":{{ current_slot_description | to_json_escaped_string }}},"slots":[{% for slot in flow_slots %}{"name":"{{ slot.name }}","value":"{{ slot.value }}","type":"{{ slot.type }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":"{{ slot.allowed_values }}"{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}{% if active_agent %},"active_agent":{"name":"{{ active_agent.name }}","description":{{ active_agent.description | to_json_escaped_string }}}{% endif %}{% if completed_agents %},"completed_agents":[{% for agent in completed_agents %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}```{% else %}You are currently not inside any flow.{% endif %}---## Conversation History{{ current_conversation }}---## TaskCreate an action list with one action per line in response to the user's last message: """{{ user_message }}""".Your action list:
```

**Prompt Templates of the `SearchReadyLLMCommandGenerator`**

- GPT-4o with Agent Support
- Claude 3.5 Sonnet with Agent Support

The prompt template for the `gpt-4o-2024-11-20` model with agent support is as follows:

```
## Task DescriptionYour task is to analyze the current conversation context and generate a list of actions to start new business processes that we call flows, to extract slots, or respond to off-topic and knowledge requests.---## Available Flows and SlotsUse the following structured data:```json{"flows":[{% for flow in available_flows %}{"name":"{{ flow.name }}","description":{{ flow.description | to_json_escaped_string }}{% if flow.agent_info %},"sub-agents":[{% for agent in flow.agent_info %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}{% if flow.slots %},"slots":[{% for slot in flow.slots %}{"name":"{{ slot.name }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":{{ slot.allowed_values }}{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}```---## Available Actions:* `start flow flow_name`: Start a flow. For example, `start flow transfer_money` or `start flow list_contacts`.* `set slot slot_name slot_value`: Set a slot for the active flow. For example, `set slot transfer_money_recipient Freddy`. Can be used to correct and change previously set values. ONLY use slots that are explicitly defined in the flow's slot list.* `disambiguate flows flow_name1 flow_name2 ... flow_name_n`: When a message could refer to multiple flows, list the possible flows as options to clarify. Example: `disambiguate flows list_contacts add_contact remove_contact`.* `search and reply`: Provide a response from the knowledge base to address the user’s inquiry when no flows fit, including domain knowledge, FAQs, and all off-topic or social messages.* `cancel flow`: Cancel the current flow if the user requests it.* `repeat message`: Repeat the last bot message.{% if active_agent %}* `continue agent`: Continue the currently active agent {{ active_agent.name }}. This has HIGHEST PRIORITY when an agent is active and the user is responding to agent questions.{% endif %}{% if completed_agents %}* `restart agent agent_name`: Restart the agent with the given name, in case the user wants to change some answer to a previous question asked by the agent. For example, `restart agent car_research_agent` if the user changed his mind about the car he wants to buy. ONLY use agents that are listed in the `completed_agents` section.{% endif %}---## General Instructions### Start Flow* Only start a flow if the user's message is clear and fully addressed by that flow's description and purpose.* Pay close attention to exact wording and scope in the flow description — do not assume or “stretch” the intended use of a flow.### Set Slot* Do not fill slots with abstract values or placeholders.* For categorical slots try to match the user message with allowed slot values. Use "other" if you cannot match it.* Set the boolean slots based on the user response. Map positive responses to `True`, and negative to `False`.* Extract text slot values exactly as provided by the user. Avoid assumptions, format changes, or partial extractions.* ONLY use `set slot` with slots that are explicitly defined in the current flow's slot list. Do NOT create or assume slots that don't exist.### Disambiguate Flows* Use `disambiguate flows` when the user's message matches multiple flows and you cannot decide which flow is most appropriate.* If the user message is short and not precise enough to start a flow or `search and reply`, disambiguate.* If a single flow is a strong/plausible fit, prefer starting that flow directly.* If a user's message unambiguously and distinctly matches multiple flows, start all relevant flows at once (rather than disambiguating).### Search and Reply* Only start `search and reply` if the user intent is clear.* Flow Priority: If you are unsure between starting a flow or `search and reply`, always prioritize starting a flow.### Cancel Flow* Do not cancel any flow unless the user explicitly requests it.* Multiple flows can be started without cancelling the previous, if the user wants to pursue multiple processes.{% if active_agent or completed_agents %}### Agents{% if active_agent %}* When an agent is active, ALWAYS prioritize `continue agent` over `search and reply` unless the user is clearly asking something unrelated to the agent's task.{% endif %}{% if completed_agents %}* ONLY use `restart agent` with agents that are listed in the `completed_agents` section. Do NOT restart non-existent agents.{% endif %}* If you're unsure about agent names, refer to the structured data provided in the `Current State` section.{% endif %}### General Tips* Only use information provided by the user.* Strictly adhere to the provided action format.* ONLY use the exact actions listed above. Do NOT invent new actions like "respond <message>" or any other variations.* Focus on the last message and take it one step at a time.* Use the previous conversation steps only to aid understanding.---## Decision Rule Table| Condition                                                     | Action             ||---------------------------------------------------------------|--------------------|{% if active_agent %}| Agent is active and the user is responding to agent questions | continue agent     |{% endif %}| Flow perfectly matches user's message                         | start flow         || Multiple flows are equally strong, relevant matches           | disambiguate flows || User's message is unclear or imprecise                        | disambiguate flows || No flow fits at all, but knowledge base may help              | search and reply   |---## Current State{% if current_flow != None %}Use the following structured data:```json{"active_flow":{"name":"{{ current_flow }}","current_step":{"requested_slot":"{{ current_slot }}","requested_slot_description":{{ current_slot_description | to_json_escaped_string }}},"slots":[{% for slot in flow_slots %}{"name":"{{ slot.name }}","value":"{{ slot.value }}","type":"{{ slot.type }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":"{{ slot.allowed_values }}"{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}{% if active_agent %},"active_agent":{"name":"{{ active_agent.name }}","description":{{ active_agent.description | to_json_escaped_string }}}{% endif %}{% if completed_agents %},"completed_agents":[{% for agent in completed_agents %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}```{% else %}You are currently not inside any flow.{% endif %}---## Conversation History{{ current_conversation }}---## TaskCreate an action list with one action per line in response to the user's last message: """{{ user_message }}""".Your action list:
```

The prompt template for the `claude-3-5-sonnet-20240620` / `anthropic.claude-3-5-sonnet-20240620-v1:0` model with agent support is as follows:

```
## Task DescriptionYour task is to analyze the current conversation context and generate a list of actions to start new business processes that we call flows, to extract slots, or respond to off-topic and knowledge requests.---## Available Actions:* `start flow flow_name`: Start a flow. For example, `start flow transfer_money` or `start flow list_contacts`.* `set slot slot_name slot_value`: Set a slot for the active flow. For example, `set slot transfer_money_recipient Freddy`. Can be used to correct and change previously set values. ONLY use slots that are explicitly defined in the flow's slot list.* `disambiguate flows flow_name1 flow_name2 ... flow_name_n`: When a message could refer to multiple flows, list the possible flows as options to clarify. Example: `disambiguate flows list_contacts add_contact remove_contact`.* `search and reply`: Provide a response from the knowledge base to address the user's inquiry when no flows fit, including domain knowledge, FAQs, and all off-topic or social messages.* `cancel flow`: Cancel the current flow if the user requests it.* `repeat message`: Repeat the last bot message.{% if active_agent %}* `continue agent`: Continue the currently active agent {{ active_agent.name }}. This has HIGHEST PRIORITY when an agent is active and the user is responding to agent questions.{% endif %}{% if completed_agents %}* `restart agent agent_name`: Restart the agent with the given name, in case the user wants to change some answer to a previous question asked by the agent. For example, `restart agent car_research_agent` if the user changed his mind about the car he wants to buy. ONLY use agents that are listed in the `completed_agents` section.{% endif %}---## General Instructions### Start Flow* Only start a flow if the user's message is clear and fully addressed by that flow's description and purpose.* Pay close attention to exact wording and scope in the flow description — do not assume or “stretch” the intended use of a flow.### Set Slot* Do not fill slots with abstract values or placeholders.* For categorical slots, try to match the user message with allowed slot values. Use "other" if you cannot match it.* Set the boolean slots based on the user's response. Map positive responses to `True`, and negative to `False`.* Extract text slot values exactly as provided by the user. Avoid assumptions, format changes, or partial extractions.* ONLY use `set slot` with slots that are explicitly defined in the current flow's slot list. Do NOT create or assume slots that don't exist.### Disambiguate Flows* Use `disambiguate flows` when the user's message matches multiple flows and you cannot decide which flow is most appropriate.* If the user message is short and not precise enough to start a flow or `search and reply`, disambiguate.* If a single flow is a strong/plausible fit, prefer starting that flow directly.* If a user's message unambiguously and distinctly matches multiple flows, start all relevant flows at once (rather than disambiguating).### Search and Reply* Only start `search and reply` if the user intent is clear.* Flow Priority: If you are unsure between starting a flow or `search and reply`, always prioritize starting a flow.### Cancel Flow* Do not cancel any flow unless the user explicitly requests it.* Multiple flows can be started without cancelling the previous, if the user wants to pursue multiple processes.{% if active_agent or completed_agents %}### Agents{% if active_agent %}* When an agent is active, ALWAYS prioritize `continue agent` over `search and reply` unless the user is clearly asking something unrelated to the agent's task.{% endif %}{% if completed_agents %}* ONLY use `restart agent` with agents that are listed in the `completed_agents` section. Do NOT restart non-existent agents.{% endif %}* If you're unsure about agent names, refer to the structured data provided in the `Current State` section.{% endif %}### General Tips* Only use information provided by the user.* Strictly adhere to the provided action format.* ONLY use the exact actions listed above. Do NOT invent new actions like "respond <message>" or any other variations.* Focus on the last message and take it one step at a time.* Use the previous conversation steps only to aid understanding.---## Decision Rule Table| Condition                                                     | Action             ||---------------------------------------------------------------|--------------------|{% if active_agent %}| Agent is active and the user is responding to agent questions | continue agent     |{% endif %}| Flow perfectly matches user's message                         | start flow         || Multiple flows are equally strong, relevant matches           | disambiguate flows || User's message is unclear or imprecise                        | disambiguate flows || No flow fits at all, but knowledge base may help              | search and reply   |---## Available Flows and SlotsUse the following structured data:```json{"flows":[{% for flow in available_flows %}{"name":"{{ flow.name }}","description":{{ flow.description | to_json_escaped_string }}{% if flow.agent_info %},"sub-agents":[{% for agent in flow.agent_info %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}{% if flow.slots %},"slots":[{% for slot in flow.slots %}{"name":"{{ slot.name }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":{{ slot.allowed_values }}{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}```---## Current State{% if current_flow != None %}Use the following structured data:```json{"active_flow":{"name":"{{ current_flow }}","current_step":{"requested_slot":"{{ current_slot }}","requested_slot_description":{{ current_slot_description | to_json_escaped_string }}},"slots":[{% for slot in flow_slots %}{"name":"{{ slot.name }}","value":"{{ slot.value }}","type":"{{ slot.type }}"{% if slot.description %},"description":{{ slot.description | to_json_escaped_string }}{% endif %}{% if slot.allowed_values %},"allowed_values":"{{ slot.allowed_values }}"{% endif %}}{% if not loop.last %},{% endif %}{% endfor %}]}{% if active_agent %},"active_agent":{"name":"{{ active_agent.name }}","description":{{ active_agent.description | to_json_escaped_string }}}{% endif %}{% if completed_agents %},"completed_agents":[{% for agent in completed_agents %}{"name":"{{ agent.name }}","description":{{ agent.description | to_json_escaped_string }}}{% if not loop.last %},{% endif %}{% endfor %}]{% endif %}}```{% else %}You are currently not inside any flow.{% endif %}---## Conversation History{{ current_conversation }}---## TaskCreate an action list with one action per line in response to the user's last message: """{{ user_message }}""".Your action list:
```

#### Template Rendering[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#template-rendering 'Direct link to Template Rendering')

We updated the `render_template` method to include sub agent information.

If you customized the rendering method, add the new sub agent information to your implementation. The key changes to the function are highlighted below.

```
def render_template(    self,    message: Message,    tracker: DialogueStateTracker,    startable_flows: FlowsList,    all_flows: FlowsList,) -> str:    """Render the jinja template to create the prompt for the LLM.    Args:        message: The current message from the user.        tracker: The tracker containing the current state of the conversation.        startable_flows: The flows startable at this point in time by the user.        all_flows: all flows present in the assistant    Returns:        The rendered prompt template.    """    # need to make this distinction here because current step of the    # top_calling_frame would be the call step, but we need the collect step from    # the called frame. If no call is active calling and called frame are the same.    top_calling_frame = top_flow_frame(tracker.stack)    top_called_frame = top_flow_frame(tracker.stack, ignore_call_frames=False)    top_flow = top_calling_frame.flow(all_flows) if top_calling_frame else None    current_step = top_called_frame.step(all_flows) if top_called_frame else None    flow_slots = self.prepare_current_flow_slots_for_template(        top_flow, current_step, tracker    )    current_slot, current_slot_description = self.prepare_current_slot_for_template(        current_step    )    current_slot_type = None    current_slot_allowed_values = None    if current_slot:        current_slot_type = (            slot.type_name            if (slot := tracker.slots.get(current_slot)) is not None            else None        )        current_slot_allowed_values = allowed_values_for_slot(            tracker.slots.get(current_slot)        )    has_agents = Configuration.get_instance().available_agents.has_agents()    current_conversation = tracker_as_readable_transcript(        tracker, highlight_agent_turns=has_agents    )    latest_user_message = sanitize_message_for_prompt(message.get(TEXT))    current_conversation += f"\nUSER: {latest_user_message}"    inputs: Dict[str, Any] = {        "available_flows": self.prepare_flows_for_template(            startable_flows,            tracker,            add_agent_info=has_agents,        ),        "current_conversation": current_conversation,        "flow_slots": flow_slots,        "current_flow": top_flow.id if top_flow is not None else None,        "current_slot": current_slot,        "current_slot_description": current_slot_description,        "current_slot_type": current_slot_type,        "current_slot_allowed_values": current_slot_allowed_values,        "user_message": latest_user_message,    }    if has_agents:        inputs["active_agent"] = (            get_active_agent_info(tracker, top_flow.id) if top_flow else None        )        inputs["completed_agents"] = get_completed_agents_info(tracker)    return self.compile_template(self.prompt_template).render(**inputs)
```

### LLM Clients[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llm-clients 'Direct link to LLM Clients')

We updated the signature of the `completion` and `acompletion` functions in our `LLMClient` protocol to support LLM calls with tools:

```
def completion(    self, messages: Union[List[dict], List[str], str], **kwargs: Any) -> LLMResponse:
```

```
async def acompletion(    self, messages: Union[List[dict], List[str], str], **kwargs: Any) -> LLMResponse:
```

The `**kwargs` are passed through to the underlying `LiteLLM` completion functions.

**Migration Required**: If you have modified any existing LLM clients or implemented custom clients, update your `completion` and `acompletion` methods to match the new signature.

Here are a code snippets of the updated implementations in `_BaseLiteLLMClient`:

- Synchronous
- Asynchronous

```
def completion(    self, messages: Union[List[dict], List[str], str], **kwargs: Any) -> LLMResponse:    """Synchronously generate completions for given list of messages.    Args:        messages: The message can be,            - a list of preformatted messages. Each message should be a dictionary                with the following keys:                - content: The message content.                - role: The role of the message (e.g. user or system).            - a list of messages. Each message is a string and will be formatted                as a user message.            - a single message as a string which will be formatted as user message.        **kwargs: Additional parameters to pass to the completion call.    Returns:        List of message completions.    Raises:        ProviderClientAPIException: If the API request fails.    """    ...    response = litellm.completion(        messages=formatted_messages, **{**arguments, **kwargs}    )    ...
```

```
async def acompletion(    self, messages: Union[List[dict], List[str], str], **kwargs: Any) -> LLMResponse:    """Asynchronously generate completions for given list of messages.    Args:        messages: The message can be,            - a list of preformatted messages. Each message should be a dictionary                with the following keys:                - content: The message content.                - role: The role of the message (e.g. user or system).            - a list of messages. Each message is a string and will be formatted                as a user message.            - a single message as a string which will be formatted as user message.        **kwargs: Additional parameters to pass to the completion call.    Returns:        List of message completions.    Raises:        ProviderClientAPIException: If the API request fails.    """    ...    response = await litellm.acompletion(        messages=formatted_messages, **{**arguments, **kwargs}    )    ...
```

#### Add `tool_calls` to `LLMResponse`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#add-tool_calls-to-llmresponse 'Direct link to add-tool_calls-to-llmresponse')

We added a `tool_calls` field to the `LLMResponse` class to capture tool calls from LLM responses. The `_format_response` function was updated to extract tool calls from the LiteLLM response.

**Migration Impact**: If you have custom code that processes `LLMResponse` objects, you may need to handle the new `tool_calls` field.

```
@dataclassclass LLMResponse:    id: str    """A unique identifier for the completion."""    choices: List[str]    """The list of completion choices the model generated for the input prompt."""    created: int    """The Unix timestamp (in seconds) of when the completion was created."""    model: Optional[str] = None    """The model used for completion."""    usage: Optional[LLMUsage] = None    """An optional details about the token usage for the API call."""    additional_info: Optional[Dict] = None    """Optional dictionary for storing additional information related to the    completion that may not be covered by other fields."""    latency: Optional[float] = None    """Optional field to store the latency of the LLM API call."""    tool_calls: Optional[List[LLMToolCall]] = None    """The list of tool calls the model generated for the input prompt."""
```

```
class LLMToolCall(BaseModel):    """A class representing a response from an LLM tool call."""    id: str    """The ID of the tool call."""    tool_name: str    """The name of the tool that was called."""    tool_args: Dict[str, Any]    """The arguments passed to the tool call."""    type: str = "function"    """The type of the tool call."""
```

### Command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#command 'Direct link to Command')

**Migration Impact**: The changes mentioned below improve conversation flow handling and agent state management. No action required unless you have custom command implementations.

#### StartFlow Command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#startflow-command 'Direct link to StartFlow Command')

We updated the `run_command_on_tracker` method to handle `StartFlow` commands when users are in `pattern_continue_interrupted` state, ensuring smooth conversation flow by cleaning up the pattern. We also added proper agent state management to handle agent interruptions.

_Flow Resumption_: Previously, `StartFlow` commands for flows already on the stack (but not active) were ignored. This behavior was updated to resume the flow instead.

```
    def run_command_on_tracker(        self,        tracker: DialogueStateTracker,        all_flows: FlowsList,        original_tracker: DialogueStateTracker,    ) -> List[Event]:        """Runs the command on the tracker.        Args:            tracker: The tracker to run the command on.            all_flows: All flows in the assistant.            original_tracker: The tracker before any command was executed.        Returns:            The events to apply to the tracker.        """        stack = tracker.stack        original_stack = original_tracker.stack        applied_events: List[Event] = []        if self.flow not in all_flows.flow_ids:            structlogger.debug(                "start_flow_command.skip_command.start_invalid_flow_id", command=self            )            return []        original_user_frame = top_user_flow_frame(original_stack)        original_top_flow = (            original_user_frame.flow(all_flows) if original_user_frame else None        )        # if the original top flow is the same as the flow to start, the flow is        # already active, do nothing        if original_top_flow is not None and original_top_flow.id == self.flow:            # in case continue_interrupted is not active, skip the already active start            # flow command            if not is_continue_interrupted_flow_active(stack):                return []            # if the continue interrupted flow is active, and the command generator            # predicted a start flow command for the flow which is on top of the stack,            # we just need to remove the pattern_continue_interrupted frame(s) from the            # stack            stack, flow_completed_events = remove_pattern_continue_interrupted_frames(                stack            )            applied_events.extend(flow_completed_events)            return applied_events + tracker.create_stack_updated_events(stack)        # if the flow is already on the stack, resume it        if (            self.flow in user_flows_on_the_stack(stack)            and original_user_frame is not None        ):            # if pattern_continue_interrupted is active, we need to remove it            # from the stack before resuming the flow            stack, flow_completed_events = remove_pattern_continue_interrupted_frames(                stack            )            applied_events.extend(flow_completed_events)            applied_events.extend(resume_flow(self.flow, tracker, stack))            # the current active flow is interrupted            applied_events.append(                FlowInterrupted(                    original_user_frame.flow_id, original_user_frame.step_id                )            )            return applied_events        frame_type = FlowStackFrameType.REGULAR        # remove the pattern_continue_interrupted frames from the stack        # if it is currently active but the user digressed from the pattern        stack, flow_completed_events = remove_pattern_continue_interrupted_frames(stack)        applied_events.extend(flow_completed_events)        if original_top_flow:            # if the original top flow is not the same as the flow to start,            # interrupt the current active flow            frame_type = FlowStackFrameType.INTERRUPT            if original_user_frame is not None:                applied_events.append(                    FlowInterrupted(                        original_user_frame.flow_id, original_user_frame.step_id                    )                )            # If there is an active agent frame, interrupt it            active_agent_stack_frame = stack.find_active_agent_frame()            if active_agent_stack_frame:                structlogger.debug(                    "start_flow_command.interrupt_agent",                    command=self,                    agent_id=active_agent_stack_frame.agent_id,                    frame_id=active_agent_stack_frame.frame_id,                    flow_id=active_agent_stack_frame.flow_id,                )                active_agent_stack_frame.state = AgentState.INTERRUPTED                applied_events.append(                    AgentInterrupted(                        active_agent_stack_frame.agent_id,                        active_agent_stack_frame.flow_id,                    )                )        structlogger.debug("start_flow_command.start_flow", command=self)        stack.push(UserFlowStackFrame(flow_id=self.flow, frame_type=frame_type))        return applied_events + tracker.create_stack_updated_events(stack)
```

#### Cancel Command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#cancel-command 'Direct link to Cancel Command')

We updated the `run_command_on_tracker` method to properly handle agent cancellation when flows are canceled, ensuring active agent stack frames are removed and agents are properly canceled.

```
def run_command_on_tracker(    self,    tracker: DialogueStateTracker,    all_flows: FlowsList,    original_tracker: DialogueStateTracker,) -> List[Event]:    """Runs the command on the tracker.    Args:        tracker: The tracker to run the command on.        all_flows: All flows in the assistant.        original_tracker: The tracker before any command was executed.    Returns:        The events to apply to the tracker.    """    stack = tracker.stack    original_stack = original_tracker.stack    applied_events: List[Event] = []    user_frame = top_user_flow_frame(        original_stack, ignore_call_and_link_frames=False    )    current_flow = user_frame.flow(all_flows) if user_frame else None    if not current_flow:        structlogger.debug(            "cancel_command.skip_cancel_flow.no_active_flow", command=self        )        return []    if agent_frame := original_tracker.stack.find_active_agent_stack_frame_for_flow(        current_flow.id    ):        structlogger.debug(            "cancel_command.remove_agent_stack_frame",            command=self,            frame=agent_frame,        )        remove_agent_stack_frame(stack, agent_frame.agent_id)        applied_events.append(            AgentCancelled(agent_id=agent_frame.agent_id, flow_id=current_flow.id)        )    # we pass in the original dialogue stack (before any of the currently    # predicted commands were applied) to make sure we don't cancel any    # frames that were added by the currently predicted commands.    canceled_frames = self.select_canceled_frames(original_stack)    stack.push(        CancelPatternFlowStackFrame(            canceled_name=current_flow.readable_name(                language=tracker.current_language            ),            canceled_frames=canceled_frames,        )    )    if user_frame:        applied_events.append(FlowCancelled(user_frame.flow_id, user_frame.step_id))    return applied_events + tracker.create_stack_updated_events(stack)
```

#### Clarify Command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#clarify-command 'Direct link to Clarify Command')

We updated the `run_command_on_tracker` method to properly signal agent interruption when clarification is needed.

```
def run_command_on_tracker(    self,    tracker: DialogueStateTracker,    all_flows: FlowsList,    original_tracker: DialogueStateTracker,) -> List[Event]:    """Runs the command on the tracker.    Args:        tracker: The tracker to run the command on.        all_flows: All flows in the assistant.        original_tracker: The tracker before any command was executed.    Returns:        The events to apply to the tracker.    """    flows = [all_flows.flow_by_id(opt) for opt in self.options]    clean_options = [flow.id for flow in flows if flow is not None]    if len(clean_options) != len(self.options):        structlogger.debug(            "clarify_command.altered_command.dropped_clarification_options",            command=self,            original_options=self.options,            cleaned_options=clean_options,        )    if len(clean_options) == 0:        structlogger.debug(            "clarify_command.skip_command.empty_clarification", command=self        )        return []    stack = tracker.stack    relevant_flows = [all_flows.flow_by_id(opt) for opt in clean_options]    names = [        flow.readable_name(language=tracker.current_language)        for flow in relevant_flows        if flow is not None    ]    applied_events: List[Event] = []    # if the top stack frame is an agent stack frame, we need to    # update the state to INTERRUPTED and add an AgentInterrupted event    if top_stack_frame := stack.top():        if isinstance(top_stack_frame, AgentStackFrame):            applied_events.append(                AgentInterrupted(                    top_stack_frame.agent_id,                    top_stack_frame.flow_id,                )            )            top_stack_frame.state = AgentState.INTERRUPTED    stack.push(ClarifyPatternFlowStackFrame(names=names))    return applied_events + tracker.create_stack_updated_events(stack)
```

#### ChitChat Command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#chitchat-command 'Direct link to ChitChat Command')

We updated the `run_command_on_tracker` method to properly signal agent interruption when handling chitchat.

```
def run_command_on_tracker(    self,    tracker: DialogueStateTracker,    all_flows: FlowsList,    original_tracker: DialogueStateTracker,) -> List[Event]:    """Runs the command on the tracker.    Args:        tracker: The tracker to run the command on.        all_flows: All flows in the assistant.        original_tracker: The tracker before any command was executed.    Returns:        The events to apply to the tracker.    """    stack = tracker.stack    applied_events: List[Event] = []    # if the top stack frame is an agent stack frame, we need to    # update the state to INTERRUPTED and add an AgentInterrupted event    if top_stack_frame := stack.top():        if isinstance(top_stack_frame, AgentStackFrame):            applied_events.append(                AgentInterrupted(                    top_stack_frame.agent_id,                    top_stack_frame.flow_id,                )            )            top_stack_frame.state = AgentState.INTERRUPTED    stack.push(ChitchatPatternFlowStackFrame())    return applied_events + tracker.create_stack_updated_events(stack)
```

#### KnowledgeAnswer Command[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#knowledgeanswer-command 'Direct link to KnowledgeAnswer Command')

We updated the `run_command_on_tracker` method to properly signal agent interruption when handling knowledge requests.

```
def run_command_on_tracker(    self,    tracker: DialogueStateTracker,    all_flows: FlowsList,    original_tracker: DialogueStateTracker,) -> List[Event]:    """Runs the command on the tracker.    Args:        tracker: The tracker to run the command on.        all_flows: All flows in the assistant.        original_tracker: The tracker before any command was executed.    Returns:        The events to apply to the tracker.    """    stack = tracker.stack    applied_events: List[Event] = []    # if the top stack frame is an agent stack frame, we need to    # update the state to INTERRUPTED and add an AgentInterrupted event    if top_stack_frame := stack.top():        if isinstance(top_stack_frame, AgentStackFrame):            applied_events.append(                AgentInterrupted(                    top_stack_frame.agent_id,                    top_stack_frame.flow_id,                )            )            top_stack_frame.state = AgentState.INTERRUPTED    stack.push(SearchPatternFlowStackFrame())    return applied_events + tracker.create_stack_updated_events(stack)
```

### Tracing for Jaeger[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#tracing-for-jaeger 'Direct link to Tracing for Jaeger')

We updated the port configuration for Jaeger tracing collection.

**Migration Required**: Update your Jaeger configuration to use the new port settings.

**Updated Docker Command**:

```
docker run --rm --name jaeger \  -p 16686:16686 \  -p 4317:4317 \  -p 4318:4318 \  -p 5778:5778 \  -p 9411:9411 \  cr.jaegertracing.io/jaegertracing/jaeger:2.10.0
```

**Updated Configuration**:

Update your `endpoints.yml` file with the new port configuration:

```
tracing:  type: jaeger  host: 0.0.0.0  port: 4317  service_name: rasa  sync_export: ~
```

The Jaeger UI is now accessible at [`http://localhost:16686/search`](http://localhost:16686/search).

## Rasa Pro 3.12 to Rasa Pro 3.13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-312-to-rasa-pro-313 'Direct link to Rasa Pro 3.12 to Rasa Pro 3.13')

### LLM Judge Model Change in E2E Testing[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llm-judge-model-change-in-e2e-testing 'Direct link to LLM Judge Model Change in E2E Testing')

Starting with Rasa Pro v3.13.x, the default model for the LLM Judge in E2E tests has changed from `gpt-4o-mini` to `gpt-4.1-mini`, see [Generative Response LLM Judge Configuration](https://rasa.com/docs/reference/testing/assertions/#generative-response-llm-judge-configuration). The new model may produce lower scores for the `generative_response_is_relevant` and `generative_response_is_grounded` assertions, which can cause previously passing responses to be incorrectly marked as failures (false negatives).

Action Required:

- Lower the thresholds for `generative_response_is_relevant` and `generative_response_is_grounded` in your E2E test configuration to reduce the risk of false negatives.
- Alternatively, if you prefer not to lower the thresholds, configure the LLM Judge to use a more performant model (note: this may increase costs). For details on configuring the LLM Judge, see the [E2E testing documentation](https://rasa.com/docs/pro/testing/evaluating-assistant/).

## Rasa Pro 3.11 to Rasa Pro 3.12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-311-to-rasa-pro-312 'Direct link to Rasa Pro 3.11 to Rasa Pro 3.12')

### Custom LLM-based Command Generators[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#custom-llm-based-command-generators 'Direct link to Custom LLM-based Command Generators')

In order to improve [slot filling in CALM](https://rasa.com/docs/reference/primitives/slots/#calm-slot-mappings) and allow all types of command generators to [issue commands at every conversation turn](https://rasa.com/docs/reference/config/components/llm-command-generators/#interaction-with-other-types-of-command-generators), we have made the following changes which you should consider to benefit from the new CALM slot filling improvements:

- added a new method `_check_commands_overlap` to the base class `CommandGenerator`. This method checks if the commands issued by the current command generator overlap with the commands issued by other command generators. This method returns the final deduplicated commands. This method is called by the `predict_commands` method of the `CommandGenerator` children classes.
- added two new methods `_check_start_flow_command_overlap` and `_filter_slot_commands` to the base class `CommandGenerator` that will raise `NotImplementedError` if not implemented by the child class. These methods are already implemented by the `LLMBasedCommandGenerator` and `NLUCommandAdapter` classes to uphold the [prioritization system of the commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#prioritization-of-commands).
- added a new method `_get_prior_commands` to the base class `CommandGenerator`. This method returns a list of commands that have been issued by other command generators prior to the one currently running. This method is called by the `predict_commands` method of any command generators that inherit from the `CommandGenerator` class. This prior commands can be either returned in case of an empty tracker or flows, or included to the newly issued commands. For example:

```
prior_commands = self._get_prior_commands(tracker)if tracker is None or flows.is_empty():    return prior_commands# custom command generation logic blockreturn self._check_commands_overlap(prior_commands, commands)
```

- added a new method `_should_skip_llm_call` to the `LLMBasedCommandGenerator`. This method returns `True` only if `minimize_num_calls` is set to True and either prior commands contain a `StartFlow` command or a `SetSlot` command for the slot that is requested by an active `collect` flow step. This method is called by the `predict_commands` method of the `LLMBasedCommandGenerator` children classes. If the method returns `True`, the LLM call is skipped and the method returns the prior commands.
- moved the `_check_commands_against_slot_mappings` static method from the `CommandGenerator` to the `LLMBasedCommandGenerator` class. This method is used to check if the issued LLM commands are relevant to the slot mappings. The method is called by the `predict_commands` method of the `LLMBasedCommandGenerator` children classes.

### Migration from `SingleStepLLMCommandGenerator`s to the `CompactLLMCommandGenerator`s[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators 'Direct link to migration-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators')

It is recommended to use the new `CompactLLMCommandGenerator` with optimized prompts for the `gpt-4o-2024-11-20` and `claude-sonnet-3.5-20240620` models. Using the `CompactLLMCommandGenerator` can significantly reduce costs - approximately 10 times, according to our tests.

If you've built a custom command generator that extends `SingleStepLLMCommandGenerator`, we recommend migrating to the new command generator by inheriting the class from `CompactLLMCommandGenerator`.

```
# Old class definition:from rasa.dialogue_understanding.generators import SingleStepLLMCommandGeneratorclass MyCommandGenerator(SingleStepLLMCommandGenerator):    ...# New class definition:from rasa.dialogue_understanding.generators import CompactLLMCommandGeneratorclass MyCommandGenerator(CompactLLMCommandGenerator):    ...
```

### Migration from `SingleStepLLMCommandGenerator`s to the `CompactLLMCommandGenerator`s with the custom commands[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators-with-the-custom-commands 'Direct link to migration-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators-with-the-custom-commands')

If yo've built a custom command generator that extends `SingleStepLLMCommandGenerator` and you've defined new commands or overridden Rasa's default commands, you should:

- Update the `parse_commands` method to reflect the changes in the command parsing logic.
- Update the custom command classes so they are compatible with the latest command interface. For details on updating and implementing custom command classes, please refer to [How to customize existing commands](https://rasa.com/docs/pro/customize/command-generator/#how-to-customize-existing-commands) section.

In the new implementation, command parsing has been delegated to a dedicated parsing utility method `parse_commands` which can be imported from `rasa.dialogue_understanding.generator.command_parser` This method handles the parsing of the predicted LLM output into commands more effectively and flexibly, especially when using customized or newly introduced command types.

Here is the new recommended pattern for your command generator's `parse_commands` method:

```
# Import the utility method under a different name to prevent confusion with the# command generator's `parse_commands`from rasa.dialogue_understanding.generator.command_parser import (    parse_commands    as parse_commands_using_command_parsers,)class CustomCommandGenerator(CompactLLMCommandGenerator):    """Custom implementation of the LLM command generator."""    ...    @classmethod    def parse_commands(        cls, actions: Optional[str], tracker: DialogueStateTracker, flows: FlowsList    ) -> List[Command]:        """Parse the actions returned by the LLM into intents and entities as commands.        Args:            actions: The actions returned by the LLM.            tracker: The tracker containing the current state of the conversation.            flows: The current list of active flows.        Returns:            The parsed commands.        """        commands = parse_commands_using_command_parsers(            actions,            flows,            # Register any custom command classes you have created here            additional_commands=[CustomCommandClass1, CustomCommandClass2, ...],            # If your custom command classes replaces or extends default commands,            # specify the defaults commands for removal here            default_commands_to_remove=[HumandHandoffCommand, ...]        )        if not commands:            structlogger.warning(                f"{cls.__name__}.parse_commands",                message="No commands were parsed from the LLM actions.",                actions=actions,            )        return commands
```

### Migration of the custom prompt from `SingleStepLLMCommandGenerator`s to the `CompactLLMCommandGenerator`s[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-of-the-custom-prompt-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators 'Direct link to migration-of-the-custom-prompt-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators')

If you've customized the default prompt template previously used with the `SingleStepLLMCommandGenerator` and are now migrating to the `CompactLLMCommandGenerator`, you must update this template to use the new prompt commands syntax. This updated command syntax is specifically optimized for the capabilities of the new `CompactLLMCommandGenerator`.

For more details on the new prompt, refer to the documentation [here](https://rasa.com/docs/reference/config/components/llm-command-generators/#prompt-template)

### Update to `utter_corrected_previous_input` default utterance[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#update-to-utter_corrected_previous_input-default-utterance 'Direct link to update-to-utter_corrected_previous_input-default-utterance')

The text of the default `utter_corrected_previous_input` utterance has been updated to use a new correction frame context property `context.new_slot_values` instead of `context.corrected_slots.values`. The new utterance is:

```
"Ok, I am updating {{ context.corrected_slots.keys()|join(', ') }} to {{ context.new_slot_values | join(', ') }} respectively."
```

### LLM Judge Config Format Change in E2E Testing[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llm-judge-config-format-change-in-e2e-testing 'Direct link to LLM Judge Config Format Change in E2E Testing')

The custom configuration of the LLM Judge used by E2E testing with assertions has been updated to use the `llm_judge` key which follows the same structure as other generative components in Rasa. This can either use [model groups](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups) configuration or the individual model configuration option. The `llm_judge` key can be used in the `conftest.yml` file as shown below:

```
llm_judge:  llm:    provider: "openai"    model: "gpt-4-0613"  embeddings:    provider: "openai"    model: "text-embedding-ada-002"
```

### `action` property in custom slot mapping replaced with `run_action_every_turn`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#action-property-in-custom-slot-mapping-replaced-with-run_action_every_turn 'Direct link to action-property-in-custom-slot-mapping-replaced-with-run_action_every_turn')

With the deprecation of the [custom slot mapping](https://rasa.com/docs/reference/primitives/slots/#custom-slot-mappings) in favor of the new [controlled mapping](https://rasa.com/docs/reference/primitives/slots/#controlled-slot-mappings) type, the `action` property associated with the custom slot mapping has been replaced with the `run_action_every_turn` [property](https://rasa.com/docs/reference/primitives/slots/#run_action_every_turn). For this reason, if you prefer not to run these custom actions at every turn, it is recommended you remove the `action` property from your slot mappings.

## Rasa Pro 3.9 to Rasa Pro 3.10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-39-to-rasa-pro-310 'Direct link to Rasa Pro 3.9 to Rasa Pro 3.10')

### LLM/Embedding Configuration[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llmembedding-configuration 'Direct link to LLM/Embedding Configuration')

The LLM and embedding configurations have been updated to use the `provider` key instead of the `type` key. These changes apply to all providers, with some examples provided for reference.

**Cohere**

```
llm:  provider: "cohere" # instead of "type: cohere"  model: "command-r"
```

**Vertex AI**

```
llm:  provider: "vertex_ai" # instead of "type: vertexai"  model: "gemini-pro"
```

**Hugging Face Hub**

```
llm:  provider: "huggingface" # instead of "type: huggingface_hub"  model: "HuggingFaceH4/zephyr-7b-beta" # instead of "repo_id: HuggingFaceH4/zephyr-7b-beta"
```

**llama.cpp**

The support for loading models directly have been removed. You need to deploy the model to a server and use the server URL to load the model. For instance a llama.cpp server can be run using the following command, `./llama-server -m your_model.gguf --port 8080`.

For more information on llama.cpp server, refer to the [llama.cpp documentation](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#web-server) The assistant can be configured as:

```
llm:  provider: "self-hosted" # instead of "type: llamacpp"  api_base: "http://localhost:8000/v1" # instead of "model_path: "/path/to/model.bin""  model: "ggml-org/Meta-Llama-3.1-8B-Instruct-Q4_0-GGUF"
```

**vLLM**

The model can be deployed and served through `vLLM==0.6.0`. For instance a vLLM server can be run using the following command, `vllm serve your_model`

For more information on vLLM server, refer to the [vLLM documentation](https://docs.vllm.ai/en/stable/serving/openai_compatible_server.html#openai-compatible-server) The assistant can be configured as:

```
llm:  provider: "self-hosted" # instead of "type: vllm_openai"  api_base: "http://localhost:8000/v1"  model: "NousResearch/Meta-Llama-3-8B-Instruct"  # the name of the model you have deployed
```

note

CALM exclusively utilizes the chat completions endpoint of the model server, so it's essential that the model's tokenizer includes a chat template. Models lacking a chat template will not be compatible with CALM anymore.

Backward compatibility has been maintained for `OpenAI` and `Azure` configurations. For all other providers, ensure the use of the `provider` key and review the configuration against the [documentation](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/).

#### Disabling the cache[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#disabling-the-cache 'Direct link to Disabling the cache')

For Rasa Pro versions `<= 3.9.x`, the correct way to disable the cache was:

```
llm:    model: ...    cache: false
```

Rasa Pro `3.10.0` onwards, this has changed since we rely on LiteLLM to manage caching. To avoid errors, change your configuration to -

```
llm:    model: ...    cache:       no-cache: true
```

### Custom Components using an LLM[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#custom-components-using-an-llm 'Direct link to Custom Components using an LLM')

As of Rasa Pro 3.10, the backend for sending LLM and Embedding API requests has undergone a significant change. The previous LangChain version `0.0.329` has been replaced with [LiteLLM](https://www.litellm.ai/).

This shift can potentially break custom implementations of components that configure and send API requests to chat completion and embedding endpoints. Specifically, the following components are impacted:

- [SingleStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#singlestepllmcommandgenerator)
- [MultiStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#multistepllmcommandgenerator)
- [ContextualResponseRephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/)
- [EnterpriseSearchPolicy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/)
- [IntentlessPolicy](https://rasa.com/docs/reference/config/policies/intentless-policy/)
- [FlowRetrieval](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows)
- [LLMBasedRouter](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter)

If your project contains custom components based on any of the affected components listed above, you will need to verify and possibly refactor your code to ensure compatibility with LiteLLM.

#### Changes to `llm_factory`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-llm_factory 'Direct link to changes-to-llm_factory')

The `llm_factory` is used across all components that configure and send API requests to an LLM. Previously, the `llm_factory` relied on LangChain's [mapping](https://github.com/langchain-ai/langchain/blob/ecee4d6e9268d71322bbf31fd16c228be304d45d/langchain/llms/__init__.py#L110) to instantiate LangChain clients.

Rasa Pro 3.10 onwards, the `llm_factory` returns clients that conform to the new `LLMClient` protocol. This impacts any custom component that was previously relying on `LangChain` types.

If you have overridden components, such as a command generator, you will need to update your code to handle the new return type of `LLMClient`. This includes adjusting method calls and ensuring compatibility with the new protocol.

The following method calls will need to be adjusted if you have overridden them:

- `SingleStepLLMCommandGenerator.invoke_llm`
- `MultiStepLLMCommandGenerator.invoke_llm`
- `ContextualResponseRephraser.rephrase`
- `EnterpriseSearchPolicy.predict_action_probabilities`
- `IntentlessPolicy.generate_answer`
- `LLMBasedRouter.predict_commands`

Here's an example of how to update your code:

- Rasa 3.9 - LangChain
- Rasa 3.10 - LiteLLM

```
from rasa.shared.utils.llm import llm_factory# get the llm client via factoryllm = llm_factory(config, default_config)# get the llm response synchronouslysync_completion: str = llm.predict(prompt)# get the llm response asynchronouslyasync_completion: str = await llm.apredict(prompt)
```

```
from rasa.shared.utils.llm import llm_factoryfrom rasa.shared.providers.llm.llm_client import LLMClientfrom rasa.shared.providers.llm.llm_response import LLMResponse# get the llm client via factoryllm: LLMClient = llm_factory(config, default_config)# get the llm response synchronouslysync_response: LLMResponse = llm.completion(prompt)  # or llm.completion([prompt_1, prompt_2,..., prompt_n])sync_completion: str = sync_response.choices[0]# get the llm response asynchronouslyasync_response: LLMResponse = await llm.acompletion(prompt)  # or llm.acompletion([prompt_1, prompt_2,..., prompt_n])async_completion: str = async_response.choices[0]
```

#### Changes to `embedder_factory`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-embedder_factory 'Direct link to changes-to-embedder_factory')

The `embedder_factory` is used across all components that configure and send API requests to an embedding model. Previously, the `embedder_factory` returned LangChain's embedding clients of [Embeddings](https://github.com/langchain-ai/langchain/blob/ecee4d6e9268d71322bbf31fd16c228be304d45d/langchain/embeddings/base.py#L6) type.

Rasa Pro 3.10 onwards, the `embedder_factory` returns clients that conform to the new `EmbeddingClient` protocol. This change is part of the move to LiteLLM, and it impacts any custom components that were previously relying on LangChain types.

If you have overridden components that rely on instantiating clients with `embedder_factory` you will need to update your code to handle the new return type of `EmbeddingClient`. This includes adjusting method calls and ensuring compatibility with the new protocol.

The following method calls will need to be adjusted if you have overridden them:

- `FlowRetrieval.load`
- `FlowRetrieval.populate`
- `EnterpriseSearchPolicy.load`
- `EnterpriseSearchPolicy.train`
- `IntentlessPolicy.load`
- Or if you have overridden the `IntentlessPolicy.embedder` attribute.

Here's an example of how to update your code:

- Rasa 3.9 - LangChain
- Rasa 3.10 - LiteLLM

```
from rasa.shared.utils.llm import embedder_factory# get the embedding client via factoryembedder = embedder_factory(config, default_config)# get the embedding response synchronouslyvectors: List[List[float]] = embedder.embed_documents([doc_1, doc_2])# get the embedding response asynchronouslyvectors: List[List[float]] = await embedder.aembed_documents([doc_1, doc_2])
```

```
from rasa.shared.utils.llm import embedder_factoryfrom rasa.shared.providers.embedding.embedding_client import EmbeddingClientfrom rasa.shared.providers.embedding.embedding_response import EmbeddingResponse# get the embedding client via factoryembedder: EmbeddingClient = embedder_factory(config, default_config)# get the embedding response synchronouslysync_response: EmbeddingResponse = embedder.embed([doc_1, doc_2])vectors: List[List[float]] = sync_response.data# get the embedding response asynchronouslyasync_response: EmbeddingResponse = await embedder.aembed([doc_1, doc_2])vectors: List[List[float]] = async_response.data
```

#### Changes to `invoke_llm`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-invoke_llm 'Direct link to changes-to-invoke_llm')

The previous implementation of `invoke_llm` method in `SingleStepLLMCommandGenerator`, `MultiStepLLMCommandGenerator`, and the deprecated `LLMCommandGenerator` used `llm_factory` to instantiate LangChain clients. Since the factory now returns clients that conform to the new `LLMClient` protocol, any custom overrides of the `invoke_llm` method will need to be updated to accommodate the new return type.

Below you can find the `invoke_llm` method from Rasa Pro 3.9 and its updated version in Rasa Pro 3.10:

- Rasa 3.9
- Rasa 3.10

```
async def invoke_llm(self, prompt: Text) -> Optional[Text]:    """Use LLM to generate a response.    Args:        prompt: The prompt to send to the LLM.    Returns:        The generated text.    Raises:        ProviderClientAPIException if an error during API call.    """    llm = llm_factory(self.config.get(LLM_CONFIG_KEY), DEFAULT_LLM_CONFIG)    try:        return await llm.apredict(prompt)    except Exception as e:        structlogger.error("llm_based_command_generator.llm.error", error=e)        raise ProviderClientAPIException(            message="LLM call exception", original_exception=e        )
```

```
async def invoke_llm(self, prompt: Text) -> Optional[Text]:    """Use LLM to generate a response.    Args:        prompt: The prompt to send to the LLM.    Returns:        The generated text.    Raises:        ProviderClientAPIException if an error during API call.    """    llm = llm_factory(self.config.get(LLM_CONFIG_KEY), DEFAULT_LLM_CONFIG)    try:        llm_response = await llm.acompletion(prompt)        return llm_response.choices[0]    except Exception as e:        structlogger.error("llm_based_command_generator.llm.error", error=e)        raise ProviderClientAPIException(            message="LLM call exception", original_exception=e        )
```

#### Changes to `SingleStepLLMCommandGenerator.predict_commands`[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-singlestepllmcommandgeneratorpredict_commands 'Direct link to changes-to-singlestepllmcommandgeneratorpredict_commands')

For `SingleStepLLMCommandGenerator`, the `predict_commands` method now includes a call to `self._update_message_parse_data_for_fine_tuning(message, commands, flow_prompt)`. This function is essential for enabling the [fine-tuning recipe](https://rasa.com/docs/pro/customize/fine-tuning-llm/).

If you have overridden the `predict_commands` method, you need to manually add this call to ensure proper functionality:

```
async def predict_commands(        self,        message: Message,        flows: FlowsList,        tracker: Optional[DialogueStateTracker] = None,        **kwargs: Any,    ) -> List[Command]:    ...    action_list = await self.invoke_llm(flow_prompt)    commands = self.parse_commands(action_list, tracker, flows)    self._update_message_parse_data_for_fine_tuning(message, commands, flow_prompt)    return commands
```

#### Changes to the default configuration dictionary[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-the-default-configuration-dictionary 'Direct link to Changes to the default configuration dictionary')

The default configurations for the following components have been updated:

- [SingleStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#singlestepllmcommandgenerator)
- [MultiStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#multistepllmcommandgenerator)
- [ContextualResponseRephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/)
- [EnterpriseSearchPolicy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/)
- [IntentlessPolicy](https://rasa.com/docs/reference/config/policies/intentless-policy/)
- [FlowRetrieval](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows)
- [LLMBasedRouter](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter)

If you have custom implementations based on the default configurations for any of these components, ensure that your configuration dictionary aligns with the updates shown in the tables below, as the defaults have changed.

Default **LLM configuration** keys have been updated from:

```
DEFAULT_LLM_CONFIG = {    "_type": "openai",    "model_name": ...,    "request_timeout": ...,    "temperature": ...,    "max_tokens": ...,}
```

to:

```
DEFAULT_LLM_CONFIG = {    "provider": "openai",    "model": ...,    "temperature": ...,    "max_tokens": ...,    "timeout": ...,}
```

Similarly, default **embedding configuration** keys have been updated from:

```
DEFAULT_EMBEDDINGS_CONFIG = {    "_type": "openai",    "model": ...,}
```

to:

```
DEFAULT_EMBEDDINGS_CONFIG = {    "provider": "openai",    "model": ...,}
```

Be sure to update your custom configurations to reflect these changes in order to ensure continued functionality.

### Dropped support for Python 3.8[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#dropped-support-for-python-38 'Direct link to Dropped support for Python 3.8')

Dropped support for Python 3.8 ahead of [Python 3.8 End of Life in October 2024](https://devguide.python.org/versions/#supported-versions).

In Rasa Pro versions `3.10.0`, `3.9.11` and `3.8.13`, we needed to pin the TensorFlow library version to 2.13.0rc1 in order to remove critical vulnerabilities; this resulted in poor user experience when installing these versions of Rasa Pro with `uv pip`. Removing support for Python 3.8 will make it possible to upgrade to a stabler version of TensorFlow.

## Rasa Pro 3.8 to Rasa Pro 3.9[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-38-to-rasa-pro-39 'Direct link to Rasa Pro 3.8 to Rasa Pro 3.9')

### LLMCommandGenerator[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llmcommandgenerator 'Direct link to LLMCommandGenerator')

Starting from Rasa Pro 3.9 the former `LLMCommandGenerator` is replaced by `SingleStepLLMCommandGenerator`. The `LLMCommandGenerator` is now deprecated and will be removed in version `4.0.0`.

The `SingleStepLLMCommandGenerator` differs from the `LLMCommandGenerator` in how it handles failures of the `invoke_llm` method. Specifically, if the `invoke_llm` method call fails in `SingleStepLLMCommandGenerator`, it raises a `ProviderClientAPIException`. In contrast, the `LLMCommandGenerator` simply returns `None` when the method call fails.

### Slot Mappings[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#slot-mappings 'Direct link to Slot Mappings')

In case you had been using `custom` slot mapping type for slots set with the prediction of the LLM-based command generator, you need to update your assistant's slot configuration to use the new [`from_llm`](https://rasa.com/docs/reference/primitives/slots/#from_llm) slot mapping type. Note that even if you have written custom slot validation actions (following the `validate_<slot_name>` convention) for slots set by the LLM-based command generator, you need to update your assistant's slot configuration to use the new `from_llm` slot mapping type.

For [slots that are set only via a custom action](https://rasa.com/docs/reference/primitives/slots/#custom-slot-mappings) e.g. slots set by external sources only, you must add the action name to the slot mapping:

```
slots:  slot_name:    type: text    mappings:      - type: custom        action: custom_action_name
```

## Rasa Pro 3.8.0 to Rasa Pro 3.8.1[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-380-to-rasa-pro-381 'Direct link to Rasa Pro 3.8.0 to Rasa Pro 3.8.1')

### Poetry Installation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#poetry-installation 'Direct link to Poetry Installation')

Starting from Rasa Pro 3.8.1 in the `3.8.x` minor series, we have upgraded the version Poetry for managing dependencies in the Rasa Pro Python package to `1.8.2`. To install the latest micro versions of Rasa Pro in your project, you must first [upgrade Poetry](https://python-poetry.org/docs/cli/#self-update) to version `1.8.2`:

```
poetry self update 1.8.2
```

## Rasa Pro 3.7 to 3.8[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-37-to-38 'Direct link to Rasa Pro 3.7 to 3.8')

info

Starting from `3.8.0`, Rasa and Rasa Plus have been merged into a single artifact, named Rasa Pro.

### Installation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#installation 'Direct link to Installation')

Following the merge we renamed the resulting python package and Docker image to `rasa-pro`.

#### Python package[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#python-package 'Direct link to Python package')

Rasa Pro python package, for `3.8.0` and onward, is located at:

```
https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python
```

Name of the package is `rasa-pro`.

Example of how to install the package:

```
pip install  --extra-index-url=https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple rasa-pro==3.8.0
```

While python package name was changed, the import process remains the same:

```
import rasa.corefrom rasa import train
```

For more information on how to install Rasa Pro, please refer to the [Python installation guide](https://rasa.com/docs/pro/installation/python/).

#### Helm Chart / Docker Image[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#helm-chart--docker-image 'Direct link to Helm Chart / Docker Image')

Rasa Pro docker image, for `3.8.0` and onward, is located at:

```
europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro
```

Example how to pull the image:

```
docker pull europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0
```

For more information on how to install Rasa Pro Docker image, please refer to the [Docker installation guide](https://rasa.com/docs/pro/installation/docker/).

### Component Yaml Configuration Changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#component-yaml-configuration-changes 'Direct link to Component Yaml Configuration Changes')

Follow the below instructions to update the configuration of Rasa Pro components in the 3.8 version:

- [`ConcurrentRedisLockStore`](https://rasa.com/docs/reference/integrations/lock-stores/#concurrentredislockstore) - update `endpoints.yml` to `type: concurrent_redis`:

```
lock_store:  type: concurrent_redis
```

- [`ContextualResponseRephraser`](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/)\- update `endpoints.yml` to either `type: rephrase` or `type: rasa.core.ContextualResponseRephraser`:

```
nlg:  type: rephrase
```

- [Audiocodes](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/) and Vier CVG channels can be specified in `credentials.yml` using directly their channel name:

```
audiocodes:  token: "sample_token"vier_cvg:  ...
```

- [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) and [`IntentlessPolicy`](https://rasa.com/docs/reference/config/policies/intentless-policy/) - update `config.yml` to only use the policy class name:

```
    policies:      - name: EnterpriseSearchPolicy      - name: IntentlessPolicy
```

### Changes to default behaviour[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-behaviour 'Direct link to Changes to default behaviour')

info

With Rasa Pro 3.8, we introduced a couple of changes that rectifies the default behaviour of certain components. We believe these changes align better with the principles of CALM. If you are migrating an assistant built with Rasa Pro 3.7, please ensure you have checked if these changes affect your assistant.

#### Prompt Rendering[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#prompt-rendering 'Direct link to Prompt Rendering')

Rasa Pro 3.8 introduces a new feature [flow-retrieval](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows) which ensures that only the flows that are relevant to the conversation context are included in the prompt sent to the LLM in the `LLMCommandGenerator`. This helps the assistant scale to a higher number of flows and also reduces the LLM costs.

This feature is **enabled by default** and we recommend to use it if the assistant has more than 40 flows. By default, the feature uses embedding models from OpenAI, but if you are using a different provider (for e.g. Azure), please ensure -

1. An embedding model is configured with the provider.
2. `LLMCommandGenerator` has been configured correctly to connect to the embedding provider. For example, see the section on [configuration required to connect to Azure OpenAI service](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure-openai-service)

If you wish to disable the feature you can configure the `LLMCommandGenerator` as:

config.yml

```
pipeline:  - name: SingleStepLLMCommandGenerator    ...    flow_retrieval:      active: false    ...
```

#### Processing Chitchat[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#processing-chitchat 'Direct link to Processing Chitchat')

The default behaviour in Rasa Pro 3.7 to handle chitchat utterances was to rely on free form generative responses. This can lead to the assistant sending unwanted responses or responding to out of scope user utterances. The new default behaviour in Rasa Pro 3.8 is to rely on [`IntentlessPolicy`](https://rasa.com/docs/reference/config/policies/intentless-policy/) to respond to chitchat utterances using pre-defined responses only.

If you were relying on free form generative responses to handle chitchat in Rasa Pro 3.7, you will now see a warning message when you train the same assistant with Rasa Pro 3.8 - " `pattern_chitchat` has an action step with `action_trigger_chitchat`, but `IntentlessPolicy` is not configured". This appears because the default definition of `pattern_chitchat` has been modified in Rasa Pro 3.8 to:

```
pattern_chitchat:  description: handle interactions with the user that are not task-oriented  name: pattern chitchat  steps:    - action: action_trigger_chitchat
```

For the assistant to be able to handle chitchat utterances, you have two options:

1. If you are happy with free-form generative responses for such user utterances, then you can override `pattern_chitchat` to:
 
 ```
    pattern_chitchat:  description: handle interactions with the user that are not task-oriented  name: pattern chitchat  steps:    - action: utter_free_chitchat_response
    ```
 
2. If you want to switch to using pre-defined responses, you should first add `IntentlessPolicy` to the `policies` section of the config -
 
 ```
    policies:  - name: IntentlessPolicy
    ```
 

Next, you should add response templates for the pre-defined responses you want the assistant to consider when [responding to a chitchat user utterance](https://rasa.com/docs/reference/config/policies/intentless-policy/#chitchat).

#### Handling of categorical slots[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#handling-of-categorical-slots 'Direct link to Handling of categorical slots')

Rasa Pro versions `<= 3.7.8` used to store the value of a categorical slot in the same casing as it was either specified in the user message or predicted by the LLM in a `SetSlot` command. This wasn't necessarily same as the casing used in the corresponding possible value defined for that slot in the domain. For e.g, if the categorical slot was defined to have `[A, B, C]` as the possible values and the prediction was to set it to `a` then the slot would be set to `a`. This lead to problems downstream when that slot had to be used in other primitives i.e. flows or custom action.

Rasa Pro `3.7.9` fixes this by always storing the slot value in the same casing as defined in the domain. So, in the above example, the slot would now be stored as `A` instead of `a`. This ensures that the user is writing business logic for slot comparisons, for e.g. `if` conditions in flows, using the same casing as defined by them in the domain.

If you are migrating from Rasa pro versions `<= 3.7.8`, please double check your flows and custom actions to make sure none of them break because of this change.

#### Update default signature of LLM calls[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#update-default-signature-of-llm-calls 'Direct link to Update default signature of LLM calls')

In Rasa Pro `>= 3.8` we switched from doing synchronous LLM calls to asynchronous calls. We updated all components that use an LLM, e.g.

- `LLMCommandGenerator`
- `ContextualResponseRephraser`
- `EnterpriseSearchPolicy`
- `IntentlessPolicy`

This can potentially break assistants migrating to 3.8 that have sub-classed one of these components in their own custom components.

For example, the method `predict_commands` in the `LLMCommandGenerator` is now `async` and needs to `await` the methods `_generate_action_list_using_llm` and `flow_retrieval.filter_flows` as these methods are also async. For more information on asyncio please check their [documentation](https://docs.python.org/3/library/asyncio.html).

### Dependency Upgrades[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#dependency-upgrades 'Direct link to Dependency Upgrades')

We've updated our core dependencies to enhance functionality and performance across our platform.

#### Spacy 3.7.x[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#spacy-37x 'Direct link to Spacy 3.7.x')

Upgraded from `>=3.6` to `>=3.7`.

We have transitioned to using Spacy version 3.7.x to benefit from the latest enhancements in natural language processing. If you're using any spacy models with your assistant, please update them to Spacy 3.7.x compatible models.

#### Pydantic 2.x[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#pydantic-2x 'Direct link to Pydantic 2.x')

Upgraded from `>=1.10.9,<1.10.10` to `^2.0`.

Along with the Spacy upgrade, we have moved to Pydantic version 2.x, which necessitates updates to Pydantic models. For assistance with updating your models, please refer to the [Pydantic Migration Guide](https://docs.pydantic.dev/latest/migration/#install-pydantic-v2). This ensures compatibility with the latest improvements in data validation and settings management.

## Rasa Pro 3.7.9 to Rasa Pro 3.7.10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-379-to-rasa-pro-3710 'Direct link to Rasa Pro 3.7.9 to Rasa Pro 3.7.10')

### Poetry Installation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#poetry-installation-1 'Direct link to Poetry Installation')

Starting from Rasa Pro 3.7.10 in the `3.7.x` minor series, we have upgraded the version Poetry for managing dependencies in the Rasa Pro Python package to `1.8.2`. To install Rasa Pro in your project, you must first [upgrade Poetry](https://python-poetry.org/docs/cli/#self-update) to version `1.8.2`:

```
poetry self update 1.8.2
```

## Rasa Pro 3.7.8 to Rasa Pro 3.7.9[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-378-to-rasa-pro-379 'Direct link to Rasa Pro 3.7.8 to Rasa Pro 3.7.9')

### Changes to default behaviour[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-behaviour-1 'Direct link to Changes to default behaviour')

#### Handling of categorical slots[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#handling-of-categorical-slots-1 'Direct link to Handling of categorical slots')

Rasa Pro versions `<= 3.7.8` used to store the value of a categorical slot in the same casing as it was either specified in the user message or predicted by the LLM in a `SetSlot` command. This wasn't necessarily same as the casing used in the corresponding possible value defined for that slot in the domain. For e.g, if the categorical slot was defined to have `[A, B, C]` as the possible values and the prediction was to set it to `a` then the slot would be set to `a`. This lead to problems downstream when that slot had to be used in other primitives i.e. flows or custom action.

Rasa Pro `3.7.9` fixes this by always storing the slot value in the same casing as defined in the domain. So, in the above example, the slot would now be stored as `A` instead of `a`. This ensures that the user is writing business logic for slot comparisons, for e.g. `if` conditions in flows, using the same casing as defined by them in the domain.

If you are migrating from Rasa pro versions `<= 3.7.8`, please double check your flows and custom actions to make sure none of them break because of this change.

## Rasa 3.6 to Rasa Pro 3.7[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-36-to-rasa-pro-37 'Direct link to Rasa 3.6 to Rasa Pro 3.7')

### Installation[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#installation-1 'Direct link to Installation')

info

Starting from Rasa 3.7.0, Rasa has moved to a new package registry and Docker registry. You will need to update your package registry to install Rasa 3.7.0 and later versions. If you are a Rasa customer, please reach out to your Rasa account manager or support obtain a [license](https://rasa.com/docs/pro/installation/licensing/).

#### Python package[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#python-package-1 'Direct link to Python package')

Rasa python package for `3.7.0` has been moved to python package registry.

```
https://europe-west3-python.pkg.dev/rasa-releases/rasa-plus-py
```

Name of the package is `rasa`.

Example of how to install the package:

```
pip install  --extra-index-url=https://europe-west3-python.pkg.dev/rasa-releases/rasa-plus-py/simple rasa==3.7.0
```

For more information on how to install Rasa Pro, please refer to the [Python installation guide](https://rasa.com/docs/pro/installation/python/).

#### Helm Chart / Docker Image[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#helm-chart--docker-image-1 'Direct link to Helm Chart / Docker Image')

Rasa docker image for `3.7.0` is located at:

```
europe-west3-docker.pkg.dev/rasa-releases/rasa-docker/rasa
```

Example how to pull the image:

```
docker pull europe-west3-docker.pkg.dev/rasa-releases/rasa-docker/rasa:3.7.0
```

For more information on how to install Rasa Pro Docker image, please refer to the [Docker installation guide](https://rasa.com/docs/pro/installation/docker/).

## Migrating from older versions[​](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migrating-from-older-versions 'Direct link to Migrating from older versions')

For migrating from Rasa Open Source versions, please refer to the [migration guide](https://legacy-docs-oss.rasa.com/docs/rasa/migration-guide).

- [Rasa Pro 3.15 to Rasa Pro 3.16](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-315-to-rasa-pro-316)
 - [Default Models for LLM-based Components](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#default-models-for-llm-based-components)
 - [Changes to Default Command-Generator Prompt Templates](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-command-generator-prompt-templates)
 - [Changes to Default ReAct Sub-agent Prompt Templates](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-react-sub-agent-prompt-templates)
 - [Changes to Agent Protocol and Custom Tool Interface](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-agent-protocol-and-custom-tool-interface)
 - [Support for `user_id` in `DialogueStateTracker`](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#support-for-user_id-in-dialoguestatetracker)
 - [Removal of the `HumanHandoff` / `hand over` command](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#removal-of-the-humanhandoff--hand-over-command)
- [Rasa Pro 3.14 to Rasa Pro 3.15](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-314-to-rasa-pro-315)
 - [Changes to `invoke_llm` Method Signature](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-invoke_llm-method-signature)
 - [Changes to Pattern Clarification](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-pattern-clarification)
 - [Changes to `AgentInput` and `AgentOutput`](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-agentinput-and-agentoutput)
 - [Changes to Agent Protocol](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-agent-protocol)
 - [Updated Default Prompts to Support Current Date Time in Prompts](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#updated-default-prompts-to-support-current-date-time-in-prompts)
- [Rasa Pro 3.13 to Rasa Pro 3.14](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-313-to-rasa-pro-314)
 - [Dependencies](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#dependencies)
 - [Pattern Continue Interrupted](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#pattern-continue-interrupted)
 - [Command Generator](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#command-generator)
 - [LLM Clients](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llm-clients)
 - [Command](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#command)
 - [Tracing for Jaeger](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#tracing-for-jaeger)
- [Rasa Pro 3.12 to Rasa Pro 3.13](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-312-to-rasa-pro-313)
 - [LLM Judge Model Change in E2E Testing](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llm-judge-model-change-in-e2e-testing)
- [Rasa Pro 3.11 to Rasa Pro 3.12](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-311-to-rasa-pro-312)
 - [Custom LLM-based Command Generators](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#custom-llm-based-command-generators)
 - [Migration from `SingleStepLLMCommandGenerator`s to the `CompactLLMCommandGenerator`s](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators)
 - [Migration from `SingleStepLLMCommandGenerator`s to the `CompactLLMCommandGenerator`s with the custom commands](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators-with-the-custom-commands)
 - [Migration of the custom prompt from `SingleStepLLMCommandGenerator`s to the `CompactLLMCommandGenerator`s](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migration-of-the-custom-prompt-from-singlestepllmcommandgenerators-to-the-compactllmcommandgenerators)
 - [Update to `utter_corrected_previous_input` default utterance](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#update-to-utter_corrected_previous_input-default-utterance)
 - [LLM Judge Config Format Change in E2E Testing](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llm-judge-config-format-change-in-e2e-testing)
 - [`action` property in custom slot mapping replaced with `run_action_every_turn`](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#action-property-in-custom-slot-mapping-replaced-with-run_action_every_turn)
- [Rasa Pro 3.9 to Rasa Pro 3.10](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-39-to-rasa-pro-310)
 - [LLM/Embedding Configuration](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llmembedding-configuration)
 - [Custom Components using an LLM](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#custom-components-using-an-llm)
 - [Dropped support for Python 3.8](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#dropped-support-for-python-38)
- [Rasa Pro 3.8 to Rasa Pro 3.9](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-38-to-rasa-pro-39)
 - [LLMCommandGenerator](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#llmcommandgenerator)
 - [Slot Mappings](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#slot-mappings)
- [Rasa Pro 3.8.0 to Rasa Pro 3.8.1](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-380-to-rasa-pro-381)
 - [Poetry Installation](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#poetry-installation)
- [Rasa Pro 3.7 to 3.8](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-37-to-38)
 - [Installation](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#installation)
 - [Component Yaml Configuration Changes](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#component-yaml-configuration-changes)
 - [Changes to default behaviour](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-behaviour)
 - [Dependency Upgrades](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#dependency-upgrades)
- [Rasa Pro 3.7.9 to Rasa Pro 3.7.10](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-379-to-rasa-pro-3710)
 - [Poetry Installation](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#poetry-installation-1)
- [Rasa Pro 3.7.8 to Rasa Pro 3.7.9](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-378-to-rasa-pro-379)
 - [Changes to default behaviour](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#changes-to-default-behaviour-1)
- [Rasa 3.6 to Rasa Pro 3.7](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-36-to-rasa-pro-37)
 - [Installation](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#installation-1)
- [Migrating from older versions](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#migrating-from-older-versions)