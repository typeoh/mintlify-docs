# Source: https://rasa.com/docs/studio/build/actions/create-a-custom-action

On this page

This guide will show you how to use **Custom Actions** in Rasa Studio to integrate data from sources like databases or APIs. Custom actions allow your assistant to fetch information, update records, and connect with external tools, improving the quality and relevance of conversations.

### What are Custom Actions?

Custom Actions allow your assistant to execute any code you define as part of a flow. This could include tasks like retrieving account details, booking a meeting, or retrieving data from external systems.

## How to create Custom action[​](https://rasa.com/docs/studio/build/actions/create-a-custom-action/#how-to-create-custom-action 'Direct link to How to create Custom action')

1. Select the "Custom Action" step from the menu. ![img](https://rasa.com/docs/assets/images/money-transfer-add-custom-action-bdf35b918eacd387e41e972ed6d83027.png)
 
2. Select an existing custom action or click on "Create custom action" to create a new one. ![img](https://rasa.com/docs/assets/images/money-transfer-create-custom-action-a0c62b210bde82a44d8a88189b3e8338.png)
 
3. In the window that appears, enter the name of the custom action, ensuring it exactly matches the one implemented in your Docker container. In the "Description field "provide a detailed description of what the action does in free form. Note: This description will be read by whoever implements this action so it should describe what this action should achieve, what slot(s) it should use and what slot value to return in which case.
 
 ![img](https://rasa.com/docs/assets/images/money-transfer-new-custom-action-a65be48d5f24ef1e415e7fc408855e3e.png)
 
4. You can edit the name or description of a custom action, or delete it, by selecting "Manage custom actions" from the drop-down menu. Note that when you use a custom action in many places, when you edit or delete this action, it will also affect other flows and steps as well. ![img](https://rasa.com/docs/assets/images/manage-custom-actions-85f11ac375d6028c34e73edf96acd081.png)
 

- [How to create Custom action](https://rasa.com/docs/studio/build/actions/create-a-custom-action/#how-to-create-custom-action)