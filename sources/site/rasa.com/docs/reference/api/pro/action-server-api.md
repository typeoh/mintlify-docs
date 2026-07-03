# Source: https://rasa.com/docs/reference/api/pro/action-server-api

- postCore request to execute a custom action

[API docs by Redocly](https://redocly.com/redoc/)

# Rasa SDK - Action Server Endpoint (0.0.0)

Download OpenAPI specification:[Download](https://rasa.com/docs/redocusaurus/action-server.yaml)

HTTP API of the action server which is used by Rasa to execute custom actions.

## Core request to execute a custom action

Rasa Core sends a request to the action server to execute a certain custom action. As a response to the action call from Core, you can modify the tracker, e.g. by setting slots and send responses back to the user.

##### Request Body schema: application/json

required

Describes the action to be called and provides information on the current state of the conversation.

| next\_action | 
string

The name of the action which should be executed.

 |
| --- | --- |
| sender\_id | 

string

Unique id of the user who is having the current conversation.

 |
| tracker | 

object (Tracker)

Conversation tracker which stores the conversation state.

 |
| domain | 

object (Domain)

The bot's domain.

 |

### Responses

**200**

Action was executed succesfully.

**400**

Action execution was rejected. This is the same as returning an `ActionExecutionRejected` event.

**500**

The action server encountered an exception while running the action.

post/

Local development action server

http://localhost:5055/webhook/

### Request samples

- Payload

Content type

application/json

Copy

Expand all Collapse all

`{  -   "next_action": "string",      -   "sender_id": "string",      -   "tracker": {          -   "sender_id": "default",              -   "user_id": "usr_a1b2c3d4e5f6",              -   "current_session_id": "4b3c1e2a-91f0-4d8e-b123-abc123def456",              -   "slots": [                  -   {                          -   "slot_name": "slot_value"                                           }                               ],              -   "latest_message": {                  -   "entities": [                          -   {                                  -   "start": 0,                                      -   "end": 0,                                      -   "value": "string",                                      -   "entity": "string",                                      -   "confidence": 0                                                       }                                           ],                      -   "intent": {                          -   "confidence": 0.6323,                              -   "name": "greet"                                           },                      -   "intent_ranking": [                          -   {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       }                                           ],                      -   "text": "Hello!",                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                      -   "metadata": { },                      -   "commands": [                          -   {                                  -   "command": "start flow",                                      -   "flow": "transfer_money"                                                       }                                           ],                      -   "flows_from_semantic_search": [                          -   [                                  -   "transfer_money",                                      -   0.9035494923591614                                                       ]                                           ],                      -   "flows_in_prompt": [                          -   "transfer_money"                                           ]                               },              -   "latest_event_time": 1537645578.314389,              -   "followup_action": "string",              -   "paused": false,              -   "stack": [                  -   {                          -   "frame_id": "8UJPHH5C",                              -   "flow_id": "transfer_money",                              -   "step_id": "START",                              -   "frame_type": "regular",                              -   "type": "flow"                                           }                               ],              -   "events": [                  -   {                          -   "event": "user",                              -   "timestamp": 1774447464.622012,                              -   "metadata": {                                  -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                                      -   "model_id": "80687713793f4d07957e8e51f73df866",                                      -   "assistant_id": "my-assistant"                                                       },                              -   "text": "string",                              -   "input_channel": "string",                              -   "message_id": "string",                              -   "parse_data": {                                  -   "entities": [                                          -   {                                                  -   "start": 0,                                                      -   "end": 0,                                                      -   "value": "string",                                                      -   "entity": "string",                                                      -   "confidence": 0                                                                               }                                                                   ],                                      -   "intent": {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   },                                      -   "intent_ranking": [                                          -   {                                                  -   "confidence": 0.6323,                                                      -   "name": "greet"                                                                               }                                                                   ],                                      -   "text": "Hello!",                                      -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                                      -   "metadata": { },                                      -   "commands": [                                          -   {                                                  -   "command": "start flow",                                                      -   "flow": "transfer_money"                                                                               }                                                                   ],                                      -   "flows_from_semantic_search": [                                          -   [                                                  -   "transfer_money",                                                      -   0.9035494923591614                                                                               ]                                                                   ],                                      -   "flows_in_prompt": [                                          -   "transfer_money"                                                                   ]                                                       },                              -   "anonymized_at": "string"                                           }                               ],              -   "latest_input_channel": "rest",              -   "latest_action_name": "action_listen",              -   "latest_action": {                  -   "action_name": "string",                      -   "action_text": "string"                               },              -   "active_loop": {                  -   "name": "restaurant_form"                               }                   },      -   "domain": {          -   "config": {                  -   "store_entities_as_slots": false                               },              -   "intents": [                  -   {                          -   "property1": {                                  -   "use_entities": true                                                       },                              -   "property2": {                                  -   "use_entities": true                                                       }                                           }                               ],              -   "entities": [                  -   "person",                      -   "location"                               ],              -   "slots": {                  -   "property1": {                          -   "auto_fill": true,                              -   "initial_value": "string",                              -   "type": "string",                              -   "values": [                                  -   "string"                                                       ]                                           },                      -   "property2": {                          -   "auto_fill": true,                              -   "initial_value": "string",                              -   "type": "string",                              -   "values": [                                  -   "string"                                                       ]                                           }                               },              -   "responses": {                  -   "property1": {                          -   "text": "string"                                           },                      -   "property2": {                          -   "text": "string"                                           }                               },              -   "actions": [                  -   "action_greet",                      -   "action_goodbye",                      -   "action_listen"                               ]                   }       }`

### Response samples

- 200
- 400

Content type

application/json

Copy

Expand all Collapse all

`{  -   "events": [          -   {                  -   "event": "user",                      -   "timestamp": 1774447464.622012,                      -   "metadata": {                          -   "session_id": "eed351b9-a996-4bb8-83ea-1d18e7cd4905",                              -   "model_id": "80687713793f4d07957e8e51f73df866",                              -   "assistant_id": "my-assistant"                                           },                      -   "text": "string",                      -   "input_channel": "string",                      -   "message_id": "string",                      -   "parse_data": {                          -   "entities": [                                  -   {                                          -   "start": 0,                                              -   "end": 0,                                              -   "value": "string",                                              -   "entity": "string",                                              -   "confidence": 0                                                                   }                                                       ],                              -   "intent": {                                  -   "confidence": 0.6323,                                      -   "name": "greet"                                                       },                              -   "intent_ranking": [                                  -   {                                          -   "confidence": 0.6323,                                              -   "name": "greet"                                                                   }                                                       ],                              -   "text": "Hello!",                              -   "message_id": "b2831e73-1407-4ba0-a861-0f30a42a2a5a",                              -   "metadata": { },                              -   "commands": [                                  -   {                                          -   "command": "start flow",                                              -   "flow": "transfer_money"                                                                   }                                                       ],                              -   "flows_from_semantic_search": [                                  -   [                                          -   "transfer_money",                                              -   0.9035494923591614                                                                   ]                                                       ],                              -   "flows_in_prompt": [                                  -   "transfer_money"                                                       ]                                           },                      -   "anonymized_at": "string"                               }                   ],      -   "responses": [          -   {                  -   "text": "string"                               }                   ]       }`