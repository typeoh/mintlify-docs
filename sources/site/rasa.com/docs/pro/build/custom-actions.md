# Source: https://rasa.com/docs/pro/build/custom-actions

On this page

A Rasa assistant can execute different types of actions to respond to the user or update conversation state:

- **Responses** _User-facing messages_ defined in your assistant. This is what you will use most frequently to send text, images, buttons, etc.
 
- **Default Actions** _Built-in actions_ that handle certain events or conversation situations out-of-the-box (e.g., `action_restart`, `action_session_start`). These can be overridden with your own custom logic if needed.
 
- **Custom Actions** _User-defined actions_ that can run any code you want (e.g., call external APIs, query databases, or retrieve specific data). Custom actions are the focus of this page.
 

## What Are Custom Actions?[​](https://rasa.com/docs/pro/build/custom-actions/#what-are-custom-actions 'Direct link to What Are Custom Actions?')

A **custom action** lets you execute arbitrary logic within your assistant—for example, retrieving data from an external API or performing a complex database query. Because you can run any code, custom actions offer maximum flexibility.

Keep Logic Out of Custom Actions and Inside Flows

You should avoid hiding your core business logic inside a custom action. Flows (in YAML or via the Studio UI) define how the conversation should proceed in a transparent, maintainable way. A custom action should do just the “raw work”—for example, fetching an API response or returning a database record—and let your flow decide what happens next based on that result.

**Example of “flow-first” design**:

flows.yml

```
flows:  restaurant_booking:    description: "Book a table at a restaurant"    steps:      # ...      - action: check_restaurant_availability        next:          - if: has_availability            then:              - action: utter_has_availability          - else:            - action: utter_no_availability
```

actions.py

```
# Minimal custom action codeclass CheckRestaurantAvailability(Action):    def name(self):        return "check_restaurant_availability"    def run(self, dispatcher, tracker, domain):        # Example: call an API to see if there's availability        has_availability = True        # Return the result in a slot so the flow can branch deterministically        return [SlotSet("has_availability", has_availability)]
```

By keeping the branching logic in the flow, anyone inspecting it can quickly understand how your assistant behaves.

## Rasa Action Server[​](https://rasa.com/docs/pro/build/custom-actions/#rasa-action-server 'Direct link to Rasa Action Server')

When your assistant predicts a custom action, it needs to run your custom code. You can do this in two ways:

1. **Use a standalone Action Server**
 - The Rasa server calls the Action Server with information about the conversation.
 - The Action Server executes your code and returns any responses or events to Rasa.
 - This keeps your custom code isolated from the main assistant server (helpful for security or scaling).
2. **Run custom actions directly on the Rasa Assistant**
 - Write your custom actions in Python and configure them to run directly in the same process as the assistant.
 - This may simplify deployment and reduce latency but requires that your Rasa environment be set up securely to handle sensitive credentials.

Run custom actions directly on the Rasa Assistant if you want simpler deployment, lower latency, and a unified environment for faster development. However, if you need to secure sensitive credentials or prefer isolating resource-intensive components, an external action server gives you greater flexibility, control, and security.

For more details on either approach, see the Reference section on the [Action Server](https://rasa.com/docs/action-server/) and [Custom Actions](https://rasa.com/docs/reference/primitives/custom-actions/).

## How to Write Custom Actions[​](https://rasa.com/docs/pro/build/custom-actions/#how-to-write-custom-actions 'Direct link to How to Write Custom Actions')

1. **Create Your Action File**
 - If using Python, you typically write an `Action` class.
 - If using another language, you need to implement the [webhook API spec](https://rasa.com/docs/reference/api/pro/action-server-api/).
2. **Implement the Logic**
 - Perform the external API calls, database queries, or any other needed logic.
 - Store retrieved data in slots (e.g., `SlotSet("customer_id", customer_id)`) so that your flow can condition on it.
3. **Return Events and Responses**
 - Return the events (e.g. `SlotSet`, `FollowupAction`) needed to update the conversation state.
 - Optionally, return responses to immediately send messages back to the user.
4. **Add Your Action to the Assistant Configuration**
 - List the custom action’s name in your assistant domain or configuration so that your flows can call it.
5. **Decide on Hosting**
 - **Standalone Action Server**: Configure your `endpoints.yml` to point to the Action Server’s URL.
 - **Integrated Execution**: In your `endpoints.yml`, set `actions_module` to point to your Python module.
6. **Train & Test**
 - Once your custom actions are properly listed in your flows and domain, re-train (compile) your assistant and test by running a conversation.

### Next Steps[​](https://rasa.com/docs/pro/build/custom-actions/#next-steps 'Direct link to Next Steps')

- For a deeper look at the parameters and payloads involved, see the Reference section on [Custom Actions](https://rasa.com/docs/reference/primitives/custom-actions/).

- [What Are Custom Actions?](https://rasa.com/docs/pro/build/custom-actions/#what-are-custom-actions)
- [Rasa Action Server](https://rasa.com/docs/pro/build/custom-actions/#rasa-action-server)
- [How to Write Custom Actions](https://rasa.com/docs/pro/build/custom-actions/#how-to-write-custom-actions)
 - [Next Steps](https://rasa.com/docs/pro/build/custom-actions/#next-steps)