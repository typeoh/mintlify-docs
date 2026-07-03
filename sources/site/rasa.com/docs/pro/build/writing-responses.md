# Source: https://rasa.com/docs/pro/build/writing-responses

On this page

Responses are the messages your assistant sends to end users. These can be simple text messages or more complex UI elements such as images and buttons. In CALM, a “response” is one of the fundamental building blocks your assistant uses to communicate information, ask clarifying questions, and guide the user.

- **Where:** You define responses as YAML entries (usually in a file such as `domain.yml` or `responses.yml`).
- **Usage:** When an assistant flow or pattern decides to “say” something, it references one of these response definitions.

Below are the core steps to define and use responses in your CALM assistant. For advanced options—like channel-specific messages, conditional responses, and dynamic button creation—refer to [Responses](https://rasa.com/docs/reference/primitives/responses/) in the Reference section.

### 1\. Define Your Responses[​](https://rasa.com/docs/pro/build/writing-responses/#1-define-your-responses 'Direct link to 1. Define Your Responses')

In CALM, you typically define all your responses under a `responses` key in your domain or responses file. Each response name starts with `utter_`. For example:

domain.yml

```
responses:  utter_greet:    - text: "Hi there!"  utter_bye:    - text: "See you next time!"
```

- **Purpose:** Each `utter_` response is a distinct piece of content your assistant can send to the user.
- **Where It’s Used:** You can reference the response by its name (e.g., `utter_greet`) in a flow step, or even in a custom action.

### 2\. Use Variables in Responses (Optional)[​](https://rasa.com/docs/pro/build/writing-responses/#2-use-variables-in-responses-optional 'Direct link to 2. Use Variables in Responses (Optional)')

If you want the assistant to include slot values or other dynamic content, enclose variable names in curly braces:

domain.yml

```
responses:  utter_greet:    - text: "Hey, {name}. How are you?"
```

- **How It Works**
 
 CALM will replace `{name}` with the value of the `name` slot (if it’s filled).
 
- **From Custom Actions**
 
 In a custom action, you can pass extra data to fill those variables:
 
 actions.py
 
 ```
    # ...dispatcher.utter_message(  template="utter_greet",  name="Sara")
    ```
 

### 3\. Provide Response Variations (Optional)[​](https://rasa.com/docs/pro/build/writing-responses/#3-provide-response-variations-optional 'Direct link to 3. Provide Response Variations (Optional)')

To keep your assistant more engaging, you can define multiple text entries for the same `utter_` response. CALM will randomly pick one at runtime:

domain.yml

```
responses:  utter_greet:    - text: "Hey, {name}. How are you?"    - text: "Hi there, {name}! Hope you’re well."
```

### 4\. Add Rich Content (Buttons, Images, etc.)[​](https://rasa.com/docs/pro/build/writing-responses/#4-add-rich-content-buttons-images-etc 'Direct link to 4. Add Rich Content (Buttons, Images, etc.)')

A response can go beyond text. You can add buttons, images, or custom JSON payloads for richer channels.

**Buttons Example**:

domain.yml

```
responses:  utter_ask_insurance_type:    - text: "Would you like motor or home insurance?"      buttons:        - title: "Motor insurance"          # Overwrites NLU with the /inform intent + entity          payload: '/inform{"insurance":"motor"}'        - title: "Home insurance"          payload: '/inform{"insurance":"home"}'
```

When the user clicks a button, it sends the specified payload directly to your assistant.

For more details—e.g., setting multiple slots or using `/SetSlots()`—see the [Reference → Responses → Buttons](https://rasa.com/docs/reference/primitives/responses/#buttons).

**Images Example**:

domain.yml

```
responses:  utter_cheer_up:    - text: "Here is something to cheer you up:"      image: "https://i.imgur.com/nGF1K8f.jpg"
```

### 5\. Use Responses in Flows or Actions[​](https://rasa.com/docs/pro/build/writing-responses/#5-use-responses-in-flows-or-actions 'Direct link to 5. Use Responses in Flows or Actions')

Once your response definitions are in place, you can use them in:

- **Flows -** In a flow step, simply call the response by name—e.g., `utter_greet`.
- **Custom Actions -** From a custom action, dispatch the response via `dispatcher.utter_message(template="utter_greet")`.

For example, in a flow:

flows.yml

```
flows:  greet_user:    description: "Greet the user"    steps:      - action: utter_greet      # ...
```

When the assistant runs the `utter_greet` step, it sends one of the variations you defined for `utter_greet`.

- [1\. Define Your Responses](https://rasa.com/docs/pro/build/writing-responses/#1-define-your-responses)
- [2\. Use Variables in Responses (Optional)](https://rasa.com/docs/pro/build/writing-responses/#2-use-variables-in-responses-optional)
- [3\. Provide Response Variations (Optional)](https://rasa.com/docs/pro/build/writing-responses/#3-provide-response-variations-optional)
- [4\. Add Rich Content (Buttons, Images, etc.)](https://rasa.com/docs/pro/build/writing-responses/#4-add-rich-content-buttons-images-etc)
- [5\. Use Responses in Flows or Actions](https://rasa.com/docs/pro/build/writing-responses/#5-use-responses-in-flows-or-actions)