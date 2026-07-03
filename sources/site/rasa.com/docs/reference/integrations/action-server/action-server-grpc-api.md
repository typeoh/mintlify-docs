# Source: https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api

On this page

## Input and Output[​](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#input-and-output 'Direct link to Input and Output')

Following is the gRPC API definition for the Rasa Action Server. The API is defined in the [`proto/action_webhook.proto`](https://github.com/RasaHQ/rasa-sdk/blob/66bb1322e81322f0ee04dd2166c9d470ba1325dd/proto/action_webhook.proto) file in the Rasa SDK repository.

action\_webhook.proto

```
syntax = "proto3";package action_server_webhook;import "google/protobuf/struct.proto";service ActionService {    rpc Webhook (WebhookRequest) returns (WebhookResponse);    rpc Actions (ActionsRequest) returns (ActionsResponse);}message ActionsRequest {}message ActionsResponse {    repeated google.protobuf.Struct actions = 1;}message Tracker {    string sender_id = 1;    google.protobuf.Struct slots = 2;    google.protobuf.Struct latest_message = 3;    repeated google.protobuf.Struct events = 4;    bool paused = 5;    optional string followup_action = 6;    map<string, string> active_loop = 7;    optional string latest_action_name = 8;    repeated google.protobuf.Struct stack = 9;}message Intent {    string string_value = 1;    google.protobuf.Struct dict_value = 2;}message Entity {    string string_value = 1;    google.protobuf.Struct dict_value = 2;}message Action {    string string_value = 1;    google.protobuf.Struct dict_value = 2;}message Domain {    google.protobuf.Struct config = 1;    google.protobuf.Struct session_config = 2;    repeated Intent intents = 3;    repeated Entity entities = 4;    google.protobuf.Struct slots = 5;    google.protobuf.Struct responses = 6;    repeated Action actions = 7;    google.protobuf.Struct forms = 8;    repeated google.protobuf.Struct e2e_actions = 9;}message WebhookRequest {    string next_action = 1;    string sender_id = 2;    Tracker tracker = 3;    Domain domain = 4;    string version = 5;    optional string domain_digest = 6;}message WebhookResponse {    repeated google.protobuf.Struct events = 1;    repeated google.protobuf.Struct responses = 2;}
```

## Error Handling[​](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#error-handling 'Direct link to Error Handling')

When Rasa server is communicating with the action server over gRPC protocol, and an error occurs, the action server should return appropriate error message containing proper gRPC status code and details.

Following are the possible error scenarios and the expected error response from the action server.

Rasa Python action server error handling support

Rasa Python action server provided by Rasa in the for of Rasa SDK is already handling these error scenarios and returning appropriate error messages to the Rasa server. If you are using a custom action server, you need to handle these error scenarios accordingly.

### Custom action is not found[​](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#custom-action-is-not-found 'Direct link to Custom action is not found')

In case that the action server does not find the custom action, it should return:

- an error with gRPC status code `NOT_FOUND`
- details in the error message in the format of:
 
 ```
    {    "action_name" : { "type":  "string" },    "message" : { "type": "string" },    "resource_type" : "ACTION"}
    ```
 
 where `action_name` is the name of the action that was not found, `message` is a human-readable error message, and `resource_type` is the type of the resource that was not found.

### Custom action failed during execution[​](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#custom-action-failed-during-execution 'Direct link to Custom action failed during execution')

In case that the custom action fails during execution, it should return:

- an error with gRPC status code `INTERNAL`
- details in the error message in the format of:
 
 ```
    {    "action_name" : { "type":  "string" },    "message" : { "type": "string" }}
    ```
 
 where `action_name` is the name of the action that failed, and `message` is a human-readable error message.

### Domain is not found[​](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#domain-is-not-found 'Direct link to Domain is not found')

In case that the action server does not find the domain, which corresponds to the [`domain_digest`](https://rasa.com/docs/reference/integrations/action-server/actions/#domain_digest) of the assistant, it should return:

- an error with gRPC status code `NOT_FOUND`
- details in the error message in the format of:
 
 ```
    {    "action_name" : { "type":  "string" },    "message" : { "type": "string" },    "resource_type" : "DOMAIN"}
    ```
 
 where `action_name` is the name of the action which was targeted for invocation, `message` is a human-readable error message, and `resource_type` is the type of the resource that was not found.

- [Input and Output](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#input-and-output)
- [Error Handling](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#error-handling)
 - [Custom action is not found](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#custom-action-is-not-found)
 - [Custom action failed during execution](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#custom-action-failed-during-execution)
 - [Domain is not found](https://rasa.com/docs/reference/integrations/action-server/action-server-grpc-api/#domain-is-not-found)