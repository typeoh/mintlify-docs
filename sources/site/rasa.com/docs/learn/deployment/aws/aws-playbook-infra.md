# Source: https://rasa.com/docs/learn/deployment/aws/aws-playbook-infra

warning

Before you begin, ensure you've completed the [Setup Steps](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/) to set environment variables and clone a repo that this section requires.

To deploy the required infrastructure into your AWS environment, use the provided script in our [deployment-playbooks](https://github.com/RasaHQ/rasa-deployment-playbooks) repo.

```
./aws/deploy/deploy-infrastructure.sh
```

If you'd like to understand more about exactly what the script is doing, open it in your IDE of choice and review the comments which explain what is being deployed at each step.