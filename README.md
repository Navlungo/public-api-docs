# Navlungo Api

<a name="overview"></a>

## Önemli Notlar

1. Size iletilen client bilgileri ile yalnızca ilgili ortamda işlem yapabilirsiniz. QA ortam için iletilen client bilgileri ile production apilerine erişemezsiniz.
2. QA ve Production ortamları izole olduklarından production kullanıcı bilgileriniz ile qa ortamda login olamaz, giriş yapamazsınız. QA'de işlem yapmak için kullanıcı ihtiyacınız var ise, kayıt ol butonu aracılığı ile yeni bir kullanıcı oluşturup testlerinizde kullanabilirsiniz.
3. QA ortamı varsayılan olarak hafta içi gece ve hafta sonları kapalıdır, apilerden yanıt alamayabilirsiniz. Çalışma planlamanız ve bilgisini önceden vermeniz halinde, ortamı açık tutabiliriz.

## 1. Genel Bakış

Navlungo Api ile Navlungo çözüm ortaklarına, kendi müşterilerine en uygun deneyimi geliştirebilmeleri için gönderi süreçlerine programatik erişim sağlanır. İki ayrı entegrasyon modeli sunulmaktadır:

| Bölüm                                    | Kim adına çalışır                                       | Yetkilendirme                          | Kapsam                                                                                                           |
| ---------------------------------------- | ------------------------------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| [Store API](./store-api/README.md)       | Navlungo **kullanıcısı** adına (pazaryeri senaryosu)    | OAuth2 authorization_code (varsayılan) | Mağaza ekleme/güncelleme, sipariş oluşturma, express teklif, siparişi sevkiyata dönüştürme, etiket, takip, belge |
| [Shipping API](./shipping-api/README.md) | Doğrudan **istemci** (client) adına, sunucudan sunucuya | OAuth2 client_credentials              | Teklif alma, gönderi oluşturma, etiket, belge, takip, ETGB ve iptal işlemlerinin tamamı tek bir client hesabıyla |

Hangi modelin size uygun olduğuna Navlungo entegrasyon ekibi ile birlikte karar verilir; client bilgileri buna göre tanımlanır.

---

## 2. Bölümler

### Store API

[Genel Bakış ve Yetkilendirme](./store-api/README.md)</br>
[Token Apisi](./store-api/token.md)</br>
[Express Teklif Apisi](./store-api/quote.md)</br>
[Mağaza Apisi](./store-api/store.md)</br>
[Kargo Takip Apisi](./store-api/cargoTracking.md)</br>
[Shipment Api](./store-api/shipment.md)</br>

### Shipping API

[Genel Bakış ve Yetkilendirme](./shipping-api/README.md)</br>
[Token Apisi](./shipping-api/token.md)</br>
[Teklif Apisi](./shipping-api/rates.md)</br>
[Gönderi Apisi](./shipping-api/shipment.md)</br>
[Gönderi Takip Apisi](./shipping-api/cargoTracking.md)</br>
