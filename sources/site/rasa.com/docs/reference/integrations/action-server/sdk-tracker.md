# Source: https://rasa.com/docs/reference/integrations/action-server/sdk-tracker

On this page

The `Tracker` class represents a Rasa conversation tracker. It lets you access your bot's memory in your custom actions. You can get information about past events and the current state of the conversation through `Tracker` attributes and methods.

## Attributes[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#attributes 'Direct link to Attributes')

The following are available as attributes of a `Tracker` object:

- `sender_id` - The unique ID of person talking to the bot.
 
- `slots` - The list of slots that can be filled as defined in the “ref”domains.
 
- `latest_message` - A dictionary containing the attributes of the latest message: `intent`, `entities` and `text`.
 
- `events` - A list of all previous events.
 
- `active_loop` - The name of the currently active loop.
 
- `latest_action_name` - The name of the last action the bot executed.
 

## Methods[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#methods 'Direct link to Methods')

The available methods from the `Tracker` are:

### Tracker.current\_state[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackercurrent_state 'Direct link to Tracker.current_state')

Return the current tracker state as an object.

- **Return type**

`Dict[str, Any]`

### Tracker.is\_paused[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackeris_paused 'Direct link to Tracker.is_paused')

State whether the tracker is currently paused.

- **Return type**

`bool`

### Tracker.get\_latest\_entity\_values[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_latest_entity_values 'Direct link to Tracker.get_latest_entity_values')

Get entity values found for the passed entity type and optional role and group in latest message. If you are only interested in the first entity of a given type use:

```
next(tracker.get_latest_entity_values(“my_entity_name”), None)
```

If no entity is found, then `None` is the default result.

- **Parameters**
 
 - `entity_type` – the entity type of interest
 
 - `entity_role` – optional entity role of interest
 
 - `entity_group` – optional entity group of interest
 
- **Returns**
 

List of entity values.

- **Return type**

`Iterator[str]`

### Tracker.get\_latest\_input\_channel[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_latest_input_channel 'Direct link to Tracker.get_latest_input_channel')

Get the name of the input\_channel of the latest UserUttered event

- **Return type**

`Optional[str]`

### Tracker.events\_after\_latest\_restart[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerevents_after_latest_restart 'Direct link to Tracker.events_after_latest_restart')

Return a list of events after the most recent restart.

- **Return type**

`List[Dict]`

### Tracker.get\_slot[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_slot 'Direct link to Tracker.get_slot')

Retrieves the value of a slot.

- **Parameters**
 
 - `key` – the name of the slot of which to retrieve the value
- **Return type**
 

`Optional[Any]`

### Tracker.get\_intent\_of\_latest\_message[​](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_intent_of_latest_message 'Direct link to Tracker.get_intent_of_latest_message')

Retrieves the user's latest intent.

- **Parameters**
 
 - `skip_fallback_intent` (default: `True`) – Optionally skip the `nlu_fallback` intent and return the next highest ranked.
- **Returns**
 

The intent of the latest message if available.

- **Return type**

`Optional[Text]`

- [Attributes](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#attributes)
- [Methods](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#methods)
 - [Tracker.current\_state](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackercurrent_state)
 - [Tracker.is\_paused](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackeris_paused)
 - [Tracker.get\_latest\_entity\_values](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_latest_entity_values)
 - [Tracker.get\_latest\_input\_channel](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_latest_input_channel)
 - [Tracker.events\_after\_latest\_restart](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerevents_after_latest_restart)
 - [Tracker.get\_slot](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_slot)
 - [Tracker.get\_intent\_of\_latest\_message](https://rasa.com/docs/reference/integrations/action-server/sdk-tracker/#trackerget_intent_of_latest_message)