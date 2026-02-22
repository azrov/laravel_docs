# Laravel 12 Dokümantasyonu: Kimlik Doğrulama (Authentication)

## Giriş

Birçok web uygulaması, kullanıcılarının uygulamayla kimlik doğrulaması yapması ve "oturum açması" için bir yol sağlar. Bu özelliği web uygulamalarında uygulamak karmaşık ve potansiyel olarak riskli bir girişim olabilir. Bu nedenle Laravel, kimlik doğrulamayı hızlı, güvenli ve kolay bir şekilde uygulamanız için ihtiyaç duyduğunuz araçları size vermeye çalışır.

Özünde, Laravel'in kimlik doğrulama olanakları (facilities) "gardiyanlar" (guards) ve "sağlayıcılardan" (providers) oluşur. Gardiyanlar, kullanıcıların her istek için nasıl kimlik doğrulayacağını tanımlar. Örneğin, Laravel, oturum depolama (session storage) ve çerezleri (cookies) kullanarak durumu koruyan bir `session` gardiyanı ile birlikte gelir.

Sağlayıcılar, kullanıcıların kalıcı depolama alanınızdan (persistent storage) nasıl alınacağını tanımlar. Laravel, kullanıcıları Eloquent ve veritabanı sorgu oluşturucuyu (database query builder) kullanarak almak için destekle birlikte gelir. Ancak, uygulamanız için gerektiğinde ek sağlayıcılar tanımlamakta özgürsünüz.

Uygulamanızın kimlik doğrulama yapılandırma dosyası `config/auth.php` konumunda bulunur. Bu dosya, Laravel'in kimlik doğrulama servislerinin davranışını ayarlamak için iyi belgelenmiş birkaç seçenek içerir.

Gardiyanlar ve sağlayıcılar, "roller" (roles) ve "izinler" (permissions) ile karıştırılmamalıdır. Kullanıcı eylemlerini izinler aracılığıyla yetkilendirme (authorization) hakkında daha fazla bilgi edinmek için lütfen yetkilendirme dokümantasyonuna bakın.

### Başlangıç Kitleri (Starter Kits)
Hızlı başlamak mı istiyorsunuz? Yeni bir Laravel uygulamasına bir Laravel uygulama başlangıç kiti (application starter kit) kurun. Veritabanınızı migrate ettikten sonra, tarayıcınızda `/register` veya uygulamanıza atanmış herhangi bir URL'ye gidin. Başlangıç kitleri, tüm kimlik doğrulama sisteminizin iskelesini (scaffolding) oluşturacaktır!

Son Laravel uygulamanızda bir başlangıç kiti kullanmamayı tercih etseniz bile, bir başlangıç kiti kurmak, Laravel'in tüm kimlik doğrulama işlevselliğini gerçek bir Laravel projesinde nasıl uygulayacağınızı öğrenmek için harika bir fırsat olabilir. Laravel başlangıç kitleri sizin için kimlik doğrulama controller'ları, rotaları (routes) ve görünümleri (views) içerdiğinden, Laravel'in kimlik doğrulama özelliklerinin nasıl uygulanabileceğini öğrenmek için bu dosyaların içindeki kodu inceleyebilirsiniz.

### Veritabanıyla İlgili Hususlar (Database Considerations)
Varsayılan olarak Laravel, `app/Models` dizininizde bir `App\Models\User` Eloquent modeli içerir. Bu model, varsayılan Eloquent kimlik doğrulama sürücüsü ile kullanılabilir.

Uygulamanız Eloquent kullanmıyorsa, Laravel sorgu oluşturucuyu kullanan `database` kimlik doğrulama sağlayıcısını kullanabilirsiniz. Uygulamanız MongoDB kullanıyorsa, MongoDB'nin resmi Laravel kullanıcı kimlik doğrulama dokümantasyonuna göz atın.

`App\Models\User` modeli için veritabanı şemasını oluştururken, `password` sütununun uzunluğunun en az 60 karakter olduğundan emin olun. Elbette, yeni Laravel uygulamalarında bulunan kullanıcılar tablosu migration'ı zaten bu uzunluğu aşan bir sütun oluşturur.

Ayrıca, `users` (veya benzeri) tablonuzun 100 karakterlik, boş değer alabilen (nullable), bir dize olan `remember_token` sütunu içerdiğini doğrulamalısınız. Bu sütun, uygulamanızda oturum açarken "beni hatırla" seçeneğini seçen kullanıcılar için bir token saklamak üzere kullanılacaktır. Yine, yeni Laravel uygulamalarında bulunan varsayılan kullanıcılar tablosu migration'ı zaten bu sütunu içerir.

### Ekosisteme Genel Bakış (Ecosystem Overview)
Laravel, kimlik doğrulamayla ilgili birkaç paket sunar. Devam etmeden önce, Laravel'deki genel kimlik doğrulama ekosistemini gözden geçireceğiz ve her paketin amacını tartışacağız.

İlk olarak, kimlik doğrulamanın nasıl çalıştığını düşünün. Bir web tarayıcısı kullanırken, kullanıcı bir oturum açma formu aracılığıyla kullanıcı adını ve parolasını sağlayacaktır. Bu kimlik bilgileri (credentials) doğruysa, uygulama kimliği doğrulanmış kullanıcı hakkındaki bilgileri kullanıcının oturumunda (session) saklayacaktır. Tarayıcıya gönderilen bir çerez, oturum kimliğini (session ID) içerir, böylece uygulamaya yapılan sonraki istekler kullanıcıyı doğru oturumla ilişkilendirebilir. Oturum çerezi alındıktan sonra, uygulama oturum kimliğine dayalı olarak oturum verilerini alacak, kimlik doğrulama bilgilerinin oturumda saklandığını not edecek ve kullanıcıyı "kimliği doğrulanmış" olarak kabul edecektir.

Uzaktaki bir servisin bir API'ye erişmek için kimlik doğrulaması yapması gerektiğinde, web tarayıcısı olmadığı için çerezler tipik olarak kimlik doğrulama için kullanılmaz. Bunun yerine, uzaktaki servis her istekte API'ye bir API token'ı gönderir. Uygulama, gelen token'ı geçerli API token'larından oluşan bir tabloya karşı doğrulayabilir ve isteği, o API token'ıyla ilişkili kullanıcı tarafından yapılmış gibi "kimlik doğrulayabilir".

#### Laravel'in Yerleşik Tarayıcı Kimlik Doğrulama Servisleri (Laravel's Built-in Browser Authentication Services)
Laravel, tipik olarak `Auth` ve `Session` facade'leri aracılığıyla erişilen yerleşik kimlik doğrulama ve oturum servisleri içerir. Bu özellikler, web tarayıcılarından başlatılan istekler için çerez tabanlı (cookie-based) kimlik doğrulama sağlar. Bir kullanıcının kimlik bilgilerini doğrulamanıza ve kullanıcının kimliğini doğrulamanıza izin veren metotlar sağlarlar. Ek olarak, bu servisler uygun kimlik doğrulama verilerini otomatik olarak kullanıcının oturumunda saklayacak ve kullanıcının oturum çerezini gönderecektir. Bu servislerin nasıl kullanılacağına dair bir tartışma bu dokümantasyonun içinde yer almaktadır.

##### Uygulama Başlangıç Kitleri (Application Starter Kits)
Bu dokümantasyonda tartışıldığı gibi, uygulamanızın kendi kimlik doğrulama katmanını oluşturmak için bu kimlik doğrulama servisleriyle manuel olarak etkileşime geçebilirsiniz. Ancak, daha hızlı başlamanıza yardımcı olmak için, tüm kimlik doğrulama katmanının sağlam, modern iskelesini (scaffolding) sağlayan ücretsiz başlangıç kitleri yayınladık.

#### Laravel'in API Kimlik Doğrulama Servisleri (Laravel's API Authentication Services)
Laravel, API token'larını yönetmenize ve API token'larıyla yapılan istekleri kimlik doğrulamanıza yardımcı olacak iki isteğe bağlı paket sunar: Passport ve Sanctum. Lütfen bu kütüphanelerin ve Laravel'in yerleşik çerez tabanlı kimlik doğrulama kütüphanelerinin birbirini dışlamadığını (mutually exclusive) unutmayın. Bu kütüphaneler öncelikle API token kimlik doğrulamasına odaklanırken, yerleşik kimlik doğrulama servisleri çerez tabanlı tarayıcı kimlik doğrulamasına odaklanır. Birçok uygulama, hem Laravel'in yerleşik çerez tabanlı kimlik doğrulama servislerini hem de Laravel'in API kimlik doğrulama paketlerinden birini kullanacaktır.

##### Passport
Passport, çeşitli OAuth2 "grant türleri" sunan ve çeşitli token türleri çıkarmanıza izin veren bir OAuth2 kimlik doğrulama sağlayıcısıdır. Genel olarak, bu, API kimlik doğrulaması için sağlam ve karmaşık bir pakettir. Ancak, çoğu uygulama OAuth2 spesifikasyonu tarafından sunulan karmaşık özellikleri gerektirmez, bu da hem kullanıcılar hem de geliştiriciler için kafa karıştırıcı olabilir. Ayrıca, geliştiriciler tarihsel olarak Passport gibi OAuth2 kimlik doğrulama sağlayıcılarını kullanarak SPA uygulamalarının veya mobil uygulamaların kimliğini nasıl doğrulayacakları konusunda kafa karışıklığı yaşamışlardır.

##### Sanctum
OAuth2'nin karmaşıklığına ve geliştirici kafa karışıklığına yanıt olarak, hem bir web tarayıcısından gelen birinci taraf (first-party) web isteklerini hem de token'lar aracılığıyla API isteklerini ele alabilen daha basit, daha düzenli bir kimlik doğrulama paketi oluşturmaya koyulduk. Bu hedef, Laravel Sanctum'un piyasaya sürülmesiyle gerçekleştirildi. Sanctum, birinci taraf bir web arayüzüne ek olarak bir API sunacak veya arka uç Laravel uygulamasından ayrı olarak var olan bir tek sayfa uygulaması (SPA) tarafından desteklenecek veya bir mobil istemci sunan uygulamalar için tercih edilen ve önerilen kimlik doğrulama paketi olarak düşünülmelidir.

Laravel Sanctum, uygulamanızın tüm kimlik doğrulama sürecini yönetebilen hibrit bir web / API kimlik doğrulama paketidir. Bu, Sanctum tabanlı uygulamalar bir istek aldığında, Sanctum'un öncelikle isteğin kimliği doğrulanmış bir oturuma başvuran bir oturum çerezi içerip içermediğini belirlemesi nedeniyle mümkündür. Sanctum bunu, daha önce tartıştığımız Laravel'in yerleşik kimlik doğrulama servislerini çağırarak gerçekleştirir. İstek bir oturum çerezi aracılığıyla kimlik doğrulaması yapılmıyorsa, Sanctum isteği bir API token'ı için inceler. Bir API token'ı mevcutsa, Sanctum isteği bu token'ı kullanarak kimlik doğrulayacaktır. Bu süreç hakkında daha fazla bilgi edinmek için lütfen Sanctum'un "nasıl çalışır" (how it works) dokümantasyonuna bakın.

#### Özet ve Yığınınızı Seçme (Summary and Choosing Your Stack)
Özetle, uygulamanıza bir tarayıcı kullanılarak erişilecekse ve monolitik bir Laravel uygulaması oluşturuyorsanız, uygulamanız Laravel'in yerleşik kimlik doğrulama servislerini kullanacaktır.

Ardından, uygulamanız üçüncü taraflarca tüketilecek bir API sunuyorsa, uygulamanız için API token kimlik doğrulaması sağlamak üzere Passport veya Sanctum arasında seçim yapacaksınız. Genel olarak, Sanctum, "kapsamlar" (scopes) veya "yetenekler" (abilities) için destek de dahil olmak üzere API kimlik doğrulaması, SPA kimlik doğrulaması ve mobil kimlik doğrulaması için basit, eksiksiz bir çözüm olduğu için mümkün olduğunda tercih edilmelidir.

Bir Laravel arka ucu tarafından desteklenecek bir tek sayfa uygulaması (SPA) oluşturuyorsanız, Laravel Sanctum kullanmalısınız. Sanctum kullanırken, ya kendi arka uç kimlik doğrulama rotalarınızı manuel olarak uygulamanız ya da kayıt, parola sıfırlama, e-posta doğrulama ve daha fazlası gibi özellikler için rotalar ve controller'lar sağlayan başsız (headless) bir kimlik doğrulama arka uç servisi olarak Laravel Fortify'i kullanmanız gerekecektir.

Uygulamanız kesinlikle OAuth2 spesifikasyonu tarafından sağlanan tüm özelliklere ihtiyaç duyduğunda Passport seçilebilir.

Ve hızlıca başlamak isterseniz, halihazırda Laravel'in yerleşik kimlik doğrulama servislerinden oluşan tercih edilen kimlik doğrulama yığınımızı kullanan yeni bir Laravel uygulaması başlatmanın hızlı bir yolu olarak uygulama başlangıç kitlerimizi önermekten mutluluk duyarız.

## Kimlik Doğrulama Hızlı Başlangıcı (Authentication Quickstart)

Dokümantasyonun bu kısmı, kullanıcıların kimliğini Laravel uygulama başlangıç kitleri aracılığıyla doğrulamayı tartışır; bu, hızlıca başlamanıza yardımcı olacak UI iskelesi (UI scaffolding) içerir. Laravel'in kimlik doğrulama sistemleriyle doğrudan entegre olmak isterseniz, kullanıcıları manuel olarak kimlik doğrulama hakkındaki dokümantasyona göz atın.

### Bir Başlangıç Kiti Kurma (Install a Starter Kit)
İlk olarak, bir Laravel uygulama başlangıç kiti kurmalısınız. Başlangıç kitlerimiz, kimlik doğrulamayı yeni Laravel uygulamanıza dahil etmek için güzel tasarlanmış başlangıç noktaları sunar.

### Kimliği Doğrulanmış Kullanıcıyı Alma (Retrieving the Authenticated User)
Bir başlangıç kitinden bir uygulama oluşturduktan ve kullanıcıların kaydolmasına ve uygulamanızda kimlik doğrulaması yapmasına izin verdikten sonra, genellikle o anda kimliği doğrulanmış kullanıcıyla etkileşime geçmeniz gerekecektir. Gelen bir isteği işlerken, `Auth` facade'ının `user` metodu aracılığıyla kimliği doğrulanmış kullanıcıya erişebilirsiniz:
```php
use Illuminate\Support\Facades\Auth;

// O anda kimliği doğrulanmış kullanıcıyı al...
$user = Auth::user();

// O anda kimliği doğrulanmış kullanıcının ID'sini al...
$id = Auth::id();
```
Alternatif olarak, bir kullanıcının kimliği doğrulandıktan sonra, bir `Illuminate\Http\Request` örneği aracılığıyla kimliği doğrulanmış kullanıcıya erişebilirsiniz. Unutmayın, tip belirtilmiş (type-hinted) sınıflar otomatik olarak controller metotlarınıza enjekte edilecektir. `Illuminate\Http\Request` nesnesini tip belirterek, isteğin `user` metodu aracılığıyla uygulamanızdaki herhangi bir controller metodundan kimliği doğrulanmış kullanıcıya kolayca erişebilirsiniz:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Mevcut bir uçuş için uçuş bilgilerini güncelle.
     */
    public function update(Request $request): RedirectResponse
    {
        $user = $request->user();

        // ...

        return redirect('/flights');
    }
}
```

#### Geçerli Kullanıcının Kimliğinin Doğrulanıp Doğrulanmadığını Belirleme (Determining if the Current User is Authenticated)
Gelen HTTP isteğini yapan kullanıcının kimliğinin doğrulanıp doğrulanmadığını belirlemek için `Auth` facade'ındaki `check` metodunu kullanabilirsiniz. Bu metod, kullanıcının kimliği doğrulanmışsa `true` döndürecektir:
```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // Kullanıcı oturum açmış...
}
```
`check` metodunu kullanarak bir kullanıcının kimliğinin doğrulanıp doğrulanmadığını belirlemek mümkün olsa da, kullanıcının belirli rotalara/controller'lara erişmesine izin vermeden önce kimliğinin doğrulandığını doğrulamak için tipik olarak bir ara yazılım (middleware) kullanacaksınız. Bunun hakkında daha fazla bilgi edinmek için rotaları koruma hakkındaki dokümantasyona göz atın.

### Rotaları Koruma (Protecting Routes)
Rota ara yazılımları (route middleware), yalnızca kimliği doğrulanmış kullanıcıların belirli bir rotaya erişmesine izin vermek için kullanılabilir. Laravel, `Illuminate\Auth\Middleware\Authenticate` sınıfı için bir ara yazılım takma adı (middleware alias) olan `auth` ara yazılımı ile birlikte gelir. Bu ara yazılım Laravel tarafından dahili olarak zaten takma adlandırılmış olduğundan, tek yapmanız gereken ara yazılımı bir rota tanımına eklemektir:
```php
Route::get('/flights', function () {
    // Yalnızca kimliği doğrulanmış kullanıcılar bu rotaya erişebilir...
})->middleware('auth');
```

#### Kimliği Doğrulanmamış Kullanıcıları Yönlendirme (Redirecting Unauthenticated Users)
`auth` ara yazılımı, kimliği doğrulanmamış bir kullanıcı algıladığında, kullanıcıyı `login` adlı rotaya yönlendirecektir. Bu davranışı, uygulamanızın `bootstrap/app.php` dosyası içindeki `redirectGuestsTo` metodunu kullanarak değiştirebilirsiniz:
```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectGuestsTo('/login');

    // Bir closure kullanarak...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

#### Kimliği Doğrulanmış Kullanıcıları Yönlendirme (Redirecting Authenticated Users)
`guest` ara yazılımı, kimliği doğrulanmış bir kullanıcı algıladığında, kullanıcıyı `dashboard` veya `home` adlı rotaya yönlendirecektir. Bu davranışı, uygulamanızın `bootstrap/app.php` dosyası içindeki `redirectUsersTo` metodunu kullanarak değiştirebilirsiniz:
```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectUsersTo('/panel');

    // Bir closure kullanarak...
    $middleware->redirectUsersTo(fn (Request $request) => route('panel'));
})
```

#### Bir Gardiyan (Guard) Belirtme
`auth` ara yazılımını bir rotaya eklerken, kullanıcının kimliğini doğrulamak için hangi "gardiyanın" kullanılması gerektiğini de belirtebilirsiniz. Belirtilen gardiyan, `auth.php` yapılandırma dosyanızdaki `guards` dizisindeki anahtarlardan birine karşılık gelmelidir:
```php
Route::get('/flights', function () {
    // Yalnızca kimliği doğrulanmış kullanıcılar bu rotaya erişebilir...
})->middleware('auth:admin');
```

### Oturum Açma Hız Sınırlaması (Login Throttling)
Uygulama başlangıç kitlerimizden birini kullanıyorsanız, oturum açma denemelerine otomatik olarak hız sınırlaması (rate limiting) uygulanacaktır. Varsayılan olarak, kullanıcı birkaç denemeden sonra doğru kimlik bilgilerini sağlayamazsa bir dakika boyunca oturum açamayacaktır. Hız sınırlaması, kullanıcının kullanıcı adı/e-posta adresine ve IP adresine özgüdür.