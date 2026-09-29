# Navlungo Shipping Api

<a name="overview"></a>

## Önemli Notlar

1. Size iletilen client bilgileri ile yalnızca ilgili ortamda işlem yapabilirsiniz. QA ortam için iletilen client bilgileri ile production apilerine erişemezsiniz.
2. QA ortamı varsayılan olarak hafta içi gece ve hafta sonları kapalıdır, apilerden yanıt alamayabilirsiniz. Çalışma planlamanız ve bilgisini önceden vermeniz halinde, ortamı açık tutabiliriz.
3. Client secret yalnızca client oluşturulduğunda bir kez paylaşılır ve Navlungo tarafında saklanmaz. Secret'ınızı kaybederseniz yeni bir client tanımlanması gerekir.

## 1. Genel Bakış

Shipping Api ile Navlungo çözüm ortakları, kendi sistemlerinden **sunucudan sunucuya** bağlanarak teklif alabilir, gönderi oluşturabilir, etiket ve belge yönetebilir, gönderiyi takip edebilir ve iptal edebilir. [Store API](../store-api/README.md)'den farklı olarak bu api'de bir Navlungo kullanıcısı adına değil, doğrudan **client** adına işlem yapılır; client Navlungo tarafından çözüm ortağının hesabına (shipper) bağlanır ve tüm gönderiler bu hesap altında oluşur.

- Teklif seçildiğinde gönderi "depoya ulaşması bekleniyor" statüsünde oluşur; ödeme bu api üzerinden alınmaz, gönderi depoya ulaştığında Navlungo tarafında yapılır.

---

<a name="authorization"></a>

## 2. Yetkilendirme

Bu api'nin tüm kaynakları OAuth2 **client_credentials** akışı ile alınan access token ile çağrılır. Token, Navlungo kimlik sunucusundan alınır ve `Authorization: Bearer <access_token>` başlığı ile gönderilir. Detaylar için [Token Apisi](./token.md).

Client credentials akışı client_id ve client_secret'ın gönderilmesini gerektirdiği için token istekleri mutlaka güvenli bir **server side** uygulama tarafından yapılmalıdır. Bu akışta refresh_token üretilmez; token süresi dolduğunda yeni token alınır.

<a name="scopes"></a>

### 2.1. Scope'lar

Her endpoint bir scope ile korunur. Token alırken kullanılacak endpointlerin scope'ları boşlukla ayrılarak gönderilir. Client'a tanımlı olmayan bir scope istenirse token üretilmez.

| Scope                        | Endpointler                                                                                                                                     |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **shipping_rates_write**     | [createRates](./rates.md#createRates)                                                                                                           |
| **shipping_shipments_write** | [createShipment](./shipment.md#createShipment), [uploadDocuments](./shipment.md#uploadDocuments), [voidShipment](./shipment.md#voidShipment)    |
| **shipping_shipments_read**  | [getShipment](./shipment.md#getShipment), [getTracking](./cargoTracking.md#getTracking), [getEtgbDownloadUrl](./shipment.md#getEtgbDownloadUrl) |
| **shipping_labels_write**    | [createLabel](./shipment.md#createLabel)                                                                                                        |

Token'da ilgili scope yoksa endpoint `401` ve `Authentication/InvalidScope` tipinde hata döner.

<a name="claims"></a>

### 2.2. Yetkiler (Client Claim'leri)

Bazı yetenekler scope'tan bağımsız olarak client tanımında verilir. Bu yetkiler client oluşturulurken tanımlanır, sonradan Navlungo tarafından eklenebilir; eklenen yetki **yeni alınan token** ile geçerli olur.

| Yetki                             | Yoksa                                                                                                         |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **can_access_last_mile_tracking** | Takip ve etiket yanıtlarında `lastMileTrackingNumber` ve `lastMileCarrier` alanları yer almaz                 |
| **can_create_labels**             | [createLabel](./shipment.md#createLabel) `401` `Authentication/InvalidClaim` döner                            |
| **can_download_labels**           | Etiket yanıtında `label` alanı yer almaz                                                                      |
| **can_create_domestic_shipments** | Gönderici ve alıcı ülkesi aynı olan gönderi `400` `shipment.fromanddestinationcountrycannotbesame.error` alır |

---

## 3. Gönderi Oluşturma Akışı

**Tanımlar**

- **Client**: Navlungo Shipping Api'sini kullanan kurum (client_id ve client_secret'a sahiptir). Navlungo tarafında bir hesaba (shipper) bağlıdır; tüm gönderiler bu hesap altında oluşur.
- **reference**: Navlungo gönderi referansı. [createRates](./rates.md#createRates) yanıtında döner ve sonraki tüm çağrılarda gönderiyi tanımlar.

**Akış**

1. Client, [createRates](./rates.md#createRates) ile adres, ürün ve paket bilgilerini gönderir. Navlungo bir gönderi kaydı açar ve `reference`, `searchId` ve teklif listesini (`rates`) döndürür.
2. Client, [createShipment](./shipment.md#createShipment) ile `reference`, `searchId` ve seçtiği teklifin `rateId` değerini göndererek teklifi seçer. Gönderi "depoya ulaşması bekleniyor" statüsüne geçer ve Navlungo takip numarası atanır.
3. İsteğe bağlı olarak [createLabel](./shipment.md#createLabel) ile taşıyıcı etiketi oluşturulur, [uploadDocuments](./shipment.md#uploadDocuments) ile gönderiye belge eklenir.
4. [getShipment](./shipment.md#getShipment) ve [getTracking](./cargoTracking.md#getTracking) ile gönderi durumu ve takip hareketleri izlenir.
5. Depoya ulaşmamış gönderi [voidShipment](./shipment.md#voidShipment) ile iptal edilebilir; teslim edilen gönderinin ETGB belgesi [getEtgbDownloadUrl](./shipment.md#getEtgbDownloadUrl) ile alınır.

**Dikkat Edilmesi Gereken Noktalar**

- Her [createRates](./rates.md#createRates) çağrısı yeni bir gönderi referansı açar; teklif seçilmeyen referanslar `rate-not-selected` statüsünde kalır. Aynı gönderi için mevcut `reference`, `searchId` ve `rateId` ile ilerlenmelidir.
- Tekliflerin geçerlilik süresi yoktur; [createShipment](./shipment.md#createShipment) sırasında teklif yeniden doğrulanır, geçersizse `rateselectionfailed.error` döner ve yeni bir [createRates](./rates.md#createRates) çağrısı gerekir.
- `isRequired: true` olan ek hizmetler [createShipment](./shipment.md#createShipment) isteğinde `selectedServices` içinde gönderilmelidir.
- İptal, taşıyıcıda açılmış etiketi geri almaz.

---

## 4. Biçim ve Ortak Kurallar

### URI şeması

_Host_ : api.navlungo.com
_Host(Test)_ : api-qa.navlungo.com
_Schemes_ : HTTPS
_Ön ek_ : `api/shipping/v1`

- İstek ve yanıt gövdeleri JSON, alan adları camelCase'dir. `Content-Type: application/json` başlığı gönderilmelidir.
- Tarihler UTC ve `dd.MM.yyyy HH:mm` biçimindedir. Örnek: `"17.09.2026 10:15"`.
- Değeri olmayan alanlar `null` döner. Yalnızca yetkiye bağlı alanlar (`lastMileTrackingNumber`, `lastMileCarrier`, `label`) yetki yoksa yanıtta hiç yer almaz.
- Ağırlık yanıtlarda her zaman kilogram, boyutlar santimetre cinsindendir. İstekte `weightUnit`/`dimensionUnit` ile `lb`/`in` gönderilebilir; değerler kg/cm'ye çevrilerek saklanır.

### Rate Limit

Sınırlar client bazında, 10 saniyelik kayan pencere ile uygulanır. Aşıldığında `429` ve aşağıdaki gövde döner:

```
{ "code": "too.many.requests", "message": "Too many requests. Please try again later.", "data": [] }
```

| Endpoint                                                    | Limit (10 sn) |
| ----------------------------------------------------------- | ------------- |
| POST api/shipping/v1/rates                                  | 10            |
| POST api/shipping/v1/shipments                              | 5             |
| GET api/shipping/v1/shipments?reference=                    | 20            |
| GET api/shipping/v1/shipments/{reference}/tracking          | 20            |
| POST api/shipping/v1/shipments/{reference}/labels           | 3             |
| POST api/shipping/v1/shipments/{reference}/documents        | 5             |
| GET api/shipping/v1/shipments/{reference}/etgb/download-url | 5             |
| POST api/shipping/v1/shipments/{reference}/void             | 5             |

### Hata Yanıtları

İki farklı hata gövdesi vardır.

**İş kuralı hataları** (`400`, `404`) [Error](#error) nesnesi ile döner:

```
{ "code": "ratealreadyselected.error", "message": "A rate has already been selected for this reference.", "data": [] }
```

Kayıt bulunamadığında her endpoint `404` ve `record.not.found` döner; başka bir hesaba ait gönderi de aynı yanıtı alır.

**Alan doğrulama ve yetkilendirme hataları** (`400`, `401`) [ProblemDetails](#problemDetails) nesnesi ile döner. Alan doğrulama hatasında `extensions.validationErrors` dizisi hatalı alanları içerir:

```
{
  "type": "Common/Validation", "status": 400, "title": "Verification Error",
  "detail": "An error occurred during request verification", "path": "/api/shipping/v1/rates",
  "extensions": { "validationErrors": [ { "propertyName": "Packages[0].Weight", "errorMessage": "'Weight' must be greater than '0'." } ] }
}
```

| Durum                                                | Yanıt                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------- |
| Token gönderilmedi                                   | `401` `Common/Unauthenticated`                                |
| Token geçersiz veya süresi dolmuş                    | `401` `Authentication/InvalidToken`                           |
| Token'da gerekli scope yok                           | `401` `Authentication/InvalidScope`                           |
| Client'ta gerekli yetki (claim) yok                  | `401` `Authentication/InvalidClaim`                           |
| Client askıya alınmış veya bu api için tanımlı değil | `401` (`Common/Forbidden` veya `Authentication/InvalidToken`) |
| Beklenmeyen hata                                     | `500` `Common/Unexpected`                                     |

Gövde doğrulaması yetki kontrolünden önce çalışır: eksik veya hatalı alanlı bir istek, token olmasa da `400` alan doğrulama hatası döner.

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

---

<a name="valueLists"></a>

## 5. Değer Listeleri

| Alan                                            | Değerler                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `shipmentType`                                  | `sales`, `sample`, `gift`, `micro-export`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `currency` (gönderi ve ürün)                    | `EUR`, `USD`, `TRY`, `GBP`, `CAD`, `SAR`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `weightUnit` / `dimensionUnit`                  | `kg`/`cm` (varsayılan) veya `lb`/`in`; ikisi birlikte verilir                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `packages[].type`                               | `box`, `envelope`, `document`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `countryCode`, `originCountry`                  | ISO 3166-1 alpha-2, büyük harf: `TR`, `US`, `DE`                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `rates[].serviceType`                           | `express`, `eco-express`, `expedited`, `ups-standart`, `registered-small-package`, `registered-package-air`, `us-eco-cargo`, `US-EXPS`, `INT-ECO`, `USPM`, `US-GRD`, `DM-FGR`, `DM-USGR`, `DM-FGE`, `widect-usps`, `widect-amazon-shipping`, `widect-ups-ground`, `widect-b2b`, `economy-road`, `navlungo-bundle` (yük hizmetleri `express` olarak döner)                                                                                                                                                                      |
| `rates[].services[].code`, `selectedServices[]` | `ddp`, `insurance`, `customs-tax-duty`, `atr`; yanıtta ayrıca `premium-tracking` görünebilir, seçilemez                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `status` (gönderi)                              | `rate-not-selected`, `awaiting-warehouse-arrival`, `in-warehouse`, `dispatched`, `cancelled`                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `checkpoints[].status` (takip)                  | `Pending`, `Info Received`, `In Transit`, `Out For Delivery`, `Attempt Fail`, `Available For Pickup`, `Delivered`, `Exception`, `Expired`. Taşıyıcı hareketlerinde birleşik yazılır (`InTransit`); ayrıntı için [Takip Durumları](./cargoTracking.md#trackingStatuses)                                                                                                                                                                                                                         |
| `carrier` (gönderi detayı)                      | Taşıyıcı adı: `United Parcel Service`, `Federal Express`, `DHL International`, `TNT Express`, `PTT`, `Aramex`, `Asendia`, `THY`, `You Parcel`, `Pts`, `ShipStation For Usps`, `Navlungo`                                                                                                                                                                                                                                                                                                                                       |
| `lastMileCarrier`                               | Taşıyıcı kodu: `ups`, `fedex`, `dhl`, `tnt`, `gls`, `usps`, `aramex`, `dpd`, `pts` vb.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `label.contentType`                             | `application/pdf`, `image/png`, `text/html`, `application/zip`                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `documents[].type`                              | `msds`, `e-archive`, `tsca`, `fda`, `eori-no`, `ddp`, `insurance-policy`, `battery-form-label`, `no-amazon-fba-tag`, `original-invoice`, `ozon-label`, `metal-composite-document`, `navlungo-label`, `last-mile-label`, `others`, `aphis-core-admissibility-guidance-form`, `diamond-certificate`, `epa-noa`, `epa-vne-form`, `interim-footwear-invoice`, `fsvp-importer-consent-form`, `lacey-act-ppq-505`, `section-232`, `watch-breakout`, `usda-ams-organics-form`, `prior-notices`, `packing-list`, `return-label`, `atr` |

---

## 6. Operasyonlar

[Token Apisi](./token.md)</br>
[Teklif Apisi](./rates.md)</br>
[Gönderi Apisi](./shipment.md)</br>
[Gönderi Takip Apisi](./cargoTracking.md)</br>
