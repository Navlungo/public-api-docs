# Gönderi Takip Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile istemci bir kargoya dair takip bilgilerini anlık olarak sorgulayabilir. Yeni bir takip isteğine ait detaylar Navlungo ilgili sorguyu kaydedip detaylarına ulaştıktan sonra 10-60 dk içerisinde ulaşılabilir olur.

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

- [getCargoTrackingInfo](#getCargoTrackingInfo)<br>

<a name="paths"></a>

## Paths

<a name="getCargoTrackingInfo"></a>

### GET tracking/v1/cargo

**Operasyon: getCargoTrackingInfo**

#### Açıklama

Express taşımalar için kargo takip bilgilerini döndürür.

#### Parametreler

| Tip             | İsim                                 | Açıklama                    | Şema                         |
| --------------- | ------------------------------------ | --------------------------- | ---------------------------- |
| **Querystring** | **courier** <br>_zorunlu_            | Taşıyıcı firma kodu         | < [Courier](#courier) > Enum |
| **Querystring** | **trackingNumber** <br>_zorunlu_     | Takip numarası              | -                            |
| **Querystring** | **originCountry** <br>_zorunlu_      | Kargonun çıkış yaptığı ülke | -                            |
| **Querystring** | **destinationCountry** <br>_zorunlu_ | Teslimat ülkesi             | -                            |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema                                  |
| --------- | --------------------------------------------------------------- | ------------------------------------- |
| **200**   | Başarılı                                                        | [TrackingResponse](#trackingResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error)                       |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error)                       |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error)                       |

<a name="definitions"></a>

####Tanımlar

<a name="courier"></a>

| Kod          | Taşıyıcı |
| ------------ | -------- |
| **dhl**      | DHL      |
| **fedex**    | FedEx    |
| **tnt**      | TNT      |
| **ups**      | UPS      |
| **dpd**      | DPD      |
| **gls**      | GLS      |
| **aramex**   | ARAMEX   |
| **hermes**   | HERMES   |
| **hellmann** | HELLMAN  |

<a name="trackingResponse"></a>

**TrackingResponse**

| Ad                      | Açıklama                      | Her zaman mevcut | Şema                                       |
| ----------------------- | ----------------------------- | ---------------- | ------------------------------------------ |
| **trackingNumber**      | Takip numarası                | Evet             | string                                     |
| **courier** <br>        | Taşıyıcı firma kodu           | Evet             | string                                     |
| **courierTrackingLink** | Taşıyıcı firmanın takip linki | Hayır            | string                                     |
| **status** <br>         | Sevkiyatın durumu             | Evet             | < [TrackingStatus](#trackingStatus) > Enum |
| **signedBy**            | Varsa teslim alan kişi        | Hayır            | string                                     |
| **shipmentType**        | Taşıyıcı firmanın servis türü | Hayır            | string                                     |
| **checkpoints**         | Sevkiyata dair hareketler     | Hayır            | < [Checkpoint](#checkpoint) > Array        |

<a name="checkpoint"></a>

**Checkpoint**

| Ad                               | Açıklama                          | Şema                                       |
| -------------------------------- | --------------------------------- | ------------------------------------------ |
| **checkpointTime** <br>_zorunlu_ | Olay zamanı                       | string                                     |
| **status** <br>_zorunlu_         | Olay sırasında sevkiyatın statüsü | < [TrackingStatus](#trackingStatus) > Enum |
| **message**                      | Olaya dair mesaj                  | string                                     |

<a name="trackingStatus"></a>

**TrackingStatus**

| Ad                     | Açıklama                                                                                                    |
| ---------------------- | ----------------------------------------------------------------------------------------------------------- |
| **InfoReceived**       | Carrier has received request from shipper and is about to pick up the shipment.                             |
| **InTransit**          | Carrier has accepted or picked up shipment from shipper. The shipment is on the way.                        |
| **OutForDelivery**     | Carrier is about to deliver the shipment , or it is ready to pickup.                                        |
| **AttemptFail**        | Carrier attempted to deliver but failed, and usually leaves a notice and will try to deliver again.         |
| **Delivered**          | The shipment was delivered successfully.                                                                    |
| **AvailableForPickup** | The package arrived at a pickup point near you and is available for pickup.                                 |
| **Exception**          | Custom hold, undelivered, returned shipment to sender or any shipping exceptions.                           |
| **Expired**            | Shipment has no tracking information for 30 days since added.                                               |
| **Pending**            | New shipments added that are pending to track, or new shipments without tracking information available yet. |
