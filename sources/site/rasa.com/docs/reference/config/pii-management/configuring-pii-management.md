# Source: https://rasa.com/docs/reference/config/pii-management/configuring-pii-management

On this page

## Configuration Overview[​](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#configuration-overview 'Direct link to Configuration Overview')

To configure PII management in your Rasa project, you need to define the `privacy` YAML config in your `endpoints` YAML configuration. This configuration includes settings for tracker store PII management and anonymization rules.

For example, you can define the following `privacy` configuration in your `endpoints.yml` file:

endpoints.yml

```
privacy:  tracker_store_settings:    deletion:      min_after_session_end: 3000      cron: "30 0 * * *"    anonymization:      min_after_session_end: 120      cron: "30 1 * * *"  rules:    - slot: credit_card_number      anonymization:          type: redact          redaction_char: "#"          keep_right: 4
```

Let's dive into the details of each configuration section below.

### Tracker Store Configuration[​](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#tracker-store-configuration 'Direct link to Tracker Store Configuration')

When you configure tracker store anonymization or deletion, you must configure a [Lock Store](https://rasa.com/docs/reference/integrations/lock-stores/). The background privacy manager requires a lock store and acquires a per-`sender_id` lock during anonymization and deletion jobs to avoid race conditions when reading and writing trackers. Both anonymization and deletion use `update()` on the [tracker store](https://rasa.com/docs/reference/integrations/tracker-stores/) (deletion only when events need to be retained), so these operations complete in a single atomic operation (no separate delete-then-save).

The `tracker_store_settings` section allows you to configure PII management for the tracker store, including deletion and anonymization settings:

- `deletion`: This section allows you to configure the deletion of PII data from the tracker store.
 - `min_after_session_end`: The minimum number of minutes after which the session data is deleted from the tracker store. Must be greater than the `min_after_session_end` value in the `anonymization` section. When `USER_CHAT_INACTIVITY_IN_MINUTES` is unset, eligibility is event-based (see [Tracker variants and session eligibility](https://rasa.com/docs/reference/config/pii-management/overview/#tracker-variants-and-session-eligibility)); when set, it must be greater than the env value (time-based behavior is deprecated and applied per session).
 - `cron`: The cron expression that defines when the deletion job should run. In the example, the job will run every day at 00:30 UTC. Must be different from the `cron` expression in the `anonymization` section.
- `anonymization`: This section allows you to configure the anonymization of PII data in the tracker store.
 - `min_after_session_end`: The minimum number of minutes after which the session data is anonymized in the tracker store (grace period after session end). Must be less than the `min_after_session_end` value in the `deletion` section. When `USER_CHAT_INACTIVITY_IN_MINUTES` is unset, eligibility is event-based; when set, it must be greater than the env value (time-based behavior is deprecated per session).
 - `cron`: The cron expression that defines when the anonymization job should run. In the example, the job will run every day at 01:30 UTC. Must be different from the `cron` expression in the `deletion` section.

Both `deletion` and `anonymization` sections are optional, and you can configure either or both of them based on your requirements. If you do decide to configure them, you must provide both `min_after_session_end` and `cron` settings.

### Anonymization Rules[​](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#anonymization-rules 'Direct link to Anonymization Rules')

The `rules` section allows you to define specific anonymization rules for slots that contain PII data.

- `slot`: The name of the slot that contains PII data.
- `anonymization`: The anonymization method to be applied to the slot value.
 - `type`: The type of anonymization, either `redact` or `mask`.
 - `redaction_char`: The character used for redaction (only applicable for `redact` type).
 - `keep_left`: The number of characters to keep to the left of the redaction character (default is `0`).
 - `keep_right`: The number of characters to keep to the right of the redaction character (default is `0`).

### Event Broker Configuration[​](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#event-broker-configuration 'Direct link to Event Broker Configuration')

You can also configure the event broker to stream anonymized events to dedicated topics or queues. The supported event broker types are Kafka and RabbitMQ, you can learn more about how to configure the streaming of anonymized events in the [Kafka Event Broker documentation](https://rasa.com/docs/reference/integrations/event-brokers/#sending-anonymized-events) and [RabbitMQ Event Broker documentation](https://rasa.com/docs/reference/integrations/event-brokers/#publishing-anonymized-events).

## Integrating with Consent Management Platforms[​](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#integrating-with-consent-management-platforms 'Direct link to Integrating with Consent Management Platforms')

Rasa does not provide a built-in consent management platform (CMP) for managing user consent and PII data compliance. However, you can integrate Rasa with leading third-party CMPs like OneTrust, Usercentrics, Cookiebot, and CookieYes among others, to ensure GDPR, CCPA, and other privacy regulation compliance within your conversational AI workflows.

You can implement this integration through a custom action that interacts with the CMP API. This custom action can check if the user has provided consent for PII data collection or can store the consent to the CMP if the consent was collected via a custom flow.

Here's an example for when you want to retrieve the consent status of a user id from the CMP API:

1. Create custom actions that interact with CMP APIs to store, retrieve and validate user consent:

actions.py

```
from rasa_sdk import Action, Trackerfrom rasa_sdk.executor import CollectingDispatcherfrom rasa_sdk.events import SlotSetimport requestsfrom typing import Dict, List, Any, Textclass ActionCheckUserConsent(Action):    """Retrieve user consent status from CMP."""    def name(self) -> Text:        return "action_check_user_consent"    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:        user_id = tracker.sender_id # assuming the user ID is the identifier used in the CMP        # if user_id is not the sender_id, you may need to fetch it from a different source        # for example, from the message metadata        # user_id = tracker.latest_message.get("metadata").get("user_id")        consent_data = self._get_user_consent(user_id)        # Replace with your transformation logic to simplify the consent data        # For example, you might want to extract specific consent categories        transformed_consent_data = consent_data        if transformed_consent_data:            return [                SlotSet("has_analytics_consent", transformed_consent_data.get("analytics", False)),                SlotSet("has_marketing_consent", transformed_consent_data.get("marketing", False)),                SlotSet("consent_valid", True)            ]        else:            return [SlotSet("consent_valid", False)]    def _get_user_consent(self, user_id: str) -> Dict:        """Call CMP API to get consent status."""        try:            # Example: OneTrust API call            # Replace with your CMP API call to fetch consent data            headers = {"Authorization": f"Bearer {ONETRUST_API_KEY}"}            response = requests.get(                f"https://your-tenant.onetrust.com/api/ConsentManager/v3/datasubjects/profile?identifier={user_id}",                headers=headers            )            if response.status_code == 200:                return response.json()            else:                return {}        except Exception as e:            logger.error(f"Failed to fetch consent: {e}")        return Noneclass ActionVerifyConsentForDataCollection(Action):    """Verify consent before collecting sensitive data."""    def name(self) -> Text:        return "action_verify_consent_for_data_collection"    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:        has_consent = tracker.get_slot("has_analytics_consent")        data_type = tracker.get_slot("requested_data_type")  # e.g., "personal_info", "preferences"        # Map data types to required consent categories        consent_requirements = {            "personal_info": "has_analytics_consent",            "preferences": "has_marketing_consent",            "behavior_tracking": "has_analytics_consent"        }        required_consent_slot = consent_requirements.get(data_type)        has_required_consent = tracker.get_slot(required_consent_slot) if required_consent_slot else False        return [SlotSet("data_collection_allowed", has_required_consent)]class ActionRequestConsent(Action):    """Guide user to provide consent through CMP."""    def name(self) -> Text:        return "action_request_consent"    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:        consent_url = "https://your-website.com/privacy-preferences"        dispatcher.utter_message(            text="To provide you with personalized assistance, I need your consent to collect certain information. "                 f"Please visit {consent_url} to manage your privacy preferences, then return to continue our conversation."        )        return [SlotSet("consent_request_sent", True)]
```

2. Use consent information in flows to branch off conversation logic:

domain.yml

```
flows:  personalized_assistance:    description: "Provide personalized help with consent validation"    steps:      - action: action_check_user_consent        next:          - if: consent_valid            then:              - if: has_analytics_consent                then:                  - action: utter_can_provide_personalized_help                  - collect: user_preference              - else:                  - action: utter_limited_assistance_available                  - action: action_request_consent          - else:              - action: utter_consent_check_failed              - action: action_request_consent  data_collection_flow:    description: "Collect user data with consent verification"    steps:      - action: action_verify_consent_for_data_collection        next:          - if: data_collection_allowed            then:              - action: utter_ask_for_information              - action: action_store_user_data            else:              - action: utter_cannot_collect_data              - action: action_request_consent
```

- [Configuration Overview](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#configuration-overview)
 - [Tracker Store Configuration](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#tracker-store-configuration)
 - [Anonymization Rules](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#anonymization-rules)
 - [Event Broker Configuration](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#event-broker-configuration)
- [Integrating with Consent Management Platforms](https://rasa.com/docs/reference/config/pii-management/configuring-pii-management/#integrating-with-consent-management-platforms)