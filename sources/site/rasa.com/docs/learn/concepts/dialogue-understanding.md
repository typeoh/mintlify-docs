# Source: https://rasa.com/docs/learn/concepts/dialogue-understanding

On this page

Dialogue understanding is defined as the assistant's ability to interpret user input, and determine the next best step in the conversation. When a user sends a message to a Rasa assistant, the dialogue understanding component interprets the input and generates a list of commands that represent _how the user wants to progress the conversation_.

## How dialogue understanding works[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#how-dialogue-understanding-works 'Direct link to How dialogue understanding works')

CALM assistants use an LLM for dialogue understanding within a controlled framework. A structured prompt ensures reliable and predictable interpretation, allowing the system to suggest the next steps but not execute them. This ensures the assistant follows predefined logic and operates strictly with the guidelines and responses you define.

![dialogue understanding](https://rasa.com/docs/assets/images/concepts_dialogue_understanding-9ef34071885866144670a67a1518c168.png)

Here’s how it works, step by step:

1. **Context-aware understanding:** When a user messages a CALM assistant, the system is prompted to read the full conversation transcript, including the latest message and collected slots, ensuring it understands the input in context.
2. **Identifying the next best steps:** The system is additionally prompted to outline the most relevant next steps for the user. These next steps are communicated as a set of "commands" for the [dialogue manager](https://rasa.com/docs/learn/concepts/dialogue-management/) to follow and might include instructions to continue a flow, start a new flow, or activate conversation pattern flows to handle something unexpected.
3. **Keeping the conversation on track:** Each command issued by the dialogue understanding module can only leverage existing flows and knowledge bases. That means the system cannot create new workflows or generate responses beyond what has already been designed.

If you're curious to learn more about how these commands are generated, check out the reference page on Rasa's [command generators](https://rasa.com/docs/reference/config/components/llm-command-generators/).

## Dialogue understanding in action[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#dialogue-understanding-in-action 'Direct link to Dialogue understanding in action')

Let's look at the dialogue understanding component in action to understand how this works in practice.

### 1\. Common commands[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#1-common-commands 'Direct link to 1. Common commands')

The simplest and most common commands involve the `start flow` and `set slot` commands. For those familiar with NLU-based chatbots, these commands resemble intents and entities. In the example below, the system selects these commands because the user initiated a new topic (`start flow book_flight`) and provided key information (`set slot destination: Singapore`).

User:

I want to book a flight from London to Singapore.

start flow book\_flight; set slot origin London; set slot destination Singapore

### 2\. Advanced commands[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#2-advanced-commands 'Direct link to 2. Advanced commands')

Other commands are useful when it isn't clear what the user wants to do. Sometimes you might need commands like `clarify` or `cancel` to help the user progress the conversation.

User: card

clarify flows freeze\_card, cancel\_card, order\_new\_card

Bot: Would you like to freeze or cancel your card, or order a new one?

User: cancel

start flow cancel\_card

Let's examine how the system works to find the best way to assist:

- The user sends a single-word message: _card_
- Your assistant has multiple flows defined that could be relevant to the user
- The dialogue understanding component generates a `clarify` command with the potentially relevant flows
- The user sends another single-word message: _cancel_
- Looking at the whole conversation, the dialogue understanding component has enough information to start the `cancel_card` flow.

It's important to understand how the dialogue understanding component interprets context:

- The `clarify flows` command appears because your assistant has multiple card-related flows defined. This demonstrates how the DU component considers your defined business logic. If only one card-related flow existed, it would have directly generated a `start flow` command instead.
- When the user responds with a single word, "cancel," the system correctly interprets this as a request to cancel a card. This happens because the dialogue understanding component analyzes the entire conversation history. The same word "cancel" might result in different commands in a different context.

### Key dialogue understanding commands[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#key-dialogue-understanding-commands 'Direct link to Key dialogue understanding commands')

| Command | Meaning |
| --- | --- |
| `start flow` | The user wants to complete a specific flow |
| `cancel flow` | The user does not want to continue with the current flow |
| `clarify flows` | Multiple flows could be relevant to what the user is asking for |
| `set slot` | The user has provided a value for one of the slots required for this flow |
| `correct slot` | The user has provided an updated value for a slot |
| `chitchat` | The user is making small-talk |
| `knowledge answer` | The user is asking a question that should have an answer in the knowledge base |
| `human handoff` | The user wants to speak to a human |

## LLM vs. NLU vs Hybrid approaches[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#llm-vs-nlu-vs-hybrid-approaches 'Direct link to LLM vs. NLU vs Hybrid approaches')

Choosing between an LLM and an NLU model for dialogue understanding depends on factors like accuracy, cost, latency, and system requirements. With CALM, you can use both an LLM for more flexible, context-aware conversations and an NLU model for faster, cost-effective processing.

### Benefits of an LLM[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#benefits-of-an-llm 'Direct link to Benefits of an LLM')

LLMs outperform NLU classifiers in key dialogue understanding tasks. Below are key advantages of using LLMs for dialogue understanding:

- **Context awareness:** Analyzes the full conversation history, not just the latest input, leading to more accurate and contextually relevant decisions.
- **Less maintenance required:** Minimizes the need for intent classification, data labeling, and retraining, making system management easier.
- **Nuanced interactions:** Handles ambiguous, indirect, or nuanced language, improving user experience and reducing failure rates.

### Benefits of an NLU model[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#benefits-of-an-nlu-model 'Direct link to Benefits of an NLU model')

You can use an NLU model for dialogue understanding if you prefer. Here’s why you might choose to stick with NLU:

- **No LLM approval or readiness:** If you cannot use LLMs, NLU provides a stable alternative. However, remember that when you choose this path, you can still use flows, but CALM dialogue understanding output is limited to `set slot` and `start flow` commands. You won't benefit from additional commands and conversation patterns.
- **Optimizing Cost and Latency:** LMs require more resources than NLU models, increasing cost and latency. This can be a critical factor if you're scaling your assistant or building a voice application. Sometimes, it makes sense to use NLU, where it performs well, reserving LLMs for more complex tasks to optimize efficiency.

### Benefits of a hybrid approach[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#benefits-of-a-hybrid-approach 'Direct link to Benefits of a hybrid approach')

CALM lets you combine NLU and language models within the same system for the best of both worlds. This means you continue to define business logic in Flows and choose whether NLU or an LLM activates each flow. This helps:

- **Freedom to choose** Use NLU where it works well, and LLMs to provide extra resilience where you need them.
- **Unified System** Many other platforms simply orchestrate to and from an NLU system and an agentic framework. With CALM the systems don't compete but are connected.

## Evaluating dialogue understanding accuracy[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#evaluating-dialogue-understanding-accuracy 'Direct link to Evaluating dialogue understanding accuracy')

You can use a set of test cases to evaluate and monitor the accuracy of your Dialogue Understanding component. These should ideally be sourced from production conversations and reflect the way that users actually speak. A dialogue understanding test case contains:

- The transcript of a conversation between a user and your assistant.
- The correct commands which should be predicted after each user message.

You can then run Rasa's [end-to-end test](https://rasa.com/docs/pro/testing/evaluating-assistant/) to evaluate accuracy with your test cases. This will generate a report with different metrics.

## Learn more[​](https://rasa.com/docs/learn/concepts/dialogue-understanding/#learn-more 'Direct link to Learn more')

- Learn to write flow descriptions in [code](https://rasa.com/docs/pro/build/writing-flows/)
- Learn to write flow descriptions in [Rasa Studio](https://rasa.com/docs/studio/build/flow-building/trigger-flows/)

- [How dialogue understanding works](https://rasa.com/docs/learn/concepts/dialogue-understanding/#how-dialogue-understanding-works)
- [Dialogue understanding in action](https://rasa.com/docs/learn/concepts/dialogue-understanding/#dialogue-understanding-in-action)
 - [1\. Common commands](https://rasa.com/docs/learn/concepts/dialogue-understanding/#1-common-commands)
 - [2\. Advanced commands](https://rasa.com/docs/learn/concepts/dialogue-understanding/#2-advanced-commands)
 - [Key dialogue understanding commands](https://rasa.com/docs/learn/concepts/dialogue-understanding/#key-dialogue-understanding-commands)
- [LLM vs. NLU vs Hybrid approaches](https://rasa.com/docs/learn/concepts/dialogue-understanding/#llm-vs-nlu-vs-hybrid-approaches)
 - [Benefits of an LLM](https://rasa.com/docs/learn/concepts/dialogue-understanding/#benefits-of-an-llm)
 - [Benefits of an NLU model](https://rasa.com/docs/learn/concepts/dialogue-understanding/#benefits-of-an-nlu-model)
 - [Benefits of a hybrid approach](https://rasa.com/docs/learn/concepts/dialogue-understanding/#benefits-of-a-hybrid-approach)
- [Evaluating dialogue understanding accuracy](https://rasa.com/docs/learn/concepts/dialogue-understanding/#evaluating-dialogue-understanding-accuracy)
- [Learn more](https://rasa.com/docs/learn/concepts/dialogue-understanding/#learn-more)