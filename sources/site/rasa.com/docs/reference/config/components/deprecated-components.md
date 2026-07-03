# Source: https://rasa.com/docs/reference/config/components/deprecated-components

On this page

## SingleStepLLMCommandGenerator[​](https://rasa.com/docs/reference/config/components/deprecated-components/#singlestepllmcommandgenerator 'Direct link to SingleStepLLMCommandGenerator')

Warning

The `SingleStepLLMCommandGenerator` is deprecated and no longer recommended for use in Rasa 3.x. It will be removed in Rasa `4.0.0`.

To interpret the user's message in context, the current implementation of the `SingleStepLLMCommandGenerator` uses _in-context learning_, information about the current state of the conversation, and flows defined in your assistant. Descriptions and slot definitions of each flow are included in the prompt as relevant information. However, to scale to a large number of flows, the LLM-based command generator includes only the flows that are relevant to the current state of the conversation, see [flow retrieval](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows).

#### Prompt Template[​](https://rasa.com/docs/reference/config/components/deprecated-components/#prompt-template 'Direct link to Prompt Template')

The default prompt template serves as a dynamic framework enabling the `SingleStepLLMCommandGenerator` to render prompts. The template consists of a **static component**, as well as **dynamic components** that get filled in when rendering a prompt:

- Current state of the conversation - This part of the template captures the ongoing dialogue.
- Defined flows and slots - This part of the template provides the context and structure for the conversation. It outlines the overarching theme, guiding the model's understanding of the conversation's purpose.
- Active flow and slot - Active elements within the conversation that require the model's attention.

The prompt template for the `SingleStepLLMCommandGenerator` is as follows:

```
Your task is to analyze the current conversation context and generate a list of actions to start new business processes that we call flows, to extract slots, or respond to small talk and knowledge requests.These are the flows that can be started, with their description and slots:{% for flow in available_flows %}{{ flow.name }}: {{ flow.description }}    {% for slot in flow.slots -%}    slot: {{ slot.name }}{% if slot.description %} ({{ slot.description }}){% endif %}{% if slot.allowed_values %}, allowed values: {{ slot.allowed_values }}{% endif %}    {% endfor %}{%- endfor %}===Here is what happened previously in the conversation:{{ current_conversation }}==={% if current_flow != None %}You are currently in the flow "{{ current_flow }}".You have just asked the user for the slot "{{ current_slot }}"{% if current_slot_description %} ({{ current_slot_description }}){% endif %}.{% if flow_slots|length > 0 %}Here are the slots of the currently active flow:{% for slot in flow_slots -%}- name: {{ slot.name }}, value: {{ slot.value }}, type: {{ slot.type }}, description: {{ slot.description}}{% if slot.allowed_values %}, allowed values: {{ slot.allowed_values }}{% endif %}{% endfor %}{% endif %}{% else %}You are currently not in any flow and so there are no active slots.This means you can only set a slot if you first start a flow that requires that slot.{% endif %}If you start a flow, first start the flow and then optionally fill that flow's slots with information the user provided in their message.The user just said """{{ user_message }}""".===Based on this information generate a list of actions you want to take. Your job is to start flows and to fill slots where appropriate. Any logic of what happens afterwards is handled by the flow engine. These are your available actions:* Slot setting, described by "SetSlot(slot_name, slot_value)". An example would be "SetSlot(recipient, Freddy)"* Starting another flow, described by "StartFlow(flow_name)". An example would be "StartFlow(transfer_money)"* Cancelling the current flow, described by "CancelFlow()"* Clarifying which flow should be started. An example would be Clarify(list_contacts, add_contact, remove_contact) if the user just wrote "contacts" and there are multiple potential candidates. It also works with a single flow name to confirm you understood correctly, as in Clarify(transfer_money).* Intercepting and handle user messages with the intent to bypass the current step in the flow, described by "SkipQuestion()". Examples of user skip phrases are: "Go to the next question", "Ask me something else".* Responding to knowledge-oriented user messages, described by "SearchAndReply()"* Responding to a casual, non-task-oriented user message, described by "ChitChat()".* Handing off to a human, in case the user seems frustrated or explicitly asks to speak to one, described by "HumanHandoff()".{% if is_repeat_command_enabled %}* Repeat the last bot messages, described by "RepeatLastBotMessages()". This is useful when the user asks to repeat the last bot messages.{% endif %}===Write out the actions you want to take, one per line, in the order they should take place.Do not fill slots with abstract values or placeholders.Only use information provided by the user.Only start a flow if it's completely clear what the user wants. Imagine you were a person reading this message. If it's not 100% clear, clarify the next step.Don't be overly confident. Take a conservative approach and clarify before proceeding.If the user asks for two things which seem contradictory, clarify before starting a flow.If it's not clear whether the user wants to skip the step or to cancel the flow, cancel the flow.Strictly adhere to the provided action types listed above.Focus on the last message and take it one step at a time.Use the previous conversation steps only to aid understanding.Your action list:
```

#### Customization[​](https://rasa.com/docs/reference/config/components/deprecated-components/#customization 'Direct link to Customization')

You can customize the `SingleStepLLMCommandGenerator` as much as you wish. General customization options that are available for both `LLMCommandGenerators` are listed in the section [General Customizations](https://rasa.com/docs/reference/config/components/llm-command-generators/#general-customization).

##### Customizing the Prompt Template[​](https://rasa.com/docs/reference/config/components/deprecated-components/#customizing-the-prompt-template 'Direct link to Customizing the Prompt Template')

If you cannot get something to work via editing the flow and slot descriptions (see section [customizing the prompt](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-the-prompt)), you can go one level deeper and customise the prompt template used to drive the `SingleStepLLMCommandGenerator`. To do this, write your own prompt as a jinja2 template and provide it to the component as a file:

config.yml

```
pipeline:  - name: SingleStepLLMCommandGenerator    prompt_template: prompts/command-generator.jinja2
```

Deprecation Warning

The former `LLMCommandGenerator`'s `prompt` configuration is replaced by `SingleStepLLMCommandGenerator`'s `prompt_template` in version `3.9.0`. The `prompt` configuration variable is now deprecated and will be removed in version `4.0.0`.

The prompt template also allows the utilization of variables to incorporate dynamic information. You can access the comprehensive list of available variables here to use in your custom prompt template.

| Variable | Type | Description |
| --- | --- | --- |
| `available_flows` | List\[Dict\[str, Any\]\] | A list of all flows available in the assistant. |
| `current_conversation` | str | A readable representation of the current conversation. 
 
_a simple example:_ 
```
 USER: hello\nAI: hi\nUSER: What is my transfer limit?
```

 |
| `current_flow` | str | ID of the current active flow. 
 
_example:_ 

```
 check_transfer_limit 
```

 |
| `current_slot` | str | Name of the currently asked slot. 
 
_example:_ 

```
 transfer_limit_type 
```

 |
| `current_slot_description` | str | Description of the currently asked collect step. 
 
_example:_ 

```
 Daily or monthly transfer limit. 
```

 |
| `current_slot_type` | str | Type of slot currently asked in the collect step. 
 
_example:_ 

```
 categorical 
```

 |
| `current_slot_allowed_values` | str | Allowed values of the slot in the currently asked collect step. 
 
_example:_ 

```
 ["daily", "monthly"] 
```

 |
| `flow_slots` | List\[Dict\[str, Any\]\] | A list of slots from the current active flow. |
| `user_message` | str | The latest message sent by the user. 
 
_example:_ 

```
 What is my transfer limit? 
```

 |

- Iterating over the `flow_slots` variable can be useful to create a prompt that lists all the slots of the current active flow,

```
{% for slot in flow_slots -%}- name: {{ slot.name }}, value: {{ slot.value }}, type: {{ slot.type }}, description: {{ slot.description}}{% if slot.allowed_values %}, allowed values: {{ slot.allowed_values }}{% endif %}{% endfor %}
```

| Variable | Type | Description |
| --- | --- | --- |
| `slot.name` | str | Name of the slot. 
 
_example:_ 
```
 transfer_money_has_sufficient_funds 
```

 |
| `slot.description` | str | Description of the slot. 
 
_example:_ 

```
 Checks if there is sufficient balance 
```

 |
| `slot.value` | str | Value of the slot. 
 
_example:_ 

```
 True 
```

 |
| `slot.type` | str | Type of the slot. 
 
_example:_ 

```
 bool 
```

 |
| `slot.allowed_values` | List\[str\] | List of allowed values for the slot. 
 
_example:_ 

 `[True, False]` 

 |

- Iterating over the `available_flows` variable can be useful to create a prompt that lists all the flows,

```
{% for flow in available_flows %}{{ flow.name }}: {{ flow.description }}    {% for slot in flow.slots -%}    slot: {{ slot.name }}{% if slot.description %} ({{ slot.description }}){% endif %}{% if slot.allowed_values %}, allowed values: {{ slot.allowed_values }}{% endif %}    {% endfor %}{%- endfor %}
```

| Variable | Type | Description |
| --- | --- | --- |
| `flow.name` | str | Name of the flow. 
 
_example:_ 
```
 transfer_money 
```

 |
| `flow.description` | str | Description of the flow. 
 
_example:_ 

```
 This flow lets users send money. 
```

 |
| `flow.slots` | List\[Dict\[str, Any\]\] | A list of slots from the flow. |

## MultiStepLLMCommandGenerator[​](https://rasa.com/docs/reference/config/components/deprecated-components/#multistepllmcommandgenerator 'Direct link to MultiStepLLMCommandGenerator')

Warning

The `MultiStepLLMCommandGenerator` is deprecated and no longer recommended for use in Rasa 3.x. It will be removed in Rasa `4.0.0`.

The `MultiStepLLMCommandGenerator` also uses _in-context learning_ to interpret the user's message in context, but breaks down the task into several steps to make the job of the LLM easier. The component was designed to enable cheaper and smaller LLMs, such as `gpt-3.5-turbo`, as viable alternatives to costlier but more powerful models such as `gpt-4-0613`.

The steps are:

- handling the flows (starting, ending, etc.) and
- filling out the slots

Accordingly, instead of just a single prompt that handles everything, the `MultiStepLLMCommandGenerator` has two prompts:

- `handle_flows` and
- `fill_slots`.

The following diagram shows which prompt is used when:

If no flow is currently active, the `handle_flows` prompt is used to start or clarify flows. If a flow is started, next the `fill_slots` prompt is executed to fill any slots of the newly started flow.

If a flow is currently active, the `fill_slots` prompt is called to fill any new slots of the currently active flow. If the user message (also) indicates that, for example, a new flow should be started or the active flow should be canceled, a `ChangeFlow` command is triggered. This results in calling the `handle_flows` prompt to start, cancel, or clarify flows. If that prompt leads to starting a flow, the `fill_slots` prompt is executed again to fill any slots of that new flow.

#### Prompt Templates[​](https://rasa.com/docs/reference/config/components/deprecated-components/#prompt-templates 'Direct link to Prompt Templates')

The default prompt templates serve as a dynamic framework enabling the `MultiStepLLMCommandGenerator` to render prompts. The templates consists of a **static component**, as well as **dynamic components** that get filled in when rendering a prompt.

- handle\_flows
- fill\_slots

- **Current state of the conversation**: This part of the template captures the ongoing dialogue.
- **Defined flows**: This part of the template provides the context and structure for the conversation. It outlines the overarching theme, guiding the model's understanding of the conversation's purpose.
- **Active flow**: Displays the name of the current active flow (if any). Used within conditional statements to customize the message about the flow's status.

- **Current state of the conversation**: This part of the template captures the ongoing dialogue.
- **Defined flows and slots**: This part of the template provides the context and structure for the conversation. It outlines the overarching theme, guiding the model's understanding of the conversation's purpose.
- **Active flow and slot**: Active elements within the conversation that require the model's attention.
- **Extract slots for a business process**: This part of the template focuses on identifying and filling in the necessary slots within the conversation.

#### Customization[​](https://rasa.com/docs/reference/config/components/deprecated-components/#customization-1 'Direct link to Customization')

You can customize the `MultiStepLLMCommandGenerator` as much as you wish. General customization options that are available for both `LLMCommandGenerators` are listed in the section [General Customizations](https://rasa.com/docs/reference/config/components/llm-command-generators/#general-customization).

##### Customizing The Prompt Template[​](https://rasa.com/docs/reference/config/components/deprecated-components/#customizing-the-prompt-template-1 'Direct link to Customizing The Prompt Template')

If you cannot get something to work via editing your flow and slot descriptions (see section [customizing the prompt](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-the-prompt)), you can go one level deeper and customise the prompt templates used to drive the `MultiStepLLMCommandGenerator`. To do this, write your own prompt as a jinja2 template and provide it to the component as a file:

config.yml

```
pipeline:  - name: MultiStepLLMCommandGenerator    prompt_templates:      handle_flows:        file_path: prompts/handle_flows_template.jinja2      fill_slots:        file_path: prompts/fill_slots_template.jinja2
```

You can customize both prompts using the example configuration above, or you can choose to customize only a specific prompt.

The prompt template also allows the utilization of variables to incorporate dynamic information.

##### handle\_flows[​](https://rasa.com/docs/reference/config/components/deprecated-components/#handle_flows 'Direct link to handle_flows')

Here is a comprehensive list of available variables to use in your custom `handle_flows` prompt template:

| Variable | Type | Description |
| --- | --- | --- |
| `available_flows` | List\[Dict\[str, Any\]\] | A list of all flows available in the assistant. |
| `current_conversation` | str | A readable representation of the current conversation. 
 
_a simple example:_ 
```
 USER: hello\nAI: hi\nUSER: I need to send money
```

 |
| `current_flow` | str | ID of the current active flow. 
 
_example:_ 

```
 transfer_money 
```

 |
| `last_user_message` | str | The latest message sent by the user. 
 
_example:_ 

```
 I want to transfer money 
```

 |

- Iterating over the `available_flows` variable can be useful to create a prompt that lists all the flows,

```
{% for flow in available_flows %}- {{ flow.name }}: {{ flow.description }}{%- endfor %}
```

| Variable | Type | Description |
| --- | --- | --- |
| `flow.name` | str | Name of the flow. 
 
_example:_ 
```
 transfer_money 
```

 |
| `flow.description` | str | Description of the flow. 
 
_example:_ 

```
 This flow lets users send money. 
```

 |

By default following commands can be predicted in this step: `StartFlow`, `Clarify`, `CancelFlow`, `CannotHandle`.

##### fill\_slots[​](https://rasa.com/docs/reference/config/components/deprecated-components/#fill_slots 'Direct link to fill_slots')

Here is a comprehensive list of available variables to use in your custom `fill_slots` prompt template:

| Variable | Type | Description |
| --- | --- | --- |
| `available_flows` | List\[Dict\[str, Any\]\] | A list of all flows available in the assistant. |
| `current_conversation` | str | A readable representation of the current conversation. 
 
_a simple example:_ 
```
 USER: hello\nAI: hi\nUSER: I need to send money
```

 |
| `current_flow` | str | ID of the current active flow. 
 
_example:_ 

```
 transfer_money 
```

 |
| `current_slot` | str | Name of the currently asked slot. 
 
_example:_ 

```
 transfer_money_recipient 
```

 |
| `current_slot_description` | str | Description of the currently asked collect step. 
 
_example:_ 

```
 the name of the person 
```

 |
| `current_slot_type` | str | The type of the current slot. 
 
_example:_ 

```
 the name of the person 
```

 |
| `current_slot_allowed_values` | str | The allowed values or options for the slot currently being asked for within the active flow. 
 
_example of allowed values for a slot:_ 

```
 Size: [Small, Medium, Large] 
```

 |
| `flow_slots` | List\[Dict\[str, Any\]\] | A list of slots from the current active flow. |
| `last_user_message` | str | The most recent message sent by the user. 
 
_example:_ 

```
 I want to transfer money 
```

 |
| `top_flow_is_pattern` | bool | Indicates whether the current active flow is a pattern. |
| `top_user_flow` | Flow | The top user flow object in the internal dialog stack. |
| `top_user_flow_slots` | List\[Dict\[str, Any\]\] | The slots associated with the top user flow. |
| `flow_active` | bool | Indicates whether a flow is currently active. |

- Iterating over the `flow_slots` or the `top_user_flow_slots` variable can be useful to create a prompt that lists all the slots of the current active flow (can be a pattern) or the top user flow,

```
{% for slot in flow_slots -%}- name: {{ slot.name }}, value: {{ slot.value }}, type: {{ slot.type }}, description: {{ slot.description}}{% if slot.allowed_values %}, allowed values: {{ slot.allowed_values }}{% endif %}{% endfor %}
```

| Variable | Type | Description |
| --- | --- | --- |
| `slot.name` | str | Name of the slot. 
 
_example:_ 
```
 transfer_money_has_sufficient_funds 
```

 |
| `slot.description` | str | Description of the slot. 
 
_example:_ 

```
 Checks if there is sufficient balance 
```

 |
| `slot.value` | str | Value of the slot. 
 
_example:_ 

```
 True 
```

 |
| `slot.type` | str | Type of the slot. 
 
_example:_ 

```
 bool 
```

 |
| `slot.allowed_values` | List\[str\] | List of allowed values for the slot. 
 
_example:_ 

 `[True, False]` 

 |

- Iterating over the `available_flows` variable can be useful to create a prompt that lists all the flows,

```
{% for flow in available_flows %}- {{ flow.name }}: {{ flow.description }}{%- endfor %}
```

| Variable | Type | Description |
| --- | --- | --- |
| `flow.name` | str | Name of the flow. 
 
_example:_ 
```
 transfer_money 
```

 |
| `flow.description` | str | Description of the flow. 
 
_example:_ 

```
 This flow lets users send money. 
```

 |

By default only `SetSlot` commands can be predicted in this step.

##### Prompt Tuning[​](https://rasa.com/docs/reference/config/components/deprecated-components/#prompt-tuning 'Direct link to Prompt Tuning')

Apart from the prompt customization proposed in the section on [Customizing The Prompt](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-the-prompt), the `MultiStepLLMCommandGenerator` might benefit from some additional prompt tuning.

##### Few-shot learning[​](https://rasa.com/docs/reference/config/components/deprecated-components/#few-shot-learning 'Direct link to Few-shot learning')

Few-shot learning is a machine learning approach in which an AI model learns to make accurate predictions by being trained on a very small number of labeled examples. Our internal experiments showed that adding some examples of user message to command pairs to the prompt template `handle_flows` helped to improve the performance of the LLM, especially for the `Clarify` command.

To do so, curate a small list of user message - action list pairs of examples that are specific to the domain of your assistant. Next, you can add them to the prompt template `handle_flows` after line 22. Here is an example:

```
===Here are some examples of user messages -> action list:1. "book" -> Clarify(book_hotel, book_restaurant, book_flight)2. "I want to book a hotel in Paris." -> StartFlow(book_hotel)3. "Can you help me with my booking?" -> Clarify(book_hotel, book_restaurant, book_flight)4. "Nevermind, stop it." -> CancelFlow()
```

##### Language of the prompt[​](https://rasa.com/docs/reference/config/components/deprecated-components/#language-of-the-prompt 'Direct link to Language of the prompt')

In our internal experiments, we have found that smaller LLMs benefit from having the complete prompt in a single language. Hence, if your assistant is built to understand and respond to a user in a language other than English and the flows descriptions as well as the collect step descriptions are in that same language, the LLM might benefit from translating the prompt template to that language as well. For example, assume users are talking in German to your assistant and the flow descriptions and collect step descriptions are also in German. To simplify the job of the LLM it might help to translate the complete prompt template to German as well.

You can use strong LLMs, such as `gpt-4-0613`, for translating. However, we recommend to manually check the prompt templates after translating it automatically.

##### Formatting[​](https://rasa.com/docs/reference/config/components/deprecated-components/#formatting 'Direct link to Formatting')

In both prompt templates, `handle_flows` and `fill_slots`, we iterate over the flows and/or the slots. Depending on the descriptions you wrote for those, it might make sense to update the for loop in the prompt templates.

For example, if your slot descriptions are rather long and contain a list of bullet points, the slot list in the final prompt might look like this:

```
The current user message started the process "transfer_money".Here are its slots:- recipient (text): - name of the person to send money to- should be a proper name- amount (float): - the amount of money to send- do not extract currency values- amount should be a float
```

As you see it is quite hard to distinguish between the description and the actual slots.

To help the LLM to clearly distinguish between the slot and the description, we recommend to update the for loop in the jinja2 prompt template as follows:

```
{% for slot in flow_slots %}{{loop.index}}. {{ slot.name }} ({{slot.type}}){% if slot.description %}: {{ slot.description}} {% endif %}{% if slot.allowed_values %}(allowed values: {{ slot.allowed_values }}){% endif %}{%- endfor %}
```

This results in the following:

```
The current user message started the process "transfer_money".Here are its slots:1. recipient (text): - name of the person to send money to- should be a proper name2. amount (float): - the amount of money to send- do not extract currency values- amount should be a float
```

#### Current Limitations[​](https://rasa.com/docs/reference/config/components/deprecated-components/#current-limitations 'Direct link to Current Limitations')

The `MultiStepLLMCommandGenerator` will be able to generate the following commands:

- in `handle_flows` step: `StartFlow`, `Clarify`, `CancelFlow`, `CannotHandle`.
- in `fill_slots` step: `SetSlot`. The CannotHandle command will be triggered from the `handle_flows` prompt when the scope of the user message is beyond starting, canceling, or clarifying a flow. The command will trigger the cannot handle pattern to indicate that the user message can not be treated as expected.

- [SingleStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#singlestepllmcommandgenerator)
- [MultiStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#multistepllmcommandgenerator)