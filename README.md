# Navlungo Api

<a name="overview"></a>

## 1. Genel Bakış

Navlungo Api ile, Navlungo çözüm ortaklarına express teklif arama ve mağaza yönetim süreçlerine programatik erişim sağlayarak kendi müşterilerine en uygun deneyimi geliştirmelerine yardımcı olmak amaçlanmaktadır.

- Express Teklif Apisi ile; faturalanabilir ağırlığı 300 kg'a kadar olan yükler için teklif alınabilir.
- Mağaza Apisi ile;
  - Mağaza ekleme, güncelleme ve silme işlemleri,
  - Sipariş oluşturma,
  - Bir siparişi Navlungo'da sevkiyata dönüştürme işlemleri yapılabilir.

### 2. Yetkilendirme

Bu api ile erişim sağlanacak tüm kaynaklara OAuth2 protokolü ile oluşturulan access tokenların gönderilmesi gerekmektedir. Navlungo API'lerinde yetkilendirme senaryoya ve istemci tanımına göre iki farklı şekilde yapılır. Bunlar;

- Authorization Code(varsayılan yetkilendirme türü)
- Client Credentials (varsayılan olarak verilmez. Sadece server to server kullanımlar için uygundur. Senaryonuz client-credentials akışı gerektiriyorsa Navlungo ekibi ile senaryonunun üzerinden geçmeniz gerekmektedir.)

#### 2.1. Authorization Code

Bu akışta istemciye access ve refresh token verilmeden önce web üzerinden ilgili kaynaklara erişim için kullanıcının rızası alınır. Kullanıcı onayından sonra Navlungo tarafından istemciye gönderilen tek seferlik yetkilendirme kodu ile istemci **kullanıcı adına izin verilen kaynaklar üzerinde işlem yapabilir**.

[Yetkilendirme akışı videolu anlatımı](https://www.loom.com/share/058aec858ee648f4a40390f811ad606b)

![AuthorizationCode](authorization/authorization_code_flow.png?raw=true "AuthorizationCode")

**Authorization Code İsteği**

Client akışın sonraki aşamalarında token almakta kullanacağı authorization code isteğini https://navlungo.com/authorize adresine ( test ortamı için https://qa.navlungo.com/authorize ) aşağıdaki parametrelerin değerini querystring aracılığı ile gönderir.

| Ad                               | Açıklama                                                                                                                                                                                                                                                                |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **client_id** <br>_zorunlu_      | Navlungo'nun çözüm ortağına sağladığı tekil istemci idsi                                                                                                                                                                                                                |
| **code_challenge** <br>_zorunlu_ | Akışın ilerleyen aşamalarında istemciyi tekrar doğrulamak için kullanılacak bilgi. Istemci authorization_code akışını başlatmadan önce oluşturduğu rastgele bir bilgiyi SHA256 ile hashleyip base64 ile encodeladıktan sonra elde ettiği hashi bu alanda göndermelidir. |

İstemci tarafından oluşturulan authorization kod istek örneği;

```
https://navlungo.com/authorize?client_id=fb4cd4c9-5569-43ed-9698-e18de5b77f35&code_challenge=y3oUIHf8JtMcKhLTcwpVBdal/6r2uJYErygLZb4VXog=
```

Navlungo.com'da tüm kontroller başarılı ise istemciye dönülen authorization code response'u

```
https://callback_url?code=27194b5b-88be-4bb6-bb0c-f6c1b4cde1c2&client_id=fb4cd4c9-5569-43ed-9698-e18de5b77f35&code_challenge=y3oUIHf8JtMcKhLTcwpVBdal/6r2uJYErygLZb4VXog=
```

Navlungo.com authorization_code'u başarı ile ürettiği durumda;

- Entegrasyonlar yapılırken client tarafından sağlanan url'ye (callback_url) üretilen authorization code'u querystring üzerindeki code parametresiyle gönderir.
- İstemci tarafından yapılan ilk istekteki diğer tüm querystring parametrelerini ve değerlerini de callback url'ye geri döndürür.
- İstemci aldığı authorization code ile daha sonra [Token Apisi](./token.md) yardımı ile access token ve refresh token alabilir.

#### 2.2. Client Credentials

Bu akışta Navlungo'nun istemciye verdiği bilgiler ile ( client_id ve client_secret ) ile istemci adına access_token yaratılır. Alınan bu access_token ile **istemci adına uygun Navlungo api kaynakları üzerinde işlem yapılabilir**. Client credentials akışı id ve secret bilgisinin gönderilmesini gerektirdiği için bu akış tipinde yapılacak token istekleri mutlaka güvenli bir **server side** uygulama tarafından yapılmalıdır. Id ve secret bilgileri istenmeyen kişiler tarafından ele geçirilirse istemci adına işlem yapabilirler.

Akıştaki bariz güvenlik çekinceleri sebebi ile Oauth2 protokolü client_credentials akışında refresh_token yaratımına izin vermemektedir.

**ÖNEMLİ** : Bu yetkilendirme mekanizması her senaryoyu desteklememektedir. Varsayılan olarak Navlungo entegrasyon başvurusu yapıldığında authorization code mekanizması için tanımlama yapılır!

### 3. Operasyonlar

[Token Apisi](./token.md)<br>
[Express Teklif Apisi](./quote.md)<br>
[Mağaza Apisi](./store.md)<br>
[Kargo Takip Apisi](./cargoTracking.md)<br>
