# Source: https://rasa.com/docs/pro/build/assistant-memory

On this page

In a conversational experience, context matters just as much as the latest user message. Information derived either from the situational context, user profiles or user provided information—like a user’s age, an appointment date, or a selected product—often influences how the assistant behaves later on.

**Slots** provide this memory in CALM. They preserve data about the user or the external world so that your assistant can leverage it for business logic, personalization, or simply to maintain continuity. If a user provides their email address once, you don’t want them to repeat it in every subsequent interaction. Slots let you store that information and use it across the entire conversation.

Slots in CALM are key-value pairs that represent stateful information your assistant has collected or inferred. Each slot has:

- A **name** (e.g., `phone_number` or `user_name`)
- A **type** (e.g., `text`, `boolean`, `categorical`, etc.)
- (Optionally) a **default** or **initial** value

CALM primarily depends on an [LLM-based **Command Generator**](https://rasa.com/docs/reference/config/components/llm-command-generators/) to decide what to do next—**including** issuing commands that set slot values. However, you can still configure slot mappings to rely on an NLU pipeline if you choose to combine NLU-based extraction with CALM. In that case, the `NLUCommandAdapter` can issue `set slot` commands based on extracted entities or intents.

When building flows, you’ll often see [collect steps](https://rasa.com/docs/reference/primitives/flow-steps/#collect) that explicitly request slot values from users. But at any point in the conversation, the LLM can also set or update slots—if you permit it via the slot’s configuration.

## How to Write Slots[​](https://rasa.com/docs/pro/build/assistant-memory/#how-to-write-slots 'Direct link to How to Write Slots')

Slots are defined in your **domain** file under the `slots:` key. Each slot entry includes at least:

- The slot **name**
- The slot **type** (e.g., `text`, `bool`, `float`, `categorical`, `any`, etc.)
- (Optionally) `mappings` that specify how the slot should be filled

Below is a minimal example of a slot definition using LLM-based filling:

domain.yml

```
slots:  user_name:    type: text    mappings:      - type: from_llm
```

In this example:

- `user_name` is a `text` slot.
- It will be **filled by the LLM** at any point in the conversation, if the LLM issues the correct `set slot` command.

You can also combine an NLU pipeline (for classic intent/entity extraction) with CALM by giving the slot an NLU-based mapping as well, for example:

domain.yml

```
slots:  user_name:    type: text    mappings:      - type: from_entity        entity: person      - type: from_llm
```

In this case, the `NLUCommandAdapter` uses the recognized `person` entity to set the `user_name` slot. However, if no value is extracted for the `person` entity, then the LLM gets a chance to issue a value for it.

You can read more [here](https://rasa.com/docs/reference/primitives/slots/#impact-of-slot-mappings-in-different-scenarios) about how a combination of NLU-based extractors and LLMs can be used to efficiently extract slots.

### Defining Slots in Flows[​](https://rasa.com/docs/pro/build/assistant-memory/#defining-slots-in-flows 'Direct link to Defining Slots in Flows')

Within a flow, you often collect slot values using the `collect` step:

flows.yml

```
flows:  my_flow:    description: My flow    steps:    # ...    - collect: user_name      description: "The user’s full name"
```

This step instructs the LLM that you want to retrieve `user_name` from the user. If the user’s response is accepted, the LLM issues a `set slot` command, and `user_name` gets stored in the conversation state.

## How Does Slot Validation Work in CALM?[​](https://rasa.com/docs/pro/build/assistant-memory/#how-does-slot-validation-work-in-calm 'Direct link to How Does Slot Validation Work in CALM?')

Slot validation ensures that the values you store are meaningful or properly formatted for your use case. CALM offers three paths for validating slots:

1. **Global slot validation defined on a domain level**

Domain-level slot validation lives in the domain.yml file under each slot definition and it enforces universal, technical constraints (e.g., correct formatting, correct data type) whenever a slot is set or updated during the conversation, which ensures that if the user or the LLM tries to set a slot that doesn’t meet the condition, the assistant rejects it right away—before continuing the conversation. This behaviour is controlled by the `refill_utter` property.

Example:

domain.yml

```
 slots:  phone_number:    type: text    mappings:      - type: from_llm    validation:      rejections:        - if: not (slots.phone_number matches "^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$")          utter: utter_invalid_phone_number      refill_utter: "utter_refill_phone_number"
```

This domain-level validation is strictly for basic data correctness—things like format, range, or type checks. It’s not meant to handle more advanced, contextual logic (like checking if the phone number is already associated with an account in your database).

2. **Lightweight validations with rejections on a flow-level**
 
 In the flow’s `collect` step, you can define `rejections` that check the format or basic conditions on the extracted slot value. If the condition is not met, the assistant rejects the slot value and prompts the user again.
 
 flows.yml
 
 ```
    flows:  my_flow:    description: My flow    steps:    # ...    - collect: phone_number      description: "User's phone number in (xxx) xxx-xxxx format"      rejections:        - if: not ( slots.phone_number matches "^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$" )          utter: utter_invalid_phone_number
    ```
 
 This allows validation checks on the level of the collect step (e.g., contextual logic, matching a regex, ensuring a numeric range, etc.) without writing any custom code.
 
3. **Advanced validations with custom actions**
 
 If you need to call an external API or database to decide whether a slot value is valid, you can move the validation logic to a custom action:
 
 flows.yml
 
 ```
    flows:  my_flow:    description: My flow    steps:    # ...    - action: action_check_phone_number_has_account      next:        - if: slots.phone_number_has_account          then:            - action: utter_inform_phone_number_has_account            - set_slots:                - phone_number: null          next: "collect_phone_number"
    ```
 
 The [custom action](https://rasa.com/docs/pro/build/custom-actions/) can accept or reject the new slot value. Rejecting the value might reset the slot and re-ask for the user input, or redirect the user to a different flow.
 

For more detail on advanced slot mappings, slot types and slot validation, visit the [page on Slots](https://rasa.com/docs/reference/primitives/slots/) in the Reference.

- [How to Write Slots](https://rasa.com/docs/pro/build/assistant-memory/#how-to-write-slots)
 - [Defining Slots in Flows](https://rasa.com/docs/pro/build/assistant-memory/#defining-slots-in-flows)
- [How Does Slot Validation Work in CALM?](https://rasa.com/docs/pro/build/assistant-memory/#how-does-slot-validation-work-in-calm)