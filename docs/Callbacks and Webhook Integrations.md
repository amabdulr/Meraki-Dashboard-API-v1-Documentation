# Callbacks and Webhook Integrations

This chapter explains how callbacks and webhooks operate within the Meraki Dashboard API, highlighting when and why to use them, the differences between callbacks and alerts, and the structure and purpose of callback responses.

---

## Callbacks

A callback is a notification mechanism that

* provides asynchronous results for long-running API operations,
* uses webhooks to deliver data when operations complete, and
* complies with OpenAPI v3 specifications for asynchronous processing.

Callbacks let you receive results from operations like Live Tools or Action Batches without needing to poll for updates. This approach improves efficiency and scalability in network management.

Note: The Callback implementation aligns with the OpenAPI v3 specification for standardized asynchronous operations.

---

## Webhooks

A webhook is an outbound API request that

* originates from Meraki to an external service,
* transmits structured data (such as callback or alert payloads), and
* supports templating for customizing message formats.

Callbacks leverage the Meraki webhook system, supporting both pre-configured receivers and dynamic URLs. In addition, webhook templates can format the HTTP message body and headers for customized integrations.

[Meraki Webhooks Guide](https://meraki.io/webhooks)

---

## Differences between alerts and callbacks

Alerts and callbacks are webhook-driven notifications that differ in purpose and schema:

* An **alert** is an event-driven notification for system occurrences like device outages or sensor triggers. It is identified by an `alertId` and contains its data in the `alertData` object.
* A **callback** delivers the result of an asynchronous API operation such as an action batch or a device ping. It uses a `callbackId` and embeds results in the `message` object.

### Contrast table

| Feature          | Alerts              | Callbacks                          |
| ---------------- | ------------------- | ---------------------------------- |
| Purpose          | Event notification  | Result delivery for API operations |
| Identifier       | `alertId`           | `callbackId`                       |
| Data location    | `alertData`         | `message`                          |
| Trigger source   | System event        | API operation                      |
| Typical use case | Device goes offline | Action batch completes             |

---
### Example webhook for callbacks

```json
{
  "callbackId": "643451796760560204",
  "organization": {
    "id": "123",
    "name": "Org in Company"
  },
  "message": {
    "pingId": "643451796835419513",
    "status": "complete",
	...
  }
}
```
### Example webhook for alerts

```json
{
  "version": "0.1",
  "sharedSecret": "secret",
  "sentAt": "2022-09-07T14:14:14.591941Z",
  "organizationId":"123",
  "organizationName":"Org in Company",
  "alertType": "APs went down",
  "alertLevel": "critical",
  "alertData": {
  ...
  }
}
```

## Use cases for callbacks 

Callbacks are suitable for workflows where operations require time to process and immediate API responses are not essential. However, timely notification upon completion is critical. When you use callbacks, you optimize resource usage, reduce the need for continuous polling, and streamline workflows


* **Bulk configuration**: Enables large-scale device changes without synchronous blocking, reducing API rate-limit risks.

When pushing configuration changes to hundreds or thousands of devices, waiting for each action batch to respond synchronously can be time-consuming or may cause rate-limit issues. With callbacks, you initiate the asynchronous requests and receive updates as each job completes its configurations.

* **Monitoring**: Facilitates event-driven updates to messaging tools or databases without polling.

 In large networks, continuously polling each request for status is inefficient and generates significant API usage. Callbacks enable an event-driven approach, where results can be sent to a group messaging service or database when they are available. 

* **Event-driven automation**: Supports downstream processing after a job finishes, triggering next steps in automation pipelines.

Use callbacks to trigger other processes or workflows within your system. For example, once a request completes and a webhook has been sent, an automated system can continue its next operation using those results.

---

## Callback status endpoint 

The callback status endpoint provides detailed information about an operation initiated using a callback.

**Operation**: `GET /organizations/{organizationId}/webhooks/callbacks/statuses/{callbackId}`

[API Docs](https://developer.cisco.com/meraki/api-v1/get-organization-webhooks-callbacks-status/)


### Response schema

These are the fields that are returned from the `/callbacks/statuses` endpoint.

* `callbackId`: Unique ID for the callback.
* `status`: Current status of the callback.
* `errors`: List of error messages, if any.
* `createdBy`: Information about the user or admin who triggered the callback.
* `webhook`: Details about the webhook used for the callback, including:
	* `url`: The receiver URL for the callback results.
 	* `httpServer`: Information about the HTTP server that receives the callback data.
  	* `payloadTemplate`: Details about the payload template used for the callback.
  	* `sentAt`: Timestamp indicating when the callback was dispatched to the webhook receiver.


#### Status values
This table lists the possible `status` values.

| Status      | Meaning                                  |
| ----------- | ---------------------------------------- |
| `completed` | The operation has successfully finished. |
| `failed`    | The operation encountered an error.      |
| `running`   | The operation is still in progress.   

---
##### Example response

An example JSON response from the `/callbacks/statuses` endpoint:

```json
{
  "callbackId": "1284392014819",
  "status": "completed",
  "errors": ["Callback failed"],
  "createdBy": {
    "adminId": "212406"
  },
  "webhook": {
    "url": "https://webhook.site/28efa24e-f830-4d9f-a12b-fbb9e5035031",
    "httpServer": {
      "id": "aHR0cHM6Ly93d3cuZXhhbXBsZS5jb20vd2ViaG9va3M="
    },
    "payloadTemplate": {
      "id": "wpt_2100"
    },
    "sentAt": "2018-02-11T00:00:00.090210Z"
  }
}
```

### Differentiate alerts and callbacks using tailored webhook templates 

Webhook payloads can represent either alerts or callbacks, distinguished by the type of event and identifying fields within the payload.

- Alert payloads include an `alertId` field. They indicate an event notification, such as a triggered rule.
- Callback payloads include a `callbackId` field. They represent responses to asynchronous operations or requests.
- If neither field is present, the payload type may be unknown or unsupported.

#### Liquid template for distinguishing payload types

```liquid
{% if callbackId %}
  {# Handle as Callback #}

{% elsif alertId %}
  {# Handle as Webhook Alert #}

{% else %}
  {# Unknown payload type #}
{% endif %}
```

To customize the structure and security of your callback webhooks, refer to the official documentation:

[Webhook Payload Templates Guide](https://developer.cisco.com/meraki/webhooks/payload-templates-overview/)
