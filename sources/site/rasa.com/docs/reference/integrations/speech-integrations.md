# Source: https://rasa.com/docs/reference/integrations/speech-integrations

On this page

## Audio Format[​](https://rasa.com/docs/reference/integrations/speech-integrations/#audio-format 'Direct link to Audio Format')

Rasa uses a common intermediate audio format called `RasaAudioBytes` that acts as a standard data format to prevent complexity between different channels, ASR engines, and TTS engines.

Rasa supports following audio formats:

- **8 bit 8KHz μ-law (mulaw) encoding mono channel**
- **Linear 16 bit 24KHz PCM mono channel**
- **Linear 16 bit 48KHz PCM mono channel**

Rasa will automatically try to negotiate **Linear 16 bit 24KHz PCM mono** audio format for AudioCodes and Jambonz channels.

Other channels, except for browser channel, will use **8 bit 8KHz μ-law (mulaw) encoding mono channel**.

Browser channel supports all three mentioned formats. By default, browser channel will use **Linear 16 bit 48KHz PCM mono**. Choosing the audio format is done at Rasa start by supplying `sample_rate` config property in `credentials.yml`:

- 8 bit 8KHz mulaw mono
- 16 bit 24KHz PCM mono
- 16 bit 48KHz PCM mono

credentials.yml

```
browser:  url: localhost  sample_rate: 8000
```

credentials.yml

```
browser:  url: localhost  sample_rate: 24000
```

credentials.yml

```
browser:  url: localhost  sample_rate: 48000
```

Rime TTS and Browser channel

Rime TTS does not support 48KHz sample rate. If Browser channel is set to 48KHz sample rate, Rasa will resample audio received from Rime from 24KHz to 48KHz.

Rasa uses the library [audioop-lts](https://pypi.org/project/audioop-lts/) for conversion between audio encodings (functions like `ulaw2lin()` or [`lin2ulaw()`](https://docs.python.org/3.12/library/audioop.html#audioop.lin2ulaw) and `ratecv`).

## Automatic Speech Recognition (ASR)[​](https://rasa.com/docs/reference/integrations/speech-integrations/#automatic-speech-recognition-asr 'Direct link to Automatic Speech Recognition (ASR)')

This section describes the supported integrations with Automatic Speech Recognition (ASR) or Speech To Text (STT) services.

### Deepgram[​](https://rasa.com/docs/reference/integrations/speech-integrations/#deepgram 'Direct link to Deepgram')

Use the environment variable `DEEPGRAM_API_KEY` for Deepgram API Key. You can request a key from [Deepgram](https://deepgram.com/). It can be configured in a Voice Stream channel as follows:

credentials.yml

```
browser_audio:  # ... other configuration  asr:    name: deepgram
```

Turn Detection

Deepgram uses two mechanisms to detect when a speaker has finished talking:

1. **Endpointing**: Uses Voice Activity Detection (VAD) to detect silence after speech
2. **UtteranceEnd**: Looks at word timings to detect gaps between words

The configuration parameters `endpointing` and `utterance_end_ms` below control these features respectively. For noisy environments, `utterance_end_ms` may be more reliable as it ignores non-speech audio. [Read more on Deepgram Documentation](https://developers.deepgram.com/docs/understanding-end-of-speech-detection)

#### Configuration parameters[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-parameters 'Direct link to Configuration parameters')

- `endpoint`: Optional, defaults to `api.deepgram.com` - The endpoint URL for the Deepgram API.
- `endpointing`: Optional, defaults to `400` - Number of milliseconds of silence to determine the end of speech.
- `language` (deprecated, use `language_map`): Optional, defaults to `en` - The language code for the speech recognition.
- `model` (deprecated, use `language_map`): Optional, defaults to `nova-2-general` - The model to be used for speech recognition.
- `smart_format`: Optional, defaults to `true` - Boolean value to enable or disable [Deepgram's smart formatting](https://developers.deepgram.com/docs/smart-format).
- `utterance_end_ms`: Optional, defaults to `1000` - Time in milliseconds to wait before considering an utterance complete.
- `language_map`: Optional, defaults to mapping for english language and model - multilingual agent's mapping between languages set in `config.yml` and `language` and `model` parameters for Deepgram.

Language and Model Configuration

`language` and `model` are mutually exclusive with `language_map`.

#### Deepgram Multilingual Support[​](https://rasa.com/docs/reference/integrations/speech-integrations/#deepgram-multilingual-support 'Direct link to Deepgram Multilingual Support')

To configure Deepgram for multilingual agent use `language_map` property. Keys listed in the map must match languages (`language` and `additional_langauges`) mentioned in `config.yml`. Deepgram supports `language` and `model` as configuration parameters inside `language_map`.

config.yml

```
recipe: default.v1language: enadditional_languages:  - de-DEpipeline:- name: SingleStepLLMCommandGeneratorpolicies:- name: FlowPolicyassistant_id: 20240418-073244-narrow-archive
```

credentials.yml

```
browser_audio:  asr:    name: deepgram    language_map:      en:        language: eng        model: mistv2      de-DE:        language: de        model: mistv2
```

To see the full list of supported languages and models for Deepgram visit [Models & Languages Overview](https://developers.deepgram.com/docs/models-languages-overview).

### Azure[​](https://rasa.com/docs/reference/integrations/speech-integrations/#azure 'Direct link to Azure')

Requires the python library `azure-cognitiveservices-speech`. The API Key can be set with the environment variable `AZURE_SPEECH_API_KEY`. Sample configuration looks as follows:

credentials.yml

```
browser_audio:  # ... other configuration  asr:    name: azure
```

#### Configuration parameters[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-parameters-1 'Direct link to Configuration parameters')

- `language` (deprecated, use `language_map`): Required. The language code for the speech recognition. (See [Azure documentation](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support?tabs=stt) for a list of languages).
- `speech_region`: Optional, defaults to `None` - The region identifier for the Azure Speech service, such as `westus`. Ensure that the region matches the region of your subscription.
- `speech_endpoint`: Optional, defaults to `None` - The service endpoint to connect to. You can use it when you have Azure Speech service behind a reverse proxy.
- `speech_host`: Optional, defaults to `None` - The service host to connect to. Standard resource path will be assumed. Format is "protocol://host:port" where ":port" is optional.
- `language_map`: Optional, defaults to mapping for english language - multilingual agent's mapping between languages set in `config.yml` and `language` parameter for Azure.

While `speech_region`, `speech_endpoint` and `speech_host` are optional parameters. They cannot be all empty at the same time. In that case, speech\_region is set to `eastus`.

When connecting to Azure Cloud, parameter `speech_region` is enough. Here is an example config,

credentials.yml

```
browser_audio:  server_url: localhost  asr:    name: azure    language: de-DE    speech_region: germanywestcentral  tts:    name: azure    language: de-DE    voice: de-DE-KatjaNeural    speech_region: germanywestcentral
```

Language Configuration

`language` and `language_map` are mutually exclusive properties.

#### Azure ASR Multilingual Support[​](https://rasa.com/docs/reference/integrations/speech-integrations/#azure-asr-multilingual-support 'Direct link to Azure ASR Multilingual Support')

info

New in Rasa 3.16.0

To configure Azure ASR for multilingual agent use `language_map` property. Keys listed in the map must match languages (`language` and `additional_langauges`) mentioned in `config.yml`. Azure ASR supports `language` as configuration parameter inside `language_map`.

config.yml

```
recipe: default.v1language: enadditional_languages:  - de-DEpipeline:- name: SingleStepLLMCommandGeneratorpolicies:- name: FlowPolicyassistant_id: 20240418-073244-narrow-archive
```

credentials.yml

```
browser_audio:  asr:    name: azure    speech_region: germanywestcentral    language_map:      en:        language: 'en-US'      de-DE:        language: 'de-DE'
```

To see the full list of supported languages and models for Azure ASR visit [Azure ASR Languages](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support?tabs=stt).

### Others[​](https://rasa.com/docs/reference/integrations/speech-integrations/#others 'Direct link to Others')

Looking for integration with a different ASR service? You can [create your own custom ASR component](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-asr).

## Text To Speech (TTS)[​](https://rasa.com/docs/reference/integrations/speech-integrations/#text-to-speech-tts 'Direct link to Text To Speech (TTS)')

This section describes the supported integrations with Text To Speech (TTS) services. Unless otherwise mentioned, the built-in TTS integrations support input text streaming of generative responses. This means that as the LLM generates text, it is streamed directly to the TTS service in chunks rather than waiting for the complete response before starting speech synthesis. This significantly reduces the response latency of the assistant, which is critical for natural-sounding voice conversations.

### Azure TTS[​](https://rasa.com/docs/reference/integrations/speech-integrations/#azure-tts 'Direct link to Azure TTS')

The API Key can be set with the environment variable `AZURE_SPEECH_API_KEY`. Sample configuration looks as follow:

credentials.yml

```
browser_audio:  # ... other configuration  tts:    name: azure
```

SSML Support

Azure TTS does not support streaming of generative responses [to preserve SSML (Speech Synthesis Markup Language)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/python/tts-text-stream#set-global-properties) functionality. SSML allows for advanced speech control including pronunciation, pauses, emphasis, and voice characteristics.

#### Configuration parameters[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-parameters-2 'Direct link to Configuration parameters')

- `language` (deprecated, use `language_map`): Optional, defaults to `en-US` - The language code for the text-to-speech conversion. (See [Azure documentation](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support?tabs=tts) for a list of languages and voices).
- `voice` (deprecated, use `language_map`): Optional, defaults to `en-US-JennyNeural` - The voice to be used for the text-to-speech conversion. Voice defines the specific characteristic of the voice, such as speaker's gender, age and speaking style.
- `timeout`: Optional, defaults to `10` - The timeout duration in seconds for the text-to-speech request.
- `speech_region`: Optional, defaults to `None` - The region identifier for the Azure Speech service. Ensure that the region matches the region of your subscription.
- `endpoint`: Optional, defaults to `None` - The service endpoint for Azure Speech service.
- `language_map`: Optional, defaults to mapping for english language and voice - multilingual agent's mapping between languages set in `config.yml` and `language` and `voice` parameters for Azure TTS.

Language Configuration

`language` and `voice` properties are mutually exclusive with `language_map`.

#### Azure TTS Multilingual Support[​](https://rasa.com/docs/reference/integrations/speech-integrations/#azure-tts-multilingual-support 'Direct link to Azure TTS Multilingual Support')

info

New in Rasa 3.16.0

To configure Azure TTS for multilingual agent use `language_map` property. Keys listed in the map must match languages (`language` and `additional_langauges`) mentioned in `config.yml`. Azure TTS supports `language` and `voice` as configuration parameters inside `language_map`.

config.yml

```
recipe: default.v1language: enadditional_languages:  - de-DEpipeline:- name: SingleStepLLMCommandGeneratorpolicies:- name: FlowPolicyassistant_id: 20240418-073244-narrow-archive
```

credentials.yml

```
browser_audio:  tts:    name: azure    speech_region: germanywestcentral    language_map:      en:        language: en-US        voice: en-US-JennyMultilingualNeural      de-DE:        language: de-DE        voice: de-DE-KatjaNeural
```

To see the full list of supported languages and models for Azure TTS visit [Azure TTS Languages](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support?tabs=tts).

### Cartesia TTS[​](https://rasa.com/docs/reference/integrations/speech-integrations/#cartesia-tts 'Direct link to Cartesia TTS')

Use the environment variable `CARTESIA_API_KEY` for Cartesia API Key. The API Key requires a [Cartesia](https://www.cartesia.ai/) account. It can be configured in a Voice Stream channel as follows,

credentials.yml

```
browser_audio:  # ... other configuration  tts:    name: cartesia
```

#### Configuration parameters[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-parameters-3 'Direct link to Configuration parameters')

- `language` (deprecated, use `language_map`): Optional, defaults to `en` - The language code for the text-to-speech conversion.
- `voice`: Optional, defaults to `248be419-c632-4f23-adf1-5324ed7dbf1d` - The `id` of the voice to use for text-to-speech conversion. The parameter will be passed to the Cartesia API as `"voice": {"mode": "id","id": "VALUE"}`
- `timeout`: Optional, defaults to `10` - The timeout duration in seconds for the text-to-speech request.
- `model_id`: Optional, defaults to `sonic-latest` - The model ID to be used for the text-to-speech conversion.
- `version`: Optional, defaults to `2026-03-01` - The version of the model to be used for the text-to-speech conversion.
- `endpoint`: Optional, defaults to `https://api.cartesia.ai/tts/sse` - The endpoint URL for the Cartesia API.
- `language_map`: Optional, defaults to mapping for english language - multilingual agent's mapping between languages set in `config.yml` and `language` parameters for Cartesia.

Language Configuration

`language` and `language_map` are mutually exclusive properties.

#### Cartesia Multilingual Support[​](https://rasa.com/docs/reference/integrations/speech-integrations/#cartesia-multilingual-support 'Direct link to Cartesia Multilingual Support')

info

New in Rasa 3.16.0

To configure Cartesia for multilingual agent use `language_map` property. Keys listed in the map must match languages (`language` and `additional_langauges`) mentioned in `config.yml`. Cartesia supports `language` as configuration parameter inside `language_map`.

config.yml

```
recipe: default.v1language: enadditional_languages:  - de-DEpipeline:- name: SingleStepLLMCommandGeneratorpolicies:- name: FlowPolicyassistant_id: 20240418-073244-narrow-archive
```

credentials.yml

```
browser_audio:  tts:    name: cartesia    language_map:      en:        language: en      de-DE:        language: de
```

To see the full list of supported languages and models for Cartesia TTS visit [Cartesia Supported Languages](https://docs.cartesia.ai/build-with-cartesia/tts-models/latest#language-support).

### Deepgram TTS[​](https://rasa.com/docs/reference/integrations/speech-integrations/#deepgram-tts 'Direct link to Deepgram TTS')

Use the environment variable `DEEPGRAM_API_KEY` for Deepgram API Key. You can request a key from [Deepgram](https://deepgram.com/). It can be configured in a Voice Stream channel as follows:

credentials.yml

```
browser_audio:  # ... other configuration  tts:    name: deepgram
```

#### Configuration parameters[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-parameters-4 'Direct link to Configuration parameters')

Deepgram does not use the parent class parameters of `language` or `voice` as each model is uniquely identified using the format `[modelname]-[voicename]-[language]`.

- `model_id` (deprecated, use `language_map`): Optional, defaults to `aura-2-andromeda-en` - The list of available options can be found in [Deepgram Documentation](https://developers.deepgram.com/docs/tts-models).
- `endpoint`: Optional, defaults to `wss://api.deepgram.com/v1/speak` - The endpoint URL for the Deepgram API.
- `timeout`: Optional, defaults to `30` - The timeout duration in seconds for the text-to-speech request.
- `language_map`: Optional, defaults to mapping for english language - multilingual agent's mapping between languages set in `config.yml` and `model` parameter for Deepgram.

Language Configuration

`model_id` and `language_map` are mutually exclusive properties.

#### Deepgram TTS Multilingual Support[​](https://rasa.com/docs/reference/integrations/speech-integrations/#deepgram-tts-multilingual-support 'Direct link to Deepgram TTS Multilingual Support')

info

New in Rasa 3.16.0

To configure Deepgram TTS for multilingual agent use `language_map` property. Keys listed in the map must match languages (`language` and `additional_langauges`) mentioned in `config.yml`. Deepgram TTS supports `model` as configuration parameter inside `language_map`.

config.yml

```
recipe: default.v1language: enadditional_languages:  - de-DEpipeline:- name: SingleStepLLMCommandGeneratorpolicies:- name: FlowPolicyassistant_id: 20240418-073244-narrow-archive
```

credentials.yml

```
browser_audio:  tts:    name: deepgram    language_map:      en:        model: aura-2-andromeda-en      de-DE:        model: aura-2-julius-de
```

To see the full list of supported languages and models for Deepgram TTS visit [Deepgram TTS Languages](https://developers.deepgram.com/docs/tts-models#language-support).

### Rime TTS[​](https://rasa.com/docs/reference/integrations/speech-integrations/#rime-tts 'Direct link to Rime TTS')

Use the environment variable `RIME_API_KEY` for Rime API Key. You can request a key from [Rime](https://rime.ai/). It can be configured in a Voice Stream channel as follows:

credentials.yml

```
browser_audio:  # ... other configuration  tts:    name: rime
```

No Input Text Streaming

Rime TTS does not support input text streaming of generative responses. The complete response text is sent to the TTS service at once, which may result in higher latency compared to TTS services that support streaming.

#### Configuration parameters[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-parameters-5 'Direct link to Configuration parameters')

Rasa uses the Rime [Websockets JSON API](https://docs.rime.ai/api-reference/endpoint/websockets-json),

- `language` (deprecated, use `language_map`): Optional, defaults to `eng` - The language code for the text-to-speech conversion. [See Rime Documentation for details.](https://docs.rime.ai/api-reference/voices)
- `voice` (deprecated, use `language_map`): Optional, defaults to `cove` - The speaker voice to use for text-to-speech conversion. [Please refer to Rime Documentation.](https://docs.rime.ai/api-reference/voices)
- `model_id`: Optional, defaults to `mistv2` - The model ID to be used for text-to-speech conversion.
- `endpoint`: Optional, defaults to `wss://users.rime.ai/ws2` - The endpoint URL for the Rime API.
- `speed_alpha`: Optional, defaults to `1.0` - Controls the speed of speech synthesis.
- `segment`: Optional, defaults to `immediate` - Segment mode for synthesis. Use "immediate" for low latency.
- `timeout`: Optional, defaults to `30` - The timeout duration in seconds for the text-to-speech request.
- `no_text_normalization`: Optional, defaults to `False` - turns off text normalization, to reduce the amount of computation needed to prepare input text for TTS inference. Rime recommends [enabling it to reduce response latency](https://docs.rime.ai/api-reference/latency#recommendations-for-reducing-response-time)
- `language_map`: Optional, defaults to mapping for english language - multilingual agent's mapping between languages set in `config.yml` and `language` and `voice` parameter for Rime TTS.

Language Configuration

`language` and `voice` properties are mutually exclusive with `language_map`.

#### Rime TTS Multilingual Support[​](https://rasa.com/docs/reference/integrations/speech-integrations/#rime-tts-multilingual-support 'Direct link to Rime TTS Multilingual Support')

info

New in Rasa 3.16.0

To configure Rime TTS for multilingual agent use `language_map` property. Keys listed in the map must match languages (`language` and `additional_langauges`) mentioned in `config.yml`. Rime TTS supports `language` and `voice` as configuration parameters inside `language_map`.

config.yml

```
recipe: default.v1language: enadditional_languages:  - de-DEpipeline:- name: SingleStepLLMCommandGeneratorpolicies:- name: FlowPolicyassistant_id: 20240418-073244-narrow-archive
```

credentials.yml

```
browser_audio:  tts:    name: rime    language_map:      en:        language: eng        voice: abbie      de-DE:        language: ger        voice: amalia
```

To see the full list of supported languages and voices for Rime TTS visit [Rime TTS Languages](https://users.rime.ai/data/voices/all-v2.json).

### Others[​](https://rasa.com/docs/reference/integrations/speech-integrations/#others-1 'Direct link to Others')

Looking for integration with a different TTS service? You can [create your own custom TTS component](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-tts).

## Custom ASR[​](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-asr 'Direct link to Custom ASR')

You can implement your own custom ASR component as a Python class to integrate with any third-party speech recognition service. A custom ASR component must subclass the `ASREngine` class from `rasa.core.channels.voice_stream.asr.asr_engine`.

Your custom ASR component will receive audio in the [RasaAudioBytes format](https://rasa.com/docs/reference/integrations/speech-integrations/#audio-format) and may need to convert it to your service's expected format.

### Required Methods[​](https://rasa.com/docs/reference/integrations/speech-integrations/#required-methods 'Direct link to Required Methods')

Your custom ASR component must implement the following methods:

- `open_websocket_connection()`: Establish a websocket connection to your ASR service
- `from_config_dict(config: Dict)`: Class method to create an instance from configuration dictionary
- `signal_audio_done()`: Signal to the ASR service that audio input has ended
- `rasa_audio_bytes_to_engine_bytes(chunk: RasaAudioBytes)`: Convert Rasa audio format to your engine's expected format
- `engine_event_to_asr_event(event: Any)`: Convert your engine's events to Rasa's `ASREvent` format
- `get_default_config()`: Static method that returns the default configuration for your component

### Optional Methods[​](https://rasa.com/docs/reference/integrations/speech-integrations/#optional-methods 'Direct link to Optional Methods')

You may also override these methods as needed:

- `send_keep_alive()`: Send keep-alive messages to maintain the connection. The default implementation is only a `pass` statement.
- `close_connection()`: Custom cleanup when closing the connection. Default implementation is as follows,

```
async def close_connection(self) -> None:  if self.asr_socket:    await self.asr_socket.close()
```

### ASR Events[​](https://rasa.com/docs/reference/integrations/speech-integrations/#asr-events 'Direct link to ASR Events')

Your `engine_event_to_asr_event` method should return appropriate `ASREvent` objects:

- `UserIsSpeaking(transcript)`: For interim/partial transcripts while the user is speaking
- `NewTranscript(transcript)`: For final transcripts when the user has finished speaking

See [Configuration](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-for-custom-components) for details on how to configure your custom ASR component.

### Example Implementation[​](https://rasa.com/docs/reference/integrations/speech-integrations/#example-implementation 'Direct link to Example Implementation')

Here's an example based on the Deepgram implementation structure:

custom\_asr.py

```
import jsonimport osfrom dataclasses import dataclassfrom typing import Any, Dict, Optionalfrom urllib.parse import urlencodeimport websocketsfrom websockets.legacy.client import WebSocketClientProtocolfrom rasa.core.channels.voice_stream.asr.asr_engine import ASREngine, ASREngineConfigfrom rasa.core.channels.voice_stream.asr.asr_event import (    ASREvent,    NewTranscript,    UserIsSpeaking,)from rasa.core.channels.voice_stream.audio_bytes import RasaAudioBytes@dataclassclass MyASRConfig(ASREngineConfig):    api_key: str = ""    endpoint: str = "wss://api.example.com/v1/speech"    language: str = "en-US"class MyASR(ASREngine[MyASRConfig]):    required_env_vars = ("MY_ASR_API_KEY",)  # Optional: required environment variables    required_packages = ("my_asr_package",)  # Optional: required Python packages    def __init__(self, config: Optional[MyASRConfig] = None):        super().__init__(config)        self.accumulated_transcript = ""    async def open_websocket_connection(self) -> WebSocketClientProtocol:        """Connect to the ASR system."""        api_key = os.environ["MY_ASR_API_KEY"]        headers = {"Authorization": f"Bearer {api_key}"}        return await websockets.connect(            self._get_api_url_with_params(),            extra_headers=headers        )    def _get_api_url_with_params(self) -> str:        """Build API URL with query parameters."""        query_params = {            "language": self.config.language,            "encoding": "mulaw",            "sample_rate": "8000",            "interim_results": "true"        }        return f"{self.config.endpoint}?{urlencode(query_params)}"    @classmethod    def from_config_dict(cls, config: Dict) -> "MyASR":        """Create instance from configuration dictionary."""        asr_config = MyASRConfig.from_dict(config)        return cls(asr_config)    async def signal_audio_done(self) -> None:        """Signal to the ASR service that audio input has ended."""        await self.asr_socket.send(json.dumps({"type": "stop_audio"}))    def rasa_audio_bytes_to_engine_bytes(self, chunk: RasaAudioBytes) -> bytes:        """Convert Rasa audio format to engine format."""        # For most services, you can return the chunk directly        # since it's already in mulaw format        return chunk    def engine_event_to_asr_event(self, event: Any) -> Optional[ASREvent]:        """Convert engine response to ASREvent."""        data = json.loads(event)                if data.get("type") == "transcript":            transcript = data.get("text", "")                        if data.get("is_final"):                # Final transcript - user finished speaking                full_transcript = self.accumulated_transcript + " " + transcript                self.accumulated_transcript = ""                return NewTranscript(full_transcript.strip())            elif transcript:                # Interim transcript - user is still speaking                return UserIsSpeaking(transcript)                        return None    @staticmethod    def get_default_config() -> MyASRConfig:        """Get default configuration."""        return MyASRConfig(            endpoint="wss://api.example.com/v1/speech",            language="en-US"        )    async def send_keep_alive(self) -> None:        """Send keep-alive message if supported by your service."""        if self.asr_socket is not None:            await self.asr_socket.send(json.dumps({"type": "keep_alive"}))
```

This structure allows you to integrate any speech recognition service with Rasa's voice capabilities while maintaining compatibility with the existing voice stream infrastructure.

## Custom TTS[​](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-tts 'Direct link to Custom TTS')

You can implement your own custom TTS component as a Python class to integrate with any third-party text-to-speech service. A custom TTS component must subclass the `TTSEngine` class from `rasa.core.channels.voice_stream.tts.tts_engine`.

Your custom TTS component must output audio in the [RasaAudioBytes format](https://rasa.com/docs/reference/integrations/speech-integrations/#audio-format) and convert it using the `engine_bytes_to_rasa_audio_bytes` method.

### Required Methods[​](https://rasa.com/docs/reference/integrations/speech-integrations/#required-methods-1 'Direct link to Required Methods')

Your custom TTS component must implement the following methods:

- `synthesize(text: str, config: Optional[T])`: Generate speech from text, returning an async iterator of `RasaAudioBytes` chunks
- `engine_bytes_to_rasa_audio_bytes(chunk: bytes)`: Convert your engine's audio format to Rasa's audio format
- `from_config_dict(config: Dict)`: Class method to create an instance from configuration dictionary
- `get_default_config()`: Static method that returns the default configuration for your component

### Optional Methods[​](https://rasa.com/docs/reference/integrations/speech-integrations/#optional-methods-1 'Direct link to Optional Methods')

You may also override these methods as needed:

- `connect(config: Optional[T])`: Establish connection to the TTS engine if necessary. Default implementation does nothing.
- `close_connection()`: Custom cleanup when closing connections (e.g., closing websockets or HTTP sessions). Default implementation does nothing.
- `send_text_chunk(text: str)`: Send text chunks to the TTS system for streaming synthesis. Used with `stream_audio()` for real-time streaming.
- `signal_text_done()`: Signal TTS engine to process any remaining buffered text and prepare to end the stream.
- `stream_audio()`: Stream audio output from the TTS engine. Continuously yields audio chunks as they are produced by the engine.

### Streaming vs Non-Streaming TTS[​](https://rasa.com/docs/reference/integrations/speech-integrations/#streaming-vs-non-streaming-tts 'Direct link to Streaming vs Non-Streaming TTS')

The TTSEngine supports both streaming and non-streaming modes:

- **Non-streaming**: Implement only `synthesize()` method for simple request-response synthesis
- **Streaming**: Set `streaming_input = True` and implement `send_text_chunk()`, `signal_text_done()`, and `stream_audio()` methods for real-time streaming synthesis

### Required Class Attributes[​](https://rasa.com/docs/reference/integrations/speech-integrations/#required-class-attributes 'Direct link to Required Class Attributes')

You can optionally define these class attributes:

- `required_env_vars`: Tuple of required environment variable names
- `required_packages`: Tuple of required Python package names
- `streaming_input`: Boolean indicating if the engine supports streaming input (defaults to `False`)

See [Configuration](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-for-custom-components) for details on how to configure your custom TTS component.

### Example Implementation[​](https://rasa.com/docs/reference/integrations/speech-integrations/#example-implementation-1 'Direct link to Example Implementation')

Here's an example of an implementation for TTS streaming:

custom\_tts.py

```
import osfrom dataclasses import dataclassfrom typing import AsyncIterator, Dict, Optionalfrom urllib.parse import urlencodeimport aiohttpfrom aiohttp import ClientTimeout, WSMsgTypefrom rasa.core.channels.voice_stream.audio_bytes import RasaAudioBytesfrom rasa.core.channels.voice_stream.tts.tts_engine import (    TTSEngine,    TTSEngineConfig,    TTSError,)@dataclassclass MyTTSConfig(TTSEngineConfig):    endpoint: str = "wss://api.example.com/v1/speak"    model_id: str = "en-US-standard"class MyTTS(TTSEngine[MyTTSConfig]):    session: Optional[aiohttp.ClientSession] = None    required_env_vars = ("MY_TTS_API_KEY",)  # Optional: required environment variables    required_packages = ("aiohttp",)  # Optional: required Python packages    streaming_input = True    ws: Optional[aiohttp.ClientWebSocketResponse] = None    def __init__(self, config: Optional[MyTTSConfig] = None):        super().__init__(config)        timeout = ClientTimeout(total=self.config.timeout)        # All class instances share the same session        if self.__class__.session is None or self.__class__.session.closed:            self.__class__.session = aiohttp.ClientSession(timeout=timeout)    async def connect(self, config: Optional[MyTTSConfig] = None) -> None:        """Establish WebSocket connection to the TTS engine."""        headers = {            "Authorization": f"Bearer {os.environ['MY_TTS_API_KEY']}",        }                query_params = {            "model": merged_config.model_id,            "language": merged_config.language,            "voice": merged_config.voice,            "encoding": "mulaw",            "sample_rate": "8000",        }                ws_url = f"{merged_config.endpoint}?{urlencode(query_params)}"                try:            self.ws = await self.session.ws_connect(                ws_url,                headers=headers,                timeout=float(self.config.timeout) if self.config.timeout else 30,            )        except Exception as e:            raise TTSError(f"Failed to connect to TTS service: {e}")    async def close_connection(self) -> None:        """Close WebSocket connection if it exists."""        if self.ws and not self.ws.closed:            await self.ws.close()            self.ws = None    async def send_text_chunk(self, text: str) -> None:        """Send text chunk to TTS engine for streaming synthesis."""        await self.ws.send_json({"text": text})    async def signal_text_done(self) -> None:        """Signal TTS engine that all text has been sent."""        await self.ws.send_json({"operation": "flush"})    async def stream_audio(self) -> AsyncIterator[RasaAudioBytes]:        """Stream audio output from the TTS engine."""        try:            async for msg in self.ws:                if msg.type == WSMsgType.TEXT:                    data = msg.json()                    if data.get("type") == "audio":                        # Assume base64 encoded audio data                        import base64                        audio_bytes = base64.b64decode(data.get("data", ""))                        if audio_bytes:                            yield self.engine_bytes_to_rasa_audio_bytes(audio_bytes)                    elif data.get("type") == "error":                        raise TTSError(f"TTS error: {data.get('message')}")                elif msg.type == WSMsgType.CLOSED:                    break                elif msg.type == WSMsgType.ERROR:                    raise TTSError("WebSocket error during audio streaming")        except Exception as e:            raise TTSError(f"Error during audio streaming: {e}")    async def synthesize(        self, text: str, config: Optional[MyTTSConfig] = None    ) -> AsyncIterator[RasaAudioBytes]:        """Generate speech from text using streaming WebSocket."""        await self.connect(config)                try:            await self.send_text_chunk(text)            await self.signal_text_done()                        async for audio_chunk in self.stream_audio():                yield audio_chunk        finally:            await self.close_connection()    def engine_bytes_to_rasa_audio_bytes(self, chunk: bytes) -> RasaAudioBytes:        """Convert the generated TTS audio bytes into Rasa audio bytes."""        # If your service returns audio in mulaw format already, return directly        return RasaAudioBytes(chunk)                # If your service returns audio in a different format (e.g., PCM linear),        # you'll need to convert it. For example, to convert from linear PCM:        # import audioop        # mulaw_data = audioop.lin2ulaw(chunk, 2)  # 2 = 16-bit samples        # return RasaAudioBytes(mulaw_data)    @staticmethod    def get_default_config() -> MyTTSConfig:        return MyTTSConfig(            endpoint="wss://api.example.com/v1/speak",            model_id="en-US-standard",            language="en-US",            voice="female-1",            timeout=30,        )    @classmethod    def from_config_dict(cls, config: Dict) -> "MyTTS":        return cls(MyTTSConfig.from_dict(config))
```

Non-Streaming TTS

If your TTS service doesn't support continuous input streaming (i.e., you need to send all text at once), set `streaming_input = False` and only implement the `synthesize()` method. You can skip implementing `connect()`, `send_text_chunk()`, `signal_text_done()`, and `stream_audio()` methods. The `synthesize()` method should handle the entire synthesis process from text to audio.

## Configuration for Custom Components[​](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-for-custom-components 'Direct link to Configuration for Custom Components')

To use a custom ASR or TTS component, you need to supply credentials for it in your `credentials.yml` file. The configuration should contain the **module path** of your custom class and any required configuration parameters.

The module path follows the format `path.to.module.ClassName`. For example:

- A class `MyASR` in file `addons/custom_asr.py` has module path `addons.custom_asr.MyASR`
- A class `MyTTS` in file `addons/custom_tts.py` has module path `addons.custom_tts.MyTTS`

### Custom ASR Configuration Example[​](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-asr-configuration-example 'Direct link to Custom ASR Configuration Example')

credentials.yml

```
browser_audio:  # ... other configuration  asr:    name: addons.custom_asr.MyASR    api_key: "your_api_key"    endpoint: "wss://api.example.com/v1/speech"    language: "en-US"    # any other custom parameters your ASR needs
```

### Custom TTS Configuration Example[​](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-tts-configuration-example 'Direct link to Custom TTS Configuration Example')

credentials.yml

```
browser_audio:  # ... other configuration  tts:    name: addons.custom_tts.MyTTS    api_key: "your_api_key"    endpoint: "wss://api.example.com/v1/speak"    language: "en-US"    voice: "en-US-JennyNeural"    timeout: 30    # any other custom parameters your TTS needs
```

Any custom parameters you define in your configuration class (e.g., `MyASRConfig` or `MyTTSConfig`) can be passed through the credentials file and will be available in your component via `self.config`.

- [Audio Format](https://rasa.com/docs/reference/integrations/speech-integrations/#audio-format)
- [Automatic Speech Recognition (ASR)](https://rasa.com/docs/reference/integrations/speech-integrations/#automatic-speech-recognition-asr)
 - [Deepgram](https://rasa.com/docs/reference/integrations/speech-integrations/#deepgram)
 - [Azure](https://rasa.com/docs/reference/integrations/speech-integrations/#azure)
 - [Others](https://rasa.com/docs/reference/integrations/speech-integrations/#others)
- [Text To Speech (TTS)](https://rasa.com/docs/reference/integrations/speech-integrations/#text-to-speech-tts)
 - [Azure TTS](https://rasa.com/docs/reference/integrations/speech-integrations/#azure-tts)
 - [Cartesia TTS](https://rasa.com/docs/reference/integrations/speech-integrations/#cartesia-tts)
 - [Deepgram TTS](https://rasa.com/docs/reference/integrations/speech-integrations/#deepgram-tts)
 - [Rime TTS](https://rasa.com/docs/reference/integrations/speech-integrations/#rime-tts)
 - [Others](https://rasa.com/docs/reference/integrations/speech-integrations/#others-1)
- [Custom ASR](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-asr)
 - [Required Methods](https://rasa.com/docs/reference/integrations/speech-integrations/#required-methods)
 - [Optional Methods](https://rasa.com/docs/reference/integrations/speech-integrations/#optional-methods)
 - [ASR Events](https://rasa.com/docs/reference/integrations/speech-integrations/#asr-events)
 - [Example Implementation](https://rasa.com/docs/reference/integrations/speech-integrations/#example-implementation)
- [Custom TTS](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-tts)
 - [Required Methods](https://rasa.com/docs/reference/integrations/speech-integrations/#required-methods-1)
 - [Optional Methods](https://rasa.com/docs/reference/integrations/speech-integrations/#optional-methods-1)
 - [Streaming vs Non-Streaming TTS](https://rasa.com/docs/reference/integrations/speech-integrations/#streaming-vs-non-streaming-tts)
 - [Required Class Attributes](https://rasa.com/docs/reference/integrations/speech-integrations/#required-class-attributes)
 - [Example Implementation](https://rasa.com/docs/reference/integrations/speech-integrations/#example-implementation-1)
- [Configuration for Custom Components](https://rasa.com/docs/reference/integrations/speech-integrations/#configuration-for-custom-components)
 - [Custom ASR Configuration Example](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-asr-configuration-example)
 - [Custom TTS Configuration Example](https://rasa.com/docs/reference/integrations/speech-integrations/#custom-tts-configuration-example)