# Source: https://rasa.com/docs/reference/config/session-management/overview

On this page

New in 3.16.0

Rasa Pro 3.16 introduces session management capabilities including tracking end users across multiple conversations through the `user_id` field, explicit session lifecycle events, per-session identity via `session_id`, and a proactive inactivity timer with configurable persistence.

Rasa provides comprehensive session management capabilities that allow you to:

- Track individual conversations through unique `sender_id` identifiers.
- Associate multiple conversations with a single end user using `user_id`.
- Retrieve and analyze user behavior across different conversations.
- Manage session lifecycle and state persistence.
- Identify sessions uniquely with an automatically generated `session_id`.
- Configure inactivity timeouts that fire proactively and persist across server restarts.

## Key Concepts[​](https://rasa.com/docs/reference/config/session-management/overview/#key-concepts 'Direct link to Key Concepts')

**What is a conversation?** A conversation is a contained stream of events between a user and the agent (messages, actions, slot updates, flow steps, etc.). It can be paused, resumed, and optionally ended permanently. Once ended with a `SessionEnded` event, it cannot be reopened.

**What is a session?** A session is a bounded slice of a conversation — usually one continuous interaction window (like a single chat visit). A conversation can contain multiple sessions over its lifetime.

**What is the difference between a new session and a new conversation?** A new conversation is a user-facing concept — it's what users expect when they click "New chat" or "Start over": a fresh thread in the UI with no previous context.

A new session is a Rasa system concept — it's how Rasa separates interaction windows for the same user (for example after a timeout). A new session may reset the assistant's state depending on your configuration, but it still belongs to the same end user.

In practice: "new conversation" is the experience you design, while "new session" is the boundary Rasa uses under the hood. They often map 1:1 but don't have to.

## Understanding Session Identifiers[​](https://rasa.com/docs/reference/config/session-management/overview/#understanding-session-identifiers 'Direct link to Understanding Session Identifiers')

Rasa uses three types of identifiers to manage conversations:

### Conversation ID (`sender_id`)[​](https://rasa.com/docs/reference/config/session-management/overview/#conversation-id-sender_id 'Direct link to conversation-id-sender_id')

The `sender_id` uniquely identifies a single conversation. Each conversation between a user and your assistant has its own `sender_id`, which remains constant throughout that specific conversation.

### User ID (`user_id`)[​](https://rasa.com/docs/reference/config/session-management/overview/#user-id-user_id 'Direct link to user-id-user_id')

The `user_id` associates multiple conversations with a single end user. This enables you to:

- Track user behavior across different conversations.
- Retrieve conversation history for a specific user.
- Analyze patterns and preferences across a user's interactions.

The `user_id` is optional and must be provided by your client application.

### Session ID (`session_id`)[​](https://rasa.com/docs/reference/config/session-management/overview/#session-id-session_id 'Direct link to session-id-session_id')

Every session has a unique `session_id` (UUID v4) that is automatically generated and stamped on every event's metadata for the lifetime of that session.

Unlike `sender_id` and `user_id`, `session_id` requires no configuration — it is automatic.

## Session Lifecycle[​](https://rasa.com/docs/reference/config/session-management/overview/#session-lifecycle 'Direct link to Session Lifecycle')

A conversation session begins when:

1. A `SessionStarted` event is triggered (either automatically or manually).
2. A message is received from a user with a new `sender_id`.
3. A `UserUttered` or `ConversationResumed` event is received on an inactive tracker.

A conversation session ends when:

1. A session timer fires due to inactivity and a `ConversationInactive` event is emitted.
2. A `SessionEnded` event is emitted.

Each conversation is stored independently in the tracker store, identified by its unique `sender_id`. Multiple conversations can be associated with the same `user_id`.

## Backward Compatibility[​](https://rasa.com/docs/reference/config/session-management/overview/#backward-compatibility 'Direct link to Backward Compatibility')

1. Existing conversations without `user_id` continue to work seamlessly:

- Trackers without `user_id` will have `user_id=None`.
- Migration of existing data is not required, but recommended if you want to retrieve conversation trackers by user\_id for past conversations prior to the upgrade.
- All tracker stores support both conversations with and without `user_id`.

2. Existing conversations with events that do not have `session_id` in their metadata continue to work seamlessly:

- Old events will not have `session_id` in their metadata.
- New events will have `session_id` in their metadata, and the tracker will have a `current_session_id` field that reflects the active session.
- Existing trackers will have a mix of events with and without `session_id`.

3. If you have a custom tracker store implementation, take a look at the [Rasa Pro Migration Guide](https://rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#support-for-user_id-in-dialoguestatetracker) for details on what changes are required to adopt the new session management feature.

- [Key Concepts](https://rasa.com/docs/reference/config/session-management/overview/#key-concepts)
- [Understanding Session Identifiers](https://rasa.com/docs/reference/config/session-management/overview/#understanding-session-identifiers)
 - [Conversation ID (`sender_id`)](https://rasa.com/docs/reference/config/session-management/overview/#conversation-id-sender_id)
 - [User ID (`user_id`)](https://rasa.com/docs/reference/config/session-management/overview/#user-id-user_id)
 - [Session ID (`session_id`)](https://rasa.com/docs/reference/config/session-management/overview/#session-id-session_id)
- [Session Lifecycle](https://rasa.com/docs/reference/config/session-management/overview/#session-lifecycle)
- [Backward Compatibility](https://rasa.com/docs/reference/config/session-management/overview/#backward-compatibility)