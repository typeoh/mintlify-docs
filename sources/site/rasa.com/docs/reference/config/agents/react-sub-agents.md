# Source: https://rasa.com/docs/reference/config/agents/react-sub-agents

On this page

info

ReAct Sub Agents are currently in **beta** and are available starting from **Rasa 3.14.0**.

ReAct sub agents are built-in autonomous agents that can dynamically utilize [MCP (Model Context Protocol)](https://modelcontextprotocol.io/docs/getting-started/intro) tools and custom tools defined in Python code to complete tasks. They follow a [ReAct](https://arxiv.org/abs/2210.03629) (**re**asoning + **act**ing) loop: a large language model (LLM) chooses which tools to invoke based on the ongoing conversation context.

Two types of ReAct sub agents are currently supported:

1. **[General-purpose ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/#general-purpose-react-sub-agent)**: Handles open-ended tasks and requires explicit completion signaling
2. **[Task-specific ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/#task-specific-react-sub-agent)**: Focuses on filling specific slots with automatic completion based on exit conditions

## General-purpose ReAct Sub Agent[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#general-purpose-react-sub-agent 'Direct link to General-purpose ReAct Sub Agent')

A general-purpose ReAct sub agent is designed for open-ended, exploratory tasks where the agent itself determines when the task is complete. Unlike task-specific agents that have predefined exit conditions, general-purpose agents require explicit completion signaling through a built-in [`task_completed` tool](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-task_completed-tool).

### Key Characteristics[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#key-characteristics 'Direct link to Key Characteristics')

- **Open-ended tasks**: Handles exploratory conversations and complex, multi-step tasks
- **Autonomous completion**: The agent decides when it has fully completed its assigned task
- **Explicit signaling**: Uses the built-in [`task_completed` tool](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-task_completed-tool) to signal completion
- **No exit conditions**: Invoked via [autonomous call steps without exit conditions](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps)

### The `task_completed` Tool[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-task_completed-tool 'Direct link to the-task_completed-tool')

General-purpose ReAct sub agents have access to a built-in `task_completed` tool that ends the ReAct loop and returns control to the flow. The tool is registered automatically and always uses the name `task_completed` so the runtime can recognize completion. The tool has **no parameters**. When calling `task_completed`, the model should include a short, natural user-facing message in the same response. This message appears directly to the user and serves as a brief closing line for that turn.

You can change or replace this tool by subclassing `MCPOpenAgent` and overriding [`get_task_completed_tool()`](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-task_completed-tool-and-empty-final-messages).

#### Tool definition[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#tool-definition 'Direct link to Tool definition')

task\_completed tool schema

```
{  "type": "function",  "function": {    "name": "task_completed",    "description": "Call this tool exactly once when your primary task is FULLY completed. This tool accepts no arguments. You MUST also include text in the same response: a natural, conversational follow-up to the user's last message (e.g. acknowledge their choice, wish them well, close warmly). Do NOT summarize what you did or what happened in the conversation. Do NOT include any inner thoughts or explanations. A response with only the tool call and no text is invalid. Keep it short and natural.",    "parameters": {      "type": "object",      "properties": {},      "additionalProperties": false    },    "strict": true  }}
```

Exact wording can change between Rasa versions; override [`get_task_completed_tool()`](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-task_completed-tool-and-empty-final-messages) if you need to give different instructions to the model.

#### How Completion Works[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#how-completion-works 'Direct link to How Completion Works')

1. **Agent decision**: The ReAct sub agent autonomously determines when its primary task is complete and the execution loop should finish.
2. **Tool call**: When ready to complete, the model calls the `task_completed` tool once. The tool has no parameters; the closing line for the user is agent message in the same turn (a short follow-up to the user’s last message—not a recap of the whole conversation, and not a message argument on the tool).
3. **Completion signal**: The `task_completed` call terminates the ReAct sub agent’s execution loop. If the model also invoked other MCP tools in that same response, Rasa still processes those tool results before the run completes. The run ends with status `completed`.
4. **User response**: The user sees the streamed agent message from that turn. The tool itself only returns a fixed internal result (`Task completed`); that value is not a user-facing parameter.
5. **Control return**: Control returns to the main flow, which can then proceed to the next step.

#### `task_completed` with no agent message[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#task_completed-with-no-agent-message 'Direct link to task_completed-with-no-agent-message')

If the model calls `task_completed` with no agent message in the same turn, Rasa does not finish the run yet. Rasa adds a one-turn system message: do not call tools, only send a short closing line to the user. It calls the LLM again with tools disabled. The run then completes from that text-only reply, without calling `task_completed` again.

## Task-specific ReAct Sub Agent[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#task-specific-react-sub-agent 'Direct link to Task-specific ReAct Sub Agent')

A task-specific ReAct sub agent is designed for structured data collection tasks where specific slots need to be filled with values. Unlike general-purpose agents that determine their own completion, task-specific agents automatically complete when predefined exit conditions are met, making them ideal for appointment booking, form filling, or data collection scenarios.

### Key Characteristics[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#key-characteristics-1 'Direct link to Key Characteristics')

- **Structured data collection**: Focuses on filling specific slots with values.
- **Automatic completion**: Completes when exit conditions are satisfied.
- **Built-in slot tools**: Automatically gets [`set_slot_<slot_name>` tools](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-set_slot_slot_name-tools) for each slot in exit conditions.
- **Exit condition driven**: Invoked via [autonomous call steps with `exit_if` conditions](https://rasa.com/docs/reference/primitives/flow-steps/#specifying-exit-conditions).

### The `set_slot_<slot_name>` Tools[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-set_slot_slot_name-tools 'Direct link to the-set_slot_slot_name-tools')

Task-specific ReAct sub agents automatically receive built-in `set_slot_<slot_name>` tools for each slot mentioned in the `exit_if` conditions. These tools are dynamically generated based on the slot definitions and cannot be disabled.

#### Tool Generation[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#tool-generation 'Direct link to Tool Generation')

For each slot in the exit conditions, a tool is created with the following pattern:

- **Tool name**: `set_slot_<slot_name>` (e.g., `set_slot_user_name`, `set_slot_appointment_date`)
- **Dynamic description**: Includes slot type and allowed values (for categorical slots)
- **Slot-specific parameters**: Tailored to the slot's data type and constraints

#### Example Tool Definition[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#example-tool-definition 'Direct link to Example Tool Definition')

For a slot named `user_name` of type `text`:

set\_slot\_user\_name tool schema (example)

```
{  "type": "function",  "function": {    "name": "set_slot_user_name",    "description": "Set the slot 'user_name' to a specific value. The slot type is text.",    "parameters": {      "type": "object",      "properties": {        "slot_value": {          "type": "string",          "description": "The value to assign to the slot."        }      },      "required": ["slot_value"],      "additionalProperties": false    },    "strict": true  }}
```

For a slot named `category` of type `categorical` with allowed values `["A", "B", "C"]`:

set\_slot\_category tool schema (example, categorical)

```
{  "type": "function",  "function": {    "name": "set_slot_category",    "description": "Set the slot 'category' to a specific value. The slot type is categorical. The allowed values are: ['A', 'B', 'C'].",    "parameters": {      "type": "object",      "properties": {        "slot_value": {          "type": "string",          "description": "The value to assign to the slot."        }      },      "required": ["slot_value"],      "additionalProperties": false    },    "strict": true  }}
```

#### How Slot Setting Works[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#how-slot-setting-works 'Direct link to How Slot Setting Works')

1. **Tool Availability**: The ReAct sub agent automatically receives `set_slot_<slot_name>` tools for all slots mentioned in the `exit_if` conditions.
2. **Value Assignment**: When the ReAct sub agent calls a `set_slot_<slot_name>` tool, the slot value is immediately updated.
3. **Condition Evaluation**: After each slot update, the system checks if all exit conditions are satisfied.
4. **Automatic Completion**: Once all exit conditions are met, the ReAct sub agent automatically completes without sending a response.
5. **Control Return**: Control returns to the main flow, which can proceed to the next step.

## Configuration[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration 'Direct link to Configuration')

ReAct sub agents extend the [basic sub agent configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration) with additional settings.

In addition to the required `agent` section, ReAct sub agents support the following configuration options:

ReAct sub agent config.yml (illustrative)

```
# Basic agent information (required - see overview)agent:  name: stock_explorer  protocol: RASA # Optional: protocol for agent connections, default is 'RASA'  description: "Agent that helps users research and analyze stock options"# ReAct-specific configurationconfiguration:  llm:  # Optional: Same format as other Rasa LLM configs    model_group: openai-gpt-5-1  prompt_template: sub_agents/stock_explorer/prompt_template.jinja2  # Optional: prompt template file  timeout: 30  # Optional: seconds before timing out autonomous run  max_retries: 3  # Optional: connection retry attempts for MCP  include_date_time: true  # Optional: enable datetime in prompts (default: true)  timezone: "UTC"  # Optional: IANA timezone name (default: "UTC")  enable_filler_messages: true  # Optional: intermediate messages before tools (default: true)  tool_timeout: 30.0  # Optional: per-tool call timeout in seconds (must be > 0)  module: "path.to.custom.module"  # Optional: custom module for sub agent customization# MCP server connectionsconnections:  mcp_servers:  - name: trade_server    include_tools:  # Optional: specify which tools to include    - find_symbol    - get_company_news    - apply_technical_analysis    - fetch_live_price    exclude_tools:  # Optional: tools to exclude    - place_buy_order    - view_positions
```

### Configuration Section[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration-section 'Direct link to Configuration Section')

The `configuration` section is optional in the ReAct sub agent's `config.yml` file and provides additional customization options for LLM and connection settings.

- **`configuration.llm`**: Defines which Large Language Model (LLM) powers the ReAct sub agent's reasoning and tool use. The configuration format matches the standard used for other Rasa LLM settings. For more information, refer to the [LLM Configuration](https://rasa.com/docs/reference/config/components/llm-configuration/) documentation. If not specified, the default OpenAI chat model (`gpt-5.1-2025-11-13`) is used.
 
- **`configuration.prompt_template`**: Path to a Jinja2 template for [customizing the prompt](https://rasa.com/docs/reference/config/agents/react-sub-agents/#modifying-the-prompt-template) of this ReAct sub agent. By default, Rasa uses the shipped template for your agent type (`mcp_open_agent_prompt_template.jinja2` for general-purpose, `mcp_task_agent_prompt_template.jinja2` for task-specific). **Conversation history is not part of that template**: Rasa renders the template as the **system** message and appends recent user and assistant utterances as separate chat messages when calling the LLM.
 
- **`configuration.timeout`**: Specifies how many seconds to wait before timing out autonomous sub agent execution. There is no timeout by default.
 
- **`configuration.max_retries`**: Number of retry attempts if MCP server connection fails. The default is 3 attempts.
 
- **`configuration.include_date_time`** (optional, default: `true`): Enable or disable datetime information in prompts. When enabled, prompts include current date, time, and day of the week to help the model understand temporal references.
 
- **`configuration.timezone`** (optional, default: `"UTC"`): A valid IANA timezone name (e.g., `"America/New_York"`, `"Europe/London"`, `"Asia/Tokyo"`).
 
- **`configuration.enable_filler_messages`** (optional, default: `true`): Enables intermediate assistant messages for ReAct sub-agents. When enabled, the model includes a short text response alongside tool calls, which is sent to the user before tool execution to reduce perceived latency. Rasa logs this as a BotUttered event with filler metadata to keep the dialogue state consistent. Disable to skip intermediate messages and related metadata.
 
- **`configuration.tool_timeout`** (optional): Maximum time in seconds for a single tool invocation.
 

sub\_agents/stock\_explorer/config.yml

```
configuration:  llm:    model-group: openai-gpt-5-1  include_date_time: true  # Defaults to true  timezone: "America/New_York"  # Defaults to UTC if not specified
```

Timezone Format

The `timezone` parameter must be a valid [IANA timezone](https://www.iana.org/time-zones) name. Common examples include:

- `"UTC"`
- `"America/New_York"`
- `"Europe/London"`
- `"Asia/Tokyo"`

If an invalid timezone is provided, Rasa will raise a `ValidationError` during agent initialization.

### Connections Section[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#connections-section 'Direct link to Connections Section')

The `connections` section is mandatory for ReAct sub agents and specifies which MCP servers the ReAct sub agent should connect to. MCP servers are configured in your `endpoints.yml` file. For more details, see the [MCP Server Integration](https://rasa.com/docs/reference/integrations/mcp-servers/) page.

- **`connections.mcp_servers` (required)**: A list of one or more MCP servers that this ReAct agent connects to. At least one MCP server must be specified, and each server configuration follows this structure:
 - **`name` (required)**: Name to reference this MCP server
 - **Tool Filtering (optional)**:
 - `include_tools`: List of specific MCP tools available to this sub agent
 - `exclude_tools`: List of tools to explicitly hide from this sub agent (useful for restricting certain actions)

For each MCP server, you can define which tools to include or exclude. The `include_tools` and `exclude_tools` parameters are mutually exclusive.

## Intermediate messages[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#intermediate-messages 'Direct link to Intermediate messages')

Rasa uses **intermediate messages** or filler messages for user-visible updates while a sub agent is still working. For **ReAct** sub agents, intermediate messages are **currently** implemented as **tool acknowledgements**: brief agent message that the model emits in the **same** LLM response as its tool calls. Rasa forwards that text to the user **before** MCP or custom tools run.

**Default behavior:** `enable_filler_messages` defaults to `true`. The built-in Jinja2 prompt adds a “Tool Usage Acknowledgments” section that tells the model to keep those intermediate lines short, avoid repeating the user’s request, and vary wording.

**Delivery:** Intermediate message text uses the same path as other generative assistant text from the ReAct loop. Rasa checks the output channel’s streaming capabilities: when the output channel supports streaming, the LLM completion runs in streaming mode and each text delta is forwarded immediately to the output channel. When streaming is not supported, Rasa uses a non-streaming completion and sends the full text once. Voice-oriented channels can apply slightly longer silence **after** an intermediate message before the next assistant message so pacing sounds natural.

**Configuration:** Set `enable_filler_messages: false` under `configuration` in the sub agent’s `config.yml` to disable intermediate-message prompt instructions and filler-specific event metadata.

For more context on streaming and channels, see [Messaging and voice channels](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/) and [Speech integrations](https://rasa.com/docs/reference/integrations/speech-integrations/).

## Custom intermediate messages[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#custom-intermediate-messages 'Direct link to Custom intermediate messages')

ReAct sub agents have access to the output channel, so you can send **additional** intermediate messages beyond the default model-driven ones—for example a fixed “Working on it…” line at the start of a turn.

To send custom intermediate messages, override the `send_message` method in your custom ReAct sub agent class:

send\_message override (intermediate messages)

```
async def send_message(    self, agent_input: AgentInput, output_channel: Optional[OutputChannel] = None) -> AgentOutput:    generated_events: list[Event] = []    if output_channel is not None and agent_input.recipient_id is not None:        message = "Starting to work on your request..."        # Send the intermediate message in background without awaiting it        async_task = asyncio.create_task(            output_channel.send_text_message(                recipient_id=agent_input.recipient_id,                text=message,            )        )        # Append BotUttered only if the async send succeeded        def _on_send_done(t: asyncio.Task) -> None:            if not t.exception():                generated_events.append(BotUttered(text=message))        async_task.add_done_callback(_on_send_done)    # Delegate to the parent implementation which contains the main logic    agent_output = await super().send_message(agent_input, output_channel=output_channel)    # Append any generated events to the agent output    agent_output.events = agent_output.events or []    agent_output.events.extend(generated_events)    return agent_output
```

## Resume after interruption[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#resume-after-interruption 'Direct link to Resume after interruption')

On resume, Rasa invokes the ReAct sub agent again with the current conversation context. The default prompt adds a system instruction that the run was interrupted and that the agent may use the new context or ask again. If you use a [custom prompt template](https://rasa.com/docs/reference/config/agents/react-sub-agents/#modifying-the-prompt-template), you can branch on `resumed_after_interruption` and use `resumed_last_request` when you need the last agent message from before the interruption.

## Customization[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customization 'Direct link to Customization')

ReAct sub agents can be customized in two ways:

- [Modifying the Prompt Template](https://rasa.com/docs/reference/config/agents/react-sub-agents/#modifying-the-prompt-template)
- [Customizing the Runtime Engine](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-runtime-engine)

### Modifying the Prompt Template[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#modifying-the-prompt-template 'Direct link to Modifying the Prompt Template')

You can customize the prompt template of the sub agent to:

- Include any logic for how the sub agent should operate.
- Pass extra context via slots. You can mention any slot in the prompt template via the `slots` namespace, for example: `{{ slots.time }}`.
- Access datetime information via the `current_datetime` variable (available when `include_date_time` is enabled). This is a datetime object that you can use with methods like `strftime()`, `time()`, `tzname()`, etc. For example: `{{ current_datetime.strftime("%d %B, %Y") }}`.

The modified prompt template can be specified in the [sub agent's configuration](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration).

#### Task description (general-purpose)[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#task-description-general-purpose 'Direct link to Task description (general-purpose)')

For general-purpose agents, the string in **`agent.description`** in `config.yml` is passed into the default prompt as **`{{ description }}`** under **Primary Task** (or whatever heading you use in a custom template). That string is the main place to define what “done” means for the run (scope, tone, required outcomes).

To change how that text is framed or combined with other context, edit **`configuration.prompt_template`** and adjust the Jinja around `{{ description }}`. You can also pull in slots (for example `{{ slots.my_slot }}`) or datetime context as described above.

#### Default Prompt Templates[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#default-prompt-templates 'Direct link to Default Prompt Templates')

Rasa Pro ships two default Jinja2 templates—`mcp_open_agent_prompt_template.jinja2` for general-purpose agents and `mcp_task_agent_prompt_template.jinja2` for task-specific ones. The tabs below show the complete shipped templates.

When Rasa renders a template, it passes in variables such as `description`, `slot_names` (task-specific only), `current_datetime`, `enable_filler_messages`, `resumed_after_interruption`, `resumed_last_request`, and `restarted`. Do not embed conversation history in the Jinja: Rasa uses the rendered output as the system message and adds recent user and assistant turns as separate chat messages in the same LLM request.

- General-purpose ReAct sub agent
- Task-specific ReAct sub agent

mcp\_open\_agent\_prompt\_template.jinja2

```
You are a helpful assistant that should assist the user in the best possible way.### Primary Task{{ description }}### Instructions* Always make sure to output responses to the user in a clear, helpful format that fits the conversational flow.* Always avoid asking multiple questions at once. Ask questions sequentially one at a time and wait for the user's response before proceeding to next question.* Always avoid making assumptions about what values to pass into tools. Ask for clarification if a user's request is ambiguous.* **Completing the task**: When your primary task is fully done, you MUST call the `task_completed` tool exactly once. You must ALWAYS call `task_completed` together with a message—never call it with empty or missing text. In that same response you MUST include text: a natural, short follow-up to the user's *last* message (e.g. acknowledge their choice, wish them well). Do NOT summarize what you did or what happened in the conversation—reply to what they just said. Both the tool call and the text are required; a response with only the tool call and no message is invalid.* Do NOT call `task_completed` unless you're certain the primary task (NOT slot corrections, clarifications, or follow-up questions) is fully completed. Focus on the task given in the task description above.* Strictly avoid making up information or ability to take some action which is not available in `tool` provided.{% if enable_filler_messages %}### Tool Usage AcknowledgmentsWhen you need to use a tool to assist the user:* ALWAYS include a brief, natural acknowledgment message BEFORE making tool calls.* **Do NOT repeat or paraphrase the user's request**: No "So you want…", "Looking for X as you said…", "Checking your criteria…", or restating their words. The user already knows what they asked. Use only a short action phrase (e.g. "Checking that for you.", "Looking into it.", "One moment—I'll find that.", "Let me pull that up.").* **Vary your phrasing**: Do not start every acknowledgment with the same opener. Rotate between different openings; match the tone to the request when it helps.* Keep acknowledgments brief and direct. AVOID phrases that imply waiting or delays.* The acknowledgment is sent before the tool runs. Your follow-up message must NOT repeat the request or what you did—only deliver the new information.### Follow-up message (after a tool returns)**Rule: Do not repeat anything that was already said.** The user already stated their request; you already acknowledged it. Your follow-up adds only NEW information.* **Never repeat or paraphrase**: (a) the user's request, topic, or criteria, (b) your acknowledgment, or (c) that you "looked up", "searched", or "found" something. Forbidden phrases include: "that match your…", "here are the [X] you asked for", "I found…", "Based on your…", "As you requested…", "You asked for…", "Here's what I found for…".* **Start with a full sentence** that leads into the results (e.g. "Here are some options.", "I've got a few that could work."). Do not start with a fragment ("Options:", "Here they are:") or with a sentence that re-describes what the user wanted.* **Then**: Go straight to the list, data, or next question. Nothing else.* Assume the user remembers their request. Your job is only to deliver the new information.{% endif %}{% if resumed_after_interruption %}### Resume after interruptionYour execution was interrupted.{% if resumed_last_request %} You were asking about: {{ resumed_last_request }}.{% endif %} The context in between might help you to answer the request. If that is the case use that context. If not, ask again.{% endif %}{% if restarted %}### Agent restartedThe following user/assistant messages may include a section marked **"Previous run (completed)."** Those messages are from a run that already finished. Treat the current interaction as a fresh start and collect the required information from the user again.{% endif %}{% if current_datetime %}### Date & Time Context- Current date: {{ current_datetime.strftime("%d %B, %Y") }}   (YYYY-MM-DD)- Current time: {{ current_datetime.strftime("%H:%M:%S") }} ({{ current_datetime.tzname() }})  (HH:MM:SS, 24-hour format with timezone)- Current day: {{ current_datetime.strftime("%A") }}     (Day of week){% endif %}
```

mcp\_task\_agent\_prompt\_template.jinja2

```
You are a helpful assistant that should assist the user in the best possible way.### Description of your capabilities{{ description }}### Task* Use the tools available to gather the required information to set the following slots: {{ slot_names }}.* In order to set the slot values, use the `set_slot_<slot_name>` tool.### Instructions* Always make sure to output responses to the user in a clear, helpful format that fits the conversational flow.* Always avoid asking multiple questions at once. Ask questions sequentially one at a time and wait for the user's response before proceeding to next question.* Always avoid making assumptions about what values to pass into tools. Ask for clarification if a user's request is ambiguous.* Strictly avoid making up information or ability to take some action which is not available in `tool` provided.{% if enable_filler_messages %}### Tool Usage AcknowledgmentsWhen you need to use a tool to assist the user:* ALWAYS include a brief, natural acknowledgment message BEFORE making tool calls.* **Do NOT repeat or paraphrase the user's request**: No "So you want…", "Looking for X as you said…", "Checking your criteria…", or restating their words. The user already knows what they asked. Use only a short action phrase (e.g. "Checking that for you.", "Looking into it.", "One moment—I'll find that.", "Let me pull that up.").* **Vary your phrasing**: Do not start every acknowledgment with the same opener. Rotate between different openings; match the tone to the request when it helps.* Keep acknowledgments brief and direct. AVOID phrases that imply waiting or delays.* The acknowledgment is sent before the tool runs. Your follow-up message must NOT repeat the request or what you did—only deliver the new information.### Follow-up message (after a tool returns)**Rule: Do not repeat anything that was already said.** The user already stated their request; you already acknowledged it. Your follow-up adds only NEW information.* **Never repeat or paraphrase**: (a) the user's request, topic, or criteria, (b) your acknowledgment, or (c) that you "looked up", "searched", or "found" something. Forbidden phrases include: "that match your…", "here are the [X] you asked for", "I found…", "Based on your…", "As you requested…", "You asked for…", "Here's what I found for…".* **Start with a full sentence** that leads into the results (e.g. "Here are some options.", "I've got a few that could work."). Do not start with a fragment ("Options:", "Here they are:") or with a sentence that re-describes what the user wanted.* **Then**: Go straight to the list, data, or next question. Nothing else.* Assume the user remembers their request. Your job is only to deliver the new information.{% endif %}{% if resumed_after_interruption %}### Resume after interruptionYour execution was interrupted.{% if resumed_last_request %} You were asking about: {{ resumed_last_request }}.{% endif %} The context in between might help you to answer the request. If that is the case use that context. If not, ask again.{% endif %}{% if restarted %}### Agent restartedThe following user/assistant messages may include a section marked **"Previous run (completed)."** Those messages are from a run that already finished. Do not set slot values from them. Treat the current interaction as a fresh start and collect the required information from the user again.{% endif %}{% if current_datetime %}### Date & Time Context- Current date: {{ current_datetime.strftime("%d %B, %Y") }}   (YYYY-MM-DD)- Current time: {{ current_datetime.strftime("%H:%M:%S") }} ({{ current_datetime.tzname() }})  (HH:MM:SS, 24-hour format with timezone)- Current day: {{ current_datetime.strftime("%A") }}     (Day of week){% endif %}
```

### Customizing the Runtime Engine[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-runtime-engine 'Direct link to Customizing the Runtime Engine')

Runtime customization for ReAct sub agents — whether general-purpose or task-specific — follows the same pattern. You extend the appropriate abstract base class and override the available customization hooks.

- General-purpose ReAct Sub Agent
- Task-specific ReAct Sub Agent

To create a custom general-purpose ReAct sub agent, inherit from `MCPOpenAgent` and override methods to modify behavior such as tool definitions or input/output processing:

MCPOpenAgent subclass (skeleton)

```
from typing import Any, Dict, Listfrom rasa.agents.protocol.mcp.mcp_open_agent import MCPOpenAgentfrom rasa.agents.schemas import (    AgentInput,    AgentOutput,    AgentToolContext,    AgentToolResult,    AgentToolSchema,)class StockAnalysisAgent(MCPOpenAgent):    def get_custom_tool_definitions() -> List[Dict[str, Any]]:        ...    async def process_input(self, agent_input: AgentInput) -> AgentInput:        ...    async def process_output(self, output: AgentOutput) -> AgentOutput:        ...
```

#### Customizing the `task_completed` tool and empty final messages[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-task_completed-tool-and-empty-final-messages 'Direct link to customizing-the-task_completed-tool-and-empty-final-messages')

**Tool schema and description**

Override the static method `get_task_completed_tool()` to change what the LLM sees (for example stricter or looser closing instructions). Keep the function name `task_completed` and a valid function-calling schema so the completion loop still recognizes the tool. Add a `deepcopy` of the default tool and replace only the description string, for example on the same subclass as your other overrides:

```
from copy import deepcopyfrom typing import Any, Dictfrom rasa.agents.protocol.mcp.mcp_open_agent import MCPOpenAgentclass StockAnalysisAgent(MCPOpenAgent):    @staticmethod    def get_task_completed_tool() -> Dict[str, Any]:        tool = deepcopy(MCPOpenAgent.get_task_completed_tool())        tool["function"]["description"] = (            "Call once when the research task is done. Same rules as default, "            "but end with a one-line offer to help with follow-up questions."        )        return tool    # get_custom_tool_definitions, process_input, process_output, ...
```

**When the model calls `task_completed` with no user-facing text**

If the assistant turn contains a `task_completed` tool call but **no assistant message text**, Rasa does not treat that as the final reply to the user. It appends a **system follow-up instruction** and calls the LLM again **without tools**, asking for a short, natural closing message only. After that text is produced, the run completes as usual.

To change that recovery behavior (tone, length, language), override the static method `get_system_message_for_empty_content_at_task_completion()` and return your own system message dict with the same shape (`role` and `content` keys as in the parent implementation).

```
@staticmethoddef get_system_message_for_empty_content_at_task_completion() -> Dict[str, str]:    return {        "role": "system",        "content": (            "The task is already complete. Reply with one short sentence in "            "formal tone. Do not call tools."        ),    }
```

caution

Do not remove the `task_completed` tool or rename it unless you replace the completion mechanism with equivalent logic; the general-purpose protocol expects this signal to exit the loop.

To create a custom task-specific ReAct sub agent, inherit from `MCPTaskAgent` and override methods to modify behavior such as tool definitions or input/output processing:

MCPTaskAgent subclass (skeleton)

```
from rasa.agents.protocol.mcp.mcp_task_agent import MCPTaskAgentfrom rasa.agents.schemas import (    AgentInput,    AgentOutput,    AgentToolContext,    AgentToolResult,    AgentToolSchema,)class AppointmentBookingAgent(MCPTaskAgent):    def get_custom_tool_definitions() -> List[Dict[str, Any]]:        ...    async def process_input(self, output: AgentInput) -> AgentInput:        ...    async def process_output(self, output: AgentOutput) -> AgentOutput:    ...
```

Once implemented, the custom sub agent class can be referenced directly in the [ReAct sub agent's configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration-file):

config.yml (custom module)

```
# Basic sub agent informationagent:  ...# Core configurationconfiguration:  module: # path to the python class containing the customizations
```

#### Adding custom tools[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#adding-custom-tools 'Direct link to Adding custom tools')

You can add custom tools that are independent of the MCP server but essential for the sub agent to accomplish its task.

To define a custom tool, implement it as a Python function. The executor always receives the LLM tool arguments and an `AgentToolContext` (from `rasa.agents.schemas`) whose `metadata` field is a copy of the current request’s `AgentInput.metadata` (or an empty dict). Use `context.metadata` for backend-only data such as auth or user context; do not rely on the model to populate those keys—they are not part of the tool schema the LLM sees.

Custom tool executor (example)

```
async def _recommend_cars(    self, arguments: Dict[str, Any], context: AgentToolContext) -> AgentToolResult:    tenant_id = context.metadata.get("tenant_id")    # your function implementation using arguments and context.metadata
```

_Note_: The `arguments` parameter contains only the parameters defined on the tool schema. Extract structured values the model should supply from `arguments`; read orchestration metadata from `context.metadata`.

Avoid blocking operations in async tools

Tool executors are asynchronous. Do not use blocking calls (e.g. `time.sleep`, synchronous HTTP clients like `requests`, long-running CPU-bound loops) inside them. Prefer non-blocking alternatives (e.g. `await asyncio.sleep(...)`, `httpx.AsyncClient`, database drivers with async support), or run CPU-bound work in an executor via `asyncio.to_thread`.

The function must always return an instance of `AgentToolResult`, which includes the following fields:

AgentToolResult fields

```
class AgentToolResult(BaseModel):    tool_name: str    result: Optional[str] = None    is_error: bool = False    error_message: Optional[str] = None
```

To add custom tools, implement the `get_custom_tool_definitions` method. This method should return a list of tool definitions, each adhering to the [OpenAI function calling specification](https://platform.openai.com/docs/guides/function-calling#defining-functions). Additionally, each tool definition must include a `tool_executor` key, which should reference the function to execute when the tool is invoked:

Custom tool definition with tool\_executor (example)

```
{    "type": "function",    "function": {        "name": "recommend_cars",        "description": "Analyze search results and return structured car recommendations",        "parameters": {            "type": "object",            "properties": {                "search_results": {                    "type": "string",                    "description": "The search results to analyze for car recommendations",                },                "max_recommendations": {                    "type": "integer",                    "description": "Maximum number of recommendations to return",                    "default": 3,                },            },            "required": ["search_results", "max_recommendations"],            "additionalProperties": False,        },        "strict": True,    },    "tool_executor": self._recommend_cars}
```

#### Processing Input and Output[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#processing-input-and-output 'Direct link to Processing Input and Output')

For information about customizing input and output processing, see the [common customization section](https://rasa.com/docs/reference/config/agents/overview-agents/#input-processing-customization) in the overview page.

## Key Differences Between General-Purpose and Task-Specific ReAct Sub Agents[​](https://rasa.com/docs/reference/config/agents/react-sub-agents/#key-differences-between-general-purpose-and-task-specific-react-sub-agents 'Direct link to Key Differences Between General-Purpose and Task-Specific ReAct Sub Agents')

| Category | General-Purpose | Task-Specific |
| --- | --- | --- |
| **Purpose** | Open-ended exploratory conversations where the agent determines when it is done | Goal-oriented tasks that need to fill specific slots with values |
| **Completion** | Requires calling a `task_completed` tool to finish its execution loop | Automatically completes when exit conditions are met |
| **Exit Conditions** | Explicit signaling of completion via the `task_completed` tool | Completion determined by evaluating the `exit_if` conditions |
| **Built-in Slot Tools** | No automatic slot-setting tools | Automatically gets `set_slot_<slot_name>` tools for each slot mentioned in the exit conditions |
| **Final response** | The model sends agent message (short closing follow-up to the user’s last message) in the same turn as a `task_completed` tool call; the tool has no parameters and only signals that the run is done | Completes silently without sending a response, allowing the flow to continue to the next step |

- [General-purpose ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/#general-purpose-react-sub-agent)
 - [Key Characteristics](https://rasa.com/docs/reference/config/agents/react-sub-agents/#key-characteristics)
 - [The `task_completed` Tool](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-task_completed-tool)
- [Task-specific ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/#task-specific-react-sub-agent)
 - [Key Characteristics](https://rasa.com/docs/reference/config/agents/react-sub-agents/#key-characteristics-1)
 - [The `set_slot_<slot_name>` Tools](https://rasa.com/docs/reference/config/agents/react-sub-agents/#the-set_slot_slot_name-tools)
- [Configuration](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration)
 - [Configuration Section](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration-section)
 - [Connections Section](https://rasa.com/docs/reference/config/agents/react-sub-agents/#connections-section)
- [Intermediate messages](https://rasa.com/docs/reference/config/agents/react-sub-agents/#intermediate-messages)
- [Custom intermediate messages](https://rasa.com/docs/reference/config/agents/react-sub-agents/#custom-intermediate-messages)
- [Resume after interruption](https://rasa.com/docs/reference/config/agents/react-sub-agents/#resume-after-interruption)
- [Customization](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customization)
 - [Modifying the Prompt Template](https://rasa.com/docs/reference/config/agents/react-sub-agents/#modifying-the-prompt-template)
 - [Customizing the Runtime Engine](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-runtime-engine)
- [Key Differences Between General-Purpose and Task-Specific ReAct Sub Agents](https://rasa.com/docs/reference/config/agents/react-sub-agents/#key-differences-between-general-purpose-and-task-specific-react-sub-agents)