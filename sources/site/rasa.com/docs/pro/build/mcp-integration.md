# Source: https://rasa.com/docs/pro/build/mcp-integration

On this page

New Beta Feature in 3.14

Rasa supports native integration of MCP servers.

This feature is in a beta (experimental) stage and may change in future Rasa versions. We welcome your feedback on this feature.

## Overview[​](https://rasa.com/docs/pro/build/mcp-integration/#overview 'Direct link to Overview')

Rasa integrates with [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers to connect your agent to external APIs, databases, and other services. MCP servers expose tools that your agent can [use directly in flows](https://rasa.com/docs/pro/build/mcp-integration/#using-an-mcp-tool-in-a-flow) or provide to [ReAct style sub agents](https://rasa.com/docs/pro/build/mcp-integration/#dynamic-selection-of-tools-in-an-autonomous-step) for dynamic decision-making.

## Defining an MCP Server[​](https://rasa.com/docs/pro/build/mcp-integration/#defining-an-mcp-server 'Direct link to Defining an MCP Server')

Define your MCP servers in the `endpoints.yml` file:

endpoints.yml

```
mcp_servers:  - name: trade_server    url: http://localhost:8080    type: http
```

For detailed information on available authentication methods and other parameters, head over to the [reference page](https://rasa.com/docs/reference/integrations/mcp-servers/).

To send server-side context on every tool call without exposing it to the LLM, configure optional [`meta_map`](https://rasa.com/docs/reference/integrations/mcp-servers/#passing-metadata-to-tools-meta_map) on the MCP server in `endpoints.yml`.

## Using an MCP tool in a flow[​](https://rasa.com/docs/pro/build/mcp-integration/#using-an-mcp-tool-in-a-flow 'Direct link to Using an MCP tool in a flow')

Use MCP tools directly in flows with the [`call` step](https://rasa.com/docs/reference/primitives/flow-steps/#calling-an-mcp-tool), specifying input/output mappings:

flows.yml

```
flows:  buy_order:    description: helps users place a buy order for a particular stock    steps:    - collect: stock_name    - collect: order_quantity    - action: check_feasibility      next:        - if: slots.order_feasible is True          then:            - call: place_buy_order  # MCP tool name              mcp_server: trade_server  # MCP server where tool is available              mapping:                input:                - param: ticker_symbol  # tool parameter name                  slot: stock_name      # slot to send as value                - param: quantity                  slot: order_quantity                output:                - slot: order_status     # slot to store results                  value: result.structuredContent.order_status.success        - else:            - action: utter_invalid_order              next: END
```

This way of directly invoking tools from flows avoids having to write any [custom action code](https://rasa.com/docs/pro/build/custom-actions/) just for the purpose of integrating with external APIs.

### Tool Results and Output Handling[​](https://rasa.com/docs/pro/build/mcp-integration/#tool-results-and-output-handling 'Direct link to Tool Results and Output Handling')

MCP tools return results in two possible formats:

#### Structured Content[​](https://rasa.com/docs/pro/build/mcp-integration/#structured-content 'Direct link to Structured Content')

When tools provide an output schema (see [for example](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema)), you get structured data as output:

```
{  "result": {    "structuredContent": {      "order_status": {"success": true, "order_id": "bcde786f1"}    }  }}
```

In such a case, specific values from the resulting dictionary can be accessed using the dot notation:

```
  - call: place_buy_order    mcp_server: trade_server    ...    output:    - slot: order_status      value: result.structuredContent.order_status.success
```

#### Unstructured Content[​](https://rasa.com/docs/pro/build/mcp-integration/#unstructured-content 'Direct link to Unstructured Content')

When the invoked tool has no output schema is defined, the entire output is captured as a serialized string:

```
{  "result": {    "content": [      {        "type": "text",        "text": "{\"order_status\": {\"success\": true, \"order_id\": \"bcde786f1\"}"      }    ],  }}
```

You can still use the dot notation to capture the output but the complete serialized string will have to be captured in the slot.

```
output:- slot: order_data  value: result.content
```

## Dynamic selection of tools in an autonomous step[​](https://rasa.com/docs/pro/build/mcp-integration/#dynamic-selection-of-tools-in-an-autonomous-step 'Direct link to Dynamic selection of tools in an autonomous step')

Flows can also contain [**autonomous** steps](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps), where the business logic is dynamically figured out at runtime, based on the available context and tools from the MCP server. In order to do so, Rasa requires creation of [ReAct sub agents](https://rasa.com/docs/reference/config/agents/react-sub-agents/).

### Sub agent Configuration[​](https://rasa.com/docs/pro/build/mcp-integration/#sub-agent-configuration 'Direct link to Sub agent Configuration')

Each sub agent which has access to MCP tools operates in a [ReAct](https://arxiv.org/abs/2210.03629) loop, determining which tools to call based on the conversation context.

To create a sub agent, add a folder specific to that agent in the `sub_agents/` directory of your rasa agent:

```
your_project/├── config.yml├── domain/├── data/flows/└── sub_agents/    └── stock_explorer/        ├── config.yml        └── prompt_template.jinja2  # optional
```

Configure the agent in `sub_agents/stock_explorer/config.yml`:

sub\_agents/stock\_explorer/config.yml

```
# Basic agent informationagent:  name: stock_explorer  description: "Agent that helps users research and analyze stock options"# MCP server connectionsconnections:  mcp_servers:  - name: trade_server    include_tools:   # optional - specify which tools to include    - find_symbol    - get_company_news    - apply_technical_analysis    - fetch_live_price    exclude_tools:   # optional - tools to exclude    - place_buy_order    - view_positions
```

Add a `configuration` block when you need LLM settings, [intermediate messages](https://rasa.com/docs/reference/config/agents/react-sub-agents/#intermediate-messages) (enabled by default for ReAct; implemented as tool acknowledgements), or other options.

More details on available configuration parameters can be found in the [reference section](https://rasa.com/docs/reference/config/agents/react-sub-agents/#configuration).

### Invoking a sub agent[​](https://rasa.com/docs/pro/build/mcp-integration/#invoking-a-sub-agent 'Direct link to Invoking a sub agent')

A sub agent can be invoked from a flow using the [`call` step](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps):

flows.yml

```
flows:  stock_investment_research:    description: helps research and analyze stock investment options    steps:    - call: stock_explorer  # runs until agent signals completion
```

A sub agent remains active until it signals a completion by itself or it meets a defined criteria, for e.g. -

flows.yml

```
flows:  stock_investment_research:    description: helps research and analyze stock investment options    steps:    - call: stock_explorer      exit_if:        - slots.user_satisfied is True
```

Here, `stock_explorer` agent will keep running until the `user_satisfied` slot is not set to `True`.

To read more details about the runtime execution of a ReAct style sub agent inside Rasa, head over to the [reference documentation](https://rasa.com/docs/reference/config/agents/overview-agents/#how-rasa-interacts-with-sub-agents).

### Selective Tool Access[​](https://rasa.com/docs/pro/build/mcp-integration/#selective-tool-access 'Direct link to Selective Tool Access')

You can have fine-grained control over the specific MCP tools a sub agent can access by using the `include_tools` and `exclude_tools` properties in the agent configuration:

sub\_agents/restricted\_agent/config.yml

```
connections:  mcp_servers:  - name: trade_server    include_tools:     # only allow specific tools    - find_symbol    - get_company_news    exclude_tools:     # explicitly block dangerous operations    - place_buy_order    - delete_account  - name: analytics_server    exclude_tools:     # block admin-only tools    - admin_analytics
```

### Customization[​](https://rasa.com/docs/pro/build/mcp-integration/#customization 'Direct link to Customization')

A React style sub agent's behaviour can be customized by one of the three modes:

1. **Custom prompt templates** - Include specific instructions and slot context
2. **Python customization modules** - Override agent behavior with custom python classes
3. **Additional tools** - Add Python-based tools alongside existing tools from MCP servers

See the [Sub Agents Reference](https://rasa.com/docs/reference/config/agents/react-sub-agents/#customization) for detailed customization options.

## Best Practices[​](https://rasa.com/docs/pro/build/mcp-integration/#best-practices 'Direct link to Best Practices')

### MCP Server Setup[​](https://rasa.com/docs/pro/build/mcp-integration/#mcp-server-setup 'Direct link to MCP Server Setup')

- Define servers in `endpoints.yml` for consistency with other Rasa endpoints.
- Use descriptive server names that indicate their purpose.
- Ensure MCP servers are accessible from your Rasa environment.

### Tool Usage in Flows[​](https://rasa.com/docs/pro/build/mcp-integration/#tool-usage-in-flows 'Direct link to Tool Usage in Flows')

- Always account for both structured and unstructured response content from tools.
- Use clear parameter and slot names for maintainability.

### Security Considerations[​](https://rasa.com/docs/pro/build/mcp-integration/#security-considerations 'Direct link to Security Considerations')

- Use `include_tools` to provide only necessary tools for security.
- Use `exclude_tools` to block sensitive or dangerous operations.
- Set clear exit conditions to prevent infinite loops.
- Limit context sharing to necessary slots only.
- Prefer [`meta_map`](https://rasa.com/docs/reference/integrations/mcp-servers/#passing-metadata-to-tools-meta_map) for auth or user context sent to MCP servers: those values are not part of tool schemas shown to the LLM, and Rasa avoids logging or telemetry leakage of `meta_map` values (structure and key names only where needed).
- Regularly audit agent permissions and access to tools.

- [Overview](https://rasa.com/docs/pro/build/mcp-integration/#overview)
- [Defining an MCP Server](https://rasa.com/docs/pro/build/mcp-integration/#defining-an-mcp-server)
- [Using an MCP tool in a flow](https://rasa.com/docs/pro/build/mcp-integration/#using-an-mcp-tool-in-a-flow)
 - [Tool Results and Output Handling](https://rasa.com/docs/pro/build/mcp-integration/#tool-results-and-output-handling)
- [Dynamic selection of tools in an autonomous step](https://rasa.com/docs/pro/build/mcp-integration/#dynamic-selection-of-tools-in-an-autonomous-step)
 - [Sub agent Configuration](https://rasa.com/docs/pro/build/mcp-integration/#sub-agent-configuration)
 - [Invoking a sub agent](https://rasa.com/docs/pro/build/mcp-integration/#invoking-a-sub-agent)
 - [Selective Tool Access](https://rasa.com/docs/pro/build/mcp-integration/#selective-tool-access)
 - [Customization](https://rasa.com/docs/pro/build/mcp-integration/#customization)
- [Best Practices](https://rasa.com/docs/pro/build/mcp-integration/#best-practices)
 - [MCP Server Setup](https://rasa.com/docs/pro/build/mcp-integration/#mcp-server-setup)
 - [Tool Usage in Flows](https://rasa.com/docs/pro/build/mcp-integration/#tool-usage-in-flows)
 - [Security Considerations](https://rasa.com/docs/pro/build/mcp-integration/#security-considerations)