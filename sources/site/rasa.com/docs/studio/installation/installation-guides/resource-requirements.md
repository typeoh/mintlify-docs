# Source: https://rasa.com/docs/studio/installation/installation-guides/resource-requirements

Below are the recommended CPU and Memory requirements for the individual pods. You can experiment with different values according to your cluster usage metrics and adjust accordingly. These values can be set under the resources section under each component in the values.yaml file

```
resources:  limits:    cpu: 2000m    memory: 2000Mi  requests:    cpu: 1000m    memory: 1000Mi
```

| Pod | CPU | Memory |
| --- | --- | --- |
| backend | 1000m | 2000Mi |
| keycloak | 1000m | 2000Mi |
| web-client | minimal/default | minimal/default |
| event-ingestion | 500m | 1000Mi |
| Rasa model-service | 2000m | 4000Mi |

info

The resource requirements for the `rasa model-service` pod will depend on the number of trainings and active models that are being run. Training of a model normally requires around 800 MB of RAM and 1000m of CPU units. Allocate higher CPU resources if you anticipate a higher number of parallel trainings.

The resource requirements for the `event-ingestion` will depend on the load the production assistant expects. We recommend you to start with the default values and monitor the resource usage in your cluster.