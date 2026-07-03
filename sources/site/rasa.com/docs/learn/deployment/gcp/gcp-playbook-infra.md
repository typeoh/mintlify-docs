# Source: https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-infra

warning

Before you begin, ensure you've completed the [Setup Steps](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/) to set environment variables and clone a repo that this section requires.

To deploy the required infrastructure into your GCP environment, use the provided script in our [deployment-playbooks](https://github.com/RasaHQ/rasa-deployment-playbooks) repo.

```
./gcp/deploy/deploy-infrastructure.sh
```

If you'd like to understand more about exactly what the script is doing, open it in your IDE of choice and review the comments which explain what is being deployed at each step.

The first part of this script will attempt to enable the following GCP Services in your project which are required for the deployment.

- [Compute Engine API](https://docs.cloud.google.com/compute/docs/reference/rest/v1), for creating the virtual network.
- [Service Networking API](https://docs.cloud.google.com/service-infrastructure/docs/service-networking/reference/rest), to allow us to automatically manage networking.
- [Kubernetes Engine API](https://console.developers.google.com/apis/api/container.googleapis.com), to allow us to deploy a Google Kubernetes Engine (GKE) cluster.
- [Cloud SQL API](https://console.developers.google.com/apis/api/sqladmin.googleapis.com), to allow us to deploy a managed Cloud SQL PostgreSQL instance.
- [Google Cloud Memorystore for Redis API](https://console.developers.google.com/apis/api/redis.googleapis.com), to allow us to deploy a managed Redis instance.
- [Cloud DNS API](https://console.developers.google.com/apis/api/dns.googleapis.com), to allow us to create the required DNS records for the services.
- [IAM API](https://console.developers.google.com/apis/api/iam.googleapis.com), to allow us to manage IAM roles and service accounts.

This process will fail if you haven't linked the project to a billing account, which enables you to pay for the resources you deploy.