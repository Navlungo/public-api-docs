# Mağaza Apisi

<a name="overview"></a>

## Genel Bakış

Navlungo kullanıcıları mağazalarını bağlayarak siparişlerini Navlungo arayüzlerinde görüntüleyebilmektedir. Bu Api'ler ile sisteme kayıt olmuş bir kullanıcı arayüze girmeden mağazasını bağlayabilir ve mağaza siparişlerinde kullanılacak olan varsayılan adreslerini belirleyebilir.

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

- [createStore](#createStore)<br>
- [getStores](#getStores)<br>
- [getStore](#getStore)<br>
- [updateStore](#updateStore)<br>
- [updateStoreAddress](#updateStoreAddress)<br>
- [updateInvoiceAddress](#updateInvoiceAddress)<br>
- [deleteStore](#deleteStore)<br>
- [shipStoreOrder](#shipStoreOrder)<br>
- [trackOrder](#trackOrder)<br>

<a name="paths"></a>

## Paths

<a name="createStore"></a>

### POST stores/v1

**Operasyon: createStore**

#### Açıklama

İşlem yapılan kullanıcıya ait bir mağaza kaydı oluşturur ve mağazanın varsayılan fatura ile gönderim adres bilgilerini yaratır.

#### Parametreler

| Tip      | İsim                   | Açıklama                          | Şema                                      |
| -------- | ---------------------- | --------------------------------- | ----------------------------------------- |
| **Body** | **body** <br>_zorunlu_ | Mağaza yaratmak için gerekli şema | [CreateStoreRequest](#createStoreRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema                                        |
| --------- | --------------------------------------------------------------- | ------------------------------------------- |
| **200**   | Başarılı                                                        | [CreateStoreResponse](#createStoreResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error)                             |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error)                             |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error)                             |

<a name="definitions"></a>

## Tanımlar

<a name="createStoreRequest"></a>

### CreateStoreRequest

| Ad                               | Açıklama                                                                                                                                                      | Şema                           |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| **name** <br>_zorunlu_           | Mağaza adı                                                                                                                                                    | string                         |
| **url** <br>_opsiyonel_          | Url'i üzerinden tanım yapılan mağazalar için store url bilgisidir.                                                                                            | string                         |
| **storeId** <br>_opsiyonel_      | Mağazanın tekil Id'sidir. Gönderilmediği takdirde Navlungo'da yaratılan tekil id geri döndürülecektir. Mağaza için iletişim bu id üzerinden gerçekleşecektir. | string                         |
| **storeAddress** <br>_zorunlu_   | Mağaza adresi bilgisidir.                                                                                                                                     | < [Address](#address) > object |
| **invoiceAddress** <br>_zorunlu_ | Mağaza'nın gönderi oluştururken kullanılacak fatura adresidir.                                                                                                | < [Address](#address) > object |

<a name="address"></a>

### Address

Mağazanın adres bilgilerinin modelidir.

| Ad                                     | Açıklama                                                                                      | Şema   |
| -------------------------------------- | --------------------------------------------------------------------------------------------- | ------ |
| **type** <br>_zorunlu_                 | Adres bireysel mi kurumsal mı olduğunun bilgisidir. Bireysel: Individual, Kurumsal: Corporate | string |
| **companyName** <br>_opsiyonel_        | Kurumsal adres için şirket ismi                                                               | string |
| **identificationNumber** <br>_zorunlu_ | Vergi kimlik numarası veya TC Kimlik numarası                                                 | string |
| **taxOffice** <br>_opsiyonel_          | Vergi dairesi(kurumsal adresler için zorunlu)                                                 | string |
| **contactName** <br>_zorunlu_          | Kontak kişi adı                                                                               | string |
| **contactPhone** <br>_zorunlu_         | Kontak kişinin telefon numarası                                                               | string |
| **contactMail** <br>_zorunlu_          | Kontak kişinin email adresi                                                                   | string |
| **countryCode** <br>_zorunlu_          | 2 haneli global ülke kodu                                                                     | string |
| **state** <br>_opsiyonel_              | Global eyalet kodu                                                                            | string |
| **city** <br>_zorunlu_                 | [Şehir bilgisi](./city-town.md)                                                               | string |
| **town** <br>_zorunlu_                 | [İlçe bilgisi](./city-town.md)                                                                | string |
| **postalCode** <br>_zorunlu_           | Posta kodu                                                                                    | string |
| **firstLine** <br>_zorunlu_            | İlk adres satırı. Minimum 10 karakter, maximum 30 karakter.                                   | string |
| **secondLine** <br>_opsiyonel_         | İkinci adres satırı.Maximum 30 karakter.                                                      | string |
| **thirdLine** <br>_opsiyonel_          | Üçüncü adres satırı. Maximum 30 karakter.                                                     | string |

<a name="createStoreResponse"></a>

### createStoreResponse

| Ad                        | Açıklama                         | Şema   |
| ------------------------- | -------------------------------- | ------ |
| **storeId** <br>_zorunlu_ | Yaratılan mağazanın tekil Id'si. | string |

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

<hr/>
<a name="getStores"></a>

### GET stores/v1

**Operasyon: getStores**

#### Açıklama

Kullanıcıya ait tüm mağazaları getirir. Mağaza yoksa geriye boş dizi döner.

#### Parametreler

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema                                    |
| --------- | --------------------------------------------------------------- | --------------------------------------- |
| **200**   | Başarılı                                                        | [GetStoresResponse](#GetStoresResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error)                         |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error)                         |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error)                         |

<a name="definitions"></a>

## Tanımlar

<a name="GetStoresResponse"></a>

### GetStoresResponse

| Ad                       | Açıklama       | Şema                      |
| ------------------------ | -------------- | ------------------------- |
| **stores** <br>_zorunlu_ | Mağaza listesi | < [Store](#store) > array |

<a name="store"></a>

### Store

| Ad                                 | Açıklama                  | Şema                         |
| ---------------------------------- | ------------------------- | ---------------------------- |
| **storeId** <br>_zorunlu_          | Mağaza'nın tekil id'si    | string                       |
| **name** <br>_zorunlu_             | Mağaza'nın adı            | string                       |
| **url** <br>_opsiyonel_            | Mağaza'nın url bilgisi    | string                       |
| **storeAddress** <br>_opsiyonel_   | Mağazanın gönderim adresi | <[Address](#address)> object |
| **invoiceAddress** <br>_opsiyonel_ | Mağazanın fatura adresi   | <[Address](#address)> object |

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

<hr/>
<a name="getStore"></a>

### GET /stores/v1/{id}

**Operasyon: getStore**

#### Açıklama

Kullanıcıya ait tüm mağazalardan talep edilen mağazanın detaylarını getirir.

#### Parametreler

| Ad        | Açıklama               | Şema                  |
| --------- | ---------------------- | --------------------- | ------ |
| **Query** | **{id}** <br>_zorunlu_ | Mağazanın tekil id'si | string |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema                                  |
| --------- | --------------------------------------------------------------- | ------------------------------------- |
| **200**   | Başarılı                                                        | [GetStoreResponse](#GetStoreResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error)                       |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error)                       |
| **404**   | Aranan mağaza bulunamadı.                                       | [Error](#error)                       |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error)                       |

<a name="definitions"></a>

## Tanımlar

<a name="GetStoreResponse"></a>

### GetStoreResponse

| Ad                      | Açıklama         | Şema                       |
| ----------------------- | ---------------- | -------------------------- |
| **store** <br>_zorunlu_ | Mağaza bilgileri | < [Store](#store) > object |

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

<hr/>
<a name="updateStore"></a>

### PATCH /stores/v1/{id}

**Operasyon: updateStore**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın bilgilerini günceller.

#### Parametreler

| Tip       | İsim                   | Açıklama              | Şema   |
| --------- | ---------------------- | --------------------- | ------ |
| **Query** | **{id}** <br>_zorunlu_ | Mağazanın tekil id'si | string |
| **Body**  | **Name** <br>_zorunlu_ | Mağaza adı            | string |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
| **404**   | Mağaza bulunamadı                                               | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error) |

<a name="definitions"></a>

## Tanımlar

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

<hr/>
<a name="updateStoreAddress"></a>

### PUT /stores/v1/{id}/storeAddress

**Operasyon: updateStoreAddress**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın adres bilgilerini günceller.

#### Parametreler

| Tip       | İsim                      | Açıklama              | Şema                         |
| --------- | ------------------------- | --------------------- | ---------------------------- |
| **Query** | **{id}** <br>_zorunlu_    | Mağazanın tekil id'si | string                       |
| **Body**  | **address** <br>_zorunlu_ | Mağazanın yeni adresi | <[Address](#address)> object |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
| **404**   | Mağaza bulunamadı                                               | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error) |

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

<hr/>
<a name="updateInvoiceAddress"></a>

### PUT /stores/v1/{id}/invoiceAddress

**Operasyon: updateInvoiceAddress**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın fatura adres bilgilerini günceller.

#### Parametreler

| Tip       | İsim                      | Açıklama              | Şema                         |
| --------- | ------------------------- | --------------------- | ---------------------------- |
| **Query** | **{id}** <br>_zorunlu_    | Mağazanın tekil id'si | string                       |
| **Body**  | **address** <br>_zorunlu_ | Mağazanın yeni adresi | <[Address](#address)> object |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
| **404**   | Mağaza bulunamadı                                               | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error) |

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

<hr/>
<a name="deleteStore"></a>

### DELETE /stores/v1/{id}

**Operasyon: deleteStore**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazayı pasife çeker.

#### Parametreler

| Tip       | İsim                   | Açıklama              | Şema   |
| --------- | ---------------------- | --------------------- | ------ |
| **Query** | **{id}** <br>_zorunlu_ | Mağazanın tekil id'si | string |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error) |

>

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

<hr/>
<a name="shipStoreOrder"></a>

### POST /stores/v2/{store_id}/orders/{order_reference}/ship

**Operasyon: shipStoreOrder**

#### Açıklama

Belirtilen mağaza ve sipariş için seçilen teklif üzerinden sevkiyat işlemlerini başlatır.

#### Parametreler

| Tip      | İsim                              | Açıklama           | Şema                                          |
| -------- | --------------------------------- | ------------------ | --------------------------------------------- |
| **Path** | **store_id** <br>_zorunlu_        | Mağaza ID          | Guid                                          |
| **Path** | **order_reference** <br>_zorunlu_ | Sipariş referansı  | string                                        |
| **Body** | **body** <br>_zorunlu_            | Sevkiyat bilgileri | [StoreShipmentRequest](#storeShipmentRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                                    |
| --------- | -------------------------------------------------------------- | --------------------------------------- |
| **200**   | Başarılı                                                       | [ShipOrderResponse](#shipOrderResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.          | [Error](#error)                         |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                         |
| **404**   | Mağaza veya sipariş bulunamadı                                 | [Error](#error)                         |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                   | [Error](#error)                         |

<a name="definitions"></a>

## Tanımlar

<a name="storeShipmentRequest"></a>

### StoreShipmentRequest

| Ad                               | Açıklama                           | Şema |
| -------------------------------- | ---------------------------------- | ---- |
| **quoteReference** <br>_zorunlu_ | Teklifin tekil id'si               | Guid |
| **searchId** <br>_zorunlu_       | Teklif arama işleminin tekil id'si | Guid |

<a name="shipOrderResponse"></a>

### ShipOrderResponse

| Ad                                  | Açıklama                                                                                    | Şema         |
| ----------------------------------- | ------------------------------------------------------------------------------------------- | ------------ |
| **shipmentId** <br>_zorunlu_        | Sevkiyatın tekil tanımlayıcısı                                                              | Guid         |
| **shipmentReference** <br>_zorunlu_ | Gönderi numarası                                                                            | string       |
| **trackingUrl** <br>_zorunlu_       | Gönderi takip bağlantısı                                                                    | string       |
| **cargoLabels** <br>_zorunlu_       | Depo'da kargo kabul için gerekli olan barkod numara listesi. Paket başına bir numara döner. | List<string> |

<a name="trackOrder"></a>

### GET /stores/v1/{id}/ordertracking?orderReference={orderId}

#### Açıklama

Bir siparişe ait varsa takip bilgisini geri döndürür. Sipariş ve mağza bilgileri hatalı girilirse 403, siparişe ait takip bilgisi henüz oluşmamışsa 404 döndürür.

#### Parametreler

| Tip             | İsim                             | Açıklama   | Şema   |
| --------------- | -------------------------------- | ---------- | ------ |
| **Query**       | **id** <br>_zorunlu_             | Mağaza id  | string |
| **QueryString** | **orderReference** <br>_zorunlu_ | Sipariş id | string |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                                  |
| --------- | -------------------------------------------------------------- | ------------------------------------- |
| **200**   | Başarılı                                                       | [TrackingResponse](#trackingResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.          | [Error](#error)                       |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                       |
| **403**   | Mağaza veya sipariş bilgisi hatalı                             | [Error](#error)                       |
| **403**   | Takip numarası henüz oluşmamış                                 | [Error](#error)                       |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                   | [Error](#error)                       |

<a name="definitions"></a>

## Tanımlar

<a name="trackingResponse"></a>

### TrackingResponse

| Ad                               | Açıklama                      | Şema                                       |
| -------------------------------- | ----------------------------- | ------------------------------------------ |
| **trackingNumber** <br>_zorunlu_ | Takip numarası                | string                                     |
| **courier** <br>_zorunlu_        | Taşıyıcı firma kodu           | string                                     |
| **courierTrackingLink**          | Taşıyıcı firmanın takip linki | string                                     |
| **status** <br>_zorunlu_         | Sevkiyatın durumu             | < [TrackingStatus](#trackingStatus) > Enum |
| **signedBy**                     | Varsa teslim alan kişi        | string                                     |
| **shipmentType**                 | Taşıyıcı firmanın servis türü | string                                     |
| **checkpoints**                  | Sevkiyata dair hareketler     | < [Checkpoint](#checkpoint) > Array        |

<a name="checkpoint"></a>

### Checkpoint

| Ad                               | Açıklama                          | Şema                                       |
| -------------------------------- | --------------------------------- | ------------------------------------------ |
| **checkpointTime** <br>_zorunlu_ | Olay zamanı                       | string                                     |
| **status** <br>_zorunlu_         | Olay sırasında sevkiyatın statüsü | < [TrackingStatus](#trackingStatus) > Enum |
| **message**                      | Olaya dair mesaj                  | string                                     |

<a name="trackingStatus"></a>

### TrackingStatus

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
