# Source: https://rasa.com/docs/pro/customize/assistant-tone

On this page

From a conversation design and linguistics perspective, the “how” of messaging is as important as the “what.” If the wording shifts abruptly from formal to casual, or if the assistant’s style changes in the middle of a conversation, it can reduce user trust and satisfaction.

CALM decouples business logic and conversation patterns from actual spoken or written content:

- **Flows** determine the overall conversation structure (the “what” and “when”).
- **Patterns** are triggered when users deviate, clarify, or repair parts of the conversation.
- **Responses** are the assistant’s prompts, messages, or follow-ups.

While you can script your Responses in your domain according to your brand voice, patterns (for repairs or clarifications) also need to reflect your assistant’s brand voice. You don’t want your default pattern responses to sound generic; they should adhere to your brand voice as well.

Because CALM uses conversation patterns that can be triggered in the context of many flows, you can use the Contextual Response Rephraser to “centralize” how messages are “finalized” according to your brand voice before being shown to users.

## Ways of Customizing Assistant Tone in Rasa[​](https://rasa.com/docs/pro/customize/assistant-tone/#ways-of-customizing-assistant-tone-in-rasa 'Direct link to Ways of Customizing Assistant Tone in Rasa')

There are two main ways to adapt your assistant’s messaging style:

1. **Response Rephraser**
 
 Dynamically rewrites your responses using an LLM prompt. You can instruct the LLM to adopt or maintain a specific personality or tone.
 
2. **Conditional Response Variations**
 
 Predefine different text variations (often for different slot conditions, contexts, or channels). This method is more static, but ensures each user scenario has a hand-crafted, on-brand response.
 

## What Is the Response Rephraser?[​](https://rasa.com/docs/pro/customize/assistant-tone/#what-is-the-response-rephraser 'Direct link to What Is the Response Rephraser?')

The Response Rephraser is an LLM-powered component that takes a templated response (for example, “I’m sorry, I can’t help with that”) and rephrases it while preserving your original meaning and factual content. Its key benefits are:

- **Dynamically adapting tone**: Use a single base response but produce many variations, all preserving your brand guidelines.
- **Contextual awareness**: The rephraser can read the conversation history and user input, ensuring rephrasings make sense in context.
- **Easy maintenance**: If you decide to tweak your brand style (e.g., become more informal), you can simply update the LLM prompt, rather than rewriting all your responses.

See the [Response Rephraser reference page](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/) for more detail on parameters like the LLM model, temperature, or how to provide a custom prompt template.

## What Are Conditional Response Variations?[​](https://rasa.com/docs/pro/customize/assistant-tone/#what-are-conditional-response-variations 'Direct link to What Are Conditional Response Variations?')

Conditional response variations let you define **several static message templates** under the same response name but tailor them to different states of the conversation. For example, you might have:

- A casual greeting if the `user_is_repeat_customer` slot is set to `true`.
- A more formal greeting otherwise.

When the assistant triggers the response, CALM checks any conditions you’ve set (slot values, channel, etc.) and picks the matching variant. These variations are not LLM-based—they are “what you see is what you get” messages, drawn from your domain or responses files.

See the [Conditional Response Variations reference](https://rasa.com/docs/reference/primitives/responses/#conditional-response-variations) for detailed YAML examples and capabilities.

## When to Use Which?[​](https://rasa.com/docs/pro/customize/assistant-tone/#when-to-use-which 'Direct link to When to Use Which?')

- **Use the Response Rephraser** if you want:
 - _Dynamic, LLM-powered rewording_ that can adapt to your brand voice, summarizing or refining the text while retaining the original meaning.
 - A simpler way to unify tone across your entire assistant, especially for messages that appear in repair patterns or emergent flows.
- **Use Conditional Response Variations** if you want:
 - _Strict control_ over the exact wording in specific contexts or channels (e.g., “VIP members get a special greeting”).
 - Variation without an external LLM call, or you need a guaranteed brand-approved statement for certain user segments.

Of course, you can also combine them—some responses can have multiple static variants, and you still run them through the Rephraser for final polishing.

## How to Create Conditional Response Variations[​](https://rasa.com/docs/pro/customize/assistant-tone/#how-to-create-conditional-response-variations 'Direct link to How to Create Conditional Response Variations')

1. **In your domain file**, provide multiple responses under the same response name.
2. **Add a `condition` block** for each variant to specify which slot values must match (or which channel must match).
3. Always **include a default fallback** response (with no condition) in case none of the conditions are met.

domain.yml

```
responses:  utter_greet:    - condition:        - type: slot          name: logged_in          value: true      text: "Hey, welcome back! How are you?"    - text: "Hello! How can I help you today?
```

For more on advanced usage, see the Reference → Conditional Response Variations.

## How to Customize the Response Rephraser[​](https://rasa.com/docs/pro/customize/assistant-tone/#how-to-customize-the-response-rephraser 'Direct link to How to Customize the Response Rephraser')

You can configure the Response Rephraser to ensure it outputs messages aligned with your desired personality or brand identity.

### 1\. Enabling Rephraser Across All Responses[​](https://rasa.com/docs/pro/customize/assistant-tone/#1-enabling-rephraser-across-all-responses 'Direct link to 1. Enabling Rephraser Across All Responses')

In your `endpoints.yml` file, add:

endpoints.yml

```
nlg:  type: rephrase  rephrase_all: true
```

This automatically attempts to rephrase every utterance. If you want some responses left untouched, annotate them with `metadata: { rephrase: false }` in your domain.

### 2\. Enabling Rephraser for Specific Responses[​](https://rasa.com/docs/pro/customize/assistant-tone/#2-enabling-rephraser-for-specific-responses 'Direct link to 2. Enabling Rephraser for Specific Responses')

If you prefer a more selective approach:

- Enable `type: rephrase` in `endpoints.yml` **without** setting `rephrase_all: true`.
 
- Add `metadata: { rephrase: true }` to only the responses you’d like rephrased:
 
 domain.yml
 
 ```
    responses:  utter_greet:    - text: "Hello there!"      metadata:        rephrase: true
    ```
 

### 3\. Setting a Custom Prompt[​](https://rasa.com/docs/pro/customize/assistant-tone/#3-setting-a-custom-prompt 'Direct link to 3. Setting a Custom Prompt')

You can supply your own Jinja2 prompt template to the rephraser. This is especially important to define a _tone or style_. For example:

endpoints.yml

```
nlg:  type: rephrase  prompt: "prompts/brand-tone-rephraser-template.jinja2"
```

Inside that `.jinja2` file, you could add instructions such as:

> “Use a casual, friendly tone in second-person. Always address the user by name if available.”

You can also override the default prompt for a _single_ response by setting `rephrase_prompt` in its metadata (see the reference docs for an example).

## How to Test Rephrased Responses?[​](https://rasa.com/docs/pro/customize/assistant-tone/#how-to-test-rephrased-responses 'Direct link to How to Test Rephrased Responses?')

Because the final assistant message may be partially (or entirely) generated by an LLM, it’s crucial to **test for both correctness and style**:

1. [**Generative Response Is Relevant**](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-relevant-assertion)
 
 Ensures the rephrased message is on-topic and aligns with the user’s query.
 
 tests/e2e\_test\_cases.yml
 
 ```
    assertions:  - generative_response_is_relevant:      threshold: 0.90
    ```
 
2. [**Generative Response Is Grounded**](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-grounded-assertion)
 
 Ensures the rephrased message remains factually accurate to the original domain response or RAG context.
 
 tests/e2e\_test\_cases.yml
 
 ```
    assertions:  - generative_response_is_grounded:      threshold: 0.90      ground_truth: "Free upgrades require a Platinum membership."
    ```
 

You can add these assertions in end-to-end tests to confirm that the LLM’s rewriting (1) remains brand-appropriate, (2) is accurate, and (3) is still relevant to the user’s input. See E2E Testing for more on these assertion types.

- [Ways of Customizing Assistant Tone in Rasa](https://rasa.com/docs/pro/customize/assistant-tone/#ways-of-customizing-assistant-tone-in-rasa)
- [What Is the Response Rephraser?](https://rasa.com/docs/pro/customize/assistant-tone/#what-is-the-response-rephraser)
- [What Are Conditional Response Variations?](https://rasa.com/docs/pro/customize/assistant-tone/#what-are-conditional-response-variations)
- [When to Use Which?](https://rasa.com/docs/pro/customize/assistant-tone/#when-to-use-which)
- [How to Create Conditional Response Variations](https://rasa.com/docs/pro/customize/assistant-tone/#how-to-create-conditional-response-variations)
- [How to Customize the Response Rephraser](https://rasa.com/docs/pro/customize/assistant-tone/#how-to-customize-the-response-rephraser)
 - [1\. Enabling Rephraser Across All Responses](https://rasa.com/docs/pro/customize/assistant-tone/#1-enabling-rephraser-across-all-responses)
 - [2\. Enabling Rephraser for Specific Responses](https://rasa.com/docs/pro/customize/assistant-tone/#2-enabling-rephraser-for-specific-responses)
 - [3\. Setting a Custom Prompt](https://rasa.com/docs/pro/customize/assistant-tone/#3-setting-a-custom-prompt)
- [How to Test Rephrased Responses?](https://rasa.com/docs/pro/customize/assistant-tone/#how-to-test-rephrased-responses)