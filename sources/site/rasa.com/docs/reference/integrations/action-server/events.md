# Source: https://rasa.com/docs/reference/integrations/action-server/events

On this page

Conversations in Rasa are represented as a sequence of events. Custom actions can influence the course of a conversation by returning events in the response to the action server request.

Not all events are typically returned by custom actions since they are tracked automatically by Rasa (e.g. user messages). Others events can only be tracked if they are returned by a custom action.

## Event Types[​](https://rasa.com/docs/reference/integrations/action-server/events/#event-types 'Direct link to Event Types')

### `slot`[​](https://rasa.com/docs/reference/integrations/action-server/events/#slot 'Direct link to slot')

Sets a slot on the tracker. It can set a slot to a value, or reset a slot by setting its value to `null`.

**Automatic Tracking**:

- When a slot is filled by an entity of the same name.

A custom action is needed to set any slot not auto-filled by an entity.

**JSON**:

```
{  "event": "slot",  "name": "departure_airport",  "value": "BER"}
```

**Parameters**:

- `name`: Name of the slot to set
- `value`: Value to set the slot to. The datatype must match the [type](https://rasa.com/docs/reference/primitives/slots/#slot-types) of the slot

**Rasa Class**: `rasa.core.events.SlotSet`

### `reset_slots`[​](https://rasa.com/docs/reference/integrations/action-server/events/#reset_slots 'Direct link to reset_slots')

Resets all slots on the tracker to `null`.

**Automatic Tracking**: Never

**JSON**:

```
{  "event": "reset_slots"}
```

**Rasa Class**: `rasa.core.events.AllSlotsReset`

### `reminder`[​](https://rasa.com/docs/reference/integrations/action-server/events/#reminder 'Direct link to reminder')

Schedules an intent to be triggered at a certain time in the future.

**Automatic Tracking**: Never

**JSON**:

```
{  "event": "reminder",  "intent": "my_intent",  "entities": { "entity1": "value1", "entity2": "value2" },  "date_time": "2018-09-03T11:41:10.128172",  "name": "my_reminder",  "kill_on_user_msg": true}
```

**Parameters**:

- `intent`: Intent which the reminder will trigger
- `entities`: Entities to send with the intent
- `date_time`: Date at which the execution of the action should be triggered. This should either be in UTC or include a timezone.
- `name`: ID of the reminder. If there are multiple reminders with the same id only the last will be run.
- `kill_on_user_msg`: Whether a user message before the trigger time will abort the reminder

**Rasa Class**: `rasa.core.events.ReminderScheduled`

### `cancel_reminder`[​](https://rasa.com/docs/reference/integrations/action-server/events/#cancel_reminder 'Direct link to cancel_reminder')

Cancels a scheduled reminder or reminders. All reminders which match the supplied parameters will be cancelled.

**Automatic Tracking**: Never

**JSON**:

```
{  "event": "cancel_reminder",  "name": "my_reminder",  "intent": "my_intent",  "entities": [    { "entity": "entity1", "value": "value1" },    { "entity": "entity2", "value": "value2" }  ],  "date_time": "2018-09-03T11:41:10.128172"}
```

**Parameters**:

- `intent`: Intent which the reminder will trigger
- `entities`: Entities to send with the intent
- `date_time`: Date at which the execution of the action should be triggered. This should either be in UTC or include a timezone.
- `name`: ID of the reminder.

**Rasa Class**: `rasa.core.events.ReminderCancelled`

### `pause`[​](https://rasa.com/docs/reference/integrations/action-server/events/#pause 'Direct link to pause')

Stops the bot from responding to user messages. The conversation will remain paused and no actions will be predicted until the conversation is explicitly [resumed](https://rasa.com/docs/reference/integrations/action-server/events/#resume).

**Automatic Tracking**: Never

**JSON**:

```
{  "event": "pause"}
```

**Rasa Class**: `rasa.core.events.ConversationPaused`

### `resume`[​](https://rasa.com/docs/reference/integrations/action-server/events/#resume 'Direct link to resume')

Resume a previously paused conversation. Once this event is added to the tracker the bot will start predicting actions again. It will not predict actions for user messages received while the conversation was paused.

**Automatic Tracking**: Never

**JSON**:

```
{  "event": "resume"}
```

**Rasa Class**: `rasa.core.events.ConversationResumed`

### `followup`[​](https://rasa.com/docs/reference/integrations/action-server/events/#followup 'Direct link to followup')

Force a follow up action, bypassing action prediction.

**Automatic Tracking**: Never

**JSON**:

```
{  "event": "followup",  "name": "my_action"}
```

**Parameters**:

- `name`: The name of the follow up action that will be executed.

**Rasa Class**: `rasa.core.events.FollowupAction`

### `rewind`[​](https://rasa.com/docs/reference/integrations/action-server/events/#rewind 'Direct link to rewind')

Reverts all side effects of the last user message and removes the last `user` event from the tracker.

**Automatic Tracking**:

**JSON**:

```
{  "event": "rewind"}
```

**Rasa Class**: `rasa.core.events.UserUtteranceReverted`

### `undo`[​](https://rasa.com/docs/reference/integrations/action-server/events/#undo 'Direct link to undo')

Undoes all side effects of the last bot action and removes the last bot action from the tracker.

**Automatic Tracking**:

**JSON**:

```
{  "event": "undo"}
```

**Rasa Class**: `rasa.core.events.ActionReverted`

### `restart`[​](https://rasa.com/docs/reference/integrations/action-server/events/#restart 'Direct link to restart')

Resets the tracker. After a `restart` event, there will be no conversation history and no record of the restart.

**Automatic Tracking**:

- When the `/restart` default intent is triggered.

**JSON**:

```
{  "event": "restart"}
```

**Rasa Class**: `rasa.core.events.Restarted`

### `session_started`[​](https://rasa.com/docs/reference/integrations/action-server/events/#session_started 'Direct link to session_started')

Starts a new conversation by resetting the tracker and running the default action `ActionSessionStart`. This action will by default carry over existing `SlotSet` events to a new conversation session. You can configure this behaviour in your domain file under `session_config`.

**Automatic Tracking**:

- Whenever a user starts a conversation with the bot for the first time.
- Whenever a session expires (after `session_expiration_time` specified in the domain), and the user resumes their conversation

Restarting a conversation with [`restart`](https://rasa.com/docs/reference/integrations/action-server/events/#restart) event **does not** automatically cause a `session_started` event.

**JSON**:

```
{  "event": "session_started"}
```

**Rasa Class**: `rasa.core.events.SessionStarted`

### `user`[​](https://rasa.com/docs/reference/integrations/action-server/events/#user 'Direct link to user')

The user sent a message to the bot.

**Automatic Tracking**:

- When the user sends a message to the bot.

This event is not usually returned by a custom action.

**JSON**:

```
{  "event": "user",  "text": "Hey",  "parse_data": {    "intent": {      "name": "greet",      "confidence": 0.9    },    "entities": []  },  "metadata": {}}
```

**Parameters**:

- `text`: Text of the user message
- `parse_data`: Parsed data of user message. This is ordinarily filled by NLU.
- `metadata`: Arbitrary metadata that comes with the user message

**Rasa Class**: `rasa.core.events.UserUttered`

### `bot`[​](https://rasa.com/docs/reference/integrations/action-server/events/#bot 'Direct link to bot')

The bot sent a message to the user.

**Automatic Tracking**:

- Whenever `responses` are returned by a custom action
- Whenever responses are sent to the user directly without being returned by a custom action (e.g. `utter_` actions)

This event is not usually returned explicitly by a custom action; `responses` would be returned instead.

**JSON**:

```
{  "event": "bot",  "text": "Hey there!",  "data": {}}
```

**Parameters**:

- `text`: The text the bot sends to the user
- `data`: Any non-text elements of the bot response. The structure of `data` matches that of `responses` given in the [API spec](https://rasa.com/docs/reference/api/pro/action-server-api/).

**Rasa Class**: `rasa.core.events.BotUttered`

### `action`[​](https://rasa.com/docs/reference/integrations/action-server/events/#action 'Direct link to action')

Logs an action called by the bot. Only the action itself is logged; the events that the action creates are logged separately when they are applied.

**Automatic Tracking**:

- Any action (including custom actions and responses) that is called, even if the action does not execute successfully.

This event is not usually returned explicitly by a custom action.

**JSON**:

```
{  "event": "action",  "name": "my_action"}
```

**Parameters**:

- `name`: Name of the action that was called

**Rasa Class**: `rasa.core.events.ActionExecuted`

- [Event Types](https://rasa.com/docs/reference/integrations/action-server/events/#event-types)
 - [`slot`](https://rasa.com/docs/reference/integrations/action-server/events/#slot)
 - [`reset_slots`](https://rasa.com/docs/reference/integrations/action-server/events/#reset_slots)
 - [`reminder`](https://rasa.com/docs/reference/integrations/action-server/events/#reminder)
 - [`cancel_reminder`](https://rasa.com/docs/reference/integrations/action-server/events/#cancel_reminder)
 - [`pause`](https://rasa.com/docs/reference/integrations/action-server/events/#pause)
 - [`resume`](https://rasa.com/docs/reference/integrations/action-server/events/#resume)
 - [`followup`](https://rasa.com/docs/reference/integrations/action-server/events/#followup)
 - [`rewind`](https://rasa.com/docs/reference/integrations/action-server/events/#rewind)
 - [`undo`](https://rasa.com/docs/reference/integrations/action-server/events/#undo)
 - [`restart`](https://rasa.com/docs/reference/integrations/action-server/events/#restart)
 - [`session_started`](https://rasa.com/docs/reference/integrations/action-server/events/#session_started)
 - [`user`](https://rasa.com/docs/reference/integrations/action-server/events/#user)
 - [`bot`](https://rasa.com/docs/reference/integrations/action-server/events/#bot)
 - [`action`](https://rasa.com/docs/reference/integrations/action-server/events/#action)