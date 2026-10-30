# Express Quote API

<a name="overview"></a>

## Overview

Navlungo can generate express service quotes for loads with a chargeable weight of up to 300 kg. This API will give Navlungo solution partners access to express quote search processes.

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

[createOrderQuote](#createOrderQuote)<br>

<a name="paths"></a>

## Paths

<a name="createOrderQuote"></a>

### POST stores/v2/{store_id}/orders

**Operation: createOrderQuote**

#### Description

Creates a quote for the specified store using the order information and package details.

#### Parameters

| Type     | Name                        | Description                           | Schema                                  |
| -------- | --------------------------- | ------------------------------------- | --------------------------------------- |
| **Path** | **store_id** <br>_required_ | Store identifier                      | string                                  |
| **Body** | **body** <br>_required_     | The schema required to create a quote | [OrderQuoteRequest](#orderQuoteRequest) |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Schema                                    |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **200**   | Successful                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | [OrderQuoteResponse](#orderQuoteResponse) |
| **400**   | A request validation error occurred or the request is invalid. Product identifier errors ([EU Product Identifiers](#euProductIdentifiers)):<br> `producterrors.merchantproductidentifierrequiredforeuropeanunion.error`, `producterrors.nonstandardisedmanufacturerproductidentifierrequiredforeuropeanunion.error` - Receiver is in an EU member state, the order total is 150 EUR or less and a product identifier is missing<br> `Common/Validation` - Product identifier exceeds 100 characters (`extensions.validationErrors`) | [Error](#error)                           |
| **401**   | Authorization error. The access token is invalid or has expired                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | [Error](#error)                           |
| **500**   | An unexpected error occurred during the request.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | [Error](#error)                           |

<a name="definitions"></a>

## Definitions

<a name="generateQuoteRequest"></a>

### GenerateQuoteRequest

| Name                                  | Description                                                                                                                                                                                   | Schema                  |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| **originCountry** <br>_required_      | ISO2 code of the country the load departs from                                                                                                                                                | string                  |
| **originRegion** <br>_optional_       | Code of the region the load departs from. Preferably the postal code should be sent. If this field is not sent correctly, additional domestic transport costs may be reflected in the prices. | string                  |
| **destinationCountry** <br>_required_ | ISO2 code of the country the load is sent to                                                                                                                                                  | string                  |
| **loads** <br>_required_              | Package information of the loads to be transported                                                                                                                                            | < [Load](#load) > array |

<a name="orderQuoteRequest"></a>

### OrderQuoteRequest

| Name                            | Description                                                    | Schema                        |
| ------------------------------- | -------------------------------------------------------------- | ----------------------------- |
| **order** <br>_required_        | Order information                                              | [Order](#order)               |
| **packages** <br>_required_     | Information of the packages to be transported                  | < [Package](#package) > array |
| **shipmentType** <br>_required_ | Shipment type. Enum: "sales", "sample", "micro-export", "gift" | string                        |

<a name="order"></a>

### Order

| Name                                   | Description                  | Schema                              |
| -------------------------------------- | ---------------------------- | ----------------------------------- |
| **orderReference** <br>_required_      | Order reference code         | string                              |
| **currencyCode** <br>_required_        | Currency code                | string                              |
| **receiverAddress** <br>_required_     | Receiver address information | [ReceiverAddress](#receiverAddress) |
| **receiverEmail** <br>_optional_       | Receiver e-mail address      | string                              |
| **receiverPhoneNumber** <br>_required_ | Receiver phone number        | string                              |
| **orderItems** <br>_required_          | Products in the order        | < [OrderItem](#orderItem) > array   |

<a name="receiverAddress"></a>

### ReceiverAddress

| Name                           | Description           | Schema |
| ------------------------------ | --------------------- | ------ |
| **contactName** <br>_required_ | Receiver contact name | string |
| **countryCode** <br>_required_ | Country code (ISO2)   | string |
| **state** <br>_optional_       | State                 | string |
| **town** <br>_required_        | District              | string |
| **city** <br>_required_        | City                  | string |
| **postalCode** <br>_required_  | Postal code           | string |
| **firstLine** <br>_required_   | Address line          | string |

<a name="orderItem"></a>

### OrderItem

| Name                                                            | Description                                                                                                                                                                                                                                                                                                           | Schema |
| --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **quantity** <br>_required_                                     | Product quantity                                                                                                                                                                                                                                                                                                      | int    |
| **price** <br>_required_                                        | Product price                                                                                                                                                                                                                                                                                                         | string |
| **description** <br>_required_                                  | Product description                                                                                                                                                                                                                                                                                                   | string |
| **sku** <br>_required_                                          | Stock code. If `merchantProductIdentifier` is not sent and the SKU is at most 100 characters, the SKU is used as the merchant product identifier                                                                                                                                                                      | string |
| **hsCode** <br>_required_                                       | Product HS code                                                                                                                                                                                                                                                                                                       | string |
| **merchantProductIdentifier** <br>_optional_                    | Merchant product identifier (Merchant PID), usually the merchant's SKU. Required if the receiver is in an EU member state and the order total is 150 EUR or less (except for `document` packages), at most 100 characters. If not sent, `sku` is used. See [EU Product Identifiers](#euProductIdentifiers)            | string |
| **nonStandardisedManufacturerProductIdentifier** <br>_optional_ | The manufacturer's non-standardised product identifier (NS-PID), e.g. the manufacturer's model or part number. Required if the receiver is in an EU member state and the order total is 150 EUR or less (except for `document` packages), at most 100 characters. See [EU Product Identifiers](#euProductIdentifiers) | string |
| **standardisedManufacturerProductIdentifier** <br>_optional_    | Standardised product identifier (S-PID): GTIN, EAN, UPC or ISBN, at most 100 characters. Should be sent if the product has a standardised identifier                                                                                                                                                                  | string |

<a name="euProductIdentifiers"></a>

#### EU Product Identifiers (PID)

Under EU customs regulation, from 1 November 2026 shipments going to the EU require product identifiers for every product in the customs declaration. Navlungo applies this rule to orders whose receiver country (`order.receiverAddress.countryCode`) is an EU member state and whose total product amount (`quantity` × `price`) is **150 EUR or less** in EUR equivalent. If `order.currencyCode` is not EUR, the amount is converted to EUR at the exchange rate of the day the shipment is created. The check is performed in this request, while the shipment is being created; if an identifier is missing, neither the shipment nor the quote is created.

- `merchantProductIdentifier` and `nonStandardisedManufacturerProductIdentifier` are required for every product. An empty or whitespace-only value counts as missing; the request returns `400` with `producterrors.merchantproductidentifierrequiredforeuropeanunion.error` or `producterrors.nonstandardisedmanufacturerproductidentifierrequiredforeuropeanunion.error`.
- If `merchantProductIdentifier` is not sent, the product's `sku` value (if it is at most 100 characters) is used as the merchant product identifier. For products that already have an SKU, adding only `nonStandardisedManufacturerProductIdentifier` is sufficient.
- These fields are not required for `document` type packages.
- `standardisedManufacturerProductIdentifier` is not required; it should be sent if the product has a GTIN, EAN, UPC or ISBN code.
- Leading and trailing whitespace in the values is trimmed. All three fields can be at most 100 characters; for a value that exceeds this, the request returns `400` with `Common/Validation` and the field is indicated in `extensions.validationErrors`.
- For EU orders with a total above 150 EUR and for receivers outside the EU, the fields are optional.
- For EU shipments, the entered values are forwarded to the carrier regardless of the amount. Since some carriers may also request this information for shipments above 150 EUR, it is recommended to send it for EU shipments regardless of the amount.

Example product for an order whose receiver is in an EU member state and whose total is 150 EUR or less (`currencyCode` `EUR`, 2 × 25 EUR = 50 EUR):

```
"orderItems": [
  {
    "sku": "TS-COTTON-M-WHT", "description": "Cotton t-shirt, short sleeve", "quantity": 2, "price": 25, "hsCode": "610910000000",
    "merchantProductIdentifier": "TS-COTTON-M-WHT",
    "nonStandardisedManufacturerProductIdentifier": "YT-6109-M",
    "standardisedManufacturerProductIdentifier": "8690000000001"
  }
]
```

<a name="package"></a>

### Package

| Name                        | Description                          | Schema  |
| --------------------------- | ------------------------------------ | ------- |
| **quantity** <br>_required_ | Number of packages                   | int     |
| **type** <br>_required_     | Package type (e.g. "box")            | string  |
| **weight** <br>_required_   | Weight of the package in kilograms   | decimal |
| **width** <br>_required_    | Width of the package in centimetres  | decimal |
| **length** <br>_required_   | Length of the package in centimetres | decimal |
| **height** <br>_required_   | Height of the package in centimetres | decimal |

<a name="orderQuoteResponse"></a>

### OrderQuoteResponse

| Name                        | Description                                                                      | Schema                              |
| --------------------------- | -------------------------------------------------------------------------------- | ----------------------------------- |
| **searchId** <br>_required_ | Unique id of the created search request                                          | string                              |
| **quotes** <br>_optional_   | The generated quotes. An empty array is returned if no quote could be generated. | < [StoreQuote](#storeQuote) > array |

<a name="storeQuote"></a>

### StoreQuote

| Name                                  | Description                               | Schema                                            |
| ------------------------------------- | ----------------------------------------- | ------------------------------------------------- |
| **quoteReference** <br>_required_     | Reference id of the quote                 | string                                            |
| **price** <br>_required_              | Quote amount                              | decimal                                           |
| **currency** <br>_required_           | Quote currency                            | string                                            |
| **serviceType** <br>_required_        | Service type (express, eco-express, etc.) | string                                            |
| **minTransitTime** <br>_optional_     | Minimum transit time (days)               | int                                               |
| **maxTransitTime** <br>_optional_     | Maximum transit time (days)               | int                                               |
| **description** <br>_required_        | Quote description                         | string                                            |
| **carrier** <br>_required_            | Carrier company                           | string                                            |
| **additionalServices** <br>_required_ | Additional service information            | < [AdditionalService](#additionalService) > array |

<a name="additionalService"></a>

### AdditionalService

| Name                           | Description                                                 | Schema  |
| ------------------------------ | ----------------------------------------------------------- | ------- |
| **serviceCode** <br>_required_ | Additional service code. Valid values: "ddp"                | string  |
| **priceAmount** <br>_required_ | Additional service amount                                   | decimal |
| **currency** <br>_required_    | Additional service currency                                 | string  |
| **isRequired** <br>_required_  | Value indicating whether the additional service is required | boolean |

<a name="load"></a>

### Load

Packaging information of the load to be transported.

| Name                      | Description                                                                           | Schema  |
| ------------------------- | ------------------------------------------------------------------------------------- | ------- |
| **type** <br>_required_   | Package type. Must be Box or Envelope. (Case sensitive)                               | string  |
| **count** <br>_required_  | Number of packages. Can only be 1 for Envelope type packages.                         | string  |
| **weight** <br>_required_ | Weight of the package in kilograms. Can be at most 2.5 kg for Envelope type packages. | decimal |
| **width** <br>_optional_  | Width of the package in centimetres. This value is required for Box type packages.    | decimal |
| **length** <br>_optional_ | Length of the package in centimetres. This value is required for Box type packages.   | decimal |
| **height** <br>_optional_ | Height of the package in centimetres. This value is required for Box type packages.   | decimal |

<a name="quoteResponse"></a>

### QuoteResponse

| Name                                     | Description                                                                      | Schema                    |
| ---------------------------------------- | -------------------------------------------------------------------------------- | ------------------------- |
| **quoteRequestReference** <br>_required_ | Reference id of the created quote request                                        | string                    |
| **quotes** <br>_optional_                | The generated quotes. An empty array is returned if no quote could be generated. | < [Quote](#quote) > array |

<a name="quote"></a>

### Quote

Quote information

| Name                              | Description                        | Schema  |
| --------------------------------- | ---------------------------------- | ------- |
| **quoteReference** <br>_required_ | Reference id of the quote          | string  |
| **price** <br>_required_          | Quote amount                       | decimal |
| **currency** <br>_required_       | Quote currency                     | string  |
| **proposal** <br>_required_       | Quote type. ( express, ecoExpress) | string  |
| **minTransitTime** <br>_optional_ | Minimum transit time (days)        | int     |
| **maxTransitTime** <br>_optional_ | Maximum transit time (days)        | int     |
| **description** <br>_required_    | Quote description                  | string  |
| **isDoorToDoor** <br>_required_   | Is it door to door?                | bool    |

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
