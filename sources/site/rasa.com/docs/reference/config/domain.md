# Source: https://rasa.com/docs/reference/config/domain

On this page

In Rasa, your `domain` defines the universe in which your assistant operates. Specifically, it lists:

- The `responses` that can be used as templated messages to send to a user.
- The custom `actions` that can be predicted by dialogue policies.
- The `slots` that act as your assistant's memory throughout a conversation.
- Session configuration parameters including inactivity timeout.

If you are building an NLU-based assistant, refer to the [Domain documentation](https://legacy-docs-oss.rasa.com/docs/rasa/domain) to see how intents, entities, slot mappings, and slot featurization can be configured in your domain.

## Multiple Domain Files[​](https://rasa.com/docs/reference/config/domain/#multiple-domain-files 'Direct link to Multiple Domain Files')

The domain can be defined as a single YAML file or split across multiple files in a directory. When split across multiple files, the domain contents will be read and automatically merged together. You can also manage your responses, slots, custom actions in [Rasa Studio](https://rasa.com/docs/studio/intro/).

Using the [command line interface](https://rasa.com/docs/reference/api/command-line-interface/#rasa-train), you can train a model with split domain files by running:

```
rasa train --domain path_to_domain_directory
```

## Responses[​](https://rasa.com/docs/reference/config/domain/#responses 'Direct link to Responses')

Responses are templated messages that your assistant can send to your user. Responses can contain rich content like buttons, images, and custom json payloads. Every response is also an [action](https://rasa.com/docs/reference/config/domain/#actions), meaning that it can be used directly in an `action` step in a flow. Responses can be defined directly in the domain file under the `responses` key. For more information on responses and how to define them, see [Responses](https://rasa.com/docs/reference/primitives/responses/).

## Actions[​](https://rasa.com/docs/reference/config/domain/#actions 'Direct link to Actions')

[Actions](https://rasa.com/docs/reference/primitives/actions/) are the things your bot can do. For example, an action could:

- respond to a user,
 
- make an external API call,
 
- query a database, or
 
- just about anything!
 

All [custom actions](https://rasa.com/docs/reference/primitives/custom-actions/) should be listed in your domain.

Rasa also has [default actions](https://rasa.com/docs/reference/primitives/default-actions/) which you do not need to list in your domain.

## Slots[​](https://rasa.com/docs/reference/config/domain/#slots 'Direct link to Slots')

Slots are your assistant's memory. They act as a key-value store which can be used to store information the user provided (e.g. their home city) as well as information gathered about the outside world (e.g. the result of a database query). Check out the [Slots reference](https://rasa.com/docs/reference/primitives/slots/) for more information.

## Session configuration[​](https://rasa.com/docs/reference/config/domain/#session-configuration 'Direct link to Session configuration')

A conversation session represents the dialogue between the assistant and the user. Conversation sessions can begin in three ways:

1. the user begins the conversation with the assistant,
 
2. the user sends their first message after a configurable period of inactivity, or
 
3. a manual session start is triggered with the `/session_start` intent message.
 

You can define the period of inactivity after which a new conversation session is triggered in the domain under the `session_config` key.

Available parameters are:

- `session_expiration_time` defines the time of inactivity in minutes after which a new session will begin.
- `carry_over_slots_to_new_session` determines whether existing set slots should be carried over to new sessions.

The default session configuration looks as follows:

```
session_config:  session_expiration_time: 60  # value in minutes, 0 means infinitely long  carry_over_slots_to_new_session: true  # set to false to forget slots between sessions
```

This means that if a user sends their first message after 60 minutes of inactivity, a new conversation session is triggered, and that any existing slots are carried over into the new session. Setting the value of `session_expiration_time` to `0` means that sessions will not end (note that the `action_session_start` action will still be triggered at the very beginning of conversations).

note

A session start triggers the default action `action_session_start`. Its default implementation moves all existing slots into the new session. Note that all conversations begin with an `action_session_start`. Overriding this action could for instance be used to initialize the tracker with slots from an external API call, or to start the conversation with a bot message. The docs on [Customizing the session start action](https://rasa.com/docs/reference/primitives/default-actions/#customization) shows you how to do that.

## Select which actions should receive domain[​](https://rasa.com/docs/reference/config/domain/#select-which-actions-should-receive-domain 'Direct link to Select which actions should receive domain')

New in 3.4.3

You can control if an action should receive a domain or not.

To do this you must first enable selective domain in you endpoint configuration for `action_endpoint` in `endpoints.yml`.

endpoints.yml

```
action_endpoint: url: "http://localhost:5055/webhook" # URL to your action server enable_selective_domain: true
```

**After selective domain for custom actions is enabled, domain will be sent only to those custom actions which have specifically stated that they need it.** To specify if an action needs the domain add `{send_domain: true}` to custom action in the list of actions in `domain.yml`:

domain.yml

```
actions:  - action_hello_world: {send_domain: True} # will receive domain  - action_calculate_mass_of_sun # will not receive domain  - validate_my_form # will receive domain
```

- [Multiple Domain Files](https://rasa.com/docs/reference/config/domain/#multiple-domain-files)
- [Responses](https://rasa.com/docs/reference/config/domain/#responses)
- [Actions](https://rasa.com/docs/reference/config/domain/#actions)
- [Slots](https://rasa.com/docs/reference/config/domain/#slots)
- [Session configuration](https://rasa.com/docs/reference/config/domain/#session-configuration)
- [Select which actions should receive domain](https://rasa.com/docs/reference/config/domain/#select-which-actions-should-receive-domain)