# Source: https://rasa.com/docs/pro/deploy/llm-routing

On this page

When building assistants that use Large Language Models (LLMs), it’s often important to distribute or “route” requests across multiple deployments or even multiple providers. For instance, you might deploy the same model in multiple regions to reduce latency, or switch among providers to balance usage and cost. Rasa provides a **multi-LLM router** that does this work for you behind the scenes.

The router can also handle embedding models in the same way. In this guide, you’ll learn how to:

- Define multiple model groups in your `endpoints.yml`
- Select a routing strategy (e.g., `simple-shuffle`, `least-busy`)
- Configure caching or store usage data in Redis
- Use the router for LLM completions and for embeddings

Note

The Multi-LLM router feature requires Rasa Pro 3.11.0 and above.

## Why Use Multi-LLM Routing?[​](https://rasa.com/docs/pro/deploy/llm-routing/#why-use-multi-llm-routing 'Direct link to Why Use Multi-LLM Routing?')

Some scenarios where you might want to configure multiple LLM providers or deployments:

- **Load balancing**: Distribute requests across multiple deployments to avoid hitting rate limits or usage caps on a single LLM deployment.
- **Latency optimization**: Direct user requests to the nearest or least busy region to reduce response times.
- **Failover**: If one deployment goes down or fails to respond, seamlessly switch to the next available deployment.
- **Cost optimization**: Route requests based on token usage or cost per token to stay within budget.

## Basic Configuration[​](https://rasa.com/docs/pro/deploy/llm-routing/#basic-configuration 'Direct link to Basic Configuration')

All multi-LLM routing happens in your `endpoints.yml` under **model groups**. A model group defines one or more LLM (or embedding) deployments that share the same underlying model. You then specify a `router` block with your chosen routing strategy and any additional parameters.

### Minimal Example[​](https://rasa.com/docs/pro/deploy/llm-routing/#minimal-example 'Direct link to Minimal Example')

endpoints.yml

```
model_groups:  - id: azure_llm_deployments    models:      - provider: azure        deployment: rasa-gpt-4        api_base: https://azure-deployment/        api_version: "2024-02-15-preview"        api_key: ${MY_AZURE_API_KEY}    router:      routing_strategy: simple-shuffle
```

1. **`id`:** A unique identifier for this group.
2. **`models`:** One or more LLM deployments to include in the group.
3. **`routing_strategy`:** How requests are distributed across the models.

Recommendation

Add multiple identical deployments (e.g., different regions or hosts) to ensure consistent outputs. Mixing fundamentally different models (e.g., a compact mini model vs a larger frontier model) in the same group may lead to unpredictable behavior.

## Using Multiple Deployments[​](https://rasa.com/docs/pro/deploy/llm-routing/#using-multiple-deployments 'Direct link to Using Multiple Deployments')

Below, we define **two** deployments in the same model group and apply a shuffle routing strategy:

endpoints.yml

```
model_groups:  - id: azure_llm_deployments    models:      - provider: azure        deployment: gpt-4-instance-france        api_base: https://azure-deployment-france/        api_version: "2024-02-15-preview"        api_key: ${MY_AZURE_API_KEY_FRANCE}      - provider: azure        deployment: gpt-4-instance-canada        api_base: https://azure-deployment-canada/        api_version: "2024-02-15-preview"        api_key: ${MY_AZURE_API_KEY_CANADA}    router:      routing_strategy: simple-shuffle
```

### Available Routing Strategies[​](https://rasa.com/docs/pro/deploy/llm-routing/#available-routing-strategies 'Direct link to Available Routing Strategies')

- **`simple-shuffle`**: Distributes requests by randomly shuffling across deployments.
- **`least-busy`**: Routes to whichever deployment currently has the fewest ongoing requests.
- **`latency-based-routing`**: Routes to the deployment with the lowest measured response time.
- **`cost-based-routing`**: Requires Redis. Routes to whichever deployment so far has the lowest total cost.
- **`usage-based-routing`**: Requires Redis. Routes to the deployment with the lowest total token usage over time.

To learn more or configure advanced strategies, see the **Reference → LLM Configuration** docs.

## Multiple Model Groups[​](https://rasa.com/docs/pro/deploy/llm-routing/#multiple-model-groups 'Direct link to Multiple Model Groups')

You can define **separate** model groups for different base models or usage patterns. Each group is configured independently, allowing you to route requests for different model sizes or deployments separately, for example.

endpoints.yml

```
model_groups:  - id: azure_gpt4_deployments    models:      - provider: azure        deployment: gpt-4-instance-france        ...      - provider: azure        deployment: gpt-4-instance-canada        ...    router:      routing_strategy: least-busy  - id: azure_gpt35_turbo_deployments    models:      - provider: azure        deployment: gpt-35-instance-france        ...      - provider: azure        deployment: gpt-35-instance-canada        ...    router:      routing_strategy: simple-shuffle
```

## Customizing the Router[​](https://rasa.com/docs/pro/deploy/llm-routing/#customizing-the-router 'Direct link to Customizing the Router')

There are multiple ways to customize router behavior. For instance:

- **`cooldown_time`**: Time in seconds after a deployment fails before the router retries using that deployment.
- **`allowed_fails`**: How many failures are allowed before a deployment is considered unavailable.
- **`num_retries`**: How many times to retry a failed request.

endpoints.yml

```
model_groups:  - id: azure_llm_deployments    models:      - provider: azure        deployment: rasa-gpt-4        ...    router:      routing_strategy: simple-shuffle      cooldown_time: 10      allowed_fails: 2      num_retries: 3
```

For the full list of configurable parameters, see the **Reference → LLM Configuration** docs.

## Embeddings Routing[​](https://rasa.com/docs/pro/deploy/llm-routing/#embeddings-routing 'Direct link to Embeddings Routing')

You can use **the same** multi-LLM router approach for embedding models. For example:

endpoints.yml

```
model_groups:  - id: azure_embeddings_deployments    models:      - provider: azure        deployment: text-embeddings-instance-france        ...      - provider: azure        deployment: text-embeddings-instance-canada        ...    router:      routing_strategy: simple-shuffle
```

This is helpful if you maintain multiple region-specific embedding deployments or want to route embeddings based on usage and cost.

## Self-Hosted Models and Other Providers[​](https://rasa.com/docs/pro/deploy/llm-routing/#self-hosted-models-and-other-providers 'Direct link to Self-Hosted Models and Other Providers')

The router supports many types of model deployments, including self-hosted LLM endpoints (e.g., vLLM, Llama.cpp, Ollama). Configuration follows the same pattern:

endpoints.yml

```
model_groups:  - id: vllm_deployments    models:      - provider: self-hosted        model: meta-llama/Meta-Llama-3-8B        api_base: "http://localhost:8000/v1"      - provider: self-hosted        model: meta-llama/Meta-Llama-3-8B        api_base: "http://localhost:8001/v1"    router:      routing_strategy: least-busy      # If not using chat-completions endpoint:      use_chat_completions_endpoint: false
```

Note

If you need a proxy or custom connectivity (e.g., via LiteLLM proxy), you can place the proxy URL under `api_base` for each model.

---

## Caching[​](https://rasa.com/docs/pro/deploy/llm-routing/#caching 'Direct link to Caching')

By default, the router can cache responses to reduce load and improve performance. To enable caching in production, you must configure a **persistent store** like Redis. In-memory caching is possible, but not recommended in production since it won’t persist across restarts.

endpoints.yml

```
router:  routing_strategy: simple-shuffle  cache_responses: true
```

## Redis Setup for Advanced Routing[​](https://rasa.com/docs/pro/deploy/llm-routing/#redis-setup-for-advanced-routing 'Direct link to Redis Setup for Advanced Routing')

If you want to use **cost-based** or **usage-based** routing, configure Redis in your `endpoints.yml`:

endpoints.yml

```
model_groups:  - id: azure_llm_deployments    models:      - provider: azure        deployment: rasa-gpt-4        ...    router:      routing_strategy: cost-based-routing      redis_host: localhost      redis_port: 6379      redis_password: ${REDIS_PASSWORD}
```

Alternatively, specify a `redis_url` directly:

endpoints.yml

```
router:  routing_strategy: cost-based-routing  redis_url: "redis://:mypassword@host:port"
```

**That’s it!** You have now set up **multi-LLM routing** in your Rasa (CALM) assistant. If you need more details on advanced configurations or want to dig deeper into how each parameter works under the hood, head over to the [Reference](https://rasa.com/docs/reference/deployment/multi-llm-routing/).

- [Why Use Multi-LLM Routing?](https://rasa.com/docs/pro/deploy/llm-routing/#why-use-multi-llm-routing)
- [Basic Configuration](https://rasa.com/docs/pro/deploy/llm-routing/#basic-configuration)
 - [Minimal Example](https://rasa.com/docs/pro/deploy/llm-routing/#minimal-example)
- [Using Multiple Deployments](https://rasa.com/docs/pro/deploy/llm-routing/#using-multiple-deployments)
 - [Available Routing Strategies](https://rasa.com/docs/pro/deploy/llm-routing/#available-routing-strategies)
- [Multiple Model Groups](https://rasa.com/docs/pro/deploy/llm-routing/#multiple-model-groups)
- [Customizing the Router](https://rasa.com/docs/pro/deploy/llm-routing/#customizing-the-router)
- [Embeddings Routing](https://rasa.com/docs/pro/deploy/llm-routing/#embeddings-routing)
- [Self-Hosted Models and Other Providers](https://rasa.com/docs/pro/deploy/llm-routing/#self-hosted-models-and-other-providers)
- [Caching](https://rasa.com/docs/pro/deploy/llm-routing/#caching)
- [Redis Setup for Advanced Routing](https://rasa.com/docs/pro/deploy/llm-routing/#redis-setup-for-advanced-routing)