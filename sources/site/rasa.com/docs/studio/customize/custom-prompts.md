# Source: https://rasa.com/docs/studio/customize/custom-prompts

On this page

This guide will walk you through the process of customizing prompt templates in Rasa Studio.

By the end, you’ll know how to view and edit assistant prompts, adjust responses to better match your assistant’s tone, and manage prompt versions — all directly from the Studio interface.

### What Are Prompts?[​](https://rasa.com/docs/studio/customize/custom-prompts/#what-are-prompts 'Direct link to What Are Prompts?')

Prompts are short pieces of text that guide your assistant for specific tasks. Currently in Rasa there are three different types of prompts:

1. **Rephraser Prompts:** Control tone and phrasing for assistant's using LLM-Rephraser. 
 _For the full reference [Read the Reference](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/#prompt)_.
2. **Enterprise Search Prompt:** Specify how your assistant retrieves and presents information from knowledge sources. 
 _For the full reference [read here](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/#customization)_.
3. **Command Generator Prompt (Advanced):** Guide how your assistant translates user input and context into actions. 
 _For more guidance on when to customize [read here](https://rasa.com/docs/pro/customize/command-generator/#customizing-the-prompt-template)_. _For the full reference [read here](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-the-prompt)_.

### Who Can Edit Prompts?[​](https://rasa.com/docs/studio/customize/custom-prompts/#who-can-edit-prompts 'Direct link to Who Can Edit Prompts?')

Users with the **Content Editor** role can edit prompts for individual messages. For editing global prompts you need the **Developer** role assigned.

For more details on roles and how to set them up [see the docs here](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#roles-overview).

## How to Edit Prompts in Studio[​](https://rasa.com/docs/studio/customize/custom-prompts/#how-to-edit-prompts-in-studio 'Direct link to How to Edit Prompts in Studio')

### Editing a Prompt for a Specific Response[​](https://rasa.com/docs/studio/customize/custom-prompts/#editing-a-prompt-for-a-specific-response 'Direct link to Editing a Prompt for a Specific Response')

1. Go to any message in the CMS.
2. Make sure that **Rephraser** is enabled for that message.
3. Update the text in the prompt editor.

![img](https://rasa.com/docs/assets/images/response-rephraser-prompt-2efb896516a507400a9794e7bc2cc14a.png)

This is helpful when you want to tweak the tone or phrasing for a single message.

### Editing Global Prompts[​](https://rasa.com/docs/studio/customize/custom-prompts/#editing-global-prompts 'Direct link to Editing Global Prompts')

1. Open your assistant’s **Settings**.
2. For Command Generator or Enterprise Search go to the **Configuration** tab. ![img](https://rasa.com/docs/assets/images/enterprise-search-prompt-973895672d7b58312e19ca9dbaa5c619.png) 
 For Rephraser go to **Assistant Language**. ![img](https://rasa.com/docs/assets/images/global-rephraser-prompt-548c91ee1a3ae41cc83f06cf1c9eda94.png)
3. Expand the prompt.
4. Make your changes and click **Save**.

✅ You’ll see a small label when the prompt is customized with details about the last editor.

A Note on Editing Command Generator Prompts

Altering your command generator **fundamentally changes your assistant behaviour** and can have unpredictable side effects. We strongly recommend that you ensure that you have [end to end test coverage](https://rasa.com/docs/pro/testing/evaluating-assistant/) in place **before** you make changes to this prompt to ensure that your assistant continues to behave as expected.

### Resetting a Prompt to Default[​](https://rasa.com/docs/studio/customize/custom-prompts/#resetting-a-prompt-to-default 'Direct link to Resetting a Prompt to Default')

_To undo your changes:_

1. Go to the customized prompt.
2. Click the **Reset to Default** icon.
3. Confirm that you want to remove the customization.

![img](https://rasa.com/docs/assets/images/reset-prompt-a12435f74bd9567d597f1bd1231003ae.png)

✅ The default prompt will be restored and used moving forward.

## What's Next?[​](https://rasa.com/docs/studio/customize/custom-prompts/#whats-next 'Direct link to What's Next?')

Customizing prompts is a powerful way to shape how your assistant communicates and behaves. Once you’ve made changes, we recommend running end to end tests and trying out your assistant behaviour in Inspector.

- [What Are Prompts?](https://rasa.com/docs/studio/customize/custom-prompts/#what-are-prompts)
- [Who Can Edit Prompts?](https://rasa.com/docs/studio/customize/custom-prompts/#who-can-edit-prompts)
- [How to Edit Prompts in Studio](https://rasa.com/docs/studio/customize/custom-prompts/#how-to-edit-prompts-in-studio)
 - [Editing a Prompt for a Specific Response](https://rasa.com/docs/studio/customize/custom-prompts/#editing-a-prompt-for-a-specific-response)
 - [Editing Global Prompts](https://rasa.com/docs/studio/customize/custom-prompts/#editing-global-prompts)
 - [Resetting a Prompt to Default](https://rasa.com/docs/studio/customize/custom-prompts/#resetting-a-prompt-to-default)
- [What's Next?](https://rasa.com/docs/studio/customize/custom-prompts/#whats-next)