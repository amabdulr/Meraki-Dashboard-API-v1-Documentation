# API Call Budgets

Your API call budget is a limit that 
- limits how many API calls you can make within a specific time."
- safeguards your network against runaway applications and malicious behavior.
- helps you manage call consumption.

Key attributes of an API call budget include:
- defined number of requests per second,
- shared budgets among applications, and
- mechanisms to handle excess calls.

In this article, you will:
- learn **key concepts**, such as action batches and configuration templates. 
- find **rate limits** for organizations and IP addresses.
- learn **how to detect** an exceeded rate limit.
- learn **how to recover gracefully** and keep your application running.
- discover **best practices** to provision and monitor your network and avoid exceeding the rate limit.
- get **troubleshooting tips** to pinpoint and resolve the root causes of an exceeded rate limit.


## Action batches 

An action batch is a tool that  
- bundles multiple configuration requests into a single transaction,  
- supports bulk POST, PUT, and DELETE operations synchronously or asynchronously, and  
- reduces individual API calls to optimize budgets

For more information, see the following:
- [Action Batches](https://developer.cisco.com/meraki/api-v1/action-batches-overview/#action-batches)
- [The GitHub Demo](https://developer.cisco.com/codeexchange/github/repo/shiyuechengineer/action-batches/)
- [The Meraki Blog](https://meraki.cisco.com/blog/2019/06/action-batches-a-recipe-for-success/)
  
## Configuration templates

Use a configuration template to define a standard set of network settings and apply them across multiple networks, helping you manage your network consistently and at scale

Configuration templates support various settings, including VLAN settings, firewall rules, and SD-WAN policies. These templates can be created and managed through the Meraki Dashboard.

For more information, see [Meraki configuration templates](https://documentation.meraki.com/General_Administration/Templates_and_Config_Sync/Managing_Multiple_Networks_with_Configuration_Templates).

## Rate limits per organization
You can make upto 10 requests per second per organization, regardless of the number of applications interacting with that organization.

To accommodate short bursts of activity, you can send an extra 10 requests in the first second, for a total of 30 requests in two seconds.

This limit is shared across all API applications in the organization using [API authentication](https://developer.cisco.com/meraki/api-v1/authorization/), making it essential to coordinate API usage across systems. Monitor your organization’s request patterns to stay within rate limits.

| Metric               | Value                                        |
|-------------------------|-------------------------------------------------------------|
| **Steady-state budget** | 10 requests per second per organization                     |
| **Burst allowance**     | +10 requests in the first second (max 30 requests in 2s)   |
| **Scope**               | Shared across all API applications using the organization’s API key  |

As an organization administrator, check whether multiple applications are using your API budget. Navigate to **Organization > Configure > API & Webhooks**, and choose **API Analytics**. 
You can also use an API to get the [organization's API activity overview](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests-overview-response-codes-by-interval/). 

## Rate limits per source IP address
You can send up to 100 requests per second from each source IP address, regardless of the number of API clients working from that address.
| Metric                  | Value                                      |
|-------------------------|-------------------------------------------------------------|
| **Quota**               | 100 requests per second per source IP                                      |
| **Scope**               | Shared by all clients using that IP  |

## Response codes for exceeding rate limit
When the rate limit is exceeded, the API returns a `429` status code. 

The response includes a `Retry-After` header, which tells you how long to wait before sending the next request.

The response body generally includes an error message structured as:

```JSON
{
    "errors": [
        "API rate limit exceeded for organization"
    ]
}
```

## Handle exceeded rate limits
**Purpose**: Helps you keep your application running smoothly, even if your application exceeds the API rate limits.

**Context**: Rate limits protect resources and ensure fair API usage. If your application exceeds these limits, you receive a `429` response. You must handle this response in code to prevent application failures.

**Before you begin**: 
- Review the rate limit policies listed above.
- Understand how to interpret HTTP status codes, especially status code `429`.

Follow these steps when your application exceeds the rate-limit:
1. Monitor the HTTP response code of each API calls.
2. If you receive a `429` status code, retrieve the `Retry-After` header to determine the wait duration.
3. Implement a backoff mechanism:
   - Pause for one or two seconds before making subsequent API calls.
   - Increase the wait duration if request volume is high.
4. For Python applications:
   - Try [the official Meraki Python library](https://github.com/meraki/dashboard-api-python), which includes automatic retry and backoff handling. Here is an example implementation:

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

**Result**: Your application handles the exceeded rate limit gracefully, minimizing downtime and errors.

## Best practices for optimizing API usage 

Follow these best practices during provisioning and monitoring to ensure that your network performs efficiently. The overall API calls are reduced. Rate limits are not exceeded.

### Best Practices for Provisioning 
- **Use action batches** to group multiple POST, PUT, and DELETE calls into a single request. This reduces overhead and speeds up execution.
- **Retrieve configuration changes efficiently**
    - Most 429 errors happen when you poll information too often after the first day of network deployment, such as for the list of [networks](https://developer.cisco.com/meraki/api-v1/get-organization-networks/) or [policy objects](https://developer.cisco.com/meraki/api-v1/get-organization-policy-objects/) in an organization. These values rarely change after initial deployment. A better strategy is to use [getOrganizationConfigurationChanges](https://developer.cisco.com/meraki/api-v1/get-organization-configuration-changes/) to retrieve a snapshot of all configuration changes. 
- **Use configuration templates**
    - Combine multiple network-specific requests into a single template update.
    - Let Meraki handle updates to all bound networks for you, reducing your API calls.
### Best Practices for Monitoring  
- **Replace an inefficient API operation with an efficient one**
   - Use the most efficient API calls available for your needs, especically if your application includes monitoring features.

| **Use Case**                                  | **Less Efficient Operation**                                                                 | **More Efficient Operation**                                                                                             |
|--------------------------------------------|----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| Retrieving Network Topology Information    | Single-device operation [getDeviceLldpCdp](https://developer.cisco.com/meraki/api-v1/get-device-lldp-cdp/)  | Network-wide [getNetworkTopologyLinkLayer](https://developer.cisco.com/meraki/api-v1/get-network-topology-link-layer/) provides the complete topology for the network   |
| Retrieving Device Uplink Information       | Per-device IP information retrieval                                                           | Organization-wide [getOrganizationDevicesUplinksAddressesByDevice](https://developer.cisco.com/meraki/api-v1/get-organization-devices-uplinks-addresses-by-device/) |
| Retrieving Device Hardware Details         | Single-device operation [getDevice](https://developer.cisco.com/meraki/api-v1/get-device/)   | Organization-wide [getOrganizationDevices](https://developer.cisco.com/meraki/api-v1/get-organization-devices/)           |
| Retrieving Device Hardware Details         | Network-wide operation [getNetworkDevices](https://developer.cisco.com/meraki/api-v1/get-network-devices/) | Organization-wide [getOrganizationDevices](https://developer.cisco.com/meraki/api-v1/get-organization-devices/) provides information for hundreds of devices at a time in a paginated list          |
| Monitoring Device Availability (Status)    | Re-polling all device statuses                                                               | Organization-wide [getOrganizationDevicesAvailabilitiesChangeHistory](https://developer.cisco.com/meraki/api-v1/get-organization-devices-availabilities-change-history/) catches up on device availability (status) changes since your last org-wide poll instead of re-polling the information for all devices. |
| Retrieving Network Clients                 | Single-device API operation [getDeviceClients](https://developer.cisco.com/meraki/api-v1/get-device-clients/) | Network-wide [getNetworkClients](https://developer.cisco.com/meraki/api-v1/get-network-clients/)                          |

# Troubleshoot rate limit issues

**Purpose**: Find and resolve exceedec rate limits to restore uninterrupted API operation.  

**Context**: When your application (or another application sharing your organization or IP) receives `429` errors despite backoff strategies.  

**Before you begin**:  
    - Understand the organization- and IP-level call budgets.  
    - Implemented basic `429` handling using the `Retry-After` header.  

Follow these steps to troubleshoot rate limit issues:

1. **Verify adherence to best practices** by reviewing the “Best practices for optimizing API usage” section. If you use a [partner application](https://marketplace.cisco.com/en-US/home), contact the developer to discuss the application behavior or budget consumption. 
2. **Check recent API activity** on the Meraki dashboard. See [Checking recent API activity](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests-overview-response-codes-by-interval/)
3. **Audit your scripts** that run with little to no maintenance. These can degrade performance and unnecessarily consume your call budget. See [Audit your organization's API consumption](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests/).

**Result**: You will identify what caused the exceeded rate limits.

# References
- For more information about call budgets and rate limits, see the [our developer community](https://community.meraki.com/t5/Developers-APIs/bd-p/api).
- Meraki uses the [token bucket model](https://en.wikipedia.org/wiki/Token_bucket) to implement this rate-limiting mechanism.

