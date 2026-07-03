# Source: https://rasa.com/docs/studio/build/set-up-your-assistant

On this page

This guide will help guide you through some best practices for configuring your assistants in Studio. By the end of it, you'll have configured your assistant and be ready to start building amazing conversational experiences. Let's dive in!

## Step 1: Log In to Rasa Studio[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#step-1-log-in-to-rasa-studio 'Direct link to Step 1: Log In to Rasa Studio')

1. Log in to Rasa Studio as a user with the `superuser` or `developer` role. 
 For more details on roles and what they can do, check out our [user role guide](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/).  ![image](https://rasa.com/docs/assets/images/log-in-developer-5b60879ecd2227ca1bfb6e7bbb8c2840.png)
 
2. Create or select an assistant you want to configure. ![image](https://rasa.com/docs/assets/images/create-assistant-project-a3dc6725e19c1668b3418d66aacbcdfd.png)
 

## Step 2: Configure Your Assistant[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#step-2-configure-your-assistant 'Direct link to Step 2: Configure Your Assistant')

1. Navigate to the "Assistant settings" page. ![image](https://rasa.com/docs/assets/images/assistant-settings-tab-f6dbeb7c5d8fe43c6326b2740e6967ce.png)
 
2. Click on the "General settings" tab. Here, you will find general information for your assistant like Assistant name and Assistant API ID. These settings aren't editable. ![image](https://rasa.com/docs/assets/images/general-settings-f8698526e8da78a835cb379ff50626b4.png)
 
3. Click on the "Configuration" tab. Here, you will find editors for the default `config.yml` and `endpoints.yml` files." ![image](https://rasa.com/docs/assets/images/assistant-settings-c32f57485dac56bc8ac26172bd28bf1a.png)
 

Additionally, there is an option to "Always include the following flows in the prompt." This feature ensures that selected flows are always considered by the LLM during dialogue management, regardless of other constraints. This is particularly useful for flows that are crucial to the assistant's functionality and need to be readily available during conversations. For more details on flow retrieval, please refer to the [Retrieving Relevant Flows documentation](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows).

### Configuration Files[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#configuration-files 'Direct link to Configuration Files')

You can configure LLMs and their deployments, embeddings, NLU, Core, and Action server in these files.

#### Config[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#config 'Direct link to Config')

The `config.yml` file is central to configuring your assistant's behavior. It allows you to customize:

- NLU (Natural Language Understanding) components
- Core model settings
- Training pipeline
- Policies
- LLM (Large Language Model) configurations and deployments
- Embedding settings

#### Endpoints[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#endpoints 'Direct link to Endpoints')

The `endpoints.yml` file specifies various service endpoints used by Rasa Studio, including:

- [Tracker store](https://rasa.com/docs/reference/integrations/tracker-stores/)
- [Event broker](https://rasa.com/docs/reference/integrations/event-brokers/)
- [NLG (Natural Language Generation) server](https://rasa.com/docs/reference/integrations/nlg/)
- [Action server](https://rasa.com/docs/action-server/)

## Optional: Configure Additional Languages[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#optional-configure-additional-languages 'Direct link to Optional: Configure Additional Languages')

Configuring additional languages enables your assistant to deliver localized content and offer a better experience for users who speak different languages. It also allows you to manage and translate responses directly in Studio.

1. Navigate to the Assistant Languages tab ![Navigate to assistant language settings](https://rasa.com/docs/assets/images/assistant-settings-tab-language-b186f73d150f6726c2ad330e708f3a78.png)

- From the sidebar menu, select “Assistant language.”
- This opens the panel where you can set your assistant’s default and additional languages.

2. Set the Default Language

- You can update your default language from this tab. This language is the primary language your assistant uses and will act as the fallback if translations are missing or errors occur.

🚨 Destructive Action

Be careful when updating the default language—you’ll need to provide a full set of default content for the newly selected language before being able to train.

3. Add Additional Languages

![Configure additional languages in Studio](https://rasa.com/docs/assets/images/assistant-settings-language-c7cf869211290f745dd9001c83414f9b.png)

- Select the additional languages you want your assistant to support _(e.g., start typing “Port” to filter Portuguese variants)_.
- Select the appropriate one(s).
- Save your changes.

note

Changes to language settings automatically update your config—no manual changes are needed.

4. (Optional) Add a Custom Language

- If your target language isn’t listed, click “Add custom language” at the bottom of the list.
- This allows you to manually define a language key and name.

#### About Language Codes[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#about-language-codes 'Direct link to About Language Codes')

Rasa uses the [BCP 47 standard](https://www.rfc-editor.org/info/bcp47) for encoding languages. This standard supports a wide range of use cases across global markets.

You can find specific language codes in the [IANA Language Subtag Registry](https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry).

#### 📌 Advanced Tips:[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#-advanced-tips 'Direct link to 📌 Advanced Tips:')

- **Custom dialects:** Use `x-<language-code>` for private use (e.g., `x-de-formal` for formal language variants).
- **Regional variants:** We recommend to keep your language codes simple - however, if you have country level localization - use region-prefixed codes like `en-US` 🇺🇸 or `en-UK` 🇬🇧 to support localized behavior.

These formats ensure consistent behavior across Rasa Studio and other parts of the Rasa stack.

## Further Reading[​](https://rasa.com/docs/studio/build/set-up-your-assistant/#further-reading 'Direct link to Further Reading')

For more detailed technical information on `config.yml` and `endpoints.yml` files, refer to the [Rasa documentation on model configuration](https://rasa.com/docs/reference/config/overview/).

- [Step 1: Log In to Rasa Studio](https://rasa.com/docs/studio/build/set-up-your-assistant/#step-1-log-in-to-rasa-studio)
- [Step 2: Configure Your Assistant](https://rasa.com/docs/studio/build/set-up-your-assistant/#step-2-configure-your-assistant)
 - [Configuration Files](https://rasa.com/docs/studio/build/set-up-your-assistant/#configuration-files)
- [Optional: Configure Additional Languages](https://rasa.com/docs/studio/build/set-up-your-assistant/#optional-configure-additional-languages)
- [Further Reading](https://rasa.com/docs/studio/build/set-up-your-assistant/#further-reading)