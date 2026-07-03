# Source: https://rasa.com/docs/learn/concepts/dialogue-management

On this page

The dialogue manager is the part of Rasa that decides how to take the best next step based on the user’s input and the current conversation state. In CALM, you guide these decisions by creating Flows—high-level outlines that define the key steps and business logic your assistant follows to complete a task. Instead of mapping every possible turn, you can focus on the essential steps. This keeps interactions structured while allowing for flexible, dynamic conversations.

## How dialogue management works[​](https://rasa.com/docs/learn/concepts/dialogue-management/#how-dialogue-management-works 'Direct link to How dialogue management works')

In the last section, we learned how Rasa's [dialogue understanding](https://rasa.com/docs/learn/concepts/dialogue-understanding/) component uses an LLM to generate a set of internal commands representing how the user wants to progress the conversation. Once commands are issued, they are passed on to the dialogue manager to guide the next steps:

![dialogue understanding](https://rasa.com/docs/assets/images/concepts_dialogue_management-46c8f43da139ee2802dc7b24fd60dd3c.png)

Here’s how the dialogue manager processes these commands, step by step:

- **Receive the commands:** The dialogue understanding component delivers a set of commands to the dialogue manager, these commands might include something like `StartFlow("transfer_money")` or `SetSlot(transfer_amount, 100)`
- **Take the next steps:** Based on these commands, the dialogue manager will begin to take the next steps. This might involve routing to and from flows, collecting information, interacting with backend systems, and delivering responses —all while staying within the boundaries of the business logic defined in the flows.
- **Handle exceptions:** Sometimes, users will deviate from the paths outlined in flows—asking questions outside the scope of the AI assistant, changing topics, correcting, or interrupting the conversation. In these cases, the dialogue understanding module will issue commands that leverage conversation pattern flows —system flows provided by Rasa that can handle unexpected interactions and guide the assistant back on course.

If you are curious how Rasa manages multiple active flows, dive into how the system activates and moves through [flows](https://rasa.com/docs/reference/config/policies/flow-policy/).

## Anatomy of a flow[​](https://rasa.com/docs/learn/concepts/dialogue-management/#anatomy-of-a-flow 'Direct link to Anatomy of a flow')

A flow represents a structured sequence of steps that guide a conversation toward completing a specific task or process. Flows define what should happen at each stage of the interaction—this is known as **business logic**. Each step in a flow determines how the assistant responds, gathers information, or moves to the next part of the conversation.

A flow is made up of one or more steps, which serve as the building blocks of conversation logic:

- **Action**: Performs a task, such as sending a response or calling an API.
- **Collect**: Gathers useful user input and stores it in a variable (a "slot" in Rasa).
- **Set Slots**: Directly assigns a value to a slot.
- **Link**: Connects one flow to another after completion (e.g., directing users to a CSAT form after a successful flow).
- **Call**: Calls another flow mid-conversation (e.g., triggering user authentication) and returns upon completion.
- **Conditions**: Creates branching logic based on customer profiles, collected information, channels, and more.

## Flow building in action[​](https://rasa.com/docs/learn/concepts/dialogue-management/#flow-building-in-action 'Direct link to Flow building in action')

Let’s build a simple flow to help users reset their password by collecting their email and sending a reset link.

### Steps to build the flow[​](https://rasa.com/docs/learn/concepts/dialogue-management/#steps-to-build-the-flow 'Direct link to Steps to build the flow')

1. **Describe the flow** – Give the flow a description so the system knows when to trigger it.
2. **Collect information** – Ask for the user's email (`collect` step).
3. **Trigger a backend service** – Send a reset email (`action` step).
4. **Confirm with the user** – Inform them that the reset email is on its way (`action` step).

### Visualization of the flow[​](https://rasa.com/docs/learn/concepts/dialogue-management/#visualization-of-the-flow 'Direct link to Visualization of the flow')

Below, you can see the password reset flow visualized in both **Rasa Studio** (no-code UI) and **Rasa Pro** (pro-code UI). The arrows illustrate how UI elements map to code-based steps.

![Pro and no code flow](https://rasa.com/docs/assets/images/concepts_dm_password_reset_arrows-1f8fc464c0e33aeb3423be93bd0e36e4.png)

The most important thing to understand about business logic in Rasa is that it does _not_ define all the possible paths a conversation can take.

A flow defines:

- What information you need to collect from the user
- What data you need to read & write via APIs
- Any branching logic based on the information that's gathered

If you previously designed AI assistants using flowcharts of possible conversation paths, you'll find that flows in Rasa are much simpler to build, modular, and easy to maintain.

## Learn more[​](https://rasa.com/docs/learn/concepts/dialogue-management/#learn-more 'Direct link to Learn more')

- Learn how to build flows in [Rasa Studio](https://rasa.com/docs/studio/build/flow-building/introduction/)
- Learn how to build flows in [Rasa Pro](https://rasa.com/docs/pro/build/writing-flows/)
- How to use conditions to build [branching logic](https://rasa.com/docs/reference/primitives/conditions/)

- [How dialogue management works](https://rasa.com/docs/learn/concepts/dialogue-management/#how-dialogue-management-works)
- [Anatomy of a flow](https://rasa.com/docs/learn/concepts/dialogue-management/#anatomy-of-a-flow)
- [Flow building in action](https://rasa.com/docs/learn/concepts/dialogue-management/#flow-building-in-action)
 - [Steps to build the flow](https://rasa.com/docs/learn/concepts/dialogue-management/#steps-to-build-the-flow)
 - [Visualization of the flow](https://rasa.com/docs/learn/concepts/dialogue-management/#visualization-of-the-flow)
- [Learn more](https://rasa.com/docs/learn/concepts/dialogue-management/#learn-more)