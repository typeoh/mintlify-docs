# Source: https://rasa.com/docs/reference/config/agents/external-sub-agents

On this page

New Beta Feature in 3.14

Rasa supports stateful execution of external agents via A2A protocol.

This feature is in a beta (experimental) stage and may change in future Rasa versions. We welcome your feedback on this feature.

External sub agents connected via the [A2A (Agent-to-Agent) protocol](https://a2a-protocol.org/latest/) operate as autonomous entities that can handle complex, multi-turn conversations independently. When invoked through a [`call` step](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps) in your flows, these agents take control of the conversation and interact with users until their task is complete.

## A2A Protocol Connection[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#a2a-protocol-connection 'Direct link to A2A Protocol Connection')

Rasa connects to external sub agents through the A2A (Agent-to-Agent) protocol, which provides a standardized way for different AI agents to communicate and collaborate. Here's how the connection process works:

### Connection Process[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#connection-process 'Direct link to Connection Process')

1. **Agent Card Resolution**: Rasa first retrieves the external sub agent's capabilities by loading its agent card, which can be either:
 
 - A local JSON file path
 - A remote URL pointing to the agent card
2. **Client Initialization**: Rasa creates an A2A client configured with:
 
 - Authentication credentials (if required)
 - Supported transport protocols (JSON-RPC, HTTP JSON, gRPC)
 - Streaming capabilities for real-time communication
 - Timeout and retry settings
3. **Health Check**: Before establishing the connection, Rasa performs a health check by sending a test message to verify that the external sub agent is responsive and properly configured.
 
4. **Message Exchange**: Once connected, Rasa communicates with the external sub agent by:
 
 - Sending user messages and conversation context
 - Receiving responses and structured data
 - Handling task status updates and completion signals

### Request metadata on A2A messages[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#request-metadata-on-a2a-messages 'Direct link to Request metadata on A2A messages')

Rasa forwards the entire `AgentInput.metadata` dictionary on each outgoing A2A user [`Message`](https://a2a-protocol.org/latest/specification/) as `Message.metadata`. That lets your remote agent read orchestration or backend-only data—such as authentication tokens, tenant IDs, or feature flags—without putting those values in the message **parts** that drive the language model.

Rasa also sets `Message.context_id` and `Message.task_id` from the metadata entries with keys `context_id` and `task_id` (see `rasa.agents.constants`: `A2A_AGENT_CONTEXT_ID_KEY` and `A2A_AGENT_TASK_ID_KEY`). Those keys are used for session continuity across turns; you can supply additional keys the same way.

To attach custom metadata, override `process_input` on a custom `A2AAgent` class and merge into `input.metadata` before returning the `AgentInput`. The default metadata already includes values the flow executor maintains (for example sender id, agent id, model id, and exit conditions where applicable).

### Supported Transport Protocols[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#supported-transport-protocols 'Direct link to Supported Transport Protocols')

The A2A protocol supports multiple transport mechanisms:

- **JSON-RPC**: Lightweight remote procedure calls over HTTP
- **HTTP JSON**: Simple HTTP-based JSON messaging
- **gRPC**: High-performance RPC framework with streaming support

During connection, the transport protocol is automatically selected, following the server’s preference when available, and otherwise falling back to the best available protocol.

### Authentication[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#authentication 'Direct link to Authentication')

External sub agents can require authentication for secure communication. Rasa supports various authentication methods including API keys, OAuth 2.0, and pre-issued tokens, which are configured in the external sub agent's [configuration file](https://rasa.com/docs/reference/config/agents/external-sub-agents/#authentication).

### Intermediate Messages[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#intermediate-messages 'Direct link to Intermediate Messages')

External sub agents connected via the A2A protocol can send intermediate messages when Rasa receives task updates with status `"submitted"` or `"working"`. These messages are

- **immediately sent to the user**: Intermediate messages are forwarded to the user as soon as they are received, without waiting for task completion.
- **tracked in conversation history**: Rasa automatically creates bot uttered events for these messages, ensuring they are properly tracked in the tracker store.

important

Ensure that intermediate messages sent by your A2A agent are meaningful and provide value to users. Messages should offer useful updates about task progress, status changes, or relevant information that helps users understand what the agent is doing.

### Background Task Cancellation[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#background-task-cancellation 'Direct link to Background Task Cancellation')

Long-running A2A operations (both polling and streaming) run in the background and are cancelled automatically when the conversation reaches an inactive or terminal state.

Rasa cancels background A2A processing in the following cases:

- `ConversationInactive` is emitted by session timeout while an A2A task is still running.
- `SessionEnded` or `ConversationInactive` is appended through the tracker events API (`POST /conversations/{conversation_id}/tracker/events`, see [Rasa Pro REST API](https://rasa.com/docs/reference/api/pro/http-api/)).
- Closing the Inspector browser tab, which automatically cancels in-flight A2A operations for that conversation.

For details about session timeout and lifecycle events, see [Session Lifecycle](https://rasa.com/docs/reference/config/session-management/session-lifecycle/).

You can also cancel background A2A processing manually:

- From custom code (for example, a [custom connector](https://rasa.com/docs/reference/channels/custom-connectors/)), by calling `Agent.cancel_background_tasks(sender_id)`.
- From external systems, by calling `POST /cancel_background_tasks/<sender_id>` on the REST channel endpoint.

When cancellation is triggered (automatically or manually), the in-flight A2A operation is stopped promptly instead of continuing until polling timeout.

### Best Practices[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#best-practices 'Direct link to Best Practices')

An A2A agent can [respond with a `Task` or a `Message`](https://a2a-protocol.org/dev/topics/key-concepts/#agent-response-task-or-message). `Task`s should be used for long-running operations and `Message`s for immediate responses.

Rasa maps all `Message` objects to `INPUT_REQUIRED` state, meaning the response is forwarded to the user while the external sub agent waits for input and continues running.

**Recommended Approaches:**

- **Task-only**: Use `Task`s for the complete workflow
- **Hybrid**: Use `Task`s for main operations and `Message`s **only** for clarifications

This approach aligns with [Google's recommendations](https://a2a-protocol.org/dev/topics/life-of-a-task/#agent-response-message-or-task) for agent design and provides better control over conversation flow and state management.

## Configuration[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#configuration 'Direct link to Configuration')

External sub agents extend the [basic sub agent configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration) with A2A protocol-specific settings.

In addition to the required `agent` section, external sub agents support the following configuration options:

```
# Basic agent information (required - see overview)agent:  name: car_shopping_agent  protocol: A2A  description: "Helps users shop for cars by connecting them with dealers and facilitating purchases"# A2A-specific configurationconfiguration:  agent_card: ./sub_agents/car_shopping_agent/agent_card.json  # Required: path or URL to agent card  module: "path.to.custom.module"  # Optional: custom module for sub agent customization  max_polling_time: 120  # Optional: max seconds to poll for task completion (default: 60)  polling_initial_delay: 1.0  # Optional: starting delay between polls in seconds (default: 0.5)  # auth: ...  # Optional: see Authentication below
```

### Configuration Parameters[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#configuration-parameters 'Direct link to Configuration Parameters')

The `configuration` section in an external sub agent's `config.yml` must include the required field **`agent_card`**. All other fields in this section are optional.

- **`agent_card` (required)**: Location of the agent card defined by the [A2A protocol](https://a2a-protocol.org/latest/specification/#57-sample-agent-card). It can be:
 
 - A local file path (relative to the project root), or
 - A URL pointing to the agent card JSON
- **`module` (optional)**: Path to a custom Python module for [sub agent customization](https://rasa.com/docs/reference/config/agents/external-sub-agents/#customization), using the same pattern as other sub agents. See [Creating a Custom Sub Agent](https://rasa.com/docs/reference/config/agents/external-sub-agents/#creating-a-custom-sub-agent).
 
- **`max_polling_time` (optional, default: `60`)**: When Rasa polls an A2A task until it reaches a terminal state, this is the maximum total time in seconds to spend in that polling loop. Leave unset to use the default. When set, the value must be greater than zero.
 
 important
 
 If you increase `max_polling_time` for long-running A2A tasks, set `TICKET_LOCK_LIFETIME` to a value greater than or equal to `max_polling_time`. If the lock lifetime is shorter than the A2A polling window, the conversation lock can expire while the previous turn is still processing, and a new user message from the same sender can trigger concurrent tracker writes (race condition). See [Environment Variables](https://rasa.com/docs/reference/config/environment-variables/#backing-services) and [Lock Stores](https://rasa.com/docs/reference/integrations/lock-stores/).
 
- **`polling_initial_delay` (optional, default: `0.5`)**: Starting delay in seconds between polling attempts while the task is not yet terminal; later waits increase with exponential backoff, capped at `5` seconds per polling interval. Leave unset to use the default. When set, the value must be greater than zero.
 
- **`auth` (optional)**: Credentials for connecting to a secured external agent. Supports an API key (Bearer or custom header), OAuth 2.0 client credentials, or a pre-issued token. See [Authentication](https://rasa.com/docs/reference/config/agents/external-sub-agents/#authentication) for examples and for which parameters must reference environment variables.
 

### Authentication[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#authentication-1 'Direct link to Authentication')

External sub agents support multiple authentication methods for connecting to external services:

- **API Key**: Static key attached as `Authorization: Bearer <token>`
- **OAuth 2.0 (Client Credentials)**: Automatic token retrieval with client ID/secret
- **Pre-issued Token**: Direct token usage until expiry

Configure authentication with **`configuration.auth`** in your `config.yml` file. Below are several examples:

- API Key
- API Key (Custom Header)
- OAuth 2.0
- Pre-issued Token

```
configuration:  agent_card: ./sub_agents/shopping_agent/agent_card.json  auth:    api_key: "${API_KEY}"
```

```
configuration:  agent_card: ./sub_agents/shopping_agent/agent_card.json  auth:    api_key: "${API_KEY}"    header_name: "X-API-Key"    header_format: "{key}"
```

```
configuration:  agent_card: ./sub_agents/shopping_agent/agent_card.json  auth:    oauth:      client_id: "${CLIENT_ID}"      client_secret: "${CLIENT_SECRET}"      token_url: "https://auth.company.com/oauth/token"      scope: "read:users"
```

```
configuration:  agent_card: ./sub_agents/shopping_agent/agent_card.json  auth:    token: "${ACCESS_TOKEN}"
```

The `$` syntax is **required** for the following sensitive parameters:

- `api_key`
- `token`
- `client_secret`

This ensures that they are not stored in plain text in your configuration files. For other parameters like `client_id`, using the `$` syntax is optional — you can either reference an environment variable using the `$` syntax or provide the value directly in the configuration.

## Customization[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#customization 'Direct link to Customization')

External sub agents can be customized by implementing a Python interface. This enables you to modify how the external sub agent processes inputs and outputs, allowing for sophisticated data transformation and business logic integration.

Customization is particularly useful when you need to filter slot values to send only relevant information to the external sub agent, extract structured data from sub agent responses and store it in slots for downstream use, apply custom business rules to agent inputs and outputs, or convert between different data formats or schemas.

### Creating a Custom Sub Agent[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#creating-a-custom-sub-agent 'Direct link to Creating a Custom Sub Agent')

To customize an external sub agent, create a Python class that inherits from `A2AAgent` and override the necessary methods. Here is an example:

```
from rasa.agents.protocol.a2a.a2a_agent import A2AAgentfrom rasa.agents.schemas import (    AgentInput,    AgentOutput)class CarShoppingAgent(A2AAgent):    async def process_input(self, input: AgentInput) -> AgentInput:        # Customize input processing        return input    async def process_output(self, output: AgentOutput) -> AgentOutput:        # Customize output processing        return output
```

Make sure to specify your custom external sub agent class in the [external sub agent's configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration-file):

```
agent:  name: car_shopping_agent  protocol: A2A  description: "Helps users shop for cars by connecting them with dealers"configuration:  agent_card: ./sub_agents/car_shopping_agent/agent_card.json  module: "sub_agents.car_shopping_agent.custom_agent.CarShoppingAgent"
```

### Input and Output Processing Customization[​](https://rasa.com/docs/reference/config/agents/external-sub-agents/#input-and-output-processing-customization 'Direct link to Input and Output Processing Customization')

For information about customizing input and output processing, see the [common customization section](https://rasa.com/docs/reference/config/agents/overview-agents/#input-processing-customization) in the overview page.

- [A2A Protocol Connection](https://rasa.com/docs/reference/config/agents/external-sub-agents/#a2a-protocol-connection)
 - [Connection Process](https://rasa.com/docs/reference/config/agents/external-sub-agents/#connection-process)
 - [Request metadata on A2A messages](https://rasa.com/docs/reference/config/agents/external-sub-agents/#request-metadata-on-a2a-messages)
 - [Supported Transport Protocols](https://rasa.com/docs/reference/config/agents/external-sub-agents/#supported-transport-protocols)
 - [Authentication](https://rasa.com/docs/reference/config/agents/external-sub-agents/#authentication)
 - [Intermediate Messages](https://rasa.com/docs/reference/config/agents/external-sub-agents/#intermediate-messages)
 - [Background Task Cancellation](https://rasa.com/docs/reference/config/agents/external-sub-agents/#background-task-cancellation)
 - [Best Practices](https://rasa.com/docs/reference/config/agents/external-sub-agents/#best-practices)
- [Configuration](https://rasa.com/docs/reference/config/agents/external-sub-agents/#configuration)
 - [Configuration Parameters](https://rasa.com/docs/reference/config/agents/external-sub-agents/#configuration-parameters)
 - [Authentication](https://rasa.com/docs/reference/config/agents/external-sub-agents/#authentication-1)
- [Customization](https://rasa.com/docs/reference/config/agents/external-sub-agents/#customization)
 - [Creating a Custom Sub Agent](https://rasa.com/docs/reference/config/agents/external-sub-agents/#creating-a-custom-sub-agent)
 - [Input and Output Processing Customization](https://rasa.com/docs/reference/config/agents/external-sub-agents/#input-and-output-processing-customization)