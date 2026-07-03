# Source: https://rasa.com/docs/pro/build/integrating-external-agents

On this page

New Beta Feature in 3.14

Rasa supports stateful execution of external agents via A2A protocol.

This feature is in a beta (experimental) stage and may change in future Rasa versions. We welcome your feedback on this feature.

## Overview[​](https://rasa.com/docs/pro/build/integrating-external-agents/#overview 'Direct link to Overview')

Rasa can act as an intelligent orchestrator that coordinates a network of multiple agents through the [Agent-to-Agent (A2A) protocol](https://a2a-protocol.org/latest/#what-is-a2a-protocol). This is particularly useful when at least one of the agents is externally built, i.e. by a different department, by a third party, or it's a legacy project on a different tech stack. Integrating each of these agents into a single system enables contextually rich conversations through single unified conversational interface.

## The Orchestrator Model[​](https://rasa.com/docs/pro/build/integrating-external-agents/#the-orchestrator-model 'Direct link to The Orchestrator Model')

In this architecture, your Rasa agent serves as the **orchestrator** that:

1. **Owns the conversation flow** - Determines when and which external agents to involve
2. **Manages business logic** - Enforces business rules, compliance, and conversation patterns
3. **Controls agent handoffs** - Decides when to delegate tasks and when to resume control
4. **Maintains conversation context** - Preserves user context across different agent interactions
5. **Allows context engineering** - Allows fine-grained control over context passed between agents
6. **Handles final responses** - Ensures consistent user experience regardless of which agent provided the information

## Configuring External Sub-Agents[​](https://rasa.com/docs/pro/build/integrating-external-agents/#configuring-external-sub-agents 'Direct link to Configuring External Sub-Agents')

### Registering the sub agent[​](https://rasa.com/docs/pro/build/integrating-external-agents/#registering-the-sub-agent 'Direct link to Registering the sub agent')

Configure external sub agents in the `sub_agents/` folder of your Rasa agent:

```
your_project/├── config.yml├── domain/├── data/flows/└── sub_agents/    ├── analytics_agent/       └── config.yml
```

The A2A protocol prescribes using an [agent card](https://a2a-protocol.org/latest/specification/#55-agentcard-object-structure) to expose any externally running agent's capabilities. The agent card can be provided as part of the `config.yml` defined for each external agent -

sub\_agents/analytics\_agent/config.yml

```
# Basic agent informationagent:  name: analytics_agent  protocol: a2a  description: "External agent that provides analytical capabilities"# A2A configurationconfiguration:  agent_card: ./sub_agents/shopping_agent/agent_card.json
```

More configuration options can be found in the [reference page](https://rasa.com/docs/reference/config/agents/external-sub-agents/#configuration).

## Invoking the External Agent[​](https://rasa.com/docs/pro/build/integrating-external-agents/#invoking-the-external-agent 'Direct link to Invoking the External Agent')

The orchestrating layer in Rasa provides you the control to invoke an external agent when your business logic needs you to do so by using the [`call` step](https://rasa.com/docs/reference/primitives/flow-steps/#autonomous-steps) in a flow -

flows.yml

```
flows:  product_enquiry:    description: Handle questions related to analysis needed on the product    steps:    - collect: query    - call: analytics_agent  # External A2A agent
```

For more information on how Rasa passes control and receives response from an external agent via A2A, head over to the documentation on [External Sub Agents](https://rasa.com/docs/reference/config/agents/overview-agents/#how-rasa-interacts-with-sub-agents).

## Context Management[​](https://rasa.com/docs/pro/build/integrating-external-agents/#context-management 'Direct link to Context Management')

The orchestrating agent in Rasa can control what context is shared with external agents by customizing the client interfacing with the external agent.

To do so, define a custom python module -

sub\_agents/analytics\_agent/custom\_agent.py

```
from rasa.agents.protocol.a2a.a2a_agent import A2AAgentfrom rasa.agents.schemas import AgentInputclass AnalyticsAgent(A2AAgent):    async def process_input(self, input: AgentInput) -> AgentInput:        # Filter slots to only include relevant context        filtered_slots = [            slot for slot in input.slots            if slot.name in ["user_id", "account_type", "inquiry_category"]        ]        # Create new input with filtered context        return AgentInput(            id=input.id,            user_message=input.user_message,            slots=filtered_slots,  # Only share necessary slots            conversation_history=input.conversation_history,            events=input.events,            metadata=input.metadata,            timestamp=input.timestamp        )
```

Provide the customized interface to your sub agent's config:

sub\_agents/analytics\_agent/config.yml

```
agent:  name: analytics_agent  protocol: a2a  description: "External agent that provides analytical capabilities"configuration:  agent_card: ./sub_agents/analytics_agent/agent_card.json  module: sub_agents.analytics_agent.custom_agent.AnalyticsAgent
```

For more information on possible customizations, head over to the [reference documentation](https://rasa.com/docs/reference/config/agents/external-sub-agents/#customization).

- [Overview](https://rasa.com/docs/pro/build/integrating-external-agents/#overview)
- [The Orchestrator Model](https://rasa.com/docs/pro/build/integrating-external-agents/#the-orchestrator-model)
- [Configuring External Sub-Agents](https://rasa.com/docs/pro/build/integrating-external-agents/#configuring-external-sub-agents)
 - [Registering the sub agent](https://rasa.com/docs/pro/build/integrating-external-agents/#registering-the-sub-agent)
- [Invoking the External Agent](https://rasa.com/docs/pro/build/integrating-external-agents/#invoking-the-external-agent)
- [Context Management](https://rasa.com/docs/pro/build/integrating-external-agents/#context-management)