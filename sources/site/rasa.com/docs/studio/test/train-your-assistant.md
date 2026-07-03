# Source: https://rasa.com/docs/studio/test/train-your-assistant

On this page

This guide will walk you through the process of training your assistant in Rasa Studio. By the end, you'll understand how to initiate training, monitor its progress, resolve common errors, and debug issues.

## Before You Start[​](https://rasa.com/docs/studio/test/train-your-assistant/#before-you-start 'Direct link to Before You Start')

Training a model requires specific permissions that are not available to every Studio role. Ensure your role has the [right access](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/) before continuing.

## About Training[​](https://rasa.com/docs/studio/test/train-your-assistant/#about-training 'Direct link to About Training')

Training compiles your flows, NLU data, and other inputs into a functional model, that you can use for testing or deploy to production.

Rasa Studio provides two ways to train your assistant:

**[⭐️ Training in Studio (Recommended)](https://rasa.com/docs/studio/test/train-your-assistant/#training-in-studio)**: 
For rapid iteration and testing your assistant within the Studio UI.

**[Training your assistant locally with Rasa](https://rasa.com/docs/studio/test/train-your-assistant/#training-your-assistant-locally-using-rasa-pro)**: 
Useful for technical users who have additional custom components or advanced integration steps that they want incorporated before training.

note

If you're interested in diving into technical details of how training works see more in our [Training Architecture Reference](https://rasa.com/docs/reference/architecture/studio/#studio-model-service-container).

## Training in Studio[​](https://rasa.com/docs/studio/test/train-your-assistant/#training-in-studio 'Direct link to Training in Studio')

### Get Started[​](https://rasa.com/docs/studio/test/train-your-assistant/#get-started 'Direct link to Get Started')

1. **Access Training**
 
 Navigate to the **Flow builder**. You should be able to see the **"Train"** button:
 
 ![Train](https://rasa.com/docs/assets/images/Train-dfd2838587696f7b668da07334f8e74f.png)
 
2. **Select Flows**
 
 In the **Train** panel, make sure all the flows you want to include in training are selected. Note that NLU, responses and all other primitives saved in Studio are always included in training.
 
 ![Select Flows](https://rasa.com/docs/assets/images/select-flows-6c680497d965de8156704f9dc4e2cdb2.png)
 
3. **Select Flow Version** (optional)
 
 If you're using flow versioning and have saved some flows as "Stable," you’ll also see a switch to select between the "Draft" and "Stable" versions. Choose either the latest stable version or the current draft to include in training. Select the "Stable" version if you don’t want to include the latest changes made to the flow. [Learn more about creating a stable version of flows](https://rasa.com/docs/studio/build/content-management/version-control/#save-a-stable-version).
 
 ![Select Flow Version](https://rasa.com/docs/assets/images/select-flow-version-7945167e5dd23920952092e180955880.png)
 
4. **Train**
 
 Once your flows are ready, click the **"Start training"** button. Studio will validate your flows before beginning the training process. When the training is completed successfully, test the newly created assistant version on [Try your assistant](https://rasa.com/docs/studio/test/try-your-assistant/).
 

### Validate and Improve[​](https://rasa.com/docs/studio/test/train-your-assistant/#validate-and-improve 'Direct link to Validate and Improve')

Rasa Studio validates your flows before the training process begins. This step helps you catch common issues—like missing intents or invalid syntax—early in the process, saving time and effort.

1. **Resolve Validation Errors**
 
 If errors are found during validation, you’ll see an **"Unable to train"** message.
 
 - Click on the flow names in the error list to locate and fix issues.
 - Examples of errors include missing intents, invalid syntax, or unsupported actions.
 
 ![image](https://rasa.com/docs/assets/images/validation-error-62c00fce132c0a8fce1f742a094496c3.png)
 
2. **Monitor Progress**
 
 After validation, Studio will display a training progress message. Other users can view the status but cannot stop or restart training until it’s complete.
 
 ![image](https://rasa.com/docs/assets/images/training-in-progress-761bde9a54ac972f202cdd3b12b24eaf.png)
 

### How to Manage Training Outcomes[​](https://rasa.com/docs/studio/test/train-your-assistant/#how-to-manage-training-outcomes 'Direct link to How to Manage Training Outcomes')

Once validation is complete, training begins automatically. Studio provides details on the outcomes as well as logs to help you debug any issues that might come up.

1. **Handle Training Outcomes**

- **Successful Training**: A green checkmark and success message indicate the model is ready.
 
 ![image](https://rasa.com/docs/assets/images/training-success-51eba0262a976637533fc9e4c6de6882.png)
 
- **Training Failure**: If training fails, download the logs from the **Versions** page to investigate. Typical causes include invalid training data or conflicting rules.
 
 ![image](https://rasa.com/docs/assets/images/download-training-log-60b8e9f3db86d01bcfc456cc1655b900.png)
 

2. **Stop Training (Optional)**
 
 If necessary, the user who initiated training can stop it. This will terminate the process and display a status update.
 
 ![image](https://rasa.com/docs/assets/images/stopped-f09ee8fb34ef8149cbc86c7290846309.png)
 

### Troubleshooting Tips[​](https://rasa.com/docs/studio/test/train-your-assistant/#troubleshooting-tips 'Direct link to Troubleshooting Tips')

1. **Access Logs and Versions**

Go to the **Versions** page to view trained versions of your assistant. Use the ellipsis (three dots) next to a version to download the model or training logs for further analysis.

![image](https://rasa.com/docs/assets/images/failed-b9825da4467ccdddbf6945c1a91be54a.png)

2. **Reproduce Errors Locally**
 - Download the model input files from Studio.
 - Set up a matching local Rasa Pro environment.
 - Run:
 
 ```
        rasa train --debug
        ```
 This will provide detailed error messages to help identify the issue.

## Training Your Assistant Locally using Rasa Pro[​](https://rasa.com/docs/studio/test/train-your-assistant/#training-your-assistant-locally-using-rasa-pro 'Direct link to Training Your Assistant Locally using Rasa Pro')

### Step One: Download your Assistant Files[​](https://rasa.com/docs/studio/test/train-your-assistant/#step-one-download-your-assistant-files 'Direct link to Step One: Download your Assistant Files')

**Option 1:** Downloading via the Studio UI 

- Navigate to the **Versions** page in Studio, locate the version you want to train, and click the ellipsis (three dots) to download the assistant files.
 
 ![download-files](https://rasa.com/docs/assets/images/download-training-log-60b8e9f3db86d01bcfc456cc1655b900.png)
 

**Option 2:** Downloading via the Rasa Pro CLI 

Replace `[assistant-name]` with the name of your assistant.

```
rasa studio download [assistant-name]
```

### Step Two: Train Your Assistant[​](https://rasa.com/docs/studio/test/train-your-assistant/#step-two-train-your-assistant 'Direct link to Step Two: Train Your Assistant')

Place your assistant files in the working directory and run the following command to start training:

```
rasa train
```

For more detailed local training steps using the Rasa Pro you can take a look at the technical reference to [train your assistant locally](https://rasa.com/docs/reference/api/command-line-interface/#rasa-train).

- [Before You Start](https://rasa.com/docs/studio/test/train-your-assistant/#before-you-start)
- [About Training](https://rasa.com/docs/studio/test/train-your-assistant/#about-training)
- [Training in Studio](https://rasa.com/docs/studio/test/train-your-assistant/#training-in-studio)
 - [Get Started](https://rasa.com/docs/studio/test/train-your-assistant/#get-started)
 - [Validate and Improve](https://rasa.com/docs/studio/test/train-your-assistant/#validate-and-improve)
 - [How to Manage Training Outcomes](https://rasa.com/docs/studio/test/train-your-assistant/#how-to-manage-training-outcomes)
 - [Troubleshooting Tips](https://rasa.com/docs/studio/test/train-your-assistant/#troubleshooting-tips)
- [Training Your Assistant Locally using Rasa Pro](https://rasa.com/docs/studio/test/train-your-assistant/#training-your-assistant-locally-using-rasa-pro)
 - [Step One: Download your Assistant Files](https://rasa.com/docs/studio/test/train-your-assistant/#step-one-download-your-assistant-files)
 - [Step Two: Train Your Assistant](https://rasa.com/docs/studio/test/train-your-assistant/#step-two-train-your-assistant)