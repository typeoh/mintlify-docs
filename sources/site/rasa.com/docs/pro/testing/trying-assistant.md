# Source: https://rasa.com/docs/pro/testing/trying-assistant

On this page

Once you’re ready to talk to your newly built flow, train your assistant by running:

```
rasa train
```

After you have trained your assistant successfully, start talking to it in your browser by running:

```
rasa inspect
```

This command opens **Inspector**, a browser-based tool that launches a local version of your assistant, allowing you to chat with it and watch what happens under the hood in real time.

## Interface Description[​](https://rasa.com/docs/pro/testing/trying-assistant/#interface-description 'Direct link to Interface Description')

The **`rasa inspect`** command starts an assistant based on the last trained model and opens a new tab in the browser with the following four panes (right-to-left and top-down):

- **Chat Widget**, where you can field-test a conversation with your assistant,
- **Flow View** visualizing the currently active flow and step of the conversation,
- **Stack View** listing bottom-up the activated flows and their latest steps, and
- **Slots, End-2-End Test and Tracker State View** containing details of the data captured during the chat.

### Chat Widget[​](https://rasa.com/docs/pro/testing/trying-assistant/#chat-widget 'Direct link to Chat Widget')

Start a free-form chat with your assistant. The three remaining debugging panes will show the intrinsics of the conversation development and allow to check if the correct flows are activated at the right moments and if the conversation is proceeding as planned.

### Flow View[​](https://rasa.com/docs/pro/testing/trying-assistant/#flow-view 'Direct link to Flow View')

Visualize the currently active flow in a flowchart and highlight the latest step. By clicking on the rows of the stack table, you can view the specific flows activated at different points in the conversation. At the bottom of the pane you can find a "restart conversation" button, which will discard any previous information and start a new chat. A page reload will cause the same behavior.

### Stack View[​](https://rasa.com/docs/pro/testing/trying-assistant/#stack-view 'Direct link to Stack View')

The stack section gives a bottom-up chronological overview of the activated flows and their latest steps. The topmost item in that table represents the currently active flow step that the assistant is trying to complete. The flows listed below are 'paused' and will resume once every flow in a row above them has been complete.

- **ID**: A unique identifier for each stack frame.
- **Flow**: The active flow's identifier, linked to the **`flows.yml`** file.
- **Step ID**: The current step's identifier, also found in the **`flows.yml`** file.

### Slots, End-2-End Test and Tracker State View[​](https://rasa.com/docs/pro/testing/trying-assistant/#slots-end-2-end-test-and-tracker-state-view 'Direct link to Slots, End-2-End Test and Tracker State View')

Here you can check details of the data collected so far:

#### Slots[​](https://rasa.com/docs/pro/testing/trying-assistant/#slots 'Direct link to Slots')

[Slots](https://rasa.com/docs/reference/config/domain/#slots) are dynamic variables where the assistant stores and retrieves information throughout the conversation. They are defined in your **`domain.yml`** file.

#### End-2-End Test[​](https://rasa.com/docs/pro/testing/trying-assistant/#end-2-end-test 'Direct link to End-2-End Test')

The test conversation carried out in the chat widget is represented here in the [end-to-end test format](https://rasa.com/docs/reference/testing/test-cases/). This helps to quickly transform the conversation into a test case which when added to your end-to-end test set can enhance the robustness of the assistant.

#### Tracker State[​](https://rasa.com/docs/pro/testing/trying-assistant/#tracker-state 'Direct link to Tracker State')

The tracker state view offers detailed information about past events and the current state of a conversation. It presents the conversation history through a series of recorded events, such as the message sent by the user accompanied by the command predicted by the LLM, slots that are set, conversation repair patterns triggered and actions executed by the bot.

## Inspecting Voice Assistants[​](https://rasa.com/docs/pro/testing/trying-assistant/#inspecting-voice-assistants 'Direct link to Inspecting Voice Assistants')

Rasa Inspector can also be used for testing voice conversations. To get started, add the built-in `browser_audio` channel in the `credentials.yml` file of the assistant with the ASR and TTS services of your choice. In this example, we will use Cartesia and Deepgram:

credentials.yml

```
browser_audio:  server_url: localhost  asr:    name: deepgram  tts:    name: cartesia
```

Ensure that the appropriate API keys are added to the environment variables, in this case, `CARTESIA_API_KEY` and `DEEPGRAM_API_KEY`. Checkout the [speech integrations](https://rasa.com/docs/reference/integrations/speech-integrations/) page for more information.

Launch the Inspector with the `--voice` flag using the command `rasa inspect --voice`.

## Inspecting Conversations over External Channels[​](https://rasa.com/docs/pro/testing/trying-assistant/#inspecting-conversations-over-external-channels 'Direct link to Inspecting Conversations over External Channels')

Rasa Inspector can also be used to debug conversations happening on external channels. When you have certain channels defined in `credentials.yml`, you can run Rasa with Inspector using the command `rasa run --inspect`.

Rasa will create Inspector URLs for each channel. You can start a conversation over the external channel and view the conversation on Rasa Inspector along with all the relevant information about the conversation. It can be used to inspect conversations over any external channel including voice channels (like Twilio Media Streams).

For example, to inspect conversations over Rest channel. Ensure that `credentials.yml` contains the channel you would like to inspect:

credentials.yml

```
rest:
```

Lauch Rasa Server with Inspector with the command `rasa run --inspect`. Open the inspector URL mentioned in the logs:

```
Development inspector for channel rest is running.To inspect conversations, visit http://localhost:5005/webhooks/rest/inspect.html
```

Now, the first conversation started over the REST channel will be visible on the Inspector. You can try it with the following cURL request:

```
curl  -X POST \  'http://localhost:5005/webhooks/rest/webhook' \  --header 'Content-Type: application/json' \  --data-raw '{  "sender": "test_user",  "message": "I would like to transfer money"}'
```

- [Interface Description](https://rasa.com/docs/pro/testing/trying-assistant/#interface-description)
 - [Chat Widget](https://rasa.com/docs/pro/testing/trying-assistant/#chat-widget)
 - [Flow View](https://rasa.com/docs/pro/testing/trying-assistant/#flow-view)
 - [Stack View](https://rasa.com/docs/pro/testing/trying-assistant/#stack-view)
 - [Slots, End-2-End Test and Tracker State View](https://rasa.com/docs/pro/testing/trying-assistant/#slots-end-2-end-test-and-tracker-state-view)
- [Inspecting Voice Assistants](https://rasa.com/docs/pro/testing/trying-assistant/#inspecting-voice-assistants)
- [Inspecting Conversations over External Channels](https://rasa.com/docs/pro/testing/trying-assistant/#inspecting-conversations-over-external-channels)