# Source: https://rasa.com/docs/rasa-pro/command-line-interface

On this page

## Cheat Sheet[​](https://rasa.com/docs/reference/api/command-line-interface/#cheat-sheet 'Direct link to Cheat Sheet')

### Pro specific[​](https://rasa.com/docs/reference/api/command-line-interface/#pro-specific 'Direct link to Pro specific')

The following commands are relevant to all assistants built with Rasa.

| Command | Effect |
| --- | --- |
| `rasa init` | Creates a new project with example training data, actions, and config files. |
| `rasa train` | Trains a model using your NLU data and stories, saves trained model in `./models`. |
| `rasa shell` | Loads your trained model and lets you talk to your assistant on the command line. |
| `rasa run` | Starts a server with your trained model. |
| `rasa run actions` | Starts an action server using the Rasa SDK. |
| `rasa test e2e` | Runs end-to-end testing fully integrated with the action server that serves as acceptance testing. |
| `rasa data convert e2e` | Converts sample conversation data into end-to-end test cases. |
| `rasa llm finetune prepare-data` | Prepares data to fine-tune a base model for the task of command generator. |
| `rasa inspect` | Opens Rasa Inspector. |
| `rasa data validate` | Checks the domain, NLU, flows and conversation data for inconsistencies. |
| `rasa export` | Exports conversations from a tracker store to an event broker. |
| `rasa marker upload` | Upload marker configurations to Analytics Data Pipeline |
| `rasa license` | Display licensing information. |
| `rasa -h` | Shows all available commands. |
| `rasa --version` | Shows version information about Rasa, Python and the expiration date for Rasa License |

### Studio specific[​](https://rasa.com/docs/reference/api/command-line-interface/#studio-specific 'Direct link to Studio specific')

If your team is using **Rasa Studio**, these CLI commands let you manage and sync your assistant between your local environment and your Studio deployment:

| Command | Description |
| --- | --- |
| `rasa studio config` | Sets up local configuration to connect to your Rasa Studio deployment. |
| `rasa studio login` | Authenticates and retrieves credentials from your Rasa Studio instance. |
| `rasa studio upload` | Uploads your local assistant as a new project in Rasa Studio. |
| `rasa studio download <assistant-name>` | Downloads the full assistant from Rasa Studio into your current directory. |
| `rasa studio link <assistant-name>` | Links your local project to a specific assistant in Rasa Studio. |
| `rasa studio push` | Pushes your local changes to Studio. |
| `rasa studio pull` | Pulls the latest changes from Studio and merges them into your local project. |
| `rasa studio train` | Trains a model combining local + Studio data and saves trained model in `./models`. |

## Rasa Pro Commands[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-pro-commands 'Direct link to Rasa Pro Commands')

### Logging[​](https://rasa.com/docs/reference/api/command-line-interface/#logging 'Direct link to Logging')

note

If you run into character encoding issues on Windows like: `UnicodeEncodeError: 'charmap' codec can't encode character ...` or the terminal is not displaying colored messages properly, prepend `winpty` to the command you would like to run. For example `winpty rasa init` instead of `rasa init`

#### Setting log levels[​](https://rasa.com/docs/reference/api/command-line-interface/#setting-log-levels 'Direct link to Setting log levels')

Rasa produces log messages at several different levels (eg. warning, info, error and so on). You can control which level of logs you would like to see with `--verbose` (same as `-v`) or `--debug` (same as `-vv`) as optional command line arguments. See each command below for more explanation on what these arguments mean.

In addition to CLI arguments, several environment variables allow you to control log output in a more granular way. With these environment variables, you can configure log levels for messages created by external libraries such as Matplotlib, Pika, and Kafka. These variables follow [standard logging level in Python](https://docs.python.org/3/library/logging.html#logging-levels). Currently, following environment variables are supported:

1. LOG\_LEVEL\_LIBRARIES: This is the general environment variable to configure log level for the main libraries Rasa uses. It covers Tensorflow, `asyncio`, APScheduler, SocketIO, Matplotlib, RabbitMQ, Kafka.
2. LOG\_LEVEL\_MATPLOTLIB: This is the specialized environment variable to configure log level only for Matplotlib.
3. LOG\_LEVEL\_RABBITMQ: This is the specialized environment variable to configure log level only for AMQP libraries, at the moment it handles log levels from `aio_pika` and `aiormq`.
4. LOG\_LEVEL\_KAFKA: This is the specialized environment variable to configure log level only for kafka.
5. LOG\_LEVEL\_PRESIDIO: This is the specialized environment variable to configure log level only for Presidio, at the moment it handles log levels from `presidio_analyzer` and `presidio_anonymizer`.
6. LOG\_LEVEL\_FAKER: This is the specialized environment variable to configure log level only for Faker.
7. LOG\_LEVEL\_MLFLOW: This is the specialized environment variable to configure log level only for MLFlow.
8. LOG\_LEVEL\_PYMONGO: This is the specialized environment variable to configure log level only for PyMongo.

General configuration (`LOG_LEVEL_LIBRARIES`) has less priority than library level specific configuration (`LOG_LEVEL_MATPLOTLIB`, `LOG_LEVEL_RABBITMQ` etc); and CLI parameter sets the lowest level log messages which will be handled. This means variables can be used together with a predictable result. As an example:

```
LOG_LEVEL_LIBRARIES=ERROR LOG_LEVEL_MATPLOTLIB=WARNING LOG_LEVEL_KAFKA=DEBUG rasa shell --debug
```

The above command run will result in showing:

- messages with `DEBUG` level and higher by default (due to `--debug`)
- messages with `WARNING` level and higher for Matplotlib
- messages with `DEBUG` level and higher for kafka
- messages with `ERROR` level and higher for other libraries not configured

Note that CLI config sets the lowest level log messages to be handled, hence the following command will set the log level to `INFO` (due to `--verbose`) and no debug messages will be seen (library level configuration will not have any effect):

```
LOG_LEVEL_LIBRARIES=DEBUG LOG_LEVEL_MATPLOTLIB=DEBUG rasa shell --verbose
```

As an aside, CLI log level sets the level at the root logger (which has the important handler - `coloredlogs` handler); this means even if an environment variable sets a library logger to a lower level, the root logger will reject messages from that library. If not specified, the CLI log level is set to `INFO`.

#### Log Level LLM Components[​](https://rasa.com/docs/reference/api/command-line-interface/#log-level-llm-components 'Direct link to Log Level LLM Components')

Rasa provides enhanced control over the debugging process of LLM-driven components via a fine-grained, customizable logging specified through environment variables.

For example, set the `LOG_LEVEL_LLM` environment variable to enable detailed logging at the desired level for all the LLM components or specify the component you are debugging by setting for example the `LOG_LEVEL_LLM_ENTERPRISE_SEARCH` environment variable:

```
export LOG_LEVEL_LLM=INFOexport LOG_LEVEL_LLM_COMMAND_GENERATOR=INFOexport LOG_LEVEL_LLM_ENTERPRISE_SEARCH=DEBUGexport LOG_LEVEL_LLM_INTENTLESS_POLICY=INFOexport LOG_LEVEL_LLM_REPHRASER=INFOexport LOG_LEVEL_NLU_COMMAND_ADAPTER=INFOexport LOG_LEVEL_LLM_BASED_ROUTER=INFO
```

These settings override logging level for the specified components.

The `LOG_LEVEL_LLM_COMMAND_GENERATOR` variable applies to all types of [LLM-based command generators](https://rasa.com/docs/reference/config/components/llm-command-generators/).

### Custom logging configuration[​](https://rasa.com/docs/reference/api/command-line-interface/#custom-logging-configuration 'Direct link to Custom logging configuration')

v3.4

info

The Rasa CLI now includes a new argument `--logging-config-file` which accepts a YAML file as value.

You can now configure any logging formatters or handlers in a separate YAML file. The logging config YAML file must follow the [Python built-in dictionary schema](https://docs.python.org/3/library/logging.config.html#dictionary-schema-details), otherwise it will fail validation. You can pass this file as argument to the `--logging-config-file` CLI option and use it with any of the rasa commands.

#### Custom logging configuration example[​](https://rasa.com/docs/reference/api/command-line-interface/#custom-logging-configuration-example 'Direct link to Custom logging configuration example')

The following example illustrates how to customize the logging configuration using a YAML file. Here we define a custom formatter, a stream handler for the root logger and a file handler for the rasa logger.

```
version: 1disable_existing_loggers: falseformatters:    customFormatter:        format: "{\"time\": \"%(asctime)s\", \"name\": \"[%(name)s]\", \"levelname\": \"%(levelname)s\", \"message\": \"%(message)s\"}"handlers:  console:    class: logging.StreamHandler    level: INFO    formatter: customFormatter    stream: ext://sys.stdout  file:    class: logging.FileHandler    filename: "rasa_debug.log"    level: DEBUG    formatter: customFormatterloggers:  root:    handlers: [console]  rasa:    handlers: [file]    propagate: 0
```

info

In Rasa Pro 3.9, running `rasa shell` or `rasa interactive` in debug mode could result in `BlockingIOError` when using the default logging configuration. This issue is resolved by using a custom logging configuration file. If you encounter this issue, you can use the above example to create a custom logging configuration file and pass it to the `--logging-config-file` argument.

### rasa init[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-init 'Direct link to rasa init')

This command sets up a complete assistant for you with some example training data:

```
rasa init
```

With no arguments, `rasa init` creates the following files:

```
.├── actions│   ├── __init__.py│   └── actions.py├── config.yml├── credentials.yml├── data│   ├── nlu.yml│   └── stories.yml├── domain.yml├── endpoints.yml├── models│   └── <timestamp>.tar.gz└── tests   └── test_stories.yml
```

It will ask you if you want to train an initial model using this data. If you answer no, the `models` directory will be empty.

This is the best way to get started writing an NLU assistant. You can run `rasa train`, `rasa shell` and `rasa test` without any additional configuration.

Rasa supplies two other templates in addition to the default NLU template described above. Both of these are great ways to get started building your own CALM bots:

- `rasa init --template calm` generates a CALM assistant with [flows](https://rasa.com/docs/reference/primitives/flows/) and a [custom action](https://rasa.com/docs/reference/primitives/custom-actions/) to manage a simple contact list.
- `rasa init --template tutorial` generates the codebase used in the [CALM Tutorial](https://rasa.com/docs/pro/tutorial/).

### rasa train[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-train 'Direct link to rasa train')

The following command trains a Rasa model:

```
rasa train
```

If you have existing models in your directory (under `models/` by default), only the parts of your model that have changed will be re-trained. For example, if you edit your NLU training data and nothing else, only the NLU part will be trained.

If you want to train an NLU or dialogue model individually, you can run `rasa train nlu` or `rasa train core`. If you provide training data only for one one of these, `rasa train` will fall back to one of these commands by default.

`rasa train` will store the trained model in the directory defined by `--out`, `models/` by default. The name of the model by default is `<timestamp>.tar.gz`. If you want to name your model differently, you can specify the name using the `--fixed-model-name` flag.

By default validation is run before training the model. If you want to skip validation, you can use the `--skip-validation` flag. If you want to fail on validation warnings, you can use the `--fail-on-validation-warnings` flag. The `--validation-max-history` is analogous to the `--max-history` argument of `rasa data validate`.

Run `rasa train --help` to see the full list of arguments.

### rasa shell[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-shell 'Direct link to rasa shell')

You can start a chat session by running:

```
rasa shell
```

By default, this will load up the latest trained model. You can specify a different model to be loaded by using the `--model` flag.

If you start the shell with an NLU-only model, `rasa shell` will output the intents and entities predicted for any message you enter.

If you have trained a combined Rasa model but only want to see what your model extracts as intents and entities from text, you can use the command `rasa shell nlu`.

To increase the logging level for debugging, run:

```
rasa shell --debug
```

note

In order to see the typical greetings and/or session start behavior you might see in an external channel, you will need to explicitly send `/session_start` as the first message. Otherwise, the session start behavior will begin as described in [Session configuration](https://rasa.com/docs/reference/config/domain/#session-configuration).

The following arguments can be used to configure the command. Most arguments overlap with `rasa run`; see the [following section](https://rasa.com/docs/reference/api/command-line-interface/#rasa-run) for more info on those arguments.

Note that the `--connector` argument will always be set to `cmdline` when running `rasa shell`. This means all credentials in your credentials file will be ignored, and if you provide your own value for the `--connector` argument it will also be ignored.

Run `rasa shell --help` to see the full list of arguments.

### rasa run[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-run 'Direct link to rasa run')

To start a server running your trained model, run:

```
rasa run
```

By default the Rasa server uses HTTP for its communication. To secure the communication with SSL and run the server on HTTPS, you need to provide a valid certificate and the corresponding private key file. You can specify these files as part of the `rasa run` command. If you encrypted your keyfile with a password during creation, you need to add the `--ssl-password` as well.

```
rasa run --ssl-certificate myssl.crt --ssl-keyfile myssl.key --ssl-password mypassword
```

Rasa by default listens on each available network interface. You can limit this to a specific network interface using the `-i` command line option.

```
rasa run -i 192.168.69.150
```

Rasa will by default connect to all channels specified in your credentials file. To connect to a single channel and ignore all other channels in your credentials file, specify the name of the channel in the `--connector` argument.

```
rasa run --connector rest
```

The name of the channel should match the name you specify in your credentials file. For supported channels see [the page about messaging and voice channels](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/).

Run `rasa run --help` to see the full list of arguments.

For more information on important additional parameters, see [Model Storage](https://rasa.com/docs/reference/integrations/model-storage/)

See the Rasa [REST API](https://rasa.com/docs/reference/api/pro/rasa-pro-rest-api/) page for detailed documentation of all the endpoints.

### rasa run actions[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-run-actions 'Direct link to rasa run actions')

To start an action server with the Rasa SDK, run:

```
rasa run actions
```

Run `rasa run actions --help` to see the full list of arguments.

### rasa visualize[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-visualize 'Direct link to rasa visualize')

To generate a graph of your stories in the browser, run:

```
rasa visualize
```

If your stories are located somewhere other than the default location `data/`, you can specify their location with the `--stories` flag.

Run `rasa visualize --help` to see the full list of arguments.

### rasa test e2e[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-test-e2e 'Direct link to rasa test e2e')

v3.5

info

You can now use end-to-end testing to test your assistant as a whole, including dialogue management and custom actions.

To run [end-to-end testing](https://rasa.com/docs/pro/testing/evaluating-assistant/) on your trained model, run:

```
rasa test e2e
```

This will test your latest trained model on any end-to-end test cases you have. If you want to use a different model, you can specify it using the `--model` flag.

info

By adding the `--coverage-report` flag you obtain a report describing how well your end-to-end tests cover the assistant's flows in terms of share of steps tested per flow. The report includes a histogram of tested commands and allows you to specify the output path with the `--coverage-output-path` flag.

This feature is currently released in a beta version. The feature might change in the future. If you want to enable this beta feature, set the environment variable `RASA_PRO_BETA_FINE_TUNING_RECIPE=true`.

New in 3.15.0

- You can now specify a custom output path for the end-to-end test results using the `-o` or `--e2e-results` flag.
- You can also now export failed end-to-end tests using the `-f` or `--e2e-failed-tests` flag. This also accepts an optional output path for the failed tests file. This file can then be used to re-run only the failed tests by passing it as an argument to the `rasa test e2e` command.

Here are some of the arguments available:

- positional argument for the path to the test cases file or directory containing the test cases: `rasa test e2e <path>` If unspecified, the default path is `tests/e2e_test_cases.yml`.
- optional argument for the trained model: `-model <path>`
- optional argument for retrieving the trained model from [**remote storage**](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-cloud): `-remote-storage <remote-storage-location>`
- optional argument for the `endpoints.yml` file: `-endpoints <path>`
- optional argument for stopping the test run at first failure: `rasa test e2e --fail-fast`
- optional argument for exporting the test results to `e2e_results.yml` file: `rasa test e2e -o` or `rasa test e2e -o <output-path>`. When you specify the `-o` flag without an output path, the passed and failed test results yml files will be saved to the `tests/` subdirectory in the current working directory.
- optional argument for exporting failed tests to `e2e_failed_tests_{timestamp}.yml` file: `rasa test e2e -f` or `rasa test e2e -f <output-path>`. When you specify the `-f` flag without an output path, the failed test file will be saved to the `tests/` subdirectory in the current working directory.
- optional argument for creating a coverage report: `rasa test e2e --coverage-report`
- optional argument for specifying the output directory for the coverage report: `rasa test e2e --coverage-output-path`
- you can optionally dump results to `stdout` or to a file (`o`), plus coverage details with `--coverage-report`.

Run `rasa test e2e --help` to see the full list of arguments.

### rasa llm finetune prepare-data[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-llm-finetune-prepare-data 'Direct link to rasa llm finetune prepare-data')

v3.10

info

This command is part of the [fine-tuning recipe](https://rasa.com/docs/pro/customize/fine-tuning-llm/) available starting with version `3.10.0`. As this feature is a **beta** feature, please set the environment variable `RASA_PRO_BETA_FINETUNING_RECIPE` to `true` to enable it.

This command creates a dataset of prompt to commands pairs from E2E tests that can be used to [fine-tune a base model](https://rasa.com/docs/pro/customize/fine-tuning-llm/) for the task of command generation. To execute the command run

```
rasa llm finetune prepare-data <path-to-e2e-test-cases>
```

Here are some of the arguments available:

```
positional arguments:  path-to-e2e-test-cases                        Input file or folder containing end-to-end test cases. (default: e2e_tests)options:  -o OUT, --out OUT     The output folder to store the data to. (default: output)  -m MODEL, --model MODEL                        Path to a trained Rasa model. If a directory is specified, it will use the latest model in this directory. (default: models)Rephrasing Module:  --num-rephrases  {0, ..., 49}                        Number of rephrases to be generated per user utterance. (default: 10)  --rephrase-config REPHRASE_CONFIG                        Path to config file that contains the configuration of the rephrasing module. (default: None)Train/Test Split Module:  --train-frac TRAIN_FRAC                        The amount of data that should go into the training dataset. The value should be >0.0 and <=1.0. (default: 0.8)  --output-format [{instruction,conversational}]                        Format of the output file. (default: instruction)
```

Run `rasa finetune prepare-data --help` to see all available arguments.

### Resulting file structure[​](https://rasa.com/docs/reference/api/command-line-interface/#resulting-file-structure 'Direct link to Resulting file structure')

```
output/├── 1_command_annotations/          # conversations extracted from your E2E tests├── 2_rephrasings/                  # same conversations + generated rephrasings├── 3_llm_finetune_data/│   └── llm_ft_data.jsonl           # single JSONL file consumed by the fine-tuner├── 4_train_test_split/│   ├── e2e_tests/                  # subsets of the original tests│   │   ├── train.yaml              # test cases that fall into the training split│   │   └── validation.yaml         # test cases that fall into the validation split│   └── ft_splits/│       ├── train.jsonl             # training data for the LLM│       └── test.jsonl              # held-out evaluation data├── params.yaml                     # run parameters└── result_summary.yaml             # short report of what was generated
```

### rasa inspect[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-inspect 'Direct link to rasa inspect')

v3.7

info

This command is part of Rasa's new Conversational AI with Language Models (CALM) approach and available starting with version `3.7.0`.

Opens the [Rasa Inspector](https://rasa.com/docs/pro/testing/trying-assistant/), a debugging tool that offers developers an in-depth look into the conversational mechanics of their Rasa assistant.

Run `rasa inspect --help` to see the full list of arguments.

## rasa inspect --nextgen[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-inspect---nextgen 'Direct link to rasa inspect --nextgen')

v3.16

Opens a preview of the nextgen Rasa Inspector, which will include both the legacy inspector features from Pro, Studio and additional features for better debugging and development. This will replace the old Rasa Inspector in 3.17.

### rasa data validate[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-data-validate 'Direct link to rasa data validate')

You can check your domain, NLU data, flows or story data for mistakes and inconsistencies. To validate your data, run this command:

```
rasa data validate
```

The validator searches for errors in the data, e.g. two intents that have some identical training examples. The validator also checks if you have any stories where different assistant actions follow from the same dialogue history. Conflicts between stories will prevent a model from learning the correct pattern for a dialogue. To learn more about the checks performed by the validator on flows, continue reading in the [next section](https://rasa.com/docs/reference/api/command-line-interface/#validate-flows).

Searching for the `assistant_id` key introduced in 3.5

The validator will check whether the `assistant_id` key is present in the config file and will issue a warning if this key is missing or if the default value has not been changed.

If you pass a `max_history` value to one or more policies in your `config.yml` file, provide the smallest of those values in the validator command using the `--max-history <max_history>` flag.

#### Validate flows[​](https://rasa.com/docs/reference/api/command-line-interface/#validate-flows 'Direct link to Validate flows')

The validator will perform the following checks on flows:

- determine whether flow names or descriptions are unique after stripping punctuation
- verify whether logical expressions in [conditions](https://rasa.com/docs/reference/primitives/conditions/) or `collect` step [rejections](https://rasa.com/docs/reference/primitives/flow-steps/#slot-validation) are valid `pypred` expressions
- determine whether slots used in flows are defined in the domain
- disallow `list` slots from being used in flows `collect` steps: CALM supports only filling slots with values of type `int`, `string` or `bool` in flows.
- disallow `dialogue_stack` internal slot from being used in flows
- ensure that `bool` and `categorical` slots are validated against acceptable values in conditions

For every failure, the validator will log an error and exit the command with exit code 1.

You can validate flows only by running this command:

```
rasa data validate flows
```

### rasa export[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-export 'Direct link to rasa export')

To export events from a tracker store using an event broker, run:

```
rasa export
```

You can specify the location of the environments file, the minimum and maximum timestamps of events that should be published, as well as the conversation IDs that should be published. Run `rasa export --help` to see the full list of arguments.

### rasa license[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-license 'Direct link to rasa license')

v3.3

Use `rasa license` to display information about licensing in Rasa, especially information about 3rd party dependencies licenses.

Run `rasa license --help` to see the full list of arguments.

## Rasa Studio Commands[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-commands 'Direct link to Rasa Studio Commands')

The CLI commands for Rasa Studio enable you to manage updates between your local project and changes made by your team in Studio.

1. Connect to a Studio Deployment: `rasa studio config`
2. Login and authenticate: `rasa studio login`
3. Upload or Download a full project: `rasa studio upload/download`
4. Link a specific assistant project: `rasa studio link <assistant-project-name>`
5. Push and pull updates between Studio and your local project: `rasa studio push/pull`

![d2][base64-image]

### rasa studio config[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-config 'Direct link to rasa studio config')

v3.7

info

This command is available from Rasa Pro 3.7.0 and requires Rasa Studio

This command prompts for parameters of Rasa Studio installation and configures rasa to target that Rasa Studio instance when executing `rasa studio` commands. Configuration is saved to: `$HOME/.config/rasa/global.yml`

The command will use default arguments for the configuration of the authentication server (realm name, client id and authentication url). If you want to use a different configuration, you can specify the parameters by running the command with `rasa studio config --advanced`.

The command will overwrite the existing configuration file with the new configuration.

Example:

```
rasa studio config
```

The command will use SSL strict verification by default to verify the connection to the Rasa Studio authentication server. If you want to skip the strict verification of this connection, you can use the `--disable-verify` or `-x` flag:

```
rasa studio config --disable-verify
```

Run `rasa studio config --help` to see the full list of arguments.

### rasa studio login[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-login 'Direct link to rasa studio login')

v3.7

This command is used to retrieve the access token from Rasa Studio. All other studio commands use this token to authenticate with Rasa Studio. The token is saved to: `$HOME/.config/rasa/studio_token.yaml`

Example:

```
rasa studio login --username my_user_name --password my_password
```

Run `rasa studio login --help` to see the full list of arguments.

### rasa studio upload[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-upload 'Direct link to rasa studio upload')

v3.13

new in 3.13

You can now upload and download a full project using the Rasa CLI as well as link a Studio project to a local project for easier syncing.

Uploads an assistant from local files to Rasa Studio.

#### Import of NLU-based assistants[​](https://rasa.com/docs/reference/api/command-line-interface/#import-of-nlu-based-assistants 'Direct link to Import of NLU-based assistants')

For NLU-based assistants, it will upload the intent and entity definitions to Rasa Studio to an existing assistant in Rasa Studio. When arguments for specifying which intents or entities to upload are not given, all intents and entities get uploaded. When uploading an intent, all entities used in annotations of that intent's utterance examples are uploaded as well.

tip

At the moment, only some intents and entities can be uploaded to Studio. The following can't be uploaded:

- Retrieval intents
- Entities that have `entity_group`
- Intents with `use_entities` and `ignore_entities`
- Entities with `influence_conversation`

Example:

```
rasa studio upload
```

Run `rasa studio upload --help` to see the full list of arguments.

##### Overwriting an existing assistant[​](https://rasa.com/docs/reference/api/command-line-interface/#overwriting-an-existing-assistant 'Direct link to Overwriting an existing assistant')

new in 3.16

You can now delete an existing assistant automatically before uploading by using the `--dangerously-delete-existing` flag.

By default, if an assistant with the same name already exists in Studio, the CLI will ask whether you want to link your local project to that existing assistant. To skip this prompt and automatically delete the existing assistant before uploading, use the `--dangerously-delete-existing` flag:

```
rasa studio upload --dangerously-delete-existing
```

This deletes the existing assistant and all its data without any confirmation prompt, then proceeds with the upload. It is intended for CI/CD workflows where the assistant must be replaced on every run. Use with caution.

##### Possible errors[​](https://rasa.com/docs/reference/api/command-line-interface/#possible-errors 'Direct link to Possible errors')

###### Assistant name errors[​](https://rasa.com/docs/reference/api/command-line-interface/#assistant-name-errors 'Direct link to Assistant name errors')

These include the following:

- `Assistant with name <assistant_name> already exists`
- `<assistant_name> is not a valid name`

A valid assistant name will not exceed the length of 128 characters and will not contain spaces.

###### Invalid YAML errors[​](https://rasa.com/docs/reference/api/command-line-interface/#invalid-yaml-errors 'Direct link to Invalid YAML errors')

If something is wrong with the YAML files structure, a specific error will be logged. You will see these errors when, for example, a required field is missing for an action, slot, response, config or flow.

Examples:

```
Invalid domain: responses.utter_greeting.0.text: RequiredInvalid flows: flows.transfer_money.description: Required
```

###### Reference errors[​](https://rasa.com/docs/reference/api/command-line-interface/#reference-errors 'Direct link to Reference errors')

If a flow references a response, slot, action or another flow (with a `link` step), the following errors will be logged:

```
Can't find <response/slot/action> utter_ask_add_contact_handle in domainCan't find flow <flow_name> in flows
```

###### Unsupported feature errors[​](https://rasa.com/docs/reference/api/command-line-interface/#unsupported-feature-errors 'Direct link to Unsupported feature errors')

Not all the features available in Rasa Pro are supported by Rasa Studio. Trying to import an assistant with unsupported features will result in an error. To find out which versions of Studio support the version of Rasa Pro you are using, check the [compatibility matrix](https://rasa.com/docs/reference/changelogs/compatibility-matrix/).

Examples:

```
Flows with cycles are not supported, flow: <flow_name>Comparing two slots is not supported. Condition: slots.recurrent_payment_end_date < slots.recurrent_payment_start_dateHaving multiple rejections on one slot is not supported, collect: <slot_name>
```

###### Authentication errors[​](https://rasa.com/docs/reference/api/command-line-interface/#authentication-errors 'Direct link to Authentication errors')

User needs to be logged into Rasa Studio before uploading. Use the [`rasa studio login`](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-login) command.

### rasa studio download[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-download 'Direct link to rasa studio download')

v3.13

This command downloads a specified assistant project from Rasa Studio and creates a folder using the assistant name.

The following data is supported:

- configuration
- endpoints
- custom prompts
- flows (for CALM assistants)
- responses
- slots
- custom action declarations
- intents
- entities

Example:

```
rasa studio download my_awesome_assistant
```

Creates a folder at `./my_awesome_assistant`

Run `rasa studio download --help` to see the full list of arguments.

### rasa studio link[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-link 'Direct link to rasa studio link')

v3.13

Links your local assistant to a project in Rasa Studio. You can specify the assistant name as an argument:

```
rasa studio link my_assistant_name
```

Once linked, all subsequent commands (like `download`, `upload`, `pull`, and `push`) will refer to this assistant.

### rasa studio pull[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-pull 'Direct link to rasa studio pull')

v3.13

Pulls the latest changes from your Rasa Studio assistant into your local project.

You can either pull the entire assistant:

```
rasa studio pull
```

Or pull a specific section (e.g., just the configuration or endpoints):

```
rasa studio pull config
```

Current supported sections include: `config`, `endpoints`.

### rasa studio push[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-push 'Direct link to rasa studio push')

v3.13

Pushes the latest changes from your local project to your Rasa Studio assistant.

You can either push everything:

```
rasa studio upload
```

Or push a specific section:

```
rasa studio push config
```

Supported sections include: `config`, `endpoints`.

## Legacy commands[​](https://rasa.com/docs/reference/api/command-line-interface/#legacy-commands 'Direct link to Legacy commands')

### rasa studio upload[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-upload-1 'Direct link to rasa studio upload')

v3.7

Uploads an assistant from local files to Rasa Studio.

#### Import of NLU-based assistants[​](https://rasa.com/docs/reference/api/command-line-interface/#import-of-nlu-based-assistants-1 'Direct link to Import of NLU-based assistants')

For NLU-based assistants, it will upload the intent and entity definitions to Rasa Studio to an existing assistant in Rasa Studio. When arguments for specifying which intents or entities to upload are not given, all intents and entities get uploaded. When uploading an intent, all entities used in annotations of that intent's utterance examples are uploaded as well.

tip

At the moment, only some intents and entities can be uploaded to Studio. The following can't be uploaded:

- Retrieval intents
- Entities that have `entity_group`
- Intents with `use_entities` and `ignore_entities`
- Entities with `influence_conversation`

Example:

```
rasa studio upload
```

Run `rasa studio upload --help` to see the full list of arguments.

#### Import of CALM assistants[​](https://rasa.com/docs/reference/api/command-line-interface/#import-of-calm-assistants 'Direct link to Import of CALM assistants')

To upload a CALM assistant to Rasa Studio, run this command with `--calm` flag.

Important!

- When uploading a CALM assistant, a new Rasa Studio assistant with specified name will be created. This is different from the NLU-based assistant upload, which will reuse an existing Rasa Studio assistant.
 
- During CALM upload, we also upload `config` and `endpoints` that can be edited in the UI.
 

Example:

```
rasa studio upload --calm
```

### rasa studio download[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-download-1 'Direct link to rasa studio download')

v3.7

This command downloads the data from Rasa Studio and saves it to files inside `data` folder. If local files use a single domain file, it is updated accordingly. If there is a domain folder instead, domain changes are written to `<domain_folder>/studio_domain.yml`.

The command downloads Studio data that is available in Studio but not in local files. The following data is supported:

- configuration
- endpoints
- custom prompts
- flows (for CALM assistants)
- responses
- slots
- custom action declarations
- intents
- entities

The `--overwrite` flag can be used to overwrite the existing data in the existing files when a primitive has the same ID as the one downloaded from Rasa Studio. Special cases:

- If an intent exists in local files, but Studio has examples missing locally, they will be downloaded.
- If local `config` and `endpoints` files exist during the download of a CALM assistant, the user needs to confirm their intent to overwrite them, even when the `--overwrite` flag is provided.

Example:

```
rasa studio download my_awesome_assistant -d my_domain_folder
```

Run `rasa studio download --help` to see the full list of arguments.

### rasa studio train[​](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-train 'Direct link to rasa studio train')

v3.7

This command is analogous to `rasa train`. This command combines data from local files and Rasa Studio to train a model. In case both Studio and local files have a primitive\](../primitives/index.mdx) with the same ID, local one is used for training.

Example:

```
rasa studio train my_awesome_assistant -d my_domain_folder
```

Run `rasa studio train --help` to see the full list of arguments.

### Other[​](https://rasa.com/docs/reference/api/command-line-interface/#other 'Direct link to Other')

For a full list of legacy commands, please head over to [this reference](https://legacy-docs-oss.rasa.com/docs/rasa/command-line-interface/).

- [Cheat Sheet](https://rasa.com/docs/reference/api/command-line-interface/#cheat-sheet)
 - [Pro specific](https://rasa.com/docs/reference/api/command-line-interface/#pro-specific)
 - [Studio specific](https://rasa.com/docs/reference/api/command-line-interface/#studio-specific)
- [Rasa Pro Commands](https://rasa.com/docs/reference/api/command-line-interface/#rasa-pro-commands)
 - [Logging](https://rasa.com/docs/reference/api/command-line-interface/#logging)
 - [Custom logging configuration](https://rasa.com/docs/reference/api/command-line-interface/#custom-logging-configuration)
 - [rasa init](https://rasa.com/docs/reference/api/command-line-interface/#rasa-init)
 - [rasa train](https://rasa.com/docs/reference/api/command-line-interface/#rasa-train)
 - [rasa shell](https://rasa.com/docs/reference/api/command-line-interface/#rasa-shell)
 - [rasa run](https://rasa.com/docs/reference/api/command-line-interface/#rasa-run)
 - [rasa run actions](https://rasa.com/docs/reference/api/command-line-interface/#rasa-run-actions)
 - [rasa visualize](https://rasa.com/docs/reference/api/command-line-interface/#rasa-visualize)
 - [rasa test e2e](https://rasa.com/docs/reference/api/command-line-interface/#rasa-test-e2e)
 - [rasa llm finetune prepare-data](https://rasa.com/docs/reference/api/command-line-interface/#rasa-llm-finetune-prepare-data)
 - [Resulting file structure](https://rasa.com/docs/reference/api/command-line-interface/#resulting-file-structure)
 - [rasa inspect](https://rasa.com/docs/reference/api/command-line-interface/#rasa-inspect)
- [rasa inspect --nextgen](https://rasa.com/docs/reference/api/command-line-interface/#rasa-inspect---nextgen)
 - [rasa data validate](https://rasa.com/docs/reference/api/command-line-interface/#rasa-data-validate)
 - [rasa export](https://rasa.com/docs/reference/api/command-line-interface/#rasa-export)
 - [rasa license](https://rasa.com/docs/reference/api/command-line-interface/#rasa-license)
- [Rasa Studio Commands](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-commands)
 - [rasa studio config](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-config)
 - [rasa studio login](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-login)
 - [rasa studio upload](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-upload)
 - [rasa studio download](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-download)
 - [rasa studio link](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-link)
 - [rasa studio pull](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-pull)
 - [rasa studio push](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-push)
- [Legacy commands](https://rasa.com/docs/reference/api/command-line-interface/#legacy-commands)
 - [rasa studio upload](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-upload-1)
 - [rasa studio download](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-download-1)
 - [rasa studio train](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-train)
 - [Other](https://rasa.com/docs/reference/api/command-line-interface/#other)