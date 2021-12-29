# Express Taşımalar İçin Arama Apisi

<a name="overview"></a>

## Genel Bakış

Navlungo, faturalanabilir ağırlığı 300kg'a kadar olan yükler için express hizmet teklifi üretebilmektedir. Bu API ile Navlungo çözüm ortaklarına express teklif arama süreçleri için erişim sağlanacaktır.

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

[createSearch](#searches)<br>
[createStore](#createStore)<br>
[getStore](#getStore)<br>
[updateStore](#updateStore)<br>
[updateStoreAddress](#updateStoreAddress)<br>

<a name="paths"></a>

## Paths

<a name="searches"></a>

- ### POST /v1/searches

**Operasyon: createSearch**

#### Açıklama

Gönderilen yük bilgileri ve taşımanın yapılacağı ülkelere göre bir arama kaydı oluşturur ve yapılan arama sonucunda oluşturulan teklifleri listeler.

#### Parametreler

| Tip      | İsim                    | Açıklama                         | Şema                                        |
| -------- | ----------------------- | -------------------------------- | ------------------------------------------- |
| **Body** | **body** <br>_required_ | Arama yaratmak için gerekli şema | [CreateSearchRequest](#createSearchRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                       | Şema                              |
| --------- | -------------------------------------------------------------- | --------------------------------- |
| **200**   | Başarılı                                                       | [SearchResponse](#searchResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.          | [Error](#error)                   |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş | [Error](#error)                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                   | [Error](#error)                   |

<a name="definitions"></a>

## Tanımlar

<a name="createSearchRequest"></a>

### CreateSearchRequest

| Ad                                   | Açıklama                                                                                                                                                               | Şema                    |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| **originCountry** <br>_zorunlu_      | Yükün çıkışının yapılacağı ülkenin ISO2 kodu                                                                                                                           | string                  |
| **originRegion** <br>_opsiyonel_     | Yükün çıkışının yapılcağı bölgenin kodu. Tercihen posta kodu gönderilmeli. Bu alanın doğru gönderilmemesi durumunda fiyatlara ek iç taşıma maliyetleri yansıtılabilir. | string                  |
| **destinationCountry** <br>_zorunlu_ | Yükün gönderileceği ülkenin ISO2 kodu                                                                                                                                  | string                  |
| **loads** <br>_zorunlu_              | Taşıması yapılacak yüklere ait paket bilgileri                                                                                                                         | < [Load](#load) > array |

<a name="load"></a>

### Load

Taşıması yapılacak yüke ait paketleme bilgisi.

| Ad                         | Açıklama                                                                     | Şema    |
| -------------------------- | ---------------------------------------------------------------------------- | ------- |
| **count** <br>_zorunlu_    | Paket adedi. Envelope tipindeki paketler için yalnızca 1 olabilir.           | string  |
| **type** <br>_zorunlu_     | Paket türü. Box veya Envelope olmalı. (büyük küçük harf duyarlı)             | string  |
| **weight** <br>_zorunlu_   | Paketin ağırlığı. Envelope tipindeki paketler için en fazla 2.5 kg olabilir. | decimal |
| **width** <br>_opsiyonel_  | Paketin eni. Box tipindeki paketlerde bu değer zorunludur.                   | decimal |
| **length** <br>_opsiyonel_ | Paketin boyu. Box tipindeki paketlerde bu değer zorunludur.                  | decimal |
| **height** <br>_opsiyonel_ | Paketin yüksekliği. Box tipindeki paketlerde bu değer zorunludur.            | decimal |

<a name="searchResponse"></a>

### SearchResponse

| Ad                                | Açıklama                                                                    | Şema                      |
| --------------------------------- | --------------------------------------------------------------------------- | ------------------------- |
| **searchReference** <br>_zorunlu_ | Aramanın referans id'si                                                     | string                    |
| **quotes** <br>_opsiyonel_        | Arama sonucunda üretilen teklifler. Hiç teklif üretilemezse boş dizi döner. | < [Quote](#quote) > array |

<a name="quote"></a>

### Quote

Express aramaya üretilen teklif bilgisi

| Ad                                 | Açıklama                            | Şema    |
| ---------------------------------- | ----------------------------------- | ------- |
| **quoteReference** <br>_zorunlu_   | Teklifin referans id'si             | string  |
| **price** <br>_zorunlu_            | Teklif tutarı                       | decimal |
| **currency** <br>_zorunlu_         | Teklif kuru                         | string  |
| **proposal** <br>_zorunlu_         | Teklif türü. ( express, ecoExpress) | string  |
| **minTransitTime** <br>_opsiyonel_ | Minimum taşıma süresi(gün)          | int     |
| **maxTransitTime** <br>_opsiyonel_ | Maksimum taşıma süresi(gün)         | int     |
| **description** <br>_zorunlu_      | Teklif açıklaması                   | string  |
| **isDoorToDoor** <br>_zorunlu_     | Kapıdan kapıya mı?                  | bool    |

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

<hr/>
<a name="createStore"></a>

- ### POST /v1/stores

**Operasyon: createStore**

#### Açıklama

İşlem yapılan kullanıcıya ait bir mağaza kaydı oluşturur ve mağazanın varsayılan fatura ile gönderim adres bilgilerini yaratır.

#### Parametreler

| Tip      | İsim                    | Açıklama                          | Şema                                      |
| -------- | ----------------------- | --------------------------------- | ----------------------------------------- |
| **Body** | **body** <br>_required_ | Mağaza yaratmak için gerekli şema | [CreateStoreRequest](#createStoreRequest) |

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
| **storeId** <br>_optional_       | Mağazanın tekil Id'sidir. Gönderilmediği takdirde Navlungo'da yaratılan tekil id geri döndürülecektir. Mağaza için iletişim bu id üzerinden gerçekleşecektir. | string                         |
| **storeAddress** <br>_zorunlu_   | Mağaza adresi bilgisidir.                                                                                                                                     | < [Address](#address) > object |
| **invoiceAddress** <br>_zorunlu_ | Mağaza'nın gönderi oluştururken kullanılacak fatura adresidir.                                                                                                | < [Address](#address) > object |

<a name="address"></a>

### Address

Mağazanın adres bilgilerinin modelidir.

| Ad                                      | Açıklama                                                                                      | Şema   |
| --------------------------------------- | --------------------------------------------------------------------------------------------- | ------ |
| **type** <br>_zorunlu_                  | Adres bireysel mi kurumsal mı olduğunun bilgisidir. Bireysel: Individual, Kurumsal: Corporate | string |
| **companyName** <br>_optional_          | Kurumsal adres için şirket ismi                                                               | string |
| **identificationNumber** <br>_required_ | Veri kimlik numarası veya TC Kimlik numarası                                                  | string |
| **taxOffice** <br>_opsiyonel_           | Vergi dairesi                                                                                 | string |
| **contactName** <br>_zorunlu_           | Kontak kişi adı                                                                               | string |
| **contactPhone** <br>_zorunlu_          | Kontak kişinin telefon numarası                                                               | string |
| **contactMail** <br>_zorunlu_           | Kontak kişinin email adresi                                                                   | string |
| **countryCode** <br>_zorunlu_           | 2 haneli global ülke kodu                                                                     | string |
| **state** <br>_optional_                | Global eyalet kodu                                                                            | string |
| **city** <br>_zorunlu_                  | Şehir bilgisi                                                                                 | string |
| **town** <br>_zorunlu_                  | İlçe bilgisi                                                                                  | string |
| **postalCode** <br>_zorunlu_            | Posta kodu                                                                                    | string |
| **firstLine** <br>_zorunlu_             | İlk adres satırı. Minimum 10 karakter, maximum 30 karakter.                                   | string |
| **secondLine** <br>_optional_           | İkinci adres satırı.Maximum 30 karakter.                                                      | string |
| **thirdLine** <br>_optional_            | Üçüncü adres satırı. Maximum 30 karakter.                                                     | string |
| **thirdLine** <br>_zorunlu_             | Üçüncü adres satırı                                                                           | string |

<a name="createStoreResponse"></a>

### createStoreResponse

| Ad                        | Açıklama                         | Şema   |
| ------------------------- | -------------------------------- | ------ |
| **storeId** <br>_zorunlu_ | Yaratılan mağazanın tekil Id'si. | string |

<a name="store"></a>

### Store

| Ad                              | Açıklama               | Şema                         |
| ------------------------------- | ---------------------- | ---------------------------- |
| **storeId** <br>_zorunlu_       | Mağaza'nın tekil id'si | string                       |
| **name** <br>_zorunlu_          | Mağaza'nın adı         | string                       |
| **url** <br>_optional_          | Mağaza'nın url bilgisi | string                       |
| **storeType** <br>_optional_    | Çözüm ortağı           | string                       |
| **storeAddress** <br>_optional_ | Çözüm ortağı           | <[Address](#address)> object |

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

- ### GET /v1/stores

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

- ### GET /v1/stores/{id}

**Operasyon: getStore**

#### Açıklama

Kullanıcıya ait tüm mağazalardan talep edilen mağazanın detaylarını getirir.

#### Parametreler

| Ad        | Açıklama                | Şema                  |
| --------- | ----------------------- | --------------------- | ------ |
| **Query** | **{id}** <br>_required_ | Mağazanın tekil id'si | string |

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

- ### PUT /v1/stores/{id}

**Operasyon: updateStore**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın bilgilerini günceller.

#### Parametreler

| Tip       | İsim                    | Açıklama              | Şema   |
| --------- | ----------------------- | --------------------- | ------ |
| **Query** | **{id}** <br>_required_ | Mağazanın tekil id'si | string |
| **Body**  | **Name** <br>_required_ | Mağaza adı            | string |

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

- ### PUT /v1/stores/{id}

**Operasyon: updateStoreAddress**

#### Açıklama

Query parametresi olarak verilen id'ye ait mağazanın bilgilerini günceller.

#### Parametreler

| Tip       | İsim                    | Açıklama              | Şema   |
| --------- | ----------------------- | --------------------- | ------ |
| **Query** | **{id}** <br>_required_ | Mağazanın tekil id'si | string |
| **Body**  | **Name** <br>_required_ | Mağaza adı            | string |

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
