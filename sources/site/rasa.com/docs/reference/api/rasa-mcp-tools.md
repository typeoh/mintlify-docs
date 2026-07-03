# Source: https://rasa.com/docs/reference/api/rasa-mcp-tools

On this page

Rasa MCP Tools are included in Rasa Pro 3.16 and later.

v3.16

## Cheat Sheet[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#cheat-sheet 'Direct link to Cheat Sheet')

| Tool | Group | What it does |
| --- | --- | --- |
| `search_rasa_documentation` | Documentation | Search official Rasa docs |
| `list_project_flow_definitions` | Introspection | List all flows |
| `list_project_slot_definitions` | Introspection | List all slots |
| `list_project_response_definitions` | Introspection | List all responses |
| `get_flow` | Introspection | Get one flow by ID or name |
| `get_slot` | Introspection | Get one slot by name |
| `get_response` | Introspection | Get one response by name |
| `list_project_custom_actions_in_domain` | Introspection | List custom actions declared in domain |
| `list_custom_action_implementations` | Introspection | List custom action Python classes |
| `list_default_action_names` | Introspection | List built-in Rasa action names |
| `get_flow_schema` | Schemas | Get the flow JSON schema |
| `get_domain_schema` | Schemas | Get the domain YAML schema |
| `get_e2e_schema` | Schemas | Get the E2E test YAML schema |
| `validate_project` | Build | Validate project config and data |
| `train_rasa_assistant` | Build | Train the assistant model |
| `talk_to_assistant` | Runtime | Send messages to the running assistant |
| `get_assistant_logs` | Runtime | Get recent assistant logs |

## CLI Commands[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#cli-commands 'Direct link to CLI Commands')

| Command | Effect |
| --- | --- |
| `rasa tools init` | Interactive setup wizard. Creates `.rasa/tools.yaml` and downloads offline docs and skills. |
| `rasa tools init -y` | Non-interactive setup with all defaults. |
| `rasa tools init --project-path PATH` | Initialize for a project in a different directory. |
| `rasa tools init docs` | Download or refresh offline documentation files. |
| `rasa tools run --mode stdio` | Start the MCP server in stdio mode (for IDE integrations). |
| `rasa tools run --mode http --port 7331` | Start the MCP server in HTTP mode on the specified port. |
| `rasa tools run --config PATH` | Start the server using settings from `.rasa/tools.yaml` in the given directory. |
| `rasa tools run --project-path PATH` | Override the project folder the server reads from. |
| `rasa tools run --rasa-server-url URL` | Override the Rasa server URL (default: `http://localhost:5005`). |

## Documentation tools[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#documentation-tools 'Direct link to Documentation tools')

### `search_rasa_documentation`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#search_rasa_documentation 'Direct link to search_rasa_documentation')

Search the official Rasa documentation for authoritative information. Returns relevant documentation about Rasa concepts, APIs, best practices, configuration, and troubleshooting with links to official docs.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string | yes | The search query to find relevant Rasa documentation entries. |

**Returns:** Matching documentation snippets with source links.

**Sample prompts:**

```
Search the Rasa docs for how to configure slot validation.
```

```
Look up how collect steps work in Rasa flows.
```

## Project introspection tools[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#project-introspection-tools 'Direct link to Project introspection tools')

### `list_project_flow_definitions`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_flow_definitions 'Direct link to list_project_flow_definitions')

List all flow definitions in the project. Returns flow ID, name, and file path for each flow.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `data_folder` | string | no | `"data"` | Folder containing flow YAML files, relative to project root. |

**Returns:** List of flows with `id`, `name`, and `file_path`.

**Sample prompt:**

```
List all flows in this project.
```

### `list_project_slot_definitions`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_slot_definitions 'Direct link to list_project_slot_definitions')

List all slot definitions from the project domain file(s). Returns slot name, type, and file path.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `domain_folder` | string | no | `"domain"` | Folder containing domain YAML files, relative to project root. |

**Returns:** List of slots with `name`, `type`, and `file_path`.

**Sample prompt:**

```
List all slots in this project and their types.
```

### `list_project_response_definitions`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_response_definitions 'Direct link to list_project_response_definitions')

List all response (utterance) definitions from the project domain file(s). Returns response name and file path.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `domain_folder` | string | no | `"domain"` | Folder containing domain YAML files, relative to project root. |

**Returns:** List of responses with `name` and `file_path`.

**Sample prompt:**

```
Show me all responses defined in this project.
```

### `get_flow`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_flow 'Direct link to get_flow')

Get a single flow by flow ID (YAML key) or flow name. Returns flow metadata and full definition including steps, triggers, and branching logic.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `flow_id` | string | yes | — | Flow ID (YAML key) or human-readable flow name. |
| `data_folder` | string | no | `"data"` | Folder containing flow YAML files. |

**Returns:** Full flow definition with metadata.

**Sample prompt:**

```
Get the transfer_money flow and explain its steps.
```

### `get_slot`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_slot 'Direct link to get_slot')

Get a single slot by name. Returns slot metadata and full definition from the domain.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `slot_name` | string | yes | — | Slot name. |
| `domain_folder` | string | no | `"domain"` | Folder containing domain YAML files. |

**Returns:** Full slot definition with type, mappings, and metadata.

**Sample prompt:**

```
Get the recipient slot definition and explain its mappings.
```

### `get_response`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_response 'Direct link to get_response')

Get a single response (utterance) by name. Returns response metadata and full definition including text, images, buttons, and custom payloads.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `response_name` | string | yes | — | Response name (e.g. `utter_greet`). |
| `domain_folder` | string | no | `"domain"` | Folder containing domain YAML files. |

**Returns:** Full response definition with all variations.

**Sample prompt:**

```
Get the utter_ask_recipient response.
```

### `list_project_custom_actions_in_domain`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_custom_actions_in_domain 'Direct link to list_project_custom_actions_in_domain')

List all custom actions declared in the domain file(s). Returns action name and file path where the action is registered.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `domain_folder` | string | no | `"domain"` | Folder containing domain YAML files. |

**Returns:** List of custom action names with `name` and `file_path`. This lists domain declarations, not Python implementations.

**Sample prompt:**

```
What custom actions are declared in the domain?
```

### `list_custom_action_implementations`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_custom_action_implementations 'Direct link to list_custom_action_implementations')

List all custom action Python implementations in the project. Returns action name, class name, and file path for each action.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `actions_folder` | string | no | auto-detected | Path to the actions folder relative to project root. Auto-detects from `endpoints.yml` or defaults to `"actions"`. |

**Returns:** List of action implementations with `action_name`, `class_name`, and `file_path`.

**Notes:**

- If `error` is set: the actions folder was not found. Check `endpoints.yml` or specify the folder.
- If `count=0` and no error: the actions folder exists but contains no action classes.

**Sample prompt:**

```
List all custom action implementations and their file locations.
```

### `list_default_action_names`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_default_action_names 'Direct link to list_default_action_names')

List all built-in default action names provided by Rasa. These actions are available without configuration and can be overridden.

**Parameters:** None.

**Returns:** List of default action name strings.

**Sample prompt:**

```
What are the default built-in Rasa actions?
```

## Schema tools[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#schema-tools 'Direct link to Schema tools')

### `get_flow_schema`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_flow_schema 'Direct link to get_flow_schema')

Get the official Rasa flow schema in JSON Schema format. Use this to validate flow YAML or generate new flows.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `calm_only` | boolean | no | `true` | Return only CALM-related properties, excluding NLU-specific fields. |

**Returns:** JSON Schema document describing flow structure: name, description, step types, branching logic (`if`/`then`/`else`), collect steps, flow guards, and more.

**Sample prompt:**

```
Get the flow schema so I can write a valid new flow.
```

### `get_domain_schema`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_domain_schema 'Direct link to get_domain_schema')

Get the official Rasa domain schema in YAML schema format. Use this to validate domain YAML or generate domain files.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `calm_only` | boolean | no | `true` | Return only CALM-related properties, excluding NLU-specific fields. |

**Returns:** YAML schema document describing domain structure: slots, custom actions, responses, and more.

**Sample prompt:**

```
Get the domain schema. I need to add a new boolean slot correctly.
```

### `get_e2e_schema`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_e2e_schema 'Direct link to get_e2e_schema')

Get the official Rasa E2E test schema in YAML schema format. Use this to validate or generate end-to-end test files.

**Parameters:** None.

**Returns:** YAML schema document describing E2E test structure: test cases, steps (user and bot messages), fixtures, metadata, stub custom actions, and assertions.

**Sample prompt:**

```
Get the E2E test schema, then write tests for the transfer_money flow.
```

## Build and validation tools[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#build-and-validation-tools 'Direct link to Build and validation tools')

### `validate_project`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#validate_project 'Direct link to validate_project')

Validate the assistant project configuration and training data. Runs comprehensive checks on domain, flows, config, and training data.

**Parameters:** None (reads from the configured project folder).

**Returns:** Pass/fail status with a list of errors and warnings.

**Notes:**

- Can take 60+ seconds for large projects.
- Always run this after making changes and before training.

**Sample prompts:**

```
Validate this project and tell me what's wrong.
```

```
I made changes to the domain. Validate and fix any issues.
```

### `train_rasa_assistant`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#train_rasa_assistant 'Direct link to train_rasa_assistant')

Train the Rasa assistant with the current project configuration. Creates a new model in the `models/` directory.

**Parameters:** None (reads from the configured project folder).

**Returns:** Training status (success/failure), model path, and any errors.

**Notes:**

- Can take several minutes for large projects.
- Only call this after validation passes.
- Each training run produces a new timestamped model file.

**Sample prompts:**

```
Train the assistant.
```

```
Validate and then train. If validation fails, fix the issues first.
```

## Runtime testing and debugging tools[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#runtime-testing-and-debugging-tools 'Direct link to Runtime testing and debugging tools')

### `talk_to_assistant`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#talk_to_assistant 'Direct link to talk_to_assistant')

Test the assistant by sending a sequence of messages and verifying responses. Creates a new conversation for each call.

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `messages` | list\[string\] | yes | — | List of user messages to send in sequence. Each message is sent after the assistant responds to the previous one. |
| `rasa_server_url` | string | no | `http://localhost:5005` | Override the Rasa server URL. Leave empty to use the configured default. |

**Returns:** Structured response with:

- Conversation history (user messages and assistant responses)
- Tracker context showing conversation state, active flows, and slot values

**Prerequisites:** The Rasa assistant must be running (`rasa run` or `rasa inspect`).

**Sample prompts:**

```
Talk to the assistant: "I want to send money", "to Jen", "50 dollars", "yes".Verify the transfer_money flow completes successfully.
```

```
Test the happy path: send "hello" then "I need to transfer money to Bob".Check which flow was triggered and what slots were filled.
```

### `get_assistant_logs`[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_assistant_logs 'Direct link to get_assistant_logs')

Get recent log entries from the Rasa assistant for troubleshooting.

**Parameters:** None.

**Returns:** Recent log entries as text.

**Sample prompts:**

```
Get the assistant logs and explain why the last conversation failed.
```

```
Show me the assistant logs. I think a custom action is throwing an error.
```

## Typical workflow[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#typical-workflow 'Direct link to Typical workflow')

A common sequence for building a feature end-to-end:

1. **Discover** — `list_project_flow_definitions`, `list_project_slot_definitions`, `list_project_response_definitions`
2. **Understand schemas** — `get_flow_schema`, `get_domain_schema`, `get_e2e_schema`
3. **Implement** — write flows, domain entries, and custom actions
4. **Validate** — `validate_project`
5. **Train** — `train_rasa_assistant`
6. **Test** — `talk_to_assistant`
7. **Debug** — `get_assistant_logs` if behavior is unexpected

## Related[​](https://rasa.com/docs/reference/api/rasa-mcp-tools/#related 'Direct link to Related')

- [Rasa MCP Tools](https://rasa.com/docs/pro/installation/rasa-mcp-tools/) — setup and client configuration
- [Command Line Interface](https://rasa.com/docs/reference/api/command-line-interface/) — Rasa CLI reference
- [Prompt-Driven Agent Tutorial](https://rasa.com/docs/learn/ai-assisted-development/) — build features with prompts

- [Cheat Sheet](https://rasa.com/docs/reference/api/rasa-mcp-tools/#cheat-sheet)
- [CLI Commands](https://rasa.com/docs/reference/api/rasa-mcp-tools/#cli-commands)
- [Documentation tools](https://rasa.com/docs/reference/api/rasa-mcp-tools/#documentation-tools)
 - [`search_rasa_documentation`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#search_rasa_documentation)
- [Project introspection tools](https://rasa.com/docs/reference/api/rasa-mcp-tools/#project-introspection-tools)
 - [`list_project_flow_definitions`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_flow_definitions)
 - [`list_project_slot_definitions`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_slot_definitions)
 - [`list_project_response_definitions`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_response_definitions)
 - [`get_flow`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_flow)
 - [`get_slot`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_slot)
 - [`get_response`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_response)
 - [`list_project_custom_actions_in_domain`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_project_custom_actions_in_domain)
 - [`list_custom_action_implementations`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_custom_action_implementations)
 - [`list_default_action_names`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#list_default_action_names)
- [Schema tools](https://rasa.com/docs/reference/api/rasa-mcp-tools/#schema-tools)
 - [`get_flow_schema`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_flow_schema)
 - [`get_domain_schema`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_domain_schema)
 - [`get_e2e_schema`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_e2e_schema)
- [Build and validation tools](https://rasa.com/docs/reference/api/rasa-mcp-tools/#build-and-validation-tools)
 - [`validate_project`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#validate_project)
 - [`train_rasa_assistant`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#train_rasa_assistant)
- [Runtime testing and debugging tools](https://rasa.com/docs/reference/api/rasa-mcp-tools/#runtime-testing-and-debugging-tools)
 - [`talk_to_assistant`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#talk_to_assistant)
 - [`get_assistant_logs`](https://rasa.com/docs/reference/api/rasa-mcp-tools/#get_assistant_logs)
- [Typical workflow](https://rasa.com/docs/reference/api/rasa-mcp-tools/#typical-workflow)
- [Related](https://rasa.com/docs/reference/api/rasa-mcp-tools/#related)