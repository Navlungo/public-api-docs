# Gönderi Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile [createRates](./rates.md#createRates) sonucunda açılan gönderi referansı üzerinde teklif seçilir, gönderi detayı alınır, taşıyıcı etiketi ve belgeler yönetilir, ETGB belgesi indirilir ve gönderi iptal edilir.

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

Bu api Oauth2 **client_credentials** akışı ile oluşturulan token'lar ile çağrılabilir. Her operasyonun scope'u kendi başlığında belirtilmiştir; etiket oluşturma ayrıca **can_create_labels** yetkisi gerektirir. Bkz. [Yetkilendirme](./README.md#authorization).

### Operasyonlar

[createShipment](#createShipment)<br>
[getShipment](#getShipment)<br>
[createLabel](#createLabel)<br>
[uploadDocuments](#uploadDocuments)<br>
[getEtgbDownloadUrl](#getEtgbDownloadUrl)<br>
[voidShipment](#voidShipment)<br>

<a name="paths"></a>

## Paths

<a name="createShipment"></a>

### POST api/shipping/v1/shipments

**Operasyon: createShipment**

_Scope_ : shipping_shipments_write

#### Açıklama

[createRates](./rates.md#createRates) ile açılan gönderi için teklifi seçer. Gönderi "depoya ulaşması bekleniyor" (`awaiting-warehouse-arrival`) statüsüne geçer ve Navlungo takip numarası atanır. Bu adımda ödeme alınmaz. Teklif seçilirken yeniden doğrulanır; geçersizse yeni bir [createRates](./rates.md#createRates) çağrısı gerekir.

#### Rate Limit

- Client başına 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                   | Açıklama                        | Şema                                            |
| -------- | ---------------------- | ------------------------------- | ----------------------------------------------- |
| **Body** | **body** <br>_zorunlu_ | Teklif seçmek için gerekli şema | [CreateShipmentRequest](#createShipmentRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Şema                                                |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | [ShipmentCreatedResponse](#shipmentCreatedResponse) |
| **400**   | İstek doğrulamasında hata oluştu ([ProblemDetails](#problemDetails)) veya iş kuralı hatası ([Error](#error)):<br> `shipment.shipmentcannotbeupdatedafterithasbeencanceled.error` - Gönderi iptal edilmiş<br> `ratealreadyselected.error`, `shipment.shipmentisnotintheinitstate.error` - Teklif zaten seçilmiş (ikincisi eşzamanlı istekte)<br> `shipment.ddpisrequiredbutnotselected.error`, `shipment.customstaxdutyisrequiredbutnotselected.error` - Zorunlu ek hizmet `selectedServices` içinde yok<br> `rateselectionfailed.error` - `searchId`/`rateId` uyuşmuyor veya teklif artık geçerli değil | [Error](#error)                                     |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_shipments_write` scope'u yok                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | [ProblemDetails](#problemDetails)                   |
| **404**   | `record.not.found` - Gönderi bulunamadı veya başka bir hesaba ait                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [Error](#error)                                     |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [Error](#error)                                     |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | [ProblemDetails](#problemDetails)                   |

#### Örnek İstek Body

```
{ "reference": 458213, "searchId": "3f0c9a4e-7b2d-4c7e-9a1b-2e5d6f7a8b9c", "rateId": "9b7e2c11-4d3a-4f6e-8c2a-1d2e3f4a5b6c", "selectedServices": ["customs-tax-duty", "insurance"] }
```

#### Örnek Yanıt

```
{
  "shipmentId": "b7e6c1a2-9f3d-4e2b-8a1c-7d5f6e8a9b0c",
  "reference": 458213,
  "trackingNumber": "NVL000000458213",
  "trackingUrl": "https://navlungo.com/track?trackingNumber=NVL000000458213",
  "packageLabels": ["458213-1", "458213-2"],
  "totalPrice": 62.1, "currency": "USD", "chargeableWeight": 4.0
}
```

#### Request Model

<a name="createShipmentRequest"></a>

##### CreateShipmentRequest

| Ad                                   | Açıklama                                                                                                                                       | Şema             |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| **reference** <br>_zorunlu_          | Gönderi referansı ([createRates](./rates.md#createRates) yanıtı)                                                                               | long             |
| **searchId** <br>_zorunlu_           | Teklif aramasının kimliği ([createRates](./rates.md#createRates) yanıtı)                                                                       | string (Guid)    |
| **rateId** <br>_zorunlu_             | Seçilen teklifin kimliği                                                                                                                       | string (Guid)    |
| **selectedServices** <br>_opsiyonel_ | Seçilen ek hizmet kodları: `ddp`, `insurance`, `customs-tax-duty`, `atr`; tekrar edemez. Teklifte `isRequired: true` olan hizmetler zorunludur | < string > array |

#### Response Model

<a name="shipmentCreatedResponse"></a>

##### ShipmentCreatedResponse

| Ad                                 | Açıklama                                                                 | Şema             |
| ---------------------------------- | ------------------------------------------------------------------------ | ---------------- |
| **shipmentId** <br>_zorunlu_       | Gönderinin tekil id'si                                                   | string (Guid)    |
| **reference** <br>_zorunlu_        | Gönderi referansı                                                        | long             |
| **trackingNumber** <br>_zorunlu_   | Navlungo takip numarası (`NVL` + 12 haneye sıfırla doldurulmuş referans) | string           |
| **trackingUrl** <br>_zorunlu_      | Navlungo takip sayfası adresi                                            | string           |
| **packageLabels** <br>_zorunlu_    | Paket etiket numaraları                                                  | < string > array |
| **totalPrice** <br>_zorunlu_       | Seçilen teklif ve ek hizmetlerin toplam tutarı                           | decimal          |
| **currency** <br>_zorunlu_         | Para birimi                                                              | string           |
| **chargeableWeight** <br>_zorunlu_ | Faturalanabilir ağırlık (kg)                                             | decimal          |

---

<a name="getShipment"></a>

### GET api/shipping/v1/shipments?reference={reference}

**Operasyon: getShipment**

_Scope_ : shipping_shipments_read

#### Açıklama

Gönderi detayını döndürür. Teklif seçilmemişse `trackingNumber`, `carrier`, `serviceType` ve `price` alanları `null` gelir.

#### Rate Limit

- Client başına 10 saniyede en fazla 20 istek yapılabilir

#### Parametreler

| Tip       | İsim                        | Açıklama                  | Şema |
| --------- | --------------------------- | ------------------------- | ---- |
| **Query** | **reference** <br>_zorunlu_ | Gönderi referans numarası | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                              | Şema                                              |
| --------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| **200**   | Başarılı                                                                                              | [ShipmentDetailResponse](#shipmentDetailResponse) |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_shipments_read` scope'u yok | [ProblemDetails](#problemDetails)                 |
| **404**   | `record.not.found` - Gönderi bulunamadı veya başka bir hesaba ait                                     | [Error](#error)                                   |
| **429**   | Rate limit aşıldı                                                                                     | [Error](#error)                                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                           | [ProblemDetails](#problemDetails)                 |

#### Örnek Yanıt

```
{
  "shipmentId": "b7e6c1a2-9f3d-4e2b-8a1c-7d5f6e8a9b0c",
  "reference": 458213,
  "status": "awaiting-warehouse-arrival",
  "trackingNumber": "NVL000000458213",
  "carrier": "United Parcel Service",
  "serviceType": "express",
  "price": { "amount": 62.1, "currency": "USD" },
  "chargeableWeight": 4.0,
  "warehouseArrivalDate": null,
  "dispatchDate": null,
  "iossNumber": null,
  "senderAddress": { "contactName": "Ayşe Yılmaz", "companyName": "Yılmaz Tekstil A.Ş.", "countryCode": "TR", "stateCode": null, "postalCode": "34710", "city": "İstanbul", "town": "Kadıköy", "firstLine": "Caferağa Mah. Moda Cad. No:12", "secondLine": null, "thirdLine": null, "email": "ayse@yilmaztekstil.com", "phoneCode": "+90", "phoneNumber": "5321234567" },
  "receiverAddress": { "contactName": "John Carter", "companyName": null, "countryCode": "US", "stateCode": "NY", "postalCode": "10001", "city": "New York", "town": null, "firstLine": "350 5th Ave Suite 4400", "secondLine": null, "thirdLine": null, "email": "john.carter@example.com", "phoneCode": "+1", "phoneNumber": "2125550147" },
  "products": [ { "description": "Cotton t-shirt, short sleeve", "hsCode": "6109.10.00.12", "originCountryCode": "TR", "unitPrice": 12.5, "currency": "USD", "quantity": 10 } ],
  "packages": [ { "type": "box", "weight": 2.0, "length": 40.64, "height": 20.32, "width": 30.48 }, { "type": "box", "weight": 2.0, "length": 40.64, "height": 20.32, "width": 30.48 } ]
}
```

#### Response Model

<a name="shipmentDetailResponse"></a>

##### ShipmentDetailResponse

| Ad                                       | Açıklama                                                                                                                          | Şema                                      |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **shipmentId** <br>_zorunlu_             | Gönderinin tekil id'si                                                                                                            | string (Guid)                             |
| **reference** <br>_zorunlu_              | Gönderi referansı                                                                                                                 | long                                      |
| **status** <br>_zorunlu_                 | Gönderi durumu: `rate-not-selected`, `awaiting-warehouse-arrival`, `in-warehouse`, `dispatched`, `cancelled`                      | string                                    |
| **trackingNumber** <br>_opsiyonel_       | Navlungo takip numarası. Teklif seçilmemişse `null`                                                                               | string                                    |
| **carrier** <br>_opsiyonel_              | Taşıyıcı adı. Teklif seçilmemişse `null`                                                                                          | string                                    |
| **serviceType** <br>_opsiyonel_          | Servis türü. Teklif seçilmemişse `null`                                                                                           | string                                    |
| **price** <br>_opsiyonel_                | Toplam tutar. Teklif seçilmemişse `null`                                                                                          | [Price](#price)                           |
| **chargeableWeight** <br>_zorunlu_       | Faturalanabilir ağırlık (kg)                                                                                                      | decimal                                   |
| **warehouseArrivalDate** <br>_opsiyonel_ | Depoya ulaşma tarihi                                                                                                              | string (DateTime)                         |
| **dispatchDate** <br>_opsiyonel_         | Çıkış tarihi                                                                                                                      | string (DateTime)                         |
| **iossNumber** <br>_opsiyonel_           | [createRates](./rates.md#createRates) isteğinde verilen ve saklanan IOSS numarası; AB dışı hedef veya 150 EUR üstü değerde `null` | string                                    |
| **senderAddress** <br>_zorunlu_          | Gönderici çıkış adresi                                                                                                            | [Address](#address)                       |
| **receiverAddress** <br>_zorunlu_        | Alıcı adresi                                                                                                                      | [Address](#address)                       |
| **products** <br>_zorunlu_               | Ürünler                                                                                                                           | < [ProductDetail](#productDetail) > array |
| **packages** <br>_zorunlu_               | Paketler. Her satır tek bir paketi temsil eder; ölçüler her zaman kg/cm                                                           | < [PackageDetail](#packageDetail) > array |

<a name="price"></a>

##### Price

| Ad                         | Açıklama    | Şema    |
| -------------------------- | ----------- | ------- |
| **amount** <br>_zorunlu_   | Tutar       | decimal |
| **currency** <br>_zorunlu_ | Para birimi | string  |

<a name="address"></a>

##### Address

| Ad                              | Açıklama            | Şema   |
| ------------------------------- | ------------------- | ------ |
| **contactName** <br>_zorunlu_   | İlgili kişi         | string |
| **companyName** <br>_opsiyonel_ | Şirket adı          | string |
| **countryCode** <br>_zorunlu_   | Ülke kodu (ISO2)    | string |
| **stateCode** <br>_opsiyonel_   | Eyalet kodu         | string |
| **postalCode** <br>_opsiyonel_  | Posta kodu          | string |
| **city** <br>_zorunlu_          | Şehir               | string |
| **town** <br>_opsiyonel_        | İlçe                | string |
| **firstLine** <br>_zorunlu_     | Adres satırı        | string |
| **secondLine** <br>_opsiyonel_  | İkinci adres satırı | string |
| **thirdLine** <br>_opsiyonel_   | Üçüncü adres satırı | string |
| **email** <br>_opsiyonel_       | E-posta             | string |
| **phoneCode** <br>_opsiyonel_   | Telefon ülke kodu   | string |
| **phoneNumber** <br>_opsiyonel_ | Telefon numarası    | string |

<a name="productDetail"></a>

##### ProductDetail

| Ad                                  | Açıklama         | Şema    |
| ----------------------------------- | ---------------- | ------- |
| **description** <br>_opsiyonel_     | Ürün açıklaması  | string  |
| **hsCode** <br>_zorunlu_            | Ürün HS kodu     | string  |
| **originCountryCode** <br>_zorunlu_ | Menşei ülke kodu | string  |
| **unitPrice** <br>_zorunlu_         | Birim fiyat      | decimal |
| **currency** <br>_zorunlu_          | Para birimi      | string  |
| **quantity** <br>_zorunlu_          | Adet             | int     |

<a name="packageDetail"></a>

##### PackageDetail

| Ad                         | Açıklama       | Şema    |
| -------------------------- | -------------- | ------- |
| **type** <br>_zorunlu_     | Paket türü     | string  |
| **weight** <br>_zorunlu_   | Ağırlık (kg)   | decimal |
| **length** <br>_opsiyonel_ | Boy (cm)       | decimal |
| **height** <br>_opsiyonel_ | Yükseklik (cm) | decimal |
| **width** <br>_opsiyonel_  | En (cm)        | decimal |

---

<a name="createLabel"></a>

### POST api/shipping/v1/shipments/{reference}/labels

**Operasyon: createLabel**

_Scope_ : shipping*labels_write — \_Yetki* : can_create_labels

#### Açıklama

Gönderi depoya gelmeden taşıyıcıda takip numarası ve etiket oluşturur. İstek gövdesi yoktur. İdempotenttir: etiket zaten oluşturulmuşsa mevcut değerler `created: false` ile döner.

- `lastMileTrackingNumber` ve `lastMileCarrier` yalnızca `can_access_last_mile_tracking` yetkisi ile yanıtta yer alır.
- `label` yalnızca `can_download_labels` yetkisi ile ve taşıyıcı etiket dosyasını üretmişse yanıtta yer alır. Dosya henüz üretilmemişse yanıt yine `200` döner, `label` alanı bulunmaz; daha sonra yeniden çağrılmalıdır. PTT için etiket dosyası dönmez.

#### Rate Limit

- Client başına 10 saniyede en fazla 3 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                  | Şema |
| -------- | --------------------------- | ------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Şema                              |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | [LabelResponse](#labelResponse)   |
| **400**   | `shipment.shipmentcannotbeupdatedafterithasbeencanceled.error` - Gönderi iptal edilmiş<br> `ratenotselected.error` - Teklif seçilmemiş<br> `shipmentservicetypeisnotallowedforearlytracking.error` - Servis `navlungo-bundle`<br> `earlytrackingnotenabled.earlytrackingnotenabled.error`, `earlytrackingserviceerrors.invalidintegrationorshipment.error` - Taşıyıcı erken etiket oluşturmaya uygun değil<br> `lastmilefailure.error` - Taşıyıcı isteği reddetti<br> `record.already.exists` - Eşzamanlı istek, numara henüz okunamadı; yineleyin | [Error](#error)                   |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş, `shipping_labels_write` scope'u yok veya client'ta `can_create_labels` yetkisi yok (`Authentication/InvalidClaim`)                                                                                                                                                                                                                                                                                                                                                                     | [ProblemDetails](#problemDetails) |
| **404**   | `record.not.found` - Gönderi bulunamadı veya başka bir hesaba ait                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | [Error](#error)                   |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | [Error](#error)                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | [ProblemDetails](#problemDetails) |

#### Örnek Yanıt

```
{
  "trackingNumber": "NVL000000458213",
  "lastMileTrackingNumber": "1Z999AA10123456784",
  "lastMileCarrier": "ups",
  "created": true,
  "label": { "contentType": "application/pdf", "base64": "JVBERi0xLjQK..." }
}
```

#### Response Model

<a name="labelResponse"></a>

##### LabelResponse

| Ad                         | Açıklama                                                                         | Her zaman mevcut | Şema                    |
| -------------------------- | -------------------------------------------------------------------------------- | ---------------- | ----------------------- |
| **trackingNumber**         | Navlungo takip numarası                                                          | Evet             | string                  |
| **lastMileTrackingNumber** | Taşıyıcı takip numarası. Yalnızca `can_access_last_mile_tracking` yetkisi ile    | Hayır            | string                  |
| **lastMileCarrier**        | Taşıyıcı kodu (örn. `ups`). Yalnızca `can_access_last_mile_tracking` yetkisi ile | Hayır            | string                  |
| **created**                | `true` ise etiket bu çağrıda oluşturuldu, `false` ise zaten mevcuttu             | Evet             | boolean                 |
| **label**                  | Etiket dosyası. Yalnızca `can_download_labels` yetkisi ile ve dosya üretilmişse  | Hayır            | [LabelFile](#labelFile) |

<a name="labelFile"></a>

##### LabelFile

| Ad                            | Açıklama                                                                   | Şema   |
| ----------------------------- | -------------------------------------------------------------------------- | ------ |
| **contentType** <br>_zorunlu_ | Dosya tipi: `application/pdf`, `image/png`, `text/html`, `application/zip` | string |
| **base64** <br>_zorunlu_      | Base64 kodlanmış dosya içeriği                                             | string |

---

<a name="uploadDocuments"></a>

### POST api/shipping/v1/shipments/{reference}/documents

**Operasyon: uploadDocuments**

_Scope_ : shipping_shipments_write

#### Açıklama

Gönderiye belge kaydı açar ve her belge için dosyanın yükleneceği adresi döndürür. Dosya bu adrese ayrı bir `PUT` isteği ile, `Content-Type` başlığı yanıttaki `contentType` değeriyle yüklenir; yükleme adresi kısa süreli üretilir, adresi aldıktan sonra **5 dakika** içinde yükleyin. Belge kaydı dosya yüklenmeden önce oluşur. Belgelerden biri reddedilirse hiçbiri kaydedilmez.

#### Rate Limit

- Client başına 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                            | Şema                                              |
| -------- | --------------------------- | ----------------------------------- | ------------------------------------------------- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası           | long                                              |
| **Body** | **body** <br>_zorunlu_      | Belge kaydı açmak için gerekli şema | [UploadDocumentsRequest](#uploadDocumentsRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                    | Şema                                                |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                                                                                                                    | [UploadDocumentsResponse](#uploadDocumentsResponse) |
| **400**   | İstek doğrulamasında hata oluştu ([ProblemDetails](#problemDetails)) veya iş kuralı hatası ([Error](#error)):<br> `shipment.shipmentcannotbeupdatedafterithasbeencanceled.error` - Gönderi iptal edilmiş<br> `document.invalidtypeforearchiveinfo.error` - `eArchiveInfo` e-Arşiv dışı bir tipte gönderildi | [Error](#error)                                     |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_shipments_write` scope'u yok                                                                                                                                                                                                      | [ProblemDetails](#problemDetails)                   |
| **404**   | `record.not.found` - Gönderi bulunamadı veya başka bir hesaba ait                                                                                                                                                                                                                                           | [Error](#error)                                     |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                           | [Error](#error)                                     |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                 | [ProblemDetails](#problemDetails)                   |

#### Örnek İstek Body

```
{
  "documents": [
    { "fileName": "invoice-458213.pdf", "type": "original-invoice" },
    { "fileName": "earsiv-458213.pdf", "type": "e-archive", "eArchiveInfo": { "date": "2026-09-15T00:00:00", "number": "ABC2026000000001" } }
  ]
}
```

#### Örnek Yanıt

```
{
  "documents": [
    { "fileName": "invoice-458213.pdf", "type": "original-invoice", "contentType": "application/pdf",
      "uploadUrl": "https://navlungo-prod-shipment-docs.s3.eu-west-1.amazonaws.com/458213/original-invoice/invoice-458213.pdf?X-Amz-Expires=300&X-Amz-Signature=..." }
  ],
  "alreadyExistingFileNames": ["earsiv-458213.pdf"]
}
```

#### Request Model

<a name="uploadDocumentsRequest"></a>

##### UploadDocumentsRequest

| Ad                          | Açıklama                                                 | Şema                                            |
| --------------------------- | -------------------------------------------------------- | ----------------------------------------------- |
| **documents** <br>_zorunlu_ | Belgeler. En az 1; `fileName` istek içinde tekrar edemez | < [ShipmentDocument](#shipmentDocument) > array |

<a name="shipmentDocument"></a>

##### ShipmentDocument

| Ad                               | Açıklama                                                                      | Şema                          |
| -------------------------------- | ----------------------------------------------------------------------------- | ----------------------------- |
| **fileName** <br>_zorunlu_       | Dosya adı, en fazla 255 karakter, gönderi içinde benzersiz                    | string                        |
| **type** <br>_zorunlu_           | Belge tipi. Bkz. [Değer Listeleri](./README.md#valueLists) `documents[].type` | string                        |
| **eArchiveInfo** <br>_opsiyonel_ | e-Arşiv bilgileri. `type` `e-archive` ise zorunlu, diğer tiplerde gönderilmez | [EArchiveInfo](#eArchiveInfo) |

<a name="eArchiveInfo"></a>

##### EArchiveInfo

| Ad                       | Açıklama                             | Şema              |
| ------------------------ | ------------------------------------ | ----------------- |
| **date** <br>_zorunlu_   | e-Arşiv fatura tarihi                | string (DateTime) |
| **number** <br>_zorunlu_ | e-Arşiv fatura numarası, 16 karakter | string            |

#### Response Model

<a name="uploadDocumentsResponse"></a>

##### UploadDocumentsResponse

| Ad                                         | Açıklama                                                                                   | Şema                                                |
| ------------------------------------------ | ------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| **documents** <br>_zorunlu_                | Bu çağrıda kaydedilen belgeler ve yükleme adresleri                                        | < [DocumentUploadInfo](#documentUploadInfo) > array |
| **alreadyExistingFileNames** <br>_zorunlu_ | Aynı adla zaten kayıtlı olduğu için atlanan dosya adları; bunlar için yeni adres üretilmez | < string > array                                    |

<a name="documentUploadInfo"></a>

##### DocumentUploadInfo

| Ad                            | Açıklama                              | Şema   |
| ----------------------------- | ------------------------------------- | ------ |
| **fileName** <br>_zorunlu_    | Dosya adı                             | string |
| **type** <br>_zorunlu_        | Belge tipi                            | string |
| **contentType** <br>_zorunlu_ | Yüklemede kullanılacak `Content-Type` | string |
| **uploadUrl** <br>_zorunlu_   | Dosyanın `PUT` ile yükleneceği adres  | string |

---

<a name="getEtgbDownloadUrl"></a>

### GET api/shipping/v1/shipments/{reference}/etgb/download-url

**Operasyon: getEtgbDownloadUrl**

_Scope_ : shipping_shipments_read

#### Açıklama

Teslim edilmiş gönderinin ETGB numarasını ve varsa PDF indirme adresini döndürür. İndirme adresi kısa süreli üretilir; **2 dakika** içinde indirin. Kayıt Excel listesinden geliyorsa `fileName` ve `downloadUrl` `null`, `etgbNumber` döner.

#### Rate Limit

- Client başına 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                  | Şema |
| -------- | --------------------------- | ------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                   | Şema                                                |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| **200**   | Başarılı                                                                                                                                   | [EtgbDownloadUrlResponse](#etgbDownloadUrlResponse) |
| **400**   | `shipmentnotdelivered.error` - Gönderi henüz teslim edilmedi<br> `etgbdocumentnotfound.error` - ETGB kaydı henüz oluşmadı, sonra yineleyin | [Error](#error)                                     |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_shipments_read` scope'u yok                                      | [ProblemDetails](#problemDetails)                   |
| **404**   | `record.not.found` - Gönderi bulunamadı, başka bir hesaba ait veya iptal edilmiş                                                           | [Error](#error)                                     |
| **429**   | Rate limit aşıldı                                                                                                                          | [Error](#error)                                     |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                | [ProblemDetails](#problemDetails)                   |

#### Örnek Yanıt

```
{
  "reference": 458213,
  "etgbNumber": "26341300EX000123",
  "fileName": "26341300EX000123.pdf",
  "downloadUrl": "https://navlungo-prod-etgb-docs.s3.eu-west-1.amazonaws.com/pdf_files/2026/9/21/26341300EX000123.pdf?X-Amz-Expires=120&X-Amz-Signature=..."
}
```

#### Response Model

<a name="etgbDownloadUrlResponse"></a>

##### EtgbDownloadUrlResponse

| Ad                              | Açıklama                                               | Şema   |
| ------------------------------- | ------------------------------------------------------ | ------ |
| **reference** <br>_zorunlu_     | Gönderi referansı                                      | long   |
| **etgbNumber** <br>_zorunlu_    | ETGB numarası                                          | string |
| **fileName** <br>_opsiyonel_    | PDF dosya adı. Kayıt Excel listesinden ise `null`      | string |
| **downloadUrl** <br>_opsiyonel_ | PDF indirme adresi. Kayıt Excel listesinden ise `null` | string |

---

<a name="voidShipment"></a>

### POST api/shipping/v1/shipments/{reference}/void

**Operasyon: voidShipment**

_Scope_ : shipping_shipments_write

#### Açıklama

Depoya ulaşmamış gönderiyi iptal eder; teklif seçilmemiş gönderi de iptal edilebilir. İstek gövdesi yoktur. İptal, taşıyıcıda açılmış etiketi geri almaz. İptal sonrası gönderi detayında `status: "cancelled"` görünür; [createShipment](#createShipment) ve [createLabel](#createLabel) bu referansı reddeder.

#### Rate Limit

- Client başına 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                  | Şema |
| -------- | --------------------------- | ------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                              | Şema                                          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                                                                                                                                                                                              | [VoidShipmentResponse](#voidShipmentResponse) |
| **400**   | `shipmentalreadyinwarehouse.error` - Gönderi depoya ulaştığı için iptal edilemez<br> `shipment.shipmentalreadycancelled.error` - Gönderi zaten iptal edilmiş<br> `shipment.shipmentcannotbecancelledwhenbundlereferenceisexists.error` - Pakete (bundle) bağlı gönderi iptal edilemez<br> `notpublicapishipment.error` - Gönderi bu api üzerinden oluşturulmadığı için iptal edilemez | [Error](#error)                               |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_shipments_write` scope'u yok                                                                                                                                                                                                                                                                                | [ProblemDetails](#problemDetails)             |
| **404**   | `record.not.found` - Gönderi bulunamadı veya başka bir hesaba ait                                                                                                                                                                                                                                                                                                                     | [Error](#error)                               |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                                                                                                     | [Error](#error)                               |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                                                                                           | [ProblemDetails](#problemDetails)             |

#### Örnek Yanıt

```
{ "reference": 458213, "status": "cancelled", "cancelledAt": "17.09.2026 10:15" }
```

#### Response Model

<a name="voidShipmentResponse"></a>

##### VoidShipmentResponse

| Ad                            | Açıklama              | Şema              |
| ----------------------------- | --------------------- | ----------------- |
| **reference** <br>_zorunlu_   | Gönderi referansı     | long              |
| **status** <br>_zorunlu_      | Her zaman `cancelled` | string            |
| **cancelledAt** <br>_zorunlu_ | İptal zamanı          | string (DateTime) |

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
