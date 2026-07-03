# Source: https://rasa.com/docs/learn/best-practices/conversation-design

On this page

This guide is intended as an introduction to how to create conversations that are helpful and feel natural.

It's designed specifically with **designers** in mind, but it's also useful for anyone who wants to learn how to get the most out of Rasa.

### Topics covered[​](https://rasa.com/docs/learn/best-practices/conversation-design/#topics-covered 'Direct link to Topics covered')

- What makes a good conversation? CxD
- How CxD works with CALM
- How to write responses that feel human
- How to test and improve through feedback

## What is Conversation Design (CxD)?[​](https://rasa.com/docs/learn/best-practices/conversation-design/#what-is-conversation-design-cxd 'Direct link to What is Conversation Design (CxD)?')

**Conversation Design** is the process of researching, conceptualizing, and creating conversational interactions between human users and AI. It is an interdisciplinary field drawing from UX design, linguistics, conversation analysis, content management, human-computer interaction, and related studies.

Conversation design is both human-centered and data-driven.

## What is CALM?[​](https://rasa.com/docs/learn/best-practices/conversation-design/#what-is-calm 'Direct link to What is CALM?')

**Rasa CALM** is a state-of-the-art hybrid approach to building conversational AI assistants. It combines 'flows' — predefined sets of steps to complete tasks that represent business processes — with the power of language models to:

- Understand user intentions
- Handle edge cases
- Repair conversations
- Generate responses where appropriate

To read more about CALM, see the [CALM documentation](https://rasa.com/docs/learn/concepts/calm/).

## How does CALM affect the CxD process?[​](https://rasa.com/docs/learn/best-practices/conversation-design/#how-does-calm-affect-the-cxd-process 'Direct link to How does CALM affect the CxD process?')

As a conversation designer, each project starts with **discovery**, focusing on the use cases, scope, and tasks for the AI assistant. Extensive research is conducted on:

- Target audience interaction goals and needs
- Conversation habits
- Language styles

### Outcomes of the Discovery Phase:[​](https://rasa.com/docs/learn/best-practices/conversation-design/#outcomes-of-the-discovery-phase 'Direct link to Outcomes of the Discovery Phase:')

The research leads to user journeys that are converted into _conversational flows_ — dynamic representations of possible conversation paths between the user and AI assistant. These flows execute **Business Logic** in CALM.

### Designing with CALM:[​](https://rasa.com/docs/learn/best-practices/conversation-design/#designing-with-calm 'Direct link to Designing with CALM:')

You don't need complex flowcharts with many interlinked branches. Instead, each flow can focus on specific tasks, such as:

- Responding to user questions
- Connecting to a knowledge base for requested information
- Collecting user information in a series of steps
- Performing actions (e.g., checking a balance, blocking a card, booking an appointment)
- Transferring to a human operator

### Building a Flow in CALM:[​](https://rasa.com/docs/learn/best-practices/conversation-design/#building-a-flow-in-calm 'Direct link to Building a Flow in CALM:')

While building flows in CALM, you can predefine:

- The assistant's responses (which could also be generated)
- Information to collect from the user (_slots_) and subsequent actions
- How to act based on user inputs (_logic_)
- Next steps in the conversation (_links_)

CALM's **Dialogue Understanding** module leverages language models to interpret user statements and intentions. As a designer:

- You aren't required to build sets of intents, entities, and variations for every scenario.
- However, you can still do so for specific scenarios if desired.

### LLM-Based `CommandGenerator`:[​](https://rasa.com/docs/learn/best-practices/conversation-design/#llm-based-commandgenerator 'Direct link to llm-based-commandgenerator')

The `CommandGenerator` translates user input into commands that drive the conversation forward by triggering flows, operations, repair patterns, and more. It considers:

- Conversation history
- Context

#### Key Customization Options:[​](https://rasa.com/docs/learn/best-practices/conversation-design/#key-customization-options 'Direct link to Key Customization Options:')

- **Prompting the LLM:** Use flow descriptions and slot definitions to guide the LLM in generating appropriate commands. [Learn more about prompting](https://rasa.com/docs/reference/config/components/llm-command-generators/#customization).
- **Flow Retrieval:** Pre-select relevant flows for a given conversation. [Learn more about flow retrieval](https://rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows).

## CxD Workflow in CALM[​](https://rasa.com/docs/learn/best-practices/conversation-design/#cxd-workflow-in-calm 'Direct link to CxD Workflow in CALM')

### Outside of Rasa: Designer Responsibilities[​](https://rasa.com/docs/learn/best-practices/conversation-design/#outside-of-rasa-designer-responsibilities 'Direct link to Outside of Rasa: Designer Responsibilities')

- Gather data on user needs, language style, and conversational habits
- Define user personas, map content, and user journeys
- Create an AI assistant personality
- Write sample dialogues
- Draft conversational flows
- Plan error handling, escalation strategies, and handovers (if needed)
- Prepare user testing rounds and protocols

### Inside Rasa: Builder Responsibilities[​](https://rasa.com/docs/learn/best-practices/conversation-design/#inside-rasa-builder-responsibilities 'Direct link to Inside Rasa: Builder Responsibilities')

- Build flows based on designs and user journeys
- Write efficient flow and slot descriptions to guide the LLM
- Create responses (if needed) that align with personality guidelines
- Prompt the LLM to generate or rephrase responses (if needed)
- Customize conversation repair patterns per error handling strategies
- Write end-to-end (e2e) tests based on sample dialogues
- Connect the assistant to knowledge sources and implement RAG
- Instruct the LLM on generating texts for RAG responses
- Debug designs and test the assistant using Rasa Inspector

- [Topics covered](https://rasa.com/docs/learn/best-practices/conversation-design/#topics-covered)
- [What is Conversation Design (CxD)?](https://rasa.com/docs/learn/best-practices/conversation-design/#what-is-conversation-design-cxd)
- [What is CALM?](https://rasa.com/docs/learn/best-practices/conversation-design/#what-is-calm)
- [How does CALM affect the CxD process?](https://rasa.com/docs/learn/best-practices/conversation-design/#how-does-calm-affect-the-cxd-process)
 - [Outcomes of the Discovery Phase:](https://rasa.com/docs/learn/best-practices/conversation-design/#outcomes-of-the-discovery-phase)
 - [Designing with CALM:](https://rasa.com/docs/learn/best-practices/conversation-design/#designing-with-calm)
 - [Building a Flow in CALM:](https://rasa.com/docs/learn/best-practices/conversation-design/#building-a-flow-in-calm)
 - [LLM-Based `CommandGenerator`:](https://rasa.com/docs/learn/best-practices/conversation-design/#llm-based-commandgenerator)
- [CxD Workflow in CALM](https://rasa.com/docs/learn/best-practices/conversation-design/#cxd-workflow-in-calm)
 - [Outside of Rasa: Designer Responsibilities](https://rasa.com/docs/learn/best-practices/conversation-design/#outside-of-rasa-designer-responsibilities)
 - [Inside Rasa: Builder Responsibilities](https://rasa.com/docs/learn/best-practices/conversation-design/#inside-rasa-builder-responsibilities)