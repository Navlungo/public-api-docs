# Navlungo Api

<a name="overview"></a>

## Genel Bakış

Navlungo Api ile, Navlungo çözüm ortaklarına express teklif arama ve mağza yönetim süreçlerine programatik erişim sağlayarak kendi müşterilerine en uygun deneyimi geliştirmelerine yardımcı olmak amaçlanmaktadır.

- Express Teklif Apisi ile; faturalanabilir ağırlığı 300 kg'a kadar olan yükler için teklif alınabilir.
- Mağza Apisi ile;
  - Mağza ekleme, güncelleme ve silme işlemleri,
  - Sipariş oluşturma,
  - Bir siparişi Navlungo'da sevkiyata dönüştürme işlemleri yapılabilir.

### Yetkilendirme

Bu api ile erişim sağlanacak tüm kaynaklara OAuth2 protokolü ile oluşturulan access tokenların gönderilmesi gerekmektedir. Yetkilendirme akışı şu şekilde çalışır;

![AuthorizationCode](authorization/authorization_code_flow.png?raw=true "AuthorizationCode")

**Authorization Code İsteği**

Client daha sonra token almakta kullanacağı authorization code isteğini https://navlungo.com/authorize adresine aşağıdaki parametrelerin değerini querystring aracılığı ile gönderir.

| Ad                               | Açıklama                                                                                                                                                                                                                                   |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **client_id** <br>_zorunlu_      | Navlungo'nun çözüm ortağına sağladığı tekil istemci idsi                                                                                                                                                                                   |
| **code_challenge** <br>_zorunlu_ | Akışın ilerleyen aşamalarında istemciyi tekrar doğrulamak için kullanılacak bilgi. Istemci authorization_code akışını başlatmadan önce oluşturduğu rastgele bir bilgiyi SHA256 ile hashleyerek, elde ettiği hashi bu alanda göndermelidir. |

İstemci tarafından oluşturulan authorization kod istek örneği;

```
https://navlungo.com/authorize?client_id=fb4cd4c9-5569-43ed-9698-e18de5b77f35code_challenge=y3oUIHf8JtMcKhLTcwpVBdal/6r2uJYErygLZb4VXog=
```

Navlungo.com'da tüm kontroller başarılı ise istemciye dönülen authorization code response'u

```
https://callback_url?code=27194b5b-88be-4bb6-bb0c-f6c1b4cde1c2&client_id=fb4cd4c9-5569-43ed-9698-e18de5b77f35&code_challenge=y3oUIHf8JtMcKhLTcwpVBdal/6r2uJYErygLZb4VXog=
```

Navlungo.com authorization_code'u başarı ile ürettiği durumda;

- Entegrasyonlar yapılırken client tarafından sağlanan url'ye (callback_url) üretilen authorization code'u querystring üzerindeki code parametresiyle gönderir.
- İstemci tarafından yapılan ilk istekteki diğer tüm querystring parametrelerini ve değerlerini de callback url'ye geri döndürür.
- İstemci aldığı authorization code ile daha sonra [Token Apisi](./token.md) yardımı ile access token ve refresh token alabilir.

### Operasyonlar

[Token Apisi](./token.md)<br>
[Express Teklif Apisi](./quote.md)<br>
[Mağza Apisi](./store.md)<br>
