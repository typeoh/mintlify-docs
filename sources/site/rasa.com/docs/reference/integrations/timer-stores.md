# Source: https://rasa.com/docs/reference/integrations/timer-stores

On this page

Rasa maintains one active timer per conversation to track inactivity. To ensure timers persist across server restarts and work in multi-pod deployments, Rasa uses a timer store. Rasa provides built-in timer store implementations to suit different deployment scenarios and requirements.

## InMemoryTimerStore (default)[​](https://rasa.com/docs/reference/integrations/timer-stores/#inmemorytimerstore-default 'Direct link to InMemoryTimerStore (default)')

- **Description**
 
 `InMemoryTimerStore` is the default timer store and it stores timers in memory. Suitable for development and testing as timers are lost on server restart.
 
- **Configuration**
 
 To use the `InMemoryTimerStore` no configuration is needed.
 

## RedisTimerStore[​](https://rasa.com/docs/reference/integrations/timer-stores/#redistimerstore 'Direct link to RedisTimerStore')

### Description[​](https://rasa.com/docs/reference/integrations/timer-stores/#description 'Direct link to Description')

`RedisTimerStore` maintains conversation timers using Redis as a persistence layer. This is the recommended option for production deployments as it supports persistence across server restarts and works in multi-pod environments.

With Redis:

- Timers survive server restarts.
- All pods in a horizontally-scaled deployment share the same timer state.
- Timers that expired while the server was down are recovered and processed on startup.

### Configuration[​](https://rasa.com/docs/reference/integrations/timer-stores/#configuration 'Direct link to Configuration')

To set up the timer store with Redis the following steps are required:

1. Add required configuration to your `endpoints.yml`
 
 ```
    timer_store:    type: "redis"    host: <host of the redis instance, e.g. localhost>    port: <port of your redis instance, usually 6379>    password: <password used for authentication>    db: <number of your database within redis, e.g. 0. Not used in cluster mode>    key_prefix: <alphanumeric value to prepend to timer store keys>    # high availability parameters    deployment_mode: <standard, cluster, or sentinel>    endpoints: <list of redis cluster/sentinel node addresses in the format host:port>    sentinel_service: <name of the redis sentinel service, only used in sentinel mode>
    ```
 

## Fallback Timer Store[​](https://rasa.com/docs/reference/integrations/timer-stores/#fallback-timer-store 'Direct link to Fallback Timer Store')

If Redis is configured but becomes unavailable, Rasa automatically falls back to the `InMemoryTimerStore`. Sessions continue to be managed without crashing, and a warning is logged when the fallback is activated. When Redis recovers, Rasa resumes using it and logs that the fallback has been deactivated.

## Custom Timer Stores[​](https://rasa.com/docs/reference/integrations/timer-stores/#custom-timer-stores 'Direct link to Custom Timer Stores')

If you need a timer store which is not available out of the box, you can implement your own. To write a custom timer store, extend `SessionTimerStore` from `rasa.core.timer_stores.timer_store`. Your constructor must accept an `endpoint_config` parameter:

```
from rasa.core.timer_stores.timer_store import SessionTimerStorefrom rasa.utils.endpoints import EndpointConfigclass MyCustomTimerStore(SessionTimerStore):  def __init__(self, endpoint_config: EndpointConfig = None):      ...
```

Your custom timer store class must implement all of the following methods:

- `close`: releases any resources (connections, file handles, etc.) held by the store. Must respect the following signature:

```
  def close(self) -> None:    """Close and clean up the timer store."""
```

- `store_timer`: persists a timer entry for a conversation. Must respect the following signature:

```
  async def store_timer(    self,    sender_id: str,    session_id: Optional[str],    scheduled_time: float,    metadata: Optional[Dict[str, Any]] = None,  ) -> None:    """Store a session timer for the given conversation.    Args:        sender_id: The conversation ID.        session_id: The current session ID, used to validate the timer            has not been superseded when it fires.        scheduled_time: Unix timestamp (seconds) when the timer should fire.        metadata: Optional key-value data to attach to the timer.    """
```

- `delete_timer`: removes a timer entry. Must respect the following signature:

```
  async def delete_timer(    self,    sender_id: str,    only_if_scheduled_time: Optional[float] = None,  ) -> bool:    """Delete the timer for the given conversation.    Args:        sender_id: The conversation ID.        only_if_scheduled_time: If set, delete only when the stored timer's            scheduled_time equals this value (for atomic multi-pod claiming).            If None, delete unconditionally.    Returns:        True if a timer was found and deleted, False otherwise.    """
```

- `get_timer`: retrieves the current timer for a conversation, or None if none exists. Must respect the following signature:

```
  async def get_timer(self, sender_id: str) -> Optional[SessionTimer]:    """Get the timer for the given conversation.    Args:        sender_id: The conversation ID.    Returns:        The SessionTimer if one exists for this conversation, None otherwise.    """
```

- `get_expired_timers`: returns all timers whose `scheduled_time` is at or before `cutoff_time`. Must respect the following signature:

```
  async def get_expired_timers(    self, cutoff_time: Optional[float] = None  ) -> List[SessionTimer]:    """Return all timers that have expired.    Args:        cutoff_time: Unix timestamp threshold. Timers with            scheduled_time <= cutoff_time are returned.            Defaults to the current time if None.    Returns:        List of expired SessionTimer objects. Must not delete them.    """
```

The base class provides two helpers that can be used by the `get_expired_timers` method:

- `_check_time_for_expired(cutoff_time)`: returns `cutoff_time` if set, otherwise `time.time()`.
- `_filter_expired(timers, check_time)`: filters an iterable of `SessionTimer` objects to those with `scheduled_time <= check_time`.

Another built-in helper method available for use in your custom timer store is:

- `SessionTimer.create_timer(sender_id, session_id, scheduled_time, metadata)`: constructs a `SessionTimer` from the arguments passed to `store_timer`.

### Configuration[​](https://rasa.com/docs/reference/integrations/timer-stores/#configuration-1 'Direct link to Configuration')

Put the module path to your custom timer store class in `endpoints.yml` under the `timer_store` key. Any additional fields are passed through as part of the `EndpointConfig` and are accessible via `endpoint_config.kwargs`:

endpoints.yml

```
timer_store:  type: path.to.your.module.MyCustomTimerStore  a_parameter: a value  another_parameter: another value
```

- [InMemoryTimerStore (default)](https://rasa.com/docs/reference/integrations/timer-stores/#inmemorytimerstore-default)
- [RedisTimerStore](https://rasa.com/docs/reference/integrations/timer-stores/#redistimerstore)
 - [Description](https://rasa.com/docs/reference/integrations/timer-stores/#description)
 - [Configuration](https://rasa.com/docs/reference/integrations/timer-stores/#configuration)
- [Fallback Timer Store](https://rasa.com/docs/reference/integrations/timer-stores/#fallback-timer-store)
- [Custom Timer Stores](https://rasa.com/docs/reference/integrations/timer-stores/#custom-timer-stores)
 - [Configuration](https://rasa.com/docs/reference/integrations/timer-stores/#configuration-1)