# Source: https://rasa.com/docs/reference/config/policies/overview

On this page

In Rasa, policies are the components responsible for dialogue management. In a CALM-based assistant, the [FlowPolicy](https://rasa.com/docs/reference/config/policies/flow-policy/) is responsible for executing your business logic. If you're building an NLU-based assistant, you can read about the relevant policies [here](https://legacy-docs-oss.rasa.com/docs/rasa/policies).

Policies in CALM

[Dialogue Understanding](https://rasa.com/docs/learn/concepts/dialogue-understanding/) in CALM accounts for the context of a conversation, so the role of the dialogue manager is simpler than it is in an NLU-based assistant.

You can customize the policies your assistant uses by specifying the `policies` key in your project's `config.yml`. In most cases, the default configuration should meet your needs, customization is there for advanced use-cases.

There are different policies to choose from, and you can include multiple policies in a single configuration. Here's an example of what a list of policies might look like:

- Rasa Pro <=3.7.x
- Rasa Pro >=3.8.x

config.yml

```
# ...policies:  - name: FlowPolicy  - name: rasa_plus.ml.EnterpriseSearchPolicy
```

config.yml

```
# ...policies:  - name: FlowPolicy  - name: EnterpriseSearchPolicy
```

## Action Selection[​](https://rasa.com/docs/reference/config/policies/overview/#action-selection 'Direct link to Action Selection')

At every turn, each policy defined in your configuration gets a chance to predict a next [action](https://rasa.com/docs/reference/primitives/actions/) with a certain confidence level. A policy can also decide not to predict any action. The policy that predicts with the highest confidence decides the assistant's next action.

Maximum number of predictions

By default, when using CALM assistant (or routing to it, in case of [coexistence of NLU-based and CALM systems](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/)) the maximum number of next action predictions after each user message is 1000. To update this value, you can set the environment variable `MAX_NUMBER_OF_PREDICTIONS_CALM` to the desired number of maximum predictions. For an NLU-based assistant the maximum is set in the environment variable `MAX_NUMBER_OF_PREDICTIONS` with a default of 10.

- [Action Selection](https://rasa.com/docs/reference/config/policies/overview/#action-selection)