# Source: https://rasa.com/docs/reference/primitives/starting-flows

On this page

## Starting Flows[​](https://rasa.com/docs/reference/primitives/starting-flows/#starting-flows 'Direct link to Starting Flows')

Flows can be triggered by one of the following Rasa components: the LLM-based Command Generator ([SearchReadyLLMCommandGenerator](https://rasa.com/docs/reference/config/components/llm-command-generators/#searchreadyllmcommandgenerator) or [CompactLLMCommandGenerator](https://rasa.com/docs/reference/config/components/llm-command-generators/#compactllmcommandgenerator), or the [NLUCommandAdapter](https://rasa.com/docs/reference/config/components/nlu-command-adapter/).

LLM-based Command Generators use the descriptions of each of your flows, the running conversation, and other context to decide when to start a flow. It's important to write clear and distinct descriptions for each of your flows to help the LLM decide which flow to trigger, or to issue a [clarify](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference) command if the user hasn't provided enough information.

The `NLUCommandAdapter` uses a predicted intent to start a flow. In order to trigger a flow via the `NLUCommandAdapter` you need to have an [NLU trigger](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger) defined for your flow.

[Flow guards](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards) are conditions that have to be met before a flow can be started. Adding flow guards provides additional control over when flows are triggered.

### NLU Trigger[​](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger 'Direct link to NLU Trigger')

The `nlu_trigger` field is optional. If present, it contains a list of intents that can start the flow.

If you don't want to use a confidence threshold, list the intent names:

flows.yml

```
flows:  my_flow:    description: "A flow triggered with <intent-name>"    nlu_trigger:      - intent: <intent-name>    steps:      - action: my_action
```

If you only want the flow to trigger if the confidence is above a threshold, use the following syntax:

flows.yml

```
flows:  my_flow:    description: "A flow triggered with <intent-name>"    nlu_trigger:      - intent:          name: <intent-name>          confidence_threshold: 0  # threshold value, optional    steps:      - action: my_action
```

#### Multiple Intents in NLU Trigger[​](https://rasa.com/docs/reference/primitives/starting-flows/#multiple-intents-in-nlu-trigger 'Direct link to Multiple Intents in NLU Trigger')

You can list multiple intents for the `nlu_trigger`. If any of these intents is predicted, the flow will be started by the [NLUCommandAdapter](https://rasa.com/docs/reference/config/components/nlu-command-adapter/).

caution

In order to actually use `nlu_trigger`, you need to add the [NLUCommandAdapter](https://rasa.com/docs/reference/config/components/nlu-command-adapter/) before the `LLMCommandGenerator` to your NLU pipeline in the config file.

## Preventing Flows from Starting[​](https://rasa.com/docs/reference/primitives/starting-flows/#preventing-flows-from-starting 'Direct link to Preventing Flows from Starting')

### Flow Guards[​](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards 'Direct link to Flow Guards')

Flow guards are specified by adding an additional `if` field to the flow definition. For example, the following flow for showing the user's latest bill can only be triggered if the slots `authenticated` and `email_verified` are both `true`.

flows.yml

```
flows:  show_latest_bill:    description: A flow with a flow guard.    if: slots.authenticated AND slots.email_verified    steps:      - action: my_action
```

If the [condition](https://rasa.com/docs/reference/primitives/conditions/) after the `if` key is not met, the flow cannot be started. However, there are some exceptions to this:

1. The flow is triggered through a [link](https://rasa.com/docs/reference/primitives/flow-steps/#link) step from another flow.
2. The flow is triggered through a [call](https://rasa.com/docs/reference/primitives/flow-steps/#call) step from another flow.
3. The flow has defined intents with an [NLU trigger](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger). In this case, intent trigger messages, for example`/initialize_conversation`, can "force start" a flow for a targeted intent.

If you have a flow which should _exclusively_ be started via a [link](https://rasa.com/docs/reference/primitives/flow-steps/#link) or [call](https://rasa.com/docs/reference/primitives/flow-steps/#call) step, you can specify that by adding `if: False`. For example:

flows.yml

```
flows:  feedback_form:    description: A flow should only be linked to.    if: False    steps:      - action: my_action
```

- [Starting Flows](https://rasa.com/docs/reference/primitives/starting-flows/#starting-flows)
 - [NLU Trigger](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger)
- [Preventing Flows from Starting](https://rasa.com/docs/reference/primitives/starting-flows/#preventing-flows-from-starting)
 - [Flow Guards](https://rasa.com/docs/reference/primitives/starting-flows/#flow-guards)