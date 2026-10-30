# Shipment Tracking API

<a name="overview"></a>

## Overview

With this API, the client can query the tracking information of a cargo in real time. The details of a new tracking request become available within 10-60 minutes after Navlungo registers the query and retrieves its details.

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

This API can be called with tokens issued through the OAuth2 **client_credentials** flow. When obtaining the token, the value **public_cargo_tracking** must be sent in the scope parameter.

### Operations

- [getShipmentTracking](#getShipmentTracking)<br>

<a name="paths"></a>

## Paths

<a name="getShipmentTracking"></a>

### GET shipments/v1/{reference}/tracking

**Operation: getShipmentTracking**

#### Description

Returns the cargo tracking information by the shipment reference number.

#### Parameters

| Type     | Name                         | Description               | Schema |
| -------- | ---------------------------- | ------------------------- | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number | long   |

#### Responses

| HTTP Code | Description                                                      | Schema                                |
| --------- | ---------------------------------------------------------------- | ------------------------------------- |
| **200**   | Successful                                                       | [TrackingResponse](#trackingResponse) |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error)                       |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error)                       |
| **404**   | Shipment not found.                                              | [Error](#error)                       |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error)                       |

<a name="definitions"></a>

## Definitions

<a name="trackingResponse"></a>

### TrackingResponse

| Name                     | Description                                | Always present | Schema                              |
| ------------------------ | ------------------------------------------ | -------------- | ----------------------------------- |
| **trackingNumber**       | Tracking number                            | Yes            | string                              |
| **originCountry**        | Code of the country the cargo departs from | Yes            | string                              |
| **destinationCountry**   | Delivery country code                      | Yes            | string                              |
| **pickupDate**           | Pickup date                                | Yes            | string (DateTime)                   |
| **expectedDeliveryDate** | Expected delivery date                     | No             | string (DateTime)                   |
| **carrier**              | Carrier company code                       | Yes            | string                              |
| **originLocation**       | Origin location detail                     | Yes            | string                              |
| **destinationLocation**  | Destination location detail                | Yes            | string                              |
| **checkpoints**          | Tracking events of the shipment            | Yes            | < [Checkpoint](#checkpoint) > Array |

<a name="checkpoint"></a>

### Checkpoint

| Name                                | Description                      | Always present | Schema            |
| ----------------------------------- | -------------------------------- | -------------- | ----------------- |
| **checkpointTime** <br>_required_   | Event time                       | Yes            | string (DateTime) |
| **status** <br>_required_           | Event status                     | Yes            | string            |
| **subStatusMessage** <br>_required_ | Sub-status message               | Yes            | string            |
| **subStatusDescription**            | Sub-status description           | No             | string            |
| **country**                         | Country where the event occurred | No             | string            |
| **city**                            | City where the event occurred    | No             | string            |
| **zip**                             | Postal code                      | No             | string            |
| **location**                        | Location detail                  | No             | string            |

<a name="error"></a>

### Error

General error object

| Name                           | Description                                                            | Schema |
| ------------------------------ | ---------------------------------------------------------------------- | ------ |
| **type** <br>_required_        | Error type (in path form, e.g. Authentication/InvalidToken)            | string |
| **status** <br>_required_      | Status code of the error                                               | int    |
| **problemCode** <br>_optional_ | Error code                                                             | string |
| **title** <br>_required_       | Error title                                                            | string |
| **detail** <br>_required_      | Detailed description of the error                                      | string |
| **path** <br>_required_        | The URL where the error occurred                                       | string |
| **extensions** <br>_optional_  | Detail information about the error. Its content may vary by error type | object |
