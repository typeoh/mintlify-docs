# Source: https://rasa.com/docs/reference/config/pii-management/overview

On this page

New in 3.13.0

Rasa provides tools to help you manage personally identifiable information (PII) collected by your assistant.

Rasa provides a comprehensive solution for managing personally identifiable information (PII) collected by your assistant. The PII management capability allows you to:

- Identify and anonymize PII from slot events, user and bot messages.
- Configure PII anonymization for specific slots.
- Manage PII data retention policies in the tracker store.
- Stream anonymized events to event brokers. Among supported event brokers are Kafka and RabbitMQ.

## PII Identification[​](https://rasa.com/docs/reference/config/pii-management/overview/#pii-identification 'Direct link to PII Identification')

Rasa adopts a tiered approach to PII identification, which includes:

1. **Slot-based PII identification**: The sensitive data is stored in a slot whose name is defined in the `privacy` YAML config. This enables Rasa to use the existing [CALM slot filling](https://rasa.com/docs/reference/primitives/slots/#calm-slot-mappings) mechanisms to identify PII in user messages, bot responses and [slot events](https://rasa.com/docs/reference/primitives/events/#slot-event). To learn more about how to configure PII slots, see the [Anonymization Rules](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#anonymization-rules) section below.
 
2. **GLiNER PII identification**: Optional integration with [GLiNER PII model](https://huggingface.co/urchade/gliner_multi_pii-v1) to identify PII in two particular edge cases:
 
 - end chat user could be specifying some PII in their user message that is not captured by any domain slot.
 - usage of free text slots like problem description or customer notes that could have multiple types of PII To learn more about how to configure GLiNER PII identification, see the [GLiNER requirements](https://rasa.com/docs/reference/config/pii-management/prerequisites/#gliner-requirements) section below.

## PII Anonymization[​](https://rasa.com/docs/reference/config/pii-management/overview/#pii-anonymization 'Direct link to PII Anonymization')

Rasa supports two types of PII anonymization:

- **redaction**: replaces PII plaintext value with a redaction character (default is `*`) across the full length of the PII value. This anonymization method can be configured to keep N characters of the PII value to the left or right of the redaction character. For example, for a credit card number `1234-5678-9012-3456` and a redaction character `*` with `left=0` and `right=4`, the anonymized value would be `****-****-****-3456`.
- **masking**: replaces PII plaintext value with the uppercase slot or entity name (e.g. `[EMAIL_ADDRESS]`).

To learn more about how to configure PII anonymization, see the [Anonymization Rules](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#anonymization-rules) section below.

## Performance[​](https://rasa.com/docs/reference/config/pii-management/overview/#performance 'Direct link to Performance')

The PII management capability helps you comply with data protection regulations and ensure that sensitive user data is handled appropriately without compromising the functionality of your assistant. Rasa ensures that the PII management capability does not affect the performance of your assistant.

### PII Management Jobs[​](https://rasa.com/docs/reference/config/pii-management/overview/#pii-management-jobs 'Direct link to PII Management Jobs')

The PII management jobs, such as publishing anonymized events to the supported event brokers, anonymization and deletion in the tracker store, are run in the background using a job scheduler. This allows your assistant to continue processing user requests without significant delays.

When anonymization or deletion in the tracker store is configured, a [Lock Store](https://rasa.com/docs/reference/integrations/lock-stores/) is required. The background privacy manager acquires a per-`sender_id` lock while anonymization or deletion jobs perform read-modify-write on trackers, which prevents race conditions when multiple processes or jobs access the same conversation. Both anonymization and deletion use the tracker store's `update` method (deletion only when events need to be retained), so these operations complete in a single atomic operation on the [tracker store](https://rasa.com/docs/reference/integrations/tracker-stores/) (including SQL, which no longer uses a separate delete/save sequence).

If you have configured both the anonymization and deletion jobs with overlapping cron triggers, Rasa ensures that both jobs are run sequentially to avoid a potential race condition in updating the tracker store. First the anonymization job is run, followed by the deletion job as long as the deletion job is due to run once the anonymization job is completed.

Recommendations

We recommend that you configure the PII management jobs to run at a low traffic time to minimize the number of reads and writes to the tracker store. This will help reduce the load on the tracker store.

Monitor the logs of the PII management jobs to ensure that they are running successfully and to identify any potential issues with the anonymization or deletion processes.

New in 3.16

The `USER_CHAT_INACTIVITY_IN_MINUTES` environment variable is deprecated for PII eligibility. Session management improvements introduce new ways to handle different tracker variants (legacy, new, and hybrid). Leave the env var unset to use event-based eligibility with `ConversationInactive`/`SessionEnded` and `session_id` grouping. See [Session Management](https://rasa.com/docs/reference/config/session-management/overview/) for details.

### Tracker Variants and Session Eligibility[​](https://rasa.com/docs/reference/config/pii-management/overview/#tracker-variants-and-session-eligibility 'Direct link to Tracker Variants and Session Eligibility')

Privacy cron jobs (anonymization and deletion) process trackers according to how sessions are represented in events:

- **Legacy trackers**: No `session_id` in any event metadata. They can contain multiple `ActionExecuted(action_session_start)` events (e.g. after session expiry a new session starts). Sessions are split by `action_session_start` first. Eligibility for anonymization or deletion is time-based using `LEGACY_DEFAULT_INACTIVITY_MINUTES` (30 minutes when `USER_CHAT_INACTIVITY_IN_MINUTES` is unset) plus `min_after_session_end`.
 
- **New trackers**: Events have `session_id` in metadata. They have `ConversationInactive` after session timeout; a subset also have `SessionEnded`. Events are grouped by `session_id` (not split by `action_session_start`). A session is eligible for anonymization or deletion only if it contains `ConversationInactive` or `SessionEnded`; retention uses `min_after_session_end` after that event.
 
- **Hybrid trackers**: A tracker can have both legacy events (no `session_id`) and new events (with `session_id`). Legacy events form a single prefix (all events before the >=3.16 rasa version upgrade); after the upgrade all new events have `session_id`. The legacy prefix is expanded via the same logic as legacy trackers (split by `action_session_start`), and each sub-session is evaluated individually; `session_id` runs are processed per run so reassembly preserves order.
 

When `USER_CHAT_INACTIVITY_IN_MINUTES` is **unset**, “new” trackers are processed based on `ConversationInactive`/`SessionEnded` events and `session_id` grouping (with a grace period of `min_after_session_end`), while “legacy” trackers without `session_id` use a fixed 30 minutes plus `min_after_session_end`. When the environment variable is **set**, time-based eligibility is still applied but is deprecated and applied per session.

### Deletion Cron Job[​](https://rasa.com/docs/reference/config/pii-management/overview/#deletion-cron-job 'Direct link to Deletion Cron Job')

The deletion cron job is responsible for deleting PII data from the tracker store, and it runs periodically based on the configured cron schedule.

The deletion job removes PII data that is no longer needed, based on the configured retention period and the [session eligibility logic](https://rasa.com/docs/reference/config/pii-management/overview/#tracker-variants-and-session-eligibility) above. It loops through conversation sessions in the tracker store (including trackers that contain multiple sessions, split or grouped as described for legacy, new, and hybrid trackers). A session is considered eligible for deletion when it has ended and the retention period has passed:

- for event-based behavior (when `USER_CHAT_INACTIVITY_IN_MINUTES` is unset), sessions that contain `SessionEnded` use `min_after_session_end` after that event.
- for legacy segments, 30 minutes plus `min_after_session_end`; when the env var is set, time-based eligibility (env var value + `min_after_session_end`) is applied per session but is deprecated.

When the job encounters sessions that are not eligible for deletion, either because they are still active or because they have not reached the end of the retention period, the job retains these events in the tracker store by overwriting the pre-existing tracker with only the retained events. If the reconstruction of the tracker with retained events raises `JsonPatchConflict` or `JsonPointerException` errors (e.g. due to dialogue stack updates spanning multiple sessions), the job skips the deletion for that tracker and logs an error. To reset the tracker's dialogue stack and allow deletion of the entire tracker to proceed in the next run, you can append a `SessionEnded` event to the tracker using the [tracker API](https://rasa.com/docs/reference/api/pro/http-api/)(`POST /conversations/{conversation_id}/tracker/events`).

- [PII Identification](https://rasa.com/docs/reference/config/pii-management/overview/#pii-identification)
- [PII Anonymization](https://rasa.com/docs/reference/config/pii-management/overview/#pii-anonymization)
- [Performance](https://rasa.com/docs/reference/config/pii-management/overview/#performance)
 - [PII Management Jobs](https://rasa.com/docs/reference/config/pii-management/overview/#pii-management-jobs)
 - [Tracker Variants and Session Eligibility](https://rasa.com/docs/reference/config/pii-management/overview/#tracker-variants-and-session-eligibility)
 - [Deletion Cron Job](https://rasa.com/docs/reference/config/pii-management/overview/#deletion-cron-job)