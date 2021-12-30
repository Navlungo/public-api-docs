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

<a name="paths"></a>

## Paths

<a name="searches"></a>

### POST /v1/searches

**Operasyon: createSearch**

#### Açıklama

Gönderilen yük bilgileri ve taşımanın yapılacağı ülkelere göre bir arama kaydı oluşturur ve yapılan arama sonucunda oluşturulan teklifleri listeler.

#### Parametreler

| Tip      | İsim                   | Açıklama                         | Şema                                        |
| -------- | ---------------------- | -------------------------------- | ------------------------------------------- |
| **Body** | **body** <br>_zorunlu_ | Arama yaratmak için gerekli şema | [CreateSearchRequest](#createSearchRequest) |

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
