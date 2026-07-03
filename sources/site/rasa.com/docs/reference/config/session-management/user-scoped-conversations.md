# Source: https://rasa.com/docs/reference/config/session-management/user-scoped-conversations

On this page

## Providing User IDs[​](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#providing-user-ids 'Direct link to Providing User IDs')

The `user_id` must be included in the message payload metadata sent from your frontend to Rasa's input channel:

```
{  "sender": "conversation_12345",  "message": "Hello",  "metadata": {    "user_id": "usr_a1b2c3d4e5f6"  }}
```

**Important Security Note**: For data protection, `user_id` should be a random, non-sensitive identifier (such as a UUID). Do not use personally identifiable information like email addresses, phone numbers, or names, as `user_id` values may appear in logs.

The `user_id` is persisted in all supported [tracker stores](https://rasa.com/docs/reference/integrations/tracker-stores/) ([InMemory](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying), [SQL](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-1), [Redis](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-2), [Mongo](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-3) and [Dynamo](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-4)). If `user_id` is `null`, it is omitted from the serialized tracker JSON.

The tracker's current state also includes `user_id`:

```
{  "sender_id": "conversation_12345",  "user_id": "usr_a1b2c3d4e5f6",  "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",  "events": [...]}
```

### `user_id` in event broker payloads[​](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#user_id-in-event-broker-payloads 'Direct link to user_id-in-event-broker-payloads')

Every event streamed to an event broker now includes `user_id` alongside `sender_id`:

```
{  "sender_id": "conversation_12345",  "user_id": "usr_a1b2c3d4e5f6",  "event": "user",  "text": "Hello"}
```

This allows downstream consumers (analytics pipelines, data warehouses, CRMs) to group events by user without additional joins.

## Retrieving Conversations by User[​](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#retrieving-conversations-by-user 'Direct link to Retrieving Conversations by User')

Once `user_id` is set, you can retrieve all conversations for that user via the [tracker API endpoint](https://rasa.com/docs/reference/api/pro/http-api/)(`GET /users/{user_id}/trackers`). The endpoint returns serialized tracker data — including events — read directly from storage without replaying conversation history through `DialogueStateTracker`. Call it on the server-side and expose only what the client needs. This powers use cases like conversation history lists, support dashboards, and "resume a past session" flows.

`current_session_id` in responses

The `current_session_id` field in each returned tracker is derived from the metadata of the last stored event. It is `null` when the last event is `ConversationInactive`, correctly reflecting that no active session exists at the time of the request.

## Backward Compatibility[​](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#backward-compatibility 'Direct link to Backward Compatibility')

To support user-scoped tracker querying, you must implement the `get_trackers_by_user_id()` method in your custom tracker store. This method enables efficient retrieval of all conversations for a specific user with pagination and ordering support. More details can be found in the [Rasa Pro Migration Guide](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#4-implement-get_serialized_trackers_by_user_id-method).

- [Providing User IDs](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#providing-user-ids)
 - [`user_id` in event broker payloads](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#user_id-in-event-broker-payloads)
- [Retrieving Conversations by User](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#retrieving-conversations-by-user)
- [Backward Compatibility](https://rasa.com/docs/reference/config/session-management/user-scoped-conversations/#backward-compatibility)