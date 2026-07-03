# Source: https://rasa.com/docs/reference/config/policies/flow-policy

On this page

New in 3.7

The _Flow Policy_ is part of Rasa's new [Conversational AI with Language Models (CALM)](https://rasa.com/docs/learn/concepts/calm/) approach and available starting with version `3.7.0`.

Adding Flow Policy and NLU-based policies together

If you are migrating an NLU-based assistant to CALM and want to do so iteratively, then check out the [guide on migration to CALM](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/) for instructions on how to do so.

The Flow Policy is a state machine that deterministically executes the business logic defined in your [flows](https://rasa.com/docs/reference/primitives/flows/).

The Flow Policy oversees your assistant's state, handles state transitions, and starts new flows when required for [Conversation Repair](https://rasa.com/docs/learn/concepts/conversation-patterns/).

### Adding the Flow Policy to your assistant[​](https://rasa.com/docs/reference/config/policies/flow-policy/#adding-the-flow-policy-to-your-assistant 'Direct link to Adding the Flow Policy to your assistant')

To use the Flow Policy, add it to the list of policies in your `config.yml`.

config.yml

```
# ...policies:  - name: FlowPolicy
```

The Flow Policy does not have any additional configuration parameters.

### How does it work?[​](https://rasa.com/docs/reference/config/policies/flow-policy/#how-does-it-work 'Direct link to How does it work?')

The `FlowPolicy` employs a dialogue stack structure (Last In First Out) along with internal slots to manage the state of a conversation.

### Managing the State[​](https://rasa.com/docs/reference/config/policies/flow-policy/#managing-the-state 'Direct link to Managing the State')

The Flow Policy manages state using a "dialogue stack". Whenever a flow is started, it is pushed on to the dialogue stack, and this stack keeps track of the current position in each of those flows. The dialogue stack follows a "last in, first out" sequence, meaning that the flow that was started most recently will be completed first. Once it has completed, the next most recently started flow will continue.

Consider the `transfer_money` flow from the [tutorial](https://rasa.com/docs/pro/tutorial/):

flows.yml

```
flows:  transfer_money:    description: |      This flow lets users send money to friends      and family, in US Dollars.    steps:      - collect: recipient      - collect: amount      - action: utter_transfer_complete
```

When the conversation reaches a `collect` step, your assistant will ask the user for information to help it fill the corresponding slot. In the first step, your assistant will say _"Who would you like to send money to?"_ and then wait for a response.

When the user responds, the [Dialogue Understanding](https://rasa.com/docs/learn/concepts/dialogue-understanding/) pipeline will generate a sequence of commands which will determine what happens next. In the simplest case, the user says something like _"to Jen"_ and the command `SetSlot("recipient", "Jen")` is generated.

The flow has collected the information it needed and the flow policy proceeds to the next step. If instead the user says something like _"I want to send 100 dollars to Jen"_, the commands `SetSlot("recipient", "Jen"), SetSlot("amount", 100)` will be generated, and the flow policy will skip directly to the final step in the flow.

There are many things a user might say _other than_ providing the value of the slot your assistant has requested. They may clarify that they didn't want to send money after all, ask a clarifying question, or change their mind about something they said earlier. Those cases are handled by [Conversation Repair](https://rasa.com/docs/learn/concepts/conversation-patterns/).

### Starting New Flows[​](https://rasa.com/docs/reference/config/policies/flow-policy/#starting-new-flows 'Direct link to Starting New Flows')

A flow can be started in several ways:

- A flow is started when a Rasa component puts the flow on the stack. For example, the [LLM Command Generator](https://rasa.com/docs/reference/config/components/llm-command-generators/) puts a flow on the stack when it determines that a flow would be a good fit for the current conversation.
- One flow can ["link" to another flow](https://rasa.com/docs/reference/primitives/flow-steps/#link), which will initiate the linked flow and return to the original flow once the linked flow completes.
- In the case of [Conversation Repair](https://rasa.com/docs/learn/concepts/conversation-patterns/), default flows can be automatically added by the `FlowPolicy`.

- [Adding the Flow Policy to your assistant](https://rasa.com/docs/reference/config/policies/flow-policy/#adding-the-flow-policy-to-your-assistant)
- [How does it work?](https://rasa.com/docs/reference/config/policies/flow-policy/#how-does-it-work)
- [Managing the State](https://rasa.com/docs/reference/config/policies/flow-policy/#managing-the-state)
- [Starting New Flows](https://rasa.com/docs/reference/config/policies/flow-policy/#starting-new-flows)