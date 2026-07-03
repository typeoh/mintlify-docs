# Source: https://rasa.com/docs/reference/integrations/action-server/sdk-events

On this page

Internally, Rasa conversations are represented as a list of [events](https://rasa.com/docs/reference/integrations/action-server/events/). Rasa SDK provides classes for each event, and takes care of turning instances of event classes into properly formatted event payloads.

This page is about the event classes in `rasa_sdk`. The side effects of events and their underlying payloads are identical regardless of whether you use `rasa_sdk` or another action server. For details about the side effects of an event, its underlying payload and the class in Rasa it is translated to see the [documentation for events for all action servers](https://rasa.com/docs/reference/integrations/action-server/events/) (also linked to in each section).

Importing events

All events written in a Rasa SDK action server need to be imported from `rasa_sdk.events`.

## Event Classes[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#event-classes 'Direct link to Event Classes')

### SlotSet[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#slotset 'Direct link to SlotSet')

```
rasa_sdk.events.SlotSet(    key: Text,    value: Any = None,    timestamp: Optional[float] = None)
```

**Underlying event**: [`slot`](https://rasa.com/docs/reference/integrations/action-server/events/#slot)

**Parameters**:

- `key`: Name of the slot to set
- `value`: Value to set the slot to. The datatype must match the [type](https://rasa.com/docs/reference/primitives/slots/#slot-types) of the slot
- `timestamp`: Optional timestamp of the event

**Example**

```
evt = SlotSet(key = "name", value = "Mary")
```

### AllSlotsReset[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#allslotsreset 'Direct link to AllSlotsReset')

```
rasa_sdk.events.AllSlotsReset(timestamp: Optional[float] = None)
```

**Underlying event**: [`reset_slots`](https://rasa.com/docs/reference/integrations/action-server/events/#reset_slots)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = AllSlotsReset()
```

### ReminderScheduled[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#reminderscheduled 'Direct link to ReminderScheduled')

```
rasa_sdk.events.ReminderScheduled(    intent_name: Text,    trigger_date_time: datetime.datetime,    entities: Optional[Union[List[Dict[Text, Any]], Dict[Text, Text]]] = None,    name: Optional[Text] = None,    kill_on_user_message: bool = True,    timestamp: Optional[float] = None,)
```

**Underlying event**: [`reminder`](https://rasa.com/docs/reference/integrations/action-server/events/#reminder)

**Parameters**:

- `intent_name`: Intent which the reminder will trigger
- `trigger_date_time`: Datetime at which the execution of the action should be triggered.
- `entities`: Entities to send with the intent
- `name`: ID of the reminder. If there are multiple reminders with the same id only the last will be run.
- `kill_on_user_message`: Whether a user message before the trigger time will abort the reminder
- `timestamp`: Optional timestamp of the event

**Example**:

```
from datetime import datetimeevt = ReminderScheduled(    intent_name = "EXTERNAL_dry_plant",    trigger_date_time = datetime(2020, 9, 15, 0, 36, 0, 851609),    entities = [{"name": "plant","value":"orchid"}],    name = "remind_water_plants",)
```

### ReminderCancelled[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#remindercancelled 'Direct link to ReminderCancelled')

```
ReminderCancelled(    name: Optional[Text] = None,    intent_name: Optional[Text] = None,    entities: Optional[Union[List[Dict[Text, Any]], Dict[Text, Text]]] = None,    timestamp: Optional[float] = None,)
```

**Underlying event**: [`cancel_reminder`](https://rasa.com/docs/reference/integrations/action-server/events/#cancel_reminder)

**Parameters**:

- `name`: ID of the reminder.
- `intent_name`: Intent which the reminder triggers
- `entities`: Entities sent with the intent
- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = ReminderCancelled(name = "remind_water_plants")
```

### ConversationPaused[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#conversationpaused 'Direct link to ConversationPaused')

```
ConversationPaused(timestamp: Optional[float] = None)
```

**Underlying event**: [`pause`](https://rasa.com/docs/reference/integrations/action-server/events/#slot)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = ConversationPaused()
```

### ConversationResumed[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#conversationresumed 'Direct link to ConversationResumed')

```
ConversationResumed(timestamp: Optional[float] = None)
```

**Underlying event**: [`resume`](https://rasa.com/docs/reference/integrations/action-server/events/#resume)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = ConversationResumed()
```

### FollowupAction[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#followupaction 'Direct link to FollowupAction')

```
FollowupAction(    name: Text,    timestamp: Optional[float] = None)
```

**Underlying event**: [`followup`](https://rasa.com/docs/reference/integrations/action-server/events/#followup)

**Parameters**:

- `name`: The name of the follow up action that will be executed.
- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = FollowupAction(name = "action_say_goodbye")
```

### UserUtteranceReverted[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#userutterancereverted 'Direct link to UserUtteranceReverted')

```
UserUtteranceReverted(timestamp: Optional[float] = None)
```

**Underlying event**: [`rewind`](https://rasa.com/docs/reference/integrations/action-server/events/#rewind)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = UserUtteranceReverted()
```

### ActionReverted[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#actionreverted 'Direct link to ActionReverted')

```
ActionReverted(timestamp: Optional[float] = None)
```

**Underlying event**: [`undo`](https://rasa.com/docs/reference/integrations/action-server/events/#undo)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = ActionReverted()
```

### Restarted[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#restarted 'Direct link to Restarted')

```
Restarted(timestamp: Optional[float] = None)
```

**Underlying event**: [`restart`](https://rasa.com/docs/reference/integrations/action-server/events/#restart)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = Restarted()
```

### SessionStarted[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#sessionstarted 'Direct link to SessionStarted')

```
SessionStarted(timestamp: Optional[float] = None)
```

**Underlying event**: [`session_started`](https://rasa.com/docs/reference/integrations/action-server/events/#session_started)

**Parameters**:

- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = SessionStarted()
```

### UserUttered[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#useruttered 'Direct link to UserUttered')

```
UserUttered(    text: Optional[Text],    parse_data: Optional[Dict[Text, Any]] = None,    timestamp: Optional[float] = None,    input_channel: Optional[Text] = None,)
```

**Underlying event**: [`user`](https://rasa.com/docs/reference/integrations/action-server/events/#user)

**Parameters**:

- `text`: Text of the user message
- `parse_data`: Parsed data of user message. This is ordinarily filled by NLU.
- `input_channel`: The channel on which the message was received
- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = UserUttered(text = "Hallo bot")
```

### BotUttered[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#botuttered 'Direct link to BotUttered')

```
BotUttered(    text: Optional[Text] = None,    data: Optional[Dict[Text, Any]] = None,    metadata: Optional[Dict[Text, Any]] = None,    timestamp: Optional[float] = None,)
```

**Underlying event**: [`bot`](https://rasa.com/docs/reference/integrations/action-server/events/#bot)

**Parameters**:

- `text`: The text the bot sends to the user
- `data`: Any non-text elements of the bot response. The structure of `data` matches that of `responses` given in the [API spec](https://rasa.com/docs/reference/api/pro/action-server-api/).
- `metadata`: Arbitrary key-value metadata
- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = BotUttered(text = "Hallo user")
```

### ActionExecuted[​](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#actionexecuted 'Direct link to ActionExecuted')

```
ActionExecuted(    action_name,    policy=None,    confidence: Optional[float] = None,    timestamp: Optional[float] = None,)
```

**Underlying event**: [`action`](https://rasa.com/docs/reference/integrations/action-server/events/#action)

**Parameters**:

- `action_name`: Name of the action that was called
- `policy`: The policy used to predict the action
- `confidence`: The confidence with which the action was predicted
- `timestamp`: Optional timestamp of the event

**Example**:

```
evt = ActionExecuted("action_greet_user")
```

- [Event Classes](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#event-classes)
 - [SlotSet](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#slotset)
 - [AllSlotsReset](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#allslotsreset)
 - [ReminderScheduled](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#reminderscheduled)
 - [ReminderCancelled](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#remindercancelled)
 - [ConversationPaused](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#conversationpaused)
 - [ConversationResumed](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#conversationresumed)
 - [FollowupAction](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#followupaction)
 - [UserUtteranceReverted](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#userutterancereverted)
 - [ActionReverted](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#actionreverted)
 - [Restarted](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#restarted)
 - [SessionStarted](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#sessionstarted)
 - [UserUttered](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#useruttered)
 - [BotUttered](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#botuttered)
 - [ActionExecuted](https://rasa.com/docs/reference/integrations/action-server/sdk-events/#actionexecuted)