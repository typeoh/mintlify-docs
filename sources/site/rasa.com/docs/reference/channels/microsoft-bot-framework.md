# Source: https://rasa.com/docs/reference/channels/microsoft-bot-framework

On this page

You first have to create a Microsoft app to get credentials. Once you have them you can add these to your `credentials.yml`.

The endpoint URL that Microsoft Bot Framework should send messages to will look like `http://<host>:<port>/webhooks/botframework/webhook`, replacing the host and port with the appropriate values from your running Rasa server.

## Running on Microsoft Bot Framework[​](https://rasa.com/docs/reference/channels/microsoft-bot-framework/#running-on-microsoft-bot-framework 'Direct link to Running on Microsoft Bot Framework')

Add the Botframework credentials to your `credentials.yml`:

```
botframework:  app_id: "MICROSOFT_APP_ID"  app_password: "MICROSOFT_APP_PASSWORD"
```

Restart your Rasa server to make the new channel endpoint available for Microsoft Bot Framework to send messages to.

- [Running on Microsoft Bot Framework](https://rasa.com/docs/reference/channels/microsoft-bot-framework/#running-on-microsoft-bot-framework)