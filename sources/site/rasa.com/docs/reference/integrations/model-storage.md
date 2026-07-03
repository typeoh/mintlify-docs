# Source: https://rasa.com/docs/reference/integrations/model-storage

On this page

You can load your trained model in three different ways:

1. Load the model from your local disk (see [Load Model from Disk](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-disk))
 
2. Fetch the model from your own HTTP server (see [Load Model from Server](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-server))
 
3. Fetch the model from cloud storage like S3 (see [Load Model from Cloud](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-cloud))
 

By default all commands of Rasa's CLI will load models from your local disk.

## Load Model from Disk[​](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-disk 'Direct link to Load Model from Disk')

By default, models will be loaded from your local disk. You can specify the path to your model with the `--model` parameter:

```
rasa run --model models/20190506-100418.tar.gz
```

If you want to load the latest model in a directory, you can specify a directory instead of a file:

```
rasa run --model models/
```

Rasa will check all the models in that directory and load the one that was trained most recently.

If you don't specify a `--model` argument, Rasa will look for models in the `models/` directory. The two following calls will load the same model:

```
# this command will load the same modelrasa run --model models/# ... as this command (using defaults)rasa run
```

## Load Model from Server[​](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-server 'Direct link to Load Model from Server')

You can configure the Rasa server to regularly fetch a model from a server and deploy it.

### How to Configure Rasa[​](https://rasa.com/docs/reference/integrations/model-storage/#how-to-configure-rasa 'Direct link to How to Configure Rasa')

You can configure the HTTP server to fetch models from another URL by adding it to your `endpoints.yml`:

endpoints.yml

```
models:  url: http://my-server.com/models/default  wait_time_between_pulls: 10   # In seconds, optional, default: 100
```

The server will query the `url` for a zipped model every `wait_time_between_pulls` seconds.

If you want to pull the model only when starting up the server, you can set the time between pulls to `null`:

endpoints.yml

```
models:  url: http://my-server.com/models/default  wait_time_between_pulls: null  # fetches model only once
```

### How to Configure Your Server[​](https://rasa.com/docs/reference/integrations/model-storage/#how-to-configure-your-server 'Direct link to How to Configure Your Server')

Rasa will send a `GET` request to the URL you specified in the `endpoints.yml`, e.g. `http://my-server.com/models/default` in the above examples. You can use any URL. The `GET` request will contain an `If-None-Match` header that contains the model hash of the last model it downloaded. An example request from Rasa Open Source to your server would look like this:

```
curl --header "If-None-Match: d41d8cd98f00b204e9800998ecf8427e" http://my-server.com/models/default
```

The response of your server to this `GET` request should be one of these:

- a status code of `200`, a zipped Rasa Model and set the `ETag` header in the response to the hash of the model.
- a status code of `304` and an empty response if the `If-None-Match` header of the request matches the model you want your server to return.

Rasa uses the `If-None-Match` and `ETag` headers for caching. Setting the headers will avoid re-downloading the same model over and over, saving bandwidth and compute resources.

## Load Model from Cloud[​](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-cloud 'Direct link to Load Model from Cloud')

You can also configure the Rasa server to fetch your model from a remote storage by indicating the `remote-storage` CLI option when starting the Rasa server. This allows you to retrieve your models from a cloud storage service like Amazon S3, Google Cloud Storage, or Azure Storage.

Ensure you always specify the model name with the `--model` parameter when running the Rasa server, for example:

```
rasa run --model 20190506-100418.tar.gz --remote-storage aws
```

We highly recommend using a dedicated cloud storage bucket for your models.

If your model is stored in a sub folder on your cloud storage, you can specify the path to the model using the `--model` parameter. Alternatively, you can also provide the environment variable `REMOTE_STORAGE_PATH` and point to a sub folder within your bucket: note that this latter method is deprecated and will be removed in the next 4.0 major release.

```
rasa run --model path/to/your/model/20190506-100418.tar.gz --remote-storage aws
```

The zipped model will be downloaded from cloud storage, unzipped, and deployed. Rasa supports loading models from:

- [Amazon S3](https://aws.amazon.com/s3/),
- [Google Cloud Storage](https://cloud.google.com/storage/),
- [Azure Storage](https://azure.microsoft.com/services/storage/) and
- custom implementations for [Other Remote Storages](https://rasa.com/docs/reference/integrations/model-storage/#other-remote-storages).

### Amazon S3 Storage[​](https://rasa.com/docs/reference/integrations/model-storage/#amazon-s3-storage 'Direct link to Amazon S3 Storage')

Amazon S3 is supported using the `boto3` package which is already included in Rasa.

For Rasa to be able to authenticate and download the model, you need to set up an authentication method:

- use the required AWS access environment variables:
 - `AWS_ACCESS_KEY_ID`: environment variable containing your AWS S3 access key ID
 - `AWS_SECRET_ACCESS_KEY`: environment variable containing your AWS S3 secret access key
- if you are running Rasa on an AWS service like EC2, you can use an IAM role with the necessary permissions to access the S3 bucket. The IAM role should include the following permissions:
 
 ```
    {  "Version": "2012-10-17",  "Statement": [    {      "Effect": "Allow",      "Action": [        "s3:PutObject",        "s3:GetObject",        "s3:ListBucket"      ],      "Resource": [        "arn:aws:s3:::your-bucket-name",        "arn:aws:s3:::your-bucket-name/*"      ]    }  ]}
    ```
 

Additionally, you should ensure to set the following environment variables before running any command requiring the storage:

- `AWS_DEFAULT_REGION`: environment variable specifying the region of your AWS S3 bucket
 
- `BUCKET_NAME`: environment variable specifying the S3 bucket
 
- `AWS_ENDPOINT_URL`: The complete URL to use for the AWS S3 requests. You need to specify a complete URL (including the "http/https" scheme). For example, you could use one of the [Amazon S3 endpoints](https://docs.aws.amazon.com/general/latest/gr/s3.html#s3_region) documented for the region you are using. If you are using a custom endpoint, you can specify it here. Note that by setting the bucket name to `BUCKET_NAME` environment variable, you should not provide the bucket or object URL to `AWS_ENDPOINT_URL`.
 

Once all environment variables are set, you can start the Rasa server with `remote-storage` option set to `aws`:

```
rasa run --model 20190506-100418.tar.gz --remote-storage aws
```

### Google Cloud Storage[​](https://rasa.com/docs/reference/integrations/model-storage/#google-cloud-storage 'Direct link to Google Cloud Storage')

Google Cloud Storage (GCS) is supported using the `google-cloud-storage` package which is already included in Rasa.

If you are running Rasa on Google App Engine or Compute Engine, the auth credentials are already set up (for the GCS in the same project). In this case, you can skip setting any additional environment variables.

If you are running locally or on a machine outside of GAE or GCE you need to provide the authentication details to Rasa manually:

1. Check out the [GCS documentation](https://cloud.google.com/iam/docs/keys-create-delete#creating) to create a service account key.
2. Set an environment variable called `GOOGLE_APPLICATION_CREDENTIALS` to the path of a service account key file with access to your GCS.
3. Set an environment variable called `BUCKET_NAME` to the name of your GCS bucket.

Once all environment variable is set, you can start the Rasa server with `remote-storage` option set to `gcs`:

```
rasa run --model 20190506-100418.tar.gz --remote-storage gcs
```

### Azure Storage[​](https://rasa.com/docs/reference/integrations/model-storage/#azure-storage 'Direct link to Azure Storage')

Azure Storage is supported using the `azure-storage-blob` package which is already included in Rasa.

For Rasa to be able to authenticate and download the model, you need to set the following environment variables before running any command requiring the storage:

- `AZURE_CONTAINER`: environment variable containing your azure container name
 
- `AZURE_ACCOUNT_NAME`: environment variable containing your azure account name
 
- `AZURE_ACCOUNT_KEY`: environment variable containing your account key
 

Once all environment variables are set, you can start the Rasa server with `remote-storage` option set to `azure`:

```
rasa run --model 20190506-100418.tar.gz --remote-storage azure
```

### Other Remote Storages[​](https://rasa.com/docs/reference/integrations/model-storage/#other-remote-storages 'Direct link to Other Remote Storages')

If you want to use any other Cloud Storage, you can provide your own python implementation of the `rasa.core.persistor.Persistor` class, which must inherit from the `Persistor` interface. This allows you to implement your own logic for retrieving and persisting models to your remote storage.

We recommend you add your custom persistor module to the root of the assistant project before starting the Rasa server, so that Rasa can find your custom persistor class. For example, this is how your project structure could look like:

```
my_assistant_project/├── actions/│   ├── __init__.py│   ├── actions.py├── data/├── models/├── tests/├── domain.yml├── config.yml├── credentials.yml├── endpoints.yml├── remote_storage_persistor.py  # Your custom persistor module
```

You can start the Rasa server with `remote-storage` option set to the module path of your persistor implementation:

```
rasa run --remote-storage <your module>.<class name> --model 20190506-100418.tar.gz
```

If you are running the [Rasa server with Docker](https://rasa.com/docs/pro/installation/docker/), you can mount the custom persistor module path to the container using the `-v` option:

```
docker run -v path/to/your/assistant/folder:/app --env_file=path/to/your/env/file rasa/rasa-pro:<RASA_VERSION> run --remote-storage <your module>.<class name> --model 20190506-100418.tar.gz
```

#### Persistor Interface[​](https://rasa.com/docs/reference/integrations/model-storage/#persistor-interface 'Direct link to Persistor Interface')

The `Persistor` interface requires you to implement the following methods:

```
import abcfrom typing import Optionalclass Persistor(abc.ABC):    @abc.abstractmethod    def _retrieve_tar(self, filename: str, target_path: Optional[str] = None) -> None:        """Downloads a model previously persisted to cloud storage.        Args:            filename: The name of the file in cloud storage.            target_path: The path on disk where the model should be downloaded to.        """        raise NotImplementedError    @abc.abstractmethod    def _persist_tar(self, file_key: str, tar_name: str) -> None:        """Uploads a model to cloud storage.        Args:            file_key: The file path to be persisted in cloud storage.            tar_name: The name of the model that should be uploaded.        """        raise NotImplementedError    @abc.abstractmethod    def _retrieve_tar_size(self, filename: str, target_path: Optional[str] = None) -> int:        """Returns the size of the model that has been persisted to cloud storage."""        raise NotImplementedError
```

#### Example Persistor Implementations[​](https://rasa.com/docs/reference/integrations/model-storage/#example-persistor-implementations 'Direct link to Example Persistor Implementations')

Here are some example custom implementations of the `Persistor` interface for the following remote storage types:

- [JFrog Artifactory](https://rasa.com/docs/reference/integrations/model-storage/#jfrog-artifactory)
- [Sonatype Nexus Repository](https://rasa.com/docs/reference/integrations/model-storage/#sonatype-nexus-repository)

##### JFrog Artifactory[​](https://rasa.com/docs/reference/integrations/model-storage/#jfrog-artifactory 'Direct link to JFrog Artifactory')

You can use the [JFrog Artifactory](https://jfrog.com/artifactory/) as a remote storage for your models. To use it, you can implement the `Persistor` interface:

my\_custom\_persistor.py

```
from tempfile import NamedTemporaryFilefrom typing import Optionalimport requestsfrom rasa.core.persistor import Persistorclass JFrogArtifactoryPersistor(Persistor):    def _retrieve_tar(self, filename: str, target_path: Optional[str] = None) -> None:        """Downloads a model previously persisted to cloud storage.        Args:            filename: The name of the file in cloud storage.            target_path: The path on disk where the model should be downloaded to.        """        # add auth header for JFrog Artifactory        headers = {            "X-JFrog-Art-API": "<replace_with_your_reference_token>"        }        response = requests.get(            f"https://<jfrog_instance_placeholder>/artifactory/<repo_key_placeholder>/{filename}",            headers=headers,        )        if response.status_code != 200:            raise ValueError(                f"Failed to retrieve model {filename}. "                f"Status code: {response.status_code}, Response: {response.text}"            )        with NamedTemporaryFile(dir=target_path, suffix=".tar.gz", delete=False) as file:            file.write(response.content)            file.flush()    def _persist_tar(self, file_key: str, tar_name: str) -> None:        """Uploads a model to cloud storage.        Args:            file_key: The file path to be persisted in cloud storage.            tar_name: The name of the model that should be uploaded.        """        # add auth header for JFrog Artifactory        headers = {            "X-JFrog-Art-API": "<replace_with_your_reference_token>"        }        with open(tar_name, "rb") as file:            data = file.read()            response = requests.put(                f"https://<jfrog_instance_name_placeholder>/artifactory/<repo_key_placeholder>/{file_key}",                headers=headers,                data=data,            )        if response.status_code != 201:            raise ValueError(                f"Failed to persist model {tar_name}. "                f"Status code: {response.status_code}, Response: {response.text}"            )    def _retrieve_tar_size(self, filename: str, target_path: Optional[str] = None) -> int:        """Returns the size of the model that has been persisted to cloud storage."""        # add auth header for JFrog Artifactory        headers = {            "X-JFrog-Art-API": "<replace_with_your_reference_token>"        }        response = requests.head(            f"https://<jfrog_instance_name_placeholder>/artifactory/<repo_key_placeholder>/{filename}",            headers=headers,        )        if response.status_code != 200:            raise ValueError(                f"Failed to retrieve model size for {filename}. "                f"Status code: {response.status_code}, Response: {response.text}"            )        return int(response.headers.get("Content-Length", 0))
```

Then run the Rasa server with the `remote-storage` option set to your custom persistor class. For example, if you have implemented the above custom persistor class called `JFrogArtifactoryPersistor` in a module `my_custom_persistor.py`, you can run the Rasa server like this:

```
rasa run --remote-storage my_custom_persistor.JFrogArtifactoryPersistor --model path/to/<model_name>.tar.gz
```

###### Sonatype Nexus Repository[​](https://rasa.com/docs/reference/integrations/model-storage/#sonatype-nexus-repository 'Direct link to Sonatype Nexus Repository')

You can use the [Sonatype Nexus Repository](https://www.sonatype.com/products/sonatype-nexus-repository) as a remote storage for your models. To use it, you can implement the `Persistor` interface:

my\_custom\_persistor.py

```
import jsonfrom pathlib import Pathfrom tempfile import NamedTemporaryFilefrom typing import Optionalimport requestsfrom rasa.core.persistor import PersistorREPOSITORY = "<repository_name_placeholder>"  # Replace with your Nexus repository nameAUTH = ("username", "password")  # Replace with your Nexus credentialsNEXUS_DOMAIN = "localhost:8081"  # Replace with your Nexus domainclass SonatypeNexusRepositoryPersistor(Persistor):    def _retrieve_tar(self, filename: str, target_path: Optional[str] = None) -> None:        response = requests.get(            f"http://{NEXUS_DOMAIN}/repository/{REPOSITORY}/{filename}",            auth=AUTH,        )        print(f"Downloading {filename}...")        if response.status_code != 200:            raise ValueError(                f"Failed to retrieve model {filename}. "                f"Status code: {response.status_code}, Response: {response.text}"            )        with NamedTemporaryFile(dir=target_path, suffix=".tar.gz", delete=False) as file:            file.write(response.content)            file.flush()    def _persist_tar(self, file_key: str, tar_name: str) -> None:        with open(tar_name, "rb") as file:            data = file.read()            response = requests.put(                f"http://{NEXUS_DOMAIN}/repository/{REPOSITORY}/{tar_name}",                auth=AUTH,                data=data,            )        if response.status_code != 201:            raise ValueError(                f"Failed to persist model {tar_name}. "                f"Status code: {response.status_code}, Response: {response.text}"            )    def _retrieve_tar_size(self, filename: str, target_path: Optional[str] = None) -> int:        response = requests.head(            f"http://{NEXUS_DOMAIN}/repository/{REPOSITORY}/{filename}",            auth=AUTH        )        if response.status_code != 200:            raise ValueError(                f"Failed to retrieve model size for {filename}. "                f"Status code: {response.status_code}, Response: {response.text}"            )        return int(response.headers.get("Content-Length", 0))
```

Then run the Rasa server with the `remote-storage` option set to your custom persistor class. For example, if you have implemented the above custom persistor class called `SonatypeNexusRepositoryPersistor` in a module `my_custom_persistor.py`, you can run the Rasa server like this:

```
rasa run --remote-storage my_custom_persistor.SonatypeNexusRepositoryPersistor --model path/to/<model_name>.tar.gz
```

## Save Model To Cloud[​](https://rasa.com/docs/reference/integrations/model-storage/#save-model-to-cloud 'Direct link to Save Model To Cloud')

New in 3.10

You can now also save models to cloud after training.

You can also configure the Rasa server to upload your model to a remote storage:

```
rasa train --fixed-model-name 20190506-100418.tar.gz --remote-storage aws
```

We highly recommend using a dedicated cloud storage bucket for your models.

note

It is useful to provide a fixed model name while pushing model to remote storage, as you can refer the same name while downloading and running the rasa bot. If no fixed model name is provided, rasa will generate a model name and upload it to remote storage.

Rasa supports uploading models to:

- [Amazon S3](https://aws.amazon.com/s3/),
- [Google Cloud Storage](https://cloud.google.com/storage/),
- [Azure Storage](https://azure.microsoft.com/services/storage/) and
- custom implementations for [Other Remote Storages](https://rasa.com/docs/reference/integrations/model-storage/#other-remote-storages).

You can specify the path on the cloud storage by using the `--out` parameter during training, along with the `--remote-storage` option:

```
rasa train --fixed-model-name 20190506-100418.tar.gz --remote-storage aws --out path/to/models
```

If the `--out` parameter is not specified, the model will be uploaded to the default `/models` directory in the cloud storage bucket.

- [Load Model from Disk](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-disk)
- [Load Model from Server](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-server)
 - [How to Configure Rasa](https://rasa.com/docs/reference/integrations/model-storage/#how-to-configure-rasa)
 - [How to Configure Your Server](https://rasa.com/docs/reference/integrations/model-storage/#how-to-configure-your-server)
- [Load Model from Cloud](https://rasa.com/docs/reference/integrations/model-storage/#load-model-from-cloud)
 - [Amazon S3 Storage](https://rasa.com/docs/reference/integrations/model-storage/#amazon-s3-storage)
 - [Google Cloud Storage](https://rasa.com/docs/reference/integrations/model-storage/#google-cloud-storage)
 - [Azure Storage](https://rasa.com/docs/reference/integrations/model-storage/#azure-storage)
 - [Other Remote Storages](https://rasa.com/docs/reference/integrations/model-storage/#other-remote-storages)
- [Save Model To Cloud](https://rasa.com/docs/reference/integrations/model-storage/#save-model-to-cloud)