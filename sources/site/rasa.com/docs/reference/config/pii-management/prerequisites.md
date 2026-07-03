# Source: https://rasa.com/docs/reference/config/pii-management/prerequisites

On this page

## Prerequisites[​](https://rasa.com/docs/reference/config/pii-management/prerequisites/#prerequisites 'Direct link to Prerequisites')

To use the PII management capability, you need to have the following prerequisites in place:

- Rasa Pro version 3.13.0 or later installed.
- Defined `privacy` YAML config in your Rasa project.
- A [Lock Store](https://rasa.com/docs/reference/integrations/lock-stores/) when tracker store anonymization or deletion is configured (required by the background privacy manager to avoid race conditions during read-modify-write on trackers).

### GLiNER Requirements[​](https://rasa.com/docs/reference/config/pii-management/prerequisites/#gliner-requirements 'Direct link to GLiNER Requirements')

To use the GLiNER PII identification, you need to have the following prerequisites in place:

- Rasa Pro version 3.13.0 or later installed with the `pii` optional extra, for example:
 
 ```
    uv pip install "rasa-pro[pii]"poetry add rasa-pro -E pii
    ```
 
- [GLiNER PII model](https://huggingface.co/urchade/gliner_multi_pii-v1) downloaded and available in your Rasa project.

#### GLiNER Model Download[​](https://rasa.com/docs/reference/config/pii-management/prerequisites/#gliner-model-download 'Direct link to GLiNER Model Download')

To download the GLiNER PII model prior to starting the Rasa assistant, run the following script in your Rasa venv:

download\_model.py

```
import osfrom pathlib import Pathfrom gliner import GLiNERfrom transformers import AutoTokenizerdef download_model(model_path: Path, model_name: str) -> None:    """Download a Gliner model to the specified directory."""    # Check if the directory already exists    if not os.path.exists(model_path):        # Create the directory        os.makedirs(model_path)    # The default tokenizer for Gliner is DeBERTa v2, which results in    # protobuf issues as we are using protobuf 5.29.5.    # Use BERT tokenizer instead of DeBERTa v2.    print(f"Downloading GLiNER model: {model_name}")    print("Using BERT tokenizer to avoid protobuf issues...")    try:        # Load a BERT tokenizer that's compatible        tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")        # Load GLiNER model with custom tokenizer        model = GLiNER.from_pretrained(model_name, tokenizer=tokenizer)        model.save_pretrained(model_path)        print(f"Successfully downloaded and saved model to {model_path}")    except Exception as e:        print(f"Error with BERT tokenizer: {e}")        print("Trying with default tokenizer...")        # Fallback to default tokenizer        model = GLiNER.from_pretrained(model_name)        model.save_pretrained(model_path)        print(f"Successfully downloaded and saved model to {model_path}")if __name__ == "__main__":    local_model_path = Path("./gliner_model").resolve() # You can modify this path to your desired location    download_model(        model_path=local_model_path,        model_name="urchade/gliner_multi_pii-v1"    )
```

This script downloads the GLiNER PII model and saves it to the specified directory. Then set the `GLINER_MODEL_PATH` [environment variable](https://rasa.com/docs/reference/config/pii-management/prerequisites/#environment-variables) to the path where the model is saved.

To create a custom Docker image for the assistant, you can modify the `Dockerfile` in your Rasa Pro root project to include the model download script:

Dockerfile

```
FROM rasa/rasa-pro:3.13.0 AS builderUSER root# Install dependenciesRUN python3 -m venv /opt/venv &&  \    . /opt/venv/bin/activate &&  \    pip install --no-cache-dir -U "pip==24.*" &&  \    pip install --no-cache-dir "rasa-pro[pii]===3.13.0" &&  \    # pin transformers to avoid connection errors with their \    # OpenTelemetry metrics collector occurring from >=4.53.0    pip install --no-cache-dir "transformers==4.52.4"# Download HF modelCOPY ./gliner_model_download_script.py gliner_model_download_script.py# Force slow tokenizers and use pure-Python protobuf only for the download step# This is needed to avoid Protobuf 5 compatibility issues with the fast tokenizersRUN TRANSFORMERS_NO_FAST_TOKENIZER=1 PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python3 -m gliner_model_download_scriptFROM builderWORKDIR /appCOPY --from=builder /app/gliner_model /app/gliner_modelRUN --mount=type=bind,target=/app/gliner_model# Update permissionsRUN chown -R 1001:0 /app/gliner_model && \    chmod -R g=u /app/gliner_model && \    chmod o+wr /app/gliner_modelUSER 1001# Check if the model files are downloaded correctlyRUN ls -l /app/gliner_model
```

After building the Docker image: `docker build . -t pii-assistant:latest`, you can run the assistant with the GLiNER PII model available in your Rasa project using this docker compose file which also requires `BOT_PATH` environment variable to be set to the path of your Rasa project:

docker-compose.yml

```
x-license: &license  RASA_LICENSE: ${RASA_LICENSE}  OPENAI_API_KEY: ${OPENAI_API_KEY}x-pii-config: &pii-env-vars  USER_CHAT_INACTIVITY_IN_MINUTES: 15  GLINER_MODEL_PATH: "/app/gliner_model"services:  pii_assistant:    image: pii-assistant:latest    container_name: pii_assistant    volumes:      - "${BOT_PATH}:/app/bot"    working_dir: /app/bot    entrypoint: ""    command: rasa run -p 5005 --enable-api    ports:      - "5005:5005"    networks:      - default    environment:      <<: [ *license, *pii-env-vars ]    healthcheck:      test: curl localhost:5005 || exit 1      interval: 10s      retries: 10      start_period: 15s      timeout: 10s    user: "rasa"
```

#### Machine Spec Requirements with GLiNER[​](https://rasa.com/docs/reference/config/pii-management/prerequisites/#machine-spec-requirements-with-gliner 'Direct link to Machine Spec Requirements with GLiNER')

The Rasa Pro image with pre-downloaded GLiNER model will be larger than the standard Rasa Pro image due to the GLiNER model size. If you are running the Rasa assistant with the GLiNER model in a Docker container, ensure that your machine has sufficient resources to run the model.

In addition, if you are running your Rasa assistant with multiple Sanic workers, the GLiNER model will be loaded into memory for each worker and will require additional 1 - 2GB of memory per worker. Ensure to choose your assistant's number of Sanic workers according to the available memory on your machine. For example, a machine with 32GB of RAM can run up to 10 Sanic workers with the GLiNER model loaded in memory.

### Environment Variables[​](https://rasa.com/docs/reference/config/pii-management/prerequisites/#environment-variables 'Direct link to Environment Variables')

You can configure the PII management capability using the following environment variables:

New in 3.16

`USER_CHAT_INACTIVITY_IN_MINUTES` is deprecated. Prefer leaving it unset so that eligibility uses [session management](https://rasa.com/docs/reference/config/session-management/overview/) (event-based with `ConversationInactive`/`SessionEnded` and `session_id` grouping) and different [tracker variants](https://rasa.com/docs/reference/config/pii-management/overview/#tracker-variants-and-session-eligibility) are handled accordingly.

- `USER_CHAT_INACTIVITY_IN_MINUTES`: When **unset**, “new” trackers (events with `session_id`) are processed for PII anonymization or deletion based on `ConversationInactive`/`SessionEnded` events and `session_id` grouping, with a grace period of `min_after_session_end`. “Legacy” trackers (no `session_id`) use a fixed 30 minutes plus `min_after_session_end`. When **set**, time-based eligibility (env value + `min_after_session_end`) is applied per session; this behavior is deprecated. Default when used for legacy/time-based logic is `30` minutes.
- `GLINER_MODEL_PATH`: The path to the downloaded GLiNER PII model in your Rasa project.
- `HUGGINGFACE_HUB_CACHE_DIR`: The path to the HuggingFace Hub cache directory if defined when downloading the GLiNER model.

- [Prerequisites](https://rasa.com/docs/reference/config/pii-management/prerequisites/#prerequisites)
 - [GLiNER Requirements](https://rasa.com/docs/reference/config/pii-management/prerequisites/#gliner-requirements)
 - [Environment Variables](https://rasa.com/docs/reference/config/pii-management/prerequisites/#environment-variables)