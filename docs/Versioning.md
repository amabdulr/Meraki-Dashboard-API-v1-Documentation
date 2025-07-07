### API and API Operation Versioning

As part of Cisco's API-first strategy, Cisco Meraki is committed to optimizing the value of your integrations with its API offerings. In adherence to our API contract, we have defined a clear policy regarding breaking changes, detailing the preventative measures in place and the process we follow should such changes become necessary.

In summary, we will:

- Clearly differentiate between backward-compatible (non-breaking) and non-backward-compatible (breaking) changes to help developers understand their impact while developing applications that consume Cisco Meraki APIs.
- Avoid breaking changes as a best practice and by default.
- Provide advance notice for any necessary breaking changes through developer communication channels.

The [Cisco Meraki Dashboard API - OpenAPI Specification (OAS)](https://github.com/meraki/openapi) serves as the definitive guide for understanding the officially supported functionalities of the Cisco Meraki Dashboard API. It establishes a consistent framework that ensures clarity and reliability for developers interacting with the API.

### Version Strategy
After an API version is released, only backward-compatible changes are implemented. Here are the backward-compatible revisions:

* new API resources
* new optional request parameters to existing API methods
* new properties to existing API responses
* Change the order of properties in existing API responses

### V1 Release Schedule

|   |Week 1   |Week 2   |Week 3   |Week 4   |
|---|---|---|---|---|
|Month 1   |1.0.0   |   |   |   |
|  |1.0.0-beta   |1.0.0-beta.1   |1.0.0-beta.2   |1.0.0-beta.3   |
|Month 2   |1.1.0   |   |   |   |
|  |1.1.0-beta   |1.1.0-beta.1   |1.1.0-beta.2   |1.1.0-beta.3   |
|Month 3    |1.2.0   |   |   |   |
|  |1.2.0-beta   |1.2.0-beta.1   |1.2.0-beta.2   |1.2.0-beta.3   |

Any updates related to the Cisco Meraki Dashboard API are available in the [Release Notes](https://developer.cisco.com/meraki/whats-new/).
