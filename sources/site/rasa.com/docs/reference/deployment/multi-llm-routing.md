# Source: https://rasa.com/docs/reference/deployment/multi-llm-routing

On this page

New in 3.11

The _Multi-LLM-Routing_ is available starting with version `3.11.0`.

## Overview[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#overview 'Direct link to Overview')

The Router for LLM and embeddings is a feature that allows you to distribute and load balance requests across multiple LLMs and embeddings. This feature uses [LiteLLM](https://litellm.vercel.app/docs/routing) under the hood to implement the routing logic.

## Configuration[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuration 'Direct link to Configuration')

To enable the Router for LLM and embeddings, you need to define the `router` key to your [model group](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups) configuration in your `endpoints.yml` file.

endpoints.yml

```
   model_groups:     - id: azure_llm_deployments       models:         - provider: azure           deployment: rasa-gpt-4           api_base: https://azure-deployment/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY}       router:           routing_strategy: simple-shuffle
```

The above example demonstrates a router with just one model. You can add multiple models to the `models` key in the model group configuration. There are no limit to the number models that you can add to the model group configuration.

endpoints.yml

```
   model_groups:     - id: azure_llm_deployments       models:         - provider: azure           deployment: gpt-4-instance-france           api_base: https://azure-deployment-france/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_FRANCE}         - provider: azure           deployment: gpt-4-instance-canada           api_base: https://azure-deployment-canada/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_CANADA}       router:           routing_strategy: simple-shuffle
```

warning

The model deployments in the model group configuration are advised to be using the same underlying model. For example, defining different versions of `gpt-4` or a mix of `gpt-4` and `gpt-3.5-turbo` is not recommended because the component relying on the model group will not receive consistent outputs from the different underlying models. The routing feature is designed to distribute requests across different deployments of the same model.

The `routing_strategy` key defines the routing strategy that the Router will use to distribute requests.

The following routing strategies are available without any additional configuration:

- `simple-shuffle`: The Router will shuffle the requests and distribute them based on RPM (requests per minute) or weight.
- `least-busy`: The Router will distribute requests to the deployment that has least number of ongoing requests.
- `latency-based-routing`: The Router will distribute requests to the deployment with the lowest response time.

The following routing strategies require additional redis setup as they work based on the caching mechanism:

- `cost-based-routing`: The Router will distribute requests to the deployment with the lowest cost.
- `usage-based-routing`: The Router will distribute requests to the deployment with the lowest TPM (Tokens per minute) usage.

Refer to the [LiteLLM's routing strategy documentation](https://litellm.vercel.app/docs/routing#advanced---routing-strategies-%EF%B8%8F) for more information on the routing strategies.

### Additional configuration parameters for the Router[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#additional-configuration-parameters-for-the-router 'Direct link to Additional configuration parameters for the Router')

The Router can be configured with additional parameters like `cooldown_time`, `num_retries` and `allowed_fails` to fine-tune the routing logic. Refer to the [LiteLLM's routing configuration documentation](https://docs.litellm.ai/docs/routing#init-params-for-the-litellmrouter) for more information on the configuration parameters.

endpoints.yml

```
   router:       routing_strategy: simple-shuffle       cooldown_time: 10 # in seconds - time to wait before retrying a failed request.       allowed_fails: 2 # number of allowed fails before the deployment is marked for cooldown.       num_retries: 3 # number of retries.
```

### Configuring the Router for multiple model groups[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-multiple-model-groups 'Direct link to Configuring the Router for multiple model groups')

endpoints.yml

```
   model_groups:     - id: azure_gpt4_deployments       models:         - provider: azure           deployment: gpt-4-instance-france           api_base: https://azure-deployment-france/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_FRANCE}         - provider: azure           deployment: gpt-4-instance-canada           api_base: https://azure-deployment-canada/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_CANADA}       router:           routing_strategy: least-busy     - id: azure_gpt35_turbo_deployments       models:         - provider: azure           deployment: gpt-35-instance-france           api_base: https://azure-deployment-france/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_FRANCE}         - provider: azure           deployment: gpt-35-instance-canada           api_base: https://azure-deployment-canada/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_CANADA}       router:           routing_strategy: simple-shuffle
```

The above example demonstrates a configuration with two model groups. Each model group has two models and uses the `least-busy` and `simple-shuffle` routing strategies respectively. The router settings for each model group are defined under the `router` key in the model group configuration and are independent of each other.

### Configuring the Router for embeddings[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-embeddings 'Direct link to Configuring the Router for embeddings')

While the examples above demonstrate the configuration for LLMs, the Router can also be configured for embeddings and all the routing settings and strategies mentioned above can be used for embeddings as well.

endpoints.yml

```
   model_groups:     - id: azure_embeddings_deployments       models:         - provider: azure           deployment: text-embeddings-instance-france           api_base: https://azure-deployment-embeddings-france/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_FRANCE}         - provider: azure           deployment: text-embeddings-instance-canada           api_base: https://azure-deployment-embeddings-canada/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_CANADA}       router:           routing_strategy: simple-shuffle
```

### Configuring the Router for other providers[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-other-providers 'Direct link to Configuring the Router for other providers')

The Router can be configured for other providers as well. The configuration for other providers is similar to the Azure provider configuration demonstrated above. Refer to the [LLM and embeddings provider documentation](https://rasa.com/docs/reference/config/components/llm-configuration/) for more information on the provider specific configuration.

## Configuring the Router for self-hosted deployments[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-self-hosted-deployments 'Direct link to Configuring the Router for self-hosted deployments')

The Router can also be configured for self-hosted deployments. The configuration for self-hosted deployments is similar to the Azure provider configuration demonstrated above. Some examples for different providers are shown below:

### vLLM[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#vllm 'Direct link to vLLM')

endpoints.yml

```
   model_groups:     - id: vllm_deployments       models:         - provider: self-hosted           model: meta-llama/Meta-Llama-3-8B #model name           api_base: "http://localhost:8000/v1" # hosted on port 8000           # use_chat_completions_endpoint: false # Can't be used here when using router.         - provider: self-hosted           model: meta-llama/Meta-Llama-3-8B #model name           api_base: "http://localhost:8001/v1" # hosted on port 8001       router:           routing_strategy: least-busy           use_chat_completions_endpoint: false
```

The `use_chat_completions_endpoint` parameter is used to enable/disable the chat completions endpoint for the model. This parameter is optional and is set to `true` by default. For more information, refer to the [LLM configuration documentation](https://rasa.com/docs/reference/config/components/llm-configuration/#self-hosted-model-server).

With the Router enabled, the `use_chat_completions_endpoint` parameter should be set as a router level setting and not at the model level.

### LLama.cpp[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#llamacpp 'Direct link to LLama.cpp')

endpoints.yml

```
   model_groups:     - id: llamacpp_deployments       models:         - provider: self-hosted           model: ggml-org/Meta-Llama-3.1-8B-Instruct-Q4_0-GGUF #model name           api_base: "http://localhost:8080/v1" # hosted on port 8080         - provider: self-hosted           model: ggml-org/Meta-Llama-3.1-8B-Instruct-Q4_0-GGUF #model name           api_base: "http://localhost:8081/v1" # hosted on port 8081       router:           routing_strategy: least-busy
```

### Ollama[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#ollama 'Direct link to Ollama')

endpoints.yml

```
   model_groups:     - id: ollama_deployments       models:         - provider: ollama           model: llama3.1 #model name           api_base: "http://localhost:11434" # hosted on port 11434         - provider: ollama           model: llama3.1 #model name           api_base: "http://localhost:11435" # hosted on port 11435       router:           routing_strategy: least-busy
```

### Connecting via a proxy[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#connecting-via-a-proxy 'Direct link to Connecting via a proxy')

endpoints.yml

```
   model_groups:     - id: vllm_deployments       models:         - provider: self-hosted           model: meta-llama/Meta-Llama-3-8B #model name           api_base: "http://your-proxy-url-1" # URL of the proxy server         - provider: self-hosted           model: meta-llama/Meta-Llama-3-8B #model name           api_base: "http://your-proxy-url-2" # URL of the proxy server       router:           routing_strategy: least-busy
```

### Connecting via litellm proxy[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#connecting-via-litellm-proxy 'Direct link to Connecting via litellm proxy')

endpoints.yml

```
   model_groups:     - id: litellm_proxy_deployments       models:         - provider: litellm_proxy # provider name           model: gpt-4-instance-1 # model name           api_base: "http://localhost:4000" # URL of the litellm proxy server         - provider: litellm_proxy # provider name           model: gpt-4-instance-2 # model name           api_base: "http://localhost:4000" # URL of the litellm proxy server       router:           routing_strategy: least-busy
```

## Caching[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#caching 'Direct link to Caching')

The Router caches the responses from the deployments to improve the response time and reduce the load on the deployments. The cache settings can be configured through the `cache_responses` key in the `router` configuration.

endpoints.yml

```
   router:       routing_strategy: simple-shuffle       cache_responses: true
```

Caching uses in-memory storage by default which should not be used in production. When using the router, caching should be enabled with a persistent storage like Redis.

### Configuration for Redis based routing strategies[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuration-for-redis-based-routing-strategies 'Direct link to Configuration for Redis based routing strategies')

To use the `cost-based-routing` or `usage-based-routing` routing strategies, you need to define the redis connection settings under the `router` key in your `endpoints.yml` file.

endpoints.yml

```
   model_groups:     - id: azure_llm_deployments       models:         - provider: azure           deployment: rasa-gpt-4           api_base: https://azure-deployment/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY}       router:           routing_strategy: cost-based-routing           redis_host: localhost # Can also be set through environment - ${REDIS_HOST}           redis_port: 6379 # Can also be set through environment - ${REDIS_PORT}           redis_password: ${REDIS_PASSWORD}
```

The `redis_host`, `redis_port`, and `redis_password` keys define the connection settings for the redis server. The redis connection can also be configured through `redis_url` key.

endpoints.yml

```
   router:       routing_strategy: cost-based-routing       redis_url: "redis://:mypassword@host:port"
```

### Configuring the Router for multiple model groups with Redis[​](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-multiple-model-groups-with-redis 'Direct link to Configuring the Router for multiple model groups with Redis')

endpoints.yml

```
   model_groups:     - id: azure_gpt4_deployments       models:         - provider: azure           deployment: gpt-4-instance-france           api_base: https://azure-deployment-france/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_FRANCE}         - provider: azure           deployment: gpt-4-instance-canada           api_base: https://azure-deployment-canada/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_CANADA}       router:           routing_strategy: least-busy           redis_host: localhost           redis_port: 6379           redis_password: ${REDIS_PASSWORD}     - id: azure_gpt35_turbo_deployments       models:         - provider: azure           deployment: gpt-35-instance-france           api_base: https://azure-deployment-france/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_FRANCE}         - provider: azure           deployment: gpt-35-instance-canada           api_base: https://azure-deployment-canada/           api_version: "2024-02-15-preview"           api_key: ${MY_AZURE_API_KEY_CANADA}       router:           routing_strategy: simple-shuffle           redis_host: localhost           redis_port: 6379           redis_password: ${REDIS_PASSWORD}
```

- [Overview](https://rasa.com/docs/reference/deployment/multi-llm-routing/#overview)
- [Configuration](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuration)
 - [Additional configuration parameters for the Router](https://rasa.com/docs/reference/deployment/multi-llm-routing/#additional-configuration-parameters-for-the-router)
 - [Configuring the Router for multiple model groups](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-multiple-model-groups)
 - [Configuring the Router for embeddings](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-embeddings)
 - [Configuring the Router for other providers](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-other-providers)
- [Configuring the Router for self-hosted deployments](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-self-hosted-deployments)
 - [vLLM](https://rasa.com/docs/reference/deployment/multi-llm-routing/#vllm)
 - [LLama.cpp](https://rasa.com/docs/reference/deployment/multi-llm-routing/#llamacpp)
 - [Ollama](https://rasa.com/docs/reference/deployment/multi-llm-routing/#ollama)
 - [Connecting via a proxy](https://rasa.com/docs/reference/deployment/multi-llm-routing/#connecting-via-a-proxy)
 - [Connecting via litellm proxy](https://rasa.com/docs/reference/deployment/multi-llm-routing/#connecting-via-litellm-proxy)
- [Caching](https://rasa.com/docs/reference/deployment/multi-llm-routing/#caching)
 - [Configuration for Redis based routing strategies](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuration-for-redis-based-routing-strategies)
 - [Configuring the Router for multiple model groups with Redis](https://rasa.com/docs/reference/deployment/multi-llm-routing/#configuring-the-router-for-multiple-model-groups-with-redis)