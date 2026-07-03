# Source: https://rasa.com/docs/reference/integrations/tracker-stores

On this page

Tracker stores are responsible for persisting conversation history and state in Rasa. Each conversation is represented as a tracker, which contains the sequence of events that have occurred during the conversation. Tracker stores enable your assistant to maintain context across multiple conversation sessions and retrieve historical conversation data.

Rasa provides several built-in tracker store implementations to suit different deployment scenarios and requirements:

- **InMemoryTrackerStore**: Stores conversations in memory (default). Suitable for development and testing.
- **SQLTrackerStore**: Uses SQL databases (PostgreSQL, SQLite, Oracle) for persistent storage.
- **RedisTrackerStore**: Leverages Redis for fast in-memory storage with optional persistence.
- **MongoTrackerStore**: Stores conversations in MongoDB, a document-oriented NoSQL database.
- **DynamoTrackerStore**: Uses AWS DynamoDB for cloud-based storage.

You can also implement a custom tracker store by extending the base `TrackerStore` class if you need to integrate with a different storage backend.

## User-Scoped Tracker Querying[​](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-tracker-querying 'Direct link to User-Scoped Tracker Querying')

New in 3.16

Rasa tracker stores now support user-scoped querying of conversations (referred to as trackers) and durable conversation metadata.

All tracker stores support user-scoped tracker querying and durable conversation-level properties such as `user_id` and `conversation_started_timestamp`. These properties enable efficient retrieval of all conversations for a specific user.

### Conversation Properties[​](https://rasa.com/docs/reference/integrations/tracker-stores/#conversation-properties 'Direct link to Conversation Properties')

Tracker stores persist the following fields in the tracker object:

- **`user_id`**: The unique identifier for the user associated with the conversation. This field is used for querying all conversations belonging to a specific user. Serialization of trackers omits `null` `user_id` values to optimize storage.
- **`conversation_started_timestamp`**: The timestamp when the conversation was first started. This field is used for ordering conversations chronologically.
- **`current_session_id`**: The session ID for the tracker's current session, derived from the metadata of the last stored event. This value is `null` when the last event is `ConversationInactive`, correctly reflecting that no active session exists.

info

The `conversation_started_timestamp` is automatically backfilled on save/update for backward compatibility with existing trackers that may not have this field.

Each tracker store implementation provides optimized mechanisms for user-scoped retrieval, ordering, and pagination as described in their respective sections below.

## InMemoryTrackerStore (default)[​](https://rasa.com/docs/reference/integrations/tracker-stores/#inmemorytrackerstore-default 'Direct link to InMemoryTrackerStore (default)')

`InMemoryTrackerStore` is the default tracker store. It is used if no other tracker store is configured. It stores the conversation history in memory.

note

As this store keeps all history in memory, the entire history is lost if you restart the Rasa server.

### **Configuration**[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration 'Direct link to configuration')

No configuration is needed to use the `InMemoryTrackerStore`.

### User-Scoped Querying[​](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying 'Direct link to User-Scoped Querying')

The `InMemoryTrackerStore` implements user-scoped tracker querying by scanning all stored tracker keys and retrieving each tracker to filter by `user_id`.

#### Implementation Details[​](https://rasa.com/docs/reference/integrations/tracker-stores/#implementation-details 'Direct link to Implementation Details')

- **Full Scan**: The implementation scans all stored tracker keys in memory and retrieves each tracker to filter by `user_id`. This approach is feasible for in-memory storage due to the typically smaller dataset size.
- **Ordering and Pagination**: After filtering, the trackers are ordered by `conversation_started_timestamp` and `sender_id`, and paginated in memory using standard Python sorting and slicing techniques.

## SQLTrackerStore[​](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore 'Direct link to SQLTrackerStore')

You can use an `SQLTrackerStore` to store your assistant's conversation history in an SQL database.

### Configuration[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-1 'Direct link to Configuration')

To set up Rasa with SQL the following steps are required:

1. Add required configuration to your `endpoints.yml`:

endpoints.yml

```
tracker_store:    type: SQL    dialect: "postgresql"  # the dialect used to interact with the db    url: ""  # (optional) host of the sql db, e.g. "localhost"    db: "rasa"  # path to your db    username:  # username used for authentication    password:  # password used for authentication    query: # optional dictionary to be added as a query string to the connection URL      driver: my-driver
```

2. To start the Rasa server using your SQL backend, add the `--endpoints` flag, e.g.:

```
rasa run -m models --endpoints endpoints.yml
```

#### Configuration Parameters[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-parameters 'Direct link to Configuration Parameters')

- `domain` (default: `None`): Domain object associated with this tracker store
 
- `dialect` (default: `sqlite`): The dialect used to communicate with your SQL backend. Consult the [SQLAlchemy docs](https://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls) for available dialects.
 
- `url` (default: `None`): URL of your SQL server
 
- `port` (default: `None`): Port of your SQL server
 
- `db` (default: `rasa.db`): The path to the database to be used
 
- `username` (default: `None`): The username which is used for authentication
 
- `password` (default: `None`): The password which is used for authentication
 
- `event_broker` (default: `None`): Event broker to publish events to
 
- `login_db` (default: `None`): Alternative database name to which initially connect, and create the database specified by `db` (PostgreSQL only)
 
- `query` (default: `None`): Dictionary of options to be passed to the dialect and/or the DBAPI upon connect
 

#### Compatible Databases[​](https://rasa.com/docs/reference/integrations/tracker-stores/#compatible-databases 'Direct link to Compatible Databases')

The following databases are officially compatible with the `SQLTrackerStore`:

- PostgreSQL
- Oracle > 11.0
- SQLite

#### Configuring Oracle[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuring-oracle 'Direct link to Configuring Oracle')

To use the SQLTrackerStore with Oracle, there are a few additional steps. First, create a database `tracker` in your Oracle database and create a user with access to it. Create a sequence in the database with the following command, where username is the user you created (read more about creating sequences in the [Oracle Documentation](https://docs.oracle.com/cd/B28359_01/server.111/b28310/views002.htm#ADMIN11794)):

```
CREATE SEQUENCE username.events_seq;
```

Next you have to extend the Rasa image to include the necessary drivers and clients. First download the [Oracle Instant Client](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html), rename it to `oracle.rpm` and store it in the directory from where you'll be building the docker image. Copy the following into a file called `Dockerfile`:

```
FROM rasa/rasa:latest-full# Switch to root user to install packagesUSER rootRUN apt-get update -qq && apt-get install -y --no-install-recommends alien libaio1 && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*# Copy in oracle instaclient# https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.htmlCOPY oracle.rpm oracle.rpm# Install the Python wrapper library for the Oracle driversRUN pip install cx-Oracle# Install Oracle client librariesRUN alien -i oracle.rpmUSER 1001
```

Then build the docker image:

```
docker build . -t rasa-oracle:latest-oracle-full
```

Now you can configure the tracker store in the `endpoints.yml` as described above, and start the container. The `dialect` parameter with this setup will be `oracle+cx_oracle`.

#### Using IAM roles to authenticate to an AWS RDS SQL tracker store[​](https://rasa.com/docs/reference/integrations/tracker-stores/#using-iam-roles-to-authenticate-to-an-aws-rds-sql-tracker-store 'Direct link to Using IAM roles to authenticate to an AWS RDS SQL tracker store')

New in 3.14

You can use IAM authentication to connect to an AWS RDS database without needing to provide a username and password.

If your Rasa instance is running on an AWS service that supports IAM roles (e.g. EC2), you can use IAM authentication to connect to an AWS RDS database without needing to provide a username and password. To do so, you need to ensure that your RDS database is configured to allow IAM authentication. Once your RDS instance is up, connect to it and run the following SQL command to enable IAM authentication for the database user you want to use:

```
GRANT rds_iam TO <db_username>;
```

You also need to set up your Rasa instance with an appropriate IAM role that has permission to access the RDS database. The IAM role should include the following permission to allow the IAM role to generate the temporary authentication token for the db user:

```
{    "Version": "2012-10-17",    "Statement": [        {            "Effect": "Allow",            "Action": [                "rds-db:connect"            ],            "Resource": [                "arn:aws:rds-db:<region>:<account-id>:dbuser:<DbiResourceId for RDS PostgreSQL>/<db_username>"            ]        }    ]}
```

Once you have set up the IAM role and configured your RDS database, you can configure the following environment variables in your Rasa instance:

- `IAM_CLOUD_PROVIDER`: Set this to `aws`.
- `AWS_DEFAULT_REGION`: Set this to the AWS region where your RDS database is located.
- `RDS_SQL_DB_AWS_IAM_ENABLED`: Set this to `true` to enable IAM authentication for RDS connections.
- `SQL_TRACKER_STORE_SSL_MODE`: Set this to the desired SSL mode for the connection (Check your deployed SQL database documentation for supported values. For example: see [PostgreSQL SSL Mode descriptions](https://www.postgresql.org/docs/current/libpq-ssl.html#LIBPQ-SSL-PROTECTION-MODES)).
- `SQL_TRACKER_STORE_SSL_ROOT_CERTIFICATE`: (Optional) Set this to the path of the CA certificate to use for SSL verification if using `verify-ca` or `verify-full` SSL modes. This can be downloaded for the particular region from [here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesAllRegions).

You will also need to update your `endpoints.yml` to use the IAM authentication by omitting the `password` field in the `tracker_store` configuration:

endpoints.yml

```
tracker_store:    type: sql    dialect: "postgresql"    url: "<rds-endpoint>"    db: "<db-name>"    username: "<db-username>"    port: 5432
```

When you start your Rasa instance, it will use the IAM role to generate temporary credentials to log in to the RDS database instead of using static credentials. The temporary credentials will be automatically refreshed every 15 minutes.

### User-Scoped Querying[​](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-1 'Direct link to User-Scoped Querying')

The `SQLTrackerStore` implements user-scoped tracker querying through a dedicated `users` table that maps `sender_id` to `user_id` along with the timestamp when the conversation was started.

note

Load test your SQL database with a realistic volume of conversations and users to ensure that the user-scoped querying performs well under expected production loads. Monitor query performance and optimize indices as needed based on your specific usage patterns.

#### Implementation Details[​](https://rasa.com/docs/reference/integrations/tracker-stores/#implementation-details-1 'Direct link to Implementation Details')

- **Users Table**: A separate `users` table is created with columns for `sender_id` (mapped to `user_id`) and `conversation_started_timestamp`. The table uses dialect-specific upsert operations for PostgreSQL and sqlite and a generic approach (update followed by insert if needed) for other SQL databases to maintain compatibility.
- **Bulk Retrieval**: User-scoped queries use two bulk operations: a paginated `users` table query to retrieve `sender_id`s for the given `user_id`, followed by a single `events WHERE sender_id IN (...)` fetch to load all events for those conversations. This ensures query count stays constant regardless of conversation volume.
- **Database-Level Ordering and Pagination**: The `sender_id`s are ordered by `conversation_started_timestamp` and paginated using SQL `LIMIT`/`OFFSET` clauses for optimal performance.
- **Automatic Cleanup**: When a conversation is deleted, the corresponding entry in the users table is automatically cleaned up to maintain referential integrity.
- **Efficient Indexing**: Appropriate indices are created on the `users` table for all fields to optimize query performance for user-scoped lookups.

## RedisTrackerStore[​](https://rasa.com/docs/reference/integrations/tracker-stores/#redistrackerstore 'Direct link to RedisTrackerStore')

You can store your assistant's conversation history in [Redis](https://redis.io/) by using the `RedisTrackerStore`. Redis is a fast in-memory key-value store which can optionally also persist data.

### High Availability Support[​](https://rasa.com/docs/reference/integrations/tracker-stores/#high-availability-support 'Direct link to High Availability Support')

New in 3.14

Redis high availability support is now available for the `RedisTrackerStore`. You can now deploy with Redis Cluster for horizontal scaling or Redis Sentinel for automatic failover.

The `RedisTrackerStore` now supports Redis high availability deployments through Redis Cluster and Redis Sentinel modes, enabling enterprise-grade scalability and reliability for production deployments. The supported deployments are:

- Redis Cluster Mode: Provides horizontal scaling and is compatible with cloud-hosted Redis services.
- Redis Sentinel Mode: Offers high availability through automatic master/slave failover.
- Standard Mode: Maintains backward compatibility with existing single-instance deployments.

### Configuration[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-2 'Direct link to Configuration')

To set up Rasa with Redis the following steps are required:

1. Add required configuration to your `endpoints.yml`:

endpoints.yml

```
tracker_store:    type: redis    url: <url of the redis instance, e.g. localhost>    port: <port of your redis instance, usually 6379>    key_prefix: <alphanumeric value to prepend to tracker store keys>    db: <number of your database within redis, e.g. 0. Not used in cluster mode>    password: <password used for authentication>    use_ssl: <whether or not the communication is encrypted, default `false`>    # high availability parameters introduced in 3.14    deployment_mode: <standard, cluster, or sentinel>    endpoints: <list of redis cluster/sentinel node addresses in the format host:port, only used in cluster or sentinel mode>    sentinel_service: <name of the redis sentinel service, only used in sentinel mode>
```

2. To start the Rasa server using your SQL backend, add the `--endpoints` flag, e.g.:

```
rasa run -m models --endpoints endpoints.yml
```

#### Configuration Parameters[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-parameters-1 'Direct link to Configuration Parameters')

- `url` (default: `localhost`): The url of your redis instance
 
- `port` (default: `6379`): The port which redis is running on
 
- `db` (default: `0`): The number of your redis database. _Ignored in cluster mode as Redis Cluster always uses database 0_
 
- `key_prefix` (default: `None`): The prefix to prepend to tracker store keys. Must be alphanumeric
 

- `username` (default: `None`): Username used for authentication
 
- `password` (default: `None`): Password used for authentication (`None` equals no authentication)
 

- `record_exp` (default: `None`): Record expiry in seconds
 
- `use_ssl` (default: `False`): whether or not to use SSL for transit encryption
 
- `deployment_mode` (default: `standard`): Deployment mode of Redis. One of `standard`, `cluster`, or `sentinel`.
 
- `endpoints` (default: `None`): List of Redis cluster node addresses in the format `host:port`. Used for cluster and sentinel modes. For cluster mode, these are cluster node endpoints. For sentinel mode, these are sentinel instance endpoints.
 
- `sentinel_service` (default: `mymaster`): Name of the Redis sentinel service. Only used in sentinel mode.
 

##### Deployment Mode Details[​](https://rasa.com/docs/reference/integrations/tracker-stores/#deployment-mode-details 'Direct link to Deployment Mode Details')

- Standard Mode: Connects to a single Redis instance using url and port parameters.
- Cluster Mode: Connects to a Redis Cluster for horizontal scaling. If endpoints is not provided, falls back to auto-discovery using url and port.
- Sentinel Mode: Connects to Redis Sentinel for high availability with automatic master/slave failover.

#### Choosing a Deployment Mode[​](https://rasa.com/docs/reference/integrations/tracker-stores/#choosing-a-deployment-mode 'Direct link to Choosing a Deployment Mode')

- Use standard mode for simple deployments with a single Redis instance.
- Use cluster mode for horizontal scaling and when using cloud Redis services that require cluster mode.
- Use sentinel mode for high availability with master/slave replication and automatic failover.

Using the same redis instance as lock-store and tracker store

You must not use the same Redis instance as both lock store and tracker store. If the Redis instance becomes unavailable, the conversation will hang because there is no fall back mechanism implemented for the lock store (as it is for the tracker store interfaces).

### Using IAM to authenticate to AWS ElastiCache for Redis[​](https://rasa.com/docs/reference/integrations/tracker-stores/#using-iam-to-authenticate-to-aws-elasticache-for-redis 'Direct link to Using IAM to authenticate to AWS ElastiCache for Redis')

New in 3.14

You can use IAM authentication to connect to AWS ElastiCache for Redis without needing to provide static credentials.

If your Rasa instance is running on an AWS service that supports IAM roles (e.g. EC2), you can use IAM authentication to connect to AWS ElastiCache for Redis without needing to provide static credentials. To do so, you need to ensure that your AWS ElastiCache cluster or replication group is configured to allow IAM authentication by creating an AWS ElastiCache user with IAM authentication mode enabled.

You also need to set up your Rasa instance with an appropriate IAM role that has the permissions to access the AWS ElastiCache cluster or replication group:

```
{    "Version": "2012-10-17",    "Statement": [        {            "Effect": "Allow",            "Action": [                "elasticache:Connect"            ],            "Resource": "*"        }    ]}
```

Once you have set up the IAM role and configured your ElastiCache cluster or replication group, you can configure the following environment variables in your Rasa instance:

- `IAM_CLOUD_PROVIDER`: Set this to `aws`.
- `AWS_DEFAULT_REGION`: Set this to the AWS region where your ElastiCache cluster or replication group is located.
- `AWS_ELASTICACHE_CLUSTER_NAME`: Set this to the name of your ElastiCache cluster or replication group.
- `ELASTICACHE_REDIS_AWS_IAM_ENABLED`: Set this to `true` to enable IAM authentication for ElastiCache connections.

You can configure the lock store in your `endpoints.yml` file to not use a username and password:

endpoints.yml

```
tracker_store:    type: redis    url: "master redis host of your elasticache cluster or replication group"    port: "6379"    use_ssl: true    username: your-iam-username
```

When you start your Rasa instance, it will use the IAM role to generate temporary credentials to log in to the AWS ElastiCache cluster instead of using static credentials. The temporary credentials will be automatically refreshed every 15 minutes.

### User-Scoped Querying[​](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-2 'Direct link to User-Scoped Querying')

The `RedisTrackerStore` implements user-scoped tracker querying through a secondary index that enables O(1) lookup of all conversations for a specific user.

#### Implementation Details[​](https://rasa.com/docs/reference/integrations/tracker-stores/#implementation-details-2 'Direct link to Implementation Details')

- **Secondary Index**: A Redis Sorted Set is maintained with the key pattern `user_trackers:{user_id}`. This index stores all conversation IDs (sender\_ids) for each user, enabling efficient O(1) lookup.
- **Batch Operations**: When retrieving multiple conversations for a user, the tracker store uses Redis `MGET` (multi-get) operations to fetch all tracker data in a single batch, reducing network round trips.
- **Index Cleanup**: When a conversation is deleted, the corresponding entry is removed from the user's sorted-set index. Orphaned index entries (entries whose tracker key no longer exists in Redis) are pruned in a single batched `ZREM` call rather than one call per orphan.

warning

[Redis cluster mode does not support multi-key operations](https://redis.readthedocs.io/en/stable/clustering.html#multi-key-commands) like `MGET` across different hash slots. As a result, user-scoped querying in cluster mode falls back to using non-atomic `MGET` retrieval of tracker data. This may lead to higher latency, inconsistent and/or partial results if some keys are located on different cluster nodes that are temporarily unavailable. For optimal performance and consistency, it is recommended to use Redis standard or sentinel modes when leveraging user-scoped querying features.

## MongoTrackerStore[​](https://rasa.com/docs/reference/integrations/tracker-stores/#mongotrackerstore 'Direct link to MongoTrackerStore')

You can store your assistant's conversation history in [MongoDB](https://www.mongodb.com/) using the `MongoTrackerStore`. MongoDB is a free and open-source cross-platform document-oriented NoSQL database.

### Configuration[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-3 'Direct link to Configuration')

1. Add required configuration to your `endpoints.yml`:

endpoints.yml

```
tracker_store:    type: mongod    url: <url to your mongo instance, e.g. mongodb://localhost:27017>    db: <name of the db within your mongo instance, e.g. rasa>    username: <username used for authentication>    password: <password used for authentication>    auth_source: <database name associated with the user's credentials>
```

You can also add more advanced configurations (like enabling ssl) by appending a parameter to the url field, e.g. `mongodb://localhost:27017/?ssl=true`.

2. To start the Rasa server using your configured MongoDB instance, add the `--endpoints` flag, for example:

```
rasa run -m models --endpoints endpoints.yml
```

#### Configuration Parameters[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-parameters-2 'Direct link to Configuration Parameters')

- `url` (default: `mongodb://localhost:27017`): URL of your MongoDB
 
- `db` (default: `rasa`): The database name which should be used
 
- `username` (default: `0`): The username which is used for authentication
 
- `password` (default: `None`): The password which is used for authentication
 
- `auth_source` (default: `admin`): database name associated with the user's credentials.
 
- `collection` (default: `conversations`): The collection name which is used to store the conversations
 

### User-Scoped Querying[​](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-3 'Direct link to User-Scoped Querying')

The `MongoTrackerStore` implements user-scoped querying through MongoDB indices and aggregation pipelines for efficient querying and ordering.

#### Implementation Details[​](https://rasa.com/docs/reference/integrations/tracker-stores/#implementation-details-3 'Direct link to Implementation Details')

- **Database Indices**: The tracker store creates indices on `user_id` and a compound index on `(conversation_started_timestamp, sender_id)` to optimize query performance.
- **Aggregation Pipeline**: User-scoped queries use MongoDB aggregation pipelines to efficiently filter, sort, and paginate conversations. This allows for complex queries with ordering and pagination at the database level.
- **User ID Restoration**: The implementation ensures that `user_id` is properly restored from stored tracker data, maintaining data consistency across conversation sessions.

## DynamoTrackerStore[​](https://rasa.com/docs/reference/integrations/tracker-stores/#dynamotrackerstore 'Direct link to DynamoTrackerStore')

You can store your assistant's conversation history in [DynamoDB](https://aws.amazon.com/dynamodb/) by using a `DynamoTrackerStore`. DynamoDB is a hosted NoSQL database offered by Amazon Web Services (AWS).

### Configuration[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-4 'Direct link to Configuration')

1. Add required configuration to your `endpoints.yml`:

endpoints.yml

```
tracker_store:    type: dynamo    table_name: <name of the table to create, e.g. rasa>    region: <name of the region associated with the client>
```

2. To start the Rasa server using your configured `DynamoDB` instance, add the `--endpoints` flag, e.g.:

```
rasa run -m models --endpoints endpoints.yml
```

#### Configuration Parameters[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-parameters-3 'Direct link to Configuration Parameters')

- `table_name` (default: `states`): name of the DynamoDB table
 
- `region` (default: `us-east-1`): name of the region associated with the client
 

info

In case the table with `table_name` does not exist, Rasa will create it for you when run with single sanic worker.

In case Rasa is run with multiple sanic workers, the table should be created before running Rasa. If it's not found, an error will be logged and an exception will be raised.

### User-Scoped Querying[​](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-4 'Direct link to User-Scoped Querying')

The `DynamoTrackerStore` implements user-scoped querying through DynamoDB Global Secondary Indexes (GSI).

#### Implementation Details[​](https://rasa.com/docs/reference/integrations/tracker-stores/#implementation-details-4 'Direct link to Implementation Details')

- **Metadata Persistence**: The tracker store saves both `user_id` and `conversation_started_timestamp` (stored as Decimal type) in the `update_item` db operation when saving trackers.
- **Global Secondary Index (GSI)**: A GSI named `user_id-index` is created with `user_id` as the partition key and `conversation_started_timestamp` as the sort key.
- **Full-Scan Fallback**: If the GSI is not available or not yet created, the implementation falls back to a full table scan which uses a filter expression by `user_id`.

note

Ensure the GSI is created before running the Rasa Pro server. The GSI creation may take some time depending on the size of your table. Load test your DynamoDB setup to ensure that the GSI is properly utilized for user-scoped queries and that performance meets your requirements.

## Custom Tracker Store[​](https://rasa.com/docs/reference/integrations/tracker-stores/#custom-tracker-store 'Direct link to Custom Tracker Store')

If you need a tracker store which is not available out of the box, you can implement your own. This is done by extending the base class `TrackerStore` and one of the provided mixin classes that implement the `serialise_tracker` method: `SerializedTrackerAsText` or `SerializedTrackerAsDict`.

To write a custom tracker store, extend the `TrackerStore` base class. Your constructor has to provide a parameter `host`. The constructor also needs to make a `super` call to the base class `TrackerStore` using `domain` and `event_broker` arguments:

```
super().__init__(domain, event_broker, **kwargs)
```

Your custom tracker store class must also implement the following methods:

- `save`: saves the conversation to the tracker store. Must respect the following signature:

```
    async def save(self, tracker: DialogueStateTracker) -> None:        """Save tracker to the tracker store.        Args:            tracker: Tracker to be saved.        """
```

- `update`: updates an existing tracker in the store (e.g. after anonymization or when retaining events during deletion). When supported, this allows a single atomic write instead of delete-then-save. The [PII anonymization and deletion jobs](https://rasa.com/docs/reference/config/pii-management/overview/#pii-management-jobs) use this method (deletion only when events need to be retained). Must respect the following signature:

```
    async def update(        self,        tracker_to_keep: DialogueStateTracker,        apply_deletion_only: bool = True,    ) -> None:        """Update existing tracker in the tracker store.        Args:            tracker_to_keep: Tracker with updated content (e.g. anonymized events or retained events after deletion).            apply_deletion_only: If True (default), only apply deletion semantics; if False, perform a full content update (e.g. for anonymization).        """
```

- `retrieve`: retrieves tracker for the latest conversation session. Must respect the following signature:

```
    async def retrieve(self, sender_id: str) -> Optional[DialogueStateTracker]:        """Retrieve tracker for the given sender_id.        Args:            sender_id: Conversation ID to retrieve the tracker for.        Returns:            Tracker for the given sender_id.        """
```

- `keys`: returns the set of values for the tracker store's primary key. Must respect the following signature:

```
    async def keys(self) -> Iterable[str]:        """Return the set of values for the tracker store's primary key."""
```

- `delete`: deletes the conversation corresponding to the given `sender_id` in the tracker store. Must respect the following signature:

```
    async def delete(self, sender_id: str) -> None:        """Delete tracker for the given sender_id.        Args:            sender_id: Conversation ID to delete the tracker for.        """
```

- `get_serialized_trackers_by_user_id`: retrieves serialized event dicts for all conversations belonging to a specific user, with pagination and ordering support. This method powers the `GET /users/{user_id}/trackers` endpoint and returns data directly from storage without replaying events through `DialogueStateTracker`. Must respect the following signature:

```
    async def get_serialized_trackers_by_user_id(        self,        user_id: str,        limit: Optional[int] = None,        skip: Optional[int] = None,    ) -> List[Dict[str, Any]]:        """Retrieve serialized trackers for the given user_id.        Args:            user_id: User ID to retrieve trackers for.            limit: Optional limit on the number of trackers to return.            skip: Optional offset for pagination.        Returns:            List of serialized tracker dicts for the given user_id, ordered            by conversation_started_timestamp and sender_id, without replaying            events through DialogueStateTracker.        """
```

This method can use built-in helper methods from the `TrackerStore` parent class:

- `_sort_key_serialized(self, tracker_dict: Dict[str, Any]) -> Tuple[float, str]`: Returns a tuple of `(conversation_started_timestamp, sender_id)` for sorting trackers.
- `_apply_pagination_serialized(self, trackers: List[Dict[str, Any]], skip: Optional[int], limit: Optional[int]) -> List[Dict[str, Any]]`: Applies pagination to a list of serialized trackers using `skip` (offset) and `limit` parameters.

### Configuration[​](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-5 'Direct link to Configuration')

Put the module path to your custom tracker store and the parameters you require in your `endpoints.yml`:

endpoints.yml

```
tracker_store:  type: path.to.your.module.Class  url: localhost  a_parameter: a value  another_parameter: another value
```

## Fallback Tracker Store[​](https://rasa.com/docs/reference/integrations/tracker-stores/#fallback-tracker-store 'Direct link to Fallback Tracker Store')

In case the primary tracker store configured in `endpoints.yml` becomes unavailable, the Rasa agent will issue an error message and fall back on the `InMemoryTrackerStore` implementation. A new dialogue session will be started for each turn, which will be saved separately in the `InMemoryTrackerStore` fallback.

As soon as the primary tracker store comes back up, it will replace the fallback tracker store and save the conversation from this point going forward. However, note that any previous states saved in the `InMemoryTrackerStore` fallback will be lost.

- [User-Scoped Tracker Querying](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-tracker-querying)
 - [Conversation Properties](https://rasa.com/docs/reference/integrations/tracker-stores/#conversation-properties)
- [InMemoryTrackerStore (default)](https://rasa.com/docs/reference/integrations/tracker-stores/#inmemorytrackerstore-default)
 - [**Configuration**](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration)
 - [User-Scoped Querying](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying)
- [SQLTrackerStore](https://rasa.com/docs/reference/integrations/tracker-stores/#sqltrackerstore)
 - [Configuration](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-1)
 - [User-Scoped Querying](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-1)
- [RedisTrackerStore](https://rasa.com/docs/reference/integrations/tracker-stores/#redistrackerstore)
 - [High Availability Support](https://rasa.com/docs/reference/integrations/tracker-stores/#high-availability-support)
 - [Configuration](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-2)
 - [Using IAM to authenticate to AWS ElastiCache for Redis](https://rasa.com/docs/reference/integrations/tracker-stores/#using-iam-to-authenticate-to-aws-elasticache-for-redis)
 - [User-Scoped Querying](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-2)
- [MongoTrackerStore](https://rasa.com/docs/reference/integrations/tracker-stores/#mongotrackerstore)
 - [Configuration](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-3)
 - [User-Scoped Querying](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-3)
- [DynamoTrackerStore](https://rasa.com/docs/reference/integrations/tracker-stores/#dynamotrackerstore)
 - [Configuration](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-4)
 - [User-Scoped Querying](https://rasa.com/docs/reference/integrations/tracker-stores/#user-scoped-querying-4)
- [Custom Tracker Store](https://rasa.com/docs/reference/integrations/tracker-stores/#custom-tracker-store)
 - [Configuration](https://rasa.com/docs/reference/integrations/tracker-stores/#configuration-5)
- [Fallback Tracker Store](https://rasa.com/docs/reference/integrations/tracker-stores/#fallback-tracker-store)