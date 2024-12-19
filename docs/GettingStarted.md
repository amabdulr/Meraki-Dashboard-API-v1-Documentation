# Getting started with Meraki dashboard API

The Meraki Dashboard API is a programming interface that allows users to interact with and manage their Meraki networks programmatically. It provides access to various operations such as retrieving organizations, networks, devices, and device uplink addresses. The API supports multiple authentication methods, including bearer tokens, and offers flexibility in accessing data related to network configurations and status.

# Authorization using bearer token 
The Meraki Dashboard API requires authorization using a Bearer token (or BEARER_TOKEN), which can be
- the Meraki API Key, or
- the OAuth access token.

Include the following in your request header:

```JSON
{
 "Authorization": "Bearer <BEARER_TOKEN>"
}
```

```curl
curl https://api.meraki.com/api/v1/organizations \
  -H 'Authorization: Bearer {BEARER_TOKEN}'
```

```Python
import meraki
dashboard = meraki.DashboardAPI(BEARER_TOKEN)
```

For more information, see [Authorization](#!authorization).

# Using Postman for Meraki API 

Use Postman to explore and interact with the Meraki API.  

**Before you begin**: Ensure that you have Postman installed and your Bearer token ready. 
Follow these steps to use Postman:

- **Step 1:** Go to [Postman and our Postman collection](https://documenter.getpostman.com/view/897512/SzYXYfmJ).
- **Step 2:** Import the collection by clicking the 'Run in Postman' button.
- **Step 3:** Use your Bearer token for authorization. (comment: add link to the authorization section)
- **Step 4:** Explore the endpoints available in the collection.

**Result:** You can now interact with Meraki networks using Postman. 

# Using Python library for Meraki API 

Use the Meraki Python library to interact with the API programmatically.  

**Before you begin**: Ensure Python and pip are installed on your system.  

Follow these steps to use the Python library:

- **Step 1:** Install the library using the command `pip install meraki`.
- **Step 2:** Import the library in your Python script with `import meraki`.
- **Step 3:** Initialize the Dashboard API with `dashboard = meraki.DashboardAPI(BEARER_TOKEN)`.

**Result:** You can now perform API operations within your Python scripts.


# Finding organization ID 

Retrieve your organization ID to perform further operations.  

Follow these steps to get your organization ID:

- **Step 1:** List the organisations you have access to. Send a GET request to the `/organizations` endpoint. For more information, see [Get Organization](##!get-organizations) API.

### Example Request:

```cURL
curl https://api.meraki.com/api/v1/organizations \
  -L -H 'Authorization: Bearer {MERAKI-API-KEY}'
```

```Python
import meraki
dashboard = meraki.DashboardAPI(API_KEY)
response = dashboard.organizations.getOrganizations()
```

- **Step 2:** Retrieve the `id` from the response. This `id` is organization’s identifier.

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

```Python
>>> print(response)
[{'id': '549236', 'name': 'DevNet Sandbox'}]
```
**Note:** Irrelevant response attributes are omitted from the examples for brevity.

**Result:** You have your organization ID for subsequent API requests. 

**Post-requisite:** Note the organization ID for subsequent API requests.  

# Finding network ID 
List the networks of your organization using the organization ID.

**Before You Begin:** Ensure you have the organization ID. 

Follow these steps to get your network ID:

- **Step 1:** Send a GET request to the /organizations/:organizationId/networks endpoint. For more information, see [Get Organization Networks](##!get-organization-networks).
- **Step 2:** Provide your organization ID and Bearer Token in the request.

### Example Request: 

`GET /organizations/:organizationId/networks`

```cURL
curl https://api.meraki.com/api/v1/organizations/{organizationId}/networks \
  -L -H 'Authorization: Bearer {MERAKI-API-KEY}'
```

```Python
import meraki
dashboard = meraki.DashboardAPI(API_KEY)
response = dashboard.organizations.getOrganizationNetworks(org_id)
```

- **Step 3:** Copy the `id  from the response. This is the network identifier. 

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
**Result:** You obtain the network identifier for network-specific operations.

**Post-requisite:** Note the network identifier for subsequent API requests.

# Finding devices and serials 
List the devices in your organization to obtain the serial numbers of the devices.

**Before You Begin:** 
Ensure you have
- the organization ID, and the
- (optional) the network identifier. This parameter helps you narrow down the devices to a specific network in your organization. 

Follow these steps to find devices and their serials:

- **Step 1**: Send a GET request to the /organizations/:organizationId/devices endpoint.
- **Step 2**: Include your organization identifier and API key in the request.

### Example Request:

`GET /organizations/:organizationId/devices`

```cURL
curl https://api.meraki.com/api/v1/organizations/{organizationId}/devices \
  -L -H 'Authorization: Bearer {BEARER_TOKEN}'
```

```Python
import meraki
dashboard = meraki.DashboardAPI(BEARER_TOKEN)
response = dashboard.organizations.getOrganizationDevices({organizationId})
```
**Note:** This example does not use the network identifier parameter.

- **Step 3**: Note the value of the `serial` field of each device from the response.

### Example Response:


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

```Python
>>> print(response)
[{ 'name': 'My AP', 'lat': 37.4180951010362, 'lng': -122.098531723022, 'address': '1600 Pennsylvania Ave', 'notes': 'My AP note', 'tags': [ 'recently-added' ], 'networkId': 'N_24329156', 'serial': 'Q234-ABCD-5678', 'model': 'MR34', 'mac': '00:11:22:33:44:55', 'lanIp': '1.2.3.4', 'firmware': 'wireless-25-14', 'productType': 'wireless'}]
```

**Result**: You have the serial numbers of each device in your organization or your network. 

**Post-requisite:** Note the value of the `serial` field. You can use this value in requests requiring a serial number as a path or query parameter. If you have two devices in your organization or network, note the values of both devices.

# Getting devices uplink addresses 

Retrieve uplink addresses for specific devices using their serials.

**Before You Begin:** 
Ensure you have
- the organization ID, and the
- (optional) the serial number of devices in your network or organization. 

Follow these steps to get uplink addresses:

**Step 1**: Send a GET request to the /organizations/:organizationId/devices/uplinks/addresses/byDevice endpoint. For more information, see [Get organization devices uplink addresses by device](##!get-organization-devices-uplinks-addresses-by-device)

**Step 2**: Include serials[] query parameters for the devices.

### Example request for one device:

`GET /organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial}`

```cURL
curl https://api.meraki.com/api/v1/organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial} \
  -L -H 'Authorization: Bearer {MERAKI-API-KEY}'
```

```Python
import meraki
dashboard = meraki.DashboardAPI(API_KEY)
response = dashboard.organizations.getOrganizationDevicesUplinksAddressesByDevice({organizationId}, serials=["{serial}"])
```

### Example request for two devices:

`GET /organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial1}&serials[]={serial1}&serials[]={serial2}`

```cURL
curl https://api.meraki.com/api/v1/organizations/:organizationId/devices/uplinks/addresses/byDevice?serials[]={serial1}&serials[]={serial2} \
  -L -H 'Authorization: Bearer {MERAKI-API-KEY}'
```

```Python
import meraki
dashboard = meraki.DashboardAPI(API_KEY)
response = dashboard.organizations.getOrganizationDevicesUplinksAddressesByDevice({organizationId}, serials=["{serial1}", "{serial2}"])
```

**Result**: You receive the uplink addresses for the specified devices.

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

### Response for two devices

```Python
>>> print(response)
[{'mac': '00:11:22:33:44:55', 'name': 'My Switch 1', 'network': {'id': 'L_24329156'}, 'productType': 'switch', 'serial': '{serial1}', 'tags': ['example', 'switch'], 'uplinks': [{'interface': 'man1', 'addresses': [{'protocol': 'ipv4', 'address': '10.0.1.2', 'gateway': '10.0.1.1', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['208.67.222.222', '208.67.220.220']}, 'public': {'address': '78.11.19.49'}}, {'protocol': 'ipv6', 'address': '2600:1700:ae0::c8ff:fe1e:12d2', 'gateway': 'fe80::fe1b:202a', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['::', '::']}, 'public': {'address': None}}]}]}, {'mac': '00:11:22:33:44:55', 'name': 'My Switch 2', 'network': {'id': 'L_24329156'}, 'productType': 'switch', 'serial': '{serial2}', 'tags': ['example', 'switch'], 'uplinks': [{'interface': 'man1', 'addresses': [{'protocol': 'ipv4', 'address': '10.0.1.3', 'gateway': '10.0.1.1', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['208.67.222.222', '208.67.220.220']}, 'public': {'address': '78.11.19.49'}}, {'protocol': 'ipv6', 'address': '2600:1700:ae0:f84c::9c2f', 'gateway': 'fe80::aa46::202a', 'assignmentMode': 'dynamic', 'nameservers': {'addresses': ['::', '::']}, 'public': {'address': None}}]}]}]
```

# Country-specific base URI  
In most parts of the world, every API request will begin with the following **base URI**:

> `https://api.meraki.com/api/v1`

For organizations hosted in the following country dashboard, please specify the respective base URI instead:

|  Country         |  URI                              |
|------------------|-----------------------------------|
| Canada           | `https://api.meraki.ca/api/v1`    |
| China            | `https://api.meraki.cn/api/v1`    |
| India            | `https://api.meraki.in/api/v1`    |
| United States FedRAMP | `https://api.gov-meraki.com/api/v1` |


For more information about path schema, see [here](PathSchema.md). 


