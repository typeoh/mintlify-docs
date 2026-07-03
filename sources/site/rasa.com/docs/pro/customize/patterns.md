# Source: https://rasa.com/docs/pro/customize/patterns

On this page

Rasa assistants use patterns (pre-built flow templates) to handle conversation repair and some system behaviour. By separating patterns from the business-logic flows, you avoid having to weave every possible “what if the user changes their mind” scenario into your main flows. Instead, your flows can stay focused on the high-level steps for accomplishing tasks (booking a flight, transferring money, etc.), while the conversation patterns makes sure your flows are adaptive to how the user wants to accomplish their goal.

By default, every common conversation repair scenario—like corrections, digressions, or incomplete data—has a pre-written pattern in Rasa. However, you can override those defaults to better suit your assistant’s needs.

Digging into patterns

For more information on the concept of patterns, head to the [Home section on Patterns](https://rasa.com/docs/learn/concepts/conversation-patterns/).

For a complete list of patterns and their low-level implementations, head to the [corresponding section in the Reference](https://rasa.com/docs/reference/primitives/patterns/).

## Why Customise Patterns?[​](https://rasa.com/docs/pro/customize/patterns/#why-customise-patterns 'Direct link to Why Customise Patterns?')

### Templated Flows for Repair[​](https://rasa.com/docs/pro/customize/patterns/#templated-flows-for-repair 'Direct link to Templated Flows for Repair')

Think of a pattern as a **reusable template** for handling a repeatable scenario (cancellation, correction, human handoff, etc.). Because patterns need to work anywhere in your assistant, you typically want to keep them general rather than making them skill-specific.

### Balancing Consistency and Adaptation[​](https://rasa.com/docs/pro/customize/patterns/#balancing-consistency-and-adaptation 'Direct link to Balancing Consistency and Adaptation')

Although you want to keep these templated flows broadly applicable, you may still want to:

- **Adapt the language** of responses (e.g., brand or domain-specific wording).
- **Request user confirmation** before updating slots (such as in the correction pattern).
- **Hand over to a live agent** when certain issues arise.

For purely language-level customization (i.e., to keep brand voice consistent), consider using the Response Rephraser to automatically adapt the text in the pattern’s default responses. You can find more information on customizing the tone of your assistant [here](https://rasa.com/docs/pro/customize/assistant-tone/).

## How to Customise Patterns[​](https://rasa.com/docs/pro/customize/patterns/#how-to-customise-patterns 'Direct link to How to Customise Patterns')

### 1\. Override a Pattern Flow[​](https://rasa.com/docs/pro/customize/patterns/#1-override-a-pattern-flow 'Direct link to 1. Override a Pattern Flow')

To customise a pattern, create a flow in your `flows.yml` file **with the same name** as the pattern. For instance, if you want to change how a cancellation is handled, create a flow named `pattern_cancel_flow`:

flows.yml

```
flows:  pattern_cancel_flow:    description: Custom cancellation flow    steps:      - action: action_cancel_flow      - action: utter_flow_cancelled_rasa
```

When your assistant sees a user cancellation, it will **use your new `pattern_cancel_flow`** instead of the default one.

For existing default pattern implementations see to the Reference section on [Patterns](https://rasa.com/docs/reference/primitives/patterns/#reference-default-pattern-configuration).

_Keep pattern overrides concise. Most patterns interrupt an ongoing flow, so it’s best to return the user to the main conversation quickly._

### 2\. Override Default Actions or Responses[​](https://rasa.com/docs/pro/customize/patterns/#2-override-default-actions-or-responses 'Direct link to 2. Override Default Actions or Responses')

Some patterns make use of **default actions**, like `action_correct_flow_slot` or `action_trigger_chitchat`. If you need more control over the logic:

1. Create a custom action in your `actions.py`.
2. Reference it in your custom pattern flow and your `domain.yml`.

Likewise, the default text responses are in your domain’s `responses:` section. Override them with your own text or tie them into the Rephraser if needed.

### 3\. Link Patterns to Other Flows[​](https://rasa.com/docs/pro/customize/patterns/#3-link-patterns-to-other-flows 'Direct link to 3. Link Patterns to Other Flows')

You can also “jump” from a pattern to another flow (for example, if the user clarifies something and you want to start a brand-new flow). To do this, add a **link** step at the end of your pattern:

flows.yml

```
flows:  pattern_clarification:    description: "Clarification pattern"    steps:      - noop: true        next:          - if: context.names            then:            - action: action_clarify_flows              next:                - if: context.names contains "transfer_money"                  then:                    - link: transfer_money                - else: clarify_options          - else:            - action: utter_clarification_no_options_rasa              next: END      - id: clarify_options        action: utter_clarification_options_rasa
```

When the user clarifies that they want to transfer money, you directly start the `transfer_money` flow.

### 4\. Human Handoff and Other Special Cases[​](https://rasa.com/docs/pro/customize/patterns/#4-human-handoff-and-other-special-cases 'Direct link to 4. Human Handoff and Other Special Cases')

For certain special patterns like `pattern_human_handoff` or `pattern_chitchat`:

- **Human handoff**: Override `pattern_human_handoff` to add your own logic for connecting to a live agent.
- **Chitchat**: By default, chitchat is turned off, and the assistant responds with the default `utter_cannot_handle` response. You can switch to free-form generation by overriding `pattern_chitchat` and calling an LLM-based response.

For more information on customizing patterns, see to the Reference section on [Patterns](https://rasa.com/docs/reference/primitives/patterns/#common-modifications).

- [Why Customise Patterns?](https://rasa.com/docs/pro/customize/patterns/#why-customise-patterns)
 - [Templated Flows for Repair](https://rasa.com/docs/pro/customize/patterns/#templated-flows-for-repair)
 - [Balancing Consistency and Adaptation](https://rasa.com/docs/pro/customize/patterns/#balancing-consistency-and-adaptation)
- [How to Customise Patterns](https://rasa.com/docs/pro/customize/patterns/#how-to-customise-patterns)
 - [1\. Override a Pattern Flow](https://rasa.com/docs/pro/customize/patterns/#1-override-a-pattern-flow)
 - [2\. Override Default Actions or Responses](https://rasa.com/docs/pro/customize/patterns/#2-override-default-actions-or-responses)
 - [3\. Link Patterns to Other Flows](https://rasa.com/docs/pro/customize/patterns/#3-link-patterns-to-other-flows)
 - [4\. Human Handoff and Other Special Cases](https://rasa.com/docs/pro/customize/patterns/#4-human-handoff-and-other-special-cases)