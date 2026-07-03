# Source: https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s

On this page

warning

Before you begin, ensure you've completed all of the previous sections to deploy required infrastructure and to set environment variables that this section requires.

Now that we've deployed all of the infrastructure into Google Cloud, we'll set up some tools on your Kubernetes cluster:

- Istio, a service mesh that will ensure that communication between different Rasa product components is encrypted in transit.
- ExternalDNS, a tool that will allow the cluster to create the DNS records it needs automatically using GCP Cloud DNS.
- Cert-manager, a tool to ensure we can automatically issue and renew TLS certificates for secure communication.

We'll use scripts from our [deployment-playbooks](https://github.com/RasaHQ/rasa-deployment-playbooks) repo throughout this section to make things easier. If you'd like to understand more about exactly what each script is doing, open it in your IDE of choice and review the comments which explain what is being deployed at each step.

### Cluster Setup[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#cluster-setup 'Direct link to Cluster Setup')

Firstly, we'll set Istio on your cluster, the service mesh that will ensure that communication between different Rasa product components is encrypted in transit, on your cluster.

To deploy the required infrastructure into your GCP environment, use the provided script in our companion repo:

```
./gcp/configure/configure-cluster.sh
```

The final output of this script will instruct you to create an `NS` DNS record for the domain you're using. Check your DNS provider's documentation for more information about how to do this. Once you have updated your DNS registrar with the zone name servers, wait for the DNS propagation. You cam verify if zone is properly delegated to GCP with command:

```
dig +short NS $DOMAIN
```

It should print the same GCP nameservers that were output at the end when you ran `./gcp/configure/configure-cluster.sh` above. If so, you can continue further on with the playbook.

### Install external-dns[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#install-external-dns 'Direct link to Install external-dns')

We will now install external-dns onto our cluster. This tool allows your Kubernetes cluster to create the DNS records it needs automatically using the GCP Cloud DNS zone that you've just configured.

```
./gcp/configure/install-external-dns.sh
```

### Install cert-manager[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#install-cert-manager 'Direct link to Install cert-manager')

We will now install cert-manager which will handle issuing TLS certificates automatically using LetsEncrypt.

```
./gcp/configure/install-cert-manager.sh
```

You may see a warning stating `Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0, the default value changed from Never to Always.`. This is fine.

### Collect Required Parameters[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#collect-required-parameters 'Direct link to Collect Required Parameters')

Most of the infrastructure work is now done, so we'll gather together some info that we'll use later in the playbook to make deploying the Rasa applications a bit easier.

Collect some information about the GCP infrastructure you've deployed and set them as environment variables:

```
source ./gcp/configure/get-infra-values.sh
```

These environment variables are now set and ready to be used in your application configurations.

### Test Ingress Setup[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#test-ingress-setup 'Direct link to Test Ingress Setup')

We've deployed and configured tooling onto the cluster to manage the creation of DNS records and TLS certificates. In this section, we'll test whether it's all working properly by temporarily deploying a tiny HTTP server called HTTPBin to check. If the test fails, the script will suggest some commands you can run to help identify the source of the issue.

```
./gcp/configure/test-ingress.sh
```

You may see a warning stating `Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0, the default value changed from Never to Always.`. This is fine.

- [Cluster Setup](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#cluster-setup)
- [Install external-dns](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#install-external-dns)
- [Install cert-manager](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#install-cert-manager)
- [Collect Required Parameters](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#collect-required-parameters)
- [Test Ingress Setup](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-k8s/#test-ingress-setup)