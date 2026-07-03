# Source: https://rasa.com/docs/studio/analyze/conversation-review

On this page

Conversation Review allows you to review conversations users have had with your assistant. It helps you detect patterns in conversations that might explain why certain skills are not working well—whether due to bad design, limited scope, bad routing, or implementation error.

Configure Rasa assistant for Conversation Review

In order for Studio to display conversation data, you need to configure your Rasa assistant to send conversation data to Studio via a Kafka broker. Make sure that your assistant and Studio are both connected to same Kafka instance and are listening to the same topic. You can review the technical reference on [the event broker here](https://rasa.com/docs/reference/integrations/event-brokers/#kafka-event-broker) to know more about Kafka configuration in Rasa Pro. Make sure the that `assistant_id` field in the `config.yml` of your Rasa assistant is set to the same value as the assistant name in Studio.

## Getting started[​](https://rasa.com/docs/studio/analyze/conversation-review/#getting-started 'Direct link to Getting started')

Choose an assistant from the dropdown menu in the left navigation bar.

![Select an assistant](https://rasa.com/docs/assets/images/assistant-selection-895e16c03053eaf1537882e6f9bd629d.png)

Select Conversation Review in the left navigation bar.

![Conversation Review page](https://rasa.com/docs/assets/images/conversation-view-b75d2d7186797a7d2d6626161a73f7fc.png)

You will now see Conversation Review’s Conversations Table

![See all conversations in Conversation Review](https://rasa.com/docs/assets/images/conversation-list-100e77fc55fb44c80a766a1cf6b4ef70.png)

## Conversations Table[​](https://rasa.com/docs/studio/analyze/conversation-review/#conversations-table 'Direct link to Conversations Table')

The Conversations Table lists all conversations your assistant has had with your users. By default, the Conversations Table shows the 100 most recent conversations occurring within the last seven days, sorted by descending start time.

The Conversations Table has the following columns:

| Column Name | Description |
| --- | --- |
| Conversation ID | The unique ID associated with a user conversation. Also called a Session or User ID. |
| Session Start | The timestamp for the first event in the conversation session. Note that the session start time in the Conversations Table and the session start time elsewhere may differ: Studio auto-adjusts the default UTC timestamp to reflect the time in your local timezone. |
| Tags | Tags associated with the Conversation ID that you or your teammates have added. |
| Messages | The number of messages the user sent during the conversation. |
| Reviewed | When you or another user has viewed a conversation and marked it as reviewed, a green checkbox will appear in this column. |

### Filters[​](https://rasa.com/docs/studio/analyze/conversation-review/#filters 'Direct link to Filters')

Applying filters to the Conversations Table helps you quickly identify conversations that need analysis. To access filters, click the Filters icon in the top left corner of the Conversations Table.

![image](https://rasa.com/docs/assets/images/conversation-review-improvements-2-ca7e2d4693e31cc744708f98fbd0a84c.png)

The following filters are available:

| Column Name | Description |
| --- | --- |
| Timeframe | The timestamp for the first event in the conversation session |
| Conversation ID | The unique ID associated with a user conversation. |
| Predicted Intent | The intents predicted during the course of the conversation. |
| Flow (CALM assistants only) | The Flows started during the course of the conversation. |
| Tags | Tags associated with the Conversation ID. |
| Response | Assistant responses used during the course of the conversation. |
| Channel | Channel where the conversation occurred. |
| Language | Language in which the conversation happened. |
| Slot | Filter by any slot value used during the conversation. |
| UserID | Filter by user's unique identifier if present. |

## Conversation Details Panel[​](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-details-panel 'Direct link to Conversation Details Panel')

The Details panel shows session, user and event data that will help you with conversation evaluation. The panel shows the following session-level data for every conversation:

![View specific conversation details](https://rasa.com/docs/assets/images/cv-view-guide-eb1f64946c54988465f6a397cac7b523.png)

| Column Name | Description |
| --- | --- |
| Conversation ID | The unique ID assigned to the conversation you are viewing. |
| Session Start | The timestamp for the first event in the conversation session. |
| Model ID | The unique ID assigned to the model your assistant is using. |
| Channel | The channel where the conversation took place. |
| User Messages | The total number of user messages sent during the session. |
| Flows (CALM assistants only) | The Flows started during the course of the conversation. |
| Flows attributes | The attributes associated with the Flow. |
| Predicted intents (NLU assistants only) | The intents predicted during the course of the conversation. |
| Predicted entities (NLU assistants only) | The entities predicted during the course of the conversation. |
| Tags | Tags associated with the Conversation ID. |

### Managing Tags[​](https://rasa.com/docs/studio/analyze/conversation-review/#managing-tags 'Direct link to Managing Tags')

The Details section is also where you can manage tags for the conversation you’re reviewing. Scroll to the bottom of the details panel to add or remove tags associated with a conversation.

[Learn more about tags and how to use Tags.](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-tags)

## Conversation Stream[​](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-stream 'Direct link to Conversation Stream')

The Conversation Stream is a turn-by-turn breakdown of the conversation you selected for review. Clicking on messages, responses and certain event types will show you more information in the Details panel.

### User Messages[​](https://rasa.com/docs/studio/analyze/conversation-review/#user-messages 'Direct link to User Messages')

User messages are shown on the right hand side of the stream. Clicking on a message will show more details about the message in the Details panel:

- **Message timestamp**: The time at which the message was sent.
- **Message ID**: The unique ID of the user message.
- **Message in conversation**: The sequential number of the message in the conversation. ![image](https://rasa.com/docs/assets/images/cv-user-message-21a01fee93e9c8fdbde12fa35d79e30d.png)

If you are analyzing an NLU assistant, you will also see:

- Predicted intents with confidence scores

### Events[​](https://rasa.com/docs/studio/analyze/conversation-review/#events 'Direct link to Events')

Slot and Flow events are represented with a colored banner. Clicking on the banner reveals additional details about the events.

#### Slot events[​](https://rasa.com/docs/studio/analyze/conversation-review/#slot-events 'Direct link to Slot events')

Slots are your bot's memory. They store information a user has provided (for example, their name or phone number) or information the assistant can access about the user (for example, their account ID or device in use). The bot can then refer to this information throughout the conversation. When a slot is set during the course of a conversation, you will see a yellow banner. Clicking on the banner will show more details about the set slot in the Details panel:

- **Event timestamp**: The time at which the Slot event took place.
- **Event ID**: The unique ID associated with the event.
- **Event in Conversation**: The sequence of the event in the conversation.
- **Slot Name**: The name of the slot that was set.
- **Slot Value**: The information about the user or conversation that is now stored in the slot. ![image](https://rasa.com/docs/assets/images/cv-slot-event-c45dcce88c6f67ee3a1bcc1193249eb4.png)

#### Flow events (CALM assistants only)[​](https://rasa.com/docs/studio/analyze/conversation-review/#flow-events-calm-assistants-only 'Direct link to Flow events (CALM assistants only)')

CALM assistants are powered by Flows. Flows are abstract representations of the business processes your assistant completes. If you are analyzing a conversation with a CALM assistant, you will see the following Flow events:

- **Flow started**: The assistant started a Flow in response to a user question.
- **Flow interrupted**: The Flow stopped before completion. It will continue once the interrupted flow is finished.
- **Flow resumed**: The Flow restarted after stopping. Flow restart at the point where it left off.
- **Flow completed**: The Flow was completed within the session window.
- **Flow cancelled**: The user cancelled the Flow during the conversation session.

When a Flow event happens during the course of a conversation, you will see a blue banner. Clicking on the banner will show more details about the Flow event in the Details panel:

- **Timestamp of event**: The time at which the Flow event took place.
- **Event ID**: The unique ID associated with the event.
- **Event in Conversation**: The sequence of the event in the conversation.
- **Flow name**: The name of the Flow.
- **Flow event type**: The type of Flow event triggered. ![image](https://rasa.com/docs/assets/images/cv-slot-event-c45dcce88c6f67ee3a1bcc1193249eb4.png)

## Conversation Tags[​](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-tags 'Direct link to Conversation Tags')

Tags are a way for your team to supercharge your review workflows.

Imagine a scenario where you are analyzing a conversation between a user and a weather app assistant. During the conversation, the user requested information on the pollen index for their zip code. However, the assistant was designed to only answer questions directly related to weather events, and so the question triggered the assistant’s fallback behavior. With tags, you could:

- Batch this conversation with similar ones by assigning a tag like `out-of-scope` to denote the user asked a question outside of the assistant’s domain
- Batch this conversation with similar ones by assigning a tag like `topic-pollen-index` to denote the user asked a question related to the pollen index for their location
- Specify what action the team should take next by assigning a tag like `needs-further-analysis`, `implement-behavior`, or a even a knowledgeable teammate’s name. ![image](https://rasa.com/docs/assets/images/cv-tags-e59bba81426d6d2887de2acf24a4b6d6.png) These conversations can then be easily surfaced in the Conversations Table.

## Automatic Conversation Deletion[​](https://rasa.com/docs/studio/analyze/conversation-review/#automatic-conversation-deletion 'Direct link to Automatic Conversation Deletion')

To manage data retention and ensure compliance with data protection regulations, old conversations can be automatically deleted periodically. [Learn more about automatic conversation deletion](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/)

## Conversations via API[​](https://rasa.com/docs/studio/analyze/conversation-review/#conversations-via-api 'Direct link to Conversations via API')

For developers looking to programmatically tag and delete conversations, please refer to our [Conversations API Documentation](https://rasa.com/docs/reference/api/studio/conversation-api/).

- [Getting started](https://rasa.com/docs/studio/analyze/conversation-review/#getting-started)
- [Conversations Table](https://rasa.com/docs/studio/analyze/conversation-review/#conversations-table)
 - [Filters](https://rasa.com/docs/studio/analyze/conversation-review/#filters)
- [Conversation Details Panel](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-details-panel)
 - [Managing Tags](https://rasa.com/docs/studio/analyze/conversation-review/#managing-tags)
- [Conversation Stream](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-stream)
 - [User Messages](https://rasa.com/docs/studio/analyze/conversation-review/#user-messages)
 - [Events](https://rasa.com/docs/studio/analyze/conversation-review/#events)
- [Conversation Tags](https://rasa.com/docs/studio/analyze/conversation-review/#conversation-tags)
- [Automatic Conversation Deletion](https://rasa.com/docs/studio/analyze/conversation-review/#automatic-conversation-deletion)
- [Conversations via API](https://rasa.com/docs/studio/analyze/conversation-review/#conversations-via-api)