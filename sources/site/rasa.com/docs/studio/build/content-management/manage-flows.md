# Source: https://rasa.com/docs/studio/build/content-management/manage-flows

On this page

This guide will show you how to manage flows using Rasa Studio. You’ll learn to navigate, edit, delete, and organize flows, helping you maintain a clean and efficient workspace for your assistant.

## Finding your flows[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#finding-your-flows 'Direct link to Finding your flows')

There are 2 ways to navigate through flows:

1. The flow table: It is the first thing you see when you visit the Flow builder page. Here, you can view the description and state of each flow, sorted by the date of the last edit.

![image](https://rasa.com/docs/assets/images/flow-table-ed29a6f09cd97c94e5fed2b87d830e9d.png)

2. The flow list: To access this list, click on the "See flows" button in the upper left corner of the canvas.

![image](https://rasa.com/docs/assets/images/flow-list-627a4484d73a86a379df1c7b3485248b.png)

## Editing flows[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#editing-flows 'Direct link to Editing flows')

1. To edit a flow’s description, open the flow and select the **Start** step. You can update the description in the side panel on the right.

![image](https://rasa.com/docs/assets/images/edit-flows-295b3a238d84bb03f0653f4a143d25a7.png)

2. To rename a flow or change its ID, click the pencil icon next to the flow’s ID.

![image](https://rasa.com/docs/assets/images/edit-flows-2-66f1470ace06e9b976d489c73e6025fc.png)

### (Optional) Translating flow names[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#optional-translating-flow-names 'Direct link to (Optional) Translating flow names')

Assistants will sometimes refer to flows in conversation when cancelling or clarifying an action. If you have a multi-language assistant you can provide a user friendly flow name that will be used in this case and localize for your supported languages.

![image](https://rasa.com/docs/assets/images/flow-name-translation-f71cab30465985784b502d294bbf4970.png)

## Duplicating flows[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#duplicating-flows 'Direct link to Duplicating flows')

You can duplicate an existing flow to reuse its logic:

1. Hover over the flow name and click **Duplicate**.

![image](https://rasa.com/docs/assets/images/duplicate-flows-edb0cf2a2a4442d0897ff13a7232ea4e.png)

2. A copy of the flow will appear with “Copy” in the name and description — you can rename and adjust it as needed.

![image](https://rasa.com/docs/assets/images/duplicate-flows-2-c38f4b9f71e190a96622a56a00d7cc04.png)

## Deleting flows[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#deleting-flows 'Direct link to Deleting flows')

1. To delete a flow, hover over the flow name and click **Delete**.

![image](https://rasa.com/docs/assets/images/delete-flows-61f77ea9faec62467b81bbff84a519ee.png)

2. In the modal that opens, confirm deletion by typing the flow name.

![image](https://rasa.com/docs/assets/images/delete-flows2-f123058efe0359c2a11b7f8224d65453.png)

## Searching flows[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#searching-flows 'Direct link to Searching flows')

You can search for flows by typing their names into the search bar.

![image](https://rasa.com/docs/assets/images/search-flows-01c6342b97a1173ae48a7b87a5cef414.png)

## Deleting flow steps[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#deleting-flow-steps 'Direct link to Deleting flow steps')

Deleting a step other than Logic doesn’t remove the elements used in that step. For example, deleting a reply step only removes it from this section of the flow; the reply used in that step remains intact.

![image](https://rasa.com/docs/assets/images/logic-delete-branch-9c54be53be7a3c519f8aced3f595a78a.png)

## Adding step labels[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#adding-step-labels 'Direct link to Adding step labels')

When adding a step, its type dictates the displayed information. Details added to the step are used to present it. If the automatic display isn’t helpful, you can add labels to the steps to make them more descriptive and easier to navigate.

![image](https://rasa.com/docs/assets/images/add-label-91728a6198219999ab43cf2c5c424c9c.png)

## Navigating flow canvas[​](https://rasa.com/docs/studio/build/content-management/manage-flows/#navigating-flow-canvas 'Direct link to Navigating flow canvas')

Here are the navigation options supported in Flow Builder:

- Zoom in using keyboard and mouse
- Zoom out using keyboard and mouse
- Adjust the screen display size
- Move around the canvas using keyboard and mouse

![image](https://rasa.com/docs/assets/images/navigation-3c24771ec2afced107dab37f9ceb93bb.png)

- [Finding your flows](https://rasa.com/docs/studio/build/content-management/manage-flows/#finding-your-flows)
- [Editing flows](https://rasa.com/docs/studio/build/content-management/manage-flows/#editing-flows)
 - [(Optional) Translating flow names](https://rasa.com/docs/studio/build/content-management/manage-flows/#optional-translating-flow-names)
- [Duplicating flows](https://rasa.com/docs/studio/build/content-management/manage-flows/#duplicating-flows)
- [Deleting flows](https://rasa.com/docs/studio/build/content-management/manage-flows/#deleting-flows)
- [Searching flows](https://rasa.com/docs/studio/build/content-management/manage-flows/#searching-flows)
- [Deleting flow steps](https://rasa.com/docs/studio/build/content-management/manage-flows/#deleting-flow-steps)
- [Adding step labels](https://rasa.com/docs/studio/build/content-management/manage-flows/#adding-step-labels)
- [Navigating flow canvas](https://rasa.com/docs/studio/build/content-management/manage-flows/#navigating-flow-canvas)