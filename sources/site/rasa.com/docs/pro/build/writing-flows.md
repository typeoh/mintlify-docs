# Source: https://rasa.com/docs/pro/build/writing-flows

On this page

A **flow** in CALM is a structured sequence of steps for completing a specific user goal, like blocking a credit card, changing an address, adding a payee or researching stocks to invest in. It helps the agent orchestrate between -

1. Well-defined prescriptive business logic.
2. Dynamic business logic run by ReAct sub agents inside Rasa.
3. Externalized business logic by invoking sub agents outside Rasa.

This is done via [prescriptive steps](https://rasa.com/docs/reference/primitives/flow-steps/#prescriptive-steps) for well-defined business logic, [autonomous steps](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps) for business logic determined at runtime, or a mix of both approaches in a single flow. This hybrid approach gives you the flexibility to enforce critical business rules while letting AI handle complex, undefined, or tedious-to-script logic.

By separating the business logic from the rest of the conversation, you get:

- **Clarity**: Each flow focuses on a single job or outcome (e.g., “Block Credit Card”).
- **Reusability**: Flow logic can be called from other flows or triggered on its own.
- **Maintainability**: You can refine or change the flow logic without affecting the entire conversation design.

There are two types of flows in CALM: flows that you write to express your business logic and patterns or system flows that come out of the box with CALM. Before we move on to explain how flows work, a word on these conversation patterns:

### Conversation Patterns (System Flows)[​](https://rasa.com/docs/pro/build/writing-flows/#conversation-patterns-system-flows 'Direct link to Conversation Patterns (System Flows)')

In addition to the flows you write for your domain-specific tasks, Rasa provides **patterns**—pre-defined, reusable flows that handle “meta” conversational situations or repairs. For instance, if a user cancels a flow midway or wants to clarify a previously collected piece of information, a **pattern** steps in to handle this detour. These patterns work like templates: they can be triggered whenever relevant, so your assistant can handle common conversational patterns consistently.

Read more about customizing patterns under [Customizing Patterns](https://rasa.com/docs/pro/customize/patterns/)

## How Do Flows Work?[​](https://rasa.com/docs/pro/build/writing-flows/#how-do-flows-work 'Direct link to How Do Flows Work?')

### Triggering Flows[​](https://rasa.com/docs/pro/build/writing-flows/#triggering-flows 'Direct link to Triggering Flows')

CALM uses an [LLM “command generator” prompt](https://rasa.com/docs/pro/customize/command-generator/) that contains the conversation history, the relevant flows, slots, and conversation patterns. Essentially, CALM leverages the LLM to parse the user’s request into structured commands, referencing all pertinent context—including conversation history, current state, and flow definitions. This approach ensures that when the user’s goal matches the description of a given flow, that flow will be triggered. Flows can also be:

- Started by a direct [NLU trigger](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger) (e.g., when a recognized intent maps to a flow).
- [Linked](https://rasa.com/docs/reference/primitives/flow-steps/#link) or [called](https://rasa.com/docs/reference/primitives/flow-steps/#call) from inside another flow (for subflows or follow-up tasks).

### Dialogue Stack[​](https://rasa.com/docs/pro/build/writing-flows/#dialogue-stack 'Direct link to Dialogue Stack')

When a flow (or pattern) is activated, it’s placed on top of a **dialogue stack** (like stacking plates). The topmost flow is always active. Once that flow finishes or is canceled, the system returns to the next flow on the stack. This structure ensures that your assistant’s logic remains organized, even when users interrupt or pivot to new tasks.

## How to Write Flows[​](https://rasa.com/docs/pro/build/writing-flows/#how-to-write-flows 'Direct link to How to Write Flows')

Flows support three approaches: **prescriptive steps**, **autonomous steps**, and **hybrid flows** that combine both. Use prescriptive steps for critical business logic that must be enforced consistently. Use autonomous steps when business logic is undefined, highly variable, or too complex to script manually.

Writing a flow in CALM involves capturing the _essential steps_ to fulfill a user request without hardcoding every possible conversation path. You define flows as YAML in your `flows.yml` (or multiple YAML files), focusing on the business logic:

1. **Give the flow an ID and a clear description**
 
 flows.yml
 
 ```
    flows:  block_card:    description: Block a user's credit card when requested    steps: []
    ```
 
 - The `description` is critical for the LLM to understand when to pick this flow.
2. **Add the steps**
 
 Each step specifies what your assistant should do:
 
 - **Collect** user information:
 
 flows.yml
 
 ```
        flows:  block_card:    description: Block a user's credit card when requested    steps:    - collect: card_number      description: “The 16-digit card number to block”
        ```
 
 - **Take an action** (e.g., a [custom action](https://rasa.com/docs/pro/build/custom-actions/) or a response):
 
 flows.yml
 
 ```
        flows:  block_card:    description: Block a user's credit card when requested    steps:    # ...    - action: action_block_card_in_backend
        ```
 
 - **Set or reset slots**:
 
 flows.yml
 
 ```
        flows:  block_card:    description: Block a user's credit card when requested    steps:    # ...    - set_slots:      - card_number: null
        ```
 
 - **Call or link other flows** for subflows or follow-ups:
 
 flows.yml
 
 ```
        flows:  block_card:    description: Block a user's credit card when requested    steps:    # ...    - call: authenticate_user_flow
        ```
 
 flows.yml
 
 ```
        flows:  block_card:    description: Block a user's credit card when requested    steps:    # ...    - link: collect_feedback
        ```
 
 - **Include LLM driven autonomous steps** for dynamic logic:
 
 flows.yml
 
 ```
        flows:  investment_research:    description: Help user research and analyze investment options    steps:    - call: research_stocks
        ```
 
 Here, `research_stocks` can be either a ReAct style [autonomous sub agent interacting with tools from an MCP server](https://rasa.com/docs/pro/build/mcp-integration/#dynamic-selection-of-tools-in-an-autonomous-step) or an [external agent connected via A2A](https://rasa.com/docs/pro/build/integrating-external-agents/).
 
 
 More information on different step types can be found on the [reference page](https://rasa.com/docs/reference/primitives/flow-steps/).
 
3. **Include branching if needed**
 
 You can add simple conditional logic (like checking if a slot is filled or if a user is already authenticated):
 
 flows.yml
 
 ```
    flows:  block_card:    description: Block a user's credit card when requested    steps:    # ...    - collect: user_authenticated      next:        - if: not slots.user_authenticated          then:            - action: utter_ask_for_login            - link: authenticate_user_flow
    ```
 
4. **Force slot collection (suppress interruptions) in collect steps if needed**
 

You can choose to set the `force_slot_filling` property to `true` if you want the assistant to ignore any other commands and only focus on filling text type slots. This is especially useful when you want to collect feedback or longer text input from the user in collect steps.

flows.yml

```
flows:  block_card:    description: Block a user's credit card when requested    steps:    # ...    - collect: feedback      force_slot_filling: true
```

More information on forcing slot collection can be found on the [Flow reference page](https://rasa.com/docs/reference/primitives/flow-steps/#suppressing-interruptions).

5. **Leverage conversation patterns**
 
 You don’t need to write custom branching logic for every possible user detour. Flows represent the business logic your assistant is supposed to drive throughout conversation. Instead, rely on patterns (built-in flows) to accommodate user detours.
 

Flows in CALM can do more than these basics—such as advanced branching, slot validation, or subflow calls. For a deeper look at each property, step type, or YAML configuration, check out the [Flows reference](https://rasa.com/docs/reference/primitives/flows/).

## Importance of Clear Descriptions[​](https://rasa.com/docs/pro/build/writing-flows/#importance-of-clear-descriptions 'Direct link to Importance of Clear Descriptions')

Each flow has a **description** that briefly explains what the flow accomplishes. The LLM reads these descriptions to decide which flow to start. A concise, specific description reduces errors in flow selection. For example:

> Good: “Block a user’s credit card if they suspect fraud or want to freeze it”

> Less useful: “Card blocking request”

## Key Takeaways[​](https://rasa.com/docs/pro/build/writing-flows/#key-takeaways 'Direct link to Key Takeaways')

1. **Flows define business logic**: They can use prescriptive steps for guaranteed processes, autonomous steps for dynamic logic, or combine both approaches as needed.
2. **LLMs + flows**: The LLM remains flexible in interpreting user input and context, while flows make sure the assistant sticks to rules and processes.
3. **Choose the right approach**: Use prescriptive steps for critical business rules, autonomous steps for undefined or complex logic, and hybrid flows to get the best of both worlds.
4. **Write clear and detailed descriptions**: They help the LLM reliably select the right flow at the right time.
5. **Use patterns for conversation repair**: Don't clutter your flow with every possible detour. Let patterns handle cancellations, clarifications, or other unexpected conversation turns.

With flows, you maintain control over complex processes while giving the LLM room to shine in adapting to user input. Start by mapping out the tasks your assistant must support, split them into distinct flows, choose the right mix of prescriptive and autonomous steps, and keep descriptions tight. That's all you need to harness the best of both worlds: rigid business logic and flexible, human-like conversations.

- [Conversation Patterns (System Flows)](https://rasa.com/docs/pro/build/writing-flows/#conversation-patterns-system-flows)
- [How Do Flows Work?](https://rasa.com/docs/pro/build/writing-flows/#how-do-flows-work)
 - [Triggering Flows](https://rasa.com/docs/pro/build/writing-flows/#triggering-flows)
 - [Dialogue Stack](https://rasa.com/docs/pro/build/writing-flows/#dialogue-stack)
- [How to Write Flows](https://rasa.com/docs/pro/build/writing-flows/#how-to-write-flows)
- [Importance of Clear Descriptions](https://rasa.com/docs/pro/build/writing-flows/#importance-of-clear-descriptions)
- [Key Takeaways](https://rasa.com/docs/pro/build/writing-flows/#key-takeaways)