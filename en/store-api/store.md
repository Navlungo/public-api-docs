# Store API

<a name="overview"></a>

## Overview

Navlungo users can connect their stores and view their orders in the Navlungo interfaces. With these APIs, a user registered in the system can connect their store without entering the interface and set the default addresses to be used in store orders.

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

- [createStore](#createStore)<br>
- [getStores](#getStores)<br>
- [getStore](#getStore)<br>
- [updateStore](#updateStore)<br>
- [updateStoreAddress](#updateStoreAddress)<br>
- [updateInvoiceAddress](#updateInvoiceAddress)<br>
- [deleteStore](#deleteStore)<br>
- [addStoreIossNumber](#addStoreIossNumber)<br>
- [deleteStoreIossNumber](#deleteStoreIossNumber)<br>
- [shipStoreOrder](#shipStoreOrder)<br>
- [trackOrder](#trackOrder)<br>

<a name="paths"></a>

## Paths

<a name="createStore"></a>

### POST stores/v1

**Operation: createStore**

#### Description

Creates a store record belonging to the acting user and creates the store's default invoice and shipping address information.

#### Parameters

| Type     | Name                    | Description                           | Schema                                    |
| -------- | ----------------------- | ------------------------------------- | ----------------------------------------- |
| **Body** | **body** <br>_required_ | The schema required to create a store | [CreateStoreRequest](#createStoreRequest) |

#### Responses

| HTTP Code | Description                                                      | Schema                                      |
| --------- | ---------------------------------------------------------------- | ------------------------------------------- |
| **200**   | Successful                                                       | [CreateStoreResponse](#createStoreResponse) |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error)                             |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error)                             |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error)                             |

<a name="definitions"></a>

## Definitions

<a name="createStoreRequest"></a>

### CreateStoreRequest

| Name                              | Description                                                                                                                                               | Schema                         |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| **name** <br>_required_           | Store name                                                                                                                                                | string                         |
| **url** <br>_optional_            | The store url for stores defined via their url.                                                                                                           | string                         |
| **storeId** <br>_optional_        | The unique Id of the store. If not sent, the unique id created in Navlungo will be returned. Communication for the store will take place through this id. | string                         |
| **storeAddress** <br>_required_   | The store's address information.                                                                                                                          | < [Address](#address) > object |
| **invoiceAddress** <br>_required_ | The invoice address the store will use when creating shipments.                                                                                           | < [Address](#address) > object |

<a name="address"></a>

### Address

The model of the store's address information.

| Name                                    | Description                                                                               | Schema |
| --------------------------------------- | ----------------------------------------------------------------------------------------- | ------ |
| **type** <br>_required_                 | Whether the address is individual or corporate. Allowed values: `Individual`, `Corporate` | string |
| **companyName** <br>_optional_          | Company name for a corporate address                                                      | string |
| **identificationNumber** <br>_required_ | Tax identification number or Turkish ID number                                            | string |
| **taxOffice** <br>_optional_            | Tax office (required for corporate addresses)                                             | string |
| **contactName** <br>_required_          | Contact person name                                                                       | string |
| **contactPhone** <br>_required_         | Contact person's phone number                                                             | string |
| **contactMail** <br>_required_          | Contact person's email address                                                            | string |
| **countryCode** <br>_required_          | 2-letter global country code                                                              | string |
| **state** <br>_optional_                | Global state code                                                                         | string |
| **city** <br>_required_                 | [City information](./city-town.md)                                                        | string |
| **town** <br>_required_                 | [District information](./city-town.md)                                                    | string |
| **postalCode** <br>_required_           | Postal code                                                                               | string |
| **firstLine** <br>_required_            | First address line. Minimum 10 characters, maximum 30 characters.                         | string |
| **secondLine** <br>_optional_           | Second address line. Maximum 30 characters.                                               | string |
| **thirdLine** <br>_optional_            | Third address line. Maximum 30 characters.                                                | string |

<a name="createStoreResponse"></a>

### createStoreResponse

| Name                       | Description                         | Schema |
| -------------------------- | ----------------------------------- | ------ |
| **storeId** <br>_required_ | The unique Id of the created store. | string |

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

<hr/>
<a name="getStores"></a>

### GET stores/v1

**Operation: getStores**

#### Description

Retrieves all stores belonging to the user. If there are no stores, an empty array is returned.

#### Parameters

#### Responses

| HTTP Code | Description                                                      | Schema                                  |
| --------- | ---------------------------------------------------------------- | --------------------------------------- |
| **200**   | Successful                                                       | [GetStoresResponse](#GetStoresResponse) |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error)                         |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error)                         |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error)                         |

<a name="definitions"></a>

## Definitions

<a name="GetStoresResponse"></a>

### GetStoresResponse

| Name                      | Description | Schema                    |
| ------------------------- | ----------- | ------------------------- |
| **stores** <br>_required_ | Store list  | < [Store](#store) > array |

<a name="store"></a>

### Store

| Name                              | Description                  | Schema                       |
| --------------------------------- | ---------------------------- | ---------------------------- |
| **storeId** <br>_required_        | The unique id of the store   | string                       |
| **name** <br>_required_           | The name of the store        | string                       |
| **url** <br>_optional_            | The url of the store         | string                       |
| **storeAddress** <br>_optional_   | The store's shipping address | <[Address](#address)> object |
| **invoiceAddress** <br>_optional_ | The store's invoice address  | <[Address](#address)> object |

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

<hr/>
<a name="getStore"></a>

### GET /stores/v1/{id}

**Operation: getStore**

#### Description

Retrieves the details of the requested store among all stores belonging to the user.

#### Parameters

| Name      | Description             | Schema                     |
| --------- | ----------------------- | -------------------------- | ------ |
| **Query** | **{id}** <br>_required_ | The unique id of the store | string |

#### Responses

| HTTP Code | Description                                                      | Schema                                |
| --------- | ---------------------------------------------------------------- | ------------------------------------- |
| **200**   | Successful                                                       | [GetStoreResponse](#GetStoreResponse) |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error)                       |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error)                       |
| **404**   | The requested store was not found.                               | [Error](#error)                       |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error)                       |

<a name="definitions"></a>

## Definitions

<a name="GetStoreResponse"></a>

### GetStoreResponse

| Name                     | Description       | Schema                     |
| ------------------------ | ----------------- | -------------------------- |
| **store** <br>_required_ | Store information | < [Store](#store) > object |

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

<hr/>
<a name="updateStore"></a>

### PATCH /stores/v1/{id}

**Operation: updateStore**

#### Description

Updates the information of the store with the id given as the query parameter.

#### Parameters

| Type      | Name                    | Description                | Schema |
| --------- | ----------------------- | -------------------------- | ------ |
| **Query** | **{id}** <br>_required_ | The unique id of the store | string |
| **Body**  | **Name** <br>_required_ | Store name                 | string |

#### Responses

| HTTP Code | Description                                                      | Schema          |
| --------- | ---------------------------------------------------------------- | --------------- |
| **200**   | Successful                                                       |                 |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error) |
| **404**   | Store not found                                                  | [Error](#error) |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error) |

<a name="definitions"></a>

## Definitions

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

<hr/>
<a name="updateStoreAddress"></a>

### PUT /stores/v1/{id}/storeAddress

**Operation: updateStoreAddress**

#### Description

Updates the address information of the store with the id given as the query parameter.

#### Parameters

| Type      | Name                       | Description                | Schema                       |
| --------- | -------------------------- | -------------------------- | ---------------------------- |
| **Query** | **{id}** <br>_required_    | The unique id of the store | string                       |
| **Body**  | **address** <br>_required_ | The store's new address    | <[Address](#address)> object |

#### Responses

| HTTP Code | Description                                                      | Schema          |
| --------- | ---------------------------------------------------------------- | --------------- |
| **200**   | Successful                                                       |                 |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error) |
| **404**   | Store not found                                                  | [Error](#error) |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error) |

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

<hr/>
<a name="updateInvoiceAddress"></a>

### PUT /stores/v1/{id}/invoiceAddress

**Operation: updateInvoiceAddress**

#### Description

Updates the invoice address information of the store with the id given as the query parameter.

#### Parameters

| Type      | Name                       | Description                | Schema                       |
| --------- | -------------------------- | -------------------------- | ---------------------------- |
| **Query** | **{id}** <br>_required_    | The unique id of the store | string                       |
| **Body**  | **address** <br>_required_ | The store's new address    | <[Address](#address)> object |

#### Responses

| HTTP Code | Description                                                      | Schema          |
| --------- | ---------------------------------------------------------------- | --------------- |
| **200**   | Successful                                                       |                 |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error) |
| **404**   | Store not found                                                  | [Error](#error) |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error) |

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

<hr/>
<a name="deleteStore"></a>

### DELETE /stores/v1/{id}

**Operation: deleteStore**

#### Description

Deactivates the store with the id given as the query parameter.

#### Parameters

| Type      | Name                    | Description                | Schema |
| --------- | ----------------------- | -------------------------- | ------ |
| **Query** | **{id}** <br>_required_ | The unique id of the store | string |

#### Responses

| HTTP Code | Description                                                      | Schema          |
| --------- | ---------------------------------------------------------------- | --------------- |
| **200**   | Successful                                                       |                 |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error) |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error) |

>

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

<hr/>
<a name="addStoreIossNumber"></a>

### POST /stores/v2/{id}/ioss

**Operation: addStoreIossNumber**

#### Description

Defines an IOSS (Import One-Stop Shop) number for the specified store. The defined number is used in European Union shipments that are created from the store's orders and whose amount is below 150 EUR. If the store already has an active IOSS number, the old number is deactivated and the new number becomes active.

#### Parameters

| Type     | Name                    | Description                                  | Schema                                                  |
| -------- | ----------------------- | -------------------------------------------- | ------------------------------------------------------- |
| **Path** | **{id}** <br>_required_ | The unique id of the store                   | string                                                  |
| **Body** | **body** <br>_required_ | The schema required to define an IOSS number | [AddStoreIossNumberRequest](#addStoreIossNumberRequest) |

#### Responses

| HTTP Code | Description                                                                                                                 | Schema          |
| --------- | --------------------------------------------------------------------------------------------------------------------------- | --------------- |
| **200**   | Successful                                                                                                                  |                 |
| **400**   | A request validation error occurred, the IOSS number format is invalid or the store is not eligible for an IOSS definition. | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired.                                                            | [Error](#error) |
| **404**   | Store not found                                                                                                             | [Error](#error) |
| **500**   | An unexpected error occurred during the request.                                                                            | [Error](#error) |

<a name="definitions"></a>

## Definitions

<a name="addStoreIossNumberRequest"></a>

### AddStoreIossNumberRequest

| Name                          | Description                                                                            | Schema |
| ----------------------------- | -------------------------------------------------------------------------------------- | ------ |
| **iossNumber** <br>_required_ | IOSS number. Must start with "IM" and be 12 characters in total. Example: IM1234567890 | string |

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

<hr/>
<a name="deleteStoreIossNumber"></a>

### DELETE /stores/v2/{id}/ioss

**Operation: deleteStoreIossNumber**

#### Description

Removes the active IOSS number of the specified store. The request also returns successfully if the store has no active IOSS number.

#### Parameters

| Type     | Name                    | Description                | Schema |
| -------- | ----------------------- | -------------------------- | ------ |
| **Path** | **{id}** <br>_required_ | The unique id of the store | string |

#### Responses

| HTTP Code | Description                                                      | Schema          |
| --------- | ---------------------------------------------------------------- | --------------- |
| **200**   | Successful                                                       |                 |
| **400**   | A request validation error occurred or the request is invalid.   | [Error](#error) |
| **401**   | Authorization error. The access token is invalid or has expired. | [Error](#error) |
| **404**   | Store not found                                                  | [Error](#error) |
| **500**   | An unexpected error occurred during the request.                 | [Error](#error) |

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

<hr/>
<a name="shipStoreOrder"></a>

### POST /stores/v2/{store_id}/orders/{order_reference}/ship

**Operation: shipStoreOrder**

#### Description

Starts the shipment process for the specified store and order through the selected quote. The selectable additional services are returned by the createOrderQuote API. An additional service that is not returned for the relevant quote cannot be used at the shipOrder stage.

#### Parameters

| Type     | Name                               | Description          | Schema                                        |
| -------- | ---------------------------------- | -------------------- | --------------------------------------------- |
| **Path** | **store_id** <br>_required_        | Store ID             | Guid                                          |
| **Path** | **order_reference** <br>_required_ | Order reference      | string                                        |
| **Body** | **body** <br>_required_            | Shipment information | [StoreShipmentRequest](#storeShipmentRequest) |

#### Responses

| HTTP Code | Description                                                     | Schema                                  |
| --------- | --------------------------------------------------------------- | --------------------------------------- |
| **200**   | Successful                                                      | [ShipOrderResponse](#shipOrderResponse) |
| **400**   | A request validation error occurred or the request is invalid.  | [Error](#error)                         |
| **401**   | Authorization error. The access token is invalid or has expired | [Error](#error)                         |
| **404**   | Store or order not found                                        | [Error](#error)                         |
| **500**   | An unexpected error occurred during the request.                | [Error](#error)                         |

<a name="definitions"></a>

## Definitions

<a name="storeShipmentRequest"></a>

### StoreShipmentRequest

| Name                                          | Description                                                     | Schema           |
| --------------------------------------------- | --------------------------------------------------------------- | ---------------- |
| **quoteReference** <br>_required_             | The unique id of the quote                                      | Guid             |
| **searchId** <br>_required_                   | The unique id of the quote search operation                     | Guid             |
| **selectedAdditionalServices** <br>_required_ | List of selected additional service codes. Valid values: "ddp". | < string > array |

<a name="shipOrderResponse"></a>

### ShipOrderResponse

| Name                                 | Description                                                                                                 | Schema       |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- | ------------ |
| **shipmentId** <br>_required_        | Unique identifier of the shipment                                                                           | Guid         |
| **shipmentReference** <br>_required_ | Shipment number                                                                                             | string       |
| **trackingUrl** <br>_required_       | Shipment tracking link                                                                                      | string       |
| **cargoLabels** <br>_required_       | List of barcode numbers required for cargo acceptance at the warehouse. One number is returned per package. | List<string> |
| **chargeableWeight** <br>_optional_  | Total chargeable weight (kg)                                                                                | decimal      |

<a name="trackOrder"></a>

### GET /stores/v1/{id}/ordertracking?orderReference={orderId}

#### Description

Returns the tracking information of an order, if any. Returns 403 if the order and store information is entered incorrectly, and 404 if the tracking information for the order has not been created yet.

#### Parameters

| Type            | Name                              | Description | Schema |
| --------------- | --------------------------------- | ----------- | ------ |
| **Query**       | **id** <br>_required_             | Store id    | string |
| **QueryString** | **orderReference** <br>_required_ | Order id    | string |

#### Responses

| HTTP Code | Description                                                     | Schema                                |
| --------- | --------------------------------------------------------------- | ------------------------------------- |
| **200**   | Successful                                                      | [TrackingResponse](#trackingResponse) |
| **400**   | A request validation error occurred or the request is invalid.  | [Error](#error)                       |
| **401**   | Authorization error. The access token is invalid or has expired | [Error](#error)                       |
| **403**   | Store or order information is incorrect                         | [Error](#error)                       |
| **403**   | Tracking number not created yet                                 | [Error](#error)                       |
| **500**   | An unexpected error occurred during the request.                | [Error](#error)                       |

<a name="definitions"></a>

## Definitions

<a name="trackingResponse"></a>

### TrackingResponse

| Name                              | Description                                  | Schema                                     |
| --------------------------------- | -------------------------------------------- | ------------------------------------------ |
| **trackingNumber** <br>_required_ | Tracking number                              | string                                     |
| **courier** <br>_required_        | Carrier company code                         | string                                     |
| **courierTrackingLink**           | The carrier company's tracking link          | string                                     |
| **status** <br>_required_         | Status of the shipment                       | < [TrackingStatus](#trackingStatus) > Enum |
| **signedBy**                      | The person who received the shipment, if any | string                                     |
| **shipmentType**                  | The carrier company's service type           | string                                     |
| **checkpoints**                   | Tracking events of the shipment              | < [Checkpoint](#checkpoint) > Array        |

<a name="checkpoint"></a>

### Checkpoint

| Name                              | Description                             | Schema                                     |
| --------------------------------- | --------------------------------------- | ------------------------------------------ |
| **checkpointTime** <br>_required_ | Event time                              | string                                     |
| **status** <br>_required_         | Status of the shipment during the event | < [TrackingStatus](#trackingStatus) > Enum |
| **message**                       | Message about the event                 | string                                     |

<a name="trackingStatus"></a>

### TrackingStatus

| Name                   | Description                                                                                                 |
| ---------------------- | ----------------------------------------------------------------------------------------------------------- |
| **InfoReceived**       | Carrier has received request from shipper and is about to pick up the shipment.                             |
| **InTransit**          | Carrier has accepted or picked up shipment from shipper. The shipment is on the way.                        |
| **OutForDelivery**     | Carrier is about to deliver the shipment , or it is ready to pickup.                                        |
| **AttemptFail**        | Carrier attempted to deliver but failed, and usually leaves a notice and will try to deliver again.         |
| **Delivered**          | The shipment was delivered successfully.                                                                    |
| **AvailableForPickup** | The package arrived at a pickup point near you and is available for pickup.                                 |
| **Exception**          | Custom hold, undelivered, returned shipment to sender or any shipping exceptions.                           |
| **Expired**            | Shipment has no tracking information for 30 days since added.                                               |
| **Pending**            | New shipments added that are pending to track, or new shipments without tracking information available yet. |
