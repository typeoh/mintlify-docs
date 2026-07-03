# Source: https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide

On this page

## Requirements[​](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/#requirements 'Direct link to Requirements')

To deploy Rasa Studio using Docker Compose, you'll need:

- Docker (version 20.10.0 or higher).
- Docker Compose (version 2.0.0 or higher) (Not required if you have Docker Desktop installed).
- At least 4GB of available RAM.
- A Rasa license key with an entitlement to use Studio.
- An OpenAI API key or credentials for another LLM if you wish to use LLM powered features.

## Get Started[​](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/#get-started 'Direct link to Get Started')

The Docker Compose file to get started with Rasa Studio can be found [here](https://github.com/RasaHQ/studio-docker-compose/). You should clone this repo to your local machine to begin.

1. Rasa Studio requires some environment variables to be set before you can start the application. Open the `.env` file and set the values within. We recommend that you begin with the [latest version](https://www.rasa.com/docs/reference/changelogs/studio-changelog) of Rasa Studio and the [latest compatible version](https://www.rasa.com/docs/reference/changelogs/rasa-pro-changelog) of Rasa Pro. You can check for compatibility using our [Compatibility Matrix](https://www.rasa.com/docs/reference/changelogs/compatibility-matrix#studiopro-versions-compatibility).
 
2. Start all the services for Rasa Studio by running
 
 ```
    docker compose up
    ```
 
3. The startup process will show logs from all services in your terminal. The application is ready when you see the following message from the startup-helper service:
 
 ```
    🚀 Rasa Studio is ready!📱 Studio URL: http://localhost:8080🔐 Keycloak User Management URL: http://localhost:8081/auth/
    ```
 
 If you prefer to run the services in the background, you can use:
 
 ```
    docker compose up -d
    ```
 
 Then monitor the progress with:
 
 ```
    docker compose logs -f startup-helper
    ```
 

## Access Rasa Studio[​](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/#access-rasa-studio 'Direct link to Access Rasa Studio')

Once the application is ready, the services will be available on different ports and paths:

```
Main Application: http://localhost:8080Keycloak Admin Console: http://localhost:8081/auth/Backend API: http://localhost:4000/apiModel Service: http://localhost:8000
```

You can follow our instructions to activate your license and set up users [here](https://www.rasa.com/docs/studio/installation/setup-guides/license-activation).

The Docker Compose setup sets some default credentials for you to use which you can find in our repo's [README file](https://github.com/RasaHQ/studio-docker-compose/?tab=readme-ov-file#credentials-reference). Also documented there are the environment variables you can use to optionally override these default credentials at deployment time by adding the mentioned variables to the `.env` file before you run `docker compose up`.

- [Requirements](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/#requirements)
- [Get Started](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/#get-started)
- [Access Rasa Studio](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/#access-rasa-studio)