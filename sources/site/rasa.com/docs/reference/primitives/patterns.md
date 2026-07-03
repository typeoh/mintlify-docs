# Source: https://rasa.com/docs/reference/primitives/patterns

On this page

## Configurations[​](https://rasa.com/docs/reference/primitives/patterns/#configurations 'Direct link to Configurations')

### Default Behavior[​](https://rasa.com/docs/reference/primitives/patterns/#default-behavior 'Direct link to Default Behavior')

Rasa ships a **default behavior for every [conversation repair](https://rasa.com/docs/learn/concepts/conversation-patterns/) case** that works out-of-the-box. Each case is handled through a pattern which is a special flow designed specifically to handle the case:

- `pattern_continue_interrupted` for interruptions.
- `pattern_correction` for corrections.
- `pattern_cancel_flow` for cancellations.
- `pattern_skip_question` for skipping collect steps.
- `pattern_chitchat` for chitchat.
- `pattern_completed` for completion.
- `pattern_clarification` for clarification.
- `pattern_internal_error` for internal errors.
- `pattern_cannot_handle` for cannot handle.
- `pattern_human_handoff` for human handoff.
- `pattern_session_start` for procatively starting a session by the assistant.
- `pattern_validate_slot` for applying real-time validations on slot values which are strictly independent of any business logic.
- `pattern_repeat_bot_messages` for when the user asks the assistant to repeat an utterance.
- `pattern_customer_satisfaction` for collecting customer satisfaction feedback at the end of a conversation.

Voice-specific patterns:

- `pattern_user_silence` for handling user silences in voice assistants.

The syntax for each of these flows is the same as any other flow.

info

Conversation repair cases are expected to work out-of-the-box. This means that if the [default behaviour](https://rasa.com/docs/reference/primitives/patterns/#reference-default-pattern-configuration) is good enough for the assistant's use case, then the flow corresponding to the pattern handling the repair case is not needed in the assistant's project directory.

info

The [Contextual Response Rephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/) helps the default responses from patterns to fit in naturally with the context of the conversations.

### Modifying default behaviour[​](https://rasa.com/docs/reference/primitives/patterns/#modifying-default-behaviour 'Direct link to Modifying default behaviour')

It is possible to override the default behaviour of each conversation repair case by creating a flow with the same name as that of the pattern used to handle the corresponding case, like `pattern_correction`. If the pattern uses a default action which needs to be modified, you can override the implementation of the default action by implementing a new custom action and use that custom action in the flow.

It is possible to add a [link step](https://rasa.com/docs/reference/primitives/flow-steps/#link) from a pattern to a [flow](https://rasa.com/docs/reference/primitives/flows/), except for `pattern_internal_error`, where link steps are not allowed.

Additionally, you can link a pattern to the `pattern_human_handoff` or the `pattern_chitchat`.

info

Make sure the assistant is re-trained after the modification is completed.

info

Since most of these patterns interrupt another flow, they should be kept short and simple.

### Sample Configuration[​](https://rasa.com/docs/reference/primitives/patterns/#sample-configuration 'Direct link to Sample Configuration')

Modify Rasa's response when a flow concludes:

flows.yml

```
flows:  pattern_completed:    description: Completion of a user's flow    steps:      - action: utter_can_do_something_else
```

domain.yml

```
responses:  utter_can_do_something_else:    - text: "Is there anything else I can assist you with?"
```

## Common Modifications[​](https://rasa.com/docs/reference/primitives/patterns/#common-modifications 'Direct link to Common Modifications')

Here are some common modifications to the default behavior.

### Requiring Confirmation[​](https://rasa.com/docs/reference/primitives/patterns/#requiring-confirmation 'Direct link to Requiring Confirmation')

You can change the default implementation for a correction and require a confirmation from the user before a slot is updated, e.g. this would result in a conversation like this:

User: I want to send some money to Joe

Bot: How much money do you want to send?

User: 50$

Bot: Do you want to send 50$ to Joe? (Yes/No)

User: Oh wait!! I meant to say to John, not Joe!

Bot:

Do you want to update the recipient to John? (Yes/No)

User: Yes!

Bot: Updated recipient to John

Bot: Do you want to send 50$ to John? (Yes/No)

User: ...

A common correction scenario

To achieve the above confirmation, create a flow named `pattern_correction` which is defined as follows:

flows.yml

```
flows:  pattern_correction:    description: Confirm a previous correction of a slot value.    steps:      - noop: true        next:          - if: context.is_reset_only            then:              - action: action_correct_flow_slot                next: END          - else: confirm_first      - id: confirm_first        collect: confirm_slot_correction        next:          - if: not slots.confirm_slot_correction            then:              - action: utter_not_corrected_previous_input                next: END          - else:              - action: action_correct_flow_slot              - action: utter_corrected_previous_input                next: END
```

Also make sure to add the used responses and slots to your domain file:

domain.yml

```
slots:  confirm_slot_correction:    type: boolresponses:  utter_ask_confirm_slot_correction:    - text: "Do you want to update the {{ context.corrected_slots.keys()|join(', ') }}?"      buttons:        - payload: "yes"          title: "Yes"        - payload: "no"          title: "No, please keep the previous information"      metadata:        rephrase: True        template: jinja  utter_not_corrected_previous_input:    - text: "Ok, I did not correct the previous input."      metadata:        rephrase: True
```

### Implementing a Human Handoff[​](https://rasa.com/docs/reference/primitives/patterns/#implementing-a-human-handoff 'Direct link to Implementing a Human Handoff')

Currently, the default behaviour for a human handoff is to inform the user that the assistant cannot help with the request. However, in scenarios where customer service is available, implementing a human handoff becomes relevant. You can implement a human handoff by writing a custom action and overriding the flow named `pattern_human_handoff`:

flows.yml

```
flows:  pattern_human_handoff:    description: Human handoff implementation    steps:      - collect: confirm_human_handoff        next:          - if: slots.confirm_human_handoff            then:            - action: action_human_handoff              next: END          - else:            - action: utter_human_handoff_cancelled              next: END
```

Also make sure to add the used actions, responses and slots to your domain file:

domain.yml

```
slots:  confirm_human_handoff:    type: bool    mappings:      - type: controlledactions:  - action: action_human_handoffresponses:  utter_ask_confirm_human_handoff:    - text: "Do you want to be connected to a human agent?"      buttons:        - payload: "yes"          title: "Yes"        - payload: "no"          title: "No"  utter_human_handoff_cancelled:    - text: "Ok, I understand you don't want to be connected to a human agent. Is there something else I can help you with?"      metadata:        rephrase: True
```

### React dependent on the current flow[​](https://rasa.com/docs/reference/primitives/patterns/#react-dependent-on-the-current-flow 'Direct link to React dependent on the current flow')

You can change a pattern's behaviour depending on the flow that was interrupted. This can be done by using the `context` object in the `if` condition of a pattern:

flows.yml

```
flows:  pattern_cancel_flow:    description: A meta flow that's started when a flow is cancelled.    steps:      - id: decide_cancel_step        noop:          - if: context.canceled_name = "transfer money"            then: inform_user          - else: cancel_flow # skips the inform step      - id: inform_user        action: utter_flow_cancelled_rasa        next: cancel_flow      - id: cancel_flow        action: action_cancel_flow
```

In the above example, the `inform_user` step is only used if the flow that was interrupted is called `transfer_money`.

### Free form generation for chitchat[​](https://rasa.com/docs/reference/primitives/patterns/#free-form-generation-for-chitchat 'Direct link to Free form generation for chitchat')

By default, chitchat is turned off, and the assistant responds with the default `utter_cannot_handle` response.

To switch to free-form generated responses, override the default behavior of `pattern_chitchat` by creating a flow named `pattern_chitchat` which is defined as follows:

flows.yml

```
flows:  pattern_chitchat:    description: handle interactions with the user that are not task-oriented    name: pattern chitchat    steps:      - action: utter_free_chitchat_response
```

warning

Free-form responses will be generated using an LLM. There's a possibility that the assistant could answer queries outside of the intended domain.

#### Fallback to free-form responses for not relevant answers in Enterprise Search[​](https://rasa.com/docs/reference/primitives/patterns/#fallback-to-free-form-responses-for-not-relevant-answers-in-enterprise-search 'Direct link to Fallback to free-form responses for not relevant answers in Enterprise Search')

In case the [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) is used and the [relevancy check](https://rasa.com/docs/reference/config/policies/generative-search/#relevancy-check) is enabled, the assistant will fall back to `pattern_cannot_handle` in case no relevant answer could be found in the knowledge base and a predefined response is triggered.

To update this behaviour, you can [override](https://rasa.com/docs/pro/customize/patterns/#1-override-a-pattern-flow) the `pattern_cannot_handle` flow to link to the `pattern_chitchat` flow in order to allow the assistant to answer with a free-form response.

flows.yml

```
flows:  pattern_cannot_handle:    description: Conversation repair flow for addressing failed command generation scenarios    name: pattern_cannot_handle    steps:      - noop: true        next:          # Fallback for ChitChat command when IntentlessPolicy isn't set, but          # pattern_chitchat invokes action_trigger_chitchat          - if: context.reason is "cannot_handle_chitchat"            then:              - action: utter_cannot_handle                next: "END"          # Fallback for things that are not supported          - if: context.reason is "cannot_handle_not_supported"            then:              - action: utter_cannot_handle                next: END          # Fallback when no relevant answer to the user query has been found.          # This is used by the EnterpriseSearchPolicy.          - if: context.reason is "cannot_handle_no_relevant_answer"            then:              - link: pattern_chitchat          # Default          - else:              - action: utter_ask_rephrase                next: END
```

### Skipping clarification[​](https://rasa.com/docs/reference/primitives/patterns/#skipping-clarification 'Direct link to Skipping clarification')

By default, clarification will check on the users intention by asking the user to choose from a list of flows. In some cases, it may be desirable to skip clarification and move directly to starting a flow by adding a link step directly to that flow.

flows.yml

```
flows:  pattern_clarification:    description: Conversation repair flow for handling ambiguous requests that could match multiple flows    name: pattern clarification    steps:      - noop: true        next:          - if: context.names            then:            - action: action_clarify_flows              next:                - if: context.names contains "<name of a flow>"                  then:                    - link: <name_of_a_flow>                - else: clarify_options          - else:            - action: utter_clarification_no_options_rasa              next: END      - id: clarify_options        action: utter_clarification_options_rasa
```

### Preventing multiple Clarifications[​](https://rasa.com/docs/reference/primitives/patterns/#preventing-multiple-clarifications 'Direct link to Preventing multiple Clarifications')

By default, clarification will continue to check on the users intentions regardless of the number of times it has asked. This can be avoided by counting the number of clarification requests and linking to the human handoff pattern if this count reaches some threshold.

flows.yml

```
flows:  pattern_clarification:    description: Conversation repair flow for handling ambiguous requests that could match multiple flows    name: pattern clarification    steps:      - noop: true        next:          - if: context.names            then:            - action: action_clarify_flows            - action: action_increase_clarification_count              next:                - if: slots.clarification_count > CLARIFICATION_LIMIT                  then:                    - link: pattern_human_handoff                - else: clarify_options          - else:            - action: utter_clarification_no_options_rasa              next: END      - id: clarify_options        action: utter_clarification_options_rasa
```

This would require the [custom action](https://rasa.com/docs/reference/primitives/custom-actions/) `action_increase_clarification_count` to be [implemented](https://github.com/RasaHQ/rasa-calm-demo/blob/main/actions/action_increase_clarification_count.py) and added to the `domain.yml` along with the `clarification_count` slot.

### Triggering clarification when multiple `StartFlow` commands are predicted[​](https://rasa.com/docs/reference/primitives/patterns/#triggering-clarification-when-multiple-startflow-commands-are-predicted 'Direct link to triggering-clarification-when-multiple-startflow-commands-are-predicted')

New in 3.15

You can now configure the assistant to trigger a clarification request when multiple `StartFlow` commands are predicted by the Command Generator while no flow is currently active.

By default, when the [Command Generator](https://rasa.com/docs/reference/config/components/llm-command-generators/) predicts multiple `StartFlow` commands while no flow is currently active, both flows are started sequentially. To trigger a clarification request instead, set the `CLARIFY_ON_MULTIPLE_START_FLOWS` environment variable to `True`.

**Note:** This setting only applies when there is no active flow in the conversation. If a flow is already active, the default behavior continues regardless of this setting.

### Configuring the maximum number of clarification options[​](https://rasa.com/docs/reference/primitives/patterns/#configuring-the-maximum-number-of-clarification-options 'Direct link to Configuring the maximum number of clarification options')

New in 3.15

A configurable limit has been introduced for the number of clarification options presented to the user.

When a clarification is triggered, the maximum number of flows shown as options is 3 by default. To change this limit, set the `initial_value` of the `max_clarification_options` slot to your desired value in your domain.

domain.yml

```
slots:  max_clarification_options:    type: float    initial_value: 2.0 # Limit clarification options to 2 flows
```

### Using the rephraser for repeats[​](https://rasa.com/docs/reference/primitives/patterns/#using-the-rephraser-for-repeats 'Direct link to Using the rephraser for repeats')

By default, the repeat pattern triggers an action which collects all bot utterances since the last user utterance and sends those again. This works well for assistants where LLM generated output should not be sent directly. If your setup allows for LLM generated output, you can improve the experience with the repeat pattern a lot by allowing more specific repeats from a wider context of messages.

#### Overwriting the repeat pattern[​](https://rasa.com/docs/reference/primitives/patterns/#overwriting-the-repeat-pattern 'Direct link to Overwriting the repeat pattern')

First, update your flows to override the repeat pattern definition, and introduce a custom response (in this example it's called `utter_repeat_pattern_response`):

flows.yml

```
flows:  pattern_repeat_bot_messages:    description: Voice conversation repair pattern to repeat bot messages    name: pattern repeat bot messages    steps:      - action: utter_repeat_pattern_response
```

#### Customise the response[​](https://rasa.com/docs/reference/primitives/patterns/#customise-the-response 'Direct link to Customise the response')

In your domain file, define the new response (`utter_repeat_pattern_response` in our example), with the rephraser turned on, together with a custom `rephrase_prompt`. The `rephrase_prompt` property is required here to implement the repeating logic.

domain.yml

```
version: "3.1"responses:  utter_repeat_pattern_response:    # the text property is a fallback when the rephraser fails due to connection issues    - text: "Sorry, I'm not able to answer that right now."      metadata:        rephrase: True        rephrase_prompt: |            The following is a conversation with an AI assistant. The assistant is            helpful, creative, clever, and very friendly. Repeat the requested            information staying close to the original conversation content and retaining            its meaning. Use simple {{language}}.            Context / previous conversation with the user:            {{history}}            Last user message:            {{current_input}}            AI assistant response:
```

#### Configuring the Rephraser[​](https://rasa.com/docs/reference/primitives/patterns/#configuring-the-rephraser 'Direct link to Configuring the Rephraser')

The following `endpoints.yml` config section makes sure that the rephraser is turned on, and that it gets to see the raw conversation history for the last 10 steps. Keep in mind that increasing the size of the conversation history turns here will increase it for all rephrasings done by the assistant. Have a look at the [rephraser's documentation](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/) to learn more about this component.

endpoints.yml

```
nlg:  - type: rephrase    summarize_history: False    max_historical_turns: 10
```

## Common Voice-specific Pattern Modifications[​](https://rasa.com/docs/reference/primitives/patterns/#common-voice-specific-pattern-modifications 'Direct link to Common Voice-specific Pattern Modifications')

### Handling Call Start and Call Metadata[​](https://rasa.com/docs/reference/primitives/patterns/#handling-call-start-and-call-metadata 'Direct link to Handling Call Start and Call Metadata')

New in 3.11

We have unified the call handling patterns across Voice Channel Connectors, all voice channels handle call starts, ends and metadata in a similar manner.

The assistant will receive the message `/session_start` when the call is picked up along with the call metadata. This intent triggers the Session Start pattern. Here's a customized pattern that sends `utter_greet` when the call connects:

flows.yml

```
flows:  pattern_session_start:    description: Flow for starting the conversation    name: pattern session start    nlu_trigger:      - intent: session_start    steps:      - action: utter_greet
```

The following call metadata is received for Twilio:

- `call_id` is the unique call identifier from Twilio (`CallSid` parameter as sent by Twilio Voice)
- `user_phone` is the phone number of the user (`Caller`)
- `bot_phone` is the phone number of the bot (`Called`)
- `direction` is the call direction. It can be either `inbound` or `outbound`

Action `action_session_start` is triggered at the beginning of each Rasa session and it can be used to set certain slots based on this metadata. These slots can be used in your utterances for a dynamic greeting. Here is an example:

```
from rasa_sdk import Action, Trackerfrom rasa_sdk.events import SlotSetfrom rasa_sdk.executor import CollectingDispatcherimport logginglogger = logging.getLogger(__name__)class ActionSessionStart(Action):    def name(self) -> str:        return "action_session_start"    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker,            domain: dict) -> list:        # get the call metadata from the tracker        metadata = tracker.get_slot("session_started_metadata")        logger.info(f"🤙 action_session_start's metadata: {metadata}")        # set appropriate slots        if metadata:            return [                SlotSet("user_phone", metadata.get("user_phone")),                SlotSet("bot_phone", metadata.get("bot_phone")),            ]        return []
```

### Handling the End of a Call[​](https://rasa.com/docs/reference/primitives/patterns/#handling-the-end-of-a-call 'Direct link to Handling the End of a Call')

Call can be ended by the user or by the assistant.

- When the call is ended by the user `/session_end` message is received by the assistant along with a [`SessionEnded` event](https://rasa.com/docs/reference/primitives/events/#session-ended-event) to the conversation.
 
- Assistant flows can use the default action `action_hangup` to disconnect calls. This action also will add a `SessionEnded` event to the conversation.
 

### Using Responses relevant to Voice Channels[​](https://rasa.com/docs/reference/primitives/patterns/#using-responses-relevant-to-voice-channels 'Direct link to Using Responses relevant to Voice Channels')

It is recommended to use channel specific responses with voice channels. This can be done with [channel specific responses](https://rasa.com/docs/reference/primitives/responses/#channel-specific-response-variations):

domain.yml

```
responses:  utter_setup_guide:  - text: "Click the 👉 button or visit https://example.com/setup-guide"  - text: ""To continue setup, open our website and go to the setup guide""    channel: "twilio_media_streams"
```

In the above example, note that emoji doesn't translate well to speech. URLs are difficult to conprehend due to the temporal nature of voice vs permanence of text. Words like "Type" or "Click" assume interactions that aren't possible on a phone call. Special characters are awkward when spoken.

You can also use SSML in the responses to allow for more customisation in the audio responses from the assistant.

domain.yml

```
responses:  utter_contact_support:  - text: "Call our support team at 1-800-555-0123"  - text: |      <speak>        You can reach our support team at        <say-as interpret-as="telephone">1 800 555 0123</say-as>        <break time="500ms"/>        Our agents are available 24/7.      </speak>    channel: "twilio_media_streams"
```

## Interruption Handling[​](https://rasa.com/docs/reference/primitives/patterns/#interruption-handling 'Direct link to Interruption Handling')

Beta Feature

Interruption handling is currently in beta. This feature is available for selected Voice Stream Channels only.

Interruption handling allows your voice assistant to detect and respond when users interrupt while the assistant is speaking - a natural behavior in human conversation. When enabled, the assistant can stop speaking as soon as it detects the user has starting to talk, creating a more natural and responsive conversational experience.

### How It Works[​](https://rasa.com/docs/reference/primitives/patterns/#how-it-works 'Direct link to How It Works')

Interruptions are identified from partial transcripts sent by the Automatic Speech Recognition (ASR) engine. The assistant monitors these partial transcripts in real-time and can stop its current response when it detects the user is speaking.

### Supported Channels[​](https://rasa.com/docs/reference/primitives/patterns/#supported-channels 'Direct link to Supported Channels')

Interruption handling is available for the following Voice Stream Channels:

- Browser Audio ([used by Rasa Voice Inspector](https://rasa.com/docs/pro/testing/trying-assistant/#inspecting-voice-assistants))
- Twilio Media Streams
- Jambonz Stream

### Configuration[​](https://rasa.com/docs/reference/primitives/patterns/#configuration 'Direct link to Configuration')

By default, interruption handling is **disabled**. You can enable and configure it using the `interruptions` key in your channel configuration in `credentials.yml`:

credentials.yml

```
browser_audio:  server_url: localhost  interruptions:    enabled: true    min_words: 3  asr:    name: "deepgram"  tts:    name: cartesia
```

#### Configuration Parameters[​](https://rasa.com/docs/reference/primitives/patterns/#configuration-parameters 'Direct link to Configuration Parameters')

- **`enabled`** (boolean, default: `false`): Enables or disables interruption handling for the channel.
- **`min_words`** (integer, default: `3`): The minimum number of words required in a partial transcript to trigger an interruption. This helps filter out backchannels (brief responses like "hmm", "yeah", "ok") that shouldn't interrupt the assistant.

### Disabling Interruptions for Specific Responses[​](https://rasa.com/docs/reference/primitives/patterns/#disabling-interruptions-for-specific-responses 'Direct link to Disabling Interruptions for Specific Responses')

You may want to prevent interruptions for certain critical information that users must hear completely. You can disable interruptions for specific responses using the `allow_interruptions` property in your domain:

domain.yml

```
responses:  utter_current_balance:    - text: You still have {current_balance} in your account.      allow_interruptions: false  utter_important_terms:    - text: Please note the following terms and conditions...      allow_interruptions: false
```

By default, `allow_interruptions` is `true` for all responses.

### Best Practices[​](https://rasa.com/docs/reference/primitives/patterns/#best-practices 'Direct link to Best Practices')

- **Start with the default `min_words` value**: The default value of 3 words helps prevent false interruptions from backchannels while still being responsive to genuine user input.
- **Test thoroughly**: Different ASR engines may produce partial transcripts with varying speeds and accuracy. Test your configuration with your specific ASR provider.
- **Use `allow_interruptions: false` sparingly**: Only disable interruptions for truly critical information. Overusing this can make the conversation feel unnatural and frustrating for users.
- **Consider user experience**: Some users may speak more slowly or with pauses. If you increase `min_words` too high, the assistant may feel less responsive.

## Reference: Default Pattern Configuration[​](https://rasa.com/docs/reference/primitives/patterns/#reference-default-pattern-configuration 'Direct link to Reference: Default Pattern Configuration')

For reference, here is the complete default configuration for conversation repair:

Default Patterns

```
version: "3.1"responses:  utter_ask_continue_conversation:    - text: "Is there anything else I can help you with?"      metadata:        rephrase: True  utter_ask_continue_interrupted_flow_confirmation:    - text: "Would you like to continue with {{context.interrupted_flow_options}}?"      metadata:        rephrase: True        template: jinja  utter_ask_csat_score:    - text: "How would you rate your experience today?"      metadata:        rephrase: True      buttons:        - title: "👍 Satisfied"          payload: "/SetSlots(csat_score=satisfied)"        - title: "👎 Not Satisfied"          payload: "/SetSlots(csat_score=unsatisfied)"  utter_ask_interrupted_flow_to_continue:    - text: "Would you like to resume {{context.interrupted_flow_options}}?"      metadata:        rephrase: True        template: jinja  utter_ask_rephrase:    - text: I’m sorry I am unable to understand you, could you please rephrase?  utter_ask_still_there:    - text: "Hello, are you still there?"      metadata:        rephrase: True  utter_boolean_slot_rejection:    - text: "Sorry, the value you provided, `{{value}}`, is not valid. Please respond with a valid value."      metadata:        rephrase: True        template: jinja  utter_can_do_something_else:    - text: "What exactly can I help you with?"      metadata:        rephrase: True  utter_cannot_handle:    - text: I'm sorry, I'm not trained to help with that.  utter_categorical_slot_rejection:    - text: "Sorry, you responded with an invalid value - `{{value}}`. Please select one of the available options."      metadata:        rephrase: True        template: jinja  utter_clarification_no_options_rasa:    - text: "I can help, but I need more information. Please tell me a bit more about what you would like to do."      metadata:        rephrase: True        template: jinja  utter_clarification_options_rasa:    - text: "I can help, but I need more information. Which of these would you like to do: {{context.clarification_options}}?"      metadata:        rephrase: True        template: jinja  utter_closing_words:    - text: "Okay, I'll be around in case you need further help."      metadata:        rephrase: True  utter_corrected_previous_input:    - text: "Ok, I am updating {{ context.corrected_slots.keys()|join(', ') }} to {{ context.new_slot_values | join(', ') }} respectively."      metadata:        rephrase: True        template: jinja  utter_csat_thank_you_satisfied:    - text: "Thank you for your feedback! I'm glad I could help."      metadata:        rephrase: True  utter_csat_thank_you_unsatisfied:    - text: "Thank you for your feedback. I'm sorry I couldn't fully meet your expectations."      metadata:        rephrase: True  utter_float_slot_rejection:    - text: "Sorry, it seems the value you provided `{{value}}` is not a valid number. Please provide a valid number in your response."      metadata:        rephrase: True        template: jinja  utter_flow_cancelled_rasa:    - text: "Okay, stopping {{ context.canceled_name }}."      metadata:        rephrase: True        template: jinja  utter_free_chitchat_response:    - text: "Sorry, I'm not able to answer that right now."      metadata:        rephrase: True        rephrase_prompt: |          You are an incredibly friendly assistant. Generate a short          response to the user's comment in simple english.          User: {{current_input}}          Response:  utter_human_handoff_not_available:    - text: I understand you want to be connected to a human agent, but that's something I cannot help you with at the moment. Is there something else I can help you with?      metadata:        rephrase: True  utter_inform_code_change:    - text: There has been an update to my code. I need to wrap up our running dialogue and start from scratch.      metadata:        rephrase: True  utter_inform_hangup:    - text: I haven’t heard from you, so I’ll end our conversation shortly.      metadata:        rephrase: True  utter_internal_error_rasa:    - text: Sorry, I am having trouble with that. Please try again in a few minutes.  utter_no_knowledge_base:    - text: I am afraid, I don't know the answer. At this point, I don't have access to a knowledge base.      metadata:        rephrase: True  utter_no_relevant_answer_found:    - text: I’m sorry, I can’t help with that.      metadata:        rephrase: True  utter_skip_question_answer:    - text: I'm here to provide you with the best assistance, and in order to do so, I kindly request that we complete this step together. Your input is essential for a seamless experience!      metadata:        rephrase: True  utter_user_input_empty_error_rasa:    - text: I see an empty message. What can I assist you with?  utter_user_input_too_long_error_rasa:    - text: I'm sorry, but your message is too long for me to process. Please keep your message concise and within {% if context.info.max_characters %}{{context.info.max_characters}} characters.{% else %}a reasonable length.{% endif %}      metadata:        template: jinjaslots:  confirm_correction:    type: bool    mappings:      - type: from_llm  silence_timeout:    type: float    initial_value: 7.0    max_value: 1000000  consecutive_silence_timeouts:    type: float    initial_value: 0.0    max_value: 1000000  interrupted_flow_to_continue:    type: text    mappings:      - type: from_llm  continue_interrupted_flow_confirmation:    type: bool    mappings:      - type: from_llm  max_clarification_options:    type: float    initial_value: 3.0  continue_conversation:    type: bool    mappings:      - type: from_llm  csat_score:    type: categorical    values:      - satisfied      - unsatisfied    mappings:      - type: from_llmflows:  pattern_cancel_flow:    description: Conversation repair flow that starts when a flow is cancelled    name: pattern_cancel_flow    steps:      - action: action_cancel_flow      - action: utter_flow_cancelled_rasa  pattern_cannot_handle:    description: Conversation repair flow for addressing failed command generation scenarios    name: pattern_cannot_handle    steps:      - noop: true        next:          # Fallback for ChitChat command when IntentlessPolicy isn't set, but          # pattern_chitchat invokes action_trigger_chitchat          - if: context.reason is "cannot_handle_chitchat"            then:              - action: utter_cannot_handle                next: "END"          # Fallback for things that are not supported          - if: context.reason is "cannot_handle_not_supported"            then:              - action: utter_cannot_handle                next: END          # Fallback when no relevant answer to the user query has been found.          # This is used by the EnterpriseSearchPolicy.          - if: context.reason is "cannot_handle_no_relevant_answer"            then:              - action: utter_no_relevant_answer_found                next: END          # Default          - else:              - action: utter_ask_rephrase                next: END  pattern_chitchat:    description: Conversation repair flow for off-topic interactions that won't disrupt the main conversation    name: pattern chitchat    steps:      - action: utter_cannot_handle      # To enable free-form response use:      # - action: utter_free_chitchat_response  pattern_clarification:    description: Conversation repair flow for handling ambiguous requests that could match multiple flows    name: pattern clarification    steps:      - noop: true        next:          - if: context.names            then: action_clarify_flows          - else:            - action: utter_clarification_no_options_rasa              next: END      - id: action_clarify_flows        action: action_clarify_flows      - action: utter_clarification_options_rasa    pattern_code_change:    description: Conversation repair flow for cleaning the stack after an assistant update    name: pattern code change    steps:      - action: utter_inform_code_change      - action: action_clean_stack  pattern_collect_information:    description: Flow for collecting information from users    name: pattern collect information    steps:      - id: start        action: action_run_slot_rejections      - action: validate_{{context.collect}}        next:          - if: "slots.{{context.collect}} is not null"            then: END          - else: ask_collect      - id: ask_collect        action: "{{context.utter}}"      - action: "{{context.collect_action}}"      - action: action_listen        next: start  pattern_completed:    description: Flow that asks if the user needs more help after completing their initiated use cases    name: pattern completed    steps:      - collect: continue_conversation        description: Set this slot to `True` if the user needs further help and wants to continue the conversation. Set it to `False` otherwise.        ask_before_filling: true        next:          - if: slots.continue_conversation            then:            - action: utter_can_do_something_else              next: END          - else:            - action: utter_closing_words              next: link_csat            - id: link_csat              link: pattern_customer_satisfaction  pattern_continue_interrupted:    description: Conversation repair flow for managing when users switch between different flows    name: pattern continue interrupted    steps:      - noop: true        next:          - if: context.multiple_flows_interrupted            then: collect_interrupted_flow_to_continue          - else: collect_continue_interrupted_flow_confirmation      - id: collect_interrupted_flow_to_continue        collect: interrupted_flow_to_continue        description: "Fill this slot with the name of the flow the user wants to continue. If the user does not want to continue any of the interrupted flows, fill this slot with 'none'."        next:          - if: slots.interrupted_flow_to_continue is not "none"            then:              - action: action_continue_interrupted_flow                next: END          - else:              - action: action_cancel_interrupted_flows                next: END      - id: collect_continue_interrupted_flow_confirmation        collect: continue_interrupted_flow_confirmation        description: "If the user wants to continue the interrupted flow, fill this slot with true. If the user does not want to continue the interrupted flow, fill this slot with false."        next:          - if: slots.continue_interrupted_flow_confirmation            then:              - action: action_continue_interrupted_flow                next: END          - else:              - action: action_cancel_interrupted_flows                next: END  pattern_correction:    description: Conversation repair flow for managing user input changes or error corrections    name: pattern correction    steps:      - action: action_correct_flow_slot        next:          - if: not context.is_reset_only            then:              - action: utter_corrected_previous_input                next: END          - else: END  pattern_customer_satisfaction:    description: Flow that collects customer satisfaction feedback after conversation completion    name: pattern customer satisfaction    run_pattern_completed: false    steps:      - collect: csat_score        description: Set this slot based on the user's satisfaction with the conversation. Set to 'satisfied' if the user is happy, 'unsatisfied' if the user is not happy.        ask_before_filling: true      - id: csat_response        noop: true        next:          - if: slots.csat_score == "satisfied"            then:              - action: utter_csat_thank_you_satisfied                next: END          - if: slots.csat_score == "unsatisfied"            then:              - action: utter_csat_thank_you_unsatisfied                next: END          - else: END  pattern_human_handoff:    description: Conversation repair flow for switching users to a human agent if their request can't be handled    name: pattern human handoff    steps:      - action: action_handoff_metric      - action: utter_human_handoff_not_available  pattern_internal_error:    description: Conversation repair flow for informing users about internal errors    name: pattern_internal_error    steps:      - noop: true        next:          - if: context.error_type is "rasa_internal_error_user_input_too_long"            then:              - action: utter_user_input_too_long_error_rasa                next: END          - if: context.error_type is "rasa_internal_error_user_input_empty"            then:              - action: utter_user_input_empty_error_rasa                next: END          - else:              - action: utter_internal_error_rasa                next: END  pattern_repeat_bot_messages:    description: Conversation repair flow for repeating previous messages    name: pattern repeat bot messages    steps:      - action: action_repeat_bot_messages  pattern_restart:    description: Flow for restarting the conversation    name: pattern restart    nlu_trigger:      - intent: restart    steps:      - action: action_restart  pattern_search:    description: Flow for handling knowledge-based questions    name: pattern search    steps:      - action: utter_no_knowledge_base      # - action: action_trigger_search to use doc search policy if present  pattern_session_start:    description: Flow for starting the conversation    name: pattern session start    nlu_trigger:      - intent: session_start    steps:      - action: action_session_start  pattern_skip_question:    description: Conversation repair flow for managing user intents to skip questions (steps)    name: pattern skip question    steps:      - action: utter_skip_question_answer  pattern_user_silence:    description: Reacting to user silence in voice bots    name: pattern react to silence    nlu_trigger:      - intent: silence_timeout    persisted_slots:      - consecutive_silence_timeouts    steps:      - noop: true        next:          - if: "slots.consecutive_silence_timeouts = 0.0"            then: set_slots_consecutive_silence_timeouts          - if: "slots.consecutive_silence_timeouts = 1.0"            then: set_slots_consecutive_silence_timeouts_2          - if: "slots.consecutive_silence_timeouts > 1.0"            then: message_utter_inform_hangup          - else: END      - id: set_slots_consecutive_silence_timeouts        set_slots:          - consecutive_silence_timeouts: 1.0      - action: action_repeat_bot_messages        next: END      - id: set_slots_consecutive_silence_timeouts_2        set_slots:          - consecutive_silence_timeouts: 2.0      - action: utter_ask_still_there        next: END      - id: message_utter_inform_hangup        action: utter_inform_hangup      - action: action_hangup  pattern_validate_slot:    description: Flow for running validations on slots    name: pattern validate slot    steps:      - id: start        action: action_run_slot_rejections        next:          - if: "slots.{{context.validate}} is not null"            then: END          - else: ask_refill      - id: ask_refill        action: "{{context.refill_utter}}"      - action: "{{context.refill_action}}"      - action: action_listen        next: start
```

- [Configurations](https://rasa.com/docs/reference/primitives/patterns/#configurations)
 - [Default Behavior](https://rasa.com/docs/reference/primitives/patterns/#default-behavior)
 - [Modifying default behaviour](https://rasa.com/docs/reference/primitives/patterns/#modifying-default-behaviour)
 - [Sample Configuration](https://rasa.com/docs/reference/primitives/patterns/#sample-configuration)
- [Common Modifications](https://rasa.com/docs/reference/primitives/patterns/#common-modifications)
 - [Requiring Confirmation](https://rasa.com/docs/reference/primitives/patterns/#requiring-confirmation)
 - [Implementing a Human Handoff](https://rasa.com/docs/reference/primitives/patterns/#implementing-a-human-handoff)
 - [React dependent on the current flow](https://rasa.com/docs/reference/primitives/patterns/#react-dependent-on-the-current-flow)
 - [Free form generation for chitchat](https://rasa.com/docs/reference/primitives/patterns/#free-form-generation-for-chitchat)
 - [Skipping clarification](https://rasa.com/docs/reference/primitives/patterns/#skipping-clarification)
 - [Preventing multiple Clarifications](https://rasa.com/docs/reference/primitives/patterns/#preventing-multiple-clarifications)
 - [Triggering clarification when multiple `StartFlow` commands are predicted](https://rasa.com/docs/reference/primitives/patterns/#triggering-clarification-when-multiple-startflow-commands-are-predicted)
 - [Configuring the maximum number of clarification options](https://rasa.com/docs/reference/primitives/patterns/#configuring-the-maximum-number-of-clarification-options)
 - [Using the rephraser for repeats](https://rasa.com/docs/reference/primitives/patterns/#using-the-rephraser-for-repeats)
- [Common Voice-specific Pattern Modifications](https://rasa.com/docs/reference/primitives/patterns/#common-voice-specific-pattern-modifications)
 - [Handling Call Start and Call Metadata](https://rasa.com/docs/reference/primitives/patterns/#handling-call-start-and-call-metadata)
 - [Handling the End of a Call](https://rasa.com/docs/reference/primitives/patterns/#handling-the-end-of-a-call)
 - [Using Responses relevant to Voice Channels](https://rasa.com/docs/reference/primitives/patterns/#using-responses-relevant-to-voice-channels)
- [Interruption Handling](https://rasa.com/docs/reference/primitives/patterns/#interruption-handling)
 - [How It Works](https://rasa.com/docs/reference/primitives/patterns/#how-it-works)
 - [Supported Channels](https://rasa.com/docs/reference/primitives/patterns/#supported-channels)
 - [Configuration](https://rasa.com/docs/reference/primitives/patterns/#configuration)
 - [Disabling Interruptions for Specific Responses](https://rasa.com/docs/reference/primitives/patterns/#disabling-interruptions-for-specific-responses)
 - [Best Practices](https://rasa.com/docs/reference/primitives/patterns/#best-practices)
- [Reference: Default Pattern Configuration](https://rasa.com/docs/reference/primitives/patterns/#reference-default-pattern-configuration)