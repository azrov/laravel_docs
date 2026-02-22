# Laravel 12 Dokümantasyonu: CSRF Koruması

## Giriş

Siteler arası istek sahteciliği (cross-site request forgeries - CSRF), yetkili bir kullanıcı adına yetkisiz komutların gerçekleştirildiği bir tür kötü niyetli istismardır. Neyse ki Laravel, uygulamanızı siteler arası istek sahteciliği (CSRF) saldırılarından korumayı kolaylaştırır.

#### Güvenlik Açığının Açıklaması (An Explanation of the Vulnerability)

Siteler arası istek sahteciliğine aşina değilseniz, bu güvenlik açığının nasıl istismar edilebileceğine dair bir örnek tartışalım. Uygulamanızın, kimliği doğrulanmış kullanıcının e-posta adresini değiştirmek için bir `POST` isteği kabul eden bir `/user/email` rotasına sahip olduğunu hayal edin. Büyük olasılıkla, bu rota, kullanıcının yeni e-posta adresini değiştirmek için bir `email` giriş alanı bekler. **CSRF koruması olmadan**, kötü niyetli bir web sitesi, uygulamanızın `/user/email` rotasına işaret eden ve kötü niyetli kullanıcının kendi e-posta adresini gönderen bir HTML formu oluşturabilir:
```html
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="kotu-niyetli-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```
Kötü niyetli web sitesi, sayfa yüklendiğinde formu otomatik olarak gönderirse, kötü niyetli kullanıcının tek yapması gereken, uygulamanızın habersiz bir kullanıcısını kendi web sitelerini ziyaret etmeleri için kandırmaktır ve böylece kullanıcının e-posta adresi uygulamanızda değiştirilecektir.

Bu güvenlik açığını önlemek için, gelen her `POST`, `PUT`, `PATCH` veya `DELETE` isteğini, kötü niyetli uygulamanın erişemeyeceği gizli bir oturum değeri (secret session value) için incelememiz gerekir.

## CSRF İsteklerini Önleme (Preventing CSRF Requests)

Laravel, uygulama tarafından yönetilen her aktif kullanıcı oturumu için otomatik olarak bir CSRF "token"ı oluşturur. Bu token, kimliği doğrulanmış kullanıcının, uygulamaya gerçekten istek yapan kişi olduğunu doğrulamak için kullanılır. Bu token kullanıcının oturumunda saklandığı ve oturum her yeniden oluşturulduğunda değiştiği için, kötü niyetli bir uygulama buna erişemez.

Geçerli oturumun CSRF token'ına, isteğin oturumu (request's session) veya `csrf_token` yardımcı fonksiyonu aracılığıyla erişilebilir:
```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```
Uygulamanızda bir "POST", "PUT", "PATCH" veya "DELETE" HTML formu tanımladığınız her zaman, CSRF koruma ara yazılımının isteği doğrulayabilmesi için forma gizli bir `_token` alanı eklemelisiniz. Kolaylık olması açısından, gizli token giriş alanını oluşturmak için `@csrf` Blade yönergesini kullanabilirsiniz:
```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Şuna eşdeğerdir... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```
Varsayılan olarak `web` ara yazılım grubuna dahil edilen `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` ara yazılımı, istek girdisindeki token'ın oturumda saklanan token ile eşleşip eşleşmediğini otomatik olarak doğrulayacaktır. Bu iki token eşleştiğinde, isteği başlatanın kimliği doğrulanmış kullanıcı olduğunu biliriz.

### CSRF Token'ları ve Tek Sayfa Uygulamalar (SPA'lar) (CSRF Tokens & SPAs)
Laravel'i bir API arka ucu olarak kullanan bir Tek Sayfa Uygulaması (SPA) oluşturuyorsanız, API'nizle kimlik doğrulama ve CSRF güvenlik açıklarına karşı koruma hakkında bilgi için Laravel Sanctum dokümantasyonuna başvurmalısınız.

### URI'leri CSRF Korumasının Dışında Tutma (Excluding URIs From CSRF Protection)
Bazen bir dizi URI'yi CSRF korumasının dışında tutmak isteyebilirsiniz. Örneğin, ödemeleri işlemek için Stripe kullanıyorsanız ve onların webhook sistemini kullanıyorsanız, Stripe webhook işleyici rotanızı CSRF korumasının dışında tutmanız gerekecektir, çünkü Stripe rotalarınıza hangi CSRF token'ını göndereceğini bilemez.

Tipik olarak, bu tür rotaları Laravel'in `routes/web.php` dosyasındaki tüm rotalara uyguladığı `web` ara yazılım grubunun dışına yerleştirmelisiniz. Ancak, uygulamanızın `bootstrap/app.php` dosyasındaki `validateCsrfTokens` metoduna URI'lerini sağlayarak belirli rotaları da hariç tutabilirsiniz:
```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```
Kolaylık olması açısından, testler çalıştırılırken tüm rotalar için CSRF ara yazılımı otomatik olarak devre dışı bırakılır.

## X-CSRF-TOKEN

CSRF token'ını bir POST parametresi olarak kontrol etmeye ek olarak, varsayılan olarak `web` ara yazılım grubuna dahil edilen `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` ara yazılımı, `X-CSRF-TOKEN` istek başlığını (request header) da kontrol edecektir. Örneğin, token'ı bir HTML meta etiketinde (meta tag) saklayabilirsiniz:
```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```
Ardından, jQuery gibi bir kütüphaneye token'ı otomatik olarak tüm istek başlıklarına eklemesi talimatını verebilirsiniz. Bu, eski JavaScript teknolojisini kullanan AJAX tabanlı uygulamalarınız için basit ve kullanışlı CSRF koruması sağlar:
```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

## X-XSRF-TOKEN

Laravel, geçerli CSRF token'ını, framework tarafından oluşturulan her yanıtla birlikte gönderilen şifrelenmiş bir `XSRF-TOKEN` çerezinde (cookie) saklar. Çerez değerini, `X-XSRF-TOKEN` istek başlığını ayarlamak için kullanabilirsiniz.

Bu çerez öncelikle geliştirici kolaylığı olarak gönderilir, çünkü Angular ve Axios gibi bazı JavaScript framework'leri ve kütüphaneleri, aynı kaynaktan (same-origin) yapılan isteklerde bu değeri otomatik olarak `X-XSRF-TOKEN` başlığına yerleştirir.

Varsayılan olarak, `resources/js/bootstrap.js` dosyası, sizin için `X-XSRF-TOKEN` başlığını otomatik olarak gönderecek olan Axios HTTP kütüphanesini içerir.