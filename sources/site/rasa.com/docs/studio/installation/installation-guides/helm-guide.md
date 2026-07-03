# Source: https://rasa.com/docs/studio/installation/installation-guides/helm-guide

On this page

## Requirements[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#requirements 'Direct link to Requirements')

To deploy Studio with Helm you need:

- **Rasa Account:** An enterprise Rasa account as well as Rasa and Rasa Studio license keys.
- **LLM API Key:** OpenAI API key or credentials for another LLM.
- **Helm:** An installed version of [Helm](https://helm.sh/docs/intro/install/#helm) `>=3.8.0`.
- **Kubernetes Cluster:** A running Kubernetes cluster. To create a cluster, see documentation on [Google Cloud Platform](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster), [AWS](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html), and [Azure](https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster?tabs=azure-cli).
- **kubectl:** A working installation to manage your cluster. To install kubectl, see documentation on [Google Cloud Platform](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#generate_kubeconfig_entry), [AWS](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html), and [Azure](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli#connect-to-the-cluster).

## Get Started[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#get-started 'Direct link to Get Started')

### Step 1: Pull the official Studio helm chart[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-1-pull-the-official-studio-helm-chart 'Direct link to Step 1: Pull the official Studio helm chart')

You can run the below-mentioned command to pull the latest version chart from Rasa's Google Artifact Registry.

```
helm pull oci://europe-west3-docker.pkg.dev/rasa-releases/helm-charts/studio
```

This will download the chart file (for example `studio-1.0.3.tgz`) to your machine. Extract this file to see the chart templates and `values.yaml` file.

For the complete documentation of the Helm Chart, see [Studio Helm Chart](https://helm.rasa.com/charts/studio/).

### Step 2: Create a value file[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-2-create-a-value-file 'Direct link to Step 2: Create a value file')

A value file contains the configuration options and parameters for the Helm chart. You can customize these options based on your requirements. A sample `values.yaml` file is available in the Studio Helm Chart. The file can be seen if you extract the `.tgz` chart file you downloaded from the Google Artifact Registry. Follow these steps to create your value file:

1. Make a copy of `values.yaml` file which can be found inside extracted chart directory. Let us call it `my-values.yaml`. This file will serve as your custom value file.
2. Open `my-values.yaml` in a text editor and modify the configuration options according to your needs. Ensure that you review and update values related to the way your Kubernetes cluster is set up. The chart's readme file will provide you with all the keys that are available with their default values.
3. Save and close the `my-values.yaml` file.

tip

For PostgreSQL database you must [percentage-encode special characters](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding) in any part of your connection URL - including passwords. For example, `p@$$w0rd` becomes `p%40%24%24w0rd`.

### Step 3: Create Kubernetes secrets[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-3-create-kubernetes-secrets 'Direct link to Step 3: Create Kubernetes secrets')

To store sensitive values mentioned in the above section it is recommended to create Kubernetes secrets. To create them

1. Make a copy of the `secrets.yaml` file which can be found inside extracted chart directory. Let us call it `my-secrets.yaml`.
2. Create base64 encoded values of the secrets and update the yaml file with them. You can run `echo -n "my-secret-key" | base64` to quickly create `base64` encoded values of your secrets.
3. Once all the secrets are updated run `kubectl apply -f my-secrets.yaml -n <namespace>`. This creates the Kubernetes secret in the namespace where you plan to deploy Studio.
4. These secrets are then referenced and passed onto the pods inside the cluster when Studio is deployed with `my-values.yaml` file created in the previous step.

- Helm Chart v2.2.1 and above
- Helm Chart v2.1.9 and below

```
DATABASE_PASSWORD: // Database password for the Studio databasesKAFKA_SASL_PASSWORD: // Kakfa password if you use the SASL mechanismKEYCLOAK_ADMIN_PASSWORD: // Password to the admin user interface of Keycloak (Studio's user management system). You should use this credential to login to http://<external-ip-or-hostname>/authKEYCLOAK_API_PASSWORD: // Password to access Keycloak API. Studio uses this internally to communicate with KeycloakRASA_LICENSE_SECRET_KEY: // Rasa license keyOPENAI_API_KEY_SECRET_KEY: // OpenAI API key
```

```
DATABASE_URL: // Database URL of the Studio backend database. Format is `postgresql://${DB_USER}:${DB_PASS}@${DB_HOST}:${DB_PORT}/${DB_NAME}?schema=public`DATABASE_PASSWORD: // Database password for the Studio databasesKAFKA_SASL_PASSWORD: // Kakfa password if you use the SASL mechanismKEYCLOAK_ADMIN_PASSWORD: // Password to the admin user interface of Keycloak (Studio's user management system). You should use this credential to login to http://<external-ip-or-hostname>/authKEYCLOAK_API_PASSWORD: // Password to access Keycloak API. Studio uses this internally to communicate with KeycloakRASA_LICENSE_SECRET_KEY: // Rasa license keyOPENAI_API_KEY_SECRET_KEY: // OpenAI API key
```

### Step 4: Deploy Rasa Studio using Helm[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-4-deploy-rasa-studio-using-helm 'Direct link to Step 4: Deploy Rasa Studio using Helm')

With the Helm chart and value file prepared, you are ready to deploy Rasa Studio in your Kubernetes cluster. Run the following command:

```
helm upgrade <release-name> <path-to-chart-folder> -f <path-to-value-file> -n <namespace> --install
```

`<release-name>`: The name for your Rasa Studio release ie. `rasa-studio`.

`<path-to-chart-folder>`: The path to the extracted folder of the `tgz` file you obtained by downloading the chart from the Google Artifactory registry.

`<path-to-value-file>`: The path to your custom value file `my-values.yaml`

`<namespace>`: The Kubernetes namespace you want to deploy to.

Example command:

```
helm upgrade rasa-studio ./studio -f ./my-values.yaml -n studio --install
```

Helm will begin the deployment process, creating the necessary resources and configurations based on the provided values.

### Step 5: Monitor deployment and access Rasa Studio[​](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-5-monitor-deployment-and-access-rasa-studio 'Direct link to Step 5: Monitor deployment and access Rasa Studio')

Once the deployment is complete, you can monitor the deployment status by running:

```
kubectl get pods -n <namespace>
```

Ensure that all pods are running and ready before accessing Rasa Studio.

You can access Rasa Studio by visiting the URL `https://<external-ip-or-hostname>` in your web browser.

For further information on advanced configuration options and maintenance tasks, refer to the Rasa Studio Helm charts documentation (`README.md` in the downloaded Helm chart and inline comments in `values.yaml` file) and the official [Helm](https://helm.sh/docs/) documentation.

🎉 That's it! You can now proceed to [activate your license](https://rasa.com/docs/studio/installation/setup-guides/license-activation/)

- [Requirements](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#requirements)
- [Get Started](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#get-started)
 - [Step 1: Pull the official Studio helm chart](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-1-pull-the-official-studio-helm-chart)
 - [Step 2: Create a value file](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-2-create-a-value-file)
 - [Step 3: Create Kubernetes secrets](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-3-create-kubernetes-secrets)
 - [Step 4: Deploy Rasa Studio using Helm](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-4-deploy-rasa-studio-using-helm)
 - [Step 5: Monitor deployment and access Rasa Studio](https://rasa.com/docs/studio/installation/installation-guides/helm-guide/#step-5-monitor-deployment-and-access-rasa-studio)