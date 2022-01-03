# Teklif Api'leri

<a name="overview"></a>

## Genel Bakış

Express aramalar api'si kullanılarak faturalanabilir ağırlığı 300 kg'a kadar olan yükler için birden fazla teklif alınabilir. Bu teklif seçme api'si kullanılarak sistemde bulunun bir teklif ile tekliflerden biri seçilerek ilişkilendirilebilir.

### Versiyon Bilgisi

_Versiyon_ : v1

### URI şeması

_Host_ : api.navlungo.com  
_Schemes_ : HTTPS

### Kabul Edilen Girdi Formatları

- `application/json`

### Üretilen Çıktı Formatları

- `application/json`

### Operasyonlar

[selectQuote](#selectQuote)<br>

<a name="paths"></a>

## Paths

<a name="selectQuote"></a>

### POST /v1/quotes/{id}/select

**Operasyon: selectQuote**

#### Açıklama

Query parametre olarak verilen teklif seçilerek order id ile ilişkilendirilir.

#### Parametreler

| Tip       | İsim                   | Açıklama                        | Şema                                      |
| --------- | ---------------------- | ------------------------------- | ----------------------------------------- |
| **Query** | **id** <br>_zorunlu_   | Teklifin tekil id'si            | string                                    |
| **Body**  | **body** <br>_zorunlu_ | Teklif seçmek için gerekli şema | [selectQuoteRequest](#selectQuoteRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema            |
| --------- | -------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                       |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.          | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                   | [Error](#error) |

<a name="definitions"></a>

## Tanımlar

<a name="selectQuoteRequest"></a>

### selectQuoteRequest

| Ad                             | Açıklama                                       | Şema   |
| ------------------------------ | ---------------------------------------------- | ------ |
| **orderId** <br>_zorunlu_      | Siparişin tekil id'si                          | string |
| **shipmentType** <br>_zorunlu_ | Sevkiyatın tipi (Gift,Sample,Sale,MicroExport) | string |

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
