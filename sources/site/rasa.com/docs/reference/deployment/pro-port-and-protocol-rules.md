# Source: https://rasa.com/docs/reference/deployment/pro-port-and-protocol-rules

On this page

A Rasa Pro deployment consists of various components that need to communicate with each other. This page offers an overview of the outbound connection requirements for a Rasa Pro deployment.

The following diagram shows the port requirements for a Rasa Pro deployment:

![image](https://rasa.com/docs/assets/images/rasa-pro-deployment-port-rules-b6fe6891540bb4f32a2ab2ef1d1436e0.png)

## Port and Protocol Requirements[​](https://rasa.com/docs/reference/deployment/pro-port-and-protocol-rules/#port-and-protocol-requirements 'Direct link to Port and Protocol Requirements')

| Component | Port | Protocol | Purpose |
| --- | --- | --- | --- |
| Rasa Pro Server | 5005 | HTTP/HTTPS | Rest API & Webhooks |
| Action Server | 5055 | HTTP/HTTPS/gRPC | Custom actions |
| Rasa Pro Services | 8732 | HTTP/HTTPS | Health Check |
| NLG Server | 5056 | HTTP/HTTPS | Response Generation |
| NLU Server | 5000 | HTTP/HTTPS | NLU Processing |
| PostgreSQL | 5432 | PostgreSQL Wire | Tracker store / Rasa Pro Services Analytics Database |
| MongoDB | 27017 | MongoDB Wire | Tracker Store |
| Redis | 6379 | RESP | Tracker Store / Lock Store / Cache |
| Kafka | 2181 | Kafka | Event Streaming |
| RabbitMQ | 5672 | AMQP | Event Streaming |
| Duckling | 8000 | HTTP/HTTPS | Entity Extraction |
| Jaeger | 6831/16686 | Thrift/HTTP/HTTPS | Tracing |
| OTEL Collector | 4318 | HTTP/HTTPS | Tracing |
| Qdrant | 6333/6334 | HTTP/HTTPS/gRPC | Information Retrieval |
| Milvus | 19530 | gRPC | Information Retrieval |

### Networking Rules[​](https://rasa.com/docs/reference/deployment/pro-port-and-protocol-rules/#networking-rules 'Direct link to Networking Rules')

- Rasa Pro Server: Needs access to all other components.
- Rasa Pro Services: Only needs Kafka, PostgreSQL.

- [Port and Protocol Requirements](https://rasa.com/docs/reference/deployment/pro-port-and-protocol-rules/#port-and-protocol-requirements)
 - [Networking Rules](https://rasa.com/docs/reference/deployment/pro-port-and-protocol-rules/#networking-rules)