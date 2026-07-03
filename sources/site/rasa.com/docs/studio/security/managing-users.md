# Source: https://rasa.com/docs/studio/security/managing-users

On this page

## Guides for Managing Users[​](https://rasa.com/docs/studio/security/managing-users/#guides-for-managing-users 'Direct link to Guides for Managing Users')

Admins can manage users at any time by performing the following actions:

### Delete a User[​](https://rasa.com/docs/studio/security/managing-users/#delete-a-user 'Direct link to Delete a User')

tip

Do not modify or delete the `realmadmin` user, as it is required for Studio to interact with Keycloak APIs.

1. Go to `Users` and locate the user to delete.
 
2. Open the kebab menu (three dots) and select **Delete**. ![Delete User][base64-image]
 
3. Confirm the deletion.
 

### Update User Permissions[​](https://rasa.com/docs/studio/security/managing-users/#update-user-permissions 'Direct link to Update User Permissions')

1. Select the user in the `Users` list.
 
2. Go to the `Groups` tab and update their group memberships.
 
 ![Update Groups](https://rasa.com/docs/assets/images/keycloak-update-user-groups-4e67f94d5d5e60c55e2c097b9959d739.png)
 

### Log Out a User[​](https://rasa.com/docs/studio/security/managing-users/#log-out-a-user 'Direct link to Log Out a User')

1. Navigate to the `Sessions` tab for a selected user.
 
2. Select **Sign out** from the session menu.
 
 ![Sign Out](https://rasa.com/docs/assets/images/keycloak-user-session-90c50e1ce1939cf337fa223269adaf73.png)
 

## Optional: Configure Email Notifications[​](https://rasa.com/docs/studio/security/managing-users/#optional-configure-email-notifications 'Direct link to Optional: Configure Email Notifications')

Keycloak can send emails for tasks like password resets and account verification. To enable email:

1. Go to `Realm Settings` > `Email`.
 
2. Enter your SMTP server settings and save changes.
 
 ![Email Setup](https://rasa.com/docs/assets/images/keycloak-email-tab-bed15e42359a0d05b3ca461676f913b5.png)
 

For detailed setup, refer to [Keycloak Email Configuration](https://www.keycloak.org/docs/latest/server_admin/#_email).

- [Guides for Managing Users](https://rasa.com/docs/studio/security/managing-users/#guides-for-managing-users)
 - [Delete a User](https://rasa.com/docs/studio/security/managing-users/#delete-a-user)
 - [Update User Permissions](https://rasa.com/docs/studio/security/managing-users/#update-user-permissions)
 - [Log Out a User](https://rasa.com/docs/studio/security/managing-users/#log-out-a-user)
- [Optional: Configure Email Notifications](https://rasa.com/docs/studio/security/managing-users/#optional-configure-email-notifications)