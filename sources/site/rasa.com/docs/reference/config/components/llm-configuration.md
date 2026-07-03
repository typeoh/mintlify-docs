# Source: https://rasa.com/docs/reference/config/components/llm-configuration

On this page

LLM Configuration for Rasa Pro 3.10 and below

For Rasa Pro versions `3.10` and below, refer to the [LLM Configuration for `<=3.10`](https://rasa.com/docs/reference/config/components/llm-configuration/) page.

## Overview[​](https://rasa.com/docs/reference/config/components/llm-configuration/#overview 'Direct link to Overview')

This page applies to the following components which use LLMs:

- [SearchReadyLLMCommandGenerator](https://rasa.com/docs/reference/config/components/llm-command-generators/#searchreadyllmcommandgenerator)
- [CompactLLMCommandGenerator](https://rasa.com/docs/reference/config/components/llm-command-generators/#compactllmcommandgenerator)
- [EnterpriseSearchPolicy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/)
- [ContextualResponseRephraser](https://rasa.com/docs/reference/primitives/contextual-response-rephraser/)
- [LLMBasedRouter](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter)
- [SingleStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#singlestepllmcommandgenerator) (deprecated)
- [MultiStepLLMCommandGenerator](https://rasa.com/docs/reference/config/components/deprecated-components/#multistepllmcommandgenerator) (deprecated)
- [IntentlessPolicy](https://rasa.com/docs/reference/config/policies/intentless-policy/) (deprecated)

All the above components can be configured to change:

- the LLM provider
- the model(s) to be used

Starting with version Rasa Pro `3.10`, CALM uses [LiteLLM](https://litellm.vercel.app/) under the hood to integrate with different LLM providers. Hence, all [LiteLLM's integrated providers](https://litellm.vercel.app/docs/providers) are supported with CALM as well. We explicitly mention the settings required for the most frequently used ones in the sections below.

## Declaring LLM deployments[​](https://rasa.com/docs/reference/config/components/llm-configuration/#declaring-llm-deployments 'Direct link to Declaring LLM deployments')

LLM deployments are always declared in groups comprising of 1 or more deployments. The below sections explain how to declare these groups.

### Model Groups[​](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups 'Direct link to Model Groups')

Model groups allow you to define multiple models under a single ID which can be accessed by any component. Model groups are defined in the `endpoints.yml` file under the `model_groups` key, separating model definitions from individual component configurations. For example:

endpoints.yml

```
   model_groups:     - id: openai-direct # Unique identifier for the model group       models:         ...
```

- The `id` key uniquely identifies the model group.
- The `models` key lists all model deployments in that group.
- Each model in the list includes a configuration, explained in the following sections.

### Defining a single model group[​](https://rasa.com/docs/reference/config/components/llm-configuration/#defining-a-single-model-group 'Direct link to Defining a single model group')

endpoints.yml

```
   model_groups:     - id: openai-direct # Unique identifier for the model group       models:         - provider: openai           model: gpt-5.1-2025-11-13
```

#### Required Parameters[​](https://rasa.com/docs/reference/config/components/llm-configuration/#required-parameters 'Direct link to Required Parameters')

There are certain required parameters for each model group:

1. `provider` - Unique identifier of the LLM provider to be used.
2. `model` - Specifies the name of the model identifier available from the LLM provider's documentation, for e.g. `gpt-5.1-2025-11-13`.

#### Optional Parameters[​](https://rasa.com/docs/reference/config/components/llm-configuration/#optional-parameters 'Direct link to Optional Parameters')

Each model group also accepts inference time parameters like `temperature`, etc which are optional but can be useful in extracting the best performance out of the model being used. Please refer to the [official LiteLLM documentation](https://litellm.vercel.app/docs/completion/input) for a list of such parameters supported.

When configuring a particular provider, there are a few provider specific settings which are explained under each [provider's individual sub-section below](https://rasa.com/docs/reference/config/components/llm-configuration/).

important

If you switch to a different LLM provider, all default parameters for the old provider will be overriden with the default parameters of the new provider.

E.g. If a provider sets `temperature=0.7` as the default value and you switch to a different LLM provider, this default will be ignored and it is up to you to set the temperature for the new provider.

#### Reasoning models and `reasoning_effort`[​](https://rasa.com/docs/reference/config/components/llm-configuration/#reasoning-models-and-reasoning_effort 'Direct link to reasoning-models-and-reasoning_effort')

For models that support it, `reasoning_effort` is an optional inference-time parameter that controls how much internal reasoning the model applies before returning an answer. Higher values typically increase latency and cost; lower values favor faster responses. Allowed values depend on the provider and model (for example `none`, `minimal`, `low`, and `high` for many OpenAI reasoning-capable models). See provider documentation for more details, e.g. [OpenAI Developers: Reasoning models](https://developers.openai.com/api/docs/guides/reasoning) or [Claude API Docs: Effort](https://platform.claude.com/docs/en/build-with-claude/effort).

You can set `reasoning_effort` alongside other model settings in each entry under `models` in `endpoints.yml`, for example:

endpoints.yml

```
   model_groups:     - id: openai-reasoning       models:         - provider: openai           model: gpt-5-mini-2025-08-07           reasoning_effort: "minimal"
```

By default, Rasa tries to apply the lowest appropriate `reasoning_effort` for the resolved model name (using provider metadata where available). To override this behavior, set `reasoning_effort` explicitly in your model configuration. Models that do not support this parameter do not receive a default.

#### Referencing environment variables in the model configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#referencing-environment-variables-in-the-model-configuration 'Direct link to Referencing environment variables in the model configuration')

To reference environment variables in the model configuration, you can use the `${}` syntax. For example:

endpoints.yml

```
   model_groups:     - id: openai-direct       models:         - provider: openai           model: gpt-5.1-2025-11-13           api_key: ${MY_OPENAI_API_KEY}
```

In the above example, the `api_key` parameter references the environment variable `MY_OPENAI_API_KEY`.

endpoints.yml

```
   model_groups:     - id: my_azure_deployment       models:         - provider: azure           deployment: ${AZURE_DEPLOYMENT_NAME}           api_base: ${AZURE_API_BASE}           api_version: ${AZURE_API_VERSION}           api_key: ${MY_AZURE_API_KEY}           timeout: 7
```

In the above example, the `deployment`, `api_base`, `api_version`, and `api_key` parameters reference the environment variables `AZURE_DEPLOYMENT_NAME`, `AZURE_API_BASE`, `AZURE_API_VERSION`, and `MY_AZURE_API_KEY` respectively. The variables are set in the environment using the `export` command in Unix-based systems and the `setx` command in Windows systems. Not setting these variables will result in an error when the assistant is started.

LLM API health check

The model config and the connection to the LLM provider can be validated by setting the `LLM_API_HEALTH_CHECK` environment variable to `true`.

```
export LLM_API_HEALTH_CHECK=true
```

By default, the variable is set to `False`. When set to `True`, all LLM deployments defined will be checked for availability by making a test API request.

For **Azure OpenAI** configurations that specify a `deployment` but no `model`, a probe request is made during inference-time initialization to resolve the underlying model name so defaults such as `reasoning_effort` can be applied correctly. This probe is made even when `LLM_API_HEALTH_CHECK` is set to `False`.

### Using a model group in a component[​](https://rasa.com/docs/reference/config/components/llm-configuration/#using-a-model-group-in-a-component 'Direct link to Using a model group in a component')

Components using an LLM can be configured to use any of the declared model groups in the component's configuration. To use a model group, you can specify the `model_group` key under the `llm` key. For example:

config.yml

```
   recipe: default.v1   language: en   pipeline:   - name: CompactLLMCommandGenerator     llm:        model_group: openai-direct
```

### Defining multiple model groups[​](https://rasa.com/docs/reference/config/components/llm-configuration/#defining-multiple-model-groups 'Direct link to Defining multiple model groups')

endpoints.yml

```
   model_groups:     - id: openai-gpt-5-1       models:         - provider: openai           model: gpt-5.1-2025-11-13     - id: openai-gpt-5-mini       models:         - provider: openai           model: gpt-5-mini-2025-08-07
```

The examples above illustrate how to define a model groups consisting of a single model deployment. In order to handle a larger volume of conversations, it is recommended to include multiple model deployments within a model group. To do so you can add additional deployments to the `models` list as explained in the [Multi-LLM routing](https://rasa.com/docs/reference/deployment/multi-llm-routing/) page.

### Using different model groups in different components[​](https://rasa.com/docs/reference/config/components/llm-configuration/#using-different-model-groups-in-different-components 'Direct link to Using different model groups in different components')

config.yml

```
   recipe: default.v1   pipeline:   - name: LLMBasedRouter     calm_entry:       sticky: ...     nlu_entry:       sticky: ...     non_sticky: ...     llm:        model_group: openai-gpt-5-mini   - name: CompactLLMCommandGenerator     llm:        model_group: openai-gpt-5-1
```

Multiple components can rely on the same model group and a single component can use multiple models defined in a model group, via the [LLM router](https://rasa.com/docs/reference/deployment/multi-llm-routing/).

## Chat completion models[​](https://rasa.com/docs/reference/config/components/llm-configuration/#chat-completion-models 'Direct link to Chat completion models')

Default Provider

CALM is LLM agnostic and can be configured with different LLMs, but OpenAI is the default model provider. Majority of our experiments have been with models available on OpenAI or OpenAI Azure service. The performance of your assistant may vary when using other LLMs, but improvements can be made by tuning flow and collect step descriptions.

### OpenAI[​](https://rasa.com/docs/reference/config/components/llm-configuration/#openai 'Direct link to OpenAI')

#### API Token[​](https://rasa.com/docs/reference/config/components/llm-configuration/#api-token 'Direct link to API Token')

The API token authenticates your requests to the OpenAI API.

To configure the API token, follow these steps:

1. If you haven't already, sign up for an account on the OpenAI platform.
 
2. Navigate to the [OpenAI Key Management page](https://platform.openai.com/account/api-keys), and click on the "Create New Secret Key" button to initiate the process of obtaining `<your-api-key>`.
 
3. The API key can be set in the model configuration or through an environment variable.
 

To set the API key in the config, you can use the `api_key` parameter in the model configuration:

```
   model_groups:     - id: openai-direct       models:         - provider: openai           model: gpt-5.1-2025-11-13           api_key: ${MY_OPENAI_API_KEY}
```

info

The `api_key` parameter can be set in the model configuration for each model in the `model_groups` section of the `endpoints.yml` file. For security reasons, the value of the `api_key` must reference an environment variable, as demonstrated above. This approach ensures sensitive information is securely stored. Directly assigning the API key in the configuration file is not allowed, as it could potentially expose the key to unauthorized access.

To set the API key as an environment variable, you can use the following command in a terminal or command prompt:

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

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration 'Direct link to Configuration')

There are no additional OpenAI specific parameters to be configured. However, there could be model specific parameters like `temperature` that you might want to modify. Names for such parameters can found in [OpenAI's API documentation](https://platform.openai.com/docs/api-reference/chat) and defined under `llm` key of the component's configuration. Please refer to [LiteLLM's documentation](https://litellm.vercel.app/docs/providers/openai#openai-chat-completion-models) to know the list of models supported from the OpenAI platform.

#### Model deprecations[​](https://rasa.com/docs/reference/config/components/llm-configuration/#model-deprecations 'Direct link to Model deprecations')

OpenAI regularly publishes a deprecation schedule for its models. This schedule can be accessed in the [documentation published by OpenAI](https://platform.openai.com/docs/deprecations).

### Azure OpenAI Service[​](https://rasa.com/docs/reference/config/components/llm-configuration/#azure-openai-service 'Direct link to Azure OpenAI Service')

#### API Token[​](https://rasa.com/docs/reference/config/components/llm-configuration/#api-token-1 'Direct link to API Token')

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

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-1 'Direct link to Configuration')

To access models provided by [Azure OpenAI Service](https://azure.microsoft.com/en-in/products/ai-services/openai-service), there are a few additional parameters that need to be configured:

- `provider` - Set to `azure`.
- `api_base` - The URL for your Azure OpenAI instance. An example might look like this: `https://my-azure.openai.azure.com/`.
- `api_version` - The API version to use for this operation. This follows the YYYY-MM-DD format and the value should be enclosed in single or double quotes.
- `deployment` - Name of the deployment on Azure.

Model specific parameters like `temperature` can be defined as well. Refer to [OpenAI Azure service's API documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#request-body) for information on available parameter names.

A complete example configuration of the `CompactLLMCommandGenerator` using Azure OpenAI Service would look like this:

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: my_azure_deployment
```

endpoints.yml

```
   model_groups:     - id: my_azure_deployment       models:         - provider: azure           deployment: rasa-gpt-4o           api_base: https://my-azure.openai.azure.com/           api_version: "2025-02-01-preview"           api_key: ${MY_AZURE_API_KEY}           timeout: 7
```

- Linux/MacOS
- Windows

```
export MY_AZURE_API_KEY=<your-api-key>
```

```
setx MY_AZURE_API_KEY <your-api-key>
```

This will apply to future cmd prompt window, so you will need to open a new one to use that variable.

A more comprehensive example using the Azure OpenAI service in more CALM components is available [here](https://rasa.com/docs/reference/config/components/llm-configuration/#azure).

#### Model deprecations[​](https://rasa.com/docs/reference/config/components/llm-configuration/#model-deprecations-1 'Direct link to Model deprecations')

Azure regularly publishes a deprecation schedule for its models that come under the OpenAI Azure Service. This schedule can be accessed in the [documentation published by Azure](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/model-retirements).

#### Debugging[​](https://rasa.com/docs/reference/config/components/llm-configuration/#debugging 'Direct link to Debugging')

If you encounter timeout errors, configure `timeout` parameter to a larger value. The exact value depends on how your azure instance is configured.

### Amazon Bedrock[​](https://rasa.com/docs/reference/config/components/llm-configuration/#amazon-bedrock 'Direct link to Amazon Bedrock')

#### Requirements:[​](https://rasa.com/docs/reference/config/components/llm-configuration/#requirements 'Direct link to Requirements:')

1. Make sure you have `rasa-pro>=3.11.x` installed.
 
2. Install `boto3>=1.28.57`.
 
3. Make sure your AWS credentials are accessible via credentials, IAM role, or environment variables.
 
 - If you are using AWS credentials, ensure that the `~/.aws/credentials` file is set up with the correct access key and secret key.
 - If you are using an IAM role, ensure that the role has the necessary permissions to access Amazon Bedrock models and to have requested model access to the model of choice in AWS Bedrock. For example, you can use the following policy to allow access to all Bedrock models and to grant access to model invocation logging configuration, which is required during LLM client validation:
 
 ```
    {    "Version": "2012-10-17",    "Statement": [        {            "Effect": "Allow",            "Action": [                "bedrock:InvokeModel",                "bedrock:InvokeModelWithResponseStream"            ],            "Resource": "*"        },        {            "Effect": "Allow",            "Action": [                "bedrock:GetModelInvocationLoggingConfiguration"            ],            "Resource": "*"        }    ]}
    ```
 
 note
 
 `bedrock:InvokeModelWithResponseStream` is required for any streaming feature, including the `ContextualResponseRephraser` (`nlg.type: rephrase`). Without it, streaming calls will fail even if `bedrock:InvokeModel` is granted.
 
 - If you are using environment variables, ensure that the following variables are set:
 - `AWS_ACCESS_KEY_ID`
 - `AWS_SECRET_ACCESS_KEY`
 - `AWS_REGION`
 
 Alternatively set them via secrets in the environment and reference them in the model yaml configuration.
 
4. (Optional) Might have to set `AWS_SESSION_TOKEN` if your organisation mandates the usage of [temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) for security in your preferred authentication method.
 

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-2 'Direct link to Configuration')

Edit `config.yaml` to use an appropriate `model_group` from the `endpoints.yaml` file:

- Secrets set in environment
- Secrets set in configuration
- IAM role configuration

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: amazon_bedrock # Model group ID
```

endpoints.yml

```
   model_groups:     - id: amazon_bedrock       models:         - provider: bedrock           model: anthropic.claude-instant-v1
```

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: amazon_bedrock # Model group ID
```

endpoints.yml

```
   model_groups:     - id: amazon_bedrock       models:         - provider: bedrock           model: anthropic.claude-instant-v1           aws_access_key_id: ${MY_AWS_ACCESS_KEY_ID} # reference secret environment variable.           aws_secret_access_key: ${MY_AWS_SECRET_ACCESS_KEY} # reference secret environment variable.           aws_region_name: eu-central-1 # Does not require ${} as it is a not a secret.           aws_session_token: ${MY_AWS_SESSION_TOKEN} # reference secret environment variable.
```

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: amazon_bedrock # Model group ID
```

endpoints.yml

```
   model_groups:     - id: amazon_bedrock       models:         - provider: bedrock           model: anthropic.claude-3-7-sonnet-20250219-v1:0           # No need to set aws_access_key_id, aws_secret_access_key, aws_region_name or aws_session_token           # as they are automatically picked up from the IAM role attached to the Rasa Pro EC2 instance.           # You should provide the inference profile ARN to the model_id parameter.           model_id: arn:aws:bedrock:us-east-1:123456789012:inference-profile/us.anthropic.claude-3-7-sonnet-20250219-v1:0           aws_region_name: us-east-1
```

Set `provider` to `bedrock` and `model` to the model name you want to use.

Model specific parameters like `temperature` can be defined as well. Refer to LiteLLM's documentation for information on [available parameter names](https://litellm.vercel.app/docs/completion/input#translated-openai-params) and [supported models](https://litellm.vercel.app/docs/providers/bedrock#supported-aws-bedrock-models).

### Mistral[​](https://rasa.com/docs/reference/config/components/llm-configuration/#mistral 'Direct link to Mistral')

#### Requirements[​](https://rasa.com/docs/reference/config/components/llm-configuration/#requirements-1 'Direct link to Requirements')

1. Set the API key for Mistral platform to an environment variable `MISTRAL_API_KEY`.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-3 'Direct link to Configuration')

Define a deployment in `endpoints.yaml` and use the deployment in `config.yaml`

endpoints.yml

```
   model_groups:     - id: mistral_models       models:         - provider: mistral           model: mistral-large-latest           # api_key: ${MISTRAL_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: mistral_models # Model group ID
```

### Gemini - Google AI Studio[​](https://rasa.com/docs/reference/config/components/llm-configuration/#gemini---google-ai-studio 'Direct link to Gemini - Google AI Studio')

#### Requirements:[​](https://rasa.com/docs/reference/config/components/llm-configuration/#requirements-2 'Direct link to Requirements:')

1. Make sure you have `rasa-pro>=3.11.x` installed.
2. Install python package `google-generativeai`.
3. Get API Key at [https://aistudio.google.com/](https://aistudio.google.com/) .
4. Set the API key to an environment variable `GEMINI_API_KEY` or set it in the model configuration.

Once the above steps are complete, edit `config.yaml` to use an appropriate `model_group` from the `endpoints.yaml` and set `provider` to `gemini`:

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: gemini_llm # Model group ID
```

endpoints.yml

```
   model_groups:     - id: gemini_llm       models:         - provider: gemini           model: gemini-pro           # api_key: ${MY_GEMINI_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

Refer to LiteLLM's documentation to know which [additional parameters](https://litellm.vercel.app/docs/providers/gemini#supported-openai-params) and [models](https://litellm.vercel.app/docs/providers/gemini#chat-models) are supported.

### HuggingFace Inference Endpoints[​](https://rasa.com/docs/reference/config/components/llm-configuration/#huggingface-inference-endpoints 'Direct link to HuggingFace Inference Endpoints')

#### Requirements:[​](https://rasa.com/docs/reference/config/components/llm-configuration/#requirements-3 'Direct link to Requirements:')

1. Make sure you have `rasa-pro>=3.11.x` installed.
2. Set an API Key to the environment variable `HUGGINGFACE_API_KEY` or set it in the model configuration.
3. Edit `config.yaml` to use an appropriate `model_group` from the `endpoints.yaml`, set `provider` to `huggingface` and `api_base` to the base URL of the deployed endpoint:

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: huggingface_llm # Model group ID
```

endpoints.yml

```
   model_groups:     - id: huggingface_llm       models:         - provider: huggingface           model: meta-llama/CodeLlama-7b-Instruct-hf           api_base: "https://my-endpoint.huggingface.cloud"           # api_key: ${MY_HUGGINGFACE_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

### Self Hosted Model Server[​](https://rasa.com/docs/reference/config/components/llm-configuration/#self-hosted-model-server 'Direct link to Self Hosted Model Server')

CALM's components can also be configured to work with an open source LLM that is hosted on an open source model server like [vLLM](https://github.com/vllm-project/vllm)(recommended), [Ollama](https://ollama.com/) or [Llama.cpp web server](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#web-server). The only requirement is that the model server should adhere to the [OpenAI API format](https://platform.openai.com/docs/api-reference/introduction).

Once you have your model server running, configure the CALM assistant's `config.yaml` file to use a `model_group` from the `endpoints.yaml` file:

**vLLM**

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: self_hosted_llm # Model group ID
```

endpoints.yml

```
   model_groups:     - id: self_hosted_llm       models:         - provider: self-hosted           model: meta-llama/CodeLlama-7b-Instruct-hf           api_base: "https://my-endpoint/v1"           # api_key: ${HOSTED_VLLM_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

Important to note:

1. Recommended version of `vllm` to use is `0.6.0`.
 
2. CALM exclusively utilizes the chat completions endpoint of the model server, so it's essential that the model's tokenizer includes a chat template. Models lacking a chat template will have to set the `use_chat_completions_endpoint` parameter to `false` in the `model_groups` configuration.
 
 config.yml
 
 ```
        - name: CompactLLMCommandGenerator      llm:        model_group: self_hosted_llm # Model group ID
    ```
 
 endpoints.yml
 
 ```
      model_groups:    - id: self_hosted_llm      models:        - provider: self-hosted          model: meta-llama/CodeLlama-7b-Instruct-hf          api_base: "https://my-endpoint/v1"          use_chat_completions_endpoint: false
    ```
 
3. `model` should contain the name of the model supplied to the vllm startup command, for example if your model server is started with:
 
 ```
    vllm serve meta-llama/CodeLlama-7b-Instruct-hf
    ```
 
 `model` should be set to `meta-llama/CodeLlama-7b-Instruct-hf`.
 
4. `api_base` should contain the full exposed URL of the model server with `v1` attached as suffix to the URL.
 
5. If required, Set an API Key to the environment variable `HOSTED_VLLM_API_KEY` or set it in the model configuration.
 

**Ollama**

Once the ollama model server is running, edit the `config.yaml` file to use a model\_group from the `endpoints.yaml` file:

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: ollama_llm # Model group ID
```

endpoints.yml

```
   model_groups:     - id: ollama_llm       models:         - provider: ollama           model: llama3.1           api_base: "https://my-endpoint"           # api_key: ${OLLAMA_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

### Other Providers[​](https://rasa.com/docs/reference/config/components/llm-configuration/#other-providers 'Direct link to Other Providers')

info

If you want to try one of these providers, it is recommended to install Rasa Pro versions `>= 3.11`.

Other than the above mentioned providers, we have also tested support for the following providers:

| Platform | `provider` | API-KEY variable |
| --- | --- | --- |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` |
| Cohere | `cohere` | `COHERE_API_KEY` |
| Together AI | `together_ai` | `TOGETHERAI_API_KEY` |
| Groq | `groq` | `GROQ_API_KEY` |

For each of the above ones, ensure you have set an environment variable named by the value in `API-KEY variable` column to the API key of that platform or set it in the model configuration, and set the `provider` parameter under `llm` key of the component's config to the value in `provider` column.

## Embedding models[​](https://rasa.com/docs/reference/config/components/llm-configuration/#embedding-models 'Direct link to Embedding models')

To configure components that use an embedding model, reference the `model_group` from `endpoints.yaml` under the `embeddings` key

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    llm:      model: gpt_llm    flow_retrieval:      embeddings:        model_group: text_embedding_model
```

endpoints.yml

```
   model_groups:     - id: text_embedding_model       models:         - provider: openai           model: text-embedding-3-large           # api_key: ${OPENAI_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

The `embeddings` property needs the `model_group` key to be configured in the `config.yml` file and the corresponding model group should be defined in the `endpoints.yml` file.

The `models` key under `model_groups` in `endpoints.yaml` should contain the following parameters:

1. `model` - Specifies the name of the model identifier available from the LLM provider's documentation, for e.g. `text-embedding-3-large`.
 
2. `provider` - Unique identifier of the provider to be used for invoking the specified model, for e.g. `openai`
 

When configuring a particular provider, there are a few provider specific settings which are explained under each provider's individual sub-section below.

### OpenAI[​](https://rasa.com/docs/reference/config/components/llm-configuration/#openai-1 'Direct link to OpenAI')

OpenAI is used as the default embedding model provider. To start using, ensure you have configured an API token as you would do for a [chat completion model from OpenAI platform](https://rasa.com/docs/reference/config/components/llm-configuration/#api-token)

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-4 'Direct link to Configuration')

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    llm:      model_group: openai_llm    flow_retrieval:      embeddings:        model_group: text_embedding_model
```

endpoints.yml

```
   model_groups:     - id: openai_llm       models:         - provider: openai           model: gpt-5.1-2025-11-13           # api_key: ${OPENAI_API_KEY_1} # Optional, if you want to set the API key in the model configuration.      - id: text_embedding_model        models:          - provider: openai            model: text-embedding-3-large            # api_key: ${OPENAI_API_KEY_2} # Optional, if you want to set the API key in the model configuration.
```

### Azure OpenAI Service[​](https://rasa.com/docs/reference/config/components/llm-configuration/#azure-openai-service-1 'Direct link to Azure OpenAI Service')

Ensure you have configured an API token as you would do for a [chat completion model for Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration/#api-token-1)

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-5 'Direct link to Configuration')

Configuring an embedding model from Azure OpenAI Service needs values for the same set of parameters that are required for configuring a [chat completion model from Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-1)

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    llm:      model_group: openai_direct    flow_retrieval:      embeddings:        model_group: azure_embedding_model
```

endpoints.yml

```
   model_groups:     - id: openai_direct       models:         - provider: openai           model: gpt-5.1-2025-11-13           # api_key: ${OPENAI_API_KEY} # Optional, if you want to set the API key in the model configuration.     - id: azure_embedding_model       models:         - provider: azure           deployment: test-embeddings           api_base: https://my-azure.openai.azure.com/           api_version: "2024-02-15-preview"           timeout: 7           # api_key: ${AZURE_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

info

From rasa pro `3.11`, deployments from multiple azure subscriptions can be used in the model configurations.

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    llm:      model_group: azure_llm    flow_retrieval:      embeddings:        model_group: azure_embedding_model
```

endpoints.yml

```
   model_groups:     - id: azure_llm       models:         - provider: azure           deployment: rasa-gpt-4           api_base: https://azure-server1.com/           api_version: "2024-02-15-preview"           api_key: ${AZURE_API_KEY_1}     - id: azure_embedding_model       models:         - provider: azure           deployment: test-embeddings           api_base: https://azure-server2.com/           api_version: "2024-02-15-preview"           api_key: ${AZURE_API_KEY_1}
```

### Amazon Bedrock[​](https://rasa.com/docs/reference/config/components/llm-configuration/#amazon-bedrock-1 'Direct link to Amazon Bedrock')

Configuring an embedding model from amazon bedrock needs the same pre-requisites as a [chat completion model](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-2). Please ensure you have addressed these before proceeding further.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-6 'Direct link to Configuration')

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    llm:      model_group: openai_llm    flow_retrieval:      embeddings:        model_group: amazon_bedrock_model
```

endpoints.yml

```
   model_groups:     - id: openai_llm       models:         - provider: openai           model: gpt-5.1-2025-11-13           # api_key: ${OPENAI_API_KEY} # Optional, if you want to set the API key in the model configuration.     - id: amazon_bedrock_model       models:         - provider: bedrock           model: amazon.titan-embed-text-v1           # aws_access_key_id: ${MY_AWS_ACCESS_KEY_ID} # Optional, if you want to set the AWS access key in the model configuration.           # aws_secret_access_key: ${MY_AWS_SECRET_ACCESS_KEY} # Optional, if you want to set the AWS secret access key in the model configuration.           # aws_region_name: eu-central-1 # Optional, if you want to set the AWS region name in the model configuration.           # aws_session_token: ${MY_AWS_SESSION_TOKEN} # Optional, if you want to set the AWS session token in the model configuration.
```

Please refer to LiteLLM's documentation on [list of supported embedding models](https://litellm.vercel.app/docs/providers/bedrock#supported-aws-bedrock-embedding-models) from Amazon Bedrock

### Mistral[​](https://rasa.com/docs/reference/config/components/llm-configuration/#mistral-1 'Direct link to Mistral')

Configuring an embedding model from mistral needs the same pre-requisites as a [chat completion model](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-3). Please ensure you have addressed these before proceeding further.

Define a deployment in `endpoints.yaml` and use the deployment in `config.yaml`

endpoints.yml

```
   model_groups:     - id: mistral_llm       models:         - provider: mistral           model: mistral-large-latest           # api_key: ${MISTRAL_API_KEY} # Optional, if you want to set the API key in the model configuration.     - id: mistral_embed       models:         - provider: mistral           model: mistral-embed           # api_key: ${MISTRAL_API_KEY} # Optional, if you want to set the API key in the model configuration.
```

config.yml

```
    - name: CompactLLMCommandGenerator      llm:        model_group: mistral_llm # Model group ID      flow_retrieval:        embeddings:          model_group: mistral_embed
```

### In-Memory[​](https://rasa.com/docs/reference/config/components/llm-configuration/#in-memory 'Direct link to In-Memory')

CALM also provides an option to load lightweight embedding models in-memory without needing them to be exposed over an API. It uses the sentence transformers library under the hood to load and run inference on them.

#### Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuration-7 'Direct link to Configuration')

config.yml

```
pipeline:  - name: CompactLLMCommandGenerator    llm:      model_group: openai_direct    flow_retrieval:      embeddings:        model_group: huggingface_embedding_model
```

endpoints.yml

```
   model_groups:     - id: openai_direct       models:         - provider: openai           model: gpt-5.1-2025-11-13           # api_key: ${OPENAI_API_KEY} # Optional, if you want to set the API key in the model configuration.      - id: huggingface_embedding_model        models:          - provider: huggingface            model: BAAI/bge-small-en-v1.5            model_kwargs: # used during instantiation              device: "cpu"            encode_kwargs: # used during inference              normalize_embeddings: true
```

- `model` parameter can take as value either any embedding model repository available on the HuggingFace hub or a path to a local model.
- `model_kwargs` parameter is used to provide [load time arguments](https://sbert.net/docs/package_reference/sentence_transformer/SentenceTransformer.html#id1) to the sentence transformer library.
- `encode_kwargs` parameter is used to provide [inference time arguments](https://sbert.net/docs/package_reference/sentence_transformer/SentenceTransformer.html#sentence_transformers.SentenceTransformer.encode) to the sentence transformer library.

### Other Providers[​](https://rasa.com/docs/reference/config/components/llm-configuration/#other-providers-1 'Direct link to Other Providers')

Other than the above mentioned providers, we have also tested support for the following providers -

| Platform | `provider` | API-KEY variable |
| --- | --- | --- |
| Cohere | `cohere` | `COHERE_API_KEY` |
| Voyage AI | `voyage` | `VOYAGE_API_KEY` |

For each of the above ones, ensure you have set an environment variable named by the value in `API-KEY variable` column to the API key of that platform or set it in the model configuration, and set the `provider` parameter under `llm` key of the component's config to the value in `provider` column.

## Configuring self-signed SSL certificates[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuring-self-signed-ssl-certificates 'Direct link to Configuring self-signed SSL certificates')

In environments where a proxy performs TLS interception, Rasa may need to be configured to trust the certificates used by your proxy. By default, certificates are loaded from the OS certificate store. However, if your setup involves custom self-signed certificates, you can specify these by setting the `RASA_CA_BUNDLE` environment variable.

This variable points to the path of the certificate file that Rasa should use to validate SSL connections:

```
export RASA_CA_BUNDLE="path/to/your/certificate.pem"
```

info

The `REQUESTS_CA_BUNDLE` environment variable is deprecated and will no longer be supported in future versions. Please use RASA\_CA\_BUNDLE instead to ensure compatibility.

## Configuring Proxy URLs[​](https://rasa.com/docs/reference/config/components/llm-configuration/#configuring-proxy-urls 'Direct link to Configuring Proxy URLs')

In environments where LLM requests need to be routed through a proxy, Rasa relies on LiteLLM to handle proxy configurations. LiteLLM supports configuring proxy URLs through the `HTTP_PROXY` and `HTTPS_PROXY` environment variables.

To ensure that all LLM requests are routed through the proxy, you can set the environment variables as follows:

```
export HTTP_PROXY="http://your-proxy-url:port"export HTTPS_PROXY="https://your-proxy-url:port"
```

Another way to configure the proxy is to set the `api_base` parameter in the model configuration to the proxy URL:

endpoints.yml

```
   model_groups:     - id: self_hosted_llm       models:         - provider: self-hosted           model: meta-llama/CodeLlama-7b-Instruct-hf           api_base: http://your-proxy-url:port
```

## Recommended Models[​](https://rasa.com/docs/reference/config/components/llm-configuration/#recommended-models 'Direct link to Recommended Models')

The table below documents the versions of each model we recommend for use with various Rasa components. As new models are published, Rasa will test these and where appropriate add them as a recommended model.

| Component | Providing platform | Recommended models |
| --- | --- | --- |
| `CompactLLMCommandGenerator`, `SearchReadyLLMCommandGenerator` | OpenAI, Azure, Anthropic, Bedrock | `gpt-5.1-2025-11-13` (default chat model; see [LLM Command Generators](https://rasa.com/docs/reference/config/components/llm-command-generators/#types-of-llm-based-command-generators) for prompt templates mapped to other models) |
| `LLMBasedRouter` | OpenAI, Azure | `gpt-5-mini-2025-08-07` |
| `ContextualResponseRephraser` | OpenAI, Azure | `gpt-5.1-2025-11-13` |
| `EnterpriseSearchPolicy` | OpenAI, Azure | `gpt-5-mini-2025-08-07` (generative LLM); default embeddings `text-embedding-3-large` |

## OAuth Authentication for Azure OpenAI[​](https://rasa.com/docs/reference/config/components/llm-configuration/#oauth-authentication-for-azure-openai 'Direct link to OAuth Authentication for Azure OpenAI')

New in Rasa 3.12

Rasa supports OAuth authentication for instances deployed on Azure OpenAI. This feature includes built-in implementation for OAuth over Entra ID and allows customers to provide their own custom OAuth integration for Azure OpenAI instances as a custom component.

You can specify the OAuth configuration under the `oauth` field in the model group. The `api_key` and `oauth` fields are mutually exclusive; an error will be thrown if neither or both are specified.

### Setup[​](https://rasa.com/docs/reference/config/components/llm-configuration/#setup 'Direct link to Setup')

Prerequisites you will need:

- [Azure Entra ID app](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app?tabs=certificate%2Cexpose-a-web-api) which provides the access token (JWT).
- An API gateway ([Azure API Management service](https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts)) between the user and Azure OpenAI instance. API gateway is setup to validate the JWT access token and forward the request to the Azure OpenAI instance.

### Authentication Flow[​](https://rasa.com/docs/reference/config/components/llm-configuration/#authentication-flow 'Direct link to Authentication Flow')

1. Configure Rasa with OAuth method and required parameters.
2. Rasa contacts Entra ID to obtain an access token using the `azure-identity` library.
3. The access token is injected into the completion request sent to the API gateway (Azure API Management service).
4. If access token is valid, the API gateway forwards the request to the Azure OpenAI instance.

### Authentication Options[​](https://rasa.com/docs/reference/config/components/llm-configuration/#authentication-options 'Direct link to Authentication Options')

Rasa supports several OAuth methods for Azure OpenAI instances:

1. **Client Secret Authentication**: Use `azure_entra_id_client_secret` for client secret authentication.
2. **Client Certificate Authentication**: Use `azure_entra_id_client_certificate` for client certificate authentication.
3. **Default Azure Credentials**: Use `azure_entra_id_default` for Azure SDK default credentials resolution.
4. **Custom OAuth Component**: Implement a custom OAuth component by inheriting from the `OAuth` interface.

OAuth authentication method is configured per-model in the model group. The following sections provide examples of each authentication method.

### Client Secret Authentication[​](https://rasa.com/docs/reference/config/components/llm-configuration/#client-secret-authentication 'Direct link to Client Secret Authentication')

| Field | Description | Type | Required | Default | Can be set in the environment |
| --- | --- | --- | --- | --- | --- |
| `type` | Fixed `azure_entra_id_client_secret` | String | Yes | | Yes |
| `scopes` | OAuth scopes | String or List | Yes | | Yes |
| `client_id` | Azure client ID | String | Yes | | Yes |
| `tenant_id` | Azure tenant ID | String | Yes | | Yes |
| `client_secret` | Azure client secret | String | Yes | | Yes |
| `authority_host` | Azure authority host | String | No | `https://login.microsoftonline.com` | Yes |

Example:

```
model_groups:  - id: devtribe_oauth_client_secret    models:      - provider: azure        model: gpt-4        deployment: test-gpt-4        api_base: ${AZURE_API_GATEWAY_URL_FOR_AZURE_OPENAI_INSTANCE}        api_version: 2024-03-01-preview        oauth:          type: azure_entra_id_client_secret          scopes: ${AZURE_APP_SCOPE}          client_id: ${AZURE_CLIENT_ID}          tenant_id: ${AZURE_TENANT_ID}          client_secret: ${AZURE_CLIENT_SECRET}          authority_host: ${AZURE_AUTHORITY} # optional
```

### Client Certificate Authentication[​](https://rasa.com/docs/reference/config/components/llm-configuration/#client-certificate-authentication 'Direct link to Client Certificate Authentication')

| Field | Description | Type | Required | Default | Can be set in the environment |
| --- | --- | --- | --- | --- | --- |
| `type` | Fixed `azure_entra_id_client_certificate` | String | Yes | | Yes |
| `scopes` | OAuth scopes | String or List | Yes | | Yes |
| `client_id` | Azure client ID | String | Yes | | Yes |
| `tenant_id` | Azure tenant ID | String | Yes | | Yes |
| `certificate_path` | Path to the client certificate (both key and certificate in one file) | String | Yes | | Yes |
| `certificate_password` | Certificate password | String | No | | Yes |
| `send_certificate_chain` | Send certificate chain | Boolean | No | `False` | Yes |
| `authority_host` | Azure authority host | String | No | `https://login.microsoftonline.com` | Yes |
| `disable_instance_discovery` | \[Disables instance discovery\] ([https://learn.microsoft.com/en-us/dotnet/api/azure.identity.clientsecretcredentialoptions.disableinstancediscovery?view=azure-dotnet](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.clientsecretcredentialoptions.disableinstancediscovery?view=azure-dotnet)) | Boolean | No | `False` | Yes |

Example:

```
model_groups:  - id: devtribe_oauth_client_certificate    models:      - provider: azure        model: gpt-4        deployment: test-gpt-4        api_base: ${AZURE_API_GATEWAY_URL_FOR_AZURE_OPENAI_INSTANCE}        api_version: 2024-03-01-preview        oauth:          type: azure_entra_id_client_certificate          scopes: ${AZURE_APP_SCOPE}          client_id: ${AZURE_CLIENT_ID}          tenant_id: ${AZURE_TENANT_ID}          certificate_path: ${AZURE_CERTIFICATE_PATH}          certificate_password: ${AZURE_CERTIFICATE_PASSWORD} # optional          send_certificate_chain: False # optional          authority_host: ${AZURE_AUTHORITY} # optional          disable_instance_discovery: False # optional
```

### Default Azure Credentials Resolver[​](https://rasa.com/docs/reference/config/components/llm-configuration/#default-azure-credentials-resolver 'Direct link to Default Azure Credentials Resolver')

This approach uses the Default Azure Credentials resolver. This method sequentially resolves between several authentication methods, built-in in Azure SDK, and uses the first successful one. You can read more about the [DefaultAzureCredentials](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) class in the Azure Identity Library documentation.

| Field | Description | Type | Required | Default | Can be set in the environment |
| --- | --- | --- | --- | --- | --- |
| `type` | Fixed `azure_entra_id_default` | String | Yes | | Yes |
| `scopes` | OAuth scopes | String or List | Yes | | Yes |

```
model_groups:  - id: devtribe_oauth_default    models:      - provider: azure        model: gpt-4        deployment: test-gpt-4        api_base: ${AZURE_API_GATEWAY_URL_FOR_AZURE_OPENAI_INSTANCE}        api_version: 2024-03-01-preview        oauth:          type: azure_entra_id_default          scopes: ${AZURE_SCOPE}
```

This method is useful when you want to:

- Use builtin Azure SDK env vars for authentication
- Authenticate using the Azure CLI

### OAuth with LiteLLM Router Configuration[​](https://rasa.com/docs/reference/config/components/llm-configuration/#oauth-with-litellm-router-configuration 'Direct link to OAuth with LiteLLM Router Configuration')

When using the LiteLLM router, you can configure OAuth authentication for multiple models. Here's an example configuration:

```
model_groups:  - id: devtribe_oauth_mixed    models:      - provider: azure        deployment: test-gpt-4        api_base: ${AZURE_API_GATEWAY_URL_1}        api_version: 2024-03-01-preview        oauth:          type: azure_entra_id_client_secret          scopes: ${AZURE_APP_SCOPE_1}          client_id: ${AZURE_CLIENT_ID_1}          tenant_id: ${AZURE_TENANT_ID_1}          client_secret: ${AZURE_CLIENT_SECRET_1}      - provider: azure        deployment: test-gpt-4o        api_base: ${AZURE_API_GATEWAY_URL_2}        api_version: 2024-03-01-preview        oauth:          type: azure_entra_id_default          scopes: ${AZURE_SCOPE_2}    router:      routing_strategy: least-busy
```

In this example, we have two models with different authentication methods:

1. The first model uses client secret authentication.
2. The second model uses the Default Azure Credentials resolver.

The router is configured to use the "least-busy" routing strategy.

### Custom OAuth Component[​](https://rasa.com/docs/reference/config/components/llm-configuration/#custom-oauth-component 'Direct link to Custom OAuth Component')

Customers can implement a custom OAuth component by inheriting from the `OAuth` interface and implementing the `from_config` and `get_bearer_token` methods. This allows for tailored OAuth implementations when needed.

Example:

We have a custom OAuth component `AzureEntraIDClientCreds` which implements the `OAuth` interface and is located at: `addons/oauth/azure_entra_id_client_creds.py`. This path is relative to the working directory of the Rasa instance.

azure\_entra\_id\_client\_creds.py

```
from rasa.shared.providers._configs.oauth_config import OAuth@dataclassclass AzureEntraIDClientCreds(OAuth):    client_id: str    client_secret: str    tenant_id: str    scopes: List[str]    @classmethod    def from_config(cls, config: Dict[str, Any]) -> CustomOAuth:        # Method from_config is        # called with the config: Dict[str, Any]        # (what is under the oauth section in endpoints.yml)        # and returns an instance of the class which implements the OAuth interface.        # Implementation        scopes = config.get("scopes")        if isinstance(scopes, str):            scopes = [scopes]        return cls(            client_id=config.get("client_id"),            client_secret=config.get("client_secret"),            tenant_id=config.get("tenant_id"),            scopes=scopes,        )    def get_bearer_token(self) -> str:        # Method get_bearer_token returns a bearer token which        # will be used in a call to the Azure OpenAI        # instance which is protected by the Gateway (Azure API Management service).        # Implementation        client = create_azure_entra_id_client_credentials(            client_id=self.client_id,            client_secret=self.client_secret,            tenant_id=self.tenant_id,            authority_host=self.authority_host,            disable_instance_discovery=self.disable_instance_discovery,        )        return client.get_token(*self.scopes).token# We cache the client credential creation using the `lru_cache`# decorator to avoid creating a new client for each request.# This makes sure that refresh token is used to obtain the access# token when the token expires and that client credentials/certificate are not# used to obtain the token for each request.@lru_cachedef create_azure_entra_id_client_credentials(    client_id: str,    client_secret: str,    tenant_id: str,    authority_host: Optional[str] = None,    disable_instance_discovery: bool = False,) -> ClientSecretCredential:   return ClientSecretCredential(        client_id=client_id,        client_secret=client_secret,        tenant_id=tenant_id,        authority=authority_host,        disable_instance_discovery=disable_instance_discovery,    )
```

endpoints.yml

```
model_groups:  - id: devtribe_oauth_client_secret    models:      - provider: azure        model: gpt-4        deployment: test-gpt-4        api_base: ${AZURE_API_GATEWAY_URL_FOR_AZURE_OPENAI_INSTANCE}        api_version: 2024-03-01-preview        oauth:          # path to the custom OAuth component relative to the working directory          type: addons.oauth.azure_entra_id_client_creds.AzureEntraIDClientCreds          scopes: ${AZURE_SCOPE_CUSTOM}          client_id: ${AZURE_CLIENT_ID_CUSTOM}          tenant_id: ${AZURE_TENANT_ID_CUSTOM}          client_secret: ${AZURE_CLIENT_SECRET}
```

## FAQ[​](https://rasa.com/docs/reference/config/components/llm-configuration/#faq 'Direct link to FAQ')

### Does OpenAI use my data to train their models?[​](https://rasa.com/docs/reference/config/components/llm-configuration/#does-openai-use-my-data-to-train-their-models 'Direct link to Does OpenAI use my data to train their models?')

No. OpenAI does not use your data to train their models. From their [website](https://openai.com/enterprise-privacy/):

> We do not train our models on your business data by default.

## Example Configurations[​](https://rasa.com/docs/reference/config/components/llm-configuration/#example-configurations 'Direct link to Example Configurations')

### Azure[​](https://rasa.com/docs/reference/config/components/llm-configuration/#azure 'Direct link to Azure')

A comprehensive example which includes:

- `llm` and `embeddings` configuration for components in `config.yml`:
 - `EnterpriseSearchPolicy`
 - `SearchReadyLLMCommandGenerator`
 - `flow_retrieval` in 3.8.x
- `llm` configuration for rephrase in `endpoints.yml` (`ContextualResponseRephraser`)

endpoints.yml

```
nlg:  type: rephrase  llm:    model_group: azure_llmmodel_groups:  - id: azure_llm    models:      - provider: azure        deployment: rasa-gpt-4o        api_base: https://my-azure.openai.azure1.com/        api_version: "2024-02-15-preview"        # api_key: ${MY_AZURE_API_KEY_1} # Optional, if you want to set the API key in the model configuration.        timeout: 7  - id: azure_embeddings    models:      - provider: azure        deployment: rasa-embedding-small        api_base: https://my-azure.openai.azure2.com/        api_version: "2024-02-15-preview"        # api_key: ${MY_AZURE_API_KEY_2} # Optional, if you want to set the API key in the model configuration.        timeout: 7
```

config.yml

```
    recipe: default.v1    language: en    pipeline:    - name: SearchReadyLLMCommandGenerator      llm:        model_group: azure_llm      flow_retrieval:        embeddings:          model_group: azure_embeddings    policies:    - name: FlowPolicy    - name: EnterpriseSearchPolicy      vector_store:        type: "faiss"        threshold: 0.0      llm:        model_group: azure_llm      embeddings:        model_group: azure_embeddings
```

- [Overview](https://rasa.com/docs/reference/config/components/llm-configuration/#overview)
- [Declaring LLM deployments](https://rasa.com/docs/reference/config/components/llm-configuration/#declaring-llm-deployments)
 - [Model Groups](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups)
 - [Defining a single model group](https://rasa.com/docs/reference/config/components/llm-configuration/#defining-a-single-model-group)
 - [Using a model group in a component](https://rasa.com/docs/reference/config/components/llm-configuration/#using-a-model-group-in-a-component)
 - [Defining multiple model groups](https://rasa.com/docs/reference/config/components/llm-configuration/#defining-multiple-model-groups)
 - [Using different model groups in different components](https://rasa.com/docs/reference/config/components/llm-configuration/#using-different-model-groups-in-different-components)
- [Chat completion models](https://rasa.com/docs/reference/config/components/llm-configuration/#chat-completion-models)
 - [OpenAI](https://rasa.com/docs/reference/config/components/llm-configuration/#openai)
 - [Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration/#azure-openai-service)
 - [Amazon Bedrock](https://rasa.com/docs/reference/config/components/llm-configuration/#amazon-bedrock)
 - [Mistral](https://rasa.com/docs/reference/config/components/llm-configuration/#mistral)
 - [Gemini - Google AI Studio](https://rasa.com/docs/reference/config/components/llm-configuration/#gemini---google-ai-studio)
 - [HuggingFace Inference Endpoints](https://rasa.com/docs/reference/config/components/llm-configuration/#huggingface-inference-endpoints)
 - [Self Hosted Model Server](https://rasa.com/docs/reference/config/components/llm-configuration/#self-hosted-model-server)
 - [Other Providers](https://rasa.com/docs/reference/config/components/llm-configuration/#other-providers)
- [Embedding models](https://rasa.com/docs/reference/config/components/llm-configuration/#embedding-models)
 - [OpenAI](https://rasa.com/docs/reference/config/components/llm-configuration/#openai-1)
 - [Azure OpenAI Service](https://rasa.com/docs/reference/config/components/llm-configuration/#azure-openai-service-1)
 - [Amazon Bedrock](https://rasa.com/docs/reference/config/components/llm-configuration/#amazon-bedrock-1)
 - [Mistral](https://rasa.com/docs/reference/config/components/llm-configuration/#mistral-1)
 - [In-Memory](https://rasa.com/docs/reference/config/components/llm-configuration/#in-memory)
 - [Other Providers](https://rasa.com/docs/reference/config/components/llm-configuration/#other-providers-1)
- [Configuring self-signed SSL certificates](https://rasa.com/docs/reference/config/components/llm-configuration/#configuring-self-signed-ssl-certificates)
- [Configuring Proxy URLs](https://rasa.com/docs/reference/config/components/llm-configuration/#configuring-proxy-urls)
- [Recommended Models](https://rasa.com/docs/reference/config/components/llm-configuration/#recommended-models)
- [OAuth Authentication for Azure OpenAI](https://rasa.com/docs/reference/config/components/llm-configuration/#oauth-authentication-for-azure-openai)
 - [Setup](https://rasa.com/docs/reference/config/components/llm-configuration/#setup)
 - [Authentication Flow](https://rasa.com/docs/reference/config/components/llm-configuration/#authentication-flow)
 - [Authentication Options](https://rasa.com/docs/reference/config/components/llm-configuration/#authentication-options)
 - [Client Secret Authentication](https://rasa.com/docs/reference/config/components/llm-configuration/#client-secret-authentication)
 - [Client Certificate Authentication](https://rasa.com/docs/reference/config/components/llm-configuration/#client-certificate-authentication)
 - [Default Azure Credentials Resolver](https://rasa.com/docs/reference/config/components/llm-configuration/#default-azure-credentials-resolver)
 - [OAuth with LiteLLM Router Configuration](https://rasa.com/docs/reference/config/components/llm-configuration/#oauth-with-litellm-router-configuration)
 - [Custom OAuth Component](https://rasa.com/docs/reference/config/components/llm-configuration/#custom-oauth-component)
- [FAQ](https://rasa.com/docs/reference/config/components/llm-configuration/#faq)
 - [Does OpenAI use my data to train their models?](https://rasa.com/docs/reference/config/components/llm-configuration/#does-openai-use-my-data-to-train-their-models)
- [Example Configurations](https://rasa.com/docs/reference/config/components/llm-configuration/#example-configurations)
 - [Azure](https://rasa.com/docs/reference/config/components/llm-configuration/#azure)