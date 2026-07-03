# Source: https://rasa.com/docs/reference/integrations/mcp-servers

On this page

New Beta Feature in 3.14

Rasa supports native integration of MCP servers.

This feature is in a beta (experimental) stage and may change in future Rasa versions. We welcome your feedback on this feature.

MCP servers allow your Rasa agent to connect to external services and APIs through the [Model Context Protocol](https://modelcontextprotocol.io/). These servers expose tools that your agent can use [directly in flows](https://rasa.com/docs/reference/primitives/flow-steps/#calling-an-mcp-tool) or provide to [autonomous sub agents](https://rasa.com/docs/reference/config/agents/overview-agents/#how-to-use-sub-agents) for dynamic decision-making.

## Basic Configuration[​](https://rasa.com/docs/reference/integrations/mcp-servers/#basic-configuration 'Direct link to Basic Configuration')

MCP servers are configured in your `endpoints.yml` file.

```
mcp_servers:  - name: trade_server    url: http://localhost:8080    type: http
```

The following fields are required:

- **`name`**: A unique identifier for the MCP server
- **`url`**: The URL where the MCP server is running
- **`type`**: The server type (currently supports `http` and `https`)

## Multiple MCP Servers[​](https://rasa.com/docs/reference/integrations/mcp-servers/#multiple-mcp-servers 'Direct link to Multiple MCP Servers')

You can configure multiple MCP servers to connect to different services:

```
mcp_servers:  - name: inventory_server    url: http://localhost:8080    type: http  - name: payment_server    url: https://api.payment-service.com    type: https  - name: customer_database    url: http://localhost:9000    type: http
```

Each server can expose different tools and be used independently in your flows or by different sub agents. Server names must be unique across all MCP servers. If you attempt to configure multiple servers with the same name, Rasa will raise a validation error during startup.

## Authentication[​](https://rasa.com/docs/reference/integrations/mcp-servers/#authentication 'Direct link to Authentication')

MCP servers support multiple authentication methods for connecting to external services:

- **API Key**: Static key attached as `Authorization: Bearer <token>`
- **OAuth 2.0 (Client Credentials)**: Automatic token retrieval with client ID/secret
- **Pre-issued Token**: Direct token usage until expiry

Authentication settings are specified using additional parameters in your MCP server configuration.

- API Key
- API Key (Custom Header)
- OAuth 2.0
- Pre-issued Token

```
mcp_servers:  - name: secure_api_server    url: https://api.example.com    type: https    api_key: "${API_KEY}"
```

```
mcp_servers:  - name: custom_header_server    url: https://api.example.com    type: https    api_key: "${API_KEY}"    header_name: "X-API-Key"    header_format: "{key}"
```

```
mcp_servers:  - name: oauth_server    url: https://api.example.com    type: https    oauth:      client_id: "${CLIENT_ID}"      client_secret: "${CLIENT_SECRET}"      token_url: "https://auth.example.com/oauth/token"      scope: "read:data write:data"
```

```
mcp_servers:  - name: token_server    url: https://api.example.com    type: https    token: "${ACCESS_TOKEN}"
```

The `$` syntax is **required** for the following sensitive parameters:

- `api_key`
- `token`
- `client_secret`

This ensures that they are not stored in plain text in your configuration files. For other parameters like `client_id`, using the `$` syntax is optional — you can either reference an environment variable using the `$` syntax or provide the value directly in the configuration.

## Advanced Configuration[​](https://rasa.com/docs/reference/integrations/mcp-servers/#advanced-configuration 'Direct link to Advanced Configuration')

### Custom Headers[​](https://rasa.com/docs/reference/integrations/mcp-servers/#custom-headers 'Direct link to Custom Headers')

You can specify custom headers for API key authentication:

```
mcp_servers:  - name: custom_auth_server    url: https://api.example.com    type: https    api_key: "${API_KEY}"    header_name: "X-Custom-Auth"    header_format: "Bearer {key}"
```

### OAuth 2.0 Scopes[​](https://rasa.com/docs/reference/integrations/mcp-servers/#oauth-20-scopes 'Direct link to OAuth 2.0 Scopes')

For OAuth 2.0 authentication, you can optionally pass in scope, audience, and timeout:

```
mcp_servers:  - name: oauth_server_with_scopes    url: https://api.example.com    type: https    oauth:      client_id: "${CLIENT_ID}"      client_secret: "${CLIENT_SECRET}"      token_url: "https://auth.example.com/oauth/token"      scope: "read:users write:orders admin:settings" # Optional: scopes for the OAuth 2.0 token      audience: "https://api.example.com" # Optional: audience for the OAuth 2.0 token      timeout: 10 # Optional: timeout for the OAuth 2.0 token
```

## Complete Example[​](https://rasa.com/docs/reference/integrations/mcp-servers/#complete-example 'Direct link to Complete Example')

Here's a comprehensive example showing multiple MCP servers with different authentication methods:

```
mcp_servers:  # Public API with API key  - name: weather_service    url: https://api.weather.com    type: https    api_key: "${WEATHER_API_KEY}"    # Internal service with OAuth  - name: internal_database    url: https://db.internal.com    type: https    oauth:      client_id: "${DB_CLIENT_ID}"      client_secret: "${DB_CLIENT_SECRET}"      token_url: "https://auth.internal.com/oauth/token"      scope: "database:read database:write"    # Local development server  - name: local_tools    url: http://localhost:8080    type: http    # Service with custom header authentication  - name: custom_auth_server    url: https://api.example.com    type: https    api_key: "${API_KEY}"    header_name: "X-API-Key"    header_format: "{key}"
```

## Passing metadata to tools (`meta_map`)[​](https://rasa.com/docs/reference/integrations/mcp-servers/#passing-metadata-to-tools-meta_map 'Direct link to passing-metadata-to-tools-meta_map')

Remote MCP tool calls can include a protocol-level `_meta` object (when supported by your MCP client SDK) so the server receives context that **does not** appear in the tool arguments the LLM fills. In Rasa, this is configured per server with optional `meta_map` in `endpoints.yml`:

- **`static`**: Fixed string key-value pairs always merged into `_meta` (for example API version or environment labels).
- **`from_slots`**: A list of `{ slot, param }` entries. For each entry, the current value of the named slot (from the tracker / agent input) is sent as `_meta[param]`.

```
mcp_servers:  - name: internal_api    url: http://internal-api:8000/mcp/    meta_map:      from_slots:        - slot: user_id          param: user_id        - slot: role          param: user_role      static:        api_version: "v2"        source: "rasa_agent"
```

If `meta_map` is omitted or has no `from_slots` entries, Rasa still sends any `static` entries. Slot values are read from the same agent input used for the ReAct turn; ensure the slot is set before the tool runs if you map it in `from_slots`.

Older MCP Python SDK releases that do not support a `meta` parameter on `call_tool` are still supported: Rasa detects this and falls back to calling without metadata so tools keep working; once the SDK supports `meta`, metadata is sent automatically.

For tools implemented in Python inside your ReAct sub-agent (not via an MCP server), use [custom tool executors](https://rasa.com/docs/reference/config/agents/react-sub-agents/#adding-custom-tools) and `AgentToolContext.metadata` instead of `meta_map`.

### Reading metadata on the MCP server[​](https://rasa.com/docs/reference/integrations/mcp-servers/#reading-metadata-on-the-mcp-server 'Direct link to Reading metadata on the MCP server')

On the wire, Rasa sends the resolved `meta_map` object as JSON-RPC `params._meta` on each `tools/call` request. Keys are exactly the `param` names defined in from `from_slots` and the those specified under `static` (for example `user_id`, `user_role`, `api_version`, `source` in the sample above).

With the [official MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk), that payload is available as `CallToolRequest.params.meta` (the field is serialized as `_meta`). If you build the server with **FastMCP**, inject `Context` and read the same values from the request metadata object (unknown keys are allowed on `meta` alongside standard fields like `progressToken`):

```
def get_request_meta(ctx: Context | None) -> dict:    """Request _meta as a dict. Safe to call with None (returns {})."""    if ctx is None:        return {}    meta = getattr(ctx.request_context, "meta", None)    if meta is None:        return {}    return meta.model_dump(exclude_none=True)@mcp.tool()def sum(a: int, b: int, context: Context | None = None) -> str:    meta_dict = get_request_meta(context)    api_version = meta_dict.get("api_version", "v1")    ...
```

If you use the low-level `Server` API and a custom `tools/call` handler, read the same fields from the incoming `CallToolRequest` (for example `req.params.meta`) before invoking your tool implementation.

**Privacy and observability:** Rasa does not attach this metadata to LLM tracing fields. At debug log level, only the **keys** of the resolved metadata are logged, not values.

## Validation[​](https://rasa.com/docs/reference/integrations/mcp-servers/#validation 'Direct link to Validation')

Rasa validates MCP server configurations to ensure:

- Server type is either `http` or `https`
- Name and URL are not empty
- Sensitive parameters (API keys, tokens, secrets) are properly referenced using environment variables
- OAuth configuration includes all required fields
- If `meta_map.from_slots` is set, every referenced `slot` exists in the domain (training / `rasa train` validation)

If validation fails, Rasa will provide specific error messages to help you fix the configuration.

- [Basic Configuration](https://rasa.com/docs/reference/integrations/mcp-servers/#basic-configuration)
- [Multiple MCP Servers](https://rasa.com/docs/reference/integrations/mcp-servers/#multiple-mcp-servers)
- [Authentication](https://rasa.com/docs/reference/integrations/mcp-servers/#authentication)
- [Advanced Configuration](https://rasa.com/docs/reference/integrations/mcp-servers/#advanced-configuration)
 - [Custom Headers](https://rasa.com/docs/reference/integrations/mcp-servers/#custom-headers)
 - [OAuth 2.0 Scopes](https://rasa.com/docs/reference/integrations/mcp-servers/#oauth-20-scopes)
- [Complete Example](https://rasa.com/docs/reference/integrations/mcp-servers/#complete-example)
- [Passing metadata to tools (`meta_map`)](https://rasa.com/docs/reference/integrations/mcp-servers/#passing-metadata-to-tools-meta_map)
 - [Reading metadata on the MCP server](https://rasa.com/docs/reference/integrations/mcp-servers/#reading-metadata-on-the-mcp-server)
- [Validation](https://rasa.com/docs/reference/integrations/mcp-servers/#validation)