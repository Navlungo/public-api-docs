# Token API

<a name="overview"></a>

## Overview

This API supports the OAuth2 authorization_code and refresh_token flows.

### Version Information

_Version_ : v1

### URI scheme

_Host_ : api.navlungo.com  
_Host(Test)_ : api-qa.navlungo.com
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

### POST v1/oauth/token

**Operation: token**

#### Description

Issues a token as a result of the OAuth refresh_token or authorization_code flows.

#### Parameters

| Type       | Name                             | Description                                                                                                                                                                                                         |
| ---------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **string** | **client_id** <br>_required_     | The id given to the client by Navlungo                                                                                                                                                                              |
| **string** | **client_secret** <br>_required_ | The secret given to the client by Navlungo                                                                                                                                                                          |
| **string** | **grant_type** <br>_required_    | Flow type. authorization_code, client_credentials or refresh_token                                                                                                                                                  |
| **string** | **scope** <br>_optional_         | The API scopes to be used. In the authorization_code flow, at least the values openid offline_access must be sent. For the client_credentials flow, the documented scopes of the endpoints to be used must be sent. |
| **string** | **code** <br>_optional_          | Authorization code. Required for the authorization_code flow.                                                                                                                                                       |
| **string** | **code_verifier** <br>_optional_ | The unhashed form of the SHA256-hashed code_challenge value sent in the first step of the authorization_code flow. Required for the authorization_code flow.                                                        |
| **string** | **refresh_token** <br>_optional_ | Required if the flow type is refresh_token.                                                                                                                                                                         |

#### Responses

| HTTP Code | Description                                                    | Schema                           |
| --------- | -------------------------------------------------------------- | -------------------------------- |
| **200**   | Successful                                                     | [TokenResponse](#token-response) |
| **400**   | A request validation error occurred or the request is invalid. | [Error](#error)                  |
| **401**   | Authorization error.                                           | [Error](#error)                  |
| **500**   | An unexpected error occurred during the request.               | [Error](#error)                  |

<a name="token-response"></a>

### TokenResponse

| Name                             | Description                                                                       | Schema |
| -------------------------------- | --------------------------------------------------------------------------------- | ------ |
| **access_token** <br>_required_  | Access Token                                                                      | string |
| **token_type** <br>_required_    | Token type                                                                        | int    |
| **expires_in** <br>_required_    | The period the token is valid for, in seconds                                     | string |
| **refresh_token** <br>_optional_ | The issued refresh token (always sent empty in the client_credentials flow.)      | string |
| **id_token** <br>optional        | The issued Id Token. May be empty depending on the client's settings in Navlungo. | string |

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
