# Source: https://rasa.com/docs/reference/integrations/action-server/actions

On this page

When a Rasa assistant calls a custom action, it sends a request to the action server. Rasa only knows about whatever events and responses come back in the request response; it's up to the action server to call the correct code based on the action name that Rasa provides.

To better understand what happens when Rasa calls a custom action, consider the following example:

You have deployed a weather bot to both Facebook and Slack. The user can ask for the weather with the intent `ask_weather`. There is a slot `location` which will be filled if the user has specified a location. The action `action_tell_weather` will use an API to get the weather forecast , using a default location if the user doesn't specify one. The action will set the `temperature` to the maximum temperature of the weather forecast. The message returned will differ according to the channel they are using.

## Protocols supported by the action server[​](https://rasa.com/docs/reference/integrations/action-server/actions/#protocols-supported-by-the-action-server 'Direct link to Protocols supported by the action server')

Action server supports two protocols for communication with Rasa server:

- [HTTP API](https://rasa.com/docs/reference/integrations/action-server/actions/#http-protocol)
- [gRPC](https://rasa.com/docs/reference/integrations/action-server/actions/#grpc-protocol)

### HTTP(S) Protocol[​](https://rasa.com/docs/reference/integrations/action-server/actions/#https-protocol 'Direct link to HTTP(S) Protocol')

API specification for HTTP protocol can be found [here](https://rasa.com/docs/reference/api/pro/action-server-api/).

Compressed body in HTTP requests

Rasa has the capability to compress the HTTP request body for custom actions. By default, this option is off to keep backward compatibility with older versions of custom action servers which did not receive the compressed body in the HTTP request for custom actions. To enable this option, set environment variable COMPRESS\_ACTION\_SERVER\_REQUEST to `True`.

Rasa SDK versions `3.2.2`, `3.3.1`, `3.4.1` and upwards, support both compressed and non-compressed body in the HTTP request for running custom action server. There is no additional setup required.

#### HTTP Protocol[​](https://rasa.com/docs/reference/integrations/action-server/actions/#http-protocol 'Direct link to HTTP Protocol')

To connect to the action server using the HTTP protocol, you need to set protocol schema to `http` in the `url` of the `action_endpoint` in your `endpoints.yml` file:

```
action_endpoint:  url: "http://localhost:5055/webhook"
```

When using the HTTP protocol, make sure that [the action server is started with HTTP enabled](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#http-protocol).

#### HTTPS Protocol[​](https://rasa.com/docs/reference/integrations/action-server/actions/#https-protocol-1 'Direct link to HTTPS Protocol')

To use the HTTPS protocol to connect to action server, set the protocol schema to `https` in the `url` of the `action_endpoint` and provide CA certificate in `cafile` field in your `endpoints.yml` file:

```
action_endpoint:  url: "https://localhost:5055/webhook"  cafile: "/path/to/ssl_ca_certificate"
```

When using the HTTPS protocol, make sure that:

- [the action server is started with HTTPS enabled](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#https-protocol)
- the certificate is signed by a trusted CA
- the server certificate has a list of hosts set correctly

### GRPC Protocol[​](https://rasa.com/docs/reference/integrations/action-server/actions/#grpc-protocol 'Direct link to GRPC Protocol')

New in 3.9

API specification for gRPC protocol can be found [here](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/). Data schema for gRPC protocol is closely aligned with the HTTP protocol.

We use compression by default in gRPC protocol. It cannot be disabled.

We support both secure (TLS) and insecure gRPC connections.

#### Insecure gRPC connection[​](https://rasa.com/docs/reference/integrations/action-server/actions/#insecure-grpc-connection 'Direct link to Insecure gRPC connection')

To connect to the action server over insecure gRPC connection, you need to set the protocol schema to `grpc` in the `url` field of the `action_endpoint` in your `endpoints.yml` file:

```
action_endpoint:  url: "grpc://localhost:5055"
```

When using an insecure gRPC connection, make sure that [the action server is started with insecure connection enabled](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#insecure-grpc-connection).

#### Secure (TLS) gRPC connection[​](https://rasa.com/docs/reference/integrations/action-server/actions/#secure-tls-grpc-connection 'Direct link to Secure (TLS) gRPC connection')

To connect to the action server over a secure gRPC connection, you need to provide the path to the SSL CA certificate, in the `cafile` field in your `endpoints.yml` file:

```
action_endpoint:  url: "grpc://localhost:5055"  cafile: "/path/to/ssl_ca_certificate"
```

When using a secure gRPC connection, make sure that:

- [the action server is started with secure connection enabled](https://rasa.com/docs/reference/integrations/action-server/running-action-server/#secure-grpc-connection)
- the certificate is signed by a trusted CA
- the server certificate has list of hosts set correctly

## Custom Action Input[​](https://rasa.com/docs/reference/integrations/action-server/actions/#custom-action-input 'Direct link to Custom Action Input')

Note: HTTP protocol is used in the example below. The input format for gRPC protocol follows a similar data layout. For example, the json payload below can be serialized to the gRPC request object and send it to the action server over gRPC protocol.

- Input Layout
- JSON Example

Your action server receives the following payload from the Rasa server:

Some fields in the payload are optional and may not be present in all requests. Fields marked as `object` have a very complex structure and are not shown here.

```
# pseudo coderequest {    next_action: { type: string }    sender_id: { type: string }    tracker: { type: object }    domain: { type: optional[object] }    domain_digest: { type: optonal[string] }    version: { type: string }}domain {    config: { type: object }    session_config: { type: object }    intents: { type: object }    entities: { type: object }    slots: { type: object }    responses: { type: object }    actions: { type: object }    forms: { type: object }    e2e_actions: { type: object }}tracker {    sender_id: { type: string }    slots: { type: object }    latest_message: { type: object }    events: { type: array_of_objects }    paused: { type: bool }    followup_action: { type: optional[string] }    active_loop: { type: map }    latest_action_name: { type: string }    stack: { type: array_of_objects }}
```

```
    {      "next_action": "action_tell_weather",      "sender_id": "2687378567977106",      "tracker": {        "sender_id": "2687378567977106",        "slots": {          "location": null,          "temperature": null        },        "latest_message": {          "text": "/ask_weather",          "intent": {            "name": "ask_weather",            "confidence": 1          },          "intent_ranking": [            {              "name": "ask_weather",              "confidence": 1            }          ],          "entities": []        },        "followup_action": null,        "paused": false,        "events": [          {            "event": "action",            "timestamp": 1599850576.654908,            "name": "action_session_start",            "policy": null,            "confidence": null          },          {            "event": "session_started",            "timestamp": 1599850576.654916          },          {            "event": "action",            "timestamp": 1599850576.654928,            "name": "action_listen",            "policy": null,            "confidence": null          },          {            "event": "user",            "timestamp": 1599850576.655345,            "text": "/ask_weather",            "parse_data": {              "text": "/ask_weather",              "intent": {                "name": "ask_weather",                "confidence": 1              },              "intent_ranking": [                {                  "name": "ask_weather",                  "confidence": 1                }              ],              "entities": []            },            "input_channel": "facebook",            "message_id": "3f2f2317dada4908b7a841fd3eab6bf9",            "metadata": {}          }        ],        "active_form": {},        "latest_action_name": "action_listen"      },      "domain": {        "config": {          "store_entities_as_slots": true        },        "session_config": {          "session_expiration_time": 60,          "carry_over_slots_to_new_session": true        },        "intents": [          {            "greet": {              "use_entities": true            }          },          {            "ask_weather": {              "use_entities": true            }          }        ],        "entities": [],        "slots": {          "location": {            "type": "rasa.core.slots.UnfeaturizedSlot",            "initial_value": null,            "auto_fill": true          },          "temperature": {            "type": "rasa.core.slots.UnfeaturizedSlot",            "initial_value": null,            "auto_fill": true          }        },        "responses": {          "utter_greet": [            {              "text": "Hey! How are you?"            }          ]        },        "actions": ["action_tell_weather", "utter_greet"],        "forms": []      },      "version": "2.0.0"    }
```

## Fields in the payload[​](https://rasa.com/docs/reference/integrations/action-server/actions/#fields-in-the-payload 'Direct link to Fields in the payload')

Following is the description of the fields in the payload.

### `next_action`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#next_action 'Direct link to next_action')

The `next_action` field tells your action server what action to run. Your actions don't have to be implemented as classes, but they do have to be callable by name.

In the example case, your action server should run the action `action_tell_weather`.

### `sender_id`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#sender_id 'Direct link to sender_id')

The `sender_id` tells you the unique ID of the user having the conversation. Its format varies according to the input channel. What it tells you about the user also depends on the input channel and how the user is identified by the channel.

In the example case, the `sender_id` is not used for anything.

### `tracker`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#tracker 'Direct link to tracker')

The `tracker` contains information about the conversation, including a history of events and a record of all slots:

- `sender_id`: The same `sender_id` as is available in the top level of the payload
- `slots`: Each slot in your bot's domain and its value at the current time
- `latest_message`: The attributes of the latest message
- `followup_action`: The action called was a forced follow up action
- `paused`: Whether the conversation is currently paused
- `events`: A list of all previous [events](https://rasa.com/docs/reference/integrations/action-server/events/)
- `active_loop`: The name of the currently active form, if any
- `active_form`: (Deprecated, replaced by `active_loop`) The name of the currently active form, if any
- `latest_action_name`: The name of the last action the bot executed
- `stack`: Current dialogue stack

In the example case, your custom action uses the value of the `location` slot (if it is set) to get the weather forecast.

### `domain`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#domain 'Direct link to domain')

The `domain` is a json representation of your `domain.yaml` file. It is unlikely that a custom action will refer to its contents, as they are static and do not indicate the state of the conversation.

You can control if an action should receive a domain or not. Visit [selective-domain](https://rasa.com/docs/reference/config/domain/#select-which-actions-should-receive-domain)

Caching the domain object for improved performance

From version `3.9.0` onwards, Rasa action server caches the domain object in the action server. This is done to avoid sending the domain object over the network for every request. Check the `domain_digest` field to learn more about the domain file used to generate the domain object.

### `domain_digest`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#domain_digest 'Direct link to domain_digest')

Used to check if the cached domain object in the action server is up-to-date. Contains the name of the trained model file which is loaded by Rasa.

Starting from version `3.9.0`, Rasa SDK caches the domain object in the action server, so that it does not need to be sent with every request. To make sure that the action server is aware of the latest domain changes, you can send the `domain_digest` field in the request. Alongside `domain_digest`, you can also send the domain object in the request. The action server will compare the digest with the one it has cached. In case the digests (the one retained in Action server and the one sent over network) do not match, the Action server will act as follows:

- If `domain` was not sent in the request, it will respond with a `RETRY (449)` response code in case of HTTP protocol or a `NOT_FOUND` error code with `DOMAIN_NOT_FOUND` error detail in case of gRPC protocol. In the next request, Rasa server will send the domain object.
- If `domain` was sent in the request, the action server will update the cached domain object with the new domain object sent in the request.

### `version`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#version 'Direct link to version')

This is the version of the Rasa server. A custom action is also unlikely to refer to this, although you might use it in a verification step if your action server is only compatible with certain Rasa versions.

## Custom Action Output[​](https://rasa.com/docs/reference/integrations/action-server/actions/#custom-action-output 'Direct link to Custom Action Output')

The Rasa server expects a dictionary of `events` and `responses` as a response to a custom action call.

### `events`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#events 'Direct link to events')

[Events](https://rasa.com/docs/reference/integrations/action-server/events/) are how your action server can influence the conversation. In the example case, your custom action should store the maximum temperature in the `temperature` slot, so it needs to return a [`slot` event](https://rasa.com/docs/reference/integrations/action-server/events/#slot). To set the slot and do nothing else, your response payload would look like this:

```
{  "events": [    {      "event": "slot",      "timestamp": null,      "name": "temperature",      "value": "30"    }  ],  "responses": []}
```

Note that events will be applied to the tracker in the order you list them; with `slot` events, the order won't matter, but with other event types it can.

### `responses`[​](https://rasa.com/docs/reference/integrations/action-server/actions/#responses 'Direct link to responses')

A response can be of any of the response types described in the [documentation on rich responses](https://rasa.com/docs/reference/primitives/responses/#rich-responses). See the response sample of the [API spec](https://rasa.com/docs/reference/api/pro/action-server-api/) for the expected formats.

In the example case, you want to send the user a message with the weather forecast. To send a regular text message, the response payload would look like this:

```
{  "events": [    {      "event": "slot",      "timestamp": null,      "name": "temperature",      "value": "30"    }  ],  "responses": [    {      "text": "This is your weather forecast!"    }  ]}
```

When this response is sent back to the Rasa server, Rasa will apply the `slot` event and two responses to the tracker, and return both messages to the user.

## Special Action Types[​](https://rasa.com/docs/reference/integrations/action-server/actions/#special-action-types 'Direct link to Special Action Types')

There are special action types that are automatically triggered under certain circumstances, namely [default actions](https://rasa.com/docs/reference/primitives/default-actions/) and [slot validation actions](https://legacy-docs-oss.rasa.com/docs/rasa/slot-validation-actions/). These special action types have predefined naming conventions that must be followed to maintain the automatic triggering behavior.

You can customize a default action by implementing a custom action with exactly the same name. Please see the [docs on default actions](https://rasa.com/docs/reference/primitives/default-actions/) for the expected behavior of each action.

Slot validation actions are run on every user turn, depending on whether a form is active or not. A slot validation action that should run when a form is not active must be called `action_validate_slot_mappings`. A slot validation action that should run when a form is active must be called `validate_<form name>`. These actions are expected to return `SlotSet` events only and to behave like the Rasa SDK [`ValidationAction` class](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationaction-class-implementation).

- [Protocols supported by the action server](https://rasa.com/docs/reference/integrations/action-server/actions/#protocols-supported-by-the-action-server)
 - [HTTP(S) Protocol](https://rasa.com/docs/reference/integrations/action-server/actions/#https-protocol)
 - [GRPC Protocol](https://rasa.com/docs/reference/integrations/action-server/actions/#grpc-protocol)
- [Custom Action Input](https://rasa.com/docs/reference/integrations/action-server/actions/#custom-action-input)
- [Fields in the payload](https://rasa.com/docs/reference/integrations/action-server/actions/#fields-in-the-payload)
 - [`next_action`](https://rasa.com/docs/reference/integrations/action-server/actions/#next_action)
 - [`sender_id`](https://rasa.com/docs/reference/integrations/action-server/actions/#sender_id)
 - [`tracker`](https://rasa.com/docs/reference/integrations/action-server/actions/#tracker)
 - [`domain`](https://rasa.com/docs/reference/integrations/action-server/actions/#domain)
 - [`domain_digest`](https://rasa.com/docs/reference/integrations/action-server/actions/#domain_digest)
 - [`version`](https://rasa.com/docs/reference/integrations/action-server/actions/#version)
- [Custom Action Output](https://rasa.com/docs/reference/integrations/action-server/actions/#custom-action-output)
 - [`events`](https://rasa.com/docs/reference/integrations/action-server/actions/#events)
 - [`responses`](https://rasa.com/docs/reference/integrations/action-server/actions/#responses)
- [Special Action Types](https://rasa.com/docs/reference/integrations/action-server/actions/#special-action-types)