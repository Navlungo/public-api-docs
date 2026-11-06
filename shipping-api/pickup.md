# Toplama Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile istemci, [createShipment](./shipment.md#createShipment) ile teklifi seçilmiş gönderileri için adresten toplama (pickup) talebi açabilir, toplama talebini iptal edebilir ve kendi toplama taleplerini listeleyebilir. Bir toplama talebi bir veya birden fazla (en fazla 50) gönderiyi kapsar.

**Kurallar**

- Toplama saat aralığı sabittir: **10:00–15:00**.
- Pazar günü ve geçmiş tarih için talep açılamaz. Aynı gün için Türkiye saati (TRT, UTC+3) 09:00 (06:00 UTC) ve sonrasında talep açılamaz; aynı gün talebi en geç 08:59 TRT'de açılabilir. Reddedilen tarih `pickupdatenotallowed.error` ile döner.
- Gönderiler client'a ait, teklifi seçilmiş, iptal edilmemiş, depoya ulaşmamış ve başka bir toplama talebine bağlı olmayan gönderiler olmalıdır.
- Adres; `countryCode` (ISO 3166-1 alpha-2), `city`, `town` ve `postalCode` adlarıyla Navlungo'nun toplama bölgeleri listesinde aranır (büyük-küçük harf ve baştaki/sondaki boşluk önemsizdir). Bölge bulunamazsa veya hizmet verilmiyorsa `pickuprequests.pickuprequestedinnotavailablearea.error` döner.
- Başka bir hesaba ait gönderi veya toplama talebi için `404` `record.not.found` döner.

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

Bu api Oauth2 **client_credentials** akışı ile oluşturulan token'lar ile çağrılabilir. Her operasyonun scope'u kendi başlığında belirtilmiştir. Toplama scope'ları (**shipping_pickups_write**, **shipping_pickups_read**) self-servis ile oluşturulan anahtarlar dahil tüm client'lara varsayılan olarak tanımlıdır; token isteğinde scope parametresine eklenir. Bkz. [Yetkilendirme](./README.md#authorization).

### Operasyonlar

[createPickup](#createPickup)<br>
[cancelPickup](#cancelPickup)<br>
[getPickups](#getPickups)<br>

<a name="paths"></a>

## Paths

<a name="createPickup"></a>

### POST api/shipping/v1/pickups

**Operasyon: createPickup**

_Scope_ : shipping_pickups_write

#### Açıklama

Bir veya birden fazla gönderi için tek bir toplama talebi açar. Talep `created` statüsünde oluşur ve bağlı gönderilere toplama talebi atanır. Kurallar için [Genel Bakış](#overview).

#### Rate Limit

- Client başına 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                       | Açıklama      | Şema                                      |
| -------- | -------------------------- | ------------- | ----------------------------------------- |
| **Body** | **body** <br>_zorunlu_     | Toplama talebi | [CreatePickupRequest](#createPickupRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | Şema                              |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | [PickupResponse](#pickupResponse) |
| **400**   | `pickupdatenotallowed.error` - Toplama tarihi geçersiz (Pazar, geçmiş tarih veya aynı gün için 09:00 TRT sonrası)<br> `shipmentratenotselected.error` - Gönderi için teklif seçilmemiş<br> `shipmentcancelled.error` - Gönderi iptal edilmiş<br> `shipmentalreadyinwarehouse.error` - Gönderi depoya ulaşmış<br> `pickuprequests.pickuprequestedinnotavailablearea.error` - Adres toplama bölgelerinde bulunamadı veya bölgeye hizmet verilmiyor<br> `pickuprequests.alreadyexistrequestforcurrentshipments.error` - Gönderilerden biri için zaten toplama talebi var<br> `pickuprequests.faileduetocarrierexception.error`, `pickuprequests.faileduetoproposalexception.error`, `pickuprequests.failedduetochargeableweightorshipmentcountexception.error` - Toplama kuralları talebe izin vermiyor | [Error](#error) |
| **400**   | Alan doğrulama hatası (örn. `shipmentReferences` boş veya 50'den fazla, `address`/`contact` alanları eksik, hatalı e-posta, `note` 500 karakterden uzun)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | [ProblemDetails](#problemDetails) |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_pickups_write` scope'u yok                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | [ProblemDetails](#problemDetails) |
| **404**   | `record.not.found` - Gönderilerden biri bulunamadı veya başka bir hesaba ait                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | [Error](#error)                   |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | [Error](#error)                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [ProblemDetails](#problemDetails) |

#### Örnek İstek Body

```
{
  "shipmentReferences": [458213, 458214],
  "pickupDate": "10.11.2026 00:00",
  "address": {
    "countryCode": "TR",
    "city": "Ankara",
    "town": "Cankaya",
    "postalCode": "06690",
    "addressLine": "Ataturk Bulvari No 1"
  },
  "contact": {
    "name": "Ayse Yilmaz",
    "email": "ayse@example.com",
    "phone": { "code": "+90", "number": "5551112233" }
  },
  "note": "Zili calin"
}
```

#### Örnek Yanıt

```
{
  "reference": "1793000000",
  "status": "created",
  "pickupDate": "10.11.2026 00:00",
  "timeWindowStart": "10:00",
  "timeWindowEnd": "15:00",
  "address": { "countryCode": "TR", "city": "Ankara", "town": "Cankaya", "postalCode": "06690", "addressLine": "Ataturk Bulvari No 1" },
  "contact": { "name": "Ayse Yilmaz", "email": "ayse@example.com", "phoneCode": "+90", "phoneNumber": "5551112233" },
  "shipmentReferences": [458213, 458214],
  "note": "Zili calin",
  "createdAt": "05.11.2026 10:15",
  "cancelReason": null
}
```

#### Request Model

<a name="createPickupRequest"></a>

##### CreatePickupRequest

| Ad                                | Açıklama                                                                                                          | Şema                                 |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| **shipmentReferences** <br>_zorunlu_ | Toplanacak gönderi referansları. En az 1, en fazla 50; her biri 0'dan büyük. Tekrarlayan değerler tekilleştirilir | long[]                               |
| **pickupDate** <br>_zorunlu_      | Toplama günü, `dd.MM.yyyy HH:mm` (örn. `10.11.2026 00:00`; saat kısmı yok sayılır, ISO 8601 `2026-11-10T00:00:00` de kabul edilir); saat aralığı sabit 10:00–15:00'tir | string (DateTime)                        |
| **address** <br>_zorunlu_         | Toplama adresi                                                                                                    | [PickupAddress](#pickupAddress)      |
| **contact** <br>_zorunlu_         | Adreste iletişim kurulacak kişi                                                                                   | [PickupContactRequest](#pickupContactRequest) |
| **note** <br>_opsiyonel_          | Kurye için not. En fazla 500 karakter                                                                             | string                               |

<a name="pickupAddress"></a>

##### PickupAddress

| Ad                             | Açıklama                                                                 | Şema   |
| ------------------------------ | ------------------------------------------------------------------------ | ------ |
| **countryCode** <br>_zorunlu_  | Ülke kodu, ISO 3166-1 alpha-2 (örn. `TR`)                                | string |
| **city** <br>_zorunlu_         | Şehir adı (toplama bölgeleri listesindeki adla eşleşmelidir)             | string |
| **town** <br>_zorunlu_         | İlçe adı (toplama bölgeleri listesindeki adla eşleşmelidir)              | string |
| **postalCode** <br>_zorunlu_   | Posta kodu (toplama bölgeleri listesindeki kodla eşleşmelidir)           | string |
| **addressLine** <br>_zorunlu_  | Açık adres. En fazla 500 karakter                                        | string |

<a name="pickupContactRequest"></a>

##### PickupContactRequest

| Ad                        | Açıklama                 | Şema                      |
| ------------------------- | ------------------------ | ------------------------- |
| **name** <br>_zorunlu_    | Ad soyad. En fazla 50 karakter                 | string                    |
| **email** <br>_zorunlu_   | Geçerli e-posta adresi. En fazla 100 karakter   | string                    |
| **phone** <br>_zorunlu_   | Telefon                  | [PickupPhone](#pickupPhone) |

<a name="pickupPhone"></a>

##### PickupPhone

| Ad                       | Açıklama                    | Şema   |
| ------------------------ | --------------------------- | ------ |
| **code** <br>_zorunlu_   | Ülke telefon kodu (`+90`). En fazla 25 karakter   | string |
| **number** <br>_zorunlu_ | Telefon numarası. En fazla 21 karakter            | string |

---

<a name="cancelPickup"></a>

### POST api/shipping/v1/pickups/{reference}/cancel

**Operasyon: cancelPickup**

_Scope_ : shipping_pickups_write

#### Açıklama

Toplama talebini iptal eder. Yalnızca `created` ve `scheduled` statüsündeki talepler iptal edilebilir; iptal sonrası bağlı gönderiler serbest kalır ve yeni bir toplama talebine eklenebilir. Yanıt, `status: "cancelled"` ve dolu `cancelReason` ile güncel toplama nesnesidir.

#### Rate Limit

- Client başına 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                                                                                  | Şema                                      |
| -------- | --------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------------- |
| **Path** | **reference** <br>_zorunlu_ | Toplama talebi referansı ([createPickup](#createPickup) yanıtındaki `reference`, örn. `1793000000`) | string                                    |
| **Body** | **body** <br>_zorunlu_      | İptal gerekçesi                                                                           | [CancelPickupRequest](#cancelPickupRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                  | Şema                              |
| --------- | --------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **200**   | Başarılı                                                                                                  | [PickupResponse](#pickupResponse) |
| **400**   | `pickuprequests.wrongstatus.error` - Talep bu statüde iptal edilemez (toplanmış veya zaten iptal edilmiş) | [Error](#error)                   |
| **400**   | Alan doğrulama hatası (`reason` boş veya 500 karakterden uzun)                                            | [ProblemDetails](#problemDetails) |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_pickups_write` scope'u yok      | [ProblemDetails](#problemDetails) |
| **404**   | `record.not.found` - Toplama talebi bulunamadı veya başka bir hesaba ait                                  | [Error](#error)                   |
| **429**   | Rate limit aşıldı                                                                                         | [Error](#error)                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                               | [ProblemDetails](#problemDetails) |

#### Örnek İstek Body

```
{ "reason": "Planlar degisti" }
```

#### Örnek Yanıt

```
{
  "reference": "1793000000",
  "status": "cancelled",
  "pickupDate": "10.11.2026 00:00",
  "timeWindowStart": "10:00",
  "timeWindowEnd": "15:00",
  "address": { "countryCode": "TR", "city": "Ankara", "town": "Cankaya", "postalCode": "06690", "addressLine": "Ataturk Bulvari No 1" },
  "contact": { "name": "Ayse Yilmaz", "email": "ayse@example.com", "phoneCode": "+90", "phoneNumber": "5551112233" },
  "shipmentReferences": [458213, 458214],
  "note": "Zili calin",
  "createdAt": "05.11.2026 10:15",
  "cancelReason": "Planlar degisti"
}
```

#### Request Model

<a name="cancelPickupRequest"></a>

##### CancelPickupRequest

| Ad                       | Açıklama                           | Şema   |
| ------------------------ | ---------------------------------- | ------ |
| **reason** <br>_zorunlu_ | İptal gerekçesi. En fazla 500 karakter | string |

---

<a name="getPickups"></a>

### GET api/shipping/v1/pickups

**Operasyon: getPickups**

_Scope_ : shipping_pickups_read

#### Açıklama

Client'a ait toplama taleplerini sayfalı listeler; en yeni talep önce gelir.

#### Rate Limit

- Client başına 10 saniyede en fazla 20 istek yapılabilir

#### Parametreler

| Tip       | İsim                              | Açıklama                                                                 | Şema   |
| --------- | --------------------------------- | ------------------------------------------------------------------------ | ------ |
| **Query** | **currentPage** <br>_zorunlu_     | Sayfa numarası. En az 1                                                  | int    |
| **Query** | **pageSize** <br>_zorunlu_        | Sayfa boyutu. 1 ile 100 arası                                            | int    |
| **Query** | **status** <br>_opsiyonel_        | Statü filtresi: `created`, `scheduled`, `picked-up`, `cancelled`         | string |
| **Query** | **shipmentReference** <br>_opsiyonel_ | Gönderi referansı; yalnızca bu gönderiyi kapsayan talepler döner. 0'dan büyük | long   |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                  | Şema                                      |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **200**   | Başarılı                                                                                                                                  | [PickupPagedResponse](#pickupPagedResponse) |
| **400**   | Alan doğrulama hatası (`currentPage` veya `pageSize` eksik ya da geçersiz, `status` geçersiz değer, `shipmentReference` 0 veya altı)      | [ProblemDetails](#problemDetails)         |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_pickups_read` scope'u yok                                       | [ProblemDetails](#problemDetails)         |
| **429**   | Rate limit aşıldı                                                                                                                         | [Error](#error)                           |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                               | [ProblemDetails](#problemDetails)         |

#### Örnek İstek Body

```
GET /api/shipping/v1/pickups?currentPage=1&pageSize=10&status=created&shipmentReference=458213
```

#### Örnek Yanıt

```
{
  "currentPage": 1,
  "pageCount": 1,
  "pageSize": 10,
  "totalRecordCount": 1,
  "results": [
    {
      "reference": "1793000000",
      "status": "created",
      "pickupDate": "10.11.2026 00:00",
      "timeWindowStart": "10:00",
      "timeWindowEnd": "15:00",
      "address": { "countryCode": "TR", "city": "Ankara", "town": "Cankaya", "postalCode": "06690", "addressLine": "Ataturk Bulvari No 1" },
      "contact": { "name": "Ayse Yilmaz", "email": "ayse@example.com", "phoneCode": "+90", "phoneNumber": "5551112233" },
      "shipmentReferences": [458213, 458214],
      "note": "Zili calin",
      "createdAt": "05.11.2026 10:15",
      "cancelReason": null
    }
  ]
}
```

#### Response Model

<a name="pickupPagedResponse"></a>

##### PickupPagedResponse

| Ad                               | Açıklama                 | Şema                                  |
| -------------------------------- | ------------------------ | ------------------------------------- |
| **currentPage** <br>_zorunlu_    | Geçerli sayfa            | int                                   |
| **pageCount** <br>_zorunlu_      | Toplam sayfa sayısı      | int                                   |
| **pageSize** <br>_zorunlu_       | Sayfa boyutu             | int                                   |
| **totalRecordCount** <br>_zorunlu_ | Toplam kayıt sayısı    | int                                   |
| **results** <br>_zorunlu_        | Toplama talepleri        | [PickupResponse](#pickupResponse)[]   |

---

## Common Models

<a name="pickupResponse"></a>

### PickupResponse

| Ad                                  | Açıklama                                                                   | Şema                                      |
| ----------------------------------- | -------------------------------------------------------------------------- | ----------------------------------------- |
| **reference** <br>_zorunlu_         | Toplama talebi referansı                                                   | string                                    |
| **status** <br>_zorunlu_            | Talep statüsü, bkz. [Statüler](#pickupStatuses)                            | string                                    |
| **pickupDate** <br>_zorunlu_        | Toplama günü (`dd.MM.yyyy HH:mm`, saat `00:00`)                            | string (DateTime)                         |
| **timeWindowStart** <br>_zorunlu_   | Toplama aralığı başlangıcı (`HH:mm`), şu an her zaman `10:00`              | string                                    |
| **timeWindowEnd** <br>_zorunlu_     | Toplama aralığı bitişi (`HH:mm`), şu an her zaman `15:00`                  | string                                    |
| **address** <br>_zorunlu_           | Toplama adresi                                                             | [PickupAddress](#pickupAddress)           |
| **contact** <br>_zorunlu_           | İletişim kişisi                                                            | [PickupContactResponse](#pickupContactResponse) |
| **shipmentReferences** <br>_zorunlu_ | Talebe bağlı gönderi referansları                                         | long[]                                    |
| **note** <br>_zorunlu_              | Kurye notu; yoksa `null`                                                   | string                                    |
| **createdAt** <br>_zorunlu_         | Talebin oluşturulma zamanı (UTC)                                           | string (DateTime)                         |
| **cancelReason** <br>_zorunlu_      | İptal gerekçesi; iptal edilmemişse `null`                                  | string                                    |

<a name="pickupContactResponse"></a>

### PickupContactResponse

| Ad                             | Açıklama         | Şema   |
| ------------------------------ | ---------------- | ------ |
| **name** <br>_zorunlu_         | Ad soyad         | string |
| **email** <br>_zorunlu_        | E-posta adresi   | string |
| **phoneCode** <br>_zorunlu_    | Telefon ülke kodu | string |
| **phoneNumber** <br>_zorunlu_  | Telefon numarası | string |

<a name="pickupStatuses"></a>

### Statüler

| Değer       | Açıklama                                                           |
| ----------- | ------------------------------------------------------------------ |
| `created`   | Talep oluşturuldu                                                  |
| `scheduled` | Talep planlandı (rota planlama servisine iletildi)                 |
| `picked-up` | Gönderiler adresten toplandı                                       |
| `cancelled` | Talep iptal edildi (kullanıcı, yönetici veya kurye/operatör tarafından) |

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
