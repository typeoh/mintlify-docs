# Source: https://rasa.com/docs/studio/build/flow-building/sending-messages

On this page

This guide will show you how to create and manage **Message** steps in Rasa Studio. You'll learn to craft clear, engaging responses and refine them to make your assistant’s interactions more natural.

### What is a Message?

A message is what your assistant sends to the user. This can include text, buttons, images, or rich content like quick replies. Messages can be customized for different conversation scenarios.

## How to create a Message step[​](https://rasa.com/docs/studio/build/flow-building/sending-messages/#how-to-create-a-message-step 'Direct link to How to create a Message step')

1. Select the **Send message** step in the menu.
 
 ![image](https://rasa.com/docs/assets/images/welcome-flow-message-144546ec9f1c60742d0441a745cd9312.png)
 
2. Click **Select or create response**.
 
 ![image](https://rasa.com/docs/assets/images/welcome-flow-create-message-936c453aba68552d8e9febef9772ce76.png)
 
3. In the modal, select **Create new response**.
 
 ![image](https://rasa.com/docs/assets/images/create-new-response-89f527039b6462742db09079d6cdcb1b.png)
 
4. Enter the response name and the text.
 
 ![image](https://rasa.com/docs/assets/images/welcome-flow-utter-greet-dd2f1350418826cef889ef58250f81af.png)
 
5. The **Use contextual response rephraser** option is off by default. This feature enables the assistant to rephrase its responses by prompting an LLM, based on the conversation's context. It helps make the assistant's responses feel more organic and conversational.
 
 Read More
 
 When you create a new response, there’s an option called "Use Contextual Response Rephraser" that automatically follows a general setting (called `rephrase_all`) defined in the system’s configuration file.
 
 If you leave this option as it is, the system won’t assign a specific rephrasing setting just for that response—it will just use the general setting.
 
 However, if you decide to change this option to something different, the system will remember your choice and apply it specifically to that response.
 
 No matter what you choose, the interface will always show you whether rephrasing is turned on or off, based on either your specific choice (if you made one) or the general setting.
 
 ![image](https://rasa.com/docs/assets/images/money-transfer-rephraser-7a39129eabb753cb8cac7851b63bff29.png)
 
6. If you prefer full control over responses, but still want them to sound more natural and less repetitive, you can manually add as many variations as you want by clicking the **Add variation** button.
 
 ![image](https://rasa.com/docs/assets/images/utter-greet-add-variations-4fa4014c084b8db3a75e80684e1b55c5.png)
 
7. You can edit or delete previously created responses by clicking on the drop-down menu and selecting the **Manage responses** option.
 
 ![image](https://rasa.com/docs/assets/images/manage-messages-dc1795e48288a8bdd1121a9a54bd745a.png)
 

## Links[​](https://rasa.com/docs/studio/build/flow-building/sending-messages/#links 'Direct link to Links')

Within the response, you can provide users with links to extend your assistant's functionality.

![image](https://rasa.com/docs/assets/images/tya-links-4b9e21ec97414393f9f4d3a64f95052c.png)

### How to add links[​](https://rasa.com/docs/studio/build/flow-building/sending-messages/#how-to-add-links 'Direct link to How to add links')

1. Click **Add link** in your response.
 
 ![image](https://rasa.com/docs/assets/images/add-link-2d781a149030eba0ba47d09734e978ec.png)
 
2. Choose between reusing an existing link or creating a new one. If you create a new link, enter a title for it. This title will be the text displayed on the link for users to click.
 
 ![image](https://rasa.com/docs/assets/images/add-link-title-b257e84abbfd191863f71387616afa52.png)
 
3. Enter the link itself. Links can be of the following types:
 
 - **External URL**, e.g., `https://rasa.com`
 - **Internal link** within your application, e.g., `myapp://rasa/account?login`
 - **Phone call** or **email link**, e.g., `tel:+11111111111` or `mailto:hi@rasa.com`
 
 ![image](https://rasa.com/docs/assets/images/add-link-types-e15b1ae11cb395895d243eedd2c5189a.png)
 
4. Continue clicking **Add link** and fill in the information for as many links as you need.
 
 ![image](https://rasa.com/docs/assets/images/add-more-links-9286c884e1c111b7365fd3453ef18654.png)
 
5. To reuse an existing link, choose **Select existing link** and pick the link you'd like to reuse from the dropdown menu.
 
 ![image](https://rasa.com/docs/assets/images/reuse-link-e38ec5c14da5705bc48b30183faf1fea.png)
 
6. Use the icons next to the links to edit or delete them from the response.
 
 ![image](https://rasa.com/docs/assets/images/edit-delete-link-b05fff3af404b162f41835904193347b.png)
 

### Reordering links[​](https://rasa.com/docs/studio/build/flow-building/sending-messages/#reordering-links 'Direct link to Reordering links')

You can drag and drop links directly in the response editor — giving you full control over the order in which options are shown to users.

![image](https://rasa.com/docs/assets/images/reorder-links-eb617aa4df357a9880364ae370c42ef2.png)

## Referencing slot values in a response[​](https://rasa.com/docs/studio/build/flow-building/sending-messages/#referencing-slot-values-in-a-response 'Direct link to Referencing slot values in a response')

This feature enables you to utilize previously collected user information, such as their name or age. It also allows you to confirm user-provided input by incorporating a slot value directly into your response.

1. **Identify the slot**: First, determine which specific [slot](https://rasa.com/docs/studio/build/flow-building/collect/#slots) you want to reference. Understand the flows where this slot is used and the factors that can influence its values. If a slot is utilized across multiple flows, you will see the following alert:
 
 ![image](https://rasa.com/docs/assets/images/Untitled-af951731c792180ed272ae7420ad7e00.png)
 
2. **Insert the slot**: To include the slot in your response, enclose the slot name within curly brackets at the desired position. For example, if you have a slot named "user\_name" and you want to greet the user by their name, your response might look like this:
 
 ```
    Hello, {user_name}! How can I assist you today?
    ```
 
3. **Test and improve**: After completing the training, go to the [Try Your Assistant](https://rasa.com/docs/studio/test/try-your-assistant/) page to evaluate how the assistant handles the response. In the assistant’s response, the slot value should dynamically replace the placeholder with the actual value stored. For example, if the user's name is "John" the response should read: "Hello, John! How can I assist you today?
 

## Custom responses[​](https://rasa.com/docs/studio/build/flow-building/sending-messages/#custom-responses 'Direct link to Custom responses')

Custom responses allow you to go beyond simple text and create rich, interactive experiences tailored to your users and channels. You can use any YAML structure to define a custom response — from carousels and cards to events or metadata your frontend channel understands.

1. When editing a response, switch the tab to Custom YAML and add the code of your custom component. You can combine plain text and custom YAML in the same response if needed.
 
 ![image](https://rasa.com/docs/assets/images/custom-yaml-9131cf83cdfcf83ac9a3ca9073d3df40.png)
 
2. When testing in [Try your assistant](https://rasa.com/docs/studio/test/try-your-assistant/), Rasa sends custom components as JSONs. Styling and rendering are handled entirely by your frontend or messaging channel.
 
 ![image](https://rasa.com/docs/assets/images/custom-68e760bcb3b84521e3dd30d043bef733.png)
 

- [How to create a Message step](https://rasa.com/docs/studio/build/flow-building/sending-messages/#how-to-create-a-message-step)
- [Links](https://rasa.com/docs/studio/build/flow-building/sending-messages/#links)
 - [How to add links](https://rasa.com/docs/studio/build/flow-building/sending-messages/#how-to-add-links)
 - [Reordering links](https://rasa.com/docs/studio/build/flow-building/sending-messages/#reordering-links)
- [Referencing slot values in a response](https://rasa.com/docs/studio/build/flow-building/sending-messages/#referencing-slot-values-in-a-response)
- [Custom responses](https://rasa.com/docs/studio/build/flow-building/sending-messages/#custom-responses)