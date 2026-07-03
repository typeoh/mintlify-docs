# Source: https://rasa.com/docs/reference/config/components/nlu-command-adapter

On this page

## How the NLUCommandAdapter Works[​](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#how-the-nlucommandadapter-works 'Direct link to How the NLUCommandAdapter Works')

The `NLUCommandAdapter` uses the classic way to start flows, such as using predicted intents by an intent classifier. It looks at the predicted intent from the [intent classifier](https://rasa.com/docs/reference/config/components/nlu-components/#intent-classifiers) and tries to find a flow with a corresponding [NLU trigger](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger) defined. If a flow has a NLU trigger matching the predicted intent and the confidence is larger than the given threshold defined in the NLU trigger, the `NLUCommandAdapter` will return a `StartFlow` command to begin the corresponding flow.

## Using the NLUCommandAdapter[​](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#using-the-nlucommandadapter 'Direct link to Using the NLUCommandAdapter')

To use this component in your assistant, add the `NLUCommandAdapter` to your NLU pipeline in the `config.yml` file. You also need to have an [intent classifier](https://rasa.com/docs/reference/config/components/nlu-components/#intent-classifiers) listed in your NLU pipeline. Read more about the `config.yml` file [here](https://rasa.com/docs/reference/config/overview/).

config.yml

```
pipeline:# - ...  - name: NLUCommandAdapter# - ...
```

## When to use the NLUCommandAdapter[​](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#when-to-use-the-nlucommandadapter 'Direct link to When to use the NLUCommandAdapter')

We recommend to use the `NLUCommandAdapter` in two scenarios:

- You want to use NLU data containing intent and examples along with the CALM paradigm. Using the `NLUCommandAdapter` you can initiate a flow based on a predicted intent, given you already have a solid intent classifier in place. Once the flow is initiated, the business logic would be executed as usual in the CALM paradigm with commands predicted by the `LLMCommandGenerator` and policies predicting the next best action.
 
- You want to minimize the costs by not making an API call to the LLM each time. The `NLUCommandAdapter` does not make any API call to a LLM compared to the `LLMCommandGenerator`. Using the `NLUCommandAdapter` saves some costs. Make sure you have a solid intent classifier in place when using the `NLUCommandAdapter`; otherwise, incorrect flows will begin.
 
 New in 3.12
 
 If you are using a LLM-based command generator alongside the `NLUCommandAdapter` in the config pipeline, note that both the LLM-based command generator and the `NLUCommandAdapter` can now issue commands at any given conversation turn. To enable this behaviour, you should set the `minimize_num_calls` boolean parameter to `false` in the [LLM-based command generator configuration](https://rasa.com/docs/reference/config/components/llm-command-generators/#interaction-with-other-types-of-command-generators).
 

## Customization[​](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#customization 'Direct link to Customization')

To restrict the length of user messages, you can set the `user_input.max_characters` (default value is 420 characters).

config.yml

```
pipeline:  - name: NLUCommandAdapter    user_input:      max_characters: 420
```

- [How the NLUCommandAdapter Works](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#how-the-nlucommandadapter-works)
- [Using the NLUCommandAdapter](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#using-the-nlucommandadapter)
- [When to use the NLUCommandAdapter](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#when-to-use-the-nlucommandadapter)
- [Customization](https://rasa.com/docs/reference/config/components/nlu-command-adapter/#customization)