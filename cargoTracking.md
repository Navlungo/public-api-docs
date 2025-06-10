# Gönderi Takip Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile istemci bir kargoya dair takip bilgilerini anlık olarak sorgulayabilir. Yeni bir takip isteğine ait detaylar, Navlungo ilgili sorguyu kaydedip detaylarına ulaştıktan sonra 10-60 dk içerisinde ulaşılabilir olur.

### Versiyon Bilgisi

_Versiyon_ : v1

### URI şeması

_Host_ : api.navlungo.com
_Host(Test)_ : api-qa.navlungo.com
_Schemes_ : HTTPS

### Kabul Edilen Girdi Formatları

- `application/json`

### Üretilen Çıktı Formatları

- `application/json`

### Yetkilendirme

Bu api Oauth2 **client_credentials** akışı ile oluşturulan token'lar ile çağrılabilir. Token alırken scope parametresine **public_cargo_tracking** değeri gönderilmelidir.

### Operasyonlar

- [getShipmentTracking](#getShipmentTracking)<br>

<a name="paths"></a>

## Paths

<a name="getShipmentTracking"></a>

### GET shipments/v1/{reference}/tracking

**Operasyon: getShipmentTracking**

#### Açıklama

Gönderi referans numarası ile kargo takip bilgilerini döndürür.

#### Parametreler

| Tip      | İsim                        | Açıklama                  | Şema |
| -------- | --------------------------- | ------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema                                  |
| --------- | --------------------------------------------------------------- | ------------------------------------- |
| **200**   | Başarılı                                                        | [TrackingResponse](#trackingResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error)                       |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error)                       |
| **404**   | Gönderi bulunamadı.                                             | [Error](#error)                       |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error)                       |

<a name="definitions"></a>

## Tanımlar

<a name="trackingResponse"></a>

### TrackingResponse

| Ad                       | Açıklama                         | Her zaman mevcut | Şema                                |
| ------------------------ | -------------------------------- | ---------------- | ----------------------------------- |
| **trackingNumber**       | Takip numarası                   | Evet             | string                              |
| **originCountry**        | Kargonun çıkış yaptığı ülke kodu | Evet             | string                              |
| **destinationCountry**   | Teslimat ülkesi kodu             | Evet             | string                              |
| **pickupDate**           | Alım tarihi                      | Evet             | string (DateTime)                   |
| **expectedDeliveryDate** | Beklenen teslimat tarihi         | Hayır            | string (DateTime)                   |
| **carrier**              | Taşıyıcı firma kodu              | Evet             | string                              |
| **originLocation**       | Çıkış lokasyonu detayı           | Evet             | string                              |
| **destinationLocation**  | Varış lokasyonu detayı           | Evet             | string                              |
| **checkpoints**          | Sevkiyata dair hareketler        | Evet             | < [Checkpoint](#checkpoint) > Array |

<a name="checkpoint"></a>

### Checkpoint

| Ad                                 | Açıklama               | Her zaman mevcut | Şema              |
| ---------------------------------- | ---------------------- | ---------------- | ----------------- |
| **checkpointTime** <br>_zorunlu_   | Olay zamanı            | Evet             | string (DateTime) |
| **status** <br>_zorunlu_           | Olay durumu            | Evet             | string            |
| **subStatusMessage** <br>_zorunlu_ | Alt durum mesajı       | Evet             | string            |
| **subStatusDescription**           | Alt durum açıklaması   | Hayır            | string            |
| **country**                        | Olay gerçekleşen ülke  | Hayır            | string            |
| **city**                           | Olay gerçekleşen şehir | Hayır            | string            |
| **zip**                            | Posta kodu             | Hayır            | string            |
| **location**                       | Lokasyon detayı        | Hayır            | string            |

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
