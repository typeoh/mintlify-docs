# Source: https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model

On this page

New in 3.10

This page relates to the [fine-tuning recipe](https://rasa.com/docs/pro/customize/fine-tuning-llm/), which is a **beta** feature available starting with version `3.10.0`

After you have fine-tuned a base language model for the task of command generation, you can deploy it to be used by a CALM assistant.

The deployed fine-tuned model must be accessible by the assistant via an [litellm compatible API](https://docs.litellm.ai/docs/providers).

It is highly recommended that you use the [vLLM](https://docs.vllm.ai/en/stable/index.html) model serving library, as it provides a number of important features such as batching and prefix caching.

This page gives an overview on the requirements and options for deploying a fine-tuned model in different environments with links to more detailed guides in our reference section. The same instructions can also be used to deploy any language model from [Hugging Face](https://huggingface.co/models), not only ones that you have fine-tuned.

info

After deploying your model, you will have to [update your assistant's config](https://rasa.com/docs/reference/config/components/llm-configuration/#self-hosted-model-server) for it to use the model for command generation.

## Hardware Requirements[​](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#hardware-requirements 'Direct link to Hardware Requirements')

Hosting large language models requires a GPU to achieve fast inference latency.

If you are using a language model with billions of parameters, such as Llama 3.1, it is highly recommended that you use a GPU with a relatively large amount of memory. For example, an 8 Billion parameter model needs at least 16GB of GPU memory to fit the weights when using 16-bit precision. On top of that, some memory is required for processing one or multiple queries at once, and for caching activations from prior queries (prefix caching) to speed up processing of similar queries. In our experience, a GPU with 40GB of memory provides enough memory to operate a 8B parameter model at scale.

Suitable GPUs for low-latency inference:

- A100 (40GB/80GB)
- L40S (48GB)
- H100 (80GB)

## Expected Throughput and Latency[​](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#expected-throughput-and-latency 'Direct link to Expected Throughput and Latency')

With one of the recommended GPUs you can process around 5 requests per second for command generation at a median response time below 500ms. Note that requests per second is not equal to the number of users. For example, in a voice bot, a user might only interact with the system every 10 seconds because the interaction loop involves user speaking, processing, system responding, user listening, deciding and speaking again. Thus, 5 requests per second would equate to around 50 concurrent users.

info

The exact latency and throughput numbers will depend on the size of your prompt, which in turn is influenced by things such as the number of flows, their description length, whether you are using flow retrieval, etc.

Also take a look at our [reference section on further optimizations](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#optimizations)

## Options for deploying the fine-tuned model[​](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#options-for-deploying-the-fine-tuned-model 'Direct link to Options for deploying the fine-tuned model')

The following options show different scenarios for using the [vLLM](https://docs.vllm.ai/en/stable/index.html) model serving library with your fine-tuned model.

- If you like to serve the model for testing purposes right on the machine where you trained it, you can [use vLLM directly](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-vllm-directly)
- You can also [use docker to start vLLM on a VM](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-vllm-via-docker).
- For production use you can [deploy the LLM to a kubernetes cluster](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#kubernetes-cluster)
- If you are on AWS, [Sagemaker Inference Endpoints](https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/sagemaker-deploy-models.html) are another option for production use. Try them out with our [guide in the reference section](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#sagemaker-inference-endpoints).

- [Hardware Requirements](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#hardware-requirements)
- [Expected Throughput and Latency](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#expected-throughput-and-latency)
- [Options for deploying the fine-tuned model](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#options-for-deploying-the-fine-tuned-model)