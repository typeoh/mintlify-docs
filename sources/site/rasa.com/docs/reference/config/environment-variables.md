# Source: https://rasa.com/docs/reference/config/environment-variables

On this page

When running Rasa, the `RASA_LICENSE` environment variable needs to contain the Rasa license key.

### Backing services[​](https://rasa.com/docs/reference/config/environment-variables/#backing-services 'Direct link to Backing services')

| Environment Variable | Description | Default |
| --- | --- | --- |
| `POSTGRESQL_SCHEMA` | The postgres schema for the [`SQLTrackerStore`](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore). | `"public"` |
| `POSTGRESQL_POOL_SIZE` | The limit of the number of open connections for the [`SQLTrackerStore`](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore). | `50` |
| `POSTGRESQL_MAX_OVERFLOW` | The [maximum overflow](https://docs.sqlalchemy.org/en/13/core/pooling.html#sqlalchemy.pool.QueuePool.params.max_overflow) of pool size for the [`SQLTrackerStore`](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore). | `100` |
| `RABBITMQ_SSL_CLIENT_CERTIFICATE` | The path to the SSL client certificate for the [`PikaEventBroker`](https://rasa.com/docs/reference/integrations/event-brokers/#pika-event-broker-for-rabbitmq). | |
| `RABBITMQ_SSL_CLIENT_KEY` | The path to the SSL client key for the [`PikaEventBroker`](https://rasa.com/docs/reference/integrations/event-brokers/#pika-event-broker-for-rabbitmq). | |
| `RASA_ENVIRONMENT` | The Rasa environment for the [`PikaEventBroker`](https://rasa.com/docs/reference/integrations/event-brokers/#pika-event-broker-for-rabbitmq) and the [`KafkaEventBroker`](https://rasa.com/docs/reference/integrations/event-brokers/#kafka-event-broker). | |
| `SECRET_MANAGER` | The type of [secret manager](https://rasa.com/docs/reference/integrations/secrets-managers/). | `"vault"` |
| `TICKET_LOCK_LIFETIME` | The lifetime of ticket locks, in seconds. It configures the [lock store](https://rasa.com/docs/reference/integrations/lock-stores/). | `60` |
| `VAULT_URL` | The URl of the [HashiCorp Vault](https://rasa.com/docs/reference/integrations/secrets-managers/#hashicorp-vault-secrets-manager). | |
| `VAULT_TOKEN` | The token to authenticate to the [HashiCorp Vault](https://rasa.com/docs/reference/integrations/secrets-managers/#hashicorp-vault-secrets-manager). | |
| `VAULT_NAMESPACE` | The namespace of the [HashiCorp Vault](https://rasa.com/docs/reference/integrations/secrets-managers/#hashicorp-vault-secrets-manager). | |
| `VAULT_RASA_SECRETS_PATH` | The path to Rasa secrets in the [HashiCorp Vault](https://rasa.com/docs/reference/integrations/secrets-managers/#hashicorp-vault-secrets-manager). | `"rasa-secrets"` |
| `VAULT_TRANSIT_MOUNT_POINT` | The mount point of the [HashiCorp Vault](https://rasa.com/docs/reference/integrations/secrets-managers/#hashicorp-vault-secrets-manager). | |
| `IAM_CLOUD_PROVIDER` | The cloud provider for IAM authentication. Supported value is `aws`. | |
| `AWS_DEFAULT_REGION` | The AWS region for IAM authentication. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. | |
| `ELASTICACHE_REDIS_AWS_IAM_ENABLED` | Set to `true` to enable IAM authentication for ElastiCache Redis connections. | |
| `AWS_ELASTICACHE_CLUSTER_NAME` | The AWS ElastiCache Redis cluster or replication group name. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. | |
| `RDS_SQL_DB_AWS_IAM_ENABLED` | Set to `true` to enable IAM authentication for RDS connections. | |
| `SQL_TRACKER_STORE_SSL_MODE` | The SSL mode for the [`SQLTrackerStore`](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore). Check your deployed SQL database documentation for supported values. For example: see [PostgreSQL SSL Mode descriptions](https://www.postgresql.org/docs/current/libpq-ssl.html#LIBPQ-SSL-PROTECTION-MODES) | |
| `SQL_TRACKER_STORE_SSL_ROOT_CERTIFICATE` | The path to the root certificate for the [`SQLTrackerStore`](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore). | |
| `KAFKA_MSK_AWS_IAM_ENABLED` | Set to `true` to enable IAM authentication for MSK connections. | |

### Model Storage[​](https://rasa.com/docs/reference/config/environment-variables/#model-storage 'Direct link to Model Storage')

Environment variables that configure model storage on cloud providers are described in the following sections:

- [Amazon S3 Storage](https://rasa.com/docs/reference/integrations/model-storage/#amazon-s3-storage)
- [Google Cloud Storage](https://rasa.com/docs/reference/integrations/model-storage/#google-cloud-storage)
- [Azure Storage](https://rasa.com/docs/reference/integrations/model-storage/#azure-storage)

### Dialogue Management[​](https://rasa.com/docs/reference/config/environment-variables/#dialogue-management 'Direct link to Dialogue Management')

| Environment Variable | Description | Default |
| --- | --- | --- |
| `RASA_DUCKLING_HTTP_URL` | The URL of the Duckling service powering the [`DucklingEntityExtractor`](https://rasa.com/docs/reference/config/components/nlu-components/#ducklingentityextractor). | |
| `MAX_NUMBER_OF_PREDICTIONS` | The maximum of predictions after each user message for a NLU-based assistant. See [Action Selection](https://rasa.com/docs/reference/config/policies/overview/#action-selection). | `10` |
| `MAX_NUMBER_OF_PREDICTIONS_CALM` | The maximum of predictions after each user message for a CALM-based assistant. See [Action Selection](https://rasa.com/docs/reference/config/policies/overview/#action-selection). | `1000` |

### Observability[​](https://rasa.com/docs/reference/config/environment-variables/#observability 'Direct link to Observability')

| Environment Variable | Description | Default |
| --- | --- | --- |
| `LOG_LEVEL` | The log level for Rasa and Rasa Pro. | `"INFO"` |
| `LOG_LEVEL_LIBRARIES` | The log level for 3rd-party libraries. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#setting-log-levels). | `"ERROR"` |
| `LOG_LEVEL_KAFKA` | The log level for `kafka` library. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#setting-log-levels). | `"ERROR"` |
| `LOG_LEVEL_RABBITMQ` | The log level for `rabbitmq` library. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#setting-log-levels). | `"ERROR"` |
| `LOG_LEVEL_PYMONGO` | The log level for `pymongo` library. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#setting-log-levels). | `"ERROR"` |
| `LOG_LEVEL_LLM` | The log level for all `LLM` components. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#log-level-llm-components). | `"DEBUG"` |
| `LOG_LEVEL_LLM_COMMAND_GENERATOR` | The log level for `LLMCommandGenerator` prompt. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#log-level-llm-components). | `"DEBUG"` |
| `LOG_LEVEL_LLM_ENTERPRISE_SEARCH` | The log level for `EnterpriseSearchPolicy` prompt. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#log-level-llm-components). | `"DEBUG"` |
| `LOG_LEVEL_LLM_INTENTLESS_POLICY` | The log level for `IntentlessPolicy` prompt. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#log-level-llm-components). | `"DEBUG"` |
| `LOG_LEVEL_LLM_REPHRASER` | The log level for `ContextualResponseRephraser` prompt. More info [here](https://rasa.com/docs/reference/api/command-line-interface/#log-level-llm-components). | `"DEBUG"` |
| `RASA_TELEMETRY_ENABLED` | Toggle telemetry reporting. More info [here](https://rasa.com/docs/reference/telemetry/) | `true` |
| `RASA_TELEMETRY_DEBUG` | Toggle debug information for telemetry reporting. | `false` |
| `TRACING_SERVICE_NAME` | The top-level service name when sending traces. More info [here](https://rasa.com/docs/reference/integrations/tracing/) | `"rasa"` |
| `RASA_TRACING_DEBUGGING_ENABLED` | Allow tracing of `action_metadata` attribute in traces of predicted actions by `EnterpriseSearchPolicy`. | `false` |

### Beta Feature Flags[​](https://rasa.com/docs/reference/config/environment-variables/#beta-feature-flags 'Direct link to Beta Feature Flags')

| Environment Variable | Description | Default |
| --- | --- | --- |
| `RASA_PRO_BETA_STUB_CUSTOM_ACTION` | Stub custom actions during E2E Testing. More info [here](https://rasa.com/docs/reference/testing/test-cases/#stubbing-custom-actions). | `false` |
| `RASA_PRO_BETA_E2E_CONVERSION` | Convert ideal sample conversations into e2e test cases. More info [here](https://rasa.com/docs/reference/testing/test-case-conversion/). | `false` |
| `RASA_PRO_BETA_FINE_TUNING_RECIPE` | Ensure comprehensive coverage of your system for fine-tuning. More info [here](https://rasa.com/docs/pro/customize/fine-tuning-llm/). | `false` |
| `RASA_PRO_BETA_REPEAT_COMMAND` | Voice conversational repair pattern which defines the behaviour when a user asks the assistant to repeat the last message. More info [here](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference). | `false` |
| `RASA_PRO_BETA_VIER` | Enables usage of the `vier_cvg` voice connector. | `false` |

### Advanced configuration[​](https://rasa.com/docs/reference/config/environment-variables/#advanced-configuration 'Direct link to Advanced configuration')

| Environment Variable | Description | Default |
| --- | --- | --- |
| `COMPRESS_ACTION_SERVER_REQUEST` | Toggle compression of HTTP requests sent to the [Action Server](https://rasa.com/docs/reference/integrations/action-server/actions/). | `false` |
| `RASA_MAX_CACHE_SIZE` | The maximum size of the training cache, in MB. | `1000` |
| `RASA_CACHE_NAME` | The filename of the training cache. | `"cache.db"` |
| `RASA_CACHE_DIRECTORY` | The directory for the training cache. | `".rasa/cache/"` |
| `RASA_SHELL_STREAM_READING_TIMEOUT_IN_SECONDS` | The stream reading timeout for [`rasa shell`](https://rasa.com/docs/reference/api/command-line-interface/#rasa-shell), in seconds. | `10` |
| `SANIC_BACKLOG` | The number of unaccepted connections that the [HTTP server](https://rasa.com/docs/reference/api/pro/rasa-pro-rest-api/) and [NLG server](https://rasa.com/docs/reference/integrations/nlg/) will allow before refusing new connections. | `100` |
| `SANIC_WORKERS` | The number of HTTP workers when enabling the [HTTP server](https://rasa.com/docs/reference/api/pro/rasa-pro-rest-api/). | `1` |
| `READ_YAML_FILE_CACHE_MAXSIZE` | The maximum size of the LRU (Least Recently Used) cache for reading and parsing YAML files. | `256` |

## Rasa Pro Services[​](https://rasa.com/docs/reference/config/environment-variables/#rasa-pro-services 'Direct link to Rasa Pro Services')

The Rasa Pro Services docker container supports configuration through several environment variables. The following table lists the available environment variables:

| Environment Variable | Description | Default |
| --- | --- | --- |
| `RASA_LICENSE` | **Required**. The license key for Rasa Pro Services. | |
| `KAFKA_BROKER_ADDRESS` | **Required**. The address of the Kafka broker. | |
| `KAFKA_TOPIC` | **Required**. The topic Rasa publishes events to and Rasa consumes from. | `rasa_core_events` |
| `LOGGING_LEVEL` | Set the log level of the application. Valid levels are DEBUG, INFO, WARNING, ERROR, CRITICAL. (Available from 3.0.2) | `INFO` |
| `FORCE_JSON_LOGGING` | Whether or not to use json logging. | |
| `RASA_ANALYTICS_DB_URL` | The URL of the data lake to store analytics data in. | |
| `RASA_ANALYTICS_CONSUMER_ID` | The Consumer group name used when registering the consumers with Kafka. | `rasa-analytics-group` |
| `RASA_ANALYTICS_CONSUMER_COUNT` | The number of Kafka consumers. | `1` |
| `RASA_ANALYTICS_DLQ` | The Queue used to publish events that resulted in a processing failure. | `rasa-analytics-dlq` |
| `KAFKA_SASL_MECHANISM` | The SASL mechanism to use for authentication. | |
| `KAFKA_SASL_USERNAME` | The username for SASL authentication. | |
| `KAFKA_SASL_PASSWORD` | The password for SASL authentication. | |
| `KAFKA_SECURITY_PROTOCOL` | The security protocol to use for communication with Kafka. Supported mechanisms are `PLAINTEXT`, `SASL_PLAINTEXT`, `SASL_SSL` and `SSL` | |
| `KAFKA_SSL_CA_LOCATION` | The filepath for SSL CA Certificate that will be used to connect with Kafka (Available from `3.1.0b1`) | |
| `KAFKA_SSL_CERTFILE_LOCATION` | The filepath for SSL client Certificate that will be used to connect with Kafka (Available from `3.6.0`) | |
| `KAFKA_SSL_KEYFILE_LOCATION` | The filepath for SSL Keyfile that will be used to connect with Kafka (Available from `3.6.0`) | |
| `IAM_CLOUD_PROVIDER` | The cloud provider for IAM authentication. Supported value is `aws` (Available from `3.6.0`) | |
| `AWS_DEFAULT_REGION` | The AWS region for IAM authentication. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. (Available from `3.6.0`) | |
| `RASA_ANALYTICS_DB_HOST_NAME` | The hostname of the PostgreSQL db to store analytics data in. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. (Available from `3.6.0`) | |
| `RASA_ANALYTICS_DB_PORT` | The port of the PostgreSQL db to store analytics data in. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. (Available from `3.6.0`) | |
| `RASA_ANALYTICS_DB_NAME` | The database name of the PostgreSQL db to store analytics data in. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. (Available from `3.6.0`) | |
| `RASA_ANALYTICS_DB_USERNAME` | The username of the PostgreSQL db to store analytics data in. Required if `IAM_CLOUD_PROVIDER` is set to `aws`. (Available from `3.6.0`) | |
| `RASA_ANALYTICS_DB_SSL_MODE` | The SSL mode of the PostgreSQL db to store analytics data in. (Available from `3.6.0`) | |
| `RASA_ANALYTICS_DB_SSL_CA_LOCATION` | The path to the root certificate of the PostgreSQL db to store analytics data in. (Available from `3.6.0`) | |
| `RDS_SQL_DB_AWS_IAM_ENABLED` | Set to `true` to enable IAM authentication for RDS connections. | |
| `KAFKA_MSK_AWS_IAM_ENABLED` | Set to `true` to enable IAM authentication for MSK connections. | |
| `RUN_ANALYTICS_DB_MIGRATIONS` | Set to `false` to not run the analytics database migrations on startup. | `true` |

## Rasa Studio[​](https://rasa.com/docs/reference/config/environment-variables/#rasa-studio 'Direct link to Rasa Studio')

These environment variables take precedence over the values in the `global.yml` file, that is created with `rasa studio config`.

| Environment Variable | Description | Default |
| --- | --- | --- |
| `RASA_STUDIO_AUTH_SERVER_URL` | (Rasa Pro) URL of the Authentication Server. | |
| `RASA_STUDIO_CLI_STUDIO_URL` | (Rasa Pro) URL of the Studio Data Endpoint. | |
| `RASA_STUDIO_CLI_REALM_NAME_KEY` | (Rasa Pro) Name of the Keycloak realm which hold the data about client ID and client secret. | |
| `RASA_STUDIO_CLI_CLIENT_ID_KEY` | (Rasa Pro) Client ID used to authenticate `rasa studio` CLI tool with Keycloak | |

## Rasa SDK[​](https://rasa.com/docs/reference/config/environment-variables/#rasa-sdk 'Direct link to Rasa SDK')

The Rasa SDK docker container supports configuration through several environment variables. The following table lists the available environment variables:

| Environment Variable | Description | Default |
| --- | --- | --- |
| `ACTION_SERVER_SANIC_WORKERS` | The number of Sanic HTTP workers in the Action server. | `1` |
| `LOG_LEVEL_LIBRARIES` | The log level of third-party libraries. See [Log level configuration](https://rasa.com/docs/reference/api/command-line-interface/#setting-log-levels). | `ERROR` |

- [Backing services](https://rasa.com/docs/reference/config/environment-variables/#backing-services)
- [Model Storage](https://rasa.com/docs/reference/config/environment-variables/#model-storage)
- [Dialogue Management](https://rasa.com/docs/reference/config/environment-variables/#dialogue-management)
- [Observability](https://rasa.com/docs/reference/config/environment-variables/#observability)
- [Beta Feature Flags](https://rasa.com/docs/reference/config/environment-variables/#beta-feature-flags)
- [Advanced configuration](https://rasa.com/docs/reference/config/environment-variables/#advanced-configuration)
- [Rasa Pro Services](https://rasa.com/docs/reference/config/environment-variables/#rasa-pro-services)
- [Rasa Studio](https://rasa.com/docs/reference/config/environment-variables/#rasa-studio)
- [Rasa SDK](https://rasa.com/docs/reference/config/environment-variables/#rasa-sdk)