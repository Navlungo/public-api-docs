# Gönderi Takip Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile istemci, gönderi referansı ile gönderinin takip özetini ve hareketlerini sorgulayabilir. Takip numarası teklif seçiminden kısa süre sonra atanır; atanmadan önce yapılan sorgular `trackingnotfound.error` döner ve yeniden denenmelidir.

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

Bu api Oauth2 **client_credentials** akışı ile oluşturulan token'lar ile çağrılabilir. Token alırken scope parametresine **shipping_shipments_read** değeri gönderilmelidir. `lastMileTrackingNumber` alanı yalnızca client'ta **can_access_last_mile_tracking** yetkisi varsa döner.

### Operasyonlar

[getTracking](#getTracking)<br>

<a name="paths"></a>

## Paths

<a name="getTracking"></a>

### GET api/shipping/v1/shipments/{reference}/tracking

**Operasyon: getTracking**

#### Açıklama

Gönderi referansı ile takip özetini ve hareket listesini döndürür.

#### Rate Limit

- Client başına 10 saniyede en fazla 20 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                                                                                  | Şema |
| -------- | --------------------------- | ----------------------------------------------------------------------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası ([createRates](./rates.md#createRates) yanıtındaki `reference`) | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                           | Şema                                  |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| **200**   | Başarılı                                                                                                                           | [TrackingResponse](#trackingResponse) |
| **400**   | `trackingnotfound.error` - Takip numarası henüz atanmadı, yeniden deneyin<br> Diğer kodlar takip servisinden olduğu gibi aktarılır | [Error](#error)                       |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_shipments_read` scope'u yok                              | [ProblemDetails](#problemDetails)     |
| **404**   | `record.not.found` - Gönderi bulunamadı veya başka bir hesaba ait                                                                  | [Error](#error)                       |
| **429**   | Rate limit aşıldı                                                                                                                  | [Error](#error)                       |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                        | [ProblemDetails](#problemDetails)     |

#### Örnek Yanıt

```
{
  "reference": 306889,
  "trackingNumber": "NVL000000306889",
  "lastMileTrackingNumber": "1Z9A77910409209526",
  "originCountry": "TUR", "destinationCountry": "USA",
  "pickupDate": "23.09.2026 08:29", "expectedDeliveryDate": null,
  "carrier": "NVL", "originLocation": "İstanbul İstanbul,34710,TR,Turkey TUR", "destinationLocation": "New York New York,10001,US,United States  USA",
  "checkpoints": [
    { "checkpointTime": "23.09.2026 09:22", "status": "In Transit", "subStatusMessage": "In Transit", "subStatusDescription": "Shipment on the way", "country": "TUR", "city": "İstanbul", "zip": "34710", "location": "İstanbul,34710,TR,Turkey" },
    { "checkpointTime": "23.09.2026 08:29", "status": "Info Received", "subStatusMessage": "Info Received", "subStatusDescription": "The carrier received a request from the shipper and is about to pick up the shipment", "country": "TUR", "city": "İstanbul", "zip": "34710", "location": "İstanbul,34710,TR,Turkey" }
  ]
}
```

<a name="definitions"></a>

## Tanımlar

<a name="trackingResponse"></a>

### TrackingResponse

| Ad                         | Açıklama                                                                                         | Her zaman mevcut | Şema                                |
| -------------------------- | ------------------------------------------------------------------------------------------------ | ---------------- | ----------------------------------- |
| **reference**              | Gönderi referansı                                                                                | Evet             | long                                |
| **trackingNumber**         | Navlungo takip numarası (`NVL` + 12 haneye sıfırla doldurulmuş referans)                         | Evet             | string                              |
| **lastMileTrackingNumber** | Taşıyıcı takip numarası. Yalnızca `can_access_last_mile_tracking` yetkisi varsa yanıtta yer alır | Hayır            | string                              |
| **originCountry**          | Çıkış ülkesi (ISO 3166-1 alpha-3, örn. `TUR`)                                                    | Evet             | string                              |
| **destinationCountry**     | Varış ülkesi (ISO 3166-1 alpha-3, örn. `USA`)                                                    | Evet             | string                              |
| **pickupDate**             | Alım tarihi                                                                                      | Evet             | string (DateTime)                   |
| **expectedDeliveryDate**   | Beklenen teslimat tarihi                                                                         | Hayır            | string (DateTime)                   |
| **carrier**                | Takip servisindeki taşıyıcı kodu (örn. `NVL`)                                                    | Evet             | string                              |
| **originLocation**         | Çıkış lokasyonu detayı                                                                           | Evet             | string                              |
| **destinationLocation**    | Varış lokasyonu detayı                                                                           | Evet             | string                              |
| **checkpoints**            | Gönderiye dair hareketler                                                                        | Evet             | < [Checkpoint](#checkpoint) > array |

<a name="checkpoint"></a>

### Checkpoint

| Ad                       | Açıklama                                                                                                                  | Her zaman mevcut | Şema              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ---------------- | ----------------- |
| **checkpointTime**       | Olay zamanı                                                                                                               | Evet             | string (DateTime) |
| **status**               | Olay durumu. Değerler için [Takip Durumları](#trackingStatuses)                                                           | Evet             | string            |
| **subStatusMessage**     | Alt durum mesajı                                                                                                          | Evet             | string            |
| **subStatusDescription** | Alt durum açıklaması                                                                                                      | Hayır            | string            |
| **country**              | Olayın gerçekleştiği ülke (alpha-3)                                                                                       | Hayır            | string            |
| **city**                 | Olayın gerçekleştiği şehir                                                                                                | Hayır            | string            |
| **zip**                  | Posta kodu                                                                                                                | Hayır            | string            |
| **location**             | Lokasyon detayı                                                                                                           | Hayır            | string            |

<a name="trackingStatuses"></a>

### Takip Durumları

`checkpoints` en yeniden en eskiye sıralıdır; gönderinin güncel durumu ilk elemanın `status` değeridir.

`status` aşağıdaki sabit listeden döner. Navlungo kaynaklı hareketler boşluklu (`In Transit`), taşıyıcı (last mile) hareketleri birleşik (`InTransit`) yazımla gelir. Karşılaştırmayı boşlukları yok sayarak ve büyük/küçük harfe duyarsız yapın.

| status                                        | Açıklama                                                                                                            |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `Pending`                                     | Taşıyıcıda henüz takip bilgisi yok                                                                                  |
| `Info Received` / `InfoReceived`              | Gönderi bilgisi alındı, gönderi henüz teslim alınmadı                                                               |
| `In Transit` / `InTransit`                    | Gönderi yolda (aktarma merkezi, gümrük işlemleri dahil)                                                             |
| `Out For Delivery` / `OutForDelivery`         | Gönderi dağıtıma çıktı                                                                                              |
| `Attempt Fail` / `AttemptFail`                | Teslimat denemesi başarısız oldu, taşıyıcı genellikle tekrar dener                                                  |
| `Available For Pickup` / `AvailableForPickup` | Gönderi teslim noktasında, alıcının teslim alması bekleniyor                                                        |
| `Delivered`                                   | Gönderi teslim edildi                                                                                               |
| `Exception`                                   | Teslimatta sorun var (gümrük gecikmesi, hatalı adres, reddedilme, iade vb.); ayrıntı `subStatusMessage` alanındadır |
| `Expired`                                     | Uzun süredir takip bilgisi gelmiyor                                                                                 |

---

## Common Models

<a name="error"></a>

### Error

İş kuralı hatası nesnesi

| Ad                        | Açıklama               | Şema   |
| ------------------------- | ---------------------- | ------ |
| **code** <br>_zorunlu_    | Hata kodu              | string |
| **message** <br>_zorunlu_ | Hata mesajı            | string |
| **data** <br>_opsiyonel_  | Hataya ait ek bilgiler | array  |

<a name="problemDetails"></a>

### ProblemDetails

Doğrulama ve yetkilendirme hatası nesnesi

| Ad                              | Açıklama                                                        | Şema   |
| ------------------------------- | --------------------------------------------------------------- | ------ |
| **type** <br>_zorunlu_          | Hata tipi (path şeklinde örneğin Authentication/InvalidToken)   | string |
| **status** <br>_zorunlu_        | Hataya ait statü kodu                                           | int    |
| **problemCode** <br>_opsiyonel_ | Hata kodu                                                       | string |
| **title** <br>_zorunlu_         | Hata başlığı                                                    | string |
| **detail** <br>_zorunlu_        | Hataya ait detaylı açıklama                                     | string |
| **path** <br>_zorunlu_          | Hatanın oluştuğu url                                            | string |
| **extensions** <br>_opsiyonel_  | Hataya ait detay bilgiler. Hata türüne göre içeriği değişebilir | object |
