# Source: https://rasa.com/docs/learn/ai-assisted-development

On this page

Build a conversational feature from scratch using your IDE copilot and Rasa MCP Tools. Each step is a prompt you paste into your IDE chat. By the end, you will have a tested, working feature — and you will have used most of the 17 MCP tools along the way.

## Before you start[​](https://rasa.com/docs/learn/ai-assisted-development/#before-you-start 'Direct link to Before you start')

- A Rasa project with MCP tools connected. If you don't have one yet, follow the [Developer Quickstart](https://rasa.com/docs/learn/quickstart/pro/) first.
- Your IDE open **from the project root** with `rasa-tools` showing as connected.

Coming from the quickstart?

If you just ran the "propose 3 verticals" prompt, pick the one you liked best and use it as your feature throughout this tutorial. The example below uses **"check my order status"** — swap in your own idea wherever you see it.

## Step 1: Explore your project[​](https://rasa.com/docs/learn/ai-assisted-development/#step-1-explore-your-project 'Direct link to Step 1: Explore your project')

Understand what the agent already does before adding anything new.

```
Use rasa-tools to list all flows, slots, and responses in this project.Summarize what the agent can do today in a few sentences.
```

**Tools used:** `list_project_flow_definitions`, `list_project_slot_definitions`, `list_project_response_definitions`

## Step 2: Design the feature[​](https://rasa.com/docs/learn/ai-assisted-development/#step-2-design-the-feature 'Direct link to Step 2: Design the feature')

Ask the copilot to look up how Rasa flows work, then design the feature based on your project's current state.

```
I want to add a "check my order status" feature to this agent.Use rasa-tools to search the Rasa docs for how flows and collect steps work.Then design the feature:1. User goal2. Slots needed (new vs. reused from the existing project)3. Happy path steps4. One edge case5. Two test scenarios (happy path + edge case)Keep it under 15 bullets.
```

**Tools used:** `search_rasa_documentation`, plus the introspection tools from step 1

Review the design before moving on. Adjust the slot names or edge cases if they don't fit your project.

## Step 3: Write tests first[​](https://rasa.com/docs/learn/ai-assisted-development/#step-3-write-tests-first 'Direct link to Step 3: Write tests first')

Get the correct E2E test format from Rasa, then write tests before any implementation.

```
Use rasa-tools to get the E2E test schema.Write end-to-end tests for "check my order status":- Happy path: user asks for order status, provides an order number, gets a status back- Edge case: user starts the flow but cancelsAdd the tests to the existing test file (don't overwrite what's already there).
```

**Tools used:** `get_e2e_schema`

## Step 4: Implement[​](https://rasa.com/docs/learn/ai-assisted-development/#step-4-implement 'Direct link to Step 4: Implement')

Get the flow and domain schemas so the implementation is valid YAML from the start.

```
Use rasa-tools to get the flow schema and the domain schema.Then implement "check my order status":1. Create the flow2. Add new slots and responses to the domain3. Write a stub custom action that returns a mock order statusConstraints:- Reuse existing slots and responses where possible- One flow = one user goal- Follow the schemas exactly
```

**Tools used:** `get_flow_schema`, `get_domain_schema`

## Step 5: Validate and train[​](https://rasa.com/docs/learn/ai-assisted-development/#step-5-validate-and-train 'Direct link to Step 5: Validate and train')

```
Use rasa-tools to validate this project.If validation passes, train the assistant.If it fails, fix the errors and retry until both pass.Return the validation result and the training result.
```

**Tools used:** `validate_project`, `train_rasa_assistant`

This can take a minute or two. If training fails, the copilot should fix the issue and retry automatically.

## Step 6: Talk to your agent[​](https://rasa.com/docs/learn/ai-assisted-development/#step-6-talk-to-your-agent 'Direct link to Step 6: Talk to your agent')

Start the Rasa server in a separate terminal:

```
rasa inspect
```

Then test the feature with a real conversation:

```
Use rasa-tools to talk to the assistant:"hi", "I want to check my order status", "order 12345"Verify that the correct flow was triggered and the order status was returned.If something looks wrong, get the assistant logs and explain the issue.
```

**Tools used:** `talk_to_assistant`, `get_assistant_logs`

## What you just used[​](https://rasa.com/docs/learn/ai-assisted-development/#what-you-just-used 'Direct link to What you just used')

| Step | What happened | MCP tools |
| --- | --- | --- |
| Explore | Mapped the existing project | `list_project_flow_definitions`, `list_project_slot_definitions`, `list_project_response_definitions` |
| Design | Searched docs and designed the feature | `search_rasa_documentation` |
| Test | Wrote E2E tests in the correct format | `get_e2e_schema` |
| Implement | Built valid flow, domain, and action code | `get_flow_schema`, `get_domain_schema` |
| Validate & train | Caught errors early, then trained | `validate_project`, `train_rasa_assistant` |
| Talk | Ran a real conversation and debugged | `talk_to_assistant`, `get_assistant_logs` |

## Tips for prompting[​](https://rasa.com/docs/learn/ai-assisted-development/#tips-for-prompting 'Direct link to Tips for prompting')

- **One prompt per step.** Don't try to do everything at once.
- **Validate before training.** It catches errors in seconds instead of minutes.
- **Ask for concise output.** "Return only changed files and tool results" cuts the noise.
- **Debug with logs.** When a conversation doesn't go as expected, `get_assistant_logs` usually explains why.
- **Iterate.** Add another edge case, a second feature, or connect a real API. Each round follows the same explore → design → test → implement → validate → train → talk cycle.

## Next steps[​](https://rasa.com/docs/learn/ai-assisted-development/#next-steps 'Direct link to Next steps')

- [Rasa MCP Tools API Reference](https://rasa.com/docs/reference/api/rasa-mcp-tools/) — full list of all 17 tools with parameters and sample prompts
- [Rasa MCP Tools Setup](https://rasa.com/docs/pro/installation/rasa-mcp-tools/) — client configuration for Cursor, VS Code, Claude Code, and JetBrains
- [Flows](https://rasa.com/docs/learn/concepts/calm/) — how Rasa flows work under the hood
- [End-to-End Testing](https://rasa.com/docs/pro/testing/evaluating-assistant/) — writing and running E2E tests

- [Before you start](https://rasa.com/docs/learn/ai-assisted-development/#before-you-start)
- [Step 1: Explore your project](https://rasa.com/docs/learn/ai-assisted-development/#step-1-explore-your-project)
- [Step 2: Design the feature](https://rasa.com/docs/learn/ai-assisted-development/#step-2-design-the-feature)
- [Step 3: Write tests first](https://rasa.com/docs/learn/ai-assisted-development/#step-3-write-tests-first)
- [Step 4: Implement](https://rasa.com/docs/learn/ai-assisted-development/#step-4-implement)
- [Step 5: Validate and train](https://rasa.com/docs/learn/ai-assisted-development/#step-5-validate-and-train)
- [Step 6: Talk to your agent](https://rasa.com/docs/learn/ai-assisted-development/#step-6-talk-to-your-agent)
- [What you just used](https://rasa.com/docs/learn/ai-assisted-development/#what-you-just-used)
- [Tips for prompting](https://rasa.com/docs/learn/ai-assisted-development/#tips-for-prompting)
- [Next steps](https://rasa.com/docs/learn/ai-assisted-development/#next-steps)