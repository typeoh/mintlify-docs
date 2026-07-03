# Source: https://rasa.com/docs/reference/channels/jambonz-stream

On this page

New in 3.13

From Rasa Pro 3.13, you can stream conversation audio directly from Jambonz to your Rasa Assistant.

This channel is a voice-stream channel connector to Jambonz where the conversation audio is streamed directly to Rasa. Rasa also offers a voice-ready channel connector with Jambonz, read about it on [Jambonz as Voice Gateway](https://rasa.com/docs/reference/channels/jambonz/).

## Configure Rasa Assistant[​](https://rasa.com/docs/reference/channels/jambonz-stream/#configure-rasa-assistant 'Direct link to Configure Rasa Assistant')

Use the built-in channel `jambonz_stream` to configure your Rasa Assistant. Create or edit the `credentials.yml` file at the root of your assistant directory to add `jambonz_stream` channel. Here's an example:

credentials.yml

```
jambonz_stream:  server_url: "<your-domain>"  asr:    name: deepgram  tts:    name: cartesia  # Optional configurations  username: "<username>"  password: "<password>"
```

The channel configuration accepts the following properties:

- `server_url` (required): The domain at which Rasa server is available. Do not include protocol (ws:// or wss://). For example, if your server is deployed on `https://example.ngrok.app`, `server_url` should be `example.ngrok.app`.
 
- `asr` (required): Configuration for Automatic Speech Recognition. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#automatic-speech-recognition-asr) for a list of ASR engines for which Rasa provides built-in integration with.
 
- `tts` (required): Configuration for Text-To-Speech. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#text-to-speech-tts) for a list of TTS engines for which Rasa provides built-in integration with.
 
- `username` (optional): Basic authentication username for Jambonz Stream channel.
 
- `password` (optional): Basic authentication password for Jambonz Stream channel.
 
- `interruptions` (optional): Configuration for interruption handling. This allows the assistant to detect when users interrupt while the assistant is speaking and respond more naturally. See [Interruption Handling](https://rasa.com/docs/reference/primitives/patterns/#interruption-handling) for more information.
 

You can run the assistant using the command `rasa run`. You'll need a URL accessible by Jambonz for your Rasa assistant. For development, you can use tools like [ngrok](https://ngrok.com/) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/).

You can also run it with a development inspector using `rasa run --inspect`. To see all available parameters for this command, use `rasa run -h`.

Bot URLs for development

Visit this [section](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/#testing-channels-on-your-local-machine) to learn how to generate the required bot URL when testing the channel on your local machine.

## Configuration on Jambonz[​](https://rasa.com/docs/reference/channels/jambonz-stream/#configuration-on-jambonz 'Direct link to Configuration on Jambonz')

This channel connector uses Jambonz Voice Streaming API. You'll need a Jambonz deployment, an account, and a phone number to route calls to your Rasa assistant.

### Prerequisites[​](https://rasa.com/docs/reference/channels/jambonz-stream/#prerequisites 'Direct link to Prerequisites')

Before you begin, ensure:

- You have access to [Jambonz Cloud](https://jambonz.cloud/) or your own on-premise deployment
- You have the required permissions for creating applications and phone number configurations
- You have a carrier configured that provides your phone number (e.g., Twilio)

### Setup Steps[​](https://rasa.com/docs/reference/channels/jambonz-stream/#setup-steps 'Direct link to Setup Steps')

1. **Create an Application**
 
 Log into your Jambonz console and create a new application. Depending on your domain, the webhook URL will be with method POST,
 
 ```
    https://<your-domain>/webhooks/jambonz_stream/webhook
    ```
 
 For development with ngrok, it would look like:
 
 ```
    https://recently-communal-duckling.ngrok-free.app/webhooks/jambonz_stream/webhook
    ```
 
2. **Configure Basic Authentication (Optional)**
 
 If your Rasa channel is secured with basic authentication, select "Use HTTP basic authentication" in the application configuration and provide the username and password that match your Rasa credentials.
 
3. **Set up Phone Number**
 
 Configure a phone number in Jambonz and point it to the application you created in the previous step. This phone number will be used to receive calls that will be routed to your Rasa assistant.
 
4. **Configure Call Status Webhook (Optional)**
 
 For additional call monitoring, you can configure a call status webhook URL:
 
 ```
    https://<your-domain>/webhooks/jambonz_stream/call_status
    ```
 

## Usage[​](https://rasa.com/docs/reference/channels/jambonz-stream/#usage 'Direct link to Usage')

### Receiving messages from a user[​](https://rasa.com/docs/reference/channels/jambonz-stream/#receiving-messages-from-a-user 'Direct link to Receiving messages from a user')

When a user speaks on the phone, Jambonz will stream the audio directly to your Rasa assistant. The configured ASR engine will convert speech to text, and this message will be interpreted by Rasa just like any other channel. You can then drive the conversation with flows.

### Sending messages to a user[​](https://rasa.com/docs/reference/channels/jambonz-stream/#sending-messages-to-a-user 'Direct link to Sending messages to a user')

Your bot will respond with text messages like with any other channel. The configured TTS engine will convert the text to speech and stream it back to the user through Jambonz.

Here is an example:

```
utter_greet:  - text: "Hello! How can I help you today?"
```

note

Only text messages are supported. Images, attachments, and buttons cannot be used with voice stream channels.

## Call Metadata[​](https://rasa.com/docs/reference/channels/jambonz-stream/#call-metadata 'Direct link to Call Metadata')

Metadata about the call can be accessed by the slot `session_started_metadata` at the beginning of the call. The following fields are available:

| Field Name | Description | Source |
| --- | --- | --- |
| `call_id` | The unique call identifier from Jambonz | `callSid` parameter as sent by Jambonz |
| `user_phone` | The phone number of the user | `from` parameter sent by Jambonz |
| `bot_phone` | The phone number of the bot | `to` parameter sent by Jambonz |
| `stream_id` | The unique stream identifier | `callSid` parameter (same as call\_id) |

A [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization) can be used to store this information to a slot.

## Audio Streaming Details[​](https://rasa.com/docs/reference/channels/jambonz-stream/#audio-streaming-details 'Direct link to Audio Streaming Details')

The Jambonz Stream channel uses WebSocket connections with the `audio.jambonz.org` subprotocol to stream audio. The channel automatically handles:

- **Audio Format Conversion**: Converts between Jambonz's L16 PCM format (8kHz) and Rasa's internal μ-law format
- **Bidirectional Audio**: Supports both receiving audio from users and sending synthesized speech back
- **Real-time Processing**: Streams audio in real-time for low-latency conversations
- **Connection Management**: Handles WebSocket connection lifecycle and error recovery

- [Configure Rasa Assistant](https://rasa.com/docs/reference/channels/jambonz-stream/#configure-rasa-assistant)
- [Configuration on Jambonz](https://rasa.com/docs/reference/channels/jambonz-stream/#configuration-on-jambonz)
 - [Prerequisites](https://rasa.com/docs/reference/channels/jambonz-stream/#prerequisites)
 - [Setup Steps](https://rasa.com/docs/reference/channels/jambonz-stream/#setup-steps)
- [Usage](https://rasa.com/docs/reference/channels/jambonz-stream/#usage)
 - [Receiving messages from a user](https://rasa.com/docs/reference/channels/jambonz-stream/#receiving-messages-from-a-user)
 - [Sending messages to a user](https://rasa.com/docs/reference/channels/jambonz-stream/#sending-messages-to-a-user)
- [Call Metadata](https://rasa.com/docs/reference/channels/jambonz-stream/#call-metadata)
- [Audio Streaming Details](https://rasa.com/docs/reference/channels/jambonz-stream/#audio-streaming-details)