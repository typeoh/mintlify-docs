# Source: https://rasa.com/docs/reference/integrations/tracing

On this page

Distributed tracing tracks requests as they flow through a distributed system (in this case: a Rasa assistant), sending data about the requests to a **tracing backend** which collects all trace data and enables inspecting it. Trace data helps you understand the flow of requests through both the components of a single service (Rasa itself), and across different distributed services, for example, your action server.

### Supported Tracing Backends/Collectors[​](https://rasa.com/docs/reference/integrations/tracing/#supported-tracing-backendscollectors 'Direct link to Supported Tracing Backends/Collectors')

To trace requests in Rasa, you can either use [Jaeger](https://www.jaegertracing.io/) as a backend, or use the [OTEL Collector (OpenTelemetry Collector)](https://opentelemetry.io/docs/collector/). to collect traces and then send them to the backend of your choice. See [Configuring a Tracing Backend or Collector](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector) for instructions.

### Rasa Channels[​](https://rasa.com/docs/reference/integrations/tracing/#rasa-channels 'Direct link to Rasa Channels')

Trace context sent along with requests using the [W3C Trace Context Specification](https://www.w3.org/TR/trace-context/) via the REST channel is used to continue tracing in Rasa.

#### Rasa Inspector[​](https://rasa.com/docs/reference/integrations/tracing/#rasa-inspector 'Direct link to Rasa Inspector')

If you have enabled tracing in Rasa and are using the [Rasa Inspector](https://rasa.com/docs/pro/testing/trying-assistant/) debugging tool to try your assistant, note that in addition to the expected tracing span for the `Agent.handle_message` method call, the tracing backend will collect independent tracing spans for the `MessageProcessor.get_tracker` method calls. This is expected behaviour because the Rasa Inspector tool uses the Rasa [HTTP API endpoints](https://rasa.com/docs/reference/api/pro/http-api/) to retrieve the conversation tracker which is required by the Inspector interface.

### Action Server[​](https://rasa.com/docs/reference/integrations/tracing/#action-server 'Direct link to Action Server')

The trace context from Rasa is sent along with requests to the custom action server using the [W3C Trace Context Specification](https://www.w3.org/TR/trace-context/) and then used to continue tracing the request through the custom action server.

Tracing is continued in the action server by instrumenting the webhook that receives custom actions. See [Action server attributes](https://rasa.com/docs/reference/integrations/tracing/#action-executor-attributes) for the attributes captured as part of the trace context.

See [traced events](https://rasa.com/docs/reference/integrations/tracing/#traced-events) for details on what attributes are made available as part of the trace context in Rasa.

## Questions Tracing Can Help Answer[​](https://rasa.com/docs/reference/integrations/tracing/#questions-tracing-can-help-answer 'Direct link to Questions Tracing Can Help Answer')

Tracing can help troubleshoot issues in development and production, by answering questions such as:

- How does a user message request get processed across different components i.e. dialogue understanding components (NLU, `CommandGenerator`, `CommandProcessorComponent`), policies, and action server?
- Why has my Rasa assistant decided to execute a certain action?
- Why has my Rasa assistant been slow to respond?
- Why have my custom actions been slow to execute?
- What is my OpenAI prompt token usage?
- What is the performance of my Rasa assistant across different flows?
- What is the performance of my Rasa assistant across different LLM models?
- What is the performance of my Rasa assistant across different vector stores?

## Configuring a Tracing Backend or Collector[​](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector 'Direct link to Configuring a Tracing Backend or Collector')

To configure a tracing backend or collector, add a `tracing` entry to your endpoints i.e. in your `endpoints.yml` file, or in the relevant section of your Helm values in a deployment.

### Jaeger[​](https://rasa.com/docs/reference/integrations/tracing/#jaeger 'Direct link to Jaeger')

To configure a Jaeger tracing backend, specify the `type` as `jaeger`.

endpoints.yml

```
tracing:  type: jaeger  host: localhost  port: 4317  service_name: rasa  sync_export: ~
```

tip

If you come across the error "OSError: \[Errno 40\] Message too long", read the instructions [here to resolve it](https://rasa.com/docs/pro/installation/troubleshooting/#oserror-errno-40-message-too-long)

### OTEL Collector[​](https://rasa.com/docs/reference/integrations/tracing/#otel-collector 'Direct link to OTEL Collector')

Collectors are components that collect traces in a vendor-agnostic way and then forward them to various backends. For example, the OpenTelemetry Collector (OTEL) can collect traces from multiple different components and instrumentation libraries, and then export them to multiple different backends e.g. jaeger.

To configure an OTEL Collector, specify the `type` as `otlp`.

endpoints.yml

```
tracing:  type: otlp  endpoint: my-otlp-host:4317  insecure: false  service_name: rasa  root_certificates: ./tests/unit/tracing/fixtures/ca.pem
```

## Traced Events[​](https://rasa.com/docs/reference/integrations/tracing/#traced-events 'Direct link to Traced Events')

The Rasa service areas that are traceable cover the actions required to:

- **train a model** (i.e., the training of each graph component)
- **handle a message**

### Model Training[​](https://rasa.com/docs/reference/integrations/tracing/#model-training 'Direct link to Model Training')

Tracing is enabled for model training by instrumenting Rasa [`GraphTrainer`](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/engine/training/graph_trainer/#graphtrainer-objects) and [`GraphNode`](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/engine/graph/#graphnode-objects) classes.

#### `GraphTrainer` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#graphtrainer-attributes 'Direct link to graphtrainer-attributes')

The following attributes can be inspected during training of `GraphTrainer`:

- `training_type` of model configuration:
 - `"NLU"`
 - `"CORE"`
 - `"BOTH"`
 - `"END-TO-END"`
- `language` of model configuration
- `recipe_name` used in the `config.yml` file
- `output_filename`: the location where the packaged model is saved
- `is_finetuning`: boolean argument, if `True` enables incremental training

#### `GraphNode` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#graphnode-attributes 'Direct link to graphnode-attributes')

The following attributes are captured during the training (as well as prediction during message handling) of every graph node:

- `node_name`
- `component_class`
- `fn_name`: method of component class that gets called

### Message Handling[​](https://rasa.com/docs/reference/integrations/tracing/#message-handling 'Direct link to Message Handling')

The following Rasa classes are instrumented to enable tracing during message handling:

- `Agent`
- `MessageProcessor`
- [`TrackerStore`](https://rasa.com/docs/reference/integrations/tracker-stores/)
- [`LockStore`](https://rasa.com/docs/reference/integrations/lock-stores/)
- [`CompactLLMCommandGenerator`](https://rasa.com/docs/reference/integrations/tracing/#compactllmcommandgenerator-attributes)
- [`SearchReadyLLMCommandGenerator`](https://rasa.com/docs/reference/integrations/tracing/#searchreadycommandgenerator-attributes)
- [`NLUCommandAdapter`](https://rasa.com/docs/reference/integrations/tracing/#nlucommandadapter-attributes)
- [`FlowPolicy`](https://rasa.com/docs/reference/config/policies/flow-policy/)
- [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/integrations/tracing/#enterprisesearchpolicy-attributes)
- [`InformationRetrieval`](https://rasa.com/docs/reference/integrations/tracing/#informationretrieval-attributes)
- [`EndpointConfig`](https://rasa.com/docs/reference/integrations/tracing/#endpointconfig-attributes)

In addition, the following Python modules were instrumented to enable tracing during message handling:

- command processor module, i.e. utility functions leveraged by the `CommandProcessorComponent` to pre-process [predicted commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference)
- flow executor module, i.e. utility functions leveraged by `FlowPolicy` to advance flows

Namely, these operations are now traceable:

- receiving a message
- parsing the message
- predicting [commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference)
- [pre-processing commands](https://rasa.com/docs/reference/integrations/tracing/#command-processor-module-attributes)
- predicting the next action
- running the action
- advancing [flows](https://rasa.com/docs/reference/integrations/tracing/#flow-executor-module-attributes)
- searching documents in [vector stores](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/#vector-store) for enterprise search
- generating LLM answers by policies e.g. [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/)
- tracing [prompt token usage](https://rasa.com/docs/reference/integrations/tracing/#tracing-prompt-token-usage)
- retrieving and saving the tracker
- locking the conversation
- publishing to the event broker
- making requests to the action server or nlg server
- passing the trace context to the action server

#### Tracing prompt token usage[​](https://rasa.com/docs/reference/integrations/tracing/#tracing-prompt-token-usage 'Direct link to Tracing prompt token usage')

New in 3.8

Tracing prompt token usage for OpenAI models is available starting with version `3.8.0`.

Tracing prompt token usage is available for the following classes if you're using OpenAI models:

- `CompactLLMCommandGenerator` class
- `SearchReadyLLMCommandGenerator` class
- `EnterpriseSearchPolicy` class
- `ContextualResponseRephraser` class

The prompt token usage is captured as part of the trace context and can be used to monitor the usage of prompt tokens in the LLM answer generation process. This is only captured if one of instrumented classes mentioned above is configured to enable capturing the length of the prompt tokens. For example, the `CompactLLMCommandGenerator` can be configured to trace the length of the prompt tokens by setting the `trace_prompt_tokens` attribute to `true` in the `config.yml` file:

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    trace_prompt_tokens: true
```

It is highly recommended to enable tracing of prompt tokens only in development and not in production, because it could increase assistant response latency.

#### `Agent` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#agent-attributes 'Direct link to agent-attributes')

Tracing the `Agent` instance handling a message captures the following attributes:

- `input_channel`: the name of the channel connector
- `sender_id`: the conversation id
- `model_id`: a unique identifier for the model
- `model_name`: the model name

#### `MessageProcessor` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#messageprocessor-attributes 'Direct link to messageprocessor-attributes')

The following `MessageProcessor` attributes are extracted during the tracing:

- `number_of_events`: number of events in tracker
- `action_name`: the name of the predicted and executed action
- `sender_id`: the conversation id of the `DialogueStateTracker` object
- `message_id`: the unique message id

The latter three attributes are also injected in the trace context that gets passed to the requests made to the custom action server.

#### `TrackerStore` & `LockStore` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#trackerstore--lockstore-attributes 'Direct link to trackerstore--lockstore-attributes')

Observable `TrackerStore` and `LockStore` attributes include:

- `number_of_streamed_events`: number of new events to stream
- `broker_class`: the `EventBroker` on which the new events are published
- `lock_store_class`: Name of lock store used to lock conversations while messages are actively processed

#### `CompactLLMCommandGenerator` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#compactllmcommandgenerator-attributes 'Direct link to compactllmcommandgenerator-attributes')

The following attributes are captured as part of the trace context of the `CompactLLMCommandGenerator`:

- `class_name`: the name of the instrumented component class
- `llm_model`: the name of the LLM used
- `llm_type`: the type of LLM used
- `embeddings`: the embeddings used
- `llm_temperature`: the temperature used for LLM answer generation
- `request_timeout`: the timeout for the LLM request
- `llm_engine`: the engine used for LLM answer generation
- `len_prompt_tokens`: the token length of the prompt (optional, only supported for OpenAI models). To enable this attribute, see instructions in the [Tracing prompt token usage](https://rasa.com/docs/reference/integrations/tracing/#tracing-prompt-token-usage) section.

#### `SearchReadyCommandGenerator` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#searchreadycommandgenerator-attributes 'Direct link to searchreadycommandgenerator-attributes')

The following attributes are captured as part of the trace context of the `SearchReadyCommandGenerator`:

- `class_name`: the name of the instrumented component class
- `llm_model`: the name of the LLM used
- `llm_type`: the type of LLM used
- `embeddings`: the embeddings used
- `llm_temperature`: the temperature used for LLM answer generation
- `request_timeout`: the timeout for the LLM request
- `llm_engine`: the engine used for LLM answer generation
- `len_prompt_tokens`: the token length of the prompt (optional, only supported for OpenAI models). To enable this attribute, see instructions in the [Tracing prompt token usage](https://rasa.com/docs/reference/integrations/tracing/#tracing-prompt-token-usage) section.

#### `NLUCommandAdapter` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#nlucommandadapter-attributes 'Direct link to nlucommandadapter-attributes')

New in 3.8

Tracing the described `NLUCommandAdapter` attributes is available starting with version `3.8.0`.

The following attributes are captured as part of the trace context of the `NLUCommandAdapter`:

- `commands`: the predicted commands
- `intent`: the predicted intent of the user message that the `NLUCommandAdapter` receives as input

#### Command Processor Module Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#command-processor-module-attributes 'Direct link to Command Processor Module Attributes')

New in 3.8

Tracing the described command processor module attributes is available starting with version `3.8.0`.

The following attributes are captured as part of the trace context of the command processor module functions:

1. `execute_commands` function:
 - `number_of_events`: the number of events in the tracker
 - `sender_id`: the conversation id of the `DialogueStateTracker` object
2. `validate_state_of_commands` function:
 - `cleaned_up_commands`: list of cleaned up commands
3. `clean_up_commands` function:
 - `commands`: list of originally parsed commands from the LLM answer
 - `current_context`: the current context of the dialogue stack
4. `remove_duplicated_set_slots` function:
 - `resulting_events`: list of events prior to removing duplicated set slot events; note that slot values are removed to prevent PII leakage

#### Flow Executor Module Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#flow-executor-module-attributes 'Direct link to Flow Executor Module Attributes')

New in 3.8

Tracing the described flow executor module attributes is available starting with version `3.8.0`.

The following attributes are captured as part of the trace context of the flow executor module functions:

1. `advance_flow` function:
 - `available_actions`: list of available actions
 - `current_context`: the current context of the dialogue stack
2. `advance_flows_until_next_action` function:
 - `action_name`: the name of the action to be executed
 - `score`: the score of the executed action
 - `metadata`: the prediction metadata
 - `events`: list of event names if available
3. `run_step` function:
 - `step_custom_id`: the custom id of the step if available
 - `step_description`: the description of the step if available
 - `current_flow_id`: the id of the current flow
 - `current_context`: the current context of the dialogue stack

#### `Policy` subclasses attributes[​](https://rasa.com/docs/reference/integrations/tracing/#policy-subclasses-attributes 'Direct link to policy-subclasses-attributes')

New in 3.8

Tracing the described `Policy` subclasses' attributes is available starting with version `3.8.0`.

The following attributes are captured as part of the trace context of subclasses of the `Policy` interface, e.g. `FlowPolicy`, `EnterpriseSearchPolicy`:

- `priority`: the priority of the policy which made the prediction
- `events`: a list of event names which are applied independent of whether the policy wins against other policies or not
- `optional_events`: a list of optional event names if available else `None` - these events are applied if the policy wins against other policies
- `is_end_to_end_prediction`: a boolean indicating if the prediction used the text of the user message instead of the intent
- `is_no_user_prediction`: a boolean indicating if the prediction uses neither the text of the user message nor the intent
- `diagnostic_data`: intermediate results or other information that is not necessary for Rasa to function, but intended for debugging and fine-tuning purposes
- `action_metadata`: additional metadata that can be passed by policies

#### `EnterpriseSearchPolicy` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#enterprisesearchpolicy-attributes 'Direct link to enterprisesearchpolicy-attributes')

New in 3.8

Tracing the described `EnterpriseSearchPolicy` attributes is available starting with version `3.8.0`.

The `EnterpriseSearchPolicy._generate_llm_answer` method captures the same attributes as the [`CompactLLMCommandGenerator` class](https://rasa.com/docs/reference/integrations/tracing/#compactllmcommandgenerator-attributes).

#### `InformationRetrieval` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#informationretrieval-attributes 'Direct link to informationretrieval-attributes')

New in 3.8

Tracing the described `InformationRetrieval` subclasses' attributes is available starting with version `3.8.0`.

The following attributes are captured as part of the trace context of the `InformationRetrieval` subclasses, e.g. `Milvus_Store`, `Qdrant_Store`:

- `query`: the query used to search the vector store
- `document_metadata`: the metadata of the documents retrieved from the vector store

#### `EndpointConfig` Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#endpointconfig-attributes 'Direct link to endpointconfig-attributes')

New in 3.8

Tracing the described `EndpointConfig` attributes is available starting with version `3.8.0`.

The following attributes are captured as part of the trace context of the `EndpointConfig`:

- `url`: the url of the endpoint
- `request_body_size_in_bytes`: the size of the request body in bytes

## Tracing in the Action Server[​](https://rasa.com/docs/reference/integrations/tracing/#tracing-in-the-action-server 'Direct link to Tracing in the Action Server')

API Requests are traced as they flow through the action server by instrumenting the webhook that receives custom actions and other classes involved in the execution of custom actions.

New in 3.8

Additional classes are now instrumented to improve tracing in the action server.

The following classes are instrumented;

- [ValidationAction](https://rasa.com/docs/reference/integrations/action-server/validation-action/#validationaction-class): the base class for custom actions extracting and validating slots.
- [ActionExecutor](https://rasa.com/docs/reference/integrations/tracing/#action-executor-attributes) - the class that executes the custom actions.

### Webhook Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#webhook-attributes 'Direct link to Webhook Attributes')

The following attributes are captured as part of the trace context of the webhook that receives custom actions;

- `http.method`: the http method used to make the request
- `http.route`: the endpoint of the request
- `next_action`: the name of the next action to be executed
- `version`: the rasa version used
- `sender_id`: the id of the conversation
- `message_id`: the unique message id

### Action Executor Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#action-executor-attributes 'Direct link to Action Executor Attributes')

The following attributes are captured as part of the trace context of the action executor;

- `action_name`: the name of the action to be executed
- `sender_id`: the id of the conversation
- `events`: a list of returned events
- `slots`: a list of filled slots by the executed custom action
- `utters`: a list of executed utterances

### Slot Validation Action Attributes[​](https://rasa.com/docs/reference/integrations/tracing/#slot-validation-action-attributes 'Direct link to Slot Validation Action Attributes')

The following attributes are captured as part of the trace context of Slot Validation Actions;

- `class_name`: the name of the instrumented component class
- `action_name`: the name of the action to be executed
- `sender_id`: the id of the conversation
- `events`: a list of returned events
- `slots`: a list of filled slots by the executed custom action
- `utters`: a list of executed utterances
- `message_count`: the number of messages
- `slots_to_validate`: a list of recently filled slots to validate

### Debugging custom actions performance[​](https://rasa.com/docs/reference/integrations/tracing/#debugging-custom-actions-performance 'Direct link to Debugging custom actions performance')

New in 3.8

You can now continue tracing the request further along your custom actions code.

It is now possible to debug the performance of your custom actions by tracing specific parts of your custom actions code. This can be achieved by creating spans to trace the execution of these parts.

In order to create more spans, you can retrieve the [tracer object](https://opentelemetry.io/docs/languages/python/instrumentation/#acquire-tracer) from the `ActionExecutorTracerRegister` component.

actions.py

```
# import the ActionExecutorTracerRegister componentfrom rasa_sdk.tracing.tracer_register import ActionExecutorTracerRegister
```

To create a span as documented in the [OTEL documentation](https://opentelemetry.io/docs/languages/python/instrumentation/#creating-spans), corresponding to traces from a specific part of your custom actions code, you can embed the following code snippet in the `run` method of the custom action:

actions.py

```
# retrieve the tracer objecttracer = ActionExecutorTracerRegister().get_tracer()# create a spanwith tracer.start_as_current_span("span_name") as span:  # your code here  span.set_attribute("attribute_name", "attribute_value")
```

For example, a complete custom action that implements a custom span is shown below:

actions.py

```
import requestsimport jsonfrom rasa_sdk import Actionfrom rasa_sdk.tracing.tracer_register import ActionExecutorTracerRegisterclass ActionCheckSufficientFunds(Action):  def name(self):    return "action_check_sufficient_funds"  def run(    self,    dispatcher: CollectingDispatcher,    tracker: Tracker,    domain: Dict[Text, Any]  ) -> List[Dict[Text, Any]]:    tracer = ActionExecutorTracerRegister().get_tracer()    with tracer.start_as_current_span("span_name"):      balance = 1000 # hardcoded balance from tutorial purposes      transfer_amount = tracker.get_slot("amount")      has_sufficient_funds = transfer_amount <= balance      # set trace attributes      span.set_attribute("has_sufficient_funds", has_sufficient_funds)      return [SlotSet("has_sufficient_funds", has_sufficient_funds)]
```

Enabling and disabling tracing in the action server is also done in the same way as described [below](https://rasa.com/docs/reference/integrations/tracing/#enabling--disabling). The same Tracing Backends/Collectors listed [above](https://rasa.com/docs/reference/integrations/tracing/#supported-tracing-backendscollectors) are also supported for the action server. See [Configuring a Tracing Backend or Collector](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector) for further instructions.

### Enabling / Disabling[​](https://rasa.com/docs/reference/integrations/tracing/#enabling--disabling 'Direct link to Enabling / Disabling')

Tracing is automatically enabled in Rasa by [configuring a supported tracing backend](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector). No further action is required to enable tracing.

You can disable tracing by leaving the `tracing:` configuration key empty in your endpoints file.

- [Supported Tracing Backends/Collectors](https://rasa.com/docs/reference/integrations/tracing/#supported-tracing-backendscollectors)
- [Rasa Channels](https://rasa.com/docs/reference/integrations/tracing/#rasa-channels)
- [Action Server](https://rasa.com/docs/reference/integrations/tracing/#action-server)
- [Questions Tracing Can Help Answer](https://rasa.com/docs/reference/integrations/tracing/#questions-tracing-can-help-answer)
- [Configuring a Tracing Backend or Collector](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector)
 - [Jaeger](https://rasa.com/docs/reference/integrations/tracing/#jaeger)
 - [OTEL Collector](https://rasa.com/docs/reference/integrations/tracing/#otel-collector)
- [Traced Events](https://rasa.com/docs/reference/integrations/tracing/#traced-events)
 - [Model Training](https://rasa.com/docs/reference/integrations/tracing/#model-training)
 - [Message Handling](https://rasa.com/docs/reference/integrations/tracing/#message-handling)
- [Tracing in the Action Server](https://rasa.com/docs/reference/integrations/tracing/#tracing-in-the-action-server)
 - [Webhook Attributes](https://rasa.com/docs/reference/integrations/tracing/#webhook-attributes)
 - [Action Executor Attributes](https://rasa.com/docs/reference/integrations/tracing/#action-executor-attributes)
 - [Slot Validation Action Attributes](https://rasa.com/docs/reference/integrations/tracing/#slot-validation-action-attributes)
 - [Debugging custom actions performance](https://rasa.com/docs/reference/integrations/tracing/#debugging-custom-actions-performance)
 - [Enabling / Disabling](https://rasa.com/docs/reference/integrations/tracing/#enabling--disabling)