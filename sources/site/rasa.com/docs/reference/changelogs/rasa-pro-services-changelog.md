# Source: https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog

On this page

All notable changes to Rasa Pro Services will be documented in this page. This product adheres to [Semantic Versioning](https://semver.org/) starting with version 3.3 (initial version).

## \[3.8.1\] - 2026-04-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#381---2026-04-17 'Direct link to [3.8.1] - 2026-04-17')

Rasa Pro Services 3.8.1 (2026-04-17)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes 'Direct link to Bugfixes')

- Updated the following Poetry dependencies with vulnerability patches:
 
 - `requests` to `2.33.1` fixing CVE-2026-25645.
 - `cryptography` to `46.0.7` fixing CVE-2026-39892.
 - `setuptools` to `82.0.1`
 
 Additionally, updated pip installed in Dockerfile to `==26.x` fixing CVE-2026-24049 vulnerability in `wheel` and updated `setuptools` to the version locked in `poetry.lock` which updated `jaraco.context` dependency to `6.1.0` fixing CVE-2026-23949.
 

## \[3.8.0\] - 2026-03-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#380---2026-03-26 'Direct link to [3.8.0] - 2026-03-26')

Rasa Pro Services 3.8.0 (2026-03-26)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements 'Direct link to Improvements')

- Add the ability to disable stack event processing by the analytics service through an environment variable. Set the `DISABLE_STACK_EVENT_PROCESSING` variable to `true` to disable analytics stack event processing. By default, stack event processing is enabled.
- Add new composite and covering indexes to existing tables in Rasa Analytics to improve the performance of high-load analytics queries and reduce CPU utilization. These indexes will complement the current indexes and help optimize queries that frequently filter by sender\_id, timestamp or session\_id.
- Add a new stack event query for the `event_was_processed` check to handle edge cases when stack events are stored in the `rasa_event` table but not in the `rasa_dialogue_stack_frame` table because of being skipped during processing and published to the DLQ. This allows the DLQ stack events to be reprocessed correctly when the source Kafka topic offset is reset to the minimum offset among the events published to the DLQ.
- Update Analytics session processing to use the `session_id` provided by Rasa Pro 3.16 in `event.metadata.session_id` directly as the session identifier, replacing the Analytics-generated UUID. A new `rasa_user` table is introduced to track end users across multiple conversations, populated from the `user_id` field on Kafka event payloads. Events from older Rasa versions without `metadata.session_id` continue to be processed correctly via the existing heuristic session detection.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-1 'Direct link to Bugfixes')

- Upgrade `urllib3`, `werkzeug` and `brotli` to fix security vulnerabilities.
- Update urllib3 to 2.6.3 to address CVE-2026-21441. Update werkzeug to 3.1.5 to address CVE-2026-21860.
- Update flask to 3.1.3 to address CVE-2026-27205 Update werkzeug to 3.1.6 to address CVE-2026-27199 Update cryptography to 46.0.5 to address CVE-2026-26007 Update sentry-sdk to 2.42.1 to address CVE-2024-40647
- Pin Certifi to 2026.2.25 to address CVE-2024-39689 Updated PyJWT to 2.12.0 to address CVE-2026-32597

## \[3.7.0\] - 2025-11-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#370---2025-11-26 'Direct link to [3.7.0] - 2025-11-26')

Rasa Pro Services 3.7.0 (2025-11-26)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-1 'Direct link to Improvements')

- Add the original Rasa event message offset to the header of the message being sent to the Kafka dead-letter-queue. This allows easier tracing of the original message in case of errors. The offset header is named `original_offset`.
- Allow skipping of db migration run when starting the Analytics service by setting the environment variable RUN\_ANALYTICS\_DB\_MIGRATIONS to "false". This can be useful in scenarios where migrations have already been applied or when managing migrations separately.
- Add database indexes to analytics tables to improve query performance. The following indexes were added to speed up analytics throughput and reduce query execution time during event processing and transformation:
 - `idx_rasa_dialogue_stack_frame_sender_session_seq` on `rasa_dialogue_stack_frame` table
 - `idx_rasa_event_pattern_query` on `rasa_event` table
 - `idx_rasa_event_stack_query` on `rasa_event` table
 - `idx_rasa_event_was_processed` on `rasa_event` table
 - `idx_rasa_sender_sender_key` on `rasa_sender` table
 - `idx_rasa_session_sender_seq` on `rasa_session` table
 - `idx_rasa_turn_sender_session_seq` on `rasa_turn` table

## \[3.6.2\] - 2025-11-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#362---2025-11-25 'Direct link to [3.6.2] - 2025-11-25')

Rasa Pro Services 3.6.2 (2025-11-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-2 'Direct link to Bugfixes')

- Implement custom timestamp type inheriting from `sqlalchemy.types.TypeDecorator`. This custom type normalizes and denormalizes timestamp columns across all db tables to ensure consistent timezone handling during inserting and querying.

## \[3.6.1\] - 2025-10-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#361---2025-10-29 'Direct link to [3.6.1] - 2025-10-29')

Rasa Pro Services 3.6.1 (2025-10-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-3 'Direct link to Bugfixes')

- Catch all exceptions raised during DialogueStackUpdated event processing and skip the event if an exception occurs. Add the skipped events to the DLQ topic, while allowing other events in the batch to be processed normally.
- Update `urllib3` version to `2.5.0` to address security vulnerabilities CVE-2024-37891 & CVE-2025-50181.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.6.0\] - 2025-10-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#360---2025-10-10 'Direct link to [3.6.0] - 2025-10-10')

Rasa Pro Services 3.6.0 (2025-10-10)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features 'Direct link to Features')

- Add support for IAM authentication for AWS RDS database in the analytics service. To enable this feature, set the following environment variables:
 
 - `IAM_CLOUD_PROVIDER`: set to `aws` to enable IAM authentication for AWS RDS
 - `RASA_ANALYTICS_DB_HOST_NAME`: `<your-db-hostname>` the hostname of the RDS instance
 - `RASA_ANALYTICS_DB_PORT`: `<your-db-port>` the port of the RDS instance
 - `RASA_ANALYTICS_DB_NAME`: `<your-db-name>` the name of the database
 - `RASA_ANALYTICS_DB_USERNAME`: `<your-db-username>` the username to connect to the database
 - `AWS_DEFAULT_REGION`: `<your-aws-region>` the AWS region where the RDS instance is hosted
 
 Additionally, you can also set the following optional environment variables:
 
 - `RASA_ANALYTICS_DB_SSL_MODE`: the SSL mode to use when connecting to the database, e.g. `verify-full` or `verify-ca`
 - `RASA_ANALYTICS_DB_SSL_CA_LOCATION`: the path to the SSL root certificate to use when connecting to the database
 
 When these environment variables are set, the analytics service will use IAM authentication to connect to the AWS RDS database.
 
- Add support for IAM authentication for AWS Managed Streaming for Apache Kafka in the analytics service. To enable this feature, set the following environment variables:
 
 - `IAM_CLOUD_PROVIDER`: set to `aws` to enable IAM authentication for AWS MSK
 - `AWS_DEFAULT_REGION`: `<your-aws-region>` the AWS region where the MSK instance is hosted
 - `KAFKA_SECURITY_PROTOCOL`: set to `SASL_SSL` to use SASL over SSL
 - `KAFKA_SASL_MECHANISM`: set to `OAUTHBEARER` to use OAuth Bearer token authentication
 - `KAFKA_SSL_CA_LOCATION`: the path to the SSL root certificate to use when connecting to the MSK cluster, this can be downloaded from Amazon Trust Services.
 
 When these environment variables are set, the analytics service will use IAM authentication to generate temporary credentials to connect to AWS MSK.
 
- Configure Analytics service to connect to Kafka broker using mTLS. To enable this feature, set the following environment variables:
 
 - `KAFKA_SSL_CERTFILE_LOCATION`: Path to the client certificate file.
 - `KAFKA_SSL_KEYFILE_LOCATION`: Path to the client private key file.

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-2 'Direct link to Improvements')

- Add new environment variables for each AWS service integration (RDS, MSK) that indicates whether to use IAM authentication when connecting to the service:
 - `KAFKA_MSK_AWS_IAM_ENABLED` - set to `true` to enable IAM authentication for MSK connections.
 - `RDS_SQL_DB_AWS_IAM_ENABLED` - set to `true` to enable IAM authentication for RDS connections.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-4 'Direct link to Bugfixes')

- Refactor the logic for creating new sessions. It now gets the previous event context directly from the database which makes the logic more robust against cases where the in-memory event stream might not have the previous event.

## \[3.5.10\] - 2025-11-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#3510---2025-11-25 'Direct link to [3.5.10] - 2025-11-25')

Rasa Pro Services 3.5.10 (2025-11-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-5 'Direct link to Bugfixes')

- Implement custom timestamp type inheriting from `sqlalchemy.types.TypeDecorator`. This custom type normalizes and denormalizes timestamp columns across all db tables to ensure consistent timezone handling during inserting and querying.

## \[3.5.9\] - 2025-10-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#359---2025-10-29 'Direct link to [3.5.9] - 2025-10-29')

Rasa Pro Services 3.5.9 (2025-10-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-6 'Direct link to Bugfixes')

- Catch all exceptions raised during DialogueStackUpdated event processing and skip the event if an exception occurs. Add the skipped events to the DLQ topic, while allowing other events in the batch to be processed normally.
- Update `urllib3` version to `2.5.0` to address security vulnerabilities CVE-2024-37891 & CVE-2025-50181.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-1 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.5.8\] - 2025-10-10[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#358---2025-10-10 'Direct link to [3.5.8] - 2025-10-10')

Rasa Pro Services 3.5.8 (2025-10-10)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-7 'Direct link to Bugfixes')

- Refactor the logic for creating new sessions. It now gets the previous event context directly from the database which makes the logic more robust against cases where the in-memory event stream might not have the previous event.

## \[3.5.7\] - 2025-10-06[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#357---2025-10-06 'Direct link to [3.5.7] - 2025-10-06')

Rasa Pro Services 3.5.7 (2025-10-06)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-8 'Direct link to Bugfixes')

- Add the installation of `zlib` OS dependency to Analytics Dockerfile to fix build issues when running the service.

## \[3.5.6\] - 2025-09-11[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#356---2025-09-11 'Direct link to [3.5.6] - 2025-09-11')

Rasa Pro Services 3.5.6 (2025-09-11)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-9 'Direct link to Bugfixes')

- Upgrade alpine base image to 3.19 in order to a security vulnerability in alpine:3.17

## \[3.5.5\] - 2025-09-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#355---2025-09-03 'Direct link to [3.5.5] - 2025-09-03')

Rasa Pro Services 3.5.5 (2025-09-03)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-10 'Direct link to Bugfixes')

- Upgrade vulnerabilities:
 - `requests` to version `2.32.5`
 - `cryptography` to version `43.0.3`
- Fix vulnerabilities in outdated `setuptools` and `pip` installed via the `python:3.9-slim` original base image used by the Analytics Docker image build. Replace `python:3.9-slim` base image with `alpine:3.17` which installs python 3.10. Drop python 3.9 support which has reached its EOL.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-2 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.5.4\] - 2025-07-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#354---2025-07-29 'Direct link to [3.5.4] - 2025-07-29')

Rasa Pro Services 3.5.4 (2025-07-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-11 'Direct link to Bugfixes')

- Upgrade the following dependencies to fix security vulnerabilities in the analytics service:
 
 - `setuptools` to version 78.1.1
 - `gunicorn` to version 23.0.0
 - `redshift-connector` to version 2.1.8
- Check if the message has already been processed before processing it. Commit message offset only after it was processed successfully. If message processing fails retry the transaction. If the message offset commit fails, retry the command.
 
 Added environment variables:
 
 - `RETRY_CONNECTION_COUNT` - how many times the connection to the Kafka broker is retried when it fails, Confluent Kafka producer and consumer are recreated on each retry
 - `RETRY_DB_TRANSACTION_COUNT` - how many times the DB transaction is retried when it fails
 - `KAFKA_SOCKET_KEEP_ALIVE_ENABLED` - whether the socket keep alive is enabled for the Kafka connection. Corresponds to `socket.keepalive.enable` in librdkafka.
 - `KAFKA_METADATA_MAX_AGE_MS` - the maximum age of the Kafka metadata in milliseconds. Corresponds to `metadata.max.age.ms` in librdkafka.
 - `KAFKA_PRODUCER_RETRIES` - how many times the librdkafka producer retries sending a message when it fails. Corresponds to `retries` in librdkafka.
 - `KAFKA_PRODUCER_TIMEOUT_MS` - the timeout for the librdkafka Kafka producer in milliseconds. Corresponds to `request.timeout.ms` in librdkafka.
 - `KAFKA_PRODUCER_PARTITIONER` - the partitioner used by the librdkafka Kafka producer. Corresponds to `partitioner` in librdkafka.
 - `KAFKA_COMPRESSION_CODEC` - the compression codec used by the librdkafka Kafka producer. Corresponds to `compression.codec` in librdkafka.
 - `KAFKA_CONSUMER_HEARTBEAT_INTERVAL_MS` - the heartbeat interval for the librdkafka Kafka consumer in milliseconds. Corresponds to `heartbeat.interval.ms` in librdkafka.
 - `KAFKA_CONSUMER_SESSION_TIMEOUT_MS` - the session timeout for the librdkafka Kafka consumer in milliseconds. Corresponds to `session.timeout.ms` in librdkafka.
 - `KAFKA_CONSUMER_MAX_POLL_INTERVAL_MS` - the maximum poll interval for the librdkafka Kafka consumer in milliseconds. Corresponds to `max.poll.interval.ms` in librdkafka.
 - `MAX_MESSAGES_TO_FETCH` - how many messages are fetched from the queue at once, default is 10
 
 For more info about the librdkafka configuration options see [https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md).
 

## \[3.5.3\] - 2025-06-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#353---2025-06-27 'Direct link to [3.5.3] - 2025-06-27')

Rasa Pro Services 3.5.3 (2025-06-27)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-12 'Direct link to Bugfixes')

- Handle `JsonPatchConflict` exceptions gracefully when loading pre-existing dialogue stack events or when applying current event patches. This prevents the Analytics service from crashing due to conflicts in the JSON patch operations.

## \[3.5.2\] - 2025-05-21[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#352---2025-05-21 'Direct link to [3.5.2] - 2025-05-21')

Rasa Pro Services 3.5.2 (2025-05-21)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-13 'Direct link to Bugfixes')

- Fixed a bug in Rasa Analytics for Session Creation Logic. A session can be created by events `slot` event with the slot `session_started_metadata` followed by `action` event for `action_session_start` OR just the event `action` for `action_session_start` alone.

## \[3.5.1\] - 2025-05-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#351---2025-05-12 'Direct link to [3.5.1] - 2025-05-12')

Rasa Pro Services 3.5.1 (2025-05-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-14 'Direct link to Bugfixes')

- Fix the processing of stack events by the analytics service in the case of multiple parallel conversations whose events are split across several batches consumed by the service.
- Replace sqlalchemy's `DateTime` type with postgresql dialect specific `TIMESTAMP` type in columns that record date and time. This is required to keep fractional seconds precision which is essential when retrieving `stack` events from the database to reconstruct the dialogue stack for each different conversation id.

## \[3.5.0\] - 2025-03-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#350---2025-03-20 'Direct link to [3.5.0] - 2025-03-20')

Rasa Pro Services 3.5.0 (2025-03-20)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-3 'Direct link to Improvements')

- MTS: Update MTS to train without the need for NFS.
- MRS: Update MRS to run with bot config and model trained from remote storage instead of using NFS.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-15 'Direct link to Bugfixes')

- Updated `jinja2`, `werkzeug`, `idna`, `requests`, `zipp` and `urllib3` to address security vulnerabilities.

## \[3.4.1\] - 2025-05-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#341---2025-05-12 'Direct link to [3.4.1] - 2025-05-12')

Rasa Pro Services 3.4.1 (2025-05-12)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-16 'Direct link to Bugfixes')

- Updated `jinja2`, `werkzeug`, `idna`, `requests`, `zipp` and `urllib3` to address security vulnerabilities.
- Fix the processing of stack events by the analytics service in the case of multiple parallel conversations whose events are split across several batches consumed by the service.
- Replace sqlalchemy's `DateTime` type with postgresql dialect specific `TIMESTAMP` type in columns that record date and time. This is required to keep fractional seconds precision which is essential when retrieving `stack` events from the database to reconstruct the dialogue stack for each different conversation id.

## \[3.4.0\] - 2024-12-12[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#340---2024-12-12 'Direct link to [3.4.0] - 2024-12-12')

Rasa Pro Services 3.4.0 (2024-12-12)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-4 'Direct link to Improvements')

- Added Python 3.10 and Python 3.11 support to Rasa Analytics

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-17 'Direct link to Bugfixes')

- MTS and MRS: Add retry capability to Kubernetes API calls on `5xx` errors.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-3 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.3.5\] - 2024-10-29[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#335---2024-10-29 'Direct link to [3.3.5] - 2024-10-29')

Rasa Pro Services 3.3.5 (2024-10-29)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-18 'Direct link to Bugfixes')

- MTS and MRS: Add retry capability to Kubernetes API calls on `5xx` errors.

## \[3.3.4\] - 2024-10-02[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#334---2024-10-02 'Direct link to [3.3.4] - 2024-10-02')

Rasa Pro Services 3.3.4 (2024-10-02)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-19 'Direct link to Bugfixes')

- Return status code 503 when the Analytics service is unavailable during a healthcheck request. This allows the user to implement liveness probes that could automatically restart the service upon failure.

## \[3.3.3\] - 2024-09-25[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#333---2024-09-25 'Direct link to [3.3.3] - 2024-09-25')

Rasa Pro Services 3.3.3 (2024-09-25)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-20 'Direct link to Bugfixes')

- MTS: Fix bug (ATO-2257) when training pod's status is not caught when MTS consumer job restarts.
- \[MTS\] Update certifi to 2023.7.22 to resolve vulnerability CVE-2023-37920.

## \[3.3.2\] - 2024-05-28[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#332---2024-05-28 'Direct link to [3.3.2] - 2024-05-28')

Rasa Pro Services 3.3.2 (2024-05-28)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-5 'Direct link to Improvements')

- MRS: Upload rasa pod logs to remote storage for user-friendly access to logs in case the running of the assistant fails.

## \[3.3.1\] - 2024-05-27[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#331---2024-05-27 'Direct link to [3.3.1] - 2024-05-27')

Rasa Pro Services 3.3.1 (2024-05-27)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-21 'Direct link to Bugfixes')

- Allow users to specify image pull secrets for MTS / MRS

## \[3.3.0\] - 2024-04-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#330---2024-04-03 'Direct link to [3.3.0] - 2024-04-03')

Rasa Pro Services 3.3.0 (2024-04-03)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-6 'Direct link to Improvements')

- Align column names representing the `flow_id` as defined in the yaml file across `rasa_flow_status`, `rasa_llm_command` and `rasa_dialogue_stack_frame` tables:
 - `rasa_llm_command` table: `flow_name` column has been renamed to `flow_identifier`.
 - `rasa_dialogue_stack_frame` table: `active_flow` column has been renamed to `active_flow_identifier`.
- MTS: Handle MTS Job Consumer restarts when rasa training pod continues to run. If the `TrainingManager` finds a running training job when it restarts, it will check the status of the pod:
 - If the pod is pending or running, it will watch the pod until training is complete.
 - If the pod has completed, it will upload the logs and trained model (only if it is present).
- MTS: Accept `nlu` in the config data of CALM assistants in order to use `nlu_triggers`.
- MTS and MRS: Support debug logs in rasa pods.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-4 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.2.3\] - 2023-12-20[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#323---2023-12-20 'Direct link to [3.2.3] - 2023-12-20')

Rasa Pro Services 3.2.3 (2023-12-20)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-7 'Direct link to Improvements')

- MTS: Setup alembic schema migration mechanism for the database of the model training orchestrator. Add initial table creation migration file.
- \[MTS\] Add capability to configure log level for MTS orchestrator. Log level can be configured through `LOG_LEVEL` environment variable. Default log level is `INFO`.

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-22 'Direct link to Bugfixes')

- Fix telemetry reporting in shipped Docker images.

## \[3.2.2\] - 2023-12-05[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#322---2023-12-05 'Direct link to [3.2.2] - 2023-12-05')

Rasa Pro Services 3.2.2 (2023-12-05)

### Bugfixes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-23 'Direct link to Bugfixes')

- Remove obsolete component RemoteGCSFetcher from MTS orchestrator.

## \[3.2.1\] - 2023-12-01[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#321---2023-12-01 'Direct link to [3.2.1] - 2023-12-01')

Rasa Pro Services 3.2.1 (2023-12-01)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-8 'Direct link to Improvements')

- Align column names representing the `flow_id` as defined in the yaml file across `rasa_flow_status`, `rasa_llm_command` and `rasa_dialogue_stack_frame` tables:
 - `rasa_llm_command` table: `flow_name` column has been renamed to `flow_identifier`.
 - `rasa_dialogue_stack_frame` table: `active_flow` column has been renamed to `active_flow_identifier`.
- Add environment variables to control resource requirements and limits for Rasa pod. MTS and MRS job consumers can now be configured to use specify resource requirements and limits for the Rasa pod. This can be done by setting the following environment variables in the Rasa pod:
 - RASA\_REQUESTS\_CPU
 - RASA\_REQUESTS\_MEMORY
 - RASA\_LIMITS\_CPU
 - RASA\_LIMITS\_MEMORY

## \[3.2.0\] - 2023-11-22[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#320---2023-11-22 'Direct link to [3.2.0] - 2023-11-22')

Rasa Pro Services 3.2.0 (2023-11-22)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features-1 'Direct link to Features')

- Added new table `rasa_dialogue_stack_frame` to store active flow names and steps for each event sequence in the conversation.
- Add new table `rasa_llm_command` to store LLM generated commands for each user message. Add new column in the `_rasa_raw_event` table to store the serialized LLM generated commands.
- Add new table `rasa_flow_status` to store the transformations of rasa flow events. Add new columns in the `_rasa_raw_event` table to store the flow\_id and step\_id of these events where applicable.

## \[3.1.1\] - 2023-07-17[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#311---2023-07-17 'Direct link to [3.1.1] - 2023-07-17')

Rasa Pro Services 3.1.1 (2023-07-17)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-5 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.1.0\] - 2023-07-03[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#310---2023-07-03 'Direct link to [3.1.0] - 2023-07-03')

Rasa Pro Services 3.1.0 (2023-07-03)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features-2 'Direct link to Features')

- Added Real Time Processing of Markers. Markers can now be evaluated real time by the Analytics Data Pipeline. We've added event handlers for evaluation all events from Kafka to extract markers. The extracted markers are saved into `rasa_marker` database table. These markers are evaluated with the patterns stored in `rasa_pattern` table.
 
 Added API endpoints to create patterns in `rasa_pattern` table. This endpoint is used by Rasa Plus for `rasa markers upload` command.
 

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-6 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.2\] - 2023-06-13[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#302---2023-06-13 'Direct link to [3.0.2] - 2023-06-13')

Rasa Pro Services 3.0.2 (2023-06-13)

### Improvements[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-9 'Direct link to Improvements')

- Adds an environment variable to control logging level of the application.

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-7 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.1\] - 2022-10-26[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#301---2022-10-26 'Direct link to [3.0.1] - 2022-10-26')

Rasa Pro Services 3.0.1 (2023-10-26)

### Miscellaneous internal changes[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-8 'Direct link to Miscellaneous internal changes')

_Miscellaneous internal changes._

## \[3.0.0\] - 2022-10-24[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#300---2022-10-24 'Direct link to [3.0.0] - 2022-10-24')

Rasa Pro Services 3.0.0 (2023-10-24)

### Features[​](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features-3 'Direct link to Features')

- Analytics Data Pipeline helps visualize and process Rasa assistant metrics in the tooling (BI tools, data warehouses) of your choice. Visualizations and analysis of the production assistant and its conversations allow you to assess ROI and improve the performance of the assistant over time.

- [\[3.8.1\] - 2026-04-17](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#381---2026-04-17)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes)
- [\[3.8.0\] - 2026-03-26](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#380---2026-03-26)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-1)
- [\[3.7.0\] - 2025-11-26](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#370---2025-11-26)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-1)
- [\[3.6.2\] - 2025-11-25](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#362---2025-11-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-2)
- [\[3.6.1\] - 2025-10-29](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#361---2025-10-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-3)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes)
- [\[3.6.0\] - 2025-10-10](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#360---2025-10-10)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-2)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-4)
- [\[3.5.10\] - 2025-11-25](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#3510---2025-11-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-5)
- [\[3.5.9\] - 2025-10-29](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#359---2025-10-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-6)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-1)
- [\[3.5.8\] - 2025-10-10](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#358---2025-10-10)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-7)
- [\[3.5.7\] - 2025-10-06](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#357---2025-10-06)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-8)
- [\[3.5.6\] - 2025-09-11](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#356---2025-09-11)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-9)
- [\[3.5.5\] - 2025-09-03](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#355---2025-09-03)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-10)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-2)
- [\[3.5.4\] - 2025-07-29](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#354---2025-07-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-11)
- [\[3.5.3\] - 2025-06-27](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#353---2025-06-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-12)
- [\[3.5.2\] - 2025-05-21](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#352---2025-05-21)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-13)
- [\[3.5.1\] - 2025-05-12](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#351---2025-05-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-14)
- [\[3.5.0\] - 2025-03-20](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#350---2025-03-20)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-3)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-15)
- [\[3.4.1\] - 2025-05-12](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#341---2025-05-12)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-16)
- [\[3.4.0\] - 2024-12-12](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#340---2024-12-12)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-4)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-17)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-3)
- [\[3.3.5\] - 2024-10-29](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#335---2024-10-29)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-18)
- [\[3.3.4\] - 2024-10-02](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#334---2024-10-02)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-19)
- [\[3.3.3\] - 2024-09-25](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#333---2024-09-25)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-20)
- [\[3.3.2\] - 2024-05-28](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#332---2024-05-28)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-5)
- [\[3.3.1\] - 2024-05-27](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#331---2024-05-27)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-21)
- [\[3.3.0\] - 2024-04-03](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#330---2024-04-03)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-6)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-4)
- [\[3.2.3\] - 2023-12-20](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#323---2023-12-20)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-7)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-22)
- [\[3.2.2\] - 2023-12-05](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#322---2023-12-05)
 - [Bugfixes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#bugfixes-23)
- [\[3.2.1\] - 2023-12-01](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#321---2023-12-01)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-8)
- [\[3.2.0\] - 2023-11-22](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#320---2023-11-22)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features-1)
- [\[3.1.1\] - 2023-07-17](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#311---2023-07-17)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-5)
- [\[3.1.0\] - 2023-07-03](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#310---2023-07-03)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features-2)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-6)
- [\[3.0.2\] - 2023-06-13](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#302---2023-06-13)
 - [Improvements](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#improvements-9)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-7)
- [\[3.0.1\] - 2022-10-26](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#301---2022-10-26)
 - [Miscellaneous internal changes](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#miscellaneous-internal-changes-8)
- [\[3.0.0\] - 2022-10-24](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#300---2022-10-24)
 - [Features](https://rasa.com/docs/reference/changelogs/rasa-pro-services-changelog/#features-3)