# Express Shipment API

<a name="overview"></a>

## Overview

Navlungo provides an API that lets you create and manage labels for your shipments. With this API, Navlungo solution partners can manage their shipments and create their labels.

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

### Operations

[createLabel](#createLabel)<br>
[getLabel](#getLabel)<br>
[getTracking](#getTracking)<br>
[getShipment](#getShipment)<br>
[uploadDocument](#uploadDocument)<br>
[getEtgbDownloadUrl](#getEtgbDownloadUrl)<br>
[voidShipment](#voidShipment)<br>

<a name="paths"></a>

## Paths

<a name="createLabel"></a>

### POST api/shipments/v1/{shipmentId}/label

**Operation: createLabel**

#### Description

Creates a label for the specified shipment reference. The user must be authorized for this operation.

#### Rate Limit

- At most 3 requests per 10 seconds for each IP and User-Agent combination

#### Parameters

| Type     | Name                          | Description                                                                                                                     | Schema |
| -------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **Path** | **shipmentId** <br>_required_ | Unique id of the shipment (the shipmentId value in the response of the /stores/v2/{store_id}/orders/{order_reference}/ship API) | Guid   |

#### Responses

| HTTP Code | Description                                                     | Schema                          |
| --------- | --------------------------------------------------------------- | ------------------------------- |
| **200**   | Successful                                                      | [LabelResponse](#labelResponse) |
| **400**   | A request validation error occurred or the request is invalid   | [Error](#error)                 |
| **401**   | Authorization error. The access token is invalid or has expired | [Error](#error)                 |
| **429**   | Rate limit exceeded                                             | [Error](#error)                 |
| **500**   | An unexpected error occurred during the request                 | [Error](#error)                 |

#### Response Model

##### LabelResponse

| Name                                      | Description                       | Schema |
| ----------------------------------------- | --------------------------------- | ------ |
| **lastMileTrackingNumber** <br>_required_ | Last mile carrier tracking number | string |

---

<a name="getLabel"></a>

### GET api/shipments/v1/{shipmentId}/label

**Operation: getLabel**

#### Description

Retrieves the label created for the specified shipment reference. The user must be authorized for this operation.

#### Rate Limit

- At most 3 requests per 10 seconds for each IP and User-Agent combination

#### Parameters

| Type     | Name                          | Description                                                                                                                     | Schema |
| -------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **Path** | **shipmentId** <br>_required_ | Unique id of the shipment (the shipmentId value in the response of the /stores/v2/{store_id}/orders/{order_reference}/ship API) | Guid   |

#### Responses

| HTTP Code | Description                                                     | Schema                                                |
| --------- | --------------------------------------------------------------- | ----------------------------------------------------- |
| **200**   | Successful                                                      | [LabelDownloadUrlResponse](#labelDownloadUrlResponse) |
| **400**   | A request validation error occurred or the request is invalid   | [Error](#error)                                       |
| **401**   | Authorization error. The access token is invalid or has expired | [Error](#error)                                       |
| **404**   | Label not found                                                 | [Error](#error)                                       |
| **429**   | Rate limit exceeded                                             | [Error](#error)                                       |
| **500**   | An unexpected error occurred during the request                 | [Error](#error)                                       |

#### Response Model

##### LabelDownloadUrlResponse

| Name                        | Description         | Schema |
| --------------------------- | ------------------- | ------ |
| **labelUrl** <br>_required_ | Label download link | string |

---

<a name="getTracking"></a>

### GET api/shipments/v1/{reference}/tracking

**Operation: getTracking**

#### Description

Retrieves the tracking information for the specified shipment reference. The user must be authorized for this operation.

#### Rate Limit

- At most 20 requests per 10 seconds for each IP and User-Agent combination

#### Parameters

| Type     | Name                         | Description                                                                                | Schema |
| -------- | ---------------------------- | ------------------------------------------------------------------------------------------ | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number (the quoteReference value in the stores/v1/{id}/orders/ship API) | long   |

#### Responses

| HTTP Code | Description                                                     | Schema                                                      |
| --------- | --------------------------------------------------------------- | ----------------------------------------------------------- |
| **200**   | Successful                                                      | [TrackingCheckpointsResponse](#trackingCheckpointsResponse) |
| **400**   | A request validation error occurred or the request is invalid   | [Error](#error)                                             |
| **401**   | Authorization error. The access token is invalid or has expired | [Error](#error)                                             |
| **404**   | Shipment or tracking information not found                      | [Error](#error)                                             |
| **429**   | Rate limit exceeded                                             | [Error](#error)                                             |
| **500**   | An unexpected error occurred during the request                 | [Error](#error)                                             |

#### Response Model

##### TrackingCheckpointsResponse

| Name                                    | Description             | Schema                              |
| --------------------------------------- | ----------------------- | ----------------------------------- |
| **trackingNumber** <br>_required_       | Tracking number         | string                              |
| **originCountry** <br>_required_        | Origin country          | string                              |
| **destinationCountry** <br>_required_   | Destination country     | string                              |
| **pickupDate** <br>_required_           | Pickup date             | datetime                            |
| **expectedDeliveryDate** <br>_optional_ | Estimated delivery date | datetime                            |
| **carrier** <br>_required_              | Carrier                 | string                              |
| **originLocation** <br>_required_       | Origin location         | string                              |
| **destinationLocation** <br>_required_  | Destination location    | string                              |
| **checkpoints** <br>_required_          | Tracking checkpoints    | < [Checkpoint](#checkpoint) > array |

##### Checkpoint

| Name                                    | Description            | Schema   |
| --------------------------------------- | ---------------------- | -------- |
| **checkpointTime** <br>_required_       | Checkpoint time        | datetime |
| **status** <br>_required_               | Status                 | string   |
| **subStatusMessage** <br>_required_     | Sub-status message     | string   |
| **subStatusDescription** <br>_optional_ | Sub-status description | string   |
| **country** <br>_optional_              | Country                | string   |
| **city** <br>_optional_                 | City                   | string   |
| **zip** <br>_optional_                  | Postal code            | string   |
| **location** <br>_optional_             | Location               | string   |

---

<a name="getShipment"></a>

### GET api/shipments/v1/{reference}

**Operation: getShipment**

#### Description

Retrieves the current information of the shipment for the specified shipment reference; the current price, sender and receiver addresses, product contents and the current measurements of the packages are returned. The user must be authorized for this operation.

#### Rate Limit

- At most 20 requests per 10 seconds for each IP and User-Agent combination

#### Parameters

| Type     | Name                         | Description                                                                                                                            | Schema |
| -------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **Path** | **reference** <br>_required_ | Shipment reference number (the shipmentReference value in the response of the /stores/v2/{store_id}/orders/{order_reference}/ship API) | long   |

#### Responses

| HTTP Code | Description                                                     | Schema                                            |
| --------- | --------------------------------------------------------------- | ------------------------------------------------- |
| **200**   | Successful                                                      | [ShipmentDetailResponse](#shipmentDetailResponse) |
| **400**   | A request validation error occurred or the request is invalid   | [Error](#error)                                   |
| **401**   | Authorization error. The access token is invalid or has expired | [Error](#error)                                   |
| **404**   | Shipment not found                                              | [Error](#error)                                   |
| **429**   | Rate limit exceeded                                             | [Error](#error)                                   |
| **500**   | An unexpected error occurred during the request                 | [Error](#error)                                   |

#### Response Model

<a name="shipmentDetailResponse"></a>

##### ShipmentDetailResponse

| Name                                     | Description                                                                            | Schema                                      |
| ---------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------- |
| **shipmentId** <br>_required_            | Unique id of the shipment                                                              | Guid                                        |
| **reference** <br>_required_             | Shipment reference number                                                              | long                                        |
| **warehouseArrivalDate** <br>_optional_  | Date the shipment entered the warehouse (empty if it has not arrived at the warehouse) | datetime                                    |
| **warehouseDispatchDate** <br>_optional_ | Date the shipment left the warehouse (empty if it has not left the warehouse)          | datetime                                    |
| **price** <br>_required_                 | Current price of the shipment                                                          | [Price](#shipmentDetailPrice)               |
| **senderAddress** <br>_required_         | Sender address                                                                         | [Address](#shipmentDetailAddress)           |
| **receiverAddress** <br>_required_       | Receiver address                                                                       | [Address](#shipmentDetailAddress)           |
| **products** <br>_required_              | Proforma invoice product contents                                                      | < [Product](#shipmentDetailProduct) > array |
| **packages** <br>_required_              | Current measurements of the packages (updated after the warehouse measurement)         | < [Package](#shipmentDetailPackage) > array |

<a name="shipmentDetailPrice"></a>

##### Price

| Name                        | Description | Schema  |
| --------------------------- | ----------- | ------- |
| **amount** <br>_required_   | Amount      | decimal |
| **currency** <br>_required_ | Currency    | string  |

<a name="shipmentDetailAddress"></a>

##### Address

| Name                           | Description                                    | Schema |
| ------------------------------ | ---------------------------------------------- | ------ |
| **contactName** <br>_required_ | Name of the person to contact                  | string |
| **companyName** <br>_optional_ | Company name                                   | string |
| **countryCode** <br>_required_ | Country code (ISO 3166-1 alpha-2)              | string |
| **stateCode** <br>_optional_   | State code                                     | string |
| **postalCode** <br>_optional_  | Postal code                                    | string |
| **city** <br>_required_        | City                                           | string |
| **town** <br>_optional_        | District (only returned in the sender address) | string |
| **firstLine** <br>_required_   | Address line 1                                 | string |
| **secondLine** <br>_optional_  | Address line 2                                 | string |
| **thirdLine** <br>_optional_   | Address line 3                                 | string |
| **email** <br>_optional_       | E-mail address                                 | string |
| **phoneCode** <br>_optional_   | Phone country code                             | string |
| **phoneNumber** <br>_optional_ | Phone number                                   | string |

<a name="shipmentDetailProduct"></a>

##### Product

| Name                                 | Description            | Schema  |
| ------------------------------------ | ---------------------- | ------- |
| **description** <br>_optional_       | Product description    | string  |
| **hsCode** <br>_required_            | HS (GTİP) code         | string  |
| **originCountryCode** <br>_required_ | Country of origin code | string  |
| **unitPrice** <br>_required_         | Unit price             | decimal |
| **currency** <br>_required_          | Currency               | string  |
| **quantity** <br>_required_          | Quantity               | int     |

<a name="shipmentDetailPackage"></a>

##### Package

| Name                      | Description                                    | Schema  |
| ------------------------- | ---------------------------------------------- | ------- |
| **type** <br>_required_   | Package type (`Box`, `Envelope`, `Document`)   | string  |
| **weight** <br>_required_ | Weight (kg)                                    | decimal |
| **length** <br>_optional_ | Length (cm) (only returned for the `Box` type) | decimal |
| **height** <br>_optional_ | Height (cm) (only returned for the `Box` type) | decimal |
| **width** <br>_optional_  | Width (cm) (only returned for the `Box` type)  | decimal |

---

<a name="getEtgbDownloadUrl"></a>

### GET api/shipments/v1/{shipmentId}/etgb/download-url

**Operation: GetEtgbDownloadUrl**

#### Description

Retrieves the download link of the ETGB (Electronic Commerce Customs Declaration) document for the specified shipment. The user must be authorized for this operation. The ETGB document can only be downloaded after the shipment has been delivered. In some cases where the ETGB document has not been obtained from customs, only the etgbNumber may be returned instead of the download url.

#### Rate Limit

- At most 5 requests per 10 seconds for each IP and User-Agent combination

#### Parameters

| Type     | Name                          | Description               | Schema |
| -------- | ----------------------------- | ------------------------- | ------ |
| **Path** | **shipmentId** <br>_required_ | Unique id of the shipment | Guid   |

#### Responses

| HTTP Code | Description                                                                                                                                                                | Schema                                            |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| **200**   | Successful                                                                                                                                                                 | [EtgbDownloadUrlReponse](#etgbDownloadUrlReponse) |
| **400**   | A request validation error occurred or the request is invalid<br> `shipmentnotfound.error` - Shipment not found<br> `etgbdocumentnotfound.error` - ETGB document not found | [Error](#error)                                   |
| **401**   | Authorization error.                                                                                                                                                       |                                                   |
| **400**   | The ETGB document cannot be downloaded because the shipment has not been delivered<br> `shipmentnotdelivered.error` - Shipment not delivered                               | [Error](#error)                                   |
| **429**   | Rate limit exceeded                                                                                                                                                        | [Error](#error)                                   |

#### Response Model

##### EtgbDownloadUrlReponse

| Name                           | Description   | Schema |
| ------------------------------ | ------------- | ------ |
| **FileName** <br>_nullable_    | File name     | string |
| **DownloadUrl** <br>_nullable_ | Download link | string |
| **EtgbNumber** <br>_required_  | ETGB Number   | string |

---

<a name="uploadDocument"></a>

### POST api/shipments/v1/{shipmentId}/documents

**Operation: UploadDocument**

#### Description

Uploads documents for the specified shipment. The user must be authorized for this operation.

The API returns an AWS S3 presigned URL for uploading the documents. This URL is valid for 10 minutes. The client must upload the file to the specified URL within this period. When the endpoint response is successful, the system assumes that the file has been uploaded. Therefore, this endpoint should only be called when the file will be uploaded immediately afterwards.

#### Rate Limit

- At most 1 request per 2 seconds for each IP and User-Agent combination

#### Parameters

| Type     | Name                          | Description                            | Schema                                                          |
| -------- | ----------------------------- | -------------------------------------- | --------------------------------------------------------------- |
| **Path** | **shipmentId** <br>_required_ | Unique id of the shipment              | Guid                                                            |
| **Body** | **request** <br>_required_    | Information of the documents to upload | [UpdateShipmentDocumentRequest](#updateShipmentDocumentRequest) |

#### Responses

| HTTP Code | Description                                                   | Schema                                                            |
| --------- | ------------------------------------------------------------- | ----------------------------------------------------------------- |
| **200**   | Successful                                                    | [UpdateShipmentDocumentResponse](#UpdateShipmentDocumentResponse) |
| **400**   | A request validation error occurred or the request is invalid | [Error](#error)                                                   |
| **401**   | Authorization error.                                          |
| **404**   | Shipment not found                                            | [Error](#error)                                                   |
| **429**   | Rate limit exceeded                                           | [Error](#error)                                                   |

#### Request Model

##### UpdateShipmentDocumentRequest

| Name                                 | Description   | Schema                                                    |
| ------------------------------------ | ------------- | --------------------------------------------------------- |
| **ShipmentDocuments** <br>_required_ | Document list | < [ShipmentDocumentModel](#shipmentDocumentModel) > array |

##### ShipmentDocumentModel

| Name                            | Description                                                                                                                                                         | Schema                        |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| **FileName** <br>_required_     | File name                                                                                                                                                           | string                        |
| **Type** <br>_required_         | Document type. Valid types: `msds`, `e-archive`, `tsca`, `fda`, `eori-no`, `ddp`, `insurance-policy`, `battery-form-label`, `no-amazon-fba-tag`, `original-invoice` | string                        |
| **EArchiveInfo** <br>_optional_ | e-Archive information (only required if `Type` is `e-archive`, otherwise it is not taken into account.)                                                             | [EArchiveInfo](#earchiveInfo) |

##### EArchiveInfo

| Name                      | Description                              | Schema   |
| ------------------------- | ---------------------------------------- | -------- |
| **Date** <br>_required_   | e-Archive date                           | datetime |
| **Number** <br>_required_ | e-Archive number (must be 16 characters) | string   |

#### Example Request Body

```json
{
  "ShipmentDocuments": [
    {
      "Type": "e-archive",
      "FileName": "x.pdf",
      "EArchiveInfo": {
        "Date": "2023-10-27",
        "Number": "NAV2023000000123"
      }
    }
  ]
}
```

#### Response Model

##### UpdateShipmentDocumentResponse

| Name                                        | Description                       | Schema                                              |
| ------------------------------------------- | --------------------------------- | --------------------------------------------------- |
| **Documents** <br>_required_                | Upload information                | < [DocumentUploadInfo](#documentUploadInfo) > array |
| **AlreadyExistingFileNames** <br>_required_ | Names of files that already exist | < string > array                                    |

##### DocumentUploadInfo

| Name                        | Description     | Schema              |
| --------------------------- | --------------- | ------------------- |
| **FileName** <br>_required_ | File name       | string              |
| **Type** <br>_required_     | Document type   | string              |
| **UrlInfo** <br>_required_  | URL information | [UrlInfo](#urlInfo) |

##### UrlInfo

| Name                           | Description  | Schema |
| ------------------------------ | ------------ | ------ |
| **ContentType** <br>_required_ | Content type | string |
| **UploadUrl** <br>_required_   | Upload URL   | string |

---

<a name="voidShipment"></a>

### POST api/shipments/v1/{shipmentId}/void

**Operation: voidShipment**

#### Description

Voids the specified shipment. Only shipments that have not yet arrived at the warehouse can be voided.

#### Rate Limit

- At most 2 requests per 1 second for each IP and User-Agent combination

#### Parameters

| Type     | Name                          | Description                                                                                                                     | Schema |
| -------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **Path** | **shipmentId** <br>_required_ | Unique id of the shipment (the shipmentId value in the response of the /stores/v2/{store_id}/orders/{order_reference}/ship API) | Guid   |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Schema          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| **200**   | Successful (no response body is returned)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | -               |
| **400**   | A request validation error occurred or the request is invalid<br> `shipmentnotfound.error` - Shipment not found<br> `unauthorized.action` - You are not authorized to perform this operation<br> `notpublicapishipment.error` - The shipment cannot be voided because it was not created through the API<br> `shipmentalreadyinwarehouse.error` - The shipment cannot be voided because it has arrived at the warehouse<br> `shipment.shipmentalreadycancelled.error` - Shipment has already been cancelled<br> `shipment.shipmentcannotbecancelledwhenbundlereferenceisexists.error` - A shipment with a bundle reference cannot be voided | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | [Error](#error) |
| **429**   | Rate limit exceeded                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | [Error](#error) |
| **500**   | An unexpected error occurred during the request                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | [Error](#error) |

---

## Common Models

### Error

General error object

| Name                       | Description                            | Schema |
| -------------------------- | -------------------------------------- | ------ |
| **code** <br>_required_    | Error code                             | string |
| **message** <br>_required_ | Error message                          | string |
| **data** <br>_optional_    | Additional information about the error | object |
