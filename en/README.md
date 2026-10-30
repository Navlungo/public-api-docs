# Navlungo API

> 🇹🇷 [Türkçe](../README.md)

<a name="overview"></a>

## Important Notes

1. The client credentials provided to you can only be used in the corresponding environment. You cannot access the production APIs with the client credentials provided for the QA environment.
2. Since the QA and Production environments are isolated, you cannot log in to the QA environment with your production user credentials. If you need a user to work in QA, you can create a new user via the sign-up button and use it in your tests.
3. The QA environment is shut down by default on weekday nights and at weekends, so you may not get a response from the APIs. If you plan your work and let us know in advance, we can keep the environment up.

## 1. Overview

The Navlungo API gives Navlungo solution partners programmatic access to shipment processes so that they can build the most suitable experience for their own customers. Two separate integration models are offered:

| Section                                  | Acts on behalf of                          | Authorization                       | Scope                                                                                                                       |
| ---------------------------------------- | ------------------------------------------ | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| [Store API](./store-api/README.md)       | A Navlungo **user** (marketplace scenario) | OAuth2 authorization_code (default) | Creating/updating stores, creating orders, express quotes, converting an order into a shipment, labels, tracking, documents |
| [Shipping API](./shipping-api/README.md) | The **client** itself, server to server    | OAuth2 client_credentials           | Getting rates, creating shipments, labels, documents, tracking, ETGB and void operations, all under a single client account |

Which model suits you is decided together with the Navlungo integration team; the client credentials are defined accordingly.

---

## 2. Sections

### Store API

[Overview and Authorization](./store-api/README.md)</br>
[Token API](./store-api/token.md)</br>
[Express Quote API](./store-api/quote.md)</br>
[Store API](./store-api/store.md)</br>
[Cargo Tracking API](./store-api/cargoTracking.md)</br>
[Shipment API](./store-api/shipment.md)</br>

### Shipping API

[Overview and Authorization](./shipping-api/README.md)</br>
[Token API](./shipping-api/token.md)</br>
[Rates API](./shipping-api/rates.md)</br>
[Shipment API](./shipping-api/shipment.md)</br>
[Shipment Tracking API](./shipping-api/cargoTracking.md)</br>
[Postman Collection](./shipping-api/Navlungo-Shipping-API.postman_collection.json)</br>
