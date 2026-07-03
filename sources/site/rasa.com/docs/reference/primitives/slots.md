# Source: https://rasa.com/docs/reference/primitives/slots

On this page

## Slots[​](https://rasa.com/docs/reference/primitives/slots/#slots 'Direct link to Slots')

Slots are your assistant's memory. They act as a key-value store which can be used to store information the user provided (e.g. their home city) as well as information gathered about the outside world (e.g. the result of a database query).

Slots are defined in the slots section of your domain with their name, [type](https://rasa.com/docs/reference/primitives/slots/#slot-types) and default value. Different slot types exist to restrict the possible values a slot can take.

note

If you decide to fill slots through [response buttons](https://rasa.com/docs/reference/primitives/responses/#buttons) where the [payload syntax](https://rasa.com/docs/reference/primitives/responses/#payload-syntax) issues `SetSlot` command(s), note that the slot name must not include certain characters such as `(`, `)`, `=` or `,`.

### Slot Types[​](https://rasa.com/docs/reference/primitives/slots/#slot-types 'Direct link to Slot Types')

#### Text Slot[​](https://rasa.com/docs/reference/primitives/slots/#text-slot 'Direct link to Text Slot')

A text slot can take on any string value.

- **Example**
 
 ```
    slots:  cuisine:    type: text
    ```
 
- **Allowed Values**
 
 Any string
 

#### Boolean Slot[​](https://rasa.com/docs/reference/primitives/slots/#boolean-slot 'Direct link to Boolean Slot')

A boolean slot can only take on the values `true` or `false`. This is useful when you want to store a binary value.

- **Example**
 
 ```
    slots:  confirmation:    type: bool
    ```
 
- **Allowed Values**
 
 `true` or `false`
 

#### Categorical Slot[​](https://rasa.com/docs/reference/primitives/slots/#categorical-slot 'Direct link to Categorical Slot')

A categorical slot can only take on values from a predefined set. This is useful when you want to restrict the possible values a slot can take.

If the user provides a value where the casing does not match the casing of the values defined in the domain, the value will be coerced to the correct casing. For example, if the user provides the value `LOW` for a slot with values `low`, `medium`, `high`, the value will be converted to `low` and stored in the slot.

If you define a categorical slot with a list of values, where multiple of the values coerce to the same value, a warning will be issued and you should remove one of the values from the set in the domain. For example, if you define a categorical slot with values `low`, `medium`, `high`, and `Low`, the value `Low` will be coerced to `low` and a warning will be issued.

- **Example**
 
 ```
    slots:  risk_level:    type: categorical    values:      - low      - medium      - high
    ```
 

#### Float Slot[​](https://rasa.com/docs/reference/primitives/slots/#float-slot 'Direct link to Float Slot')

A float slot can only take on floating point values. This is useful when you want to store a number with a decimal point.

- **Example**
 
 ```
    slots:  temperature:    type: float
    ```
 

#### Any Slot[​](https://rasa.com/docs/reference/primitives/slots/#any-slot 'Direct link to Any Slot')

This slot type can take on any value. This is useful when you want to store any type of information, including structured data like dictionaries.

- **Example**
 
 ```
    slots:  shopping_items:    type: any
    ```
 

#### List Slot[​](https://rasa.com/docs/reference/primitives/slots/#list-slot 'Direct link to List Slot')

A list slot can take on a list of values. Note that the list slot type is only supported in [custom actions](https://rasa.com/docs/reference/primitives/custom-actions/) when building an assistant with [CALM](https://rasa.com/docs/learn/concepts/calm/). List slots cannot be filled with flows in either the [`collect`](https://rasa.com/docs/reference/primitives/flow-steps/#collect) or [`set_slots`](https://rasa.com/docs/reference/primitives/flow-steps/#set-slots) flow step types.

### Resetting a slot[​](https://rasa.com/docs/reference/primitives/slots/#resetting-a-slot 'Direct link to Resetting a slot')

To reset a slot in a flow, you can set it to `null` using the [set\_slots step](https://rasa.com/docs/reference/primitives/flow-steps/#set-slots):

```
- set_slots:  slot_name: null
```

If you want to reset a slot in a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/), set its value to `None`.

Slots that are empty are not eligible for [correction](https://rasa.com/docs/reference/primitives/patterns/#requiring-confirmation).

### CALM Slot Mappings[​](https://rasa.com/docs/reference/primitives/slots/#calm-slot-mappings 'Direct link to CALM Slot Mappings')

New in 3.9.0

When building an assistant with [CALM](https://rasa.com/docs/learn/concepts/calm/), you can configure slot filling to either use [nlu-based predefined](https://rasa.com/docs/reference/primitives/slots/#nlu-based-predefined-slot-mappings) slot mappings or the newly introduced [`from_llm`](https://rasa.com/docs/reference/primitives/slots/#from_llm) slot mapping type.

#### NLU-based predefined slot mappings[​](https://rasa.com/docs/reference/primitives/slots/#nlu-based-predefined-slot-mappings 'Direct link to NLU-based predefined slot mappings')

You can continue using the [nlu-based predefined](https://legacy-docs-oss.rasa.com/docs/rasa/domain#slot-mappings) slot mappings such as [`from_entity`](https://legacy-docs-oss.rasa.com/docs/rasa/domain#from_entity) or [`from_intent`](https://legacy-docs-oss.rasa.com/docs/rasa/domain#from_intent) when building an assistant with CALM. In addition to including tokenizers, featurizers, intent classifiers, and entity extractors to your pipeline, you must also add the [`NLUCommandAdapter`](https://rasa.com/docs/reference/config/components/nlu-command-adapter/) to the `config.yml` file. The `NLUCommandAdapter` will match the output of the NLU pipeline (intents and entities) against the slot mappings defined in the domain file. If the slot mappings are satisfied, the `NLUCommandAdapter` will issue [`set slot` commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference) to fill the slots.

Recommendations

1. We recommend adding the [`FallbackClassifier`](https://legacy-docs-oss.rasa.com/docs/rasa/components#fallbackclassifier) to the nlu pipeline to guard against low confidence scores for intents when these are used in `from_intent` slot mappings.
2. We recommend setting [`ask_before_filling: true`](https://rasa.com/docs/reference/primitives/flow-steps/#always-asking-questions) at the `collect` flow steps for slots that can be filled by the same entity in the same flow. This prevents the assistant from greedily filling all the slots with the same entity at the same time, when only one of the slots was requested.

- Rasa Pro < 3.12.0
- Rasa Pro >= 3.12.0

If during message processing, the `NLUCommandAdapter` issues commands, then the following command generators in the pipeline such as [LLM-based command generators](https://rasa.com/docs/reference/config/components/llm-command-generators/) will be entirely bypassed. As a consequence, LLM-based command generators will not be able to fill slots by issuing [`set slot` commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference) at any point in the conversation flow. If the LLM-based command generator issues commands to fill slots with nlu-based predefined mappings, these `set slot` commands from LLM-based command generator are ignored. If no other commands were predicted for the same turn, then the assistant will trigger the `cannot_handle` [conversation repair pattern](https://rasa.com/docs/learn/concepts/conversation-patterns/).

Sometimes the user message may contain intentions that go beyond setting a slot. For example, the user message may contain an entity that fills a slot but also starts a digression that must be handled. In such cases, we recommend using [NLU triggers](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger) to handle those specific intents within flows. Please refer to the [**Impact of slot mappings in different scenarios**](https://rasa.com/docs/reference/primitives/slots/#impact-of-slot-mappings-in-different-scenarios) section for more details.

note

In a CALM assistant built with flows and using NLU components to process the message, the default action [`action_extract_slots`](https://legacy-docs-oss.rasa.com/docs/rasa/default-actions#action_extract_slots) will not run, because the slot set events are applied to the dialogue tracker during command execution. This ensures that this default action does not overwrite CALM `set slot`(../config/components/llm-command-generators.mdx#command-reference) commands and does not duplicate `SlotSet` events that were already applied to the dialogue tracker.

In the case of [coexistence](https://rasa.com/docs/pro/calm-with-nlu/coexistence/), the `action_extract_slots` action will be executed only when the NLU-based [system](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#key-terms) is active.

If you are using a LLM-based command generator alongside the `NLUCommandAdapter` in the config pipeline, note that both the LLM-based command generator and the `NLUCommandAdapter` can now issue commands by default at any given conversation turn. These commands can be issued to fill slots with nlu-based predefined mappings or with the `from_llm` mapping type.

New in 3.12

A slot can now be defined with both nlu-based predefined mappings and the `from_llm` mapping type. The prior restriction that the `from_llm` mapping type cannot be used with nlu-based predefined mappings has been removed. For an in-depth explanation of the impact of slot mappings in different scenarios, refer to the [**Impact of slot mappings in different scenarios**](https://rasa.com/docs/reference/primitives/slots/#impact-of-slot-mappings-in-different-scenarios) section.

#### `from_llm`[​](https://rasa.com/docs/reference/primitives/slots/#from_llm 'Direct link to from_llm')

You can use the `from_llm` slot mapping type to fill slots with values generated by [LLM-based command generators](https://rasa.com/docs/reference/config/components/llm-command-generators/). This is the default slot mapping type if the mappings are not explicitly defined in the domain file.

Here is an example:

```
slots:  user_name:    type: text    mappings:      - type: from_llm
```

In this example, the `user_name` slot will be filled with the value generated by the LLM-based command generator. The LLM-based command generator is allowed to fill this slot at any point in the conversation flow, not just at the corresponding `collect` step for this slot.

- Rasa Pro < 3.12.0
- Rasa Pro >= 3.12.0

If you have defined additional NLU-based components in the `config.yml` pipeline, these components will continue to process the user message however they will not be able to fill `from_llm` slots. The `NLUCommandAdapter` will skip any slots with `from_llm` mappings and will not issue [`set slot` commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference) to fill these slots. Please refer to the [**Impact of slot mappings in different scenarios**](https://rasa.com/docs/reference/primitives/slots/#impact-of-slot-mappings-in-different-scenarios) section for more details.

Note that a slot must not have both `from_llm` and NLU-based predefined mappings or [custom slot mappings](https://rasa.com/docs/reference/primitives/slots/#custom-slot-mappings). If you define a slot with `from_llm` mapping, you cannot define any other mapping types for that slot.

##### allow\_nlu\_correction[​](https://rasa.com/docs/reference/primitives/slots/#allow_nlu_correction 'Direct link to allow_nlu_correction')

By default, the LLM-based command generator is not allowed to correct slots that have been filled by the NLU-based pipeline. If you want to allow the LLM-based command generator to correct slots that have been filled by the NLU-based pipeline, you can set the `allow_nlu_correction` property to `true` in the `from_llm` slot mapping:

```
slots:  username:    type: text    mappings:      - type: from_llm        allow_nlu_correction: true      - type: from_entity        entity: username
```

In this example, the `username` slot can be updated by the LLM-based command generator even if the slot has been previously filled by the NLU-based pipeline.

#### Mapping Conditions[​](https://rasa.com/docs/reference/primitives/slots/#mapping-conditions 'Direct link to Mapping Conditions')

You can define conditions for slot mappings to be satisfied before the slot is filled. The conditions are defined as a list of conditions under the `conditions` key. Each condition can specify the flow id that must be active to the `active_flow` property.

This is particularly useful if you define several slots mapped to the same entity, but you do not want to fill all of them when the entity is extracted.

For example:

```
entities:- personslots:  first_name:    type: text    mappings:      - type: from_entity        entity: person        conditions:          - active_flow: greet_user  last_name:    type: text    mappings:      - type: from_entity        entity: person        conditions:          - active_flow: issue_invoice
```

#### Controlled Slot Mappings[​](https://rasa.com/docs/reference/primitives/slots/#controlled-slot-mappings 'Direct link to Controlled Slot Mappings')

New in 3.12

The `controlled` slot mapping type can be used to define slots that should be filled by a custom action, [response button payload](https://rasa.com/docs/reference/primitives/responses/#payload-syntax), or a [`set_slots` flow step](https://rasa.com/docs/reference/primitives/flow-steps/#set-slots).

You can use the `controlled` mapping type to define slots that should be filled with values in a controlled manner. Slots that capture state or context necessary for the assistant to function are good examples of such slots.

For example:

```
slots:   is_logged_in:      type: bool      mappings:        - type: controlled
```

Slots that only define the new `controlled` slot mapping will not be available to be filled by the NLU or LLM components. Note that this slot mapping can still be used alongside these other slot mapping types, however this comes with the risk of the slot being filled by the NLU or LLM components in a probabilistic manner.

##### run\_action\_every\_turn[​](https://rasa.com/docs/reference/primitives/slots/#run_action_every_turn 'Direct link to run_action_every_turn')

In order to fill a slot with the `controlled` mapping type at every conversation turn, you can set the `run_action_every_turn` property to the name of the custom action that should fill the slot:

```
slots:  username:    type: text    mappings:      - type: controlled        run_action_every_turn: action_fill_username
```

##### coexistence\_system[​](https://rasa.com/docs/reference/primitives/slots/#coexistence_system 'Direct link to coexistence_system')

If you are building a [coexistence assistant](https://rasa.com/docs/pro/calm-with-nlu/coexistence/) where different `controlled` slots are set by custom actions in different subsystems, you must indicate which coexistence system is allowed to fill the slot. You can achieve this by setting the `coexistence_system` property in the slot mapping configuration. This property is a string that must match one of the available categorical values: `NLU`, `CALM`, `SHARED` (when either system can set the slot).

For example:

```
slots:  username:    type: text    mappings:      - type: controlled        run_action_every_turn: action_fill_username        coexistence_system: NLU
```

#### Custom Slot Mappings[​](https://rasa.com/docs/reference/primitives/slots/#custom-slot-mappings 'Direct link to Custom Slot Mappings')

Deprecation Warning

The `custom` slot mapping type is deprecated and will be removed in the next major release. Please use the [`controlled` slot mapping type](https://rasa.com/docs/reference/primitives/slots/#controlled-slot-mappings) instead for slots that should be filled deterministically by a custom action.

The `action` property in the slot mapping is deprecated and will be removed in the next major release. Please use the [`run_action_every_turn` property](https://rasa.com/docs/reference/primitives/slots/#run_action_every_turn) instead for slots that should be filled by a custom action at every conversation turn.

You can use the `custom` mapping type to define custom slot mappings for slots that should be filled by a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/). The custom action must be specified in the `action` property of the slot mapping. You must also list the action in the domain file under the `actions` key.

For example:

domain.yml

```
actions:- action_fill_user_nameslots:  user_name:    type: text    mappings:      - type: custom        action: action_fill_user_name
```

In this example, the `user_name` slot will be filled by the `action_fill_user_name` custom action. The custom action must return a `SlotSet` event with the slot name and value to fill the slot.

Note that if you're using the [`action_ask_<slot_name>` naming convention](https://rasa.com/docs/reference/primitives/flow-steps/#using-an-action-to-ask-for-information-in-collect-step) for requesting user input via a custom action, but the slot is filled by the value generated by the LLM-based command generator, you should not define a custom slot mapping for that slot. Instead, use `from_llm` mapping type, because `custom` mapping type is reserved for slots that are set by a custom action returning a `SlotSet` event (e.g. for slots set by external sources). You can continue using the `action_ask_<slot_name>` convention to request user input for slots that are filled by the LLM-based command generator.

If you are using custom validation actions (using the `validate_<slot_name>` naming convention) to validate slot values extracted by the LLM-based generator from the end user's input, you should not define custom slot mappings for those slots either. Instead, use the `from_llm` mapping type for those slots.

warning

If you are training with the `--skip-validation` flag and you have defined slots with custom slot mappings that do not specify the `action` property in the domain file, nor do they have corresponding `action_ask_<slot_name>` custom actions to request these slots, you will not receive errors at training time. However, at runtime, [`FlowPolicy`](https://rasa.com/docs/reference/config/policies/flow-policy/) will first cancel the user flow in progress and then trigger [`pattern_internal_error`](https://rasa.com/docs/learn/concepts/conversation-patterns/).

You can also run this check via the [`rasa data validate`](https://rasa.com/docs/reference/api/command-line-interface/#rasa-data-validate) command.

#### Impact of slot mappings in different scenarios[​](https://rasa.com/docs/reference/primitives/slots/#impact-of-slot-mappings-in-different-scenarios 'Direct link to Impact of slot mappings in different scenarios')

This section clarifies which components in a CALM assistant built with flows and a NLU pipeline are responsible for filling slots in different scenarios when the flow is at either the collect step for slot `name` or at any other step.

- Rasa Pro < 3.12.0
- Rasa Pro >= 3.12.0

1. Assume slot `name` is defined with the `from_llm` mapping type.

| Capability | Collect step for slot `name` | Any other step that does not collect the slot |
| --- | --- | --- |
| LLM-based generator is active | ✅ | ✅ |
| NLU components e.g. intent classifiers, entity extractors, are active | ✅ | ✅ |
| Can the LLM-based generator fill slot `name` | ✅ | ✅ |
| Can the `NLUCommandAdapter` fill slot `name` | ❌ | ❌ |

Main takeaway is that the `NLUCommandAdapter` cannot fill slots with `from_llm` mappings at any point in the conversation.

2. Assume slot `name` is defined with one of the NLU-based predefined mappings such as `from_entity`.

| Capability | Collect step for slot `name` | Any other step that does not collect the slot |
| --- | --- | --- |
| LLM-based generator is active | ❌ | ✅ |
| NLU components e.g. intent classifiers, entity extractors, are active | ✅ | ✅ |
| Can the LLM-based generator fill slot `name` | ❌ | ❌ |
| Can the `NLUCommandAdapter` fill slot `name` | ✅ | ✅ |

Main takeaways:

- The LLM-based generator cannot fill slots with NLU-based predefined mappings at any point in the conversation.
- The LLM-based generator will not be active at the collect step for slot `name`. If you expect the user utterance to contain digressions or other intentions beyond information for setting a slot, you should use [NLU triggers](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger) to handle those specific intents within flows.
- The LLM-based generator can fill other slots at steps where slot `name` is not collected and they have `from_llm` mapping type.

Assume that you have defined a slot `name` with both `from_entity` and the `from_llm` mapping types. The following scenarios describe the expected behaviour of filling the slot `name`:

| Scenario | Outcome |
| --- | --- |
| 
Slot `name` is currently empty. 
The NLU pipeline extracts the entity value that fills the slot `name`. 
The LLM also extracts a value for slot `name`.

 | 

The NLU based mapping takes higher priority. 
The slot mapping `from_entity` accepts the extracted value from the NLU pipeline. 
The LLM value is ignored.

 |
| 

Slot `name` is currently empty. 
The NLU pipeline does not extract the entity that fills the slot. 
The LLM extracts a value for slot `name`.

 | 

The LLM extracts a value for slot `name`. 
The slot mapping `from_llm` accepts the extracted value from the LLM.

 |
| 

Slot `name` has already been filled previously by the NLU mapping. 
The NLU pipeline extracts a new value from the latest user message that updates the slot. 
The LLM also extracts a value for slot `name`.

 | 

The slot mapping `from_entity` accepts the extracted value from the NLU pipeline as a correction, because the NLU based mapping takes higher priority. 
The LLM value is ignored.

 |
| 

Slot `name` has already been filled previously by the NLU mapping. 
The NLU pipeline does not extract any new value from the latest user message. 
The LLM extracts a value for slot `name`.

 | 

The LLM extracted value is ignored, because the LLM is not allowed to correct NLU-filled slots by default. 
If you want to allow the LLM to correct the NLU-filled slot, you can set the `allow_nlu_correction` [property](https://rasa.com/docs/reference/primitives/slots/#allow_nlu_correction) to `true` in the slot mapping.

 |
| 

Slot `name` has already been filled previously by the LLM. 
The NLU pipeline extracts a new value from the latest user message that updates the slot. 
The LLM also extracts a value for slot `name`.

 | 

The slot mapping `from_entity` accepts the extracted value from the NLU pipeline as a correction because the NLU based mapping takes higher priority. 
The LLM value is ignored.

 |
| 

Slot `name` has already been filled previously by the LLM. 
The NLU pipeline does not extract any new value from the latest user message. 
The LLM extracts a value for slot `name`.

 | 

The slot mapping `from_llm` accepts the extracted value from the LLM as a correction. 
The LLM extracted value is accepted as a correction because the LLM is allowed to correct LLM-filled slots.

 |

### Initial slot values[​](https://rasa.com/docs/reference/primitives/slots/#initial-slot-values 'Direct link to Initial slot values')

You can provide an initial value for any slot in your domain file:

```
slots:  num_fallbacks:    type: float    initial_value: 0
```

### Persistence of Slots during Coexistence[​](https://rasa.com/docs/reference/primitives/slots/#persistence-of-slots-during-coexistence 'Direct link to Persistence of Slots during Coexistence')

In [Coexistence of NLU-based and CALM systems](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/) the action [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing) resets all slots and hides events from featurization for the NLU-based system policies to prevent them from seeing events that originated while CALM was active. However, you might want to share some slots that both CALM and the NLU-based system should be able to use. One use case for these slots are basic user profile slots. Both the NLU-based system and CALM should likely be able to know whether a user is logged in or not, what their username is, or what channel they are using. If you are storing this kind of data in slots you can annotate those slot definitions with the option `shared_for_coexistence: True`.

```
version: "3.1"slots:  user_channel:    type: categorical    values:      - web      - teams    shared_for_coexistence: True  user_name:    type: text    shared_for_coexistence: True
```

In the coexistence mode, if the option `shared_for_coexistence` is NOT set to `true`, it invalidates the [`reset_after_flow_ends: False` property](https://rasa.com/docs/reference/primitives/flow-steps/#resetting-slots-at-the-end-of-a-flow) in the flow definition. In order for the slot value to be retained throughout the conversation, the `shared_for_coexistence` must be set to `true`.

### Real-Time Slot validation[​](https://rasa.com/docs/reference/primitives/slots/#real-time-slot-validation 'Direct link to Real-Time Slot validation')

New in 3.12

You can now define validation rules that are strictly independent of business logic directly in the domain file. These rules enforce constraints on slot values when they are collected during the conversation in real time.

You can now validate slot values in real-time as they are collected at any point during a conversation. This can be achieved by adding a `validation` key to the slot definition in the domain file. The `validation` key expects a mandatory `rejections` property and an optional `refill_utter` property.

The `rejections` section is a list of mappings. Each mapping must have `if` and `utter` mandatory properties:

- the `if` property is a condition written in natural language and evaluated using the [pypred](https://github.com/armon/pypred) library. In the condition, you can only use the currently defined slot name.
- the `utter` property is the name of the [response](https://rasa.com/docs/reference/primitives/responses/) the assistant will send if the condition evaluates to `True`.

The `refill_utter` key is optional and it defines the response the assistant will send to ask the user to refill the slot value if validation fails. If undefined, Rasa will look for a response called `utter_ask_{slot_name}` instead.

Here is an example:

```
slots:  phone_number:    type: text    mappings:      - type: from_llm    validation:      rejections:        - if: not (slots.phone_number matches "^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$")          utter: utter_invalid_phone_number      refill_utter: "utter_refill_phone_number" # defaults to utter_ask_phone_number
```

#### When to use real-time slot validation[​](https://rasa.com/docs/reference/primitives/slots/#when-to-use-real-time-slot-validation 'Direct link to When to use real-time slot validation')

The real-time slot validation feature serves as a mechanism to enforce specific constraints on slot values provided at any point during a conversation, regardless of which flow uses these slots. These validations act as universal rules that apply whenever and wherever these slots are used throughout the system.

Your assistant will not proceed with the conversation until the user provides a valid value for the slot, as per the defined constraints. This is particularly useful for ensuring data integrity and consistency across all interactions with the assistant.

Rather than being tied to any particular business logic, these constraints function as standalone checks that focus solely on ensuring the technical correctness of the collected data. By defining these validations at the slot level, you establish consistent data quality standards that automatically apply across all flows that utilize these slots.

note

For more business-logic specific validations, you can define slot validation in [flows](https://rasa.com/docs/reference/primitives/flow-steps/#slot-validation).

For more complex validation logic, you can also define slot validation in a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/). Note this custom action must follow this naming convention: `validate_{slot_name}`.

These will still continue to run as before only at the step where the validation is defined and not in real-time.

#### Allowed Validation Types[​](https://rasa.com/docs/reference/primitives/slots/#allowed-validation-types 'Direct link to Allowed Validation Types')

The following validation checks can be defined in the domain file using the [pypred](https://github.com/armon/pypred) library.

note

These validations are limited to the capabilities supported by the [pypred](https://github.com/armon/pypred) library.

- **Regex Matching**: Validate inputs against specific patterns (e.g email addresses, phone numbers, zip codes, registration numbers, etc.)
- **Length Validation**: Ensure input text meets minimum and maximum length requirements (e.g usernames, passwords, IDs)
- **Data Type Validation**: Ensure inputs conform to specific type categories (integers only, numerical values, alphanumeric strings)
- **Range Checks**: For numerical inputs, verify that values fall within a specified range (e.g 18-65 for age, 1-100 for quantity, minimum/maximum thresholds)
- **Date Format Validation**: Validate date inputs against specific formats and logical constraints (e.g YYYY-MM-DD, no future birth dates)
- **List or Enumeration Matching**: Check if inputs match predefined valid options (e.g colors, sizes, categories)
- **Prefix/Suffix Checks**: Verify inputs begin or end with required characters or strings (e.g product codes, reference numbers)
- **Case Sensitivity Checks**: Ensure inputs follow case requirements (e.g lowercase usernames, uppercase codes)
- **Whitespace Validation**: Check for improper spacing patterns in inputs (e.g unwanted leading, trailing, or excessive internal spaces)
- **Special Character Filtering**: Restrict or validate special characters to maintain data integrity and security

#### Custom validation for slots[​](https://rasa.com/docs/reference/primitives/slots/#custom-validation-for-slots 'Direct link to Custom validation for slots')

You can also add custom validation logic for validating slots in a custom action which must be defined as `validate_<slot_name>`.

By default Rasa will run this custom action every time the slot is being collected by the assistant.

This action is written with the help of `rasa_sdk` library. see [reference](https://rasa.com/docs/reference/integrations/action-server/validation-action/#how-to-implement-custom-validation-in-calm-assistants)

- [Slots](https://rasa.com/docs/reference/primitives/slots/#slots)
 - [Slot Types](https://rasa.com/docs/reference/primitives/slots/#slot-types)
 - [Resetting a slot](https://rasa.com/docs/reference/primitives/slots/#resetting-a-slot)
 - [CALM Slot Mappings](https://rasa.com/docs/reference/primitives/slots/#calm-slot-mappings)
 - [Initial slot values](https://rasa.com/docs/reference/primitives/slots/#initial-slot-values)
 - [Persistence of Slots during Coexistence](https://rasa.com/docs/reference/primitives/slots/#persistence-of-slots-during-coexistence)
 - [Real-Time Slot validation](https://rasa.com/docs/reference/primitives/slots/#real-time-slot-validation)