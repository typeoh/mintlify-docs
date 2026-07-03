# Source: https://rasa.com/docs/reference/primitives/custom-actions

On this page

info

In many cases, you can call tools of an MCP (Model Context Protocol) server directly from your flow steps as an alternative to writing custom actions. Directly invoking MCP tools is often a simpler and more maintainable way to integrate with external APIs, databases, or services. Consider this approach before implementing a custom action. See [Calling an MCP Tool](https://rasa.com/docs/reference/primitives/flow-steps/#calling-an-mcp-tool) for more details.

For details on how to implement a custom action, see the [SDK documentation](https://rasa.com/docs/reference/integrations/action-server/running-action-server/). Any custom action that you want to use in your stories should be added into the actions section of your [domain](https://rasa.com/docs/reference/config/domain/).

When the dialogue engine predicts a custom action to be executed, it will call the action server, with the following information:

```
{  "next_action": "string",  "tracker": {    "conversation_id": "default",    "sender_id": "string",    "slots": {},    "latest_message": {},    "latest_event_time": 1537645578.314389,    "followup_action": "string",    "paused": false,    "events": [],    "latest_input_channel": "rest",    "active_loop": {},    "latest_action": {}  },  "domain": {    "config": {},    "session_config": {},    "intents": [],    "entities": [],    "slots": {},    "responses": {},    "actions": [],    "forms": {},    "e2e_actions": []  },  "version": "version"}
```

Your action server should respond with a list of events and responses:

```
{  "events": [{}],  "responses": [{}]}
```

## Running Custom Actions Directly by the Assistant[​](https://rasa.com/docs/reference/primitives/custom-actions/#running-custom-actions-directly-by-the-assistant 'Direct link to Running Custom Actions Directly by the Assistant')

New in 3.10

You can now run Python custom actions directly on the Rasa Assistant without the need for a separate Action Server.

Building an assistant often involves executing custom logic like API calls or database queries. Traditionally, this requires an Action Server. However, for a more integrated and efficient approach, you can run custom actions written in Python directly on the Rasa Assistant. This simplifies deployment and enhances the overall efficiency of your assistant.

### Benefits of Direct Custom Action Execution[​](https://rasa.com/docs/reference/primitives/custom-actions/#benefits-of-direct-custom-action-execution 'Direct link to Benefits of Direct Custom Action Execution')

- **Simplified Architecture**: Removes the need for a separate Action Server, reducing the complexity of your architecture and deployment process.
- **Lower Latency**: Improves response times by eliminating the network roundtrips required to communicate with an external Action Server.
- **Unified Environment**: Executes all components of your assistant within the same environment, making debugging and local development smoother.
- **Cost Efficiency**: Reduces infrastructure costs as there is no need to maintain an additional server for handling custom actions.

### Disadvantages of Direct Custom Action Execution[​](https://rasa.com/docs/reference/primitives/custom-actions/#disadvantages-of-direct-custom-action-execution 'Direct link to Disadvantages of Direct Custom Action Execution')

- **Higher Effort To Secure Rasa Environment**: The Rasa assistant will need access to the same sensitive resources required by the custom actions to access remote services (i.e. tokens, credentials), which may introduce security risks. Therefore, the Rasa instance should be secured properly by running in a more protected environment.

### How to Configure the Feature[​](https://rasa.com/docs/reference/primitives/custom-actions/#how-to-configure-the-feature 'Direct link to How to Configure the Feature')

To use this feature, you need to update the `action_endpoint` section of the `endpoints.yml` file. Add the `actions_module` field and specify the path to your custom actions Python package. This package will be imported and used directly by the Rasa Assistant to run the actions.

For example, consider this is your project structure:

```
my_project/├── actions/│   ├── __init__.py│   ├── actions.py├── data/├── models/├── tests/├── domain.yml├── config.yml├── credentials.yml├── endpoints.yml
```

You can configure the `actions_module` field in the `endpoints.yml` file as follows:

endpoints.yml

```
action_endpoint:    actions_module: "actions"
```

This configuration is mutually exclusive with the `url` field, which you would typically use for pointing to an external Action Server. If both `url` and `actions_module` are specified, `actions_module` will be prioritized.

info

Every time you update the custom actions code, you must restart the Rasa Assistant to reflect the changes via the `rasa run` or `rasa inspect` commands.

- [Running Custom Actions Directly by the Assistant](https://rasa.com/docs/reference/primitives/custom-actions/#running-custom-actions-directly-by-the-assistant)
 - [Benefits of Direct Custom Action Execution](https://rasa.com/docs/reference/primitives/custom-actions/#benefits-of-direct-custom-action-execution)
 - [Disadvantages of Direct Custom Action Execution](https://rasa.com/docs/reference/primitives/custom-actions/#disadvantages-of-direct-custom-action-execution)
 - [How to Configure the Feature](https://rasa.com/docs/reference/primitives/custom-actions/#how-to-configure-the-feature)