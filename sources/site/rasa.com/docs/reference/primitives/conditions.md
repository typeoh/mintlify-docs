# Source: https://rasa.com/docs/reference/primitives/conditions

On this page

## Conditions[​](https://rasa.com/docs/reference/primitives/conditions/#conditions 'Direct link to Conditions')

Conditions are used in flows in three different places:

- In [flow guards](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards) to determine whether a flow can be started.
- In the [next](https://rasa.com/docs/reference/primitives/flows/#steps) field of a flow step to choose between branches of your business logic.
- In the `rejections` field of a `collect` step to [validate slot values](https://rasa.com/docs/reference/primitives/flow-steps/#slot-validation).

### Syntax[​](https://rasa.com/docs/reference/primitives/conditions/#syntax 'Direct link to Syntax')

Conditions in flows are written using natural language that can include logical operators, conditional operators and other constructs. They are evaluated with the [pypred](https://github.com/armon/pypred) library.

These conditions support the following operators,

- `not`: Negates a condition.
- `and`: Combines two conditions with logical AND.
- `or`: Combines two conditions with logical OR.
- `>`: Greater than.
- `>=`: Greater than or equal to.
- `<`: Less than.
- `<=`: Less than or equal to.
- `=`: Equal to.
- `!=`: Not equal to.
- `is`: Checks for identity.
- `is not`: Checks for non-identity.
- `contains`: Subset operator that checks if a set contains a value
- `matches`: Uses regular expressions to match strings.

#### Parentheses[​](https://rasa.com/docs/reference/primitives/conditions/#parentheses 'Direct link to Parentheses')

Use parentheses to group expressions and control the order of evaluation. For example:

```
- collect: age  next:    - if: slots.age < 18      then: under_18_step    - if: (slots.age > 18 and (consent = "yes" or consent = "y"))      then: consent_accept    - else: consent_decline
```

#### Subset Operator[​](https://rasa.com/docs/reference/primitives/conditions/#subset-operator 'Direct link to Subset Operator')

Subset operator `contains` can be used in the format `SET contains VALUE` where `SET` is the set of possible values and `VALUE` being the name of slot. For example:

```
- collect: emergency  next:    - if: "{'WARN' 'ERR' 'CRIT'} contains slots.error_level"      then: handoff    - else: everything_okay
```

`contains` operator can also be used to identify substrings when used as `slots.product contains "Rasa"` however this check is case-sensitive.

#### Constants[​](https://rasa.com/docs/reference/primitives/conditions/#constants 'Direct link to Constants')

- String literals: Enclose in single or double quotes (`'example'` or `"example"`).
- Numeric literals: Numbers without quotes (`42`).
- Constants: `true`, `false`, `undefined`, `null`, `empty`

#### Regular Expressions[​](https://rasa.com/docs/reference/primitives/conditions/#regular-expressions 'Direct link to Regular Expressions')

When using the `matches` operator, you can include regular expression modifiers. The regex modifier should be enclosed in quotes. For example, this flow step checks if the `zipcode` slot contains a United States zip code :

```
- collect: zipcode  description: ask zipcode and check if its a zip code  next:    - if: slots.zipcode matches "\d{5}(-\d{4})?"      then: ask_payment    - else: wrong_zipcode
```

A case-insensitive substring comparison can be made with the condition `product matches "(?i).*rasa.*"` which checks for the substring `rasa` within the variable `product`.

#### Empty Values[​](https://rasa.com/docs/reference/primitives/conditions/#empty-values 'Direct link to Empty Values')

If you want to check if a boolean slot is not set, you need to use the syntax `<boolean-slot> is null`.

To check if a text slot is not set or empty, use the syntax `not <text-slot>`.

#### Examples[​](https://rasa.com/docs/reference/primitives/conditions/#examples 'Direct link to Examples')

Here are some examples of conditions that demonstrate the use of different constructs.

```
# Simple conditionsage > 18name is "Alice"name is emptystatus = "active"status is not null# Combining Conditionsage > 21 and gender = "female"category = "electronics" or category = "computers"status = "active" and (priority = 1 or priority = 2)status = empty or status is nulldescription matches "/error \d{3}/i" and (severity = "high" or source contains "server")
```

## Namespaces[​](https://rasa.com/docs/reference/primitives/conditions/#namespaces 'Direct link to Namespaces')

Namespaces are used to access different types of data in predicates used in [branching conditions](https://rasa.com/docs/reference/primitives/flow-steps/#next-property), [slot validation](https://rasa.com/docs/reference/primitives/flow-steps/#slot-validation), and [flow guards](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards). There are two available namespaces: `slots` and `context`. The `slots` namespace is used to access slot values, while the `context` namespace is used to access the current dialogue frame properties.

### Slots[​](https://rasa.com/docs/reference/primitives/conditions/#slots 'Direct link to Slots')

The `slots` namespace is used to access slot values. The slot name must be prefixed with `slots.` to be accessible in the condition. For example:

```
- id: some_question  collect: age  next:    - if: slots.age < 18      then: under_18_step    - else: over_18_step
```

Make sure to have the slot defined in the [domain](https://rasa.com/docs/reference/config/domain/). If the slot is not defined in the domain or the slot is not prefixed with `slots.` namespace, the validation that runs during training will fail with an appropriate error.

### Context[​](https://rasa.com/docs/reference/primitives/conditions/#context 'Direct link to Context')

The `context` namespace is used to access the properties of the current [dialogue frame](https://rasa.com/docs/reference/primitives/conditions/#dialogue-frames). The property must be prefixed with `context.` to be accessible in the predicate. For example:

```
  pattern_completed:    description:  a flow has been completed and there is nothing else to be done    steps:      - noop: true        next:          - if: context.previous_flow_name != "greeting"            then:              - action: utter_what_can_help_with                next: END          - else: stop      - id: stop        action: action_stop
```

You can also use jinja templating to access the context namespace. For example:

```
  pattern_completed:    description:  a flow has been completed and there is nothing else to be done    steps:      - noop: true        next:          - if: "{{context.previous_flow_name}}" != "greeting"            then:              - action: utter_what_can_help_with                next: END          - else: stop      - id: stop        action: action_stop
```

#### Dialogue Frames[​](https://rasa.com/docs/reference/primitives/conditions/#dialogue-frames 'Direct link to Dialogue Frames')

The dialogue manager organizes the advancement of flows (both user defined and built-in) in a dialogue frame stack. The dialogue frame stack represents a LIFO (Last-In-First-Out) stack of dialogue frames. Different types of dialogue frames are mapped to built-in conversational patterns that enable [conversation repair](https://rasa.com/docs/learn/concepts/conversation-patterns/).

Each dialogue frame has a `flow_id` and `step_id` property. The `flow_id` is the id of the current flow and the `step_id` is the id of the current step in the flow.

The following dialogue frames types are available:

1. [cancel](https://rasa.com/docs/reference/primitives/conditions/#cancel): handles flow cancellation
2. [chitchat](https://rasa.com/docs/reference/primitives/conditions/#chitchat): handles chitchat
3. [clarify](https://rasa.com/docs/reference/primitives/conditions/#clarify): handles clarification
4. [collect information](https://rasa.com/docs/reference/primitives/conditions/#collect-information): handles information collection
5. [completion](https://rasa.com/docs/reference/primitives/conditions/#completion): handles flow completion
6. [continue interrupted](https://rasa.com/docs/reference/primitives/conditions/#continue-interrupted): handles continuation of interrupted flows
7. [correction](https://rasa.com/docs/reference/primitives/conditions/#correction): handles correction
8. [internal error](https://rasa.com/docs/reference/primitives/conditions/#internal-error): handles internal errors
9. [search](https://rasa.com/docs/reference/primitives/conditions/#knowledge-search): handles knowledge search
10. [skip question](https://rasa.com/docs/reference/primitives/conditions/#skip-question): handles skipping of information collection
11. [code change](https://rasa.com/docs/reference/primitives/conditions/#code-change): cleans the stack after an assistant update
12. [can not handle](https://rasa.com/docs/reference/primitives/conditions/#can-not-handle): handles situations where the assistant cannot handle
13. [human handoff](https://rasa.com/docs/reference/primitives/conditions/#human-handoff): handles handoff to human
14. [validate slot](https://rasa.com/docs/reference/primitives/conditions/#validate-slot): handles real-time slot validations that are strictly independent of business logic
15. [customer satisfaction](https://rasa.com/docs/reference/primitives/conditions/#customer-satisfaction): handles customer satisfaction feedback collection at the end of a conversation

##### Cancel[​](https://rasa.com/docs/reference/primitives/conditions/#cancel 'Direct link to Cancel')

The `flow_id` is `pattern_cancel_flow`. The `step_id` is `START`. In addition, the following properties are available:

- `canceled_name`: the name of the flow that should be canceled
- `canceled_frames`: the list of stack frames that should be canceled

##### Chitchat[​](https://rasa.com/docs/reference/primitives/conditions/#chitchat 'Direct link to Chitchat')

The `flow_id` is `pattern_chitchat`. The `step_id` is `START`.

##### Clarify[​](https://rasa.com/docs/reference/primitives/conditions/#clarify 'Direct link to Clarify')

The `flow_id` is `pattern_clarify`. The `step_id` is `START`. In addition, the following properties are available:

- `names`: the list of names of the flows that the user can choose from
- `clarification_options`: the options that the user can choose from as a string

##### Collect Information[​](https://rasa.com/docs/reference/primitives/conditions/#collect-information 'Direct link to Collect Information')

The `flow_id` is `pattern_collect_information`. The `step_id` is `START`. In addition, the following properties are available:

- `collect`: the name of the slot that should be filled
- `utter`: the response that should be executed to ask the user for information
- `collect_action`: the action that should be executed to ask the user for information
- `rejections`: the optional list of validation checks that should be executed if the user provides invalid information

Note that if you are using the `context.collect` property in your flow, you must write this in jinja templating style and prefix it with the `slots.` namespace. This is because `context.collect` corresponds to the slot name, and every slot must be preceded by the `slots.` namespace.

For example:

```
  pattern_collect_information:    name: "pattern_collect_information"    description:  the assistant is collecting information from the user    steps:      - id: check        next:          - if: "slots.{{context.collect}} is not null"            then:              - action: utter_ask_age                next: listen          - else: END      - id: listen        action: action_listen
```

##### Completion[​](https://rasa.com/docs/reference/primitives/conditions/#completion 'Direct link to Completion')

The `flow_id` is `pattern_completed`. The `step_id` is `START`. In addition, the following properties are available:

- `previous_flow_name`: the name of the flow that has been completed

##### Continue Interrupted[​](https://rasa.com/docs/reference/primitives/conditions/#continue-interrupted 'Direct link to Continue Interrupted')

The `flow_id` is `pattern_continue_interrupted`. The `step_id` is `START`. In addition, the following properties are available:

- `previous_flow_name`: the name of the flow that was interrupted
- `interrupted_flow_names`: names of the previous flows that were interrupted
- `interrupted_flow_ids`: ids of the previous flows that were interrupted
- `interrupted_flow_options`: options of interrupted flows that the user can choose from as a string
- `multiple_flows_interrupted`: boolean indicating whether multiple flows were interrupted

##### Correction[​](https://rasa.com/docs/reference/primitives/conditions/#correction 'Direct link to Correction')

The `flow_id` is `pattern_correction`. The `step_id` is `START`. In addition, the following properties are available:

- `is_reset_only`: indicates if the correction is only a reset of the flow
- `corrected_slots`: slot key-value pairs that should be corrected
- `reset_flow_id`: the ID of the flow to reset to, defaults to `None`
- `reset_step_id`: the ID of the step to reset to, defaults to `None`

##### Internal Error[​](https://rasa.com/docs/reference/primitives/conditions/#internal-error 'Direct link to Internal Error')

The `flow_id` is `pattern_internal_error`. The `step_id` is `START`. In addition, the following properties are available:

- `error_type`: string indicates the type of error
- `info`: additional info to be provided to the user

##### Knowledge Search[​](https://rasa.com/docs/reference/primitives/conditions/#knowledge-search 'Direct link to Knowledge Search')

The `flow_id` is `pattern_search`. The `step_id` is `START`.

##### Skip Question[​](https://rasa.com/docs/reference/primitives/conditions/#skip-question 'Direct link to Skip Question')

The `flow_id` is `pattern_skip_question`. The `step_id` is `START`.

##### Code Change[​](https://rasa.com/docs/reference/primitives/conditions/#code-change 'Direct link to Code Change')

The `flow_id` is `pattern_code_change`. The `step_id` is `START`.

##### Can Not Handle[​](https://rasa.com/docs/reference/primitives/conditions/#can-not-handle 'Direct link to Can Not Handle')

The `flow_id` is `pattern_cannot_handle`. The `step_id` is `START`. In addition, the following properties are available:

- `reason`: string indicates the reason why the assistant cannot handle

##### Human Handoff[​](https://rasa.com/docs/reference/primitives/conditions/#human-handoff 'Direct link to Human Handoff')

The `flow_id` is `pattern_human_handoff`. The `step_id` is `START`.

##### Validate Slot[​](https://rasa.com/docs/reference/primitives/conditions/#validate-slot 'Direct link to Validate Slot')

New in 3.12

You can now enable [real-time slot validation](https://rasa.com/docs/reference/primitives/slots/#real-time-slot-validation) in Rasa Pro 3.12.0.

The `flow_id` is `pattern_validate_slot`. The `step_id` is `START`. In addition, the following properties are available:

- `validate`: the name of the slot that should be validated
- `refill_utter`: the response that should be executed to ask the user to refill the invalid slot
- `refill_action`: the action that should be executed to ask the user to refill the invalid slot
- `rejections`: the list of validation checks that should be executed if the user provides information for a slot that requires validation

Note that if you are using the `context.validate` property in your flow, you must write this in jinja templating style and prefix it with the `slots.` namespace. This is because `context.validate` corresponds to the slot name, and every slot must be preceded by the `slots.` namespace.

For example:

```
  pattern_validate_slot:    name: "pattern_validate_slot"    description:  the assistant is validating information received from the user    steps:      - id: check        next:          - if: "slots.{{context.validate}} is not null"            then:              - action: utter_refill_age                next: listen          - else: END      - id: listen        action: action_listen
```

##### Customer Satisfaction[​](https://rasa.com/docs/reference/primitives/conditions/#customer-satisfaction 'Direct link to Customer Satisfaction')

New in 3.16

You can now enable customer satisfaction feedback collection in Rasa Pro 3.16.0.

The `flow_id` is `pattern_customer_satisfaction`. This pattern is triggered from `pattern_completed` via a `link` step when the user opts to end the conversation.

- [Conditions](https://rasa.com/docs/reference/primitives/conditions/#conditions)
 - [Syntax](https://rasa.com/docs/reference/primitives/conditions/#syntax)
- [Namespaces](https://rasa.com/docs/reference/primitives/conditions/#namespaces)
 - [Slots](https://rasa.com/docs/reference/primitives/conditions/#slots)
 - [Context](https://rasa.com/docs/reference/primitives/conditions/#context)