# Source: https://rasa.com/docs/reference/config/overview

On this page

You can customise many aspects of how your assistant project works by modifying the following files: `config.yml`, `endpoints.yml`, and `domain.yml`.

A minimal configuration for a [CALM](https://rasa.com/docs/learn/concepts/calm/) assistant looks like this:

config.yml

```
recipe: default.v1language: enassistant_id: 20230405-114328-tranquil-mustardpipeline:  - name: CompactLLMCommandGeneratorpolicies:  - name: rasa.core.policies.flow_policy.FlowPolicy
```

Default Configuration

For backwards compatibility, running `rasa init` will create an NLU-based assistant. To create a CALM assistant with the right `config.yml`, add the additional `--template` argument:

```
rasa init --template calm
```

## Assistant ID[​](https://rasa.com/docs/reference/config/overview/#assistant-id 'Direct link to Assistant ID')

The `assistant_id` key should be a unique value and allows you to distinguish multiple deployed assistants. This id is added to each event's metadata, together with the model id. See [event brokers](https://rasa.com/docs/reference/integrations/event-brokers/) for more information. Note that if the config file does not include this required key or the placeholder default value is not replaced, a random assistant name will be generated and added to the configuration every time you run `rasa train`.

## Recipe[​](https://rasa.com/docs/reference/config/overview/#recipe 'Direct link to Recipe')

The `recipe` key only needs to be modified if you want to use a [custom graph recipe](https://rasa.com/docs/reference/config/components/graph-recipe/). The vast majority of projects should use the default value `"default.v1"`.

## Language[​](https://rasa.com/docs/reference/config/overview/#language 'Direct link to Language')

- The `language` key sets the primary language your assistant supports. Use a two-letter [ISO 639-1 code](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes) (e.g., "en" for English).
- `additional_languages` key lists codes of other languages your assistant supports.

With these settings, your assistant will default to its primary language but can recognize and respond in all configured languages. You can further translate your assistant’s content. For more details, refer to our [Translating Your Assistant](https://rasa.com/docs/pro/build/translating-your-assistant#built-in-language-slot) guide.

Here’s the example for assistant which is using English as default while also supporting Italian, German, and French:

**config.yml**

```
# ...language: "en"# Default language: Englishadditional_languages:  - "it"# Italian  - "de"# German  - "fr"# French# ...
```

You can use any valid language or locale-specific code following the BCP 47 standard:

- Basic language codes: e.g., "en", "de", "it".
- Locale-specific codes: e.g., "en-US", "fr-CA", "de-CH".
- Custom language codes: e.g., "x-en-formal".

Make sure all language codes adhere strictly to this format to avoid unexpected validation errors.

BCP 47 Standard

Rasa adheres to the [BCP 47](https://en.wikipedia.org/wiki/IETF_language_tag) standard for language codes. This ensures compatibility with platforms such as Twilio Voice, Genesys Cloud, and Amazon Connect.

## Pipeline[​](https://rasa.com/docs/reference/config/overview/#pipeline 'Direct link to Pipeline')

The `pipeline` key lists the components which will be used to process and understand the messages that end users send to your assistant. In a CALM assistant, the output of your components pipeline is a list of [commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference).

The main component in your pipeline is the `LLMCommandGenerator`. Here is what an example configuration looks like:

config.yml

```
  pipeline:    - name: CompactLLMCommandGenerator      llm:        model_group: openai_llm      flow_retrieval:        embeddings:          model_group: openai_embeddings      user_input:        max_characters: 420
```

endpoints.yml

```
   model_groups:     - id: openai_direct       models:         - model: "gpt-5.1-2025-11-13"           provider: "openai"           timeout: 7           temperature: 1.0     - id: openai_embeddings       models:         - model: "text-embedding-3-large"           provider: "openai"
```

The full set of configurable parameters is listed [here](https://rasa.com/docs/reference/config/components/llm-command-generators/).

All components which make use of LLMs have common configuration parameters which are listed [here](https://rasa.com/docs/reference/config/components/llm-configuration/)

## Policies[​](https://rasa.com/docs/reference/config/overview/#policies 'Direct link to Policies')

The `policies` key lists the [dialogue policies](https://rasa.com/docs/reference/config/policies/overview/) your assistant will use to progress the conversation.

config.yml

```
policies:  - name: rasa.core.policies.flow_policy.FlowPolicy
```

The [FlowPolicy](https://rasa.com/docs/reference/config/policies/flow-policy/) currently doesn't have an additional configuration parameters.

## Silence Timeout Handling[​](https://rasa.com/docs/reference/config/overview/#silence-timeout-handling 'Direct link to Silence Timeout Handling')

Silence timeouts help your assistant handle situations where the user doesn’t respond. For now, this setting only works with voice-stream channels, such as:

- Twilio Media Streams
- Browser Audio
- Genesys
- Jambonz Stream
- Audiocodes Stream

There are two types of timeouts you can configure.

### Global Silence Timeout[​](https://rasa.com/docs/reference/config/overview/#global-silence-timeout 'Direct link to Global Silence Timeout')

- Rasa Pro 3.13>
- Rasa Pro >= 3.14

You can set a default silence timeout across your assistant by adding this to your `endpoints.yml`:

endpoints.yml

```
interaction_handling:  global_silence_timeout: 7
```

This means the assistant will wait 7 seconds (or your configured value) for a user reply before treating it as **silence** and triggering [fallback logic](https://rasa.com/docs/reference/config/overview/#customizing-the-assistants-response-to-silence).

By default the global silence timeout is set to 7 seconds.

You can set the default silence timeout across your assistant on a per-channel basis by adding `silence_timeout` to your channel configuration in `credentials.yml`:

credentials.yml

```
twilio_media_streams:    silence_timeout: 10
```

This means that on twilio\_media\_streams channel the assistant will wait 10 seconds (or your configured value) for a user reply before treating it as **silence** and triggering [fallback logic](https://rasa.com/docs/reference/config/overview/#customizing-the-assistants-response-to-silence).

### Local (Per-Step) Silence Timeout[​](https://rasa.com/docs/reference/config/overview/#local-per-step-silence-timeout 'Direct link to Local (Per-Step) Silence Timeout')

You can override the global value for specific **Collect** steps:

```
steps:  - collect:      name: ask_email      silence_timeout: 10
```

or for a specific channel in that step:

```
steps:  - collect:      name: ask_email      silence_timeout:        twilio_media_streams: 10
```

For channels not listed, the timeout set in credentials or global silence timeout of 7 seconds (if not set for a channel in `credentials.yml`) will be used.

This allows you to fine-tune the timing for specific questions. For example, you may want to:

- Wait **longer** on more complex or sensitive questions (e.g., "Can you describe your issue?")
- Use **shorter** timeouts for quick prompts (e.g., yes/no questions)

Tailoring silence handling this way improves the conversational experience.

### Enabling/Disabling Silence Timeouts[​](https://rasa.com/docs/reference/config/overview/#enablingdisabling-silence-timeouts 'Direct link to Enabling/Disabling Silence Timeouts')

If you want to disable silence detection so it never triggers during a conversation, you can set the timeout to a very high value. For example, to disable it globally:

- Rasa Pro 3.13>
- Rasa Pro >= 3.14

endpoints.yml

```
interaction_handling:  global_silence_timeout: 70000
```

credentials.yml

```
twilio_media_streams:  silence_timeout: 70000
```

Use this approach if you want to avoid fallback interruptions but still need a valid numeric value for configuration or platform compatibility.

Disabling Silence Timeouts at step level

If silence timeout is set at the step level, that value has precedence over the global or channel-specific setting. In order to disable silence timeout for a specific step, set it to a very high value (e.g., 70000 seconds) in that step's configuration.

### Customizing the Assistant's Response to Silence[​](https://rasa.com/docs/reference/config/overview/#customizing-the-assistants-response-to-silence 'Direct link to Customizing the Assistant's Response to Silence')

When the silence timeout is reached, the assistant triggers the `pattern_user_silence`. You can customize how your assistant responds to silence by modifying this pattern.

👉 [Learn more about patterns configuration](https://rasa.com/docs/reference/primitives/patterns/)

## Minimum pacing between bot messages (voice streams)[​](https://rasa.com/docs/reference/config/overview/#minimum-pacing-between-bot-messages-voice-streams 'Direct link to Minimum pacing between bot messages (voice streams)')

On **voice stream** channels, Rasa can enforce a minimum gap between consecutive bot audio segments so back-to-back utterances do not sound rushed. When the next bot message is ready, the channel compares the time since the previous message finished playing against a configured minimum. If not enough time has passed, it **streams silence** for the remaining duration on the same audio path as speech (it does not block the server with a sleep), then plays the next message.

This applies to the same class of streaming voice connectors as [silence timeout handling](https://rasa.com/docs/reference/config/overview/#silence-timeout-handling).

### Regular utterances vs filler[​](https://rasa.com/docs/reference/config/overview/#regular-utterances-vs-filler 'Direct link to Regular utterances vs filler')

Two types of minimum gaps are used:

- **Between ordinary consecutive bot messages:** a shorter default gap (**1 second**).
- **After a filler message:** a longer default gap (**2 seconds**). Filler messages are short, streamed assistant audio played while longer operations are in progress, for example, brief assistant text sent alongside tool calls from ReAct agents when enabled. A longer pause follows these messages to provide clearer separation before the next substantive response.

If there is no prior bot audio in the turn (for example, the first reply after the user speaks), pacing does not insert a delay ahead of that reply.

note

These delays are fixed defaults in the voice streaming stack.

- [Assistant ID](https://rasa.com/docs/reference/config/overview/#assistant-id)
- [Recipe](https://rasa.com/docs/reference/config/overview/#recipe)
- [Language](https://rasa.com/docs/reference/config/overview/#language)
- [Pipeline](https://rasa.com/docs/reference/config/overview/#pipeline)
- [Policies](https://rasa.com/docs/reference/config/overview/#policies)
- [Silence Timeout Handling](https://rasa.com/docs/reference/config/overview/#silence-timeout-handling)
 - [Global Silence Timeout](https://rasa.com/docs/reference/config/overview/#global-silence-timeout)
 - [Local (Per-Step) Silence Timeout](https://rasa.com/docs/reference/config/overview/#local-per-step-silence-timeout)
 - [Enabling/Disabling Silence Timeouts](https://rasa.com/docs/reference/config/overview/#enablingdisabling-silence-timeouts)
 - [Customizing the Assistant's Response to Silence](https://rasa.com/docs/reference/config/overview/#customizing-the-assistants-response-to-silence)
- [Minimum pacing between bot messages (voice streams)](https://rasa.com/docs/reference/config/overview/#minimum-pacing-between-bot-messages-voice-streams)
 - [Regular utterances vs filler](https://rasa.com/docs/reference/config/overview/#regular-utterances-vs-filler)