# Express Shipment API

<a name="overview"></a>

## Genel Bakış

Navlungo, gönderileriniz için etiket oluşturma ve yönetme imkanı sunan bir API sağlamaktadır. Bu API ile Navlungo çözüm ortakları gönderilerini yönetebilir ve etiketlerini oluşturabilirler.

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

[createLabel](#createLabel)<br>
[getLabel](#getLabel)<br>
[getTracking](#getTracking)<br>
[uploadDocument](#uploadDocument)<br>

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

- Her IP ve User-Agent kombinasyonu için 2 saniyede en fazla 1 istek yapılabilir

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

| HTTP Kodu | Açıklama                                                       | Şema                                                              |
| --------- | -------------------------------------------------------------- | ----------------------------------------------------------------- |
| **200**   | Başarılı                                                       | [UpdateShipmentDocumentResponse](#UpdateShipmentDocumentResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz           | [Error](#error)                                                   |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                                                   |
| **404**   | Gönderi bulunamadı                                             | [Error](#error)                                                   |
| **429**   | Rate limit aşıldı                                              | [Error](#error)                                                   |

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

## Common Models

### Error

Genel hata nesnesi

| Ad                        | Açıklama               | Şema   |
| ------------------------- | ---------------------- | ------ |
| **code** <br>_zorunlu_    | Hata kodu              | string |
| **message** <br>_zorunlu_ | Hata mesajı            | string |
| **data** <br>_opsiyonel_  | Hataya ait ek bilgiler | object |
