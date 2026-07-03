# Source: https://rasa.com/docs/reference/channels/rocketchat

On this page

## Getting Credentials[​](https://rasa.com/docs/reference/channels/rocketchat/#getting-credentials 'Direct link to Getting Credentials')

**How to set up Rocket.Chat:**

1. Create a user that will be used to post messages, and set its credentials at credentials file.
 
2. Create a Rocket.Chat outgoing webhook by logging in as admin to Rocket.Chat and going to **Administration > Integrations > New Integration**.
 
3. Select **Outgoing Webhook**.
 
4. Set **Event Trigger** section to value **Message Sent**.
 
5. Fill out the details, including the channel you want the bot listen to. Optionally, it is possible to set the **Trigger Words** section with `@yourbotname` so that the bot doesn't trigger on everything that is said.
 
6. In the **URLs** section, set the URL to `http://<host>:<port>/webhooks/rocketchat/webhook`, replacing the host and port with the appropriate values from your running Rasa server.
 

For more information on the Rocket.Chat Webhooks, see the [Rocket.Chat Guide](https://docs.rocket.chat/docs/integrations).

## Running on RocketChat[​](https://rasa.com/docs/reference/channels/rocketchat/#running-on-rocketchat 'Direct link to Running on RocketChat')

Add the RocketChat credentials to your `credentials.yml`:

```
rocketchat:  user: "yourbotname"  password: "YOUR_PASSWORD"  server_url: "https://demo.rocket.chat"
```

Restart your Rasa server to make the new channel endpoint available for RocketChat to send messages to.

- [Getting Credentials](https://rasa.com/docs/reference/channels/rocketchat/#getting-credentials)
- [Running on RocketChat](https://rasa.com/docs/reference/channels/rocketchat/#running-on-rocketchat)