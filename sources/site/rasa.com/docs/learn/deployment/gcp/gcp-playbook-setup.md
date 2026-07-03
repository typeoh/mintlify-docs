# Source: https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup

On this page

## Prerequisites[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#prerequisites 'Direct link to Prerequisites')

Before you begin, you will need to have access to a Google Cloud Platform project - you can use an existing one but we recommend [creating a new project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project) to ensure there's no resource conflicts and you can easily track billing and access controls.

Within the project, your user will need to have the following roles, which you can check in the Google Cloud console under IAM & Admin > IAM:

- Compute Admin
- Service Networking Admin
- Kubernetes Engine Admin
- Cloud SQL Admin
- Cloud Memorystore Redis Admin
- DNS Administrator
- Service Usage Admin

Alternatively, you can grant yourself the Owner or Editor role on the project which will be sufficient.

Your GCP project must be linked to a billing account to ensure that you can pay for the resources you deploy.

Finally, you must have a valid Rasa license key with entitlements for Rasa Pro and Rasa Studio.

## Install Tools[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#install-tools 'Direct link to Install Tools')

Next, you'll need to install some tools on your local machine. If you already have these tools installed, please ensure they are updated to the latest version.

- gcloud CLI allows you to interact with GCP to deploy the infrastructure. Installation instructions are [here](https://cloud.google.com/sdk/docs/install).
- kubectl allows you to interact with the Kubernetes cluster you deploy to configure it. Installation instructions are [here](https://kubernetes.io/docs/tasks/tools/#kubectl).
- The gke-cloud-auth-plugin allows you to authenticate with your Google Kubernetes Engine cluster and interact with it using kubectl. Install if after you've set up gcloud with `gcloud components install gke-gcloud-auth-plugin`.
- helm which allows you to deploy the Rasa products onto your cluster. Installation instructions are [here](https://helm.sh/docs/intro/install/).
- envsubst which allows us to easily inject the values of environment variables into our deployment commands. This is included with most Linux distros and can be installed on MacOS with `brew install gettext`. Windows users can utilise it via Git Bash.

Once you've installed these tools, you should authenticate with GCP:

```
gcloud auth login
```

## Specify Input Parameters[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#specify-input-parameters 'Direct link to Specify Input Parameters')

You'll need to configure some values which the rest of the deployment process will use. We'll configure these as environment variables - make sure you change these to values that are correct for your situation:

Begin by cloning our [deployment-playbooks](https://github.com/RasaHQ/rasa-deployment-playbooks) repo locally with:

```
git clone https://github.com/RasaHQ/rasa-deployment-playbooks.git
```

Next, open the file `gcp/setup/environment-variables.sh` and follow the instructions to edit the required values inside. Once you're happy with the values you've set, let's ensure these values are available for you to use in your shell throughout the rest of the playbook. Be especially sure to set all of the values in the first section to ensure you can proceed with the rest of the playbook. **A valid Rasa Pro license key must also be set in here or the deployment steps will not work.**

```
source gcp/setup/environment-variables.sh
```

## Set Up gcloud CLI[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#set-up-gcloud-cli 'Direct link to Set Up gcloud CLI')

Finally, let's set up the gcloud CLI for the rest of the deployment process:

```
gcloud config set project $PROJECT_IDgcloud config set disable_prompts True
```

You can safely ignore a warning that says `Your active project does not match the quota project in your local Application Default Credentials file. This might result in unexpected quota issues.`.

You can then validate your configuration with:

```
gcloud auth listgcloud config list
```

You should see here that the active account is your Google user and that you're working in the correct project you expect.

- [Prerequisites](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#prerequisites)
- [Install Tools](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#install-tools)
- [Specify Input Parameters](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#specify-input-parameters)
- [Set Up gcloud CLI](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/#set-up-gcloud-cli)