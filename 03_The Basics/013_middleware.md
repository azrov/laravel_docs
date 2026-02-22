# Laravel 12 Dokümantasyonu: Ara Yazılımlar (Middleware)

## Giriş

Ara yazılımlar (middleware), uygulamanıza giren HTTP isteklerini incelemek ve filtrelemek için kullanışlı bir mekanizma sağlar. Örneğin, Laravel, uygulamanızın kullanıcısının kimlik doğrulamasının yapılıp yapılmadığını doğrulayan bir ara yazılım içerir. Kullanıcının kimlik doğrulaması yapılmamışsa, ara yazılım kullanıcıyı uygulamanızın giriş ekranına yönlendirecektir. Ancak, kullanıcının kimlik doğrulaması yapılmışsa, ara yazılım isteğin uygulamanın daha da içine doğru ilerlemesine izin verecektir.

Kimlik doğrulama dışında çeşitli görevleri yerine getirmek için ek ara yazılımlar yazılabilir. Örneğin, bir günlük kaydı (logging) ara yazılımı, uygulamanıza gelen tüm istekleri kaydedebilir. Laravel'de kimlik doğrulama ve CSRF koruması için ara yazılımlar dahil olmak üzere çeşitli ara yazılımlar bulunur; ancak, tüm kullanıcı tanımlı ara yazılımlar tipik olarak uygulamanızın `app/Http/Middleware` dizininde bulunur.

## Ara Yazılım Tanımlama (Defining Middleware)

Yeni bir ara yazılım oluşturmak için `make:middleware` Artisan komutunu kullanın:
```bash
php artisan make:middleware EnsureTokenIsValid
```
Bu komut, `app/Http/Middleware` dizininiz içine yeni bir `EnsureTokenIsValid` sınıfı yerleştirecektir. Bu ara yazılımda, yalnızca sağlanan `token` girişi belirtilen bir değerle eşleşiyorsa rotaya erişime izin vereceğiz. Aksi takdirde, kullanıcıları `/home` URI'sine geri yönlendireceğiz:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Gelen bir isteği işle.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```
Gördüğünüz gibi, verilen `token` gizli token'ımızla eşleşmezse, ara yazılım istemciye bir HTTP yönlendirmesi döndürecektir; aksi takdirde, istek uygulamanın daha da içine iletilecektir. İsteği uygulamanın daha derinlerine iletmek (ara yazılımın "geçişine" izin vermek için), `$next` geri çağrısını (callback) `$request` ile çağırmalısınız.

Ara yazılımları, HTTP isteklerinin uygulamanıza ulaşmadan önce geçmesi gereken bir dizi "katman" olarak hayal etmek en iyisidir. Her katman isteği inceleyebilir ve hatta tamamen reddedebilir.

Tüm ara yazılımlar servis kabı (service container) aracılığıyla çözümlenir (resolve edilir), bu nedenle bir ara yazılımın kurucusunda (constructor) ihtiyacınız olan herhangi bir bağımlılığı tip belirtebilirsiniz (type-hint).

#### Ara Yazılımlar ve Yanıtlar (Middleware and Responses)
Elbette, bir ara yazılım, isteği uygulamanın daha derinlerine iletmeden önce veya sonra görevler gerçekleştirebilir. Örneğin, aşağıdaki ara yazılım, istek uygulama tarafından işlenmeden önce bir görev gerçekleştirecektir:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Eylem gerçekleştir (Perform action)

        return $next($request);
    }
}
```
Ancak, bu ara yazılım görevini istek uygulama tarafından işlendikten sonra gerçekleştirecektir:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Eylem gerçekleştir (Perform action)

        return $response;
    }
}
```

## Ara Yazılım Kaydetme (Registering Middleware)

### Global Ara Yazılımlar (Global Middleware)
Bir ara yazılımın uygulamanıza yapılan her HTTP isteği sırasında çalışmasını istiyorsanız, onu uygulamanızın `bootstrap/app.php` dosyasındaki global ara yazılım yığınına (stack) ekleyebilirsiniz:
```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware): void {
     $middleware->append(EnsureTokenIsValid::class);
})
```
`withMiddleware` closure'ına sağlanan `$middleware` nesnesi, `Illuminate\Foundation\Configuration\Middleware`'in bir örneğidir ve uygulamanızın rotalarına atanan ara yazılımları yönetmekten sorumludur. `append` metodu, global ara yazılım listesinin sonuna bir ara yazılım ekler. Listeye bir ara yazılımı başa eklemek isterseniz, `prepend` metodunu kullanmalısınız.

#### Laravel'in Varsayılan Global Ara Yazılımlarını Manuel Olarak Yönetme
Laravel'in global ara yazılım yığınını manuel olarak yönetmek isterseniz, Laravel'in varsayılan global ara yazılım yığınını `use` metoduna sağlayabilirsiniz. Ardından, varsayılan ara yazılım yığınını gerektiği gibi ayarlayabilirsiniz:
```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->use([
        \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class,
        // \Illuminate\Http\Middleware\TrustHosts::class,
        \Illuminate\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ]);
})
```

### Rotalara Ara Yazılım Atama (Assigning Middleware to Routes)
Belirli rotalara ara yazılım atamak isterseniz, rotayı tanımlarken `middleware` metodunu çağırabilirsiniz:
```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```
`middleware` metoduna bir ara yazılım adları dizisi ileterek rotaya birden çok ara yazılım atayabilirsiniz:
```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

#### Ara Yazılımları Hariç Tutma (Excluding Middleware)
Bir rota grubuna ara yazılım atarken, bazen gruptaki tek bir rotaya ara yazılımın uygulanmasını engellemeniz gerekebilir. Bunu `withoutMiddleware` metodunu kullanarak gerçekleştirebilirsiniz:
```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```
Belirli bir ara yazılım kümesini tüm bir rota tanımları grubundan da hariç tutabilirsiniz:
```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```
`withoutMiddleware` metodu yalnızca rota ara yazılımlarını kaldırabilir ve global ara yazılımlar için geçerli değildir.

### Ara Yazılım Grupları (Middleware Groups)
Bazen birden çok ara yazılımı tek bir anahtar altında gruplayarak rotalara atamayı kolaylaştırmak isteyebilirsiniz. Bunu, uygulamanızın `bootstrap/app.php` dosyası içindeki `appendToGroup` metodunu kullanarak gerçekleştirebilirsiniz:
```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);

    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```
Ara yazılım grupları, tekil ara yazılımlarla aynı sözdizimi kullanılarak rotalara ve controller eylemlerine atanabilir:
```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

#### Laravel'in Varsayılan Ara Yazılım Grupları
Laravel, web ve API rotalarınıza uygulamak isteyebileceğiniz yaygın ara yazılımları içeren önceden tanımlanmış `web` ve `api` ara yazılım gruplarını içerir. Laravel'in bu ara yazılım gruplarını otomatik olarak ilgili `routes/web.php` ve `routes/api.php` dosyalarına uyguladığını unutmayın:

| web Ara Yazılım Grubu |
| :--- |
| `Illuminate\Cookie\Middleware\EncryptCookies` |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession` |
| `Illuminate\View\Middleware\ShareErrorsFromSession` |
| `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

| api Ara Yazılım Grubu |
| :--- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

Bu gruplara ara yazılım eklemek veya başa eklemek isterseniz, uygulamanızın `bootstrap/app.php` dosyası içindeki `web` ve `api` metotlarını kullanabilirsiniz. `web` ve `api` metotları, `appendToGroup` metoduna kullanışlı alternatiflerdir:
```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```
Hatta Laravel'in varsayılan ara yazılım grubu girişlerinden birini kendi özel ara yazılımınızla değiştirebilirsiniz:
```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```
Veya bir ara yazılımı tamamen kaldırabilirsiniz:
```php
$middleware->web(remove: [
    StartSession::class,
]);
```

#### Laravel'in Varsayılan Ara Yazılım Gruplarını Manuel Olarak Yönetme
Laravel'in varsayılan `web` ve `api` ara yazılım gruplarındaki tüm ara yazılımları manuel olarak yönetmek isterseniz, grupları tamamen yeniden tanımlayabilirsiniz. Aşağıdaki örnek, `web` ve `api` ara yazılım gruplarını varsayılan ara yazılımlarıyla tanımlayacak ve gerektiğinde bunları özelleştirmenize olanak tanıyacaktır:
```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);

    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```
Varsayılan olarak, `web` ve `api` ara yazılım grupları, `bootstrap/app.php` dosyası tarafından uygulamanızın ilgili `routes/web.php` ve `routes/api.php` dosyalarına otomatik olarak uygulanır.

### Ara Yazılım Takma Adları (Middleware Aliases)
Uygulamanızın `bootstrap/app.php` dosyasında ara yazılımlara takma adlar (aliases) atayabilirsiniz. Ara yazılım takma adları, belirli bir ara yazılım sınıfı için kısa bir takma ad tanımlamanıza olanak tanır; bu, özellikle uzun sınıf adlarına sahip ara yazılımlar için kullanışlı olabilir:
```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```
Ara yazılım takma adı uygulamanızın `bootstrap/app.php` dosyasında tanımlandıktan sonra, ara yazılımı rotalara atarken takma adı kullanabilirsiniz:
```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```
Kolaylık olması açısından, Laravel'in yerleşik ara yazılımlarından bazıları varsayılan olarak takma adlandırılmıştır. Örneğin, `auth` ara yazılımı, `Illuminate\Auth\Middleware\Authenticate` ara yazılımı için bir takma addır. Aşağıda varsayılan ara yazılım takma adlarının bir listesi bulunmaktadır:

| Takma Ad | Ara Yazılım |
| :--- | :--- |
| `auth` | `Illuminate\Auth\Middleware\Authenticate` |
| `auth.basic` | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth` |
| `auth.session` | `Illuminate\Session\Middleware\AuthenticateSession` |
| `cache.headers` | `Illuminate\Http\Middleware\SetCacheHeaders` |
| `can` | `Illuminate\Auth\Middleware\Authorize` |
| `guest` | `Illuminate\Auth\Middleware\RedirectIfAuthenticated` |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword` |
| `precognitive` | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests` |
| `signed` | `Illuminate\Routing\Middleware\ValidateSignature` |
| `subscribed` | `\Spark\Http\Middleware\VerifyBillableIsSubscribed` |
| `throttle` | `Illuminate\Routing\Middleware\ThrottleRequests` veya `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` |
| `verified` | `Illuminate\Auth\Middleware\EnsureEmailIsVerified` |

### Ara Yazılımları Sıralama (Sorting Middleware)
Nadiren de olsa, ara yazılımlarınızın belirli bir sırayla yürütülmesi gerekebilir, ancak rotaya atandıklarında sıraları üzerinde kontrolünüz olmayabilir. Bu durumlarda, uygulamanızın `bootstrap/app.php` dosyasındaki `priority` metodunu kullanarak ara yazılım önceliğinizi belirtebilirsiniz:
```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

## Ara Yazılım Parametreleri (Middleware Parameters)

Ara yazılımlar ek parametreler de alabilir. Örneğin, uygulamanızın belirli bir eylemi gerçekleştirmeden önce kimliği doğrulanmış kullanıcının belirli bir "role" sahip olduğunu doğrulaması gerekiyorsa, ek bir argüman olarak bir rol adı alan bir `EnsureUserHasRole` ara yazılımı oluşturabilirsiniz.

Ek ara yazılım parametreleri, `$next` argümanından sonra ara yazılıma iletilecektir:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Gelen bir isteği işle.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Yönlendir...
        }

        return $next($request);
    }

}
```
Rota tanımlandığında, parametreler ara yazılım adından sonra `:` kullanılarak ayrılarak belirtilebilir. Birden çok parametre virgülle ayrılmalıdır:
```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```