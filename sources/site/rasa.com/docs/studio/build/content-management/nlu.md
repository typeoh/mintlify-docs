# Source: https://rasa.com/docs/studio/build/content-management/nlu

On this page

You can create your own Intents and Entities to use as triggers in your Rasa Assistant.

Important Note

Using intents in your Rasa assistant is completely optional. The recommended approach for building assistants is to use a [CALM approach](https://rasa.com/docs/learn/concepts/calm/) which leverages Large Language Models (LLMs) to trigger conversation flows dynamically, without relying heavily on pre-defined intents. This approach often leads to more flexible and scalable assistants, especially in complex use cases.

## Intents in Studio[​](https://rasa.com/docs/studio/build/content-management/nlu/#intents-in-studio 'Direct link to Intents in Studio')

Managing intents in Studio is an optional way to use NLU within your Rasa assistant. To add a new intent in your assistant:

### What is a Intent?

An intent represents the goal or purpose behind a user’s message. For example, a user might express the intent to book a ticket, ask for a weather update, or say hello. Intents help the assistant determine what the user wants to achieve.

### Create an Intent[​](https://rasa.com/docs/studio/build/content-management/nlu/#create-an-intent 'Direct link to Create an Intent')

1. **Create Intents:** Click the Create intent button at the top right.
2. **Mark Ready for Training:** Toggle the Ready for Training checkbox to remove the `Under Development` label and include the intent in training.
3. **Edit Intents:** Click the Edit button to rename an intent.
4. **Delete Intents:** Click the Delete button to remove an intent. Before deleting, make sure all associated examples and annotations are moved or deleted.

## Conversation Examples[​](https://rasa.com/docs/studio/build/content-management/nlu/#conversation-examples 'Direct link to Conversation Examples')

Examples in Rasa Studio are real or simulated user inputs used to train your assistant to recognize and respond to intents.

You can annotate these examples directly within Studio to tag relevant entities or refine intent classification. Additionally, you can upload annotated real conversations to further enhance your assistant's ability to identify intents accurately, ensuring a better match to real-world scenarios.

### Create a New Example[​](https://rasa.com/docs/studio/build/content-management/nlu/#create-a-new-example 'Direct link to Create a New Example')

1. **Add Examples**: Use the Add an example text field under the intent.
2. **Delete Examples**: Select examples and click Delete.
3. **Reclassify Examples**: Assign examples to a different intent using Classify to another intent.
4. **Edit Example Text**: Click the edit icon next to an example to update its text.
5. **Annotate Entities**: Highlight words in examples to tag them as entities. Use the to open the annotation panel and specify the entity type, role, or synonym.

### What is a Entity?

An entity is a specific piece of information extracted from a user’s message. They provide additional context to the intent. For example, in the message "Book a flight to Paris," the intent might be "book\_flight," and the entity would be "Paris" (destination)

## Best Practices[​](https://rasa.com/docs/studio/build/content-management/nlu/#best-practices 'Direct link to Best Practices')

1. **Provide Clear Intent Names**: Use descriptive, distinct names for each intent.
2. **Use Diverse Examples**: Provide varied examples for better training.
3. **Be Consistent with Annotation**: Accurately annotate entities to enhance model performance.
4. **Keep your NLU Data Up to Date**: Periodically review and refine your NLU data.

For more details on our recommendations for getting the most out of your assistant, see our [Best Practices for Great Conversations](https://rasa.com/docs/learn/best-practices/conversation-design/).

- [Intents in Studio](https://rasa.com/docs/studio/build/content-management/nlu/#intents-in-studio)
 - [Create an Intent](https://rasa.com/docs/studio/build/content-management/nlu/#create-an-intent)
- [Conversation Examples](https://rasa.com/docs/studio/build/content-management/nlu/#conversation-examples)
 - [Create a New Example](https://rasa.com/docs/studio/build/content-management/nlu/#create-a-new-example)
- [Best Practices](https://rasa.com/docs/studio/build/content-management/nlu/#best-practices)