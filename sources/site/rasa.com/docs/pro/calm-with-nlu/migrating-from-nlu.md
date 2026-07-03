# Source: https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu

On this page

New in 3.8

The _Coexistence_ feature helps you to migrate from your NLU-based assistant to the [Conversational AI with Language Models (CALM)](https://rasa.com/docs/learn/concepts/calm/) approach and is available starting with version `3.8.0`.

Coexistence allows you to run a single assistant that uses both the [Conversational AI with Language Models](https://rasa.com/docs/learn/concepts/calm/) (CALM) approach and an [NLU-based system](https://legacy-docs-oss.rasa.com/docs/rasa/model-configuration) in parallel for the purpose of migrating from NLU-based assistants to CALM in an iterative fashion. This way you do not need to do a waterfall migration and can instead start gathering confidence in using CALM in production with small steps.

info

For an in-depth explanation of the _Coexistence_ feature please take a look at [Coexistence](https://rasa.com/docs/pro/calm-with-nlu/coexistence/).

## Glossary[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#glossary 'Direct link to Glossary')

- **CALM**: [CALM](https://rasa.com/docs/learn/concepts/calm/) (Conversational AI with Language Models), Rasa’s new system for creating powerful yet steerable conversational experiences with large language models fast.
- **NLU-based system**: [NLU-based systems](https://legacy-docs-oss.rasa.com/docs/rasa/model-configuration) rely on NLU components to process incoming user messages and detect intents and entities. Further, rules and stories are used to steer the next action.
- **System**: Whenever we use phrases like “the two systems” or “either of the systems” we refer to CALM or the NLU-based system.
- **Skill**: A small self-contained piece of business logic that lets the user achieve a goal or answer one or multiple questions. Examples of skills are transferring money, checking account balance, or answering frequently asked questions. Oftentimes, skills require the user to provide multiple pieces of information across multiple steps.
- **Skill interruption**: A skill is interrupted when during its execution another skill is started without stopping the earlier skill. For example, while in the process of sending money and being asked to confirm the transaction, users might do a final check of of their account balance or transaction history before giving the final confirmation. In many cases, it is desirable to return to the interrupted skill after the interrupting skill is finished.
- **Topic area**: A collection of skills that are closely related and oftentimes happen together and interrupt one another. An example for a topic area would be “investment” containing skills such as providing security quotes, buying and selling securities, portfolio management, providing quarterly and yearly results of a company, etc.

## Getting Started[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#getting-started 'Direct link to Getting Started')

important

In coexistence, CALM and the NLU-based system both live in the same assistant but are separated. While it is possible to use both systems sequentially, i.e. triggering a NLU-based system skill after CALM skills are finished and vice versa, it is not possible for a NLU-based system skill to interrupt a CALM skill and vice versa. Skill interruptions are limited to happening within one of the systems.

### Identification of Topic Areas[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#identification-of-topic-areas 'Direct link to Identification of Topic Areas')

To get started using coexistence, it is of ample importance that you identify individual skills and at least one topic area in your assistant for migration. Individual skills are useful to identify because they most commonly turn into individual flows in CALM. Topic areas are important because they group skills that should be migrated together and allow for easy skill interruptions in a single conversation session.

For example, if your bot allows for transferring money and you want to move that skill to CALM, you probably also want to move:

- skills such as
 - checking balance
 - seeing transaction history
 - adding contacts (if users first have to add someone as a contact before sending money)
- answering of frequently asked questions around the topic of money transfer

Moving these other skills as well will allow users to start these skills while having started a money transfer. If the other skills would remain in the NLU-based system only, they could only be started after the money transfer is finished or aborted.

In some cases, you might want skills to be available in both systems, in that case you should keep a copy of that skill in the NLU-based system as well. There can be two reasons to keep a skill in the NLU-based system:

- The skill is commonly used by topics in both systems.
- It's a high stakes skill, but because skill interruptions can happen within one system only, you would like the bot to still be able to handle it in both systems.

Once you have identified skills and a first topic area to migrate, you can start writing [flows](https://rasa.com/docs/reference/primitives/flows/).

In the next steps we take a look at how to integrate your new flows with the remainder of your NLU-based system.

## Updating your NLU pipeline and policies[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#updating-your-nlu-pipeline-and-policies 'Direct link to Updating your NLU pipeline and policies')

First of all you need to decide whether the routing mechanism should be based on intents or whether an LLM should decide where incoming messages should be routed to. You can choose between two different [router components](https://rasa.com/docs/reference/config/components/coexistence-routers/): `IntentBasedRouter` and `LLMBasedRouter`. [`IntentBasedRouter`](https://rasa.com/docs/reference/config/components/coexistence-routers/#intentbasedrouter) routes the messages based on the predicted intent of your NLU components. The [`LLMBasedRouter`](https://rasa.com/docs/reference/config/components/coexistence-routers/#llmbasedrouter) leverages an LLM to decide whether an incoming message should be routed to the NLU-based system or CALM. For more information on those components, please read the [router components documentation](https://rasa.com/docs/reference/config/components/coexistence-routers/).

Once you have decided which router component to use, you need to add it to your config file together with a LLM-based command generator and `FlowPolicy`. For more information on the LLM-based command generators or the `FlowPolicy`, please consult the documentation for [flows](https://rasa.com/docs/pro/build/writing-flows/).

After adding the components, your `config.yml` might look like the following, depending on which router you chose and which NLU-based system components you employ:

```
recipe: default.v1language: enpipeline:- name: LLMBasedRouter  calm_entry:    sticky: "handles everything around contacts"- name: WhitespaceTokenizer- name: CountVectorsFeaturizer- name: CountVectorsFeaturizer  analyzer: char_wb  min_ngram: 1  max_ngram: 4- name: LogisticRegressionClassifier- name: CompactLLMCommandGenerator  llm:    provider: openai    model: gpt-5.1-2025-11-13    timeout: 7    max_tokens: 256policies:- name: FlowPolicy- name: RulePolicy- name: MemoizationPolicy  max_history: 10- name: TEDPolicy
```

### Adding the routing slot to your domain[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#adding-the-routing-slot-to-your-domain 'Direct link to Adding the routing slot to your domain')

The routing slot is used to store the routing decision of the router component, e.g. whether a message should be routed to the NLU-based system or CALM.

It’s important that you add the routing slot to your domain like the following:

```
version: "3.1"slots:  route_session_to_calm:    type: bool    influence_conversation: false
```

This needs to be exactly this way. It needs to be a boolean slot to capture the three cases based on its value:

- `None`: no routing active, router will be engaged
- `True`: routing to CALM is active
- `False`: routing to the NLU-based system is active

A routing session is active when the routing slot is either set to `True` or `False`. Incoming messages are routed accordingly. If the routing slot is reset and no routing is active, e.g. set to `None`, the router will be engaged again on the next incoming message. To reset the routing slot the new default action [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing) can be called (see [section](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#managing-routing-resets)).

The `influence_conversation: false` is important to not trip up the NLU-based system policies based on the value of this slot.

If you want to have one of the system active right from the beginning of the conversation, you can give the slot an `initial_value`. During routing resets, this slot will always be set to `None` and not to its initial value.

warning

If you want to reset the routing slot, call the new default action [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing). Do not update the value of the routing slot in isolation, e.g. in a custom action, as it might lead to unexpected behaviour of the assistant. Always use [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing) if your objective is to reset routing.

### Managing routing resets[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#managing-routing-resets 'Direct link to Managing routing resets')

Resetting the routing slot is important to be able to achieve multiple skills in a single session. To reset the routing the new default action [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing) needs to be called.

**Resetting when CALM is finished**

Resetting once CALM is finished is relatively straightforward. Whenever all started flows are finished, CALM triggers the flow `pattern_completed`. We can leverage this fact and adjust this pattern slightly as follows:

```
flows:    pattern_completed:      description: all flows have been completed and there is nothing else to be done      name: pattern completed      steps:        - action: utter_can_do_something_else        - action: action_reset_routing
```

In the original [definition of `pattern_completed`](https://rasa.com/docs/reference/primitives/patterns/#reference-default-pattern-configuration), we just ask the user whether we can do something else for them. In this version, we also reset the routing. Make sure to add this snippet to your flows to overwrite the default implementation of `pattern_completed` that comes with CALM.

This setup will take care of all the following cases:

- User starts a flow and finishes or cancels it.
- User starts multiple flows in sequence or simultaneously and finishes or cancels them all.
- User starts a flow and asks knowledge questions or uses chitchat while in a flow and then finishes or cancels the flow.
- User starts a CALM session directly with a knowledge related question.

**Resetting when the NLU-based system is finished**

In contrast to CALM, where the boundaries of skills are well defined, the NLU-based system policies use more loosely connected fragments of business logic to describe skills. If you want to allow the user to use the CALM part of your assistant after a skill was completed in the NLU-based system, you will have to add the [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing) at the appropriate places in your rules and stories.

Usually, this will be at the end of different stories and rules. Potentially you have some conversation parts that you carry out at the end of each interaction. These parts can be great places at whose end you can reset the routing as in the following:

```
version: "3.1"stories:- story: feedback  steps:  - checkpoint: start_of_feedback  - action: utter_ask_feedback  - intent: submit_rating  - action: store_rating  - action: utter_thank_you_rating  - action: utter_do_something_else  - action: action_reset_routing
```

It is very likely that you have to add the `action_reset_routing` at the end of multiple stories or rules, whenever you are sure that the user has just finished a skill and nothing else needs to be wrapped up in the NLU-based system such as an interrupted skill.

## Managing shared slots between CALM and the NLU-based system[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#managing-shared-slots-between-calm-and-the-nlu-based-system 'Direct link to Managing shared slots between CALM and the NLU-based system')

To prevent slots from being reset during the [`action_reset_routing`](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing) you can configure slots with the option [`shared_for_coexistence: True`](https://rasa.com/docs/reference/primitives/slots/#persistence-of-slots-during-coexistence).

Alternatively, you can store user profile information in a database and retrieve it in different places during your stories and flows and store it in slots temporarily. In this case, you won’t need to worry about the persistence of shared slots as the data is persisted in the database.

## Custom Actions[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#custom-actions 'Direct link to Custom Actions')

When moving functionality to CALM you might have to adjust your custom actions to varying degrees. Some concepts, such as dynamic forms have been integrated into the core flow functionality and should be implemented using branches in flows. It is best to reconsider the implementation of all custom actions that are moved to CALM and assess whether they should be adjusted to better play with the new paradigm.

Generally speaking, it is also possible to use the same action both in CALM and the NLU-based system. Both systems share the same action server.

## Testing your assistant[​](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#testing-your-assistant 'Direct link to Testing your assistant')

Testing your assistant using both CALM and NLU-based system can be achieved using [e2e tests](https://rasa.com/docs/pro/testing/evaluating-assistant/). The definition of these tests is not very dependent on the underlying implementation and should thus not need to change significantly when more and more skills are moved from the NLU-based system to CALM subsequently. e2e tests help you to test skill interruptions and skill triggers across paradigms.

You can still evaluate just the NLU-based system of your assistant using the command [`rasa test`](https://legacy-docs-oss.rasa.com/docs/rasa/command-line-interface/#rasa-test).