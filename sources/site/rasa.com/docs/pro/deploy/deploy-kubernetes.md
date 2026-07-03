# Source: https://rasa.com/docs/pro/deploy/deploy-kubernetes

On this page

Kubernetes (and OpenShift) provide a reliable way to run containerized applications at scale. With Kubernetes, you can:

- Orchestrate multiple Rasa Pro services (e.g., Rasa core container, Action Server, Rasa Pro Services) on any cloud or on-prem setup.
- Easily scale up or down by adding more replicas.
- Seamlessly manage rolling updates, networking, and load balancing.
- Simplify the deployment of new versions of your assistant.

If you are unfamiliar with Kubernetes or want a fully managed solution, consider [Rasa’s Managed Service](https://rasa.com/product/managed-service/).

## Deployment Requirements[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#deployment-requirements 'Direct link to Deployment Requirements')

Before deploying Rasa Pro on Kubernetes, make sure you have:

1. **A Kubernetes or OpenShift cluster**
 - Many providers (AWS EKS, Azure AKS, GCP, DigitalOcean) offer managed clusters.
 - Ensure you have `kubectl` (for Kubernetes) or `oc` (for OpenShift) installed and connected to your cluster.
2. **Helm CLI (v3.5 or newer)**
 - You’ll need it to install the Rasa Pro Helm chart.
3. **A valid Rasa License**
 - You will pass it as a secret or an environment variable in your deployment.
4. **(Optional) A Model Storage Bucket**
 - If you plan to store or mount your trained models from cloud storage (AWS S3, GCP Storage, or Azure Blob), set this up in advance.
5. **(Optional) A Kafka cluster and Data Warehouse**
 - Required if you plan to deploy Rasa Pro Services for analytics and logging.

If you do not already have the above, see your cloud provider’s documentation for setting up a Kubernetes or OpenShift cluster. For additional details on Rasa Pro environment variables or advanced configuration, refer to the [Reference](https://rasa.com/docs/reference/config/environment-variables/).

## How to Deploy Rasa[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#how-to-deploy-rasa 'Direct link to How to Deploy Rasa')

### 1\. Kubernetes/OpenShift Cluster[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#1-kubernetesopenshift-cluster 'Direct link to 1. Kubernetes/OpenShift Cluster')

- **Confirm connectivity**:
 
 ```
    kubectl version
    ```
 
 Ensure it shows both client and server versions (for OpenShift, use `oc version`).
 
- **Create a dedicated namespace** (recommended):
 
 ```
    kubectl create namespace <your-namespace>kubectl config set-context --current --namespace=<your-namespace>
    ```
 
 This helps isolate your Rasa Pro deployment from other workloads.
 

### 2\. Rasa Pro Helm Chart[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#2-rasa-pro-helm-chart 'Direct link to 2. Rasa Pro Helm Chart')

Rasa provides a Helm chart to simplify deployment. The chart is hosted on a public Artifact Registry.

1. **Download the Helm chart**:
 
 ```
    helm pull oci://europe-west3-docker.pkg.dev/rasa-releases/helm-charts/rasa
    ```
 
 This command downloads a file named `rasa-<version>.tgz`.
 
2. **Check your Helm version**:
 
 ```
    helm version --short
    ```
 
 You need v3.5 or newer.
 

For the complete documentation of the Helm Chart, see [Rasa Pro Helm Chart](https://helm.rasa.com/charts/rasa/).

### 3\. Deploy Rasa Pro[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#3-deploy-rasa-pro 'Direct link to 3. Deploy Rasa Pro')

Below is the minimal workflow for deploying Rasa Pro on Kubernetes or OpenShift using the Helm chart.

#### a) Prepare Secrets[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#a-prepare-secrets 'Direct link to a) Prepare Secrets')

1. **Create a `secrets.yml` file** (or name it as you wish) with the Rasa license and any other secret values you may need (authentication tokens, etc.). Base64-encode your secret values.
 
 secrets.yml
 
 ```
    apiVersion: v1kind: Secretmetadata:  name: rasa-secretstype: Opaquedata:  rasaProLicense: <BASE64ENCODED_LICENSE>  authToken: <BASE64ENCODED_VALUE>  jwtSecret: <BASE64ENCODED_VALUE>
    ```
 
2. **Apply the secrets**:
 
 ```
    kubectl apply -f secrets.yml
    ```
 

#### b) Create a `values.yml` for your deployment[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#b-create-a-valuesyml-for-your-deployment 'Direct link to b-create-a-valuesyml-for-your-deployment')

1. **Minimal `values.yml`** example:
 
 values.yml
 
 ```
    # Rasa Pro Containerrasa:  image:    repository: "europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro"    tag: "3.8.0-latest"  # Additional Rasa configuration can go here.# Disable the Rasa Pro Services container if not neededrasaProServices:  enabled: false
    ```
 
2. **Deploy with Helm**:
 
 ```
    helm install \    --namespace <your-namespace> \    --values values.yml \    <release-name> \    rasa-<version>.tgz
    ```
 
 This starts a Rasa Pro pod. If you need to update any configuration:
 
 ```
    helm upgrade \    --namespace <your-namespace> \    --values values.yml \    <release-name> \    rasa-<version>.tgz
    ```
 
 To remove:
 
 ```
    helm delete <release-name>
    ```
 

### 4\. Model Storage Bucket[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#4-model-storage-bucket 'Direct link to 4. Model Storage Bucket')

To load a trained model from cloud storage:

1. **Set up a bucket** on AWS S3, Azure Blob, or Google Cloud Storage, and upload your trained model.
2. **Mount or configure** the bucket for your Rasa container.

For example, in Google Cloud:

values.yml

```
rasa:  endpoints:    models:      enabled: false  volumes:    - csi:        driver: gcsfuse.csi.storage.gke.io        readOnly: true        volumeAttributes:          bucketName: <YOUR BUCKET NAME>          mountOptions: implicit-dirs,only-dir=<YOUR DIR>      name: rasa-models  volumeMounts:    - name: rasa-models      mountPath: /app/models      readOnly: true  serviceAccount:    create: true    annotations:      iam.gke.io/gcp-service-account: <YOUR SERVICE ACCOUNT EMAIL>    name: "rasa-pro-sa"  podAnnotations:    gke-gcsfuse/volumes: "true"
```

For other cloud platforms or more advanced configurations, see the [Reference](https://rasa.com/docs/reference/config/environment-variables/).

### 5\. Deploy Action Server[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#5-deploy-action-server 'Direct link to 5. Deploy Action Server')

If your assistant uses **Custom Actions**, you can build and deploy a separate Action Server container alongside your Rasa Pro container.

1. **Build your custom action image**:
 
 - Place your Python code in `actions/actions.py`.
 
 - Optionally specify any dependencies in `requirements-actions.txt`.
 
 - Create a `Dockerfile` extending the official `rasa/rasa-sdk` image:
 
 ```
        FROM rasa/rasa-sdk:latestWORKDIR /app# (Optional) for custom dependencies# COPY actions/requirements-actions.txt ./# RUN pip install -r requirements-actions.txtCOPY ./actions /app/actionsUSER 1001
        ```
 
 - Build & push the image to your container registry (e.g., DockerHub, GCR, ECR).
 
2. **Reference your Action Server in `values.yml`**:
 
 values.yml
 
 ```
    rasa:  endpoints:    actionEndpoint:      url: http://action-server:5055/webhookactionServer:  enabled: true  image:    repository: <my-docker-username>/<my-action-repo>    tag: <custom-tag>
    ```
 
3. **Upgrade the Helm release**:
 
 ```
    helm upgrade \    --namespace <your-namespace> \    --values values.yml \    <release-name> \    rasa-<version>.tgz
    ```
 

### 6\. Deploy Rasa Pro Services[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#6-deploy-rasa-pro-services 'Direct link to 6. Deploy Rasa Pro Services')

Rasa Pro Services is an optional container providing analytics, data collection, and other enterprise features. It must connect to:

- A Kafka cluster (production-ready)
- A data warehouse (e.g., PostgreSQL)

New in `rasa-1.3.0` Helm Chart

We have simplified how Rasa Pro Services handle database and Kafka configurations. Previously, these settings were passed as individual environment variables. They are now defined directly in `values.yaml` under structured configuration blocks (database and kafka).

1. **Configure Kafka** for your Rasa Pro container. In your `values.yml`, make sure:
 
 values.yml
 
 ```
    rasa:  settings:    endpoints:      eventBroker:        enabled: true        type: kafka        # other settings as needed...
    ```
 
2. **Enable Rasa Pro Services**:
 
 values.yml
 
 ```
    rasaProServices:    enabled: true    image:        repository: "europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro-services"        tag: "3.6.1-latest"    loggingLevel: "INFO"    useCloudProviderIam:        # -- useCloudProviderIam.enabled specifies whether to use cloud provider IAM for the Rasa Pro Services container.        enabled: false        # -- useCloudProviderIam.provider specifies the cloud provider for the Rasa Pro Services container. Supported value is aws        provider: "aws"        # -- useCloudProviderIam.region specifies the region for IAM authentication. Required if IAM_CLOUD_PROVIDER is set to aws.        region: "us-east-1"    database:        # -- database.enableAwsRdsIam specifies whether to use AWS RDS IAM authentication for the Rasa Pro Services container.        enableAwsRdsIam: false        # -- database.url specifies the URL of the data lake to store analytics data in. Use `hostname` if you use IAM authentication.        url: ""        # -- database.username specifies the username for the data lake to store analytics data in. Required if enableAwsRdsIam is true.        username: ""        # -- database.hostname specifies the hostname of the data lake to store analytics data in. Required if enableAwsRdsIam is true.        hostname: ""        # -- database.port specifies the port for the data lake to store analytics data in. Required if enableAwsRdsIam is true.        port: "5432"        # -- database.databaseName specifies the database name for the data lake to store analytics data in. Required if enableAwsRdsIam is true.        databaseName: ""        # -- database.sslMode specifies the SSL mode for the data lake to store analytics data in. Required if enableAwsRdsIam is true.        sslMode: ""        # -- database.sslCaLocation specifies the SSL CA location for the data lake to store analytics data in. Required if sslMode is verify-full.        sslCaLocation: ""    kafka:        # -- kafka.enableAwsMskIam specifies whether to use AWS MSK IAM authentication for the Rasa Pro Services container.        enableAwsMskIam: false        # -- kafka.brokerAddress specifies the broker address for the Rasa Pro Services container. Required if enableAwsMskIam is true.        brokerAddress: ""        # -- kafka.topic specifies the topic for the Rasa Pro Services container.        topic: "rasa-core-events"        # -- kafka.dlqTopic specifies the DLQ topic fused to publish events that resulted in a processing failure.        dlqTopic: "rasa-analytics-dlq"        # -- kafka.saslMechanism specifies the SASL mechanism for the Rasa Pro Services container. Leave empty if you are using SSL.        saslMechanism: ""        # -- kafka.securityProtocol specifies the security protocol for the Rasa Pro Services container. Supported mechanisms are PLAINTEXT, SASL_PLAINTEXT, SASL_SSL and SSL        securityProtocol: ""        # -- kafka.sslCaLocation specifies the SSL CA location for the Rasa Pro Services container.        sslCaLocation: ""        # -- kafka.sslCertFileLocation specifies the filepath for SSL client Certificate that will be used to connect with Kafka. Required if securityProtocol is SSL.        sslCertFileLocation: ""        # -- kafka.sslKeyFileLocation specifies the filepath for SSL Keyfile that will be used to connect with Kafka. Required if securityProtocol is SSL.        sslKeyFileLocation: ""        # -- kafka.consumerId specifies the consumer ID for the Rasa Pro Services container.        consumerId: "rasa-analytics-group"        # -- kafka.saslUsername specifies the SASL username for the Rasa Pro Services container. Do not set if enableAwsMskIam is true.        saslUsername: ""        # -- kafka.saslPassword specifies the SASL password for the Rasa Pro Services container. Do not set if enableAwsMskIam is true.        saslPassword:            secretName: "rasa-secrets"            secretKey: "kafkaSslPassword"
    ```
 
3. **Upgrade via Helm**:
 
 ```
    helm upgrade \    --namespace <your-namespace> \    --values values.yml \    <release-name> \    rasa-<version>.tgz
    ```
 

You can confirm the Rasa Pro Services pod is running and check the `/healthcheck` endpoint to verify status.

### 7\. Adding Environment Variables[​](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#7-adding-environment-variables 'Direct link to 7. Adding Environment Variables')

You can pass extra environment variables to any container by adding them to `values.yml`. For example, to add environment variables to the Rasa Pro container:

values.yml

```
rasa:  additionalEnv: []
```

If you have sensitive data (e.g., passwords), store them in Kubernetes secrets and reference them in `values.yml`. For example, to add environment variables from a ConfigMap or Secret:

values.yml

```
rasa:    envFrom: []    # - configMapRef:    #     name: my-configmap
```

A full list of available environment variables for Rasa Pro, Rasa Pro Services, and the Rasa Action Server can be found in the [Environment Variables reference](https://rasa.com/docs/reference/config/environment-variables/).

- [Deployment Requirements](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#deployment-requirements)
- [How to Deploy Rasa](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#how-to-deploy-rasa)
 - [1\. Kubernetes/OpenShift Cluster](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#1-kubernetesopenshift-cluster)
 - [2\. Rasa Pro Helm Chart](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#2-rasa-pro-helm-chart)
 - [3\. Deploy Rasa Pro](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#3-deploy-rasa-pro)
 - [4\. Model Storage Bucket](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#4-model-storage-bucket)
 - [5\. Deploy Action Server](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#5-deploy-action-server)
 - [6\. Deploy Rasa Pro Services](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#6-deploy-rasa-pro-services)
 - [7\. Adding Environment Variables](https://rasa.com/docs/pro/deploy/deploy-kubernetes/#7-adding-environment-variables)