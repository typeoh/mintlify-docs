# Source: https://rasa.com/docs/reference/config/session-management/session-timer

On this page

Rasa maintains one active timer per conversation. When the timer fires, a `ConversationInactive` event is emitted automatically.

For each active conversation:

1. A timer is scheduled when a session starts.
2. The timer resets every time a user message is received.
3. When the timer fires, `ConversationInactive` is emitted.
4. If `SessionEnded` is received (via custom action or REST API), the timer is immediately cancelled.

### Configuring the timeout[​](https://rasa.com/docs/reference/config/session-management/session-timer/#configuring-the-timeout 'Direct link to Configuring the timeout')

Set `session_expiration_time` in `session_config` in `domain.yml` to control how long user inactivity is allowed before the timer fires:

```
session_config:  session_expiration_time: 60              # minutes; 0 disables session expiry  carry_over_slots_to_new_session: true  start_session_after_expiry: true         # new in 3.16, true by default for backward compatibility
```

### `start_session_after_expiry`[​](https://rasa.com/docs/reference/config/session-management/session-timer/#start_session_after_expiry 'Direct link to start_session_after_expiry')

In both cases (true or false), when the timer fires, the `ConversationInactive` event is emitted and the tracker is marked as inactive. The difference is what happens when the next user message arrives.

**`true` (default — existing behaviour):**

When the next user message arrives after expiry:

1. A session boundary is created (`SessionStarted` is emitted, `action_session_start` runs).
2. A new `session_id` is generated as part of the session boundary and stamped on all subsequent events.

**`false` (new in 3.16):**

When the next user message arrives after expiry:

1. No session boundary is created (`SessionStarted` is not emitted, and `action_session_start` does not run).
2. A new `session_id` is still generated on the `UserUttered` event and stamped on all subsequent events.

note

`carry_over_slots_to_new_session` has no effect when `start_session_after_expiry` is `false`. No session boundary is created, so slots are preserved as-is. Rasa will log a warning if both are set to `false` together.

If you expect your users to return to a conversation after a period of inactivity, set `start_session_after_expiry` to `false` to retain the conversation uninterrupted and avoid unnecessary session boundaries.

### Timer store[​](https://rasa.com/docs/reference/config/session-management/session-timer/#timer-store 'Direct link to Timer store')

Because Rasa maintains one active timer per conversation, those timers need somewhere to live — especially across restarts and in multi-pod deployments. The `timer_store` in `endpoints.yml` controls where they are stored. Read more about timer store options in the [Timer Stores documentation](https://rasa.com/docs/reference/integrations/timer-stores/).

- [Configuring the timeout](https://rasa.com/docs/reference/config/session-management/session-timer/#configuring-the-timeout)
- [`start_session_after_expiry`](https://rasa.com/docs/reference/config/session-management/session-timer/#start_session_after_expiry)
- [Timer store](https://rasa.com/docs/reference/config/session-management/session-timer/#timer-store)