# Source: https://rasa.com/docs/reference/integrations/analytics

On this page

Why using Analytics?

For more information on Analytics and its benefits, see the [Analytics product documentation](https://rasa.com/docs/pro/improve/analytics/).

Analytics Data Pipeline helps visualize and process Rasa assistant metrics in the tooling (BI tools, data warehouses) of your choice. Visualizations and analysis of the production assistant and its conversations allow you to assess ROI and improve the performance of the assistant over time.

![An overview of the components of Rasa][base64-image]

Rasa will connect to your production assistant using Kafka. The pipeline will compute analytics as soon as the docker container is successfully deployed and connected to your data warehouse and Kafka instances.

## Types of metrics[​](https://rasa.com/docs/reference/integrations/analytics/#types-of-metrics 'Direct link to Types of metrics')

With the Analytics pipeline, measurement of the following metrics is possible

| Metric | Category | Meaning |
| --- | --- | --- |
| Number of conversations | Usage Analytics | Total number of conversations |
| Number of users | Usage Analytics | Total number of users |
| Number of sessions | Usage Analytics | Gross traffic to assistant |
| Channels used | Usage Analytics | Sessions by channel |
| User session count | User Analytics | Total number of user sessions or average sessions per user |
| Top N intents | Conversation Analytics | Top intents across all users |
| Avg response time | Conversation Analytics | Average response time for assistant |
| Containment rate | Business Analytics | % of conversations handled purely by assistant (not handed to human agent |
| Abandonment rate | Business Analytics | % of abandoned conversations |
| Escalation rate | Business Analytics | % of conversations escalated to human agent |

## Database Indexes[​](https://rasa.com/docs/reference/integrations/analytics/#database-indexes 'Direct link to Database Indexes')

New in 3.7.0

Rasa Pro Services 3.7.0 introduces new database indexes to improve the performance of Analytics queries during event processing.

The following indexes are created on the Analytics database to improve query performance when processing and transforming events:

- A composite index on the `session_id` and `timestamp ASC` columns of the `rasa_event` table;
- A composite index on the `sender_id`, `session_id` and `timestamp ASC` columns of the `rasa_event` table;
- A composite index on the `sender_id` and `sequence_number` columns of the `rasa_event` table;
- An index on the `sender_key` column of the `rasa_sender` table;
- A composite index on the `sender_id` and `start_sequence_number DESC` columns of the `rasa_session` table;
- A composite index on the `sender_id`, `session_id` and `start_sequence_number DESC` columns of the `rasa_turn` table;
- A composite index on the `sender_id`, `session_id` and `start_sequence_number DESC` columns of the `rasa_dialogue_stack_frame` table;

## Prerequisites[​](https://rasa.com/docs/reference/integrations/analytics/#prerequisites 'Direct link to Prerequisites')

- A production deployment of Kafka is required to set up Rasa. We recommend using [Amazon Managed Streaming for Apache Kafka](https://aws.amazon.com/msk/).
 
- A production deployment of a data lake needs to be connected to the data pipeline. Rasa directly supports the following data lakes:
 
 - [PostgreSQL](https://aws.amazon.com/rds/postgresql/) ( **recommended**. All PostgreSQL >= 11.0 are supported)
 - [Amazon Redshift](https://aws.amazon.com/redshift/)
 
 Virtually any other data lakes can be configured to sync with your deployment of PostgreSQL. You can find additional instructions on how to connect your PostgreSQL deployment to either [BigQuery](https://rasa.com/docs/reference/integrations/analytics/#bigquery) or [Snowflake](https://rasa.com/docs/reference/integrations/analytics/#snowflake) in the [Connect a data warehouse step](https://rasa.com/docs/reference/integrations/analytics/#2-connect-a-data-warehouse).
 
 We recommend managed deployments of your data lake to minimize maintenance efforts.
 

## 1\. Connect an assistant[​](https://rasa.com/docs/reference/integrations/analytics/#1-connect-an-assistant 'Direct link to 1. Connect an assistant')

To connect an assistant to Rasa Pro Services, you need to connect the assistant to an event broker. The assistant will stream all events to the event broker, which will then be consumed by Rasa Pro Services.

#### How to consume events from Rasa[​](https://rasa.com/docs/reference/integrations/analytics/#how-to-consume-events-from-rasa 'Direct link to How to consume events from Rasa')

To check how to configure Rasa to publish events to Kafka, please refer to the [Rasa event broker documentation](https://rasa.com/docs/reference/integrations/event-brokers/#kafka-event-broker).

### Configuration[​](https://rasa.com/docs/reference/integrations/analytics/#configuration 'Direct link to Configuration')

New in Rasa Pro Services 3.6

Rasa Pro Services 3.6 introduces new configuration environment variables to support IAM authentication to AWS RDS MSK:

- `IAM_CLOUD_PROVIDER`
- `AWS_DEFAULT_REGION`

Additionally, Rasa Pro Services 3.6 now support mTLS connection to the Kafka broker with the following new environment variables:

- `KAFKA_SSL_CERTFILE_LOCATION`
- `KAFKA_SSL_KEYFILE_LOCATION`

info

With versions 3.5.4 and 3.4.3. Rasa Analytics introduced new configuration options: `KAFKA_SOCKET_KEEP_ALIVE_ENABLED`, `KAFKA_METADATA_MAX_AGE_MS` `KAFKA_PRODUCER_RETRIES`, `KAFKA_PRODUCER_TIMEOUT_MS`, `KAFKA_PRODUCER_PARTITIONER`, `KAFKA_COMPRESSION_CODEC`, `KAFKA_CONSUMER_HEARTBEAT_INTERVAL_MS`, `KAFKA_CONSUMER_SESSION_TIMEOUT_MS`, `KAFKA_CONSUMER_MAX_POLL_INTERVAL_MS`, `MAX_MESSAGES_TO_FETCH`, `RETRY_CONNECTION_COUNT` and `RETRY_DB_TRANSACTION_COUNT`.

The following environment variables are available to configure Rasa Analytics:

| Environment variable name | Default | Description |
| --- | --- | --- |
| `KAFKA_BROKER_ADDRESS` | | The address of the Kafka broker. |
| `KAFKA_TOPIC` | | The Kafka topic from which Rasa events will be consumed from. |
| `KAFKA_SECURITY_PROTOCOL` | | The security protocol to use. Valid values are `PLAINTEXT`, `SASL_SSL`, `SSL`, `SASL_PLAINTEXT`. |
| `KAFKA_SASL_MECHANISM` | | The SASL mechanism to use. Valid values are `PLAIN`, `SCRAM-SHA-256`, `SCRAM-SHA-512`. Only required if `KAFKA_SECURITY_PROTOCOL` is set to `SASL_SSL` or `SASL_PLAINTEXT`. |
| `KAFKA_SASL_USERNAME` | | The username to use for SASL authentication. |
| `KAFKA_SASL_PASSWORD` | | The password to use for SASL authentication. |
| `KAFKA_SSL_CA_LOCATION` | | The location of the CA certificate for SSL connections. Only required if you are using `SSL` or `SASL_SSL` as the security protocol. |
| `KAFKA_SOCKET_KEEP_ALIVE_ENABLED` | true | (Optional) Whether the socket keep alive is enabled for the Kafka connection. |
| `KAFKA_METADATA_MAX_AGE_MS` | 180000 | (Optional) The maximum age of the Kafka metadata in milliseconds. |
| `KAFKA_PRODUCER_RETRIES` | 2 147 483 647 | (Optional) How many times the Kafka producer retries sending a message when it fails. |
| `KAFKA_PRODUCER_TIMEOUT_MS` | 5000 | (Optional) The timeout for the Kafka producer in milliseconds. |
| `KAFKA_PRODUCER_PARTITIONER` | consistent\_random | (Optional) The partitioner used by the Kafka producer. Valid values are: `random`, `consistent`, `consistent_random`, `murmur2`, `murmur2_random`, `fnv1a` and `fnv1a_random`. |
| `KAFKA_COMPRESSION_CODEC` | gzip | (Optional) The compression codec used by the Kafka producer. Valid values are: `none`, `gzip`, `snappy`, `lz4`, `zstd` and `inherit`. |
| `KAFKA_CONSUMER_HEARTBEAT_INTERVAL_MS` | 3000 | (Optional) The heartbeat interval for the Kafka consumer in milliseconds. |
| `KAFKA_CONSUMER_SESSION_TIMEOUT_MS` | 6000 | (Optional) The session timeout for the Kafka consumer in milliseconds. |
| `KAFKA_CONSUMER_MAX_POLL_INTERVAL_MS` | 600 000 | (Optional) The maximum poll interval for the Kafka consumer in milliseconds. |
| `RASA_ANALYTICS_CONSUMER_ID` | `rasa-analytics-group` | (Optional) The consumer ID to use for the Kafka consumer. |
| `RASA_ANALYTICS_CONSUMER_COUNT` | `1` | (Optional) The number of consumers to use for the Kafka consumer group. |
| `RASA_ANALYTICS_DLQ` | `rasa-analytics-dlq` | (Optional) The topic which will be used to store dead letter events. This topic must be created before starting the Analytics pipeline. |
| `MAX_MESSAGES_TO_FETCH` | `10` | (Optional) How many messages are fetched from the topic at once. |
| `RETRY_DB_TRANSACTION_COUNT` | `7` | (Optional) How many times the DB transaction is retried when it fails. |
| `RETRY_CONNECTION_COUNT` | `7` | (Optional) How many times the connection to the Kafka broker is retried when it fails when broker is not available. |
| `KAFKA_SSL_CERTFILE_LOCATION` | | (Optional) The location of the client certificate for mTLS connections. Only required if you are using `SSL` or `SASL_SSL` as the security protocol and mTLS is enabled. |
| `KAFKA_SSL_KEYFILE_LOCATION` | | (Optional) The location of the client key for mTLS connections. Only required if you are using `SSL` or `SASL_SSL` as the security protocol and mTLS is enabled. |
| `IAM_CLOUD_PROVIDER` | | (Optional) The cloud provider to use for IAM authentication. Valid values are `aws`. Only required if you want to use IAM authentication to connect to AWS RDS (PostgreSQL) and MSK. |
| `AWS_DEFAULT_REGION` | | (Optional) The AWS region to use for IAM authentication. Only required if you want to use IAM authentication to connect to AWS RDS (PostgreSQL) and MSK. |
| `RUN_ANALYTICS_DB_MIGRATIONS` | `true` | Set to `false` to not run the analytics database migrations on startup. |

To learn more about dead letter queues, please refer to the [Wikipedia article](https://en.wikipedia.org/wiki/Dead_letter_queue).

During the development phase, Rasa Analytics can be started before event topic is created. The topic can be created later on. Rasa Analytics will poll the topic for new events. If the topic is not created, Rasa Analytics will not consume any events.

Dead letter queue is not required for Rasa Analytics to work. It is only used to store events that could not be processed by Rasa Analytics.

In production environment, we recommend that the event topic and dead letter topic are created before starting Rasa Analytics.

#### Mapping between Rasa Analytics and Confluent Kafka configuration[​](https://rasa.com/docs/reference/integrations/analytics/#mapping-between-rasa-analytics-and-confluent-kafka-configuration 'Direct link to Mapping between Rasa Analytics and Confluent Kafka configuration')

The following table shows the mapping between Rasa Analytics environment variables and Confluent Kafka configuration properties

| Rasa Analytics Environment Variable | Confluent Kafka (librdkafka) Configuration Property |
| --- | --- |
| `KAFKA_BROKER_ADDRESS` | `bootstrap.servers` |
| `KAFKA_TOPIC` | `topic` |
| `KAFKA_SECURITY_PROTOCOL` | `security.protocol` |
| `KAFKA_SASL_MECHANISM` | `sasl.mechanism` |
| `KAFKA_SASL_USERNAME` | `sasl.username` |
| `KAFKA_SASL_PASSWORD` | `sasl.password` |
| `KAFKA_SSL_CA_LOCATION` | `ssl.ca.location` |
| `KAFKA_SOCKET_KEEP_ALIVE_ENABLED` | `socket.keepalive.enable` |
| `KAFKA_METADATA_MAX_AGE_MS` | `metadata.max.age.ms` |
| `KAFKA_PRODUCER_RETRIES` | `retries` |
| `KAFKA_PRODUCER_TIMEOUT_MS` | `request.timeout.ms` |
| `KAFKA_PRODUCER_PARTITIONER` | `partitioner` |
| `KAFKA_COMPRESSION_CODEC` | `compression.codec` |
| `KAFKA_CONSUMER_HEARTBEAT_INTERVAL_MS` | `heartbeat.interval.ms` |
| `KAFKA_CONSUMER_SESSION_TIMEOUT_MS` | `session.timeout.ms` |
| `KAFKA_CONSUMER_MAX_POLL_INTERVAL_MS` | `max.poll.interval.ms` |
| `RASA_ANALYTICS_CONSUMER_ID` | `group.id` |
| `KAFKA_SSL_CERTFILE_LOCATION` | `ssl.certificate.location` |
| `KAFKA_SSL_KEYFILE_LOCATION` | `ssl.key.location` |

#### Azure Event Hubs[​](https://rasa.com/docs/reference/integrations/analytics/#azure-event-hubs 'Direct link to Azure Event Hubs')

Rasa Analytics can also be connected to Azure Event Hubs. Microsoft recommends certain [configuration settings](https://learn.microsoft.com/en-us/azure/event-hubs/apache-kafka-configurations#librdkafka-configuration-properties) are set to ensure optimal performance and connection stability.

### Example Configurations[​](https://rasa.com/docs/reference/integrations/analytics/#example-configurations 'Direct link to Example Configurations')

#### Without authentication[​](https://rasa.com/docs/reference/integrations/analytics/#without-authentication 'Direct link to Without authentication')

To set up Rasa Analytics with Kafka which does not require authentication nor TLS handshake, use the following config as an example:

```
KAFKA_BROKER_ADDRESS=<URL to your Kafka broker e.g localhost:9092 >KAFKA_TOPIC=<topic name>KAFKA_SECURITY_PROTOCOL=PLAINTEXT
```

To set up Rasa Analytics with Kafka which does not require authentication but uses TLS handshake, use the following config as an example:

```
KAFKA_BROKER_ADDRESS=<URL to your Kafka broker e.g localhost:9092 >KAFKA_TOPIC=<topic name>KAFKA_SECURITY_PROTOCOL=SSL # specifies that authentication is not used by communication is over encryption using TLSKAFKA_SSL_CA_LOCATION=<CA certificate location>
```

#### With authentication[​](https://rasa.com/docs/reference/integrations/analytics/#with-authentication 'Direct link to With authentication')

To set up Rasa Analytics with Kafka which does require authentication but does not use TLS handshake, use the following config as an example:

```
KAFKA_BROKER_ADDRESS=<URL to your Kafka broker e.g localhost:9092 >KAFKA_TOPIC=<topic name>KAFKA_SECURITY_PROTOCOL=SASL_PLAINTEXTKAFKA_SASL_MECHANISM=PLAIN # specifies that username/password will not be protected with additional protection mechanismsKAFKA_SASL_USERNAME=<username>KAFKA_SASL_PASSWORD=<password>
```

To set up Rasa Analytics with Kafka which does require authentication and uses TLS handshake, use the following config as an example:

```
KAFKA_BROKER_ADDRESS=<URL to your Kafka broker e.g localhost:9092 >KAFKA_TOPIC=<topic name>KAFKA_SECURITY_PROTOCOL=SASL_SSL # specifies that credentials will be communicated over encryption using TLSKAFKA_SASL_MECHANISM=PLAIN # specifies that username/password will not be protected with additional protection mechanismsKAFKA_SASL_USERNAME=<username>KAFKA_SASL_PASSWORD=<password>KAFKA_SSL_CA_LOCATION=<CA certificate location>
```

Make sure that the SASL mechanism is set according to the broker configuration. You can also use `GSSAPI`, `OAUTHBEARER`, `SCRAM-SHA-256` or `SCRAM-SHA-512` if your broker is configured to use it for the exposed URL endpoint.

#### (Optional) Using IAM roles to authenticate to AWS RDS (PostgreSQL) and MSK[​](https://rasa.com/docs/reference/integrations/analytics/#optional-using-iam-roles-to-authenticate-to-aws-rds-postgresql-and-msk 'Direct link to (Optional) Using IAM roles to authenticate to AWS RDS (PostgreSQL) and MSK')

New in Rasa-Pro-Services 3.6

You can use IAM authentication to connect to AWS RDS (PostgreSQL) and MSK without needing to provide static credentials.

If your Rasa Pro Services (Analytics) instance is running on an AWS service that supports IAM roles (e.g. EC2), you can use IAM authentication to connect to an AWS RDS database and MSK cluster without needing to provide a username and password.

To do so, you need to ensure that:

1. your RDS database is configured to allow IAM authentication. Once your RDS instance is up, connect to it and run the following SQL command to enable IAM authentication for the database user you want to use:

```
GRANT rds_iam TO <db_username>;
```

2. your MSK cluster is configured to allow IAM authentication.
3. ensure that the topic you want to use is created on the MSK cluster.

You also need to set up your Rasa Pro Services instance with an appropriate IAM role that has permission to access the RDS database and MSK cluster. The IAM role should include the following permissions to allow the IAM role to generate the temporary credentials for each service:

```
{    "Version": "2012-10-17",    "Statement": [        {            "Effect": "Allow",            "Action": [                "rds-db:connect"            ],            "Resource": [                "arn:aws:rds-db:<region>:<account-id>:dbuser:<DbiResourceId for RDS PostgreSQL>/<db_username>"            ]        },{            "Effect": "Allow",            "Action": [                "kafka-cluster:Connect",                "kafka-cluster:DescribeCluster"            ],            "Resource": [                "arn:aws:kafka:<region_name>:<account_id>:cluster/<cluster_name>/<cluster_uuid>"            ]        },        {            "Effect": "Allow",            "Action": [                "kafka-cluster:*Topic*",                "kafka-cluster:WriteData",                "kafka-cluster:ReadData"            ],            "Resource": [                "arn:aws:kafka:<region_name>:<account_id>:topic/<cluster_name>/*"            ]        },        {            "Effect": "Allow",            "Action": [                "kafka-cluster:AlterGroup",                "kafka-cluster:DescribeGroup"            ],            "Resource": [                "arn:aws:kafka:<region_name>:<account_id>:group/<cluster_name>/*"            ]        }    ]}
```

Once you have set up the IAM role and configured your RDS database and MSK cluster, you can configure the following environment variables in your Rasa instance:

- `IAM_CLOUD_PROVIDER`: Set this to `aws` to enable IAM authentication.
- `AWS_DEFAULT_REGION`: Set this to the AWS region where your RDS database and MSK cluster are located.
- `RASA_ANALYTICS_DB_SSL_MODE`: the SSL mode to use when connecting to the database, e.g. `verify-full` or `verify-ca`
- `RASA_ANALYTICS_DB_SSL_CA_LOCATION`: the path to the SSL root certificate to use when connecting to the database. This can be downloaded for the particular region from [here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesAllRegions).
- `KAFKA_SECURITY_PROTOCOL`: Set this to `SASL_SSL` to use SASL over SSL.
- `KAFKA_SASL_MECHANISM`: Set this to `OAUTHBEARER` to use IAM authentication.
- `KAFKA_SSL_CA_LOCATION`: the path to the SSL root certificate to use when connecting to MSK. This can be downloaded from Amazon Trust Services.

When you start your Rasa Pro Services instance, it will use the IAM role to generate temporary credentials to log in to the RDS database and MSK cluster instead of using static credentials. The temporary credentials will be automatically refreshed every 15 minutes.

### Kubernetes deployment[​](https://rasa.com/docs/reference/integrations/analytics/#kubernetes-deployment 'Direct link to Kubernetes deployment')

The configuration of the assistant is the first step of [Installation and Configuration](https://rasa.com/docs/pro/deploy/deploy-kubernetes/). No additional configuration is required to connect the assistant to the Analytics pipeline. After the assistant is deployed, the Analytics pipeline will receive the data from the assistant and persist it to your data warehouse which will be configured in the next step.

## 2\. Connect a data warehouse[​](https://rasa.com/docs/reference/integrations/analytics/#2-connect-a-data-warehouse 'Direct link to 2. Connect a data warehouse')

You can choose to configure the analytics pipeline to stream the transformed conversational data to two different data warehouse types in AWS:

- [PostgreSQL](https://rasa.com/docs/reference/integrations/analytics/#postgresql)
- [Redshift](https://rasa.com/docs/reference/integrations/analytics/#redshift)

Additionally, any data warehouse is supported if it is configured in sync with your deployment of PostgreSQL, [as recommended for Redshift](https://rasa.com/docs/reference/integrations/analytics/#streaming-from-postgresql-to-redshift).

We have included brief instructions for syncing your PostgreSQL deployment with:

- [BigQuery](https://rasa.com/docs/reference/integrations/analytics/#bigquery)
- [Snowflake](https://rasa.com/docs/reference/integrations/analytics/#snowflake)

### PostgreSQL[​](https://rasa.com/docs/reference/integrations/analytics/#postgresql 'Direct link to PostgreSQL')

You can use Amazon Relational Database Service (RDS) to create a PostgreSQL DB instance which is the environment that will run your PostgreSQL database.

First, you must set up Amazon RDS by completing the instructions listed [here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html). Next, create the PostgreSQL DB instance. You can follow one of the following instruction sets:

- the AWS _Easy create_ instructions listed in the [**Creating a PostgreSQL DB instance** section](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html#CHAP_GettingStarted.Creating.PostgreSQL)
- the AWS [_Standard create_ instructions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html)

To build the DB URL in the DBAPI format specified in the [**Connect an assistance** section](https://rasa.com/docs/reference/integrations/analytics/#1-connect-an-assistant) you must enter the database credentials. You must obtain the database username and password after you select a database authentication option during the process of the PostgreSQL DB instance creation:

- **Password authentication** to use database credentials only, in which case you must enter a username for the master username, as well as generate the master password.
- [**Password and IAM DB authentication**](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html) to use IAM users and roles for the authentication of database users.
- [**Password and Kerberos authentication**](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/postgresql-kerberos.html)

Finally, when running the Analytics pipeline Docker container, set the [environment variable `RASA_ANALYTICS_DB_URL`](https://rasa.com/docs/pro/deploy/deploy-kubernetes/) to the PostgreSQL Amazon RDS DB instance URL.

### Redshift[​](https://rasa.com/docs/reference/integrations/analytics/#redshift 'Direct link to Redshift')

If you prefer instead to set up Amazon Redshift as your choice of data lake, you can choose to stream the data from the PostgreSQL source database within the Amazon RDS DB instance created at the [**PostgreSQL step**](https://rasa.com/docs/reference/integrations/analytics/#postgresql) to the Redshift target.

#### Streaming from PostgreSQL to Redshift[​](https://rasa.com/docs/reference/integrations/analytics/#streaming-from-postgresql-to-redshift 'Direct link to Streaming from PostgreSQL to Redshift')

If you meet the [prerequisites](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.Redshift.html#CHAP_Target.Redshift.Prerequisites) for using an Amazon Redshift database as a target, you will need to implement two steps:

1. configure the PostgreSQL source for AWS Database Migration Service (DMS) by following these [instructions](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.PostgreSQL.html)
2. configure the Redshift target for AWS DMS following the instructions [here](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.Redshift.html).

### BigQuery[​](https://rasa.com/docs/reference/integrations/analytics/#bigquery 'Direct link to BigQuery')

To stream data from PostgreSQL to BigQuery, you can use [Datastream for BigQuery](https://cloud.google.com/datastream/docs). Datastream for BigQuery supports several [PostgreSQL deployment types](https://cloud.google.com/datastream/docs/configure-your-source-postgresql-database#overview), including [CloudSQL](https://cloud.google.com/sql/docs/postgres).

Before you begin, make sure to check the Datastream [prerequisites](https://cloud.google.com/datastream/docs/before-you-begin), as well as additional Datastream networking connectivity [requirements](https://cloud.google.com/datastream/docs/quickstart-replication-to-bigquery#requirements).

You can closely follow [this quickstart guide](https://cloud.google.com/datastream/docs/quickstart-replication-to-bigquery) on replicating data from PostgreSQL CloudSQL to BigQuery with Datastream.

Alternatively, you can deep dive into the following Datastream set-up guides:

- [Configure your source PostgreSQL database](https://cloud.google.com/datastream/docs/configure-your-source-postgresql-database)
- Optional: use [customer-managed encryption keys](https://cloud.google.com/datastream/docs/use-cmek)
- Create a [connection profile for PostgreSQL database](https://cloud.google.com/datastream/docs/create-connection-profiles#cp4postgresdb)
- Create a [stream](https://cloud.google.com/datastream/docs/create-a-stream)

### Snowflake[​](https://rasa.com/docs/reference/integrations/analytics/#snowflake 'Direct link to Snowflake')

You can sync your PostgreSQL deployment manually or via an automated [partner solution](https://docs.snowflake.com/en/user-guide/ecosystem-etl.html).

The instructions for manual sync include the following steps:

- Extract data from PostgreSQL to file using `COPY INTO` [command](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html). You should also explore the Snowflake [data loading best practices](https://docs.snowflake.com/en/user-guide/data-load-considerations.html) before extraction.
- Stage the extracted data files to either internal or external locations such as AWS S3, Google Cloud Storage or Microsoft Azure.
- Copy staged files to Snowflake tables using `COPY INTO` [command](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html). You can decide to use [bulk data loading](https://docs.snowflake.com/en/user-guide/data-load-bulk.html) into Snowflake or to load continuously using [Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe.html). Alternatively you can also benefit from [this plugin](https://cdap.atlassian.net/wiki/spaces/DOCS/pages/694157985/Cloud+Storage+to+Snowflake+Action) to load data to an existing Snowflake table.

## 3\. Ingest past conversations (optional)[​](https://rasa.com/docs/reference/integrations/analytics/#3-ingest-past-conversations-optional 'Direct link to 3. Ingest past conversations (optional)')

When Analytics is connected to your Kafka instance, it will consume all prior events on the Kafka topic and ingest them into the database. Kafka has a retention policy for events on a [topic which defaults to 7 days](https://docs.confluent.io/platform/current/installation/configuration/topic-configs.html#topicconfigs_retention.ms).

If you want to process events from conversations that are older than the retention policy configured for the Rasa topic, you can manually ingest events from past conversations.

Manually ingesting data from past conversations requires a connection to the tracker store. The tracker store contains past conversations and a connection to the Kafka cluster. Use the `rasa export` command to export the events stored in the tracker store to Kafka:

```
rasa export --endpoints endpoints.yml
```

Configure the export to read from your production tracker store and write to Kafka as an event broker, e.g.

endpoints.yml

```
  tracker_store:      type: SQL      dialect: "postgresql"      url: "localhost"      db: "tracker"      username: postgres      password: password  event_broker:      type: kafka      topic: rasa-events      url: localhost:29092      partition_by_sender: true
```

note

Running manual ingestion of past events multiple times will result in duplicated events. There is currently no deduplication implemented in Analytics. Every ingested event will be stored in the database, even if it was processed previously.

## 4\. Connect a BI Solution[​](https://rasa.com/docs/reference/integrations/analytics/#4-connect-a-bi-solution 'Direct link to 4. Connect a BI Solution')

Connecting a business intelligence platform to the data warehouse varies for each platform. We provide example instructions for Metabase and Tableau but you can use any BI platform which supports AWS Redshift or PostgreSQL.

### Example: Metabase[​](https://rasa.com/docs/reference/integrations/analytics/#example-metabase 'Direct link to Example: Metabase')

Metabase is a free and open-source business intelligence platform. It provides a simple interface to query and visualize data. Metabase can be connected to PostgreSQL or Redshift databases.

- [Connecting Metabase to PostgreSQL](https://www.metabase.com/data_sources/postgresql)
- [Connecting Metabase to Redshift](https://www.metabase.com/data_sources/amazon-redshift)

### Example: Tableau[​](https://rasa.com/docs/reference/integrations/analytics/#example-tableau 'Direct link to Example: Tableau')

Tableau is a business intelligence platform. It provides a flexible interface to build business intelligence dashboards. Tableau can be connected to PostgreSQL or Redshift databases.

- [Connecting Tableau to PostgreSQL](https://help.tableau.com/current/pro/desktop/en-us/examples_postgresql.htm)
- [Connecting Tableau to Redshift](https://help.tableau.com/current/pro/desktop/en-us/examples_amazonredshift.htm)

## Kafka Dead-Letter-Queue Publishing Mechanism[​](https://rasa.com/docs/reference/integrations/analytics/#kafka-dead-letter-queue-publishing-mechanism 'Direct link to Kafka Dead-Letter-Queue Publishing Mechanism')

Rasa Analytics supports publishing events that could not be processed to a dead-letter-queue (DLQ) topic in Kafka. This allows you to inspect the events that could not be processed and take appropriate actions.

To enable the DLQ publishing mechanism, you need to set the `RASA_ANALYTICS_DLQ` environment variable to the name of the DLQ topic you want to use. The DLQ topic must be created before starting the Analytics pipeline. By default, the DLQ topic name is set to `rasa-analytics-dlq`.

When an event cannot be processed, it will be published to the DLQ topic with the original event payload and an additional header `original_offset` indicating the offset of the original event in the source topic.

Reasons for events being published to the DLQ topic include event processing errors such as:

- Malformed event payloads
- Database connection issues
- Data validation errors

Events that have been skipped due to errors handled during processing will also be published to the DLQ topic. An example of such a skipped event is a `stack` [event](https://rasa.com/docs/reference/primitives/events/#stack-event) that causes `JsonPointerException` or `JsonPatchConflict` errors during processing.

### How to reprocess events from the DLQ topic[​](https://rasa.com/docs/reference/integrations/analytics/#how-to-reprocess-events-from-the-dlq-topic 'Direct link to How to reprocess events from the DLQ topic')

If you want to reprocess events from the DLQ topic, we recommend you follow these steps:

1. Identify the minimum source topic offset for each partition among the events published to the DLQ topic. You can do this by consuming messages from the DLQ topic and inspecting the `original_offset` header.
2. Stop the Rasa Analytics pipeline while you reset the source topic offset identified for each partition to the minimum offset identified in step 1.
3. Restart the Rasa Analytics pipeline. It will start consuming events from the source topic starting from the partition offsets you set in step 2, including the events that were previously published to the DLQ topic. Events that had been successfully processed prior will be skipped, while the DLQ events that could not be processed will be re-processed and published to the DLQ topic again if they still cannot be processed.
4. Once you have reprocessed the events, you can choose to mark them as processed by committing the offsets in the DLQ topic or deleting the messages from the DLQ topic.

## Retry Mechanism In Event of Failures[​](https://rasa.com/docs/reference/integrations/analytics/#retry-mechanism-in-event-of-failures 'Direct link to Retry Mechanism In Event of Failures')

Rasa Analytics includes a retry mechanism to handle transient failures that may occur during event processing. The retry mechanism is designed to automatically retry failed operations a configurable number of times before giving up and publishing the event to the dead-letter-queue (DLQ) topic.

The number of retries can be configured using the `RETRY_DB_TRANSACTION_COUNT` and `RETRY_CONNECTION_COUNT` environment variables, respectively. By default, both variables are set to `7`.

The `RETRY_DB_TRANSACTION_COUNT` variable controls how many times a database transaction is retried when it fails, while the `RETRY_CONNECTION_COUNT` variable controls how many times the connection to the Kafka broker is retried when the broker is not available during operations such as message consumption, message acknowledgement, and DLQ publishing.

The retry mechanism uses an exponential backoff strategy to avoid overwhelming the system with retries. The backoff time increases exponentially with each retry attempt. Note that the retry mechanism will have an impact on the throughput event processing, so it should be configured appropriately based on the expected load and performance requirements of your system.

## Database Migration Best Practices[​](https://rasa.com/docs/reference/integrations/analytics/#database-migration-best-practices 'Direct link to Database Migration Best Practices')

### Troubleshooting Common Issues[​](https://rasa.com/docs/reference/integrations/analytics/#troubleshooting-common-issues 'Direct link to Troubleshooting Common Issues')

When database migrations and index creation are handled automatically at runtime, customers with large databases and multi-replica deployments may experience the following issues:

#### Duplicate Sender Records[​](https://rasa.com/docs/reference/integrations/analytics/#duplicate-sender-records 'Direct link to Duplicate Sender Records')

Customers may observe duplicate `sender_key` entries in the `rasa_sender` table, violating the expectation of uniqueness. This can lead to runtime failures such as:

```
MultipleResultsFound: Multiple rows were found when one or none was required
```

**Impact:** Analytics event processing fails when attempting to query sender records.

#### Runtime Database Migrations[​](https://rasa.com/docs/reference/integrations/analytics/#runtime-database-migrations 'Direct link to Runtime Database Migrations')

Runtime logs may intermittently show messages like:

```
Running database migrations...
```

These messages appear while services are already running, not just on startup. This raises concerns that migrations may be re-running unexpectedly, risking data consistency and stability.

#### Pod Restarts During Load Testing[​](https://rasa.com/docs/reference/integrations/analytics/#pod-restarts-during-load-testing 'Direct link to Pod Restarts During Load Testing')

During load tests, multiple restarts may occur in the Rasa Pro Services pod. Logs indicate failures tied to database operations, suggesting that concurrent database access during migrations can cause instability.

#### Duplicate Session Start Events[​](https://rasa.com/docs/reference/integrations/analytics/#duplicate-session-start-events 'Direct link to Duplicate Session Start Events')

The `_rasa_raw_event` table may contain multiple duplicate "session\_started" events for the same `sender_key`. This suggests possible race conditions or duplicate initialization during pod restarts or migrations.

#### Index Creation Causing Crash Loops[​](https://rasa.com/docs/reference/integrations/analytics/#index-creation-causing-crash-loops 'Direct link to Index Creation Causing Crash Loops')

For large databases (e.g., ~170 GB), index creation can take many minutes. During this time:

- Liveness and readiness probes fail
- Kubernetes restarts the pod
- All replicas attempt migrations simultaneously, opening many database connections and hitting connection limits
- Index creation is interrupted
- Pods enter a crash loop
- Rasa Pro Services cannot start successfully

Additionally, duplicate `sender_id` errors can occur due to concurrency issues when Rasa Pro Services restarts after a crash or interruption, causing events to be processed more than once.

### Best Practices[​](https://rasa.com/docs/reference/integrations/analytics/#best-practices 'Direct link to Best Practices')

To avoid these issues, follow these best practices:

#### Run Migrations Outside the Main Application Lifecycle[​](https://rasa.com/docs/reference/integrations/analytics/#run-migrations-outside-the-main-application-lifecycle 'Direct link to Run Migrations Outside the Main Application Lifecycle')

**Do not run database migrations at runtime.** Instead, run migrations before application pods start by using the following approach:

- **Dedicated migration job:** Use a Kubernetes Job to run migrations separately from the application deployment. This allows you to:
 - Run migrations as a one-time operation
 - Monitor migration completion independently
 - Ensure migrations complete before scaling up application replicas

To disable automatic migrations at runtime, set the `RUN_ANALYTICS_DB_MIGRATIONS` environment variable to `false`. For example, update the helm chart `values.yaml` under the `additionalEnv` section of the `rasaProServices` configuration:

values.yaml

```
additionalEnv:  - name: RUN_ANALYTICS_DB_MIGRATIONS    value: "false"
```

#### Verify Migration Completion After Upgrade[​](https://rasa.com/docs/reference/integrations/analytics/#verify-migration-completion-after-upgrade 'Direct link to Verify Migration Completion After Upgrade')

After upgrading Rasa Pro Services, ensure database migrations complete successfully by verifying that the following indexes are present in the relevant tables:

- A composite index on the `session_id` and `timestamp ASC` columns of the `rasa_event` table
- A composite index on the `sender_id`, `session_id` and `timestamp ASC` columns of the `rasa_event` table
- A composite index on the `sender_id` and `sequence_number` columns of the `rasa_event` table
- An index on the `sender_key` column of the `rasa_sender` table
- A composite index on the `sender_id` and `start_sequence_number DESC` columns of the `rasa_session` table
- A composite index on the `sender_id`, `session_id` and `start_sequence_number DESC` columns of the `rasa_turn` table
- A composite index on the `sender_id`, `session_id` and `start_sequence_number DESC` columns of the `rasa_dialogue_stack_frame` table

#### Optimize Consumer Configuration[​](https://rasa.com/docs/reference/integrations/analytics/#optimize-consumer-configuration 'Direct link to Optimize Consumer Configuration')

Reduce the number of Kafka consumers and partitions to three, as this configuration is already optimized. Increase them only if you observe significant lag in event processing. This helps prevent excessive database connections during migrations and reduces the risk of connection limit issues.

- [Types of metrics](https://rasa.com/docs/reference/integrations/analytics/#types-of-metrics)
- [Database Indexes](https://rasa.com/docs/reference/integrations/analytics/#database-indexes)
- [Prerequisites](https://rasa.com/docs/reference/integrations/analytics/#prerequisites)
- [1\. Connect an assistant](https://rasa.com/docs/reference/integrations/analytics/#1-connect-an-assistant)
 - [Configuration](https://rasa.com/docs/reference/integrations/analytics/#configuration)
 - [Example Configurations](https://rasa.com/docs/reference/integrations/analytics/#example-configurations)
 - [Kubernetes deployment](https://rasa.com/docs/reference/integrations/analytics/#kubernetes-deployment)
- [2\. Connect a data warehouse](https://rasa.com/docs/reference/integrations/analytics/#2-connect-a-data-warehouse)
 - [PostgreSQL](https://rasa.com/docs/reference/integrations/analytics/#postgresql)
 - [Redshift](https://rasa.com/docs/reference/integrations/analytics/#redshift)
 - [BigQuery](https://rasa.com/docs/reference/integrations/analytics/#bigquery)
 - [Snowflake](https://rasa.com/docs/reference/integrations/analytics/#snowflake)
- [3\. Ingest past conversations (optional)](https://rasa.com/docs/reference/integrations/analytics/#3-ingest-past-conversations-optional)
- [4\. Connect a BI Solution](https://rasa.com/docs/reference/integrations/analytics/#4-connect-a-bi-solution)
 - [Example: Metabase](https://rasa.com/docs/reference/integrations/analytics/#example-metabase)
 - [Example: Tableau](https://rasa.com/docs/reference/integrations/analytics/#example-tableau)
- [Kafka Dead-Letter-Queue Publishing Mechanism](https://rasa.com/docs/reference/integrations/analytics/#kafka-dead-letter-queue-publishing-mechanism)
 - [How to reprocess events from the DLQ topic](https://rasa.com/docs/reference/integrations/analytics/#how-to-reprocess-events-from-the-dlq-topic)
- [Retry Mechanism In Event of Failures](https://rasa.com/docs/reference/integrations/analytics/#retry-mechanism-in-event-of-failures)
- [Database Migration Best Practices](https://rasa.com/docs/reference/integrations/analytics/#database-migration-best-practices)
 - [Troubleshooting Common Issues](https://rasa.com/docs/reference/integrations/analytics/#troubleshooting-common-issues)
 - [Best Practices](https://rasa.com/docs/reference/integrations/analytics/#best-practices)