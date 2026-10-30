# Rates API

<a name="overview"></a>

## Overview

With this API, carrier rates are obtained by sending the shipment information (addresses, products, packages). Each call opens a new shipment record and `reference` in Navlungo; the rate is then selected with [createShipment](./shipment.md#createShipment).

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

This API can be called with tokens issued through the OAuth2 **client_credentials** flow. When obtaining the token, the value **shipping_rates_write** must be sent in the scope parameter.

### Operations

[createRates](#createRates)<br>

<a name="paths"></a>

## Paths

<a name="createRates"></a>

### POST api/shipping/v1/rates

**Operation: createRates**

#### Description

Generates carrier rates from the shipment information and opens a new shipment reference. The `reference` and `searchId` from the response, together with the `rateId` of the rate to be selected, are used in the [createShipment](./shipment.md#createShipment) request.

#### Rate Limit

- At most 10 requests per 10 seconds per client

#### Parameters

| Type     | Name                    | Description                      | Schema                        |
| -------- | ----------------------- | -------------------------------- | ----------------------------- |
| **Body** | **body** <br>_required_ | The schema required to get rates | [RatesRequest](#ratesRequest) |

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Schema                            |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **200**   | Successful. If no rate can be generated, `rates` is an empty array and `searchId` is `00000000-0000-0000-0000-000000000000`; the `reference` is still created.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | [RatesResponse](#ratesResponse)   |
| **400**   | A request validation error occurred ([ProblemDetails](#problemDetails)) or a business rule error ([Error](#error)):<br> `shipment.fromanddestinationcountrycannotbesame.error` - Sender and receiver countries are the same (the client does not have the `can_create_domestic_shipments` permission)<br> `createshipment.fromcountrycodeisnotvalid.error`, `createshipment.invoicecountrycodeisnotvalid.error`, `createshipment.destinationcountrycodeisnotvalid.error` - Invalid country code<br> `createshipment.destinationcountrystatecodeisnotvalid.error`, `createshipment.sendercountrystatecodeisnotvalid.error`, `createshipment.invoicecountrystateisnotvalid.error` - Invalid state code<br> `proformaaddresserrors.identitynumbermustbevalid.error`, `proformaaddresserrors.taxnumbermustbevalid.error` - Invalid TR identity/tax number<br> `packageerrors.envelopweightlimitexceeded.error`, `packageerrors.documentweightlimitexceeded.error`, `packageerrors.differenttypesofpackagescannotbeshippedtogether.error` - Package rules<br> `producterrors.invaliddocumenthscode.error`, `producterrors.invaliddocumentprice.error` - Product rules for document-type packages<br> `producterrors.merchantproductidentifierrequiredforeuropeanunion.error`, `producterrors.nonstandardisedmanufacturerproductidentifierrequiredforeuropeanunion.error` - Receiver is in an EU member state, the shipment total is 150 EUR or less and a product identifier is missing ([EU Product Identifiers](#euProductIdentifiers))<br> `producterrors.productidentifiertoolong.error` - Product identifier exceeds 100 characters<br> `createshipment.invalidgtipcodesareexist.error` - HS/GTİP code not recognized<br> `createshipment.countrylistcouldnotbefetch.error`, `createshipment.statelistcouldnotbefetch.error`, `ratescouldnotbegenerated.error` - Service error while generating rates | [Error](#error)                   |
| **401**   | Authorization error. The access token is invalid, has expired or does not have the `shipping_rates_write` scope                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [ProblemDetails](#problemDetails) |
| **429**   | Rate limit exceeded                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | [Error](#error)                   |
| **500**   | An unexpected error occurred during the request                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [ProblemDetails](#problemDetails) |

#### Example Request Body

```
{
  "shipmentType": "sales",
  "currency": "USD",
  "weightUnit": "lb",
  "dimensionUnit": "in",
  "iossNumber": null,
  "senderShippingAddress": {
    "contactName": "Ayşe Yılmaz", "companyName": "Yılmaz Tekstil A.Ş.", "countryCode": "TR", "stateCode": null,
    "city": "İstanbul", "town": "Kadıköy", "postalCode": "34710", "firstLine": "Caferağa Mah. Moda Cad. No:12",
    "secondLine": null, "thirdLine": null, "email": "ayse@yilmaztekstil.com", "phoneCode": "+90", "phoneNumber": "5321234567",
    "identificationNumber": "1234567890", "taxOffice": "Kadıköy"
  },
  "senderInvoiceAddress": {
    "contactName": "Ayşe Yılmaz", "companyName": "Yılmaz Tekstil A.Ş.", "countryCode": "TR", "stateCode": null,
    "city": "İstanbul", "town": "Kadıköy", "postalCode": "34710", "firstLine": "Caferağa Mah. Moda Cad. No:12",
    "secondLine": null, "thirdLine": null, "email": "ayse@yilmaztekstil.com", "phoneCode": "+90", "phoneNumber": "5321234567",
    "identificationNumber": "1234567890", "taxOffice": "Kadıköy"
  },
  "receiverAddress": {
    "contactName": "John Carter", "companyName": null, "countryCode": "US", "stateCode": "NY",
    "city": "New York", "postalCode": "10001", "firstLine": "350 5th Ave Suite 4400", "secondLine": null, "thirdLine": null,
    "email": "john.carter@example.com", "phoneCode": "+1", "phoneNumber": "2125550147", "identificationNumber": null
  },
  "products": [
    {
      "description": "Cotton t-shirt, short sleeve", "hsCode": "6109.10.00.12", "quantity": 10, "unitPrice": 12.5, "currency": "USD", "originCountry": "TR",
      "merchantProductIdentifier": "TS-COTTON-M-WHT", "nonStandardisedManufacturerProductIdentifier": "YT-6109-M", "standardisedManufacturerProductIdentifier": "8690000000001"
    }
  ],
  "packages": [
    { "type": "box", "quantity": 2, "weight": 4.4, "length": 16, "width": 12, "height": 8 }
  ]
}
```

#### Example Response

```
{
  "reference": 458213,
  "searchId": "3f0c9a4e-7b2d-4c7e-9a1b-2e5d6f7a8b9c",
  "rates": [
    {
      "rateId": "9b7e2c11-4d3a-4f6e-8c2a-1d2e3f4a5b6c",
      "price": 48.9, "currency": "USD",
      "serviceType": "express", "carrier": "United Parcel Service",
      "minTransitTime": 3, "maxTransitTime": 5,
      "description": "UPS Express Saver", "chargeableWeight": 4.0,
      "services": [
        { "code": "ddp", "price": 6.5, "currency": "USD", "isRequired": false },
        { "code": "customs-tax-duty", "price": 11.2, "currency": "USD", "isRequired": true }
      ]
    }
  ]
}
```

<a name="definitions"></a>

## Definitions

<a name="ratesRequest"></a>

### RatesRequest

| Name                                     | Description                                                                                                                                                                                                                   | Schema                              |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| **shipmentType** <br>_required_          | Shipment type. Enum: `sales`, `sample`, `gift`, `micro-export`                                                                                                                                                                | string                              |
| **currency** <br>_required_              | Shipment currency. Enum: `EUR`, `USD`, `TRY`, `GBP`, `CAD`, `SAR`. Products with a different currency are converted to this currency                                                                                          | string                              |
| **weightUnit** <br>_optional_            | Weight unit: `kg` (default) or `lb`. Given together with `dimensionUnit` (`kg`/`cm` or `lb`/`in`)                                                                                                                             | string                              |
| **dimensionUnit** <br>_optional_         | Dimension unit: `cm` (default) or `in`. `lb`/`in` values are converted to kg/cm (`1 lb = 0.453592 kg`, `1 in = 2.54 cm`, 2 decimals)                                                                                          | string                              |
| **iossNumber** <br>_optional_            | IOSS number; 12 characters, `IM` + 10 letters/digits (e.g. `IM1234567890`). Only stored and reflected in the rates for shipments going to an EU country with a total product value below 150 EUR; ignored for other shipments | string                              |
| **senderShippingAddress** <br>_required_ | Sender's origin (ship-from) address                                                                                                                                                                                           | [SenderAddress](#senderAddress)     |
| **senderInvoiceAddress** <br>_required_  | Sender's invoice address                                                                                                                                                                                                      | [SenderAddress](#senderAddress)     |
| **receiverAddress** <br>_required_       | Receiver address                                                                                                                                                                                                              | [ReceiverAddress](#receiverAddress) |
| **products** <br>_required_              | Product lines. At least 1                                                                                                                                                                                                     | < [Product](#product) > array       |
| **packages** <br>_required_              | Packages. At least 1, total `quantity` at most 20. Only one package type can be used in a shipment                                                                                                                            | < [Package](#package) > array       |

<a name="senderAddress"></a>

### SenderAddress

| Name                                    | Description                                                                                                                                                                                             | Schema |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **contactName** <br>_required_          | Contact person. 5–50 characters                                                                                                                                                                         | string |
| **companyName** <br>_optional_          | Company name, at most 256 characters. If filled, the address is considered corporate                                                                                                                    | string |
| **countryCode** <br>_required_          | Country code (ISO2), upper case                                                                                                                                                                         | string |
| **stateCode** <br>_optional_            | State code                                                                                                                                                                                              | string |
| **city** <br>_required_                 | City, at most 50 characters                                                                                                                                                                             | string |
| **town** <br>_required_                 | District, at most 128 characters. For TR see the [province and district list](../store-api/city-town.md)                                                                                                | string |
| **postalCode** <br>_required_           | Postal code, at most 32 characters                                                                                                                                                                      | string |
| **firstLine** <br>_required_            | Address line, 10–30 characters                                                                                                                                                                          | string |
| **secondLine** <br>_optional_           | Second address line, at most 30 characters                                                                                                                                                              | string |
| **thirdLine** <br>_optional_            | Third address line, at most 30 characters                                                                                                                                                               | string |
| **email** <br>_required_                | E-mail                                                                                                                                                                                                  | string |
| **phoneCode** <br>_required_            | Phone country code (e.g. `+90`), at most 25 characters                                                                                                                                                  | string |
| **phoneNumber** <br>_required_          | Phone number, 5–21 characters                                                                                                                                                                           | string |
| **identificationNumber** <br>_required_ | Identity/tax number, at most 64 characters. For TR, the 11-digit Turkish ID number (TCKN) for an individual address and the 10-digit tax ID number (VKN) for a corporate address (`companyName` filled) | string |
| **taxOffice** <br>_optional_            | Tax office, at most 256 characters                                                                                                                                                                      | string |

<a name="receiverAddress"></a>

### ReceiverAddress

| Name                                    | Description                                                                                                                                           | Schema |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **contactName** <br>_required_          | Receiver contact person. 5–50 characters                                                                                                              | string |
| **companyName** <br>_optional_          | Company name                                                                                                                                          | string |
| **countryCode** <br>_required_          | Country code (ISO2). Must differ from the sender country; it may be the same country if the client has the `can_create_domestic_shipments` permission | string |
| **stateCode** <br>_optional_            | State code                                                                                                                                            | string |
| **city** <br>_required_                 | City, at most 50 characters                                                                                                                           | string |
| **postalCode** <br>_optional_           | Postal code, at most 32 characters                                                                                                                    | string |
| **firstLine** <br>_required_            | Address line, 10–30 characters                                                                                                                        | string |
| **secondLine** <br>_optional_           | Second address line, at most 30 characters                                                                                                            | string |
| **thirdLine** <br>_optional_            | Third address line, at most 30 characters                                                                                                             | string |
| **email** <br>_optional_                | E-mail                                                                                                                                                | string |
| **phoneCode** <br>_optional_            | Phone country code. Required if `phoneNumber` is given                                                                                                | string |
| **phoneNumber** <br>_optional_          | Phone number, 5–21 characters                                                                                                                         | string |
| **identificationNumber** <br>_optional_ | Identity/tax number                                                                                                                                   | string |

<a name="product"></a>

### Product

| Name                                                            | Description                                                                                                                                                                                                                                                                                                                     | Schema  |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| **description** <br>_required_                                  | Product description, 5–500 characters                                                                                                                                                                                                                                                                                           | string  |
| **hsCode** <br>_required_                                       | Product HS code, at most 20 characters (at least 6 characters if the receiver is not `US`). Must be a real code: the US HTS code (`6109.10.00.12` format) if the receiver is `US`, the 12-digit TR GTİP code (`610910000000`) for other countries. An unrecognized code returns `createshipment.invalidgtipcodesareexist.error` | string  |
| **quantity** <br>_required_                                     | Product quantity, greater than 0                                                                                                                                                                                                                                                                                                | int     |
| **unitPrice** <br>_required_                                    | Unit price, greater than 0                                                                                                                                                                                                                                                                                                      | decimal |
| **currency** <br>_required_                                     | Product currency. Enum: `EUR`, `USD`, `TRY`, `GBP`, `CAD`, `SAR`                                                                                                                                                                                                                                                                | string  |
| **originCountry** <br>_required_                                | Country of origin code (ISO2)                                                                                                                                                                                                                                                                                                   | string  |
| **merchantProductIdentifier** <br>_optional_                    | Merchant product identifier (Merchant PID), usually the merchant's SKU. Required if the receiver is in an EU member state and the shipment total is 150 EUR or less (except for `document` packages), at most 100 characters. See [EU Product Identifiers](#euProductIdentifiers)                                               | string  |
| **nonStandardisedManufacturerProductIdentifier** <br>_optional_ | The manufacturer's non-standardised product identifier (NS-PID), e.g. the manufacturer's model or part number. Required if the receiver is in an EU member state and the shipment total is 150 EUR or less (except for `document` packages), at most 100 characters                                                             | string  |
| **standardisedManufacturerProductIdentifier** <br>_optional_    | Standardised product identifier (S-PID): GTIN, EAN, UPC or ISBN. Should be sent if the product has a standardised identifier                                                                                                                                                                                                    | string  |

<a name="euProductIdentifiers"></a>

#### EU Product Identifiers (PID)

Under EU customs regulation, from 1 November 2026 shipments going to the EU require product identifiers for every product in the customs declaration. Navlungo applies this rule to shipments whose receiver country (`receiverAddress.countryCode`) is an EU member state and whose total product amount (`quantity` × `unitPrice`) is **150 EUR or less** in EUR equivalent. Currencies other than EUR are converted to EUR at the exchange rate of the day the shipment is created:

- `merchantProductIdentifier` and `nonStandardisedManufacturerProductIdentifier` are required for every product. An empty or whitespace-only value counts as missing; the request returns `400` with `producterrors.merchantproductidentifierrequiredforeuropeanunion.error` or `producterrors.nonstandardisedmanufacturerproductidentifierrequiredforeuropeanunion.error`.
- These fields are not required for `document` type packages.
- `standardisedManufacturerProductIdentifier` is not required; it should be sent if the product has a GTIN, EAN, UPC or ISBN code.
- Leading and trailing whitespace in the values is trimmed. `merchantProductIdentifier` and `nonStandardisedManufacturerProductIdentifier` can be at most 100 characters (`producterrors.productidentifiertoolong.error`).
- For EU shipments with a total above 150 EUR and for receivers outside the EU, the fields are optional.
- For EU shipments, the entered values are forwarded to the carrier regardless of the amount. Since some carriers may also request this information for shipments above 150 EUR, it is recommended to send it for EU shipments regardless of the amount.

Example product for a shipment whose receiver is in an EU member state and whose total is 150 EUR or less (10 × 12.5 EUR = 125 EUR):

```
"products": [
  {
    "description": "Cotton t-shirt, short sleeve", "hsCode": "610910000000", "quantity": 10, "unitPrice": 12.5, "currency": "EUR", "originCountry": "TR",
    "merchantProductIdentifier": "TS-COTTON-M-WHT",
    "nonStandardisedManufacturerProductIdentifier": "YT-6109-M",
    "standardisedManufacturerProductIdentifier": "8690000000001"
  }
]
```

<a name="package"></a>

### Package

| Name                        | Description                                                                                                                                                                                                                                        | Schema  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| **type** <br>_required_     | Package type. Enum: `box`, `envelope`, `document`. `envelope` at most 3 kg per package; `document` at most 0.25 kg per package, a single product, `quantity: 1`, `unitPrice: 1`, `hsCode` `4819.60.00.00` for the US, `481960` for other countries | string  |
| **quantity** <br>_required_ | Number of packages                                                                                                                                                                                                                                 | int     |
| **weight** <br>_required_   | Package weight (in the `weightUnit` unit), greater than 0                                                                                                                                                                                          | decimal |
| **length** <br>_optional_   | Package length (in the `dimensionUnit` unit). Required for `box` and greater than 0                                                                                                                                                                | decimal |
| **width** <br>_optional_    | Package width. Required for `box` and greater than 0                                                                                                                                                                                               | decimal |
| **height** <br>_optional_   | Package height. Required for `box` and greater than 0                                                                                                                                                                                              | decimal |

<a name="ratesResponse"></a>

### RatesResponse

| Name                         | Description                                                                   | Schema                  |
| ---------------------------- | ----------------------------------------------------------------------------- | ----------------------- |
| **reference** <br>_required_ | Navlungo shipment reference. Used in all subsequent calls                     | long                    |
| **searchId** <br>_required_  | The id of the rate search. Empty GUID if there are no rates                   | string (Guid)           |
| **rates** <br>_required_     | The generated rates. An empty array is returned if no rate could be generated | < [Rate](#rate) > array |

<a name="rate"></a>

### Rate

| Name                                | Description                                                                 | Schema                                |
| ----------------------------------- | --------------------------------------------------------------------------- | ------------------------------------- |
| **rateId** <br>_required_           | Rate id. Sent in the [createShipment](./shipment.md#createShipment) request | string (Guid)                         |
| **price** <br>_required_            | Rate amount                                                                 | decimal                               |
| **currency** <br>_required_         | Rate currency                                                               | string                                |
| **serviceType** <br>_required_      | Service type. See [Value Lists](./README.md#valueLists)                     | string                                |
| **carrier** <br>_required_          | Carrier name                                                                | string                                |
| **minTransitTime** <br>_required_   | Minimum transit time (days)                                                 | int                                   |
| **maxTransitTime** <br>_required_   | Maximum transit time (days)                                                 | int                                   |
| **description** <br>_required_      | Rate description                                                            | string                                |
| **chargeableWeight** <br>_required_ | Chargeable weight (kg)                                                      | decimal                               |
| **services** <br>_required_         | Additional services                                                         | < [RateService](#rateService) > array |

<a name="rateService"></a>

### RateService

| Name                          | Description                                                                                                             | Schema  |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------- |
| **code** <br>_required_       | Additional service code: `ddp`, `insurance`, `customs-tax-duty`, `atr`, `premium-tracking` (cannot be selected)         | string  |
| **price** <br>_required_      | Additional service amount                                                                                               | decimal |
| **currency** <br>_required_   | Additional service currency                                                                                             | string  |
| **isRequired** <br>_required_ | If `true`, the service must be sent in `selectedServices` in the [createShipment](./shipment.md#createShipment) request | boolean |

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
