# Source: https://rasa.com/docs/reference/primitives/flow-steps

On this page

A flow is made up of one or more steps, each with a specific purpose. There are two supported types of flow steps:

1. [Prescriptive steps](https://rasa.com/docs/reference/primitives/flow-steps/#prescriptive-steps): Designed for well-defined, predictable business logic.
2. [Autonomous steps](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps): Allow for dynamic decision-making and flexible business logic at runtime by triggering a [sub agent](https://rasa.com/docs/reference/config/agents/overview-agents/).

Flows can freely combine both types of steps as needed.

## Shared Properties[​](https://rasa.com/docs/reference/primitives/flow-steps/#shared-properties 'Direct link to Shared Properties')

All steps share these common properties:

```
- id: "an_optional_unique_id"  next: "an id or conditions to determine which step should be executed next"
```

### Id Property[​](https://rasa.com/docs/reference/primitives/flow-steps/#id-property 'Direct link to Id Property')

The `id` field is an optional unique identifier for each step. It is used to reference the step in the `next` field of other steps.

### Next Property[​](https://rasa.com/docs/reference/primitives/flow-steps/#next-property 'Direct link to Next Property')

The `next` field specifies which step is executed after this one. If it is omitted, the following step in the list is the next one.

To link a specific step, you can use the next step's `id`:

```
- collect: name  next: the_next_step- id: the_next_step  collect: age
```

You can also create a nested structure by listing steps under the `next` field:

```
- collect: name  next:    - collect: age
```

This is sometimes easier to read, especially when using [conditions](https://rasa.com/docs/reference/primitives/conditions/) to create branching logic. For example:

```
- collect: age  next:    - if: slots.age < 18      then:        - action: utter_not_old_enough          next: END    - if: slots.age >= 18 and slots.age < 65      then: 18_to_65_step    - else: over_65_step
```

Depending on the user's input value for age, this example show the flow proceeding to different steps. Note the `age` slot must be prefixed by the `slots.` namespace to be accessible in the condition. For namespaces, navigate to the [Namespaces](https://rasa.com/docs/reference/primitives/conditions/#namespaces) section.

When a condition is met that leads to `next: END`, the flow will complete without executing further steps.

## Prescriptive Steps[​](https://rasa.com/docs/reference/primitives/flow-steps/#prescriptive-steps 'Direct link to Prescriptive Steps')

Every step in a flow has a type. The type determines what the step does and what properties it supports. A prescriptive step must have exactly one of they following keys:

- `action`
- `collect`
- `link`
- `call`
- `set_slots`
- `noop`

### Action[​](https://rasa.com/docs/reference/primitives/flow-steps/#action 'Direct link to Action')

A step with the key `action: action_do_something` instructs your assistant to execute `action_do_something` and then proceed to the next step. It will not wait for user input.

The value of the `action` key is either the name of a custom action:

```
- action: action_check_sufficient_funds
```

Or the name of a response defined in your domain:

```
- action: utter_insufficient_funds
```

### Collect[​](https://rasa.com/docs/reference/primitives/flow-steps/#collect 'Direct link to Collect')

A step with a `collect` key instructs your assistant to request information from the user to fill a slot. For example:

```
- collect: account_type # slot name
```

Your assistant will not proceed to the next step in the flow until the slot `account_type` has been filled. See the [tutorial](https://rasa.com/docs/pro/tutorial/#collecting-information-in-slots) for more information about slot filling and defining slots in your domain file.

Instead of a `response` you can also define a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/) called `action_ask_recipient` to phrase the question along with, for example, some buttons to the user. Make sure to add the custom action to your domain file.

domain.yml

```
actions:  - action_ask_recipient
```

info

You can define either a `response` or a `custom action` for your collect step. It is not possible to define both. A validation error will be thrown by Rasa if both are defined.

#### Slot Descriptions[​](https://rasa.com/docs/reference/primitives/flow-steps/#slot-descriptions 'Direct link to Slot Descriptions')

An optional `description` field can be added to the `collect` step to guide the language model in extracting slot values.

```
- collect: account_type  description: "Account type is one of checking, savings, money market, or IRA."
```

info

The description field can be used to perform slot validation:

```
- collect: zip_code  description: |    A US postal code, having either 5 or 9 digits.    For example, "60613" or "60613-1060".
```

#### Always Asking Questions[​](https://rasa.com/docs/reference/primitives/flow-steps/#always-asking-questions 'Direct link to Always Asking Questions')

By default, a `collect` step is skipped if the corresponding slot is already filled. If a `collect` step should always be asked, no matter if the underlying slot is already filled, you can set `ask_before_filling: true` on the `collect` step:

```
- collect: final_confirmation # slot name  ask_before_filling: true
```

If the "final\_confirmation" slot is already filled, the assistant will clear it and proceed to ask the `collect` step, then fill the slot again. This ensures the confirmation `collect` step is always asked.

#### Resetting slots at the End of a Flow[​](https://rasa.com/docs/reference/primitives/flow-steps/#resetting-slots-at-the-end-of-a-flow 'Direct link to Resetting slots at the End of a Flow')

By default, all slots filled via `collect` steps are reset when a flow completes. Once the flow ends, the slot's value is either reset to `null` or to the slot's [initial value](https://rasa.com/docs/reference/primitives/slots/#initial-slot-values) (if provided in the domain file under `slots`).

If you want to retain the value of a slot after the flow completes, set the `reset_after_flow_ends` property to `false`. For example:

```
- collect: user_name  reset_after_flow_ends: false
```

Deprecation Warning

Configuring `reset_after_flow_ends` in collect steps is deprecated in version `3.11.0` and will be removed in Rasa Pro `4.0.0`. Please use the [`persisted_slots`](https://rasa.com/docs/reference/primitives/flows/#persisted-slots) property at the flow level instead.

In [coexistence mode](https://rasa.com/docs/pro/calm-with-nlu/coexistence/), this behavior has to be enforced by annotating the slot definition with the property [`shared_for_coexistence: True`](https://rasa.com/docs/reference/primitives/slots/#persistence-of-slots-during-coexistence).

#### Suppressing interruptions[​](https://rasa.com/docs/reference/primitives/flow-steps/#suppressing-interruptions 'Direct link to Suppressing interruptions')

New in 3.12

You can suppress interruptions to a `collect` step by setting the `force_slot_filling` property to `true`.

By default, the assistant will process an interruption to the conversation at any time, this could be triggered by the user digressing or by the assistant incorrectly interpreting the user's message.

In some cases, you may want to suppress interruptions to a `collect` step. For example, when the slot being collected is a feedback or comments slot, and you want to ensure that the user provides their feedback before proceeding with the next step. This kind of user message could reference different topics which could incorrectly trigger other commands.

By setting `force_slot_filling: true`, the assistant will ignore any other commands and only process the `SetSlot` command for the slot being collected.

```
- collect: feedback  force_slot_filling: true
```

#### Using a different response key for the `collect` step[​](https://rasa.com/docs/reference/primitives/flow-steps/#using-a-different-response-key-for-the-collect-step 'Direct link to using-a-different-response-key-for-the-collect-step')

By default, Rasa will look for a response called `utter_ask_{slot_name}` to execute a `collect` step. You can use a [response](https://rasa.com/docs/reference/primitives/responses/) with a different key by adding an `utter` property to the step.

For example:

flows.yml

```
flows:  replace_supplementary_card:    description: This flow helps the user to replace a supplementary card for a family member.    steps:      - collect: account_type        utter: utter_ask_secondary_account_type        next: ask_account_number
```

#### Using an action to ask for information in `collect` step[​](https://rasa.com/docs/reference/primitives/flow-steps/#using-an-action-to-ask-for-information-in-collect-step 'Direct link to using-an-action-to-ask-for-information-in-collect-step')

Instead of using a templated response for the `collect` step you can also use a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/). This is useful if you, for example, want to display available values for the slot fetched from a database as buttons. Rasa will look for an action called `action_ask_{slot_name}` to execute a `collect` step. The custom action needs to strictly follow the specified naming convention. Make sure to add the custom action `action_ask_{slot_name}` to your domain file.

domain.yml

```
actions:  - action_ask_{slot_name}
```

You don't need to update the collect step itself to use the custom action. The collect step inside the flow stays the same, for example,

```
- collect: {slot_name}
```

info

You can define either a `response` or a `custom action` for your collect step. It is not allowed to define both. A validation error will be thrown by Rasa if both are defined.

#### Slot validation[​](https://rasa.com/docs/reference/primitives/flow-steps/#slot-validation 'Direct link to Slot validation')

You can define slot validation rules directly in your flow yaml file by adding a `rejections` property to any `collect` step. The `rejections` section is a list of mappings. Each mapping must have `if` and `utter` mandatory properties:

- the `if` property is a condition written in natural language and evaluated using the [pypred](https://github.com/armon/pypred) library. In the condition, you can only use the slot name that is collected in this step. At the moment, using other slots is not supported.
- the `utter` property is the name of the [response](https://rasa.com/docs/reference/primitives/responses/) the assistant will send if the condition evaluates to `True`.

Here is an example:

flows.yml

```
flows:  verify_eligibility:    description: This flow verifies if the user is eligible for creating an account.    steps:      - collect: age        rejections:          - if: slots.age < 1            utter: utter_invalid_age          - if: slots.age < 18            utter: utter_age_at_least_18        next: ask_email
```

When validation fails, your assistant will automatically try to collect the information again. The assistant will repeatedly ask for the slot until the value is not rejected.

If the predicate defined in the `if` property is invalid, the assistant will log an error and respond with the `utter_internal_error_rasa` response which by default is _Sorry, I'm having trouble understanding you right now. Please try again later._

You can override the text message for `utter_internal_error_rasa` by adding this response with the custom text in your domain file.

note

For more complex validation logic, you can also define slot validation in a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/). Note this custom action must follow this naming convention: `validate_{slot_name}`.

#### Handling user silence[​](https://rasa.com/docs/reference/primitives/flow-steps/#handling-user-silence 'Direct link to Handling user silence')

For voice-stream channels, you can configure how long the assistant waits for a user response before treating silence as input. This is useful when different questions require different amounts of time for the user to respond.

You can override the global silence timeout for a specific `collect` step by adding the `silence_timeout` property:

```
- collect: feedback  silence_timeout: 15  # wait 15 seconds before treating silence as input
```

For more granular control, you can set different timeouts for different channels:

```
- collect: feedback  silence_timeout:    twilio_media_streams: 15    genesys: 20
```

For channels not listed, the timeout configured in `credentials.yml` or the global default of 7 seconds will be used.

This allows you to fine-tune the timing for specific questions:

- Wait **longer** for complex or open-ended questions (e.g., "Can you describe your issue?")
- Use **shorter** timeouts for simple prompts (e.g., yes/no questions)

👉 [Learn more about silence timeout handling](https://rasa.com/docs/reference/config/overview/#silence-timeout-handling)

#### Requesting DTMF input[​](https://rasa.com/docs/reference/primitives/flow-steps/#requesting-dtmf-input 'Direct link to Requesting DTMF input')

For Voice Assistants, you might want to collect a slot by capturing [DTMF](https://en.wikipedia.org/wiki/DTMF_signaling) input (phone keypad presses) instead of or in addition to voice. This is useful for scenarios requiring high accuracy (such as entering account numbers or PINs) or for PCI DSS compliance reasons.

tip

When using DTMF input, it's recommended to also configure the [`silence_timeout`](https://rasa.com/docs/reference/primitives/flow-steps/#handling-user-silence) property to allow sufficient time for users to enter their input via the keypad.

You can configure DTMF input by adding a `dtmf` property to a `collect` step:

```
- collect: account_number  dtmf:    length: 6
```

##### DTMF Configuration Properties[​](https://rasa.com/docs/reference/primitives/flow-steps/#dtmf-configuration-properties 'Direct link to DTMF Configuration Properties')

The `dtmf` property accepts the following optional parameters:

**Fixed-length input:**

Use the `length` property to specify the exact number of digits expected. The input will automatically submit once the specified number of digits is entered.

```
- collect: account_number  dtmf:    length: 6  # auto-submit after 6 digits
```

**Variable-length input:**

Use the `finish_on_key` property to specify a termination key (such as `#` or `*`). The user must press this key to submit their input.

```
- collect: amount  dtmf:    finish_on_key: "#"  # wait for # key to submit
```

info

You cannot use both `length` and `finish_on_key` together. Define one or the other based on your use case.

**Controlling audio input:**

By default, audio input is allowed alongside DTMF input. To restrict input to DTMF only, set `allow_audio_input` to `false`:

```
- collect: pin  dtmf:    allow_audio_input: false    length: 4
```

When `allow_audio_input` is `false`, only DTMF input will be accepted for this step. Ensure your assistant's responses clearly indicate to users that voice input is ignored during this step.

### Link[​](https://rasa.com/docs/reference/primitives/flow-steps/#link 'Direct link to Link')

A `link` step is used to connect flows. Links can only be used as the last step in a flow. When a `link` step is reached, the current flow is ended and the targeted flow is started. You can define a `link` step as:

```
- link: "id_of_another_flow"
```

It is not possible to add any other property like an `action` or a `next` to a `link` step. Doing so, will result in an error during flow validation.

It is possible to link from a flow to the [Human Handoff](https://rasa.com/docs/learn/concepts/conversation-patterns/) conversation repair pattern. This enables catching a scenario where the conversation should be handed over to a human agent. It is not possible to link from a flow to any of the other [conversation repair pattern](https://rasa.com/docs/reference/primitives/patterns/#default-behavior)

### Call[​](https://rasa.com/docs/reference/primitives/flow-steps/#call 'Direct link to Call')

New in 3.8.0

The `call` step is available starting with version `3.8.0`.

#### Calling a Flow[​](https://rasa.com/docs/reference/primitives/flow-steps/#calling-a-flow 'Direct link to Calling a Flow')

You can use a `call` step inside a flow (parent flow) to embed another flow (child flow). When the execution reaches a `call` step, CALM starts the child flow. Once the child flow is complete, the execution continues with the parent flow.

Calling other flows helps split up and reuse functionality. It allows you to define a flow once and use it in multiple other flows. Long flows can be split into smaller ones, making them easier to understand and maintain.

You can define a `call` step as:

flows.yml

```
flows:  parent_flow:    description: "The parent flow calling a child flow."    steps:      - call: child_flow  child_flow:    description: "A child flow."    steps:      - action: utter_child_flow
```

The call step must reference an existing flow and the child flow can be defined in the same file as the one containing the parent flow or a different file.

##### Behavior of the `call` step[​](https://rasa.com/docs/reference/primitives/flow-steps/#behavior-of-the-call-step 'Direct link to behavior-of-the-call-step')

The `call` step is designed to behave as if the child flow is part of the parent flow. Hence, the following properties come out of the box:

1. If the child flow has a `next: END` in any step, the control will get passed back to the parent flow, and the next logical step will be executed.
2. Slots of the child flow can be filled upfront, even before the child flow is started. The `collect` steps in a child flow behave as if they were directly part of the parent flow.
3. Slots of the child flow can be corrected once they are filled and the control is still inside the child or the parent flow. So, even after the child flow is complete, the slots can be corrected as long as the parent flow is still active.
4. Slots of the parent flow will not be reset when the child flow is activated. The child flow can access and modify slots of the parent flow.
5. Slots of a child flow will not be reset when the control comes back to the parent flow. Instead, they will be reset together with the slots of the parent flow once that ends (unless `reset_after_flow_ends` is set to `False` for any of the slots).
6. If the `CancelFlow()` command is triggered inside the the child flow, the parent flow is also cancelled.

note

To prevent the child flow from getting triggered directly by a user message, you can add a [flow guard](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards) to the child flow:

flows.yml

```
flows:  parent_flow:    description: "A parent flow calling a child flow."    steps:      - call: child_flow  child_flow:    description: "A child flow with a flow guard."    if: False    steps:      - action: utter_child_flow
```

In the above example, the `child_flow` will never be triggered directly by a user message but only when the parent flow calls it.

##### Constraints on the `call` step[​](https://rasa.com/docs/reference/primitives/flow-steps/#constraints-on-the-call-step 'Direct link to constraints-on-the-call-step')

Calling other flows has the following constraints by design:

- A child flow can not use the `link` step because a `link` step is meant to always terminate the previous flow which would contradict with the behaviour of `call` step where the control should always be passed to the parent flow.
- Patterns can not use the call step.

Recommendation on using `link` v/s `call`

If the use case demands a flow to be initiated as a follow-up to another flow, then a `link` step is better suited to accomplish the connection between the two flows. However, if a flow needs to behave as if it was part of another larger flow and more steps need to be executed inside the larger flow after the child flow has completed, then `call` step should be used.

#### Calling an MCP Tool[​](https://rasa.com/docs/reference/primitives/flow-steps/#calling-an-mcp-tool 'Direct link to Calling an MCP Tool')

info

This feature is currently in **beta** and is available starting from **Rasa 3.14.0**.

A `call` step can also be used to invoke a tool from an MCP (Model Context Protocol) server directly within a flow. This allows you to integrate external tools without writing custom actions.

```
- call: tool_name  mcp_server: server_name  mapping:    input:    - param: parameter_name      slot: slot_name    output:    - slot: result_slot      value: result.structuredContent.value
```

The MCP server must be configured in your `endpoints.yml` file. See [MCP Server Integration](https://rasa.com/docs/reference/integrations/mcp-servers/) for setup details.

When using the `call` step with `mcp_server`, you must ensure that:

- The `mcp_server` value exactly matches the name of an MCP server defined in your `endpoints.yml`.
- The `call` value (tool name) refers to a tool that is available on that MCP server.

If `mcp_server` refers to a name not defined in your `endpoints.yml`, or if the tool does not exist on that server, the tool call will fail at runtime.

##### Mapping Configuration[​](https://rasa.com/docs/reference/primitives/flow-steps/#mapping-configuration 'Direct link to Mapping Configuration')

The `mapping` section defines how to pass data to the tool and how to store the results:

**Input Mapping**

The `input` section maps slot values to tool parameters. Each input mapping requires:

- `param`: The parameter name expected by the tool
- `slot`: The slot name containing the value to send

Here's a concrete example of input mapping for a flight booking tool:

```
mapping:  input:  - param: flight_id    slot: flight_number  - param: num_passengers    slot: passenger_count
```

**Output Mapping**

The `output` section maps tool results to slots. Each output mapping requires:

- `slot`: The slot name to store the result
- `value`: A Jinja2 expression that extracts the desired value from the tool result

Here's a concrete example of output mapping for the same flight booking tool:

```
mapping:  output:  - slot: booking_status    value: result.structuredContent.booking_status.success  - slot: booking_reference    value: result.structuredContent.booking_status.reference_number
```

The `value` field supports Jinja2 expressions, allowing you to:

- Access nested object properties using dot notation
- Apply filters and transformations
- Use conditional logic

##### Tool Result Formats[​](https://rasa.com/docs/reference/primitives/flow-steps/#tool-result-formats 'Direct link to Tool Result Formats')

MCP tools return results in two possible formats:

**Structured Content**

When tools provide an output schema, you get structured data as output:

```
{  "result": {    "structuredContent": {      "booking_status": {"success": true, "reference_number": "ABC123"}    }  }}
```

Access specific values by writing a valid Jinja2 expression (often using dot notation). For example:

```
output:  - slot: booking_status    value: result.structuredContent.booking_status.success  - slot: booking_reference    value: result.structuredContent.booking_status.reference_number
```

**Unstructured Content**

When no output schema is defined, the entire output is captured as a serialized string:

```
{  "result": {    "content": [      {        "type": "text",        "text": "{\"booking_status\": {\"success\": true, \"reference_number\": \"ABC123\"}}"      }    ]  }}
```

For unstructured content, capture the complete result in a slot and process the content later if needed:

```
output:- slot: booking_data  value: result.content
```

### Set Slots[​](https://rasa.com/docs/reference/primitives/flow-steps/#set-slots 'Direct link to Set Slots')

A `set_slots` step is used to set one or more slots. For example:

```
- set_slots:    - account_type: "savings"    - account_number: "123456789"
```

Normally, slots are either set from user input (using a `collect` step) or by fetching information from another system (using a custom action).

A `set_slot` is mostly useful to _unset_ a slot value. For example, if a user asks to transfer more money than they have available in their account, you might want to inform them. Just reset the `amount` slot rather than ending the flow.

flows.yml

```
flows:  transfer_money:    description: This flow lets users send money to friends and family.    steps:      - collect: recipient      - id: ask_amount        collect: amount        description: the number of US dollars to send      - action: action_check_sufficient_funds        next:          - if: not has_sufficient_funds            then:              - action: utter_insufficient_funds              - set_slots:                  - amount: null                next: ask_amount          - else: final_confirmation
```

### Noop[​](https://rasa.com/docs/reference/primitives/flow-steps/#noop 'Direct link to Noop')

A `noop` step can be combined with the `next` property to create a [conditional](https://rasa.com/docs/reference/primitives/conditions/#conditions) branch in a flow without necessarily running an action, uttering a response, collecting a slot or linking / calling another flow in that step itself.

For example:

```
flows:  change_address:    description: Allow a user to change their address.    steps:      - noop: true        next:          - if: not slots.authenticated            then:              - call: authenticate_user                next: ask_new_address          - else: ask_new_address      - id: ask_new_address        collect: address        ...
```

It's always necessary to add the `next` property to a `noop` step. Otherwise, validation of flows would fail during training.

## Autonomous Steps[​](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps 'Direct link to Autonomous Steps')

info

Autonomous Steps are currently in **beta** and are available starting from **Rasa 3.14.0**.

Autonomous steps allow for dynamic decision-making and flexible business logic at runtime by triggering a [sub agent](https://rasa.com/docs/reference/config/agents/overview-agents/). Unlike prescriptive steps that follow a predetermined path, autonomous steps delegate control to an AI sub agent that can make decisions, use tools, and interact with users independently.

A `call` step can invoke an autonomous sub agent that runs until completion or until specific exit conditions are met.

### Calling a Sub Agent[​](https://rasa.com/docs/reference/primitives/flow-steps/#calling-a-sub-agent 'Direct link to Calling a Sub Agent')

```
- call: agent_name
```

The agent name specified in the `call` step must match a name that has been configured as described in [the agent configuration](https://rasa.com/docs/reference/config/agents/overview-agents/#configuration). It must also be unique and must not clash with any flow name.

The sub agent runs autonomously. It can:

- Invoke different tools based on previous results
- Generate messages to request more information from users
- Signal completion when done

How a sub agent signals completion depends on the sub agent used. For more details please refer to the [sub agent documentation](https://rasa.com/docs/reference/config/agents/overview-agents/).

### Specifying Exit Conditions[​](https://rasa.com/docs/reference/primitives/flow-steps/#specifying-exit-conditions 'Direct link to Specifying Exit Conditions')

As an alternative to agent-controlled completion, you can specify explicit conditions that automatically end the sub agent as soon as they are met:

```
flows:  appointment_booking:    description: helps users book appointments    steps:      - call: booking_agent        exit_if:          - slots.appointment_time is not null      - collect: final_confirmation
```

This way of specifying exit conditions is only available for the [ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/). The sub agent automatically gets built-in `set_slot` tools for each slot mentioned in the exit conditions. This allows it to store values that will be used to evaluate when the task is complete. For more details please refer to the [ReAct Sub Agent](https://rasa.com/docs/reference/config/agents/react-sub-agents/) page.

- [Shared Properties](https://rasa.com/docs/reference/primitives/flow-steps/#shared-properties)
 - [Id Property](https://rasa.com/docs/reference/primitives/flow-steps/#id-property)
 - [Next Property](https://rasa.com/docs/reference/primitives/flow-steps/#next-property)
- [Prescriptive Steps](https://rasa.com/docs/reference/primitives/flow-steps/#prescriptive-steps)
 - [Action](https://rasa.com/docs/reference/primitives/flow-steps/#action)
 - [Collect](https://rasa.com/docs/reference/primitives/flow-steps/#collect)
 - [Link](https://rasa.com/docs/reference/primitives/flow-steps/#link)
 - [Call](https://rasa.com/docs/reference/primitives/flow-steps/#call)
 - [Set Slots](https://rasa.com/docs/reference/primitives/flow-steps/#set-slots)
 - [Noop](https://rasa.com/docs/reference/primitives/flow-steps/#noop)
- [Autonomous Steps](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps)
 - [Calling a Sub Agent](https://rasa.com/docs/reference/primitives/flow-steps/#calling-a-sub-agent)
 - [Specifying Exit Conditions](https://rasa.com/docs/reference/primitives/flow-steps/#specifying-exit-conditions)