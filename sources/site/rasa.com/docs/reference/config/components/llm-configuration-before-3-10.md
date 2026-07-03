# Source: https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10

On this page

LLM Configuration for Rasa Pro 3.11 and above

For Rasa Pro versions `3.11` and above, refer to the [LLM Configuration for `>=3.11`](https://rasa.com/docs/reference/config/components/llm-configuration/) page.

## Overview[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#overview 'Direct link to Overview')

This page applies to the following components which use LLMs:

- [SingleStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#singlestepllmcommandgenerator)
- [MultiStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#multistepllmcommandgenerator)
- [EnterpriseSearchPolicy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/)
- [IntentlessPolicy](https://rasa.com/docs/reference/config/policies/intentless-policy/)
- [ContextualResponseRephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/)
- [LLMBasedRouter](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter)

All the above components can be configured to change:

- the LLM provider
- the model to be used

Starting with version Rasa Pro `3.10`, CALM uses [LiteLLM](https://litellm.vercel.app/) under the hood to integrate with different LLM providers. Hence, all [LiteLLM's integrated providers](https://litellm.vercel.app/docs/providers) are supported with CALM as well. We explicitly mention the settings required for the most frequently used ones in the sections below.

warning

If you want to try a provider other than OpenAI / Azure OpenAI, it is recommended to install Rasa Pro versions `>= 3.10`.

## Recommended Models[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#recommended-models 'Direct link to Recommended Models')

The table below documents the versions of each model we recommend for use with various Rasa components. As new models are published, Rasa will test these and where appropriate add them as a recommended model.

| Component | Providing platform | Recommended models |
| --- | --- | --- |
| `SingleStepLLMCommandGenerator`, `EnterpriseSearchPolicy`, `IntentlessPolicy` | OpenAI, Azure | `gpt-4-0613` |
| `ContextualResponseRephraser` | OpenAI, Azure | `gpt-4-0613`, `gpt-3.5-turbo-0125` |
| `MultiStepLLMCommandGenerator` | OpenAI, Azure | `gpt-4-turbo-2024-04-09`, `gpt-3.5-turbo-0125`, `gpt-3.5-turbo-1106`, `gpt-4o-2024-08-06` |

## Chat completion models[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#chat-completion-models 'Direct link to Chat completion models')

Default Provider

CALM is LLM agnostic and can be configured with different LLMs, but OpenAI is the default model provider. Majority of our experiments have been with models available on OpenAI or OpenAI Azure service. The performance of your assistant may vary when using other LLMs, but improvements can be made by tuning flow and collect step descriptions.

To configure components that use a chat completion model as the LLM, declare the configuration under the `llm` key of that component's configuration. For example:

config.yml

```
   recipe: default.v1   language: en   pipeline:   - name: SingleStepLLMCommandGenerator     llm:        ...
```

#### Required Parameters[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#required-parameters 'Direct link to Required Parameters')

There are certain required parameters under the `llm` key:

1. `model` - Specifies the name of the model identifier available from the LLM provider's documentation, for e.g. `gpt-4-0613`
2. `provider` - Unique identifier of the provider to be used for invoking the specified model.

config.yaml

```
   recipe: default.v1   language: en   pipeline:   - name: SingleStepLLMCommandGenerator     llm:        model: gpt-4-0613        provider: openai
```

#### Optional Parameters[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#optional-parameters 'Direct link to Optional Parameters')

The `llm` key also accepts inference time parameters like `temperature`, etc which are optional but can be useful in extracting the best performance out of the model being used. Please refer to the [official LiteLLM documentation](https://litellm.vercel.app/docs/completion/input) for a list of such parameters supported.

When configuring a particular provider, there are a few provider specific settings which are explained under each provider's individual sub-section below.

important

If you switch to a different LLM provider, all default parameters for the old provider will be overriden with the default parameters of the new provider.

E.g. If a provider sets `temperature=0.7` as the default value and you switch to a different LLM provider, this default will be ignored and it is up to you to set the temperature for the new provider.

### OpenAI[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#openai 'Direct link to OpenAI')

#### API Token[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#api-token 'Direct link to API Token')

The API token authenticates your requests to the OpenAI API.

To configure the API token, follow these steps:

1. If you haven't already, sign up for an account on the OpenAI platform.
 
2. Navigate to the [OpenAI Key Management page](https://platform.openai.com/account/api-keys), and click on the "Create New Secret Key" button to initiate the process of obtaining `<your-api-key>`.
 
3. To set the API key as an environment variable, you can use the following command in a terminal or command prompt:
 

- Linux/MacOS
- Windows

```
export OPENAI_API_KEY=<your-api-key>
```

```
setx OPENAI_API_KEY <your-api-key>
```

This will apply to future cmd prompt window, so you will need to open a new one to use that variable.

Replace `<your-api-key>` with the actual API key you obtained from the OpenAI platform.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration 'Direct link to Configuration')

There are no additional OpenAI specific parameters to be configured. However, there could be model specific parameters like `temperature` that you might want to modify. Names for such parameters can found in [OpenAI's API documentation](https://platform.openai.com/docs/api-reference/chat) and defined under `llm` key of the component's configuration. Please refer to [LiteLLM's documentation](https://litellm.vercel.app/docs/providers/openai#openai-chat-completion-models) to know the list of models supported from the OpenAI platform.

#### Model deprecations[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#model-deprecations 'Direct link to Model deprecations')

OpenAI regularly publishes a deprecation schedule for its models. This schedule can be accessed in the [documentation published by OpenAI](https://platform.openai.com/docs/deprecations).

### Azure OpenAI Service[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure-openai-service 'Direct link to Azure OpenAI Service')

#### API Token[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#api-token-1 'Direct link to API Token')

The API token authenticates your requests to the Azure OpenAI Service.

Set the API token as an environment variable. You can use the following command in a terminal or command prompt:

- Linux/MacOS
- Windows

```
export AZURE_API_KEY=<your-api-key>
```

```
setx AZURE_API_KEY <your-api-key>
```

This will apply to future cmd prompt window, so you will need to open a new one to use that variable.

Replace `<your-api-key>` with the actual API key you obtained from the Azure OpenAI Service platform.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration-1 'Direct link to Configuration')

To access models provided by [Azure OpenAI Service](https://azure.microsoft.com/en-in/products/ai-services/openai-service), there are a few additional parameters that need to be configured:

- `provider` - Set to `azure`.
- `api_type` - The type of API to use. This should be set to "azure" to indicate the use of Azure OpenAI Service.
- `api_base` - The URL for your Azure OpenAI instance. An example might look like this: `https://my-azure.openai.azure.com/`.
- `api_version` - The API version to use for this operation. This follows the YYYY-MM-DD format and the value should be enclosed in single or double quotes.
- `engine`/`deployment_name` - Alias for `deployment` parameter. Name of the deployment on Azure.

Model specific parameters like `temperature` can be defined as well. Refer to [OpenAI Azure service's API documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#request-body) for information on available parameter names.

A complete example configuration of the `SingleStepLLMCommandGenerator` using Azure OpenAI Service would look like this:

- Rasa Pro <=3.7.x
- 3.8.x<=Rasa Pro<=3.9.x
- Rasa Pro >=3.10.x

config.yml

```
    - name: LLMCommandGenerator      llm:        deployment: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7
```

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7
```

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        provider: azure        deployment: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7
```

A more comprehensive example using the Azure OpenAI service in more CALM components is available [here](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure).

#### Model deprecations[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#model-deprecations-1 'Direct link to Model deprecations')

Azure regularly publishes a deprecation schedule for its models that come under the OpenAI Azure Service. This schedule can be accessed in the [documentation published by Azure](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/model-retirements).

#### Debugging[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#debugging 'Direct link to Debugging')

If you encounter timeout errors, configure `request_timeout` parameter to a larger value. The exact value depends on how your azure instance is configured.

### Amazon Bedrock[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#amazon-bedrock 'Direct link to Amazon Bedrock')

#### Requirements:[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#requirements 'Direct link to Requirements:')

1. Make sure you have `rasa-pro>=3.10.x` installed.
2. Install `boto3>=1.28.57`.
3. Set the following environment variables - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION_NAME`.
4. (Optional) Might have to set `AWS_SESSION_TOKEN` if your organisation mandates the usage of [temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) for security.

Once the above steps are complete, edit `config.yaml` to use an appropriate model and set `provider` to `bedrock`:

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        provider: bedrock        model: anthropic.claude-instant-v1
```

Model specific parameters like `temperature` can be defined as well. Refer to LiteLLM's documentation for information on [available parameter names](https://litellm.vercel.app/docs/completion/input#translated-openai-params) and [supported models](https://litellm.vercel.app/docs/providers/bedrock#supported-aws-bedrock-models).

### Gemini - Google AI Studio[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#gemini---google-ai-studio 'Direct link to Gemini - Google AI Studio')

#### Requirements:[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#requirements-1 'Direct link to Requirements:')

1. Make sure you have `rasa-pro>=3.10.x` installed.
2. Install python package `google-generativeai`.
3. Get API Key at [https://aistudio.google.com/](https://aistudio.google.com/) .
4. Set the API key to an environment variable `GEMINI_API_KEY`.

Once the above steps are complete, edit `config.yaml` to use an appropriate model and set `provider` to `gemini`:

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        provider: gemini        model: gemini-pro
```

Refer to LiteLLM's documentation to know which [additional parameters](https://litellm.vercel.app/docs/providers/gemini#supported-openai-params) and [models](https://litellm.vercel.app/docs/providers/gemini#chat-models) are supported.

### HuggingFace Inference Endpoints[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#huggingface-inference-endpoints 'Direct link to HuggingFace Inference Endpoints')

#### Requirements:[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#requirements-2 'Direct link to Requirements:')

1. Make sure you have `rasa-pro>=3.10.x` installed.
2. Set an API Key to the environment variable `HUGGINGFACE_API_KEY`.
3. Edit `config.yaml` to use an appropriate model, set `provider` to `huggingface` and `api_base` to the base URL of the deployed endpoint:

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        provider: huggingface        model: meta-llama/CodeLlama-7b-Instruct-hf        api_base: "https://my-endpoint.huggingface.cloud"
```

### Self Hosted Model Server[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#self-hosted-model-server 'Direct link to Self Hosted Model Server')

CALM's components can also be configured to work with an open source LLM that is hosted on an open source model server like [vLLM](https://github.com/vllm-project/vllm)(recommended), [Ollama](https://ollama.com/) or [Llama.cpp web server](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#web-server). The only requirement is that the model server should adhere to the [OpenAI API format](https://platform.openai.com/docs/api-reference/introduction).

Once you have your model server running, configure the CALM assistant's config.yaml:

**vLLM**

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        provider: self-hosted        model: meta-llama/CodeLlama-7b-Instruct-hf        api_base: "https://my-endpoint/v1"
```

Important to note:

1. Recommended version of `vllm` to use is `0.6.0`.
 
2. CALM exclusively utilizes the chat completions endpoint of the model server, so it's essential that the model's tokenizer includes a chat template. Models lacking a chat template will not be compatible with CALM.
 
3. `model` should contain the name of the model supplied to the vllm startup command, for example if your model server is started with:
 
 ```
    vllm serve meta-llama/CodeLlama-7b-Instruct-hf
    ```
 
 `model` should be set to `meta-llama/CodeLlama-7b-Instruct-hf`.
 
4. `api_base` should contain the full exposed URL of the model server with `v1` attached as suffix to the URL.
 

**Ollama**

Once the ollama model server is running, edit the `config.yaml` file:

config.yml

```
    - name: SingleStepLLMCommandGenerator      llm:        provider: ollama        model: llama3.1        api_base: "https://my-endpoint"
```

### Other Providers[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#other-providers 'Direct link to Other Providers')

info

If you want to try one of these providers, it is recommended to install Rasa Pro versions `>= 3.10`.

Other than the above mentioned providers, we have also tested support for the following providers:

| Platform | `provider` | API-KEY variable |
| --- | --- | --- |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` |
| Cohere | `cohere` | `COHERE_API_KEY` |
| Mistral | `mistral` | `MISTRAL_API_KEY` |
| Together AI | `together_ai` | `TOGETHERAI_API_KEY` |
| Groq | `groq` | `GROQ_API_KEY` |

For each of the above ones, ensure you have set an environment variable named by the value in `API-KEY variable` column to the API key of that platform and set the `provider` parameter under `llm` key of the component's config to the value in `provider` column.

## Embedding models[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#embedding-models 'Direct link to Embedding models')

To configure components that use an embedding model, declare the configuration under the `embeddings` key of that component's configuration. For example:

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      model: gpt-4-0613    flow_retrieval:      embeddings:        ...
```

The `embeddings` property needs two mandatory parameters:

1. `model` - Specifies the name of the model identifier available from the LLM provider's documentation, for e.g. `text-embedding-3-large`.
 
2. `provider` - Unique identifier of the provider to be used for invoking the specified model, for e.g. `openai`
 

- Rasa Pro <=3.9.x
- Rasa Pro >=3.10.x

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      model: gpt-4-0613    flow_retrieval:      embeddings:        type: openai        model: text-embedding-3-large
```

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      provider: openai      model: gpt-4-0613    flow_retrieval:      embeddings:        provider: openai        model: text-embedding-3-large
```

When configuring a particular provider, there are a few provider specific settings which are explained under each provider's individual sub-section below.

### OpenAI[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#openai-1 'Direct link to OpenAI')

OpenAI is used as the default embedding model provider. To start using, ensure you have configured an API token as you would do for a [chat completion model from OpenAI platform](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#api-token)

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration-2 'Direct link to Configuration')

- Rasa Pro <=3.9.x
- Rasa Pro >=3.10.x

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      model: gpt-4-0613    flow_retrieval:      embeddings:        type: openai        model: text-embedding-3-large
```

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      provider: openai      model: gpt-4-0613    flow_retrieval:      embeddings:        provider: openai        model: text-embedding-3-large
```

### Azure OpenAI Service[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure-openai-service-1 'Direct link to Azure OpenAI Service')

Ensure you have configured an API token as you would do for a [chat completion model for Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#api-token-1)

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration-3 'Direct link to Configuration')

Configuring an embedding model from Azure OpenAI Service needs values for the same set of parameters that are required for configuring a [chat completion model from Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration-1)

- Rasa Pro <=3.9.x
- Rasa Pro >=3.10.x

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      model: gpt-4-0613    flow_retrieval:      embeddings:        type: azure        engine: engine-embed        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7
```

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      model: gpt-4-0613    flow_retrieval:      embeddings:        provider: azure        deployment: engine-embed        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7
```

### Amazon Bedrock[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#amazon-bedrock-1 'Direct link to Amazon Bedrock')

Configuring an embedding model from amazon bedrock needs the same pre-requisites as a [chat completion model](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#requirements). Please ensure you have addressed these before proceeding further.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration-4 'Direct link to Configuration')

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      provider: openai      model: gpt-4-0613    flow_retrieval:      embeddings:        provider: bedrock        model: amazon.titan-embed-text-v1
```

Please refer to LiteLLM's documentation on [list of supported embedding models](https://litellm.vercel.app/docs/providers/bedrock#supported-aws-bedrock-embedding-models) from Amazon Bedrock

### In-Memory[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#in-memory 'Direct link to In-Memory')

CALM also provides an option to load lightweight embedding models in-memory without needing them to be exposed over an API. It uses the sentence transformers library under the hood to load and run inference on them.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuration-5 'Direct link to Configuration')

- Rasa Pro <=3.9.x
- Rasa Pro >=3.10.x

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      model: gpt-4-0613    flow_retrieval:      embeddings:        type: "huggingface"        model: "BAAI/bge-small-en-v1.5"        model_kwargs: # used during instantiation          device: 'cpu'        encode_kwargs: # used during inference          normalize_embeddings: True
```

config.yml

```
pipeline:  - name: "SingleStepLLMCommandGenerator"    llm:      provider: "openai"      model: "gpt-4-0613"    flow_retrieval:      embeddings:        provider: "huggingface_local"        model: "BAAI/bge-small-en-v1.5"        model_kwargs: # used during instantiation          device: "cpu"        encode_kwargs: # used during inference          normalize_embeddings: true
```

- `model` parameter can take as value either any embedding model repository available on the HuggingFace hub or a path to a local model.
- `model_kwargs` parameter is used to provide [load time arguments](https://sbert.net/docs/package_reference/sentence_transformer/SentenceTransformer.html#id1) to the sentence transformer library.
- `encode_kwargs` parameter is used to provide [inference time arguments](https://sbert.net/docs/package_reference/sentence_transformer/SentenceTransformer.html#sentence_transformers.SentenceTransformer.encode) to the sentence transformer library.

### Other Providers[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#other-providers-1 'Direct link to Other Providers')

Other than the above mentioned providers, we have also tested support for the following providers -

| Platform | `provider` | API-KEY variable |
| --- | --- | --- |
| Cohere | `cohere` | `COHERE_API_KEY` |
| Mistral | `mistral` | `MISTRAL_API_KEY` |
| Voyage AI | `voyage` | `VOYAGE_API_KEY` |

For each of the above ones, ensure you have set an environment variable named by the value in `API-KEY variable` column to the API key of that platform and set the `provider` parameter under `llm` key of the component's config to the value in `provider` column.

## Configuring self-signed SSL certificates[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuring-self-signed-ssl-certificates 'Direct link to Configuring self-signed SSL certificates')

In environments where a proxy performs TLS interception, Rasa may need to be configured to trust the certificates used by your proxy. By default, certificates are loaded from the OS certificate store. However, if your setup involves custom self-signed certificates, you can specify these by setting the `RASA_CA_BUNDLE` environment variable.

This variable points to the path of the certificate file that Rasa should use to validate SSL connections:

```
export RASA_CA_BUNDLE="path/to/your/certificate.pem"
```

info

The `REQUESTS_CA_BUNDLE` environment variable is deprecated and will no longer be supported in future versions. Please use RASA\_CA\_BUNDLE instead to ensure compatibility.

## Configuring Proxy URLs[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuring-proxy-urls 'Direct link to Configuring Proxy URLs')

In environments where LLM requests need to be routed through a proxy, Rasa relies on LiteLLM to handle proxy configurations. LiteLLM supports configuring proxy URLs through the `HTTP_PROXY` and `HTTPS_PROXY` environment variables.

To ensure that all LLM requests are routed through the proxy, you can set the environment variables as follows:

```
export HTTP_PROXY="http://your-proxy-url:port"export HTTPS_PROXY="https://your-proxy-url:port"
```

## FAQ[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#faq 'Direct link to FAQ')

### Does OpenAI use my data to train their models?[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#does-openai-use-my-data-to-train-their-models 'Direct link to Does OpenAI use my data to train their models?')

No. OpenAI does not use your data to train their models. From their [website](https://openai.com/enterprise-privacy/):

> We do not train our models on your business data by default.

## Example Configurations[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#example-configurations 'Direct link to Example Configurations')

### Azure[​](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure 'Direct link to Azure')

A comprehensive example which includes:

- `llm` and `embeddings` configuration for components in `config.yml`:
 - `IntentlessPolicy`
 - `EnterpriseSearchPolicy`
 - `SingleStepLLMCommandGenerator`
 - `flow_retrieval` in 3.8.x
- `llm` configuration for rephrase in `endpoints.yml` (`ContextualResponseRephraser`)

- Rasa Pro <=3.7.x
- 3.8.x <= Rasa Pro <= 3.9.x
- Rasa Pro >=3.10.x

endpoints.yml

```
    nlg:      type: rasa_plus.ml.ContextualResponseRephraser      llm:        engine: rasa-gpt-4        api_type: azure        api_version: "2024-02-15-preview"        api_base: https://my-azure.openai.azure.com        request_timeout: 7
```

config.yml

```
    recipe: default.v1    language: en    pipeline:    - name: LLMCommandGenerator      llm:        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7    policies:    - name: FlowPolicy    - name: rasa_plus.ml.IntentlessPolicy      llm:        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7      embeddings:        model: text-embedding-3-small        engine: rasa-embedding-small        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7    - name: rasa_plus.ml.EnterpriseSearchPolicy      vector_store:        type: "faiss"        threshold: 0.0      llm:        model: gpt-4-0613        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7      embeddings:        model: text-embedding-3-small        engine: rasa-embedding-small        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7
```

endpoints.yml

```
nlg:  type: rephrase  llm:    engine: rasa-gpt-4    api_type: azure    api_version: "2024-02-15-preview"    api_base: https://my-azure.openai.azure.com    request_timeout: 7
```

config.yml

```
    recipe: default.v1    language: en    pipeline:    - name: SingleStepLLMCommandGenerator      llm:        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7      flow_retrieval:        embeddings:          model: text-embedding-3-small          engine: rasa-embedding-small          api_type: azure          api_base: https://my-azure.openai.azure.com/          api_version: "2024-02-15-preview"          request_timeout: 7    policies:    - name: FlowPolicy    - name: IntentlessPolicy      llm:        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7      embeddings:        model: text-embedding-3-small        engine: rasa-embedding-small        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7    - name: EnterpriseSearchPolicy      vector_store:        type: "faiss"        threshold: 0.0      llm:        engine: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7      embeddings:        model: text-embedding-3-small        engine: rasa-embedding-small        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        request_timeout: 7
```

endpoints.yml

```
nlg:  type: rephrase  llm:    provider: azure    deployment: rasa-gpt-4    api_type: azure    api_version: "2024-02-15-preview"    api_base: https://my-azure.openai.azure.com    timeout: 7
```

config.yml

```
    recipe: default.v1    language: en    pipeline:    - name: SingleStepLLMCommandGenerator      llm:        provider: azure        deployment: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7      flow_retrieval:        embeddings:          provider: openai          deployment: rasa-embedding-small          api_type: azure          api_base: https://my-azure.openai.azure.com/          api_version: "2024-02-15-preview"          timeout: 7    policies:    - name: FlowPolicy    - name: IntentlessPolicy      llm:        provider: azure        deployment: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7      embeddings:        provider: azure        deployment: rasa-embedding-small        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7    - name: EnterpriseSearchPolicy      vector_store:        type: "faiss"        threshold: 0.0      llm:        provider: azure        deployment: rasa-gpt-4        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7      embeddings:        provider: azure        deployment: rasa-embedding-small        api_type: azure        api_base: https://my-azure.openai.azure.com/        api_version: "2024-02-15-preview"        timeout: 7
```

- [Overview](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#overview)
- [Recommended Models](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#recommended-models)
- [Chat completion models](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#chat-completion-models)
 - [OpenAI](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#openai)
 - [Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure-openai-service)
 - [Amazon Bedrock](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#amazon-bedrock)
 - [Gemini - Google AI Studio](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#gemini---google-ai-studio)
 - [HuggingFace Inference Endpoints](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#huggingface-inference-endpoints)
 - [Self Hosted Model Server](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#self-hosted-model-server)
 - [Other Providers](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#other-providers)
- [Embedding models](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#embedding-models)
 - [OpenAI](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#openai-1)
 - [Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure-openai-service-1)
 - [Amazon Bedrock](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#amazon-bedrock-1)
 - [In-Memory](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#in-memory)
 - [Other Providers](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#other-providers-1)
- [Configuring self-signed SSL certificates](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuring-self-signed-ssl-certificates)
- [Configuring Proxy URLs](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#configuring-proxy-urls)
- [FAQ](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#faq)
 - [Does OpenAI use my data to train their models?](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#does-openai-use-my-data-to-train-their-models)
- [Example Configurations](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#example-configurations)
 - [Azure](https://rasa.com/docs/reference/config/components/llm-configuration-before-3-10/#azure)