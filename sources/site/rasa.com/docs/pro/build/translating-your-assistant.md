# Source: https://rasa.com/docs/pro/build/translating-your-assistant

On this page

Supporting your users in their preferred language helps create more engaging, accessible, and inclusive assistants. Rasa enables you to easily build multilingual AI assistants that connect naturally with a global user base.

This page explains how you can set up multilingual capabilities in Rasa — from configuring supported languages and translating responses and flow names, to dynamically generating language-specific responses using the Response Rephraser.

---

## Configuring Languages in Rasa Pro[​](https://rasa.com/docs/pro/build/translating-your-assistant/#configuring-languages-in-rasa-pro 'Direct link to Configuring Languages in Rasa Pro')

For full instructions on configuring your assistant’s languages, including setting the primary language and supported additional languages, please refer to our [Configuring Your Assistant](https://rasa.com/docs/pro/build/configuring-assistant/#language) guide.

---

## Response Translations[​](https://rasa.com/docs/pro/build/translating-your-assistant/#response-translations 'Direct link to Response Translations')

You can manage multilingual responses easily by using the `translation` key within each response. You can define a single response and keep all translations organized in one place.

- Simple response
- Response with buttons

Here's how you can create a simple multilingual response:

domain.yml

```
responses:  utter_ask_help:    - text: "How can I help you?"      translation:        es: "¿En qué puedo ayudarte?"        fr: "Comment puis-je vous aider ?"        de: "Wie kann ich Ihnen helfen?"
```

This example shows a multilingual response with buttons:

domain.yml

```
responses:  utter_select_color:    - text: "Choose your favorite color:"      translation:        es: "Elige tu color favorito:"        fr: "Choisissez votre couleur préférée:"        de: "Wählen Sie Ihre Lieblingsfarbe:"      buttons:        - title: "Red"          payload: '/choose_color{"color":"red"}'          translation:            es:              title: "Rojo"              payload: '/choose_color{"color":"red"}'            fr:              title: "Rouge"              payload: '/choose_color{"color":"red"}'            de:              title: "Rot"              payload: '/choose_color{"color":"red"}'        - title: "Blue"          payload: '/choose_color{"color":"blue"}'          translation:            es:              title: "Azul"              payload: '/choose_color{"color":"blue"}'            fr:              title: "Bleu"              payload: '/choose_color{"color":"blue"}'            de:              title: "Blau"              payload: '/choose_color{"color":"blue"}'
```

You can apply this translation syntax to any text-based content your assistant sends to the user, including standard messages, button labels, and link descriptions. This keeps your multilingual assistant consistent, easy to manage, and clearly organized.

---

## Translating Flow Names[​](https://rasa.com/docs/pro/build/translating-your-assistant/#translating-flow-names 'Direct link to Translating Flow Names')

When your assistant handles common [conversation patterns](https://rasa.com/docs/reference/primitives/patterns/) (like confirming or canceling flows), it references your flow names directly. To ensure system messages use the correct localized names, you can define translations directly within your flows configuration.

Here's an example of translating a simple "Greet User" flow in your `flows.yml`:

flows.yml

```
flows:  greet_user:    name: "Greet User"    description: "Greets the user at the start of the conversation."    translation:      es:        name: "Saludar al usuario"      fr:        name: "Saluer l'utilisateur"      de:        name: "Benutzer begrüßen"    steps:      - action: utter_greet
```

By including these translations, your assistant automatically uses the correct localized name of the flow when handling built-in patterns like cancellations, confirmations, and clarifications.

---

## Enhancing Rephraser for Multilingual Output[​](https://rasa.com/docs/pro/build/translating-your-assistant/#enhancing-rephraser-for-multilingual-output 'Direct link to Enhancing Rephraser for Multilingual Output')

The [**Response Rephraser**](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/) dynamically rewords your assistant's messages during conversation by leveraging a Large Language Model (LLM). Its output language depends on the current conversation's language, determined by the assistant's built-in `language` slot.

Here's the default prompt used by the Response Rephraser:

rephraser\_prompt.py

```
The following is a conversation with an AI assistant.The assistant is helpful, creative, clever, and very friendly.Rephrase the suggested AI response staying close to the original message and retaining its meaning. Use simple {{language}}.Context / previous conversation with the user:{{history}}{{current_input}}Suggested AI Response: {{suggested_response}}Rephrased AI Response:
```

Because the prompt contains `"Use simple {{language}}"`, the Rephraser automatically generates responses in the language currently set in the user's conversation. This allows your assistant to dynamically provide localized responses whenever the detected or selected language changes.

---

## Built-In Language Slot[​](https://rasa.com/docs/pro/build/translating-your-assistant/#built-in-language-slot 'Direct link to Built-In Language Slot')

Rasa provides a built-in `language` slot to manage the current conversation language.

warning

`language` is a private keyword used by Rasa internally to manage language - to avoid naming collisions, you cannot use this keyword for custom slots. Please see more details on migration below.

This slot ensures that your assistant only uses supported language codes defined in your configuration (`language` and `additional_languages` in `config.yml`).

It is your responsibility to set the `language` slot based on your users' preferences. You can choose how to determine the appropriate language — such as from user-provided preferences, browser settings, detected IP geolocation, or any other relevant parameter.

For example, you can set the language when the conversation session starts:

```
class ActionSessionStart(Action):    def name(self) -> str:        return "action_session_start"    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: dict) -> list:        # Determine the user's language (implement your logic here)        language = get_assistant_language_code()        return [SlotSet("language", language)]
```

---

## Validation[​](https://rasa.com/docs/pro/build/translating-your-assistant/#validation 'Direct link to Validation')

To ensure that your assistant's multilingual setup is consistent and correct, Rasa provides dedicated multilingual validation commands. Run these commands from your terminal to validate your language configurations and translations:

- **General validation** - checks domain configuration, stories, rules, slots, and other related files during training:
 
 ```
    rasa data validate
    ```
 
 This command provides aggregate counts, generating one consolidated warning for missing response translations and another consolidated warning for missing flow translations.
 
- **Translations validation** - specifically verifies that translations are complete and correctly formatted:
 
 ```
    rasa data validate translations
    ```
 
 This command shows individual warnings for each missing translation.
 

#### Validation Warnings[​](https://rasa.com/docs/pro/build/translating-your-assistant/#validation-warnings 'Direct link to Validation Warnings')

When rephraser is enabled globally you will not receive these warnings as it the LLM will generate the translations during conversation runtime. When rephraser is disabled on a global or response level - the command will output warning for the following scenarios to alert you to missing translated content:

- A response is missing translated content for one of your configured languages
- A flow name translations is missing for one of your configured languages

#### Errors[​](https://rasa.com/docs/pro/build/translating-your-assistant/#errors 'Direct link to Errors')

There are a few cases where validation will trigger an error with translations to ensure that your configuration is compatible:

- Your default `language` appearing within the `additional_languages`.
- You have configured language with an invalid or incorrectly formatted language code.

---

## Migrations[​](https://rasa.com/docs/pro/build/translating-your-assistant/#migrations 'Direct link to Migrations')

If you have an existing assistant that uses conditional responses to manage language and would like to migrate to use Rasa's multi-language features - please reach out to the Rasa team and your dedicated support who can assist you with automating the migration.

To migrate from the conditional responses format to the new translation format, you can follow the example below. In this example we consider that the assistant's default language is English.

- Conditional Responses
- Translation Format

Old example using conditions:

```
utter_ask_callback_wanted:  - text: "Would you like to arrange a callback?"    condition:      - type: slot        name: language        value: "English"  - text: "Möchten Sie einen Termin vereinbaren?"    condition:      - type: slot        name: language        value: "German"  - text: "Souhaitez-vous fixer un rendez-vous?"    condition:      - type: slot        name: language        value: "French"  - text: "Would you like to arrange a callback?"
```

New example with translations:

```
utter_ask_callback_wanted:  - text: "Would you like to arrange a callback?"    # Default English text    translation:      de: "Möchten Sie einen Termin vereinbaren?"      fr: "Souhaitez-vous fixer un rendez-vous?"
```

- [Configuring Languages in Rasa Pro](https://rasa.com/docs/pro/build/translating-your-assistant/#configuring-languages-in-rasa-pro)
- [Response Translations](https://rasa.com/docs/pro/build/translating-your-assistant/#response-translations)
- [Translating Flow Names](https://rasa.com/docs/pro/build/translating-your-assistant/#translating-flow-names)
- [Enhancing Rephraser for Multilingual Output](https://rasa.com/docs/pro/build/translating-your-assistant/#enhancing-rephraser-for-multilingual-output)
- [Built-In Language Slot](https://rasa.com/docs/pro/build/translating-your-assistant/#built-in-language-slot)
- [Validation](https://rasa.com/docs/pro/build/translating-your-assistant/#validation)
- [Migrations](https://rasa.com/docs/pro/build/translating-your-assistant/#migrations)