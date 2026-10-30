# Shipment API

<a name="overview"></a>

## Overview

With this API, on the shipment reference opened as a result of [createRates](./rates.md#createRates), a rate is selected, the shipment detail is retrieved, the carrier label and documents are managed, the ETGB (Electronic Commerce Customs Declaration) document is downloaded and the shipment is voided.

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

This API can be called with tokens issued through the OAuth2 **client_credentials** flow. The scope of each operation is stated under its own heading; label creation additionally requires the **can_create_labels** permission. See [Authorization](./README.md#authorization).

### Operations

[createShipment](#createShipment)<br>
[getShipment](#getShipment)<br>
[createLabel](#createLabel)<br>
[uploadDocuments](#uploadDocuments)<br>
[getEtgbDownloadUrl](#getEtgbDownloadUrl)<br>
[voidShipment](#voidShipment)<br>

<a name="paths"></a>

## Paths

<a name="createShipment"></a>

### POST api/shipping/v1/shipments

**Operation: createShipment**

_Scope_ : shipping_shipments_write

#### Description

Selects the rate for the shipment opened with [createRates](./rates.md#createRates). The shipment moves to the "awaiting warehouse arrival" (`awaiting-warehouse-arrival`) status and a Navlungo tracking number is assigned. No payment is collected at this step. The rate is re-validated while being selected; if it is invalid, a new [createRates](./rates.md#createRates) call is required.

#### Rate Limit

- At most 5 requests per 10 seconds per client

#### Parameters

| Type     | Name                    | Description                          | Schema                                          |
| -------- | ----------------------- | ------------------------------------ | ----------------------------------------------- |
| **Body** | **body** <br>_required_ | The schema required to select a rate | [CreateShipmentRequest](#createShipmentRequest) |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Schema                                              |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **200**   | Successful                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | [ShipmentCreatedResponse](#shipmentCreatedResponse) |
| **400**   | A request validation error occurred ([ProblemDetails](#problemDetails)) or a business rule error ([Error](#error)):<br> `shipment.shipmentcannotbeupdatedafterithasbeencanceled.error` - Shipment has been cancelled<br> `ratealreadyselected.error`, `shipment.shipmentisnotintheinitstate.error` - A rate has already been selected (the latter on a concurrent request)<br> `shipment.ddpisrequiredbutnotselected.error`, `shipment.customstaxdutyisrequiredbutnotselected.error` - A required additional service is missing from `selectedServices`<br> `rateselectionfailed.error` - `searchId`/`rateId` do not match or the rate is no longer valid | [Error](#error)                                     |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_shipments_write` scope                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [ProblemDetails](#problemDetails)                   |
| **404**   | `record.not.found` - Shipment not found or belongs to another account                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | [Error](#error)                                     |
| **429**   | Rate limit exceeded                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [Error](#error)                                     |
| **500**   | An unexpected error occurred during the request                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | [ProblemDetails](#problemDetails)                   |

#### Example Request Body

```
{ "reference": 458213, "searchId": "3f0c9a4e-7b2d-4c7e-9a1b-2e5d6f7a8b9c", "rateId": "9b7e2c11-4d3a-4f6e-8c2a-1d2e3f4a5b6c", "selectedServices": ["customs-tax-duty", "insurance"] }
```

#### Example Response

```
{
  "shipmentId": "b7e6c1a2-9f3d-4e2b-8a1c-7d5f6e8a9b0c",
  "reference": 458213,
  "trackingNumber": "NVL000000458213",
  "trackingUrl": "https://navlungo.com/track?trackingNumber=NVL000000458213",
  "packageLabels": ["458213-1", "458213-2"],
  "totalPrice": 62.1, "currency": "USD", "chargeableWeight": 4.0
}
```

#### Request Model

<a name="createShipmentRequest"></a>

##### CreateShipmentRequest

| Name                                | Description                                                                                                                                                             | Schema           |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| **reference** <br>_required_        | Shipment reference ([createRates](./rates.md#createRates) response)                                                                                                     | long             |
| **searchId** <br>_required_         | The id of the rate search ([createRates](./rates.md#createRates) response)                                                                                              | string (Guid)    |
| **rateId** <br>_required_           | The id of the selected rate                                                                                                                                             | string (Guid)    |
| **selectedServices** <br>_optional_ | Selected additional service codes: `ddp`, `insurance`, `customs-tax-duty`, `atr`; duplicates are not allowed. Services with `isRequired: true` in the rate are required | < string > array |

#### Response Model

<a name="shipmentCreatedResponse"></a>

##### ShipmentCreatedResponse

| Name                                | Description                                                               | Schema           |
| ----------------------------------- | ------------------------------------------------------------------------- | ---------------- |
| **shipmentId** <br>_required_       | Unique id of the shipment                                                 | string (Guid)    |
| **reference** <br>_required_        | Shipment reference                                                        | long             |
| **trackingNumber** <br>_required_   | Navlungo tracking number (`NVL` + the reference zero-padded to 12 digits) | string           |
| **trackingUrl** <br>_required_      | Navlungo tracking page address                                            | string           |
| **packageLabels** <br>_required_    | Package label numbers                                                     | < string > array |
| **totalPrice** <br>_required_       | Total amount of the selected rate and additional services                 | decimal          |
| **currency** <br>_required_         | Currency                                                                  | string           |
| **chargeableWeight** <br>_required_ | Chargeable weight (kg)                                                    | decimal          |

---

<a name="getShipment"></a>

### GET api/shipping/v1/shipments?reference={reference}

**Operation: getShipment**

_Scope_ : shipping_shipments_read

#### Description

Returns the shipment detail. If no rate has been selected, the `trackingNumber`, `carrier`, `serviceType`, `price` and `packageLabels` fields are `null`.

#### Rate Limit

- At most 20 requests per 10 seconds per client

#### Parameters

| Type      | Name                         | Description               | Schema |
| --------- | ---------------------------- | ------------------------- | ------ |
| **Query** | **reference** <br>_required_ | Shipment reference number | long   |

#### Responses

| HTTP Code | Description                                                                                                        | Schema                                            |
| --------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| **200**   | Successful                                                                                                         | [ShipmentDetailResponse](#shipmentDetailResponse) |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_shipments_read` scope | [ProblemDetails](#problemDetails)                 |
| **404**   | `record.not.found` - Shipment not found or belongs to another account                                              | [Error](#error)                                   |
| **429**   | Rate limit exceeded                                                                                                | [Error](#error)                                   |
| **500**   | An unexpected error occurred during the request                                                                    | [ProblemDetails](#problemDetails)                 |

#### Example Response

```
{
  "shipmentId": "b7e6c1a2-9f3d-4e2b-8a1c-7d5f6e8a9b0c",
  "reference": 458213,
  "status": "awaiting-warehouse-arrival",
  "trackingNumber": "NVL000000458213",
  "carrier": "United Parcel Service",
  "serviceType": "express",
  "price": { "amount": 62.1, "currency": "USD" },
  "chargeableWeight": 4.0,
  "warehouseArrivalDate": null,
  "dispatchDate": null,
  "iossNumber": null,
  "senderAddress": { "contactName": "Ayşe Yılmaz", "companyName": "Yılmaz Tekstil A.Ş.", "countryCode": "TR", "stateCode": null, "postalCode": "34710", "city": "İstanbul", "town": "Kadıköy", "firstLine": "Caferağa Mah. Moda Cad. No:12", "secondLine": null, "thirdLine": null, "email": "ayse@yilmaztekstil.com", "phoneCode": "+90", "phoneNumber": "5321234567" },
  "receiverAddress": { "contactName": "John Carter", "companyName": null, "countryCode": "US", "stateCode": "NY", "postalCode": "10001", "city": "New York", "town": null, "firstLine": "350 5th Ave Suite 4400", "secondLine": null, "thirdLine": null, "email": "john.carter@example.com", "phoneCode": "+1", "phoneNumber": "2125550147" },
  "products": [ { "description": "Cotton t-shirt, short sleeve", "hsCode": "6109.10.00.12", "originCountryCode": "TR", "unitPrice": 12.5, "currency": "USD", "quantity": 10 } ],
  "packages": [ { "type": "box", "weight": 2.0, "length": 40.64, "height": 20.32, "width": 30.48 }, { "type": "box", "weight": 2.0, "length": 40.64, "height": 20.32, "width": 30.48 } ],
  "packageLabels": ["458213-1", "458213-2"]
}
```

#### Response Model

<a name="shipmentDetailResponse"></a>

##### ShipmentDetailResponse

| Name                                    | Description                                                                                                                                   | Schema                                    |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **shipmentId** <br>_required_           | Unique id of the shipment                                                                                                                     | string (Guid)                             |
| **reference** <br>_required_            | Shipment reference                                                                                                                            | long                                      |
| **status** <br>_required_               | Shipment status: `rate-not-selected`, `awaiting-warehouse-arrival`, `in-warehouse`, `dispatched`, `cancelled`                                 | string                                    |
| **trackingNumber** <br>_optional_       | Navlungo tracking number. `null` if no rate has been selected                                                                                 | string                                    |
| **carrier** <br>_optional_              | Carrier name. `null` if no rate has been selected                                                                                             | string                                    |
| **serviceType** <br>_optional_          | Service type. `null` if no rate has been selected                                                                                             | string                                    |
| **price** <br>_optional_                | Total amount. `null` if no rate has been selected                                                                                             | [Price](#price)                           |
| **chargeableWeight** <br>_required_     | Chargeable weight (kg)                                                                                                                        | decimal                                   |
| **warehouseArrivalDate** <br>_optional_ | Warehouse arrival date                                                                                                                        | string (DateTime)                         |
| **dispatchDate** <br>_optional_         | Dispatch date                                                                                                                                 | string (DateTime)                         |
| **iossNumber** <br>_optional_           | The IOSS number given in the [createRates](./rates.md#createRates) request and stored; `null` for non-EU destinations or values above 150 EUR | string                                    |
| **senderAddress** <br>_required_        | Sender's origin address                                                                                                                       | [Address](#address)                       |
| **receiverAddress** <br>_required_      | Receiver address                                                                                                                              | [Address](#address)                       |
| **products** <br>_required_             | Products                                                                                                                                      | < [ProductDetail](#productDetail) > array |
| **packages** <br>_required_             | Packages. Each row represents a single package; measurements are always in kg/cm                                                              | < [PackageDetail](#packageDetail) > array |
| **packageLabels** <br>_optional_        | Package label numbers (the same as in the [createShipment](#createShipment) response). `null` if no rate has been selected                    | < string > array                          |

<a name="price"></a>

##### Price

| Name                        | Description | Schema  |
| --------------------------- | ----------- | ------- |
| **amount** <br>_required_   | Amount      | decimal |
| **currency** <br>_required_ | Currency    | string  |

<a name="address"></a>

##### Address

| Name                           | Description         | Schema |
| ------------------------------ | ------------------- | ------ |
| **contactName** <br>_required_ | Contact person      | string |
| **companyName** <br>_optional_ | Company name        | string |
| **countryCode** <br>_required_ | Country code (ISO2) | string |
| **stateCode** <br>_optional_   | State code          | string |
| **postalCode** <br>_optional_  | Postal code         | string |
| **city** <br>_required_        | City                | string |
| **town** <br>_optional_        | District            | string |
| **firstLine** <br>_required_   | Address line        | string |
| **secondLine** <br>_optional_  | Second address line | string |
| **thirdLine** <br>_optional_   | Third address line  | string |
| **email** <br>_optional_       | E-mail              | string |
| **phoneCode** <br>_optional_   | Phone country code  | string |
| **phoneNumber** <br>_optional_ | Phone number        | string |

<a name="productDetail"></a>

##### ProductDetail

| Name                                 | Description            | Schema  |
| ------------------------------------ | ---------------------- | ------- |
| **description** <br>_optional_       | Product description    | string  |
| **hsCode** <br>_required_            | Product HS code        | string  |
| **originCountryCode** <br>_required_ | Country of origin code | string  |
| **unitPrice** <br>_required_         | Unit price             | decimal |
| **currency** <br>_required_          | Currency               | string  |
| **quantity** <br>_required_          | Quantity               | int     |

<a name="packageDetail"></a>

##### PackageDetail

| Name                      | Description  | Schema  |
| ------------------------- | ------------ | ------- |
| **type** <br>_required_   | Package type | string  |
| **weight** <br>_required_ | Weight (kg)  | decimal |
| **length** <br>_optional_ | Length (cm)  | decimal |
| **height** <br>_optional_ | Height (cm)  | decimal |
| **width** <br>_optional_  | Width (cm)   | decimal |

---

<a name="createLabel"></a>

### POST api/shipping/v1/shipments/{reference}/labels

**Operation: createLabel**

_Scope_ : `shipping_labels_write` — _Permission_ : `can_create_labels`

#### Description

Creates the tracking number and label at the carrier before the shipment arrives at the warehouse. There is no request body. It is idempotent: if the label has already been created, the existing values are returned with `created: false`.

- `lastMileTrackingNumber` and `lastMileCarrier` are only included in the response with the `can_access_last_mile_tracking` permission.
- `label` is only included in the response with the `can_download_labels` permission and if the carrier has produced the label file. If the file has not been produced yet, the response is still `200` but the `label` field is absent; the call should be repeated later. No label file is returned for PTT.

#### Rate Limit

- At most 3 requests per 10 seconds per client

#### Parameters

| Type     | Name                         | Description               | Schema |
| -------- | ---------------------------- | ------------------------- | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number | long   |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Schema                            |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **200**   | Successful                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | [LabelResponse](#labelResponse)   |
| **400**   | `shipment.shipmentcannotbeupdatedafterithasbeencanceled.error` - Shipment has been cancelled<br> `ratenotselected.error` - No rate selected<br> `shipmentservicetypeisnotallowedforearlytracking.error` - Service is `navlungo-bundle`<br> `earlytrackingnotenabled.earlytrackingnotenabled.error`, `earlytrackingserviceerrors.invalidintegrationorshipment.error` - The carrier is not eligible for early label creation<br> `lastmilefailure.error` - The carrier rejected the request<br> `record.already.exists` - Concurrent request, the number could not be read yet; retry | [Error](#error)                   |
| **401**   | Authorization error. The access token is invalid, has expired, does not have the `shipping_labels_write` scope or the client does not have the `can_create_labels` permission (`Authentication/InvalidClaim`)                                                                                                                                                                                                                                                                                                                                                                       | [ProblemDetails](#problemDetails) |
| **404**   | `record.not.found` - Shipment not found or belongs to another account                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [Error](#error)                   |
| **429**   | Rate limit exceeded                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | [Error](#error)                   |
| **500**   | An unexpected error occurred during the request                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | [ProblemDetails](#problemDetails) |

#### Example Response

```
{
  "trackingNumber": "NVL000000458213",
  "lastMileTrackingNumber": "1Z999AA10123456784",
  "lastMileCarrier": "ups",
  "created": true,
  "label": { "contentType": "application/pdf", "base64": "JVBERi0xLjQK..." }
}
```

#### Response Model

<a name="labelResponse"></a>

##### LabelResponse

| Name                       | Description                                                                                  | Always present | Schema                  |
| -------------------------- | -------------------------------------------------------------------------------------------- | -------------- | ----------------------- |
| **trackingNumber**         | Navlungo tracking number                                                                     | Yes            | string                  |
| **lastMileTrackingNumber** | Carrier tracking number. Only with the `can_access_last_mile_tracking` permission            | No             | string                  |
| **lastMileCarrier**        | Carrier code (e.g. `ups`). Only with the `can_access_last_mile_tracking` permission          | No             | string                  |
| **created**                | `true` if the label was created in this call, `false` if it already existed                  | Yes            | boolean                 |
| **label**                  | Label file. Only with the `can_download_labels` permission and if the file has been produced | No             | [LabelFile](#labelFile) |

<a name="labelFile"></a>

##### LabelFile

| Name                           | Description                                                               | Schema |
| ------------------------------ | ------------------------------------------------------------------------- | ------ |
| **contentType** <br>_required_ | File type: `application/pdf`, `image/png`, `text/html`, `application/zip` | string |
| **base64** <br>_required_      | Base64-encoded file content                                               | string |

---

<a name="uploadDocuments"></a>

### POST api/shipping/v1/shipments/{reference}/documents

**Operation: uploadDocuments**

_Scope_ : shipping_shipments_write

#### Description

Opens a document record for the shipment and returns, for each document, the address the file is to be uploaded to. The file is uploaded to this address with a separate `PUT` request, with the `Content-Type` header set to the `contentType` value in the response; the upload address is short-lived, upload within **5 minutes** of receiving the address. The document record is created before the file is uploaded. If any of the documents is rejected, none of them is saved.

#### Rate Limit

- At most 5 requests per 10 seconds per client

#### Parameters

| Type     | Name                         | Description                                   | Schema                                            |
| -------- | ---------------------------- | --------------------------------------------- | ------------------------------------------------- |
| **Path** | **reference** <br>_required_ | Shipment reference number                     | long                                              |
| **Body** | **body** <br>_required_      | The schema required to open a document record | [UploadDocumentsRequest](#uploadDocumentsRequest) |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                     | Schema                                              |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **200**   | Successful                                                                                                                                                                                                                                                                                                                      | [UploadDocumentsResponse](#uploadDocumentsResponse) |
| **400**   | A request validation error occurred ([ProblemDetails](#problemDetails)) or a business rule error ([Error](#error)):<br> `shipment.shipmentcannotbeupdatedafterithasbeencanceled.error` - Shipment has been cancelled<br> `document.invalidtypeforearchiveinfo.error` - `eArchiveInfo` was sent with a type other than e-Archive | [Error](#error)                                     |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_shipments_write` scope                                                                                                                                                                                                             | [ProblemDetails](#problemDetails)                   |
| **404**   | `record.not.found` - Shipment not found or belongs to another account                                                                                                                                                                                                                                                           | [Error](#error)                                     |
| **429**   | Rate limit exceeded                                                                                                                                                                                                                                                                                                             | [Error](#error)                                     |
| **500**   | An unexpected error occurred during the request                                                                                                                                                                                                                                                                                 | [ProblemDetails](#problemDetails)                   |

#### Example Request Body

```
{
  "documents": [
    { "fileName": "invoice-458213.pdf", "type": "original-invoice" },
    { "fileName": "earsiv-458213.pdf", "type": "e-archive", "eArchiveInfo": { "date": "2026-09-15T00:00:00", "number": "ABC2026000000001" } }
  ]
}
```

#### Example Response

```
{
  "documents": [
    { "fileName": "invoice-458213.pdf", "type": "original-invoice", "contentType": "application/pdf",
      "uploadUrl": "https://navlungo-prod-shipment-docs.s3.eu-west-1.amazonaws.com/458213/original-invoice/invoice-458213.pdf?X-Amz-Expires=300&X-Amz-Signature=..." }
  ],
  "alreadyExistingFileNames": ["earsiv-458213.pdf"]
}
```

#### Request Model

<a name="uploadDocumentsRequest"></a>

##### UploadDocumentsRequest

| Name                         | Description                                                        | Schema                                          |
| ---------------------------- | ------------------------------------------------------------------ | ----------------------------------------------- |
| **documents** <br>_required_ | Documents. At least 1; `fileName` cannot repeat within the request | < [ShipmentDocument](#shipmentDocument) > array |

<a name="shipmentDocument"></a>

##### ShipmentDocument

| Name                            | Description                                                                        | Schema                        |
| ------------------------------- | ---------------------------------------------------------------------------------- | ----------------------------- |
| **fileName** <br>_required_     | File name, at most 255 characters, unique within the shipment                      | string                        |
| **type** <br>_required_         | Document type. See [Value Lists](./README.md#valueLists) `documents[].type`        | string                        |
| **eArchiveInfo** <br>_optional_ | e-Archive information. Required if `type` is `e-archive`; not sent for other types | [EArchiveInfo](#eArchiveInfo) |

<a name="eArchiveInfo"></a>

##### EArchiveInfo

| Name                      | Description                             | Schema            |
| ------------------------- | --------------------------------------- | ----------------- |
| **date** <br>_required_   | e-Archive invoice date                  | string (DateTime) |
| **number** <br>_required_ | e-Archive invoice number, 16 characters | string            |

#### Response Model

<a name="uploadDocumentsResponse"></a>

##### UploadDocumentsResponse

| Name                                        | Description                                                                                                    | Schema                                              |
| ------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **documents** <br>_required_                | Documents saved in this call and their upload addresses                                                        | < [DocumentUploadInfo](#documentUploadInfo) > array |
| **alreadyExistingFileNames** <br>_required_ | File names skipped because they are already registered with the same name; no new address is produced for them | < string > array                                    |

<a name="documentUploadInfo"></a>

##### DocumentUploadInfo

| Name                           | Description                                          | Schema |
| ------------------------------ | ---------------------------------------------------- | ------ |
| **fileName** <br>_required_    | File name                                            | string |
| **type** <br>_required_        | Document type                                        | string |
| **contentType** <br>_required_ | The `Content-Type` to be used in the upload          | string |
| **uploadUrl** <br>_required_   | The address the file is to be uploaded to with `PUT` | string |

---

<a name="getEtgbDownloadUrl"></a>

### GET api/shipping/v1/shipments/{reference}/etgb/download-url

**Operation: getEtgbDownloadUrl**

_Scope_ : shipping_shipments_read

#### Description

Returns the ETGB number of a delivered shipment and, if available, the PDF download address. The download address is short-lived; download within **2 minutes**. If the record comes from the Excel list, `fileName` and `downloadUrl` are `null` and `etgbNumber` is returned.

#### Rate Limit

- At most 5 requests per 10 seconds per client

#### Parameters

| Type     | Name                         | Description               | Schema |
| -------- | ---------------------------- | ------------------------- | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number | long   |

#### Responses

| HTTP Code | Description                                                                                                                                    | Schema                                              |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **200**   | Successful                                                                                                                                     | [EtgbDownloadUrlResponse](#etgbDownloadUrlResponse) |
| **400**   | `shipmentnotdelivered.error` - Shipment has not been delivered yet<br> `etgbdocumentnotfound.error` - ETGB record not created yet, retry later | [Error](#error)                                     |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_shipments_read` scope                             | [ProblemDetails](#problemDetails)                   |
| **404**   | `record.not.found` - Shipment not found, belongs to another account or has been cancelled                                                      | [Error](#error)                                     |
| **429**   | Rate limit exceeded                                                                                                                            | [Error](#error)                                     |
| **500**   | An unexpected error occurred during the request                                                                                                | [ProblemDetails](#problemDetails)                   |

#### Example Response

```
{
  "reference": 458213,
  "etgbNumber": "26341300EX000123",
  "fileName": "26341300EX000123.pdf",
  "downloadUrl": "https://navlungo-prod-etgb-docs.s3.eu-west-1.amazonaws.com/pdf_files/2026/9/21/26341300EX000123.pdf?X-Amz-Expires=120&X-Amz-Signature=..."
}
```

#### Response Model

<a name="etgbDownloadUrlResponse"></a>

##### EtgbDownloadUrlResponse

| Name                           | Description                                                          | Schema |
| ------------------------------ | -------------------------------------------------------------------- | ------ |
| **reference** <br>_required_   | Shipment reference                                                   | long   |
| **etgbNumber** <br>_required_  | ETGB number                                                          | string |
| **fileName** <br>_optional_    | PDF file name. `null` if the record comes from the Excel list        | string |
| **downloadUrl** <br>_optional_ | PDF download address. `null` if the record comes from the Excel list | string |

---

<a name="voidShipment"></a>

### POST api/shipping/v1/shipments/{reference}/void

**Operation: voidShipment**

_Scope_ : shipping_shipments_write

#### Description

Voids a shipment that has not arrived at the warehouse; a shipment with no rate selected can also be voided. There is no request body. Voiding does not revoke a label that has already been created at the carrier. After voiding, the shipment detail shows `status: "cancelled"`; [createShipment](#createShipment) and [createLabel](#createLabel) reject this reference.

#### Rate Limit

- At most 5 requests per 10 seconds per client

#### Parameters

| Type     | Name                         | Description               | Schema |
| -------- | ---------------------------- | ------------------------- | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number | long   |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                                                                                                                            | Schema                                        |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **200**   | Successful                                                                                                                                                                                                                                                                                                                                                                                                                             | [VoidShipmentResponse](#voidShipmentResponse) |
| **400**   | `shipmentalreadyinwarehouse.error` - The shipment cannot be voided because it has arrived at the warehouse<br> `shipment.shipmentalreadycancelled.error` - Shipment has already been cancelled<br> `shipment.shipmentcannotbecancelledwhenbundlereferenceisexists.error` - A shipment linked to a bundle cannot be voided<br> `notpublicapishipment.error` - The shipment cannot be voided because it was not created through this API | [Error](#error)                               |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_shipments_write` scope                                                                                                                                                                                                                                                                                                                    | [ProblemDetails](#problemDetails)             |
| **404**   | `record.not.found` - Shipment not found or belongs to another account                                                                                                                                                                                                                                                                                                                                                                  | [Error](#error)                               |
| **429**   | Rate limit exceeded                                                                                                                                                                                                                                                                                                                                                                                                                    | [Error](#error)                               |
| **500**   | An unexpected error occurred during the request                                                                                                                                                                                                                                                                                                                                                                                        | [ProblemDetails](#problemDetails)             |

#### Example Response

```
{ "reference": 458213, "status": "cancelled", "cancelledAt": "17.09.2026 10:15" }
```

#### Response Model

<a name="voidShipmentResponse"></a>

##### VoidShipmentResponse

| Name                           | Description        | Schema            |
| ------------------------------ | ------------------ | ----------------- |
| **reference** <br>_required_   | Shipment reference | long              |
| **status** <br>_required_      | Always `cancelled` | string            |
| **cancelledAt** <br>_required_ | Cancellation time  | string (DateTime) |

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
