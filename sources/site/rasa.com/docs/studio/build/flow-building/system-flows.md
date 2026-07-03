# Source: https://rasa.com/docs/studio/build/flow-building/system-flows

On this page

In an ideal conversation, known as the "happy path," the assistant asks for information, and the user provides the correct response, allowing the conversation to flow smoothly. But in reality, conversations don’t always follow that perfect path. This is where system flows, come in.

System Flows in Rasa

It's important to note that there is a **naming difference** in Studio vs the rest of Rasa for this feature. 
What is called a **System Flow** in Studio is called a **Pattern** in Rasa. To see a complete list of what patterns are included in Rasa and detailed specs take a look at [the reference](https://rasa.com/docs/reference/primitives/patterns/).

### A Short Intro to System Flows[​](https://rasa.com/docs/studio/build/flow-building/system-flows/#a-short-intro-to-system-flows 'Direct link to A Short Intro to System Flows')

System flows are pre-built flows available out of the box, designed to handle conversations that go off track. For example, they help when:

- The assistant asks for information (like an amount of money), but the user responds with something else.
- The user interrupts the current flow and changes the topic.
- The user changes their mind about something they said earlier.

If you're interested into diving more into detail about system flows and how they work, you can read more in the Learn section on [Conversation Patterns](https://rasa.com/docs/learn/concepts/conversation-patterns/).

### System Flows in Studio[​](https://rasa.com/docs/studio/build/flow-building/system-flows/#system-flows-in-studio 'Direct link to System Flows in Studio')

To find system flows, navigate to the **System Flows** tab of the Flows page.

![View of the system flows tab in Studio](https://rasa.com/docs/assets/images/system-flows-55cd4b7fbb70cd64f3d4e3dbef38a2ce.png)

### Modifying system flows[​](https://rasa.com/docs/studio/build/flow-building/system-flows/#modifying-system-flows 'Direct link to Modifying system flows')

The default version of system flows is always included in your assistant model out of the box. While you can't disable them—since they are essential for the smooth functioning of your assistant—you can fully customize them to suit your specific business needs and conversation design best practices.

### Editing system flow texts[​](https://rasa.com/docs/studio/build/flow-building/system-flows/#editing-system-flow-texts 'Direct link to Editing system flow texts')

To change the default text used in a system flow, you can override the message with a new one. Here's how:

Select the step in a system flow you want to customize.

![Customizing a system flow in Studio](https://rasa.com/docs/assets/images/system-flow-customizing-1-27c1e1e5ba2f013ce462297cb7543c2b.png)

Click on the message name in the right panel and choose the "Create message" option.

![Creating a custom message for system flows in Studio](https://rasa.com/docs/assets/images/system-flow-customizing-2-aec3f219e302b79d9f74013e7007c75b.png)

In the modal that opens, specify the new message name and text, then click Save.

![Saving a new message for system flows in Studio](https://rasa.com/docs/assets/images/system-flow-customizing-3-37f0a16c2e763adf19fa1cfb66f8a610.png)

The message is now replaced.

![Completed message step for system flow in Studio](https://rasa.com/docs/assets/images/system-flow-customizing-4-ecf18fa21b71f86a376f20dde9bba60f.png)

### Overriding system flow logic[​](https://rasa.com/docs/studio/build/flow-building/system-flows/#overriding-system-flow-logic 'Direct link to Overriding system flow logic')

You can modify system flows just like custom ones by adding or removing steps. Let’s walk through an example of modifying the `pattern_completed` system flow that asks if the user needs more help after completing their goals or ending a conversation.

By default, the `pattern_completed` has one step—a message that asks, "What else can I help you with?" This message will be presented to the user after each flow ends unless there are specific links to other flows. However, after some flows, such as a greeting, we may want to skip this question, especially when the assistant hasn't provided any help yet. ![View Pattern Completed Flow](https://rasa.com/docs/assets/images/pattern-completed-f03eb458ca891988982a27daf2348cef.png)

To reconfigure the system flow so that the question is skipped after specific flows, we can add a Logic branch before the message to specify those flows.

![Skip Question in Pattern Completed Flow](https://rasa.com/docs/assets/images/pattern-completed-logic-3101649fe04b22ae00e2a4d6bed213dc.png)

In the first logic branch, select the context "previous flow name" and specify that the previous flow shouldn't be the one that greets the user.

![Set the Context for Pattern Completed](https://rasa.com/docs/assets/images/pattern-completed-context-9a460c0cba7e2dcf7bdbe34695eeee0d.png)

![Add a Condition for Pattern Completed](https://rasa.com/docs/assets/images/pattern-completed-condition-a69dab80496d200b487cec0ece3b01c4.png)

For the **Else** branch, which now handles the case after the "greet" flow, we want the assistant to remain silent and wait for the next question from the user, so we leave it empty.

![Add the Else Branch for Pattern Completed](https://rasa.com/docs/assets/images/pattern-completed-else-4f9869190de6ccb05d3abd943b7e0a3f.png)

After modifying a system flow, make sure to re-train your assistant and test the changes on the Try your assistant page. Resetting to default

After customizing a system flow, you will see the "Customized" label next to it. If you want to cancel your updates and reset a system flow to its default configuration, simply hover over it in the table and click the "Reset to default" button.

![View Tagged Customized System Flows](https://rasa.com/docs/assets/images/pattern-reset-to-default-e875b53600e773ff4d9d0ddb697fb4e0.png)

### Enabling Enterprise search policy via `pattern_search`[​](https://rasa.com/docs/studio/build/flow-building/system-flows/#enabling-enterprise-search-policy-via-pattern_search 'Direct link to enabling-enterprise-search-policy-via-pattern_search')

You can enable the Enterprise search policy in Studio and integrate knowledge base document search by modifying assistant configuration and `pattern_search`.

Log in with a `superuser` or `developer` role to access Assistant configuration.

![Developer Login](https://rasa.com/docs/assets/images/log-in-developer1-5b60879ecd2227ca1bfb6e7bbb8c2840.png)

Go to the "System flows" tab and open `pattern_search` to modify it.

![Search for System Flows](https://rasa.com/docs/assets/images/system-flows-search-c120c2b466605900e7a7a4cec5a0c47d.png)

Delete the message and add the custom action step instead. In the right panel, select the action named `action_trigger_search`.

![Add Custom Action Step in System Flows](https://rasa.com/docs/assets/images/pattern-search-action-23f7b29c1a72ea847cf793118c11ef57.png)

Go to the Assistant settings page to modify the configuration.

![Modify Pattern Search settings](https://rasa.com/docs/assets/images/pattern-search-settings-6ce8e275101463fb564503463fab5057.png)

In the config.yml field, add Enterprise Search Policy and specify the type of your vector store.

![Configure Enterprise Search in Studio](https://rasa.com/docs/assets/images/config-es-policy-9fed06f9a353f51c7c4cfd4334372920.png)

In the endpoints.yml, set up the connection to your vector store. Click "Save".

![Configure your Vector Store for Enterprise Search](https://rasa.com/docs/assets/images/config-es-vector-store-e19fd87c5478ac87683723e58f584202.png)

Train your assistant and test it on the Try your assistant page. You will be able to see the answers generated by the Enterprise search.

- [A Short Intro to System Flows](https://rasa.com/docs/studio/build/flow-building/system-flows/#a-short-intro-to-system-flows)
- [System Flows in Studio](https://rasa.com/docs/studio/build/flow-building/system-flows/#system-flows-in-studio)
- [Modifying system flows](https://rasa.com/docs/studio/build/flow-building/system-flows/#modifying-system-flows)
- [Editing system flow texts](https://rasa.com/docs/studio/build/flow-building/system-flows/#editing-system-flow-texts)
- [Overriding system flow logic](https://rasa.com/docs/studio/build/flow-building/system-flows/#overriding-system-flow-logic)
- [Enabling Enterprise search policy via `pattern_search`](https://rasa.com/docs/studio/build/flow-building/system-flows/#enabling-enterprise-search-policy-via-pattern_search)