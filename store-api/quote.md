# Express Teklif Apisi

<a name="overview"></a>

## Genel Bakış

Navlungo, faturalanabilir ağırlığı 300kg'a kadar olan yükler için express hizmet teklifi üretebilmektedir. Bu API ile Navlungo çözüm ortaklarına express teklif arama süreçleri için erişim sağlanacaktır.

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

[createOrderQuote](#createOrderQuote)<br>

<a name="paths"></a>

## Paths

<a name="createOrderQuote"></a>

### POST stores/v2/{store_id}/orders

**Operasyon: createOrderQuote**

#### Açıklama

Belirtilen mağaza için sipariş bilgileri ve paket detayları kullanılarak teklif oluşturur.

#### Parametreler

| Tip      | İsim                       | Açıklama                          | Şema                                    |
| -------- | -------------------------- | --------------------------------- | --------------------------------------- |
| **Path** | **store_id** <br>_zorunlu_ | Mağaza kimlik bilgisi             | string                                  |
| **Body** | **body** <br>_zorunlu_     | Teklif yaratmak için gerekli şema | [OrderQuoteRequest](#orderQuoteRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Şema                                      |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **200**   | Başarılı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | [OrderQuoteResponse](#orderQuoteResponse) |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz. Ürün tanımlayıcısı hataları ([AB Ürün Tanımlayıcıları](#euProductIdentifiers)):<br> `producterrors.merchantproductidentifierrequiredforeuropeanunion.error`, `producterrors.nonstandardisedmanufacturerproductidentifierrequiredforeuropeanunion.error` - Alıcı AB üye ülkesinde, sipariş toplamı 150 EUR veya altında ve ürün tanımlayıcısı eksik<br> `Common/Validation` - Ürün tanımlayıcısı 100 karakteri aşıyor (`extensions.validationErrors`) | [Error](#error)                           |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş                                                                                                                                                                                                                                                                                                                                                                                                                                             | [Error](#error)                           |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [Error](#error)                           |

<a name="definitions"></a>

## Tanımlar

<a name="generateQuoteRequest"></a>

### GenerateQuoteRequest

| Ad                                   | Açıklama                                                                                                                                                                | Şema                    |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| **originCountry** <br>_zorunlu_      | Yükün çıkışının yapılacağı ülkenin ISO2 kodu                                                                                                                            | string                  |
| **originRegion** <br>_opsiyonel_     | Yükün çıkışının yapılacağı bölgenin kodu. Tercihen posta kodu gönderilmeli. Bu alanın doğru gönderilmemesi durumunda fiyatlara ek iç taşıma maliyetleri yansıtılabilir. | string                  |
| **destinationCountry** <br>_zorunlu_ | Yükün gönderileceği ülkenin ISO2 kodu                                                                                                                                   | string                  |
| **loads** <br>_zorunlu_              | Taşıması yapılacak yüklere ait paket bilgileri                                                                                                                          | < [Load](#load) > array |

<a name="orderQuoteRequest"></a>

### OrderQuoteRequest

| Ad                             | Açıklama                                                     | Şema                          |
| ------------------------------ | ------------------------------------------------------------ | ----------------------------- |
| **order** <br>_zorunlu_        | Sipariş bilgileri                                            | [Order](#order)               |
| **packages** <br>_zorunlu_     | Taşıması yapılacak paket bilgileri                           | < [Package](#package) > array |
| **shipmentType** <br>_zorunlu_ | Taşıma türü. Enum: "sales", "sample", "micro-export", "gift" | string                        |

<a name="order"></a>

### Order

| Ad                                    | Açıklama                     | Şema                                |
| ------------------------------------- | ---------------------------- | ----------------------------------- |
| **orderReference** <br>_zorunlu_      | Sipariş referans kodu        | string                              |
| **currencyCode** <br>_zorunlu_        | Para birimi kodu             | string                              |
| **receiverAddress** <br>_zorunlu_     | Alıcı adresi bilgileri       | [ReceiverAddress](#receiverAddress) |
| **receiverEmail** <br>_opsiyonel_     | Alıcı e-posta adresi         | string                              |
| **receiverPhoneNumber** <br>_zorunlu_ | Alıcı telefon numarası       | string                              |
| **orderItems** <br>_zorunlu_          | Sipariş içerisindeki ürünler | < [OrderItem](#orderItem) > array   |

<a name="receiverAddress"></a>

### ReceiverAddress

| Ad                            | Açıklama          | Şema   |
| ----------------------------- | ----------------- | ------ |
| **contactName** <br>_zorunlu_ | Alıcı kişinin adı | string |
| **countryCode** <br>_zorunlu_ | Ülke kodu (ISO2)  | string |
| **state** <br>_opsiyonel_     | Eyalet            | string |
| **town** <br>_zorunlu_        | İlçe              | string |
| **city** <br>_zorunlu_        | Şehir             | string |
| **postalCode** <br>_zorunlu_  | Posta kodu        | string |
| **firstLine** <br>_zorunlu_   | Adres satırı      | string |

<a name="orderItem"></a>

### OrderItem

| Ad                                                               | Açıklama                                                                                                                                                                                                                                                                                   | Şema   |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ |
| **quantity** <br>_zorunlu_                                       | Ürün adedi                                                                                                                                                                                                                                                                                 | int    |
| **price** <br>_zorunlu_                                          | Ürün fiyatı                                                                                                                                                                                                                                                                                | string |
| **description** <br>_zorunlu_                                    | Ürün açıklaması                                                                                                                                                                                                                                                                            | string |
| **sku** <br>_zorunlu_                                            | Stok Kodu. `merchantProductIdentifier` gönderilmezse ve en fazla 100 karakterse satıcı ürün tanımlayıcısı olarak kullanılır                                                                                                                                                                | string |
| **hsCode** <br>_zorunlu_                                         | Ürün HS kodu                                                                                                                                                                                                                                                                               | string |
| **merchantProductIdentifier** <br>_opsiyonel_                    | Satıcı ürün tanımlayıcısı (Merchant PID), genellikle satıcının SKU'su. Alıcı AB üye ülkesinde ve sipariş toplamı 150 EUR veya altındaysa zorunlu (`document` paketler hariç), en fazla 100 karakter. Gönderilmezse `sku` kullanılır. Bkz. [AB Ürün Tanımlayıcıları](#euProductIdentifiers) | string |
| **nonStandardisedManufacturerProductIdentifier** <br>_opsiyonel_ | Üreticinin standart dışı ürün tanımlayıcısı (NS-PID), örneğin üreticinin model veya parça numarası. Alıcı AB üye ülkesinde ve sipariş toplamı 150 EUR veya altındaysa zorunlu (`document` paketler hariç), en fazla 100 karakter. Bkz. [AB Ürün Tanımlayıcıları](#euProductIdentifiers)    | string |
| **standardisedManufacturerProductIdentifier** <br>_opsiyonel_    | Standart ürün tanımlayıcısı (S-PID): GTIN, EAN, UPC veya ISBN, en fazla 100 karakter. Ürünün standart tanımlayıcısı varsa gönderilmelidir                                                                                                                                                  | string |

<a name="euProductIdentifiers"></a>

#### AB Ürün Tanımlayıcıları (PID)

AB gümrük düzenlemesi gereği 1 Kasım 2026'dan itibaren AB'ye giden gönderilerde gümrük beyanındaki her ürün için ürün tanımlayıcıları gerekir. Navlungo bu kuralı alıcı ülkesi (`order.receiverAddress.countryCode`) AB üyesi olan ve ürünlerin toplam tutarı (`quantity` × `price`) EUR karşılığı **150 EUR veya altında** olan siparişlerde uygular. `order.currencyCode` EUR değilse tutar, gönderinin oluşturulduğu günün kuruyla EUR'a çevrilir. Kontrol bu istekte, gönderi oluşturulurken yapılır; eksik tanımlayıcı varsa gönderi ve teklif oluşturulmaz.

- `merchantProductIdentifier` ve `nonStandardisedManufacturerProductIdentifier` her ürün için zorunludur. Boş ya da yalnızca boşluk içeren değer eksik sayılır; istek `400` ile `producterrors.merchantproductidentifierrequiredforeuropeanunion.error` veya `producterrors.nonstandardisedmanufacturerproductidentifierrequiredforeuropeanunion.error` döner.
- `merchantProductIdentifier` gönderilmezse ürünün `sku` değeri (en fazla 100 karakterse) satıcı ürün tanımlayıcısı olarak kullanılır. SKU'su dolu ürünlerde yalnızca `nonStandardisedManufacturerProductIdentifier` eklemek yeterlidir.
- `document` tipindeki paketlerde bu alanlar zorunlu değildir.
- `standardisedManufacturerProductIdentifier` zorunlu değildir; ürünün GTIN, EAN, UPC veya ISBN kodu varsa gönderilmelidir.
- Değerlerin başındaki ve sonundaki boşluklar kırpılır. Üç alan da en fazla 100 karakter olabilir; aşan değerde istek `400` ile `Common/Validation` döner ve alan `extensions.validationErrors` içinde belirtilir.
- Toplamı 150 EUR'yu aşan AB siparişlerinde ve AB dışındaki alıcılarda alanlar opsiyoneldir.
- AB gönderilerinde girilen değerler tutardan bağımsız olarak taşıyıcıya iletilir. Bazı taşıyıcılar 150 EUR üzerindeki gönderilerde de bu bilgileri isteyebileceği için AB gönderilerinde tutardan bağımsız olarak gönderilmesi önerilir.

Alıcısı AB üye ülkesinde ve toplamı 150 EUR veya altında olan (`currencyCode` `EUR`, 2 × 25 EUR = 50 EUR) bir sipariş için örnek ürün:

```
"orderItems": [
  {
    "sku": "TS-COTTON-M-WHT", "description": "Cotton t-shirt, short sleeve", "quantity": 2, "price": 25, "hsCode": "610910000000",
    "merchantProductIdentifier": "TS-COTTON-M-WHT",
    "nonStandardisedManufacturerProductIdentifier": "YT-6109-M",
    "standardisedManufacturerProductIdentifier": "8690000000001"
  }
]
```

<a name="package"></a>

### Package

| Ad                         | Açıklama                                | Şema    |
| -------------------------- | --------------------------------------- | ------- |
| **quantity** <br>_zorunlu_ | Paket adedi                             | int     |
| **type** <br>_zorunlu_     | Paket türü (Örn: "box")                 | string  |
| **weight** <br>_zorunlu_   | Kilogram cinsinden paketin ağırlığı     | decimal |
| **width** <br>_zorunlu_    | Santimetre cinsinden paketin eni        | decimal |
| **length** <br>_zorunlu_   | Santimetre cinsinden paketin boyu       | decimal |
| **height** <br>_zorunlu_   | Santimetre cinsinden paketin yüksekliği | decimal |

<a name="orderQuoteResponse"></a>

### OrderQuoteResponse

| Ad                         | Açıklama                                                    | Şema                                |
| -------------------------- | ----------------------------------------------------------- | ----------------------------------- |
| **searchId** <br>_zorunlu_ | Oluşturulan arama talebinin benzersiz kimliği               | string                              |
| **quotes** <br>_opsiyonel_ | Üretilen teklifler. Hiç teklif üretilemezse boş dizi döner. | < [StoreQuote](#storeQuote) > array |

<a name="storeQuote"></a>

### StoreQuote

| Ad                                   | Açıklama                                | Şema                                              |
| ------------------------------------ | --------------------------------------- | ------------------------------------------------- |
| **quoteReference** <br>_zorunlu_     | Teklifin referans id'si                 | string                                            |
| **price** <br>_zorunlu_              | Teklif tutarı                           | decimal                                           |
| **currency** <br>_zorunlu_           | Teklif para birimi                      | string                                            |
| **serviceType** <br>_zorunlu_        | Servis türü (express, eco-express, vb.) | string                                            |
| **minTransitTime** <br>_opsiyonel_   | Minimum taşıma süresi (gün)             | int                                               |
| **maxTransitTime** <br>_opsiyonel_   | Maksimum taşıma süresi (gün)            | int                                               |
| **description** <br>_zorunlu_        | Teklif açıklaması                       | string                                            |
| **carrier** <br>_zorunlu_            | Taşıyıcı firma                          | string                                            |
| **additionalServices** <br>_zorunlu_ | Ek hizmet bilgileri                     | < [AdditionalService](#additionalService) > array |

<a name="additionalService"></a>

### AdditionalService

| Ad                            | Açıklama                                           | Şema    |
| ----------------------------- | -------------------------------------------------- | ------- |
| **serviceCode** <br>_zorunlu_ | Ek hizmet kodu. Geçerli değerler: "ddp"            | string  |
| **priceAmount** <br>_zorunlu_ | Ek hizmet tutarı                                   | decimal |
| **currency** <br>_zorunlu_    | Ek hizmet para birimi                              | string  |
| **isRequired** <br>_zorunlu_  | Ek hizmetin zorunlu olup olmadığını belirten değer | boolean |

<a name="load"></a>

### Load

Taşıması yapılacak yüke ait paketleme bilgisi.

| Ad                         | Açıklama                                                                                        | Şema    |
| -------------------------- | ----------------------------------------------------------------------------------------------- | ------- |
| **type** <br>_zorunlu_     | Paket türü. Box veya Envelope olmalı. (Case sensitive)                                          | string  |
| **count** <br>_zorunlu_    | Paket adedi. Envelope tipindeki paketler için yalnızca 1 olabilir.                              | string  |
| **weight** <br>_zorunlu_   | Kilogram cinsinden paketin ağırlığı. Envelope tipindeki paketler için en fazla 2.5 kg olabilir. | decimal |
| **width** <br>_opsiyonel_  | Santimetre cinsinden paketin eni. Box tipindeki paketlerde bu değer zorunludur.                 | decimal |
| **length** <br>_opsiyonel_ | Santimetre cinsinden paketin boyu. Box tipindeki paketlerde bu değer zorunludur.                | decimal |
| **height** <br>_opsiyonel_ | Santimetre cinsinden paketin yüksekliği. Box tipindeki paketlerde bu değer zorunludur.          | decimal |

<a name="quoteResponse"></a>

### QuoteResponse

| Ad                                      | Açıklama                                                    | Şema                      |
| --------------------------------------- | ----------------------------------------------------------- | ------------------------- |
| **quoteRequestReference** <br>_zorunlu_ | Oluşturulan teklif isteği talebinin referans id'si          | string                    |
| **quotes** <br>_opsiyonel_              | Üretilen teklifler. Hiç teklif üretilemezse boş dizi döner. | < [Quote](#quote) > array |

<a name="quote"></a>

### Quote

Teklif bilgisi

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
