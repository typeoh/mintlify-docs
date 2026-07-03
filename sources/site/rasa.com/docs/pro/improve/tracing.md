# Source: https://rasa.com/docs/pro/improve/tracing

On this page

Tracing is a key element of observability, providing visibility into how requests flow through your assistant’s distributed components (e.g. Rasa runtime, custom actions server, external services).

Observability is the practice of instrumenting systems so you can monitor, troubleshoot, and optimize them in production. In a conversational assistant, observability typically consists of:

- **Metrics**: Quantitative measurements (e.g. average response time)
- **Logs**: Event logs emitted by various components
- **Tracing**: Detailed timelines of requests and their paths across distributed services

**Tracing** stands out by focusing on the life cycle of each individual request. When a user sends a message, the request might pass through your assistant’s LLM-based command generator, custom action server, flow logic, and external APIs. Tracing captures these hops along with relevant metadata—giving you a precise picture of where bottlenecks or unexpected behaviors may be occurring.

## Why Do You Need Tracing in Your Assistant[​](https://rasa.com/docs/pro/improve/tracing/#why-do-you-need-tracing-in-your-assistant 'Direct link to Why Do You Need Tracing in Your Assistant')

When building pro-code conversational AI solutions, teams need to quickly answer questions like:

- **What exactly happened when the assistant processed a user message?**
 
 See which flows were invoked, which custom actions ran, which commands got generated, and why.
 
- **Why is my assistant slow or occasionally unresponsive?**
 
 Is the slowness coming from an LLM call, a vector store query, or a custom action HTTP request?
 
- **How can I debug or optimize custom action performance?**
 
 Pinpoint exactly where your code is spending time—e.g. upstream or downstream dependencies in your custom actions.
 
- **How can I track LLM input and outputs, its usage and costs?**
 
 Trace the LLM’s prompt token usage, temperature settings, etc. to monitor usage in real time.
 

By having trace data in one place, you can correlate user behavior, system logs, and LLM calls to rapidly diagnose and fix issues. In production, distributed tracing is often the fastest path to root-cause analysis and performance tuning.

## How to Enable Tracing in Rasa[​](https://rasa.com/docs/pro/improve/tracing/#how-to-enable-tracing-in-rasa 'Direct link to How to Enable Tracing in Rasa')

Enabling tracing in Rasa is straightforward—simply connect it to a supported tracing backend or collector. Once configured, Rasa emits trace spans for conversation processing, LLM calls, flow transitions, custom actions, and more.

### 1\. Choose Your Tracing Backend or Collector[​](https://rasa.com/docs/pro/improve/tracing/#1-choose-your-tracing-backend-or-collector 'Direct link to 1. Choose Your Tracing Backend or Collector')

Rasa supports:

- **Jaeger** A popular open source end-to-end distributed tracing system.
- **OTEL Collector (OpenTelemetry Collector)** A vendor-agnostic approach that can forward data to Jaeger, Zipkin, or other tracing backends.

In addition, Rasa also supports a direct integration to an LLM focussed tracing and observability tool - **[Langfuse](https://langfuse.com/)**. Traces in Langfuse provide an in-depth analysis of each LLM call made during each turn of a conversation allowing for quicker accuracy, speed and cost optimizations of LLM calls.

### 2\. Configure `tracing` in Your Endpoints[​](https://rasa.com/docs/pro/improve/tracing/#2-configure-tracing-in-your-endpoints 'Direct link to 2-configure-tracing-in-your-endpoints')

In your `endpoints.yml` or Helm values, add a `tracing:` block. For example, to configure Jaeger and Langfuse as tracing tools:

endpoints.yml

```
tracing:  - type: jaeger    host: localhost    port: 6831    service_name: rasa    sync_export: ~  - type: langfuse    host: https://cloud.langfuse.com # can be localhost if self-hosted    public_key: ${LANGFUSE_PUBLIC_KEY}    private_key: ${LANGFUSE_PRIVATE_KEY}
```

Or to configure an OTEL collector and langfuse:

endpoints.yml

```
tracing:  - type: otlp    endpoint: my-otlp-host:4318    insecure: false    service_name: rasa    root_certificates: ./path/to/ca.pem  - type: langfuse    host: https://cloud.langfuse.com # can be localhost if self-hosted    public_key: ${LANGFUSE_PUBLIC_KEY}    private_key: ${LANGFUSE_PRIVATE_KEY}
```

Once you’ve done this, tracing is automatically enabled for both the runtime, the custom action server and each of the LLM calls made in between. No additional code is needed to start collecting standard spans.

For more detailed information on configuration options, head over to the reference section of [Jaegar/OLTP collectors](https://rasa.com/docs/reference/integrations/tracing/) and [langfuse integration](https://rasa.com/docs/reference/integrations/langfuse/)

### 3\. (Optional) Instrument Custom Code[​](https://rasa.com/docs/pro/improve/tracing/#3-optional-instrument-custom-code 'Direct link to 3. (Optional) Instrument Custom Code')

If you want deeper insights into custom action performance, you can add custom spans to specific parts of your code. For example, you can retrieve the tracer in your action server and wrap code sections in manual spans. This is especially useful for investigating complex logic or third-party dependencies.

info

For a full list of traced events and code snippets for custom instrumentation, see our [reference documentation](https://rasa.com/docs/reference/integrations/tracing/).

## Best Practices for Tracing[​](https://rasa.com/docs/pro/improve/tracing/#best-practices-for-tracing 'Direct link to Best Practices for Tracing')

1. **Start Tracing Early**
 
 Instrument your assistant from the beginning of development—waiting until there’s a performance issue might make it harder to diagnose.
 
2. **Trace Only Where Needed**
 
 While you can trace everything, capturing very verbose details (like prompt token usage) in production can add overhead. Often, it’s better to enable advanced tracing features in test or staging environments, then turn them off once you identify the root cause.
 
3. **Use a Single Runtime Trace Collector**
 
 Sending data to a single OTEL or Jaeger collector is simpler to maintain and ensures all trace spans appear in one place—important for diagnosing end-to-end issues.
 
4. **Correlate with Logs & Metrics**
 
 Traces alone might not be sufficient. Combine them with logs (e.g. error messages) and metrics (e.g. average token usage per conversation) to get a 360° view of system health.
 
5. **Leverage the Dialogue Stack**
 
 CALM’s dialogue stack concept means multiple flows can be active at once. Tracing helps you see which flow is top-of-stack at any given time—and why that flow was triggered.
 
6. **Trace LLM Calls Separately**
 
 Rasa uses multiple LLM based components that are all critical to fulfilling a single turn of a conversation. A lot of low effort high value runtime optimizations are found by deeply analyzing input and output of each LLM call, token usage to monitor costs, latencies to manage responsiveness of the agent. Ensure you have a langfuse instance connected and configured to your rasa agent to catch issues early on in production.
 
7. **Focus on Action Server Performance**
 
 Custom actions are often the biggest source of latencies. Use tracing spans around external API calls in your action code to detect slow dependencies.
 

Once configured, tracing gives you a powerful lens into how your assistant orchestrates the user conversation, dialogues, LLM calls, and custom actions. It is an essential tool for ensuring your assistant runs reliably in production, and for enabling fast, data-driven debugging when issues arise.

- [Why Do You Need Tracing in Your Assistant](https://rasa.com/docs/pro/improve/tracing/#why-do-you-need-tracing-in-your-assistant)
- [How to Enable Tracing in Rasa](https://rasa.com/docs/pro/improve/tracing/#how-to-enable-tracing-in-rasa)
 - [1\. Choose Your Tracing Backend or Collector](https://rasa.com/docs/pro/improve/tracing/#1-choose-your-tracing-backend-or-collector)
 - [2\. Configure `tracing` in Your Endpoints](https://rasa.com/docs/pro/improve/tracing/#2-configure-tracing-in-your-endpoints)
 - [3\. (Optional) Instrument Custom Code](https://rasa.com/docs/pro/improve/tracing/#3-optional-instrument-custom-code)
- [Best Practices for Tracing](https://rasa.com/docs/pro/improve/tracing/#best-practices-for-tracing)