# Source: https://rasa.com/docs/studio/build/content-management/version-control

On this page

Studio's new versioning feature is designed to enable multiple users to work on an assistant at the same time. **Flow Version Control** in Studio keeps track of your work as you build and allows you to review your changes, and if necessary, revert to a previous version.

This guide will lead you through Studio's version control features and how to manage your assistant changes with a team.

## View the Flow History[​](https://rasa.com/docs/studio/build/content-management/version-control/#view-the-flow-history 'Direct link to View the Flow History')

Flow History lets you view all changes made to a flow by any user.

1. Open your flow in Studio.
 
2. Click the **“History”** button located in the flow editor.
 
 ![Flow History Button](https://rasa.com/docs/assets/images/flow-history-button-4e593d04d368d89034d304f4f1bae7be.png)
 
 A panel will slide out from the left side of the screen, showing a detailed timeline of changes. This includes edits made by all users, so you always know who did what. ![Flow History Panel](https://rasa.com/docs/assets/images/flow-history-panel-c236a23f5a771ae81a7c2573482339f5.png)
 

## Save a Stable Version[​](https://rasa.com/docs/studio/build/content-management/version-control/#save-a-stable-version 'Direct link to Save a Stable Version')

Saving a stable version lets you restore a previous flow if needed. Additionally, it allows you to train your assistant using the last stable version of the flow instead of the current version with the latest changes. This ensures that team members can experiment with updates in the current version without impacting the stability of training and testing for others.

1. Once you have fully tested the flow and confirmed it is working as expected, click the **“Save stable version”** button in the editor.
 
 ![Save Stable Version](https://rasa.com/docs/assets/images/save-stable-version-6a36a4b67d0c772f6e8da2595550e916.png)
 
 **✨ Pro Tip ✨** Save a stable version after thoroughly testing your flow so your team can confidently use it for testing. Make sure to save a version before making major changes so you can easily revert to a stable state if needed.
 
2. Your saved version will appear in the Flow History panel, along with the time, date, and author of the save.
 
 ![Stable Version Created](https://rasa.com/docs/assets/images/stable-version-in-history-aeeffce467c7ea0ef0d10d0aedaa0119.png)
 

## Revert to a Previous Version[​](https://rasa.com/docs/studio/build/content-management/version-control/#revert-to-a-previous-version 'Direct link to Revert to a Previous Version')

If something goes wrong or you want to go back to a previous flow setup, reverting is simple.

1. In the Flow History panel, locate the version you want to restore.
 
2. Click the **“Revert”** icon next to that version.
 
 ![Revert to Version](https://rasa.com/docs/assets/images/restore-stable-version-0b3f9b20cdab273d3fa643443ecfc351.png)
 
3. Once reverted, the flow will be restored to the selected version, and a note will appear in the Flow History to track the event.
 
 ![Reverted to Version](https://rasa.com/docs/assets/images/reverted-version-b28e54b9e09794c3dc34e0b0fa901500.png)
 

- [View the Flow History](https://rasa.com/docs/studio/build/content-management/version-control/#view-the-flow-history)
- [Save a Stable Version](https://rasa.com/docs/studio/build/content-management/version-control/#save-a-stable-version)
- [Revert to a Previous Version](https://rasa.com/docs/studio/build/content-management/version-control/#revert-to-a-previous-version)