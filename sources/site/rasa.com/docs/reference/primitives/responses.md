# Source: https://rasa.com/docs/reference/primitives/responses

On this page

## Defining Responses[​](https://rasa.com/docs/reference/primitives/responses/#defining-responses 'Direct link to Defining Responses')

Responses go under the `responses` key in your domain file or in a separate "responses.yml" file. Each response name should start with `utter_`. For example, you could add responses for greeting and saying goodbye under the response names `utter_greet` and `utter_bye`:

domain.yml

```
intents:  - greetresponses:  utter_greet:  - text: "Hi there!"  utter_bye:  - text: "See you!"
```

If you are using [retrieval intents](https://legacy-docs-oss.rasa.com/docs/rasa/glossary#retrieval-intent) in your assistant, you also need to add responses for your assistant's replies to these intents:

```
intents:  - chitchatresponses:  utter_chitchat/ask_name:  - text: Oh yeah, I am called the retrieval bot.  utter_chitchat/ask_weather:  - text: Oh, it does look sunny right now in Berlin.
```

note

Notice the special format of response names for retrieval intents. Each name starts with `utter_`, followed by the retrieval intent's name (here `chitchat`) and finally a suffix specifying the different response keys (here `ask_name` and `ask_weather`). See [the documentation for NLU training examples](https://rasa.com/docs/reference/primitives/training-data-format/#training-examples) to learn more.

### Using Variables in Responses[​](https://rasa.com/docs/reference/primitives/responses/#using-variables-in-responses 'Direct link to Using Variables in Responses')

You can use variables to insert information into responses. Within a response, a variable is enclosed in curly brackets. For example, see the variable `name` below:

domain.yml

```
responses:  utter_greet:  - text: "Hey, {name}. How are you?"
```

When the `utter_greet` response is used, Rasa automatically fills in the variable with the value found in the slot called `name`. If such a slot doesn't exist or is empty, the variable gets filled with `None`.

Another way to fill in a variable is within a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/). In your custom action code, you can supply values to a response to fill in specific variables. If you're using the Rasa SDK for your action server, you can pass a value for the variable as a keyword argument to [`dispatcher.utter_message`](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/):

```
dispatcher.utter_message(    template="utter_greet",    name="Sara")
```

If you use a [different custom action server](https://rasa.com/docs/action-server/#other-action-servers), supply the values by adding extra parameters to the responses your server returns:

```
{  "events":[    ...  ],  "responses":[    {      "template":"utter_greet",      "name":"Sara"    }  ]}
```

### Response Variations[​](https://rasa.com/docs/reference/primitives/responses/#response-variations 'Direct link to Response Variations')

You can make your assistant's replies more interesting if you provide multiple response variations to choose from for a given response name:

domain.yml

```
responses:  utter_greet:  - text: "Hey, {name}. How are you?"  - text: "Hey, {name}. How is your day going?"
```

In this example, when `utter_greet` gets predicted as the next action, Rasa will randomly pick one of the two response variations to use.

#### IDs for Responses[​](https://rasa.com/docs/reference/primitives/responses/#ids-for-responses 'Direct link to IDs for Responses')

New in Rasa 3.6

You can now set an ID for any response. This is useful when you want to use the [NLG server](https://rasa.com/docs/reference/integrations/nlg/) to generate the response.

Type for ID is string.

Example of response variations with ID:

domain.yml

```
responses:  utter_greet:  - id: "greet_1"    text: "Hey, {name}. How are you?"  - id: "greet_2"    text: "Hey, {name}. How is your day going?"
```

### Channel-Specific Response Variations[​](https://rasa.com/docs/reference/primitives/responses/#channel-specific-response-variations 'Direct link to Channel-Specific Response Variations')

To specify different response variations depending on which channel the user is connected to, use channel-specific response variations.

In the following example, the `channel` key makes the first response variation channel-specific for the `slack` channel while the second variation is not channel-specific:

domain.yml

```
responses:  utter_ask_game:  - text: "Which game would you like to play on Slack?"    channel: "slack"  - text: "Which game would you like to play?"
```

note

Make sure the value of the `channel` key matches the value returned by the `name()` method of your input channel. If you are using a built-in channel, this value will also match the channel name used in your `credentials.yml` file.

When your assistant looks for suitable response variations under a given response name, it will first try to choose from channel-specific variations for the current channel. If there are no such variations, the assistant will choose from any response variations which are not channel-specific.

In the above example, the second response variation has no `channel` specified and can be used by your assistant for all channels other than `slack`.

caution

For each response, try to have at least one response variation without the `channel` key. This allows your assistant to properly respond in all environments, such as in new channels, in the shell and in interactive learning.

### Conditional Response Variations[​](https://rasa.com/docs/reference/primitives/responses/#conditional-response-variations 'Direct link to Conditional Response Variations')

Specific response variations can also be selected based on one or more slot values using a conditional response variation. A conditional response variation is defined in the domain or responses YAML files similarly to a standard response variation but with an additional `condition` key.

#### Predicate Conditions[​](https://rasa.com/docs/reference/primitives/responses/#predicate-conditions 'Direct link to Predicate Conditions')

New in 3.13

You can now use predicate expressions to define conditions using the `slots` [namespace](https://rasa.com/docs/reference/primitives/conditions/#slots) for conditional response variations.

This feature was originally released as a beta feature in Rasa Pro 3.12.0 and is now generally available (GA).

The `condition` key can now specify a predicate statement, similar to the usage of [conditions in flows](https://rasa.com/docs/reference/primitives/conditions/). These predicates enable you to use a variety of logical operators, comparison operators and other constructs. They are evaluated with the [pypred](https://github.com/armon/pypred) library.

For example:

domain.yml

```
responses:  utter_greet:    - condition: slots.prior_visits > 1      text: "Hey, {name}. Nice to see you again! How are you?"    - condition: not slots.prior_visits      text: "Welcome. How is your day going?"
```

In the example above, the first response variation (`"Hey, {name}. Nice to see you again! How are you?"`) will be used whenever the `utter_greet` action is executed and the `prior_visits` slot is greater than `1`. The second variation, which has a condition that the `prior_visits` slot is not set, will be used when the slot is not set.

#### `Name` and `Value` Equality Constraints[​](https://rasa.com/docs/reference/primitives/responses/#name-and-value-equality-constraints 'Direct link to name-and-value-equality-constraints')

Deprecation Warning

Writing the condition as a list of dictionaries consisting of `name` and `value` constraints is deprecated. This format will be removed in the next major release of Rasa.

The `condition` key specifies a list of slot `name` and `value` constraints. When a response is triggered during a dialogue, the constraints of each conditional response variation are checked against the current dialogue state. If all constraint slot values are equal to the corresponding slot values of the current dialogue state, the response variation is eligible to be used by your conversational assistant.

note

The comparison of dialogue state slot values and constraint slot values is performed by the equality "==" operator which requires the type of slot values to match too. For example, if the constraint is specified as `value: true`, then the slot needs to be filled with a boolean `true`, not the string `"true"`.

In the following example, we will define one conditional response variation with one constraint, that the `logged_in` slot is set to `true`:

domain.yml

```
slots:  logged_in:    type: bool    mappings:      - type: controlled  name:    type: textresponses:  utter_greet:    - condition:        - type: slot          name: logged_in          value: true      text: "Hey, {name}. Nice to see you again! How are you?"    - text: "Welcome. How is your day going?"
```

flows.yml

```
flows:  Greet:    name: Greet    description: This flow is called to greet customers at the start of the conversation.    steps:      - action: logged_in      - action: utter_Greet
```

In the example above, the first response variation (`"Hey, {name}. Nice to see you again! How are you?"`) will be used whenever the `utter_greet` action is executed and the `logged_in` slot is set to `true`. The second variation, which has no condition, will be treated as the default and used whenever `logged_in` is not equal to `true`.

#### Variation Selection[​](https://rasa.com/docs/reference/primitives/responses/#variation-selection 'Direct link to Variation Selection')

During a dialogue, Rasa will choose from all conditional response variations whose constraints are satisfied. If there are multiple eligible conditional response variations, Rasa will pick one at random. For example, consider the following response:

domain.yml

```
responses:  utter_greet:    - condition: slots.logged_in is true      text: "Hey, {name}. Nice to see you again! How are you?"    - condition: slots.eligible_for_upgrade is true      text: "Welcome, {name}. Did you know you are eligible for a free upgrade?"    - text: "Welcome. How is your day going?"
```

If `logged_in` and `eligible_for_upgrade` are both set to `true` then both the first and second response variations are eligible to be used, and will be chosen by the conversational assistant with equal probability.

You can continue using channel-specific response variations alongside conditional response variations as shown in the example below.

domain.yml

```
slots:  logged_in:    type: bool    mappings:      - type: controlled  name:    type: textresponses:  utter_greet:    - condition: slots.logged_in is true      text: "Hey, {name}. Nice to see you again on Slack! How are you?"      channel: slack    - text: "Welcome. How is your day going?"
```

Rasa will prioritize the selection of responses in the following order:

1. conditional response variations with matching channel
2. default responses with matching channel
3. conditional response variations with no matching channel
4. default responses with no matching channel

caution

It is highly recommended to always provide a default response variation without a condition to guard against those cases when no conditional response matches filled slots. If no conditional response variation is eligible to be used and no default is provided, Rasa will respond with the default utterance `utter_internal_error_rasa`.

## Rich Responses[​](https://rasa.com/docs/reference/primitives/responses/#rich-responses 'Direct link to Rich Responses')

You can make responses rich by adding visual and interactive elements. There are several types of elements that are supported across many channels:

### Buttons[​](https://rasa.com/docs/reference/primitives/responses/#buttons 'Direct link to Buttons')

You can add buttons to a response to allow the user to select from a list of options. The buttons are displayed as clickable elements in the chat window.

Each button in the list of `buttons` should have two keys:

- `title`: The text displayed on the buttons that the user sees.
- `payload`: The message sent from the user to the assistant when the button is clicked.

Button payloads can be used to either:

- [trigger intents and pass entities](https://rasa.com/docs/reference/primitives/responses/#triggering-intents-or-passing-entities) to the assistant.
- issue [commands to set slots](https://rasa.com/docs/reference/primitives/responses/#issuing-set-slot-commands)
- pass a predefined free-form string message to the assistant. Note that this option should be used if none of the above options are feasible.

In addition, buttons provide the advantage of skipping the NLU pipeline and directly annotating the user message with the intent, entities or set slot commands defined in the payload.

Check your channel

Keep in mind that it is up to the implementation of the output channel how to display the defined buttons. For example, some channels have a limit on the number of buttons you can provide. Check your channel's documentation under **Concepts > Channel Connectors** for any channel-specific restrictions.

#### Triggering Intents or Passing Entities[​](https://rasa.com/docs/reference/primitives/responses/#triggering-intents-or-passing-entities 'Direct link to Triggering Intents or Passing Entities')

Here is an example of a response that uses buttons to trigger an intent:

domain.yml

```
responses:  utter_greet:  - text: "Hey! How are you?"    buttons:    - title: "great"      payload: "/mood_great"    - title: "super sad"      payload: "/mood_sad"
```

If you would like the buttons to also pass entities to the assistant:

domain.yml

```
responses:  utter_greet:  - text: "Hey! Would you like to purchase motor or home insurance?"    buttons:    - title: "Motor insurance"      payload: '/inform{{"insurance":"motor"}}'    - title: "Home insurance"      payload: '/inform{{"insurance":"home"}}'
```

Passing multiple entities is also possible with:

```
'/intent_name{{"entity_type_1":"entity_value_1", "entity_type_2": "entity_value_2"}}'
```

overwrite nlu with buttons

You can use buttons to overwrite the NLU prediction and trigger a specific intent and entities.

Messages starting with `/` are sent handled by the `RegexInterpreter`, which expects NLU input in a shortened `/intent{entities}` format. In the example above, if the user clicks a button, the user input will be classified as either the `mood_great` or `mood_sad` intent.

You can include entities with the intent to be passed to the `RegexInterpreter` using the following format:

`/inform{"ORG":"Rasa", "GPE":"Germany"}`

The `RegexInterpreter` will classify the message above with the intent `inform` and extract the entities `Rasa` and `Germany` which are of type `ORG` and `GPE` respectively.

escaping curly braces in domain.yml

You need to write the `/intent{entities}` shorthand response with double curly braces in domain.yml so that the assistant does not treat it as a [variable in a response](https://rasa.com/docs/reference/primitives/responses/#using-variables-in-responses) and interpolate the content within the curly braces.

#### Issuing Set Slot Commands[​](https://rasa.com/docs/reference/primitives/responses/#issuing-set-slot-commands 'Direct link to Issuing Set Slot Commands')

New in 3.9.0

Starting from Rasa Pro 3.9.0, you can use buttons to issue commands to set slots.

##### Payload Syntax[​](https://rasa.com/docs/reference/primitives/responses/#payload-syntax 'Direct link to Payload Syntax')

To issue [`set slot` commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference), you can use the following format in the payload: `/SetSlots(slot_name=slot_value)`. You can define multiple slot key-value pairs in the same command. The slot names that you define in the payload should be slots that are requested via the active flow or flows that the command generator predicts a `StartFlow` command for in the same turn.

Note that there is a limit of 10 slot key-value pairs per command to prevent Regular expression Denial of Service (ReDoS) attacks.

Here is an example:

domain.yml

```
responses:  utter_contactless_limit:  - text: "Which card would you like to set the maximum contactless limit for?"    buttons:    - title: "credit"      payload: "/SetSlots(amount=100, card_type=credit)"    - title: "debit"      payload: "/SetSlots(amount=100, card_type=debit)"
```

Note that the `SetSlots` command is case-sensitive and should be written exactly as shown above. The regular expression used for extracting slot names and values from the payload does not allow the following characters:

- `=`, `,`, `(`, `)` in the slot name
- `,`, `(`, `)` in the slot value

You can also use this syntax to start a flow by first setting a slot and then branching on that slot to execute a [link](https://rasa.com/docs/reference/primitives/flow-steps/#link) or a [call](https://rasa.com/docs/reference/primitives/flow-steps/#call) step.

caution

All slot types are supported to be filled by buttons except for `list` slots.

#### Dynamic Buttons[​](https://rasa.com/docs/reference/primitives/responses/#dynamic-buttons 'Direct link to Dynamic Buttons')

You can also create a dynamic list of buttons in a reply via a custom action. Maybe the list of responses come from an API or the list of buttons is determined based on the value of another slot or the state of the conversation.

This can be done via a collect step and a custom action called `action_ask_{slot_name}`.

For example, let's say your bot needs to ask the user which of their credit cards they want help with. We would create a response without the buttons and then use a custom action to get the list of cards associated with the user.

There is also a `cards` slot with a list of all the users cards. This was loaded when the user first connected to the bot via `action_session_start`. There are also slots with the current card name and number.

domain.yml

```
slots:  current_card_name:    type: text  current_card_number:    type: text  cards:    type: listresponses:  utter_select_card:    - text: "Here are the your cards, select the one you are referring to?"
```

The `select_card` flow does a `collect: current_card_name` to request the current card from the user.

flow.yml

```
flows:  select_card:    description: This flow is called when the user has multiple cards and needs to select one.    name: Select card    # block this flow from th list of possible flows for the LLM, it should only be called from other flows    if: False    steps:      - collect: current_card_name        ask_before_filling: true        next: END
```

Create a custom action named `action_ask_current_card_name` which the flow `collect` will call.

actions.py

```
class ActionShowSlots(Action):    def name(self):        return "action_ask_current_card_name"    def run(self, dispatcher, tracker, domain):        events = []        cards = tracker.get_slot("cards")        if not cards:            dispatcher.utter_message(text="No cards found.")        else:            buttons = []            for card in cards:                buttons.append(                    {                        "title": card.get("name"),                        "payload": f"/SetSlots(current_card_name={card.get('name')}, current_card_number={card.get('number')})"                    }                )            dispatcher.utter_message(response="utter_select_card", buttons=buttons)        return events
```

You can read more about the `action_ask_{slot_name}` [here](https://rasa.com/docs/reference/primitives/flow-steps/#using-an-action-to-ask-for-information-in-collect-step)

### Images[​](https://rasa.com/docs/reference/primitives/responses/#images 'Direct link to Images')

You can add images to a response by providing a URL to the image under the `image` key:

domain.yml

```
  utter_cheer_up:  - text: "Here is something to cheer you up:"    image: "https://i.imgur.com/nGF1K8f.jpg"
```

### Custom Output Payloads[​](https://rasa.com/docs/reference/primitives/responses/#custom-output-payloads 'Direct link to Custom Output Payloads')

You can send any arbitrary output to the output channel using the `custom` key. The output channel receives the object stored under the `custom` key as a JSON payload.

Here's an example of how to send a [date picker](https://api.slack.com/reference/block-kit/block-elements#datepicker) to the [Slack Output Channel](https://rasa.com/docs/reference/channels/slack/):

domain.yml

```
responses:  utter_take_bet:  - custom:      blocks:      - type: section        text:          text: "Make a bet on when the world will end:"          type: mrkdwn        accessory:          type: datepicker          initial_date: '2019-05-21'          placeholder:            type: plain_text            text: Select a date
```

### Voice-Specific Response Properties[​](https://rasa.com/docs/reference/primitives/responses/#voice-specific-response-properties 'Direct link to Voice-Specific Response Properties')

For voice assistants, you can control specific behaviors of individual responses using voice-specific properties.

#### Controlling Interruptions[​](https://rasa.com/docs/reference/primitives/responses/#controlling-interruptions 'Direct link to Controlling Interruptions')

Beta Feature

Interruption handling is currently in beta and available for selected Voice Stream Channels only.

You can control whether users can interrupt the assistant while a specific response is being spoken using the `allow_interruptions` property:

domain.yml

```
responses:  utter_current_balance:    - text: You still have {current_balance} in your account.      allow_interruptions: false    utter_important_terms:    - text: Please note the following terms and conditions that apply to your account...      allow_interruptions: false  utter_ask_preference:    - text: What would you like to do today?      allow_interruptions: true  # This is the default
```

By default, `allow_interruptions` is `true` for all responses. Setting it to `false` ensures that users must hear the complete message before the assistant will respond to their input.

**When to use `allow_interruptions: false`:**

- Critical information that users must hear completely (account balances, terms and conditions, emergency information)
- Legal disclaimers or compliance-related content
- Important instructions that shouldn't be missed

👉 [Learn more about interruption handling](https://rasa.com/docs/reference/primitives/patterns/#interruption-handling)

## Using Responses in Conversations[​](https://rasa.com/docs/reference/primitives/responses/#using-responses-in-conversations 'Direct link to Using Responses in Conversations')

### Calling Responses as Actions[​](https://rasa.com/docs/reference/primitives/responses/#calling-responses-as-actions 'Direct link to Calling Responses as Actions')

If the name of the response starts with `utter_`, the response can directly be used as an action, without being listed in the `actions` section of your domain. You would add the response to the domain:

domain.yml

```
responses:  utter_greet:  - text: "Hey! How are you?"
```

You can use that same response as an action in your Flows:

flows.yml

```
flows:  Greet:    name: Greet    description: This flow is called to greet customers at the start of the conversation.    steps:      - action: utter_Greet
```

When the `utter_greet` action runs, it will send the message from the response back to the user.

Changing responses

If you want to change the text, or any other part of the response, you need to retrain the assistant before these changes will be picked up.

### Calling Responses from Custom Actions[​](https://rasa.com/docs/reference/primitives/responses/#calling-responses-from-custom-actions 'Direct link to Calling Responses from Custom Actions')

You can use the responses to generate response messages from your custom actions. If you're using Rasa SDK as your action server, you can use the dispatcher to generate the response message, for example:

actions.py

```
from rasa_sdk.interfaces import Actionclass ActionGreet(Action):    def name(self):        return 'action_greet'    def run(self, dispatcher, tracker, domain):        dispatcher.utter_message(template="utter_greet")        return []
```

If you use a [different custom action server](https://rasa.com/docs/action-server/#other-action-servers), your server should return the following JSON to call the `utter_greet` response:

```
{  "events": [],  "responses": [    {      "template": "utter_greet"    }  ]}
```

- [Defining Responses](https://rasa.com/docs/reference/primitives/responses/#defining-responses)
 - [Using Variables in Responses](https://rasa.com/docs/reference/primitives/responses/#using-variables-in-responses)
 - [Response Variations](https://rasa.com/docs/reference/primitives/responses/#response-variations)
 - [Channel-Specific Response Variations](https://rasa.com/docs/reference/primitives/responses/#channel-specific-response-variations)
 - [Conditional Response Variations](https://rasa.com/docs/reference/primitives/responses/#conditional-response-variations)
- [Rich Responses](https://rasa.com/docs/reference/primitives/responses/#rich-responses)
 - [Buttons](https://rasa.com/docs/reference/primitives/responses/#buttons)
 - [Images](https://rasa.com/docs/reference/primitives/responses/#images)
 - [Custom Output Payloads](https://rasa.com/docs/reference/primitives/responses/#custom-output-payloads)
 - [Voice-Specific Response Properties](https://rasa.com/docs/reference/primitives/responses/#voice-specific-response-properties)
- [Using Responses in Conversations](https://rasa.com/docs/reference/primitives/responses/#using-responses-in-conversations)
 - [Calling Responses as Actions](https://rasa.com/docs/reference/primitives/responses/#calling-responses-as-actions)
 - [Calling Responses from Custom Actions](https://rasa.com/docs/reference/primitives/responses/#calling-responses-from-custom-actions)