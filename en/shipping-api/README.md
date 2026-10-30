# Navlungo Shipping API

> 🇹🇷 [Türkçe](../../shipping-api/README.md)

<a name="overview"></a>

## Important Notes

1. The client credentials provided to you can only be used in the corresponding environment. You cannot access the production APIs with the client credentials provided for the QA environment.
2. The QA environment is shut down by default on weekday nights and at weekends, so you may not get a response from the APIs. If you plan your work and let us know in advance, we can keep the environment up.
3. The client secret is shared only once, when the client is created, and is not stored on the Navlungo side. If you lose your secret, a new client must be defined.

## 1. Overview

With the Shipping API, Navlungo solution partners can connect **server to server** from their own systems to get rates, create shipments, manage labels and documents, track shipments and void them. Unlike the [Store API](../store-api/README.md), this API does not act on behalf of a Navlungo user but directly on behalf of the **client**; the client is linked by Navlungo to the solution partner's account (shipper) and all shipments are created under this account.

- When a rate is selected, the shipment is created in the "awaiting warehouse arrival" status; payment is not collected through this API, it is handled on the Navlungo side when the shipment arrives at the warehouse.

---

<a name="authorization"></a>

## 2. Authorization

All resources of this API are called with an access token obtained through the OAuth2 **client_credentials** flow. The token is obtained from the Navlungo identity server and sent in the `Authorization: Bearer <access_token>` header. See the [Token API](./token.md) for details.

Since the client credentials flow requires sending the client_id and client_secret, token requests must be made by a secure **server side** application. No refresh_token is issued in this flow; when the token expires, a new token is obtained.

<a name="scopes"></a>

### 2.1. Scopes

Each endpoint is protected by a scope. When obtaining a token, the scopes of the endpoints to be used are sent separated by spaces. If a scope that is not defined for the client is requested, no token is issued.

| Scope                        | Endpoints                                                                                                                                       |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **shipping_rates_write**     | [createRates](./rates.md#createRates)                                                                                                           |
| **shipping_shipments_write** | [createShipment](./shipment.md#createShipment), [uploadDocuments](./shipment.md#uploadDocuments), [voidShipment](./shipment.md#voidShipment)    |
| **shipping_shipments_read**  | [getShipment](./shipment.md#getShipment), [getTracking](./cargoTracking.md#getTracking), [getEtgbDownloadUrl](./shipment.md#getEtgbDownloadUrl) |
| **shipping_labels_write**    | [createLabel](./shipment.md#createLabel)                                                                                                        |

If the token does not contain the required scope, the endpoint returns `401` with an error of type `Authentication/InvalidScope`.

<a name="claims"></a>

### 2.2. Permissions (Client Claims)

Some capabilities are granted in the client definition, independently of scopes. These permissions are defined when the client is created and can later be added by Navlungo; an added permission takes effect with a **newly obtained token**.

| Permission                        | If missing                                                                                                                    |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **can_access_last_mile_tracking** | The `lastMileTrackingNumber` and `lastMileCarrier` fields are not included in tracking and label responses                    |
| **can_create_labels**             | [createLabel](./shipment.md#createLabel) returns `401` `Authentication/InvalidClaim`                                          |
| **can_download_labels**           | The `label` field is not included in the label response                                                                       |
| **can_create_domestic_shipments** | A shipment whose sender and receiver countries are the same gets `400` `shipment.fromanddestinationcountrycannotbesame.error` |

---

## 3. Shipment Creation Flow

**Definitions**

- **Client**: The organization that uses the Navlungo Shipping API (holds a client_id and client_secret). It is linked to an account (shipper) on the Navlungo side; all shipments are created under this account.
- **reference**: The Navlungo shipment reference. It is returned in the [createRates](./rates.md#createRates) response and identifies the shipment in all subsequent calls.

**Flow**

1. The client sends the address, product and package information with [createRates](./rates.md#createRates). Navlungo opens a shipment record and returns the `reference`, `searchId` and the list of rates (`rates`).
2. The client selects a rate by sending the `reference`, `searchId` and the `rateId` of the chosen rate with [createShipment](./shipment.md#createShipment). The shipment moves to the "awaiting warehouse arrival" status and a Navlungo tracking number is assigned.
3. Optionally, a carrier label is created with [createLabel](./shipment.md#createLabel) and documents are added to the shipment with [uploadDocuments](./shipment.md#uploadDocuments).
4. The shipment status and tracking events are monitored with [getShipment](./shipment.md#getShipment) and [getTracking](./cargoTracking.md#getTracking).
5. A shipment that has not yet arrived at the warehouse can be voided with [voidShipment](./shipment.md#voidShipment); the ETGB (Electronic Commerce Customs Declaration) document of a delivered shipment is obtained with [getEtgbDownloadUrl](./shipment.md#getEtgbDownloadUrl).

**Points to Note**

- Each [createRates](./rates.md#createRates) call opens a new shipment reference; references for which no rate is selected remain in the `rate-not-selected` status. For the same shipment, proceed with the existing `reference`, `searchId` and `rateId`.
- Rates have no expiry period; the rate is re-validated during [createShipment](./shipment.md#createShipment), and if it is no longer valid, `rateselectionfailed.error` is returned and a new [createRates](./rates.md#createRates) call is required.
- Additional services with `isRequired: true` must be sent in `selectedServices` in the [createShipment](./shipment.md#createShipment) request.
- Voiding does not revoke a label that has already been created at the carrier.

---

## 4. Format and Common Rules

### URI scheme

_Host_ : api.navlungo.com
_Host(Test)_ : api-qa.navlungo.com
_Schemes_ : HTTPS
_Prefix_ : `api/shipping/v1`

- Request and response bodies are JSON and field names are camelCase. The `Content-Type: application/json` header must be sent.
- Dates are in UTC and in the `dd.MM.yyyy HH:mm` format. Example: `"17.09.2026 10:15"`.
- Fields without a value are returned as `null`. Only permission-dependent fields (`lastMileTrackingNumber`, `lastMileCarrier`, `label`) are omitted from the response entirely when the permission is missing.
- In responses, weight is always in kilograms and dimensions are in centimetres. In the request, `lb`/`in` can be sent via `weightUnit`/`dimensionUnit`; the values are converted to kg/cm and stored.

### Rate Limit

Limits are applied per client, with a 10-second sliding window. When exceeded, `429` and the following body are returned:

```
{ "code": "too.many.requests", "message": "Too many requests. Please try again later.", "data": [] }
```

| Endpoint                                                    | Limit (10 s) |
| ----------------------------------------------------------- | ------------ |
| POST api/shipping/v1/rates                                  | 10           |
| POST api/shipping/v1/shipments                              | 5            |
| GET api/shipping/v1/shipments?reference=                    | 20           |
| GET api/shipping/v1/shipments/{reference}/tracking          | 20           |
| POST api/shipping/v1/shipments/{reference}/labels           | 3            |
| POST api/shipping/v1/shipments/{reference}/documents        | 5            |
| GET api/shipping/v1/shipments/{reference}/etgb/download-url | 5            |
| POST api/shipping/v1/shipments/{reference}/void             | 5            |

### Error Responses

There are two different error bodies.

**Business rule errors** (`400`, `404`) are returned with the [Error](#error) object:

```
{ "code": "ratealreadyselected.error", "message": "A rate has already been selected for this reference.", "data": [] }
```

When a record is not found, every endpoint returns `404` and `record.not.found`; a shipment that belongs to another account gets the same response.

**Field validation and authorization errors** (`400`, `401`) are returned with the [ProblemDetails](#problemDetails) object. In a field validation error, the `extensions.validationErrors` array contains the invalid fields:

```
{
  "type": "Common/Validation", "status": 400, "title": "Verification Error",
  "detail": "An error occurred during request verification", "path": "/api/shipping/v1/rates",
  "extensions": { "validationErrors": [ { "propertyName": "Packages[0].Weight", "errorMessage": "'Weight' must be greater than '0'." } ] }
}
```

| Case                                                | Response                                                    |
| --------------------------------------------------- | ----------------------------------------------------------- |
| Token not sent                                      | `401` `Common/Unauthenticated`                              |
| Token invalid or expired                            | `401` `Authentication/InvalidToken`                         |
| Required scope missing from the token               | `401` `Authentication/InvalidScope`                         |
| Required permission (claim) missing from the client | `401` `Authentication/InvalidClaim`                         |
| Client suspended or not defined for this API        | `401` (`Common/Forbidden` or `Authentication/InvalidToken`) |
| Unexpected error                                    | `500` `Common/Unexpected`                                   |

Body validation runs before the authorization check: a request with missing or invalid fields returns a `400` field validation error even if there is no token.

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

---

<a name="valueLists"></a>

## 5. Value Lists

| Field                                           | Values                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `shipmentType`                                  | `sales`, `sample`, `gift`, `micro-export`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `currency` (shipment and product)               | `EUR`, `USD`, `TRY`, `GBP`, `CAD`, `SAR`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `weightUnit` / `dimensionUnit`                  | `kg`/`cm` (default) or `lb`/`in`; both are given together                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `packages[].type`                               | `box`, `envelope`, `document`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `countryCode`, `originCountry`                  | ISO 3166-1 alpha-2, upper case: `TR`, `US`, `DE`                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `rates[].serviceType`                           | `express`, `eco-express`, `expedited`, `ups-standart`, `registered-small-package`, `registered-package-air`, `us-eco-cargo`, `US-EXPS`, `INT-ECO`, `USPM`, `US-GRD`, `DM-FGR`, `DM-USGR`, `DM-FGE`, `widect-usps`, `widect-amazon-shipping`, `widect-ups-ground`, `widect-b2b`, `economy-road`, `navlungo-bundle` (freight services are returned as `express`)                                                                                                                                                                 |
| `rates[].services[].code`, `selectedServices[]` | `ddp`, `insurance`, `customs-tax-duty`, `atr`; `premium-tracking` may also appear in the response but cannot be selected                                                                                                                                                                                                                                                                                                                                                                                                       |
| `status` (shipment)                             | `rate-not-selected`, `awaiting-warehouse-arrival`, `in-warehouse`, `dispatched`, `cancelled`                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `checkpoints[].status` (tracking)               | `Pending`, `Info Received`, `In Transit`, `Out For Delivery`, `Attempt Fail`, `Available For Pickup`, `Delivered`, `Exception`, `Expired`. Written without spaces in carrier events (`InTransit`); see [Tracking Statuses](./cargoTracking.md#trackingStatuses) for details                                                                                                                                                                                                                                                    |
| `carrier` (shipment detail)                     | Carrier name: `United Parcel Service`, `Federal Express`, `DHL International`, `TNT Express`, `PTT`, `Aramex`, `Asendia`, `THY`, `You Parcel`, `Pts`, `ShipStation For Usps`, `Navlungo`                                                                                                                                                                                                                                                                                                                                       |
| `lastMileCarrier`                               | Carrier code: `ups`, `fedex`, `dhl`, `tnt`, `gls`, `usps`, `aramex`, `dpd`, `pts`, etc.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `label.contentType`                             | `application/pdf`, `image/png`, `text/html`, `application/zip`                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `documents[].type`                              | `msds`, `e-archive`, `tsca`, `fda`, `eori-no`, `ddp`, `insurance-policy`, `battery-form-label`, `no-amazon-fba-tag`, `original-invoice`, `ozon-label`, `metal-composite-document`, `navlungo-label`, `last-mile-label`, `others`, `aphis-core-admissibility-guidance-form`, `diamond-certificate`, `epa-noa`, `epa-vne-form`, `interim-footwear-invoice`, `fsvp-importer-consent-form`, `lacey-act-ppq-505`, `section-232`, `watch-breakout`, `usda-ams-organics-form`, `prior-notices`, `packing-list`, `return-label`, `atr` |

---

## 6. Operations

[Token API](./token.md)</br>
[Rates API](./rates.md)</br>
[Shipment API](./shipment.md)</br>
[Shipment Tracking API](./cargoTracking.md)</br>

### Postman Collection

Import the [Navlungo-Shipping-API.postman_collection.json](./Navlungo-Shipping-API.postman_collection.json) file into Postman and fill in the `clientId` and `clientSecret` variables. The collection uses the QA addresses by default; it obtains the token automatically and carries the `reference`, `searchId` and `rateId` values from the `1. Get rates` response into the subsequent requests. For production, change the `baseUrl` and `identityUrl` variables.
