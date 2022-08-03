# Token Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile oauth2 authorization_code ve refresh_token akışları desteklenmektedir.

### Versiyon Bilgisi

_Versiyon_ : v1

### URI şeması

_Host_ : api.navlungo.com  
_Host(Test)_ : api-qa.navlungo.com
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

### POST v1/oauth/token

**Operasyon: token**

#### Açıklama

Oauth refresh_token veya authorization_code akışları sonucunda token üretir.

#### Parametreler

| Tip        | İsim                              | Açıklama                                                                                                                                                                                                             |
| ---------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **string** | **client_id** <br>_zorunlu_       | İstemciye Navlungo tarafından verilen id                                                                                                                                                                             |
| **string** | **client_secret** <br>_zorunlu_   | İstemciye Navlungo tarafından verilen şifre                                                                                                                                                                          |
| **string** | **grant_type** <br>_zorunlu_      | Akış tipi. authorization_code, client_credentials veya refresh_token                                                                                                                                                 |
| **string** | **scope** <br>_opsiyonel_         | Kullanılacak api kapsamları. authorization_code akışında minimumda openid offline_access değerleri gönderilmeli. client_credentials akışı için kullanılacak endpointlerin dökümante edilmiş scope'ları gönderilmeli. |
| **string** | **code** <br>_opsiyonel_          | Authorization code. authorization_code akışı için zorunlu.                                                                                                                                                           |
| **string** | **code_verifier** <br>_opsiyonel_ | authorization_code akışında ilk adımda gönderilen SHA256 ile hashlenmiş code_challenge verisinin hashlenmemiş hali. authorization_code akışı için zorunlu.                                                           |
| **string** | **refresh_token** <br>_opsiyonel_ | Akış tipi refresh_token ise zorunlu.                                                                                                                                                                                 |

#### Yanıtlar

| HTTP Kodu | Açıklama                                              | Şema                             |
| --------- | ----------------------------------------------------- | -------------------------------- |
| **200**   | Başarılı                                              | [TokenResponse](#token-response) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz. | [Error](#error)                  |
| **401**   | Yetkilendirme hatası.                                 | [Error](#error)                  |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.          | [Error](#error)                  |

<a name="token-response"></a>

### TokenResponse

| Ad                                | Açıklama                                                                         | Şema   |
| --------------------------------- | -------------------------------------------------------------------------------- | ------ |
| **access_token** <br>_zorunlu_    | Access Token                                                                     | string |
| **token_type** <br>_zorunlu_      | Token tipi                                                                       | int    |
| **expires_in** <br>_zorunlu_      | Tokenin geçerli olduğu süre sn. cinsinden                                        | string |
| **refresh_token** <br>_opsiyonel_ | Üretilen refresh token ( client_credentials akışında her zaman boş gönderilir. ) | string |
| **id_token** <br>opsiyonel        | Üretilen Id Token. İstemcinin Navlungo'daki ayarlarına göre boş olabilir.        | string |

<a name="error"></a>

### Error

Genel hata nesnesi

| Ad                              | Açıklama                                                        | Şema   |
| ------------------------------- | --------------------------------------------------------------- | ------ |
| **type** <br>_zorunlu_          | Hata tipi (path şeklinde örneğin Authentication/InvalidToken)   | string |
| **status** <br>_zorunlu_        | Hataya ait statü kodu                                           | int    |
| **problemCode** <br>_opsiyonel_ | Hata kodu                                                       | string |
| **title** <br>_zorunlu_         | Hata başlığı                                                    | string |
| **detail** <br>_zorunlu_        | Hataya ait detaylı açıklama                                     | string |
| **path** <br>_zorunlu_          | Hatanın oluştuğu url                                            | string |
| **extensions** <br>_opsiyonel_  | Hataya ait detay bilgiler. Hata türüne göre içeriği değişebilir | object |
