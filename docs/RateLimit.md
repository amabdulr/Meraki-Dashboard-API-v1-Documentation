# API call budgets
An API call budget is a limit that defines the number of API calls an API client can make in a given amount of time. It serves as a safeguard against runaway applications and malicious behavior. Call budgets are a standard feature of high-performance APIs across industries, and managing them effectively is crucial for developers. Key attributes include a defined number of requests per second, shared budgets among applications, and mechanisms to handle excess calls.

## Rate limits per organization
Each Meraki organization has a call budget of 10 requests per second. This is regardless of the number of API applications interacting with that organization.

A burst of 10 additional requests is allowed in the first second, allowing for a maximum of 30 requests in the first 2 seconds. This budget is shared across all API applications in the organization using API authentication. You can check the recent API activity for the given organization to understand if you are sharing the budget with other applications.

This budget is shared across all API applications in the organization that leverage [API authentication](https://developer.cisco.com/meraki/api-v1/authorization/). You can [check the recent API activity](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests-overview-response-codes-by-interval/) for the given organization to understand if you are sharing the budget with other applications.

For more information on the rate-limiting technique used, see [token bucket model](https://en.wikipedia.org/wiki/Token_bucket).

## Rate limits per source IP address
Each source IP address making API requests has a call budget of 100 requests per second, regardless of the number of API clients working from that IP address.


## Response codes for rate limiting
A `429` status code is returned when the rate limit is exceeded. The response includes a `Retry-After` header indicating the wait time before making a follow-up request. When an application exceeds the rate limit, the following message will be returned in the response body:

```JSON
{
    "errors": [
        "API rate limit exceeded for organization"
    ]
}
```


## Handling rate limit exceedance
Handle cases when the API rate limit is exceeded to ensure continued application functionality.

**Before you begin**: Familiarize yourself with rate limit policies and handling HTTP response codes.

Follow these steps to manage rate limit exceedance:

Step 1: Check the HTTP response code when making API calls.

Step 2: If you receive a `429` status code, read the `Retry-After` header to determine how long to wait before retrying.

Step 3: Implement a backoff mechanism to wait for 1-2 seconds, or potentially longer if a large number of requests were made, before attempting subsequent API calls.

Step 4: Utilize the official Meraki Python library for automatic retry and backoff handling, if using Python. See [the official Meraki Python library](https://github.com/meraki/dashboard-api-python). Here is an example implementation:


```Python
response = requests.request("GET", url, headers=headers)
​
if response.status_code == 200:
    # Success logic
elif response.status_code == 429:
    time.sleep(int(response.headers["Retry-After"]))
else:
    # Handle other response codes
```

**Result**: Your application will handle rate limit exceedance gracefully, minimizing downtime and errors.

## Manage API usage during provisioning
To efficiently manage API usage and reduce the number of individual API calls during provisioning, you should use action batches and identify configuration deviations instead of repolling.

**Before you begin**: Familiarize yourself with the provisioning process and ensure you understand the concept of action batches.

**Step 1: Use action batches for bulk operations and reduce individual API calls**

Action Batches are a tool for submitting batched configuration requests in a single synchronous or asynchronous transaction. They are useful for bulk constructive or destructive operations such as `POST`, `PUT`, or `DELETE`. Action Batches help in maintaining efficient API usage and reducing the number of individual API calls.

For more information, see [our overview](https://developer.cisco.com/meraki/api-v1/action-batches-overview/#action-batches), the [GitHub Demo](https://developer.cisco.com/codeexchange/github/repo/shiyuechengineer/action-batches/), or the [Meraki Blog](https://meraki.cisco.com/blog/2019/06/action-batches-a-recipe-for-success/).

**Step 2: Identify configuration deviations instead of repolling**

Once you are confident that an organization is configured correctly, leverage [getOrganizationConfigurationChanges](https://developer.cisco.com/meraki/api-v1/get-organization-configuration-changes/) to identify deviations from the last known intended configuration. This is more efficient than re-polling every configuration setting and then running diffs in your application.

**Step 3: Use configuration templates to ensure consistent configuration***

[Meraki configuration templates](https://documentation.meraki.com/General_Administration/Templates_and_Config_Sync/Managing_Multiple_Networks_with_Configuration_Templates) are an easy way to ensure a highly consistent configuration is deployed across networks. 

**Result**: You will maintain efficient API usage and reduce the number of individual API calls during provisioning, ensuring optimal performance and minimizing the risk of hitting rate limits.


## Use the most efficient operations for your use case

Strategize your data polling. The most common cause of `429` responses is unnecessarily frequent polling of information that changes infrequently after day 1 of a network deployment, such as the list of [networks](https://developer.cisco.com/meraki/api-v1/get-organization-networks/) or [policy objects](https://developer.cisco.com/meraki/api-v1/get-organization-policy-objects/) in an organization.

Develop your application using the most efficient API calls available for your use case, especially if your application provides monitoring features.


| **Use Case**                                  | **Less Efficient Operation**                                                                 | **More Efficient Operation**                                                                                             |
|--------------------------------------------|----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| Retrieving Network Topology Information    | Single-device operation [getDeviceLldpCdp](https://developer.cisco.com/meraki/api-v1/get-device-lldp-cdp/)  | Network-wide [getNetworkTopologyLinkLayer](https://developer.cisco.com/meraki/api-v1/get-network-topology-link-layer/)    |
| Retrieving Device Uplink Information       | Per-device IP information retrieval                                                           | Organization-wide [getOrganizationDevicesUplinksAddressesByDevice](https://developer.cisco.com/meraki/api-v1/get-organization-devices-uplinks-addresses-by-device/) |
| Retrieving Device Hardware Details         | Single-device operation [getDevice](https://developer.cisco.com/meraki/api-v1/get-device/)   | Organization-wide [getOrganizationDevices](https://developer.cisco.com/meraki/api-v1/get-organization-devices/)           |
| Retrieving Device Hardware Details         | Network-wide operation [getNetworkDevices](https://developer.cisco.com/meraki/api-v1/get-network-devices/) | Organization-wide [getOrganizationDevices](https://developer.cisco.com/meraki/api-v1/get-organization-devices/)           |
| Monitoring Device Availability (Status)    | Re-polling all device statuses                                                               | Organization-wide [getOrganizationDevicesAvailabilitiesChangeHistory](https://developer.cisco.com/meraki/api-v1/get-organization-devices-availabilities-change-history/) |
| Retrieving Network Clients                 | Single-device API operation [getDeviceClients](https://developer.cisco.com/meraki/api-v1/get-device-clients/) | Network-wide [getNetworkClients](https://developer.cisco.com/meraki/api-v1/get-network-clients/)                          |




# Troubleshooting Rate Limiting Issues
Diagnose and resolve issues when your application is rate limited to ensure optimal API performance.

Before you begin: Familiarize yourself with the concept of API call budgets and rate limiting.

Follow these steps to troubleshoot rate limit issues:

Step 1: Ensure your application follows the best practices for managing call budgets. If you are using an ecosystem partner application, contact the developer to discuss partner application behavior or budget consumption. See [our best practices](https://developer.cisco.com/meraki/api-v1/rate-limit/#best-practices-and-tips-for-managing-call-budgets).

Step 2: Check the recent API activity for your organization to determine if the budget is shared with other applications. See [check the recent API activity](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests-overview-response-codes-by-interval/)

Step 3: Audit your organization's API consumption regularly to identify scripts that run with little to no maintenance, which can degrade performance and unnecessarily consume your budget. See [Audit your organization's API consumption](https://developer.cisco.com/meraki/api-v1/search/api%20requests/).

Result: By following these steps, you will effectively manage your API call budget and minimize instances of rate limiting.

For more information about call budgets and rate limits, see the [our developer community](https://community.meraki.com/t5/Developers-APIs/bd-p/api).
