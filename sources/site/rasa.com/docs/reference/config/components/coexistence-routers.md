# Source: https://rasa.com/docs/reference/config/components/coexistence-routers

On this page

The [coexistence of CALM and the NLU-based system](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/) depends on a routing mechanism, that routes messages based on their content to either system. You can choose between two different router components:

1. [`IntentBasedRouter`](https://rasa.com/docs/reference/config/components/coexistence-routers/#intentbasedrouter): The predicted intent of the NLU pipeline is used to decide where the message should go.
2. [`LLMBasedRouter`](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter): This component leverages an LLM to decide whether a message should be routed to the NLU-based system or CALM.

You can only use one of the router components in your assistant.

## IntentBasedRouter[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#intentbasedrouter 'Direct link to IntentBasedRouter')

The `IntentBasedRouter` uses the predicted intent from the NLU components and routes the message dependent on that intent. The router needs to be added to the pipeline in your config file.

important

The position of the `IntentBasedRouter` needs to be _after_ the NLU components and _before_ the Command Generators.

Depending on the other components you choose, your config file could look like the following.

config.yml

```
  recipe: default.v1  language: en  pipeline:  - name: WhitespaceTokenizer  - name: CountVectorsFeaturizer  - name: CountVectorsFeaturizer    analyzer: char_wb    min_ngram: 1    max_ngram: 4  - name: LogisticRegressionClassifier  - name: IntentBasedRouter    # additional configuration parameters  - name: CompactLLMCommandGenerator    llm:      model_group: openai_llm  policies:  - name: FlowPolicy  - name: RulePolicy  - name: MemoizationPolicy    max_history: 10  - name: TEDPolicy
```

endpoints.yml

```
   model_groups:     - id: openai_llm       models:         - provider: openai           model: gpt-5-mini-2025-08-07           timeout: 7           max_tokens: 256
```

### Configuration of the IntentBasedRouter[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#configuration-of-the-intentbasedrouter 'Direct link to Configuration of the IntentBasedRouter')

The following mandatory configuration parameters need to be configured:

- `nlu_entry`:
 - `sticky`: List of intents which should route to the NLU-based system in a sticky fashion.
 - `non_sticky`: List of intents which should route to the NLU-based system in a non sticky fashion.
- `calm_entry`:
 - `sticky`: List of intents which should route to CALM in a sticky fashion.

A full configuration of the `IntentBasedRouter` could for example look like the following.

config.yml

```
  pipeline:  # ...  - name: IntentBasedRouter    nlu_entry:      sticky:        - transfer_money        - check_balance        - search_transactions      non_sticky:        - chitchat    calm_entry:      sticky:        - book_hotel        - cancel_hotel        - list_hotel_bookings  # ...
```

info

Once the `IntentBasedRouter` assigns the session to the NLU-based system, the `LLMCommandGenerator` is going to be skipped so that no unnecessary costs are incurred.

### Handling missing intents[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#handling-missing-intents 'Direct link to Handling missing intents')

If an intent is predicted by an NLU component, but the intent is not part of any of the intents listed in the `IntentBasedRouter` and the routing session is currently not set, the message is routed according to the following rules:

1. We route to CALM if given the intent any of the NLU triggers of flows are activated (see [NLU triggers documentation](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger)).
2. We route to the NLU-based system otherwise.

## LLMBasedRouter[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter 'Direct link to LLMBasedRouter')

The `LLMBasedRouter` uses an LLM, by default `gpt-5-mini-2025-08-07`, to decide whether a message should be routed to the NLU-based system or CALM.

important

In order to use this component for your coexistence solution, you need to add it as the _first_ component to your pipeline in the config file.

Depending on the other components you choose, your config file could look like the following.

config.yml

```
  recipe: default.v1  language: en  pipeline:  - name: LLMBasedRouter    nlu_entry:      sticky: ...      non_sticky: ...    calm_entry:      sticky: handles everything around hotel bookings    llm:      model_group: openai_llm    # additional configuration parameters  - name: WhitespaceTokenizer  - name: CountVectorsFeaturizer  - name: CountVectorsFeaturizer    analyzer: char_wb    min_ngram: 1    max_ngram: 4  - name: LogisticRegressionClassifier  - name: CompactLLMCommandGenerator    llm:      model_group: openai_llm  policies:  - name: FlowPolicy  - name: RulePolicy  - name: MemoizationPolicy    max_history: 10  - name: TEDPolicy
```

endpoints.yml

```
   model_groups:     - id: openai_llm       models:         - provider: openai           model: gpt-5-mini-2025-08-07           timeout: 7           max_tokens: 256
```

### Configuring the LLMBasedRouter component[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#configuring-the-llmbasedrouter-component 'Direct link to Configuring the LLMBasedRouter component')

The `LLMBasedRouter` component has the following configuration parameters:

- `nlu_entry`:
 - `sticky`: Describes the general NLU-based system functionality. By default the value is `"handles everything else"`.
 - `non_sticky`: Describes the functionality of the NLU-based system that should not result in sticky routing to the NLU-based system. By default the value is `"handles chitchat"`.
- `calm_entry`:
 - `sticky` (required): Describes the functionality implemented in the CALM system of the assistant.
- `llm`: Configuration of the llm (see [section](https://rasa.com/docs/reference/config/components/coexistence-routers/#configuring-the-llm-of-the-llmbasedrouter)).
- `prompt`: File path to the prompt template (a jinja2 template) to use (see [section](https://rasa.com/docs/reference/config/components/coexistence-routers/#configuring-the-prompt-of-the-llmbasedrouter))

If you want to use Azure OpenAI Service, you can configure the necessary parameters as described in the [Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration/#azure-openai-service) section.

#### Configuring the prompt of the LLMBasedRouter[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#configuring-the-prompt-of-the-llmbasedrouter 'Direct link to Configuring the prompt of the LLMBasedRouter')

By default, the descriptions of the configuration are assembled into a prompt describing a routing task to an LLM:

router\_template.jinja2

```
You have to forward the user message to the right assistant.The following assistants are available:Assistant A: {{ calm_entry_sticky }}Assistant B: {{ nlu_entry_non_sticky }}Assistant C: {{ nlu_entry_sticky }}The user said: """{{ user_message }}"""Answer which assistant needs to get this message.Respond with exactly one character: A, B, or C.Do not output any other words, punctuation, or explanation.
```

The configuration parameter `nlu_entry.sticky` goes into `{{ nlu_entry_sticky }}`, `nlu_entry.non_sticky` goes into `{{ nlu_entry_non_sticky }}`, and `calm_entry.sticky` goes into `{{ calm_entry_sticky }}`.

This prompt is much simpler and 10 times shorter than the prompt of the [`CompactLLMCommandGenerator`](https://rasa.com/docs/reference/config/components/llm-command-generators/#compactllmcommandgenerator). Using a compact chat model for routing keeps cost low compared to the full `CompactLLMCommandGenerator` stack.

You can modify the prompt by writing your own prompt as a jinja2 template and provide it to the component as a file:

```
pipeline:# ...- name: LLMBasedRouter  prompt: prompts/llm-based-router-prompt.jinja2# ...
```

info

Once the `LLMBasedRouter` assigns the session to the NLU-based system, the `LLMCommandGenerator` is going to be skipped so that no unnecessary costs are incurred.

#### Configuring the LLM of the LLMBasedRouter[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#configuring-the-llm-of-the-llmbasedrouter 'Direct link to Configuring the LLM of the LLMBasedRouter')

The router prompt asks the model to reply with a single letter, `A`, `B`, or `C`. The `LLMBasedRouter` parses that answer and maps it to the coexistence routing decision.

If you use models that support `logit_bias`—for example gpt-4o-mini—you can steer routing more tightly with `max_tokens: 1` and a `logit_bias` map. `max_tokens: 1` limits the completion to a single token; `logit_bias` increases the likelihood of exactly one of the tokens `" A"`, `" B"`, or `" C"` (space plus letter), matching the three assistants in the default prompt. On OpenAI’s APIs, GPT-4-series models (such as gpt-4o and gpt-4o-mini) accept `max_tokens` and `logit_bias` for this pattern; GPT-5-family models do not, so configure `max_tokens` / `logit_bias` only when you point the router at a compatible model. Rasa used to include these fields in built-in defaults when the default router model supported them. For many OpenAI chat models, illustrative bias entries looked like:

```
max_tokens: 1logit_bias:  "362": 100  "426": 100  "356": 100
```

You must recompute token ids for your tokenizer (or drop `logit_bias`) whenever you change the model, or you risk boosting unrelated tokens and breaking routing.

Current default model: From Rasa Pro 3.16, the default router model is `gpt-5-mini-2025-08-07` (GPT-5 family), which does not support `logit_bias` or `max_tokens` for this steering pattern. Rasa therefore no longer includes `max_tokens` or `logit_bias` in the built-in defaults for `LLMBasedRouter`. Routing still relies on the model returning `A`, `B`, or `C`; with GPT-5-family defaults, completion is steered by the prompt.

If your model supports `logit_bias`: You can still set `max_tokens: 1` and `logit_bias` yourself for that model and provider.

info

Optional `logit_bias` is easy to misconfigure: wrong ids boost unrelated tokens and can break routing. Prefer relying on the prompt unless you have verified ids for your model.

##### Example configuration[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#example-configuration 'Direct link to Example configuration')

First, use the outer tabs for where LLM settings are defined in your Rasa Pro release. Then use the inner tabs to match your model: GPT-5-mini reflects the current default stack (no `logit_bias`); GPT-4o-mini is an example of a model where you can optionally add `max_tokens` and `logit_bias` if you want that extra steering.

- Rasa Pro <=3.10.x
- Rasa Pro >=3.11.x

- GPT-4o-mini
- GPT-5-mini

config.yml

```
   pipeline:   # ...   - name: LLMBasedRouter     llm:       provider: openai       model: "gpt-4o-mini"       timeout: 7       temperature: 1.0       max_tokens: 1       logit_bias:         "362": 100         "426": 100         "356": 100   # ...
```

config.yml

```
   pipeline:   # ...   - name: LLMBasedRouter     llm:       provider: openai       model: "gpt-5-mini-2025-08-07"       timeout: 7       temperature: 1.0   # ...
```

- GPT-4o-mini
- GPT-5-mini

config.yml

```
   pipeline:   # ...   - name: LLMBasedRouter     llm:       model_group: openai_llm   # ...
```

endpoints.yml

```
   model_groups:     - id: openai_llm       models:         - provider: openai           model: "gpt-4o-mini"           timeout: 7           temperature: 1.0           max_tokens: 1           logit_bias:             "362": 100             "426": 100             "356": 100
```

config.yml

```
   pipeline:   # ...   - name: LLMBasedRouter     llm:       model_group: openai_llm   # ...
```

endpoints.yml

```
   model_groups:     - id: openai_llm       models:         - provider: openai           model: "gpt-5-mini-2025-08-07"           timeout: 7           temperature: 1.0
```

### Handling failures and downtime of the LLM[​](https://rasa.com/docs/reference/config/components/coexistence-routers/#handling-failures-and-downtime-of-the-llm 'Direct link to Handling failures and downtime of the LLM')

If the LLM predicts an invalid answer, e.g. another character than `A`, `B`, or `C`, or if the API of the LLM is down and the LLM cannot be reached, the message is routed to the NLU-based system in a sticky fashion.