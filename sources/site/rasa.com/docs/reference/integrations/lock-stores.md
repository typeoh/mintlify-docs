# Source: https://rasa.com/docs/reference/integrations/lock-stores

On this page

## InMemoryLockStore (default)[​](https://rasa.com/docs/reference/integrations/lock-stores/#inmemorylockstore-default 'Direct link to InMemoryLockStore (default)')

- **Description**
 
 `InMemoryLockStore` is the default lock store. It maintains conversation locks within a single process.
 
 note
 
 This lock store should not be used when multiple Rasa servers are run in parallel.
 
- **Configuration**
 
 To use the `InMemoryTrackerStore` no configuration is needed.
 

## ConcurrentRedisLockStore[​](https://rasa.com/docs/reference/integrations/lock-stores/#concurrentredislockstore 'Direct link to ConcurrentRedisLockStore')

`ConcurrentRedisLockStore` is a new, recommended lock store that uses Redis as a persistence layer and is safe for use with multiple Rasa server replicas. See the [migration section](https://rasa.com/docs/reference/integrations/lock-stores/#migration-guide) to learn how to switch to this lock store.

### Description[​](https://rasa.com/docs/reference/integrations/lock-stores/#description 'Direct link to Description')

The `ConcurrentRedisLockStore` uses Redis as a persistence layer for instances of issued [tickets](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/core/lock/#ticket-objects) and the last issued ticket number.

The ticket number initialization begins at 1, in contrast to that of the`RedisLockStore` which begins at 0. If the ticket expires, the ticket number will not be reassigned to future tickets; as a result ticket numbers are unique to ticket instances. Ticket numbers are incremented using the Redis atomic transaction INCR on the persisted last issued ticket number.

The `ConcurrentRedisLockStore` ensures that only one Rasa instance can handle a conversation at any point in time. Therefore, this Redis implementation of the `LockStore` can handle messages received in parallel for the same conversation by different Rasa servers This is the recommended lock store for running a replicated set of Rasa servers.

### High Availability Support[​](https://rasa.com/docs/reference/integrations/lock-stores/#high-availability-support 'Direct link to High Availability Support')

New in 3.14

Redis high availability support is now available for the `ConcurrentRedisLockStore`. You can now deploy with Redis Cluster for horizontal scaling or Redis Sentinel for automatic failover.

The `ConcurrentRedisLockStore` now supports Redis high availability deployments through Redis Cluster and Redis Sentinel modes, enabling enterprise-grade scalability and reliability for production deployments. The supported deployments are:

- Redis Cluster Mode: Provides horizontal scaling and is compatible with cloud-hosted Redis services.
- Redis Sentinel Mode: Offers high availability through automatic master/slave failover.
- Standard Mode: Maintains backward compatibility with existing single-instance deployments.

### Configuration[​](https://rasa.com/docs/reference/integrations/lock-stores/#configuration 'Direct link to Configuration')

To set up Rasa with Redis the following steps are required:

1. Add required configuration to your `endpoints.yml`:

- Rasa Pro <=3.7.x
- Rasa Pro >=3.8.x

```
  lock_store:      type: rasa_plus.components.concurrent_lock_store.ConcurrentRedisLockStore      host: "host of the redis instance, e.g. localhost/"      port: "port of your redis instance, usually 6379/"      password: "password used for authentication"      db: "number of your database within redis, e.g. 0. Not used in cluster mode"      key_prefix: "alphanumeric value to prepend to lock store keys"      # high availability parameters introduced in 3.14 below      deployment_mode: "standard, cluster, or sentinel"      endpoints: "list of redis cluster/sentinel node addresses in the format host:port"      sentinel_service: "name of the redis sentinel service, only used in sentinel mode"
```

```
  lock_store:      type: concurrent_redis      host: "host of the redis instance, e.g. localhost"      port: "port of your redis instance, usually 6379"      password: "password used for authentication"      db: "number of your database within redis, e.g. 0. Not used in cluster mode"      key_prefix: "alphanumeric value to prepend to lock store keys"      # high availability parameters introduced in 3.14 below      deployment_mode: "standard, cluster, or sentinel"      endpoints: "list of redis cluster/sentinel node addresses in the format host:port"      sentinel_service: "name of the redis sentinel service, only used in sentinel mode"
```

2. To start the Rasa server using your Redis backend, add the `--endpoints` flag, e.g.:
 
 ```
    rasa run -m models --endpoints endpoints.yml
    ```
 

### Configuration Parameters[​](https://rasa.com/docs/reference/integrations/lock-stores/#configuration-parameters 'Direct link to Configuration Parameters')

- `url` (default: `localhost`): The url of your redis instance
- `port` (default: `6379`): The port which redis is running on
- `db` (default: `1`): The number of your redis database. _Ignored in cluster mode as Redis Cluster always uses database 0_
- `key_prefix` (default: `None`): The prefix to prepend to lock store keys. Must be alphanumeric
- `password` (default: `None`): Password used for authentication (`None` equals no authentication)
- `use_ssl` (default: `False`): Whether the communication is encrypted
- `socket_timeout` (default: `10`): Time in seconds after which an error is raised if Redis doesn't answer
- `deployment_mode` (default: `standard`): Deployment mode of Redis. One of `standard`, `cluster`, or `sentinel`.
- `endpoints` (default: `None`): List of Redis cluster node addresses in the format `host:port`. Used for cluster and sentinel modes. For cluster mode, these are cluster node endpoints. For sentinel mode, these are sentinel instance endpoints.
- `sentinel_service` (default: `mymaster`): Name of the Redis sentinel service. Only used in sentinel mode.

#### Deployment Mode Details[​](https://rasa.com/docs/reference/integrations/lock-stores/#deployment-mode-details 'Direct link to Deployment Mode Details')

- Standard Mode: Connects to a single Redis instance using url and port parameters.
- Cluster Mode: Connects to a Redis Cluster for horizontal scaling. If endpoints is not provided, falls back to auto-discovery using url and port.
- Sentinel Mode: Connects to Redis Sentinel for high availability with automatic master/slave failover.

### Choosing a Deployment Mode[​](https://rasa.com/docs/reference/integrations/lock-stores/#choosing-a-deployment-mode 'Direct link to Choosing a Deployment Mode')

- Use standard mode for simple deployments with a single Redis instance.
- Use cluster mode for horizontal scaling and when using cloud Redis services that require cluster mode.
- Use sentinel mode for high availability with master/slave replication and automatic failover.

### Using IAM roles to authenticate to AWS ElastiCache for Redis[​](https://rasa.com/docs/reference/integrations/lock-stores/#using-iam-roles-to-authenticate-to-aws-elasticache-for-redis 'Direct link to Using IAM roles to authenticate to AWS ElastiCache for Redis')

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
- `ELASTICACHE_REDIS_AWS_IAM_ENABLED`: Set this to `true` to enable IAM authentication for ElastiCache Redis connections.

You can configure the lock store in your `endpoints.yml` file to not use a username and password:

endpoints.yml

```
lock_store:    type: concurrent_redis    host: "master redis host of your elasticache cluster or replication group"    port: "6379"    use_ssl: true    username: your-iam-username    ssl_ca_certs: "/path/to/your/ca-certificate.pem" # Optional, path to the root certificate from Amazon Trust Services
```

When you start your Rasa instance, it will use the IAM role to generate temporary credentials to log in to the AWS ElastiCache cluster instead of using static credentials. The temporary credentials will be automatically refreshed every 15 minutes.

### Migration Guide[​](https://rasa.com/docs/reference/integrations/lock-stores/#migration-guide 'Direct link to Migration Guide')

To switch from the `RedisLockStore` to the `ConcurrentRedisLockStore`, specify the complete module path to the `ConcurrentRedisLockStore` class as `type` in `endpoints.yml`:

- Rasa Pro <=3.7.x
- Rasa Pro >=3.8.x

```
lock_store:    type: rasa_plus.components.concurrent_lock_store.ConcurrentRedisLockStore    host: "host of the redis instance, e.g. localhost"    port: "port of your redis instance, usually 6379"    password: "password used for authentication"    db: "number of your database within redis, e.g. 0. Not used in cluster mode"    key_prefix: "alphanumeric value to prepend to lock store keys"    # high availability parameters introduced in 3.14 below    deployment_mode: <standard, cluster, or sentinel>    endpoints: <list of redis cluster/sentinel node addresses in the format host:port>    sentinel_service: <name of the redis sentinel service, only used in sentinel mode>
```

```
lock_store:    type: concurrent_redis    host: "host of the redis instance, e.g.localhost"    port: "port of your redis instance, usually 6379"    password: "password used for authentication"    db: "number of your database within redis, e.g. 0. Not used in cluster mode"    key_prefix: "alphanumeric value to prepend to lock store keys"    # high availability parameters introduced in 3.14 below    deployment_mode: <standard, cluster, or sentinel>    endpoints: <list of redis cluster/sentinel node addresses in the format host:port>    sentinel_service: <name of the redis sentinel service, only used in sentinel mode>
```

You must replace the `url` field in the redis lock store configuration with a field `host` containing the hostname of the redis instance.

No database migration is required when switching to the `ConcurrentRedisLockStore`. You can use the same Redis instance and database number as you did previously when using the `RedisLockStore`. You may want to delete all the preexisting keys if using the same Redis database number. These former key-value items are no longer required by the `ConcurrentRedisLockStore` and the database can be cleared.

There is no overlap in key-value items stored when using the `RedisLockStore` and the `ConcurrentRedisLockStore`, because the `RedisLockStore` persists serialized [TicketLock](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/core/lock/#ticketlock-objects) instances while the `ConcurrentRedisLockStore` instead stores individual [`Ticket`](https://legacy-docs-oss.rasa.com/docs/rasa/reference/rasa/core/lock/#ticket-objects) instances, as well as the last issued ticket number. The `ConcurrentRedisLockStore` recreates the `TicketLock` from the persisted `Ticket` instances, which allows it to handle concurrent messages for the same conversation ID.

## RedisLockStore[​](https://rasa.com/docs/reference/integrations/lock-stores/#redislockstore 'Direct link to RedisLockStore')

### Description[​](https://rasa.com/docs/reference/integrations/lock-stores/#description-1 'Direct link to Description')

`RedisLockStore` maintains conversation locks using Redis as a persistence layer.

### High Availability Support[​](https://rasa.com/docs/reference/integrations/lock-stores/#high-availability-support-1 'Direct link to High Availability Support')

New in 3.14

Redis high availability support is now available for the `RedisLockStore`. You can now deploy with Redis Cluster for horizontal scaling or Redis Sentinel for automatic failover.

The `RedisLockStore` now supports Redis high availability deployments through Redis Cluster and Redis Sentinel modes, enabling enterprise-grade scalability and reliability for production deployments. The supported deployments are:

- Redis Cluster Mode: Provides horizontal scaling and is compatible with cloud-hosted Redis services.
- Redis Sentinel Mode: Offers high availability through automatic master/slave failover.
- Standard Mode: Maintains backward compatibility with existing single-instance deployments.

### Configuration[​](https://rasa.com/docs/reference/integrations/lock-stores/#configuration-1 'Direct link to Configuration')

To set up Rasa with Redis the following steps are required:

1. Add required configuration to your `endpoints.yml`
 
 ```
    lock_store:    type: "redis"    url: <url of the redis instance, e.g. localhost>    port: <port of your redis instance, usually 6379>    password: <password used for authentication>    db: <number of your database within redis, e.g. 0. Not used in cluster mode>    key_prefix: <alphanumeric value to prepend to lock store keys>    # high availability parameters introduced in 3.14 below    deployment_mode: <standard, cluster, or sentinel>    endpoints: <list of redis cluster/sentinel node addresses in the format host:port>    sentinel_service: <name of the redis sentinel service, only used in sentinel mode>
    ```
 
2. To start the Rasa server using your Redis backend, add the `--endpoints` flag, e.g.:
 

```
rasa run -m models --endpoints endpoints.yml
```

### Configuration Parameters[​](https://rasa.com/docs/reference/integrations/lock-stores/#configuration-parameters-1 'Direct link to Configuration Parameters')

- `url` (default: `localhost`): The url of your redis instance
 
- `port` (default: `6379`): The port which redis is running on
 
- `db` (default: `1`): The number of your redis database. _Ignored in cluster mode as Redis Cluster always uses database 0_
 
- `key_prefix` (default: `None`): The prefix to prepend to lock store keys. Must be alphanumeric
 
- `username` (default: `None`): Username used for authentication
 
- `password` (default: `None`): Password used for authentication (`None` equals no authentication)
 
- `use_ssl` (default: `False`): Whether or not the communication is encrypted
 
- `ssl_keyfile` (default: `None`): Path to an ssl private key
 
- `ssl_certfile` (default: `None`): Path to an ssl certificate
 
- `ssl_ca_certs` (default: `None`): The path to a file of concatenated CA certificates in PEM format
 
- `socket_timeout` (default: `10`): Time in seconds after which an error is raised if Redis doesn't answer
 
- `deployment_mode` (default: `standard`): Deployment mode of Redis. One of `standard`, `cluster`, or `sentinel`.
 
- `endpoints` (default: `None`): List of Redis cluster node addresses in the format `host:port`. Used for cluster and sentinel modes. For cluster mode, these are cluster node endpoints. For sentinel mode, these are sentinel instance endpoints.
 
- `sentinel_service` (default: `mymaster`): Name of the Redis sentinel service. Only used in sentinel mode.
 

#### Deployment Mode Details[​](https://rasa.com/docs/reference/integrations/lock-stores/#deployment-mode-details-1 'Direct link to Deployment Mode Details')

- Standard Mode: Connects to a single Redis instance using url and port parameters.
- Cluster Mode: Connects to a Redis Cluster for horizontal scaling. If endpoints is not provided, falls back to auto-discovery using url and port.
- Sentinel Mode: Connects to Redis Sentinel for high availability with automatic master/slave failover.

### Choosing a Deployment Mode[​](https://rasa.com/docs/reference/integrations/lock-stores/#choosing-a-deployment-mode-1 'Direct link to Choosing a Deployment Mode')

- Use standard mode for simple deployments with a single Redis instance.
- Use cluster mode for horizontal scaling and when using cloud Redis services that require cluster mode.
- Use sentinel mode for high availability with master/slave replication and automatic failover.

### Using IAM roles to authenticate to AWS ElastiCache (for Redis)[​](https://rasa.com/docs/reference/integrations/lock-stores/#using-iam-roles-to-authenticate-to-aws-elasticache-for-redis-1 'Direct link to Using IAM roles to authenticate to AWS ElastiCache (for Redis)')

New in 3.14

You can use IAM authentication to connect to AWS ElastiCache for Redis without needing to provide static credentials.

To read more about how to set up IAM authentication to AWS ElastiCache for Redis, see the [ConcurrentRedisLockStore](https://rasa.com/docs/reference/integrations/lock-stores/#using-iam-roles-to-authenticate-to-aws-elasticache-for-redis) section.

## Custom Lock Store[​](https://rasa.com/docs/reference/integrations/lock-stores/#custom-lock-store 'Direct link to Custom Lock Store')

If you need a lock store which is not available out of the box, you can implement your own. This is done by extending the base class `LockStore`.

Your custom lock store class must also implement the following methods:

- `get_lock`: fetches lock for `conversation_id` from storage; requires `conversation_id` text parameter and returns `TicketLock` instance. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/lock_store.py#L59).
- `save_lock`: commit `lock` object to storage; requires `lock` parameter which is of type `TicketLock`and returns `None`. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/lock_store.py#L67).
- `delete_lock`: deletes lock for `conversation_id` from storage; requires `conversation_id` text parameter and returns `None`. [(source code - see for signature)](https://github.com/RasaHQ/rasa/blob/main/rasa/core/lock_store.py#L63).

### Configuration[​](https://rasa.com/docs/reference/integrations/lock-stores/#configuration-2 'Direct link to Configuration')

Put the module path to your custom event broker and the parameters you require in your `endpoints.yml`:

endpoints.yml

```
lock_store:  type: path.to.your.module.Class  url: localhost  a_parameter: a value  another_parameter: another value
```