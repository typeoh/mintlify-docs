# Source: https://rasa.com/docs/pro/build/configuring-assistant

On this page

You can customise many aspects of how your assistant project works by modifying the following files: `config.yml`, `endpoints.yml`, and `domain.yml`.

## Configuration File[​](https://rasa.com/docs/pro/build/configuring-assistant/#configuration-file 'Direct link to Configuration File')

The `config.yml` file defines how your Rasa assistant processes user messages. It specifies which components, policies, and language settings your assistant will use. Here's the minimal configuration required to run a CALM assistant:

config.yml

```
recipe: default.v1language: enpipeline:  - name: CompactLLMCommandGeneratorpolicies:  - name: FlowPolicy
```

Below are the main parameters you can configure.

### Recipe[​](https://rasa.com/docs/pro/build/configuring-assistant/#recipe 'Direct link to Recipe')

- Rasa provides a default graph recipe: `default.v1`. For most projects, the default value is sufficient.
- In case you're running ML experiments or ablation studies and want to add a custom graph recipe, [this guide has you covered](https://rasa.com/docs/reference/config/components/graph-recipe/).

### Language[​](https://rasa.com/docs/pro/build/configuring-assistant/#language 'Direct link to Language')

- The `language` key sets the primary language your assistant supports. Use a [two-letter ISO 639-1 code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) (e.g., `"en"` for English).
- The `additional_languages` key lists codes of other languages your assistant supports.

👉 [Learn more about language configuration](https://rasa.com/docs/reference/config/overview/#language)

### Pipeline[​](https://rasa.com/docs/pro/build/configuring-assistant/#pipeline 'Direct link to Pipeline')

The `pipeline` section lists the components that process the latest user message and produce **commands** for the conversation. The main component in your pipeline is the `LLMCommandGenerator`.

```
  pipeline:    - name: CompactLLMCommandGenerator      llm:        model_group: openai_llm      flow_retrieval:        embeddings:          model_group: openai_embeddings      user_input:        max_characters: 420
```

👉 [See the full set of configurable parameters](https://rasa.com/docs/reference/config/components/llm-command-generators/)

### Policies[​](https://rasa.com/docs/pro/build/configuring-assistant/#policies 'Direct link to Policies')

The `policies` key lists the dialogue policies your assistant will use to progress the conversation. For CALM, you need at least the `FlowPolicy`. It doesn’t require any additional configuration parameters.

👉 [Learn more about policies](https://rasa.com/docs/reference/config/policies/overview/)

### Assistant ID[​](https://rasa.com/docs/pro/build/configuring-assistant/#assistant-id 'Direct link to Assistant ID')

The `assistant_id` key defines the unique identifier of your assistant. This ID is included in every event’s metadata, alongside the model ID. Use a distinct value to help differentiate between multiple deployed assistants.

```
  assistant_id: my_assistant
```

caution

If this required key is missing or still set to the default placeholder, a random assistant ID will be generated and added to your configuration each time you run `rasa train`.

## Endpoints[​](https://rasa.com/docs/pro/build/configuring-assistant/#endpoints 'Direct link to Endpoints')

The `endpoints.yml` file defines how your assistant connects to key services — like where to store conversations, execute custom actions, fetch trained models, or generate responses.

Below are the main parameters you can configure.

### Tracker Store — Where conversations are stored[​](https://rasa.com/docs/pro/build/configuring-assistant/#tracker-store--where-conversations-are-stored 'Direct link to Tracker Store — Where conversations are stored')

The `tracker_store` determines where Rasa keeps track of conversations. This is where your assistant remembers past interactions and makes decisions based on conversation context. You can store trackers in a file, a database (like PostgreSQL or MongoDB), or other storage backends.

👉 [How to configure tracker stores](https://rasa.com/docs/reference/integrations/tracker-stores/)

### Event Broker — Where conversation events are sent[​](https://rasa.com/docs/pro/build/configuring-assistant/#event-broker--where-conversation-events-are-sent 'Direct link to Event Broker — Where conversation events are sent')

Conversation history is comprised of **events** — every user message, action, or slot update is one. The `event_broker` sends these to other systems (e.g. for monitoring, analytics, or syncing with a data warehouse). It’s especially useful in production setups.

👉 [How to configure event brokers](https://rasa.com/docs/reference/integrations/event-brokers/)

### Action Endpoint — Where custom code runs[​](https://rasa.com/docs/pro/build/configuring-assistant/#action-endpoint--where-custom-code-runs 'Direct link to Action Endpoint — Where custom code runs')

When your assistant needs to do something dynamic — like fetching user data or making an API call — it uses custom actions. The `action_endpoint` tells Rasa where your action server is running so it can call it when needed.

👉 [How to configure action server](https://rasa.com/docs/reference/integrations/action-server/actions/)

### Models — Where trained models live[​](https://rasa.com/docs/pro/build/configuring-assistant/#models--where-trained-models-live 'Direct link to Models — Where trained models live')

The `models` section lets you configure remote model storage, such as a cloud bucket or server, where Rasa can automatically fetch the latest trained model at runtime. This is useful for CI/CD workflows where models are trained and uploaded externally.

👉 [How to configure model storage](https://rasa.com/docs/reference/integrations/model-storage/)

### Model Groups — LLM and embedding models[​](https://rasa.com/docs/pro/build/configuring-assistant/#model-groups--llm-and-embedding-models 'Direct link to Model Groups — LLM and embedding models')

The `model_groups` section is used to define LLMs and embedding models used by features like retrieval, rephraser, and command generator. You specify provider, type, and settings for each group.

👉 [How to configure model groups](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups)

### Lock Stores — Prevent processing conflicts[​](https://rasa.com/docs/pro/build/configuring-assistant/#lock-stores--prevent-processing-conflicts 'Direct link to Lock Stores — Prevent processing conflicts')

The `lock_store` manages conversation-level locks to ensure that only one message processor handles a message at a time. This prevents race conditions when multiple messages for the same user arrive close together — a common scenario in voice assistants or high-traffic setups.

Message processors are tied to Rasa processes, and deployment setup affects the lock store you should use:

- Single Rasa process (typically for development): the in-memory lock store is sufficient.
 
- Multiple Rasa processes in one pod (i.e. multiple Sanic workers): use the `RedisLockStore` or `ConcurrentRedisLockStore`.
 
- Multiple Rasa processes across multiple pods: we recommend using the `ConcurrentRedisLockStore`, as described [here](https://rasa.com/docs/reference/integrations/lock-stores/#description).
 

👉 [How to configure lock store](https://rasa.com/docs/reference/integrations/lock-stores/#custom-lock-store)

### Vector Stores — Enterprise search and flow retrieval[​](https://rasa.com/docs/pro/build/configuring-assistant/#vector-stores--enterprise-search-and-flow-retrieval 'Direct link to Vector Stores — Enterprise search and flow retrieval')

If your assistant uses Enterprise Search Policy, the `vector_store` allows you to define where the vector embeddings of the source documents are stored. It can also be used to connect to a search API that returns a set of relevant documents given a keyword or a search query.

👉 [How to configure Enterprise Search (RAG)](https://rasa.com/docs/pro/build/configuring-enterprise-search/)

👉 [How to customize flow retrieval](https://rasa.com/docs/reference/config/components/llm-command-generators/#customizing-flow-retrieval)

### NLG Server — External response generator[​](https://rasa.com/docs/pro/build/configuring-assistant/#nlg-server--external-response-generator 'Direct link to NLG Server — External response generator')

If you want the assistant’s responses to be generated dynamically by an external system (like an LLM-based server), you can configure an `nlg` endpoint. This allows you to update responses without retraining your model. To use this, the endpoint must point to an HTTP server with a `/nlg` path. For example:

```
nlg:  url: http://localhost:5055/nlg
```

👉 [How to configure NLG server](https://rasa.com/docs/reference/integrations/nlg/)

### Contextual Response Rephraser — Rephrase responses with LLMs[​](https://rasa.com/docs/pro/build/configuring-assistant/#contextual-response-rephraser--rephrase-responses-with-llms 'Direct link to Contextual Response Rephraser — Rephrase responses with LLMs')

Rasa’s built-in rephraser can automatically rewrite your templated responses using an LLM. It preserves intent and facts while making responses sound more natural or varied based on conversation context.

To enable it:

```
nlg:  type: rephrase
```

👉 [Learn more about the rephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/)

### Silence Handling — Timeout before triggering fallback[​](https://rasa.com/docs/pro/build/configuring-assistant/#silence-handling--timeout-before-triggering-fallback 'Direct link to Silence Handling — Timeout before triggering fallback')

The `silence_timeout` setting controls how long the assistant waits for a response before assuming the user is silent. Silence timeouts help your assistant handle situations where the user doesn’t respond. For now, this setting only works with voice-stream channels, such as:

- Twilio Media Streams
- Browser Audio
- Genesys
- Jambonz Stream
- Audiocodes Stream

Default is 7 seconds, but you can override the value:

```
interaction_handling:  global_silence_timeout: 7
```

👉 [Learn more about silence handling](https://rasa.com/docs/reference/config/overview/#silence-timeout-handling)

## Domain[​](https://rasa.com/docs/pro/build/configuring-assistant/#domain 'Direct link to Domain')

The `domain.yml` file defines the universe your assistant operates in — including its responses, memory (slots), and supported actions. Example:

domain.yml

```
version: "3.1"session_config:  session_expiration_time: 60  # value in minutes, 0 means no timeout  carry_over_slots_to_new_session: trueresponses:  utter_greeting:    - text: "Hello! How can I help you today?"slots:  user_name:    type: text    initial_value: nullactions:  - action_greet_user
```

### What’s in the Domain[​](https://rasa.com/docs/pro/build/configuring-assistant/#whats-in-the-domain 'Direct link to What’s in the Domain')

- [Responses](https://rasa.com/docs/reference/primitives/responses/): Templated messages your assistant can send.
- [Slots](https://rasa.com/docs/reference/primitives/slots/): Data your assistant stores about the user.
- [Actions](https://rasa.com/docs/reference/primitives/actions/): Logic or service calls your assistant can perform.
- [Session Configuration](https://rasa.com/docs/reference/config/domain/#session-configuration): Controls when conversations reset.

👉 [Learn more about Domain file](https://rasa.com/docs/reference/config/domain/)

- [Configuration File](https://rasa.com/docs/pro/build/configuring-assistant/#configuration-file)
 - [Recipe](https://rasa.com/docs/pro/build/configuring-assistant/#recipe)
 - [Language](https://rasa.com/docs/pro/build/configuring-assistant/#language)
 - [Pipeline](https://rasa.com/docs/pro/build/configuring-assistant/#pipeline)
 - [Policies](https://rasa.com/docs/pro/build/configuring-assistant/#policies)
 - [Assistant ID](https://rasa.com/docs/pro/build/configuring-assistant/#assistant-id)
- [Endpoints](https://rasa.com/docs/pro/build/configuring-assistant/#endpoints)
 - [Tracker Store — Where conversations are stored](https://rasa.com/docs/pro/build/configuring-assistant/#tracker-store--where-conversations-are-stored)
 - [Event Broker — Where conversation events are sent](https://rasa.com/docs/pro/build/configuring-assistant/#event-broker--where-conversation-events-are-sent)
 - [Action Endpoint — Where custom code runs](https://rasa.com/docs/pro/build/configuring-assistant/#action-endpoint--where-custom-code-runs)
 - [Models — Where trained models live](https://rasa.com/docs/pro/build/configuring-assistant/#models--where-trained-models-live)
 - [Model Groups — LLM and embedding models](https://rasa.com/docs/pro/build/configuring-assistant/#model-groups--llm-and-embedding-models)
 - [Lock Stores — Prevent processing conflicts](https://rasa.com/docs/pro/build/configuring-assistant/#lock-stores--prevent-processing-conflicts)
 - [Vector Stores — Enterprise search and flow retrieval](https://rasa.com/docs/pro/build/configuring-assistant/#vector-stores--enterprise-search-and-flow-retrieval)
 - [NLG Server — External response generator](https://rasa.com/docs/pro/build/configuring-assistant/#nlg-server--external-response-generator)
 - [Contextual Response Rephraser — Rephrase responses with LLMs](https://rasa.com/docs/pro/build/configuring-assistant/#contextual-response-rephraser--rephrase-responses-with-llms)
 - [Silence Handling — Timeout before triggering fallback](https://rasa.com/docs/pro/build/configuring-assistant/#silence-handling--timeout-before-triggering-fallback)
- [Domain](https://rasa.com/docs/pro/build/configuring-assistant/#domain)
 - [What’s in the Domain](https://rasa.com/docs/pro/build/configuring-assistant/#whats-in-the-domain)