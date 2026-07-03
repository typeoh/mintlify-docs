# Source: https://rasa.com/docs/pro/build/configuring-enterprise-search

On this page

A conversational assistant can broadly serve two types of user journeys:

- **Transactional**: Focus on structured steps and business logic (for example, filling in a form, canceling an order, or updating a user’s personal information). These are typically orchestrated by **flows** that strictly follow a guided script.
- **Informational**: Focus on generating free-form answers to user queries by retrieving knowledge from large or varied sources of information.

For informational use cases, the recommended approach in CALM is based on [Enterprise Search](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/), a feature that integrates RAG (Retrieval-Augmented Generation) Pipelines. When users ask an open-ended or factual question, the system finds relevant documents from a vector store or a custom data source and then uses an LLM to generate the best possible answer. This allows you to create knowledge-driven experiences without manually coding every possible Q&A flow.

### What is RAG?[​](https://rasa.com/docs/pro/build/configuring-enterprise-search/#what-is-rag 'Direct link to What is RAG?')

Retrieval-Augmented Generation (RAG) is a two-step pipeline:

1. **Retrieve**: Search for relevant content in a document store or vector database based on the user’s latest query (and potentially some conversation history).
2. **Generate**: Use an LLM to produce a response guided by the retrieved content.

This approach keeps your domain knowledge centralized and up-to-date, rather than hard-coding static answers or duplicating content in many flows.

## How to Connect RAG Pipelines[​](https://rasa.com/docs/pro/build/configuring-enterprise-search/#how-to-connect-rag-pipelines 'Direct link to How to Connect RAG Pipelines')

In Rasa Pro, retrieval-based capabilities are provided by the **Enterprise Search Policy**. A typical setup involves:

1. **Collecting and Vectorizing Documents**
 - Prepare your knowledge-base documents (e.g., PDF text, product manuals, FAQ data) and store them in your chosen vector database.
 - You can ingest documents with a data ingestion pipeline (owned by you).
 - Ensure the same embedding model is configured for both your ingestion pipeline and the assistant’s Enterprise Search Policy so that document vectors match queries.
2. **Configuring the Command Generator**
 - In your `config.yml`, ensure that you are using the [`SearchReadyLLMCommandGenerator`](https://rasa.com/docs/reference/config/components/llm-command-generators/#searchreadyllmcommandgenerator).
3. **Configuring Enterprise Search Policy**
 - In your `config.yml`, add or enable the [`EnterpriseSearchPolicy`](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/).
 - Configure it to point to the vector store where your documents live (e.g. “faiss,” “milvus,” “qdrant,” or a custom retriever).
 - Optionally, set the LLM and embeddings provider if you want to customize them (for instance, using your own fine-tuned model).
4. **Linking the Assistant to the Vector Store**
 - In your `endpoints.yml`, specify the connection details for the vector store (host, port, collection name, etc.).
 - During runtime, when a user query arrives, the policy uses the vector store to fetch relevant content.
5. **Triggering Enterprise Search**
 - Whenever the assistant (command generator) detects a knowledge-oriented question, it triggers the Enterprise Search Policy (via the built-in `pattern_search` pattern).
 - The policy retrieves relevant documents, constructs a prompt including those documents, and asks the LLM to generate a final answer.

For more details on each step (e.g., how to configure your Qdrant or Milvus hosts, how to ingest documents with a script, or advanced prompt engineering), see the [Enterprise Search Policy reference](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/).

## When to Use RAG vs. When to Use Flows[​](https://rasa.com/docs/pro/build/configuring-enterprise-search/#when-to-use-rag-vs-when-to-use-flows 'Direct link to When to Use RAG vs. When to Use Flows')

Transactional interactions—where precise multi-step logic or data gathering is required—are best handled by **flows**. They let you define a structured, guided path through conversation steps: for example, filling out a form, verifying identity, or applying consistent logic for an enterprise workflow.

RAG is more suitable for **open-ended or informational** content:

- **Single Source of Truth**: Documents are stored and updated centrally, so answers are always drawn from the latest company data. This allows multiple teams or lines of business (LOBs) to tap into one knowledge backbone rather than duplicating content in separate assistants.
- **Long-Tail Query Handling**: RAG can address a wide variety of user questions—even ones you didn’t explicitly anticipate—by retrieving information from your vector store.
- **Scalability**: You don’t need to write new flows every time documentation changes. As your knowledge base grows, RAG answers can adapt automatically.

Often, a **hybrid** approach works well:

- Transactional requests (e.g., “Open a support ticket” or “Cancel my subscription flow”) are handled by flows on the **dialogue stack**.
- Informational requests (e.g., “How do I reset my password?”) trigger a retrieval call to your knowledge base.

If the user switches from an in-progress flow to an informational question, the assistant can **push** the “knowledge search” pattern onto the dialogue stack, answer it via RAG, and then continue the previous flow. CALM’s dialogue stack will manage which flow is on top.

## How to Connect Vector Stores[​](https://rasa.com/docs/pro/build/configuring-enterprise-search/#how-to-connect-vector-stores 'Direct link to How to Connect Vector Stores')

Rasa supports an out-of-the-box integration with:

- [FAISS](https://ai.meta.com/tools/faiss/) (default, in-memory for quick prototyping)
- [Milvus](https://github.com/milvus-io/milvus/)
- [Qdrant](https://qdrant.tech/)

You can configure any of these by:

1. **Setting `vector_store.type`** in `config.yml` to `faiss`, `milvus`, or `qdrant`.
2. **Adding Connection Details** in `endpoints.yml`. For example, if you’re using Qdrant, you’ll include things like `host`, `port`, and `collection`.
3. **(Optional) Adjusting Thresholds** for similarity filtering, or specifying advanced parameters (like gRPC ports or custom authentication).

Head to the [reference documentation](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/#vector-store) of above vector stores to know more about their integration with CALM.

## Custom Information Retrievers[​](https://rasa.com/docs/pro/build/configuring-enterprise-search/#custom-information-retrievers 'Direct link to Custom Information Retrievers')

For use cases where you’re:

- Using a proprietary search engine,
- Working with specialized data sources or security constraints,
- Or you simply want more control over the retrieval logic,

Rasa allows you to plug in a **Custom Information Retriever**. This means you can override how the assistant executes a “search” step—whether it’s via a custom REST endpoint, a local database, or a special re-ranking algorithm.

**High-Level Steps to Implement a Custom Retriever**:

1. **Create a Python Class** that inherits from Rasa’s `InformationRetrieval` interface and implements:
 
 - A `connect` method to establish a connection (if needed).
 - A `search` method to query your system and return relevant documents in the expected format.
2. **Reference Your Custom Class** in `config.yml` under `vector_store.type`, using the Python module path, e.g.:
 
 config.yml
 
 ```
    # ...policies:- name: EnterpriseSearchPolicy  vector_store:    type: "my_custom_module.MyRetriever"
    ```
 
3. **Implement Any Additional Logic** needed for features like slot-based filtering, custom embeddings, or advanced caching.
 

By extending the information retrieval flow yourself, you can control precisely how documents are found, scored, and returned—while still benefiting from Rasa Pro’s LLM-driven prompt generation and conversation management.

**Further Reading**

1. For more details on custom information retrieval, head over to the Reference section on [Custom Information Retrievers](https://rasa.com/docs/reference/config/policies/custom-information-retrievers/)
 
2. For in-depth instructions on connecting a specific database, examples of ingestion scripts, or advanced policy parameters (such as customizing the LLM prompt template), head to [Enterprise Search Policy reference](https://rasa.com/docs/reference/config/policies/enterprise-search-policy/).
 

- [What is RAG?](https://rasa.com/docs/pro/build/configuring-enterprise-search/#what-is-rag)
- [How to Connect RAG Pipelines](https://rasa.com/docs/pro/build/configuring-enterprise-search/#how-to-connect-rag-pipelines)
- [When to Use RAG vs. When to Use Flows](https://rasa.com/docs/pro/build/configuring-enterprise-search/#when-to-use-rag-vs-when-to-use-flows)
- [How to Connect Vector Stores](https://rasa.com/docs/pro/build/configuring-enterprise-search/#how-to-connect-vector-stores)
- [Custom Information Retrievers](https://rasa.com/docs/pro/build/configuring-enterprise-search/#custom-information-retrievers)