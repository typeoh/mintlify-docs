# Source: https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher

On this page

A dispatcher is an instance of the `CollectingDispatcher` class used to generate responses to send back to the user.

## CollectingDispatcher[​](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/#collectingdispatcher 'Direct link to CollectingDispatcher')

`CollectingDispatcher` has one method, `utter_message`, and one attribute, `messages`. It is used in an action's `run` method to add responses to the payload returned to the Rasa server. The Rasa server will in turn add `BotUttered` events to the tracker for each response. Responses added using the dispatcher should therefore not be returned explicitly as [events](https://rasa.com/docs/reference/integrations/action-server/events/). For example, the following custom action returns no events explicitly but will return the response, "Hi, User!" to the user:

```
class ActionGreetUser(Action):    def name(self) -> Text:        return "action_greet_user"    async def run(        self,        dispatcher: CollectingDispatcher,        tracker: Tracker,        domain: Dict[Text, Any],    ) -> List[EventType]:        dispatcher.utter_message(text = "Hi, User!")        return []
```

### CollectingDispatcher.utter\_message[​](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/#collectingdispatcherutter_message 'Direct link to CollectingDispatcher.utter_message')

The `utter_message` method can be used to return any type of response to the user.

#### **Parameters**[​](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/#parameters 'Direct link to parameters')

The `utter_message` method takes the following optional arguments. Passing no arguments will result in an empty message being returned to the user. Passing multiple arguments will result in a rich response (e.g. text and buttons) being returned to the user.

- `text`: The text to return to the user.

```
dispatcher.utter_message(text = "Hey there")
```

- `image`: An image URL or file path that will be used to display an image to the user.

```
dispatcher.utter_message(image = "https://i.imgur.com/nGF1K8f.jpg")
```

- `json_message`: A custom json payload as a dictionary. It can be used to send [channel specific responses](https://rasa.com/docs/reference/primitives/responses/). The following example would return a date picker in Slack:

```
date_picker = {  "blocks":[    {      "type": "section",      "text":{        "text": "Make a bet on when the world will end:",        "type": "mrkdwn"      },      "accessory":      {        "type": "datepicker",        "initial_date": "2019-05-21",        "placeholder":        {          "type": "plain_text",          "text": "Select a date"        }      }    }  ]}dispatcher.utter_message(json_message = date_picker)
```

- `response`: The name of a response to return to the user. This response should be specified in your assistants [domain](https://rasa.com/docs/reference/config/domain/).

```
dispatcher.utter_message(response = "utter_greet")
```

- `attachment`: A URL or file path of an attachment to return to the user.

```
dispatcher.utter_message(attachment = "")
```

- `buttons`: A list of buttons to return to the user. Each button is a dictionary and should have a `title` and a `payload` key. A button can include other keys, but these will only be used if a specific channel looks for them. The button's `payload` will be sent as a user message if the user clicks the button.

```
dispatcher.utter_message(buttons = [                {"payload": "/affirm", "title": "Yes"},                {"payload": "/deny", "title": "No"},            ])
```

- `elements`: These are specific to using Facebook as a messaging channel. For details of expected format see [Facebook's documentation](https://developers.facebook.com/docs/messenger-platform/send-messages/template/generic/)
 
- `**kwargs`: arbitrary keyword arguments, which can be used to specify values for [variable interpolation in response variations](https://rasa.com/docs/reference/primitives/responses/). For example, given the following response:
 

```
responses:  utter_greet_name:    - text: Hi {name}!
```

You could specify the name with:

```
dispatcher.utter_message(response = "utter_greet_name", name = "Aimee")
```

#### **Return type**[​](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/#return-type 'Direct link to return-type')

`None`

- [CollectingDispatcher](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/#collectingdispatcher)
 - [CollectingDispatcher.utter\_message](https://rasa.com/docs/reference/integrations/action-server/sdk-dispatcher/#collectingdispatcherutter_message)