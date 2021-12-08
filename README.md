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

| Tip      | İsim                    | Açıklama                         | Şema                                        |
| -------- | ----------------------- | -------------------------------- | ------------------------------------------- |
| **Body** | **body** <br>_required_ | Arama yaratmak için gerekli şema | [CreateSearchRequest](#createSearchRequest) |

<a name="definitions"></a>

## Tanımlar

<a name="createSearchRequest"></a>
###CreateSearchRequest

| Ad                                   | Açıklama                                                                                                                                                               | Şema                    |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| **originCountry** <br>_zorunlu_      | Yükün çıkışının yapılacağı ülkenin ISO2 kodu                                                                                                                           | string                  |
| **originRegion** <br>_opsiyonel_     | Yükün çıkışının yapılcağı bölgenin kodu. Tercihen posta kodu gönderilmeli. Bu alanın doğru gönderilmemesi durumunda fiyatlara ek iç taşıma maliyetleri yansıtılabilir. | string                  |
| **destinationCountry** <br>_zorunlu_ | Yükün gönderileceği ülkenin ISO2 kodu                                                                                                                                  | string                  |
| **loads** <br>_zorunlu_              | Taşıması yapılacak yüklere ait paket bilgileri                                                                                                                         | < [Load](#load) > array |

<a name="load"></a>
###Load
Taşıması yapılacak yüke ait paketleme bilgisi.

| Ad                         | Açıklama                                                                     | Şema    |
| -------------------------- | ---------------------------------------------------------------------------- | ------- |
| **count** <br>_zorunlu_    | Paket adedi. Envelope tipindeki paketler için yalnızca 1 olabilir.           | string  |
| **type** <br>_zorunlu_     | Paket türü. Box veya Envelope olmalı. (büyük küçük harf duyarlı)             | string  |
| **weight** <br>_zorunlu_   | Paketin ağırlığı. Envelope tipindeki paketler için en fazla 2.5 kg olabilir. | decimal |
| **width** <br>_opsiyonel_  | Paketin eni. Box tipindeki paketlerde bu değer zorunludur.                   | decimal |
| **length** <br>_opsiyonel_ | Paketin boyu. Box tipindeki paketlerde bu değer zorunludur.                  | decimal |
| **height** <br>_opsiyonel_ | Paketin yüksekliği. Box tipindeki paketlerde bu değer zorunludur.            | decimal |
