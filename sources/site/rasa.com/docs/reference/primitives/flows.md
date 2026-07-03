# Source: https://rasa.com/docs/reference/primitives/flows

On this page

New in 3.7

Flows are part of Rasa's new [Conversational AI with Language Models (CALM) approach](https://rasa.com/docs/learn/concepts/calm/) and available starting with version `3.7.0`.

## Overview[​](https://rasa.com/docs/reference/primitives/flows/#overview 'Direct link to Overview')

In [CALM](https://rasa.com/docs/learn/concepts/calm/), the business logic of your AI assistant is implemented as a set of flows. Each flow describes the logical steps your AI assistant uses to complete a task. It describes the information you need from the user, data you need to retrieve from an API or a database, and branching logic based on the information collected.

A flow in Rasa only describes the logic your assistant follows, not all the potential paths conversations can take. If you're used to designing AI assistants by creating flow charts of how conversations should go, you'll see that flows in Rasa are much simpler. Check out [Rasa Studio](https://rasa.com/docs/studio/build/flow-building/introduction/) to build flows with a web interface.

To get familiar with how flows work, follow the [tutorial](https://rasa.com/docs/pro/tutorial/). This page provides a reference of the format and properties of flows.

## Hello World[​](https://rasa.com/docs/reference/primitives/flows/#hello-world 'Direct link to Hello World')

A Flow is defined using YAML syntax. Here is an example of a simple flow:

flows.yml

```
flows:  hello_world:    description: A simple flow that greets the user    steps:      - action: utter_greet
```

## Flow Properties[​](https://rasa.com/docs/reference/primitives/flows/#flow-properties 'Direct link to Flow Properties')

A flow is defined using the following properties:

flows.yml

```
flows:  a_flow: # required id    name: "A flow" # optional name    description: "required description of what the flow does"    always_include_in_prompt: false # optional boolean, defaults to false    run_pattern_completed: true # optional boolean, defaults to true    if: "condition" # optional flow guard    nlu_trigger: # optional list of intents that can start a flow     - intent: "starting_flow_intent"    persisted_slots: [] # optional list of slots that should be persisted at the conversation level after the flow ends    steps: [] # required list of steps
```

### Flow ID[​](https://rasa.com/docs/reference/primitives/flows/#flow-id 'Direct link to Flow ID')

The id is required and uniquely identifies the flow among all flows. It allows only alphanumeric characters, underscores, and hyphens, with the restriction that the first character cannot be a hyphen.

### Name[​](https://rasa.com/docs/reference/primitives/flows/#name 'Direct link to Name')

The `name` field is an optional human readable name for the flow.

### Description[​](https://rasa.com/docs/reference/primitives/flows/#description 'Direct link to Description')

The `description` field is a summary of the flow. It is required, and should describe what the flow does for the user. Writing a clear description is important, because it is used by the [Dialogue Understanding](https://rasa.com/docs/learn/concepts/dialogue-understanding/) component to decide when to start this flow. See [Starting Flows](https://rasa.com/docs/reference/primitives/starting-flows/) for more details. Additionally, for guidelines on how to write concise and clear descriptions see the section provided [here](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-the-prompt).

### Always Include in Prompt[​](https://rasa.com/docs/reference/primitives/flows/#always-include-in-prompt 'Direct link to Always Include in Prompt')

If `always_include_in_prompt` field is set to `true` and the [flow guard](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards) defined in the `if` field evaluates to `true`, the flow will be [always be included in the prompt](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows)

### NLU Trigger property[​](https://rasa.com/docs/reference/primitives/flows/#nlu-trigger-property 'Direct link to NLU Trigger property')

The `nlu_trigger` field is used to add [intents that can start the flow](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger).

### Run Pattern Completed[​](https://rasa.com/docs/reference/primitives/flows/#run-pattern-completed 'Direct link to Run Pattern Completed')

The `run_pattern_completed` field is used to determine if the [`pattern_completed`](https://rasa.com/docs/reference/primitives/patterns/#default-behavior) flow should be run when the conversation is finished. This field is only applicable to the parent flows which are not called by other flows.

### If property[​](https://rasa.com/docs/reference/primitives/flows/#if-property 'Direct link to If property')

The `if` field is used to add [flow guards](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards).

### Persisted Slots[​](https://rasa.com/docs/reference/primitives/flows/#persisted-slots 'Direct link to Persisted Slots')

By default, slots set in a `collect` and `set_slot` step are reset when the flow ends. To change this behavior, you can add these slots to the `persisted_slots` field.

The `persisted_slots` field is used to specify a list of slots whose value should be persisted after the flow ends. These slots have to be filled in either a `collect` or `set_slots` step in the flow.

flows.yml

```
flows:  transfer_money:    description: This flow lets users send money to friends and family.    persisted_slots:      - recipient      - amount    steps:      - collect: recipient      - id: ask_amount        collect: amount        description: the number of US dollars to send      - action: action_check_sufficient_funds        next:          - if: not has_sufficient_funds            then:              - action: utter_insufficient_funds              - set_slots:                  - amount: null                next: ask_amount          - else: final_confirmation
```

#### Validation Rules for `persisted_slots` property[​](https://rasa.com/docs/reference/primitives/flows/#validation-rules-for-persisted_slots-property 'Direct link to validation-rules-for-persisted_slots-property')

During training, Rasa implements the following strict validation checks for the `persisted_slots` property:

- The `persisted_slots` property cannot be simultaneously used with the [`reset_after_flow_ends`](https://rasa.com/docs/reference/primitives/flow-steps/#resetting-slots-at-the-end-of-a-flow) property of the `collect` step in a flow.
 
 A validation error will be thrown by Rasa during training if both properties are used in a flow. Only one of the two properties can be used in a flow, not both.
 
- The `persisted_slots` property should only contain slots that are filled in either a `collect` or `set_slots` step in the flow.
 
 A validation error will be thrown by Rasa during training if a slot that is not set in a `collect` or `set_slots` step is added to the `persisted_slots` property.
 
 Please note, slots set in custom actions are automatically persistent by default and should not be added to the `persisted_slots` property. Attempting to do so will also result in a validation error.
 

### Steps[​](https://rasa.com/docs/reference/primitives/flows/#steps 'Direct link to Steps')

The `steps` field is required and lists the flow steps. For details please refer to [Flow Steps](https://rasa.com/docs/reference/primitives/flow-steps/).

## Examples[​](https://rasa.com/docs/reference/primitives/flows/#examples 'Direct link to Examples')

### A basic flow with branching[​](https://rasa.com/docs/reference/primitives/flows/#a-basic-flow-with-branching 'Direct link to A basic flow with branching')

```
flows:  basic_flow_with_branching:    description: a flow with a branch    steps:      - action: utter_greet      - collect: age        next:          - if: slots.age < 18            then: too_young_step          - else: old_enough_step      - id: too_young_step        action: utter_too_young      - id: old_enough_step        action: utter_old_enough
```

The example shows a flow that branches based on the `age` slot collected from a user message.

### Linking multiple flows[​](https://rasa.com/docs/reference/primitives/flows/#linking-multiple-flows 'Direct link to Linking multiple flows')

```
flows:  transfer_money:    description: Send money to another individual    steps:      - collect: recipient_name      - collect: amount_of_money      - action: execute_transfer      - action: utter_transfer_complete      - link: collect_feedback  collect_feedback:    description: Collect feedback from user on their experience talking to the assistant    steps:      - collect: ask_rating      - action: utter_thankyou
```

This example demonstrates how the `link` step in flows enables the start of another flow as a follow-up flow. Specifically, the flow `collect_feedback` is initiated as a follow up flow after `transfer_money` is terminated at the `link` step.

### Embedding a flow inside another flow[​](https://rasa.com/docs/reference/primitives/flows/#embedding-a-flow-inside-another-flow 'Direct link to Embedding a flow inside another flow')

Let's assume a financial assistant needs to serve two use cases:

1. Transferring money to another individual
2. Adding a new recipient for transferring money

It's possible that an end user talking to the assistant wants to initiate a money transfer to an existing recipient, hence not needing the second use case. However, it is also possible that the user wants to transfer money to a new recipient in which case both the use cases need to be combined. This is exactly where a `call` step lets you accomplish both the possibilities without needing to create redundant flows. To accomplish the above use cases, you can leverage the `call` step in your flows as shown below:

flows.yml

```
flows:  collect_recipient_details:    description: Details of a money transfer recipient should be collected here    if: False    steps:      - collect: recipient_name        description: Name of a recipient who should be sent money      - collect: recipient_iban        description: IBAN of a recipient who should be sent money.      - collect: recipient_phone_number        description: Phone number of a recipient who should be sent money.  add_recipient:    description: User wants to add a new recipient for transferring money    steps:      - call: collect_recipient_details      - action: add_new_recipient  transfer_money:    description: User wants to transfer money to a new or existing recipient    steps:      - action: show_existing_recipients      - collect: need_new_recipient        next:          - if: slots.need_new_recipient            then:              - call: add_recipient                next: get_confirmation          - else:              - call: collect_recipient_details                next: get_confirmation      - id: get_confirmation        collect: transfer_confirmation        ask_before_filling: true      - action: execute_transfer
```

For the above flow structure, the LLM's prompt would contain the following flow definition:

prompt.txt

```
add_recipient: User wants to add a new recipient to transfer money    slot: recipient_name (Name of a recipient who should be sent money)    slot: recipient_iban (IBAN of a recipient who should be sent money)    slot: recipient_phone_number (Phone number of a recipient who should be sent money)transfer_money: User wants to transfer money to a new or existing recipient    slot: need_new_recipient ((True/False))    slot: recipient_name (Name of a recipient who should be sent money)    slot: recipient_iban (IBAN of a recipient who should be sent money)    slot: recipient_phone_number (Phone number of a recipient who should be sent money)    slot: transfer_confirmation ((True/False))
```

Each of the above flows are quite self-contained, accomplishing a single purpose and reused effectively to accomplish combinations of multiple use cases in a single conversation. This enhances the modularity and maintainability of flows. Also, note that no slot of a child flow is overlapping with slot of its parent flow in terms of the information they capture. This is important for the command generator's LLM to not get confused at filling such slots.

### Optionally ask for information[​](https://rasa.com/docs/reference/primitives/flows/#optionally-ask-for-information 'Direct link to Optionally ask for information')

Imagine a dress shopping AI assistant created to help users find and purchase dresses. This AI assistant includes two types of features: primary features (i.e., dress type, size) and optional features (i.e., color, material, price range, etc.). The assistant should ask for the primary features and avoid asking for the optional features, unless the user specifically requests them. This approach is key to avoid asking too many questions and overwhelming the user.

You can achieve this by setting the `ask_before_filling` property to `false` on the `collect` step and setting an `initial_value` for the slot in the domain file.

```
flows:  purchase_dress:    name: purchase dress    description: present options to the user and purchase the dress    steps:      - collect: dress_type      - collect: dress_size      - collect: dress_color        ask_before_filling: false      - collect: dress_material        ask_before_filling: false        next:          - if: slots.dress_material = "cotton"            then:              - action: action_present_cotton_dresses                next: END          - if: slots.dress_material = "silk"            then:              - action: action_present_silk_dresses                next: END          - else:              - action: action_present_all_dresses                next: END
```

domain.yml

```
slots:  dress_type:    type: categorical    values:      - "shirt"      - "pants"      - "jacket"  dress_size:    type: categorical    values:      - "small"      - "medium"      - "large"  dress_color:    type: text    initial_value: "unspecified"  dress_material:    type: categorical    initial_value: "any"    values:      - "cotton"      - "silk"      - "polyester"      - "any"
```

With `ask_before_filling` set to `false` and given that the slots (`dress_color` and `dress_material`) already have initial values, the corresponding `collect` steps will not be asked. Instead, the assistant will move to the next step. If the user explicitly provides a value for a slot, this new value will overwrite the initial one.

- [Overview](https://rasa.com/docs/reference/primitives/flows/#overview)
- [Hello World](https://rasa.com/docs/reference/primitives/flows/#hello-world)
- [Flow Properties](https://rasa.com/docs/reference/primitives/flows/#flow-properties)
 - [Flow ID](https://rasa.com/docs/reference/primitives/flows/#flow-id)
 - [Name](https://rasa.com/docs/reference/primitives/flows/#name)
 - [Description](https://rasa.com/docs/reference/primitives/flows/#description)
 - [Always Include in Prompt](https://rasa.com/docs/reference/primitives/flows/#always-include-in-prompt)
 - [NLU Trigger property](https://rasa.com/docs/reference/primitives/flows/#nlu-trigger-property)
 - [Run Pattern Completed](https://rasa.com/docs/reference/primitives/flows/#run-pattern-completed)
 - [If property](https://rasa.com/docs/reference/primitives/flows/#if-property)
 - [Persisted Slots](https://rasa.com/docs/reference/primitives/flows/#persisted-slots)
 - [Steps](https://rasa.com/docs/reference/primitives/flows/#steps)
- [Examples](https://rasa.com/docs/reference/primitives/flows/#examples)
 - [A basic flow with branching](https://rasa.com/docs/reference/primitives/flows/#a-basic-flow-with-branching)
 - [Linking multiple flows](https://rasa.com/docs/reference/primitives/flows/#linking-multiple-flows)
 - [Embedding a flow inside another flow](https://rasa.com/docs/reference/primitives/flows/#embedding-a-flow-inside-another-flow)
 - [Optionally ask for information](https://rasa.com/docs/reference/primitives/flows/#optionally-ask-for-information)