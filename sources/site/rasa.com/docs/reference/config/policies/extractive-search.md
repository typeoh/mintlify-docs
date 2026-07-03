# Source: https://rasa.com/docs/reference/config/policies/extractive-search

On this page

New Beta feature in 3.9

Rasa now supports using [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) without an additional call to LLMs for response generation. This feature is only a beta (experimental) and will change in future Rasa versions.

Extractive Search allows you to disable LLM response generation and use your questions and answer dataset to provide answers to user queries. This feature is useful when you want to provide answers to user queries from a predefined dataset without using LLMs for chat response generation. These dataset could look as follows:

```
Q: Who is Finley?A: Finley is your smart assistant for the FinX App. You can add him to your favoritemessenger and tell him what you need help with.Q: How does Finley work?A: Finley is powered by the latest chatbot technology leveraging a unique interplay oflarge language models and secure logic.
```

As the dataset already contains answers, LLM response generation is no longer needed and can be turned off. Extractive Search retrieves the most similar Question and Answer pair from the dataset.

## How Extractive Search Works[​](https://rasa.com/docs/reference/config/policies/extractive-search/#how-extractive-search-works 'Direct link to How Extractive Search Works')

Extractive Search requires documents to be ingested into a specific format so that answers can be reliably extracted by Rasa. These questions should be added as follows in the vector store:

```
[    {        "metadata": {            "title": "who_finley",            "type": "faq",            "answer": "Finley is your smart assistant for the FinX App. You can add him to your favorite messenger and tell him what you need help with.",        },        "page_content": "Who is Finley?",    },    {        "metadata": {            "title": "how_finley_work",            "type": "faq",            "answer": "Finley is powered by the latest chatbot technology leveraging a unique interplay of large language models and secure logic.",        },        "page_content": "How does Finley work?"    },]
```

This format ensures that when users ask for the query “Who is finley?”, the retrieval returns `who_finley` as the first result and Rasa can reliably extract the answer from `metadata.answer` key. Explanations for all keys:

- `metadata`: this is a mandatory field required by Enterprise Search
- `metadata.title`: \[optional\] could be useful as an ID field to refer to the QnA pair
- `metadata.answer`: contains text or markdown used to create the response that is shown to the user.
- `metadata.type`: optional field, it is useful to filter relevant documents if the knowledge base contains other things too.
- `page_content`: contains the text Question from QnA pair, only this field is vectorised by the embedding model. Any search query `q` will be compared for similarity with this field in the payload.

note

Extractive Search should be used together with `vector_store.threshold` so that only the high-confidence search results are used to respond to the user.

## Extractive Search Configuration[​](https://rasa.com/docs/reference/config/policies/extractive-search/#extractive-search-configuration 'Direct link to Extractive Search Configuration')

To configure [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) to use Extractive Search, simply set `use_generative_llm` to `false` in the Assistant’s config.yml

config.yml

```
policies:...- name: EnterpriseSearchPolicy  use_generative_llm: false
```

With this configuration, [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) returns the first search result to the chat without generating an answer with an LLM.

You can also connect to different search services using [Custom Information Retrievers](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/) while using Extractive Search.

- [How Extractive Search Works](https://rasa.com/docs/reference/config/policies/extractive-search/#how-extractive-search-works)
- [Extractive Search Configuration](https://rasa.com/docs/reference/config/policies/extractive-search/#extractive-search-configuration)