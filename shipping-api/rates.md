# Teklif Apisi

<a name="overview"></a>

## Genel Bakış

Bu api ile gönderi bilgileri (adresler, ürünler, paketler) gönderilerek taşıyıcı teklifleri alınır. Her çağrı Navlungo'da yeni bir gönderi kaydı ve `reference` açar; teklif daha sonra [createShipment](./shipment.md#createShipment) ile seçilir.

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

Bu api Oauth2 **client_credentials** akışı ile oluşturulan token'lar ile çağrılabilir. Token alırken scope parametresine **shipping_rates_write** değeri gönderilmelidir.

### Operasyonlar

[createRates](#createRates)<br>

<a name="paths"></a>

## Paths

<a name="createRates"></a>

### POST api/shipping/v1/rates

**Operasyon: createRates**

#### Açıklama

Gönderi bilgileri ile taşıyıcı tekliflerini üretir ve yeni bir gönderi referansı açar. Yanıttaki `reference`, `searchId` ve seçilecek teklifin `rateId` değeri [createShipment](./shipment.md#createShipment) isteğinde kullanılır.

#### Rate Limit

- Client başına 10 saniyede en fazla 10 istek yapılabilir

#### Parametreler

| Tip      | İsim                   | Açıklama                       | Şema                          |
| -------- | ---------------------- | ------------------------------ | ----------------------------- |
| **Body** | **body** <br>_zorunlu_ | Teklif almak için gerekli şema | [RatesRequest](#ratesRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Şema                              |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **200**   | Başarılı. Teklif üretilemezse `rates` boş dizi ve `searchId` `00000000-0000-0000-0000-000000000000` döner; `reference` yine açılmıştır.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | [RatesResponse](#ratesResponse)   |
| **400**   | İstek doğrulamasında hata oluştu ([ProblemDetails](#problemDetails)) veya iş kuralı hatası ([Error](#error)):<br> `shipment.fromanddestinationcountrycannotbesame.error` - Gönderici ve alıcı ülkesi aynı (client'ta `can_create_domestic_shipments` yetkisi yok)<br> `createshipment.fromcountrycodeisnotvalid.error`, `createshipment.invoicecountrycodeisnotvalid.error`, `createshipment.destinationcountrycodeisnotvalid.error` - Ülke kodu geçersiz<br> `createshipment.destinationcountrystatecodeisnotvalid.error`, `createshipment.sendercountrystatecodeisnotvalid.error`, `createshipment.invoicecountrystateisnotvalid.error` - Eyalet kodu geçersiz<br> `proformaaddresserrors.identitynumbermustbevalid.error`, `proformaaddresserrors.taxnumbermustbevalid.error` - TR kimlik/vergi numarası hatalı<br> `packageerrors.envelopweightlimitexceeded.error`, `packageerrors.documentweightlimitexceeded.error`, `packageerrors.differenttypesofpackagescannotbeshippedtogether.error` - Paket kuralları<br> `producterrors.invaliddocumenthscode.error`, `producterrors.invaliddocumentprice.error` - Belge tipi paket için ürün kuralları<br> `createshipment.invalidgtipcodesareexist.error` - HS/GTİP kodu tanınmadı<br> `createshipment.countrylistcouldnotbefetch.error`, `createshipment.statelistcouldnotbefetch.error`, `ratescouldnotbegenerated.error` - Teklif üretilirken servis hatası | [Error](#error)                   |
| **401**   | Yetkilendirme hatası. Access token geçersiz, süresi dolmuş veya `shipping_rates_write` scope'u yok                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | [ProblemDetails](#problemDetails) |
| **429**   | Rate limit aşıldı                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [Error](#error)                   |
| **500**   | İstek sırasında beklenmedik bir hata oluştu                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | [ProblemDetails](#problemDetails) |

#### Örnek İstek Body

```
{
  "shipmentType": "sales",
  "currency": "USD",
  "weightUnit": "lb",
  "dimensionUnit": "in",
  "iossNumber": null,
  "senderShippingAddress": {
    "contactName": "Ayşe Yılmaz", "companyName": "Yılmaz Tekstil A.Ş.", "countryCode": "TR", "stateCode": null,
    "city": "İstanbul", "town": "Kadıköy", "postalCode": "34710", "firstLine": "Caferağa Mah. Moda Cad. No:12",
    "secondLine": null, "thirdLine": null, "email": "ayse@yilmaztekstil.com", "phoneCode": "+90", "phoneNumber": "5321234567",
    "identificationNumber": "1234567890", "taxOffice": "Kadıköy"
  },
  "senderInvoiceAddress": {
    "contactName": "Ayşe Yılmaz", "companyName": "Yılmaz Tekstil A.Ş.", "countryCode": "TR", "stateCode": null,
    "city": "İstanbul", "town": "Kadıköy", "postalCode": "34710", "firstLine": "Caferağa Mah. Moda Cad. No:12",
    "secondLine": null, "thirdLine": null, "email": "ayse@yilmaztekstil.com", "phoneCode": "+90", "phoneNumber": "5321234567",
    "identificationNumber": "1234567890", "taxOffice": "Kadıköy"
  },
  "receiverAddress": {
    "contactName": "John Carter", "companyName": null, "countryCode": "US", "stateCode": "NY",
    "city": "New York", "postalCode": "10001", "firstLine": "350 5th Ave Suite 4400", "secondLine": null, "thirdLine": null,
    "email": "john.carter@example.com", "phoneCode": "+1", "phoneNumber": "2125550147", "identificationNumber": null
  },
  "products": [
    { "description": "Cotton t-shirt, short sleeve", "hsCode": "6109.10.00.12", "quantity": 10, "unitPrice": 12.5, "currency": "USD", "originCountry": "TR" }
  ],
  "packages": [
    { "type": "box", "quantity": 2, "weight": 4.4, "length": 16, "width": 12, "height": 8 }
  ]
}
```

#### Örnek Yanıt

```
{
  "reference": 458213,
  "searchId": "3f0c9a4e-7b2d-4c7e-9a1b-2e5d6f7a8b9c",
  "rates": [
    {
      "rateId": "9b7e2c11-4d3a-4f6e-8c2a-1d2e3f4a5b6c",
      "price": 48.9, "currency": "USD",
      "serviceType": "express", "carrier": "United Parcel Service",
      "minTransitTime": 3, "maxTransitTime": 5,
      "description": "UPS Express Saver", "chargeableWeight": 4.0,
      "services": [
        { "code": "ddp", "price": 6.5, "currency": "USD", "isRequired": false },
        { "code": "customs-tax-duty", "price": 11.2, "currency": "USD", "isRequired": true }
      ]
    }
  ]
}
```

<a name="definitions"></a>

## Tanımlar

<a name="ratesRequest"></a>

### RatesRequest

| Ad                                      | Açıklama                                                                                                                                                                                                              | Şema                                |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| **shipmentType** <br>_zorunlu_          | Taşıma türü. Enum: `sales`, `sample`, `gift`, `micro-export`                                                                                                                                                          | string                              |
| **currency** <br>_zorunlu_              | Gönderi para birimi. Enum: `EUR`, `USD`, `TRY`, `GBP`, `CAD`, `SAR`. Farklı para birimli ürünler bu birime çevrilir                                                                                                   | string                              |
| **weightUnit** <br>_opsiyonel_          | Ağırlık birimi: `kg` (varsayılan) veya `lb`. `dimensionUnit` ile birlikte verilir (`kg`/`cm` ya da `lb`/`in`)                                                                                                         | string                              |
| **dimensionUnit** <br>_opsiyonel_       | Boyut birimi: `cm` (varsayılan) veya `in`. `lb`/`in` değerler kg/cm'ye çevrilir (`1 lb = 0,453592 kg`, `1 in = 2,54 cm`, 2 ondalık)                                                                                   | string                              |
| **iossNumber** <br>_opsiyonel_          | IOSS numarası; 12 karakter, `IM` + 10 harf/rakam (örn. `IM1234567890`). Yalnızca AB ülkesine giden ve toplam ürün değeri 150 EUR altındaki gönderilerde saklanır ve tekliflere yansır, diğer gönderilerde yok sayılır | string                              |
| **senderShippingAddress** <br>_zorunlu_ | Gönderici çıkış adresi                                                                                                                                                                                                | [SenderAddress](#senderAddress)     |
| **senderInvoiceAddress** <br>_zorunlu_  | Gönderici fatura adresi                                                                                                                                                                                               | [SenderAddress](#senderAddress)     |
| **receiverAddress** <br>_zorunlu_       | Alıcı adresi                                                                                                                                                                                                          | [ReceiverAddress](#receiverAddress) |
| **products** <br>_zorunlu_              | Ürün satırları. En az 1                                                                                                                                                                                               | < [Product](#product) > array       |
| **packages** <br>_zorunlu_              | Paketler. En az 1, toplam `quantity` en fazla 20. Bir gönderide tek paket tipi kullanılabilir                                                                                                                         | < [Package](#package) > array       |

<a name="senderAddress"></a>

### SenderAddress

| Ad                                     | Açıklama                                                                                                                                  | Şema   |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **contactName** <br>_zorunlu_          | İlgili kişi. 5–50 karakter                                                                                                                | string |
| **companyName** <br>_opsiyonel_        | Şirket adı, en fazla 256 karakter. Doluysa adres kurumsal kabul edilir                                                                    | string |
| **countryCode** <br>_zorunlu_          | Ülke kodu (ISO2), büyük harf                                                                                                              | string |
| **stateCode** <br>_opsiyonel_          | Eyalet kodu                                                                                                                               | string |
| **city** <br>_zorunlu_                 | Şehir, en fazla 50 karakter                                                                                                               | string |
| **town** <br>_zorunlu_                 | İlçe, en fazla 128 karakter. TR için [il ve ilçe listesi](../store-api/city-town.md)                                                      | string |
| **postalCode** <br>_zorunlu_           | Posta kodu, en fazla 32 karakter                                                                                                          | string |
| **firstLine** <br>_zorunlu_            | Adres satırı, 10–30 karakter                                                                                                              | string |
| **secondLine** <br>_opsiyonel_         | İkinci adres satırı, en fazla 30 karakter                                                                                                 | string |
| **thirdLine** <br>_opsiyonel_          | Üçüncü adres satırı, en fazla 30 karakter                                                                                                 | string |
| **email** <br>_zorunlu_                | E-posta                                                                                                                                   | string |
| **phoneCode** <br>_zorunlu_            | Telefon ülke kodu (örn. `+90`), en fazla 25 karakter                                                                                      | string |
| **phoneNumber** <br>_zorunlu_          | Telefon numarası, 5–21 karakter                                                                                                           | string |
| **identificationNumber** <br>_zorunlu_ | Kimlik/vergi numarası, en fazla 64 karakter. TR için bireysel adreste 11 haneli TCKN, kurumsal adreste (`companyName` dolu) 10 haneli VKN | string |
| **taxOffice** <br>_opsiyonel_          | Vergi dairesi, en fazla 256 karakter                                                                                                      | string |

<a name="receiverAddress"></a>

### ReceiverAddress

| Ad                                       | Açıklama                                                                                                                            | Şema   |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **contactName** <br>_zorunlu_            | Alıcı kişi. 5–50 karakter                                                                                                           | string |
| **companyName** <br>_opsiyonel_          | Şirket adı                                                                                                                          | string |
| **countryCode** <br>_zorunlu_            | Ülke kodu (ISO2). Gönderici ülkesinden farklı olmalı; client'ta `can_create_domestic_shipments` yetkisi varsa aynı ülke de olabilir | string |
| **stateCode** <br>_opsiyonel_            | Eyalet kodu                                                                                                                         | string |
| **city** <br>_zorunlu_                   | Şehir, en fazla 50 karakter                                                                                                         | string |
| **postalCode** <br>_opsiyonel_           | Posta kodu, en fazla 32 karakter                                                                                                    | string |
| **firstLine** <br>_zorunlu_              | Adres satırı, 10–30 karakter                                                                                                        | string |
| **secondLine** <br>_opsiyonel_           | İkinci adres satırı, en fazla 30 karakter                                                                                           | string |
| **thirdLine** <br>_opsiyonel_            | Üçüncü adres satırı, en fazla 30 karakter                                                                                           | string |
| **email** <br>_opsiyonel_                | E-posta                                                                                                                             | string |
| **phoneCode** <br>_opsiyonel_            | Telefon ülke kodu. `phoneNumber` verilirse zorunlu                                                                                  | string |
| **phoneNumber** <br>_opsiyonel_          | Telefon numarası, 5–21 karakter                                                                                                     | string |
| **identificationNumber** <br>_opsiyonel_ | Kimlik/vergi numarası                                                                                                               | string |

<a name="product"></a>

### Product

| Ad                              | Açıklama                                                                                                                                                                                                                                                                  | Şema    |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| **description** <br>_zorunlu_   | Ürün açıklaması, 5–500 karakter                                                                                                                                                                                                                                           | string  |
| **hsCode** <br>_zorunlu_        | Ürün HS kodu, en fazla 20 karakter (alıcı `US` değilse en az 6 karakter). Gerçek bir kod olmalı: alıcı `US` ise ABD HTS kodu (`6109.10.00.12` biçimi), diğer ülkelerde 12 haneli TR GTİP (`610910000000`). Tanınmayan kod `createshipment.invalidgtipcodesareexist.error` | string  |
| **quantity** <br>_zorunlu_      | Ürün adedi, 0'dan büyük                                                                                                                                                                                                                                                   | int     |
| **unitPrice** <br>_zorunlu_     | Birim fiyat, 0'dan büyük                                                                                                                                                                                                                                                  | decimal |
| **currency** <br>_zorunlu_      | Ürün para birimi. Enum: `EUR`, `USD`, `TRY`, `GBP`, `CAD`, `SAR`                                                                                                                                                                                                          | string  |
| **originCountry** <br>_zorunlu_ | Menşei ülke kodu (ISO2)                                                                                                                                                                                                                                                   | string  |

<a name="package"></a>

### Package

| Ad                         | Açıklama                                                                                                                                                                                                                             | Şema    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| **type** <br>_zorunlu_     | Paket türü. Enum: `box`, `envelope`, `document`. `envelope` paket başına en fazla 3 kg; `document` paket başına en fazla 0,25 kg, tek ürün, `quantity: 1`, `unitPrice: 1`, `hsCode` ABD için `4819.60.00.00`, diğer ülkeler `481960` | string  |
| **quantity** <br>_zorunlu_ | Paket adedi                                                                                                                                                                                                                          | int     |
| **weight** <br>_zorunlu_   | Paket ağırlığı (`weightUnit` biriminde), 0'dan büyük                                                                                                                                                                                 | decimal |
| **length** <br>_opsiyonel_ | Paket boyu (`dimensionUnit` biriminde). `box` için zorunlu ve 0'dan büyük                                                                                                                                                            | decimal |
| **width** <br>_opsiyonel_  | Paket eni. `box` için zorunlu ve 0'dan büyük                                                                                                                                                                                         | decimal |
| **height** <br>_opsiyonel_ | Paket yüksekliği. `box` için zorunlu ve 0'dan büyük                                                                                                                                                                                  | decimal |

<a name="ratesResponse"></a>

### RatesResponse

| Ad                          | Açıklama                                                      | Şema                    |
| --------------------------- | ------------------------------------------------------------- | ----------------------- |
| **reference** <br>_zorunlu_ | Navlungo gönderi referansı. Sonraki tüm çağrılarda kullanılır | long                    |
| **searchId** <br>_zorunlu_  | Teklif aramasının kimliği. Teklif yoksa boş GUID              | string (Guid)           |
| **rates** <br>_zorunlu_     | Üretilen teklifler. Hiç teklif üretilemezse boş dizi döner    | < [Rate](#rate) > array |

<a name="rate"></a>

### Rate

| Ad                                 | Açıklama                                                                            | Şema                                  |
| ---------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------- |
| **rateId** <br>_zorunlu_           | Teklif kimliği. [createShipment](./shipment.md#createShipment) isteğinde gönderilir | string (Guid)                         |
| **price** <br>_zorunlu_            | Teklif tutarı                                                                       | decimal                               |
| **currency** <br>_zorunlu_         | Teklif para birimi                                                                  | string                                |
| **serviceType** <br>_zorunlu_      | Servis türü. Bkz. [Değer Listeleri](./README.md#valueLists)                         | string                                |
| **carrier** <br>_zorunlu_          | Taşıyıcı adı                                                                        | string                                |
| **minTransitTime** <br>_zorunlu_   | Minimum taşıma süresi (gün)                                                         | int                                   |
| **maxTransitTime** <br>_zorunlu_   | Maksimum taşıma süresi (gün)                                                        | int                                   |
| **description** <br>_zorunlu_      | Teklif açıklaması                                                                   | string                                |
| **chargeableWeight** <br>_zorunlu_ | Faturalanabilir ağırlık (kg)                                                        | decimal                               |
| **services** <br>_zorunlu_         | Ek hizmetler                                                                        | < [RateService](#rateService) > array |

<a name="rateService"></a>

### RateService

| Ad                           | Açıklama                                                                                                             | Şema    |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------- |
| **code** <br>_zorunlu_       | Ek hizmet kodu: `ddp`, `insurance`, `customs-tax-duty`, `atr`, `premium-tracking` (seçilemez)                        | string  |
| **price** <br>_zorunlu_      | Ek hizmet tutarı                                                                                                     | decimal |
| **currency** <br>_zorunlu_   | Ek hizmet para birimi                                                                                                | string  |
| **isRequired** <br>_zorunlu_ | `true` ise hizmet [createShipment](./shipment.md#createShipment) isteğinde `selectedServices` içinde gönderilmelidir | boolean |

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
