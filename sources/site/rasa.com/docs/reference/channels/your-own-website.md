# Source: https://rasa.com/docs/reference/channels/your-own-website

On this page

If you already have an existing website and want to add a Rasa assistant to it, you can use the [Rasa Chat Widget](https://rasa.com/docs/reference/channels/your-own-website/#chat-widget) a widget which you can incorporate into your existing webpage by adding a HTML snippet.

Alternatively, you can also build your own chat widget.

## REST Channels[​](https://rasa.com/docs/reference/channels/your-own-website/#rest-channels 'Direct link to REST Channels')

The `RestInput` and `CallbackInput` channels can be used for custom integrations. They provide a URL where you can post messages and either receive response messages directly, or asynchronously via a webhook.

### RestInput[​](https://rasa.com/docs/reference/channels/your-own-website/#restinput 'Direct link to RestInput')

The REST channel will provide you with a REST endpoint where you can post user messages and receive the assistant's messages in response.

Add the REST channel to your credentials.yml:

```
rest:  # you don't need to provide anything here - this channel doesn't  # require any credentials
```

Restart your Rasa server to make the REST channel available to receive messages. You can then send messages to `http://<host>:<port>/webhooks/rest/webhook`, replacing the host and port with the appropriate values from your running Rasa server.

#### Request and Response Format[​](https://rasa.com/docs/reference/channels/your-own-website/#request-and-response-format 'Direct link to Request and Response Format')

After making the `rest` input channel available, you can `POST` messages to `http://<host>:<port>/webhooks/rest/webhook`, with the following format:

```
{  "sender": "test_user", // sender ID of the user sending the message  "message": "Hi there!"}
```

The response from Rasa will be a JSON body of bot responses, for example:

```
[{ "text": "Hey Rasa!" }, { "image": "http://example.com/image.jpg" }]
```

#### Message Metadata[​](https://rasa.com/docs/reference/channels/your-own-website/#message-metadata 'Direct link to Message Metadata')

The REST channel supports sending additional metadata along with messages. This metadata can be accessed through the `session_started_metadata` slot and can contain any custom fields you want to pass to your assistant.

To include metadata in your request, add a `metadata` field to your JSON payload:

```
{  "sender": "test_user",  "message": "Hi there!",  "metadata": {    "user_id": "12345",    "source": "website",    "page_url": "https://example.com/contact",    "user_agent": "Mozilla/5.0...",    "custom_field": "any_value"  }}
```

This metadata will be available in the `session_started_metadata` slot at the beginning of the conversation and can be accessed by your actions or flows. A [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization) can be used to store this information to slots for later use in the conversation.

#### Cancel Background A2A Tasks[​](https://rasa.com/docs/reference/channels/your-own-website/#cancel-background-a2a-tasks 'Direct link to Cancel Background A2A Tasks')

If your assistant uses external sub agents over A2A and an operation is running in the background, you can cancel it manually for a specific sender by calling:

`POST http://<host>:<port>/cancel_background_tasks/<sender_id>`

Example:

```
curl -X POST http://localhost:5005/cancel_background_tasks/test_user
```

This endpoint is useful for frontends or external systems that need to explicitly stop in-flight A2A polling or streaming for a conversation.

### CallbackInput[​](https://rasa.com/docs/reference/channels/your-own-website/#callbackinput 'Direct link to CallbackInput')

The Callback channel behaves very much like the REST channel, but instead of directly returning the bot messages to the HTTP request that sends the message, it will call a URL you can specify to send bot messages.

To use the callback channel, add the credentials to your `credentials.yml`:

```
callback:  # URL to which Core will send the bot responses  url: "http://localhost:5034/bot"
```

Restart your Rasa server to make the new channel endpoint available to receive messages. You can then send messages to `http://<host>:<port>/webhooks/callback/webhook`, replacing the host and port with the appropriate values from your running Rasa server.

#### Request and Response Format[​](https://rasa.com/docs/reference/channels/your-own-website/#request-and-response-format-1 'Direct link to Request and Response Format')

After making the `callback` input available, you can `POST` messages to `http://<host>:<port>/webhooks/callback/webhook` with the following format:

```
{  "sender": "test_user", // sender ID of the user sending the message  "message": "Hi there!"}
```

If successful, the response will be `success`. Once Rasa is ready to send a message to the user, it will call the `url` specified in your `credentials.yml` with a `POST` request containing a JSON body of the bot's responses:

```
[{ "text": "Hey Rasa!" }, { "image": "http://example.com/image.jpg" }]
```

## SocketIO Channel[​](https://rasa.com/docs/reference/channels/your-own-website/#socketio-channel 'Direct link to SocketIO Channel')

The SocketIO channel uses websockets and is real-time. To use the SocketIO channel, add the credentials to your `credentials.yml`:

```
socketio:  user_message_evt: user_uttered  bot_message_evt: bot_uttered  session_persistence: true/false
```

The first two configuration values define the event names used by Rasa when sending or receiving messages over socket.io.

The socket client can pass an object named metadata to supply metadata to the channel. You can configure an alternative key using the metadata\_key setting. For example, if your client wants to pass metadata on a key named customData, the setting would be:

```
socketio:  metadata_key: customData
```

Restart your Rasa server to make the new channel endpoint available to receive messages. You can then send messages to `http://<host>:<port>/socket.io`, replacing the host and port with the appropriate values from your running Rasa server.

session persistence

By default, the SocketIO channel uses the socket id as `sender_id`, which causes the session to restart at every page reload. `session_persistence` can be set to `true` to avoid that. In that case, the frontend is responsible for generating a session id and sending it to the Rasa server by emitting the event `session_request` with `{session_id: [session_id]}` immediately after the `connect` event.

The example [Webchat](https://github.com/botfront/rasa-webchat) implements this session creation mechanism (version >= 0.5.0).

SocketIO client / server compatibility

The version of the SocketIO client connecting to Rasa must be compatible with the versions of the [python-socketio](https://github.com/miguelgrinberg/python-socketio) and [python-engineio](https://github.com/miguelgrinberg/python-engineio) packages used by Rasa. Please refer to the [`pyproject.toml`](https://github.com/RasaHQ/rasa/blob/main/pyproject.toml) file relative to your version of Rasa and the official `python-socketio` compatibility table.

### JWT Authentication[​](https://rasa.com/docs/reference/channels/your-own-website/#jwt-authentication 'Direct link to JWT Authentication')

The SocketIO channel can be optionally configured to perform JWT authentication on connect by defining the `jwt_key` and optional `jwt_method` in the `credentials.yml` file.

```
socketio:  user_message_evt: user_uttered  bot_message_evt: bot_uttered  session_persistence: true  jwt_key: my_public_key  jwt_method: HS256
```

When initially requesting the connection, the client should pass in an encoded payload as a JSON object under the key `token`:

```
{  "token": "jwt_encoded_payload"}
```

### Chat Widget[​](https://rasa.com/docs/reference/channels/your-own-website/#chat-widget 'Direct link to Chat Widget')

Once you've set up your SocketIO channel, you can use the [official Rasa Chat Widget](https://github.com/RasaHQ/chat-widget) on any webpage. Use the example HTML below and paste the URL of your Rasa instance into the `server-url` attribute:

```
<!doctype html><html lang="en">  <head>    <meta charset="UTF-8" />    <meta name="viewport" content="width=device-width, initial-scale=1.0" />    <title>HTML Example</title>    <script      type="module"      src="https://unpkg.com/@rasahq/chat-widget-ui/dist/rasa-chatwidget/rasa-chatwidget.esm.js"    ></script>    <link      rel="stylesheet"      href="https://unpkg.com/@rasahq/chat-widget-ui/dist/rasa-chatwidget/rasa-chatwidget.css"    />  </head>  <body>    <rasa-chatbot-widget server-url="https://example.com"></rasa-chatbot-widget>  </body></html>
```

For more information, including how to fully customize the widget for your website, you can check out the [full documentation](https://chatwidget.rasa.com/).

Alternatively, if you want to embed the widget in a React app, there is [a library in the NPM package repository](https://www.npmjs.com/package/@rasahq/chat-widget-react).

Deprecated Chat Widget

For information on the deprecated chat widget, [rasa-chat](https://www.npmjs.com/package/@rasahq/rasa-chat), documentation can be found [here](https://chat-widget-docs.rasa.com).

- [REST Channels](https://rasa.com/docs/reference/channels/your-own-website/#rest-channels)
 - [RestInput](https://rasa.com/docs/reference/channels/your-own-website/#restinput)
 - [CallbackInput](https://rasa.com/docs/reference/channels/your-own-website/#callbackinput)
- [SocketIO Channel](https://rasa.com/docs/reference/channels/your-own-website/#socketio-channel)
 - [JWT Authentication](https://rasa.com/docs/reference/channels/your-own-website/#jwt-authentication)
 - [Chat Widget](https://rasa.com/docs/reference/channels/your-own-website/#chat-widget)