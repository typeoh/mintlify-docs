# Source: https://rasa.com/docs/reference/channels/jambonz

On this page

New in 3.11

The Jambonz connector has been made generally available in Rasa Pro 3.11. It was released as a beta feature in Rasa Pro 3.10.

Use this channel to connect your Rasa assistant to [Jambonz](https://www.jambonz.org/). Jambonz is a Voice Gateway that can interface with various customer support platforms like Genesys, Avaya, and Twilio using SIP.

## Basic Rasa configuration[​](https://rasa.com/docs/reference/channels/jambonz/#basic-rasa-configuration 'Direct link to Basic Rasa configuration')

Create or edit your `credentials.yml` and add a new channel configuration:

credentials.yml

```
jambonz:  # Basic Authentication  username: "<username>"  password: "<password>"
```

Basic Authentication

Support for `username` and `password` parameters is only available on Rasa Pro 3.11.8+, Rasa Pro 3.12.7+ and later releases.

Jambonz channel can be secured by Basic Authentication with these parameters:

- `username` (optional): Basic authentication username for Jambonz channel.
 
- `password` (optional): Basic authentication password for Jambonz channel.
 

You can run that assistant using `rasa run`. To configure the Jambonz channel, you will need a URL to your Rasa assistant. You can use a tunneling solution like [ngrok](https://ngrok.com/) to expose your assistant for development.

Bot URLs for development

Visit this [section](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/#testing-channels-on-your-local-machine) to learn how to generate the required bot URL when testing the channel on your local machine.

## Configuring Jambonz[​](https://rasa.com/docs/reference/channels/jambonz/#configuring-jambonz 'Direct link to Configuring Jambonz')

To route calls to your Rasa assistant, you need to have a Jambonz deployment, an account and a phone number.

1. Sign up for [Jambonz cloud](https://jambonz.cloud/) or use your own on-premise deployment.
 
2. Once logged in, create a _Carrier_ which will provide your phone number (e.g. twilio).
 
3. Create an _Application_. To route calls to your assistant, Jambonz needs a webhook URL. You'll either need to deploy your assistant to a server and expose it to Jambonz or for development use a tunneling solution like [ngrok](https://ngrok.com/).
 
 The webhook URL will be in the format `wss://<your-server>/webhooks/jambonz/websocket`
 
 or in the case of ngrok look like this: `wss://recently-communal-duckling.ngrok-free.app/webhooks/jambonz/websocket`
 
4. If your channel is secured by Basic Authentication in the above step, then select "Use HTTP basic authentication" and fill Username, Password fields.
 
5. Set up a "Phone Number". The configuration of the phone number should point to the application you created in the previous step.
 

## Usage[​](https://rasa.com/docs/reference/channels/jambonz/#usage 'Direct link to Usage')

### Receiving messages from a user[​](https://rasa.com/docs/reference/channels/jambonz/#receiving-messages-from-a-user 'Direct link to Receiving messages from a user')

When a user speaks on the phone, Jambonz will send a text message (after it is processed by the speech-to-text engine) to your assistant like any other channel. This message will be interpreted by Rasa and you can then drive the conversation with flows.

### Sending messages to a user[​](https://rasa.com/docs/reference/channels/jambonz/#sending-messages-to-a-user 'Direct link to Sending messages to a user')

Your bot will respond with text messages like with any other channel. The text-to-speech engine will convert the text and deliver it as a voice message to the user.

Here is an example:

```
utter_greet:  - text: "Hello! isn’t every life and every work beautiful?"
```

note

Only text messages are allowed. Images, attachments, and buttons cannot be used with a voice channel.

### Handling conversation events[​](https://rasa.com/docs/reference/channels/jambonz/#handling-conversation-events 'Direct link to Handling conversation events')

New in 3.11

We have unified the call handling patterns across Voice Channel Connectors, all voice channels handle call starts, ends and metadata in a similar manner.

Non-voice events can also be handled by the bot. Here are a few examples:

| Event | intent | Description |
| --- | --- | --- |
| `start` | `session_start` | Jambonz will send this intent when it picks-up a phone call. By default, this intent triggers the `pattern_session_start` which you can customize. |
| `end` | `session_end` | Jambonz will send this intent when the call is disconnected by the user. |
| `DTMF` | \- | Jambonz will send DTMF (numbers on the phone pressed by a user ) as normal text messages with confidence 1.0. |

Here is a simple modification of the default session start to greet the user with an utterance:

```
flows:  pattern_session_start:    description: flow used to start the conversation    name: pattern session start    nlu_trigger:    - intent: session_start    steps:    - action: utter_greet
```

- [Basic Rasa configuration](https://rasa.com/docs/reference/channels/jambonz/#basic-rasa-configuration)
- [Configuring Jambonz](https://rasa.com/docs/reference/channels/jambonz/#configuring-jambonz)
- [Usage](https://rasa.com/docs/reference/channels/jambonz/#usage)
 - [Receiving messages from a user](https://rasa.com/docs/reference/channels/jambonz/#receiving-messages-from-a-user)
 - [Sending messages to a user](https://rasa.com/docs/reference/channels/jambonz/#sending-messages-to-a-user)
 - [Handling conversation events](https://rasa.com/docs/reference/channels/jambonz/#handling-conversation-events)