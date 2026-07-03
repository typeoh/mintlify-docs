# Source: https://rasa.com/docs/reference/config/policies/intentless-policy

On this page

warning

The _Intentless Policy_ is deprecated and not recommended for use in Rasa 3.x anymore. It will be removed in Rasa '4.0.0'.

The Intentless Policy is used to send `responses` that are defined in the domain, but are not part of any flow. This can be helpful for handling chitchat, contextual questions, and high-stakes topics, effectively. To enhance its performance and tailor it to meet specific requirements, you can customize the policy's prompt and add example conversations.

### Adding the Intentless Policy to your bot[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#adding-the-intentless-policy-to-your-bot 'Direct link to Adding the Intentless Policy to your bot')

To add `IntentlessPolicy` to your bot, add it to your `config.yml`:

- Rasa Pro <=3.7.x
- Rasa Pro >=3.8.x

config.yml

```
policies:  # ... any other policies you have  - name: rasa_plus.ml.IntentlessPolicy
```

config.yml

```
policies:  # ... any other policies you have  - name: IntentlessPolicy
```

As with all components which make use of LLMs, you can configure which provider and model to use, as well as other parameters. All of those are described [here](https://rasa.com/docs/reference/config/components/llm-configuration/).

If you want to use Azure OpenAI Service, you can configure the necessary parameters as described in the [Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration/#azure-openai-service) section.

### Customizing the Prompt Template[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#customizing-the-prompt-template 'Direct link to Customizing the Prompt Template')

You can change the prompt template used to generate a response by setting the `prompt` property in the `config.yml`:

- Rasa Pro <=3.7.x
- Rasa Pro >=3.8.x

config.yml

```
policies:# - ...  - name: rasa_plus.ml.IntentlessPolicy    prompt: prompts/intentless-policy-template.jinja2
```

config.yml

```
policies:# - ...  - name: IntentlessPolicy    prompt: prompts/intentless-policy-template.jinja2
```

The prompt is a [Jinja2](https://jinja.palletsprojects.com/en/3.0.x/) template that can be used to customize the prompt. The following variables are available in the prompt:

- `conversations`: A list of example conversations between a user and the AI that are fetched from the stories.
- `current_conversation`: The current conversation with the user.
- `responses`: A list of example responses that the assistant can send.

### Steering the Intentless Policy[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#steering-the-intentless-policy 'Direct link to Steering the Intentless Policy')

The Intentless Policy can often choose the correct response in a zero-shot fashion. That is, without providing any example conversations to the LLM.

However, you can improve the performance of the policy by adding example conversations. To do this, add [end-to-end stories](https://legacy-docs-oss.rasa.com/docs/rasa/training-data-format#test-stories) to `data/e2e_stories.yml` to your training data. These conversations will be used as examples to help the Intentless Policy learn.

data/e2e\_stories.yml

```
- story: currencies  steps:    - user: How many different currencies can I hold money in?    - action: utter_faq_4- story: automatic transfers travel  steps:    - user: Can I add money automatically to my account while traveling?    - action: utter_faq_5- story: user gives a reason why they can't visit the branch  steps:    - user: I'd like to add my wife to my credit card    - action: utter_faq_10    - user: I've got a broken leg    - action: utter_faq_11
```

### Chitchat[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#chitchat 'Direct link to Chitchat')

In an enterprise setting it may not be appropriate to use a purely generative model to handle chitchat. Teams want to ensure that the assistant is always on-brand and on-message. Using the Intentless Policy, you can define vetted, human-authored `responses` that your assistant can send. Because the Intentless Policy leverages LLMs and considers the whole context of the conversation when selecting an appropriate `response`, it is much more powerful than simply predicting an intent and triggering a fixed response to it.

### High-stakes Topics[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#high-stakes-topics 'Direct link to High-stakes Topics')

For dealing with high-stakes topics, the Intentless Policy serves as a robust alternative to the [Enterprise Search Policy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/).

For example, if users have questions about policies, legal terms, or guarantees, like: _"My situation is X, I need Y. Is that covered by my policy?"_ In these cases the EnterpriseSearchPolicy's RAG approach is risky. Even with the relevant content present in the prompt, a RAG approach allows an LLM to make interpretations of documents. Even with the relevant content present in the prompt, a RAG approach allows an LLM to make interpretations of documents. The answer users get will vary depending on the exact phrasing of their question, and may change when the underlying model changes.

For high-stakes topics, it is safest to send a self-contained, vetted answer rather than relying on a generative model. The Intentless Policy provides that capability in a CALM assistant. Note that conversation design is crucial in these cases, as you want your responses to be self-contained and unambiguous rather than just "yes" or "no".

#### Enabling the Intentless Policy[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#enabling-the-intentless-policy 'Direct link to Enabling the Intentless Policy')

In Rasa, knowledge-based questions are directed to the default `pattern_search` flow. By default it responds with `utter_no_knowledge_base` which denies the request. However, you can customize it to initiate the `action_trigger_chitchat` that triggers the Intentless Policy.

flows.yml

```
flows:  pattern_search:    description: Addressing FAQ    name: pattern search    steps:      - action: action_trigger_chitchat
```

info

When overwriting `pattern_search`, you can choose either the Intentless Policy or the Enterprise Search Policy for handling knowledge-based questions. Both policies cannot be active simultaneously.

### Interjections[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#interjections 'Direct link to Interjections')

When a flow reaches a `collect` step and your assistant asks the user for information, your user might ask a clarifying question, refuse to answer, or otherwise interject in the continuation of the flow. In these cases, the Intentless Policy can contextually select appropriate `responses`, while afterwards allowing the flow to continue.

### Testing[​](https://rasa.com/docs/reference/config/policies/intentless-policy/#testing 'Direct link to Testing')

Writing [end-to-end tests](https://rasa.com/docs/pro/testing/evaluating-assistant/) allows you to evaluate the performance of the Intentless Policy and to guard against regressions.

- [Adding the Intentless Policy to your bot](https://rasa.com/docs/reference/config/policies/intentless-policy/#adding-the-intentless-policy-to-your-bot)
- [Customizing the Prompt Template](https://rasa.com/docs/reference/config/policies/intentless-policy/#customizing-the-prompt-template)
- [Steering the Intentless Policy](https://rasa.com/docs/reference/config/policies/intentless-policy/#steering-the-intentless-policy)
- [Chitchat](https://rasa.com/docs/reference/config/policies/intentless-policy/#chitchat)
- [High-stakes Topics](https://rasa.com/docs/reference/config/policies/intentless-policy/#high-stakes-topics)
- [Interjections](https://rasa.com/docs/reference/config/policies/intentless-policy/#interjections)
- [Testing](https://rasa.com/docs/reference/config/policies/intentless-policy/#testing)