# Express Shipment API

<a name="overview"></a>

## Genel Bakış

Navlungo, gönderileriniz için etiket oluşturma ve yönetme imkanı sunan bir API sağlamaktadır. Bu API ile Navlungo çözüm ortakları gönderilerini yönetebilir ve etiketlerini oluşturabilirler.

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

### Operasyonlar

[createLabel](#createLabel)<br>
[getLabel](#getLabel)<br>
[getTracking](#getTracking)<br>
[getShipment](#getShipment)<br>
[uploadDocument](#uploadDocument)<br>
[getEtgbDownloadUrl](#getEtgbDownloadUrl)<br>
[voidShipment](#voidShipment)<br>

<a name="paths"></a>

## Paths

<a name="createLabel"></a>

### POST api/shipments/v1/{shipmentId}/label

**Operasyon: createLabel**

#### Açıklama

Belirtilen gönderi referansı için etiket oluşturur. Bu işlem için kullanıcının yetkilendirilmiş olması gerekmektedir.

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 10 saniyede en fazla 3 istek yapılabilir

#### Parametreler

| Tip      | İsim                         | Açıklama                                                                                                                 | Şema |
| -------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ---- |
| **Path** | **shipmentId** <br>_zorunlu_ | Gönderinin tekil id'si (/stores/v2/{store_id}/orders/{order_reference}/ship API'sinin response'undaki shipmentId değeri) | Guid |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                            |
| --------- | -------------------------------------------------------------- | ------------------------------- |
| **200**   | Başarılı                                                       | [LabelResponse](#labelResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz           | [Error](#error)                 |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                 |
| **429**   | Rate limit aşıldı                                              | [Error](#error)                 |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                    | [Error](#error)                 |

#### Response Model

##### LabelResponse

| Ad                                       | Açıklama                    | Şema   |
| ---------------------------------------- | --------------------------- | ------ |
| **lastMileTrackingNumber** <br>_zorunlu_ | Son taşıyıcı takip numarası | string |

---

<a name="getLabel"></a>

### GET api/shipments/v1/{shipmentId}/label

**Operasyon: getLabel**

#### Açıklama

Belirtilen gönderi referansı için oluşturulmuş etiketi getirir. Bu işlem için kullanıcının yetkilendirilmiş olması gerekmektedir.

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 10 saniyede en fazla 3 istek yapılabilir

#### Parametreler

| Tip      | İsim                         | Açıklama                                                                                                                 | Şema |
| -------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ---- |
| **Path** | **shipmentId** <br>_zorunlu_ | Gönderinin tekil id'si (/stores/v2/{store_id}/orders/{order_reference}/ship API'sinin response'undaki shipmentId değeri) | Guid |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                                                  |
| --------- | -------------------------------------------------------------- | ----------------------------------------------------- |
| **200**   | Başarılı                                                       | [LabelDownloadUrlResponse](#labelDownloadUrlResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz           | [Error](#error)                                       |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                                       |
| **404**   | Etiket bulunamadı                                              | [Error](#error)                                       |
| **429**   | Rate limit aşıldı                                              | [Error](#error)                                       |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                    | [Error](#error)                                       |

#### Response Model

##### LabelDownloadUrlResponse

| Ad                         | Açıklama             | Şema   |
| -------------------------- | -------------------- | ------ |
| **labelUrl** <br>_zorunlu_ | Etiket indirme linki | string |

---

<a name="getTracking"></a>

### GET api/shipments/v1/{reference}/tracking

**Operasyon: getTracking**

#### Açıklama

Belirtilen gönderi referansı için takip bilgilerini getirir. Bu işlem için kullanıcının yetkilendirilmiş olması gerekmektedir.

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 10 saniyede en fazla 20 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                                                                                 | Şema |
| -------- | --------------------------- | ---------------------------------------------------------------------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası (stores/v1/{id}/orders/ship API'sindeki quoteReference değeri) | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                                                        |
| --------- | -------------------------------------------------------------- | ----------------------------------------------------------- |
| **200**   | Başarılı                                                       | [TrackingCheckpointsResponse](#trackingCheckpointsResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz           | [Error](#error)                                             |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                                             |
| **404**   | Gönderi veya takip bilgisi bulunamadı                          | [Error](#error)                                             |
| **429**   | Rate limit aşıldı                                              | [Error](#error)                                             |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                    | [Error](#error)                                             |

#### Response Model

##### TrackingCheckpointsResponse

| Ad                                       | Açıklama              | Şema                                |
| ---------------------------------------- | --------------------- | ----------------------------------- |
| **trackingNumber** <br>_zorunlu_         | Takip numarası        | string                              |
| **originCountry** <br>_zorunlu_          | Çıkış ülkesi          | string                              |
| **destinationCountry** <br>_zorunlu_     | Varış ülkesi          | string                              |
| **pickupDate** <br>_zorunlu_             | Alım tarihi           | datetime                            |
| **expectedDeliveryDate** <br>_opsiyonel_ | Tahmini teslim tarihi | datetime                            |
| **carrier** <br>_zorunlu_                | Taşıyıcı              | string                              |
| **originLocation** <br>_zorunlu_         | Çıkış lokasyonu       | string                              |
| **destinationLocation** <br>_zorunlu_    | Varış lokasyonu       | string                              |
| **checkpoints** <br>_zorunlu_            | Takip noktaları       | < [Checkpoint](#checkpoint) > array |

##### Checkpoint

| Ad                                       | Açıklama               | Şema     |
| ---------------------------------------- | ---------------------- | -------- |
| **checkpointTime** <br>_zorunlu_         | Kontrol noktası zamanı | datetime |
| **status** <br>_zorunlu_                 | Durum                  | string   |
| **subStatusMessage** <br>_zorunlu_       | Alt durum mesajı       | string   |
| **subStatusDescription** <br>_opsiyonel_ | Alt durum açıklaması   | string   |
| **country** <br>_opsiyonel_              | Ülke                   | string   |
| **city** <br>_opsiyonel_                 | Şehir                  | string   |
| **zip** <br>_opsiyonel_                  | Posta kodu             | string   |
| **location** <br>_opsiyonel_             | Lokasyon               | string   |

---

<a name="getShipment"></a>

### GET api/shipments/v1/{reference}

**Operasyon: getShipment**

#### Açıklama

Belirtilen gönderi referansı için gönderinin güncel bilgilerini getirir; güncel fiyat, gönderici ve alıcı adresleri, ürün içerikleri ve paketlerin güncel ölçüleri döner. Bu işlem için kullanıcının yetkilendirilmiş olması gerekmektedir.

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 10 saniyede en fazla 20 istek yapılabilir

#### Parametreler

| Tip      | İsim                        | Açıklama                                                                                                                          | Şema |
| -------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ---- |
| **Path** | **reference** <br>_zorunlu_ | Gönderi referans numarası (/stores/v2/{store_id}/orders/{order_reference}/ship API'sinin response'undaki shipmentReference değeri) | long |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                                              |
| --------- | -------------------------------------------------------------- | ------------------------------------------------- |
| **200**   | Başarılı                                                       | [ShipmentDetailResponse](#shipmentDetailResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz           | [Error](#error)                                   |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                                   |
| **404**   | Gönderi bulunamadı                                             | [Error](#error)                                   |
| **429**   | Rate limit aşıldı                                              | [Error](#error)                                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                    | [Error](#error)                                   |

#### Response Model

<a name="shipmentDetailResponse"></a>

##### ShipmentDetailResponse

| Ad                                        | Açıklama                                                           | Şema                                    |
| ----------------------------------------- | ------------------------------------------------------------------ | --------------------------------------- |
| **shipmentId** <br>_zorunlu_              | Gönderinin tekil id'si                                             | Guid                                    |
| **reference** <br>_zorunlu_               | Gönderi referans numarası                                          | long                                    |
| **warehouseArrivalDate** <br>_opsiyonel_  | Gönderinin depoya giriş tarihi (depoya ulaşmadıysa boş döner)      | datetime                                |
| **warehouseDispatchDate** <br>_opsiyonel_ | Gönderinin depodan çıkış tarihi (depodan çıkmadıysa boş döner)     | datetime                                |
| **price** <br>_zorunlu_                   | Gönderinin güncel fiyatı                                           | [Price](#shipmentDetailPrice)           |
| **senderAddress** <br>_zorunlu_           | Gönderici adresi                                                   | [Address](#shipmentDetailAddress)       |
| **receiverAddress** <br>_zorunlu_         | Alıcı adresi                                                       | [Address](#shipmentDetailAddress)       |
| **products** <br>_zorunlu_                | Proforma fatura ürün içerikleri                                    | < [Product](#shipmentDetailProduct) > array |
| **packages** <br>_zorunlu_                | Paketlerin güncel ölçüleri (depo ölçümü sonrası güncellenmiş hali) | < [Package](#shipmentDetailPackage) > array |

<a name="shipmentDetailPrice"></a>

##### Price

| Ad                         | Açıklama    | Şema    |
| -------------------------- | ----------- | ------- |
| **amount** <br>_zorunlu_   | Tutar       | decimal |
| **currency** <br>_zorunlu_ | Para birimi | string  |

<a name="shipmentDetailAddress"></a>

##### Address

| Ad                                     | Açıklama                                       | Şema   |
| -------------------------------------- | ---------------------------------------------- | ------ |
| **contactName** <br>_zorunlu_          | İletişim kurulacak kişinin adı                 | string |
| **companyName** <br>_opsiyonel_        | Şirket adı                                     | string |
| **countryCode** <br>_zorunlu_          | Ülke kodu (ISO 3166-1 alpha-2)                 | string |
| **stateCode** <br>_opsiyonel_          | Eyalet kodu                                    | string |
| **postalCode** <br>_opsiyonel_         | Posta kodu                                     | string |
| **city** <br>_zorunlu_                 | Şehir                                          | string |
| **town** <br>_opsiyonel_               | İlçe (sadece gönderici adresinde döner)        | string |
| **firstLine** <br>_zorunlu_            | Adres satırı 1                                 | string |
| **secondLine** <br>_opsiyonel_         | Adres satırı 2                                 | string |
| **thirdLine** <br>_opsiyonel_          | Adres satırı 3                                 | string |
| **email** <br>_opsiyonel_              | E-posta adresi                                 | string |
| **phoneCode** <br>_opsiyonel_          | Telefon ülke kodu                              | string |
| **phoneNumber** <br>_opsiyonel_        | Telefon numarası                               | string |

<a name="shipmentDetailProduct"></a>

##### Product

| Ad                                  | Açıklama          | Şema    |
| ----------------------------------- | ----------------- | ------- |
| **description** <br>_opsiyonel_     | Ürün açıklaması   | string  |
| **hsCode** <br>_zorunlu_            | HS (GTİP) kodu    | string  |
| **originCountryCode** <br>_zorunlu_ | Menşe ülke kodu   | string  |
| **unitPrice** <br>_zorunlu_         | Birim fiyat       | decimal |
| **currency** <br>_zorunlu_          | Para birimi       | string  |
| **quantity** <br>_zorunlu_          | Adet              | int     |

<a name="shipmentDetailPackage"></a>

##### Package

| Ad                          | Açıklama                                                        | Şema    |
| --------------------------- | --------------------------------------------------------------- | ------- |
| **type** <br>_zorunlu_      | Paket tipi (`Box`, `Envelope`, `Document`)                      | string  |
| **weight** <br>_zorunlu_    | Ağırlık (kg)                                                    | decimal |
| **length** <br>_opsiyonel_  | Uzunluk (cm) (sadece `Box` tipinde döner)                       | decimal |
| **height** <br>_opsiyonel_  | Yükseklik (cm) (sadece `Box` tipinde döner)                     | decimal |
| **width** <br>_opsiyonel_   | Genişlik (cm) (sadece `Box` tipinde döner)                      | decimal |

---

<a name="getEtgbDownloadUrl"></a>

### GET api/shipments/v1/{shipmentId}/etgb/download-url

**Operasyon: GetEtgbDownloadUrl**

#### Açıklama

Belirtilen gönderi için ETGB dökümanı indirme linkini getirir. Bu işlem için kullanıcının yetkilendirilmiş olması gerekmektedir. ETGB dökümanı sadece gönderi teslim edildikten sonra indirilebilir. Etgb evrağı gümrükten alınmadığı bazı durumlarda download url yerine sadece etgbNumber geri dönebilir.

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 10 saniyede en fazla 5 istek yapılabilir

#### Parametreler

| Tip      | İsim                         | Açıklama               | Şema |
| -------- | ---------------------------- | ---------------------- | ---- |
| **Path** | **shipmentId** <br>_zorunlu_ | Gönderinin tekil id'si | Guid |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                           | Şema                                              |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| **200**   | Başarılı                                                                                                                                                           | [EtgbDownloadUrlReponse](#etgbDownloadUrlReponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz<br> `shipmentnotfound.error` - Gönderi bulunamadı<br> `etgbdocumentnotfound.error` - ETGB dökümanı bulunamadı | [Error](#error)                                   |
| **401**   | Yetkilendirme hatası.                                                                                                                                              |                                                   |
| **400**   | Gönderi teslim edilmediği için ETGB dökümanı indirilemez<br> `shipmentnotdelivered.error` - Gönderi teslim edilmedi                                                | [Error](#error)                                   |
| **429**   | Rate limit aşıldı                                                                                                                                                  | [Error](#error)                                   |

#### Response Model

##### EtgbDownloadUrlReponse

| Ad                             | Açıklama      | Şema   |
| ------------------------------ | ------------- | ------ |
| **FileName** <br>_nullable_    | Dosya adı     | string |
| **DownloadUrl** <br>_nullable_ | İndirme linki | string |
| **EtgbNumber** <br>_zorunlu_   | Etgb Numarası | string |

---

<a name="uploadDocument"></a>

### POST api/shipments/v1/{shipmentId}/documents

**Operasyon: UploadDocument**

#### Açıklama

Belirtilen gönderi için dökümanları yükler. Bu işlem için kullanıcının yetkilendirilmiş olması gerekmektedir.

API, dökümanları yüklemek için bir AWS S3 presigned URL'i döner. Bu URL 10 dakika boyunca geçerlidir. Bu süre zarfında client'ın dosyayı belirtilen URL'e yüklemesi gerekmektedir. Endpoint yanıtı başarılı olduğunda, sistem dosyanın yüklendiğini varsayar. Bu nedenle, dosyanın hemen yüklenmesi için bu endpoint kullanılmalıdır.

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 2 saniyede en fazla 1 istek yapılabilir

#### Parametreler

| Tip      | İsim                         | Açıklama                          | Şema                                                            |
| -------- | ---------------------------- | --------------------------------- | --------------------------------------------------------------- |
| **Path** | **shipmentId** <br>_zorunlu_ | Gönderinin tekil id'si            | Guid                                                            |
| **Body** | **request** <br>_zorunlu_    | Yüklenecek dökümanların bilgileri | [UpdateShipmentDocumentRequest](#updateShipmentDocumentRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                             | Şema                                                              |
| --------- | ---------------------------------------------------- | ----------------------------------------------------------------- |
| **200**   | Başarılı                                             | [UpdateShipmentDocumentResponse](#UpdateShipmentDocumentResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz | [Error](#error)                                                   |
| **401**   | Yetkilendirme hatası.                                |
| **404**   | Gönderi bulunamadı                                   | [Error](#error)                                                   |
| **429**   | Rate limit aşıldı                                    | [Error](#error)                                                   |

#### Request Model

##### UpdateShipmentDocumentRequest

| Ad                                  | Açıklama        | Şema                                                      |
| ----------------------------------- | --------------- | --------------------------------------------------------- |
| **ShipmentDocuments** <br>_zorunlu_ | Döküman listesi | < [ShipmentDocumentModel](#shipmentDocumentModel) > array |

##### ShipmentDocumentModel

| Ad                               | Açıklama                                                                                                                                                              | Şema                          |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| **FileName** <br>_zorunlu_       | Dosya adı                                                                                                                                                             | string                        |
| **Type** <br>_zorunlu_           | Döküman tipi. Geçerli tipler: `msds`, `e-archive`, `tsca`, `fda`, `eori-no`, `ddp`, `insurance-policy`, `battery-form-label`, `no-amazon-fba-tag`, `original-invoice` | string                        |
| **EArchiveInfo** <br>_opsiyonel_ | E-Arşiv bilgileri (sadece `Type` `e-archive` ise gereklidir, değil ise göz önünde bulundurulmaz.)                                                                     | [EArchiveInfo](#earchiveInfo) |

##### EArchiveInfo

| Ad                       | Açıklama                                 | Şema     |
| ------------------------ | ---------------------------------------- | -------- |
| **Date** <br>_zorunlu_   | E-Arşiv tarihi                           | datetime |
| **Number** <br>_zorunlu_ | E-Arşiv numarası (16 karakter olmalıdır) | string   |

#### Örnek İstek Body

```json
{
	"ShipmentDocuments": [
		{
			"Type": "e-archive",
			"FileName": "x.pdf",
			"EArchiveInfo": {
				"Date": "2023-10-27",
				"Number": "NAV2023000000123"
			}
		}
	]
}
```

#### Response Model

##### UpdateShipmentDocumentResponse

| Ad                                         | Açıklama                         | Şema                                                |
| ------------------------------------------ | -------------------------------- | --------------------------------------------------- |
| **Documents** <br>_zorunlu_                | Yükleme bilgileri                | < [DocumentUploadInfo](#documentUploadInfo) > array |
| **AlreadyExistingFileNames** <br>_zorunlu_ | Zaten var olan dosyaların adları | < string > array                                    |

##### DocumentUploadInfo

| Ad                         | Açıklama      | Şema                |
| -------------------------- | ------------- | ------------------- |
| **FileName** <br>_zorunlu_ | Dosya adı     | string              |
| **Type** <br>_zorunlu_     | Döküman tipi  | string              |
| **UrlInfo** <br>_zorunlu_  | URL bilgileri | [UrlInfo](#urlInfo) |

##### UrlInfo

| Ad                            | Açıklama      | Şema   |
| ----------------------------- | ------------- | ------ |
| **ContentType** <br>_zorunlu_ | İçerik tipi   | string |
| **UploadUrl** <br>_zorunlu_   | Yükleme URL'i | string |

---

<a name="voidShipment"></a>

### POST api/shipments/v1/{shipmentId}/void

**Operasyon: voidShipment**

#### Açıklama

Belirtilen gönderiyi iptal eder. Sadece henüz depoya ulaşmamış gönderiler iptal edilebilir. 

#### Rate Limit

- Her IP ve User-Agent kombinasyonu için 1 saniyede en fazla 2 istek yapılabilir

#### Parametreler

| Tip      | İsim                         | Açıklama                                                                                                                 | Şema |
| -------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ---- |
| **Path** | **shipmentId** <br>_zorunlu_ | Gönderinin tekil id'si (/stores/v2/{store_id}/orders/{order_reference}/ship API'sinin response'undaki shipmentId değeri) | Guid |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Şema            |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| **200**   | Başarılı (yanıt gövdesi dönmez)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | -               |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz<br> `shipmentnotfound.error` - Gönderi bulunamadı<br> `unauthorized.action` - Bu işlemi yapma yetkiniz yok<br> `notpublicapishipment.error` - Gönderi API üzerinden oluşturulmadığı için iptal edilemez<br> `shipmentalreadyinwarehouse.error` - Gönderi depoya ulaştığı için iptal edilemez<br> `shipment.shipmentalreadycancelled.error` - Gönderi zaten iptal edilmiş<br> `shipment.shipmentcannotbecancelledwhenbundlereferenceisexists.error` - Bundle referansı olan gönderi iptal edilemez | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | [Error](#error) |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | [Error](#error) |

---

## Common Models

### Error

Genel hata nesnesi

| Ad                        | Açıklama               | Şema   |
| ------------------------- | ---------------------- | ------ |
| **code** <br>_zorunlu_    | Hata kodu              | string |
| **message** <br>_zorunlu_ | Hata mesajı            | string |
| **data** <br>_opsiyonel_  | Hataya ait ek bilgiler | object |
