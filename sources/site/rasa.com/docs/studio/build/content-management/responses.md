# Source: https://rasa.com/docs/studio/build/content-management/responses

The responses section of the CMS is a centralised place for content managers and flow builders to create and manage all the responses used in an assistant, including adding specific response variations based on one or more slot values, enhancing personalization and flexibility in replies.

This feature allows users to easily build multi-lingual assistants or add segmentation to their assistants based on different criteria.

Access this feature from Responses navigation item under CMS on the navigation menu.

![image](https://rasa.com/docs/assets/images/responses-cms-b1cd81a8a05d8e46175b08485786785a.png)

### How to Create a Response[​](https://rasa.com/docs/studio/build/content-management/responses/#how-to-create-a-response 'Direct link to How to Create a Response')

1. Click on the “Create response” button on the top right of the screen.

![image](https://rasa.com/docs/assets/images/responses-01-a1ff1377d34b26decf5fb709613bacdd.png)

2. Select the Response type depending on where it will be used. You can choose from Message, Collect information or Invalid input responses types.

![image](https://rasa.com/docs/assets/images/responses-02-e0b9a3e6ec14b7a6a17d09fe568cb95a.png)

3. Type in a name and the standard response text, and buttons if it's a Collect information response.

![image](https://rasa.com/docs/assets/images/responses-03-ce24ec6e1b39e50758e2f87e54bb5e9b.png)

4. The "Use Contextual Response Rephraser" option is disabled by default. This feature enables the assistant to rephrase its responses by prompting an LLM, based on the conversation's context. It helps make the assistant's responses feel more organic and conversational.

If you prefer full control over responses, you can uncheck the rephraser and manually add message variations.

Read More

When creating a new response, the "Use Contextual Response Rephraser" option defaults to the global `rephrase_all` value set in the endpoints.yml file.

If you keep this default setting, the per-response rephrasing setting will not be explicitly set for the message.

If you change this option to a different value than the default, the per-response rephrasing setting will be set to your chosen value.

The UI will always display the computed state of rephrasing, taking into account both the per-response setting (if set) and the global `rephrase_all` value.

![image](https://rasa.com/docs/assets/images/responses-04-da2045638fe93124e25ad64c60b27ec3.png)

### Buttons and Links[​](https://rasa.com/docs/studio/build/content-management/responses/#buttons-and-links 'Direct link to Buttons and Links')

Each response can include interactive elements: **buttons** for Collect Information responses and links for **Message** responses. To add them:

1. Click **Add button** or **Add link** in your response.

![image](https://rasa.com/docs/assets/images/responses-add-button1-ce201cf2d6bf8d6f3a236fff24f56992.png)

2. In the modal that opens, choose to reuse an existing button or link, or create a new one. Learn more about [buttons](https://rasa.com/docs/studio/build/flow-building/collect/#buttons) and [links](https://rasa.com/docs/studio/build/flow-building/sending-messages/#links).

![image](https://rasa.com/docs/assets/images/responses-add-button2-287f94fe5b479daa5ea2d58a39a6331f.png)

3. Repeat **Add button** or **Add link** and fill in the details for as many options as needed.

![image](https://rasa.com/docs/assets/images/responses-add-button3-b13a5974a0adab42354ecca02592ba9d.png)

4. To reorder the options shown to the user, drag and drop them directly in the response editor.

![image](https://rasa.com/docs/assets/images/responses-add-button4-d24732ecb3dc59c47151704d83925e4d.png)

### Manage Responses[​](https://rasa.com/docs/studio/build/content-management/responses/#manage-responses 'Direct link to Manage Responses')

1. Click the **Edit** button under the response name to edit the name and type of the response. Or the **Delete** button to delete the response from the assistant.

![image](https://rasa.com/docs/assets/images/responses-05-c1e5ac3d1dd4c58376a526ba8a9f1b1b.png)

2. Click the **See flows** button to view a list of flows where the response is used.

![image](https://rasa.com/docs/assets/images/responses-06-0bf5a0f161f6a2c9fcf1153fc7ad1534.png)

3. From the list, you can navigate to a specific flow to view the context in which the response appears.

![image](https://rasa.com/docs/assets/images/responses-07-b39a32d6418cf36c9e448eb5b8513bbe.png)

### Conditional Response Variations[​](https://rasa.com/docs/studio/build/content-management/responses/#conditional-response-variations 'Direct link to Conditional Response Variations')

Use conditional response variations to customise your assistant. These conditional response variations will only be shown to the user when they meet the conditions set on the Conditions tab.

You can only create and manage conditional response variations from the Responses section on the CMS.

1. Click the “Add conditional response variation” button under the response in the navigation panel.

![image](https://rasa.com/docs/assets/images/responses-08-17e1bfb2463e98ac6bb5828e2d8150f3.png)

2. Add a name for the conditional response variation. You can decide to copy the standard response text to this conditional response variation by checking the checkbox.

![image](https://rasa.com/docs/assets/images/responses-09-de11f12490d5e6058bcab841eda4ef14.png)

3. You are in the “Response” tab, where you can add the response, buttons and any variations to the response.

![image](https://rasa.com/docs/assets/images/responses-10-6a796a673a82d91dd031340859e41db6.png)

4. Click on the “Conditions” tab, and add as many conditions as you need. Select a slot and the slot value on the dropdowns available.

![image](https://rasa.com/docs/assets/images/responses-11-4fd0a6432b8fa4279b7458230a3293c9.png)

5. Click the “Add condition” button to add more conditions.

![image](https://rasa.com/docs/assets/images/responses-12-96aff4926bb53e8e0c1ff906af92b52a.png)

### Custom Responses[​](https://rasa.com/docs/studio/build/content-management/responses/#custom-responses 'Direct link to Custom Responses')

Custom responses allow you to go beyond simple text and create rich, interactive experiences tailored to your users and channels. You can use any YAML structure to define a custom response — from carousels and buttons to event triggers or metadata your channel frontend understands.

1. When editing a response, just switch the tab to Custom YAML and add your custom component in YAML. You can combine plain text and custom YAML in the same response if needed.
 
 ![image](https://rasa.com/docs/assets/images/cms-custom-response-c7371a1618fd479e30a40f4425a55ae2.png)
 
2. When testing in [Try your assistant](https://rasa.com/docs/studio/test/try-your-assistant/), Rasa sends custom components as JSONs. Styling and rendering are handled entirely by your frontend or messaging channel.
 
 ![image](https://rasa.com/docs/assets/images/custom-68e760bcb3b84521e3dd30d043bef733.png)