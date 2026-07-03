# Source: https://rasa.com/docs/reference/channels/audiocodes-stream

On this page

New in 3.12

From Rasa Pro 3.12, you can stream conversation audio directly from AudioCodes to your Rasa Assistant.

This channel is a voice-stream channel connector to AudioCodes where the conversation audio is streamed directly to Rasa. Rasa also offers a voice-ready channel connector with AudioCodes, read about it on [AudioCodes VoiceAI Connect](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/)

## Configure Rasa Assistant[​](https://rasa.com/docs/reference/channels/audiocodes-stream/#configure-rasa-assistant 'Direct link to Configure Rasa Assistant')

Use the built-in channel `audiocodes_stream` to configure your Rasa Assistant. Create or edit the `credentials.yml` file at the root of your assistant directory to add `audiocodes_stream` channel. Here's an example:

credentials.yml

```
# websocket_url: wss://<your-domain>/webhooks/audiocodes_stream/websocketaudiocodes_stream:  server_url: "<your-domain>"  asr:    name: deepgram  tts:    name: cartesia  # Optional configurations  token: "<authentication-token>"
```

This channel expects WebSocket requests on the URL described above as "websocket\_url". The channel configuration accepts the following properties:

- `server_url` (required): The domain at which Rasa server is available. Do not include protocol (ws:// or wss://). For example, if your server is deployed on `https://example.ngrok.app`, `server_url` should be `example.ngrok.app`.
 
- `asr` (required): Configuration for Automatic Speech Recognition. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#automatic-speech-recognition-asr) for a list of ASR engines for which Rasa provides built-in integration with.
 
- `tts` (required): Configuration for Text-To-Speech. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#text-to-speech-tts) for a list of TTS engines for which Rasa provides built-in integration with.
 
- `token` (optional): An authentication token configured in AudioCodes. This field is optional, as AudioCodes permits empty values. If no token is provided, authentication will be skipped.
 

You can run the assistant using the command `rasa run`. You'll need a URL accessible by AudioCodes for your Rasa assistant. For development, you can use tools like [ngrok](https://ngrok.com/) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)

You can also run it with a development inspector using `rasa run --inspect`. To see all available parameters for this command, use `rasa run -h`.

Bot URLs for development

Visit this [section](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/#testing-channels-on-your-local-machine) to learn how to generate the required bot URL when testing the channel on your local machine.

## Configuration on AudioCodes[​](https://rasa.com/docs/reference/channels/audiocodes-stream/#configuration-on-audiocodes 'Direct link to Configuration on AudioCodes')

This channel connector uses AudioCodes Voice Streaming API. Create a bot connection for Streaming Mode bots on AudioCodes, please refer to [AudioCodes Documentation](https://techdocs.audiocodes.com/livehub/#LiveHub/AudioCodesAPI-framework.htm#Create2) for detailed instructions.

1. On the **Bot Connections** page, click on **Add new voice bot connection**.
 
2. Select **AudioCodes Bot API**
 
3. Set an appropriate name for the Bot Connection. For "Bot connection API type", select "Streaming Mode".
 
4. Set the "Bot URL". Depending on the hostname and TLS configuration, the URL would be `wss://<your-domain>/webhooks/audiocodes_stream/websocket`
 
5. Set an authentication token, ensure that Rasa channel configuration has the same authentication token.
 

You can validate your Rasa Channel configuration with the button "Validate bot connection configuration". Ensure that the Rasa Server is running, AudioCodes will open a websocket on the Bot URL and send a "connection.validate" message.

6. On the next page, select **Enable voice streaming**. For 'Language' field, leave the default setting as it has no impact on Rasa Assistant

Click **Create** to create the bot connection. You can now define a Routing to connect the Bot Connection to a phone number.

## Enable `playFinished` events[​](https://rasa.com/docs/reference/channels/audiocodes-stream/#enable-playfinished-events 'Direct link to enable-playfinished-events')

This channel connector requires `playFinished` events for certain features like silence handling and hanging up the call with `action_hangup` to work.

To enable these events, edit the bot connection that was created in the previous section. Go to the Advanced Tab and paste the following in,

```
{  "sendEventsToBot":["playFinished"]}
```

## Call Metadata[​](https://rasa.com/docs/reference/channels/audiocodes-stream/#call-metadata 'Direct link to Call Metadata')

Metadata about the call can be accessed by the slot `session_started_metadata` in the beginning of the call. Following fields are available:

| Field Name | Description | Source |
| --- | --- | --- |
| `call_id` | The unique call identifier from Audiocodes | `vaigConversationId` parameter as sent by Audiocodes |
| `user_phone` | The phone number of the user | `caller` parameter |
| `user_name` | The caller's display name | `callerDisplayName` parameter sent by Audiocodes |
| `user_host` | The caller's host | `callerHost` parameter sent by Audiocodes |
| `bot_phone` | The phone number of the bot | `callee` parameter |
| `bot_host` | The bot's host | `calleeHost` parameter sent by Audiocodes |

A [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization) can be used to store this information to a slot.

- [Configure Rasa Assistant](https://rasa.com/docs/reference/channels/audiocodes-stream/#configure-rasa-assistant)
- [Configuration on AudioCodes](https://rasa.com/docs/reference/channels/audiocodes-stream/#configuration-on-audiocodes)
- [Enable `playFinished` events](https://rasa.com/docs/reference/channels/audiocodes-stream/#enable-playfinished-events)
- [Call Metadata](https://rasa.com/docs/reference/channels/audiocodes-stream/#call-metadata)