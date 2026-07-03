# Source: https://rasa.com/docs/pro/installation/rasa-mcp-tools

On this page

**Rasa Tools are included in Rasa Pro 3.16 and later.** There is no separate package to install.

v3.16

## What are Rasa MCP Tools?[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#what-are-rasa-mcp-tools 'Direct link to What are Rasa MCP Tools?')

Rasa MCP Tools are [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) tools that let agentic systems — IDE copilots, browser copilots, and other MCP clients — **inspect, validate, train, and debug** Rasa agents using natural language.

Building Rasa agents involves business logic spread across [flows](https://rasa.com/docs/learn/concepts/calm/), [slots](https://rasa.com/docs/reference/primitives/slots/), [custom actions](https://rasa.com/docs/learn/guides/adding-custom-actions/), and configuration files. IDE copilots are strong at coding but lack Rasa-specific knowledge. Rasa MCP Tools bridge that gap by making Rasa a **first-class tool provider** for your IDE.

### What you can do[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#what-you-can-do 'Direct link to What you can do')

| Capability | Example prompt |
| --- | --- |
| **Inspect** | "What flows and slots are in this project?" |
| **Build** | "Help me add a new subagent that manages flight bookings." |
| **Validate** | "Validate my project and explain any errors." |
| **Train** | "Train the model and tell me if it succeeded." |
| **Test** | "Send 'I want to transfer money' to the assistant and show me what happens." |
| **Debug** | "Get the assistant logs and explain why the flow was cancelled." |
| **Learn** | "Search the Rasa docs for how slot validation works." |

The MCP server exposes **17 tools** across five groups: documentation search, project introspection, schema retrieval, build/validation, and runtime testing. For a complete tool-by-tool reference, see [Rasa MCP Tools API Reference](https://rasa.com/docs/reference/api/rasa-mcp-tools/).

### How it works[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#how-it-works 'Direct link to How it works')

Your **IDE agent** starts `rasa tools run`, which is the **Rasa MCP server**: a local process that speaks MCP over stdio and reads your project files directly. You do not need a separate service for most tools. A **Rasa Server** (for example `rasa run` or `rasa inspect`) is only required for the two runtime tools: `talk_to_assistant` and `get_assistant_logs`.

```
Your machine  ┌──────────────┐  MCP (stdio)   ┌──────────────────────┐  │ IDE Agent    │ ◄────────────► │ Rasa MCP Server      │  │              │                │ rasa tools run       │  │ (your IDE)   │                │ reads project files  │  └──────────────┘                └──────────┬───────────┘                                             │ optional HTTP                                             ▼                                  ┌──────────────────────┐                                  │ Rasa Server          │                                  │ rasa run / inspect   │                                  └──────────────────────┘
```

## Setup[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#setup 'Direct link to Setup')

### Prerequisites[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#prerequisites 'Direct link to Prerequisites')

- **Rasa Pro 3.16+** installed ([Python](https://rasa.com/docs/pro/installation/python/) or [Docker](https://rasa.com/docs/pro/installation/docker/))
 
- **`RASA_LICENSE`** environment variable set. Some setups (for example **Claude Code**) start **non-interactive** shells, so they may not load `~/.zshrc`. On zsh, put the license where it is always read, such as `~/.zshenv`:
 
 ```
    echo 'export RASA_LICENSE="your-license-key"' >> ~/.zshenv
    ```
 

### Quick setup with the wizard[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#quick-setup-with-the-wizard 'Direct link to Quick setup with the wizard')

Run from your **project root**:

```
rasa tools init
```

The wizard:

- Creates `.rasa/tools.yaml` with server configuration
- Downloads offline documentation into `.rasa/`
- Generates MCP configuration for your IDE
- Downloads Rasa specific skills for your project

Use `-y` to accept all defaults:

```
rasa tools init -y
```

After the wizard completes, open your IDE from the project root and check that `rasa-tools` appears in your IDE's MCP settings.

### Manual setup by client[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#manual-setup-by-client 'Direct link to Manual setup by client')

If the wizard did not configure your IDE, or you prefer manual setup, follow the instructions for your client below.

[**Cursor**\\ \\ Jump to Cursor setup](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#cursor) [**VS Code**\\ \\ Jump to VS Code setup](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#vs-code) [**Claude Code**\\ \\ Jump to Claude Code setup](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#claude-code) [**JetBrains**\\ \\ Jump to JetBrains setup](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#jetbrains)

Your IDE starts the Rasa Tools process and communicates over **stdio**. Add a server entry like the examples below.

note

Use the same `rasa` binary you use in a terminal where `rasa train` works. Always run from the **project root** unless you pass an explicit `--project-path`.

If your IDE does not inherit `RASA_LICENSE` from your shell (common when launching from Dock or Spotlight instead of a terminal), add it via the `env` field in your MCP config:

```
"env": { "RASA_LICENSE": "your-license-key" }
```

### Cursor[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#cursor 'Direct link to Cursor')

1. Open **Settings > Tools & MCP** and add a new MCP server, or create `.cursor/mcp.json` in your project root.
2. Add the following config:

.cursor/mcp.json

```
{  "mcpServers": {    "rasa-tools": {      "command": "rasa",      "args": ["tools", "run", "--mode", "stdio"]    }  }}
```

3. Ensure `rasa` is on your `PATH`. Restart Cursor or reload MCP.
4. Open your project from the **project root**.

### VS Code (Agent Mode)[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#vs-code 'Direct link to VS Code (Agent Mode)')

1. Create or open `.vscode/mcp.json` in your project root.
2. Add a server entry:

.vscode/mcp.json

```
{  "servers": {    "rasa-tools": {      "type": "stdio",      "command": "rasa",      "args": ["tools", "run", "--mode", "stdio"]    }  }}
```

3. Reload the window or restart VS Code. The server will appear in the **MCP: List Servers** command palette entry.

### Claude Code[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#claude-code 'Direct link to Claude Code')

Claude Code launches non-interactive shells, so it may not inherit your terminal's `PATH`. Use the **full path** to `rasa` (run `which rasa` inside your activated virtualenv to find it).

**Option A — CLI (recommended)**

From your **project root**:

```
claude mcp add rasa-tools -- /path/to/your/.venv/bin/rasa tools run --mode stdio
```

This writes the entry to `~/.claude.json`. To store it in the project (shareable via git), add `-s project`:

```
claude mcp add -s project rasa-tools -- /path/to/your/.venv/bin/rasa tools run --mode stdio
```

**Option B — edit `.mcp.json` directly**

Create or open `.mcp.json` in your project root:

.mcp.json

```
{  "mcpServers": {    "rasa-tools": {      "command": "/path/to/your/.venv/bin/rasa",      "args": ["tools", "run", "--mode", "stdio"]    }  }}
```

After either option, run `/mcp` in Claude Code and confirm `rasa-tools` is **Connected**.

### JetBrains (IntelliJ, PyCharm, WebStorm, etc.)[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#jetbrains 'Direct link to JetBrains (IntelliJ, PyCharm, WebStorm, etc.)')

1. Open **Settings > Tools > AI Assistant > Model Context Protocol (MCP)**.
2. Click **Add** and choose **STDIO**.
3. Enter the following JSON config:

```
{  "mcpServers": {    "rasa-tools": {      "command": "rasa",      "args": ["tools", "run", "--mode", "stdio"]    }  }}
```

4. Set the **Working directory** to your Rasa project root.
5. Click **OK** and restart the AI Assistant.

**Using a virtualenv interpreter**

If `rasa` is not on the global `PATH`, point `command` at your venv Python and run Rasa as a module:

```
{  "mcpServers": {    "rasa-tools": {      "command": "/path/to/project/.venv/bin/python",      "args": [        "-m", "rasa",        "tools", "run",        "--mode", "stdio",        "--project-path", "/path/to/rasa/project"      ],      "env": {        "RASA_LICENSE": "your_license_key_here"      }    }  }}
```

Remove `--project-path` if you always start the IDE from the project root.

## Trying it out[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#trying-it-out 'Direct link to Trying it out')

After setup, open a copilot chat in your IDE and try these prompts:

```
Use rasa-tools to summarize what this assistant can do.
```

```
Help me use rasa-tools to create a new feature for this project.Return:1. User goal2. Required slots (new vs reused)3. Happy path4. Edge cases5. Test scenariosKeep it under 15 bullets.
```

```
Validate this Rasa project and explain any issues.
```

```
Search the Rasa docs for how to add an mcp server to this assistant project.
```

If the tools respond with project information, your setup is working. See the [Prompt-Driven Agent Tutorial](https://rasa.com/docs/learn/ai-assisted-development/) for a guided walkthrough of building a feature with prompts.

For runtime testing (talking to the assistant), you also need to start the Rasa server:

```
rasa inspect
```

Then you can test conversations:

```
Talk to the assistant: "hello", "I want to send money to Jen", "$50", "yes".Check if the transfer_money flow completed.
```

## Troubleshooting[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#troubleshooting 'Direct link to Troubleshooting')

| Issue | What to do |
| --- | --- |
| IDE says "no MCP servers found" | Check that the MCP config file exists in the right location and the JSON is valid. Cursor: `.cursor/mcp.json`. VS Code: `.vscode/mcp.json`. Claude Code: `.mcp.json` or `~/.claude.json`. Reload or restart MCP in your IDE. |
| `rasa: command not found` | Ensure `rasa` is on your `PATH`. If using a virtualenv, use the full path to `rasa` (find it with `which rasa`), or point `command` at `.venv/bin/python` and use `-m rasa` in `args`. See [JetBrains example](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#jetbrains). |
| `RASA_LICENSE` not found / license error | The IDE may not inherit your shell environment. Add `"env": { "RASA_LICENSE": "..." }` to your MCP server config, or for zsh put the export in `~/.zshenv` (not only `~/.zshrc`). See [Prerequisites](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#prerequisites). |
| Claude Code: `rasa-tools` not connected (`/mcp`) | Use the full path to `rasa` from `which rasa` in your venv. Confirm `RASA_LICENSE` is set (see row above). Verify **Rasa Pro 3.16+** with `rasa --version`. See [Claude Code](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#claude-code). |
| "Project folder not configured" | Run from the project root, or add `--project-path` to your MCP args. |
| Tools work but `talk_to_assistant` fails | Start the Rasa server first: `rasa run` or `rasa inspect`. The MCP server connects to `http://localhost:5005` by default. |
| Offline docs are stale | Run `rasa tools init docs` and restart the MCP server. |
| First tool call is slow | Expected. The MCP server lazy-loads Rasa modules on first use. Subsequent calls are faster. |
| Validation/training hangs | Large projects can take 60+ seconds. If it truly hangs, check terminal for errors. |
| Server connects but no tools appear | Restart the MCP server from your IDE. In Cursor: toggle the server off/on in Settings. In VS Code: run **MCP: List Servers** and restart. In Claude Code: `/mcp` then restart. |

For general Rasa Pro installation issues, see [Installation Troubleshooting](https://rasa.com/docs/pro/installation/troubleshooting/).

## Related[​](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#related 'Direct link to Related')

- [Rasa MCP Tools API Reference](https://rasa.com/docs/reference/api/rasa-mcp-tools/) — full tool-by-tool reference with parameters and sample prompts
- [Developer Quickstart](https://rasa.com/docs/learn/quickstart/pro/) — fastest path to a working project
- [Prompt-Driven Agent Tutorial](https://rasa.com/docs/learn/ai-assisted-development/) — build your first feature with prompts

- [What are Rasa MCP Tools?](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#what-are-rasa-mcp-tools)
 - [What you can do](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#what-you-can-do)
 - [How it works](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#how-it-works)
- [Setup](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#setup)
 - [Prerequisites](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#prerequisites)
 - [Quick setup with the wizard](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#quick-setup-with-the-wizard)
 - [Manual setup by client](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#manual-setup-by-client)
 - [Cursor](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#cursor)
 - [VS Code (Agent Mode)](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#vs-code)
 - [Claude Code](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#claude-code)
 - [JetBrains (IntelliJ, PyCharm, WebStorm, etc.)](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#jetbrains)
- [Trying it out](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#trying-it-out)
- [Troubleshooting](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#troubleshooting)
- [Related](https://rasa.com/docs/pro/installation/rasa-mcp-tools/#related)