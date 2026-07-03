# Source: https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect

On this page

Use this channel to connect your Rasa assistant to [Audiocodes VoiceAI connect](https://www.audiocodes.com/solutions-products/voiceai/voiceai-connect).

## Getting Credentials[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#getting-credentials 'Direct link to Getting Credentials')

To get credentials, create a bot on the [VoiceAI connect portal](https://voiceaiconnect.audiocodes.io/).

1. Select **Bots** in the left sidebar.
2. Click on the **+** sign to create a new bot.
3. Select **Rasa Pro** as the Bot Framework
4. Set the bot URL and choose a token value.
5. "Validate the bot configuration" once you have completed the "Setting credentials" section and the bot is running. Validation will only succeed if the bot is running and the token is correctly set.

Setting the bot URL with a tunneling solution when testing locally

Visit this [section](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/#testing-channels-on-your-local-machine) to learn how to generate the required bot URL when testing the channel on your local machine.

## Setting credentials[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#setting-credentials 'Direct link to Setting credentials')

The token value chosen above will be used in the `credentials.yml`:

- Rasa Pro <=3.7.x
- Rasa Pro >=3.8.x

```
rasa_plus.channels.audiocodes.AudiocodesInput:  token: "token"
```

```
audiocodes:  token: "token"
```

You can also specify optional parameters:

| Parameter | Default value | Description |
| --- | --- | --- |
| `token` | No default value | The token to authenticate calls between your Rasa assistant and VoiceAI connect |
| `use_websocket` | `true` | If true, Rasa and AudioCodes will signal to each other over a websocket. If false, Rasa and AudioCodes will signal over HTTP API. Using websocket for signalling is better when bot needs to handle tasks which are more time consuming, thus not blocking the line so that other signals can come through. [See Audiocodes documentation for more details](https://techdocs.audiocodes.com/voice-ai-connect/Default.htm#Bot-API/ac-bot-api-mode-http.htm#Sending20Activities20over20WebSocket) |
| `keep_alive` | 120 | In seconds. For each ongoing conversation, VoiceAI Connect will periodically verify the conversation is still active on the Rasa side. |

Then restart your Rasa server to make the new channel endpoint available.

## Usage[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#usage 'Direct link to Usage')

New in 3.11

We have unified the call handling patterns across Voice Channel Connectors, all voice channels handle call starts, ends and metadata in a similar manner.

### Call Metadata[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#call-metadata 'Direct link to Call Metadata')

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

### Receiving messages from a user[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#receiving-messages-from-a-user 'Direct link to Receiving messages from a user')

When a user speaks on the phone, VoiceAI Connect will send a text message (after it is processed by the speech-to-text engine) to your assistant like any other channel. This message will be interpreted by Rasa and handled by the dialogue engine, and the response will be sent back to VoiceAI Connect to be converted to a voice message and delivered to the user.

### Sending messages to a user[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#sending-messages-to-a-user 'Direct link to Sending messages to a user')

Your bot will respond with text messages like with any other channel. The text-to-speech engine will convert the text and deliver it as a voice message to the user.

Here is an example:

```
utter_greet:  - text: "Hello! isn’t every life and every work beautiful?"
```

note

Only text messages are allowed. Images, attachments, and buttons cannot be used with a voice channel.

### DTMF and Other Events[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#dtmf-and-other-events 'Direct link to DTMF and Other Events')

Rasa receives the intent `/vaig_event_DTMF` when receiving a DTMF tone (i.e user presses digit on the keyboard of the phone). The digit(s) sent will be passed in the `value` entity.

The general pattern is that for every Audiocodes `event`, the bot will receive the `vaig_event_<event>` intent, with context information in entities.

Check the [VoiceAI Connect documentation](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/voiceai_connect.htm?TocPath=VoiceAI%2520Connect%257C_____0) for an exhaustive list of events.

note

Certain events like `playFinished` could disrupt the conversation with the assistant. We recommend that they should be disabled.

### Configuring calls[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#configuring-calls 'Direct link to Configuring calls')

You can send events from Rasa to VoiceAI Connect to make changes to the current call configuration. For example, you might want to receive a notification when the user stays silent for more than 5 seconds, or you might need to customize how DTMF digits are sent by VoiceAI Connect.

Call configuration events are sent with custom messages and are specific to the current conversation (sometimes to a message). Which means they must be part of your stories or rules so the same behaviour is applied to all conversations.

Those Rasa responses don't utter anything, they just configure the voice gateway. It is a good practice naming them differently, for example prefixing them with `utter_config_<what_it_does>`

All the supported events are exhaustively documented in the [VoiceAI Connect documentation](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/voiceai_connect.htm?TocPath=VoiceAI%2520Connect%257C_____0). We will look at one example here to illustrate the use of custom messages and events.

#### Example: changing a pin code[​](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#example-changing-a-pin-code 'Direct link to Example: changing a pin code')

In this example we create a flow to allow a user to change a pin code.

```
- rule: Set pin code  steps:    # User says "I want to change my pin code"    - intent: set_pin_code    # Send the noUserInput configuration event    - action: utter_config_no_user_input    # Send the DTMF format configuration event    - action: utter_config_dtmf_pin_code    # A standard Rasa form to collect the pin code from the user    - action: pin_code_form    - ...
```

In the domain, we can add the `utter_config_<config_event>` responses:

[`noUserInput` event](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/inactivity-detection.htm?TocPath=Bot%2520integration%257CReceiving%2520notifications%257C_____3)

```
utter_config_no_user_input:  - custom:      type: event      name: config      sessionParams:        # If user stays silent for 5 seconds or more, the notification will be sent        userNoInputTimeoutMS: 5000        # If you want to allow for more than one notification during a call        userNoInputRetries: 2        # Enable the noUserInput notification        userNoInputSendEvent: true
```

[`DTMF` event](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/receive-dtmf.htm?TocPath=Bot%2520integration%257CReceiving%2520notifications%257C_____2)

```
   utter_config_dtmf_pin_code:   - custom:      type: event      name: config      sessionParams:        # Enable grouped collection (i.e will send all digits in a single payload)        dtmfCollect: true        # If more than 5 secs have passed since a digit was pressed,        # the input is considered completed and will be sent to the bot        dtmfCollectInterDigitTimeoutMS: 5000        # If 6 digits are collected, VoiceAI will send those 6 digits        # even if the user keeps pressing buttons        dtmfCollectMaxDigits: 6        # If the user presses '#' the input is considered complete        dtmfCollectSubmitDigit: "#"
```

Now you can configure the `pin_code` slot in the `pin_code_form` to extract the pin code from the `value` entity with the `vaig_event_DTMF` intent:

```
pin_code:  type: text  influence_conversation: false  mappings:    - type: from_entity      entity: value      intent: vaig_event_DTMF      not_intent: vaig_event_noUserInput      conditions:        - active_loop: pin_code_form          requested_slot: pin_code
```

Notice how `vaig_event_noUserInput` was declared in the `not_intent` field.

Since the `vaig_event_noUserInput` intent is sent by VoiceAI Connect when the user stays silent as per our configuration, we must deactivate the form so we can pick up the conversation from a rule or a story and gracefully handle the failure.

In the following example, we simply cancel the current flow if we receive the `vaig_event_noUserInput` intent (i.e. user stays silent) while the `pin_code_form` loop is active.

```
- rule: Set pin code - happy path  steps:    - intent: set_pin_code    - action: utter_config_no_user_input    - action: utter_config_dtmf_pin_code    - action: pin_code_form    - active_loop: pin_code_form    - active_loop: null    - slot_was_set:        - requested_slot: null    - action: utter_pin_code_changed    - action: action_pin_code_cleanup- rule: Set pin code - no response - cancel.  condition:    - active_loop: pin_code_form  steps:    - intent: vaig_event_noUserInput    - action: utter_cancel_set_pin_code    - action: action_deactivate_loop    - active_loop: null
```

- [Getting Credentials](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#getting-credentials)
- [Setting credentials](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#setting-credentials)
- [Usage](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#usage)
 - [Call Metadata](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#call-metadata)
 - [Receiving messages from a user](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#receiving-messages-from-a-user)
 - [Sending messages to a user](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#sending-messages-to-a-user)
 - [DTMF and Other Events](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#dtmf-and-other-events)
 - [Configuring calls](https://rasa.com/docs/reference/channels/audiocodes-voiceai-connect/#configuring-calls)