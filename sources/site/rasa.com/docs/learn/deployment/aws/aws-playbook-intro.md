# Source: https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro

On this page

This playbook outlines an opinionated, best-practice way to install Rasa Pro and Rasa Studio on AWS. You may wish to adapt steps and configuration to meet your needs or organisational policies as required.

The playbook will walk you through the following sections:

#### Setup[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/#setup 'Direct link to Setup')

Set up your local machine to perform the setup and installation, getting us ready to deploy the infrastructure and install applications on top. We'll use our [companion Github repository](https://github.com/RasaHQ/rasa-deployment-playbooks) to make this easier, which contains scripts and sample configuration files.

#### Deploy Infrastructure[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/#deploy-infrastructure 'Direct link to Deploy Infrastructure')

Deploy the actual underlying cloud infrastructure on AWS, including a Kubernetes cluster, PostgreSQL database and associated services like networking and DNS.

#### Configure Cluster[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/#configure-cluster 'Direct link to Configure Cluster')

Configure and install some tools on the Kubernetes cluster, including:

- Istio, a service mesh that will ensure that communication between different Rasa product components is encrypted in transit.
- ExternalDNS, a tool that will allow the cluster to create the DNS records it needs automatically using Amazon Route 53.
- Cert-manager, a tool to ensure we can automatically issue and renew TLS certificates for secure communication.

#### Install Rasa Pro[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/#install-rasa-pro 'Direct link to Install Rasa Pro')

Configure and install Rasa Pro onto the cluster ready for production usage.

#### Install Rasa Studio[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/#install-rasa-studio 'Direct link to Install Rasa Studio')

Configure and install Rasa Studio onto the cluster ready for production usage.

#### Cleanup[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/#cleanup 'Direct link to Cleanup')

Optionally clear up all the infrastructure and services you've deployed through the course of this playbook.

warning

The steps in this playbook will deploy infrastructure in your AWS account meant for hosting a production, scalable deployment of Rasa Pro and Studio. This can be quite expensive - if you're just looking to try out Rasa, you should use our [Docker Compose setup](https://github.com/RasaHQ/developer-edition-docker-compose) instead.