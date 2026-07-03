# Source: https://rasa.com/docs/reference/python-versions-and-dependencies

On this page

This page provides a comprehensive overview of the Python versions and dependency groups supported by Rasa Pro.

## Supported Python Versions[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#supported-python-versions 'Direct link to Supported Python Versions')

Rasa Pro supports the following Python versions:

| Python Version | Support Status | Notes |
| --- | --- | --- |
| 3.10 | ✅ Supported | Full support |
| 3.11 | ✅ Supported | Full support |
| 3.12 | ✅ Supported | Limited support (see TensorFlow limitations below) |
| 3.13 | ✅ Supported | Limited support (see TensorFlow limitations below) |

TensorFlow Limitations

TensorFlow and related dependencies are only supported for Python < 3.12. This means that components requiring TensorFlow are not available for Python >= 3.12:

- `DIETClassifier`
- `TEDPolicy`
- `UnexpecTEDIntentPolicy`
- `ResponseSelector`
- `ConveRTFeaturizer`
- `LanguageModelFeaturizer`

If you need these components, use Python 3.10 or 3.11.

## Dependency Groups[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#dependency-groups 'Direct link to Dependency Groups')

Rasa Pro uses optional dependency groups to allow you to install only the dependencies you need for your specific use case. This helps reduce installation time and keeps your environment lean.

### Available Dependency Groups[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#available-dependency-groups 'Direct link to Available Dependency Groups')

| Dependency Group | Description | Installation Command |
| --- | --- | --- |
| `nlu` | Dependencies for NLU components | `pip install 'rasa-pro[nlu]'` |
| `channels` | Dependencies for channel connectors | `pip install 'rasa-pro[channels]'` |

### NLU Dependency Group[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#nlu-dependency-group 'Direct link to NLU Dependency Group')

The `nlu` dependency group includes dependencies required for NLU components.

These include:

- `spacy` (`^3.5.4`)
- `skops` (`~0.13.0`)
- `mitie` (`^0.7.36`)
- `jieba` (`>=0.42.1, <0.43`)
- `sklearn-crfsuite` (`~0.5.0`)
- `transformers` (`~4.38.2`)

Plus the following dependencies, which are only available for Python < 3.12:

- `tensorflow` (`^2.19.0`)
- `tensorflow-text` (`^2.19.0`)
- `tensorflow-hub` (`^0.13.0`)
- `tensorflow-metal` (`^1.2.0`)
- `tf-keras` (`^2.15.0`)
- `sentencepiece` (`~0.1.99`)
- `tensorflow-io-gcs-filesystem` (`0.31` for sys\_platform == 'win32', `0.34` for sys\_platform == 'linux', `0.34` for sys\_platform == 'linux' for "sys\_platform == 'darwin' and platform\_machine != 'arm64')

### Channels Dependency Group[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#channels-dependency-group 'Direct link to Channels Dependency Group')

The `channels` dependency group includes dependencies required for channel connectors:

- `fbmessenger` (`~6.0.0`)
- `twilio` (`~9.7.2`)
- `webexteamssdk` (`>=1.6.1,<1.7.0`)
- `mattermostwrapper` (`~2.2`)
- `rocketchat_API` (`>=1.32.0,<1.33.0`)
- `aiogram` (`~3.22.0`)
- `slack-sdk` (`~3.36.0`)
- `cvg-python-sdk` (`^0.5.1`)

Channel Dependencies Not Included

The following channels do not require additional dependencies and are included in the main installation:

- `browser_audio`
- `studio_chat`
- `socketIO`
- `rest`

## Installation Examples[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#installation-examples 'Direct link to Installation Examples')

### Basic Installation[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#basic-installation 'Direct link to Basic Installation')

```
# Install Rasa Pro with default dependenciespip install rasa-pro
```

### With NLU Components[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#with-nlu-components 'Direct link to With NLU Components')

```
# Install Rasa Pro with NLU dependenciespip install 'rasa-pro[nlu]'
```

### With Channel Connectors[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#with-channel-connectors 'Direct link to With Channel Connectors')

```
# Install Rasa Pro with channel dependenciespip install 'rasa-pro[channels]'
```

### With Both NLU and Channels[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#with-both-nlu-and-channels 'Direct link to With Both NLU and Channels')

```
# Install Rasa Pro with both NLU and channel dependenciespip install 'rasa-pro[nlu,channels]'
```

### Using uv[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#using-uv 'Direct link to Using uv')

```
# Using uv package manager (recommended for faster installation)uv pip install 'rasa-pro[nlu,channels]'
```

## When to Use Which Dependency Group[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#when-to-use-which-dependency-group 'Direct link to When to Use Which Dependency Group')

**Use the `nlu` group when:**

- You are using NLU components in your pipeline (see [NLU Components](https://rasa.com/docs/reference/config/components/nlu-components/) for a full list of all NLU components)
- You need intent classification or entity extraction capabilities
- You're using components like `DIETClassifier`, `TEDPolicy`, or `ResponseSelector`
- You're building assistants that require traditional NLU capabilities
- You need to process user input for intent recognition and entity extraction

**Use the `channels` group when:**

- You're connecting to external messaging platforms (Slack, Telegram, Facebook Messenger, etc.)
- You need voice channel connectors (Jambonz, Audiocodes, Twilio Voice, etc.)
- You're deploying to platforms like Microsoft Bot Framework, Cisco Webex Teams, or RocketChat
- You're building voice assistants that require real-time audio processing
- You need to integrate with customer support platforms or communication systems

### Use both groups when:[​](https://rasa.com/docs/reference/python-versions-and-dependencies/#use-both-groups-when 'Direct link to Use both groups when:')

- You're building a comprehensive assistant that uses both NLU components and external channel integrations
- You need the full feature set of Rasa Pro with traditional NLU capabilities and multi-platform deployment

- [Supported Python Versions](https://rasa.com/docs/reference/python-versions-and-dependencies/#supported-python-versions)
- [Dependency Groups](https://rasa.com/docs/reference/python-versions-and-dependencies/#dependency-groups)
 - [Available Dependency Groups](https://rasa.com/docs/reference/python-versions-and-dependencies/#available-dependency-groups)
 - [NLU Dependency Group](https://rasa.com/docs/reference/python-versions-and-dependencies/#nlu-dependency-group)
 - [Channels Dependency Group](https://rasa.com/docs/reference/python-versions-and-dependencies/#channels-dependency-group)
- [Installation Examples](https://rasa.com/docs/reference/python-versions-and-dependencies/#installation-examples)
 - [Basic Installation](https://rasa.com/docs/reference/python-versions-and-dependencies/#basic-installation)
 - [With NLU Components](https://rasa.com/docs/reference/python-versions-and-dependencies/#with-nlu-components)
 - [With Channel Connectors](https://rasa.com/docs/reference/python-versions-and-dependencies/#with-channel-connectors)
 - [With Both NLU and Channels](https://rasa.com/docs/reference/python-versions-and-dependencies/#with-both-nlu-and-channels)
 - [Using uv](https://rasa.com/docs/reference/python-versions-and-dependencies/#using-uv)
- [When to Use Which Dependency Group](https://rasa.com/docs/reference/python-versions-and-dependencies/#when-to-use-which-dependency-group)
 - [Use both groups when:](https://rasa.com/docs/reference/python-versions-and-dependencies/#use-both-groups-when)