# Source: https://rasa.com/docs/studio/build/flow-building/linking-flows

On this page

This guide will teach you how to **Link**, **Call**, and **Connect Steps** in Rasa Studio to structure and streamline your assistant’s logic.

You’ll learn what each step does, when to use it, and how these tools can help you build clean, modular flows that are easy to scale and maintain.

## Link Steps[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#link-steps 'Direct link to Link Steps')

### What is a Link?

A link is a mechanism in Rasa that connects flows together, enabling smooth transitions between business logic.

1. **Add a Link Step:** Select the "Link" option from the menu.

![Create a link step](https://rasa.com/docs/assets/images/create-link-step-ff7a7301d3308741d8ea1b7158970e80.png)

2. **Choose a Target Flow:** Select an existing flow or create a new one.

![Choose the target flow](https://rasa.com/docs/assets/images/money-transfer-link-create-6822f4a7f388f76ad592c793f370b821.png)

3. **Fill in Details (if creating a new flow):** Enter the necessary information in the modal that opens.

![Create a new flow to target](https://rasa.com/docs/assets/images/create-flow-leave-feedback-140ea0159f0fb66529ede4bccb0cd5e9.png)

note

A **Link** step hands over control to another flow and does **not** return to the original flow. Use this when the new flow should take over the rest of the conversation.

## Call Steps[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#call-steps 'Direct link to Call Steps')

### What are Call Steps?

A call step is a mechanism in Rasa that enables one flow to switch to another, complete tasks there, and then return, treating the second flow as a seamless extension of the first.

1. **Add a Call Step:** Select the "Call a flow and return" option from the menu.

![Call and return step](https://rasa.com/docs/assets/images/call-and-return-c69076a013fb8e8f397eca969b02537c.png)

2. **Choose a Target Flow:** Select an existing flow or create a new one.

![Select the call target](https://rasa.com/docs/assets/images/call-and-return2-e558a09e7ed484115e40ca9c513d7a43.png)

3. **Fill in Details (if creating a new flow):** Enter the necessary information in the modal that opens.

![Create a new flow to call](https://rasa.com/docs/assets/images/call-and-return3-8ff90f52492f43b902273b1e970061a8.png)

4. **Return to the Original Flow:** After the target flow finishes, control returns to the current flow and the conversation continues from the following step.

note

A **Call** step temporarily moves the conversation to another flow and returns after the target flow finishes. It's useful for handling sub-tasks like gathering user input or performing lookups.

## Connecting Steps Within a Flow[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#connecting-steps-within-a-flow 'Direct link to Connecting Steps Within a Flow')

Use connections to move between steps within the same flow — whether you're progressing, looping back, or merging branches.

To create a new connection:

1. **Open the Action Menu:** Select “Connect to” from the menu. ![Select connect to from the menu](https://rasa.com/docs/assets/images/create-node-connection-e862e0918b283b66a32c64795b6624cc.png)
 
2. **Choose a Target Step:** Select the step that you want to connect to. This can be earlier in the flow (to loop) or later in the flow (to converge). 
 📌 _You can’t connect directly to a condition. Conditions are part of a step, not standalone steps. To route logic through a condition, connect to the step that contains it._ ![Select target step](https://rasa.com/docs/assets/images/select-node-target-87f00452b495eb47021e403e69ca7a3a.png)
 

To delete a loop or connection:

1. Hover over the connection.
2. Click the delete icon.

![Delete a connection](https://rasa.com/docs/assets/images/delete-node-connection-b0d191402d903acabe2ccf40d90b9077.png)

caution

**Avoid Infinite Loops:** Always ensure there's an exit path when connecting back to earlier steps. Unintended loops may cause unexpected behavior during training or runtime.

Loops help keep flows modular and avoid duplication. Just make sure there’s always a way out.

## When to Use Link, Call, or Connections in Flows[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#when-to-use-link-call-or-connections-in-flows 'Direct link to When to Use Link, Call, or Connections in Flows')

### Use a **Link** when:[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#use-a-link-when 'Direct link to use-a-link-when')

_You want to permanently hand over the conversation to another flow._ 
The target flow is independent and should handle the rest of the interaction.

### Use a **Call** when:[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#use-a-call-when 'Direct link to use-a-call-when')

_You need to run a reusable flow and then return._ 
This is helpful for sub-tasks such as gathering information or checking conditions.

### Use **Connections** when:[​](https://rasa.com/docs/studio/build/flow-building/linking-flows/#use-connections-when 'Direct link to use-connections-when')

_You want to control the conversation within the same flow — whether moving forward, looping back, or merging multiple branches._ 
This helps reduce duplication and keeps your flows modular and easy to maintain.

By using Link, Call, and Connections intentionally, you can create modular, efficient flows that are easy to test and maintain.

- [Link Steps](https://rasa.com/docs/studio/build/flow-building/linking-flows/#link-steps)
- [Call Steps](https://rasa.com/docs/studio/build/flow-building/linking-flows/#call-steps)
- [Connecting Steps Within a Flow](https://rasa.com/docs/studio/build/flow-building/linking-flows/#connecting-steps-within-a-flow)
- [When to Use Link, Call, or Connections in Flows](https://rasa.com/docs/studio/build/flow-building/linking-flows/#when-to-use-link-call-or-connections-in-flows)
 - [Use a **Link** when:](https://rasa.com/docs/studio/build/flow-building/linking-flows/#use-a-link-when)
 - [Use a **Call** when:](https://rasa.com/docs/studio/build/flow-building/linking-flows/#use-a-call-when)
 - [Use **Connections** when:](https://rasa.com/docs/studio/build/flow-building/linking-flows/#use-connections-when)