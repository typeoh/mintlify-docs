# Source: https://rasa.com/docs/reference/deployment/automatic-conversation-deletion

On this page

## Overview[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#overview 'Direct link to Overview')

Automatic Conversation Deletion allows for the periodic removal of old conversations and their associated data from Studio. This behavior is designed to help manage data retention and comply with data protection regulations.

## Configuration[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#configuration 'Direct link to Configuration')

Automatic Deletion is controlled by two environment variables:

1. `DELETE_CONVERSATIONS_OLDER_THAN_HOURS`: Specifies the age threshold for conversations to be deleted, in hours.
2. `DELETE_CONVERSATIONS_CRON_EXPRESSION`: Defines when the deletion process should run using a cron expression.

Example helm chart configuration:

```
DELETE_CONVERSATIONS_OLDER_THAN_HOURS=720  # 30 daysDELETE_CONVERSATIONS_CRON_EXPRESSION="0 * * * *"  # Run hourly
```

## Deletion Process[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#deletion-process 'Direct link to Deletion Process')

### Default Behavior[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#default-behavior 'Direct link to Default Behavior')

The deletion is turned off by default. It will not run since default value in `DELETE_CONVERSATIONS_OLDER_THAN_HOURS` is not set. If `DELETE_CONVERSATIONS_OLDER_THAN_HOURS` is set, the cron job will run hourly (controlled by `DELETE_CONVERSATIONS_CRON_EXPRESSION`).

### Scope[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#scope 'Direct link to Scope')

The system will delete entire conversations, including all associated messages and events, **when the first message or event in the conversation is older than the specified time threshold**. This means that as soon as the first message in a conversation is older than the threshold, the entire conversation will be deleted.

### Batch Deletion[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#batch-deletion 'Direct link to Batch Deletion')

Deletion is performed in batches to manage system load and database transaction limits (100 per batch). If a deletion process is interrupted (e.g., by a server restart), it will continue from where it left off during the next scheduled run.

## Data Affected[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#data-affected 'Direct link to Data Affected')

When a conversation is deleted, the following associated data is also removed:

- Messages
- Conversation Events
- Intents
- Predicted Flows
- Channel information
- Machine Learning Model data
- Utterance Entities
- Utterance Intents

## Performance Considerations[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#performance-considerations 'Direct link to Performance Considerations')

The deletion process is designed to run periodically and in batches to minimize impact on system performance. However, during times of high message ingestion, the deletion process may have a noticeable impact on system resources.

Users should consider setting the cron schedule to run during off-peak hours.

## Limitations and Known Issues[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#limitations-and-known-issues 'Direct link to Limitations and Known Issues')

The deletion will only run while Studio is running. In rare cases, the deletion process may encounter transaction timeouts when dealing with a large number of conversations.

There is a potential for write conflicts or deadlocks during deletion operations, which may require the process to retry.

## Best Practices[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#best-practices 'Direct link to Best Practices')

Set the deletion threshold (`DELETE_CONVERSATIONS_OLDER_THAN_HOURS`) to a value that balances your data retention needs with system performance. Monitor system performance during and after deletion runs to ensure it's not negatively impacting your application.

Regularly review and adjust the cron schedule as needed based on your system's usage patterns.

## Compliance Note[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#compliance-note 'Direct link to Compliance Note')

While this feature can assist with data retention policies, users are responsible for ensuring their specific use of the system complies with relevant data protection regulations (e.g., GDPR).

## Conversations via API[​](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#conversations-via-api 'Direct link to Conversations via API')

For developers looking to programmatically tag and delete conversations, please refer to our [Conversations API Documentation](https://rasa.com/docs/reference/api/studio/conversation-api/).

- [Overview](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#overview)
- [Configuration](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#configuration)
- [Deletion Process](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#deletion-process)
 - [Default Behavior](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#default-behavior)
 - [Scope](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#scope)
 - [Batch Deletion](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#batch-deletion)
- [Data Affected](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#data-affected)
- [Performance Considerations](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#performance-considerations)
- [Limitations and Known Issues](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#limitations-and-known-issues)
- [Best Practices](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#best-practices)
- [Compliance Note](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#compliance-note)
- [Conversations via API](https://rasa.com/docs/reference/deployment/automatic-conversation-deletion/#conversations-via-api)