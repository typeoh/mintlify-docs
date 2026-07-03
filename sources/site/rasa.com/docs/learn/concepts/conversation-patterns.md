# Source: https://rasa.com/docs/learn/concepts/conversation-patterns

On this page

Real conversations are rarely linear—users switch topics, correct themselves, and ask follow-up questions. **Conversation patterns** are reusable system flows that are **provided by CALM** and enable your AI assistant to handle these non-linear interactions cooperatively, and repair the conversation when customers don’t follow the path you expect them to.

## Use conversation patterns for smarter AI assistants[​](https://rasa.com/docs/learn/concepts/conversation-patterns/#use-conversation-patterns-for-smarter-ai-assistants 'Direct link to Use conversation patterns for smarter AI assistants')

You can’t predict everything a user will say—and with **conversation patterns**, you don’t have to. Conversation patterns make your assistant more flexible and effective by:

- **Keeping conversations on track:** Handle unexpected inputs seamlessly, leading to more successful outcomes and a better user experience.
- **Simplifying design and development:** Helps the team focus on crafting great user journeys and business logic instead of accounting for every possible detour.

## How Conversation Patterns work[​](https://rasa.com/docs/learn/concepts/conversation-patterns/#how-conversation-patterns-work 'Direct link to How Conversation Patterns work')

**Here’s an overview of the different types of conversation patterns:**

| **Category** | **Conversation Patterns** |
| --- | --- |
| **Conversation Repair** | Correct mistakes and misunderstandings with **correction**, **clarification**, and **interruption** patterns. |
| **Conversation Navigation** | Navigate conversations helping users to **cancel** , **restart**, or **complete** a process. |
| **External Support** | Provide external support with **search**, **human handoff**, or **chitchat** patterns. |
| **Voice** | Prepare to **repeat** something and handle scenarios where the user remains **silent**. |
| **System Error** | Inform the user of unexpected system issues, like an **internal error** or **code change**. Activate **cannot handle** (fallback) pattern if no flow or relevant documents apply. |

## Conversation Patterns in Action[​](https://rasa.com/docs/learn/concepts/conversation-patterns/#conversation-patterns-in-action 'Direct link to Conversation Patterns in Action')

Here is an example of a conversation pattern in each category listed above:

1. **Correction (repair pattern)**: Update and correct previously stored conversational information.

User:

My name is Jonathan, not John.

correction triggered

Bot:

Thanks for letting me know! It’s important to get things right. Should I update your name to Jonathan in my records?

User:

Yes, please.

updated first name to Jonathan

Bot:

Great! I’ve updated your name to Jonathan. Let me know if there’s anything else I can assist with.

Example showcasing corrections

2. **Cancellation (navigation pattern)**: Allows a user to stop a task in the middle of one topic or task.

User:

I want to send some money to Dimitri

Bot:

How much money do you want to send?

User:

Ah, never mind. I see I have already sent it earlier.

cancellation triggered

Bot:

Okay, I am canceling the transfer.

Example showcasing error handling

3. **Start a conversation (navigation pattern)**: Conversations can be initiated by either your assistant or the user.

session started

Bot:

Hi there! How can I assist you today?

Example showcasing a simple assistant greeting

4. **Search (external support pattern)**: Enables your assistant to query data from your knowledge base with Rasa's enterprise search feature and provide helpful answers.

User:

What is your policy on returns?

pattern search started

Bot:

We accept returns within 30 days of purchase if they are returned in new condition and original packaging. Would you like to start a return process?

Example showcasing enterprise search

5. **Repeat (voice pattern)**: Enables your assistant to repeat a previous message.

Bot:

Your order #1377 is on it's way. Would you like to track it?

User:

Can you repeat that?

repeat triggered

Bot:

Of course, your order #1377 is on its way. Would you like to track it?

Example showcasing a user asking the assistant to repeat the last message

6. **Errors (system pattern)**: Invalid responses or technical issues might require apologies or alternative actions. In these scenarios, you might need to re-ask questions, confirm intent, or escalate to a human agent.

User:

Update my order details.

internal error: request timed out

Bot:

Oops! It looks like something went wrong while processing your request. Let me try that again or escalate this issue.

Example showcasing error handling

## Customizing Conversation Patterns[​](https://rasa.com/docs/learn/concepts/conversation-patterns/#customizing-conversation-patterns 'Direct link to Customizing Conversation Patterns')

Rasa provides a default structure for each pattern, but you are free to [customize](https://rasa.com/docs/pro/customize/patterns/) them to fit your use case. Since patterns are system flows, you can edit them like any other flow in Rasa Pro or Rasa Studio. We recommend keeping the [contextual response rephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/) enabled for pattern flows as it helps your assistant sound more natural, especially when repairing dynamic interactions.

## Learn more about Patterns[​](https://rasa.com/docs/learn/concepts/conversation-patterns/#learn-more-about-patterns 'Direct link to Learn more about Patterns')

- Read the complete [pattern reference documentation](https://rasa.com/docs/reference/primitives/patterns/)
- Use Rasa Studio to [edit conversation patterns](https://rasa.com/docs/studio/build/flow-building/system-flows/)

- [Use conversation patterns for smarter AI assistants](https://rasa.com/docs/learn/concepts/conversation-patterns/#use-conversation-patterns-for-smarter-ai-assistants)
- [How Conversation Patterns work](https://rasa.com/docs/learn/concepts/conversation-patterns/#how-conversation-patterns-work)
- [Conversation Patterns in Action](https://rasa.com/docs/learn/concepts/conversation-patterns/#conversation-patterns-in-action)
- [Customizing Conversation Patterns](https://rasa.com/docs/learn/concepts/conversation-patterns/#customizing-conversation-patterns)
- [Learn more about Patterns](https://rasa.com/docs/learn/concepts/conversation-patterns/#learn-more-about-patterns)