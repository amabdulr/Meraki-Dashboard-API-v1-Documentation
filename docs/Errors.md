APIs return response codes that help you understand the status of the API operation.  The response codes use standard HTTP status codes. 

Here are examples of common response codes and their descriptions:

## Common Response Codes and Descriptions

**Response Code**|**Description**
 :-------------: |:-------------:
200 - OK| 	Everything worked as expected.
400 - Bad Request| 	The request was unacceptable, often due to missing a required parameter.
401 - Unauthorized| Incorrect API key
403 - Forbidden| You don't have permissions to do that.
404 - Not Found|	The requested resource doesn't exist.
429 - Too Many Requests|	Too many requests hit the API too quickly. We recommend an exponential backoff of your requests.
500, 502, 503, 504 - Server Errors|	Meraki was unable to process the request.

## Error Responses

Some of the response codes indicate exceptions where something has failed, or that there are missing or invalid parameters.  
**Recommendation**: Ensure that your code gracefully handles all possible API exceptions.

**Insufificent response code** 
If the response code is not specific enough to determine the cause of the issue, you can also check the error messages included in the response in JSON format.  

```
{
    "errors": [
       "VLANs are not enabled for this network"
    ]
}
```

##  Unknown 5xx response code

When you interact with the Meraki dasbhoard via API, you may want details of the requests that successfully reache the Meraki dashboard.  You can find such customer-facing telemetry using the [apiRequests](https://developer.cisco.com/meraki/api-v1/search/?q=api%20requests) operations. The telemetry information provides insight into most issues that can affect the API client experience.

The response code of the [apiRequests](https://developer.cisco.com/meraki/api-v1/search/?q=api%20requests) operations are generally between 200 to 500.

Sometimes, the response code is outside this range, such as 502 or 504, and hence not part of the customer-facing telemetry. The response code indicates that the request is likely to have not reached the Meraki dashboard, and that there could be an issue between the API client and the Meraki dasbboard. 

Ocassional occurences of such API errors (where the response code is higher than 500) do not interfere with the operation of a well-developed application. However, if the number of such errors are high, contact Meraki Support for assistance.
