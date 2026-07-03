# Source: https://rasa.com/docs/learn/quickstart/pro

On this page

Build your first Rasa agent in minutes with the shortest path.

## 1) Get a free license[​](https://rasa.com/docs/learn/quickstart/pro/#1-get-a-free-license 'Direct link to 1) Get a free license')

[**Developer Edition**\\ \\ The Rasa Developer Edition is free to use for building and running agents for up to 1000 conversations/month.](https://rasa.com/rasa-pro-developer-edition-license-key-request/)

 [Get your License](https://rasa.com/rasa-pro-developer-edition-license-key-request/)

## 2) Create a project and install Rasa Pro[​](https://rasa.com/docs/learn/quickstart/pro/#2-create-a-project-and-install-rasa-pro 'Direct link to 2) Create a project and install Rasa Pro')

In your terminal:

```
mkdir rasa-agentcd rasa-agentuv venv --python 3.11source .venv/bin/activateuv pip install rasa-prorasa init --template=basic
```

Need to install **uv**? [Get it here](https://docs.astral.sh/uv/getting-started/installation/).

Set your license:

```
export RASA_LICENSE=YOUR_LICENSE_KEY
```

This template uses OpenAI as the default LLM provider. To use a different provider, define a [model group](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups) in `endpoints.yml` and reference it via `model_group` in `config.yml`.

Set your API key as an environment variable:

```
export OPENAI_API_KEY=YOUR_API_KEY
```

## 3) Connect Rasa MCP Tools[​](https://rasa.com/docs/learn/quickstart/pro/#3-connect-rasa-mcp-tools 'Direct link to 3) Connect Rasa MCP Tools')

Rasa MCP Tools are a set of tools, skills and docs that enable you to interact with your Rasa agent using natural language. **They are included in Rasa Pro 3.16 and later.**

v3.16

Run the setup wizard from your project root:

```
rasa tools init
```

The wizard creates `.rasa/tools.yaml` and downloads offline docs and skills for your project. It also generates MCP configuration for your IDE.

Important

Run your IDE from your **project root** so Rasa Tools can read `.rasa/tools.yaml`.

### Verify your setup[​](https://rasa.com/docs/learn/quickstart/pro/#verify-your-setup 'Direct link to Verify your setup')

Open your IDE and verify that Rasa Tools appear in your MCP settings. If the wizard configured your client automatically, you should see `rasa-tools` listed and enabled. If it's not, make sure to switch it on or run it from the command line.

```
rasa tools run
```

For manual configuration or more advanced setup details for specific clients (Cursor, VS Code, Claude Code, JetBrains), read more about [**Rasa MCP Tools here**](https://rasa.com/docs/pro/installation/rasa-mcp-tools/).

## 4) Build your first agent[​](https://rasa.com/docs/learn/quickstart/pro/#4-build-your-first-agent 'Direct link to 4) Build your first agent')

Open a chat with your IDE copilot and send this prompt to get started:

```
Use Rasa MCP tools to inspect this project and briefly summarize what it does in a few sentences. Then propose 3 **fun, concrete verticals**—think **book recommendation**, **flight booking**, or **concierge**—not “more of the same template.” For each: one-line pitch, 2 user stories, what you’d add (flows/slots/APIs/sub-agents/MCP servers), effort S/M/L.
```

Pick the vertical you like best — or suggest your own.

- [1) Get a free license](https://rasa.com/docs/learn/quickstart/pro/#1-get-a-free-license)
- [2) Create a project and install Rasa Pro](https://rasa.com/docs/learn/quickstart/pro/#2-create-a-project-and-install-rasa-pro)
- [3) Connect Rasa MCP Tools](https://rasa.com/docs/learn/quickstart/pro/#3-connect-rasa-mcp-tools)
 - [Verify your setup](https://rasa.com/docs/learn/quickstart/pro/#verify-your-setup)
- [4) Build your first agent](https://rasa.com/docs/learn/quickstart/pro/#4-build-your-first-agent)