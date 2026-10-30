# Navlungo API — Store and Express Quote (Store API)

> 🇹🇷 [Türkçe](../../store-api/README.md)

> This section describes the Store / Order / Express Quote integration that works on behalf of a user (authorization_code). For the new shipment integration that works server to server (client_credentials), see the [Shipping API](../shipping-api/README.md) section.

<a name="overview"></a>

## Important Notes

1. The client credentials provided to you can only be used in the corresponding environment. You cannot access the production APIs with the client credentials provided for the QA environment.
2. Since the QA and Production environments are isolated, you cannot log in to the QA environment with your production user credentials. If you need a user to work in QA, you can create a new user via the sign-up button and use it in your tests.
3. The QA environment is shut down by default on weekday nights and at weekends, so you may not get a response from the APIs. If you plan your work and let us know in advance, we can keep the environment up.

## 1. Overview

The Navlungo API aims to help Navlungo solution partners build the most suitable experience for their own customers by giving them programmatic access to express quote search and store management processes.

- With the Express Quote API, quotes can be obtained for loads with a chargeable weight of up to 300 kg.
- With the Store API,
  - Creating, updating and deleting stores,
  - Creating orders,
  - Converting an order into a shipment in Navlungo can be done.

---

### 2. Authorization

Access tokens created with the OAuth2 protocol must be sent to all resources accessed through this API. In Navlungo APIs, authorization is done in two different ways depending on the scenario and the client definition. These are;

- Authorization Code (the default authorization type)

#### 2.1. Authorization Code

<a name="auth"></a>

In this flow, before the client is given access and refresh tokens, the user's consent to access the relevant resources is obtained over the web. After the user's approval, with the one-time authorization code sent by Navlungo to the client, the client **can act on the permitted resources on behalf of the user**.

[Video walkthrough of the authorization flow](https://www.loom.com/share/058aec858ee648f4a40390f811ad606b)

![AuthorizationCode](../../store-api/authorization/authorization_code_flow.png?raw=true "AuthorizationCode")

Below you can find sample C# and PHP code blocks that perform the calculation in the example from the video. You can run the C# example in an online environment such as [.Net Fiddle](https://dotnetfiddle.net/).

C#

```
using System;

namespace VerifierHasher;

class Program
{
   static void Main(string[] args)
   {
   	var verifier = "b1f711f0-ee78-44e8-b098-84de6df69ad4"; //enter your own sample verifier here to see the resulting hash
	var verifierBytes = System.Text.Encoding.UTF8.GetBytes(verifier);

        var sha256 = System.Security.Cryptography.SHA256.Create();
	var sha256ComputedVerifierHashBytes = sha256.ComputeHash(verifierBytes);

	string verifierHashedBase64String = Convert.ToBase64String(sha256ComputedVerifierHashBytes);
	Console.WriteLine($"Verifier: {verifier}");
	Console.WriteLine($"Hashed Verifier: {verifierHashedBase64String}");
   }
}
```

PHP

```
$code_verifier = 'b1f711f0-ee78-44e8-b098-84de6df69ad4';
$code_challenge = base64_encode(hash('sha256', utf8_encode($code_verifier) , true));
```

**Authorization Code Request**

The client sends the request for the authorization code, which it will later use to obtain a token in the next stages of the flow, to https://navlungo.com/authorize (https://qa.navlungo.com/authorize for the test environment), passing the values of the following parameters in the query string.

| Name                              | Description                                                                                                                                                                                                                                          |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **client_id** <br>_required_      | The unique client id Navlungo provides to the solution partner                                                                                                                                                                                       |
| **code_challenge** <br>_required_ | Information used to re-verify the client in the later stages of the flow. Before starting the authorization_code flow, the client hashes a random value it generated with SHA256, encodes it with base64 and sends the resulting hash in this field. |

Example authorization code request created by the client;

```
https://navlungo.com/authorize?client_id=fb4cd4c9-5569-43ed-9698-e18de5b77f35&code_challenge=y3oUIHf8JtMcKhLTcwpVBdal/6r2uJYErygLZb4VXog=
```

The authorization code response returned to the client when all checks on Navlungo.com succeed

```
https://callback_url?code=27194b5b-88be-4bb6-bb0c-f6c1b4cde1c2&client_id=fb4cd4c9-5569-43ed-9698-e18de5b77f35&code_challenge=y3oUIHf8JtMcKhLTcwpVBdal/6r2uJYErygLZb4VXog=
```

When Navlungo.com successfully generates the authorization_code;

- It sends the generated authorization code to the URL provided by the client during integration (callback_url) via the code parameter in the query string.
- It also returns all other query string parameters and values from the client's initial request to the callback URL.
- With the authorization code it received, the client can then obtain an access token and refresh token with the help of the [Token API](./token.md).

---

## 3. Order and Shipment Creation Flow

Since the most common scenario with Navlungo is creating shipments through Navlungo.com, we will detail the shipment flow under this heading.

**Definitions**

- **Client**: The organization that uses the Navlungo public API. (Holds a client_id and client_secret.)
- **User**: A Navlungo.com user account. (A real account with a username and password on navlungo.com)

**Points to Note**

- To be able to create a shipment, you first need to create a store and send an order to this store.
- After the order is created, the created order can be associated with a quote and converted into a shipment.
- For store, order and shipment creation operations;
  - The integration does not work application to application. That is, you cannot perform these operations directly for the **client**. A **user** is mandatory.
  - The **Client** obtains the authority to act **on behalf of the user**, as described under the [Authorization Code](#auth) heading, and acts on behalf of that user.

**Scenarios**

_Scenario 1: Marketplace_

The Store API was developed with **marketplace scenarios** in mind, where the **Client**'s users also have a Navlungo account (Navlungo User). That is, the priority of the API is for the **Client**'s users to connect their Navlungo accounts to the **Client**'s platform, and then to automatically send their orders to Navlungo through the platform and convert them into shipments.

_Scenario 2: The Client Using Its Own Navlungo Account_

In some scenarios, the **Client** may want to use **its own Navlungo User**. In this scenario, the Client's users do not need to have a Navlungo account, and processes such as pricing are carried out through the Client's Navlungo account. With the [Authorization Code](#auth) flow, the authorization flow can be run once for the Client's own account in Navlungo and a _refresh_token_ obtained. This refresh_token can then be used repeatedly to obtain an access_token. In this way, a flow similar to integrations done with an api-key effectively emerges.

**However, although this flow resembles an integration done with an api key, it is not the same. Since this scenario does not fit the main intended use of the API, it should rather be considered an alternative solution.**

In both of the scenarios above, the shipment creation flow works as follows;

![Shipment Flow](../../store-api/shipment_flow.png?raw=true "AuthorizationCode")

1.1. The client creates a store with [createStore](./store.md#createStore). When Navlungo creates the store successfully, it returns the store id.

1.2. The client creates an order in one of its stores with [createOrderQuote](./store.md#createOrderQuote) and lists the quotes.

1.3. With [shipStoreOrder](./store.md#shipStoreOrder), the client can convert its quote into a shipment in the "awaiting warehouse arrival" status by sending, for an order reference, the desired quote reference and its searchId (returned in the [createOrderQuote](./store.md#createOrderQuote) response).

---

## 4. Operations

[Token API](./token.md)</br>
[Express Quote API](./quote.md)</br>
[Store API](./store.md)</br>
[Cargo Tracking API](./cargoTracking.md)</br>
[Shipment API](./shipment.md)</br>
