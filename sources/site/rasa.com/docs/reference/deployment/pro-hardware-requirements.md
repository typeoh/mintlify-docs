# Source: https://rasa.com/docs/reference/deployment/pro-hardware-requirements

On this page

## Minimum Hardware Requirements[​](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#minimum-hardware-requirements 'Direct link to Minimum Hardware Requirements')

The minimum requirements suggested to run the Rasa Pro server are:

- 8 vCPU
- 16 GB RAM
- 50 GB Disk

These requirements are for a single instance of Rasa Pro server running CALM (Conversational AI Lifecycle Management) only. These will ensure that your assistant can handle on average 24 concurrent requests (i.e. user messages) per second (RPS) with a response time of between 100 and 500 ms, provided you are using multiple Sanic workers (max 8).

If you are installing on [Kubernetes](https://rasa.com/docs/pro/deploy/deploy-kubernetes/) using our Helm charts, you can optionally customise the resource requests and limits for individual pods using the `values.yaml` file.

## Infrastructure Guidelines[​](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#infrastructure-guidelines 'Direct link to Infrastructure Guidelines')

We recommend starting off with these baseline recommendations and then scaling up as needed based on your usage patterns.

### AWS[​](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#aws 'Direct link to AWS')

- Instance Type: `c5.2xlarge`
- Operating System: Ubuntu LTS 22.04 or 24.04
- Disk: GP3 SSD EBS volume

### Azure[​](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#azure 'Direct link to Azure')

- Instance Type: `Fsv2_8s_v2`
- Operating System: Ubuntu LTS 22.04 or 24.04
- Disk: Premium SSD

### Google Cloud[​](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#google-cloud 'Direct link to Google Cloud')

- Instance Type: `c2-standard-8`
- Operating System: Ubuntu LTS 22.04 or 24.04
- Disk: Balanced Persistent Disk

info

The above guidelines are the minimum requirements for a CALM-only assistant.

If you are using additional components such as NLU, PII, or local embeddings, that could require using additional models other than the Rasa trained model, you will need to scale up your infrastructure accordingly.

If you choose to use [multiple Sanic workers](https://rasa.com/docs/reference/config/environment-variables/#advanced-configuration), you should ensure that:

- the number of workers does not exceed the number of CPU cores available on your machine.
- each worker has enough memory allocated to load any additional models. This is particularly important if you are using large models in custom NLU components, local embedding models or [PII management](https://rasa.com/docs/reference/config/pii-management/prerequisites/#gliner-requirements).

- [Minimum Hardware Requirements](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#minimum-hardware-requirements)
- [Infrastructure Guidelines](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#infrastructure-guidelines)
 - [AWS](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#aws)
 - [Azure](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#azure)
 - [Google Cloud](https://rasa.com/docs/reference/deployment/pro-hardware-requirements/#google-cloud)