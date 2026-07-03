# Source: https://rasa.com/docs/pro/deploy/ci-cd

On this page

**Continuous Integration (CI)** is the practice of merging code changes frequently and automatically running tests on every commit. **Continuous Deployment (CD)** means automatically deploying integrated changes to a staging or production environment once they pass testing.

For conversational AI projects, this provides several benefits:

- **Frequent Updates**: You can release improvements to your assistant more often, ensuring a shorter feedback loop.
- **Reduced Risk**: Automated tests (e.g., end-to-end tests, unit tests for custom actions) catch errors before they make it to production.
- **Consistent Quality**: By enforcing coding and model training standards in your pipeline, you maintain consistent quality in your assistant’s performance.
- **Faster Iterations**: You can address user feedback quickly and ship updates without waiting on infrequent, manual release cycles.

## CI/CD Best Practices[​](https://rasa.com/docs/pro/deploy/ci-cd/#cicd-best-practices 'Direct link to CI/CD Best Practices')

When setting up CI/CD for a Rasa assistant, keep in mind a few best practices:

1. **Use Version Control**
 
 Keep all conversation flows, prompt configurations, and custom actions in version control (e.g., Git). This way, any changes to flows, patterns, or the command generator prompt can be tracked and rolled back if needed.
 
2. **Automated Testing**
 
 Run [e2e conversation-level tests](https://rasa.com/docs/pro/testing/evaluating-assistant/) to ensure your assistant responds correctly to user messages, triggers the right flows, and properly handles conversation repair patterns.
 
3. **Environment Parity**
 
 Try to keep development, staging, and production environments as similar as possible. For example, if you’re using Docker, ensure you use the same Docker images for local development and production deployments.
 
4. **Gradual Deployment**
 
 If you have a large user base, consider rolling out changes to a subset of users first (canary or blue-green deployments) to minimize risk.
 
5. **Artifact Storage**
 
 Store trained models and compiled assistant configurations in a stable, versioned repository (e.g., a cloud bucket). This allows you to revert to a previous model if something goes wrong.
 

## How to Set Up CI/CD[​](https://rasa.com/docs/pro/deploy/ci-cd/#how-to-set-up-cicd 'Direct link to How to Set Up CI/CD')

Below is an example of how you might configure a CI/CD pipeline using **GitHub Actions**. You can follow similar steps with other CI/CD tools like GitLab CI, Jenkins, or CircleCI.

1. **Train Your Assistant**
 - Run [`rasa train`](https://rasa.com/docs/reference/api/command-line-interface/#rasa-train) to compile your flows, patterns, and conversation data into a deployable model.
 - In CALM, training compiles your rule-based flows and dialogue stack logic alongside any LLM configuration.
 - Make sure your CI environment is aligned with the same Python, Docker image, and Rasa Pro version used in production.
2. **Run Automated Tests**
 - Spin up your custom action server.
 - Run [end-to-end tests](https://rasa.com/docs/pro/testing/evaluating-assistant/) (`rasa test e2e`) to validate conversation logic (flows, patterns, slot fillings, and repairs).
 - If tests fail, the pipeline should stop and avoid deploying a broken model.
3. **Deploy to Your Environment**
 - Once your model passes all tests, upload it to a centralized model storage or directly deploy to your staging/production environment.
 - Update your orchestrator (e.g., Kubernetes, Docker Compose) to pull the new model and reload the assistant.

Below is a sample **GitHub Actions** workflow illustrating these steps:

workflow.yml

```
name: Rasa Pro CI/CD Pipelineon:  push:    branches:      - main    paths:      - 'data/**'      - 'domain/**'jobs:  train_test_deploy:    runs-on: ubuntu-latest    steps:      - name: Checkout repository        uses: actions/checkout@v3      # Pull the same Rasa Pro version used in TEST/PROD      - name: Pull Rasa Pro image        run: |          docker pull europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0-latest      # Train the assistant model      - name: Train Rasa model        run: |          docker run -v ${GITHUB_WORKSPACE}:/app \            -e OPENAI_API_KEY=${OPENAI_API_KEY} \            -e RASA_LICENSE=${RASA_LICENSE} \            -e RASA_TELEMETRY_ENABLED=false \            europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0-latest \            train --domain /app/domain --data /app/data --out /app/models        env:          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}          RASA_LICENSE: ${{ secrets.RASA_LICENSE }}      # Start the action server and run E2E tests      - name: Start Action Server and Test Rasa model        run: |          # Spin up your custom action server          docker run -d -p 5055:5055 --name action_server <YOUR-ACTION-SERVER-IMAGE>          # Wait until the action server is ready          echo "Waiting for action server to be ready..."          timeout=60          while ! curl --output /dev/null --silent --fail http://localhost:5055/health; do            printf '.'            sleep 5            timeout=$((timeout-5))            if [ "$timeout" -le 0 ]; then              echo "Action server did not become ready in time."              docker logs action_server              exit 1            fi          done          # Run E2E tests          docker run -v ${GITHUB_WORKSPACE}:/app \            --link action_server:action_server \            -e OPENAI_API_KEY=${OPENAI_API_KEY} \            -e RASA_LICENSE=${RASA_LICENSE} \            -e RASA_TELEMETRY_ENABLED=false \            europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0-latest \            test e2e /app/tests/e2e_test_cases.yml --model /app/models --endpoints /app/endpoints.yml        env:          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}          RASA_LICENSE: ${{ secrets.RASA_LICENSE }}      # Authenticate and upload the trained model to your storage      - name: Authenticate to Google Cloud        uses: google-github-actions/auth@v0.4.0        with:          credentials_json: '${{ secrets.GCP_SA_KEY }}'      - name: Configure Google Cloud project        run: |          echo $GCP_SA_KEY | gcloud auth activate-service-account --key-file=-        env:          GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}      - name: Upload model to Google Cloud Storage        run: |          gsutil cp -r models/* gs://my-model-storage/
```

In this example, the pipeline:

1. **Checks Out** your code.
2. **Pulls** the specified Rasa Pro Docker image.
3. **Trains** the assistant using your domain, data, and flows.
4. **Starts** a custom action server for testing.
5. **Runs** end-to-end tests against the trained model.
6. **Uploads** the successful build to a cloud storage bucket for deployment.

You can adapt this workflow to other CI/CD providers and storage solutions (e.g., S3, Azure Blob Storage, on-prem artifact repositories).a

With a solid CI/CD pipeline, you can rapidly iterate on your CALM-based assistant, ensuring robust, reliable, and up-to-date experiences for your users.

- [CI/CD Best Practices](https://rasa.com/docs/pro/deploy/ci-cd/#cicd-best-practices)
- [How to Set Up CI/CD](https://rasa.com/docs/pro/deploy/ci-cd/#how-to-set-up-cicd)