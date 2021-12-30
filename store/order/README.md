# Marketplace Entegrasyonu için Order Api'leri

<a name="overview"></a>

## Genel Bakış

Mağazasını başarılı bir şekilde Navlungo'ya bağlayan müşterinin siparişlerini sisteme kaydeden api'dir. Oluşan siparişler teklif almaya hazır siparişlerdir.

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

- [createOrder](#createOrder)<br>

<a name="paths"></a>

## Paths

<a name="createOrder"></a>

### POST /v1/stores/{id}/order

**Operasyon: createOrder**

#### Açıklama

İşlem yapam kullanıcıya ait iletilen id'li mağazaya sipariş ekleyen api'dir.

#### Parametreler

| Tip       | İsim                    | Açıklama                             | Şema                                      |
| --------- | ----------------------- | ------------------------------------ | ----------------------------------------- |
| **Query** | **{id}** <br>_required_ | Kullanıcının mağazasının tekil Id'si | string                                    |
| **Body**  | **body** <br>_required_ | Order yaratmak için gerekli şema     | [CreateOrderRequest](#createOrderRequest) |

#### Yanıtlar

| HTTP Kodu | Açıklama                                                        | Şema            |
| --------- | --------------------------------------------------------------- | --------------- |
| **200**   | Başarılı                                                        |                 |
| **400**   | İstek doğrulamasında hata oluştu veya istek geçersiz.           | [Error](#error) |
| **401**   | Yetkilendirme hatası. Access token geçersiz veya süresi dolmuş. | [Error](#error) |
| **500**   | İstek sırasında beklenmedik bir hata oluştu.                    | [Error](#error) |

<a name="definitions"></a>

## Tanımlar

<a name="createOrderRequest"></a>

### createOrderRequest

| Ad                      | Açıklama        | Şema                       |
| ----------------------- | --------------- | -------------------------- |
| **order** <br>_zorunlu_ | Sipariş bilgisi | < [Order](#order) > object |

<a name="order"></a>

### Order

Kaydedilecek olan siparişin modelidir.

| Ad                                      | Açıklama                                    | Şema                                   |
| --------------------------------------- | ------------------------------------------- | -------------------------------------- |
| **receiverAddress** <br>_zorunlu_       | Alıcının adres bilgisidir.                  | <[OrderAddress](#orderAddress)> object |
| **orderReference** <br>_zorunlu_        | Siparişin uniq reference Id'si              | string                                 |
| **receiverPhoneNumber** <br>_opsiyonel_ | Alıcının telefon numarası                   | string                                 |
| **receiverEmail** <br>_opsiyonel_       | Alıcının email bilgisi                      | string                                 |
| **currencyCode** <br>_zorunlu_          | 3 haneli döviz kodu                         | string                                 |
| **orderLines** <br>_zorunlu_            | Kontak kişinin telefon numarası             | <[OrderLine](#orderLine)> array        |
| **creationDateInStore** <br>_optional_  | Siparişin mağazasında ilk yaratıldığı tarih | long                                   |

<a name="orderLine"></a>

### OrderLine

Siparişin içindeki ürünlerin detay bilgileridir.

| Ad                            | Açıklama                                                             | Şema    |
| ----------------------------- | -------------------------------------------------------------------- | ------- |
| **sku** <br>_opsiyonel_       | Ürünün takibinin yapılabilmesi için belirlenen tekil id numarasıdır. | string  |
| **description** <br>_zorunlu_ | Ürün açıklaması                                                      | string  |
| **quantity** <br>_zorunlu_    | Adet bilgisi                                                         | integer |
| **imageUrl** <br>_opsiyonel_  | Ürünün tanımlayıcı resminin url'i                                    | string  |
| **price** <br>_zorunlu_       | Ürünün satış bedeli                                                  | string  |
| **harmonyCode** <br>_zorunlu_ | Ürünün sınıflandırılmasında kullanılan muhteviyat kodu               | strig   |

<a name="createOrderResponse"></a>

### createOrderResponse

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
