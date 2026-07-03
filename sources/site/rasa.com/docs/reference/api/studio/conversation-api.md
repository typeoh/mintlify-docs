# Source: https://rasa.com/docs/reference/api/studio/conversation-api

On this page

The Conversation API allows external access to retrieve, delete, and tag conversations within Rasa Studio.

## Requirements[​](https://rasa.com/docs/reference/api/studio/conversation-api/#requirements 'Direct link to Requirements')

- Required API Role `Manage conversations` (or `View conversations` for read-only operations). See [Authorization page](https://rasa.com/docs/studio/security/authorization/#creating-a-new-client-id) to learn how to access configuration.
 
- Assistant name. See [guide for user configuration](https://rasa.com/docs/studio/build/set-up-your-assistant/) for more details.
 
 ![image](https://rasa.com/docs/assets/images/conversation-api-assistant-name-b20623b4201e2debc91bbd5a21baf83d.png)
 
- Tag IDs. To see how to access page from screenshot see [guide for conversation review](https://rasa.com/docs/studio/analyze/conversation-review/).
 
 ![image](https://rasa.com/docs/assets/images/conversation-api-tag-id-9b5aa6ebe3ec628ba8bb350793667d0d.png)
 

## Quick Reference[​](https://rasa.com/docs/reference/api/studio/conversation-api/#quick-reference 'Direct link to Quick Reference')

| Type | Operation | Description |
| --- | --- | --- |
| Query | [`rawConversations`](https://rasa.com/docs/reference/api/studio/conversation-api/#rawconversations) | Retrieve conversations with optional filters |
| Query | [`conversationTags`](https://rasa.com/docs/reference/api/studio/conversation-api/#conversationtags) | List all tags for an assistant |
| Mutation | [`addConversationTagToConversation`](https://rasa.com/docs/reference/api/studio/conversation-api/#addconversationtagtoconversation) | Add a tag to a conversation |
| Mutation | [`removeConversationTagFromConversation`](https://rasa.com/docs/reference/api/studio/conversation-api/#removeconversationtagfromconversation) | Remove a tag from a conversation |
| Mutation | [`deleteConversations`](https://rasa.com/docs/reference/api/studio/conversation-api/#deleteconversations) | Delete one or more conversations |

---

## Queries[​](https://rasa.com/docs/reference/api/studio/conversation-api/#queries 'Direct link to Queries')

### `rawConversations`[​](https://rasa.com/docs/reference/api/studio/conversation-api/#rawconversations 'Direct link to rawconversations')

Retrieves conversations for an assistant with optional filtering by time range and tags.

**Schema:**

```
type Query {  rawConversations(input: RawConversationsInput!): RawConversationsOutput!}input RawConversationsInput {  assistantName: String!  fromTime: String  toTime: String  limit: Int  offset: Int  tags: [String!]}type RawConversation {  id: ID!  conversationEvents: [JSON!]!  tags: [ConversationTag!]!}type RawConversationsOutput {  conversations: [RawConversation!]!  count: Int!}
```

**Input: `RawConversationsInput`**

| Field | Description |
| --- | --- |
| `assistantName` | The unique identifier of the assistant. |
| `fromTime` | Optional start time filter in ISO 8601 UTC format (`YYYY-MM-DDTHH:MM:SS.sssZ`). |
| `toTime` | Optional end time filter in ISO 8601 UTC format (`YYYY-MM-DDTHH:MM:SS.sssZ`). |
| `limit` | Limits the number of conversations returned. Default is `10`. |
| `offset` | Skips a specified number of conversations for pagination. Default is `0`. |
| `tags` | Optional list of tag names to filter conversations by. |

**Output: `RawConversationsOutput`**

| Field | Description |
| --- | --- |
| `conversations` | A list of raw conversation objects. Each conversation contains an `id`, a list of `conversationEvents` in JSON format, and a list of `tags`. |
| `count` | Total count of conversations matching the query criteria. Not affected by `limit` and `offset`. |

**Example:**

```
curl -X POST https://<your-api-url>/api/graphql \-H "Content-Type: application/json" \-H "Authorization: Bearer <your-access-token>" \-d '{"query": "query rawConversations($input: RawConversationsInput!) { rawConversations(input: $input) { conversations { id, conversationEvents, tags { id, name } }, count } }", "variables": { "input": { "assistantName": "assistant-name", "fromTime": "2024-01-01T00:00:00Z", "toTime": "2024-01-31T23:59:59Z", "limit": 10, "offset": 0, "tags": ["example-tag"] } }}'
```

---

### `conversationTags`[​](https://rasa.com/docs/reference/api/studio/conversation-api/#conversationtags 'Direct link to conversationtags')

Retrieves all conversation tags available for a given assistant.

**Schema:**

```
type Query {  conversationTags(input: ConversationTagsInput!): [ConversationTag!]!}input ConversationTagsInput {  assistantName: String!}type ConversationTag {  id: ID!  name: String!  assistantId: ID!}
```

**Input: `ConversationTagsInput`**

| Field | Description |
| --- | --- |
| `assistantName` | The unique identifier of the assistant. |

**Output: `[ConversationTag!]!`**

Returns a list of conversation tags:

| Field | Description |
| --- | --- |
| `id` | The unique identifier of the tag. |
| `name` | The display name of the tag. |
| `assistantId` | The unique identifier of the assistant. |

**Example:**

```
curl -X POST https://<your-api-url>/api/graphql \-H "Content-Type: application/json" \-H "Authorization: Bearer <your-access-token>" \-d '{"query": "query conversationTags($input: ConversationTagsInput!) { conversationTags(input: $input) { id, name, assistantId } }", "variables": { "input": { "assistantName": "assistant-name" } }}'
```

---

## Mutations[​](https://rasa.com/docs/reference/api/studio/conversation-api/#mutations 'Direct link to Mutations')

### `addConversationTagToConversation`[​](https://rasa.com/docs/reference/api/studio/conversation-api/#addconversationtagtoconversation 'Direct link to addconversationtagtoconversation')

Adds a tag to a conversation.

**Schema:**

```
type Mutation {  addConversationTagToConversation(    input: AddConversationTagToConversationInput!  ): Boolean!}input AddConversationTagToConversationInput {  conversationId: ID!  tagId: ID!}
```

**Input: `AddConversationTagToConversationInput`**

| Field | Description |
| --- | --- |
| `conversationId` | The unique identifier of the conversation to which the tag will be added. |
| `tagId` | The unique identifier of the tag to be added to the conversation. |

**Output:** Returns `true` if the tag was successfully added.

note

If the tag already exists on the conversation, the endpoint will also return `true`. If a non-existent conversation, tag, or resource mismatch occurs, a descriptive error message with an appropriate error code is returned.

**Example:**

```
curl -X POST https://<your-api-url>/api/graphql \-H "Content-Type: application/json" \-H "Authorization: Bearer <your-access-token>" \-d '{"query": "mutation addConversationTagToConversation($input: AddConversationTagToConversationInput!) { addConversationTagToConversation(input: $input) }", "variables": { "input": { "conversationId": "12345", "tagId": "67890" } }}'
```

---

### `removeConversationTagFromConversation`[​](https://rasa.com/docs/reference/api/studio/conversation-api/#removeconversationtagfromconversation 'Direct link to removeconversationtagfromconversation')

Removes a tag from a conversation.

**Schema:**

```
type Mutation {  removeConversationTagFromConversation(    input: RemoveConversationTagFromConversationInput!  ): Boolean!}input RemoveConversationTagFromConversationInput {  conversationId: ID!  tagId: ID!}
```

**Input: `RemoveConversationTagFromConversationInput`**

| Field | Description |
| --- | --- |
| `conversationId` | Unique identifier of the conversation from which the tag should be removed. |
| `tagId` | Unique identifier of the tag to be removed from the conversation. |

**Output:** Returns `true` if the tag was successfully removed.

note

If the provided `tagId` is not linked to the `conversationId`, or if the provided `conversationId` or `tagId` is not present in the system, the endpoint will still return `true`.

**Example:**

```
curl -X POST https://<your-api-url>/api/graphql \-H "Content-Type: application/json" \-H "Authorization: Bearer <your-access-token>" \-d '{"query": "mutation removeConversationTagFromConversation($input: RemoveConversationTagFromConversationInput!) { removeConversationTagFromConversation(input: $input) }", "variables": { "input": { "conversationId": "12345", "tagId": "67890" } }}'
```

---

### `deleteConversations`[​](https://rasa.com/docs/reference/api/studio/conversation-api/#deleteconversations 'Direct link to deleteconversations')

Deletes one or more conversations.

**Schema:**

```
type Mutation {  deleteConversations(input: DeleteConversationsInput!): Int!}input DeleteConversationsInput {  conversationIds: [ID!]!}
```

**Input: `DeleteConversationsInput`**

| Field | Description |
| --- | --- |
| `conversationIds` | List of conversation IDs to delete. |

**Output:** Returns the count of deleted conversations.

note

The count only includes conversations that were actually deleted. If all IDs passed to the API were invalid, count will be zero.

**Example:**

```
curl -X POST https://<your-api-url>/api/graphql \-H "Content-Type: application/json" \-H "Authorization: Bearer <your-access-token>" \-d '{"query": "mutation deleteConversations($input: DeleteConversationsInput!) { deleteConversations(input: $input) }", "variables": { "input": { "conversationIds": ["12345"] } }}'
```

---

## Reference[​](https://rasa.com/docs/reference/api/studio/conversation-api/#reference 'Direct link to Reference')

### Conversation Events Structure[​](https://rasa.com/docs/reference/api/studio/conversation-api/#conversation-events-structure 'Direct link to Conversation Events Structure')

The `conversationEvents` field in `rawConversations` contains a list of events that occur within a conversation. These events follow a specific structure that varies by event type. For a full description of all possible events, see the [Rasa Events Documentation](https://rasa.com/docs/reference/primitives/events/).

#### User Uttered Event[​](https://rasa.com/docs/reference/api/studio/conversation-api/#user-uttered-event 'Direct link to User Uttered Event')

```
{  "sender_id": "f9e5db46255c42e8aefd7fce720d192f",  "event": "user",  "timestamp": 1692186822.4774666,  "metadata": {    "model_id": "003d55f2a67147a597cec7e1df3e96e9",    "assistant_id": "rich-response-rasa"  },  "text": "/mood_great",  "parse_data": {    "intent": {      "name": "mood_great",      "confidence": 1    },    "entities": [],    "text": "/mood_great",    "message_id": "7457d2d8174243409e39b165164c79c7",    "metadata": {},    "intent_ranking": [      {        "name": "mood_great",        "confidence": 1      }    ]  },  "input_channel": "cmdline",  "message_id": "7457d2d8174243409e39b165164c79c7"}
```

#### Bot Uttered Event[​](https://rasa.com/docs/reference/api/studio/conversation-api/#bot-uttered-event 'Direct link to Bot Uttered Event')

```
{  "sender_id": "f9e5db46255c42e8aefd7fce720d192f",  "event": "bot",  "timestamp": 1692186813.6298656,  "metadata": {    "utter_action": "utter_greet",    "model_id": "003d55f2a67147a597cec7e1df3e96e9",    "assistant_id": "rich-response-rasa"  },  "text": "Hey! How are you?",  "data": {    "elements": null,    "quick_replies": [      {        "title": "great",        "payload": "/mood_great"      },      {        "title": "super sad",        "payload": "/mood_unhappy",        "image_url": "https://i.imgur.com/l4pF9Fb.jpg"      }    ],    "buttons": [      {        "title": "great",        "payload": "/mood_great"      },      {        "title": "super sad",        "payload": "/mood_unhappy",        "image_url": "https://i.imgur.com/l4pF9Fb.jpg"      }    ],    "attachment": null,    "image": null,    "custom": null  }}
```

#### Action Executed Event[​](https://rasa.com/docs/reference/api/studio/conversation-api/#action-executed-event 'Direct link to Action Executed Event')

```
{  "sender_id": "f9e5db46255c42e8aefd7fce720d192f",  "event": "action",  "timestamp": 1692186813.2051082,  "metadata": {    "model_id": "003d55f2a67147a597cec7e1df3e96e9",    "assistant_id": "rich-response-rasa"  },  "name": "action_session_start",  "policy": null,  "confidence": 1,  "action_text": null,  "hide_rule_turn": false}
```

- [Requirements](https://rasa.com/docs/reference/api/studio/conversation-api/#requirements)
- [Quick Reference](https://rasa.com/docs/reference/api/studio/conversation-api/#quick-reference)
- [Queries](https://rasa.com/docs/reference/api/studio/conversation-api/#queries)
 - [`rawConversations`](https://rasa.com/docs/reference/api/studio/conversation-api/#rawconversations)
 - [`conversationTags`](https://rasa.com/docs/reference/api/studio/conversation-api/#conversationtags)
- [Mutations](https://rasa.com/docs/reference/api/studio/conversation-api/#mutations)
 - [`addConversationTagToConversation`](https://rasa.com/docs/reference/api/studio/conversation-api/#addconversationtagtoconversation)
 - [`removeConversationTagFromConversation`](https://rasa.com/docs/reference/api/studio/conversation-api/#removeconversationtagfromconversation)
 - [`deleteConversations`](https://rasa.com/docs/reference/api/studio/conversation-api/#deleteconversations)
- [Reference](https://rasa.com/docs/reference/api/studio/conversation-api/#reference)
 - [Conversation Events Structure](https://rasa.com/docs/reference/api/studio/conversation-api/#conversation-events-structure)