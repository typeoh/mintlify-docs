# Source: https://rasa.com/docs/pro/calm-with-nlu/coexistence

On this page

## Key Terms[​](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#key-terms 'Direct link to Key Terms')

### System

Whenever we use phrases like “the two systems” or “either of the systems” we refer to CALM or the NLU-based system.

### Stickiness

Stickiness means that once a routing decision is made by the router, it remains in place until it is reset, which normally happens at the end of a flow, story, or rule. This prevents CALM from taking over midway through a story or rule or the NLU-based system from taking over midway through a flow. Such interruptions can be problematic as the two systems handle different skills and switching from one to the other require a cleanup in order to be able to complete a skill without errors. Non-sticky routing is only used for single-turn interactions such as, for example, chit-chat.

### Skill

A small self-contained piece of business logic that lets the user achieve a goal or answer one or multiple questions. Examples of skills are transferring money, checking account balance, or answering frequently asked questions. Oftentimes, skills require the user to provide multiple pieces of information across multiple steps.

### Topic Area

A collection of skills that are closely related and oftentimes happen together and interrupt one another. An example for a topic area would be “investment” containing skills such as providing security quotes, buying and selling securities, portfolio management, providing quarterly and yearly results of a company, etc.

## Overview[​](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#overview 'Direct link to Overview')

The following graphic depicts how coexistence works on a high level.

The coexistence of [CALM](https://rasa.com/docs/learn/concepts/calm/) and the NLU-based system depends on a routing mechanism, that routes messages based on their content to either system. The routing happens in the [router component](https://rasa.com/docs/reference/config/components/coexistence-routers/). We currently offer two different router components, in the Figure 1 the [`IntentBasedRouter`](https://rasa.com/docs/reference/config/components/coexistence-routers/#intentbasedrouter) is used.

If the routing session is not yet active, e.g. the dedicated routing slot [`route_session_to_calm`](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#adding-the-routing-slot-to-your-domain) is not yet set, the router is engaged and the message is routed depending on the outcome of the routing mechanism. Once the router decided to route the message to either of the system, the routing slot will be set and a routing session is active. The routing is usually sticky. That means that the subsequent messages are routed to the same system without engaging the router again. E.g. if the routing slot is already set, the router component is not engaged and the message is routed according to the value set in the routing slot.

Once the message is routed to either of the systems, the message is processed as that system would process it usually. The message annotated with either commands or intents and entities is passed to the policies. Here, the routing slot together with CALM's dialogue stack coordinates the policies. In general, the policies of one system will not run when the other system is activated to save compute.

For the user to be able to achieve multiple skills in a single session, CALM and the NLU-based system have to signal when they are finished and ready to give control back. Skills are split across the two systems such as making a loan payment and checking on some investment news, and so it is important that the routing can be reset. CALM and the NLU-based system policies can signal that they are done with all started skills in their system and reset the routing by calling the new default action [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing). The router will be engaged again on the next incoming user message.

## Intent triggers[​](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#intent-triggers 'Direct link to Intent triggers')

Intents can be triggered in rasa using the slash syntax as in `/initialise_conversation`. These intent triggers are not processed by the NLU pipeline and thus the router will never see them. At the same time, there might be starting skills requiring the session to be assigned to either CALM or the NLU-based system. To support intent triggers while using the coexistence feature we have devised the following heuristic:

- If a route to either the NLU-based system or CALM is active, the routing is not changed.
- If no route is active:
 - We route to CALM if given the intent any of the NLU triggers of flows are activated (see [NLU triggers documentation](https://rasa.com/docs/reference/primitives/starting-flows/#nlu-trigger)).
 - We route to the NLU-based system otherwise.

This way you can design intent triggers that will start specific NLU-based system or CALM skills.

Getting Started

The _Coexistence_ feature helps you to [migrate from your NLU-based](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/) assistant to the [Conversational AI with Language Models (CALM)](https://rasa.com/docs/learn/concepts/calm/) approach and is available starting with version `3.8.0`. In order to start using the _Coexistence_ feature, follow the steps in the [user guide](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/).

- [Key Terms](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#key-terms)
- [Overview](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#overview)
- [Intent triggers](https://rasa.com/docs/pro/calm-with-nlu/coexistence/#intent-triggers)