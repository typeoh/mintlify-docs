# Source: https://rasa.com/docs/reference/channels/custom-connectors

On this page

Channels in Rasa are the abstraction that allows you to connect the Rasa Assistant to your desired platform where your users are. If the built-in channels in Rasa do not fit your needs, you can create a custom channel.

A custom channel connector must be implemented as a Python class. When building a custom channel, think of it like a two-way conversation between your desired platform and Rasa. You need:

- **InputChannel**: Receives messages from users on your platform and forwards them to Rasa for processing.
 
- **OutputChannel**: Takes Rasa's responses and sends them back to users on your platform.
 

The flow is simple: User sends message → InputChannel receives it → Rasa processes → OutputChannel sends response back to user.

This separation lets you customize how messages come in (webhook, WebSocket, etc.) independently from how responses go out (REST API calls, real-time streaming, etc.).

## InputChannel[​](https://rasa.com/docs/reference/channels/custom-connectors/#inputchannel 'Direct link to InputChannel')

A custom connector class must subclass `rasa.core.channels.channel.InputChannel` and implement at least `blueprint` and `name` methods.

### The `name` method[​](https://rasa.com/docs/reference/channels/custom-connectors/#the-name-method 'Direct link to the-name-method')

The `name` method defines the url prefix for the connector's webhook. It also defines the channel name you should use in any [channel specific response variations](https://rasa.com/docs/reference/primitives/responses/#channel-specific-response-variations) and the name you should pass to the `output_channel` query parameter on the [trigger intent endpoint](https://rasa.com/docs/reference/api/pro/http-api/).

For example, if your custom channel is named `myio`, you would define the `name` method as:

```
from rasa.core.channels.channel import InputChannelclass MyIO(InputChannel):    def name() -> Text:        """Name of your custom channel."""        return "myio"
```

You would write a response variation specific to the `myio` channel as:

domain.yml

```
responses:  utter_greet:    - text: Hi! I'm the default greeting.    - text: Hi! I'm the custom channel greeting      channel: myio
```

The webhook you give to the custom channel to call would be `http://<host>:<port>/webhooks/myio/webhook`, replacing the host and port with the appropriate values from your running Rasa server.

### The `blueprint` method[​](https://rasa.com/docs/reference/channels/custom-connectors/#the-blueprint-method 'Direct link to the-blueprint-method')

The `blueprint` method must create a [Sanic blueprint](https://sanicframework.org/en/guide/best-practices/blueprints.html#overview) that can be attached to a sanic server. Your blueprint should have at least the two routes: `health` on the route `/`, and `receive` on the route `/webhook` (see example custom channel below).

As part of your implementation of the `receive` endpoint, you will need to tell Rasa to handle the user message. You do this by calling

```
    on_new_message(      rasa.core.channels.channel.UserMessage(        text,        output_channel,        sender_id      )    )
```

Calling `on_new_message` will send the user message to the [`handle_message`](https://github.com/RasaHQ/rasa/blob/c922253fe890bb4903329d4ade764e0711d384ec/rasa/core/agent.py#L511_) method.

See more details on the `UserMessage` object [here](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/core/channels/channel#usermessage-objects).

### Optional InputChannel Methods[​](https://rasa.com/docs/reference/channels/custom-connectors/#optional-inputchannel-methods 'Direct link to Optional InputChannel Methods')

You can override these methods for additional functionality:

- **`from_credentials(credentials)`** - Class method to create channel instance from credentials configuration.

## OutputChannel[​](https://rasa.com/docs/reference/channels/custom-connectors/#outputchannel 'Direct link to OutputChannel')

The [`OutputChannel`](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/core/channels/channel#outputchannel-objects) class is responsible for sending Rasa's responses back to users on your platform. There are two main options:

1. **Use [`CollectingOutputChannel`](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/core/channels/channel#collectingoutputchannel-objects)** - Collects all bot responses in a list that you can return in your webhook response (good for REST-style channels).
2. **Create your own OutputChannel subclass** - Implement custom logic for sending responses directly to your platform (good for real-time channels like WebSocket, Slack, etc.).

### Using CollectingOutputChannel[​](https://rasa.com/docs/reference/channels/custom-connectors/#using-collectingoutputchannel 'Direct link to Using CollectingOutputChannel')

CollectingOutputChannel only collects sent messages in a list (doesn't send them anywhere). The collected messages can be accessed via the `messages` property. Here is an example of a custom Rasa channel that uses it:

custom\_channel.py

```
import asyncioimport inspectfrom sanic import Blueprint, responsefrom sanic.request import Requestfrom sanic.response import HTTPResponsefrom typing import Text, Dict, Any, Optional, Callable, Awaitableimport rasa.utils.endpointsfrom rasa.core.channels.channel import (    InputChannel,    CollectingOutputChannel,    UserMessage,)class MyIO(InputChannel):    def name() -> Text:        """Name of your custom channel."""        return "myio"    def blueprint(        self, on_new_message: Callable[[UserMessage], Awaitable[None]]    ) -> Blueprint:        custom_webhook = Blueprint(            "custom_webhook_{}".format(type(self).__name__),            inspect.getmodule(self).__name__,        )        @custom_webhook.route("/", methods=["GET"])        async def health(request: Request) -> HTTPResponse:            return response.json({"status": "ok"})        @custom_webhook.route("/webhook", methods=["POST"])        async def receive(request: Request) -> HTTPResponse:            sender_id = request.json.get("sender") # method to get sender_id            text = request.json.get("text") # method to fetch text            input_channel = self.name() # method to fetch input channel            metadata = self.get_metadata(request) # method to get metadata            collector = CollectingOutputChannel()            # include exception handling            await on_new_message(                UserMessage(                    text,                    collector,                    sender_id,                    input_channel=input_channel,                    metadata=metadata,                )            )            return response.json(collector.messages)        return custom_webhook
```

Example Implementation

For a complete example of a custom channel using CollectingOutputChannel, see the [custom\_channel.py](https://github.com/RasaHQ/rasa-calm-demo/blob/minimal-llm-assistant/addons/custom_channel.py) implementation in the rasa-calm-demo repository.

### Creating a Custom OutputChannel[​](https://rasa.com/docs/reference/channels/custom-connectors/#creating-a-custom-outputchannel 'Direct link to Creating a Custom OutputChannel')

To create your own OutputChannel, subclass `rasa.core.channels.channel.OutputChannel` and implement at minimum the `send_text_message` method:

custom\_output\_channel.py

```
from rasa.core.channels.channel import OutputChannelfrom typing import Text, Anyclass MyCustomOutputChannel(OutputChannel):    def __init__(self, webhook_url: str):        super().__init__()        self.webhook_url = webhook_url        async def send_text_message(self, recipient_id: Text, text: Text, **kwargs: Any) -> None:        """Required method: Send a simple text message."""        # Your implementation to send text to your platform        # e.g., make HTTP request, send via WebSocket, etc.        pass
```

### Required Methods[​](https://rasa.com/docs/reference/channels/custom-connectors/#required-methods 'Direct link to Required Methods')

**`send_text_message(recipient_id, text, **kwargs)`** - The only method you must implement. This handles basic text responses from Rasa.

### Optional Methods for Enhanced Capabilities[​](https://rasa.com/docs/reference/channels/custom-connectors/#optional-methods-for-enhanced-capabilities 'Direct link to Optional Methods for Enhanced Capabilities')

Override these async methods to support rich message types; these methods are asynchronous and return `None`:

- **`send_image_url(recipient_id, image, **kwargs)`** - Send an image by URL. Default will just post the url as a string.
- **`send_attachment(recipient_id, attachment, **kwargs)`** - Send a file attachment. Default will just post as a string.
- **`send_text_with_buttons(recipient_id, text, buttons, **kwargs)`** - Send text with interactive buttons. Default implementation will just post the buttons as a string.
- **`send_quick_replies(recipient_id, text, quick_replies, **kwargs)`** - Send text with quick reply options. Default implementation will just send as buttons.
- **`send_elements(recipient_id, elements, **kwargs)`** - Send carousel/card elements. Default implementation will just post the elements as a string.
- **`send_custom_json(recipient_id, json_message, **kwargs)`** - Send custom JSON payloads. Default implementation will just post the json contents as a string.

## Streaming Generative Responses[​](https://rasa.com/docs/reference/channels/custom-connectors/#streaming-generative-responses 'Direct link to Streaming Generative Responses')

If your Assistant uses generative responses from Rephraser or Enterprise Search Policy, the channel can stream these responses as soon as the tokens are generated, instead of waiting for the entire response to be prepared.

The OutputChannel class can be modified to enable streaming of responses. To implement response streaming for your channel, you can override these three optional methods (mentioned in the code snippet below) to stream the generated chunks of the response. By default, they do nothing (`pass` statement).

The OutputChannel functions described above (`send_text_message`, etc) will still be called once the complete response is ready. It is the channel's responsibility to ensure that a response is NOT sent to the platform twice.

```
class MyCustomOutputChannel(OutputChannel):    # ... other methods ...        async def send_response_chunk_start(self, recipient_id, **kwargs):        """Invoked once at the beginning of response."""        # TODO: Initialize streaming connection        pass              async def send_response_chunk(self, recipient_id, chunk, **kwargs):        """Invoked multiple times for each generated chunk/token."""        # TODO: Send chunk to platform        pass         async def send_response_chunk_end(self, recipient_id, **kwargs):        """Invoked once at the end of response."""        # TODO: Finalize streaming connection        pass
```

Example Implementation

For a complete example of a custom channel with WebSocket support and response streaming, see the [websocket\_channel.py](https://github.com/RasaHQ/rasa-calm-demo/blob/minimal-llm-assistant/addons/websocket_channel.py) implementation in the rasa-calm-demo repository.

`recipient_id` is the Rasa Sender ID and the argument `chunk` contains the text of the response chunk that is being generated. These functions are called at different points during the generation of response. See this sequence diagram for more context,

![Response Streaming Architecture](https://rasa.com/docs/assets/images/streaming-responses-73ee79cd94fa8c113061b324f84bf7da.svg)

### Considerations when Streaming Responses[​](https://rasa.com/docs/reference/channels/custom-connectors/#considerations-when-streaming-responses 'Direct link to Considerations when Streaming Responses')

#### Avoid Blocking Calls[​](https://rasa.com/docs/reference/channels/custom-connectors/#avoid-blocking-calls 'Direct link to Avoid Blocking Calls')

Avoid making any blocking calls in these functions as they would impact the assistant's latency. If you need to do anything more than simply sending the message on a WebSocket, consider using async tasks.

For example, voice channels create a task with `asyncio.create_task` for reading the audio from TTS WebSocket and sending it to the client.

#### Handle Duplicates[​](https://rasa.com/docs/reference/channels/custom-connectors/#handle-duplicates 'Direct link to Handle Duplicates')

OutputChannel must ensure that a generative response is not sent twice, once during generation from `send_response_chunk()` and later when the complete response is ready from `send_text_message()`.

OutputChannel objects are isolated per conversation. Therefore, you can use a flag to mark when the responses have been streamed. The streaming methods will always be called in order for a single response:

- `send_response_chunk_start()` - called once.
- `send_response_chunk()` - called multiple times.
- `send_response_chunk_end()` - called once.

After streaming completes, Rasa will still call `send_text_message()` with the full response text. At this point, the channel can check the flag to skip sending the response twice.

## Common Use Cases[​](https://rasa.com/docs/reference/channels/custom-connectors/#common-use-cases 'Direct link to Common Use Cases')

### Accessing Conversation State[​](https://rasa.com/docs/reference/channels/custom-connectors/#accessing-conversation-state 'Direct link to Accessing Conversation State')

The `tracker_state` property contains comprehensive conversation data including slots, active flows, intents, custom actions called, and other state information.

This information can be used to enrich the responses of your channel.

### Passing Metadata to Rasa[​](https://rasa.com/docs/reference/channels/custom-connectors/#passing-metadata-to-rasa 'Direct link to Passing Metadata to Rasa')

If you need to use extra information from your front end in your custom actions, you can pass this information using the `metadata` key of your user message. This information will accompany the user message through the Rasa server into the action server when applicable, where you can find it stored in the `tracker`. Message metadata will not directly affect NLU classification or action prediction.

tip

If you want to access the metadata which was sent with the user message that triggered the session start, you can access the special slot `session_started_metadata`. Read more about it in [action\_session\_start](https://rasa.com/docs/reference/primitives/default-actions/#customization).

## Credentials for Custom Channels[​](https://rasa.com/docs/reference/channels/custom-connectors/#credentials-for-custom-channels 'Direct link to Credentials for Custom Channels')

To use a custom channel, you need to supply credentials for it in a credentials configuration file called `credentials.yml`. This credentials file has to contain the **module path** (not the channel name) of your custom channel and any required configuration parameters.

For example, for a custom connector class called `MyIO` saved in a file `addons/custom_channel.py`, the module path would be `addons.custom_channel.MyIO`, and the credentials could look like:

credentials.yml

```
addons.custom_channel.MyIO:  username: "user_name"  another_parameter: "some value"
```

Example Configuration

For an example of configuring custom channels in credentials.yml, see the [credentials.yml](https://github.com/RasaHQ/rasa-calm-demo/blob/ac9a7ca9c574a3d5e6ba983b8f22654da59cc93d/credentials.yml#L58-L64) file in the rasa-calm-demo repository.

To make the Rasa server aware of your custom channel, specify the path to `credentials.yml` to the Rasa server at startup with the command line argument `--credentials` .

## Testing the Custom Connector Webhook[​](https://rasa.com/docs/reference/channels/custom-connectors/#testing-the-custom-connector-webhook 'Direct link to Testing the Custom Connector Webhook')

To test your custom connector, you can `POST` messages to the webhook using a JSON body with the following format:

```
{  "sender": "test_user", // sender ID of the user sending the message  "message": "Hi there!",  "metadata": {} // optional, any extra info you want to add for processing in NLU or custom actions}
```

For a locally running Rasa server, the curl request would look like this:

```
curl --request POST \     --url http://localhost:5005/webhooks/myio/webhook \     --header 'Content-Type: application/json' \     --data '{            "sender": "test_user",            "message": "Hi there!",            "metadata": {}          }'
```

- [InputChannel](https://rasa.com/docs/reference/channels/custom-connectors/#inputchannel)
 - [The `name` method](https://rasa.com/docs/reference/channels/custom-connectors/#the-name-method)
 - [The `blueprint` method](https://rasa.com/docs/reference/channels/custom-connectors/#the-blueprint-method)
 - [Optional InputChannel Methods](https://rasa.com/docs/reference/channels/custom-connectors/#optional-inputchannel-methods)
- [OutputChannel](https://rasa.com/docs/reference/channels/custom-connectors/#outputchannel)
 - [Using CollectingOutputChannel](https://rasa.com/docs/reference/channels/custom-connectors/#using-collectingoutputchannel)
 - [Creating a Custom OutputChannel](https://rasa.com/docs/reference/channels/custom-connectors/#creating-a-custom-outputchannel)
 - [Required Methods](https://rasa.com/docs/reference/channels/custom-connectors/#required-methods)
 - [Optional Methods for Enhanced Capabilities](https://rasa.com/docs/reference/channels/custom-connectors/#optional-methods-for-enhanced-capabilities)
- [Streaming Generative Responses](https://rasa.com/docs/reference/channels/custom-connectors/#streaming-generative-responses)
 - [Considerations when Streaming Responses](https://rasa.com/docs/reference/channels/custom-connectors/#considerations-when-streaming-responses)
- [Common Use Cases](https://rasa.com/docs/reference/channels/custom-connectors/#common-use-cases)
 - [Accessing Conversation State](https://rasa.com/docs/reference/channels/custom-connectors/#accessing-conversation-state)
 - [Passing Metadata to Rasa](https://rasa.com/docs/reference/channels/custom-connectors/#passing-metadata-to-rasa)
- [Credentials for Custom Channels](https://rasa.com/docs/reference/channels/custom-connectors/#credentials-for-custom-channels)
- [Testing the Custom Connector Webhook](https://rasa.com/docs/reference/channels/custom-connectors/#testing-the-custom-connector-webhook)