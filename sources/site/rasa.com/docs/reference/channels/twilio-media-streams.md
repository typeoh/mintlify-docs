# Source: https://rasa.com/docs/reference/channels/twilio-media-streams

On this page

Use this channel to connect your Rasa assistant to Twilio Media Streams for voice capabilities. Unlike the standard Twilio Voice connector, this channel handles speech-to-text (ASR) and text-to-speech (TTS) processing directly in Rasa.

## Configuring Twilio Webhook[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#configuring-twilio-webhook 'Direct link to Configuring Twilio Webhook')

Bot URLs for development

Visit this [section](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/#testing-channels-on-your-local-machine) to learn how to generate the required bot URL when testing the channel on your local machine.

Go to the [Phone Numbers](https://www.twilio.com/console/phone-numbers/incoming) section of your Twilio account and select the phone number you want to connect to Rasa. 
Select the option "Webhook, TwiML Bin, Function, Studio Flow, Proxy Service" and set the URL of your Rasa Server as webhook. Depending on the hostname, the webhook URL would be

```
https://yourdomain.com/webhooks/twilio_media_streams/webhook
```

Your webhook endpoint must be served over HTTPS. Twilio Media Streams does not accept insecure HTTP URLs.

### Using Basic Authentication[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#using-basic-authentication 'Direct link to Using Basic Authentication')

Basic Authentication

Rasa supports Basic Authentication for Twilio Media Streams in Rasa Pro 3.11.8+, Rasa Pro 3.12.7+ and latest releases.

If you want to use basic authentication for your Twilio Media Streams channel, you need to set the webhook URL in the following format:

```
https://<username>:<password>@yourdomain.com/webhooks/twilio_media_streams/webhook
```

For more info checkout the [Twilio security documentation](https://www.twilio.com/docs/usage/security#http-authentication).

## Basic Rasa Configuration[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#basic-rasa-configuration 'Direct link to Basic Rasa Configuration')

Create or edit your `credentials.yml` and add the following channel configuration:

credentials.yml

```
twilio_media_streams:  server_url: "<your-domain>"  # ASR Configuration  asr:    name: "azure"  # or "deepgram"    # Add ASR-specific configuration here  # TTS Configuration  tts:    name: "azure"  # or "cartesia"    # Add TTS-specific configuration here  # Basic Authentication  username: "<username>"  password: "<password>"
```

Basic Authentication

Support for `username` and `password` parameters is available in Rasa Pro 3.11.8+, Rasa Pro 3.12.7+ and latest releases.

Twilio Media Streams channel configuration expects the following parameters:

- `server_url` (required): The domain at which Rasa server is available. Do not include protocol (ws:// or wss://). For example, if your server is deployed on `https://example.ngrok.app`, `server_url` should be `example.ngrok.app`.
 
- `asr` (required): Configuration for Automatic Speech Recognition. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#automatic-speech-recognition-asr) for a list of ASR engines for which Rasa provides built-in integration with.
 
- `tts` (required): Configuration for Text-To-Speech. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#text-to-speech-tts) for a list of TTS engines for which Rasa provides built-in integration with.
 
- `username` (optional): Basic authentication username for Twilio Media Streams channel.
 
- `password` (optional): Basic authentication password for Twilio Media Streams channel.
 
- `interruptions` (optional): Configuration for interruption handling. This allows the assistant to detect when users interrupt while the assistant is speaking and respond more naturally. See [Interruption Handling](https://rasa.com/docs/reference/primitives/patterns/#interruption-handling) for more information.
 

Check [using basic authentication](https://rasa.com/docs/reference/channels/twilio-media-streams/#using-basic-authentication) on how to setup Twilio webhook with basic authentication.

You can run the assistant using the command `rasa run`. You'll need a URL accessible by Twilio for your Rasa assistant. For development, you can use tools like [ngrok](https://ngrok.com/) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/).

You can also run it with a development inspector using `rasa run --inspect`. To see all available parameters for this command, use `rasa run -h`.

## Usage[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#usage 'Direct link to Usage')

### Receiving Audio[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#receiving-audio 'Direct link to Receiving Audio')

When a user speaks, Twilio streams the audio to Rasa where:

1. The audio stream is collected and buffered
2. The configured ASR service converts speech to text
3. The text is processed by Rasa's NLU pipeline

### Sending Responses[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#sending-responses 'Direct link to Sending Responses')

For bot responses:

1. Rasa generates text responses
2. The configured TTS service converts text to audio
3. Audio is streamed back to Twilio

## Call Events[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#call-events 'Direct link to Call Events')

Like other voice channels, the following events are supported:

| Event | Intent | Description |
| --- | --- | --- |
| `start` | `session_start` | Triggered when call connects |
| `end` | `session_end` | Triggered when call disconnects |
| `DTMF` | \- | Phone keypad presses (sent as text messages) |

## Call Metadata[​](https://rasa.com/docs/reference/channels/twilio-media-streams/#call-metadata 'Direct link to Call Metadata')

Metadata about the call can be accessed by the slot `session_started_metadata` in the beginning of the call. Following fields are available:

| Field Name | Description | Source |
| --- | --- | --- |
| `call_id` | The unique call identifier from Twilio | `call_id` parameter as sent by Twilio |
| `user_phone` | The phone number of the user | `user_phone` parameter |
| `bot_phone` | The phone number of the bot | `bot_phone` parameter |
| `direction` | The call direction | `direction` parameter |
| `stream_id` | The unique stream identifier | `streamSid` from Twilio Media Streams |

A [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization) can be used to store this information to a slot.

- [Configuring Twilio Webhook](https://rasa.com/docs/reference/channels/twilio-media-streams/#configuring-twilio-webhook)
 - [Using Basic Authentication](https://rasa.com/docs/reference/channels/twilio-media-streams/#using-basic-authentication)
- [Basic Rasa Configuration](https://rasa.com/docs/reference/channels/twilio-media-streams/#basic-rasa-configuration)
- [Usage](https://rasa.com/docs/reference/channels/twilio-media-streams/#usage)
 - [Receiving Audio](https://rasa.com/docs/reference/channels/twilio-media-streams/#receiving-audio)
 - [Sending Responses](https://rasa.com/docs/reference/channels/twilio-media-streams/#sending-responses)
- [Call Events](https://rasa.com/docs/reference/channels/twilio-media-streams/#call-events)
- [Call Metadata](https://rasa.com/docs/reference/channels/twilio-media-streams/#call-metadata)