# Source: https://rasa.com/docs/pro/improve/observability-metrics

On this page

Metrics are runtime measurements that capture indicators of a service’s availability and performance. Unlike tracing—which helps you understand the sequence of operations for a single request—metrics provide an aggregated statistical view across multiple requests or conversations. Typical examples include **average response time**, **throughput**, and **CPU/memory consumption**. Monitoring these helps you:

- Track the health of your service.
- Quickly detect and alert on outages or anomalies.
- Quantify the impact of code or infrastructure changes on performance.

When combined with tracing, metrics give you a complete view of your deployment behavior, making it easier to debug issues and optimize resource usage.

## How To Use Metrics[​](https://rasa.com/docs/pro/improve/observability-metrics/#how-to-use-metrics 'Direct link to How To Use Metrics')

### Enabling Metrics in Rasa[​](https://rasa.com/docs/pro/improve/observability-metrics/#enabling-metrics-in-rasa 'Direct link to Enabling Metrics in Rasa')

Rasa uses an [OpenTelemetry (OTEL) Collector](https://opentelemetry.io/docs/collector/) to collect metrics and send them to your desired backend (e.g., Prometheus, Datadog, etc.).

1. **Configure OTEL in your endpoints file (or Helm values):**
 
 endpoints.yml
 
 ```
    metrics:  type: otlp  endpoint: my-otlp-host:4318  insecure: false  service_name: rasa  root_certificates: ./tests/unit/tracing/fixtures/ca.pem
    ```
 
 - `type: otlp` indicates you are using OpenTelemetry’s OTLP format.
 - `endpoint` is the URL of the OTEL Collector or metrics backend.
 - `service_name` is an identifier for your Rasa Pro service.
 - `insecure`/`root_certificates` specify how TLS is handled.
2. **Use Tracing for a Complete View (Recommended):**
 
 Metrics become even more powerful when paired with Tracing because tracing surfaces the sequence of internal method calls, while metrics aggregate their performance.
 

### Custom Metrics Collected by Rasa[​](https://rasa.com/docs/pro/improve/observability-metrics/#custom-metrics-collected-by-rasa 'Direct link to Custom Metrics Collected by Rasa')

Once configured, Rasa automatically collects several custom metrics relevant to large language model (LLM) usage and overall assistant performance:

- **CPU and Memory Usage** of any LLM-based command generator (e.g., `CompactLLMCommandGenerator`, `SearchReadyLLMCommandGenerator`) at the time of making an LLM call.
- **Prompt Token Usage** for LLM-based command generators, provided the `trace_prompt_tokens` config property is enabled.
- **Method Call Durations** for LLM-specific components, such as:
 - `EnterpriseSearchPolicy`
 - `ContextualResponseRephraser`
 - `CompactLLMCommandGenerator`
 - `SearchReadyLLMCommandGenerator`
- **HTTP Request Metrics** for the Rasa client:
 - Duration of requests to external services (action server, NLG server, etc.).
 - Request size in bytes.

### Sub-agents (ReAct and A2A)[​](https://rasa.com/docs/pro/improve/observability-metrics/#sub-agents-react-and-a2a 'Direct link to Sub-agents (ReAct and A2A)')

When a flow uses [autonomous steps](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps) to hand off to a [ReAct](https://rasa.com/docs/reference/config/agents/react-sub-agents/) or [A2A](https://rasa.com/docs/reference/config/agents/external-sub-agents/) sub-agent, Rasa emits extra OpenTelemetry **histograms** (in addition to the spans described in [Tracing](https://rasa.com/docs/pro/improve/tracing/)):

- **`agent_execution_duration`** — Wall time for each sub-agent run (`_call_agent_with_retry` in the agent executor). Useful for end-to-end latency and error rates per sub-agent. Attribute labels include:
 
 - **`agent_name`** — Sub-agent id from configuration.
 - **`protocol_type`** — How the sub-agent is connected: `mcp_open` or `mcp_task` for ReAct-style MCP sub-agents, or `a2a` for A2A sub-agents.
 - **`status`** — Final status of the sub-agent result.
- **`mcp_tool_execution_duration`** — Time spent inside an MCP tool call. The **same metric name** is used in two execution paths; use the **`execution_context`** label to distinguish them:
 
 - **`execution_context` = `flow`** — Tool invoked from a flow [MCP tool step](https://rasa.com/docs/reference/primitives/flow-steps/#calling-an-mcp-tool). Labels include **`tool_id`**, **`mcp_server`**, and **`success`**.
 - **`execution_context` = `agent`** — Tool invoked while a **ReAct MCP sub-agent** runs (`_execute_tool_call`). Labels include **`tool_name`**, **`agent_name`**, **`protocol_type`**, and **`success`**.
- **ReAct MCP sub-agent LLM usage** — For MCP-based sub-agents, each LLM `send_message` round-trip also records resource-style histograms (CPU and memory sampled like other LLM components, plus estimated prompt size and response duration):
 
 - `mcp_agent_llm_cpu_usage`
 - `mcp_agent_llm_memory_usage`
 - `mcp_agent_llm_prompt_token_usage`
 - `mcp_agent_llm_response_duration`

By collecting these telemetry metrics, you gain robust insights into how your assistant performs under real-world usage. You can proactively detect issues, understand resource consumption, and tailor your assistant’s architecture to provide the best possible experience for your users.

- [How To Use Metrics](https://rasa.com/docs/pro/improve/observability-metrics/#how-to-use-metrics)
 - [Enabling Metrics in Rasa](https://rasa.com/docs/pro/improve/observability-metrics/#enabling-metrics-in-rasa)
 - [Custom Metrics Collected by Rasa](https://rasa.com/docs/pro/improve/observability-metrics/#custom-metrics-collected-by-rasa)
 - [Sub-agents (ReAct and A2A)](https://rasa.com/docs/pro/improve/observability-metrics/#sub-agents-react-and-a2a)