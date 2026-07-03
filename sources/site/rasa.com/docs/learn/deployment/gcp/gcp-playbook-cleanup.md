# Source: https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-cleanup

warning

Following these steps will delete all of the infrastructure and configuration you have deployed in all the previous steps. You'll need to set the environment variables defined in [Setup Steps](https://rasa.com/docs/learn/deployment/gcp/gcp-playbook-setup/) before you begin.

You should only perform this step after you're finished with the Rasa Platform and wish to destroy all of the infrastructure you've deployed.

If you would like to clean up everything created in this playbook, you can run the following script. It will delete all of the Kubernetes resources created on your cluster, and then tear down and permanently delete all of the GCP infrastructure. Only run this if you're absolutely sure you wish to delete everything. The command may take up to an hour to run to completion if you deployed all of the resources in the playbook.

```
./gcp/cleanup/cleanup.sh
```