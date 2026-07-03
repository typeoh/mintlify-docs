# Source: https://rasa.com/docs/reference/telemetry/events

On this page

Telemetry events are only reported if telemetry is enabled. A detailed explanation on the reasoning behind collecting optional telemetry events can be found in our [telemetry documentation](https://rasa.com/docs/reference/telemetry/).

## CALM[​](https://rasa.com/docs/reference/telemetry/events/#calm 'Direct link to CALM')

### Response Rephrased[​](https://rasa.com/docs/reference/telemetry/events/#response-rephrased 'Direct link to Response Rephrased')

backend Triggered when a response is rephrased. Event properties:

- `rephrase_all` _(boolean)_: True if the rephraser is setup to rephrase all responses.
- `custom_prompt_template` _(string)_: A custom prompt template, if specified.
- `llm_type` _(string)_: The type of LLM.
- `llm_model` _(string)_: The model name of the LLM.

### Intentless Policy Training Started[​](https://rasa.com/docs/reference/telemetry/events/#intentless-policy-training-started 'Direct link to Intentless Policy Training Started')

backend Triggered when a user trains the IntentlessPolicy. Event properties:

### Intentless Policy Training Completed[​](https://rasa.com/docs/reference/telemetry/events/#intentless-policy-training-completed 'Direct link to Intentless Policy Training Completed')

backend Triggered when the training of IntentlessPolicy completed. Event properties:

- `embeddings_type` _(string)_: The type of embeddings.
- `embeddings_model` _(string)_: The model name for embeddings.
- `llm_type` _(string)_: The type of LLM.
- `llm_model` _(string)_: The model name of the LLM.

### Intentless Policy Predicted[​](https://rasa.com/docs/reference/telemetry/events/#intentless-policy-predicted 'Direct link to Intentless Policy Predicted')

backend Triggered when the IntentlessPolicy makes a prediction. Event properties:

- `embeddings_type` _(string)_: The type of embeddings.
- `embeddings_model` _(string)_: The model name for embeddings.
- `llm_type` _(string)_: The type of LLM.
- `llm_model` _(string)_: The model name of the LLM.
- `score` _(number)_: The prediction score.

### Enterprise Search Policy Training Started[​](https://rasa.com/docs/reference/telemetry/events/#enterprise-search-policy-training-started 'Direct link to Enterprise Search Policy Training Started')

backend Triggered when a user trains the EnterpriseSearchPolicy. Event properties:

### Enterprise Search Policy Training Completed[​](https://rasa.com/docs/reference/telemetry/events/#enterprise-search-policy-training-completed 'Direct link to Enterprise Search Policy Training Completed')

backend Triggered when the training of EnterpriseSearchPolicy completed. Event properties:

- `vector_store` _(string)_: The vector store.
- `embeddings_type` _(string)_: The type of embeddings.
- `embeddings_model` _(string)_: The model name for embeddings.
- `llm_type` _(string)_: The type of LLM.
- `llm_model` _(string)_: The model name of the LLM.
- `citation_enabled` _(boolean)_: Whether source\_citation is enabled or not.

### Enterprise Search Policy Predicted[​](https://rasa.com/docs/reference/telemetry/events/#enterprise-search-policy-predicted 'Direct link to Enterprise Search Policy Predicted')

backend Triggered when the EnterpriseSearchPolicy makes a prediction. Event properties:

- `vector_store` _(string)_: The vector store.
- `embeddings_type` _(string)_: The type of embeddings.
- `embeddings_model` _(string)_: The model name for embeddings.
- `llm_type` _(string)_: The type of LLM.
- `llm_model` _(string)_: The model name of the LLM.
- `citation_enabled` _(boolean)_: Whether source\_citation is enabled or not.

### PII Management in CALM Enabled[​](https://rasa.com/docs/reference/telemetry/events/#pii-management-in-calm-enabled 'Direct link to PII Management in CALM Enabled')

backend Triggered when PII management in CALM is enabled. Event properties:

- `num_total_rules` _(integer)_: Total number of anonymization rules defined.
- `redact_count` _(integer)_: Number of redact rules defined.
- `mask_count` _(integer)_: Number of mask rules defined.
- `stream_pii` _(boolean)_: Whether streaming PII in un-anonymized events to event brokers is enabled or not.
- `tracker_store_anonymization_enabled` _(boolean)_: Whether anonymization of eligible trackers in the tracker store is enabled or not.
- `tracker_store_deletion_enabled` _(boolean)_: Whether deletion of eligible trackers in the tracker store is enabled or not.
- `anonymization_cron_trigger` _(string)_: Cron trigger for anonymization of eligible tracker sessions in the tracker store.
- `deletion_cron_trigger` _(string)_: Cron trigger for deletion of eligible tracker sessions in the tracker store.

## End-to-End Testing[​](https://rasa.com/docs/reference/telemetry/events/#end-to-end-testing 'Direct link to End-to-End Testing')

### E2E Test Run Started[​](https://rasa.com/docs/reference/telemetry/events/#e2e-test-run-started 'Direct link to E2E Test Run Started')

backend Triggered when end-to-end testing has been started. Event properties:

- `number_of_test_cases` _(integer)_: Number of test cases to be run.
- `number_of_fixtures` _(integer)_: Number of fixtures defined globally.
- `uses_fixtures` _(boolean)_: Indicates if any fixtures have been defined globally.
- `uses_metadata` _(boolean)_: Indicates if any metadata has been defined globally.
- `number_of_metadata` _(integer)_: Number of metadata defined globally.
- `uses_assertions` _(boolean)_: Indicates if any assertions have been defined in test cases.
- `flow_started_count` _(integer)_: Number of flow\_started assertion type used in the test run.
- `flow_completed_count` _(integer)_: Number of flow\_completed assertion type used in the test run.
- `flow_cancelled_count` _(integer)_: Number of flow\_cancelled assertion type used in the test run.
- `pattern_clarification_count` _(integer)_: Number of pattern\_clarification assertion type used in the test run.
- `action_executed_count` _(integer)_: Number of action\_executed assertion type used in the test run.
- `slot_was_set_count` _(integer)_: Number of slot\_was\_set assertion type used in the test run.
- `slot_was_not_set_count` _(integer)_: Number of slot\_was\_not\_set assertion type used in the test run.
- `bot_uttered_count` _(integer)_: Number of bot\_uttered assertion type used in the test run.
- `generative_response_is_relevant_count` _(integer)_: Number of generative\_response\_is\_relevant assertion type used in the test run.
- `generative_response_is_grounded_count` _(integer)_: Number of generative\_response\_is\_grounded assertion type used in the test run.

## Model Training[​](https://rasa.com/docs/reference/telemetry/events/#model-training 'Direct link to Model Training')

### Training Started[​](https://rasa.com/docs/reference/telemetry/events/#training-started 'Direct link to Training Started')

backend A training of a Rasa machine learning model got started. The event provides information on aggregated training data statistics. Event properties:

- `language` _(string)_: Language model is trained with, e.g. 'en'.
- `training_id` _(string)_: Generated unique identifier for this training.
- `type` _(string)_: Type of model trained, either 'nlu', 'core' or 'rasa'.
- `pipeline` _(undefined)_: List of the pipeline configurations used for training.
- `policies` _(undefined)_: List of the policy configurations used for training.
- `train_schema` _(undefined)_: Training graph schema for graph recipe
- `predict_schema` _(undefined)_: Predict graph schema for graph recipe
- `num_intent_examples` _(integer)_: Number of NLU examples.
- `num_entity_examples` _(integer)_: Number of entity examples.
- `num_actions` _(integer)_: Number of actions defined in the domain.
- `num_templates` _(integer)_: Number of templates or responses defined in the domain.
- `num_conditional_response_variations` _(integer)_: Number of conditional response variations defined in the domain.
- `num_slot_mappings` _(integer)_: Number of total slot mappings defined in the domain.
- `num_custom_slot_mappings` _(integer)_: Number of custom slot mappings defined in the domain.
- `num_conditional_slot_mappings` _(integer)_: Number of slot mappings with conditions attached defined in the domain.
- `num_slots` _(integer)_: Number of slots defined in the domain.
- `num_forms` _(integer)_: Number of forms defined in the domain.
- `num_intents` _(integer)_: Number of intents defined in the domain.
- `num_entities` _(integer)_: Number of entities defined in the domain.
- `num_story_steps` _(integer)_: Number of story steps available.
- `num_lookup_tables` _(integer)_: Number of different lookup tables.
- `num_synonyms` _(integer)_: Total number of entity synonyms defined.
- `num_regexes` _(integer)_: Total number of regexes defined.
- `is_finetuning` _(boolean)_: True if a model is trained by finetuning an existing model.
- `recipe` _(string)_: Recipe used in training the model, either 'default.v1' or 'graph.v1'.
- `num_flows` _(integer)_: Number of flows.
- `num_flows_with_nlu_trigger` _(integer)_: Number of flows that have an NLU trigger defined.
- `num_flows_with_flow_guards` _(integer)_: Number of flows that have a flow guard condition.
- `num_flows_with_not_startable_flow_guards` _(integer)_: Number of flows with the flow guard condition 'if: False'.
- `num_collect_steps` _(integer)_: Number of collect steps in flows.
- `num_collect_steps_with_separate_utter` _(integer)_: Number of collect steps which have a different utterance defined.
- `num_collect_steps_with_rejections` _(integer)_: Number of collect steps with rejections included.
- `num_collect_steps_with_not_reset_after_flow_ends` _(integer)_: Number of collect steps with 'reset\_after\_flow\_ends' set to 'False'.
- `num_set_slot_steps` _(integer)_: Number of set slot steps in flows.
- `num_link_steps` _(integer)_: Number of link steps in flows.
- `num_call_steps` _(integer)_: Number of call steps in flows.
- `max_depth_of_if_construct` _(integer)_: Maximum depth of an if construct.
- `num_shared_slots_between_flows` _(integer)_: Number of slots being shared across flows.
- `llm_command_generator_model_name` _(string)_: The name of the model used in the 'LLMCommandGenerator'.
- `llm_command_generator_custom_prompt_used` _(boolean)_: True, if a custom prompt was configured for the 'LLMCommandGenerator', False otherwise.
- `multi_step_llm_command_generator_custom_handle_flows_prompt_used` _(boolean)_: True, if a custom prompt was configured for handling flows in the 'MultiStepLLMCommandGenerator', False otherwise.
- `multi_step_llm_command_generator_custom_fill_slots_prompt_used` _(boolean)_: True, if a custom prompt was configured for filling slots in the 'MultiStepLLMCommandGenerator', False otherwise.
- `flow_retrieval_enabled` _(boolean)_: True, if flow retrieval is configured for the 'LLMCommandGenerator', False otherwise.
- `flow_retrieval` _(string)_: The name of the embedding model used by flow retrieval within 'LLMCommandGenerator'.
- `agents` _(object)_: Agent configuration and usage information including MCP servers, agents, and their usage in flows.

### Training Completed[​](https://rasa.com/docs/reference/telemetry/events/#training-completed 'Direct link to Training Completed')

backend The training of a Rasa machine learning model finished. The event provides information about the resulting model. Event properties:

- `training_id` _(string)_: Generated unique identifier for this training. Can be used to join with 'Training Started'.
- `type` _(string)_: Type of model trained, either 'nlu', 'core' or 'rasa'.
- `runtime` _(integer)_: The time in seconds it took to train the model.

## Model Testing[​](https://rasa.com/docs/reference/telemetry/events/#model-testing 'Direct link to Model Testing')

### Model Core Tested[​](https://rasa.com/docs/reference/telemetry/events/#model-core-tested 'Direct link to Model Core Tested')

backend Triggered when a Core model is getting tested. Event properties:

- `project` _(string,null)_: Fingerprint of the project the tested model got trained in.
- `num_story_steps` _(integer)_: Number of story steps used for testing
- `end_to_end` _(boolean)_: Indicates if tests are running in end-to-end mode, testing message handling and dialogue handling at the same time

### Model NLU Tested[​](https://rasa.com/docs/reference/telemetry/events/#model-nlu-tested 'Direct link to Model NLU Tested')

backend Triggered when an NLU model is getting tested. Event properties:

- `num_intent_examples` _(integer)_: Number of NLU examples.
- `num_entity_examples` _(integer)_: Number of entity examples.
- `num_lookup_tables` _(integer)_: Number of different lookup tables.
- `num_synonyms` _(integer)_: Total number of entity synonyms defined.
- `num_regexes` _(integer)_: Total number of regexes defined.

## Model Serving[​](https://rasa.com/docs/reference/telemetry/events/#model-serving 'Direct link to Model Serving')

### Interactive Learning Started[​](https://rasa.com/docs/reference/telemetry/events/#interactive-learning-started 'Direct link to Interactive Learning Started')

backend Triggered when an interactive learning session got started. Event properties:

- `skip_visualization` _(boolean)_: Whether the visualization of stories should be shown during the interactive learning session
- `save_in_e2e` _(boolean)_: Whether the data should be stored in end-to-end format

### Server Started[​](https://rasa.com/docs/reference/telemetry/events/#server-started 'Direct link to Server Started')

backend Triggered when a Rasa server gets started. Event properties:

- `input_channels` _(array)_: Names of the used input channels
- `api_enabled` _(boolean)_: Indicator if the API is enabled or if only the input channel is running
- `number_of_workers` _(integer)_: Amount of Sanic workers started as part of the server
- `endpoints_nlg` _(string,null)_: Type of the used NLG endpoint
- `endpoints_nlu` _(string,null)_: Type of the used NLU endpoint
- `endpoints_action_server` _(string,null)_: Type of the used action server
- `endpoints_model_server` _(string,null)_: Type of the used model server
- `endpoints_tracker_store` _(string,null)_: Type of the used tracker store
- `endpoints_lock_store` _(string,null)_: Type of the used lock store
- `endpoints_event_broker` _(string,null)_: Type of the used event broker
- `project` _(string,null)_: Hash of the deployed model the server is started with

### Shell Started[​](https://rasa.com/docs/reference/telemetry/events/#shell-started 'Direct link to Shell Started')

backend Triggered when a shell session is started to talk to a trained bot. Event properties:

- `type` _(string)_: Type of the model, either 'nlu', 'core' or 'rasa'.

## Markers Extraction[​](https://rasa.com/docs/reference/telemetry/events/#markers-extraction 'Direct link to Markers Extraction')

### Markers Extraction Initiated[​](https://rasa.com/docs/reference/telemetry/events/#markers-extraction-initiated 'Direct link to Markers Extraction Initiated')

backend Triggered when marker extraction has been initiated. Event properties:

- `strategy` _(string)_: Strategy to use when selecting trackers to extract from.
- `only_extract` _(boolean)_: Indicates if path to write out statistics hasn't been specified.
- `seed` _(boolean)_: The seed to initialise the random number generator for use with the 'sample' strategy.
- `count` _(integer,null)_: Number of trackers to extract from (for any strategy except 'all').

### Markers Extracted[​](https://rasa.com/docs/reference/telemetry/events/#markers-extracted 'Direct link to Markers Extracted')

backend Triggered when markers have been extracted. Event properties:

- `trackers_count` _(integer)_: Number of processed trackers.

### Markers Parsed[​](https://rasa.com/docs/reference/telemetry/events/#markers-parsed 'Direct link to Markers Parsed')

backend Triggered when markers have been successfully parsed. Event properties:

- `marker_count` _(integer)_: Number of parsed markers.
- `max_depth` _(integer)_: Maximum depth of the parsed markers.
- `branching_factor` _(integer)_: Maximum number of children of any of the parsed markers.

### Markers Statistics Computed[​](https://rasa.com/docs/reference/telemetry/events/#markers-statistics-computed 'Direct link to Markers Statistics Computed')

backend Triggered when marker statistics have been computed. Event properties:

- `trackers_count` _(integer)_: Number of processed trackers.

## Data Handling[​](https://rasa.com/docs/reference/telemetry/events/#data-handling 'Direct link to Data Handling')

### Training Data Split[​](https://rasa.com/docs/reference/telemetry/events/#training-data-split 'Direct link to Training Data Split')

backend Triggered when training data gets split. Event properties:

- `fraction` _(number)_: Percentage of the data which goes into training data (the rest goes into the test set).
- `type` _(string)_: Type of data, either 'nlu', 'core' or 'rasa'.

### Training Data Validated[​](https://rasa.com/docs/reference/telemetry/events/#training-data-validated 'Direct link to Training Data Validated')

backend Triggered when training data gets validated. Event properties:

- `validation_success` _(boolean)_: whether the validation was successful

### Training Data Converted[​](https://rasa.com/docs/reference/telemetry/events/#training-data-converted 'Direct link to Training Data Converted')

backend Triggered when training data gets converted. Event properties:

- `output_format` _(string)_: target format of the converter
- `type` _(string)_: Type of data, either 'nlu', 'core', 'config' or 'nlg'.

### Tracker Exported[​](https://rasa.com/docs/reference/telemetry/events/#tracker-exported 'Direct link to Tracker Exported')

backend Triggered when conversations get exported from a tracker store through an event broker. Event properties:

- `event_broker` _(string)_: Name of the used event broker
- `tracker_store` _(string)_: Name of the used tracker store
- `number_of_exported_events` _(integer)_: Number of events exported through the event broker

### Story Visualization Started[​](https://rasa.com/docs/reference/telemetry/events/#story-visualization-started 'Direct link to Story Visualization Started')

backend Triggered when stories are getting visualized.

## Rasa Pro Services[​](https://rasa.com/docs/reference/telemetry/events/#rasa-pro-services 'Direct link to Rasa Pro Services')

### Analytics Started[​](https://rasa.com/docs/reference/telemetry/events/#analytics-started 'Direct link to Analytics Started')

backend Triggered when the Analytics pipeline is started. Event properties:

- `consumers` _(number)_: The number of Kafka consumers.

### Assistant Session Started[​](https://rasa.com/docs/reference/telemetry/events/#assistant-session-started 'Direct link to Assistant Session Started')

backend Triggered when the Analytics pipeline detects a new session start. Event properties:

## Miscellaneous[​](https://rasa.com/docs/reference/telemetry/events/#miscellaneous 'Direct link to Miscellaneous')

### Telemetry Disabled[​](https://rasa.com/docs/reference/telemetry/events/#telemetry-disabled 'Direct link to Telemetry Disabled')

backend Triggered when telemetry reporting gets disabled. Last event sent before disabling telemetry. This event is not sent, if the user never enabled telemetry reporting before deactivating it.

### Project Created[​](https://rasa.com/docs/reference/telemetry/events/#project-created 'Direct link to Project Created')

backend Triggered when a project is created using rasa init. Event properties:

- `init_directory` _(string)_: Hash of the directory path the project is created in

- [CALM](https://rasa.com/docs/reference/telemetry/events/#calm)
 - [Response Rephrased](https://rasa.com/docs/reference/telemetry/events/#response-rephrased)
 - [Intentless Policy Training Started](https://rasa.com/docs/reference/telemetry/events/#intentless-policy-training-started)
 - [Intentless Policy Training Completed](https://rasa.com/docs/reference/telemetry/events/#intentless-policy-training-completed)
 - [Intentless Policy Predicted](https://rasa.com/docs/reference/telemetry/events/#intentless-policy-predicted)
 - [Enterprise Search Policy Training Started](https://rasa.com/docs/reference/telemetry/events/#enterprise-search-policy-training-started)
 - [Enterprise Search Policy Training Completed](https://rasa.com/docs/reference/telemetry/events/#enterprise-search-policy-training-completed)
 - [Enterprise Search Policy Predicted](https://rasa.com/docs/reference/telemetry/events/#enterprise-search-policy-predicted)
 - [PII Management in CALM Enabled](https://rasa.com/docs/reference/telemetry/events/#pii-management-in-calm-enabled)
- [End-to-End Testing](https://rasa.com/docs/reference/telemetry/events/#end-to-end-testing)
 - [E2E Test Run Started](https://rasa.com/docs/reference/telemetry/events/#e2e-test-run-started)
- [Model Training](https://rasa.com/docs/reference/telemetry/events/#model-training)
 - [Training Started](https://rasa.com/docs/reference/telemetry/events/#training-started)
 - [Training Completed](https://rasa.com/docs/reference/telemetry/events/#training-completed)
- [Model Testing](https://rasa.com/docs/reference/telemetry/events/#model-testing)
 - [Model Core Tested](https://rasa.com/docs/reference/telemetry/events/#model-core-tested)
 - [Model NLU Tested](https://rasa.com/docs/reference/telemetry/events/#model-nlu-tested)
- [Model Serving](https://rasa.com/docs/reference/telemetry/events/#model-serving)
 - [Interactive Learning Started](https://rasa.com/docs/reference/telemetry/events/#interactive-learning-started)
 - [Server Started](https://rasa.com/docs/reference/telemetry/events/#server-started)
 - [Shell Started](https://rasa.com/docs/reference/telemetry/events/#shell-started)
- [Markers Extraction](https://rasa.com/docs/reference/telemetry/events/#markers-extraction)
 - [Markers Extraction Initiated](https://rasa.com/docs/reference/telemetry/events/#markers-extraction-initiated)
 - [Markers Extracted](https://rasa.com/docs/reference/telemetry/events/#markers-extracted)
 - [Markers Parsed](https://rasa.com/docs/reference/telemetry/events/#markers-parsed)
 - [Markers Statistics Computed](https://rasa.com/docs/reference/telemetry/events/#markers-statistics-computed)
- [Data Handling](https://rasa.com/docs/reference/telemetry/events/#data-handling)
 - [Training Data Split](https://rasa.com/docs/reference/telemetry/events/#training-data-split)
 - [Training Data Validated](https://rasa.com/docs/reference/telemetry/events/#training-data-validated)
 - [Training Data Converted](https://rasa.com/docs/reference/telemetry/events/#training-data-converted)
 - [Tracker Exported](https://rasa.com/docs/reference/telemetry/events/#tracker-exported)
 - [Story Visualization Started](https://rasa.com/docs/reference/telemetry/events/#story-visualization-started)
- [Rasa Pro Services](https://rasa.com/docs/reference/telemetry/events/#rasa-pro-services)
 - [Analytics Started](https://rasa.com/docs/reference/telemetry/events/#analytics-started)
 - [Assistant Session Started](https://rasa.com/docs/reference/telemetry/events/#assistant-session-started)
- [Miscellaneous](https://rasa.com/docs/reference/telemetry/events/#miscellaneous)
 - [Telemetry Disabled](https://rasa.com/docs/reference/telemetry/events/#telemetry-disabled)
 - [Project Created](https://rasa.com/docs/reference/telemetry/events/#project-created)