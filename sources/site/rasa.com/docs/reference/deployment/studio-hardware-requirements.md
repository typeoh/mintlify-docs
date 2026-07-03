# Source: https://rasa.com/docs/reference/deployment/studio-hardware-requirements

On this page

## Minimum Hardware Requirements[​](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#minimum-hardware-requirements 'Direct link to Minimum Hardware Requirements')

The minimum requirements suggested to run Rasa Studio are:

- 4 vCPU
- 8GB RAM
- 50 GB Disk

If you are installing on [Kubernetes](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/) using our Helm charts, you can optionally customise the resource requests and limits for individual pods using the `values.yaml` file.

## Suggested Infrastructure[​](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#suggested-infrastructure 'Direct link to Suggested Infrastructure')

If you are using [Docker Compose](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide/) to install Studio, we have recommend the following instance types for getting started on major cloud providers.

### AWS[​](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#aws 'Direct link to AWS')

- Instance Type: `t2.xlarge` is the cheapest possible option, but for more consistent performance consider an `m5.xlarge`. Rasa Studio does not support ARM CPUs so avoid any Graviton instance types (those with a `g` suffix).
- Operating System: Ubuntu 22.04 or 24.04
- Disk: GP2 SSD EBS volume

### Azure[​](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#azure 'Direct link to Azure')

- Instance Type: `D4as_v4`
- Operating System: Ubuntu 22.04 or 24.04
- Disk: Premium SSD

### Google Cloud[​](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#google-cloud 'Direct link to Google Cloud')

- Instance Type: `e2-standard-4`
- Operating System: Ubuntu 22.04 or 24.04
- Disk: Balanced Persistent Disk

- [Minimum Hardware Requirements](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#minimum-hardware-requirements)
- [Suggested Infrastructure](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#suggested-infrastructure)
 - [AWS](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#aws)
 - [Azure](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#azure)
 - [Google Cloud](https://rasa.com/docs/reference/deployment/studio-hardware-requirements/#google-cloud)