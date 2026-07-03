# Source: https://rasa.com/docs/studio/build/flow-building/collect

On this page

This guide will teach you how to enable your assistant to gather user data through **Collect** steps. You’ll learn how to set up slots, create prompts, and validate user inputs, enabling your assistant to handle interactions smoothly and accurately.

### What is a Collect Step?

The collect step helps your assistant gather and store information provided by the user. You define the type of data you want to collect—like a name, email, or date—as a slot, and this step helps you to get the answer you need to set the value.

This guide walks you through setting up a Collect step — from creating slots and designing prompts to advanced validation and slot-filling logic.

![image](https://rasa.com/docs/assets/images/money-transfer-collect-info-165bb9bcde153a94d2e15125581606e1.png)

## Create a Collect Step[​](https://rasa.com/docs/studio/build/flow-building/collect/#create-a-collect-step 'Direct link to Create a Collect Step')

Here's how to define what your assistant should ask and where to store the user's response.

### 1\. Describe What to Collect[​](https://rasa.com/docs/studio/build/flow-building/collect/#1-describe-what-to-collect 'Direct link to 1. Describe What to Collect')

Add a short description to instruct the assistant on what kind of information it should collect. Use a clear and consistent format.

![image](https://rasa.com/docs/assets/images/money-transfer-description-ba58285f63b49e2ccf366bd982784544.png)

### 2\. Store the Response[​](https://rasa.com/docs/studio/build/flow-building/collect/#2-store-the-response 'Direct link to 2. Store the Response')

In order to be able to store the user's response, define a new [**slot**](https://rasa.com/docs/studio/build/flow-building/collect/#slots). Use descriptive names that clearly indicate the information you want to gather.

**📌 Tip:** To keep your slots organized - prefix flow-specific slots with the flow name ie. `money_transfer_` and global ones that are shared between flows with `global_`.

![image](https://rasa.com/docs/assets/images/money-transfer-create-slot-18f0640e9ed63218a6660d419912be80.png)

![image](https://rasa.com/docs/assets/images/money-transfer-amount-slot2-a815e3c94f71a854f42cdbbc3c6f681c.png)

#### Choose the Type of Information[​](https://rasa.com/docs/studio/build/flow-building/collect/#choose-the-type-of-information 'Direct link to Choose the Type of Information')

Pick the right slot type for the information you're collecting. Read more about the [**different types of slots here**](https://rasa.com/docs/studio/build/flow-building/collect/#slot-types).

![image](https://rasa.com/docs/assets/images/money-transfer-amount-slot3-32504b4d57e69ea1b4cc9955a3b88857.png)

## Design How to Ask For Information[​](https://rasa.com/docs/studio/build/flow-building/collect/#design-how-to-ask-for-information 'Direct link to Design How to Ask For Information')

You can tailor how your assistant asks for information or presents options to your user.

### Option 1: Use a Response[​](https://rasa.com/docs/studio/build/flow-building/collect/#option-1-use-a-response 'Direct link to Option 1: Use a Response')

Click **Select or create response** and write what you'd like your assistant your assistant to say. The response name will default to `utter_ask_{slot_name}`.

![image](https://rasa.com/docs/assets/images/money-transfer-create-response-d0bd0d282531498e2858ce4582ceadee.png)

![image](https://rasa.com/docs/assets/images/create-new-response-89f527039b6462742db09079d6cdcb1b.png)

![image](https://rasa.com/docs/assets/images/money-transfer-utter-ask-recipient-6e06a4344a2ffcc6127339adad6f2bf0.png)

#### Add Buttons (Optional)[​](https://rasa.com/docs/studio/build/flow-building/collect/#add-buttons-optional 'Direct link to Add Buttons (Optional)')

Optionally, you can add buttons to allow the user to select an answer more quickly and present user with structured choices. This is particularly helpful for complex tasks or when the assistant needs specific information to proceed. [Learn more about buttons](https://rasa.com/docs/studio/build/flow-building/collect/#buttons).

![image](https://rasa.com/docs/assets/images/recipient-buttons-efeeee229afb0fe0019608fd77cabe1e.png)

### Option 2: Use a Custom Action[​](https://rasa.com/docs/studio/build/flow-building/collect/#option-2-use-a-custom-action 'Direct link to Option 2: Use a Custom Action')

Instead of using a response for the Collect step, you can use a [custom action](https://rasa.com/docs/studio/build/actions/create-a-custom-action/#how-to-create-custom-action). This is useful if, for example, you want to display dynamic data to the user as options.

The custom action must follow a specific naming convention and be called `action_ask_{slot_name}`, which is why the name will be pre-filled. Make sure to add this custom action to your [domain file outside of Studio](https://rasa.com/docs/studio/build/actions/create-a-custom-action/).

![image](https://rasa.com/docs/assets/images/collect-with-action-4ea420fff1962c2cdef0c8e41bb714f0.png)

## Configure Collection Rules[​](https://rasa.com/docs/studio/build/flow-building/collect/#configure-collection-rules 'Direct link to Configure Collection Rules')

You can fine-tune how your assistant gathers information using the options below. These help ensure the right data is collected — and in the right way for your use case.

### Ask Before Filling[​](https://rasa.com/docs/studio/build/flow-building/collect/#ask-before-filling 'Direct link to Ask Before Filling')

Normally, if a slot is already filled earlier in the conversation, the assistant won’t ask again. 
But if you want the assistant to _always_ ask — even when it already has an answer — switch on **Ask before filling**.

**When to use it:** 
Useful when you want to confirm or re-collect something every time or when the value may need to be updated in the context of a flow, like a time slot for a booking.

![Toggle on the ask before filling feature](https://rasa.com/docs/assets/images/booking-ask-before-filling-f14be4ee8780c1cb823b2a8a042c8d59.png)

### Prevent Digressions[​](https://rasa.com/docs/studio/build/flow-building/collect/#prevent-digressions 'Direct link to Prevent Digressions')

If you want to prevent users jumping topics before giving you a crucial detail, you can ensure that the assistant collects a the slot value before moving to a different flow with this feature.

**When to use it:** 
If you have some required data like an email that you need to be able to fill out a support ticket — you can use this feature to ensure your assistant doesn't digress to another topic without it.

![Toggle on the ask before filling feature](https://rasa.com/docs/assets/images/booking-prevent-digression-5d1f686fc18498b9e7be123847cc4f73.png)

### Persist Slot Value[​](https://rasa.com/docs/studio/build/flow-building/collect/#persist-slot-value 'Direct link to Persist Slot Value')

By default slots are stored in the assistant's memory during a given flow but are then reset when you change to a new topic. Switch on **Persist slot value** to have your assistant remember the info for later use in a different flow context.

**When to use it:** 
Helpful for things like remembering a user's preferences across a full conversation.

![Toggle on the persist slot value feature](https://rasa.com/docs/assets/images/booking-persist-after-flow-ends-6def10447dc0165abf90bea8a62482bf.png)

### Silcence handling[​](https://rasa.com/docs/studio/build/flow-building/collect/#silcence-handling 'Direct link to Silcence handling')

If your Rasa license includes voice, you’ll see the **Silence handling** option. It sets how long the assistant waits during voice interactions before repeating the question and checking if the user is still there. You can define a global timeout in Assistant settings or override it per Collect step for questions that may need more response time.

![image](https://rasa.com/docs/assets/images/silence-handling-672ffc79e76cf10a597a8def81f1f414.png)

## Validate[​](https://rasa.com/docs/studio/build/flow-building/collect/#validate 'Direct link to Validate')

Validation is optional but sometimes you may want to validate a slot to ensure the user has provided the correct information before proceeding to the next step in the dialogue.

1. To do this, go to the **Validate** tab of **Collect information** step.
 
 ![image](https://rasa.com/docs/assets/images/money-transfer-validate-9ba4a729f27979e8510644f8cdc888d9.png)
 
2. Prevent user from going to the next step if certain conditions are not met.
 
 ![image](https://rasa.com/docs/assets/images/money-transfer-validation-options-9544379a0cc4a7c447af01d990db8552.png)
 
3. Click **Select or create response**. If the user's answer deviates from these values, a validation response will be displayed to them.
 
 ![image](https://rasa.com/docs/assets/images/money-transfer-create-validation-response-e74a4de7c930ec930125406c0d61e5c3.png)
 
4. Select **Create new response**.
 
 ![image](https://rasa.com/docs/assets/images/create-new-response-89f527039b6462742db09079d6cdcb1b.png)
 
5. In the modal that appears, specify the text for the response, and then click "Save."
 
 ![image](https://rasa.com/docs/assets/images/validation-window-af11188b9c28c0559b0b33d8b541af9f.png)
 
6. You can edit or delete previously created validation responses by clicking on the drop-down menu and choosing the **Manage responses** option in the modal that opens.
 
 ![image](https://rasa.com/docs/assets/images/manage-validation-responses1-ffe671fb7f7dca9559a521720f56e143.png) ![image](https://rasa.com/docs/assets/images/manage-validation-responses2-fefab8c2d3810f691f86fa1ae318bf60.png)
 

## Slots[​](https://rasa.com/docs/studio/build/flow-building/collect/#slots 'Direct link to Slots')

### What is a Slot?

Think of slots as your assistant’s memory. They store important pieces of information from the conversation, like a user’s name, date, or location, and can be referenced throughout the flow.

### Slot types[​](https://rasa.com/docs/studio/build/flow-building/collect/#slot-types 'Direct link to Slot types')

#### Text[​](https://rasa.com/docs/studio/build/flow-building/collect/#text 'Direct link to Text')

A **text** slot is used to store textual information such as names of countries, cities, or personal names. It's designed to hold various types of alphanumeric characters, making it suitable for a wide range of textual data.

#### Boolean[​](https://rasa.com/docs/studio/build/flow-building/collect/#boolean 'Direct link to Boolean')

A **boolean** slot is designed to store binary information, representing either a true or false value. It's commonly used for yes/no questions or scenarios where only two opposing options exist, like true/false statements.

#### Categorical[​](https://rasa.com/docs/studio/build/flow-building/collect/#categorical 'Direct link to Categorical')

A **categorical** slot is used to store data that can take on one of a specific set of values. It's well-suited for scenarios where data can be classified into distinct categories, like labels such as low, medium, or high to describe levels of something.

#### Float[​](https://rasa.com/docs/studio/build/flow-building/collect/#float 'Direct link to Float')

A **float** slot is employed for storing numerical values that can have decimal points. It's suitable for handling continuous numeric data, such as age, temperature, or monetary amounts involving fractions.

#### Any[​](https://rasa.com/docs/studio/build/flow-building/collect/#any 'Direct link to Any')

An **any** slot is capable of storing a wide variety of data types without any specific constraints. It's a flexible option for cases where the data type might vary or is not well-defined in advance, allowing you to store arbitrary values with diverse characteristics.

## Buttons[​](https://rasa.com/docs/studio/build/flow-building/collect/#buttons 'Direct link to Buttons')

You can make your assistant even more interactive by adding buttons to the responses in the Collect Information step. Instead of making users type, they can simply click on buttons to give their answers.

![image](https://rasa.com/docs/assets/images/tya-buttons-8ef0ae5f92760425f7c0b6a8efa3d7f7.png)

### Button properties[​](https://rasa.com/docs/studio/build/flow-building/collect/#button-properties 'Direct link to Button properties')

A button has two important parts:

- **Title:** This is the text your users will see on the button.
 
- **Payload:** Think of this as a behind-the-scenes command the assistant uses when the button is clicked. You don’t have to fill it in unless you want to give your assistant extra instructions like skipping LLM calls and going straight to a specific command, intent, or entity.
 
 ![image](https://rasa.com/docs/assets/images/tya-payloads-f6a998f0f5d9da0e5091abc616a5eaf0.png)
 

### How to add buttons[​](https://rasa.com/docs/studio/build/flow-building/collect/#how-to-add-buttons 'Direct link to How to add buttons')

1. Click **Add button** in your Collect information response.
 
 ![image](https://rasa.com/docs/assets/images/add-buttons-1-a34ac4c693ebe30b595e88bf4cb37f2c.png)
 
2. Choose between reusing an existing button or creating a new one. If you create a new button, enter a title for it. This title will be the text displayed on the button for users to click.
 
 ![image](https://rasa.com/docs/assets/images/add-buttons-2-ea98836c6d10ffb63e28cfb9a5d64b14.png)
 
3. Optional: If you want the button to bypass the LLM, choose a payload. Here are the types of things you can set it to do:
 
 - **Intent** — triggers an intent.
 - **Intent and entity** — passes entities along with the intent.
 - **Set slot** — sends commands to set or reset slot values.
 - **Text** — sends a predefined free-form text to the assistant. This should only be used if none of the above options apply.
 - If you leave the field empty, it will automatically be pre-filled with the same text as the title.
 
 ![image](https://rasa.com/docs/assets/images/add-buttons6-9c2cd57aea1151aa336888fa8d71f3ef.png)
 
4. Keep clicking **Add button** and fill in the information for as many buttons as you need.
 
 ![image](https://rasa.com/docs/assets/images/add-buttons-3-365775f26ebee8684e886d357792b604.png)
 
5. If you want to reuse an existing button, select **Use existing button** and choose the button you'd like to use from the dropdown menu.
 
 ![image](https://rasa.com/docs/assets/images/add-buttons-4-cf38adfb1d6a82d698dbbcf9cf3b69cb.png)
 
6. Use the icons next to the buttons to edit or delete them from the response.
 
 ![image](https://rasa.com/docs/assets/images/edit-delete-button-2529eca4432c8028bd466b717bb891c8.png)
 

### Reordering buttons[​](https://rasa.com/docs/studio/build/flow-building/collect/#reordering-buttons 'Direct link to Reordering buttons')

You can drag and drop buttons directly in the response editor — giving you full control over the order in which options are shown to users.

![image](https://rasa.com/docs/assets/images/reorder-buttons-056d206d78062539d73d6ab9c5c25de5.png)

## Slot Mappings (Advanced)[​](https://rasa.com/docs/studio/build/flow-building/collect/#slot-mappings-advanced 'Direct link to Slot Mappings (Advanced)')

Use slot mappings to control how slot values are extracted.

### From LLM[​](https://rasa.com/docs/studio/build/flow-building/collect/#from-llm 'Direct link to From LLM')

By default, the slot value is extracted from the user's message using an LLM, which interprets the message and fills the slot accordingly. The prompt includes descriptions and slot definitions from each flow as relevant information. However, if your assistant includes NLU components (intents and entities), you might consider using alternative extraction methods.

![image](https://rasa.com/docs/assets/images/mappings_llm-31cdf8000d5f61d46b386ee3507efc98.png)

### From text[​](https://rasa.com/docs/studio/build/flow-building/collect/#from-text 'Direct link to From text')

This mapping type uses the text of the last user message to fill the slot. Optionally it can be configured with the following parameters:

- **Intent** — Only applies the mapping when this intent is predicted.
 
- **Intent is not** — Excludes the mapping when this intent is predicted.
 
- **Active flows** — Restricts the mapping to certain flows, selectable in this field.
 

![image](https://rasa.com/docs/assets/images/mappings_text-a8fea983af38f618824dcd2aef058994.png)

### From entity[​](https://rasa.com/docs/studio/build/flow-building/collect/#from-entity 'Direct link to From entity')

This mapping type fills slots based on extracted entities. The following parameters are required:

- **Entity** — The entity used to fill the slot.

Optional parameters to specify the application of the mapping include:

- **Intent** — Applies only when this intent is predicted.
 
- **Intent is not** — Excludes the mapping when this intent is predicted.
 
- **Role** — Applies only if the extracted entity has this specified role.
 
- **Active flows** — Restricts the mapping to certain flows, selectable in this field.
 

![image](https://rasa.com/docs/assets/images/mappings_entity-6ab70ca5fa2c2895e3114a775d21256c.png)

### From intent[​](https://rasa.com/docs/studio/build/flow-building/collect/#from-intent 'Direct link to From intent')

This mapping fills a slot with a specified value if the user's intent matches. If no intent is specified, the slot fills regardless, unless the intent matches those listed under "Intent is not."

Required parameter:

- **Value** — The value to fill the slot.

Optional parameters include:

- **Intent** — Applies the mapping only when this intent is predicted.
 
- **Intent is not** — Excludes the mapping when this intent is predicted.
 
- **Active flows** — Restricts the mapping to certain flows, selectable in this field.
 

![image](https://rasa.com/docs/assets/images/mappings_intent-3c61915440349fe1d13dbd793b2be207.png)

### From custom action[​](https://rasa.com/docs/studio/build/flow-building/collect/#from-custom-action 'Direct link to From custom action')

Use the "From custom action" mapping type for slots that need to be filled by a specified [custom action](https://rasa.com/docs/studio/build/actions/create-a-custom-action/). This action must be detailed in the "Custom action" field of the slot mapping.

![image](https://rasa.com/docs/assets/images/mappings_CA-e8abd10fe18c99f18908b20dde705b53.png)

In case you use multiple mappings for one slot, they will be prioritized in the order they are listed. The first slot mapping found to apply will be used to fill the slot.

Recommendation on combining slot mapping types

A slot cannot have both "From LLM" and NLU-based predefined mappings or mappings from custom actions simultaneously. If you set a slot to use the "From LLM" mapping, you must not define any other types of mappings for that slot.

[Learn more about slot mappings](https://rasa.com/docs/reference/config/domain/#slots)

- [Create a Collect Step](https://rasa.com/docs/studio/build/flow-building/collect/#create-a-collect-step)
 - [1\. Describe What to Collect](https://rasa.com/docs/studio/build/flow-building/collect/#1-describe-what-to-collect)
 - [2\. Store the Response](https://rasa.com/docs/studio/build/flow-building/collect/#2-store-the-response)
- [Design How to Ask For Information](https://rasa.com/docs/studio/build/flow-building/collect/#design-how-to-ask-for-information)
 - [Option 1: Use a Response](https://rasa.com/docs/studio/build/flow-building/collect/#option-1-use-a-response)
 - [Option 2: Use a Custom Action](https://rasa.com/docs/studio/build/flow-building/collect/#option-2-use-a-custom-action)
- [Configure Collection Rules](https://rasa.com/docs/studio/build/flow-building/collect/#configure-collection-rules)
 - [Ask Before Filling](https://rasa.com/docs/studio/build/flow-building/collect/#ask-before-filling)
 - [Prevent Digressions](https://rasa.com/docs/studio/build/flow-building/collect/#prevent-digressions)
 - [Persist Slot Value](https://rasa.com/docs/studio/build/flow-building/collect/#persist-slot-value)
 - [Silcence handling](https://rasa.com/docs/studio/build/flow-building/collect/#silcence-handling)
- [Validate](https://rasa.com/docs/studio/build/flow-building/collect/#validate)
- [Slots](https://rasa.com/docs/studio/build/flow-building/collect/#slots)
 - [Slot types](https://rasa.com/docs/studio/build/flow-building/collect/#slot-types)
- [Buttons](https://rasa.com/docs/studio/build/flow-building/collect/#buttons)
 - [Button properties](https://rasa.com/docs/studio/build/flow-building/collect/#button-properties)
 - [How to add buttons](https://rasa.com/docs/studio/build/flow-building/collect/#how-to-add-buttons)
 - [Reordering buttons](https://rasa.com/docs/studio/build/flow-building/collect/#reordering-buttons)
- [Slot Mappings (Advanced)](https://rasa.com/docs/studio/build/flow-building/collect/#slot-mappings-advanced)
 - [From LLM](https://rasa.com/docs/studio/build/flow-building/collect/#from-llm)
 - [From text](https://rasa.com/docs/studio/build/flow-building/collect/#from-text)
 - [From entity](https://rasa.com/docs/studio/build/flow-building/collect/#from-entity)
 - [From intent](https://rasa.com/docs/studio/build/flow-building/collect/#from-intent)
 - [From custom action](https://rasa.com/docs/studio/build/flow-building/collect/#from-custom-action)