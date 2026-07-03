# Source: https://rasa.com/docs/reference/integrations/langfuse

On this page

[Langfuse](https://langfuse.com/) is an open-source observability platform designed specifically for LLM applications. It provides comprehensive tracing, monitoring, and analytics capabilities that help you understand how your LLM-based components are performing in production.

## Installation[​](https://rasa.com/docs/reference/integrations/langfuse/#installation 'Direct link to Installation')

important

Before configuring Langfuse, you must install the `langfuse` package. Langfuse is included as an optional dependency in the `monitoring` extra group.

To install Langfuse, use one of the following methods:

**Using pip:**

```
pip install "rasa-pro[monitoring]"
```

**Using poetry:**

```
poetry install --extras monitoring
```

Without installing the `langfuse` package, you will see an error message indicating that Langfuse is not available, and the integration will not be configured.

## Configuration[​](https://rasa.com/docs/reference/integrations/langfuse/#configuration 'Direct link to Configuration')

Langfuse is configured through the `endpoints.yml` file by adding a `langfuse` entry to the `tracing` section. You can configure multiple tracing backends simultaneously (e.g., both Langfuse and [Jaeger](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector)).

Here is a basic configuration example:

endpoints.yml

```
tracing:  - type: langfuse    host: https://cloud.langfuse.com    public_key: ${LANGFUSE_PUBLIC_KEY}    private_key: ${LANGFUSE_PRIVATE_KEY}
```

### Configuration Options[​](https://rasa.com/docs/reference/integrations/langfuse/#configuration-options 'Direct link to Configuration Options')

| Option | Required | Description | Example |
| --- | --- | --- | --- |
| `type` | Yes | Must be set to `langfuse` | `langfuse` |
| `host` | Yes | The Langfuse server URL | `https://cloud.langfuse.com` |
| `public_key` | Yes | Your Langfuse public API key (must use `${VAR}` syntax for environment variables) | `${LANGFUSE_PUBLIC_KEY}` |
| `private_key` | Yes | Your Langfuse private API key (must use `${VAR}` syntax for environment variables) | `${LANGFUSE_PRIVATE_KEY}` |
| `timeout` | No | Request timeout in seconds | `30` |
| `debug` | No | Enable debug logging | `true` or `false` |
| `environment` | No | Environment label for traces | `production`, `staging`, `development` |
| `release` | No | Release version identifier | `v1.2.3` |
| `media_upload_thread_count` | No | Number of threads for media uploads | `4` |
| `sample_rate` | No | Sampling rate for traces (0.0 to 1.0) | `1.0` |

`public_key` and `private_key` **must** be set as environment variables, and in your `endpoints.yml` file, you reference them using the `${ENV_VAR_NAME}` syntax. For example:

1. **Set the environment variables** in your shell or deployment environment:
 
 ```
    export LANGFUSE_PUBLIC_KEY="<your-public-key>"export LANGFUSE_PRIVATE_KEY="<your-private-key>"
    ```
 
2. **Reference these variables in your `endpoints.yml`:**
 
 endpoints.yml
 
 ```
    tracing:  - type: langfuse    host: https://cloud.langfuse.com    public_key: ${LANGFUSE_PUBLIC_KEY}    private_key: ${LANGFUSE_PRIVATE_KEY}
    ```
 

This ensures your secrets are not stored directly in configuration files but injected at runtime using environment variables.

### Multiple Tracing Backends[​](https://rasa.com/docs/reference/integrations/langfuse/#multiple-tracing-backends 'Direct link to Multiple Tracing Backends')

You can configure both Langfuse and other tracing backends (like [OTLP](https://rasa.com/docs/reference/integrations/tracing/#configuring-a-tracing-backend-or-collector)) simultaneously:

endpoints.yml

```
tracing:  - type: otlp    endpoint: my-otlp-host:4317    insecure: false    service_name: rasa    root_certificates: ./tests/unit/tracing/fixtures/ca.pem  - type: langfuse    host: https://cloud.langfuse.com    public_key: ${LANGFUSE_PUBLIC_KEY}    private_key: ${LANGFUSE_PRIVATE_KEY}
```

## Traced Components[​](https://rasa.com/docs/reference/integrations/langfuse/#traced-components 'Direct link to Traced Components')

When Langfuse is configured, the following components automatically send traces:

### LLM-Based Components[​](https://rasa.com/docs/reference/integrations/langfuse/#llm-based-components 'Direct link to LLM-Based Components')

- **Command Generators**: All LLM-based command generators that generate dialogue commands
- **Contextual Response Rephraser**: Components that rephrase responses using LLMs
- **Enterprise Search Policy**: Policy that uses LLMs to generate responses from search results
- **ReAct Sub Agent**: MCP-based sub agents that use LLMs for reasoning and tool execution
- **LLM-Based Router**: Components that route conversations using LLMs

### Embedding Operations[​](https://rasa.com/docs/reference/integrations/langfuse/#embedding-operations 'Direct link to Embedding Operations')

- **Flow Retrieval**: Semantic search operations when retrieving relevant flows
- **Enterprise Search Policy**: Vector search operations when finding relevant documents

## Trace Contents[​](https://rasa.com/docs/reference/integrations/langfuse/#trace-contents 'Direct link to Trace Contents')

Each trace sent to Langfuse includes the following information:

### Standard Trace Data[​](https://rasa.com/docs/reference/integrations/langfuse/#standard-trace-data 'Direct link to Standard Trace Data')

- **Timestamp**: When the LLM or embedding call was made
- **Input**: The prompt or query sent to the LLM/embedding model
- **Output**: The response or embedding vector returned
- **Latency**: Time taken for the request to complete
- **Token Usage**: Number of prompt tokens, completion tokens, and total tokens used
- **Cost**: Calculated cost based on token usage and model pricing

### Metadata[​](https://rasa.com/docs/reference/integrations/langfuse/#metadata 'Direct link to Metadata')

Each trace includes rich metadata to help you organize and filter traces:

- **Session ID**: The conversation session identifier
- **Tags**: Component name for easy filtering (e.g., `EnterpriseSearchPolicy`, `CompactLLMCommandGenerator`)
- **Custom Metadata**: A dictionary containing:
 - **Component Name**: The class name of the component making the call
 - **Agent ID**: The ID of the agent
 - **Model ID**: The ID of the trained model being used
 - **ReAct Sub Agent Name**: (For ReAct sub agents) The name of the sub-agent

#### Customizing Metadata[​](https://rasa.com/docs/reference/integrations/langfuse/#customizing-metadata 'Direct link to Customizing Metadata')

Custom components can override the `get_llm_tracing_metadata()` method to customize the metadata sent with each trace.

- [Installation](https://rasa.com/docs/reference/integrations/langfuse/#installation)
- [Configuration](https://rasa.com/docs/reference/integrations/langfuse/#configuration)
 - [Configuration Options](https://rasa.com/docs/reference/integrations/langfuse/#configuration-options)
 - [Multiple Tracing Backends](https://rasa.com/docs/reference/integrations/langfuse/#multiple-tracing-backends)
- [Traced Components](https://rasa.com/docs/reference/integrations/langfuse/#traced-components)
 - [LLM-Based Components](https://rasa.com/docs/reference/integrations/langfuse/#llm-based-components)
 - [Embedding Operations](https://rasa.com/docs/reference/integrations/langfuse/#embedding-operations)
- [Trace Contents](https://rasa.com/docs/reference/integrations/langfuse/#trace-contents)
 - [Standard Trace Data](https://rasa.com/docs/reference/integrations/langfuse/#standard-trace-data)
 - [Metadata](https://rasa.com/docs/reference/integrations/langfuse/#metadata)