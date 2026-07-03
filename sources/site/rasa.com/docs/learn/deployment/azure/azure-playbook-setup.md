# Source: https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup

On this page

## Prerequisites[​](https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup/#prerequisites 'Direct link to Prerequisites')

Before you begin, you will need to have an Azure account. The account needs to have the "Owner" role for the Azure subscription you want deploy Rasa to.

## Install Tools[​](https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup/#install-tools 'Direct link to Install Tools')

Next, you'll need to install some tools on your local machine. If you already have these tools installed, please ensure they are updated to the latest version.

- Azure CLI allows you to interact with Azure to deploy the infrastructure. Installation instructions are [here](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).
- kubectl allows you to interact with the Kubernetes cluster you deploy to configure it. Installation instructions are [here](https://kubernetes.io/docs/tasks/tools/#kubectl).
- helm which allows you to deploy the Rasa products onto your cluster. Installation instructions are [here](https://helm.sh/docs/intro/install/).
- envsubst which allows us to easily inject the values of environment variables into our deployment commands. This is included with most Linux distros and can be installed on MacOS with `brew install gettext`. Windows users can utilise it via Git Bash.
- One of [Terraform](https://developer.hashicorp.com/terraform/tutorials/azure-get-started/install-cli) or [OpenTofu](https://opentofu.org/docs/intro/install/) to deploy the infrastructure you need onto Azure. If you're not sure what to pick, use OpenTofu.

Once you've installed these tools, you should authenticate with Azure:

```
az login
```

This will start an interactive login flow. For other authentication methods, refer to the [az login documentation](https://learn.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-login).

The login flow will prompt you to select a subscription. To check or change your current subscription run the following commands:

```
az account show
```

```
az account set --subscription "<SUBSCRIPTION_ID_OR_NAME>"
```

## Specify Input Parameters[​](https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup/#specify-input-parameters 'Direct link to Specify Input Parameters')

You'll need to configure some values which the rest of the deployment process will use. We'll configure these as environment variables - make sure you change these to values that are correct for your situation:

If you haven't already, clone our [deployment-playbooks](https://github.com/RasaHQ/deployment-playbooks) repo locally with:

```
git clone https://github.com/RasaHQ/deployment-playbooks.git
```

Next, open the file `azure/setup/environment-variables.sh` and follow the instructions to edit the required values inside. Once you're happy with the values you've set, let's ensure these values are available for you to use in your shell throughout the rest of the playbook. Be especially sure to set all of the values in the first section to ensure you can proceed with the rest of the playbook. A valid Rasa Pro license key must also be set in here or the deployment steps will not work.

```
source azure/setup/environment-variables.sh
```

- [Prerequisites](https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup/#prerequisites)
- [Install Tools](https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup/#install-tools)
- [Specify Input Parameters](https://rasa.com/docs/learn/deployment/azure/azure-playbook-setup/#specify-input-parameters)