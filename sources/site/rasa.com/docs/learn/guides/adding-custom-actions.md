# Source: https://rasa.com/docs/learn/guides/adding-custom-actions

On this page

Imagine you’re developing a digital banking assistant. A user asks, “What’s my account balance?” Instead of delivering a simple response or handing the chat over to an agent, your assistant reaches out to your banking API, retrieves the current balance, and responds with personalized information.

You can achieve this using custom actions in Rasa. **Custom Actions** are a flexible code interface to extend your assistant's capabilities beyond conversations.

Using custom actions, you can:

- **Query databases:** Retrieve user information in real-time.
- **Call APIs:** Connect to external services for up-to-date data.
- **Execute business logic:** Process information and make decisions dynamically.
- **Generate dynamic responses:** Tailor interactions based on current data.

## How it works[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#how-it-works 'Direct link to How it works')

Custom actions are run during conversations—you can specify whether you'd like them to be called as part of a flow by including it as a step or based off of a set of conditions. Once triggered, the custom action runs your code, and returns values that you can use to influence the business logic of your conversation.

### Key components[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#key-components 'Direct link to Key components')

Custom Actions in Rasa have two key methods:

- **Action name** `name`: A unique identifier that Rasa uses to trigger the action.
- **Run method** `run`: The custom logic that you want the action to run

### Input and output of actions[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#input-and-output-of-actions 'Direct link to Input and output of actions')

Actions can reference parts of the conversation state and the responses can be stored as values in the assistant's memory (as slots) or directly sent to the user as a message.

In an action you can:

- **Reference the conversation state**: This is accessible through the tracker (ie. reading slot values or checking the latest message)
- **Return events**: Save values by returning events (ie. setting slots)
- **Send messages**: Send messages directly back to the user through the dispatcher (note: we recommend to leverage responses rather than using this method for better validation support)

#### Example[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#example 'Direct link to Example')

In this example, the assistant supports the user in booking a restaurant. It first asks for some details about the user's preference and then uses that input to query an API for availability based on the user's preferences. The custom action might look something like this:

- Pro
- Studio

actions.py

```
from typing import Any, Text, Dict, Listfrom rasa_sdk import Action, Trackerfrom rasa_sdk.executor import CollectingDispatcherclass ActionCheckRestaurants(Action):    def name(self) -> Text:        return "action_check_restaurants"    def run(self, dispatcher: CollectingDispatcher,            tracker: Tracker,            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:                cuisine = tracker.get_slot('cuisine')  # Get what cuisine the user wants from the tracker        results = search_database(cuisine)     # Do something with that information        return [SlotSet("matches", results)]   # Store the results in a slot
```

You can also reference actions in the Studio flow builder:

![Action that checks restaurant availability in flow builder](https://rasa.com/docs/assets/images/restaurant_action-4063dc6a21fe8e69c31a8c2373f9883c.png)

### Storing response values[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#storing-response-values 'Direct link to Storing response values')

A typical usecase for actions is to use the return response as a stored value in your assistant's memory so that you can reference it in conditional logic or in flows. When a Custom Action queries a database or calls an API, it can store the retrieved information in slots, which your assistant can then use to, make decisions about the conversation flow, provide personalized responses to the user, remember important details for future interactions or validate user inputs against business rules.

In the previous example, once the custom action has returned the response from the API, you can then use the response values to set a slot that renders some buttons that allow your user to select their preferred booking option.

## Trying out your Actions[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#trying-out-your-actions 'Direct link to Trying out your Actions')

Rasa determines that a custom action needs to be executed based on the flow. Rasa sends conversation information, such as the conversation history, slots, and domain information to your custom action. Your custom action then sends a response back to Rasa, which might include updated slot values, responses to be sent to the user, or follow-up events.

Once you're happy with your action, you have a few different options to try it out in your assistant.

In **server mode**, an action server communicates with the main Rasa server through HTTP/HTTPS protocol. We recommend this approach for use in a production, staging or development environment. You can read more about how to deploy an action server here (../link-to-reference)

In **module mode**, custom actions are executed directly within the Rasa Assistant without the need for a separate Action Server. This method works well for local testing testing, and simpler deployments where ease of use is a priority.

## Next Steps[​](https://rasa.com/docs/learn/guides/adding-custom-actions/#next-steps 'Direct link to Next Steps')

Custom actions form the bridge between your assistant and your business systems. As you build your custom actions, we recommend implementing a comprehensive error handling system (e.g. APIs being unavailable) and providing clear feedback to users when operations don't go as planned.

For more detailed information, check our [Custom Actions Reference](https://rasa.com/docs/reference/primitives/custom-actions/).

- [How it works](https://rasa.com/docs/learn/guides/adding-custom-actions/#how-it-works)
 - [Key components](https://rasa.com/docs/learn/guides/adding-custom-actions/#key-components)
 - [Input and output of actions](https://rasa.com/docs/learn/guides/adding-custom-actions/#input-and-output-of-actions)
 - [Storing response values](https://rasa.com/docs/learn/guides/adding-custom-actions/#storing-response-values)
- [Trying out your Actions](https://rasa.com/docs/learn/guides/adding-custom-actions/#trying-out-your-actions)
- [Next Steps](https://rasa.com/docs/learn/guides/adding-custom-actions/#next-steps)