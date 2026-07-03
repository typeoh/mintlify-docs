# Source: https://rasa.com/docs/reference/integrations/event-brokers

On this page

## Format[​](https://rasa.com/docs/reference/integrations/event-brokers/#format 'Direct link to Format')

All events are streamed to the broker as serialized dictionaries every time the tracker updates its state. An example event emitted from the `default` tracker looks like this:

```
{    "sender_id": "default",    "timestamp": 1528402837.617099,    "event": "bot",    "text": "what your bot said",    "data": "some data about e.g. attachments",    "metadata": {          "a key": "a value"     }}
```

The `event` field takes the event's `type_name` (for more on event types, check out the [events](https://rasa.com/docs/reference/integrations/action-server/events/) docs).

## Kafka Event Broker[​](https://rasa.com/docs/reference/integrations/event-brokers/#kafka-event-broker 'Direct link to Kafka Event Broker')

[Kafka](https://kafka.apache.org/) is recommended for all assistants at scale. Kafka is a **requirement** when streaming events to Rasa Pro Services or Rasa Studio.

Rasa uses the [confluent-kafka](https://docs.confluent.io/platform/current/clients/confluent-kafka-python/html/index.html#) library, a Kafka client written in Python.

How to consume events in Rasa Analytics

To check how to configure Rasa Analytics to consume events from Kafka, please refer to the [Analytics documentation](https://rasa.com/docs/reference/integrations/analytics/#1-connect-an-assistant).

### Configuration[​](https://rasa.com/docs/reference/integrations/event-brokers/#configuration 'Direct link to Configuration')

To use Kafka as an event broker in Rasa, you need to set it as an `event_broker` in your `endpoints.yml` file. For Kafka, Rasa supports following properties in `even_broker` section:

endpoints.yml

```
event_broker:  type: kafka  url: localhost:9092  # required, url to your kafka broker  # Optional properties  topic: rasa_core_events # topic to which events are published  security_protocol: "SASL_PLAINTEXT" # security protocol to use, available options are: PLAINTEXT, SASL_PLAINTEXT, SSL, SASL_SSL  partition_by_sender: False # should events be partitioned by sender id  client_id: # ID to use for the producer of the events  # SASL configuration, optional  sasl_mechanism: "PLAIN" # SASL mechanism to use, available options are: PLAIN, GSSAPI, OAUTHBEARER, SCRAM-SHA-256, SCRAM-SHA-512  sasl_username: # username to use for authentication, only if security_protocol is SASL_PLAINTEXT or SASL_SSL  sasl_password: # password to use for authentication, only if security_protocol is SASL_PLAINTEXT or SASL_SSL  # TLS configuration, optional  ssl_cafile: # path to the CA certificate file, only if security_protocol is SSL or SASL_SSL  ssl_certfile: # path to the client certificate file, only if security_protocol is SSL or SASL_SSL and Kafka is configured to use client authentication  ssl_keyfile: # path to the client key file, only if security_protocol is SSL or SASL_SSL and Kafka is configured to use client authentication  ssl_check_hostname: False # whether to check the hostname of the broker against the certificate, default is True  # PII management configuration, optional  stream_pii: False # whether to stream PII events, default is True  anonymization_topics: # list of topics to publish anonymized events to, default is []   - anonymized_topic_1
```

#### Partition Key[​](https://rasa.com/docs/reference/integrations/event-brokers/#partition-key 'Direct link to Partition Key')

Rasa's Kafka producer can optionally be configured to partition messages by conversation ID. This can be configured by setting `partition_by_sender` in the `endpoints.yml` file to True. By default, this parameter is set to `False` and the producer will randomly assign a partition to each message.

#### Authentication and Authorization[​](https://rasa.com/docs/reference/integrations/event-brokers/#authentication-and-authorization 'Direct link to Authentication and Authorization')

Rasa's Kafka producer accepts the following types of security protocols: `SASL_PLAINTEXT`, `SSL`, `PLAINTEXT` and `SASL_SSL`.

For development environments, or if the brokers servers and clients are located into the same machine, you can use simple authentication with `SASL_PLAINTEXT` or `PLAINTEXT`. By using this protocol, the credentials and messages exchanged between the clients and servers will be sent in plaintext. Thus, this is not the most secure approach, but since it's simple to configure, it is useful for simple cluster configurations. `SASL_PLAINTEXT` protocol requires the setup of the `username` and `password` previously configured in the broker server.

If the clients or the brokers in the kafka cluster are located in different machines, it's important to use the `SSL` or `SASL_SSL` protocol to ensure encryption of data and client authentication. After generating valid certificates for the brokers and the clients, the path to the certificate and key generated for the producer must be provided as arguments, as well as the CA's root certificate.

When using the `SASL_PLAINTEXT` and `SASL_SSL` protocols, the `sasl_mechanism` can be optionally configured and is set to `PLAIN` by default. Valid values for `sasl_mechanism` are: `PLAIN`, `GSSAPI`, `OAUTHBEARER`, `SCRAM-SHA-256`, and `SCRAM-SHA-512`.

If `GSSAPI` is used for the `sasl_mechanism`, you will need to additionally install [python-gssapi](https://pypi.org/project/python-gssapi/) and the necessary C library Kerberos dependencies.

If the `ssl_check_hostname` parameter is enabled, the clients will verify if the broker's hostname matches the certificate. It's used on client's connections and inter-broker connections to prevent man-in-the-middle attacks.

##### Using IAM roles to authenticate to AWS Managed Streaming for Apache Kafka (MSK)[​](https://rasa.com/docs/reference/integrations/event-brokers/#using-iam-roles-to-authenticate-to-aws-managed-streaming-for-apache-kafka-msk 'Direct link to Using IAM roles to authenticate to AWS Managed Streaming for Apache Kafka (MSK)')

New in 3.14

You can use IAM authentication to connect to AWS MSK without needing to provide a username and password.

If your Rasa instance is running on an AWS service that supports IAM roles (e.g. EC2), you can use IAM authentication to connect to an AWS MSK cluster without needing to provide a username and password.

To do so, you need to ensure that your MSK cluster is configured to allow IAM authentication. Ensure that the topic(s) you want to use are created on the MSK cluster.

You also need to set up your Rasa instance with an appropriate IAM role that has the permissions to access the MSK cluster. The IAM role should include the following permissions:

```
{    "Version": "2012-10-17",    "Statement": [        {            "Effect": "Allow",            "Action": [                "kafka-cluster:Connect",                "kafka-cluster:DescribeCluster"            ],            "Resource": [                "arn:aws:kafka:<region_name>:<account_id>:cluster/<cluster_name>/<cluster_uuid>"            ]        },        {            "Effect": "Allow",            "Action": [                "kafka-cluster:*Topic*",                "kafka-cluster:WriteData",                "kafka-cluster:ReadData"            ],            "Resource": [                "arn:aws:kafka:<region_name>:<account_id>:topic/<cluster_name>/*"            ]        },        {            "Effect": "Allow",            "Action": [                "kafka-cluster:AlterGroup",                "kafka-cluster:DescribeGroup"            ],            "Resource": [                "arn:aws:kafka:<region_name>:<account_id>:group/<cluster_name>/*"            ]        }    ]}
```

Once you have set up the IAM role and configured your AWS MSK cluster, you can configure the following environment variables in your Rasa instance:

- `IAM_CLOUD_PROVIDER`: Set this to `aws`.
- `AWS_DEFAULT_REGION`: Set this to the AWS region where your MSK cluster is located.
- `KAFKA_MSK_AWS_IAM_ENABLED`: Set this to `true` to enable IAM authentication for MSK connections, otherwise leave it unset or set to `false`.

You also need to configure the Kafka event broker in your `endpoints.yml` file to use the `SASL_SSL` security protocol and `OAUTHBEARER` SASL mechanism, as well as provide the path to the root certificate from Amazon Trust Services in the `ssl_cafile` property.

endpoints.yml

```
event_broker:    type: kafka    url: <your-msk-broker-url>    topic: rasa_core_events    security_protocol: "SASL_SSL"    sasl_mechanism: "OAUTHBEARER"    ssl_cafile: <path-to-amazon-trust-services-root-certificate>    partition_by_sender: true    client_id: <your-client-id>    ssl_check_hostname: true
```

When you start your Rasa instance, it will use the IAM role to generate temporary credentials to log in to the AWS MSK cluster instead of using static credentials. The temporary credentials will be automatically refreshed every 15 minutes.

### Example Configurations[​](https://rasa.com/docs/reference/integrations/event-brokers/#example-configurations 'Direct link to Example Configurations')

#### Without authentication[​](https://rasa.com/docs/reference/integrations/event-brokers/#without-authentication 'Direct link to Without authentication')

To set up Rasa with Kafka which does not require authentication nor TLS handshake, use following config as an example:

endpoints.yml

```
event_broker:  type: kafka  security_protocol: PLAINTEXT # specifies that no credentials will be used to auth and communication is not encrypted  topic: topic  url: localhost
```

To set up Rasa with Kafka which does not require authentication but uses TLS handshake, use following config as an example:

endpoints.yml

```
event_broker:  type: kafka  security_protocol: SSL # specifies that no credentials will be used to auth but communication is encrypted using SSL/TLS  topic: topic  url: localhost  ssl_cafile: CARoot.pem # CA certificate for the server  ssl_certfile: certificate.pem # client certificate, if Kafka requires that client present their certificate as a form of mutual domain validation  ssl_keyfile: key.pem # client key, if Kafka requires that client present their certificate as a form of mutual domain validation  ssl_check_hostname: True # verifies that the hostname of Kafka server matches the certificate
```

#### With authentication[​](https://rasa.com/docs/reference/integrations/event-brokers/#with-authentication 'Direct link to With authentication')

To set up Rasa with Kafka which requires authentication but does not use TLS handshake, use following config as an example:

endpoints.yml

```
event_broker:  type: kafka  security_protocol: SASL_PLAINTEXT # specifies that credentials will not be communicated over encryption  topic: topic  url: localhost  sasl_username: username  sasl_password: password  sasl_mechanism: PLAIN # specifies that username/password will not be protected with additional protection mechanisms
```

To set up Rasa with Kafka which requires authentication and uses TLS handshake, use following config as an example:

endpoints.yml

```
event_broker:  type: kafka  security_protocol: SASL_SSL # specifies that credentials will be communicated over enrypted connection  topic: topic  url: localhost  sasl_username: username  sasl_password: password  sasl_mechanism: PLAIN # specifies that username/password will not be protected with additional protection mechanisms  ssl_cafile: CARoot.pem # CA certificate for the server, it is used to verify the server's certificate  ssl_certfile: certificate.pem # client certificate, if Kafka requires that client present their certificate as a form of mutual correspondence validation  ssl_keyfile: key.pem # client key, if Kafka requires that client present their certificate as a form of mutual correspondence validation  ssl_check_hostname: True # verifies that the hostname of Kafka server matches the certificate
```

Make sure that the SASL mechanism is set according to the broker configuration. You can also use `GSSAPI`, `OAUTHBEARER`, `SCRAM-SHA-256` or `SCRAM-SHA-512` if your broker is configured to use it for the exposed URL endpoint.

### Sending Events to Multiple Queues[​](https://rasa.com/docs/reference/integrations/event-brokers/#sending-events-to-multiple-queues 'Direct link to Sending Events to Multiple Queues')

Kafka does not allow you to configure multiple topics.

However, multiple consumers can read from the same queue as long as they are in different consumer groups. Each consumer group will process all events independent of each other (in a sense, each group has their own reference to the last event they have processed). [Kafka: The Definitive Guide](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/ch04.html#:~:text=Kafka%20consumers%20are%20typically%20part,the%20partitions%20in%20the%20topic.)

#### Disabling Publishing of Un-anonymised Events[​](https://rasa.com/docs/reference/integrations/event-brokers/#disabling-publishing-of-un-anonymised-events 'Direct link to Disabling Publishing of Un-anonymised Events')

You can configure the event broker to not publish un-anonymised events to the configured topic. This is done by setting the `stream_pii` parameter in the `endpoints.yml` file to `false`.

#### Sending Anonymized Events[​](https://rasa.com/docs/reference/integrations/event-brokers/#sending-anonymized-events 'Direct link to Sending Anonymized Events')

If you have the PII management capability enabled, you can configure the event broker to publish anonymised events to a different topic. This is done by setting the `anonymization_topics` parameter in the `endpoints.yml` file to a list of topics. The event broker will publish anonymised events to the specified topics when the PII management capability is enabled.

## Pika Event Broker for RabbitMQ[​](https://rasa.com/docs/reference/integrations/event-brokers/#pika-event-broker-for-rabbitmq 'Direct link to Pika Event Broker for RabbitMQ')

Rasa uses [Pika](https://pika.readthedocs.io) , the Python client library for [RabbitMQ](https://www.rabbitmq.com).

### Configuration[​](https://rasa.com/docs/reference/integrations/event-brokers/#configuration-1 'Direct link to Configuration')

To use RabbitMQ as an event broker in Rasa, you need to set it as an `event_broker` in your `endpoints.yml` file.

For RabbitMQ, Rasa supports following properties in `even_broker` section:

endpoints.yml

```
event_broker:  type: pika  host: # required, hostname of your RabbitMQ broker e.g. localhost  port: 5672 # port of your RabbitMQ broker e.g. 5672  exchange_name: "rasa-exchange" # exchange name to use for publishing events  username: # required, username to use for authentication  password: # required, password to use for authentication  connection_attempts: 20 # number of connection attempts to make before giving up  retry_delay_in_seconds: 5 # time to wait before retrying connection attempts  raise_on_failure: False #whether to raise an exception on connection failure  should_keep_unpublished_messages: True # whether to keep unpublished messages in memory  queues: # list of queues to publish events to"    - rasa_core_events # default  stream_pii: False # whether to publish un-anonymised events to the configured queues. defaults to True  anonymization_queues: # list of queues to publish anonymized events to. defaults to []    - anonymized_event_queue
```

Additionally, you can use TLS with RabbitMQ by setting the following environment variables:

- `RABBITMQ_SSL_CLIENT_CERTIFICATE`: path to the SSL client certificate
- `RABBITMQ_SSL_CLIENT_KEY`: path to the SSL client key

Please note that specifying 'RABBITMQ\_SSL\_CA\_FILE' via environment variables is no longer supported, as well as specifying `RABBITMQ_SSL_KEY_PASSWORD` environment variable - please use a key file that is not encrypted instead.

### Example Configurations[​](https://rasa.com/docs/reference/integrations/event-brokers/#example-configurations-1 'Direct link to Example Configurations')

To set up Rasa with Pika for RabbitMQ use following config as an example:

endpoints.yml

```
event_broker:  type: pika  url: localhost  username: username  password: password  queues:    - queue-1  #   you may supply more than one queue to publish to  #   - queue-2  #   - queue-3  exchange_name: exchange
```

### Adding a Pika Event Broker in Python[​](https://rasa.com/docs/reference/integrations/event-brokers/#adding-a-pika-event-broker-in-python 'Direct link to Adding a Pika Event Broker in Python')

Here is how you add it using Python code:

```
import asynciofrom rasa.core.brokers.pika import PikaEventBrokerfrom rasa.core.tracker_store import InMemoryTrackerStorepika_broker = PikaEventBroker('localhost',                              'username',                              'password',                              queues=['rasa_events'],                              event_loop=event_loop                              )asyncio.run(pika_broker.connect())tracker_store = InMemoryTrackerStore(domain=domain, event_broker=pika_broker)
```

### Implementing a Pika Event Consumer[​](https://rasa.com/docs/reference/integrations/event-brokers/#implementing-a-pika-event-consumer 'Direct link to Implementing a Pika Event Consumer')

You need to have a RabbitMQ server running, as well as another application that consumes the events. This consumer to needs to implement Pika's `start_consuming()` method with a `callback` action. Here's a simple example:

```
import jsonimport pikadef _callback(ch, method, properties, body):        # Do something useful with your incoming message body here, e.g.        # saving it to a database        print("Received event {}".format(json.loads(body)))if __name__ == "__main__":    # RabbitMQ credentials with username and password    credentials = pika.PlainCredentials("username", "password")    # Pika connection to the RabbitMQ host - typically 'rabbit' in a    # docker environment, or 'localhost' in a local environment    connection = pika.BlockingConnection(        pika.ConnectionParameters("rabbit", credentials=credentials)    )    # start consumption of channel    channel = connection.channel()    channel.basic_consume(queue="rasa_events", on_message_callback=_callback, auto_ack=True)    channel.start_consuming()
```

### Sending Events to Multiple Queues[​](https://rasa.com/docs/reference/integrations/event-brokers/#sending-events-to-multiple-queues-1 'Direct link to Sending Events to Multiple Queues')

You can specify multiple event queues to publish events to. This should work for all event brokers supported by Pika (e.g. RabbitMQ)

#### Disabling Publishing of Un-anonymised Events[​](https://rasa.com/docs/reference/integrations/event-brokers/#disabling-publishing-of-un-anonymised-events-1 'Direct link to Disabling Publishing of Un-anonymised Events')

By default, Rasa will publish un-anonymised events to the configured queues. If you want to disable this and only publish anonymized events, set `stream_pii: false` in your `event_broker` configuration.

#### Publishing Anonymized Events[​](https://rasa.com/docs/reference/integrations/event-brokers/#publishing-anonymized-events 'Direct link to Publishing Anonymized Events')

If you want to publish anonymized events to a different queue, you can set the `anonymization_queues` property in your `event_broker` configuration. This should be a list of queues to publish anonymized events to. By default, this is set to an empty list, which means that anonymized events will not be published to any queue.

## SQL Event Broker[​](https://rasa.com/docs/reference/integrations/event-brokers/#sql-event-broker 'Direct link to SQL Event Broker')

It is possible to use an SQL database as an event broker. Connections to databases are established using [SQLAlchemy](https://www.sqlalchemy.org/), a Python library which can interact with many different types of SQL databases, such as [SQLite](https://sqlite.org/index.html), [PostgreSQL](https://www.postgresql.org/) and more. The default Rasa installation allows connections to SQLite and PostgreSQL databases. To see other options, please see the [SQLAlchemy documentation on SQL dialects](https://docs.sqlalchemy.org/en/13/dialects/index.html).

To set up Rasa with SQL event broker the following steps are required:

1. Add required configuration to your `endpoints.yml`

When using SQLite:

endpoints.yml

```
event_broker:  type: SQL  dialect: sqlite  db: events.db
```

When using PostgreSQL:

endpoints.yml

```
event_broker:  type: SQL  url: 127.0.0.1  port: 5432  dialect: postgresql  username: myuser  password: mypassword  db: mydatabase
```

2. To start the Rasa server using your SQL backend, add the `--endpoints` flag, e.g.:
 
 ```
    rasa run -m models --endpoints endpoints.yml
    ```
 

## FileEventBroker[​](https://rasa.com/docs/reference/integrations/event-brokers/#fileeventbroker 'Direct link to FileEventBroker')

It is possible to use the `FileEventBroker` as an event broker. This implementation will log events to a file in json format.

You can provide a path key in the `endpoints.yml` file if you wish to override the default file name: `rasa_event.log`.

## Custom Event Broker[​](https://rasa.com/docs/reference/integrations/event-brokers/#custom-event-broker 'Direct link to Custom Event Broker')

If you need an event broker which is not available out of the box, you can implement your own. This is done by extending the base class `EventBroker`.

Your custom event broker class must also implement the following base class methods:

- `from_endpoint_config`: creates an `EventBroker` object from the endpoint configuration. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/brokers/broker.py#L45).
- `publish`: publishes a json-formatted [Rasa event](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/shared/core/events/) into an event queue. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/brokers/broker.py#L63).
- `is_ready`: determine whether the event broker is ready. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/brokers/broker.py#L67).
- `close`: close the connection to an event broker. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/brokers/broker.py#L75).

Note that the `__init__` method of your custom event broker class must also implement the instance attribute `self.stream_pii` to indicate if the event broker should stream un-anonymised events or not. The default value should be `True`.

To set up Rasa with your custom event broker the following steps are required:

1. Add required configuration to your `endpoints.yml`

endpoints.yml

```
event_broker:  type: path.to.your.module.Class  url: localhost  a_parameter: a value  another_parameter: another value
```

2. To start the Rasa server using your custom backend, add the `--endpoints` flag, e.g.:
 
 ```
    rasa run -m models --endpoints endpoints.yml
    ```
 

- [Format](https://rasa.com/docs/reference/integrations/event-brokers/#format)
- [Kafka Event Broker](https://rasa.com/docs/reference/integrations/event-brokers/#kafka-event-broker)
 - [Configuration](https://rasa.com/docs/reference/integrations/event-brokers/#configuration)
 - [Example Configurations](https://rasa.com/docs/reference/integrations/event-brokers/#example-configurations)
 - [Sending Events to Multiple Queues](https://rasa.com/docs/reference/integrations/event-brokers/#sending-events-to-multiple-queues)
- [Pika Event Broker for RabbitMQ](https://rasa.com/docs/reference/integrations/event-brokers/#pika-event-broker-for-rabbitmq)
 - [Configuration](https://rasa.com/docs/reference/integrations/event-brokers/#configuration-1)
 - [Example Configurations](https://rasa.com/docs/reference/integrations/event-brokers/#example-configurations-1)
 - [Adding a Pika Event Broker in Python](https://rasa.com/docs/reference/integrations/event-brokers/#adding-a-pika-event-broker-in-python)
 - [Implementing a Pika Event Consumer](https://rasa.com/docs/reference/integrations/event-brokers/#implementing-a-pika-event-consumer)
 - [Sending Events to Multiple Queues](https://rasa.com/docs/reference/integrations/event-brokers/#sending-events-to-multiple-queues-1)
- [SQL Event Broker](https://rasa.com/docs/reference/integrations/event-brokers/#sql-event-broker)
- [FileEventBroker](https://rasa.com/docs/reference/integrations/event-brokers/#fileeventbroker)
- [Custom Event Broker](https://rasa.com/docs/reference/integrations/event-brokers/#custom-event-broker)