# Source: https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent

On this page

note

Before you begin, ensure you've finished installing Rasa Pro on Amazon Web Services: [Installation Guide](https://rasa.com/docs/learn/deployment/aws/aws-playbook-intro/)

## Recommended Next Steps[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#recommended-next-steps 'Direct link to Recommended Next Steps')

Now that you have successfully deployed Rasa Pro on AWS, you can proceed with getting a live Rasa agent in place that you can interact with.

**Plan of Action:**

- Confirm Your Agent Runs Locally
- Upload Trained Model to AWS
- Run Your Deployed Rasa Agent

---

## Confirm Your Agent Runs Locally[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#confirm-your-agent-runs-locally 'Direct link to Confirm Your Agent Runs Locally')

If you already have a Rasa agent, make sure it trains and runs successfully on your local machine.

### Don’t Have an Agent Yet?[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#dont-have-an-agent-yet 'Direct link to Don’t Have an Agent Yet?')

If you already have a Rasa agent you’ve built, you can skip this section and go directly to [Upload Trained Model to AWS](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#upload-trained-model-to-aws).

If not, feel free to use one of our pre-built **Starter Packs**. These agents come ready-made for specific industries, and you can run them as-is or customize them for your own use case.

Download the latest release of one of the following:

- [Starter Pack - Insurance](https://github.com/rasa-customers/starterpack-insurance-en/releases)
- [Starter Pack - Retail Banking](https://github.com/rasa-customers/starterpack-retail-banking-en/releases/)
- [Starter Pack - Telecom](https://github.com/rasa-customers/starterpack-telco-en/releases)

### Run Your Starter Pack Locally[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#run-your-starter-pack-locally 'Direct link to Run Your Starter Pack Locally')

Once you have downloaded your Starter Pack, follow the corresponding instructions linked below to train and run your Rasa agent locally:

- [Instructions - Insurance](https://learning.rasa.com/rasa-starter-packs/insurance/)
- [Instructions - Retail Banking](https://learning.rasa.com/rasa-starter-packs/retail-banking/)
- [Instructions - Telecom](https://learning.rasa.com/rasa-starter-packs/telecom/)

note

Make sure your local agent's `endpoints.yaml` matches the one in the deployed agent, so that the agent behaves identically in both environments. You can see the default values set by the deployment playbook [here](https://github.com/RasaHQ/rasa-deployment-playbooks/blob/main/aws/rasa/assistant/values.template.yaml). If you want to change the configuration of models for the deployed agent, change `aws/rasa/assistant/values.yaml` and rerun your `helm upgrade` command.

## Upload Trained Model to AWS[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#upload-trained-model-to-aws 'Direct link to Upload Trained Model to AWS')

After you have successfully run your Rasa agent locally, you can now move it to the cloud to share with others. This consists of two steps:

1. Upload the Rasa Model
2. Restart Rasa Kubernetes Pod

### Upload the Rasa Model[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#upload-the-rasa-model 'Direct link to Upload the Rasa Model')

Navigate to your Rasa project directory

```
cd /path/to/your/rasa/project
```

Ensure you are logged in and [authenticated with AWS](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup/#install-tools)

Upload the model to your bucket

```
aws s3 cp models/<your-model-file.tar.gz> s3://$MODEL_BUCKET/models.tar.gz
```

Verify upload

```
aws s3 ls s3://$MODEL_BUCKET/
```

### Restart Rasa[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#restart-rasa 'Direct link to Restart Rasa')

Restart Rasa to create a new pod which will pull your new model from your bucket.

Authenticate to your AWS cluster

```
aws eks --region $AWS_REGION update-kubeconfig --name $NAME
```

note

If you changed the Kubernetes deployment name from the default `rasa`, use your custom deployment name in place of `rasa` in the commands below.

Restart deployments

```
kubectl -n $NAMESPACE rollout restart deployment rasa
```

To check if the Rasa pod is up and running yet, run the following command and see if the status of the new Rasa pod is "Running".

```
kubectl -n $NAMESPACE rollout status deployment/rasa# orkubectl -n $NAMESPACE get pods -w
```

**Tip:** You can view info, warning and error messages in your pod logs:

View the pod's logs

```
kubectl -n $NAMESPACE logs <your-pod-name>
```

You have now successfully updated the model and restarted your Rasa Pro environment. Next, you’ll set up a simple front-end web page to interact with your agent.

## Interact With Your Rasa Agent[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#interact-with-your-rasa-agent 'Direct link to Interact With Your Rasa Agent')

### Interact With Your Rasa Agent from Your Browser[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#interact-with-your-rasa-agent-from-your-browser 'Direct link to Interact With Your Rasa Agent from Your Browser')

Up to now, you’ve been running and interacting with your Rasa agent **locally** from your browser, connected to your **local instance** of Rasa Pro.

Now, you'll update the connection so your browser connects to your **deployed (cloud) instance** of Rasa Pro. This way, you’ll be interacting with your Rasa agent running in the cloud rather than on your local machine.

1. In your Starter Pack project folder, go to the `chatwidget` directory
2. Open up the `index.html` file in a text editor
3. Replace

```
<rasa-chatbot-widget server-url="http://localhost:5005"
```

with the same domain you set in your `$DOMAIN` environment variable (the domain where your Rasa agent is deployed):

```
<rasa-chatbot-widget server-url="http://REPLACE_WITH_YOUR_DOMAIN"
```

4. Save the changes to `index.html`
5. Open up `index.html` in your browser

**Congratulations! You should now be able to successfully interact with your Rasa agent.**

## Connect Your Agent to Other Channels[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#connect-your-agent-to-other-channels 'Direct link to Connect Your Agent to Other Channels')

In addition to running your agent through the local widget, you can connect it to many different messaging and voice channels - for example, your own website, Slack, Facebook Messenger, or Twilio.

For a full list of supported integrations and setup guides, see the [Rasa documentation on channels](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/).

---

## Additional Resources[​](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#additional-resources 'Direct link to Additional Resources')

Explore these guides and references to deepen your understanding of Rasa and related integrations:

- [Getting Started Tutorial](https://rasa.com/docs/pro/tutorial/)
- [Rasa Learning Platform](https://learning.rasa.com/)
- [Guide to Setting Up CI/CD](https://rasa.com/docs/pro/deploy/ci-cd/)
- [Channel Integrations (Voice & Messaging)](https://rasa.com/docs/reference/channels/messaging-and-voice-channels/)
- [Analytics Setup with Kafka, Metabase, or Langfuse](https://rasa.com/docs/reference/integrations/analytics/)

- [Recommended Next Steps](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#recommended-next-steps)
- [Confirm Your Agent Runs Locally](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#confirm-your-agent-runs-locally)
 - [Don’t Have an Agent Yet?](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#dont-have-an-agent-yet)
 - [Run Your Starter Pack Locally](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#run-your-starter-pack-locally)
- [Upload Trained Model to AWS](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#upload-trained-model-to-aws)
 - [Upload the Rasa Model](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#upload-the-rasa-model)
 - [Restart Rasa](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#restart-rasa)
- [Interact With Your Rasa Agent](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#interact-with-your-rasa-agent)
 - [Interact With Your Rasa Agent from Your Browser](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#interact-with-your-rasa-agent-from-your-browser)
- [Connect Your Agent to Other Channels](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#connect-your-agent-to-other-channels)
- [Additional Resources](https://rasa.com/docs/learn/deployment/aws/aws-playbook-setup-agent/#additional-resources)