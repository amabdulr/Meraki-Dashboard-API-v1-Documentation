# OAuth 2.0 

## Overview
Meraki APIs are RESTful APIs that developers can use to programmatically manage and monitor Meraki environments. 

Previously, developers accessed these APIs only through user-scoped API keys. 

This document explains the new application-scoped authentication method based on OAuth 2.0, detailing its implementation and benefits.  The OAuth 2.0 method replaces the application's reliance on user-scoped API keys and offers several benefits not available with user-scoped API keys.


## What Is OAuth 2.0?

OAuth 2.0 is a standard authorization framework that enables integrations to access Meraki data securely, eliminating the need for administrators to reveal their credentials or API keys.

OAuth 2.0 is commonly used to allow delegated access, particularly in the context of APIs and web applications. OAuth 2.0 provides a secure and standardized way for the network administrator, to authorize third-party access to their resources while maintaining control over data.


[Learn more about the OAuth framework and definitions](https://oauth.net/2/)


## What Is an OAuth 2.0 Integration?

An OAuth 2.0 integration, referred to as integration, is a software application or system that connects to the Meraki platform and interacts with Meraki's services and data. An integration uses APIs to automate, manage, or enhance functionalities within a Meraki environment. With OAuth 2.0, integrations enable developers to securely access Meraki resources, allowing them to monitor network status, configure settings, and collect data without needing to input credentials directly. 

## Benefits of OAuth 2.0 Integrations

Using OAuth 2.0 for authentication offers several advantages compared to traditional API keys, including:

- **Flexible and least-privilege access**: Developers can request permission for a limited set of OAuth scopes, rather than having all-or-nothing access.
- **Avoid copy-pasting API keys**: OAuth 2.0 provides a secure and seamless grant flow for exchanging credentials.
- **Avoid API key rotations**: OAuth 2.0 uses short-lived access tokens. These tokens are automatically rotated in minutes.
- **Simplified auditing**: Each integration has its own identity, making it easy to trace API calls back to the integration invoking the API call. 

## Build an OAuth 2.0 Integration

You can build an OAuth 2.0 integration by following these steps:

1. Register your integration with Meraki.
2. Using the OAuth grant flow, request the administrator of your organization for permission to manage that organization.
3. Use the access token to make API calls.
4. Refresh your access token.

### 1. Register Your Integration with Meraki

1. Access the application registry at [integrate.cisco.com](https://integrate.cisco.com) using your Cisco.com credentials.
2. Create a new application. Provide the name and redirect URIs, select the relevant scopes, and enter any information requested.

**Note: `client_secret` is shown only once. Store the `client_secret` securely.** 
Scopes and redirect URIs can be edited later.

### 2. Request Permission Using an OAuth Grant Flow

#### Obtain Access Token and Refresh Token:

1. To start the OAuth process, create a trigger point in your application, such as a “Connect to Meraki” button or a link.

   When the Meraki administrator interacts with this trigger, redirect the administrator to [https://as.meraki.com/oauth/authorize](https://as.meraki.com/oauth/authorize) with the following mandatory query parameters:
  - `response_type`: Must be set as `code`
  - `client_id`: Issued when creating your application
  - `redirect_uri`: Must match one of the URIs provided when you registered your integration
  - `scope`: Your integration requests this space-separated list of scopes (see the "Understanding OAuth Scopes" section below)
  - `state`: A unique string passed back to your integration upon completion
  - `nonce` (optional): 

Here is an example link format:
   ```
https://as.meraki.com/oauth/authorize?response_type=code&client_id={client_id}&redirect_uri={redirect_url}&scope={scopes}

```

2. Implement a callback receiver in your application to respond when a request returns the redirection URL. You should expect to receive a `code` attribute as one of the request parameters. This `code` attribute is the **access grant**. Once issued, the access grant remains valid for ten minutes.

3. Use the access grant to request a refresh token and an access token. Send a POST request to [https://as.meraki.com/oauth/token](https://as.meraki.com/oauth/token) with the following:
   - Headers: `Content-Type: application/x-www-form-urlencoded`
   - Authentication: Basic authentication using the `client_id` and `client_secret`
   - Payload:
     ```json
     {
       "grant_type": "authorization_code",
       "code": "{access_code}",
       "redirect_uri": "{redirect_url}",
       "scope": "{scopes}"
     }
     ```

- The response includes the `access_token`, which is valid for 60 minutes, and the `refresh_token`, which is used to generate new `access_token`s.

**Note: Store this `refresh token` securely.**

### 3. Use the OAuth Access Token to Make API Calls
You can now make API calls to api.meraki.com using the `access_token` (just as you did earlier with your API keys). The API calls can now use the `Authorization` header with `Bearer + access_token`.

```json
{
	"Authorization": "Bearer <access_token>"
}
```

### 4. Refresh Your OAuth Access Token

This procedure is based on [RFC 6749: Refreshing an Access Token](https://datatracker.ietf.org/doc/html/rfc6749#section-6)

While an access_token expires 60 minutes after being generated, the refresh_token is long-lived and can be used to obtain new access_tokens. Send a POST request to `https://as.meraki.com/oauth/token` with the following:
- Headers: `Content-Type: application/x-www-form-urlencoded`
- Payload: `grant_type=refresh_token&refresh_token={refresh_token}`

The response includes a new refresh_token and a new access_token (valid for 60 minutes). Securely store the new tokens, as the previous refresh token will be revoked for security reasons.

**Note:** 
- **The refresh_token is automatically revoked after 90 days of inactivity**.
- **We strongly recommend that you use HTTP basic authentication.**

To know more about OAuth client authentication, see the [Client Password](https://datatracker.ietf.org/doc/html/rfc6749#section-2.3.1.) section of RFC 6749.

### 5. Revoke an OAuth Refresh Token
A refresh token can be revoked by either the Meraki **Organization admin** (resource owner) from the dashboard or by the third-party client application:
- **Dashboard revocation by administrator**: From the Meraki dashboard left-navigation pane, choose **Organization**>**Integrations**. From the **My integrations** tab, choose your integration. From the integration window that opens, from the top-right corner, click **Remove**.
Currently, the client application is not notified when its token is revoked. However, once the refresh token is revoked, all API calls using the access token and the refresh token fail.
- **Client application revocation**: You can revoke the refresh token from the client application by sending a POST request to `https://as.meraki.com/oauth/revoke` with the following:
  - Headers: `Content-Type: application/x-www-form-urlencoded`
  - Authentication: Basic Authentication using the `client_id` and `client_secret`
  - Payload: 
```
{'token': <the refresh token to be revoked>,
  'token_type_hint': ‘refresh_token}
```
If the token is successfully revoked, you will receive a 200 OK response. 
**Note: It may take up to 10 minutes for the revoked access token to stop working**.
The process of Revoking the token is based on [RFC 7009]( https://datatracker.ietf.org/doc/html/rfc7009.).

## Troubleshooting

### Supported Clusters
OAuth is currently supported only on Meraki.com. Support for the Federal Risk and Authorization Management Program (FedRAMP), as well as for China, Canada, and India, will be added in the future.

### Initial Grant Flow
**Issue 1:** The administrator cannot find the relevant organization in the dropdown menu.

**Solution:**
For an administrator to find an organization in the dropdown menu, do the following:
- Ensure that the Meraki administrator has full **Organization admin** rights. Both the "Organization admin" with read-only permissions and the "Network admin" have insufficient permissions to view the organization.
- Ensure that the application has been integrated.
- If the application has been integrated, you can revoke the it's access, and try integrating the application again. From the Meraki dashboard left-navigation pane, choose **Organization**>**Integrations**. From the **My integrations** tab, choose your integration. From the integration window that opens, from the top-right corner, click **Remove**. Now try integrating the application again. 

**Issue 2**: "An error has occurred: The requested redirect URI is malformed or doesn't match the client redirect URI."

**Solution**: Check whether the redirect URI in the request differs from the redirect URIs that were registered in the application registry.

**Issue 3**: Client authentication failed error. "An error has occurred: Client authentication failed due to an unknown client, no client authentication included, or unsupported authentication method.."

**Solution**: Check whether the client ID in the request is correct.


### Errors Returned to the Redirect URI
**Issue**: An invalid scope error is returned to the redirect URI. Here is an example of this error: 
```
https://localhost?error=invalid_scope&error_description=The+requested+scope+is+invalid%2C+unknown%2C+or+malformed.
```
In the above example, the redirect URI is `https://localhost/`.

**Solution**: 
- Check whether there is a mistake in the scopes included in the request. 
- Check whether the request includes scopes that were not included during application registrations.

**Issue**: An access denied error is returned to the redirect URI. For example, 
```
https://localhost?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
```
**Solution**: 
- Check whether the administrator has the required permissions. 

**Issue**: The provided authorization grant is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client.

**Solution**:
- Ensure that the access grant has not been used already.
- Confirm that no more than 10 minutes have passed since the access grant was generated.
- Check whether the access grant matches the expected parameters, including the redirection URI and client details.


## Understand OAuth Scopes

OAuth scopes in OAuth 2.0 are used to define and limit the access rights granted to an access token. 

When an integration requests authorization from an administrator, it  must include a list of scopes that the integration seeks access to.  The Meraki Dashboard presents these scopes to the admin during the authorization process, allowing them to approve or deny the request.

Using scopes, OAuth 2.0 offers a flexible and granular method for controlling access to resources. This enables the administrator to make informed decisions regarding the level of access granted to integrations. This mechanism supports the principle of least privilege, enhancing security and privacy.

  
Meraki provides the following two scopes:
1. **`config`**: This scope grants access to configuration features that influence the operation of the network and the overall network experience. The config scope dictates the end-user network experience and the functioning of Meraki devices, such as VPNs, VLANs, access controls, policies, SSIDs, and sensor names. Note that the `config` scope excludes admin-facing telemetry configurations, which are managed using telemetry scopes.

2. **`telemetry`**: This scope grants access to telemetry data and configurations that do not impact the end-user network experience. They include features like event logs, syslog, bandwidth utilization, client counts, and camera snapshots.

Note: The Meraki scopes can have either "read-only" or "write" permission levels.


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
