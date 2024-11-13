# OAuth 2.0 

## Overview
Meraki APIs are RESTful APIs that customers and partners can use to programmatically manage and monitor Meraki environments. Previously, API access was only possible through user-scoped API keys. From this release, Meraki introduces a new application-scoped authentication method based on OAuth 2.0. The OAuth 2.0 method replaces the application's reliance on user-scoped API keys and offers several benefits not available with user-scoped API keys.


## What is OAuth 2.0?

OAuth 2.0 is a standard authorization framework that enables integrations to access Meraki data without users revealing their credentials or API keys. OAuth 2.0  is widely used for delegated access, particularly in the context of APIs and web applications. OAuth 2.0   provides a secure and standardized way for users to authorize third-party access to their resources while maintaining control over their data.


[Learn more about the OAuth framework and definitions](https://oauth.net/2/)


## Benefits of OAuth 2.0 Integrations

OAuth 2.0 offers several advantages over traditional API keys:
- **Flexible and least-privilege access**: Developers can request permission for a limited set of OAuth scopes, rather than having an all-or-nothing access.
- **No more copying and pasting API keys**: OAuth 2.0 introduces a secure and seamless grant flow for exchanging credentials  
- **No more API key rotations**: OAuth 2.0 uses short-lived access tokens that are automatically replaced in minutes.
- **Simplified auditing**: Each integration has its own identity, making it easy to trace API calls back to the integration invoking the API call. 

## Building an OAuth 2.0 Integration

Building an OAuth 2.0 integration is simple and easy. Follow these steps:

1. Register your integration with Meraki.
2. Using the OAuth Grant Flow, request the organization admin for permissions for the organization you’d like to manage.
3. Use the Access Token to make API calls.
4. Refresh your Access Token using your Refresh Token as necessary.

### 1. Register your Integration with Meraki

1. Access the application registry at [integrate.cisco.com](https://integrate.cisco.com) using your Cisco.com credential
2. Create a new app. Provide the name, redirect URIs, select the relevant scopes, and so forth.

**Note: `client_secret` is shown only once. Store the `client_secret` securely.** 
Scopes and redirect URIs can be edited later.

### 2. Request Permission Using an OAuth Grant Flow

#### Obtaining an Access Token and Refresh Token:

1. Create a trigger point in your application to initiate the OAuth process, such as a “Connect to Meraki” button or a link.

   When the Meraki admin interacts with this trigger, redirect the admin to [https://as.meraki.com/oauth/authorize](https://as.meraki.com/oauth/authorize) with the following mandatory query parameters:
  - `response_type`: Must be set as `code`
  - `client_id`: Issued when creating your app
  - `redirect_uri`: Must match one of the URIs provided when you registered your integration.
  - `scope`: A space-separated list of scopes being requested by your integration (see scopes)
  - `state`: A unique string passed back to your integration upon completion.
  - `nonce` (optional)

Here is an example link format:
   ```
https://as.meraki.com/oauth/authorize?response_type=code&client_id={client_id}&redirect_uri={redirect_url}&scope={scopes}

```

2. Implement a callback receiver in your application to respond when a request returns the redirection URL. You should expect to receive a `code` attribute as one of the request parameters. This is the **access grant**, and it has a lifetime of 10 minutes.
3. Use the access grant to request a refresh token and an access token. Send a POST request to [https://as.meraki.com/oauth/token](https://as.meraki.com/oauth/token) with the following:
   - Headers: `Content-Type: application/x-www-form-urlencoded`
   - Authentication: Basic authentication using the `client_id` and `client_secret`
   - Payload must include:
     ```json
     {
       "grant_type": "authorization_code",
       "code": "{access_code}",
       "redirect_uri": "{redirect_url}",
       "scope": "{scopes}"
     }
     ```

- The response includes the `access_token` (valid for 60 minutes) and the `refresh_token` (used to generate new `access_token`s).

**Note: Store this `refresh token` securely.**

### 3. Use the OAuth Access Token to Make your API Calls
Congratulations! You can now make API calls to api.meraki.com using the `access_token` (just as you did earlier with your API keys). The API calls can now use the `Authorization` header with `Bearer + access_token`.

```json
{
	"Authorization": Bearer <access_token>
}
```

### 4. Refresh your OAuth Access Token Using Your OAuth Refresh Token

This procedure is based on [RFC 6749: Refreshing an Access Token](https://datatracker.ietf.org/doc/html/rfc6749#section-6)

While an access_token expires 60 minutes after being generated, the refresh_token is long-lived and can be used to obtain new access_tokens. Send a POST request to `https://as.meraki.com/oauth/token` with the following:
- Headers: `Content-Type: application/x-www-form-urlencoded`
- Payload: `grant_type=refresh_token&refresh_token={refresh_token}`

The response includes a new refresh_token and a new access_token (valid for 60 minutes). Securely store the new tokens, as the previous refresh token will be revoked for security reasons.

**Note:** 
- **The refresh_token is automatically revoked after 90 days of inactivity**.
- **It is strongly recommended that you use HTTP basic authentication.**

To know more about OAuth client authentication, see the [Client Password](https://datatracker.ietf.org/doc/html/rfc6749#section-2.3.1.) section of RFC 6749.

### 5. Revoking OAuth Refresh Tokens
A refresh token can be revoked by the Dashboard admin (resource owner) or by the 3rd party application (client application):
- **Dashboard admin revocation**: Navigate to **organization** > **integrations** > **my integrations**, and choose the relevant integration, and click **remove**.
Currently, the client application is not notified when its token is revoked. However, once the refresh token is revoked, all API calls using the access token and the refresh token will fail.
- **Client application revocation**: You can revoke the refresh token from the client application by sending a POST request to `https://as.meraki.com/oauth/revoke` with the following:
  - Headers: `Content-Type: application/x-www-form-urlencoded`
  - Authentication: Basic Authentication using the `client_id` and `client_secret`
  - Payload: 
```
{'token': <the refresh token to be revoked>,
  'token_type_hint': ‘refresh_token}
```
If successfully revoked, you will receive a 200 OK response. 
**Note: It may take up to 10 minutes for the revoked access token to stop working**.
The Client application revocation is based on [RFC 7009]( https://datatracker.ietf.org/doc/html/rfc7009.).

## Troubleshooting

### Supported Clusters
OAuth is currently supported only on Meraki.com. Support for FedRAMP, China, Canada, and India will be added in the future.

### Initial Grant Flow
**Issue 1:** The user cannot find the relevant organization in the dropdown menu.

**Solutions:**
For a user to find an organization in the dropdown menu, do the following:
- Ensure that the user has full organization admin rights. Read-only and/or network admins cannot see their organization.
- Ensure that the app has been integrated. If the app has been integrated, navigate to **organization** > **integrations** > **my integrations**. Revoke access to the app and try integrating the app once again. .  

**Issue 2**: "An error has occurred: The requested redirect URI is malformed or doesn't match the client redirect URI.

**Solution**: Verify if the redirect URI in the request is different from the redirect URIs registered in the app registry.

**Issue 3**: Client authentication failed error. "An error has occurred: Client authentication failed due to unknown client, no client authentication included, or unsupported authentication method.."

**Solution**: Verify if the client ID in the request is correct.


### Errors Returned to the Redirect URI
**Issue**: An invalid scope error is returned to the redirect URI. For example, 
```
https://localhost?error=invalid_scope&error_description=The+requested+scope+is+invalid%2C+unknown%2C+or+malformed.
```
In the above example, the redirect URI is https://localhost/.

**Solution**: 
- Verify if there is a mistake in the scopes included in the request. 
- Verify if the request includes scopes that were not included during app registrations.

**Issue**: An access denied error is returned to the redirect URI. For example, 
```
https://localhost?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
```
**Solution**: 
- Verify if the user has the required access rights. 

### Errors Exchanging Tokens
1. The provided authorization grant is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client.
- The access grant may have been used already.
- More than 10 minutes have passed since the access grant was generated.
- The access grant does ot 

## Understanding OAuth Scopes
OAuth scopes are a mechanism used in OAuth 2.0 to define and limit the access rights granted to an access token. When an integration requests authorization from the Meraki admin, it includes a list of scopes that it wants access to. The Meraki Dashboard then presents these scopes to the Meraki admin during the authorization process, allowing them to approve or deny the request.
By using scopes, OAuth 2.0 provides a flexible and granular approach to controlling access to resources, allowing Meraki admins to make informed decisions about the level of access they grant to integrations. Additionally, scopes help ensure that integrations are provisioned access by the principle of least privilege, enhancing security and privacy.

Meraki offers two types of scopes - `config` scopes and `telemetry` scopes. Each scope has two permission levels: “read-only” and “write”.

1. **`Config`** scopes grant access to configuration features that determine the operation of the network and the network experience. These features typically dictate the end-user network experience and the operation of Meraki devices, e.g. VPNs, VLANs, access controls, policies, SSIDs, and sensor names. Importantly, this excludes admin-facing telemetry configs, which can be managed via the next type of scope.
2. **`Telemetry`** scopes grant access to telemetry and telemetry configuration that does not affect the end-user experience of the network. For example, features like event log, syslog, bandwidth utilization, client count, and camera snapshots.

| Category              | Read                           | Write                          |
|-----------------------|--------------------------------|--------------------------------|
| **Dashboard**         | dashboard:iam:config:read     | dashboard:iam:config:write     |
|                       | dashboard:iam:telemetry:read  | dashboard:iam:telemetry:write  |
|                       | dashboard:general:config:read | dashboard:general:config:write |
|                       | dashboard:general:telemetry:read | dashboard:general:telemetry:write |
|                       | dashboard:licensing:config:read | dashboard:licensing:config:write |
|                       | dashboard:licensing:telemetry:read | dashboard:licensing:telemetry:write |
| **Network**           | sdwan:config:read             | sdwan:config:write             |
|                       | switch:config:read            | switch:config:write            |
|                       | wireless:config:read          | wireless:config:write          |
|                       | sdwan:telemetry:read          | sdwan:telemetry:write          |
|                       | switch:telemetry:read         | switch:telemetry:write         |
|                       | wireless:telemetry:read       | wireless:telemetry:write       |
| **IoT**               | camera:config:read            | camera:config:write            |
|                       | sensor:config:read            | sensor:config:write            |
|                       | camera:telemetry:read         | camera:telemetry:write         |
|                       | sensor:telemetry:read         | sensor:telemetry:write         |
| **Endpoint Management (SM)** | sm:telemetry:read      | sm:telemetry:write             |
|                       | sm:config:read                | sm:config:write                |
