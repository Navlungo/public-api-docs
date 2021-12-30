# Marketplace Entegrasyonu için Store Api'leri

<a name="overview"></a>

## Genel Bakış

Navlungo kullanıcıları mağazalarını bağlarak siparişlerini Navlungo arayüzlerinde görüntüleyebilmektedir. Bu Api'ler ile sisteme kayıt olmuş bir kullanıcı arayüze girmeden mağazasını bağlayabilir ve mağaza siparişlerinde kullanılacak olan varsayılan adreslerini belirleyebilir.

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

- [createStore](#createStore)<br>
- [getStores](#getStores)<br>
- [getStore](#getStore)<br>
- [updateStore](#updateStore)<br>
- [updateStoreAddress](#updateStoreAddress)<br>
- [updateInvoiceAddress](#updateInvoiceAddress)<br>
- [deleteStore](#deleteStore)<br>

<a name="paths"></a>

## Paths

<a name="createStore"></a>

### POST /v1/stores

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
| **name** <br>_zorunlu_           | Yükün çıkışının yapılacağı ülkenin ISO2 kodu                                                                                                                  | string                         |
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
| **identificationNumber** <br>_zorunlu_ | Veri kimlik numarası veya TC Kimlik numarası                                                  | string |
| **taxOffice** <br>_opsiyonel_          | Vergi dairesi                                                                                 | string |
| **contactName** <br>_zorunlu_          | Kontak kişi adı                                                                               | string |
| **contactPhone** <br>_zorunlu_         | Kontak kişinin telefon numarası                                                               | string |
| **contactMail** <br>_zorunlu_          | Kontak kişinin email adresi                                                                   | string |
| **countryCode** <br>_zorunlu_          | 2 haneli global ülke kodu                                                                     | string |
| **state** <br>_opsiyonel_              | Global eyalet kodu                                                                            | string |
| **city** <br>_zorunlu_                 | Şehir bilgisi                                                                                 | string |
| **town** <br>_zorunlu_                 | İlçe bilgisi                                                                                  | string |
| **postalCode** <br>_zorunlu_           | Posta kodu                                                                                    | string |
| **firstLine** <br>_zorunlu_            | İlk adres satırı. Minimum 10 karakter, maximum 30 karakter.                                   | string |
| **secondLine** <br>_opsiyonel_         | İkinci adres satırı.Maximum 30 karakter.                                                      | string |
| **thirdLine** <br>_opsiyonel_          | Üçüncü adres satırı. Maximum 30 karakter.                                                     | string |
| **thirdLine** <br>_zorunlu_            | Üçüncü adres satırı                                                                           | string |

<a name="createStoreResponse"></a>

### createStoreResponse

| Ad                        | Açıklama                         | Şema   |
| ------------------------- | -------------------------------- | ------ |
| **storeId** <br>_zorunlu_ | Yaratılan mağazanın tekil Id'si. | string |

<a name="store"></a>

### Store

| Ad                               | Açıklama               | Şema                         |
| -------------------------------- | ---------------------- | ---------------------------- |
| **storeId** <br>_zorunlu_        | Mağaza'nın tekil id'si | string                       |
| **name** <br>_zorunlu_           | Mağaza'nın adı         | string                       |
| **url** <br>_opsiyonel_          | Mağaza'nın url bilgisi | string                       |
| **storeType** <br>_opsiyonel_    | Çözüm ortağı           | string                       |
| **storeAddress** <br>_opsiyonel_ | Çözüm ortağı           | <[Address](#address)> object |

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

### GET /v1/stores

**Operasyon: getStores**

#### Açıklama

Kullanıcıya ait tüm mağazaları getirir.

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

### GET /v1/stores/{id}

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

### PUT /v1/stores/{id}

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

### PUT /v1/stores/{id}/storeAddress

**Operasyon: updateStoreAddress**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın adres bilgilerini günceller.

#### Parametreler

| Tip       | İsim                      | Açıklama              | Şema                         |
| --------- | ------------------------- | --------------------- | ---------------------------- |
| **Query** | **{id}** <br>_zorunlu_    | Mağazanın tekil id'si | string                       |
| **Body**  | **address** <br>_zorunlu_ | Mağaza adı            | <[Address](#address)> object |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
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

### PUT /v1/stores/{id}/invoiceAddress

**Operasyon: updateInvoiceAddress**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın fatura adres bilgilerini günceller.

#### Parametreler

| Tip       | İsim                      | Açıklama              | Şema                         |
| --------- | ------------------------- | --------------------- | ---------------------------- |
| **Query** | **{id}** <br>_zorunlu_    | Mağazanın tekil id'si | string                       |
| **Body**  | **address** <br>_zorunlu_ | Mağaza adı            | <[Address](#address)> object |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
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

### DELETE /v1/stores/{id}

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