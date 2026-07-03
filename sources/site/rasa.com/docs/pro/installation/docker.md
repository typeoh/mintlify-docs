# Source: https://rasa.com/docs/pro/installation/docker

On this page

The Rasa Pro Docker image is available on [Docker Hub](https://hub.docker.com/r/rasa/rasa-pro/tags) as `rasa/rasa-pro`.

note

In the commands below, replace `RASAVERSION` with your desired version of Rasa Pro. To find the latest version of Rasa Pro, see the [Changelog](https://rasa.com/docs/reference/changelogs/rasa-pro-changelog/).

To pull the image, run:

```
docker pull rasa/rasa-pro:RASAVERSION
```

## Optional Dependencies (Extras)[​](https://rasa.com/docs/pro/installation/docker/#optional-dependencies-extras 'Direct link to Optional Dependencies (Extras)')

Rasa Pro supports optional dependencies through "extras":

- **`nlu`**: Dependencies required for NLU components
- **`channels`**: Dependencies required for channel connectors
- **`full`**: Complete set of all optional dependencies

By default, the Rasa Pro Docker image already contains the `nlu` and `channels` dependencies.

### Extending the Docker Image[​](https://rasa.com/docs/pro/installation/docker/#extending-the-docker-image 'Direct link to Extending the Docker Image')

To extend the Docker image with optional dependencies that are not already included in the default image, create a custom Dockerfile:

```
# Extend the official Rasa Pro imageFROM rasa/rasa-pro:RASAVERSION# Install all optional dependenciesRUN pip install rasa[full]# Alternatively, install individual optional packages rather than all optional dependencies comprised in `full`RUN pip install rasa[spacy]
```

Build and run your custom image:

```
# Build the custom imagedocker build -t my-custom-rasa:latest .# Run the custom imagedocker run -v ./:/app \            -e RASA_LICENSE=${RASA_LICENSE} \            -p 5005:5005 \            my-custom-rasa:latest \            run
```

## Licensing[​](https://rasa.com/docs/pro/installation/docker/#licensing 'Direct link to Licensing')

As explained in [Licensing](https://rasa.com/docs/pro/installation/licensing/), at runtime you must provide the license key in the environment variable `RASA_LICENSE`.

After you defined the environment variable on your system, you can pass it into the docker container via the `-e` parameter, for example:

```
# execute `rasa init` via dockerdocker run -v ./:/app \            -e RASA_LICENSE=${RASA_LICENSE} \            rasa/rasa-pro:RASAVERSION \            init --no-prompt --template tutorial
```

- [Optional Dependencies (Extras)](https://rasa.com/docs/pro/installation/docker/#optional-dependencies-extras)
 - [Extending the Docker Image](https://rasa.com/docs/pro/installation/docker/#extending-the-docker-image)
- [Licensing](https://rasa.com/docs/pro/installation/docker/#licensing)