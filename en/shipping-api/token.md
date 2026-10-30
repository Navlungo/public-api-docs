# Token API

<a name="overview"></a>

## Overview

This API issues the access token to be used in Shipping API calls through the OAuth2 **client_credentials** flow. The token is obtained from the Navlungo identity server; a different host from the Shipping API endpoints is used.

### Version Information

_Version_ : v1

### URI scheme

_Host_ : identity.navlungo.com
_Host(Test)_ : identity-qa.navlungo.com
_Schemes_ : HTTPS

### Accepted Input Formats

- `application/x-www-form-urlencoded`

### Produced Output Formats

- `application/json`

### Operations

[token](#token)<br>

<a name="paths"></a>

## Paths

<a name="token"></a>

### POST connect/token

**Operation: token**

#### Description

Issues an access token through the client_credentials flow. The token is valid for 3600 seconds; no refresh token is issued, and when it expires a new token is obtained with the same request. Permissions (claims) added to the client later are only included in newly obtained tokens.

#### Parameters

| Type       | Name                             | Description                                                                                                                                                                                           |
| ---------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **string** | **grant_type** <br>_required_    | `client_credentials`                                                                                                                                                                                  |
| **string** | **client_id** <br>_required_     | The id given to the client by Navlungo                                                                                                                                                                |
| **string** | **client_secret** <br>_required_ | The secret given to the client by Navlungo                                                                                                                                                            |
| **string** | **scope** <br>_required_         | The scopes of the endpoints to be used, separated by spaces. See [Scopes](./README.md#scopes). Example: `shipping_rates_write shipping_shipments_write shipping_shipments_read shipping_labels_write` |

#### Example Request

```
POST https://identity-qa.navlungo.com/connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=8f14e45fceea167a5a36dedd4bea2543&client_secret=<secret>&scope=shipping_rates_write shipping_shipments_write shipping_shipments_read shipping_labels_write
```

#### Responses

| HTTP Code | Description                                                                                                                                                                                                                    | Schema                          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------- |
| **200**   | Successful                                                                                                                                                                                                                     | [TokenResponse](#tokenResponse) |
| **500**   | Authentication failed: `client_id`/`client_secret` is wrong (`invalid_client`) or the requested scope is not defined for the client (`invalid_scope`). The reason is returned in the `extensions.exceptionDetails.error` field | [TokenError](#tokenError)       |

#### Example Response

```
{ "access_token": "eyJhbGciOi...", "expires_in": 3600, "token_type": "Bearer", "scope": "shipping_rates_write shipping_shipments_write shipping_shipments_read shipping_labels_write" }
```

<a name="tokenResponse"></a>

### TokenResponse

| Name                            | Description                                   | Schema |
| ------------------------------- | --------------------------------------------- | ------ |
| **access_token** <br>_required_ | Access Token                                  | string |
| **token_type** <br>_required_   | Token type, always `Bearer`                   | string |
| **expires_in** <br>_required_   | The period the token is valid for, in seconds | int    |
| **scope** <br>_required_        | The scopes granted to the token               | string |

<a name="tokenError"></a>

### TokenError

| Name                           | Description                                                                                      | Schema |
| ------------------------------ | ------------------------------------------------------------------------------------------------ | ------ |
| **type** <br>_required_        | `problems/unexpectedIdsvr`                                                                       | string |
| **status** <br>_required_      | `500`                                                                                            | int    |
| **problemCode** <br>_required_ | Error code                                                                                       | int    |
| **title** <br>_required_       | Error title                                                                                      | string |
| **detail** <br>_required_      | Error description                                                                                | string |
| **extensions** <br>_required_  | The OAuth2 error code in the `exceptionDetails.error` field: `invalid_client` or `invalid_scope` | object |
