# Source: https://rasa.com/docs/reference/primitives/default-actions

On this page

Each of these actions have a default behavior, described in the sections below. In order to overwrite this default behavior, write a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/) whose `name()` method returns the same name as the default action:

```
class ActionRestart(Action):  def name(self) -> Text:      return "action_restart"  async def run(      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]  ) -> List[Dict[Text, Any]]:      # custom behavior      return [...]
```

Add this action to the actions section of your domain file so your assistant knows to use the custom definition instead of the default one:

```
actions:  - action_restart
```

caution

After adding this action to your domain file, re-train your model with `rasa train --force`. Otherwise, Rasa won't know you've changed anything and may skip re-training your dialogue model.

## action\_listen[​](https://rasa.com/docs/reference/primitives/default-actions/#action_listen 'Direct link to action_listen')

This action is predicted to signal that the assistant should do nothing and wait for the next user input.

## action\_restart[​](https://rasa.com/docs/reference/primitives/default-actions/#action_restart 'Direct link to action_restart')

This action resets the whole conversation history, including any slots that were set during it.

It can be triggered by the user in a conversation by sending a "/restart" message, if the [RulePolicy](https://legacy-docs-oss.rasa.com/docs/rasa/policies#rule-policy) is included in the model configuration. If you define an `utter_restart` response in your domain, this will be sent to the user as well.

## action\_session\_start[​](https://rasa.com/docs/reference/primitives/default-actions/#action_session_start 'Direct link to action_session_start')

This action starts a new conversation session, and is executed in the following situations:

- at the beginning of each new conversation
- after a user was inactive for a period defined by the `session_expiration_time` parameter in the domain's [session configuration](https://rasa.com/docs/reference/config/domain/#session-configuration)
- when a user sends a "/session\_start" message during a conversation

The action will reset the conversation tracker, but by default will not clear any slots that were set.

### Customization[​](https://rasa.com/docs/reference/primitives/default-actions/#customization 'Direct link to Customization')

The default behavior of the session start action is to take all existing slots and to carry them over into the next session. Let's say you do not want to carry over all slots, but only a user's name and their phone number. To do that, you'd override the `action_session_start` with a custom action that might look like this:

```
from typing import Any, Text, Dict, Listfrom rasa_sdk import Action, Trackerfrom rasa_sdk.events import SlotSet, SessionStarted, ActionExecuted, EventTypeclass ActionSessionStart(Action):    def name(self) -> Text:        return "action_session_start"    @staticmethod    def fetch_slots(tracker: Tracker) -> List[EventType]:        """Collect slots that contain the user's name and phone number."""        slots = []        for key in ("name", "phone_number"):            value = tracker.get_slot(key)            if value is not None:                slots.append(SlotSet(key=key, value=value))        return slots    async def run(      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]    ) -> List[Dict[Text, Any]]:        # the session should begin with a `session_started` event        events = [SessionStarted()]        # any slots that should be carried over should come after the        # `session_started` event        events.extend(self.fetch_slots(tracker))        # an `action_listen` should be added at the end as a user message follows        events.append(ActionExecuted("action_listen"))        return events
```

If you want to access the metadata which was sent with the user message which triggered the session start, you can access the special slot `session_started_metadata`:

```
from typing import Any, Text, Dict, Listfrom rasa_sdk import Action, Trackerfrom rasa_sdk.events import SessionStarted, ActionExecutedclass ActionSessionStart(Action):    def name(self) -> Text:        return "action_session_start"    async def run(      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]    ) -> List[Dict[Text, Any]]:        metadata = tracker.get_slot("session_started_metadata")        # Do something with the metadata        print(metadata)        # the session should begin with a `session_started` event and an `action_listen`        # as a user message follows        return [SessionStarted(), ActionExecuted("action_listen")]
```

## action\_default\_fallback[​](https://rasa.com/docs/reference/primitives/default-actions/#action_default_fallback 'Direct link to action_default_fallback')

This action undoes the last user-bot interaction and sends the `utter_default` response if it is defined. It is triggered by low action prediction confidence, if you have this [fallback mechanism](https://legacy-docs-oss.rasa.com/docs/rasa/fallback-handoff/) enabled.

note

If `action_default_fallback` is the next action predicted and executed by the assistant, this will result in a `UserUtteranceReverted` event which will unset the slots previously filled in the last user turn.

## action\_run\_slot\_rejections[​](https://rasa.com/docs/reference/primitives/default-actions/#action_run_slot_rejections 'Direct link to action_run_slot_rejections')

This action runs [slot validation](https://rasa.com/docs/reference/primitives/flow-steps/#slot-validation) rules written directly in the flows yaml file. When the assistant asks the user for information in a flow, `action_run_slot_rejections` is executed as a step in the default flow `pattern_collect_information`. If the action evaluates one of the rules to `True`, then it will reset the originally requested slot and ask the user for the slot again by dispatching the response indicated in the `utter` property. If no rule is evaluated to `True`, then the action will retain the original value which filled the slot and the assistant will continue to the next step in the flow.

## action\_trigger\_search[​](https://rasa.com/docs/reference/primitives/default-actions/#action_trigger_search 'Direct link to action_trigger_search')

note

This action requires [Enterprise Search policy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/)

This action can be used to trigger the Enterprise Search Policy from any [flow](https://rasa.com/docs/reference/primitives/flows/), rule or story. It works by manipulating the dialogue stack frame. The result of this action is an LLM generated response to the user. Enterprise Search Policy generates the response by prompting the LLM with relevant knowledge base documents, slot context and conversation transcript.

If you have an `out_of_scope` intent, here is how you can use this action:

```
rules:- rule: Out of scope  steps:  - intent: out_of_scope  - action: action_trigger_search
```

In case you're using [FallbackClassifier](https://legacy-docs-oss.rasa.com/docs/rasa/fallback-handoff/), here is how you can use this action:

```
rules:- rule: Respond with a knowledge base search if user sends a message with low NLU confidence  steps:  - intent: nlu_fallback  - action: action_trigger_search
```

## action\_reset\_routing[​](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing 'Direct link to action_reset_routing')

This action is needed for [Coexistence of NLU-based and CALM systems](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/). It acts as a softer version of the default action `action_restart`:

- It resets all slots not marked for persistence (see [section](https://rasa.com/docs/reference/primitives/slots/#persistence-of-slots-during-coexistence)).
- Instead of completely resetting the tracker, it hides all previous tracker events from the featurization for the NLU-based system policies. This way, things that happened in CALM won’t show up in the tracker for the NLU-based system policies, but you can still see the full tracker history in tools like [`rasa inspect`](https://rasa.com/docs/reference/api/command-line-interface/#rasa-inspect). If the events were not hidden, the tracker would look different for the NLU-based system policies during inference than what it had looked like during training. This would result in bad predictions. One exception to the hiding of events are `SlotSet` events for slots that are persisted (see [section](https://rasa.com/docs/reference/primitives/slots/#persistence-of-slots-during-coexistence)).

This action also resets the slot [`route_session_to_calm`](https://rasa.com/docs/pro/calm-with-nlu/migrating-from-nlu/#adding-the-routing-slot-to-your-domain), making sure the coexistence router is engaged again on the next incoming user message. This way the user can achieve multiple skills in a single session.

## action\_clean\_stack[​](https://rasa.com/docs/reference/primitives/default-actions/#action_clean_stack 'Direct link to action_clean_stack')

This action plays a crucial role in maintaining stack integrity following a bot update.

A bot update refers to a modification or enhancement made to the bot's codebase, typically to introduce new features, fix bugs, or improve performance. In this context, the bot version transition from A to B signifies an update being deployed to the bot, which may include changes to its behavior, responses, or underlying functionality. When a bot update occurs during an ongoing conversation, it necessitates special handling to ensure that the conversation remains coherent and unaffected by the update.

This action accomplishes this by setting all frames in the stack to the end step. Currently, it is utilized within the `pattern_code_change` flow.

## action\_cancel\_flow[​](https://rasa.com/docs/reference/primitives/default-actions/#action_cancel_flow 'Direct link to action_cancel_flow')

This action is designed to gracefully terminate an ongoing conversation flow, ensuring a clean and controlled interruption of the current flow.

When `action_cancel_flow` is executed, it performs the following tasks:

- It immediately stops the execution and prevents further progression of the current flow that is in progress.
- It cleans up any ongoing conversation context related to the cancelled flow.
- It resets any slots that were set during the cancelled flow.

Currently, `action_cancel_flow` is used within the `pattern_cancel_flow` flow.

## action\_hangup[​](https://rasa.com/docs/reference/primitives/default-actions/#action_hangup 'Direct link to action_hangup')

This action is meant to be used in Voice Conversations to hang up a call. It can be used within flows to disconnect a call.

This action uses the `output_channel.hangup(sender_id)` method to hang up the call.

## action\_repeat\_bot\_messages[​](https://rasa.com/docs/reference/primitives/default-actions/#action_repeat_bot_messages 'Direct link to action_repeat_bot_messages')

Verbatim repeat the last bot message. It is currently used for [conversation repair](https://rasa.com/docs/pro/customize/patterns/) in [two patterns](https://rasa.com/docs/reference/primitives/patterns/):

- `pattern_repeat_bot_messages`, when user requests to repeat something.
- `pattern_user_silence`, when a user has been silent for a while.

- [action\_listen](https://rasa.com/docs/reference/primitives/default-actions/#action_listen)
- [action\_restart](https://rasa.com/docs/reference/primitives/default-actions/#action_restart)
- [action\_session\_start](https://rasa.com/docs/reference/primitives/default-actions/#action_session_start)
 - [Customization](https://rasa.com/docs/reference/primitives/default-actions/#customization)
- [action\_default\_fallback](https://rasa.com/docs/reference/primitives/default-actions/#action_default_fallback)
- [action\_run\_slot\_rejections](https://rasa.com/docs/reference/primitives/default-actions/#action_run_slot_rejections)
- [action\_trigger\_search](https://rasa.com/docs/reference/primitives/default-actions/#action_trigger_search)
- [action\_reset\_routing](https://rasa.com/docs/reference/primitives/default-actions/#action_reset_routing)
- [action\_clean\_stack](https://rasa.com/docs/reference/primitives/default-actions/#action_clean_stack)
- [action\_cancel\_flow](https://rasa.com/docs/reference/primitives/default-actions/#action_cancel_flow)
- [action\_hangup](https://rasa.com/docs/reference/primitives/default-actions/#action_hangup)
- [action\_repeat\_bot\_messages](https://rasa.com/docs/reference/primitives/default-actions/#action_repeat_bot_messages)