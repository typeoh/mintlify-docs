# Source: https://rasa.com/docs/studio/build/flow-building/set-slots

On this page

This guide explains how to programmatically assign or clear slot values in your flows. With the **Set Slots** step, you can streamline your conversation logic by managing slot (stored user data) values independently from direct user input.

### What is a Slot?

Think of slots as your assistant’s memory. They store important pieces of information from the conversation, like a user’s name, date, or location, and can be referenced throughout the flow.

## Overview[​](https://rasa.com/docs/studio/build/flow-building/set-slots/#overview 'Direct link to Overview')

With the Set Slots step, you can:

1. **Assign a value:** Automatically update a slot value (e.g. set the 'age\_category' slot to 'Minor' based on the user's birth year).
2. **Clear a value:** Reset a slot value (e.g. clear the 'amount' slot if the user exceeds the allowed transaction limit to prompt re-entry)
3. **Persist a value:** Persist a slot value (e.g. persist the 'permission\_granted' slot after the flow ends)

![Panel for setting slots](https://rasa.com/docs/assets/images/set-slot-panel-1a947a6398cf24f36bbcc98394a56890.png)

note

This step runs silently in the background without any dialogue. If you need the assistant to prompt the user for input, consider using a [Collect Step](https://rasa.com/docs/studio/build/flow-building/collect/) instead.

### Setting a Slot Value[​](https://rasa.com/docs/studio/build/flow-building/set-slots/#setting-a-slot-value 'Direct link to Setting a Slot Value')

For example, if the user mentions their birth year during a conversation, we can automatically set the `age_category` slot to `Minor` and clear the `permission_granted` slot, streamlining the interaction accordingly.

![Set slot panel in Studio](https://rasa.com/docs/assets/images/set-slot-panel-1a947a6398cf24f36bbcc98394a56890.png)

### Setting Multiple Slots[​](https://rasa.com/docs/studio/build/flow-building/set-slots/#setting-multiple-slots 'Direct link to Setting Multiple Slots')

You can set multiple slots in one step. Simply click on "Set another slot" button to add more. You can see how many slots are set in one step by reading the number on the node.

![Set multiple slots in Studio](https://rasa.com/docs/assets/images/set-another-slot-button-55fb773395679d3e4f8b935dbe8adacc.png)

### Clearing a Slot[​](https://rasa.com/docs/studio/build/flow-building/set-slots/#clearing-a-slot 'Direct link to Clearing a Slot')

note

When you choose to clear a slot's value, its current value is reset, and the system will treat it as if it were `null`.

Use this option when a slot’s current value is no longer valid or needed. For example, if a user attempts to transfer more money than available, you can inform them of the issue and then clear the amount slot, prompting them to enter a correct value.

![Set slot operators](https://rasa.com/docs/assets/images/set-slot-operators-77ed04cef0367497f585b519f85f094c.png)

### Persisting a Slot value[​](https://rasa.com/docs/studio/build/flow-building/set-slots/#persisting-a-slot-value 'Direct link to Persisting a Slot value')

note

When you choose to persist a slot's value, that setting applies globally to all its occurrences in the flow.

Use this option when you want to persist slot’s value even after the flow ends.

![Set slot operators](https://rasa.com/docs/assets/images/set-slot-persistence-fc7112fdf5e2542ad40379bc011f42e9.png)

- [Overview](https://rasa.com/docs/studio/build/flow-building/set-slots/#overview)
 - [Setting a Slot Value](https://rasa.com/docs/studio/build/flow-building/set-slots/#setting-a-slot-value)
 - [Setting Multiple Slots](https://rasa.com/docs/studio/build/flow-building/set-slots/#setting-multiple-slots)
 - [Clearing a Slot](https://rasa.com/docs/studio/build/flow-building/set-slots/#clearing-a-slot)
 - [Persisting a Slot value](https://rasa.com/docs/studio/build/flow-building/set-slots/#persisting-a-slot-value)