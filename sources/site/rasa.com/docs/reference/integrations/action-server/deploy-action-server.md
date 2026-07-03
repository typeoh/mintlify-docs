# Source: https://rasa.com/docs/reference/integrations/action-server/deploy-action-server

On this page

## Building an Action Server Image[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#building-an-action-server-image 'Direct link to Building an Action Server Image')

If you build an image that includes your action code and store it in a container registry, you can run it as part of your deployment, without having to move code between servers. In addition, you can add any additional dependencies of systems or Python libraries that are part of your action code but not included in the base `rasa/rasa-sdk` image.

### Manually Building an Action Server[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#manually-building-an-action-server 'Direct link to Manually Building an Action Server')

To create your image:

1. Make sure your actions are defined in `actions/actions.py`. The `rasa/rasa-sdk` image will automatically look for the actions in this file.
 
2. If your actions have any extra dependencies, create a list of them in a file, `actions/requirements-actions.txt`.
 
3. Create a file named `Dockerfile` in your project directory, in which you'll extend the official SDK image, copy over your code, and add any custom dependencies (if necessary). For example:
 
 Dockerfile
 
 ```
    # Extend the official Rasa SDK imageFROM rasa/rasa-sdk:latest# Use subdirectory as working directoryWORKDIR /app# Copy any additional custom requirements, if necessary (uncomment next line)# COPY actions/requirements-actions.txt ./# Change back to root user to install dependenciesUSER root# Install extra requirements for actions code, if necessary (uncomment next line)# RUN pip install -r requirements-actions.txt# Copy actions folder to working directoryCOPY ./actions /app/actions# By best practices, don't run the code with root userUSER 1001
    ```
 

You can then build the image via the following command:

```
docker build . -t <account_username>/<repository_name>:<custom_image_tag>
```

The `<custom_image_tag>` should reference how this image will be different from others. For example, you could version or date your tags, as well as create different tags that have different code for production and development servers. You should create a new tag any time you update your code and want to re-deploy it.

### Using your Custom Action Server Image[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#using-your-custom-action-server-image 'Direct link to Using your Custom Action Server Image')

If you're building this image to make it available from another server, you should push the image to a cloud repository.

This documentation assumes you are pushing your images to [DockerHub](https://hub.docker.com/). DockerHub will let you host multiple public repositories and one private repository for free. Be sure to first [create an account](https://hub.docker.com/signup/) and [create a repository](https://hub.docker.com/signup/) to store your images. You could also push images to a different Docker registry, such as [Google Container Registry](https://cloud.google.com/container-registry), [Amazon Elastic Container Registry](https://aws.amazon.com/ecr/), or [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/).

You can push the image to DockerHub via:

```
docker login --username <account_username> --password <account_password>docker push <account_username>/<repository_name>:<custom_image_tag>
```

To authenticate and push images to a different container registry, please refer to the documentation of your chosen container registry.

## Deploy an Action Server with Rasa Pro Helm Chart[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#deploy-an-action-server-with-rasa-pro-helm-chart 'Direct link to Deploy an Action Server with Rasa Pro Helm Chart')

To run a custom action server you need a Docker image for your action server. Please see [Building an Action Server Image](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#building-an-action-server-image) for more information.

### Update values.yml[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#update-valuesyml 'Direct link to Update values.yml')

Update your **values.yml** file with the image and tag you want to use:

values.yml

```
rasa:  endpoints:    action_endpoint:        url: http://rasa-action-server:5055/webhookactionServer:  enabled: true  image:    repository: <account_username>/<repository_name>    tag: <custom_image_tag>
```

### Deploy with Helm[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#deploy-with-helm 'Direct link to Deploy with Helm')

To update the helm deployment, you can use:

```
helm upgrade \    --namespace <your namespace> \    --values values.yml \    <release name> \    rasa-<version>.tgz
```

## Environment Variables[​](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#environment-variables 'Direct link to Environment Variables')

All available environment variables for the Rasa Action Server can be found [here](https://rasa.com/docs/reference/config/environment-variables/#rasa-sdk).

- [Building an Action Server Image](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#building-an-action-server-image)
 - [Manually Building an Action Server](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#manually-building-an-action-server)
 - [Using your Custom Action Server Image](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#using-your-custom-action-server-image)
- [Deploy an Action Server with Rasa Pro Helm Chart](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#deploy-an-action-server-with-rasa-pro-helm-chart)
 - [Update values.yml](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#update-valuesyml)
 - [Deploy with Helm](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#deploy-with-helm)
- [Environment Variables](https://rasa.com/docs/reference/integrations/action-server/deploy-action-server/#environment-variables)