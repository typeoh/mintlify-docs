# Source: https://rasa.com/docs/reference/api/pro/http-api

- Server Information
 - getHealth endpoint of Rasa Server
 - getInformation about your Rasa Pro License
 - getVersion of Rasa
 - getStatus of the Rasa server
- Tracker
 - getRetrieve a conversations tracker
 - delDelete tracker for a specific conversation
 - getRetrieve all conversations for a user
 - postAppend events to a tracker
 - putReplace a trackers events
 - getRetrieve an end-to-end story corresponding to a conversation
 - postRun an action in a conversation
 - postInject an intent into a conversation
 - postPredict the next action
 - postAdd a message to a tracker
- Model
 - postTrain a Rasa model
 - postEvaluate stories
 - postPerform an intent evaluation
 - postPredict an action on a temporary state
 - postParse a message using the Rasa model
 - putReplace the currently loaded model
 - delUnload the trained model
- Flows
 - getRetrieve the flows of the assistant
- Domain
 - getRetrieve the loaded domain
- Channel Webhooks
 - postPost user message from a REST channel
 - postPost user message from a custom channel

[API docs by Redocly](https://redocly.com/redoc/)

# Rasa - Server Endpoints (1.0.0)

Download OpenAPI specification:[Download](https://rasa.com/docs/redocusaurus/pro.yaml)

The Rasa server provides endpoints to retrieve trackers of conversations as well as endpoints to modify them. Additionally, endpoints for training and testing models are provided.

## Server Information

## Health endpoint of Rasa Server

This URL can be used as an endpoint to run health checks against. When the server is running this will return 200.

### Responses

**200**

Up and running

get/

Local development server

http://localhost:5005/

### Response samples

- 200

Content type

text/plain

Copy

Hello from Rasa: 1.0.0

## Information about your Rasa Pro License

Returns the license information about your Rasa Pro License

### Responses

**200**

Rasa Pro License Information

get/license

Local development server

http://localhost:5005/license

### Response samples

- 200

Content type

application/json

Copy

`{  -   "id": "u5fn8888-e213-4c12-9542-0baslfdkjas",      -   "company": "acme",      -   "scope": "rasa:pro rasa:voice",      -   "email": "acme@email.com",      -   "expires": "2026-01-01T00:00:00+00:00"       }`

## Version of Rasa

Returns the version of Rasa.

### Responses

**200**

Version of Rasa

get/version

Local development server

http://localhost:5005/version

### Response samples

- 200

Content type

application/json

Copy

`{  -   "version": "1.0.0",      -   "minimum_compatible_version": "1.0.0"       }`

## Status of the Rasa server

Information about the server and the currently loaded Rasa model.

##### Authorizations:

_TokenAuth__JWT_

### Responses

**200**

Success

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

get/status

Local development server

http://localhost:5005/status

### Response samples

- 200
- 401
- 403
- 409

Content type

application/json

Copy

`{  -   "model_id": "75a985b7b86d442ca013d61ea4781b22",      -   "model_file": "20190429-103105.tar.gz",      -   "num_active_training_jobs": 2       }`

## Tracker

## Retrieve a conversations tracker

The tracker represents the state of the conversation. The state of the tracker is created by applying a sequence of events, which modify the state. These events can optionally be included in the response.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |
| until | 

number

Default: "None"

Example: until=1559744410

All events previous to the passed timestamp will be replayed. Events that occur exactly at the target time will be included.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

get/conversations/{conversation\_id}/tracker

Local development server

http://localhost:5005/conversations/{conversation\_id}/tracker

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "sender_id": "default",      -   "user_id": "usr_a1b2c3d4e5f6",      -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",      -   "slots": [          -   {                  -   "slot_name": "slot_value"                               }                   ],      -   "latest_message": {          -   "entities": [                  -   {                          -   "start": 0,                              -   "end": 0,                              -   "value": "string",                              -   "entity": "string",                              -   "confidence": 0                                           }                               ],              -   "intent": {                  -   "confidence": 0.6323,                      -   "name": "greet"                               },              -   "intent_ranking": [                  -   {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           }                               ],              -   "text": "Hello!",              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",              -   "metadata": { },              -   "commands": [                  -   {                          -   "command": "start flow",                              -   "flow": "transfer_money"                                           }                               ],              -   "flows_from_semantic_search": [                  -   [                          -   "transfer_money",                              -   0.9035494923591614                                           ]                               ],              -   "flows_in_prompt": [                  -   "transfer_money"                               ]                   },      -   "latest_event_time": 1537645578.314389,      -   "followup_action": "string",      -   "paused": false,      -   "stack": [          -   {                  -   "frame_id": "8UJPHH5C",                      -   "flow_id": "transfer_money",                      -   "step_id": "START",                      -   "frame_type": "regular",                      -   "type": "flow"                               }                   ],      -   "events": [          -   {                  -   "event": "user",                      -   "timestamp": 1774447464.622012,                      -   "metadata": {                          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                              -   "model_id": "80687713793f4d07957e8e51f73df866",                              -   "assistant_id": "my-assistant"                                           },                      -   "text": "string",                      -   "input_channel": "string",                      -   "message_id": "string",                      -   "parse_data": {                          -   "entities": [                                  -   {                                          -   "start": 0,                                              -   "end": 0,                                              -   "value": "string",                                              -   "entity": "string",                                              -   "confidence": 0                                                                   }                                                       ],                              -   "intent": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       },                              -   "intent_ranking": [                                  -   {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       ],                              -   "text": "Hello!",                              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                              -   "metadata": { },                              -   "commands": [                                  -   {                                          -   "command": "start flow",                                              -   "flow": "transfer_money"                                                                   }                                                       ],                              -   "flows_from_semantic_search": [                                  -   [                                          -   "transfer_money",                                              -   0.9035494923591614                                                                   ]                                                       ],                              -   "flows_in_prompt": [                                  -   "transfer_money"                                                       ]                                           },                      -   "anonymized_at": "string"                               }                   ],      -   "latest_input_channel": "rest",      -   "latest_action_name": "action_listen",      -   "latest_action": {          -   "action_name": "string",              -   "action_text": "string"                   },      -   "active_loop": {          -   "name": "restaurant_form"                   }       }`

## Delete tracker for a specific conversation

Deletes the tracker of a conversation.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

### Responses

**204**

Tracker successfully deleted.

**401**

User is not authenticated.

**403**

User has insufficient permission.

**404**

Conversation not found.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

delete/conversations/{conversation\_id}/tracker

Local development server

http://localhost:5005/conversations/{conversation\_id}/tracker

### Response samples

- 401
- 403
- 409
- 500

Content type

application/json

Copy

`{  -   "version": "1.0.0",      -   "status": "failure",      -   "reason": "NotAuthenticated",      -   "message": "User is not authenticated to access resource.",      -   "code": 401       }`

## Retrieve all conversations for a user

Retrieves all conversation trackers for a given user with pagination. All the event data is returned. Requires admin authentication (token or JWT with `role` set to `admin`).

Tracker data is read directly from storage as serialized event dicts without replaying conversation history through `DialogueStateTracker`. This keeps response latency constant regardless of conversation volume.

The `current_session_id` field in each returned tracker is derived from the metadata of the last stored event. It is `null` when the last event is `ConversationInactive`.

Use this endpoint in a backend service to control what data is exposed to clients. Direct client access should be avoided to maintain security and control over sensitive user data.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| user\_id
required

 | 

string

Example: user-123

Unique identifier of the user to retrieve conversations for

 |
| --- | --- |

##### query Parameters

| limit | 
integer

Example: limit=50

Maximum number of conversations to return (for pagination)

 |
| --- | --- |
| offset | 

integer

Example: offset=5

Number of conversations to skip (for pagination)

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

get/users/{user\_id}/trackers

Local development server

http://localhost:5005/users/{user\_id}/trackers

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "conversations": [          -   {                  -   "sender_id": "e6a3b3158a8444dd998aab17b9e1af3c",                      -   "user_id": "load-test-user-C",                      -   "conversation_started_timestamp": 1774447463.67712,                      -   "current_session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                      -   "events": [                          -   {                                  -   "event": "user",                                      -   "timestamp": 1774447464.622012,                                      -   "metadata": {                                          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                              -   "model_id": "80687713793f4d07957e8e51f73df866",                                              -   "assistant_id": "my-assistant"                                                                   },                                      -   "text": "string",                                      -   "input_channel": "string",                                      -   "message_id": "string",                                      -   "parse_data": {                                          -   "entities": [                                                  -   {                                                          -   "start": 0,                                                              -   "end": 0,                                                              -   "value": "string",                                                              -   "entity": "string",                                                              -   "confidence": 0                                                                                           }                                                                               ],                                              -   "intent": {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               },                                              -   "intent_ranking": [                                                  -   {                                                          -   "confidence": 0.6323,                                                              -   "name": "greet"                                                                                           }                                                                               ],                                              -   "text": "Hello!",                                              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                              -   "metadata": { },                                              -   "commands": [                                                  -   {                                                          -   "command": "start flow",                                                              -   "flow": "transfer_money"                                                                                           }                                                                               ],                                              -   "flows_from_semantic_search": [                                                  -   [                                                          -   "transfer_money",                                                              -   0.9035494923591614                                                                                           ]                                                                               ],                                              -   "flows_in_prompt": [                                                  -   "transfer_money"                                                                               ]                                                                   },                                      -   "anonymized_at": "string"                                                       }                                           ]                               }                   ],      -   "limit": 50,      -   "offset": 5       }`

## Append events to a tracker

Appends one or multiple new events to the tracker state of the conversation. Any existing events will be kept and the new events will be appended, updating the existing state. If events are appended to a new conversation ID, the tracker will be initialised with a new session.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |
| output\_channel | 

string

Enum: "latest" "slack" "callback" "facebook" "rocketchat" "telegram" "twilio" "webexteams" "socketio"

Example: output\_channel=slack

The bot's utterances will be forwarded to this channel. It uses the credentials listed in `credentials.yml` to connect. In case the channel does not support this, the utterances will be returned in the response body. Use `latest` to try to send the messages to the latest channel the user used. Currently supported channels are listed in the permitted values for the parameter.

 |
| execute\_side\_effects | 

boolean

Default: false

If `true`, any `BotUttered` event will be forwarded to the channel specified in the `output_channel` parameter. Any `ReminderScheduled` or `ReminderCancelled` event will also be processed.

 |

##### Request Body schema: application/json

required

One of

EventEventList

Any of

UserEventBotEventSessionStartedEventActionEventSlotEventResetSlotsEventRestartEventReminderEventCancelReminderEventPauseEventResumeEventFollowupEventExportEventUndoEventRewindEventAgentEventEntitiesAddedEventUserFeaturizationEventActionExecutionRejectedEventFormValidationEventLoopInterruptedEventFormEventActiveLoopEventStackEventFlowStartedEventFlowCompletedEvent

| event
required

 | 

string

Value: "user"

Event name

 |
| --- | --- |
| timestamp | 

number

Unix timestamp (float) of when the event was applied.

 |
| metadata | 

object

Arbitrary metadata attached to the event by the channel or model, e.g. session\_id, model\_id, assistant\_id, model\_name.

 |
| text | 

string or null

Text of user message.

 |
| input\_channel | 

string or null

 |
| message\_id | 

string or null

 |
| parse\_data | 

object (ParseResult)

NLU parser information. If set, message will not be passed through NLU, but instead this parsing information will be used.

 |
| anonymized\_at | 

string or null

ISO 8601 timestamp of when PII in this event was anonymized, or `null` if the event has not been anonymized.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/conversations/{conversation\_id}/tracker/events

Local development server

http://localhost:5005/conversations/{conversation\_id}/tracker/events

### Request samples

- Payload

Content type

application/json

Example

EventEventListEvent

Copy

Expand all Collapse all

`{  -   "event": "user",      -   "timestamp": 1774447464.622012,      -   "metadata": {          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",              -   "model_id": "80687713793f4d07957e8e51f73df866",              -   "assistant_id": "my-assistant"                   },      -   "text": "string",      -   "input_channel": "string",      -   "message_id": "string",      -   "parse_data": {          -   "entities": [                  -   {                          -   "start": 0,                              -   "end": 0,                              -   "value": "string",                              -   "entity": "string",                              -   "confidence": 0                                           }                               ],              -   "intent": {                  -   "confidence": 0.6323,                      -   "name": "greet"                               },              -   "intent_ranking": [                  -   {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           }                               ],              -   "text": "Hello!",              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",              -   "metadata": { },              -   "commands": [                  -   {                          -   "command": "start flow",                              -   "flow": "transfer_money"                                           }                               ],              -   "flows_from_semantic_search": [                  -   [                          -   "transfer_money",                              -   0.9035494923591614                                           ]                               ],              -   "flows_in_prompt": [                  -   "transfer_money"                               ]                   },      -   "anonymized_at": "string"       }`

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "sender_id": "default",      -   "user_id": "usr_a1b2c3d4e5f6",      -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",      -   "slots": [          -   {                  -   "slot_name": "slot_value"                               }                   ],      -   "latest_message": {          -   "entities": [                  -   {                          -   "start": 0,                              -   "end": 0,                              -   "value": "string",                              -   "entity": "string",                              -   "confidence": 0                                           }                               ],              -   "intent": {                  -   "confidence": 0.6323,                      -   "name": "greet"                               },              -   "intent_ranking": [                  -   {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           }                               ],              -   "text": "Hello!",              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",              -   "metadata": { },              -   "commands": [                  -   {                          -   "command": "start flow",                              -   "flow": "transfer_money"                                           }                               ],              -   "flows_from_semantic_search": [                  -   [                          -   "transfer_money",                              -   0.9035494923591614                                           ]                               ],              -   "flows_in_prompt": [                  -   "transfer_money"                               ]                   },      -   "latest_event_time": 1537645578.314389,      -   "followup_action": "string",      -   "paused": false,      -   "stack": [          -   {                  -   "frame_id": "8UJPHH5C",                      -   "flow_id": "transfer_money",                      -   "step_id": "START",                      -   "frame_type": "regular",                      -   "type": "flow"                               }                   ],      -   "events": [          -   {                  -   "event": "user",                      -   "timestamp": 1774447464.622012,                      -   "metadata": {                          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                              -   "model_id": "80687713793f4d07957e8e51f73df866",                              -   "assistant_id": "my-assistant"                                           },                      -   "text": "string",                      -   "input_channel": "string",                      -   "message_id": "string",                      -   "parse_data": {                          -   "entities": [                                  -   {                                          -   "start": 0,                                              -   "end": 0,                                              -   "value": "string",                                              -   "entity": "string",                                              -   "confidence": 0                                                                   }                                                       ],                              -   "intent": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       },                              -   "intent_ranking": [                                  -   {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       ],                              -   "text": "Hello!",                              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                              -   "metadata": { },                              -   "commands": [                                  -   {                                          -   "command": "start flow",                                              -   "flow": "transfer_money"                                                                   }                                                       ],                              -   "flows_from_semantic_search": [                                  -   [                                          -   "transfer_money",                                              -   0.9035494923591614                                                                   ]                                                       ],                              -   "flows_in_prompt": [                                  -   "transfer_money"                                                       ]                                           },                      -   "anonymized_at": "string"                               }                   ],      -   "latest_input_channel": "rest",      -   "latest_action_name": "action_listen",      -   "latest_action": {          -   "action_name": "string",              -   "action_text": "string"                   },      -   "active_loop": {          -   "name": "restaurant_form"                   }       }`

## Replace a trackers events

Replaces all events of a tracker with the passed list of events. This endpoint should not be used to modify trackers in a production setup, but rather for creating training data.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |

##### Request Body schema: application/json

required

Array

Any of

UserEventBotEventSessionStartedEventActionEventSlotEventResetSlotsEventRestartEventReminderEventCancelReminderEventPauseEventResumeEventFollowupEventExportEventUndoEventRewindEventAgentEventEntitiesAddedEventUserFeaturizationEventActionExecutionRejectedEventFormValidationEventLoopInterruptedEventFormEventActiveLoopEventStackEventFlowStartedEventFlowCompletedEvent

| event
required

 | 

string

Value: "user"

Event name

 |
| --- | --- |
| timestamp | 

number

Unix timestamp (float) of when the event was applied.

 |
| metadata | 

object

Arbitrary metadata attached to the event by the channel or model, e.g. session\_id, model\_id, assistant\_id, model\_name.

 |
| text | 

string or null

Text of user message.

 |
| input\_channel | 

string or null

 |
| message\_id | 

string or null

 |
| parse\_data | 

object (ParseResult)

NLU parser information. If set, message will not be passed through NLU, but instead this parsing information will be used.

 |
| anonymized\_at | 

string or null

ISO 8601 timestamp of when PII in this event was anonymized, or `null` if the event has not been anonymized.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

put/conversations/{conversation\_id}/tracker/events

Local development server

http://localhost:5005/conversations/{conversation\_id}/tracker/events

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`[  -   {          -   "event": "user",              -   "timestamp": 1774447464.622012,              -   "metadata": {                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                      -   "model_id": "80687713793f4d07957e8e51f73df866",                      -   "assistant_id": "my-assistant"                               },              -   "text": "string",              -   "input_channel": "string",              -   "message_id": "string",              -   "parse_data": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "anonymized_at": "string"                   }       ]`

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "sender_id": "default",      -   "user_id": "usr_a1b2c3d4e5f6",      -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",      -   "slots": [          -   {                  -   "slot_name": "slot_value"                               }                   ],      -   "latest_message": {          -   "entities": [                  -   {                          -   "start": 0,                              -   "end": 0,                              -   "value": "string",                              -   "entity": "string",                              -   "confidence": 0                                           }                               ],              -   "intent": {                  -   "confidence": 0.6323,                      -   "name": "greet"                               },              -   "intent_ranking": [                  -   {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           }                               ],              -   "text": "Hello!",              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",              -   "metadata": { },              -   "commands": [                  -   {                          -   "command": "start flow",                              -   "flow": "transfer_money"                                           }                               ],              -   "flows_from_semantic_search": [                  -   [                          -   "transfer_money",                              -   0.9035494923591614                                           ]                               ],              -   "flows_in_prompt": [                  -   "transfer_money"                               ]                   },      -   "latest_event_time": 1537645578.314389,      -   "followup_action": "string",      -   "paused": false,      -   "stack": [          -   {                  -   "frame_id": "8UJPHH5C",                      -   "flow_id": "transfer_money",                      -   "step_id": "START",                      -   "frame_type": "regular",                      -   "type": "flow"                               }                   ],      -   "events": [          -   {                  -   "event": "user",                      -   "timestamp": 1774447464.622012,                      -   "metadata": {                          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                              -   "model_id": "80687713793f4d07957e8e51f73df866",                              -   "assistant_id": "my-assistant"                                           },                      -   "text": "string",                      -   "input_channel": "string",                      -   "message_id": "string",                      -   "parse_data": {                          -   "entities": [                                  -   {                                          -   "start": 0,                                              -   "end": 0,                                              -   "value": "string",                                              -   "entity": "string",                                              -   "confidence": 0                                                                   }                                                       ],                              -   "intent": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       },                              -   "intent_ranking": [                                  -   {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       ],                              -   "text": "Hello!",                              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                              -   "metadata": { },                              -   "commands": [                                  -   {                                          -   "command": "start flow",                                              -   "flow": "transfer_money"                                                                   }                                                       ],                              -   "flows_from_semantic_search": [                                  -   [                                          -   "transfer_money",                                              -   0.9035494923591614                                                                   ]                                                       ],                              -   "flows_in_prompt": [                                  -   "transfer_money"                                                       ]                                           },                      -   "anonymized_at": "string"                               }                   ],      -   "latest_input_channel": "rest",      -   "latest_action_name": "action_listen",      -   "latest_action": {          -   "action_name": "string",              -   "action_text": "string"                   },      -   "active_loop": {          -   "name": "restaurant_form"                   }       }`

## Retrieve an end-to-end story corresponding to a conversation

The story represents the whole conversation in end-to-end format. This can be posted to the '/test/stories' endpoint and used as a test.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| until | 
number

Default: "None"

Example: until=1559744410

All events previous to the passed timestamp will be replayed. Events that occur exactly at the target time will be included.

 |
| --- | --- |
| all\_sessions | 

boolean

Default: false

Whether to fetch all sessions in a conversation, or only the latest session

- `true` - fetch all conversation sessions.
- `false` - \[default\] fetch only the latest conversation session.

 |

### Responses

**200**

Success

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

get/conversations/{conversation\_id}/story

Local development server

http://localhost:5005/conversations/{conversation\_id}/story

### Response samples

- 200
- 401
- 403
- 409
- 500

Content type

text/yml

Copy

\- story: story\_00055028
 steps:
 \- user: |
 hello
 intent: greet
 \- action: utter\_ask\_howcanhelp
 \- user: |
 I'm looking for a \[moderately priced\]{"entity": "price", "value": "moderate"} \[Indian\]{"entity": "cuisine"} restaurant for \[two\]({"entity": "people"}) people
 intent: inform
 \- action: utter\_on\_it
 \- action: utter\_ask\_location

## Run an action in a conversation Deprecated

DEPRECATED. Runs the action, calling the action server if necessary. Any responses sent by the executed action will be forwarded to the channel specified in the output\_channel parameter. If no output channel is specified, any messages that should be sent to the user will be included in the response of this endpoint.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |
| output\_channel | 

string

Enum: "latest" "slack" "callback" "facebook" "rocketchat" "telegram" "twilio" "webexteams" "socketio"

Example: output\_channel=slack

The bot's utterances will be forwarded to this channel. It uses the credentials listed in `credentials.yml` to connect. In case the channel does not support this, the utterances will be returned in the response body. Use `latest` to try to send the messages to the latest channel the user used. Currently supported channels are listed in the permitted values for the parameter.

 |

##### Request Body schema: application/json

required

| name
required

 | 

string

Name of the action to be executed.

 |
| --- | --- |
| policy | 

string or null

Name of the policy that predicted the action.

 |
| confidence | 

number or null

Confidence of the prediction.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/conversations/{conversation\_id}/execute

Local development server

http://localhost:5005/conversations/{conversation\_id}/execute

### Request samples

- Payload

Content type

application/json

Copy

`{  -   "name": "utter_greet",      -   "policy": "string",      -   "confidence": 0.987232       }`

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "tracker": {          -   "sender_id": "default",              -   "user_id": "usr_a1b2c3d4e5f6",              -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",              -   "slots": [                  -   {                          -   "slot_name": "slot_value"                                           }                               ],              -   "latest_message": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "latest_event_time": 1537645578.314389,              -   "followup_action": "string",              -   "paused": false,              -   "stack": [                  -   {                          -   "frame_id": "8UJPHH5C",                              -   "flow_id": "transfer_money",                              -   "step_id": "START",                              -   "frame_type": "regular",                              -   "type": "flow"                                           }                               ],              -   "events": [                  -   {                          -   "event": "user",                              -   "timestamp": 1774447464.622012,                              -   "metadata": {                                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                      -   "model_id": "80687713793f4d07957e8e51f73df866",                                      -   "assistant_id": "my-assistant"                                                       },                              -   "text": "string",                              -   "input_channel": "string",                              -   "message_id": "string",                              -   "parse_data": {                                  -   "entities": [                                          -   {                                                  -   "start": 0,                                                      -   "end": 0,                                                      -   "value": "string",                                                      -   "entity": "string",                                                      -   "confidence": 0                                                                               }                                                                   ],                                      -   "intent": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   },                                      -   "intent_ranking": [                                          -   {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               }                                                                   ],                                      -   "text": "Hello!",                                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                      -   "metadata": { },                                      -   "commands": [                                          -   {                                                  -   "command": "start flow",                                                      -   "flow": "transfer_money"                                                                               }                                                                   ],                                      -   "flows_from_semantic_search": [                                          -   [                                                  -   "transfer_money",                                                      -   0.9035494923591614                                                                               ]                                                                   ],                                      -   "flows_in_prompt": [                                          -   "transfer_money"                                                                   ]                                                       },                              -   "anonymized_at": "string"                                           }                               ],              -   "latest_input_channel": "rest",              -   "latest_action_name": "action_listen",              -   "latest_action": {                  -   "action_name": "string",                      -   "action_text": "string"                               },              -   "active_loop": {                  -   "name": "restaurant_form"                               }                   },      -   "messages": [          -   {                  -   "recipient_id": "string",                      -   "text": "string",                      -   "image": "string",                      -   "buttons": [                          -   {                                  -   "title": "string",                                      -   "payload": "string"                                                       }                                           ],                      -   "attachement": [                          -   {                                  -   "title": "string",                                      -   "payload": "string"                                                       }                                           ]                               }                   ]       }`

## Inject an intent into a conversation

Sends a specified intent and list of entities in place of a user message. The bot then predicts and executes a response action. Any responses sent by the executed action will be forwarded to the channel specified in the `output_channel` parameter. If no output channel is specified, any messages that should be sent to the user will be included in the response of this endpoint.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |
| output\_channel | 

string

Enum: "latest" "slack" "callback" "facebook" "rocketchat" "telegram" "twilio" "webexteams" "socketio"

Example: output\_channel=slack

The bot's utterances will be forwarded to this channel. It uses the credentials listed in `credentials.yml` to connect. In case the channel does not support this, the utterances will be returned in the response body. Use `latest` to try to send the messages to the latest channel the user used. Currently supported channels are listed in the permitted values for the parameter.

 |

##### Request Body schema: application/json

required

| name
required

 | 

string

Name of the intent to be executed.

 |
| --- | --- |
| entities | 

object or null

Entities to be passed on.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/conversations/{conversation\_id}/trigger\_intent

Local development server

http://localhost:5005/conversations/{conversation\_id}/trigger\_intent

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`{  -   "name": "greet",      -   "entities": {          -   "temperature": "high"                   }       }`

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "tracker": {          -   "sender_id": "default",              -   "user_id": "usr_a1b2c3d4e5f6",              -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",              -   "slots": [                  -   {                          -   "slot_name": "slot_value"                                           }                               ],              -   "latest_message": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "latest_event_time": 1537645578.314389,              -   "followup_action": "string",              -   "paused": false,              -   "stack": [                  -   {                          -   "frame_id": "8UJPHH5C",                              -   "flow_id": "transfer_money",                              -   "step_id": "START",                              -   "frame_type": "regular",                              -   "type": "flow"                                           }                               ],              -   "events": [                  -   {                          -   "event": "user",                              -   "timestamp": 1774447464.622012,                              -   "metadata": {                                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                      -   "model_id": "80687713793f4d07957e8e51f73df866",                                      -   "assistant_id": "my-assistant"                                                       },                              -   "text": "string",                              -   "input_channel": "string",                              -   "message_id": "string",                              -   "parse_data": {                                  -   "entities": [                                          -   {                                                  -   "start": 0,                                                      -   "end": 0,                                                      -   "value": "string",                                                      -   "entity": "string",                                                      -   "confidence": 0                                                                               }                                                                   ],                                      -   "intent": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   },                                      -   "intent_ranking": [                                          -   {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               }                                                                   ],                                      -   "text": "Hello!",                                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                      -   "metadata": { },                                      -   "commands": [                                          -   {                                                  -   "command": "start flow",                                                      -   "flow": "transfer_money"                                                                               }                                                                   ],                                      -   "flows_from_semantic_search": [                                          -   [                                                  -   "transfer_money",                                                      -   0.9035494923591614                                                                               ]                                                                   ],                                      -   "flows_in_prompt": [                                          -   "transfer_money"                                                                   ]                                                       },                              -   "anonymized_at": "string"                                           }                               ],              -   "latest_input_channel": "rest",              -   "latest_action_name": "action_listen",              -   "latest_action": {                  -   "action_name": "string",                      -   "action_text": "string"                               },              -   "active_loop": {                  -   "name": "restaurant_form"                               }                   },      -   "messages": [          -   {                  -   "recipient_id": "string",                      -   "text": "string",                      -   "image": "string",                      -   "buttons": [                          -   {                                  -   "title": "string",                                      -   "payload": "string"                                                       }                                           ],                      -   "attachement": [                          -   {                                  -   "title": "string",                                      -   "payload": "string"                                                       }                                           ]                               }                   ]       }`

## Predict the next action

Runs the conversations tracker through the model's policies to predict the scores of all actions present in the model's domain. Actions are returned in the 'scores' array, sorted on their 'score' values. The state of the tracker is not modified.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/conversations/{conversation\_id}/predict

Local development server

http://localhost:5005/conversations/{conversation\_id}/predict

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "scores": [          -   {                  -   "action": "utter_greet",                      -   "score": 1                               }                   ],      -   "policy": "policy_2_TEDPolicy",      -   "confidence": 0.057,      -   "tracker": {          -   "sender_id": "default",              -   "user_id": "usr_a1b2c3d4e5f6",              -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",              -   "slots": [                  -   {                          -   "slot_name": "slot_value"                                           }                               ],              -   "latest_message": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "latest_event_time": 1537645578.314389,              -   "followup_action": "string",              -   "paused": false,              -   "stack": [                  -   {                          -   "frame_id": "8UJPHH5C",                              -   "flow_id": "transfer_money",                              -   "step_id": "START",                              -   "frame_type": "regular",                              -   "type": "flow"                                           }                               ],              -   "events": [                  -   {                          -   "event": "user",                              -   "timestamp": 1774447464.622012,                              -   "metadata": {                                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                      -   "model_id": "80687713793f4d07957e8e51f73df866",                                      -   "assistant_id": "my-assistant"                                                       },                              -   "text": "string",                              -   "input_channel": "string",                              -   "message_id": "string",                              -   "parse_data": {                                  -   "entities": [                                          -   {                                                  -   "start": 0,                                                      -   "end": 0,                                                      -   "value": "string",                                                      -   "entity": "string",                                                      -   "confidence": 0                                                                               }                                                                   ],                                      -   "intent": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   },                                      -   "intent_ranking": [                                          -   {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               }                                                                   ],                                      -   "text": "Hello!",                                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                      -   "metadata": { },                                      -   "commands": [                                          -   {                                                  -   "command": "start flow",                                                      -   "flow": "transfer_money"                                                                               }                                                                   ],                                      -   "flows_from_semantic_search": [                                          -   [                                                  -   "transfer_money",                                                      -   0.9035494923591614                                                                               ]                                                                   ],                                      -   "flows_in_prompt": [                                          -   "transfer_money"                                                                   ]                                                       },                              -   "anonymized_at": "string"                                           }                               ],              -   "latest_input_channel": "rest",              -   "latest_action_name": "action_listen",              -   "latest_action": {                  -   "action_name": "string",                      -   "action_text": "string"                               },              -   "active_loop": {                  -   "name": "restaurant_form"                               }                   }       }`

## Add a message to a tracker

Adds a message to a tracker. This doesn't trigger the prediction loop. It will log the message on the tracker and return, no actions will be predicted or run. This is often used together with the predict endpoint.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| conversation\_id
required

 | 

string

Example: default

Id of the conversation

 |
| --- | --- |

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |

##### Request Body schema: application/json

required

| text
required

 | 

string

Message text

 |
| --- | --- |
| sender

required

 | 

string

Value: "user"

Origin of the message - who sent it

 |
| parse\_data | 

object (ParseResult)

NLU parser information. If set, message will not be passed through NLU, but instead this parsing information will be used.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/conversations/{conversation\_id}/messages

Local development server

http://localhost:5005/conversations/{conversation\_id}/messages

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`{  -   "text": "Hello!",      -   "sender": "user",      -   "parse_data": {          -   "entities": [                  -   {                          -   "start": 0,                              -   "end": 0,                              -   "value": "string",                              -   "entity": "string",                              -   "confidence": 0                                           }                               ],              -   "intent": {                  -   "confidence": 0.6323,                      -   "name": "greet"                               },              -   "intent_ranking": [                  -   {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           }                               ],              -   "text": "Hello!",              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",              -   "metadata": { },              -   "commands": [                  -   {                          -   "command": "start flow",                              -   "flow": "transfer_money"                                           }                               ],              -   "flows_from_semantic_search": [                  -   [                          -   "transfer_money",                              -   0.9035494923591614                                           ]                               ],              -   "flows_in_prompt": [                  -   "transfer_money"                               ]                   }       }`

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "sender_id": "default",      -   "user_id": "usr_a1b2c3d4e5f6",      -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",      -   "slots": [          -   {                  -   "slot_name": "slot_value"                               }                   ],      -   "latest_message": {          -   "entities": [                  -   {                          -   "start": 0,                              -   "end": 0,                              -   "value": "string",                              -   "entity": "string",                              -   "confidence": 0                                           }                               ],              -   "intent": {                  -   "confidence": 0.6323,                      -   "name": "greet"                               },              -   "intent_ranking": [                  -   {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           }                               ],              -   "text": "Hello!",              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",              -   "metadata": { },              -   "commands": [                  -   {                          -   "command": "start flow",                              -   "flow": "transfer_money"                                           }                               ],              -   "flows_from_semantic_search": [                  -   [                          -   "transfer_money",                              -   0.9035494923591614                                           ]                               ],              -   "flows_in_prompt": [                  -   "transfer_money"                               ]                   },      -   "latest_event_time": 1537645578.314389,      -   "followup_action": "string",      -   "paused": false,      -   "stack": [          -   {                  -   "frame_id": "8UJPHH5C",                      -   "flow_id": "transfer_money",                      -   "step_id": "START",                      -   "frame_type": "regular",                      -   "type": "flow"                               }                   ],      -   "events": [          -   {                  -   "event": "user",                      -   "timestamp": 1774447464.622012,                      -   "metadata": {                          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                              -   "model_id": "80687713793f4d07957e8e51f73df866",                              -   "assistant_id": "my-assistant"                                           },                      -   "text": "string",                      -   "input_channel": "string",                      -   "message_id": "string",                      -   "parse_data": {                          -   "entities": [                                  -   {                                          -   "start": 0,                                              -   "end": 0,                                              -   "value": "string",                                              -   "entity": "string",                                              -   "confidence": 0                                                                   }                                                       ],                              -   "intent": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       },                              -   "intent_ranking": [                                  -   {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       ],                              -   "text": "Hello!",                              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                              -   "metadata": { },                              -   "commands": [                                  -   {                                          -   "command": "start flow",                                              -   "flow": "transfer_money"                                                                   }                                                       ],                              -   "flows_from_semantic_search": [                                  -   [                                          -   "transfer_money",                                              -   0.9035494923591614                                                                   ]                                                       ],                              -   "flows_in_prompt": [                                  -   "transfer_money"                                                       ]                                           },                      -   "anonymized_at": "string"                               }                   ],      -   "latest_input_channel": "rest",      -   "latest_action_name": "action_listen",      -   "latest_action": {          -   "action_name": "string",              -   "action_text": "string"                   },      -   "active_loop": {          -   "name": "restaurant_form"                   }       }`

## Model

## Train a Rasa model

Trains a new Rasa model. Depending on the data given only a dialogue model, only a NLU model, or a model combining a trained dialogue model with an NLU model will be trained. The new model is not loaded by default.

##### Authorizations:

_TokenAuth__JWT_

##### query Parameters

| save\_to\_default\_model\_directory | 
boolean

Default: true

If `true` (default) the trained model will be saved in the default model directory, if `false` it will be saved in a temporary directory

 |
| --- | --- |
| force\_training | 

boolean

Default: false

Force a model training even if the data has not changed

 |
| augmentation | 

string

Default: 50

How much data augmentation to use during training

 |
| num\_threads | 

string

Default: 1

Maximum amount of threads to use when training

 |
| callback\_url | 

string

Default: "None"

Example: callback\_url=https://example.com/rasa\_evaluations

If specified the call will return immediately with an empty response and status code 204. The actual result or any errors will be sent to the given callback URL as the body of a post request.

 |

##### Request Body schema: application/yaml

required

The training data should be in YAML format.

| pipeline | 
Array of arrays

Pipeline list

 |
| --- | --- |
| policies | 

Array of arrays

Policies list

 |
| entities | 

Array of arrays

Entity list

 |
| slots | 

Array of arrays

Slots list

 |
| actions | 

Array of arrays

Action list

 |
| forms | 

Array of arrays

Forms list

 |
| e2e\_actions | 

Array of arrays

E2E Action list

 |
| responses | 

object

Bot response templates

 |
| session\_config | 

object

Session configuration options

 |
| nlu | 

Array of arrays

Rasa NLU data, array of intents

 |
| rules | 

Array of arrays

Rule list

 |
| stories | 

Array of arrays

Rasa Core stories in YAML format

 |
| flows | 

object

Rasa Pro flows in YAML format

 |
| force | 

boolean

Deprecated

Force a model training even if the data has not changed

 |
| save\_to\_default\_model\_directory | 

boolean

Deprecated

If `true` (default) the trained model will be saved in the default model directory, if `false` it will be saved in a temporary directory

 |

### Responses

**200**

Zipped Rasa model

**204**

The incoming request specified a `callback_url` and hence the request will return immediately with an empty response. The actual response will be sent to the provided `callback_url` via POST request.

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**500**

An unexpected error occurred.

post/model/train

Local development server

http://localhost:5005/model/train

### Request samples

- Payload

Content type

application/yaml

Copy

pipeline: \[\]

policies: \[\]

intents:
 \- greet
 \- goodbye

entities: \[\]

slots:
 contacts\_list:
 type: text
 mappings:
 \- type: custom
 action: list\_contacts

actions:
 \- list\_contacts

forms: {}
e2e\_actions: \[\]

responses:
 utter\_greet:
 \- text: "Hey! How are you?"

 utter\_goodbye:
 \- text: "Bye"

 utter\_list\_contacts:
 \- text: "You currently have the following contacts:\\n{contacts\_list}"

 utter\_no\_contacts:
 \- text: "You have no contacts in your list."

session\_config:
 session\_expiration\_time: 60
 carry\_over\_slots\_to\_new\_session: true

nlu:
\- intent: greet
 examples: |
 \- hey
 \- hello

\- intent: goodbye
 examples: |
 \- bye
 \- goodbye

rules:

\- rule: Say goodbye anytime the user says goodbye
 steps:
 \- intent: goodbye
 \- action: utter\_goodbye

stories:

\- story: happy path
 steps:
 \- intent: greet
 \- action: utter\_greet
 \- intent: goodbye
 \- action: utter\_goodbye

flows:
 list\_contacts:
 name: list your contacts
 description: show your contact list
 steps:
 \- action: list\_contacts
 next:
 \- if: "slots.contacts\_list"
 then:
 \- action: utter\_list\_contacts
 next: END
 \- else:
 \- action: utter\_no\_contacts
 next: END

### Response samples

- 400
- 401
- 403
- 500

Content type

application/json

Copy

`{  -   "version": "1.0.0",      -   "status": "failure",      -   "reason": "BadRequest",      -   "code": 400       }`

## Evaluate stories

Evaluates one or multiple stories against the currently loaded Rasa model.

##### Authorizations:

_TokenAuth__JWT_

##### query Parameters

| e2e | 
boolean

Default: false

Perform an end-to-end evaluation on the posted stories.

 |
| --- | --- |

##### Request Body schema: text/yml

required

string (StoriesTrainingData)

Rasa Core stories in YAML format

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/model/test/stories

Local development server

http://localhost:5005/model/test/stories

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "actions": [          -   {                  -   "action": "utter_ask_howcanhelp",                      -   "predicted": "utter_ask_howcanhelp",                      -   "policy": "policy_0_MemoizationPolicy",                      -   "confidence": 1                               }                   ],      -   "is_end_to_end_evaluation": true,      -   "precision": 1,      -   "f1": 0.9333333333333333,      -   "accuracy": 0.9,      -   "in_training_data_fraction": 0.8571428571428571,      -   "report": {          -   "conversation_accuracy": {                  -   "accuracy": 0.19047619047619047,                      -   "correct": 18,                      -   "with_warnings": 1,                      -   "total": 20                               },              -   "property1": {                  -   "intent_name": "string",                      -   "classification_report": {                          -   "greet": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100,                                      -   "confused_with": {                                          -   "chitchat": 3,                                              -   "nlu_fallback": 5                                                                   }                                                       },                              -   "micro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "macro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "weightedq avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       }                                           }                               },              -   "property2": {                  -   "intent_name": "string",                      -   "classification_report": {                          -   "greet": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100,                                      -   "confused_with": {                                          -   "chitchat": 3,                                              -   "nlu_fallback": 5                                                                   }                                                       },                              -   "micro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "macro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "weightedq avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       }                                           }                               }                   }       }`

## Perform an intent evaluation

Evaluates NLU model against a model or using cross-validation.

##### Authorizations:

_TokenAuth__JWT_

##### query Parameters

| model | 
string

Example: model=rasa-model.tar.gz

Model that should be used for evaluation. If the parameter is set, the model will be fetched with the currently loaded configuration setup. However, the currently loaded model will not be updated. The state of the server will not change. If the parameter is not set, the currently loaded model will be used for the evaluation.

 |
| --- | --- |
| callback\_url | 

string

Default: "None"

Example: callback\_url=https://example.com/rasa\_evaluations

If specified the call will return immediately with an empty response and status code 204. The actual result or any errors will be sent to the given callback URL as the body of a post request.

 |
| cross\_validation\_folds | 

integer

Default: null

Number of cross validation folds. If this parameter is specified the given training data will be used for a cross-validation instead of using it as test set for the specified model. Note that this is only supported for YAML data.

 |

##### Request Body schema:

application/x-yamlapplication/jsonapplication/x-yaml

required

string

NLU training data and model configuration. The model configuration is only required if cross-validation is used.

### Responses

**200**

NLU evaluation result

**204**

The incoming request specified a `callback_url` and hence the request will return immediately with an empty response. The actual response will be sent to the provided `callback_url` via POST request.

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/model/test/intents

Local development server

http://localhost:5005/model/test/intents

### Request samples

- Payload

Content type

application/x-yamlapplication/jsonapplication/x-yaml

No sample

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "intent_evaluation": {          -   "report": {                  -   "greet": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100,                              -   "confused_with": {                                  -   "chitchat": 3,                                      -   "nlu_fallback": 5                                                       }                                           },                      -   "micro avg": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100                                           },                      -   "macro avg": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100                                           },                      -   "weightedq avg": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100                                           }                               },              -   "accuracy": 0.19047619047619047,              -   "f1_score": 0.06095238095238095,              -   "precision": 0.036281179138321996,              -   "predictions": [                  -   {                          -   "intent": "greet",                              -   "predicted": "greet",                              -   "text": "hey",                              -   "confidence": 0.9973567                                           }                               ],              -   "errors": [                  -   {                          -   "text": "are you alright?",                              -   "intent_response_key_target": "string",                              -   "intent_response_key_prediction": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           }                               ]                   },      -   "response_selection_evaluation": {          -   "report": {                  -   "greet": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100,                              -   "confused_with": {                                  -   "chitchat": 3,                                      -   "nlu_fallback": 5                                                       }                                           },                      -   "micro avg": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100                                           },                      -   "macro avg": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100                                           },                      -   "weightedq avg": {                          -   "precision": 0.123,                              -   "recall": 0.456,                              -   "f1-score": 0.12,                              -   "support": 100                                           }                               },              -   "accuracy": 0.19047619047619047,              -   "f1_score": 0.06095238095238095,              -   "precision": 0.036281179138321996,              -   "predictions": [                  -   {                          -   "intent": "greet",                              -   "predicted": "greet",                              -   "text": "hey",                              -   "confidence": 0.9973567                                           }                               ],              -   "errors": [                  -   {                          -   "text": "are you alright?",                              -   "intent_response_key_target": "string",                              -   "intent_response_key_prediction": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           }                               ]                   },      -   "entity_evaluation": {          -   "property1": {                  -   "report": {                          -   "greet": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100,                                      -   "confused_with": {                                          -   "chitchat": 3,                                              -   "nlu_fallback": 5                                                                   }                                                       },                              -   "micro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "macro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "weightedq avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       }                                           },                      -   "accuracy": 0.19047619047619047,                      -   "f1_score": 0.06095238095238095,                      -   "precision": 0.036281179138321996,                      -   "predictions": [                          -   {                                  -   "intent": "greet",                                      -   "predicted": "greet",                                      -   "text": "hey",                                      -   "confidence": 0.9973567                                                       }                                           ],                      -   "errors": [                          -   {                                  -   "text": "are you alright?",                                      -   "intent_response_key_target": "string",                                      -   "intent_response_key_prediction": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       }                                           ]                               },              -   "property2": {                  -   "report": {                          -   "greet": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100,                                      -   "confused_with": {                                          -   "chitchat": 3,                                              -   "nlu_fallback": 5                                                                   }                                                       },                              -   "micro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "macro avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       },                              -   "weightedq avg": {                                  -   "precision": 0.123,                                      -   "recall": 0.456,                                      -   "f1-score": 0.12,                                      -   "support": 100                                                       }                                           },                      -   "accuracy": 0.19047619047619047,                      -   "f1_score": 0.06095238095238095,                      -   "precision": 0.036281179138321996,                      -   "predictions": [                          -   {                                  -   "intent": "greet",                                      -   "predicted": "greet",                                      -   "text": "hey",                                      -   "confidence": 0.9973567                                                       }                                           ],                      -   "errors": [                          -   {                                  -   "text": "are you alright?",                                      -   "intent_response_key_target": "string",                                      -   "intent_response_key_prediction": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       }                                           ]                               }                   }       }`

## Predict an action on a temporary state

Predicts the next action on the tracker state as it is posted to this endpoint. Rasa will create a temporary tracker from the provided events and will use it to predict an action. No messages will be sent and no action will be run.

##### Authorizations:

_TokenAuth__JWT_

##### query Parameters

| include\_events | 
string

Default: "AFTER\_RESTART"

Enum: "ALL" "APPLIED" "AFTER\_RESTART" "NONE"

Example: include\_events=AFTER\_RESTART

Specify which events of the tracker the response should contain.

- `ALL` - every logged event.
 
- `APPLIED` - only events that contribute to the trackers state. This excludes reverted utterances and actions that got undone.
 
- `AFTER_RESTART` - all events since the last `restarted` event. This includes utterances that got reverted and actions that got undone.
 
- `NONE` - no events.
 

 |
| --- | --- |

##### Request Body schema: application/json

required

Array

Any of

UserEventBotEventSessionStartedEventActionEventSlotEventResetSlotsEventRestartEventReminderEventCancelReminderEventPauseEventResumeEventFollowupEventExportEventUndoEventRewindEventAgentEventEntitiesAddedEventUserFeaturizationEventActionExecutionRejectedEventFormValidationEventLoopInterruptedEventFormEventActiveLoopEventStackEventFlowStartedEventFlowCompletedEvent

| event
required

 | 

string

Value: "user"

Event name

 |
| --- | --- |
| timestamp | 

number

Unix timestamp (float) of when the event was applied.

 |
| metadata | 

object

Arbitrary metadata attached to the event by the channel or model, e.g. session\_id, model\_id, assistant\_id, model\_name.

 |
| text | 

string or null

Text of user message.

 |
| input\_channel | 

string or null

 |
| message\_id | 

string or null

 |
| parse\_data | 

object (ParseResult)

NLU parser information. If set, message will not be passed through NLU, but instead this parsing information will be used.

 |
| anonymized\_at | 

string or null

ISO 8601 timestamp of when PII in this event was anonymized, or `null` if the event has not been anonymized.

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**409**

The request conflicts with the currently loaded model.

**500**

An unexpected error occurred.

post/model/predict

Local development server

http://localhost:5005/model/predict

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`[  -   {          -   "event": "user",              -   "timestamp": 1774447464.622012,              -   "metadata": {                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                      -   "model_id": "80687713793f4d07957e8e51f73df866",                      -   "assistant_id": "my-assistant"                               },              -   "text": "string",              -   "input_channel": "string",              -   "message_id": "string",              -   "parse_data": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "anonymized_at": "string"                   }       ]`

### Response samples

- 200
- 400
- 401
- 403
- 409
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "scores": [          -   {                  -   "action": "utter_greet",                      -   "score": 1                               }                   ],      -   "policy": "policy_2_TEDPolicy",      -   "confidence": 0.057,      -   "tracker": {          -   "sender_id": "default",              -   "user_id": "usr_a1b2c3d4e5f6",              -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",              -   "slots": [                  -   {                          -   "slot_name": "slot_value"                                           }                               ],              -   "latest_message": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "latest_event_time": 1537645578.314389,              -   "followup_action": "string",              -   "paused": false,              -   "stack": [                  -   {                          -   "frame_id": "8UJPHH5C",                              -   "flow_id": "transfer_money",                              -   "step_id": "START",                              -   "frame_type": "regular",                              -   "type": "flow"                                           }                               ],              -   "events": [                  -   {                          -   "event": "user",                              -   "timestamp": 1774447464.622012,                              -   "metadata": {                                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                      -   "model_id": "80687713793f4d07957e8e51f73df866",                                      -   "assistant_id": "my-assistant"                                                       },                              -   "text": "string",                              -   "input_channel": "string",                              -   "message_id": "string",                              -   "parse_data": {                                  -   "entities": [                                          -   {                                                  -   "start": 0,                                                      -   "end": 0,                                                      -   "value": "string",                                                      -   "entity": "string",                                                      -   "confidence": 0                                                                               }                                                                   ],                                      -   "intent": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   },                                      -   "intent_ranking": [                                          -   {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               }                                                                   ],                                      -   "text": "Hello!",                                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                      -   "metadata": { },                                      -   "commands": [                                          -   {                                                  -   "command": "start flow",                                                      -   "flow": "transfer_money"                                                                               }                                                                   ],                                      -   "flows_from_semantic_search": [                                          -   [                                                  -   "transfer_money",                                                      -   0.9035494923591614                                                                               ]                                                                   ],                                      -   "flows_in_prompt": [                                          -   "transfer_money"                                                                   ]                                                       },                              -   "anonymized_at": "string"                                           }                               ],              -   "latest_input_channel": "rest",              -   "latest_action_name": "action_listen",              -   "latest_action": {                  -   "action_name": "string",                      -   "action_text": "string"                               },              -   "active_loop": {                  -   "name": "restaurant_form"                               }                   }       }`

## Parse a message using the Rasa model

Predicts the intent and entities of the message posted to this endpoint. No messages will be stored to a conversation and no action will be run. This will just retrieve the NLU parse results.

##### Authorizations:

_TokenAuth__JWT_

##### query Parameters

| emulation\_mode | 
string

Enum: "WIT" "LUIS" "DIALOGFLOW"

Example: emulation\_mode=LUIS

Specify the emulation mode to use. Emulation mode transforms the response JSON to the format expected by the service specified as the emulation\_mode. Requests must still be sent in the regular Rasa format.

 |
| --- | --- |

##### Request Body schema: application/json

required

| text | 
string

Message to be parsed

 |
| --- | --- |
| message\_id | 

string

Optional ID for message to be parsed

 |

### Responses

**200**

Success

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**500**

An unexpected error occurred.

post/model/parse

Local development server

http://localhost:5005/model/parse

### Request samples

- Payload

Content type

application/json

Copy

`{  -   "text": "Hello, I am Rasa!",      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a"       }`

### Response samples

- 200
- 400
- 401
- 403
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "entities": [          -   {                  -   "start": 0,                      -   "end": 0,                      -   "value": "string",                      -   "entity": "string",                      -   "confidence": 0                               }                   ],      -   "intent": {          -   "confidence": 0.6323,              -   "name": "greet"                   },      -   "text": "Hello!",      -   "commands": [          -   {                  -   "command": "start flow",                      -   "flow": "transfer_money"                               }                   ]       }`

## Replace the currently loaded model

Updates the currently loaded model. First, tries to load the model from the local (note: local to Rasa server) storage system. Secondly, tries to load the model from the provided model server configuration. Last, tries to load the model from the provided remote storage.

##### Authorizations:

_TokenAuth__JWT_

##### Request Body schema: application/json

required

| model\_file | 
string

Path to model file

 |
| --- | --- |
| model\_server | 

object (EndpointConfig)

 |
| remote\_storage | 

string

Enum: "aws" "gcs" "azure"

Name of remote storage system

 |

### Responses

**204**

Model was successfully replaced.

**400**

Bad Request

**401**

User is not authenticated.

**403**

User has insufficient permission.

**500**

An unexpected error occurred.

put/model

Local development server

http://localhost:5005/model

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`{  -   "model_file": "/absolute-path-to-models-directory/models/20190512.tar.gz",      -   "model_server": {          -   "url": "string",              -   "params": { },              -   "headers": { },              -   "basic_auth": { },              -   "token": "string",              -   "token_name": "string",              -   "wait_time_between_pulls": 0                   },      -   "remote_storage": "aws"       }`

### Response samples

- 400
- 401
- 403
- 500

Content type

application/json

Copy

`{  -   "version": "1.0.0",      -   "status": "failure",      -   "reason": "BadRequest",      -   "code": 400       }`

## Unload the trained model

Unloads the currently loaded trained model from the server.

##### Authorizations:

_TokenAuth__JWT_

### Responses

**204**

Model was sucessfully unloaded.

**401**

User is not authenticated.

**403**

User has insufficient permission.

delete/model

Local development server

http://localhost:5005/model

### Response samples

- 401
- 403

Content type

application/json

Copy

`{  -   "version": "1.0.0",      -   "status": "failure",      -   "reason": "NotAuthenticated",      -   "message": "User is not authenticated to access resource.",      -   "code": 401       }`

## Flows

## Retrieve the flows of the assistant

Returns the assistant was trained on.

##### Authorizations:

_TokenAuth__JWT_

### Responses

**200**

Flows were successfully retrieved.

**401**

User is not authenticated.

**403**

User has insufficient permission.

**406**

Invalid header provided.

**500**

An unexpected error occurred.

get/flows

Local development server

http://localhost:5005/flows

### Response samples

- 200
- 401
- 403
- 406
- 500

Content type

application/json

Copy

Expand all Collapse all

`[  -   {          -   "id": "check_balance",              -   "name": "check balance",              -   "description": "check the user's account balance",              -   "steps": [                  -   { }                               ]                   }       ]`

## Domain

## Retrieve the loaded domain

Returns the domain specification the currently loaded model is using.

##### Authorizations:

_TokenAuth__JWT_

### Responses

**200**

Domain was successfully retrieved.

**401**

User is not authenticated.

**403**

User has insufficient permission.

**406**

Invalid header provided.

**500**

An unexpected error occurred.

get/domain

Local development server

http://localhost:5005/domain

### Response samples

- 200
- 401
- 403
- 406
- 500

Content type

application/jsonapplication/yamlapplication/json

Copy

Expand all Collapse all

`{  -   "config": {          -   "store_entities_as_slots": false                   },      -   "intents": [          -   {                  -   "property1": {                          -   "use_entities": true                                           },                      -   "property2": {                          -   "use_entities": true                                           }                               }                   ],      -   "entities": [          -   "person",              -   "location"                   ],      -   "slots": {          -   "property1": {                  -   "auto_fill": true,                      -   "initial_value": "string",                      -   "type": "string",                      -   "values": [                          -   "string"                                           ]                               },              -   "property2": {                  -   "auto_fill": true,                      -   "initial_value": "string",                      -   "type": "string",                      -   "values": [                          -   "string"                                           ]                               }                   },      -   "responses": {          -   "property1": {                  -   "text": "string"                               },              -   "property2": {                  -   "text": "string"                               }                   },      -   "actions": [          -   "action_greet",              -   "action_goodbye",              -   "action_listen"                   ]       }`

## Channel Webhooks

## Post user message from a REST channel

Post a message from the user and forward it to the assistant. Return the message of the assistant to the user.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| rest\_channel
required

 | 

string

Enum: "rest" "callback"

The REST channel used for custom integrations. They provide a URL where you can post messages and either receive response messages directly, or asynchronously via a webhook.

 |
| --- | --- |

##### Request Body schema: application/json

required

The user message payload

| sender | 
string

The sender ID

 |
| --- | --- |
| message | 

string

The message text

 |

### Responses

**200**

The message was processed successfully

**401**

User is not authenticated.

**403**

User has insufficient permission.

**406**

Invalid header provided.

**500**

An unexpected error occurred.

post/webhooks/{rest\_channel}/webhook

Local development server

http://localhost:5005/webhooks/{rest\_channel}/webhook

### Request samples

- Payload

Content type

application/json

Copy

`{  -   "sender": "default",      -   "message": "Hello!"       }`

### Response samples

- 200
- 401
- 403
- 406
- 500

Content type

application/json

Copy

Expand all Collapse all

`[  -   {          -   "recipient_id": "default",              -   "text": "Hello!"                   }       ]`

## Post user message from a custom channel

Post a message from the user and forward it to the assistant. Return the message of the assistant to the user. This is from a custom channel.

##### Authorizations:

_TokenAuth__JWT_

##### path Parameters

| custom\_channel
required

 | 

string

Example: my\_custom\_channel

The custom channel connector used for integration. They provide a URL where you can post and receive messages.

 |
| --- | --- |

##### Request Body schema: application/json

required

The user message payload

| sender | 
string

The sender ID

 |
| --- | --- |
| message | 

string

The message text

 |
| stream | 

boolean

Whether to use streaming response

 |
| input\_channel | 

string

Input channel name

 |
| metadata | 

object

Additional metadata

 |

### Responses

**200**

The message was processed successfully

**401**

User is not authenticated.

**403**

User has insufficient permission.

**406**

Invalid header provided.

**500**

An unexpected error occurred.

post/webhooks/{custom\_channel}/webhook

Local development server

http://localhost:5005/webhooks/{custom\_channel}/webhook

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`{  -   "sender": "string",      -   "message": "string",      -   "stream": true,      -   "input_channel": "string",      -   "metadata": { }       }`

### Response samples

- 200
- 401
- 403
- 406
- 500

Content type

application/json

Copy

Expand all Collapse all

`{  -   "messages": [          -   {                  -   "recipient_id": "string",                      -   "text": "string",                      -   "image": "string",                      -   "buttons": [                          -   {                                  -   "title": "string",                                      -   "payload": "string"                                                       }                                           ],                      -   "attachement": [                          -   {                                  -   "title": "string",                                      -   "payload": "string"                                                       }                                           ]                               }                   ],      -   "metadata": { },      -   "conversation_id": "string",      -   "tracker_state": {          -   "sender_id": "default",              -   "user_id": "usr_a1b2c3d4e5f6",              -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",              -   "slots": [                  -   {                          -   "slot_name": "slot_value"                                           }                               ],              -   "latest_message": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "latest_event_time": 1537645578.314389,              -   "followup_action": "string",              -   "paused": false,              -   "stack": [                  -   {                          -   "frame_id": "8UJPHH5C",                              -   "flow_id": "transfer_money",                              -   "step_id": "START",                              -   "frame_type": "regular",                              -   "type": "flow"                                           }                               ],              -   "events": [                  -   {                          -   "event": "user",                              -   "timestamp": 1774447464.622012,                              -   "metadata": {                                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                      -   "model_id": "80687713793f4d07957e8e51f73df866",                                      -   "assistant_id": "my-assistant"                                                       },                              -   "text": "string",                              -   "input_channel": "string",                              -   "message_id": "string",                              -   "parse_data": {                                  -   "entities": [                                          -   {                                                  -   "start": 0,                                                      -   "end": 0,                                                      -   "value": "string",                                                      -   "entity": "string",                                                      -   "confidence": 0                                                                               }                                                                   ],                                      -   "intent": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   },                                      -   "intent_ranking": [                                          -   {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               }                                                                   ],                                      -   "text": "Hello!",                                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                      -   "metadata": { },                                      -   "commands": [                                          -   {                                                  -   "command": "start flow",                                                      -   "flow": "transfer_money"                                                                               }                                                                   ],                                      -   "flows_from_semantic_search": [                                          -   [                                                  -   "transfer_money",                                                      -   0.9035494923591614                                                                               ]                                                                   ],                                      -   "flows_in_prompt": [                                          -   "transfer_money"                                                                   ]                                                       },                              -   "anonymized_at": "string"                                           }                               ],              -   "latest_input_channel": "rest",              -   "latest_action_name": "action_listen",              -   "latest_action": {                  -   "action_name": "string",                      -   "action_text": "string"                               },              -   "active_loop": {                  -   "name": "restaurant_form"                               }                   }       }`