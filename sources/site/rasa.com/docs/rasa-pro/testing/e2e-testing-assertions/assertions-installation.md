# Source: https://rasa.com/docs/rasa-pro/testing/e2e-testing-assertions/assertions-installation

On this page

Why testing your assistant?

For more information E2E testing and assertions, see the [E2E testing product documentation](https://rasa.com/docs/pro/testing/evaluating-assistant/).

## Configuration Prerequisites[​](https://rasa.com/docs/reference/testing/assertions/#configuration-prerequisites 'Direct link to Configuration Prerequisites')

New in 3.12

E2E testing with assertions is no longer in beta and is now generally available in Rasa.

### Generative Response LLM Judge Configuration[​](https://rasa.com/docs/reference/testing/assertions/#generative-response-llm-judge-configuration 'Direct link to Generative Response LLM Judge Configuration')

New in 3.12

You can now configure the LLM Judge model in the `conftest.yml` file to use different LLM providers other than the default `openai`. For more information on how to choose the best fit for your use-case, see the [LLM Judge Provider Bias Measurement Framework](https://rasa.com/docs/reference/testing/assertions/#llm-judge-provider-bias-measurement) section.

Rasa uses [LLM (Large Language Model) evaluation](https://mlflow.org/docs/latest/llms/llm-evaluate/index.html) to assess the relevance and factual accuracy of the assistant's generative responses. This LLM is also referred to as a "LLM-Judge" model because it assesses another model's output. In Rasa's use case, the LLM-Judge model evaluates whether the [generative response is relevant](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-relevant-assertion) to the provided input or whether the [generative response is factually accurate](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-grounded-assertion) in relation to the provided or extracted ground truth text input.

By default, the LLM Judge model is configured to use the OpenAI `gpt-4.1-mini-2025-04-14` model to benefit of the long context window. The default embeddings model is the OpenAI `text-embedding-3-large`.

If you want to use a different model, model provider or embeddings model, you can configure the LLM Judge model in the `conftest.yml` file. This new testing configuration file is automatically discoverable by Rasa as long as it is placed in the root directory of your assistant project. You can use either of the available configuration options: [model groups](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups) or individual model configurations as in the example below.

conftest.yml

```
llm_judge:  llm:    provider: openai    model: "gpt-4.1-mini-2025-04-14"  embeddings:    provider: openai    model: "text-embedding-3-large"
```

## Assertion Types[​](https://rasa.com/docs/reference/testing/assertions/#assertion-types 'Direct link to Assertion Types')

Assertions allow you to check events like flows starting, or to confirm if a generative response is relevant/grounded, among others.

Important

If a user step contains assertions, the older step types like bot: ... or utter: ... are ignored within that same step. You'll have to rely on the `bot_uttered` assertion to check the response.

Below is a comprehensive list of assertion types you can use in your E2E tests. These allow you to verify everything from flow status to the factual grounding of a generative response.

### Flow Started Assertion[​](https://rasa.com/docs/reference/testing/assertions/#flow-started-assertion 'Direct link to Flow Started Assertion')

**`flow_started`** checks if the flow with the provided id was started.

New in 3.15

You can now use the `operator` key to specify how multiple flow IDs are evaluated. If you provide multiple flow IDs, the assertion will pass if _any_ or _all_ of the specified flows were started, depending on the operator used. The available operators are `any` and `all` (which is the default behavior). When using the `all` operator, exactly one flow ID must be provided to maintain backward compatibility.

The previous behavior of the assertion (checking that a single flow was started) remains unchanged, however the syntax has been deprecated and will be removed in the next major release. Please update your test cases to use the new syntax with the `operator` key.

An example using the new syntax with the `all` operator is shown below. Note that only one flow ID is provided:

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "I want to book a flight"      assertions:        - flow_started:              operator: "all"              flow_ids:                - "flight_booking"
```

### Flow Completed Assertion[​](https://rasa.com/docs/reference/testing/assertions/#flow-completed-assertion 'Direct link to Flow Completed Assertion')

**`flow_completed`** checks if the flow with the provided id was completed. Optionally, you can specify a `flow_step_id` if you want to confirm the final flow step.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "What is the average cost of a flight from New York to San Francisco?"      assertions:        - flow_completed:            flow_id: "pattern_search"            flow_step_id: "action_trigger_search"
```

### Flow Cancelled Assertion[​](https://rasa.com/docs/reference/testing/assertions/#flow-cancelled-assertion 'Direct link to Flow Cancelled Assertion')

**`flow_cancelled`** checks if the flow with the provided id was cancelled. You can also specify a `flow_step_id` if needed.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    ... # other user steps    - user: "Wait, I changed my mind, I don't want to book a flight."      assertions:        - flow_cancelled:            flow_id: "flight_booking"            flow_step_id: "make_payment"
```

### Pattern Clarification Contains Assertion[​](https://rasa.com/docs/reference/testing/assertions/#pattern-clarification-contains-assertion 'Direct link to Pattern Clarification Contains Assertion')

**`pattern_clarification_contains`** checks if the clarification (repair) pattern was triggered and returned the expected flow names to the end user.

New in 3.15

You can now use the `operator` key to specify how multiple flow IDs are evaluated. If you provide multiple flow IDs, the assertion will pass if _any_ or _all_ of the specified flows were suggested, depending on the operator used. The available operators are `any` and `all` (which is the default behavior).

The previous behavior of the assertion (checking that all provided flow names are suggested) remains unchanged, however the syntax has been deprecated and will be removed in the next major release. Please update your test cases to use the new syntax with the `operator` key.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "make booking"      assertions:        - pattern_clarification_contains:            operator: "all"            flow_ids:            - "flight_booking"            - "hotel_booking"
```

### Slot Was Set Assertion[​](https://rasa.com/docs/reference/testing/assertions/#slot-was-set-assertion 'Direct link to Slot Was Set Assertion')

**`slot_was_set`** checks if the slot(s) with the provided name were filled with the provided value. Match the slot’s type in your domain (e.g. use boolean, integer, or float as appropriate without quotes).

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "I want to book a flight from New York to San Francisco"      assertions:        - slot_was_set:            - name: "origin"              value: "New York"            - name: "destination"              value: "San Francisco"
```

### Slot Was Not Set Assertion[​](https://rasa.com/docs/reference/testing/assertions/#slot-was-not-set-assertion 'Direct link to Slot Was Not Set Assertion')

**`slot_was_not_set`** checks if a slot was _not_ filled. If you specify a value, it checks the slot was not filled _with that value_.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "I want to book a flight to San Francisco."      assertions:        - slot_was_not_set:            - name: "origin"        - slot_was_not_set:            - name: "destination"              value: "New York"
```

If only `name` is provided, the test confirms the slot’s value remains `None` (or uninitialized).

### Action Executed Assertion[​](https://rasa.com/docs/reference/testing/assertions/#action-executed-assertion 'Direct link to Action Executed Assertion')

**`action_executed`** checks if the specified action was triggered.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "Book me a flight from New York to San Francisco tomorrow first thing in the morning."      assertions:        - action_executed: "action_book_flight"
```

### Bot Uttered Assertion[​](https://rasa.com/docs/reference/testing/assertions/#bot-uttered-assertion 'Direct link to Bot Uttered Assertion')

**`bot_uttered`** checks if the bot’s last utterance matches the provided pattern, buttons, and/or domain response name. Use `text_matches` for the utterance text, which can be a string or regex.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "I want to book a flight"      assertions:        - bot_uttered:            utter_name: utter_ask_destination            text_matches: "Where would you like to fly to?"            buttons:              - title: "New York"                payload: "/SetSlots(destination=New York)"              - title: "San Francisco"                payload: "/SetSlots(destination=San Francisco)"
```

When asserting buttons, list them in the same order as defined in your domain file or custom action.

### Bot Did Not Utter Assertion[​](https://rasa.com/docs/reference/testing/assertions/#bot-did-not-utter-assertion 'Direct link to Bot Did Not Utter Assertion')

**`bot_did_not_utter`** checks that the bot’s utterance does _not_ match the provided pattern, buttons, or domain response name.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "I want to book a flight"      assertions:        - bot_did_not_utter:            utter_name: utter_ask_payment            text_matches: "How would you like to pay?"            buttons:              - title: "Credit Card"                payload: "/set_payment_method{'method': 'credit_card'}"              - title: "PayPal"                payload: "/set_payment_method{'method': 'paypal'}"
```

### Generative Response Is Relevant Assertion[​](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-relevant-assertion 'Direct link to Generative Response Is Relevant Assertion')

**`generative_response_is_relevant`** checks if the bot’s generative response is relevant to the user’s message. A `threshold` (0–1) indicates how strictly you compare the system’s relevance score.

The LLM Judge model will generate 3 question variations addressing the bot response that is evaluated for relevance. The relevance score is the average of the cosine similarities between the user message and the generated question variations.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "What times are the flights from New York to San Francisco tomorrow?"      assertions:        - generative_response_is_relevant:            threshold: 0.90
```

You can also specify `utter_name` if you want to check a specific domain response event:

tests/e2e\_test\_cases.yml

```
- user: "Actually, I want to amend flight date to next week."  assertions:    - generative_response_is_relevant:        threshold: 0.90        utter_name: utter_ask_correction_confirmation
```

New in 3.15

Support for custom actions as a source for generative responses has been added. You can now indicate the custom action name that generated the bot response to the `utter_source` key in the assertion.

You can also specify `utter_source` in the assertion to indicate the component or custom action name that generated the bot response. This enables the assertion to be applied to a specific bot message source, e.g. Enterprise Search Policy, Contextual Response Rephraser or custom actions

tests/e2e\_test\_cases.yml

```
- user: "What times are the flights from New York to San Francisco tomorrow?"  assertions:    - generative_response_is_relevant:        threshold: 0.90        utter_source: ContextualResponseRephraser | EnterpriseSearchPolicy | <custom_action_name>
```

### Generative Response Is Grounded Assertion[​](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-grounded-assertion 'Direct link to Generative Response Is Grounded Assertion')

**`generative_response_is_grounded`** checks if the bot’s generative response is factually accurate given a ground-truth reference.

The LLM Judge will extract the atomic statements from the bot message evaluated for factual grounding. Then it will determine whether each of these statements is supported by the ground truth: yes/no. The final score is the number of grounded statements divided by the total number of statements. The threshold is the minimum score required for the assertion to pass.

tests/e2e\_test\_cases.yml

```
test_cases:- test_case: flight_booking  steps:    - user: "What is the average cost of a flight from New York to San Francisco?"      assertions:        - generative_response_is_grounded:            threshold: 0.90            ground_truth: "The average cost of a flight from New York to San Francisco is $500."
```

If the correct factual source is available in the response metadata (e.g. from an Enterprise Search lookup or rephrased domain response), the test runner can extract it automatically if you don’t provide `ground_truth` directly.

New in 3.15

Support for custom actions as a source for generative responses has been added. You can now indicate the custom action name that generated the bot response to the `utter_source` key in the assertion.

Additionally, it is recommended to define `utter_source` in the assertion to specify the component or custom action name that generated the bot response. This enables the assertion to be applied to a specific bot message source, e.g. Enterprise Search Policy, Contextual Response Rephraser or custom actions.

```
- test_case: flight_booking  steps:    - user: "What is the average cost of a flight from New York to San Francisco?"      assertions:        - generative_response_is_grounded:            threshold: 0.90            utter_source: EnterpriseSearchPolicy | ContextualResponseRephraser | <custom_action_name>
```

## LLM Judge Provider Bias Measurement[​](https://rasa.com/docs/reference/testing/assertions/#llm-judge-provider-bias-measurement 'Direct link to LLM Judge Provider Bias Measurement')

When the LLM Judge model provider is the same as that of the model used by Rasa's generative components such as the Enterprise Search Policy or Contextual Response Rephraser, there is a risk of self-preference bias. This bias can lead to overestimating or undervaluing the relevance or factual accuracy of the generative responses.

### Bias Measurement Framework[​](https://rasa.com/docs/reference/testing/assertions/#bias-measurement-framework 'Direct link to Bias Measurement Framework')

We recommend running the following self-preference bias measurement framework to evaluate the bias of the LLM Judge model on a case by case basis since results could vary depending on the domain of your assistant.

1. Compile a set of test cases that make use of both generative assertions types.
2. For every chosen model, update the `config.yml` and `nlg` endpoint to use this model to train the bot.
3. After training completes, loop through the chosen models to update the `conftest.yml` config of the LLM Judge.
4. Run the test cases from point 1 with the trained model: we only do so once per trained model, because we want the different LLM judge models in the nested loop to evaluate the same bot responses.
5. During this first run of the test cases, a human evaluator should be prompted to rate `yes/no` each bot response whether it was appropriate in response to the user question. This should be recorded as `human_preference` of data type integer: `1/0`.
6. Once we have run through all test cases and a human has rated all of them for that particular trained model, continue with running the assertions (which use the LLM Judge to evaluate these assertions: a passed assertion means rating the `llm_preference` as `1`, while a failed assertion is `0`). We also record whether the LLM judge was from the same provider from the trained model for Enterprise Search and Rephraser via `source` property: `self or other`.
7. We gather all these evaluations and calculate the bias score for the trained model, using the [Equal Opportunity inspired metric](https://arxiv.org/html/2410.21819v1#S4).

### Bias Measurement Results[​](https://rasa.com/docs/reference/testing/assertions/#bias-measurement-results 'Direct link to Bias Measurement Results')

We have measured the self-preference bias of various LLM Judge models with a small financial services bot. The bot uses both the Enterprise Search Policy and Contextual Response Rephraser to generate responses. The models we chose to test for the 3.12 release are:

- `gpt-4-0613`
- `gpt-4o-2024-11-20`
- `gpt-4o-mini-2024-07-18`
- `claude-3-5-sonnet-20241022`
- `claude-3-7-sonnet-20250219`

General guidance principles include:

- a score value of 0 indicates the absence of bias
- a value close to 1 suggests a high degree of bias.
- conversely, a value of −1 would indicate the presence of a reverse bias, where the judge model tends to undervalue responses coming from the same provider.

When interpreting the results we obtained from testing the small scale financial bot, we found that OpenAI models showed a moderate to high bias towards their own models. Anthropic models showed a moderate to high reverse bias towards their own models.

As a rule of thumb, we recommend using different providers for the LLM Judge model and the generative components of your assistant to avoid self-preference bias.

- [Configuration Prerequisites](https://rasa.com/docs/reference/testing/assertions/#configuration-prerequisites)
 - [Generative Response LLM Judge Configuration](https://rasa.com/docs/reference/testing/assertions/#generative-response-llm-judge-configuration)
- [Assertion Types](https://rasa.com/docs/reference/testing/assertions/#assertion-types)
 - [Flow Started Assertion](https://rasa.com/docs/reference/testing/assertions/#flow-started-assertion)
 - [Flow Completed Assertion](https://rasa.com/docs/reference/testing/assertions/#flow-completed-assertion)
 - [Flow Cancelled Assertion](https://rasa.com/docs/reference/testing/assertions/#flow-cancelled-assertion)
 - [Pattern Clarification Contains Assertion](https://rasa.com/docs/reference/testing/assertions/#pattern-clarification-contains-assertion)
 - [Slot Was Set Assertion](https://rasa.com/docs/reference/testing/assertions/#slot-was-set-assertion)
 - [Slot Was Not Set Assertion](https://rasa.com/docs/reference/testing/assertions/#slot-was-not-set-assertion)
 - [Action Executed Assertion](https://rasa.com/docs/reference/testing/assertions/#action-executed-assertion)
 - [Bot Uttered Assertion](https://rasa.com/docs/reference/testing/assertions/#bot-uttered-assertion)
 - [Bot Did Not Utter Assertion](https://rasa.com/docs/reference/testing/assertions/#bot-did-not-utter-assertion)
 - [Generative Response Is Relevant Assertion](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-relevant-assertion)
 - [Generative Response Is Grounded Assertion](https://rasa.com/docs/reference/testing/assertions/#generative-response-is-grounded-assertion)
- [LLM Judge Provider Bias Measurement](https://rasa.com/docs/reference/testing/assertions/#llm-judge-provider-bias-measurement)
 - [Bias Measurement Framework](https://rasa.com/docs/reference/testing/assertions/#bias-measurement-framework)
 - [Bias Measurement Results](https://rasa.com/docs/reference/testing/assertions/#bias-measurement-results)