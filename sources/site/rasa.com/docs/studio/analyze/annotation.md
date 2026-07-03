# Source: https://rasa.com/docs/studio/analyze/annotation

On this page

This guide teaches you how to annotate intents and entities in Rasa Studio, helping your assistant better understand user inputs by creating high-quality training data. You’ll learn how to assign, review, and manage annotations effectively to ensure your assistant performs accurately and reliably.

note

This feature requires that you've integrated Live Conversation Review. To enable Studio to visualize conversation data, your Rasa assistant must be configured to transmit this data to Studio using a Kafka broker. Ensure that both your Rasa Pro assistant and Studio are connected to the same Kafka instance and monitoring the same topic.

Additionally, verify that the `assistant_id` field in your Rasa assistant’s `config.yml` file matches the assistant name defined in Studio.

To dive deeper into the event broker, check out [configuring Kafka in Rasa](https://rasa.com/docs/reference/integrations/event-brokers/#kafka-event-broker).

## What is Annotation in Studio?[​](https://rasa.com/docs/studio/analyze/annotation/#what-is-annotation-in-studio 'Direct link to What is Annotation in Studio?')

Annotation in Rasa Studio allows your team to label user messages with intents and entities. Users that have the [Lead Annotator](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#roles-overview) role can assign tasks, review annotations, and approve them to build training data or export it for further use.

## Intro to the Annotation Dashboard[​](https://rasa.com/docs/studio/analyze/annotation/#intro-to-the-annotation-dashboard 'Direct link to Intro to the Annotation Dashboard')

1. Open the **Annotation Dashboard** under **Bot Tuner** in the navigation bar.
 
 ![image](https://rasa.com/docs/assets/images/Screenshot_2023-06-23_at_16.31.29-478415f9e2576f6bd08ffd4b1f7e9855.png)
 
2. View key data:
 
 - **Total Utterances**: All user messages sent to the assistant.
 - **Total Conversations**: All sessions initiated by users.
 
 ![image](https://rasa.com/docs/assets/images/CleanShot_2023-06-26_at_11.06.192x-3f366ddf443a364e47a11bbe94e93ef0.png)
 

## How to Assign Annotations[​](https://rasa.com/docs/studio/analyze/annotation/#how-to-assign-annotations 'Direct link to How to Assign Annotations')

1. Click **Assign Annotation** in the Annotation Dashboard.
 
2. Select annotators from the list. You can include yourself.
 
 ![image](https://rasa.com/docs/assets/images/CleanShot_2023-06-26_at_11.08.112x-8c52fb03ea69db83a76e120f8fa8f80d.png)
 
3. Use filters to define the messages for annotation:
 
 - **Date Range**: Filter by when messages were sent.
 - **Channels**: Choose specific communication channels.
 - **Keywords**: Filter messages containing specific words.
 - **Predicted Intent**: Include messages with specific predicted intents.
 - **Confidence Range**: Set a confidence score range for predictions.
 - **Sample Size**: Limit the number of messages in the batch.
4. Click the refresh icon to update the message count based on filters.
 
5. Click **Assign Annotation** to create the batch.
 

## How to Annotate Messages[​](https://rasa.com/docs/studio/analyze/annotation/#how-to-annotate-messages 'Direct link to How to Annotate Messages')

### Accessing Batches[​](https://rasa.com/docs/studio/analyze/annotation/#accessing-batches 'Direct link to Accessing Batches')

1. Go to **Annotation Inbox** under **Bot Tuner** to see batches assigned to you.
2. Click **Annotate** on a batch to begin.

### Bulk Annotation[​](https://rasa.com/docs/studio/analyze/annotation/#bulk-annotation 'Direct link to Bulk Annotation')

- **View Messages**: See a list of messages with predicted intents and confidence scores.
- **Tabs**:
 - **Inbox**: Messages awaiting annotation.
 - **Saved**: Annotated and saved messages.
 - **Discarded**: Messages excluded from training.

#### Annotating Intents[​](https://rasa.com/docs/studio/analyze/annotation/#annotating-intents 'Direct link to Annotating Intents')

1. Select messages and choose an intent from the dropdown.
2. Save annotations by clicking **Annotate**. Annotated messages move to the "Saved" tab.
3. Discard irrelevant messages to exclude them from training.

#### Annotating Entities[​](https://rasa.com/docs/studio/analyze/annotation/#annotating-entities 'Direct link to Annotating Entities')

1. Highlight words in the message to tag them as entities.
2. Assign an entity type, role, or synonym in the panel below the message.
3. Confirm or discard predicted entities.

### Single Annotation[​](https://rasa.com/docs/studio/analyze/annotation/#single-annotation 'Direct link to Single Annotation')

- Switch to single annotation mode for detailed review.
- See the full conversation context and annotate messages individually.

## Reviewing Annotations[​](https://rasa.com/docs/studio/analyze/annotation/#reviewing-annotations 'Direct link to Reviewing Annotations')

1. Open the **Review Inbox** to review batches.
2. Apply filters to focus on specific messages.
3. Approve annotations or make corrections as needed.
4. Mark the batch as reviewed to finalize annotations.

## Managing Annotation History[​](https://rasa.com/docs/studio/analyze/annotation/#managing-annotation-history 'Direct link to Managing Annotation History')

1. Access **Annotation History** to view approved annotations.
 
2. Filter by confidence scores, intents, or training data status.
 
3. Export annotations as a CSV or add them as training data.
 
4. Train your model to incorporate new training data.
 
 💡 Remember to retrain your model after adding new training data.
 

## Linked Features[​](https://rasa.com/docs/studio/analyze/annotation/#linked-features 'Direct link to Linked Features')

### Creating a New Intent During Annotation[​](https://rasa.com/docs/studio/analyze/annotation/#creating-a-new-intent-during-annotation 'Direct link to Creating a New Intent During Annotation')

1. Click the intent dropdown and select **Create New Intent**.
2. Enter a name for the intent (no spaces or special characters).
3. Save the intent to use it immediately in the batch.

By following these steps, you can streamline the annotation process, ensuring accurate training data for your assistant.

- [What is Annotation in Studio?](https://rasa.com/docs/studio/analyze/annotation/#what-is-annotation-in-studio)
- [Intro to the Annotation Dashboard](https://rasa.com/docs/studio/analyze/annotation/#intro-to-the-annotation-dashboard)
- [How to Assign Annotations](https://rasa.com/docs/studio/analyze/annotation/#how-to-assign-annotations)
- [How to Annotate Messages](https://rasa.com/docs/studio/analyze/annotation/#how-to-annotate-messages)
 - [Accessing Batches](https://rasa.com/docs/studio/analyze/annotation/#accessing-batches)
 - [Bulk Annotation](https://rasa.com/docs/studio/analyze/annotation/#bulk-annotation)
 - [Single Annotation](https://rasa.com/docs/studio/analyze/annotation/#single-annotation)
- [Reviewing Annotations](https://rasa.com/docs/studio/analyze/annotation/#reviewing-annotations)
- [Managing Annotation History](https://rasa.com/docs/studio/analyze/annotation/#managing-annotation-history)
- [Linked Features](https://rasa.com/docs/studio/analyze/annotation/#linked-features)
 - [Creating a New Intent During Annotation](https://rasa.com/docs/studio/analyze/annotation/#creating-a-new-intent-during-annotation)