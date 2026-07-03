# Source: https://rasa.com/docs/studio/security/authorization

On this page

## Introduction[​](https://rasa.com/docs/studio/security/authorization/#introduction 'Direct link to Introduction')

Learn how to obtain the secret and client ID required for authenticating requests to Studio API. API is built using GraphQL, enabling powerful querying and mutations for flexible interaction with Studio API. For authentication, we rely on Keycloak to manage users and secure external communication by using OpenID Connect's "Client Credentials Flow".

### Studio-External Client Overview[​](https://rasa.com/docs/studio/security/authorization/#studio-external-client-overview 'Direct link to Studio-External Client Overview')

The **studio-external** client is a default client in Keycloak for facilitating API integrations with Rasa Studio. This client is pre-configured with roles, ready for use without additional configuration.

Customers can use **studio-external** to:

- Manage conversations via APIs,
- Request urls to artifacts for CI/CD

This flexibility allows for quick integration or custom setups based on specific requirements.

**Note**: The instructions below cover both using the default client and creating a new one. If you decide to use existing **studio-external**, login to Keycloak admin and skip directly to [Obtain Client ID and Secret](https://rasa.com/docs/studio/security/authorization/#obtaining-client-id-and-secret)

## Creating a New Client ID[​](https://rasa.com/docs/studio/security/authorization/#creating-a-new-client-id 'Direct link to Creating a New Client ID')

To create a new Client ID in Keycloak, follow these steps:

1. Go to Keycloak Admin and log in. **Note**: Ensure the **rasa-studio** realm is selected from the top-left dropdown. ![image](https://rasa.com/docs/assets/images/api-keycloak-realm-5cf1f2447dff311bef4433355eb58153.png)
 
2. Navigate to the **Clients** tab and click **Create**.
 
 - Set **Client ID** to a name of you choice.
 - Set **Client type** to **OpenID Connect**.
 - Click **Next**.
 
 ![image](https://rasa.com/docs/assets/images/api-keycloak-create-clientid-d505a87517b1192147a4b462097a0e3a.png) ![image](https://rasa.com/docs/assets/images/api-keycloak-client-type-clientid-00043ce3ca4fd112cec1dd1010974458.png)
 

### Client Capability Configuration[​](https://rasa.com/docs/studio/security/authorization/#client-capability-configuration 'Direct link to Client Capability Configuration')

1. On the **Capability Config** page, enable:
 
 - **Client Authentication**.
 - **Service Account Roles** (for **Client Credentials Flow**).
 
 **Warning**: Keep other settings off.
 
 ![image](https://rasa.com/docs/assets/images/api-keycloak-capability-configuration-3cea37d4faadbc0644fabc54dfeab1eb.png)
 
2. Click **Next**.
 

### Login Settings[​](https://rasa.com/docs/studio/security/authorization/#login-settings 'Direct link to Login Settings')

On this page click **Save** to finish.

### Assigning a Role to the Client[​](https://rasa.com/docs/studio/security/authorization/#assigning-a-role-to-the-client 'Direct link to Assigning a Role to the Client')

1. Go to **Service Accounts Roles** and click on `service-account-studio-external`. **Note:** This service account user may have a different name depending on how you name your client. ![image](https://rasa.com/docs/assets/images/api-keycloak-service-account-3bfb89f22b68f2a931f63ef4ab96ae86.png)
 
2. In the **Role Mapping** tab, assign a role to your Client, e.g. **Manage conversations** to enable managing conversations via the API.
 
 ![image](https://rasa.com/docs/assets/images/api-keycloak-service-account-roles-a8e13a106e6dc5608dbcdd6e4d48dc4b.png) ![image](https://rasa.com/docs/assets/images/api-keycloak-assign-role-dfcd0ffe864f9ca3333c5bdbd00fc5a9.png)
 

### Obtaining Client ID and Secret[​](https://rasa.com/docs/studio/security/authorization/#obtaining-client-id-and-secret 'Direct link to Obtaining Client ID and Secret')

1. Return to the **Clients** tab, make sure the **rasa-studio** realm is selected, select your client, and go to the **Credentials** tab.
 
2. Click on Regenerate next to the Client Secret field to enhance security.
 
3. Make sure to note down the new **Client ID** and **Client Secret** for future use.
 
 ![image](https://rasa.com/docs/assets/images/api-keycloak-clientid-and-secret-b8a8ba65c4f96d9f0351624258d4c4db.png)
 

### Obtain access token[​](https://rasa.com/docs/studio/security/authorization/#obtain-access-token 'Direct link to Obtain access token')

To perform API requests, you must first obtain an access token using a **POST** request.

1. Create a **POST** request to:
 

https://{your-keycloak-address}/auth/realms/rasa-studio/protocol/openid-connect/token

```
For example in local environment, the URL is:```plaintexthttps://localhost:8081/auth/realms/rasa-studio/protocol/openid-connect/token
```

2.  Set **x-www-form-urlencoded** body parameters:
    -   `grant_type`:\*\* client\_credentials\*\*,
    -   `client_id`: your new Client ID name,
    -   `client_secret`: the secret obtained from [Obtain Client ID and Secret](https://rasa.com/docs/studio/security/authorization/#obtaining-client-id-and-secret)

#### Example curl Request[​](https://rasa.com/docs/studio/security/authorization/#example-curl-request 'Direct link to Example curl Request')

```
curl -X POST https://localhost:8081/auth/realms/rasa-studio/protocol/openid-connect/token \-H "Content-Type: application/x-www-form-urlencoded" \-d 'grant_type=client_credentials&client_id=<your-client-id>&client_secret=<your-client-secret>'
```

You should receive an _access\_token_.

Now, using this token as the **Authorization: Bearer _retrieved\_token_** header, you can send your API requests.

- [Introduction](https://rasa.com/docs/studio/security/authorization/#introduction)
 - [Studio-External Client Overview](https://rasa.com/docs/studio/security/authorization/#studio-external-client-overview)
- [Creating a New Client ID](https://rasa.com/docs/studio/security/authorization/#creating-a-new-client-id)
 - [Client Capability Configuration](https://rasa.com/docs/studio/security/authorization/#client-capability-configuration)
 - [Login Settings](https://rasa.com/docs/studio/security/authorization/#login-settings)
 - [Assigning a Role to the Client](https://rasa.com/docs/studio/security/authorization/#assigning-a-role-to-the-client)
 - [Obtaining Client ID and Secret](https://rasa.com/docs/studio/security/authorization/#obtaining-client-id-and-secret)
 - [Obtain access token](https://rasa.com/docs/studio/security/authorization/#obtain-access-token)