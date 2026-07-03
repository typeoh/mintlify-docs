# Source: https://rasa.com/docs/pro/customize/enterprise-search

On this page

Enterprise Search provides a way to manage informational user queries by integrating your assistant with your information retrieval pipelines. When a user asks a question requiring external knowledge—e.g., product information, support manuals, or FAQs—Enterprise Search enables your assistant to look up relevant documents from a vector store or other data repository.

With Enterprise Search, you can:

- Retrieve relevant content from your own knowledge base or documents.
- Provide the content to a Large Language Model (LLM) for answer generation, or choose an extractive approach (i.e., respond with the exact text from your repository, rather than an LLM-generated rephrasing).
- Integrate advanced retrieval patterns (e.g., slot-based search filters or re-ranking) to handle more complex user queries.

For more information on configuring Enterprise Search and integrating your RAG pipeline, head to the [Build section on Enterprise Search](https://rasa.com/docs/pro/build/configuring-enterprise-search/).

What can you customise in Enterprise Search?

## Vector Store Integrations[​](https://rasa.com/docs/pro/customize/enterprise-search/#vector-store-integrations 'Direct link to Vector Store Integrations')

Rasa’s **Enterprise Search** supports out-of-the-box integrations with multiple vector stores. These integrations allow you to set up a pipeline that indexes your documents for semantic search and returns the most relevant chunks when triggered by your assistant.

### Out-of-the-Box Vector Stores[​](https://rasa.com/docs/pro/customize/enterprise-search/#out-of-the-box-vector-stores 'Direct link to Out-of-the-Box Vector Stores')

- [Qdrant](https://qdrant.tech/)
- [Milvus](https://github.com/milvus-io/milvus/)
- [Faiss](https://ai.meta.com/tools/faiss/)

You can configure each vector store (e.g., connection parameters, threshold scores) in your `endpoints.yml` and `config.yml` files. Refer to the [Enterprise Search Policy](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/) reference for the exact configuration details and examples.

## Custom Information Retrieval[​](https://rasa.com/docs/pro/customize/enterprise-search/#custom-information-retrieval 'Direct link to Custom Information Retrieval')

Enterprise Search can also connect your own custom retrieval logic if you need something beyond the built-in integrations. This is especially useful for:

- **Slot-based retrieval**: Filter your documents based on specific slots in the conversation (e.g., filtering an FAQ or knowledge base by a user’s role or product tier).
- **Proprietary search engines**: If your organization already has a custom search pipeline or advanced ML models you want to reuse, you can integrate them via a custom retriever interface.
- **Different embedding or indexing strategies**: Use your own embeddings or indexing approach to match your domain needs.

For implementation details, such as how to create a custom retriever class and plug it into the retrieval pipeline, see the reference docs on [Custom Information Retrieval](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/).

## Extractive Search[​](https://rasa.com/docs/pro/customize/enterprise-search/#extractive-search 'Direct link to Extractive Search')

Extractive Search removes the LLM-based generation step in RAG altogether, allowing your assistant to respond with direct text snippets from your stored FAQ data. This approach can be ideal if you have a well-structured or curated knowledge base and want to reduce costs and hallucination risk by not calling an LLM for generating an answer.

1. **How It Works**
 - Each piece of Q&A content is ingested as a “document” with a question stored in the document text (page content) and the corresponding answer in the document metadata.
 - The assistant retrieves the best match from the knowledge base using a vector store or a custom retriever and responds with the metadata’s answer field directly.
2. **Why Use Extractive Search**
 - **Lower Latency and Cost**: No calls to an external LLM service are required to rephrase or synthesize an answer.
 - **Exact Answers**: Ideal when your domain demands an exact snippet from curated sources.
 - **Simplicity**: Fewer moving parts in your pipeline if your answers need not be dynamically generated or summarized.
3. **Configuration**
 - In your `config.yml`, set `use_generative_llm: false` under the `EnterpriseSearchPolicy` component.
 - Make sure your dataset is loaded into whichever store you’re using (Faiss, Milvus, Qdrant, or a custom retriever) in the Q&A format.

For more information on Extractive Search see the corresponding [reference page](https://rasa.com/docs/reference/config/policies/extractive-search/).

## Configuring fallbacks[​](https://rasa.com/docs/pro/customize/enterprise-search/#configuring-fallbacks 'Direct link to Configuring fallbacks')

Often end users ask questions that cannot be answered from the available content in the knowledge bases. In such scenarios, it is important for the assistant to abstain from responding to the user. `EnterpriseSearchPolicy` can be configured to do so by setting `check_relevancy` to `True` in its config options. By doing so, the assistant assesses whether the user's query has a relevant answer in the knowledge base. If it doesn't find a relevant answer, the control is passed to `pattern_cannot_handle`. The default behaviour of the assistant in this case is to answer with a generic response - `I’m sorry, I can’t help with that`. However, this behaviour is fully [customizable](https://rasa.com/docs/reference/config/policies/generative-search/#relevancy-check) to either respond with a different utterance, fallback to a free-form generative response, fallback to a live agent or any business logic you want the assistant to run in this scenario.

- [Vector Store Integrations](https://rasa.com/docs/pro/customize/enterprise-search/#vector-store-integrations)
 - [Out-of-the-Box Vector Stores](https://rasa.com/docs/pro/customize/enterprise-search/#out-of-the-box-vector-stores)
- [Custom Information Retrieval](https://rasa.com/docs/pro/customize/enterprise-search/#custom-information-retrieval)
- [Extractive Search](https://rasa.com/docs/pro/customize/enterprise-search/#extractive-search)
- [Configuring fallbacks](https://rasa.com/docs/pro/customize/enterprise-search/#configuring-fallbacks)