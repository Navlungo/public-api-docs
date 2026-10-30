# Shipment Tracking API

<a name="overview"></a>

## Overview

With this API, the client can query the tracking summary and tracking events of a shipment by its shipment reference. The tracking number is assigned shortly after the rate is selected; queries made before it is assigned return `trackingnotfound.error` and should be retried.

### Version Information

_Version_ : v1

### URI scheme

_Host_ : api.navlungo.com
_Host(Test)_ : api-qa.navlungo.com
_Schemes_ : HTTPS

### Accepted Input Formats

- `application/json`

### Produced Output Formats

- `application/json`

### Authorization

This API can be called with tokens issued through the OAuth2 **client_credentials** flow. When obtaining the token, the value **shipping_shipments_read** must be sent in the scope parameter. The `lastMileTrackingNumber` field is only returned if the client has the **can_access_last_mile_tracking** permission.

### Operations

[getTracking](#getTracking)<br>

<a name="paths"></a>

## Paths

<a name="getTracking"></a>

### GET api/shipping/v1/shipments/{reference}/tracking

**Operation: getTracking**

#### Description

Returns the tracking summary and the list of tracking events by the shipment reference.

#### Rate Limit

- At most 20 requests per 10 seconds per client

#### Parameters

| Type     | Name                         | Description                                                                                       | Schema |
| -------- | ---------------------------- | ------------------------------------------------------------------------------------------------- | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number (the `reference` in the [createRates](./rates.md#createRates) response) | long   |

#### Responses

| HTTP Code | Description                                                                                                                               | Schema                                |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| **200**   | Successful                                                                                                                                | [TrackingResponse](#trackingResponse) |
| **400**   | `trackingnotfound.error` - Tracking number not assigned yet, try again<br> Other codes are passed through from the tracking service as is | [Error](#error)                       |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_shipments_read` scope                        | [ProblemDetails](#problemDetails)     |
| **404**   | `record.not.found` - Shipment not found or belongs to another account                                                                     | [Error](#error)                       |
| **429**   | Rate limit exceeded                                                                                                                       | [Error](#error)                       |
| **500**   | An unexpected error occurred during the request                                                                                           | [ProblemDetails](#problemDetails)     |

#### Example Response

```
{
  "reference": 306889,
  "trackingNumber": "NVL000000306889",
  "lastMileTrackingNumber": "1Z9A77910409209526",
  "originCountry": "TUR", "destinationCountry": "USA",
  "pickupDate": "23.09.2026 08:29", "expectedDeliveryDate": null,
  "carrier": "NVL", "originLocation": "İstanbul İstanbul,34710,TR,Turkey TUR", "destinationLocation": "New York New York,10001,US,United States  USA",
  "checkpoints": [
    { "checkpointTime": "23.09.2026 09:22", "status": "In Transit", "subStatusMessage": "In Transit", "subStatusDescription": "Shipment on the way", "country": "TUR", "city": "İstanbul", "zip": "34710", "location": "İstanbul,34710,TR,Turkey" },
    { "checkpointTime": "23.09.2026 08:29", "status": "Info Received", "subStatusMessage": "Info Received", "subStatusDescription": "The carrier received a request from the shipper and is about to pick up the shipment", "country": "TUR", "city": "İstanbul", "zip": "34710", "location": "İstanbul,34710,TR,Turkey" }
  ]
}
```

<a name="definitions"></a>

## Definitions

<a name="trackingResponse"></a>

### TrackingResponse

| Name                       | Description                                                                                                             | Always present | Schema                              |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------- | ----------------------------------- |
| **reference**              | Shipment reference                                                                                                      | Yes            | long                                |
| **trackingNumber**         | Navlungo tracking number (`NVL` + the reference zero-padded to 12 digits)                                               | Yes            | string                              |
| **lastMileTrackingNumber** | Carrier tracking number. Only included in the response if the client has the `can_access_last_mile_tracking` permission | No             | string                              |
| **originCountry**          | Origin country (ISO 3166-1 alpha-3, e.g. `TUR`)                                                                         | Yes            | string                              |
| **destinationCountry**     | Destination country (ISO 3166-1 alpha-3, e.g. `USA`)                                                                    | Yes            | string                              |
| **pickupDate**             | Pickup date                                                                                                             | Yes            | string (DateTime)                   |
| **expectedDeliveryDate**   | Expected delivery date                                                                                                  | No             | string (DateTime)                   |
| **carrier**                | Carrier code in the tracking service (e.g. `NVL`)                                                                       | Yes            | string                              |
| **originLocation**         | Origin location detail                                                                                                  | Yes            | string                              |
| **destinationLocation**    | Destination location detail                                                                                             | Yes            | string                              |
| **checkpoints**            | Tracking events of the shipment                                                                                         | Yes            | < [Checkpoint](#checkpoint) > array |

<a name="checkpoint"></a>

### Checkpoint

| Name                     | Description                                                             | Always present | Schema            |
| ------------------------ | ----------------------------------------------------------------------- | -------------- | ----------------- |
| **checkpointTime**       | Event time                                                              | Yes            | string (DateTime) |
| **status**               | Event status. See [Tracking Statuses](#trackingStatuses) for the values | Yes            | string            |
| **subStatusMessage**     | Sub-status message                                                      | Yes            | string            |
| **subStatusDescription** | Sub-status description                                                  | No             | string            |
| **country**              | Country where the event occurred (alpha-3)                              | No             | string            |
| **city**                 | City where the event occurred                                           | No             | string            |
| **zip**                  | Postal code                                                             | No             | string            |
| **location**             | Location detail                                                         | No             | string            |

<a name="trackingStatuses"></a>

### Tracking Statuses

`checkpoints` are ordered from newest to oldest; the current status of the shipment is the `status` value of the first element.

`status` is returned from the fixed list below. Events originating from Navlungo use the spaced form (`In Transit`); carrier (last mile) events use the concatenated form (`InTransit`). Compare by ignoring spaces and case.

| status                                        | Description                                                                                                                             |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `Pending`                                     | No tracking information at the carrier yet                                                                                              |
| `Info Received` / `InfoReceived`              | Shipment information received, the shipment has not been picked up yet                                                                  |
| `In Transit` / `InTransit`                    | Shipment on the way (including transit hubs and customs processing)                                                                     |
| `Out For Delivery` / `OutForDelivery`         | Shipment out for delivery                                                                                                               |
| `Attempt Fail` / `AttemptFail`                | Delivery attempt failed, the carrier usually tries again                                                                                |
| `Available For Pickup` / `AvailableForPickup` | Shipment is at the pickup point, waiting for the receiver to collect it                                                                 |
| `Delivered`                                   | Shipment delivered                                                                                                                      |
| `Exception`                                   | There is a problem with the delivery (customs delay, wrong address, refusal, return, etc.); details are in the `subStatusMessage` field |
| `Expired`                                     | No tracking information has been received for a long time                                                                               |

---

## Common Models

<a name="error"></a>

### Error

Business rule error object

| Name                       | Description                            | Schema |
| -------------------------- | -------------------------------------- | ------ |
| **code** <br>_required_    | Error code                             | string |
| **message** <br>_required_ | Error message                          | string |
| **data** <br>_optional_    | Additional information about the error | array  |

<a name="problemDetails"></a>

### ProblemDetails

Validation and authorization error object

| Name                           | Description                                                            | Schema |
| ------------------------------ | ---------------------------------------------------------------------- | ------ |
| **type** <br>_required_        | Error type (in path form, e.g. Authentication/InvalidToken)            | string |
| **status** <br>_required_      | Status code of the error                                               | int    |
| **problemCode** <br>_optional_ | Error code                                                             | string |
| **title** <br>_required_       | Error title                                                            | string |
| **detail** <br>_required_      | Detailed description of the error                                      | string |
| **path** <br>_required_        | The URL where the error occurred                                       | string |
| **extensions** <br>_optional_  | Detail information about the error. Its content may vary by error type | object |
