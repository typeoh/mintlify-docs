# Source: https://rasa.com/docs/pro/customize/command-generator

On this page

At the heart of CALM’s [dialogue understanding](https://rasa.com/docs/learn/concepts/dialogue-understanding/) is a component called the [Command Generator](https://rasa.com/docs/reference/config/components/llm-command-generators/). Whenever a new user message arrives, this component takes the entire conversation context—such as active flows on the **dialogue stack**, previously filled slots, relevant patterns, and the user’s conversation history—and generates a list of high-level **commands**.

**[Commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference) can:**

- Begin a new flow.
- Terminate the current flow.
- Intercept user messages to bypass the current collection step.
- Assign a specified value to a slot.
- Request clarification.
- Provide a chitchat-style response, whether predefined or generated.
- Deliver a free-form, knowledge-based response.
- Transition the conversation to a human operator.
- Signal an internal error in handling the dialogue.
- Indicate a failure in generating any commands.

By framing user intent as commands rather than single-label “intents,” you can handle more complex scenarios—like when a user answers a question and requests a new flow simultaneously.

Tip

For more details on all command generator types, advanced configurations, and usage examples, see the [LLM Command Generators reference](https://rasa.com/docs/reference/config/components/llm-command-generators/).

## What Is the CompactLLMCommandGenerator?[​](https://rasa.com/docs/pro/customize/command-generator/#what-is-the-compactllmcommandgenerator 'Direct link to What Is the CompactLLMCommandGenerator?')

The **CompactLLMCommandGenerator** is the simplest LLM-based approach for converting user messages into commands. It operates on a single prompt that encapsulates:

1. **Conversation History**: The full or partial conversation so far, including user and assistant messages.
2. **Active Flow and Slots**: Which flow is currently on top of the dialogue stack and which slot (if any) is currently being asked for.
3. **Relevant Flows**: A subset of the flows in your assistant that are likely relevant to the user’s request (handled automatically by CALM’s _flow retrieval_ mechanism).
4. **Patterns / Repairs**: Predefined flows or patterns that can “interrupt” if the user changes their mind, wants to cancel, or triggers some other conversation repair scenario.

This single in-context prompt is passed to the underlying LLM (e.g., GPT-5.1) whenever the assistant needs to interpret the user’s latest message. The LLM’s response is turned into a list of commands, which the system executes in a single step.

### Flow Retrieval[​](https://rasa.com/docs/pro/customize/command-generator/#flow-retrieval 'Direct link to Flow Retrieval')

By default, CALM does not include all possible flows in the LLM prompt. Instead, it matches the incoming user message to each flow and includes only the top matching flows in the prompt. This keeps the prompt size manageable (and your costs lower). If you do want to disable or tweak flow retrieval, or always include certain flows, you can adjust that in your assistant configuration. For more details, see the [Flow Retrieval reference](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows).

## Customizing the Prompt Template[​](https://rasa.com/docs/pro/customize/command-generator/#customizing-the-prompt-template 'Direct link to Customizing the Prompt Template')

One of the main benefits of an LLM-based approach is **in-context learning**—that is, the ability to guide the model through instructions and context in the prompt. By default, the CompactLLMCommandGenerator uses a built-in **prompt template** that dynamically assembles relevant flows, the current conversation, and other context. However, you can override and customize this template to further tailor the model’s behavior.

### When Should You Customize?[​](https://rasa.com/docs/pro/customize/command-generator/#when-should-you-customize 'Direct link to When Should You Customize?')

1. **Flow Descriptions Aren’t Enough**
 
 Usually, you can steer the LLM by enriching your flows with clear, unambiguous descriptions and step-by-step instructions. But if you find the model still isn’t producing the commands you expect, or if you have domain-specific language the model often confuses, you may want to go further and rewrite the template.
 
2. **You Want Specific Formatting or Additional Examples**
 
 Suppose you need to show the LLM a set of few-shot examples or domain-specific instructions that can’t be captured solely in the flow/slot descriptions. A custom prompt template gives you full control over how that context appears.
 

### How to Customize[​](https://rasa.com/docs/pro/customize/command-generator/#how-to-customize 'Direct link to How to Customize')

1. **Create a Jinja2 Template**
 
 You’ll provide a custom `.jinja2` file that contains the static text you want plus references to dynamic variables (like `{{ current_flow }}`, `{{ flow_slots }}`, etc.).
 
2. **Reference That File in Your config.yml**
 
 Under `CompactLLMCommandGenerator`, set the `prompt_template` property:
 
 config.yml
 
 ```
    pipeline:  - name: CompactLLMCommandGenerator    prompt_template: prompts/my_custom_generator.jinja2
    ```
 
3. **Leverage Available Variables**
 
 In your Jinja2 file, you have access to multiple variables, such as:
 
 | Variable | Description |
 | --- | --- |
 | `current_conversation` | A readable transcript of the conversation so far. |
 | `user_message` | The latest user message. |
 | `available_flows` | A list of all flows potentially relevant to this conversation. |
 | `current_flow` | The name of the currently active flow. |
 | `flow_slots` | The slots associated with the current flow (name, value, etc.). |
 
 You can iterate over lists to print out details about flows or slots. For example:
 
 ```
    {% for flow in available_flows %}{{ flow.name }}: {{ flow.description }}{% endfor %}
    ```
 
4. **Make Your Template Clear and Consistent**
 
 - If your slot descriptions are long or bullet-pointed, consider adjusting how you render them. Use numbered lists or add separators so the LLM easily distinguishes the slot’s name from its instructions.
 - Keep the entire prompt in one language if your assistant is multilingual or works in a language other than English. Smaller LLMs often do better if the entire prompt is consistently in a single language.

Reference: Find the complete list of variables and more advanced ways to structure your prompt in the [Reference](https://rasa.com/docs/reference/config/components/llm-command-generators/) section on the command generator.

## Important Note on Customizing the Command Set[​](https://rasa.com/docs/pro/customize/command-generator/#important-note-on-customizing-the-command-set 'Direct link to Important Note on Customizing the Command Set')

For more technical details on configuration parameters, advanced prompt tuning, or combining multiple Command Generators, head over to the [Command Generators reference](https://rasa.com/docs/reference/config/components/llm-command-generators/).

While CALM’s Command Generator is designed to be flexible, the **Rasa team actively tests and maintains a fixed set of built-in commands** (e.g., `StartFlowCommand`, `CancelFlowCommand`, `SetSlotCommand`, etc.) to ensure the highest level of performance and accuracy. If you override or replace these built-in commands:

- **Reduced Accuracy Guarantees**
 
 We can’t guarantee that your assistant will maintain the same level of accuracy if you remove or rename these commands. Our tests and improvements assume that these commands exist in your system.
 
- **Potential Maintenance Overheads**
 
 Custom commands require additional testing to ensure they behave consistently across different user inputs. You’ll need to invest in ongoing QA to match the quality of the maintained commands.
 
- **Possibility of Breaking Changes**
 
 Future updates or improvements to CALM may assume the presence of these default commands. If you’ve drastically modified them, you could need to refactor your assistant for compatibility.
 

### How to customize existing commands[​](https://rasa.com/docs/pro/customize/command-generator/#how-to-customize-existing-commands 'Direct link to How to customize existing commands')

If a use case still requires customization of commands, then here is the recommended way:

1. **Override an existing command class**
 
 Identify the existing command you want to change. Define a new Python class inheriting from the corresponding command class. Override the following methods:
 
 - `command` - provides Rasa with the unique identifier of your custom command.
 - `from_dict` - handles the deserialization of the command from the JSON object stored in the tracker.
 - `from_dsl` - converts the parsed output from the LLM output into a command.
 - `to_dsl` - specifies how the command is represented when serialized back into the DSL format.
 - `regex_pattern` - used by Rasa to parse and recognize your custom command name from the output generated by the LLM.
 - `__eq__` - logic that determines whether two command instances are equal.
 
 All custom commands must inherit from the `Command` abstract class and adhere to the `PromptCommand` protocol.
 
 For example, to customize the name of the `HumanHandoffCommand` class used in your prompt, create the following:
 
 ```
    from __future__ import annotationsimport refrom dataclasses import dataclassfrom typing import Any, Dictfrom rasa.dialogue_understanding.commands import HumanHandoffCommand@dataclassclass CustomHumanHandoffCommand(HumanHandoffCommand):    """A custom human handoff command."""    # Optionally you can define the command arguments    new_command_arg: str    @classmethod    def command(cls) -> str:        """Returns the command type."""        return "custom human handoff"    @classmethod    def from_dict(cls, data: Dict[str, Any]) -> CustomHumanHandoffCommand:        """Converts the dictionary to a command.        Returns:            The converted dictionary.        """        return CustomHumanHandoffCommand(            new_command_arg=data["new_command_arg"]        )    @classmethod    def from_dsl(cls, match: re.Match, **kwargs: Any) -> CustomHumanHandoffCommand:        """Converts a DSL string to a command."""        # Example extraction: assume `new_command_arg` is captured by the first regex group        new_command_arg = match.group(1)        return CustomHumanHandoffCommand(            new_command_arg=new_command_arg        )    def to_dsl(self) -> str:        """Converts the command to a DSL string."""        return f"custom human handoff {self.new_command_arg}"    @staticmethod    def regex_pattern() -> str:        """Returns regular expression used to parse the command from the LLM output"""        return r"""^[\s\W\d]*custom human handoff ['"`]?([a-zA-Z0-9_-]+)['"`]*"""    def __eq__(self, other: object) -> bool:        return isinstance(other, CustomHumanHandoffCommand)
    ```
 
2. **Override the existing command behavior**
 
 If you also want to update the command behavior, override the `run_command_on_tracker` method within your class.
 
 ```
    @dataclassclass CustomHumanHandoffCommand(HumanHandoffCommand):    """A custom human handoff command."""    new_command_arg: str    ...    def run_command_on_tracker(        self,        tracker: DialogueStateTracker,        all_flows: FlowsList,        original_tracker: DialogueStateTracker,    ) -> List[Event]:        """Runs the command on the tracker.        Args:            tracker: The tracker to run the command on.            all_flows: All flows in the assistant.            original_tracker: The tracker before any command was executed.        Returns:            The events to apply to the tracker.        """        # Get the copy of the current dialogue stack of frames        stack = tracker.stack        # Implement your custom command behavior here        # For example, add a frame related to human handoff logic        stack.push(HumanHandoffPatternFlowStackFrame())        # Generate events reflecting the updated dialogue stack state        events = tracker.create_stack_updated_events(stack)        return events
    ```
 
3. **Create custom command generator**
 
 Create a custom command generator class by extending Rasa's default command generator. Override its `parse_commands` method as follows to handle your customized commands:
 
 custom\_command\_generator.py
 
 ```
    # Import the utility method under a different name to prevent confusion with the# command generator's `parse_commands`from rasa.dialogue_understanding.generator.command_parser import (    parse_commands as parse_commands_using_command_parsers,)# Define loggerstructlogger = structlog.get_logger()class CustomCommandGenerator(CompactLLMCommandGenerator):    @classmethod    def parse_commands(        cls, actions: Optional[str], tracker: DialogueStateTracker, flows: FlowsList    ) -> List[Command]:        commands = parse_commands_using_command_parsers(            actions,            flows,            # Register custom command            additional_commands=[CustomHumanHandoffCommand],            # Exclude replaced default command            default_commands_to_remove=[HumanHandoffCommand]        )        if not commands:            structlogger.warning(                f"{cls.__name__}.parse_commands",                message="No commands were parsed from the LLM actions.",                actions=actions,            )        return commands
    ```
 
4. **Update the prompt template**
 
 Create the custom prompt template that uses your new command name explicitly. For example:
 
 custom\_prompt\_template.jinja2
 
 ```
    ...## Available Actions:* `start flow flow_name`: Starting a flow. For example, `start flow transfer_money` or `start flow list_contacts`.* `set slot slot_name slot_value`: Slot setting. For example, `set slot transfer_money_recipient Freddy`. Can be used to correct and change previously set values.* `cancel flow`: Cancelling the current flow.* `disambiguate flows flow_name1 flow_name2 ... flow_name_n`: Disambiguate which flow should be started when user input is ambiguous by listing the potential flows as options. For example, `disambiguate flows list_contacts add_contact remove_contact ...` if the user just wrote "contacts".* `provide info`: Responding to the user's questions by supplying relevant information, such as answering FAQs or explaining services.* `offtopic reply`: Responding to casual or social user messages that are unrelated to any flows, engaging in friendly conversation and addressing off-topic remarks.* `custom human handoff new_command_arg`: <your new command description>...
    ```
 
5. **Update the `config.yml` to utilize the custom command generator and the new prompt template**
 
 config.yml
 
 ```
    pipeline:  - name: path.to.custom_command_generator.CustomCommandGenerator    prompt_template: path/to/custom_prompt_template.jinja2
    ```
 

- [What Is the CompactLLMCommandGenerator?](https://rasa.com/docs/pro/customize/command-generator/#what-is-the-compactllmcommandgenerator)
 - [Flow Retrieval](https://rasa.com/docs/pro/customize/command-generator/#flow-retrieval)
- [Customizing the Prompt Template](https://rasa.com/docs/pro/customize/command-generator/#customizing-the-prompt-template)
 - [When Should You Customize?](https://rasa.com/docs/pro/customize/command-generator/#when-should-you-customize)
 - [How to Customize](https://rasa.com/docs/pro/customize/command-generator/#how-to-customize)
- [Important Note on Customizing the Command Set](https://rasa.com/docs/pro/customize/command-generator/#important-note-on-customizing-the-command-set)
 - [How to customize existing commands](https://rasa.com/docs/pro/customize/command-generator/#how-to-customize-existing-commands)