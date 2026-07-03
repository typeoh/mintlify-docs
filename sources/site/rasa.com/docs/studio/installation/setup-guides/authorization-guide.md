# Source: https://rasa.com/docs/studio/installation/setup-guides/authorization-guide

On this page

Studio uses [Keycloak](https://www.keycloak.org/) to manage user authentication, roles, and permissions. This guide explains how to set up user roles for your team, including two main authentication options:

1. **[Simple Authentication](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#simple-authentication-setup)**: Users log in with a username and password.
2. **[Single Sign On](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#sso-setup)**: Centralized login using an identity provider.

## Roles Overview[​](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#roles-overview 'Direct link to Roles Overview')

Studio includes eight default roles to tailor access levels to your team's needs. You can choose which ones make the most sense for your team and organization:

- **SuperUser**: Oversees all of Studio’s functionality — from configuring settings to building the assistant and reviewing conversations.
- **Lead Annotator**: Oversees and reviews annotations, manages CMS content.
- **Annotator**: Annotates data and creates NLU annotations.
- **Flow Builder**: Designs conversational flows and manages NLU data.
- **NLU Editor**: Creates and edits NLU models for training.
- **Business User**: Tests assistants and interacts with flows for business insights.
- **Developer**: Handles technical tasks like exporting annotations and configuring settings.
- **Conversation Analyst**: Analyzes conversation data and manages tags.
- **Conversation Viewer**: Views conversation data and tags.

---

## Simple Authentication Setup[​](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#simple-authentication-setup 'Direct link to Simple Authentication Setup')

Follow these steps to set up users with username/password login:

1. **Log in to Keycloak**: Navigate to `https://<your-studio-url>/auth` and log in using admin credentials (`KEYCLOAK_ADMIN_USERNAME` and `KEYCLOAK_ADMIN_PASSWORD`).
 
 ![Admin Console](https://rasa.com/docs/assets/images/keycloak-admin-console-91945df85a77d7b6699b67867705ee1f.png)
 
2. **Select the Realm**: Choose the `rasa-studio` realm from the dropdown menu.
 
 ![Realm Selection](https://rasa.com/docs/assets/images/keycloak-realm-change-59e7f8e1dc13667e9783ddd1347daef5.png)
 
3. **Add a New User**:
 
 - Navigate to `Users` > `Add user`.
 - Enter user details and click **Create**.
 
 ![Add User](https://rasa.com/docs/assets/images/keycloak-add-user-bb2a1d871fac57d6fc14123907fbe92a.png)
 
4. **Assign Roles**:
 
 - Go to the `Groups` tab and add the user to the relevant groups to assign roles. ![Assign Groups](https://rasa.com/docs/assets/images/keycloak-group-selection-9a4cb3191097a426cbccfef6f45042ed.png)
5. **Set the Password**:
 
 - Go to `Credentials` and set a password.
 - Enable the "Temporary password" toggle if the user needs to reset their password on first login.
 
 ![Set Password](https://rasa.com/docs/assets/images/keycloak-set-password-model-6271dc406c8609ac0c6efd6c84b87a6c.png)
 

## SSO Setup[​](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#sso-setup 'Direct link to SSO Setup')

To configure SSO for your users:

1. **Log in to Keycloak**: Access the `Administration Console` and select the `rasa-studio` realm.
 
2. **Configure Identity Providers**:
 
 - Navigate to the `Identity Providers` section.
 - Select and configure your desired provider (e.g., Google, Azure AD).
 
 ![Identity Providers](https://rasa.com/docs/assets/images/keycloak-identity-providers-14a374132c1af1975491a62a226c812a.png)
 
3. **Follow Provider Instructions**: Refer to [Keycloak SSO Documentation](https://www.keycloak.org/docs/latest/server_admin/#sso-protocols) for specific setup steps.
 
 You can read more details on authorization in our [API Authorization Guide](https://rasa.com/docs/studio/security/authorization/) or [Managing Users Guide](https://rasa.com/docs/studio/security/managing-users/).
 

- [Roles Overview](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#roles-overview)
- [Simple Authentication Setup](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#simple-authentication-setup)
- [SSO Setup](https://rasa.com/docs/studio/installation/setup-guides/authorization-guide/#sso-setup)