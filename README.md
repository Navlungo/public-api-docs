# Navlungo Api

<a name="overview"></a>

## 1. Genel Bakış

Navlungo Api ile, Navlungo çözüm ortaklarına express teklif arama ve mağaza yönetim süreçlerine programatik erişim sağlayarak kendi müşterilerine en uygun deneyimi geliştirmelerine yardımcı olmak amaçlanmaktadır.

- Express Teklif Apisi ile; faturalanabilir ağırlığı 300 kg'a kadar olan yükler için teklif alınabilir.
- Mağaza Apisi ile;
  - Mağaza ekleme, güncelleme ve silme işlemleri,
  - Sipariş oluşturma,
  - Bir siparişi Navlungo'da sevkiyata dönüştürme işlemleri yapılabilir.

---

### 2. Yetkilendirme

Bu api ile erişim sağlanacak tüm kaynaklara OAuth2 protokolü ile oluşturulan access tokenların gönderilmesi gerekmektedir. Navlungo API'lerinde yetkilendirme senaryoya ve istemci tanımına göre iki farklı şekilde yapılır. Bunlar;

- Authorization Code(varsayılan yetkilendirme türü)
- Client Credentials (varsayılan olarak verilmez. Sadece server to server kullanımlar için uygundur. Senaryonuz client-credentials akışı gerektiriyorsa Navlungo ekibi ile senaryonunun üzerinden geçmeniz gerekmektedir.)

#### 2.1. Authorization Code

<a name="auth"></a>

Bu akışta istemciye access ve refresh token verilmeden önce web üzerinden ilgili kaynaklara erişim için kullanıcının rızası alınır. Kullanıcı onayından sonra Navlungo tarafından istemciye gönderilen tek seferlik yetkilendirme kodu ile istemci **kullanıcı adına izin verilen kaynaklar üzerinde işlem yapabilir**.

[Yetkilendirme akışı videolu anlatımı](https://www.loom.com/share/058aec858ee648f4a40390f811ad606b)

![AuthorizationCode](authorization/authorization_code_flow.png?raw=true "AuthorizationCode")

Video'daki örneğin hesaplamasını yapan örnek C# ve PHP kod bloklarını aşağıda bulabilirsiniz. c# örneğini [.Net Fiddle](https://dotnetfiddle.net/) gibi online bir ortamda koşabilirsiniz.
C#
```
using System;

namespace VerifierHasher;

class Program
{
   static void Main(string[] args)
   {
   	var verifier = "b1f711f0-ee78-44e8-b098-84de6df69ad4"; //buraya kendi örnek verifier'ınızı girerek oluşan hash'i görebilirsiniz
	var verifierBytes = System.Text.Encoding.UTF8.GetBytes(verifier);
			
        var sha256 = System.Security.Cryptography.SHA256.Create();
	var sha256ComputedVerifierHashBytes = sha256.ComputeHash(verifierBytes);
			
	string verifierHashedBase64String = Convert.ToBase64String(sha256ComputedVerifierHashBytes);
	Console.WriteLine($"Verifier: {verifier}");
	Console.WriteLine($"Hashed Verifier: {verifierHashedBase64String}");
   }
}
```

PHP
```
$code_verifier = 'b1f711f0-ee78-44e8-b098-84de6df69ad4';
$code_challenge = base64_encode(hash('sha256', utf8_encode($code_verifier) , true));
```

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

---

## 3. Sipariş ve Sevkiyat Oluşturma Akışı

Navlungo ile en sık yapılmak istenen senaryo Navlungo.com üzerinden sevkiyat oluşturma olduğu için bu başlık altında sevkiyat akışını detaylandıracağız.

**Tanımlar**

- **Client**: Navlungo public apisini kullanan kurum. (client_id ve client_secret'a sahiptir.)
- **User**: Navlungo.com kullanıcı hesabı. ( navlungo.com'da kullanıcı adı ve şifresi olan gerçek bir hesap )

**Dikkat Edilmesi Gereken Noktalar**

- Sevkiyat oluşturabilmek için önce bir mağaza oluşturmak ve bu mağzaya sipariş göndermek gerekir.
- Sipariş oluşturulduktan sonra, yaratılan sipariş bir teklif ile ilişkilendirilip sevkiyata dönüştürülebilir.
- Mağaza, sipariş ve sevkiyat oluşturma işlemleri için;
  - Entegrasyon application to application çalışmaz. Yani doğrudan **client** için bu işlemleri yapamazsınız. Bir **user** zorunludur.
  - **Client**, [Authorization Code](#auth) başlığında da tariflendiği şekilde **user adına** işlem yetkisi alır ve o kullanıcı adına işlem yapar.

**Senaryolar**

_Senaryo 1: Pazaryeri_

Mağaza apisi **Client**'ın kullanıcılarının aynı zamanda Navlungo hesabının da olduğu ( Navlungo User ) **pazaryeri senaryoları** düşünülerek geliştirilmiştir. Yani apinin önceliği **Client**'ın kullanıcılarının Navlungo hesaplarını **Client**'ın platformuna bağlaması, daha sonra platform üzerinden siparişleri Navlungo'ya otomatik göndermeleri ve sevkiyata dönüştürebilmesidir.

_Senaryo 2: Client'ın Navlungo Hesabını Kullanması_

Bazı senaryolarda **Client**, **kendi Navlungo User** 'ını kullanmak isteyebilir. Bu senaryoda Client'ın kullanıcılarının Navlungo hesabı olması gerekmez ve fiyatlama vb. süreçler Client'ın Navlungo hesabı üzerinden yapılır. [Authorization Code](#auth) akışı ile Client'ın Navlungo'daki kendi hesabı için bir kez yetkilendirme akışı çalıştırılıp _refresh_token_ alınabilir. Alınan bu refresh_token access_token alabilmek için tekrar tekrar kullanılabilir. Bu sayede de aslında api-key ile yapılan entegrasyonlara benzer bir akış ortaya çıkar.

**Fakat bu akış her ne kadar api key'le yapılan bir entegrasyona benzese de aynı değildir.Bu senaryo api'nin asıl kullanım amacına uygun olmadığı için daha çok bir alternatif çözüm olarak düşünülmelidir.**

Yukarıdaki iki senaryoda da sevkiyat oluşturma akışı şu şekilde çalışır;

![Shipment Flow](shipment_flow.png?raw=true "AuthorizationCode")

1.1. Client [createStore](/store.md#createStore) ile mağaza yaratır. Navlungo mağazayı başarı ile yarattığında geriye mağaza id'yi döndürür.

1.2. Client, istediği bir mağazasında [createOrder](/store.md#createOrder) ile sipariş yaratır.

1.3. Client, sevkiyata dönüştürmek istediği siparişleri için [generateQuote](/quote.md#quotes) yardımı ile taşıma teklifleri alır. Bu api çağrılırken siparişin paketleme bilgileri apiye argüman olarak gönderilir. Navlungo gelen isteğe göre bir veya birden fazla taşıma teklifi oluşturur.

1.4. Client, [shipOrder](/store.md#shipOrder) ile bir sipariş ve teklifi ilişkilendirerek sevkiyata dönüştürebilir.

---

## 4. Operasyonlar

[Token Apisi](./token.md)<br>
[Express Teklif Apisi](./quote.md)<br>
[Mağaza Apisi](./store.md)<br>
[Kargo Takip Apisi](./cargoTracking.md)<br>
