# Source: https://rasa.com/docs/studio/test/try-your-assistant

On this page

**Try your assistant** in Rasa Studio provides powerful tools to test and inspect how your trained assistant behaves. Whether you're reviewing the logic behind a conversation or converting an interaction into a test, this feature helps ensure everything works as expected.

## Before You Start[​](https://rasa.com/docs/studio/test/try-your-assistant/#before-you-start 'Direct link to Before You Start')

Make sure your assistant has been trained at least once.

> Need help? [See the training guide](https://rasa.com/docs/studio/test/train-your-assistant/) for more information on how to train your assistant.

## Starting a Conversation[​](https://rasa.com/docs/studio/test/try-your-assistant/#starting-a-conversation 'Direct link to Starting a Conversation')

To access the **Try your assistant** page:

1. Click **Try your assistant** in the left-hand navigation of Rasa Studio.
 
2. If your assistant supports multiple languages, choose one from the dropdown.
 
 ![Select Language](https://rasa.com/docs/assets/images/select-language-8305357f85a9cab798988147a196cad9.png)
 
3. Ensure the correct assistant version is selected.
 
 ![See Assistant Version](https://rasa.com/docs/assets/images/assistant-version-tya-fb79138ce33e29cef7a6467b290d93f0.png)
 
 Want to switch versions? Go to **Assistant Versions** and click the chat icon next to the version you want to test.
 
 ![Select Assistant Version](https://rasa.com/docs/assets/images/select-assistant-version-589f664070bd8549cab2a5d7e07f547a.png)
 

Once ready, start the conversation by sending a message.

## Switching Between Chat and Voice[​](https://rasa.com/docs/studio/test/try-your-assistant/#switching-between-chat-and-voice 'Direct link to Switching Between Chat and Voice')

If your license includes voice capabilities, you can switch between chat and voice modes:

1. Use the toggle above the message input to switch to voice. Then click "Call".
 
 ![Switch to Voice](https://rasa.com/docs/assets/images/tya-voice1-2593dd2038e2606d535f8fcf58b16097.png)
 
2. Allow microphone access, speak to your assistant, and view real-time transcripts in the chat panel. Currently, voice input is limited to English.
 
 ![Voice Conversation](https://rasa.com/docs/assets/images/tya-voice2-8d72e88554589d9a6905fc244d6ef4c5.png)
 
3. By default, Studio uses Deepgram for speech-to-text (STT) and Cartesia for text-to-speech (TTS). You can customize this setup and choose a different provider outside of Studio. [Learn more about building voice assistants](https://rasa.com/docs/pro/build/voice-assistants/)
 

## Activate Assistant Version[​](https://rasa.com/docs/studio/test/try-your-assistant/#activate-assistant-version 'Direct link to Activate Assistant Version')

The assistant version will be inactive after one hour or when the maximum number of active assistants is reached. To reactivate and test it, go to **Assistant versions** and click the arrow icon. Once reactivated, you can test the version again.

![Activate Assistant Version](https://rasa.com/docs/assets/images/activate-assistant-version-622252e3695959174c928c59e5433195.png)

## Basic Mode: Live Flow Preview[​](https://rasa.com/docs/studio/test/try-your-assistant/#basic-mode-live-flow-preview 'Direct link to Basic Mode: Live Flow Preview')

Basic Mode is the default mode when testing. It provides:

- A live chat interface
- A **Live Flow Preview** panel that shows your assistant moving through different conversation paths

Use this mode to quickly check if your assistant follows the correct logic.

![Basic Mode in Try Your Assistant](https://rasa.com/docs/assets/images/tya-basic-e7b9d7a1ce52205b428be3e5e4bb666e.png)

## Inspector Mode: Debugging and Insights[​](https://rasa.com/docs/studio/test/try-your-assistant/#inspector-mode-debugging-and-insights 'Direct link to Inspector Mode: Debugging and Insights')

Toggle **Inspector Mode** (top right corner) to unlock deeper insights:

![Enabling Inspector Mode](https://rasa.com/docs/assets/images/tya-inspector-19c04f31298fc7260088fb53440f89de.png)

#### 1\. View Events[​](https://rasa.com/docs/studio/test/try-your-assistant/#1-view-events 'Direct link to 1. View Events')

Click on any user or assistant message to view slots set, flows triggered, and other key events.

![Tracking Key Events](https://rasa.com/docs/assets/images/slots-event-details-936b5be608163293070eeea07a1d3205.png)

#### 2\. Predicted Commands[​](https://rasa.com/docs/studio/test/try-your-assistant/#2-predicted-commands 'Direct link to 2. Predicted Commands')

Select a user message to view the predicted command and understand why a specific path was taken.

![Predicted Commands in Inspector Mode](https://rasa.com/docs/assets/images/commands-02d9774c827add91ac4e970f29b8d758.png)

#### 3\. Model Details[​](https://rasa.com/docs/studio/test/try-your-assistant/#3-model-details 'Direct link to 3. Model Details')

Click on an assistant message to see the prompt, input, and token usage — useful for understanding performance and API costs.

![Model Details in Inspector Mode](https://rasa.com/docs/assets/images/model-details-e99aff82d3e07896b8e47240ff15bbf3.png)

## Speed Up Testing[​](https://rasa.com/docs/studio/test/try-your-assistant/#speed-up-testing 'Direct link to Speed Up Testing')

Beyond analysis, Inspector also provides tools to speed up your conversation testing workflow.

#### Conversation Replay[​](https://rasa.com/docs/studio/test/try-your-assistant/#conversation-replay 'Direct link to Conversation Replay')

1. Select the **Replay** icon next to a "Waiting for user input" event. This will trigger the conversation to replay back that point.
 
2. Try out different inputs for testing different outcomes of the conversation.
 
 ![Replay your assistant](https://rasa.com/docs/assets/images/replay-bd670aa55fa042066bc9c428ca006dc9.png)
 

#### Generate Tests from Chat[​](https://rasa.com/docs/studio/test/try-your-assistant/#generate-tests-from-chat 'Direct link to Generate Tests from Chat')

Once you're happy with a conversation, you can export it to **share with your team**, or **generate a test case** for your assistant.

Select the menu icon in the top right of your conversation panel.

- **Download logs** if you'd like to review the full deployment logs of your running assistant.
- **Download conversation** If you'd like save your conversation in a shareable transcript.
- **Download E2E Tests** If you'd like to save your conversation and use it as an **end-to-end (E2E) test** to validate assistant behavior.

For more details on the testing format and how to run E2E tests as part of your workflow see the [Pro guides for testing](https://rasa.com/docs/pro/testing/evaluating-assistant/).

![Download](https://rasa.com/docs/assets/images/download-5f4faa9a1d267666187a20beaca29733.png)

- [Before You Start](https://rasa.com/docs/studio/test/try-your-assistant/#before-you-start)
- [Starting a Conversation](https://rasa.com/docs/studio/test/try-your-assistant/#starting-a-conversation)
- [Switching Between Chat and Voice](https://rasa.com/docs/studio/test/try-your-assistant/#switching-between-chat-and-voice)
- [Activate Assistant Version](https://rasa.com/docs/studio/test/try-your-assistant/#activate-assistant-version)
- [Basic Mode: Live Flow Preview](https://rasa.com/docs/studio/test/try-your-assistant/#basic-mode-live-flow-preview)
- [Inspector Mode: Debugging and Insights](https://rasa.com/docs/studio/test/try-your-assistant/#inspector-mode-debugging-and-insights)
- [Speed Up Testing](https://rasa.com/docs/studio/test/try-your-assistant/#speed-up-testing)