# Getting started with Meraki dashboard API

A Meraki dashboard API is a programming interface that
- enables users to interact with and manage Meraki networks programmatically,
- provides access to operations such as retrieving organizations, networks, devices, and device uplink addresses, and
- supports multiple authentication methods, including bearer tokens, for secure access.

**Examples of operations available through the API:**
- Listing all organizations and networks
- Viewing device details and uplink addresses

**Definitions:**
- **Bearer token**: A secure access key used to authorize requests to the Meraki Dashboard API. It can be a Meraki API key or an OAuth access token.


## Authorize API requests using a bearer token

Use this task to setup secure access to the Meraki Dashboard API by authorizing requests using a bearer token.

All API requests to the Meraki Dashboard require an authorization header with a valid bearer token. This token can be either:
- a **Meraki API key**, or
- an **OAuth access token**.

**Before you begin**:
- Obtain a valid **Bearer token** (Meraki API key or OAuth token).

Follow these steps to authorize your API requests:
1. Include the `Authorization` header in your API request with the format:
   ```json
   {
     "Authorization": "Bearer <BEARER_TOKEN>"
   }
   ```

2. Use `curl` to send an authorized request:
   ```bash
   curl https://api.meraki.com/api/v1/organizations \
     -H 'Authorization: Bearer {BEARER_TOKEN}'
   ```

3. Alternatively, use the Meraki Python SDK and provide the token during initialization:
   ```python
   import meraki
   dashboard = meraki.DashboardAPI(BEARER_TOKEN)
   ```

**Additional information**:
- All endpoints require this authorization header for access.
- For more information, see [Authorization](#!authorization).

# Using Postman for Meraki API 

Use this task to interact with the Meraki Dashboard API using the Postman interface.

Postman is a powerful tool that allows you to send API requests, inspect responses, and test endpoints without writing code.

**Before you begin**:
- Ensure **Postman** is installed on your system.
- Ensure you have your bearer token ready.

Follow these steps to use Postman:

1. Go to [Postman and our Postman collection](https://documenter.getpostman.com/view/897512/SzYXYfmJ).
2. Import the collection by clicking the **Run in Postman** button.
3. Set up your **Bearer token** for authorization.
   - For guidance, refer to the **Authorization** section in Postman. For more information, see [Authorization](https://developer.cisco.com/meraki/api-v1/authorization/).
4. Explore and test the endpoints available in the collection.

**Additional information**:
- You can send requests for any API endpoint directly from Postman and visualize the responses instantly.
- Use environment variables in Postman to store your API key, organization ID, or network ID for convenience. 

**Result:** You can now interact with Meraki networks using Postman. 

# Using Python library for Meraki API 

Use this task to interact with the Meraki Dashboard API programmatically using the official Meraki Python library.

The Meraki Python SDK simplifies the process of sending requests and handling responses when working with the Meraki Dashboard API.

**Before you begin**:
- Ensure **Python** and **pip** are installed on your system.
- Ensure you have your Bearer token ready.
- For any setup troubleshooting issues, see [Python Library](https://developer.cisco.com/meraki/api-v1/python/#documentation).

Follow these steps to use the Python library:
1. Install the Meraki library using pip:
   ```bash
   pip install meraki
   ```
2. Import the Meraki library in your Python script:
   ```python
   import meraki
   ```

3. Initialize the Meraki Dashboard API client:
   ```python
   dashboard = meraki.DashboardAPI(BEARER_TOKEN)
   ```

**Additional information**:
- You can now use the `dashboard` object to access and invoke any available API endpoint in your scripts.
- For example, to get a list of organizations:
  ```python
  response = dashboard.organizations.getOrganizations()
  print(response)
  ```

**Result**: You are now ready to perform Meraki API operations directly within your Python scripts using a simple, structured, and reusable codebase.


# Finding organization ID 

Many Meraki API operations require the organization ID as a path or query parameter. Use this task to find your organization ID if you don’t already know it.

**Before you begin**:
- Ensure you have your **bearer token** ready.

Follow these steps to get your organization ID:

1. Send a GET request to the `/organizations` endpoint to list all organizations you have access to. For more information, see [Get Organization](https://developer.cisco.com/meraki/api-v1/get-organizations/) API.
   ```
   GET /organizations
   ```
2. Use `curl` to send the API request:
   ```cURL
   curl https://api.meraki.com/api/v1/organizations \
     -L -H 'Authorization: Bearer {BEARER_TOKEN}'
   ```
3. Alternatively, use the Meraki Python SDK to send the request:
   ```Python
   import meraki
   dashboard = meraki.DashboardAPI(BEARER_TOKEN)
   response = dashboard.organizations.getOrganizations()
   ```
4. Retrieve the value of the `id` field from the response. This is your **organization identifier**.

**Additional information**:
- The organization ID is required for most API endpoints and must be noted before you proceed with any organization- or network-level operations.


### Example Response:  

```JSON
Successful HTTP Status: 200
[
  {
    "id": "549236",
    "name":"DevNet Sandbox"
  }
]
```
### Example Python output:

```Python
>>> print(response)
[{'id': '549236', 'name': 'DevNet Sandbox'}]
```
**Note:** Irrelevant response attributes are omitted from the examples for brevity.

**Result**: You will obtain the organization ID needed to perform API operations such as listing devices, networks, or uplink addresses.

**Post-requisites**: Note the organization ID and keep it handy for all subsequent API requests that require it as a path or query parameter.

# Finding network ID 

Use this task to list all networks under your organization so you can use a specific network ID in subsequent API requests.

**Before you begin**:
- Ensure you have the **organization ID**.
- Ensure you have your **bearer Token** ready.

Follow these steps to get your network ID:
1. Send a GET request to the `/organizations/:organizationId/networks` endpoint to retrieve network information. For more information, see [Get Organization Networks](https://developer.cisco.com/meraki/api-v1/get-organization-networks).
2. Include your organization ID and Bearer Token in the request headers.
     ```
       `GET /organizations/:organizationId/networks`
     ```
3. Use `curl` to send the API request: 
    ```cURL
       curl https://api.meraki.com/api/v1/organizations/{organizationId}/networks \
         -L -H 'Authorization: Bearer {BEARER_TOKEN}'
    ```
4. Alternatively, use the Meraki Python SDK to send the request:
    ```Python
       import meraki
       dashboard = meraki.DashboardAPI(BEARER_TOKEN)
       response = dashboard.organizations.getOrganizationNetworks(org_id)
    ```
5. Copy the value of the `id` field from the response. This is the **network identifier**.
   
**Result**: You obtain the network identifier (network ID) needed for network-specific operations in future API requests.


### Example Response:

```JSON
Successful HTTP Status: 200
[
  {
    "id":"N_1234",
    "organizationId":"12345678",
    "type": "wireless",
    "name":"My network",
    "timeZone": "US/Pacific",
    "tags": null
  }
]
```

```Python
>>> print(response)
[{'id': 'L_646829496481105433', 'organizationId': '549236', 'name': 'DevNet Sandbox Always on READ ONLY', 'timeZone': 'America/Los_Angeles', 'tags': None, 'productTypes': ['appliance', 'switch', 'wireless'], 'type': 'combined', 'disableMyMerakiCom': False, 'disableRemoteStatusPage': True}]
```
**Additional information**:
- The network ID is required for various API endpoints that operate at the network level.
- You can use this identifier to filter devices or perform configuration operations on a specific network.

**Post-requisites**: Note the network ID for use in any follow-up tasks that require it as a path or query parameter.
  
## Find devices and their serial numbers 

Use this task to list all devices in your organization and extract their serial numbers for further API operations.

**Before you begin**:
- Ensure you have the **organization ID**.
- Optionally, obtain the **network identifier** to filter devices by a specific network.
- Ensure you have your **bearer token** ready.

Follow these steps to find devices and their serial numbers:

1. Send a GET request to the `/organizations/:organizationId/devices` endpoint.
2. Include your organization identifier and API key in the request headers.   
     ```
    `GET /organizations/:organizationId/devices`
     ```
3. Use `curl` to send the API request:
    ```cURL
    curl https://api.meraki.com/api/v1/organizations/{organizationId}/devices \
      -L -H 'Authorization: Bearer {BEARER_TOKEN}'
    ```
4. Alternatively, use the Meraki Python SDK to send the request:
    ```Python
    import meraki
    dashboard = meraki.DashboardAPI(BEARER_TOKEN)
    response = dashboard.organizations.getOrganizationDevices({organizationId})
    ```
5. Examine the response and note the value of the `serial` field for each device.

**Result**: You will obtain the serial numbers and detailed metadata of each device in your organization or specific network.

### Example response:
```JSON
Successful HTTP Status: 200
[
    {
        "name": "My AP",
        "lat": 37.4180951010362,
        "lng": -122.098531723022,
        "address": "1600 Pennsylvania Ave",
        "notes": "My AP note",
        "tags": [ "recently-added" ],
        "networkId": "N_24329156",
        "serial": "Q234-ABCD-5678",
        "model": "MR34",
        "mac": "00:11:22:33:44:55",
        "lanIp": "1.2.3.4",
        "firmware": "wireless-25-14",
        "productType": "wireless"
    }
]
```
### Example Python output:
```Python
>>> print(response)
[{ 'name': 'My AP', 'lat': 37.4180951010362, 'lng': -122.098531723022, 'address': '1600 Pennsylvania Ave', 'notes': 'My AP note', 'tags': [ 'recently-added' ], 'networkId': 'N_24329156', 'serial': 'Q234-ABCD-5678', 'model': 'MR34', 'mac': '00:11:22:33:44:55', 'lanIp': '1.2.3.4', 'firmware': 'wireless-25-14', 'productType': 'wireless'}]
```

**Post-requisites**: Use the serial number field in future requests requiring a device serial as a query or path parameter. If you have multiple devices, record the serial numbers of each one accordingly.

**Additional information**:
- You can optionally filter by a network identifier if you want to narrow the results to a specific network.
- This example does not use the network identifier parameter.

# Retrieve uplink addresses for specific devices 

Use this task to view the public and private uplink addresses for devices using their serial numbers via the Meraki Dashboard API.

**Before You Begin:** 
- Ensure you have **the organization ID**
- Optionally, have the **serial numbers** of devices whose uplink addresses you want to retrieve..
- Ensure you have your **bearer token** ready.
 

Follow these steps to get uplink addresses for specific devices:

1. Send a GET request to the `/organizations/:organizationId/devices/uplinks/addresses/byDevice` endpoint. For more information, see [Get organization devices uplink addresses by device](##!get-organization-devices-uplinks-addresses-by-device)
2. Include serials[] query parameters for the devices. Use the following formats for GET requests:
   - For a single device:
     ```
     GET /organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial}
     ```
   - For multiple devices:
     ```
     GET /organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial1}&serials[]={serial1}&serials[]={serial2}
     ```
3. Use `curl` to send the API request:
   - For a single device:
        ```cURL
        curl https://api.meraki.com/api/v1/organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial1}&serials[]={serial2} \
          -L -H 'Authorization: Bearer {BEARER_TOKEN}'
        ```
   - For multiple devices:
        ```cURL
        curl https://api.meraki.com/api/v1/organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial} \
          -L -H 'Authorization: Bearer {BEARER_TOKEN}'
        ```
4. Alternatively, use the Meraki Python SDK to perform the request:
   - For a single device:
        ```Python
        import meraki
        dashboard = meraki.DashboardAPI(BEARER_TOKEN)
        response = dashboard.organizations.getOrganizationDevicesUplinksAddressesByDevice({organizationId}, serials=["{serial}"])        ```
   - For multiple devices:
        ```Python
        import meraki
        dashboard = meraki.DashboardAPI(BEARER_TOKEN)
        response = dashboard.organizations.getOrganizationDevicesUplinksAddressesByDevice({organizationId}, serials=["{serial1}", "{serial2}"])
        ```
**Result**: You will obtain the uplink IP addresses (both public and private), gateways, assignment modes, and DNS configuration details for all specified devices.

### Example response for one device:

```JSON
Successful HTTP Status: 200
[
 {
  "mac": "00:11:22:33:44:55",
  "name": "My Switch 1",
  "network": {
   "id": "L_24329156"
  },
  "productType": "switch",
  "serial": "{serial}",
  "tags": [
   "example",
   "switch"
  ],
  "uplinks": [
   {
    "interface": "man1",
    "addresses": [
     {
      "protocol": "ipv4",
      "address": "10.0.1.2",
      "gateway": "10.0.1.1",
      "assignmentMode": "dynamic",
      "nameservers": {
       "addresses": [
        "208.67.222.222",
        "208.67.220.220"
       ]
      },
      "public": {
       "address": "78.11.19.49"
      }
     },
     {
      "protocol": "ipv6",
      "address": "2600:1700:ae0::c8ff:fe1e:12d2",
      "gateway": "fe80::fe1b:202a",
      "assignmentMode": "dynamic",
      "nameservers": {
       "addresses": [
        "::",
        "::"
       ]
      },
      "public": {
       "address": None
      }
     }
    ]
   }
  ]
 }
]
```

```Python
[{'mac': '00:11:22:33:44:55', 'name': 'My Switch 1', 'network': {'id': 'L_24329156'}, 'productType': 'switch', 'serial': '{serial}', 'tags': ['example', 'switch'], 'uplinks': [{'interface': 'man1', 'addresses': [{'protocol': 'ipv4', 'address': '10.0.1.2', 'gateway': '10.0.1.1', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['208.67.222.222', '208.67.220.220']}, 'public': {'address': '78.11.19.49'}}, {'protocol': 'ipv6', 'address': '2600:1700:ae0::c8ff:fe1e:12d2', 'gateway': 'fe80::fe1b:202a', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['::', '::']}, 'public': {'address': None}}]}]}]
```
### Example response for two devices:

```Python
>>> print(response)
[{'mac': '00:11:22:33:44:55', 'name': 'My Switch 1', 'network': {'id': 'L_24329156'}, 'productType': 'switch', 'serial': '{serial1}', 'tags': ['example', 'switch'], 'uplinks': [{'interface': 'man1', 'addresses': [{'protocol': 'ipv4', 'address': '10.0.1.2', 'gateway': '10.0.1.1', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['208.67.222.222', '208.67.220.220']}, 'public': {'address': '78.11.19.49'}}, {'protocol': 'ipv6', 'address': '2600:1700:ae0::c8ff:fe1e:12d2', 'gateway': 'fe80::fe1b:202a', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['::', '::']}, 'public': {'address': None}}]}]}, {'mac': '00:11:22:33:44:55', 'name': 'My Switch 2', 'network': {'id': 'L_24329156'}, 'productType': 'switch', 'serial': '{serial2}', 'tags': ['example', 'switch'], 'uplinks': [{'interface': 'man1', 'addresses': [{'protocol': 'ipv4', 'address': '10.0.1.3', 'gateway': '10.0.1.1', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['208.67.222.222', '208.67.220.220']}, 'public': {'address': '78.11.19.49'}}, {'protocol': 'ipv6', 'address': '2600:1700:ae0:f84c::9c2f', 'gateway': 'fe80::aa46::202a', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['::', '::']}, 'public': {'address': None}}]}]}]
```


# Country-specific base URI  
In most parts of the world, every API request will begin with the **base URI**:

> `https://api.meraki.com/api/v1`

For organizations hosted in a specific country, specify the respective base URI from the table:

|  Country         |  URI                              |
|------------------|-----------------------------------|
| Canada           | `https://api.meraki.ca/api/v1`    |
| China            | `https://api.meraki.cn/api/v1`    |
| India            | `https://api.meraki.in/api/v1`    |
| United States FedRAMP | `https://api.gov-meraki.com/api/v1` |


For more information about path schema, see [here](PathSchema.md). 


