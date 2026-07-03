# Source: https://rasa.com/docs/reference/config/components/graph-recipe

On this page

Default Recipe or Graph Recipe?

You will probably only need graph recipes if you're running ML experiments or ablation studies on an existing model. We recommend starting with the default recipe and for many applications that will be all that's needed.

We now support graph recipes in addition to the default recipe. Graph recipes provide more granular control over how execution graph schemas are built.

New in 3.1

This feature is experimental. We introduce experimental features to get feedback from our community, so we encourage you to try it out! However, the functionality might be changed or removed in the future. If you have feedback (positive or negative) please share it with us on the [Rasa Forum](https://forum.rasa.com).

## Differences with Default Recipe[​](https://rasa.com/docs/reference/config/components/graph-recipe/#differences-with-default-recipe 'Direct link to Differences with Default Recipe')

There are some differences between the default recipe and the new graph recipe. Main differences are:

- Default recipe is named `default.v1` in the config file whereas graph recipes are named `graph.v1`.
- Default recipes provide an easy to use recipe structure whereas graph recipes are more advanced and powerful.
- Default recipes are very opinionated and provide various defaults whereas graph recipes are more explicit.
- Default recipes can auto-configure themselves and dump the defaults used to the file if some sections in `config.yml` are missing, whereas graph recipes do none of this and assume what you see is what you get. There are no surprises with graph recipes.
- Default recipe divides graph configuration into mainly two parts: `pipeline` and `policies`. These can also be described as NLU and core (dialogue management) parts. For graph recipe on the other hand, the separation is between training (ie. `train_schema`) and prediction (ie. `predict_schema`).

Starting from scratch?

If you don't know which recipe to choose, use the default recipe to bootstrap your project fast. If later you find that you need more fine-grained control, you can always change your recipe to be a graph recipe.

## Graph Configuration File Structure[​](https://rasa.com/docs/reference/config/components/graph-recipe/#graph-configuration-file-structure 'Direct link to Graph Configuration File Structure')

Graph recipes share `recipe` and `language` keys with the same meaning. Similarities end there as graph recipes do not have `pipeline` or `policies` keys but they do have `train_schema` and `predict_schema` keys for determining the graph nodes during train and predict runs respectively. In addition to this, target nodes for NLU and core can be specified explicitly with graph recipes, these can be declared with `nlu_target` and `core_target`. If targets are omitted, node names used by default recipe will take over, and these are `run_RegexMessageHandler` and `select_prediction` for nlu and core respectively.

Here's an example graph recipe:

```
# The config recipe.recipe: graph.v1language: encore_target: custom_core_targetnlu_target: custom_nlu_targettrain_schema:  nodes:    # We skip schema_validator node (we only have this for DefaultV1Recipe    # since we don't do validation for the GraphV1Recipe)    finetuning_validator:      needs:        importer: __importer__      uses: rasa.graph_components.validators.finetuning_validator.FinetuningValidator      constructor_name: create      fn: validate      config:        validate_core: true        validate_nlu: true      eager: false      is_target: false      is_input: true      resource: null    nlu_training_data_provider:      needs:        importer: finetuning_validator      uses: rasa.graph_components.providers.nlu_training_data_provider.NLUTrainingDataProvider      constructor_name: create      fn: provide      config:        language: en        persist: false      eager: false      is_target: false      is_input: true      resource: null    domain_provider:      needs:        importer: finetuning_validator      uses: rasa.graph_components.providers.domain_provider.DomainProvider      constructor_name: create      fn: provide_train      config: { }      eager: false      is_target: true      is_input: true      resource: null    domain_for_core_training_provider:      needs:        domain: domain_provider      uses: rasa.graph_components.providers.domain_for_core_training_provider.DomainForCoreTrainingProvider      constructor_name: create      fn: provide      config: { }      eager: false      is_target: false      is_input: true      resource: null    story_graph_provider:      needs:        importer: finetuning_validator      uses: rasa.graph_components.providers.story_graph_provider.StoryGraphProvider      constructor_name: create      fn: provide      config:        exclusion_percentage: null      eager: false      is_target: false      is_input: true      resource: null    training_tracker_provider:      needs:        story_graph: story_graph_provider        domain: domain_for_core_training_provider      uses: rasa.graph_components.providers.training_tracker_provider.TrainingTrackerProvider      constructor_name: create      fn: provide      config: { }      eager: false      is_target: false      is_input: false      resource: null    train_MemoizationPolicy0:      needs:        training_trackers: training_tracker_provider        domain: domain_for_core_training_provider      uses: rasa.core.policies.memoization.MemoizationPolicy      constructor_name: create      fn: train      config: { }      eager: false      is_target: true      is_input: false      resource: nullpredict_schema:  nodes:    nlu_message_converter:      needs:        messages: __message__      uses: rasa.graph_components.converters.nlu_message_converter.NLUMessageConverter      constructor_name: load      fn: convert_user_message      config: {}      eager: true      is_target: false      is_input: false      resource: null    custom_nlu_target:      needs:        messages: nlu_message_converter        domain: domain_provider      uses: rasa.nlu.classifiers.regex_message_handler.RegexMessageHandler      constructor_name: load      fn: process      config: {}      eager: true      is_target: false      is_input: false      resource: null    domain_provider:      needs: {}      uses: rasa.graph_components.providers.domain_provider.DomainProvider      constructor_name: load      fn: provide_inference      config: {}      eager: true      is_target: false      is_input: false      resource:        name: domain_provider    run_MemoizationPolicy0:      needs:        domain: domain_provider        tracker: __tracker__        rule_only_data: rule_only_data_provider      uses: rasa.core.policies.memoization.MemoizationPolicy      constructor_name: load      fn: predict_action_probabilities      config: {}      eager: true      is_target: false      is_input: false      resource:        name: train_MemoizationPolicy0    rule_only_data_provider:      needs: {}      uses: rasa.graph_components.providers.rule_only_provider.RuleOnlyDataProvider      constructor_name: load      fn: provide      config: {}      eager: true      is_target: false      is_input: false      resource:        name: train_RulePolicy1    custom_core_target:      needs:        policy0: run_MemoizationPolicy0        domain: domain_provider        tracker: __tracker__      uses: rasa.core.policies.ensemble.DefaultPolicyPredictionEnsemble      constructor_name: load      fn: combine_predictions_from_kwargs      config: {}      eager: true      is_target: false      is_input: false      resource: null
```

graph targets

For NLU, default target name of `run_RegexMessageHandler` will be used, while for core (dialogue management) the target will be called `select_prediction` if omitted. Make sure you have graph nodes with relevant names in your schema definitions.

In a similar fashion, note that the default resource needed by the first graph node is fixed to be `__importer__` (representing configuration, training data etc.) for training task and it is `__message__` (representing the message received) for prediction task. Make sure your first nodes make use of these dependencies.

## Graph Node Configuration[​](https://rasa.com/docs/reference/config/components/graph-recipe/#graph-node-configuration 'Direct link to Graph Node Configuration')

As you can see in the example above, graph recipes are very much explicit and you can configure each graph node as you would like. Here is an explanation of what some of the keys mean:

- `needs`: You can define here what data your graph node requires and from which parent node. Key is the data name, whereas the value would refer to the node name.

```
needs:    messages: nlu_message_converter
```

Current graph node needs `messages` which is provided by `nlu_message_converter` node.

- `uses`: You can provide the class used to instantiate this node with this key. Please provide the full path in Python path syntax, eg.

```
uses: rasa.graph_components.converters.nlu_message_converter.NLUMessageConverter
```

You are not required to use Rasa internal graph component classes and you can use your own components here.

- `constructor_name`: This is the constructor used to instantiate your component. Example:

```
constructor_name: load
```

- `fn`: This is the function used in executing the graph component. Example:

```
fn: combine_predictions_from_kwargs
```

- `config`: You can provide any configuration parameters for your components using this key.

```
config:    language: en    persist: false
```

- `eager`: This determines if your component should be eagerly loaded when the graph is constructed or if it should wait until the runtime (this is called lazy instantiation). Usually we always instantiate lazily during training and eagerly during inference (to avoid slow first prediction).

```
eager: true
```

- `resource`: If given, graph node is loaded from this resource instead of instantiated from scratch. This is e.g. used to load a trained component for predictions.

```
resource:    name: train_RulePolicy1
```

- `is_target`: Boolean value, if `True` then this node can't be pruned during fingerprinting (it might be replaced with a cached value though). This is e.g. used for all components which train as their result always needs to be added to the model archive so that the data is available during inference.

```
is_target: false
```

- `is_input`: Boolean value; nodes with `is_input` are _always_ run (also during the fingerprint run). This makes sure that we e.g. detect changes in file contents.

```
 is_input: false
```

- [Differences with Default Recipe](https://rasa.com/docs/reference/config/components/graph-recipe/#differences-with-default-recipe)
- [Graph Configuration File Structure](https://rasa.com/docs/reference/config/components/graph-recipe/#graph-configuration-file-structure)
- [Graph Node Configuration](https://rasa.com/docs/reference/config/components/graph-recipe/#graph-node-configuration)