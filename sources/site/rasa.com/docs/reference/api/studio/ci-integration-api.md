# Source: https://rasa.com/docs/reference/api/studio/ci-integration-api

## Introduction[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#introduction 'Direct link to Introduction')

The Model Download API provides external access to retrieve download URLs for artifacts generated in Rasa Studio, facilitating CI/CD integration. Built with GraphQL, this API allows queries to request trained assistant models and associated input data in YAML format.

## Requirements[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#requirements 'Direct link to Requirements')

- Required API Role `Model download urls`. See [Authorization](https://rasa.com/docs/reference/api/studio/api-authorization/) to learn how to get an access token.
 
 ![image](https://rasa.com/docs/assets/images/keycloak-service-accounts-roles-model-download-urls-1e4763dfe193c42c7a2a0d516f60dcfe.png)
 
- Assistant API ID. See [authorization guide for configuration](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/) for more details.
 
 ![image](https://rasa.com/docs/assets/images/studio-assistant-api-id-4682d8d54a8541034f56128bf55e3dea.png)
 
- Assistant Version Name. See [user guide for training](https://rasa.com/docs/studio/test/train-your-assistant/) for more details.
 
 ![image](https://rasa.com/docs/assets/images/studio-assistant-version-name-7fa4713ebc779eafef850f514812ee4d.png)
 

## Available APIs[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#available-apis 'Direct link to Available APIs')

### Query: `ModelDownloadUrls`[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#query-modeldownloadurls 'Direct link to query-modeldownloadurls')

**Schema:**

```
  type Query {    modelDownloadUrls(input: ModelDownloadUrlsInput!): ModelDownloadUrlsOutput!  }  input ModelDownloadUrlsInput {    assistantId: ID!    assistantVersionName: ID  }  type ModelDownloadUrlsOutput {    modelFileUrl: String!    modelInputUrl: String!  }
```

#### Input: `ModelDownloadUrlsInput`[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#input-modeldownloadurlsinput 'Direct link to input-modeldownloadurlsinput')

| Field | Description |
| --- | --- |
| `assistantId` | Unique identifier of the assistant. |
| `assistantVersionName` | Optional version of the trained assistant. Default is `latest`. |

#### Output: `ModelDownloadUrlsOutput`[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#output-modeldownloadurlsoutput 'Direct link to output-modeldownloadurlsoutput')

| Field | Description |
| --- | --- |
| `modelFileUrl` | Url to download the model file `<assistant-version-name>.tar.gz`. |
| `modelInputUrl` | Url to download the model input data in YAML format. |

## Usage[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#usage 'Direct link to Usage')

The query can be used to fetch the download URLs for the trained model and the input data in YAML format for a assistant.

### Example `curl` Request[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#example-curl-request 'Direct link to example-curl-request')

Request the Model Download API using e.g. `curl`:

```
curl -X POST https://<api-url>/api/graphql \-H "Content-Type: application/json" \-H "Authorization: Bearer <access-token>" \-d '{    "query": "query ModelDownloadUrls($input: ModelDownloadUrlsInput!) { modelDownloadUrls(input: $input) { modelFileUrl modelInputUrl } }",    "variables": {      "input": {        "assistantId": "<assistant-id>",        "assistantVersionName": "<assistant-version-name>"      }    }  }'
```

### Example Response[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#example-response 'Direct link to Example Response')

The response will contain the download URLs for the trained model and the model input data.

```
{"data":  {"modelDownloadUrls":    {      "modelFileUrl":"https://<api-url>/api/v1/download/model-file/<assistant-version-name>",      "modelInputUrl":"https://<api-url>/api/v1/download/model-input/<assistant-version-name>"    }  }}
```

### Download model file[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#download-model-file 'Direct link to Download model file')

To download the model file, request the `modelFileUrl` with authorization header.

```
curl "https://<api-url>/api/v1/download/model-file/<assistant-version-name>" \    -H "Authorization: Bearer <access-token>" \    -O -J
```

The downloaded model file `<assistant-version-name>.tar.gz` can be used to deploy the trained model.

### Download model input data[​](https://rasa.com/docs/reference/api/studio/ci-integration-api/#download-model-input-data 'Direct link to Download model input data')

To download the model input YAML files, request the `modelInputUrl` with authorization header.

```
curl "https://<api-url>/api/v1/download/model-input/<assistant-version-name>" \    -H "Authorization: Bearer <access-token>" \    -O -J
```

The downloaded model input zipped directory `<assistant-version-name>_model-input.tar.gz` contains the model input data in YAML format:

- `data/flows.yaml`
- `data/nlu.yaml`
- `config.yaml`
- `credentials.yaml`
- `domain.yaml`
- `endpoints.yaml`