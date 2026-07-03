# Source: https://rasa.com/docs/reference/channels/twilio-voice

On this page

You can use the Twilio Voice connector to answer phone calls made to your Twilio phone number or Twilio SIP domain.

## Running on Twilio[​](https://rasa.com/docs/reference/channels/twilio-voice/#running-on-twilio 'Direct link to Running on Twilio')

### Connect to a Twilio Phone Number[​](https://rasa.com/docs/reference/channels/twilio-voice/#connect-to-a-twilio-phone-number 'Direct link to Connect to a Twilio Phone Number')

To forward calls from Twilio to your Rasa assistant the webhook for your phone number needs to be updated. Go to the [Phone Numbers](https://www.twilio.com/console/phone-numbers/incoming) section of your Twilio account and select the phone number you want to connect to Rasa. 
Find the `Voice & Fax` section on the screen. 
Under the `A CALL COMES IN` section add the webhook URL in the following format:

```
https://<host>:<port>/webhooks/twilio_voice/webhook
```

For your Rasa bot replacing the host and port with the appropriate values for your deployment. 
Click `Save` at the bottom of the page.

#### Using Basic Authentication[​](https://rasa.com/docs/reference/channels/twilio-voice/#using-basic-authentication 'Direct link to Using Basic Authentication')

Basic Authentication

Rasa supports Basic Authentication for Twilio Media Streams in Rasa Pro 3.10.20+, 3.11.8+, Rasa Pro 3.12.7+ and latest releases.

If you want to use basic authentication for your Twilio Voice channel, you need to set the webhook URL in the following format:

```
https://<username>:<password>@<host>:<port>/webhooks/twilio_voice/webhook
```

For more info checkout the [Twilio security documentation](https://www.twilio.com/docs/usage/security#http-authentication).

### Connect to a Twilio SIP Domain[​](https://rasa.com/docs/reference/channels/twilio-voice/#connect-to-a-twilio-sip-domain 'Direct link to Connect to a Twilio SIP Domain')

You can also connect your Twilio Voice channel to a Twilio SIP domain. You can follow the [Twilio Sending SIP](https://www.twilio.com/docs/voice/api/sending-sip) guide to forward SIP requests to your Twilio SIP domain. Once your SIP domain is configured you will have to forward incoming calls to Rasa. In the Twilio console go to the [SIP Domains](https://console.twilio.com/develop/voice/manage/sip-domains?frameUrl=/console/voice/sip/endpoints) section. Select the SIP domain you would like to use. Find the `Call Control Configuration` section and add the webhook URL (e.g. `https://<host>:<port>/webhooks/twilio_voice/webhook`) for your Rasa bot replacing the host and port with the appropriate values for your deployment. Click `Save` at the bottom of the page.

## Configure Channel in Rasa[​](https://rasa.com/docs/reference/channels/twilio-voice/#configure-channel-in-rasa 'Direct link to Configure Channel in Rasa')

In your `credentials.yml` file make sure the `twilio_voice` channel is added. Within `credentials.yml` there are a number of parameters you can set to control the behavior of your assistant. An example with definitions of each parameter is below. Unlike the Twilio text channel there is no need to add your Twilio credentials for the voice channel. Note, changing values for `enhanced` and `assistant_voice` can result in added costs from Twilio. Review the documentation below for details about these settings.

- Rasa Pro <=3.10.x
- Rasa Pro >=3.11.x

credentials.yml

```
twilio_voice:  initial_prompt: "hello"  assistant_voice: "woman"  reprompt_fallback_phrase: "I didn't get that could you repeat?"  speech_timeout: "5"  speech_model: "default"  enhanced: "false"  # Optional: Basic Authentication  username: "<username>"  password: "<password>"
```

credentials.yml

```
twilio_voice:  assistant_voice: "woman"  reprompt_fallback_phrase: "I didn't get that could you repeat?"  speech_timeout: "5"  speech_model: "default"  enhanced: "false"  # Optional: Basic Authentication  username: "<username>"  password: "<password>"
```

Basic Authentication

Support for `username` and `password` parameters is available in Rasa Pro 3.10.20+, Rasa Pro 3.11.8+, Rasa Pro 3.12.7+ and latest releases.

## Parameter Definitions[​](https://rasa.com/docs/reference/channels/twilio-voice/#parameter-definitions 'Direct link to Parameter Definitions')

New in 3.11

We have unified the call handling patterns across Voice Channel Connectors, all voice channels handle call starts, ends and metadata in a similar manner.

Due to this change the parameter Initial Prompt has been removed.

### Initial Prompt[​](https://rasa.com/docs/reference/channels/twilio-voice/#initial-prompt 'Direct link to Initial Prompt')

When Twilio receives a new call and forwards this to Rasa, Twilio does not provide a user message. In this case Rasa will act as if the user sent the message "Hello". This behavior can be configured in your `credentials.yml` file by setting the `initial_prompt` parameter to the desired input. The `initial_prompt` value will be sent to Rasa and the response will be spoken to the user to start the voice conversation. How you greet a user via a voice channel may differ from a text channel. You should review your responses and consider creating [channel-specific variations](https://rasa.com/docs/reference/primitives/responses/#channel-specific-response-variations) where necessary.

### Assistant Voice[​](https://rasa.com/docs/reference/channels/twilio-voice/#assistant-voice 'Direct link to Assistant Voice')

You can add personality to your assistant by specifying the type of voice your assistant should speak with. In the `credentials.yml` file you can add an option for `assistant_voice` to specify the type of voice of your assistant. For a list of supported voices you can check the [Twilio documentation](https://www.twilio.com/docs/voice/twiml/say#voice). By default Twilio's `woman` voice will be used. Note that you will incur additional charges for using any of the `Polly` voices. The Twilio documentation has details about [Polly pricing](https://www.twilio.com/docs/voice/twiml/say/text-speech#pricing).

### Reprompt Fallback Phrase[​](https://rasa.com/docs/reference/channels/twilio-voice/#reprompt-fallback-phrase 'Direct link to Reprompt Fallback Phrase')

Unlike text channels where users can review previous messages in the conversation and take their time to reply, with voice channels users can get confused when there is a pause in the conversation. When a long pause is detected during a conversation Rasa will repeat the last utterance from the bot to re-prompt the user for a response. If the previous bot utterance cannot be identified then the message defined in `reprompt_fallback_phrase` will be sent. By default this is set to "I'm sorry I didn't get that could you rephrase".

### Speech Timeout[​](https://rasa.com/docs/reference/channels/twilio-voice/#speech-timeout 'Direct link to Speech Timeout')

How long Twilio will collect speech from the caller. This parameter must be set to a number or "auto". If a number is provided the assistant will collect speech for the specified amount of time. If `speech_timeout` is set to "auto" then Twilio will collect speech until a pause is detected. The default setting of this parameter is "auto". You can find more details about `speech_timeout` in the [Twilio documentation](https://www.twilio.com/docs/voice/twiml/gather#speechtimeout). Note that `speech_timeout="auto"` is only compatible with `speech_model='numbers_and_commands'`. An error will be raised if an incompatible `speech_model` is used with the `speech_timeout`.

### Speech Model[​](https://rasa.com/docs/reference/channels/twilio-voice/#speech-model 'Direct link to Speech Model')

Adjusting the `speech_model` parameter can help Twilio with the accuracy of its speech-to-text transformation. Valid values are `default`, `numbers_and_commands`, and `phone_call`. The default setting is "default". You can find more details about `speech_model` in the [Twilio documentation](https://www.twilio.com/docs/voice/twiml/gather#speechmodel).

### Enhanced[​](https://rasa.com/docs/reference/channels/twilio-voice/#enhanced 'Direct link to Enhanced')

Setting `enhanced` to `true` will allow you to use Twilio's premium speech model to improve the accuracy of transcription results. Note, setting this parameter to `true` will result in higher Twilio transcription costs. This parameter is only supported if you also have set the `speech_model` parameter to `phone_call`. By default `enhanced` is set to "false". You can find more about the `enhanced` speech model option in the [Twilio documentation](https://www.twilio.com/docs/voice/twiml/gather#enhanced).

## Call Metadata[​](https://rasa.com/docs/reference/channels/twilio-voice/#call-metadata 'Direct link to Call Metadata')

Metadata about the call can be accessed by the slot `session_started_metadata` in the beginning of the call. Following fields are available:

| Field Name | Description | Source |
| --- | --- | --- |
| `call_id` | The unique call identifier from Twilio | `CallSid` parameter as sent by Twilio |
| `user_phone` | The phone number of the user | `Caller` parameter |
| `bot_phone` | The phone number of the bot | `Called` parameter |
| `direction` | The call direction | `Direction` parameter |

A [custom `action_session_start`](https://rasa.com/docs/rasa-pro/nlu-based-assistants/default-actions#customization) can be used to store this information to a slot.

- [Running on Twilio](https://rasa.com/docs/reference/channels/twilio-voice/#running-on-twilio)
 - [Connect to a Twilio Phone Number](https://rasa.com/docs/reference/channels/twilio-voice/#connect-to-a-twilio-phone-number)
 - [Connect to a Twilio SIP Domain](https://rasa.com/docs/reference/channels/twilio-voice/#connect-to-a-twilio-sip-domain)
- [Configure Channel in Rasa](https://rasa.com/docs/reference/channels/twilio-voice/#configure-channel-in-rasa)
- [Parameter Definitions](https://rasa.com/docs/reference/channels/twilio-voice/#parameter-definitions)
 - [Initial Prompt](https://rasa.com/docs/reference/channels/twilio-voice/#initial-prompt)
 - [Assistant Voice](https://rasa.com/docs/reference/channels/twilio-voice/#assistant-voice)
 - [Reprompt Fallback Phrase](https://rasa.com/docs/reference/channels/twilio-voice/#reprompt-fallback-phrase)
 - [Speech Timeout](https://rasa.com/docs/reference/channels/twilio-voice/#speech-timeout)
 - [Speech Model](https://rasa.com/docs/reference/channels/twilio-voice/#speech-model)
 - [Enhanced](https://rasa.com/docs/reference/channels/twilio-voice/#enhanced)
- [Call Metadata](https://rasa.com/docs/reference/channels/twilio-voice/#call-metadata)