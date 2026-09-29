# Token Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile Shipping Api çağrılarında kullanılacak access token, OAuth2 **client_credentials** akışı ile üretilir. Token Navlungo kimlik sunucusundan alınır; Shipping Api endpointlerinden farklı bir host kullanılır.

### Versiyon Bilgisi

_Versiyon_ : v1

### URI şeması

_Host_ : identity.navlungo.com
_Host(Test)_ : identity-qa.navlungo.com
_Schemes_ : HTTPS

### Kabul Edilen Girdi Formatları

- `application/x-www-form-urlencoded`

### Üretilen Çıktı Formatları

- `application/json`

### Operasyonlar

[token](#token)<br>

<a name="paths"></a>

## Paths

<a name="token"></a>

### POST connect/token

**Operasyon: token**

#### Açıklama

client_credentials akışı ile access token üretir. Token 3600 saniye geçerlidir; refresh token üretilmez, süre dolduğunda aynı istekle yeni token alınır. Client'a sonradan eklenen yetkiler (claim'ler) yalnızca yeni alınan token'larda yer alır.

#### Parametreler

| Tip        | İsim                            | Açıklama                                                                                                                                                                                             |
| ---------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **string** | **grant_type** <br>_zorunlu_    | `client_credentials`                                                                                                                                                                                 |
| **string** | **client_id** <br>_zorunlu_     | İstemciye Navlungo tarafından verilen id                                                                                                                                                             |
| **string** | **client_secret** <br>_zorunlu_ | İstemciye Navlungo tarafından verilen şifre                                                                                                                                                          |
| **string** | **scope** <br>_zorunlu_         | Kullanılacak endpointlerin scope'ları, boşlukla ayrılmış. Bkz. [Scope'lar](./README.md#scopes). Örnek: `shipping_rates_write shipping_shipments_write shipping_shipments_read shipping_labels_write` |

#### Örnek İstek

```
POST https://identity-qa.navlungo.com/connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=8f14e45fceea167a5a36dedd4bea2543&client_secret=<secret>&scope=shipping_rates_write shipping_shipments_write shipping_shipments_read shipping_labels_write
```

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                      | Şema                            |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                      | [TokenResponse](#tokenResponse) |
| **500**   | Kimlik doğrulama başarısız: `client_id`/`client_secret` hatalı (`invalid_client`) veya istenen scope client'a tanımlı değil (`invalid_scope`). Hata nedeni `extensions.exceptionDetails.error` alanında döner | [TokenError](#tokenError)       |

#### Örnek Yanıt

```
{ "access_token": "eyJhbGciOi...", "expires_in": 3600, "token_type": "Bearer", "scope": "shipping_rates_write shipping_shipments_write shipping_shipments_read shipping_labels_write" }
```

<a name="tokenResponse"></a>

### TokenResponse

| Ad                             | Açıklama                                  | Şema   |
| ------------------------------ | ----------------------------------------- | ------ |
| **access_token** <br>_zorunlu_ | Access Token                              | string |
| **token_type** <br>_zorunlu_   | Token tipi, her zaman `Bearer`            | string |
| **expires_in** <br>_zorunlu_   | Tokenin geçerli olduğu süre sn. cinsinden | int    |
| **scope** <br>_zorunlu_        | Token'a verilen scope'lar                 | string |

<a name="tokenError"></a>

### TokenError

| Ad                            | Açıklama                                                                                  | Şema   |
| ----------------------------- | ----------------------------------------------------------------------------------------- | ------ |
| **type** <br>_zorunlu_        | `problems/unexpectedIdsvr`                                                                | string |
| **status** <br>_zorunlu_      | `500`                                                                                     | int    |
| **problemCode** <br>_zorunlu_ | Hata kodu                                                                                 | int    |
| **title** <br>_zorunlu_       | Hata başlığı                                                                              | string |
| **detail** <br>_zorunlu_      | Hata açıklaması                                                                           | string |
| **extensions** <br>_zorunlu_  | `exceptionDetails.error` alanında OAuth2 hata kodu: `invalid_client` veya `invalid_scope` | object |
