# Pickup API

<a name="overview"></a>

## Overview

With this API, the client can open a pickup request at an address for shipments whose rate has been selected with [createShipment](./shipment.md#createShipment), cancel a pickup request and list its own pickup requests. A pickup request covers one or more (at most 50) shipments.

**Rules**

- The pickup time window is fixed: **10:00-15:00**.
- A request cannot be opened for a Sunday or a past date. For the same day, a request cannot be opened at 09:00 Turkey time (TRT, UTC+3; 06:00 UTC) or later; a same-day request can be opened at 08:59 TRT at the latest. A rejected date is returned as `pickupdatenotallowed.error`.
- The shipments must belong to the client, have a selected rate, not be cancelled, not have arrived at the warehouse and not be linked to another pickup request.
- The address is looked up in Navlungo's pickup coverage list by `countryCode` (ISO 3166-1 alpha-2), `city`, `town` and `postalCode` names (case and leading/trailing spaces are ignored). If the area is not found or is not served, `pickuprequests.pickuprequestedinnotavailablearea.error` is returned.
- A shipment or pickup request that belongs to another account returns `404` `record.not.found`.

### Version Information

_Version_ : v1

### URI Scheme

_Host_ : api.navlungo.com
_Host(Test)_ : api-qa.navlungo.com
_Schemes_ : HTTPS

### Accepted Input Formats

- `application/json`

### Produced Output Formats

- `application/json`

### Authorization

This API can be called with tokens issued through the OAuth2 **client_credentials** flow. The scope of each operation is stated under its own heading. The pickup scopes (**shipping_pickups_write**, **shipping_pickups_read**) are granted by default to every client, including keys created through self-service; add them to the scope parameter of the token request. See [Authorization](./README.md#authorization).

### Operations

[createPickup](#createPickup)<br>
[cancelPickup](#cancelPickup)<br>
[getPickups](#getPickups)<br>

<a name="paths"></a>

## Paths

<a name="createPickup"></a>

### POST api/shipping/v1/pickups

**Operation: createPickup**

_Scope_ : shipping_pickups_write

#### Description

Opens a single pickup request for one or more shipments. The request is created with the `created` status and the pickup request is assigned to the linked shipments. See the [Overview](#overview) for the rules.

#### Rate Limit

- At most 5 requests per 10 seconds per client

#### Parameters

| Type     | Name                    | Description    | Schema                                      |
| -------- | ----------------------- | -------------- | ------------------------------------------- |
| **Body** | **body** <br>_required_ | Pickup request | [CreatePickupRequest](#createPickupRequest) |

#### Responses

| HTTP Code | Description | Schema |
| --------- | ----------- | ------ |
| **200**   | Successful  | [PickupResponse](#pickupResponse) |
| **400**   | `pickupdatenotallowed.error` - The pickup date is invalid (Sunday, past date, or same day at 09:00 TRT or later)<br> `shipmentratenotselected.error` - No rate has been selected for the shipment<br> `shipmentcancelled.error` - The shipment is cancelled<br> `shipmentalreadyinwarehouse.error` - The shipment has arrived at the warehouse<br> `pickuprequests.pickuprequestedinnotavailablearea.error` - The address was not found in the pickup coverage or the area is not served<br> `pickuprequests.alreadyexistrequestforcurrentshipments.error` - One of the shipments already has a pickup request<br> `pickuprequests.faileduetocarrierexception.error`, `pickuprequests.faileduetoproposalexception.error`, `pickuprequests.failedduetochargeableweightorshipmentcountexception.error` - The pickup rules do not allow the request | [Error](#error) |
| **400**   | Field validation error (e.g. `shipmentReferences` empty or more than 50, missing `address`/`contact` fields, invalid email, `note` longer than 500 characters) | [ProblemDetails](#problemDetails) |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_pickups_write` scope | [ProblemDetails](#problemDetails) |
| **404**   | `record.not.found` - One of the shipments was not found or belongs to another account | [Error](#error) |
| **429**   | Rate limit exceeded | [Error](#error) |
| **500**   | An unexpected error occurred while processing the request | [ProblemDetails](#problemDetails) |

#### Example Request Body

```
{
  "shipmentReferences": [458213, 458214],
  "pickupDate": "10.11.2026 00:00",
  "address": {
    "countryCode": "TR",
    "city": "Ankara",
    "town": "Cankaya",
    "postalCode": "06690",
    "addressLine": "Ataturk Bulvari No 1"
  },
  "contact": {
    "name": "Ayse Yilmaz",
    "email": "ayse@example.com",
    "phone": { "code": "+90", "number": "5551112233" }
  },
  "note": "Zili calin"
}
```

#### Example Response

```
{
  "reference": "1793000000",
  "status": "created",
  "pickupDate": "10.11.2026 00:00",
  "timeWindowStart": "10:00",
  "timeWindowEnd": "15:00",
  "address": { "countryCode": "TR", "city": "Ankara", "town": "Cankaya", "postalCode": "06690", "addressLine": "Ataturk Bulvari No 1" },
  "contact": { "name": "Ayse Yilmaz", "email": "ayse@example.com", "phoneCode": "+90", "phoneNumber": "5551112233" },
  "shipmentReferences": [458213, 458214],
  "note": "Zili calin",
  "createdAt": "05.11.2026 10:15",
  "cancelReason": null
}
```

#### Request Model

<a name="createPickupRequest"></a>

##### CreatePickupRequest

| Name                                 | Description                                                                                                         | Schema                                        |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **shipmentReferences** <br>_required_ | Shipment references to be picked up. At least 1, at most 50; each greater than 0. Duplicate values are de-duplicated | long[]                                        |
| **pickupDate** <br>_required_        | Pickup day, `dd.MM.yyyy HH:mm` (e.g. `10.11.2026 00:00`; the time part is ignored, ISO 8601 `2026-11-10T00:00:00` is also accepted); the time window is fixed at 10:00-15:00 | string (DateTime)                                 |
| **address** <br>_required_           | Pickup address                                                                                                      | [PickupAddress](#pickupAddress)               |
| **contact** <br>_required_           | Contact person at the address                                                                                       | [PickupContactRequest](#pickupContactRequest) |
| **note** <br>_optional_              | Note for the courier. At most 500 characters                                                                        | string                                        |

<a name="pickupAddress"></a>

##### PickupAddress

| Name                          | Description                                                             | Schema |
| ----------------------------- | ----------------------------------------------------------------------- | ------ |
| **countryCode** <br>_required_ | Country code, ISO 3166-1 alpha-2 (e.g. `TR`)                           | string |
| **city** <br>_required_       | City name (must match the name in the pickup coverage list)             | string |
| **town** <br>_required_       | Town (district) name (must match the name in the pickup coverage list)  | string |
| **postalCode** <br>_required_ | Postal code (must match the code in the pickup coverage list)           | string |
| **addressLine** <br>_required_ | Street address. At most 500 characters                                 | string |

<a name="pickupContactRequest"></a>

##### PickupContactRequest

| Name                    | Description        | Schema                      |
| ----------------------- | ------------------ | --------------------------- |
| **name** <br>_required_  | Full name. At most 50 characters          | string                      |
| **email** <br>_required_ | Valid email address. At most 100 characters | string                     |
| **phone** <br>_required_ | Phone              | [PickupPhone](#pickupPhone) |

<a name="pickupPhone"></a>

##### PickupPhone

| Name                     | Description                  | Schema |
| ------------------------ | ---------------------------- | ------ |
| **code** <br>_required_   | Country dialing code (`+90`). At most 25 characters | string |
| **number** <br>_required_ | Phone number. At most 21 characters                 | string |

---

<a name="cancelPickup"></a>

### POST api/shipping/v1/pickups/{reference}/cancel

**Operation: cancelPickup**

_Scope_ : shipping_pickups_write

#### Description

Cancels the pickup request. Only requests in the `created` and `scheduled` statuses can be cancelled; after cancellation the linked shipments are released and can be added to a new pickup request. The response is the updated pickup object with `status: "cancelled"` and a populated `cancelReason`.

#### Rate Limit

- At most 5 requests per 10 seconds per client

#### Parameters

| Type     | Name                        | Description                                                                                         | Schema                                      |
| -------- | --------------------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| **Path** | **reference** <br>_required_ | Pickup request reference (`reference` in the [createPickup](#createPickup) response, e.g. `1793000000`) | string                                      |
| **Body** | **body** <br>_required_     | Cancellation reason                                                                                 | [CancelPickupRequest](#cancelPickupRequest) |

#### Responses

| HTTP Code | Description | Schema |
| --------- | ----------- | ------ |
| **200**   | Successful  | [PickupResponse](#pickupResponse) |
| **400**   | `pickuprequests.wrongstatus.error` - The request cannot be cancelled in this status (already picked up or already cancelled) | [Error](#error) |
| **400**   | Field validation error (`reason` empty or longer than 500 characters) | [ProblemDetails](#problemDetails) |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_pickups_write` scope | [ProblemDetails](#problemDetails) |
| **404**   | `record.not.found` - The pickup request was not found or belongs to another account | [Error](#error) |
| **429**   | Rate limit exceeded | [Error](#error) |
| **500**   | An unexpected error occurred while processing the request | [ProblemDetails](#problemDetails) |

#### Example Request Body

```
{ "reason": "Planlar degisti" }
```

#### Example Response

```
{
  "reference": "1793000000",
  "status": "cancelled",
  "pickupDate": "10.11.2026 00:00",
  "timeWindowStart": "10:00",
  "timeWindowEnd": "15:00",
  "address": { "countryCode": "TR", "city": "Ankara", "town": "Cankaya", "postalCode": "06690", "addressLine": "Ataturk Bulvari No 1" },
  "contact": { "name": "Ayse Yilmaz", "email": "ayse@example.com", "phoneCode": "+90", "phoneNumber": "5551112233" },
  "shipmentReferences": [458213, 458214],
  "note": "Zili calin",
  "createdAt": "05.11.2026 10:15",
  "cancelReason": "Planlar degisti"
}
```

#### Request Model

<a name="cancelPickupRequest"></a>

##### CancelPickupRequest

| Name                    | Description                               | Schema |
| ----------------------- | ----------------------------------------- | ------ |
| **reason** <br>_required_ | Cancellation reason. At most 500 characters | string |

---

<a name="getPickups"></a>

### GET api/shipping/v1/pickups

**Operation: getPickups**

_Scope_ : shipping_pickups_read

#### Description

Lists the client's pickup requests with paging; the newest request comes first.

#### Rate Limit

- At most 20 requests per 10 seconds per client

#### Parameters

| Type      | Name                                  | Description                                                                          | Schema |
| --------- | ------------------------------------- | ------------------------------------------------------------------------------------ | ------ |
| **Query** | **currentPage** <br>_required_        | Page number. At least 1                                                              | int    |
| **Query** | **pageSize** <br>_required_           | Page size. Between 1 and 100                                                         | int    |
| **Query** | **status** <br>_optional_             | Status filter: `created`, `scheduled`, `picked-up`, `cancelled`                      | string |
| **Query** | **shipmentReference** <br>_optional_  | Shipment reference; only requests that cover this shipment are returned. Greater than 0 | long   |

#### Responses

| HTTP Code | Description | Schema |
| --------- | ----------- | ------ |
| **200**   | Successful  | [PickupPagedResponse](#pickupPagedResponse) |
| **400**   | Field validation error (`currentPage` or `pageSize` missing or invalid, invalid `status` value, `shipmentReference` 0 or less) | [ProblemDetails](#problemDetails) |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_pickups_read` scope | [ProblemDetails](#problemDetails) |
| **429**   | Rate limit exceeded | [Error](#error) |
| **500**   | An unexpected error occurred while processing the request | [ProblemDetails](#problemDetails) |

#### Example Request

```
GET /api/shipping/v1/pickups?currentPage=1&pageSize=10&status=created&shipmentReference=458213
```

#### Example Response

```
{
  "currentPage": 1,
  "pageCount": 1,
  "pageSize": 10,
  "totalRecordCount": 1,
  "results": [
    {
      "reference": "1793000000",
      "status": "created",
      "pickupDate": "10.11.2026 00:00",
      "timeWindowStart": "10:00",
      "timeWindowEnd": "15:00",
      "address": { "countryCode": "TR", "city": "Ankara", "town": "Cankaya", "postalCode": "06690", "addressLine": "Ataturk Bulvari No 1" },
      "contact": { "name": "Ayse Yilmaz", "email": "ayse@example.com", "phoneCode": "+90", "phoneNumber": "5551112233" },
      "shipmentReferences": [458213, 458214],
      "note": "Zili calin",
      "createdAt": "05.11.2026 10:15",
      "cancelReason": null
    }
  ]
}
```

#### Response Model

<a name="pickupPagedResponse"></a>

##### PickupPagedResponse

| Name                               | Description         | Schema                              |
| ---------------------------------- | ------------------- | ----------------------------------- |
| **currentPage** <br>_required_     | Current page        | int                                 |
| **pageCount** <br>_required_       | Total page count    | int                                 |
| **pageSize** <br>_required_        | Page size           | int                                 |
| **totalRecordCount** <br>_required_ | Total record count | int                                 |
| **results** <br>_required_         | Pickup requests     | [PickupResponse](#pickupResponse)[] |

---

## Common Models

<a name="pickupResponse"></a>

### PickupResponse

| Name                                  | Description                                                              | Schema                                          |
| ------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------- |
| **reference** <br>_required_          | Pickup request reference                                                 | string                                          |
| **status** <br>_required_             | Request status, see [Statuses](#pickupStatuses)                          | string                                          |
| **pickupDate** <br>_required_         | Pickup day (`dd.MM.yyyy HH:mm`, time is `00:00`)                         | string (DateTime)                               |
| **timeWindowStart** <br>_required_    | Start of the pickup window (`HH:mm`), currently always `10:00`           | string                                          |
| **timeWindowEnd** <br>_required_      | End of the pickup window (`HH:mm`), currently always `15:00`             | string                                          |
| **address** <br>_required_            | Pickup address                                                           | [PickupAddress](#pickupAddress)                 |
| **contact** <br>_required_            | Contact person                                                           | [PickupContactResponse](#pickupContactResponse) |
| **shipmentReferences** <br>_required_ | References of the shipments linked to the request                        | long[]                                          |
| **note** <br>_required_               | Courier note; `null` if none                                             | string                                          |
| **createdAt** <br>_required_          | Creation time of the request (UTC)                                       | string (DateTime)                               |
| **cancelReason** <br>_required_       | Cancellation reason; `null` if not cancelled                             | string                                          |

<a name="pickupContactResponse"></a>

### PickupContactResponse

| Name                          | Description        | Schema |
| ----------------------------- | ------------------ | ------ |
| **name** <br>_required_        | Full name          | string |
| **email** <br>_required_       | Email address      | string |
| **phoneCode** <br>_required_   | Phone country code | string |
| **phoneNumber** <br>_required_ | Phone number       | string |

<a name="pickupStatuses"></a>

### Statuses

| Value       | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `created`   | The request has been created                                         |
| `scheduled` | The request has been scheduled (sent to the route planning service)  |
| `picked-up` | The shipments have been picked up from the address                   |
| `cancelled` | The request has been cancelled (by the user, an admin or the courier/operator) |

<a name="error"></a>

### Error

Business rule error object

| Name                      | Description                    | Schema |
| ------------------------- | ------------------------------ | ------ |
| **code** <br>_required_    | Error code                     | string |
| **message** <br>_required_ | Error message                  | string |
| **data** <br>_optional_    | Additional information on the error | array |

<a name="problemDetails"></a>

### ProblemDetails

Validation and authorization error object

| Name                            | Description                                                                 | Schema |
| ------------------------------- | --------------------------------------------------------------------------- | ------ |
| **type** <br>_required_         | Error type (as a path, e.g. Authentication/InvalidToken)                    | string |
| **status** <br>_required_       | Status code of the error                                                    | int    |
| **problemCode** <br>_optional_  | Error code                                                                  | string |
| **title** <br>_required_        | Error title                                                                 | string |
| **detail** <br>_required_       | Detailed description of the error                                           | string |
| **path** <br>_required_         | The url where the error occurred                                            | string |
| **extensions** <br>_optional_   | Detail information on the error. Its content may vary depending on the error type | object |
