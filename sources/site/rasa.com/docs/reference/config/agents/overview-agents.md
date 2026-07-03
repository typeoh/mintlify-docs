# Source: https://rasa.com/docs/reference/config/agents/overview-agents

On this page

info

Sub Agents are currently in **beta** and are available starting from **Rasa 3.14.0**.

Rasa can serve as an intelligent orchestrator, coordinating a network of multiple sub agents to handle different tasks. Two types of sub agents are currently supported:

1. [ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/): A built-in autonomous sub agent that has access to one or more [MCP (Model Context Protocol)](https://modelcontextprotocol.io/docs/getting-started/intro) servers. It operates in a ReAct loop, dynamically choosing which tools to invoke based on the conversation context.
2. [External Sub Agent](https://rasa.com/docs/reference/config/agents/external-sub-agents/): An external sub agent connected via the [A2A (Agent-to-Agent)](https://a2a-protocol.org/latest/) protocol.

## How Rasa Interacts with Sub Agents[​](https://rasa.com/docs/reference/config/agents/overview-agents/#how-rasa-interacts-with-sub-agents 'Direct link to How Rasa Interacts with Sub Agents')

Sub agents are always invoked as part of a flow execution. When a user triggers a flow that contains an [autonomous step](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps), Rasa orchestrates the sub agent interaction through a detailed process:

1. **Agent State Check**: The system checks if the sub agent is already running and in an interrupted state, resuming it if necessary.
 
2. **Agent Invocation**: Rasa prepares comprehensive context data (see [Context Sharing](https://rasa.com/docs/reference/config/agents/overview-agents/#context-sharing) below) and invokes the sub agent sharing the created context with the sub agent.
 
3. **Retry Logic**: If the sub agent encounters recoverable errors, Rasa automatically retries up to 3 times with exponential backoff.
 
4. **Response Handling**: Based on the sub agent's response status, Rasa takes different actions:
 
 - _INPUT\_REQUIRED_: Response to the user with the sub agent's message and pauses the flow to wait for user input
 - _COMPLETED_: Response to the user and continues to the next flow step
 - _FATAL\_ERROR_: Cancels the current flow and triggers error handling
5. **State Management**: The system maintains sub agent state for proper resumption and cleanup, including handling interruptions when users digress to other flows or use conversation repair. When interrupted, the orchestrator:
 
 - Pauses the sub agent's execution
 - Stores its current state and context
 - Allows the new flow to proceed
 - Offers to resume the interrupted sub agent when the digression is complete
6. **Event Integration**: Any slot updates or events returned by the sub agent are integrated back into the conversation state.
 

The specific completion mechanism depends on the sub agent type. ReAct sub agents use built-in tools to signal completion, while external sub agents signal completion through their own protocols. See [ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/) and [External Sub Agent](https://rasa.com/docs/reference/config/agents/external-sub-agents/) for more details.

This orchestration ensures robust, stateful interactions between the Rasa agent and its sub agents, with proper error handling and context preservation throughout the conversation.

### Context Sharing[​](https://rasa.com/docs/reference/config/agents/overview-agents/#context-sharing 'Direct link to Context Sharing')

To ensure sub agents have the information needed to perform their tasks effectively, Rasa shares comprehensive context with each sub agent:

- **Current user message**: The latest user input that triggered the sub agent
- **Conversation history**: A readable transcript of the entire conversation up to that point
- **Slot values**: All current slot values from the conversation, filtered to exclude system slots and include only relevant data
- **Event history**: The complete sequence of events that have occurred in the conversation
- **Agent metadata** (`AgentInput.metadata`): A dictionary Rasa fills with orchestration context (for example conversation identifiers, sender and model IDs, exit conditions for task-specific agents, and A2A `context_id` / `task_id` when resuming an external agent). You can add or adjust entries in [`process_input`](https://rasa.com/docs/reference/config/agents/overview-agents/#input-processing-customization). Values in this dictionary are **not** exposed to the LLM as user-visible message text; they are intended for your agent implementation and backends.

How metadata is consumed depends on the sub agent protocol:

- **External (A2A) agents**: The full `AgentInput.metadata` map is attached to the A2A [`Message`](https://a2a-protocol.org/latest/specification/) sent to the remote agent (`Message.metadata`), alongside `context_id` and `task_id` fields derived from the well-known keys `context_id` and `task_id` in that same map. Only the message **parts** (user text, slots payload, and so on) are shaped for the model; arbitrary metadata keys stay on the message for your A2A server to read.
- **ReAct (RASA) agents**:
 - **Custom Python tools** receive the agent metadata as [`AgentToolContext.metadata`](https://rasa.com/docs/reference/config/agents/react-sub-agents/#adding-custom-tools) (second argument to the tool executor).
 - **Remote MCP tools** can receive a separate `_meta` payload configured per MCP server with [`meta_map`](https://rasa.com/docs/reference/integrations/mcp-servers/#passing-metadata-to-tools-meta_map) in `endpoints.yml` (static values and/or values mapped from slots).

This rich context allows sub agents to understand the conversation flow, access relevant information, and make informed decisions about how to proceed.

**Bidirectional Data Flow**: Sub agents can also provide structured results alongside their response messages. These structured results can be converted into slot updates (via [customization](https://rasa.com/docs/reference/config/agents/overview-agents/#customization)) and integrated back into the conversation state, allowing the orchestrator to access and use the data collected by the sub agent in subsequent flow steps.

The specific input shared with sub agents and how their output is processed can be customized. See [Customization](https://rasa.com/docs/reference/config/agents/overview-agents/#customization) for details.

### Intermediate Messages[​](https://rasa.com/docs/reference/config/agents/overview-agents/#intermediate-messages 'Direct link to Intermediate Messages')

Sub agents can send intermediate messages to users during task execution, providing real-time updates and feedback.

The behavior of intermediate messages differs by sub agent type:

- **External sub agents**: Rasa sends intermediate messages when task updates arrive with status `"submitted"` or `"working"`.
- **ReAct sub agents**: By default, Rasa prompts the model for intermediate messages before tool execution when `enable_filler_messages` is `true` (the default). Those intermediate messages are **currently** tool acknowledgements (short assistant text in the same LLM response as tool calls). Rasa **streams** the text when the output channel’s supports streaming. You can turn that off or add more messaging via customization. See [Intermediate messages](https://rasa.com/docs/reference/config/agents/react-sub-agents/#intermediate-messages) and [Custom intermediate messages](https://rasa.com/docs/reference/config/agents/react-sub-agents/#custom-intermediate-messages).

For full details, see [ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/#intermediate-messages) and [External Sub Agent](https://rasa.com/docs/reference/config/agents/external-sub-agents/#intermediate-messages).

caution

User requests sent while a sub agent is still processing cannot be handled right now. This limitation applies to both external and ReAct sub agents. Users must wait for the sub agent to complete its task or reach an `INPUT_REQUIRED` state before their next request can be processed.

## How to Use Sub Agents[​](https://rasa.com/docs/reference/config/agents/overview-agents/#how-to-use-sub-agents 'Direct link to How to Use Sub Agents')

To use sub agents in your assistant, invoke them from your [flow steps](https://rasa.com/docs/reference/primitives/flow-steps/) using **autonomous steps**. An autonomous step delegates control to a sub agent for a specific part of the conversation, allowing it to reason independently using tools (such as MCP servers) or by connecting with an external agent via the A2A protocol.

See [Flow Steps: Autonomous Steps](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps) for details on how to configure and use this feature in your flows.

## Configuration[​](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration 'Direct link to Configuration')

All sub agents share common configuration requirements that must be set up before they can be used in your flows.

### Sub Agent Directory Structure[​](https://rasa.com/docs/reference/config/agents/overview-agents/#sub-agent-directory-structure 'Direct link to Sub Agent Directory Structure')

Each sub agent must be configured in its own dedicated subdirectory within your project:

```
your_project/├── config.yml├── domain/├── data/flows/└── sub_agents/    └── your_agent_name/        ├── config.yml        └── [additional files as needed]
```

By default, Rasa looks for a `sub_agents` directory. To use a different directory name, specify the sub agent directory via the CLI argument `--sub-agents`.

### Configuration File[​](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration-file 'Direct link to Configuration File')

Every sub agent must have a `config.yml` file in its directory with the following mandatory structure:

```
agent:  name: your_agent_name  protocol: [A2A|RASA]  # Optional: protocol for agent connections, default 'RASA'  description: "Brief description of what this agent does"configuration:  module: "path.to.custom.module"  # Optional: custom module for sub agent customization
```

#### Required Configuration Keys[​](https://rasa.com/docs/reference/config/agents/overview-agents/#required-configuration-keys 'Direct link to Required Configuration Keys')

The following keys are required in every sub agent's `config.yml`:

- **`agent.name`**: The name of the agent (must be unique and must not clash with any flow name)
- **`agent.description`**: A brief description of the sub agent's capabilities

#### Optional Configuration Keys[​](https://rasa.com/docs/reference/config/agents/overview-agents/#optional-configuration-keys 'Direct link to Optional Configuration Keys')

The following common configuration keys are optional in every sub agent's `config.yml`:

- **`agent.protocol`**: Determines the protocol used for connections:
 
 - `A2A` for external sub agents
 - `RASA` for ReAct sub agents (default)
 
 _Note:_ If you want to use an external sub agent, make sure to set `agent.protocol` to `A2A`.
 
- **`configuration.module`**: Path to a custom module for [sub agent customization](https://rasa.com/docs/reference/config/agents/overview-agents/#customization).
 

### Protocol-Specific Configuration[​](https://rasa.com/docs/reference/config/agents/overview-agents/#protocol-specific-configuration 'Direct link to Protocol-Specific Configuration')

The `config.yml` file contains additional settings depending on the sub agent type:

- **External Sub Agents (A2A)**: Require an `agent_card` path or URL.
- **ReAct Sub Agents (RASA)**: Support LLM configuration, prompt templates, timeouts, and MCP server connections.

For detailed configuration options specific to each sub agent type, see:

- [External Sub Agent Configuration](https://rasa.com/docs/reference/config/agents/external-sub-agents/#configuration)
- [ReAct Sub Agent Configuration](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration)

## Customization[​](https://rasa.com/docs/reference/config/agents/overview-agents/#customization 'Direct link to Customization')

How Rasa interacts with sub agents can be customized to fit your use case. To customize sub agents, you first need to create a custom sub agent class that inherits from the appropriate base class and overrides the necessary methods.

### Creating a Custom Sub Agent[​](https://rasa.com/docs/reference/config/agents/overview-agents/#creating-a-custom-sub-agent 'Direct link to Creating a Custom Sub Agent')

The specific base class depends on the sub agent type:

- **ReAct Sub Agents**: Inherit from `MCPOpenAgent` (general-purpose) or `MCPTaskAgent` (task-specific)
- **External Sub Agents**: Inherit from `A2AAgent`

For detailed implementation examples, see:

- [ReAct Sub Agent Customization](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customizing-the-runtime-engine)
- [External Sub Agent Customization](https://rasa.com/docs/reference/config/agents/external-sub-agents/#creating-a-custom-sub-agent)

### Input Processing Customization[​](https://rasa.com/docs/reference/config/agents/overview-agents/#input-processing-customization 'Direct link to Input Processing Customization')

Override the `process_input` method to customize how the sub agent receives and processes user input. A common use case is to filter slots so that the sub agent only receives relevant information, preventing it from being overwhelmed with unnecessary data.

The `process_input` method receives an `AgentInput` object as input:

```
class AgentInput(BaseModel):    """A class that represents the schema of the input to the agent."""    id: str  # unique identifier for the agent input    user_message: str  # the message sent by the user    slots: List[AgentInputSlot]  # list of slots containing information extracted during the conversation    conversation_history: str  # full conversation dialogue history as a string    events: List[Event]  # list of events (e.g., SlotSet events)    metadata: Dict[str, Any]  # additional custom metadata provided with the input    timestamp: Optional[str] = None  # optional timestamp indicating when the input was createdclass AgentInputSlot(BaseModel):    """A class that represents the schema of the input slot to the agent."""    name: str  # name of the slot    value: Any  # value assigned to the slot    type: str  # type of the slot (e.g., text, float, categorical)    allowed_values: Optional[List[Any]] = None  # optional list of allowed values for the slot, if applicable
```

The function should return an object of type `AgentInput`, which means you can modify the input that will be received by the sub agent.

### Output Processing Customization[​](https://rasa.com/docs/reference/config/agents/overview-agents/#output-processing-customization 'Direct link to Output Processing Customization')

Override the `process_output` method to customize how the sub agent's responses are processed and integrated back into your system. A common use case is to extract structured data from the sub agent's response and store it in slots for use by downstream flows.

The `process_output` method receives an `AgentOutput` object as input:

```
class AgentOutput(BaseModel):    """A class that represents the schema of the output from the sub agent."""    id: str  # unique identifier for the sub agent execution session    status: AgentStatus  # current status of the sub agent (e.g., running, completed, failed)    response_message: Optional[str] = None  # contains the response generated by the sub agent to be sent back to the user    events: Optional[List[SlotSet]] = None  # any Rasa events like `SlotSet` events that should be executed on the tracker once the sub agent releases control    structured_results: Optional[List[List[Dict[str, Any]]]] = None  # list of results returned by all tool invocations while the sub agent was active    metadata: Optional[Dict[str, Any]] = None  # additional metadata about the sub agent's execution that can be used for logging or debugging    timestamp: Optional[str] = None  # timestamp indicating when the sub agent completed its execution    error_message: Optional[str] = None  # contains any error messages if the sub agent encountered issues during execution
```

The function should return an object of type `AgentOutput`, which means you can modify the output created by the sub agent with more enriched information.

For more information about the `SlotSet` event, see the [SlotSet documentation](https://rasa.com/docs/reference/primitives/slots/#custom-slot-mappings).

### Protocol-Specific Customization[​](https://rasa.com/docs/reference/config/agents/overview-agents/#protocol-specific-customization 'Direct link to Protocol-Specific Customization')

For additional customization options specific to each sub agent type, see:

- [ReAct Sub Agent Customization](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customization)
- [External Sub Agent Customization](https://rasa.com/docs/reference/config/agents/external-sub-agents/#customization)

- [How Rasa Interacts with Sub Agents](https://rasa.com/docs/reference/config/agents/overview-agents/#how-rasa-interacts-with-sub-agents)
 - [Context Sharing](https://rasa.com/docs/reference/config/agents/overview-agents/#context-sharing)
 - [Intermediate Messages](https://rasa.com/docs/reference/config/agents/overview-agents/#intermediate-messages)
- [How to Use Sub Agents](https://rasa.com/docs/reference/config/agents/overview-agents/#how-to-use-sub-agents)
- [Configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration)
 - [Sub Agent Directory Structure](https://rasa.com/docs/reference/config/agents/overview-agents/#sub-agent-directory-structure)
 - [Configuration File](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration-file)
 - [Protocol-Specific Configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#protocol-specific-configuration)
- [Customization](https://rasa.com/docs/reference/config/agents/overview-agents/#customization)
 - [Creating a Custom Sub Agent](https://rasa.com/docs/reference/config/agents/overview-agents/#creating-a-custom-sub-agent)
 - [Input Processing Customization](https://rasa.com/docs/reference/config/agents/overview-agents/#input-processing-customization)
 - [Output Processing Customization](https://rasa.com/docs/reference/config/agents/overview-agents/#output-processing-customization)
 - [Protocol-Specific Customization](https://rasa.com/docs/reference/config/agents/overview-agents/#protocol-specific-customization)