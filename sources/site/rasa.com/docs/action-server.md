# Source: https://rasa.com/docs/action-server

On this page

A Rasa action server runs [custom actions](https://rasa.com/docs/reference/primitives/custom-actions/) for a Rasa conversational assistant.

## How it works[​](https://rasa.com/docs/action-server/#how-it-works 'Direct link to How it works')

When your assistant predicts a custom action, the Rasa server sends a `POST` request to the action server with a json payload including the name of the predicted action, the conversation ID, the contents of the tracker and the contents of the domain.

When the action server finishes running a custom action, it returns a json payload of [responses](https://rasa.com/docs/reference/primitives/responses/) and [events](https://rasa.com/docs/reference/integrations/action-server/events/). See the [API spec](https://rasa.com/docs/reference/api/pro/action-server-api/) for details about the request and response payloads.

The Rasa server then returns the responses to the user and adds the events to the conversation tracker.

To optimise the network traffic, the rasa server will not send a domain with each request to the action server. Instead, it will first send the digest of the domain to the action server and if action server does not have the domain with the given digest cached, it will reply to the action server with code 449 (HTTP retry) or in case of gRPC it will send error code NOT\_FOUND and in the details it will outline that domain was not found. This indicates that the rasa server must resend the request with domain included. [See more](https://rasa.com/docs/reference/integrations/action-server/actions/#domain_digest).

## SDKs for Custom Actions[​](https://rasa.com/docs/action-server/#sdks-for-custom-actions 'Direct link to SDKs for Custom Actions')

You can use an action server written in any language to run your custom actions, as long as it implements the [required APIs](https://rasa.com/docs/reference/api/pro/action-server-api/).

### Rasa SDK (Python)[​](https://rasa.com/docs/action-server/#rasa-sdk-python 'Direct link to Rasa SDK (Python)')

Rasa SDK is a Python SDK for running custom actions. Besides implementing the required APIs, it offers methods for interacting with the conversation tracker and composing events and responses. If you don't yet have an action server and don't need it to be in a language other than Python, using the Rasa SDK will be the easiest way to get started.

### Other Action Servers[​](https://rasa.com/docs/action-server/#other-action-servers 'Direct link to Other Action Servers')

If you have legacy code or existing business logic in another language, you may not want to use the Rasa SDK. In this case you can write your own action server in any language you want. The only requirement for the action server is that it provide a `/webhook` endpoint which accepts HTTP `POST` requests from the Rasa server and returns a payload of [events](https://rasa.com/docs/reference/integrations/action-server/events/) and responses. See the [API spec](https://rasa.com/docs/reference/api/pro/action-server-api/) for details about the required `/webhook` endpoint.

### Running Custom Actions Directly by the Assistant[​](https://rasa.com/docs/action-server/#running-custom-actions-directly-by-the-assistant 'Direct link to Running Custom Actions Directly by the Assistant')

For users looking to streamline their architecture by eliminating the need for an Action Server, you can explore how to run custom actions directly on the Rasa Assistant by following the guidelines [here](https://rasa.com/docs/reference/primitives/custom-actions/#running-custom-actions-directly-by-the-assistant).

- [How it works](https://rasa.com/docs/action-server/#how-it-works)
- [SDKs for Custom Actions](https://rasa.com/docs/action-server/#sdks-for-custom-actions)
 - [Rasa SDK (Python)](https://rasa.com/docs/action-server/#rasa-sdk-python)
 - [Other Action Servers](https://rasa.com/docs/action-server/#other-action-servers)
 - [Running Custom Actions Directly by the Assistant](https://rasa.com/docs/action-server/#running-custom-actions-directly-by-the-assistant)