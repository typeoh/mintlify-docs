# Source: https://rasa.com/docs/studio/deployment/architecture

## High Level Architecture[​](https://rasa.com/docs/reference/architecture/studio/#high-level-architecture 'Direct link to High Level Architecture')

Below is a high level architecture diagram of Rasa Studio. The diagram shows the main components of Rasa Studio and how they interact with each other.

_Please click on the image to view it in full size._

 

![image](https://rasa.com/docs/assets/images/studio-architecture-diagram-with-new-model-service-999739fb6c016bd0f1c8b39977933adf.svg)

## Containers[​](https://rasa.com/docs/reference/architecture/studio/#containers 'Direct link to Containers')

The below image shows the different containers that make up Rasa Studio. The containers are deployed on a Kubernetes cluster.

_Please click on the image to view it in full size._

 

![image](https://rasa.com/docs/assets/images/studio-containers-with-new-model-service-94cd304f765565c7d7950e244ff52afd.svg)

The following container images are part of the Studio deployment:

1. **Studio Web Client Container:** This container hosts the frontend service of Rasa Studio. It provides a user-friendly web interface that allows users to interact with and manage their conversational AI projects.
2. **Studio Backend Server Container:** The Studio backend server container hosts the backend service of Rasa Studio. It processes incoming requests, executes application logic and manages data. The backend server exposes APIs that the web client interacts with to perform various actions in the UI. It also exposes APIs for a developer user to use `Rasa Pro Studio` [CLI commands](https://rasa.com/docs/reference/api/command-line-interface/#rasa-studio-download) to pull/push data from/to Studio.
3. **Studio Database Migration Server Container:** This container is used specifically for running database migrations during deployment or upgrades of Rasa Studio. It ensures that the database schema is up to date and compatible with the latest version of Rasa Studio. This container runs as a Kubernetes job during deployment.
4. **Studio Keycloak Container:** Studio uses [Keycloak](https://www.keycloak.org/) as its user management system. The Keycloak container provides authentication, authorization, and user management capabilities for Rasa Studio. It handles user registration, login, password management, and access control to ensure secure access to Studio's features and functionalities.
5. **Studio Ingestion Service Container:** The Studio Ingestion Service container is responsible for ingesting conversation events from Rasa assistant via the Kafka broker. It listens to the specified Kafka topic, receives bot events, and processes them for storage and analysis within Rasa Studio.
6. **Studio Model Service Container:** This is a service native to Rasa that takes care of training and running an assistant model in Studio. This powers the `try your assistant` feature where a user can try the CALM assistant that they created and trained. The container that runs this service is based off the Rasa Pro container image version >= `3.11.0`.

### Studio Model Service Container[​](https://rasa.com/docs/reference/architecture/studio/#studio-model-service-container 'Direct link to Studio Model Service Container')

note

Once you have upgraded to Studio >= v`1.10.0`, you should run a new training request with the new model service to be able to test this model with the `try your assistant` feature.

#### Training[​](https://rasa.com/docs/reference/architecture/studio/#training 'Direct link to Training')

When training is triggered from the UI, Studio sends a training request to the Rasa Pro model service API. This training request spawns a new subprocess in Rasa that runs the `rasa train` command.

The following diagram depicts the process flow of the training request:

 

![image](https://rasa.com/docs/assets/images/model-service-process-diagram-6c734385b0d623c73644e9a728ba2607.svg)

The training process involves the following steps:

- The model service retrieves the training data from the Studio request.
- The model service creates a new directory on the local disk to store the training data.
- The model service writes the training data to the directory.
- The model service runs the `rasa train` command in the subprocess.

The model service allows for multiple trainings to be run concurrently, currently set by default to **10**. This limit is set to prevent the system resources from being overloaded with too many training processes.

If you want to update this limit, you can configure the `MAX_PARALLEL_TRAININGS` environment variable in Studio's Model Service Rasa Pro container. Make sure that the memory and CPU resources set for the Model Service container are sufficient to handle the number of parallel trainings you set. We have outlined the [resource usage recommendations](https://rasa.com/docs/reference/architecture/studio/#resource-usage-recommendations) for the Model Service container below.

When the model service container receives more requests than the defined limit, the training request fails and the user can retry at a later point in time. If the limit is reached, the model service API will return a `429 Too Many Requests` response to the Studio backend server. This response is then sent back to the Studio UI to inform the user that the training request has been rejected due to the limit being reached. If this happens regularly, we recommend to both increase the hardware resources of the Rasa Pro container and increase the limit of parallel trainings.

Once the training process is complete, Studio sends the model service API a request to run the assistant. This request involves the following steps:

- retrieves the model from the local disk or cloud storage
- spawns a new subprocess that runs the `rasa run` command

The model service allows for multiple assistants to be run concurrently, currently set by default to **10**. This limit is set to prevent the system resources from being overloaded with too many assistant processes.

If you want to update this limit, you can configure the `MAX_PARALLEL_BOT_RUNS` environment variable in Studio's Model Service Rasa Pro container. Make sure that the memory and CPU resources set for the Model Service container are sufficient to handle the number of parallel bot runs you set. We have outlined the [resource usage recommendations](https://rasa.com/docs/reference/architecture/studio/#resource-usage-recommendations) for the Model Service container below.

When the model service container receives more requests than the limit set by `MAX_PARALLEL_BOT_RUNS`, it does not queue the requests which is a limitation of the current iteration. If the limit is reached, the model service API will return a `429 Too Many Requests` response to the Studio backend server. This response is then sent back to the Studio UI to inform the user that the assistant run request has been rejected due to the limit being reached. If this happens regularly, we recommend to both increase the hardware resources of the Rasa Pro container and increase the limit of parallel bot runs.

Once the assistant is up and running, the Studio UI notifies the user that the training was successful.

#### Model Storage[​](https://rasa.com/docs/reference/architecture/studio/#model-storage 'Direct link to Model Storage')

If the training is successful, the trained model is saved to local disk by default.

If you also prefer to have the trained model uploaded to cloud storage, you should configure the `RASA_REMOTE_STORAGE` environment variable in the Studio deployment. This environment variable should be set to one of the Rasa [supported cloud storage options](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-cloud) - `aws`, `gcs`, `azure`.

Note that if the model is saved to local disk and the container is deployed on Kubernetes or Docker, you should ensure that you specify a persistent volume mounted to the following path: `/app/working-data`, where `working-data` is the default base working directory name for the model service. You can configure the working directory name by setting the `RASA_MODEL_SERVER_BASE_DIRECTORY` environment variable in the Rasa Pro container deployment.

Studio default Helm charts take care of this for you directly.

#### Try Your Assistant[​](https://rasa.com/docs/reference/architecture/studio/#try-your-assistant 'Direct link to Try Your Assistant')

When a user wants to use the `try your assistant` feature in Studio, the model service API fulfills the following tasks:

- authenticates the Studio user with the Rasa backend server to ensure that the user has the necessary permissions to chat with the assistant.
- creates a socketIO bridge between the Studio socketIO session and the running assistant. This bridge enables the Studio user to send messages to the running bot and to receive back the bot responses.

#### Environment Variables[​](https://rasa.com/docs/reference/architecture/studio/#environment-variables 'Direct link to Environment Variables')

The following environment variables can be configured on the Studio Model Service Rasa Pro container:

- `MAX_PARALLEL_TRAININGS`: The maximum number of parallel trainings that can be run by the model service. Default is `10`.
- `MAX_PARALLEL_BOT_RUNS`: The maximum number of parallel bot runs that can be run by the model service. Default is `10`.
- `RASA_REMOTE_STORAGE`: The cloud storage option where the trained model should be uploaded. Default is `None`.

The following environment variables can be configured on the Studio Backend Server Container:

- `TRAINING_POLLING_INTERVAL_MS`: The interval in ms for polling training status from Model Service. Default is `1000` ms.
- `MODEL_POLLING_INTERVAL_MS`: The interval in ms for polling model status from Model Service. Default is `2000` ms.
- `MODEL_INACTIVE_AFTER_MS`: The interval in which unused models get set inactive. Default is `1` h.

#### Resource Usage Recommendations[​](https://rasa.com/docs/reference/architecture/studio/#resource-usage-recommendations 'Direct link to Resource Usage Recommendations')

Our load tests on a standard CALM assistant show that the Studio Model Service Rasa Pro container requires:

- a minimum of `1 vCPU core` per training, the resource is released once the training is completed.
- `500 Mi/0.5 Gi` of memory per every running assistant.