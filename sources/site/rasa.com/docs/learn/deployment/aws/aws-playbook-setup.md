# Source: https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup

On this page

## Prerequisites[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#prerequisites 'Direct link to Prerequisites')

Before you begin, you will need to have access to an AWS account with the specific permissions. You can attach the following policy to your user to give it those permissions:

[Playbook IAM Policy](https://github.com/RasaHQ/rasa-deployment-playbooks/blob/main/aws/permissions/policy.json)

Alternatively, you can attach the AWS managed policy called `AdministratorAccess` to give the user broader permissions which would cover the need of this deployment playbook.

## Install Tools[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#install-tools 'Direct link to Install Tools')

Next, you'll need to install some tools on your local machine. If you already have these tools installed, please ensure they are updated to the latest version.

- AWS CLI allows you to interact with AWS to deploy the infrastructure. Installation instructions are [here](https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-install.html).
- kubectl allows you to interact with the Kubernetes cluster you deploy to configure it. Installation instructions are [here](https://kubernetes.io/docs/tasks/tools/#kubectl).
- helm which allows you to deploy the Rasa products onto your cluster. Installation instructions are [here](https://helm.sh/docs/intro/install/).
- envsubst which allows us to easily inject the values of environment variables into our deployment commands. This is included with most Linux distros and can be installed on MacOS with `brew install gettext`. Windows users can utilise it via Git Bash.
- One of [Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) or [OpenTofu](https://opentofu.org/docs/intro/install/) to deploy the infrastructure you need onto AWS. If you're not sure what to pick, use OpenTofu.

Once you've installed these tools, you should authenticate with AWS. The method you choose will depend on how your AWS environment is configured.

- Using IAM Access Key & Secret Key
- Using AWS SSO

If you're new to AWS and you're working in a new account, or an account that isn't managed by your company then follow [these](https://docs.aws.amazon.com/cli/v1/userguide/cli-authentication-user.html) instructions.

If you're working in an AWS account managed by a company, or where you know that IAM Identity Centre is configured, follow [these](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html) instructions.

## Specify Input Parameters[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#specify-input-parameters 'Direct link to Specify Input Parameters')

You'll need to configure some values which the rest of the deployment process will use. We'll configure these as environment variables - make sure you change these to values that are correct for your situation:

If you haven't already, clone our [deployment-playbooks](https://github.com/RasaHQ/deployment-playbooks) repo locally with:

```
git clone https://github.com/RasaHQ/rasa-deployment-playbooks.git
```

Next, open the file `aws/setup/environment-variables.sh` and follow the instructions to edit the required values inside. Once you're happy with the values you've set, let's ensure these values are available for you to use in your shell throughout the rest of the playbook. Be especially sure to set all the values in the first section to ensure you can proceed with the rest of the playbook. A valid Rasa Pro license key must also be set in here or the deployment steps will not work.

```
source aws/setup/environment-variables.sh
```

- [Prerequisites](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#prerequisites)
- [Install Tools](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#install-tools)
- [Specify Input Parameters](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#specify-input-parameters)