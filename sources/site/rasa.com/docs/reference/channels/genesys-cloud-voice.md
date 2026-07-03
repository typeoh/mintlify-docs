# Source: https://rasa.com/docs/reference/channels/genesys-cloud-voice

On this page

New in 3.12

The Genesys Cloud connector is available from Rasa Pro 3.12

There are two ways to connect a Voice Assistant to Genesys Cloud:

- Using the built-in Genesys Channel to stream conversation audio from Genesys to your Rasa Assistant. This is a Voice-Stream channel connector. Steps for setting this up are described in section [Stream Conversation Audio From Genesys](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#stream-conversation-audio-from-genesys)
 
- A Voice Ready channel connector can also be made using [Data Actions](https://help.mypurecloud.com/articles/about-genesys-cloud-data-actions-integration/) where transcription (speech-recognition) and speech-to-text is handled within Genesys Architect. Go to [Integration With Data Action](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#integration-with-data-action) section for more details
 

## Stream Conversation Audio From Genesys[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#stream-conversation-audio-from-genesys 'Direct link to Stream Conversation Audio From Genesys')

Genesys Channel Connector allows you to stream conversation audio to your Voice Assistant from Genesys using AudioConnector Integration.

Before you begin, ensure:

- You have [AudioConnector Integration](https://help.mypurecloud.com/articles/audio-connector-overview/) available on your Genesys Account
- You have the required permissions for creating [Architect](https://help.mypurecloud.com/articles/architect-overview/) flows for Inbound or Outbound Calls
- You have permissions to create/modify Call Routing

### Prepare Rasa Assistant[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#prepare-rasa-assistant 'Direct link to Prepare Rasa Assistant')

Create a file `credentials.yml` at the root of Rasa assistant project and add Genesys channel configuration:

credentials.yml

```
# websocket_url: wss://<your-domain>/webhooks/genesys/websocketgenesys:  api_key: "<api_key>"  client_secret: "<client_secret>"  server_url: "<your-domain>"  asr:    name: deepgram  tts:    name: cartesia
```

`wss` or `ws`?

Depending on your domain's TLS configuration, the WebSocket URL could be `ws://example.com` or `wss://example.com`.

Genesys channel configuration accepts the following properties:

- `api_key` (required): API Key configured on Genesys with AudioConnector integration. It is the value of `X-Api-Key` header.
 
- `server_url` (required): The domain at which Rasa server is available. Do not include protocol (ws:// or wss://). For example, if your server is deployed on `https://example.ngrok.app`, `server_url` should be `example.ngrok.app`.
 
- `asr` (required): Configuration for Automatic Speech Recognition. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#automatic-speech-recognition-asr) for a list of ASR engines for which Rasa provides built-in integration with.
 
- `tts` (required): Configuration for Text-To-Speech. See [Speech Integrations](https://rasa.com/docs/reference/integrations/speech-integrations/#text-to-speech-tts) for a list of TTS engines for which Rasa provides built-in integration with.
 
- `client_secret` (optional): Client Secret configured on Genesys with AudioConnector Integration. It is used to sign connection requests. It must be base-64 encoded. Signature verification is skipped if `client_secret` is not provided.
 

You can run the assistant using the command `rasa run`. You can also run it with a development inspector using `rasa run --inspect`. To see all available parameters, use `rasa run -h`.

### Install AudioConnector Integration[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#install-audioconnector-integration 'Direct link to Install AudioConnector Integration')

To install & configure AudioConnector Integration,

1. Go to **Integrations** Page, it should be listed on Genesys Cloud Admin Home.
 
2. Click "+ Integrations" button on the top right corner to install a new Integration.
 
3. Search for "AudioConnector" and install it. If you cannot find it, make sure that your Genesys Account has access to it. You might want to speak to your Genesys Account Manager or Customer Support if you have further issues.
 
4. Now that you have installed AudioConnector Integration, you can add an appropriate name and description. Go to the Configuration page to set a Base Connection URI to your Rasa Assistant, this would be the domain where your assistant is deployed.
 
 The webhook URL will be in the format, (attention, with a trailing slash!)
 
 - With TLS: `wss://example.com/webhooks/genesys/`
 - Without TLS: `ws://example.com/webhooks/genesys/`
 

WebSocket URL

If the Rasa WebSocket URL is `wss://<your-domain>/webhooks/genesys/websocket`. In this documentation we split it into:

- Base URI: `wss://<your-domain>/webhooks/genesys/`
- Connector ID: `websocket`

However any other combination should also work just OK.

5. Advanced Tab can be left blank. In Credentials tab, create a credential with an API Key and an optional client secret.
 
6. Hit Save button and change the status of Integration to **Active**. You will not see the Integration on Architect if it is not active.
 

### Create a Call Flow in Architect[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#create-a-call-flow-in-architect 'Direct link to Create a Call Flow in Architect')

Once you have installed the AudioConnector Integration, you can use it in a Call Flow with Genesys Architect.

Create a new call flow in Architect. In this call flow, create a reusable task. This task should use "Call Audio Connector" action from Toolbox (Listed under "🤖 Bot"). Set the Connector ID to `websocket`

Ensure that the Reusable Task is connected to an option in the Main Menu. Once you are done, "Publish" the flow.

### Set Call Routing[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#set-call-routing 'Direct link to Set Call Routing')

If you have created an Inbound Call Flow in the previous step, then you will need to setup a Call Route to connect a Phone Number to this Call Flow.

1. Navigate to Admin Home > Routing > Call Routing.
 
2. Create a new Call Route or modify an existing one. Select Inbound numbers that you would like to activate for the flow.
 

Don't see any phone numbers?

In case you do not see any phone numbers, please reach out to a Genesys Expert within your organization or to Genesys Support. It could be possible that the DID numbers are not made available for Call Routing

3. In the section "What call flow should be used?" select the Architect flow that you created in the previous step.

Once you save the call route, you should be able to call the phone number to connect to the Architect flow you created. Using the options in the Main Menu, you can connect to your Rasa voice assistant.

### Rate Limits[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#rate-limits 'Direct link to Rate Limits')

Audio is sent to Genesys over WebSocket as binary messages. Genesys enforces rate limits on binary messages which could impact your assistant.

These rate limits are,

- The allowed average rate per second of inbound binary data (`global.inbound.binary.average.rate.per.second`): 5
 
- The maximum number of inbound binary data messages that can be sent instantaneously (`global.inbound.binary.max`): 25
 

You can read more about these rate limits on [Genesys Documentation](https://developer.genesys.cloud/organization/organization/limits#audiohook)

To avoid rate limit Errors, Rasa buffers the audio messages and sends them only if,

- Audio Buffer is more than 32KB
- Audio Synthesis is complete

While Designing Flows

Please keep in mind these rate limits when designing flows for a Rasa Voice Assistant. Avoid using two or more utterances consecutively in flows as these will be sent as multiple binary messages.

## Call Metadata[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#call-metadata 'Direct link to Call Metadata')

Metadata about the call can be accessed by the slot `session_started_metadata` in the beginning of the call. Following fields are available:

| Field Name | Description | Source |
| --- | --- | --- |
| `call_id` | The unique call identifier from Genesys | `conversationId` parameter as sent by Genesys |
| `user_phone` | The phone number of the user | `ani` parameter, with `tel:` prefix removed |
| `bot_phone` | The phone number of the bot | `dnis` parameter as sent by Genesys |

A [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization) can be used to store this information to a slot.

## Integration With Data Action[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#integration-with-data-action 'Direct link to Integration With Data Action')

A voice-ready channel connector with Genesys can also be created using Rasa's REST channel connector and using Genesys Data Actions. This requires more familiarity with Genesys Architect and Data Actions.

You can create a Call flow in Architect with a reusable-task that would:

- Capture caller's speech using Transcribe action.
- Send the transcribed text to Rasa using Call Data Action.
- Get the bot's response and play it back to the caller using Play Audio action.

These steps can be done in a loop based on the response of Rasa. Genesys also allows using 3rd Party services for Transcription (Speech-To-Text) and Text-To-Speech. You can read more about [Speech-To-Text Engines in Genesys here](https://help.mypurecloud.com/articles/about-speech-to-text-stt-engines/) and [Text-To-Speech Engines in Genesys here](https://help.mypurecloud.com/articles/about-text-to-speech-tts-engines/)

### Prepare Rasa Assistant[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#prepare-rasa-assistant-1 'Direct link to Prepare Rasa Assistant')

Add the REST channel to your credentials.yml:

credentials.yml

```
# webhook URL: https://<your-domain>/webhooks/rest/webhookrest:  # you don't need to provide anything here - this channel doesn't  # require any credentials
```

You can run the assistant using the command `rasa run`. You can also run it with a development inspector using `rasa run --inspect`. To see all available parameters, use `rasa run -h`.

You can test your Rasa server with a curl request,

```
curl -X POST \  http://localhost:5005/webhooks/rest/webhook \  -H 'Content-Type: application/json' \  -d '{    "sender": "unique-sender-id",    "message": "Hi bot, what can you do?"  }'
```

### Create a Data Action[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#create-a-data-action 'Direct link to Create a Data Action')

On Genesys, create a [Data Action](https://help.mypurecloud.com/articles/about-the-data-actions-integrations/). You will need to specifically create a [Web Services Data Action Integration](https://help.mypurecloud.com/articles/about-web-services-data-actions-integration/).

Ensure that the payload sent by the Data Action to Rasa is in the following format:

```
{    "sender": "<genesys-call-id>",    "message": "<transcribed-user-message>"}
```

The `sender` key must have a unique key to identify the conversation while `message` contains the result of Speech-To-Text component.

### Disconnecting a Call[​](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#disconnecting-a-call 'Direct link to Disconnecting a Call')

You cannot use a Data Action to perform actions on conversations, like disconnecting the call. [Genesys documentation specifies](https://help.mypurecloud.com/faqs/can-i-use-genesys-cloud-data-actions-to-perform-actions-on-conversations/) that "Only direct participants in a conversation, such as an agent or a customer, can modify the conversation using call controls."

You can, however, modify your Architect Call Flow to disconnect a call based on the response from Data Action.

- [Stream Conversation Audio From Genesys](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#stream-conversation-audio-from-genesys)
 - [Prepare Rasa Assistant](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#prepare-rasa-assistant)
 - [Install AudioConnector Integration](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#install-audioconnector-integration)
 - [Create a Call Flow in Architect](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#create-a-call-flow-in-architect)
 - [Set Call Routing](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#set-call-routing)
 - [Rate Limits](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#rate-limits)
- [Call Metadata](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#call-metadata)
- [Integration With Data Action](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#integration-with-data-action)
 - [Prepare Rasa Assistant](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#prepare-rasa-assistant-1)
 - [Create a Data Action](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#create-a-data-action)
 - [Disconnecting a Call](https://rasa.com/docs/reference/channels/genesys-cloud-voice/#disconnecting-a-call)