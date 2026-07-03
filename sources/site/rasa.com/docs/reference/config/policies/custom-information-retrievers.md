# Source: https://rasa.com/docs/reference/config/policies/custom-information-retrievers

On this page

New in 3.9

Rasa now supports Custom Information Retrievers to be used with the [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/). This feature allows you to integrate your own custom search systems or vector stores with Rasa.

## Introduction[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#introduction 'Direct link to Introduction')

Rasa's initial integration with vector stores, such as Qdrant and Milvus, laid the foundation for more advanced information retrieval capabilities. We recognized the need to provide a more flexible and extensible interface, leading to the creation of the `InformationRetrieval` interface. This interface is a superset of vector stores and encompasses a broader range of information retrieval techniques.

The `InformationRetrieval` interface enables you to integrate not only vector stores but also custom search systems, databases, or any other mechanism for retrieving relevant information. This flexibility empowers you to customize and optimize your information retrieval strategies based on your specific use cases and requirements.

## Creating a Custom Information Retrieval Class[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#creating-a-custom-information-retrieval-class 'Direct link to Creating a Custom Information Retrieval Class')

You can implement your own custom information retrieval component as a python class. A custom information retrieval class must subclass `rasa.core.information_retrieval.InformationRetrieval` and implement `connect` and a `search` methods.

```
from rasa.utils.endpoints import EndpointConfigfrom rasa.core.information_retrieval import SearchResultList, InformationRetrievalclass MyVectorStore(InformationRetrieval):    def connect(self, config: EndpointConfig) -> None:        # Create a connection to the search system        pass    async def search(        self, query: Text, tracker_state: dict[Text, Any], threshold: float = 0.0    ) -> SearchResultList:        # Implement the search functionality to retrieve relevant results based on the query and top_n parameter.        pass
```

You can access the embeddings model defined in the Rasa configuration with `self.embeddings` as a [`langchain.schema.embeddings.Embeddings`](https://github.com/langchain-ai/langchain/blob/v0.0.329/libs/langchain/langchain/schema/embeddings.py) object.

### `connect` method[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#connect-method 'Direct link to connect-method')

The `connect` method establishes a connection to the information retrieval system. It expects one parameter.

- `config`: This is the endpoint configuration for the information retrieval component. `config.kwargs` is a python dictionary that can be used to access keys defined in `endpoints.yml` under `vector_store`.

For example:

endpoints.yml

```
vector_store:  api_key: <api_key> # user defined key  collection: rasa   # user defined key
```

### `search` method[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#search-method 'Direct link to search-method')

The `search` method queries the information retrieval system for a document and returns a `SearchResultList` object which is a list of documents that match the query.

The method expects the following parameters:

- `query`: The query string. Typically the last user message
- `tracker_state`: The current tracker state as a dictionary.
- `threshold`: The minimum similarity score to consider a document a match.

### Tracker State[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#tracker-state 'Direct link to Tracker State')

Tracker State python dictionary is a snapshot of Rasa Tracker and contains metadata about the rasa conversation. It can be used to get information about any conversation event. It has the following schema,

```
{    "sender_id": "string",    "slots": {        "additionalProp1": "string",        "additionalProp2": "string",        "additionalProp3": "string"    },    "latest_message": {        "intent": {        "name": "string",        "confidence": 0        },        "entities": [        {}        ],        "text": "string",        "message_id": "string",        "metadata": {},        "commands": [        {            "command": "string"        }        ],        "flows_from_semantic_search": [        [            "string",            0        ]        ],        "flows_in_prompt": [        "string"        ]    },    "latest_event_time": 0,    "followup_action": "string",    "paused": true,    "stack": [        {}    ],    "events": [        {        "event": "string",        "timestamp": 0,        "metadata": {},        "name": "string",        "policy": "string",        "confidence": 0,        "action_text": "string",        "hide_rule_turn": true,        "value": {}        }    ],    "latest_input_channel": "string",    "active_loop": {},    "latest_action": {        "action_name": "string"    },    "latest_action_name": "string",    "user_id": "string",    "conversation_started_timestamp": "float"}
```

Tracker State can be helpful for search systems to enable further customizations for your assistant. For example, this python function creates a chat history from tracker\_state object,

```
def get_chat_history(tracker_state: dict[str, Any]) -> dict[str, Any]:    chat_history = []    last_user_message = ""    for event in tracker_state.get("events"):        if event.get("event") == "user":            last_user_message = sanitize_message_for_prompt(event.get("text"))            chat_history.append({"role": "USER", "message": last_user_message})        elif event.get("event") == "bot":            chat_history.append({"role": "CHATBOT", "message": event.get("text")})    return chat_history
```

Slots currently active in the conversation can be accessed with `tracker_state.get("slots", {}).get(SLOT_NAME)`

### SearchResultList dataclass[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#searchresultlist-dataclass 'Direct link to SearchResultList dataclass')

SearchResultList dataclass is defined with SearchResult dataclass. Both of these dataclasses are defined as,

```
@dataclassclass SearchResult:    text: str    metadata: dict    score: Optional[float] = None@dataclassclass SearchResultList:    results: List[SearchResult]    metadata: dict
```

You can use the class method `SearchResultList.from_document_list()` to convert from a [Langchain Document object](https://python.langchain.com/v0.2/docs/integrations/document_loaders/copypaste/) type.

## Using the Custom Information Retrieval component[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#using-the-custom-information-retrieval-component 'Direct link to Using the Custom Information Retrieval component')

To configure [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) to use the custom component; set the `vector_store.type` parameter in the `config.yml` file to the module path of the custom information retrieval class.

For example, for a custom information retrieval class called `MyVectorStore` saved in a file `addons/custom_information_retrieval.py`, the module path would be `addons.custom_information_retrieval.MyVectorStore`, and the credentials could look like:

config.yml

```
policies:  - ...  - name: EnterpriseSearchPolicy    vector_store:      type: "addons.custom_information_retrieval.MyVectorStore"
```

## Usage Ideas[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#usage-ideas 'Direct link to Usage Ideas')

The Custom Information Retrieval feature opens up a range of possibilities for customizing and enhancing your assistant. Here are some potential use cases:

### Traditional Search, Vector Search or Rerankers[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#traditional-search-vector-search-or-rerankers 'Direct link to Traditional Search, Vector Search or Rerankers')

You have the flexibility to connect to any search system, whether it's a traditional BM25-based search engine, an open-source Elasticsearch instance, state-of-the-art vector stores, or even your favorite re-ranking models. This freedom allows you to stay at the forefront of IR innovations and leverage the latest advancements to enhance your conversational assistant.

### Slot-Based Retrieval[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#slot-based-retrieval 'Direct link to Slot-Based Retrieval')

With custom information retrieval, you can leverage conversation context and slots provided by the `tracker_state`. This enables slot-based retrieval, allowing you to filter search results based on the values of specific slots. For example, you could retrieve product recommendations based on the user's previously mentioned preferences or interests.

You could also add filters to the search systems based on a slot that defines user's access level.

### Customizing Search Queries[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#customizing-search-queries 'Direct link to Customizing Search Queries')

Assistant developers can customize or rephrase the search query based on the available conversation context. With custom information retrieval, you have the flexibility to incorporate additional context or modify the query to better match the user's intent.

Rephrasing can be useful for improving search accuracy or adapting queries to the specific requirements of your custom system.

### Support for Additional Embedding Models[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#support-for-additional-embedding-models 'Direct link to Support for Additional Embedding Models')

If Rasa doesn't natively support a particular embedding model that you want to use, custom information retrieval comes to the rescue. You can integrate local or fine-tuned embedding models of your choice to generate embeddings for search queries and documents.

### Library Flexibility[​](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#library-flexibility 'Direct link to Library Flexibility')

You have the freedom to use other libraries or tools of your choice for information retrieval tasks. This allows you to leverage specialized libraries or in-house tools that your organization might already be using.

These use cases highlight the versatility of custom information retrieval, empowering you to tailor your information retrieval strategies to match your specific needs and enhance the overall user experience.

- [Introduction](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#introduction)
- [Creating a Custom Information Retrieval Class](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#creating-a-custom-information-retrieval-class)
 - [`connect` method](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#connect-method)
 - [`search` method](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#search-method)
 - [Tracker State](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#tracker-state)
 - [SearchResultList dataclass](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#searchresultlist-dataclass)
- [Using the Custom Information Retrieval component](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#using-the-custom-information-retrieval-component)
- [Usage Ideas](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#usage-ideas)
 - [Traditional Search, Vector Search or Rerankers](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#traditional-search-vector-search-or-rerankers)
 - [Slot-Based Retrieval](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#slot-based-retrieval)
 - [Customizing Search Queries](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#customizing-search-queries)
 - [Support for Additional Embedding Models](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#support-for-additional-embedding-models)
 - [Library Flexibility](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/#library-flexibility)