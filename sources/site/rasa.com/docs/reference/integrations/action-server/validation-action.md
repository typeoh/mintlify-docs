# Source: https://rasa.com/docs/reference/integrations/action-server/validation-action

On this page

There is a helper class in Rasa SDK with the role of executing custom slot extraction and validation:

- `ValidationAction`: the base class for custom actions extracting and validating slots.

In order to implement custom slot extraction and validation logic, you can subclass the `ValidationAction` class.

## `ValidationAction` class[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationaction-class 'Direct link to validationaction-class')

##### NLU-based assistants

This section refers to building NLU-based assistants. If you are working with [Conversational AI with Language Models (CALM)](https://rasa.com/docs/calm), this content may not apply to you.

You can extend the `ValidationAction` class in the Rasa SDK to define custom extraction and / or validation of slots.

### How to subclass `ValidationAction`[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#how-to-subclass-validationaction 'Direct link to how-to-subclass-validationaction')

Firstly, you must add the name of this action, `action_validate_slot_mappings`, to the domain `actions` list. Note that you do not need to implement the `name` method in the custom action extending `ValidationAction`, since this is already implemented. If you override the `name` method, the custom validation action will not run because the original name is hardcoded in the call that the default action `action_extract_slots` makes to the action server.

You should create only one subclass of `ValidationAction` which should contain all extraction and validation methods for different slots according to your use-case.

With this option, you do not need to specify the `action` key in the [custom slot mapping](https://legacy-docs-oss.rasa.com/docs/rasa/domain/#custom-slot-mappings), since the default action [`action_extract_slots`](https://legacy-docs-oss.rasa.com/docs/rasa/default-actions/#action_extract_slots) runs `action_validate_slot_mappings` automatically if present in the `actions` section of the domain.

#### Validation of Slots with Predefined Mappings[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validation-of-slots-with-predefined-mappings 'Direct link to Validation of Slots with Predefined Mappings')

To validate slots with a predefined mapping, you must write functions named `validate_<slot_name>`.

In the following example, the value for slot `location` is capitalized only if the extracted value is of type string:

```
from typing import Text, Any, Dictfrom rasa_sdk import Tracker, ValidationActionfrom rasa_sdk.executor import CollectingDispatcherfrom rasa_sdk.types import DomainDictclass ValidatePredefinedSlots(ValidationAction):    def validate_location(        self,        slot_value: Any,        dispatcher: CollectingDispatcher,        tracker: Tracker,        domain: DomainDict,    ) -> Dict[Text, Any]:        """Validate location value."""        if isinstance(slot_value, str):            # validation succeeded, capitalize the value of the "location" slot            return {"location": slot_value.capitalize()}        else:            # validation failed, set this slot to None            return {"location": None}
```

#### Extraction of Custom Slot Mappings[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#extraction-of-custom-slot-mappings 'Direct link to Extraction of Custom Slot Mappings')

To define custom extraction code, write an `extract_<slot_name>` method for every slot with a custom slot mapping.

The following example shows the implementation of a custom action that extracts the slot `count_of_insults` to keep track of the user's attitude.

```
from typing import Dict, Text, Anyfrom rasa_sdk import Trackerfrom rasa_sdk.executor import CollectingDispatcherclass ValidateCustomSlotMappings(ValidationAction):    async def extract_count_of_insults(        self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict    ) -> Dict[Text, Any]:        intent_of_last_user_message = tracker.get_intent_of_latest_message()        current_count_of_insults = tracker.get_slot("count_of_insults")        if intent_of_last_user_message == "insult":           current_count_of_insults += 1        return {"count_of_insults": current_count_of_insults}
```

### `ValidationAction` class implementation[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationaction-class-implementation 'Direct link to validationaction-class-implementation')

`ValidationAction` is a subclass of the `Action` Rasa SDK class and the abstract Python `ABC` class. Therefore the class implements the `name` and `run` methods inherited from `Action`. In addition, `ValidationAction` implements more specialized methods that will be called in the `run` method:

- `get_extraction_events`: Extracts custom slots using available `extract_<slot name>` methods
- `get_validation_events`: Validates slots by calling available `validate_<slot name>` methods for each slot
- `required_slots`: Returns slots which the validation action should fill

#### Methods[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#methods 'Direct link to Methods')

##### ValidationAction.name[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationactionname 'Direct link to ValidationAction.name')

Defines the action's name: this must be hardcoded as `action_validate_slot_mappings`.

- **Returns**:
 
 Name of action
 
- **Return type**:
 
 `str`
 

##### ValidationAction.run[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationactionrun 'Direct link to ValidationAction.run')

```
async ValidationAction.run(dispatcher, tracker, domain)
```

The `run` method executes the custom extraction code defined in `extract_<slot name>` methods by calling the `get_extraction_events` method, then updates the tracker with the returned events. The `run` method will also execute custom validation code defined in `validate_<slot name>` methods via the `get_validation_events` method and add the returned events to the tracker.

###### **Parameters**[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#parameters 'Direct link to parameters')

- **dispatcher** – the dispatcher which is used to send messages back to the user. Use `dispatcher.utter_message()` or any other `rasa_sdk.executor.CollectingDispatcher` method. See the [documentation for the dispatcher](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/)
 
- **tracker** – the state tracker for the current user. You can access slot values using `tracker.get_slot(slot_name)`, the most recent user message is `tracker.latest_message.text` and any other `rasa_sdk.Tracker` property. See the [documentation for the tracker](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/).
 
- **domain** – the bot's domain
 

###### **Returns**[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#returns 'Direct link to returns')

A list of `rasa_sdk.events.Event` instances. See the [documentation for events](https://rasa.com/docs/reference/integrations/action-server/sdk-events/).

###### **Return type**[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#return-type 'Direct link to return-type')

`List`\[`Dict`\[`str`, `Any`\]\]

##### ValidationAction.required\_slots[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationactionrequired_slots 'Direct link to ValidationAction.required_slots')

```
async ValidationAction.required_slots(domain_slots, dispatcher, tracker, domain)
```

The `required_slots` method will return the `domain_slots` which is a list of all slot names mapped in the domain that do not include any slot mapping with conditions. `domain_slots` is returned by the `domain_slots` method, which only takes `Domain` as an argument.

###### **Returns**[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#returns-1 'Direct link to returns-1')

A list of slot names of type `Text`.

##### ValidationAction.get\_extraction\_events[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationactionget_extraction_events 'Direct link to ValidationAction.get_extraction_events')

```
async ValidationAction.get_extraction_events(dispatcher, tracker, domain)
```

The `get_extraction_events` method will gather the list of slot names via `required_slots` method call and then loop through every slot name to run the `extract_<slot name>` method if available.

##### **Returns**[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#returns-2 'Direct link to returns-2')

A list of `rasa_sdk.events.SlotSet` instances. See the [documentation for SlotSet events](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#slotset).

##### ValidationAction.get\_validation\_events[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationactionget_validation_events 'Direct link to ValidationAction.get_validation_events')

```
async ValidationAction.get_validation_events(dispatcher, tracker, domain)
```

The `get_validation_events` method will gather the list of slot names to validate via `required_slots` method call. Then it will get a mapping of slots which were recently set and their values via `tracker.slots_to_validate` call. Looping through this mapping of recently extracted slots, it will check if the slot is in `required_slots` then run the `validate_<slot name>` method if available for that slot.

###### **Returns**[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#returns-3 'Direct link to returns-3')

A list of `rasa_sdk.events.SlotSet` instances. See the [documentation for SlotSet events](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#slotset).

### How to implement custom validation in CALM assistants[​](https://rasa.com/docs/reference/integrations/action-server/validation-action/#how-to-implement-custom-validation-in-calm-assistants 'Direct link to How to implement custom validation in CALM assistants')

In CALM assistants, slot validation actions should be implemented like normal [actions](https://rasa.com/docs/reference/integrations/action-server/sdk-actions/) by subclassing the base `Action` class.

Here is an example of a custom slot validation action in a CALM assistant:

```
class ValidateLocation(Action):    def name(self) -> Text:        return "validate_location"        def run(        self,        dispatcher: CollectingDispatcher,        tracker: Tracker,        domain: DomainDict,    ) -> Dict[Text, Any]:        """Validate slot value."""        slot_value = tracker.get_slot("location")        if isinstance(slot_value, str):            # validation succeeded, capitalize the value of the "location" slot            return {"location": slot_value.capitalize()}        else:            # validation failed, set this slot to None            return {"location": None}
```

- [`ValidationAction` class](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationaction-class)
 - [How to subclass `ValidationAction`](https://rasa.com/docs/reference/integrations/action-server/validation-action/#how-to-subclass-validationaction)
 - [`ValidationAction` class implementation](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationaction-class-implementation)
 - [How to implement custom validation in CALM assistants](https://rasa.com/docs/reference/integrations/action-server/validation-action/#how-to-implement-custom-validation-in-calm-assistants)