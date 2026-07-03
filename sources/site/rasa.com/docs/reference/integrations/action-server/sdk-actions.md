# Source: https://rasa.com/docs/reference/integrations/action-server/sdk-actions

On this page

The `Action` class is the base class for any custom action. To define a custom action, create a subclass of the `Action` class and overwrite the two required methods, `name` and `run`. The action server will call an action according to the return value of its `name` method when it receives a request to run an action.

A skeleton custom action looks like this:

```
class MyCustomAction(Action):    def name(self) -> Text:        return "action_name"    async def run(        self, dispatcher, tracker: Tracker, domain: Dict[Text, Any],    ) -> List[Dict[Text, Any]]:        return []
```

## Methods[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#methods 'Direct link to Methods')

### Action.name[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#actionname 'Direct link to Action.name')

Defines the action's name. The name returned by this method is the one used in your bot's domain.

- **Returns**:
 
 Name of action
 
- **Return type**:
 
 `str`
 

### Action.run[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#actionrun 'Direct link to Action.run')

```
async Action.run(dispatcher, tracker, domain)
```

The `run` method executes the side effects of the action.

#### **Parameters**[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#parameters 'Direct link to parameters')

- **dispatcher** – the dispatcher which is used to send messages back to the user. Use `dispatcher.utter_message()` or any other `rasa_sdk.executor.CollectingDispatcher` method. See the [documentation for the dispatcher](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/)
 
- **tracker** – the state tracker for the current user. You can access slot values using `tracker.get_slot(slot_name)`, the most recent user message is `tracker.latest_message.text` and any other `rasa_sdk.Tracker` property. See the [documentation for the tracker](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/).
 
- **domain** – the bot's domain
 

#### **Returns**[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#returns 'Direct link to returns')

A list of `rasa_sdk.events.Event` instances. See the [documentation for events](https://rasa.com/docs/reference/integrations/action-server/sdk-events/).

#### **Return type**[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#return-type 'Direct link to return-type')

`List`\[`Dict`\[`str`, `Any`\]\]

## Example[​](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#example 'Direct link to Example')

In a restaurant bot, if the user says “show me a Mexican restaurant”, your bot could execute the action `ActionCheckRestaurants`, which might look like this:

```
from typing import Text, Dict, Any, Listfrom rasa_sdk import Actionfrom rasa_sdk.events import SlotSetclass ActionCheckRestaurants(Action):   def name(self) -> Text:      return "action_check_restaurants"   def run(self,           dispatcher: CollectingDispatcher,           tracker: Tracker,           domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:      cuisine = tracker.get_slot('cuisine')      q = "select * from restaurants where cuisine='{0}' limit 1".format(cuisine)      result = db.query(q)      return [SlotSet("matches", result if result is not None else [])]
```

This action queries a database to find restaurants matching the requested cuisine, and uses the list of restaurants found to set the value of the `matches` slot.

- [Methods](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#methods)
 - [Action.name](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#actionname)
 - [Action.run](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#actionrun)
- [Example](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/#example)