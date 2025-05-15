# API Call Budgets

An API call budget is a limit that 
- defines the number of API calls an API client can make in a given amount of time.
- safeguards against runaway applications and malicious behavior.
- guides developers on managing call consumption.

Key attributes of an API call budget include 
- defined number of requests per second,
- shared budgets among applications, and
- mechanisms to handle excess calls.

This article 
- explains **important concepts**, such as action batches and configuration templates, that are needed to understand the article.
- lists the **rate limits** for organizations and IP addresses.
- shows you **how to detect** when these rate limits are breached using the `429` status code.
- shows you **how to recover gracefully** to keep your application running.
- provides **best practices** for provisioning and monitoring a network and avoid a rate-limit breach.
- provides **troubleshooting tips** to pinpoint and resolve the root causes of a rate-limit breach.


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

A configuration template is a centralized network configuration model that
- allows administrators to define a standardized set of settings,
- enables the application of these settings across multiple networks, and
- facilitates consistent and scalable network management.

Configuration templates support various configurations, including VLAN settings, firewall rules, and SD-WAN policies. These templates can be created and managed through the Meraki Dashboard, providing a user-friendly interface for network administrators.

For more information, see [Meraki configuration templates](https://documentation.meraki.com/General_Administration/Templates_and_Config_Sync/Managing_Multiple_Networks_with_Configuration_Templates).

## Rate limits per organization
Each Meraki organization has a rate limit of 10 requests per second per organization, regardless of the number of applications interacting with that organization.

To accommodate short bursts of activity, an additional ten requests are allowed in the first second, enabling a maximum of 30 requests in the first two seconds. 

This limit is shared across all API applications in the organization using [API authentication](https://developer.cisco.com/meraki/api-v1/authorization/), making it esssential to coordinate API usage across systems. Monitoring your organization’s request patterns is one way to avoid breaching the rate limit.

| Metric               | Value                                        |
|-------------------------|-------------------------------------------------------------|
| **Steady-state budget** | 10 requests per second per organization                     |
| **Burst allowance**     | +10 requests in the first second (max 30 requests in 2s)   |
| **Scope**               | Shared across all API applications using the organization’s API key  |

You, as an organization administrator of the Meraki dashboard, can check whether your API budget is being consumed by multiple applications by navigating to **Organization > Configure > API & Webhooks** > **API Analytics**. You can also use an API to get the [organization's API activity overview](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests-overview-response-codes-by-interval/). 


## Rate limits per source IP address
Each source IP address can make up to 100 requests per second, regardless of the number of API clients working from that IP address.
| Metric                  | Value                                      |
|-------------------------|-------------------------------------------------------------|
| **Quota**               | 100 requests per second per source IP                                      |
| **Scope**               | Shared by all clients using that IP  |

## Response codes for rate limit breach
When the rate limit is breached, a `429` status code is returned. 

The response includes a `Retry-After` header, indicating the amount of time the client must wait before sending the next request.

The response body generally includes an error message structured as:

```JSON
{
    "errors": [
        "API rate limit exceeded for organization"
    ]
}
```

## Handle rate limit breach
**Purpose**: Ensure your application functions smoothly even after your application breaches API rate limits.

**Context**: APIs often enforce rate limits to protect resources and ensure fair usage. Exceeding these limits results in a `429` response, which must be handled programmatically to avoid application failures.

**Before you begin**: 
- Review the rate limit policies listed above.
- Understand how to interpret HTTP status codes, especially status code `429`.

Follow these steps to manage a rate-limit breach:
1. Monitor the HTTP response code of each API calls.
2. If you receive a `429` status code, retrieve the `Retry-After` header to determine the wait duration.
3. Implement a backoff mechanism:
   - Pause for 1-2 seconds before making subsequent API calls.
   - Increase the wait duration if request volume is high.
4. For Python applications:
   - Consider using the [the official Meraki Python library](https://github.com/meraki/dashboard-api-python), which includes  automatic retry and backoff handling. Here is an example implementation:

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

**Result**: Your application handles rate limit breaches gracefully, minimizing downtime and errors.

## Best practices for optimizing API usage 

Follow these best practices during provisioning and monitoring to ensure that your network performs efficiently. The overall API calls are reduced. Rate limits are not breached.

### Best Practices for Provisioning 
- **Use action batches** to group multiple POST, PUT, DELETE calls into a single request. This reduces overhead and speeds up execution.
- **Avoid repolling for configuration changes** 
    - The most common cause of `429` responses is unnecessarily frequent polling of information after day one of a network deployment, such as for the list of [networks](https://developer.cisco.com/meraki/api-v1/get-organization-networks/) or [policy objects](https://developer.cisco.com/meraki/api-v1/get-organization-policy-objects/) in an organization. These values rarely change after initial deployment. A better strategy is to use [getOrganizationConfigurationChanges](https://developer.cisco.com/meraki/api-v1/get-organization-configuration-changes/) to retrieve a snapshot of all configuration changes. 
- **Use configuration templates**
    - Consolidate multiple network-specific requests into a single template update.
    - Allow the Meraki backend to propagate changes to all bound networks automatically, reducing the volume of API calls.
### Best Practices for Monitoring  
- **Replace an inefficient API operation with an efficient one**
   - Develop your application using the most efficient API calls available for your use case. This is particularly relevant if your application includes monitoring features.

| **Use Case**                                  | **Less Efficient Operation**                                                                 | **More Efficient Operation**                                                                                             |
|--------------------------------------------|----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| Retrieving Network Topology Information    | Single-device operation [getDeviceLldpCdp](https://developer.cisco.com/meraki/api-v1/get-device-lldp-cdp/)  | Network-wide [getNetworkTopologyLinkLayer](https://developer.cisco.com/meraki/api-v1/get-network-topology-link-layer/) provides the complete topology for the network   |
| Retrieving Device Uplink Information       | Per-device IP information retrieval                                                           | Organization-wide [getOrganizationDevicesUplinksAddressesByDevice](https://developer.cisco.com/meraki/api-v1/get-organization-devices-uplinks-addresses-by-device/) |
| Retrieving Device Hardware Details         | Single-device operation [getDevice](https://developer.cisco.com/meraki/api-v1/get-device/)   | Organization-wide [getOrganizationDevices](https://developer.cisco.com/meraki/api-v1/get-organization-devices/)           |
| Retrieving Device Hardware Details         | Network-wide operation [getNetworkDevices](https://developer.cisco.com/meraki/api-v1/get-network-devices/) | Organization-wide [getOrganizationDevices](https://developer.cisco.com/meraki/api-v1/get-organization-devices/) provides information for hundreds of devices at a time in a paginated list          |
| Monitoring Device Availability (Status)    | Re-polling all device statuses                                                               | Organization-wide [getOrganizationDevicesAvailabilitiesChangeHistory](https://developer.cisco.com/meraki/api-v1/get-organization-devices-availabilities-change-history/) catches up on device availability (status) changes since your last org-wide poll instead of re-polling the information for all devices. |
| Retrieving Network Clients                 | Single-device API operation [getDeviceClients](https://developer.cisco.com/meraki/api-v1/get-device-clients/) | Network-wide [getNetworkClients](https://developer.cisco.com/meraki/api-v1/get-network-clients/)                          |

# Troubleshoot rate limit breach

**Purpose**: Diagnose and resolve rate-limit breaches to restore uninterrupted API operation.  

**Context**: When your application (or another application sharing your organization or IP) receives `429` errors despite backoff strategies.  

**Before you begin**:  
    - Understand the organization- and IP-level call budgets.  
    - Implemented basic `429` handling using the `Retry-After` header.  

Follow these steps to troubleshoot rate limit issues:

1. **Verify adherence to best practices** by reviewing the “Best practices for optimizing API usage” section. If you are using a [partner application](https://marketplace.cisco.com/en-US/home), contact the developer to discuss the application behavior or budget consumption. 
2. **Check recent API activity** on the Meraki dashboard. See [Checking recent API activity](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests-overview-response-codes-by-interval/)
3. **Audit your scripts** that run with little to no maintenance. These can degrade performance and unnecessarily consume your call budget. See [Audit your organization's API consumption](https://developer.cisco.com/meraki/api-v1/get-organization-api-requests/).

**Result**: You will identify the root cause of the rate-limit breach.

# References
- For more information about call budgets and rate limits, see the [our developer community](https://community.meraki.com/t5/Developers-APIs/bd-p/api).
- Meraki uses the [token bucket model](https://en.wikipedia.org/wiki/Token_bucket) to implement this rate-limiting mechanism.

