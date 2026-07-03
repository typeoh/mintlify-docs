# Source: https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio

On this page

warning

Before you begin, ensure you've completed all of the previous sections to deploy required infrastructure and to set environment variables that this section requires.

In the previous step we deployed Kafka and Rasa Pro onto your Kubernetes cluster. We will now deploy Rasa Studio onto the same cluster and connect it to the Kafka broker to allow it to communicate with Rasa Pro. This will involve:

- Configuring the Studio application and its constituent components.
- Configuring Keycloak, the authentication platform that Studio uses.
- Deploying Studio onto your cluster and making it accessible from outside the cluster.

## Studio Setup[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#studio-setup 'Direct link to Studio Setup')

First, we'll perform some initial setup for installing Rasa Studio. This script will:

- Pull the Rasa Studio Helm chart.
- Ensure some secret values are set and create a Kubernetes secret for them.

```
./gcp/studio/setup-studio.sh
```

This script will output the admin credentials for Keycloak at the end, which is the authentication provider you use to manage users for Rasa Studio. Be sure to take note of these credentials as you'll need them later.

### Values File[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#values-file 'Direct link to Values File')

The `values.yaml` file will contain all of the configuration that gets applied to your assistant by Helm. You can read about all of the available configuration options [here](https://github.com/RasaHQ/rasa-helm-charts/tree/main/charts/studio), and you can fully customise your setup to meet your organisation's needs and security policies. For simplicity, we've provided a file in the playbook repo `gcp/studio/values.template.yaml` that sets all of the key values we need to integrate Rasa with the GCP infrastructure we've already deployed. You can set these values manually and add any others you require, but we recommend you begin by automatically creating a version with all the environment variables set from the previous sections:

```
envsubst < gcp/studio/values.template.yaml > gcp/studio/values.yaml
```

If you open this newly created file, you should see that all of your values have been automatically populated from the environment variables we've been setting as we go along.

Now that all the configuration is done, you can deploy Studio using all of the configuration that you've set:

```
helm upgrade --install -n $NAMESPACE studio gcp/studio/repos/studio-helm/studio -f gcp/studio/values.yaml
```

## Configure TLS Certificate[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#configure-tls-certificate 'Direct link to Configure TLS Certificate')

Finally, we can deploy the TLS certificate for your Studio deployment. Rasa Studio deploys its own Ingress, so you don't need to do that like you did with Rasa Pro. Deploy the certificate with:

```
./gcp/studio/deploy-certificate.sh
```

You may see a warning stating `Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0, the default value changed from Never to Always.`. This is fine.

Finally, you will be able to visit your assistant's URL: [https://studio.$DOMAIN](https://studio.$DOMAIN). For example, if the value you used for $DOMAIN was example.com, you could visit your assistant on [https://studio.example.com](https://studio.example.com). If you receive an error, there may just be a delay in issuing the certificate - wait a few minutes and try again! You may see a few different error messages as components finish up deploying and certificates are issued, so don't worry if this happens. Your deployment is complete when you see a login page.

**Congratulations, you have now completed the playbook and successfully installed the Rasa Platform!**

## Next Steps[​](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#next-steps 'Direct link to Next Steps')

- [Set up users and roles for Studio](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/).
- If you ever need to uninstall Rasa Platform and remove the deployed infrastructure, you can do so by following the instructions in the [Cleanup](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-cleanup/) section.

- [Studio Setup](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#studio-setup)
 - [Values File](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#values-file)
- [Configure TLS Certificate](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#configure-tls-certificate)
- [Next Steps](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-studio/#next-steps)