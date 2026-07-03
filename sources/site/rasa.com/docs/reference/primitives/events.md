# Source: https://rasa.com/docs/reference/primitives/events

On this page

Each conversation in Rasa represents a sequence of events. Events are used to track user messages and bot responses, as well as side effects of actions that Rasa takes during a conversation, such as advancing flows or setting slots. Events are emitted by Rasa at different stages of the conversation, and they can be used to analyze and improve your assistant by querying the Rasa [analytics data pipeline](https://rasa.com/docs/reference/integrations/analytics/) or running [end-to-end tests](https://rasa.com/docs/pro/testing/evaluating-assistant/).

Events are stored in the Rasa [tracker store](https://rasa.com/docs/reference/integrations/tracker-stores/) and can be additionally accessed via the Rasa [HTTP API](https://rasa.com/docs/reference/api/pro/http-api/).

## Event Types[​](https://rasa.com/docs/reference/primitives/events/#event-types 'Direct link to Event Types')

Rasa issues different types of events throughout the course of a conversation, among which:

- **[`UserUttered`](https://rasa.com/docs/reference/primitives/events/#user-event)**: Events that represent messages sent by the user.
- **[`BotUttered`](https://rasa.com/docs/reference/primitives/events/#bot-event)**: Events that represent messages sent by the bot.
- **[`ActionExecuted`](https://rasa.com/docs/reference/primitives/events/#action-event)**: Events that represent actions taken by the bot.
- **[`SlotSet`](https://rasa.com/docs/reference/primitives/events/#slot-event)**: Events that represent slots being set.
- **[`AllSlotsReset`](https://rasa.com/docs/reference/primitives/events/#reset-slots-event)**: Events that represent the resetting of all slots.
- **[`SessionStarted`](https://rasa.com/docs/reference/primitives/events/#session-started-event)**: Events that represent the start of a new conversation session.
- **[`Restarted`](https://rasa.com/docs/reference/primitives/events/#restarted-event)**: Events that represent the reset of a conversation session.
- **[`FlowStarted`](https://rasa.com/docs/reference/primitives/events/#flow-started-event)**: Events that represent the start of a flow.
- **[`FlowCompleted`](https://rasa.com/docs/reference/primitives/events/#flow-completed-event)**: Events that represent the completion of a flow.
- **[`FlowCancelled`](https://rasa.com/docs/reference/primitives/events/#flow-cancelled-event)**: Events that represent the cancellation of a flow.
- **[`FlowInterrupted`](https://rasa.com/docs/reference/primitives/events/#flow-interrupted-event)**: Events that represent the interruption of a flow.
- **[`FlowResumed`](https://rasa.com/docs/reference/primitives/events/#flow-resumed-event)**: Events that represent the resuming of a flow.
- **[`RoutingSessionEnded`](https://rasa.com/docs/reference/primitives/events/#routing-session-ended-event)**: Events that mark the end of a routing session for coexistence.
- **[`DialogueStackUpdated`](https://rasa.com/docs/reference/primitives/events/#stack-event)**: Events that represent the update of the conversation stack.
- **[`ReminderScheduled`](https://rasa.com/docs/reference/primitives/events/#reminder-event)**: Events that represent the scheduling of a reminder.
- **[`ReminderCancelled`](https://rasa.com/docs/reference/primitives/events/#cancel-reminder-event)**: Events that represent the cancellation of a reminder.
- **[`ConversationPaused`](https://rasa.com/docs/reference/primitives/events/#pause-event)**: Events that represent the pausing of a conversation.
- **[`ConversationResumed`](https://rasa.com/docs/reference/primitives/events/#resume-event)**: Events that represent the resuming of a conversation.
- **[`ActionExecutionRejected`](https://rasa.com/docs/reference/primitives/events/#action-execution-rejected-event)**: Events that represent the rejection of an action execution.
- **[`EntitiesAdded`](https://rasa.com/docs/reference/primitives/events/#entities-event)**: Events that represent the addition of entities to the conversation state.
- **[`UserUtteranceReverted`](https://rasa.com/docs/reference/primitives/events/#rewind-event)**: Events that represent the reverting of every event which happened after the most recent user message.
- **[`ActionReverted`](https://rasa.com/docs/reference/primitives/events/#undo-event)**: Events that represent the reverting of the bot's last action.
- **[`StoryExported`](https://rasa.com/docs/reference/primitives/events/#export-story-event)**: Events that represent the export of a training data story to file.
- **[`FollowupAction`](https://rasa.com/docs/reference/primitives/events/#followup-action-event)**: Events that represent the enqueueing of a follow-up action.
- **[`ActiveLoop`](https://rasa.com/docs/reference/primitives/events/#active-loop-event)**: Events that represent the activation of a form.
- **[`LoopInterrupted`](https://rasa.com/docs/reference/primitives/events/#loop-interrupted-event)**: Events that represent the interruption of a form.
- **[`SessionEnded`](https://rasa.com/docs/reference/primitives/events/#session-ended-event)**: Events that represent the end of a conversation session.
- **[`AgentStarted`](https://rasa.com/docs/reference/primitives/events/#agent-started-event)**: Events that represent the start of a sub agent.
- **[`AgentCompleted`](https://rasa.com/docs/reference/primitives/events/#agent-completed-event)**: Events that represent the completion of a sub agent.
- **[`AgentInterrupted`](https://rasa.com/docs/reference/primitives/events/#agent-interrupted-event)**: Events that represent the interruption of a sub agent.
- **[`AgentCancelled`](https://rasa.com/docs/reference/primitives/events/#agent-cancelled-event)**: Events that represent the cancellation of a sub agent.
- **[`AgentResumed`](https://rasa.com/docs/reference/primitives/events/#agent-resumed-event)**: Events that represent the resumption of a sub agent.

All events share the following attributes:

- type name: The type of the event.
- timestamp: The timestamp of the event.
- metadata: Additional metadata about the event.

### User Event[​](https://rasa.com/docs/reference/primitives/events/#user-event 'Direct link to User Event')

The `UserUttered` event represents a message sent by the user and its type name is `user`. It contains the following additional attributes:

- `text`: The text of the user message.
- `intent`: The intent of the user message if applicable.
- `entities`: The entities extracted from the user message if applicable.
- `parse_data`: The parsed NLU data of the user message, including the intent and entities and commands.
- `input_channel`: The channel through which the user message was received.
- `message_id`: The unique identifier of the user message.
- `anonymized_at`: The timestamp when the PII data in the event was anonymized in the tracker store. The field is set to `null` if the PII data has not been anonymized.

### Bot Event[​](https://rasa.com/docs/reference/primitives/events/#bot-event 'Direct link to Bot Event')

The `BotUttered` event represents a message sent by the bot and its type name is `bot`. It contains the following additional attributes:

- `text`: The text of the bot message.
- `data`: Additional data for more complex bot utterances (for example buttons)
- `anonymized_at`: The timestamp when the PII data in the event was anonymized in the tracker store. The field is set to `null` if the PII data has not been anonymized.

### Slot Event[​](https://rasa.com/docs/reference/primitives/events/#slot-event 'Direct link to Slot Event')

The `SlotSet` event represents the setting of a slot and its type name is `slot`. It contains the following additional attributes:

- `name`: The name of the slot.
- `value`: The value of the slot.
- `anonymized_at`: The timestamp when the PII data in the event was anonymized in the tracker store. The field is set to `null` if the PII data has not been anonymized.

### Reset Slots Event[​](https://rasa.com/docs/reference/primitives/events/#reset-slots-event 'Direct link to Reset Slots Event')

The `AllSlotsReset` event represents the resetting of all slots and its type name is `reset_slots`. It does not contain any additional attributes.

### Entities Event[​](https://rasa.com/docs/reference/primitives/events/#entities-event 'Direct link to Entities Event')

The `EntitiesAdded` event represents the addition of entities to the conversation state and its type name is `entities`. It contains the following additional attributes:

- `entities`: The list of entities added to the conversation state.

### Action Event[​](https://rasa.com/docs/reference/primitives/events/#action-event 'Direct link to Action Event')

The `ActionExecuted` event represents the execution of an action and its type name is `action`. It contains the following additional attributes:

- `name`: The name of the action.
- `policy`: The policy used to predict the action.
- `confidence`: The confidence of the action prediction.

### Followup Action Event[​](https://rasa.com/docs/reference/primitives/events/#followup-action-event 'Direct link to Followup Action Event')

The `FollowupAction` event represents the enqueueing of a follow-up action and its type name is `followup`. It contains the following additional attributes:

- `name`: The name of the follow-up action to run.

### Action Execution Rejected Event[​](https://rasa.com/docs/reference/primitives/events/#action-execution-rejected-event 'Direct link to Action Execution Rejected Event')

The `ActionExecutionRejected` event represents the rejection of an action execution and its type name is `action_execution_rejected`. It contains the following additional attributes:

- `name`: The name of the action that was rejected.
- `policy`: The policy used to predict the action.
- `confidence`: The confidence of the action prediction.

### Session Started Event[​](https://rasa.com/docs/reference/primitives/events/#session-started-event 'Direct link to Session Started Event')

The `SessionStarted` event represents the start of a new conversation session and its type name is `session_started`. It does not contain any additional attributes.

### Restarted Event[​](https://rasa.com/docs/reference/primitives/events/#restarted-event 'Direct link to Restarted Event')

The `Restarted` event represents the reset of a conversation session and its type name is `restart`. It does not contain any additional attributes.

### Stack Event[​](https://rasa.com/docs/reference/primitives/events/#stack-event 'Direct link to Stack Event')

The `DialogueStackUpdated` event represents the update of the conversation stack and its type name is `stack`. It contains the following additional attributes:

- `update`: JsonPatch object dumped as a string.

### Routing Session Ended Event[​](https://rasa.com/docs/reference/primitives/events/#routing-session-ended-event 'Direct link to Routing Session Ended Event')

The `RoutingSessionEnded` event represents the end of a routing session for coexistence and its type name is `routing_session_ended`. It does not contain any additional attributes.

### Flow Started Event[​](https://rasa.com/docs/reference/primitives/events/#flow-started-event 'Direct link to Flow Started Event')

The `FlowStarted` event represents the start of a flow and its type name is `flow_started`. It contains the following additional attributes:

- `flow_id`: The id of the flow.

### Flow Interrupted Event[​](https://rasa.com/docs/reference/primitives/events/#flow-interrupted-event 'Direct link to Flow Interrupted Event')

The `FlowInterrupted` event represents the interruption of a flow and its type name is `flow_interrupted`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `step_id`: The id of the step where the flow was interrupted.

### Flow Resumed Event[​](https://rasa.com/docs/reference/primitives/events/#flow-resumed-event 'Direct link to Flow Resumed Event')

The `FlowResumed` event represents the resuming of a flow and its type name is `flow_resumed`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `step_id`: The id of the step where the flow was resumed.

### Flow Completed Event[​](https://rasa.com/docs/reference/primitives/events/#flow-completed-event 'Direct link to Flow Completed Event')

The `FlowCompleted` event represents the completion of a flow and its type name is `flow_completed`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `step_id`: The id of the step where the flow was completed.

### Flow Cancelled Event[​](https://rasa.com/docs/reference/primitives/events/#flow-cancelled-event 'Direct link to Flow Cancelled Event')

The `FlowCancelled` event represents the cancellation of a flow and its type name is `flow_cancelled`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `step_id`: The id of the step where the flow was cancelled.

### Reminder Event[​](https://rasa.com/docs/reference/primitives/events/#reminder-event 'Direct link to Reminder Event')

The `ReminderScheduled` event represents the scheduling of a reminder and its type name is `reminder`. It contains the following additional attributes:

- `intent`: The name of the intent to be triggered.
- `entities`: The entities to be used when triggering the intent.
- `date_time`: The date and time when the reminder should be triggered.
- `name`: The name of the reminder.
- `kill_on_user_message`: Whether the reminder should be cancelled if the user sends a message before the trigger date.

### Cancel Reminder Event[​](https://rasa.com/docs/reference/primitives/events/#cancel-reminder-event 'Direct link to Cancel Reminder Event')

The `ReminderCancelled` event represents the cancellation of a reminder and its type name is `cancel_reminder`. It contains the following additional attributes:

- `name`: The name of the reminder to be cancelled.
- `intent`: The name of the intent to be used to identify the reminder to be cancelled.
- `entities`: The entities to be used to identify the reminder to be cancelled.

### Rewind Event[​](https://rasa.com/docs/reference/primitives/events/#rewind-event 'Direct link to Rewind Event')

The `UserUtteranceReverted` event represents the reverting of every event which happened after the most recent user message and its type name is `rewind`. It does not contain any additional attributes.

### Undo Event[​](https://rasa.com/docs/reference/primitives/events/#undo-event 'Direct link to Undo Event')

The `ActionReverted` event represents the reverting of the bot's last action and its type name is `undo`. It does not contain any additional attributes.

### Export Story Event[​](https://rasa.com/docs/reference/primitives/events/#export-story-event 'Direct link to Export Story Event')

The `StoryExported` event represents the export of a training data story to file and its type name is `export`. It contains the following additional attributes:

- `path`: The path to the exported story file.

### Pause Event[​](https://rasa.com/docs/reference/primitives/events/#pause-event 'Direct link to Pause Event')

The `ConversationPaused` event represents the pausing of a conversation and its type name is `pause`. It does not contain any additional attributes.

### Resume Event[​](https://rasa.com/docs/reference/primitives/events/#resume-event 'Direct link to Resume Event')

The `ConversationResumed` event represents the resuming of a conversation and its type name is `resume`. It does not contain any additional attributes.

### Active Loop Event[​](https://rasa.com/docs/reference/primitives/events/#active-loop-event 'Direct link to Active Loop Event')

The `ActiveLoop` event represents the activation of a form and its type name is `active_loop`. It contains the following additional attributes:

- `name`: The name of the form.

### Loop Interrupted Event[​](https://rasa.com/docs/reference/primitives/events/#loop-interrupted-event 'Direct link to Loop Interrupted Event')

The `LoopInterrupted` event represents the interruption of a form and its type name is `loop_interrupted`. It contains the following additional attributes:

- `is_interrupted`: boolean indicating if the form execution was interrupted.

### Session Ended Event[​](https://rasa.com/docs/reference/primitives/events/#session-ended-event 'Direct link to Session Ended Event')

The `SessionEnded` event represents the end of a session and its type name is `session_ended`. Rasa adds `_reason` key to the `metadata` attribute of the event with the reason for the session end.

### Agent Started Event[​](https://rasa.com/docs/reference/primitives/events/#agent-started-event 'Direct link to Agent Started Event')

The `AgentStarted` event represents the start of a sub agent and its type name is `agent_started`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `agent_id`: The id of the sub agent.
- `context_id`: The context id needed for external sub agents connected via A2A.

### Agent Completed Event[​](https://rasa.com/docs/reference/primitives/events/#agent-completed-event 'Direct link to Agent Completed Event')

The `AgentCompleted` event represents the completion of a sub agent and its type name is `agent_completed`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `agent_id`: The id of the sub agent.
- `status`: The final status of the sub agent.

### Agent Interrupted Event[​](https://rasa.com/docs/reference/primitives/events/#agent-interrupted-event 'Direct link to Agent Interrupted Event')

The `AgentInterrupted` event represents the interruption of a sub agent and its type name is `agent_interrupted`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `agent_id`: The id of the sub agent.

### Agent Cancelled Event[​](https://rasa.com/docs/reference/primitives/events/#agent-cancelled-event 'Direct link to Agent Cancelled Event')

The `AgentCancelled` event represents the cancellation of a sub agent and its type name is `agent_cancelled`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `agent_id`: The id of the sub agent.
- `reason`: The reason for the cancellation.

### Agent Resumed Event[​](https://rasa.com/docs/reference/primitives/events/#agent-resumed-event 'Direct link to Agent Resumed Event')

The `AgentResumed` event represents the resuming of a sub agent and its type name is `agent_resumed`. It contains the following additional attributes:

- `flow_id`: The id of the flow.
- `agent_id`: The id of the sub agent.

- [Event Types](https://rasa.com/docs/reference/primitives/events/#event-types)
 - [User Event](https://rasa.com/docs/reference/primitives/events/#user-event)
 - [Bot Event](https://rasa.com/docs/reference/primitives/events/#bot-event)
 - [Slot Event](https://rasa.com/docs/reference/primitives/events/#slot-event)
 - [Reset Slots Event](https://rasa.com/docs/reference/primitives/events/#reset-slots-event)
 - [Entities Event](https://rasa.com/docs/reference/primitives/events/#entities-event)
 - [Action Event](https://rasa.com/docs/reference/primitives/events/#action-event)
 - [Followup Action Event](https://rasa.com/docs/reference/primitives/events/#followup-action-event)
 - [Action Execution Rejected Event](https://rasa.com/docs/reference/primitives/events/#action-execution-rejected-event)
 - [Session Started Event](https://rasa.com/docs/reference/primitives/events/#session-started-event)
 - [Restarted Event](https://rasa.com/docs/reference/primitives/events/#restarted-event)
 - [Stack Event](https://rasa.com/docs/reference/primitives/events/#stack-event)
 - [Routing Session Ended Event](https://rasa.com/docs/reference/primitives/events/#routing-session-ended-event)
 - [Flow Started Event](https://rasa.com/docs/reference/primitives/events/#flow-started-event)
 - [Flow Interrupted Event](https://rasa.com/docs/reference/primitives/events/#flow-interrupted-event)
 - [Flow Resumed Event](https://rasa.com/docs/reference/primitives/events/#flow-resumed-event)
 - [Flow Completed Event](https://rasa.com/docs/reference/primitives/events/#flow-completed-event)
 - [Flow Cancelled Event](https://rasa.com/docs/reference/primitives/events/#flow-cancelled-event)
 - [Reminder Event](https://rasa.com/docs/reference/primitives/events/#reminder-event)
 - [Cancel Reminder Event](https://rasa.com/docs/reference/primitives/events/#cancel-reminder-event)
 - [Rewind Event](https://rasa.com/docs/reference/primitives/events/#rewind-event)
 - [Undo Event](https://rasa.com/docs/reference/primitives/events/#undo-event)
 - [Export Story Event](https://rasa.com/docs/reference/primitives/events/#export-story-event)
 - [Pause Event](https://rasa.com/docs/reference/primitives/events/#pause-event)
 - [Resume Event](https://rasa.com/docs/reference/primitives/events/#resume-event)
 - [Active Loop Event](https://rasa.com/docs/reference/primitives/events/#active-loop-event)
 - [Loop Interrupted Event](https://rasa.com/docs/reference/primitives/events/#loop-interrupted-event)
 - [Session Ended Event](https://rasa.com/docs/reference/primitives/events/#session-ended-event)
 - [Agent Started Event](https://rasa.com/docs/reference/primitives/events/#agent-started-event)
 - [Agent Completed Event](https://rasa.com/docs/reference/primitives/events/#agent-completed-event)
 - [Agent Interrupted Event](https://rasa.com/docs/reference/primitives/events/#agent-interrupted-event)
 - [Agent Cancelled Event](https://rasa.com/docs/reference/primitives/events/#agent-cancelled-event)
 - [Agent Resumed Event](https://rasa.com/docs/reference/primitives/events/#agent-resumed-event)