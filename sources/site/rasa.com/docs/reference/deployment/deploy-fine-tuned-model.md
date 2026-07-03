# Source: https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model

On this page

This page provides detailed steps for different options and optimizations of hosting a fine-tuned LLM.

If you are looking for a more high-level overview on deploying LLMs consult the following pages:

- [Deploying Fine-Tuned LLMs Overview](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/)
- [Hardware Requirements](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#hardware-requirements)
- [Expected Throughput and Latency](https://rasa.com/docs/pro/deploy/deploy-fine-tuned-model/#expected-throughput-and-latency)

## Options for deploying the fine-tuned LLMs[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#options-for-deploying-the-fine-tuned-llms 'Direct link to Options for deploying the fine-tuned LLMs')

The following are different scenarios for using the [vLLM](https://docs.vllm.ai/en/stable/index.html) model serving library with your fine-tuned model.

### Using vLLM directly[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-vllm-directly 'Direct link to Using vLLM directly')

A quick option to try out a model right after training is to serve it right on the machine where you trained it.

Assuming that you have already [installed vLLM](https://docs.vllm.ai/en/stable/getting_started/installation.html) and your fine-tuned model files are in a directory called `finetuned_model`, you can start the vLLM server this way:

```
vllm serve finetuned_model --enable-prefix-caching
```

If you run your rasa assistant on that machine too, it will be able to access the port of your vLLM server on `localhost`. The name of the model to be used in API calls, unless overriden with flags, will be `finetuned_model`.

You can add a [model group](https://rasa.com/docs/reference/config/components/llm-configuration/#model-groups) to your `endpoints.yml` referencing this deployment like so:

endpoints.yml

```
model_groups:  - id: vllm    models:      - provider: self-hosted        model: finetuned_model        api_base: http://127.0.0.1:8000/v1        temperature: 0.0
```

Make sure to use this model group id in your rasa `config.yml` and retrain the assistant.

If you would like to run your rasa assistant on a different machine than the vLLM server, there are several different options. You could setup a [reverse proxy](https://nginx.org/) or use ssh local forwarding (`ssh -L 8000:localhost:8000 <your vllm server>`) to forward the port to your local machine. Make sure that you properly protect your service from unauthorized use and that the firewall of the infrastructure surrounding your VM allows you to reach your server through the desired protocol (https / ssh).

Refer to the [official documentation](https://docs.vllm.ai/en/stable/serving/openai_compatible_server.html#command-line-arguments-for-the-server) for all the flags available for the `vllm serve` command.

### Using vLLM via Docker[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-vllm-via-docker 'Direct link to Using vLLM via Docker')

It is recommended that you use the [official vLLM docker image](https://docs.vllm.ai/en/stable/deployment/docker.html) if you want to run a fine-tuned model on a VM instance via docker.

Assuming you have:

- already installed [Docker](https://docs.docker.com/engine/install/) and the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) in a VM instance with an [NVIDIA GPU](https://docs.docker.com/desktop/gpu/)
- the fine-tuned model files available on the instance in a directory called `finetuned_model` You can start the vLLM server on that instance using the `docker` command below.

```
docker run --runtime nvidia --gpus all \    -v "./finetuned_model:/mnt/finetuned_model" \    -p 8000:8000 --ipc="host" \    vllm/vllm-openai:latest \    --model "/mnt/finetuned_model" \    --enable-prefix-caching
```

You will then need to expose the vLLM server port to the outside world so your assistant can access it via a public IP address. Make sure that you properly protect your service from unauthorized use.

Make sure to use this model group id in your rasa `config.yml` and retrain the assistant.

### Kubernetes cluster[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#kubernetes-cluster 'Direct link to Kubernetes cluster')

It is recommended that you use the [KServe](https://github.com/kserve/kserve) model inference platform when deploying a fine-tuned model to a [Kubernetes](https://kubernetes.io) cluster for use in production, due to the [integrated vLLM runtime](https://docs.vllm.ai/en/v0.5.5/serving/deploying_with_kserve.html).

It is also recommended that you put your model files in a cloud object store, as KServe can [automatically download models from buckets](https://kserve.github.io/website/docs/model-serving/storage/storage-containers). You must first configure KServe with the credentials for your cloud storage provider, such as with [Amazon S3](https://kserve.github.io/website/docs/model-serving/storage/providers/s3) or with [Google Cloud Storage](https://kserve.github.io/website/docs/model-serving/storage/providers/gcs).

Assuming that your cluster already has [KServe installed](https://kserve.github.io/website/docs/admin-guide/serverless) and has at least one [NVIDIA GPU node with appropriate drivers](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/), you can use the manifest below to deploy a fine-tuned model from a bucket. First, make sure to update the values of:

- the `metadata.name` field with a unique name for your model inference service. It's used for the internal DNS within the Kubernetes cluster.
- the `STORAGE_URI` environment variable with the cloud storage URI for your model
- the `--served-model-name` flag with the model name to be used when calling the OpenAI API

values.yml

```
apiVersion: serving.kserve.io/v1beta1kind: InferenceServicemetadata:  name: "CHANGEME-SERVICE-NAME"spec:  predictor:    containers:    - name: "main"      image: "kserve/vllmserver:latest"      command:      - "python3"      - "-m"      - "vllm.entrypoints.openai.api_server"      args:      - "--port"      - "8000"      - "--model"      - "/mnt/models"      - "--served-model-name"      - "CHANGEME-MODEL-NAME"      - "--enable-prefix-caching"      env:      - name: "STORAGE_URI"        value: "CHANGEME-STORAGE-URI"      resources:        limits:          nvidia.com/gpu: "1"
```

If your CALM assistant is deployed in the same cluster as your fine-tuned model, you can exploit the [Kubernetes DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) and use the internal URI of your inference service in your assistant config. Otherwise, you will have to set up your own [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) and use the external IP of your service.

### Sagemaker Inference Endpoints[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#sagemaker-inference-endpoints 'Direct link to Sagemaker Inference Endpoints')

This is a recipe for the prototypical use of sagemaker inference endpoints that involves a number of manual steps.

For production deployments, we recommend using [aws cdk](https://aws.amazon.com/cdk/) to define roles and resources as code instead.

#### Prerequisites[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#prerequisites 'Direct link to Prerequisites')

To create an inference endpoint you need to have the appropriate permissions and quotas. First, create an [aws role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) with full access to sage maker.

Second, make sure that your user account has permissions 1) to assume the formerly created role and 2) to stop sagemaker endpoints.

Third, make sure that you have the quota for using your desired instance type as a sagemaker inference endpoint. There are different quotas for using instances in different AWS services such as VMs in ec2 and as sagemaker inference endpoints. Make sure that you increase the one for sagemaker inference endpoints. For example, if you want to use a `ml.g6e.2xlarge` instance as a sagemaker endpoint, there is a specific quota for that "ml.g6e.2xlarge for endpoint usage" which you can set in the service quotas section.

#### Starting an Endpoint[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#starting-an-endpoint 'Direct link to Starting an Endpoint')

Once you have taken care of the requirements, you can use the following python script to start a sagemaker inference endpoint with a model that has been uploaded to hugging face. It uses the [sagemaker python sdk](https://sagemaker.readthedocs.io/en/stable/installation.html) to start the endpoint.

Make sure that your AWS credentials and are in the environment when running the script. If you want to use a gated model from hugging face, also add the huggingface token.

```
import osimport sagemakerimport boto3from sagemaker.djl_inference.model import DJLModeliam = boto3.client('iam')role = iam.get_role(RoleName='THE-ROLE-YOU-CREATED-EARLIER')['Role']['Arn']session = sagemaker.session.Session()# The model you want the endpoint to servemodel_id = "meta-llama/Llama-3.1-8B-Instruct"# add a HF token to the env vars if the model you want to host is gatedhf_token = os.environ.get("HF_TOKEN")# vllm arguments are prefixed with OPTIONenv = {    "OPTION_ENABLE_PREFIX_CACHING": "true",}# Using aws DJL containers that come prepackaged with vLLM for large model inference# Updated containers can be found here https://github.com/aws/deep-learning-containers/blob/master/available_images.md# make sure that you to replace the region part with your desired region as cross-region image pulls fail.model = DJLModel(    model_id=model_id,    env=env,    role=role,    image_uri="763104351884.dkr.ecr.eu-central-1.amazonaws.com/djl-inference:0.33.0-lmi15.0.0-cu128",    huggingface_hub_token=hf_token)# Instance type, here an instance with a single L40S gpuinstance_type = "ml.g6e.2xlarge"endpoint_name = sagemaker.utils.name_from_base("lmi-model")# deploy model to SageMaker Inferencepredictor = model.deploy(    initial_instance_count=1,    endpoint_name=endpoint_name,    instance_type=instance_type,    container_startup_health_check_timeout=300,)
```

This script will show you the name of the endpoint that was just created. It will be something like `lmi-model-TIMESTAMP`, e.g. `lmi-model-2025-07-25-09-52-15-821`. You need to use it to tell rasa where to find your model in the next section.

#### Using an Endpoint[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-an-endpoint 'Direct link to Using an Endpoint')

In your rasa config, make sure that you add a model group to your `endpoints.yml`:

```
model_groups:  - id: sagemaker_ft    models:      - provider: sagemaker_chat        model: "sagemaker_chat/lmi-model-2025-07-25-09-52-15-821"        temperature: 0.0        timeout: 10.0
```

Afterward, make sure that you reference this model group in your `config.yml` and retrain your assistant. Also, add your AWS credentials to a `.env` file or export them. This way AWS will find the endpoint in your account and make it possible for you to use it.

info

The `sagemaker_chat` prefix to your model name is very important here, as it tells rasa to use the endpoint in chat format, i.e. it makes sure vLLM adds the needed formatting tokens to the prompt. Just using `sagemaker` as a prefix will result in using the model without these important chat formatting tokens and will result in unusable responses.

#### Stopping an Endpoint[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#stopping-an-endpoint 'Direct link to Stopping an Endpoint')

To stop an endpoint navigate to the sagemaker service in AWS, select inference, select endpoints, select your desired endpoint and delete it. The model and endpoint configs do not necessarily need to be deleted.

#### Debugging an Endpoint[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#debugging-an-endpoint 'Direct link to Debugging an Endpoint')

Sometimes, especially when trying out new configuration options, things fail. You can access the sagemaker logs by opening the endpoint details in the AWS interface (sagemaker -> inference -> endpoints) and selecting the endpoints cloud watch logs.

## Optimizations[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#optimizations 'Direct link to Optimizations')

### Prefix Caching[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#prefix-caching 'Direct link to Prefix Caching')

Prefix caching allows you to save compute on prompts that have the same start (=prefix). For all the tokens from the beginning on the prompt that are equal to a prompt that was seen before, previous computation can be reused; thereby saving time for the processing of the prompt.

This optimization is really important for command generation because the prompts have a large overlapping section at the beginning. We recommend that you turn it on by default.

For an indepth look into prefix caching, take a look at [this section in the vLLM docs](https://docs.vllm.ai/en/stable/design/prefix_caching.html)

### 8-bit loading[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#8-bit-loading 'Direct link to 8-bit loading')

vLLM allows you to dynamically quantize 16-bit models to 8-bit at load time. This allows you to gain further speedup at the cost of a little accuracy. In our experiments we saw around 1% loss of accuracy. The exact numbers will depend on your use case and even on your hardware. Make sure that the GPU you use has fp8 cores to fully make use of the quantization. For example, L40S and H100 GPUs have dedicated fp8 cores, which you can see in their specs while the A100 GPU does not. That can make fp8 slower on A100 than fp16.

To load a model in fp8 mode add the parameter `--quantization fp8` to your vLLM starting parameters.

Read more at [this section in the vLLM docs](https://docs.vllm.ai/en/v0.9.2/features/quantization/fp8.html#online-dynamic-quantization).

### 4-bit loading[​](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#4-bit-loading 'Direct link to 4-bit loading')

Loading your model in 4-bit mode can theoretically speed up the model even further at the cost of some more accuracy. You need to make sure that the [bitsandbytes library](https://pypi.org/project/bitsandbytes/) is installed to use this feature. However, the direct hardware support for this fp-4 mode is quite limited and thus can make it more of a memory than a latency optimization.

To load a model in 4-bit mode add the parameter `--quantization bitsandbytes` to your vLLM starting parameters.

Read more at [this section in the vLLM docs](https://docs.vllm.ai/en/v0.9.2/features/quantization/bnb.html).

- [Options for deploying the fine-tuned LLMs](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#options-for-deploying-the-fine-tuned-llms)
 - [Using vLLM directly](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-vllm-directly)
 - [Using vLLM via Docker](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#using-vllm-via-docker)
 - [Kubernetes cluster](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#kubernetes-cluster)
 - [Sagemaker Inference Endpoints](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#sagemaker-inference-endpoints)
- [Optimizations](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#optimizations)
 - [Prefix Caching](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#prefix-caching)
 - [8-bit loading](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#8-bit-loading)
 - [4-bit loading](https://rasa.com/docs/reference/deployment/deploy-fine-tuned-model/#4-bit-loading)