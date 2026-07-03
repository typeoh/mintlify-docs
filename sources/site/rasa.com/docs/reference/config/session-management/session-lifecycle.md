# Source: https://rasa.com/docs/reference/config/session-management/session-lifecycle

On this page

## Session ID[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#session-id 'Direct link to Session ID')

Every session has a unique `session_id` (UUID v4) automatically generated and attached to the metadata of every event in that session. No configuration is required.

### When a new `session_id` is generated[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#when-a-new-session_id-is-generated 'Direct link to when-a-new-session_id-is-generated')

| Trigger | Description |
| --- | --- |
| Brand new conversation | First event on a tracker |
| `action_session_start` / `SessionStarted` | A new session starts |
| `ConversationResumed` | Explicit resumption after inactivity |
| `UserUttered` on an inactive tracker | User reactivates an inactive conversation |

### `session_id` in event metadata[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#session_id-in-event-metadata 'Direct link to session_id-in-event-metadata')

All events within the same session share the same `session_id`. A new session carries a new UUID.

```
{  "event": "user",  "text": "Hello",  "metadata": {    "session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456"  }}
```

### `current_session_id` on the tracker[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#current_session_id-on-the-tracker 'Direct link to current_session_id-on-the-tracker')

The tracker also exposes a `current_session_id` field, so you can read the active session ID at any point during the conversation:

```
{  "sender_id": "conversation_12345",  "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",  "events": [...]}
```

## Conversation Lifecycle Events[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#conversation-lifecycle-events 'Direct link to Conversation Lifecycle Events')

### `ConversationInactive`[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#conversationinactive 'Direct link to conversationinactive')

`ConversationInactive` marks a conversation as **inactive but resumable**. It is emitted automatically by the session timer when the inactivity timeout elapses.

**Tracker effects:**

- `inactive = true` is set on the tracker.
- The conversation continues to accept new events.
- The inactive state is cleared when a `UserUttered` or `ConversationResumed` event is received.

### `SessionEnded`[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#sessionended 'Direct link to sessionended')

`SessionEnded` marks a conversation as **permanently terminated**. No further events can be appended after this point.

**Tracker effects:**

- `terminated = true` is set on the tracker.
- All subsequent attempts to append events are rejected.
- The session timer is cancelled immediately.

#### How to trigger `SessionEnded`[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#how-to-trigger-sessionended 'Direct link to how-to-trigger-sessionended')

**Via a custom action:**

```
from rasa_sdk import Action, Trackerfrom rasa_sdk.executor import CollectingDispatcherfrom rasa_sdk.events import SessionEndedclass ActionEndSession(Action):    def name(self):        return "action_end_session"    async def run(self, dispatcher, tracker: Tracker, domain):        return [SessionEnded()]
```

When a custom action returns `SessionEnded`, Rasa stops the prediction loop and flow executor immediately. No further actions are predicted or executed.

**Via the REST API:**

```
curl -X POST http://localhost:5005/conversations/{conversation_id}/tracker/events \-H "Content-Type: application/json" \-d '{"event": "session_ended"}'
```

When `SessionEnded` is posted via the API, the session timer is cancelled unconditionally, regardless of whether `execute_side_effects` is set. If an A2A sub agent task is running in the background, it is also cancelled.

Likewise, posting a `ConversationInactive` event through the same endpoint cancels in-flight background A2A processing for that conversation.

### Comparison[​](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#comparison 'Direct link to Comparison')

| Event | Tracker state | Resumable | Triggered by | Further events |
| --- | --- | --- | --- | --- |
| `ConversationInactive` | `inactive = true` | Yes | Inactivity timer | Accepted |
| `SessionEnded` | `terminated = true` | No | Custom action or REST API | Rejected. |

- [Session ID](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#session-id)
 - [When a new `session_id` is generated](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#when-a-new-session_id-is-generated)
 - [`session_id` in event metadata](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#session_id-in-event-metadata)
 - [`current_session_id` on the tracker](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#current_session_id-on-the-tracker)
- [Conversation Lifecycle Events](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#conversation-lifecycle-events)
 - [`ConversationInactive`](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#conversationinactive)
 - [`SessionEnded`](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#sessionended)
 - [Comparison](https://rasa.com/docs/reference/config/session-management/session-lifecycle/#comparison)