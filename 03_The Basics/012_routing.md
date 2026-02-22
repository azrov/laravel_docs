# Laravel 12 Dokümantasyonu: Yönlendirme (Routing)

## Temel Yönlendirme (Basic Routing)

En temel Laravel rotaları bir URI (Uniform Resource Identifier) ve bir kapatma (closure) alır; karmaşık yapılandırma dosyaları olmadan rotaları ve davranışları tanımlamak için çok basit ve anlamlı bir yöntem sağlar:
```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

### Varsayılan Rota Dosyaları (The Default Route Files)
Tüm Laravel rotaları, `routes` dizininde bulunan rota dosyalarınızda tanımlanır. Bu dosyalar, uygulamanızın `bootstrap/app.php` dosyasında belirtilen yapılandırma kullanılarak Laravel tarafından otomatik olarak yüklenir. `routes/web.php` dosyası, web arayüzünüz için olan rotaları tanımlar. Bu rotalara, oturum durumu (session state) ve CSRF koruması gibi özellikleri sağlayan `web` ara yazılım grubu (middleware group) atanır.

Çoğu uygulama için, rotalarınızı `routes/web.php` dosyanızda tanımlayarak başlayacaksınız. `routes/web.php` içinde tanımlanan rotalara, tanımlanan rotanın URL'sini tarayıcınıza girerek erişilebilir. Örneğin, tarayıcınızda `http://example.com/user` adresine giderek aşağıdaki rotaya erişebilirsiniz:
```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

#### API Rotaları (API Routes)
Uygulamanız ayrıca durumsuz (stateless) bir API sunacaksa, `install:api` Artisan komutunu kullanarak API yönlendirmesini etkinleştirebilirsiniz:
```bash
php artisan install:api
```
`install:api` komutu, üçüncü taraf API tüketicilerinin, SPA'ların veya mobil uygulamaların kimliğini doğrulamak için kullanılabilecek sağlam ve basit bir API token kimlik doğrulama koruyucusu (guard) sağlayan Laravel Sanctum'u kurar. Ayrıca, `install:api` komutu `routes/api.php` dosyasını oluşturur:
```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```
`routes/api.php` içindeki rotalar durumsuzdur (stateless) ve `api` ara yazılım grubuna atanır. Ayrıca, bu rotalara otomatik olarak `/api` URI ön eki (prefix) uygulanır, bu nedenle dosyadaki her rotaya manuel olarak uygulamanız gerekmez. Ön eki, uygulamanızın `bootstrap/app.php` dosyasını değiştirerek değiştirebilirsiniz:
```php
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api/admin',
    // ...
)
```

#### Kullanılabilir Yönlendirici Metotları (Available Router Methods)
Yönlendirici (router), herhangi bir HTTP fiiline (verb) yanıt veren rotaları kaydetmenize olanak tanır:
```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```
Bazen birden çok HTTP fiiline yanıt veren bir rota kaydetmeniz gerekebilir. Bunu `match` metodunu kullanarak yapabilirsiniz. Veya `any` metodunu kullanarak tüm HTTP fiillerine yanıt veren bir rota bile kaydedebilirsiniz:
```php
Route::match(['get', 'post'], '/', function () {
    // ...
});

Route::any('/', function () {
    // ...
});
```
Aynı URI'yi paylaşan birden çok rota tanımlarken, `get`, `post`, `put`, `patch`, `delete` ve `options` metotlarını kullanan rotalar, `any`, `match` ve `redirect` metotlarını kullanan rotalardan önce tanımlanmalıdır. Bu, gelen isteğin doğru rotayla eşleştirilmesini sağlar.

#### Bağımlılık Enjeksiyonu (Dependency Injection)
Rota geri çağırımınızın (callback) imzasında, rotanız tarafından gereken herhangi bir bağımlılığı tip belirtebilirsiniz (type-hint). Belirtilen bağımlılıklar otomatik olarak Laravel servis kabı (service container) tarafından çözümlenecek (resolve) ve geri çağırıma enjekte edilecektir. Örneğin, geçerli HTTP isteğinin otomatik olarak rota geri çağırımınıza enjekte edilmesi için `Illuminate\Http\Request` sınıfını tip belirtebilirsiniz:
```php
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

#### CSRF Koruması (CSRF Protection)
Unutmayın, web rotaları dosyasında tanımlanan `POST`, `PUT`, `PATCH` veya `DELETE` rotalarına işaret eden herhangi bir HTML formu bir CSRF token alanı içermelidir. Aksi takdirde, istek reddedilecektir. CSRF koruması hakkında daha fazla bilgiyi CSRF dokümantasyonunda okuyabilirsiniz:
```blade
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

### Yönlendirme Rotaları (Redirect Routes)
Başka bir URI'ye yönlendiren bir rota tanımlıyorsanız, `Route::redirect` metodunu kullanabilirsiniz. Bu metod, basit bir yönlendirme gerçekleştirmek için tam bir rota veya controller tanımlamak zorunda kalmamanızı sağlayan kullanışlı bir kısayol sağlar:
```php
Route::redirect('/here', '/there');
```
Varsayılan olarak, `Route::redirect` 302 durum kodu döndürür. İsteğe bağlı üçüncü parametreyi kullanarak durum kodunu özelleştirebilirsiniz:
```php
Route::redirect('/here', '/there', 301);
```
Veya 301 durum kodu döndürmek için `Route::permanentRedirect` metodunu kullanabilirsiniz:
```php
Route::permanentRedirect('/here', '/there');
```
Yönlendirme rotalarında rota parametreleri kullanırken, aşağıdaki parametreler Laravel tarafından ayrılmıştır ve kullanılamaz: `destination` ve `status`.

### Görünüm Rotaları (View Routes)
Rotanızın yalnızca bir görünüm (view) döndürmesi gerekiyorsa, `Route::view` metodunu kullanabilirsiniz. `redirect` metodu gibi, bu metod da tam bir rota veya controller tanımlamak zorunda kalmamanız için basit bir kısayol sağlar. `view` metodu, ilk argüman olarak bir URI'yi ve ikinci argüman olarak bir görünüm adını kabul eder. Ayrıca, isteğe bağlı üçüncü bir argüman olarak görünüme iletilecek bir veri dizisi (array) sağlayabilirsiniz:
```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```
Görünüm rotalarında rota parametreleri kullanırken, aşağıdaki parametreler Laravel tarafından ayrılmıştır ve kullanılamaz: `view`, `data`, `status` ve `headers`.

### Rotalarınızı Listeleme (Listing Your Routes)
`route:list` Artisan komutu, uygulamanız tarafından tanımlanan tüm rotaların bir özetini kolayca sağlayabilir:
```bash
php artisan route:list
```
Varsayılan olarak, her rotaya atanan rota ara yazılımları (middleware) `route:list` çıktısında görüntülenmez; ancak, komuta `-v` seçeneğini ekleyerek Laravel'e rota ara yazılımlarını ve ara yazılım grubu adlarını göstermesini söyleyebilirsiniz:
```bash
php artisan route:list -v

# Ara yazılım gruplarını genişlet...
php artisan route:list -vv
```
Ayrıca Laravel'e yalnızca belirli bir URI ile başlayan rotaları göstermesini söyleyebilirsiniz:
```bash
php artisan route:list --path=api
```
Ek olarak, `route:list` komutunu çalıştırırken `--except-vendor` seçeneğini sağlayarak Laravel'e üçüncü taraf paketler tarafından tanımlanan tüm rotaları gizlemesini söyleyebilirsiniz:
```bash
php artisan route:list --except-vendor
```
Benzer şekilde, `route:list` komutunu çalıştırırken `--only-vendor` seçeneğini sağlayarak Laravel'e yalnızca üçüncü taraf paketler tarafından tanımlanan rotaları göstermesini de söyleyebilirsiniz:
```bash
php artisan route:list --only-vendor
```

### Yönlendirmeyi Özelleştirme (Routing Customization)
Varsayılan olarak, uygulamanızın rotaları `bootstrap/app.php` dosyası tarafından yapılandırılır ve yüklenir:
```php
<?php

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```
Ancak bazen, uygulamanızın rotalarının bir alt kümesini içeren tamamen yeni bir dosya tanımlamak isteyebilirsiniz. Bunu başarmak için `withRouting` metoduna bir `then` closure'ı sağlayabilirsiniz. Bu closure içinde, uygulamanız için gerekli olan ek rotaları kaydedebilirsiniz:
```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```
Veya `withRouting` metoduna bir `using` closure'ı sağlayarak rota kaydı üzerinde tam kontrol bile ele alabilirsiniz. Bu argüman iletildiğinde, framework tarafından hiçbir HTTP rotası kaydedilmeyecek ve tüm rotaları manuel olarak kaydetmek sizin sorumluluğunuzda olacaktır:
```php
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

## Rota Parametreleri (Route Parameters)

### Gerekli Parametreler (Required Parameters)
Bazen URI'niz içindeki bölümleri (segment) yakalamanız gerekecektir. Örneğin, URL'den bir kullanıcının ID'sini yakalamanız gerekebilir. Bunu rota parametreleri tanımlayarak yapabilirsiniz:
```php
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```
Rotanızın gerektirdiği kadar çok rota parametresi tanımlayabilirsiniz:
```php
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```
Rota parametreleri her zaman `{}` parantezleri içine alınır ve alfabetik karakterlerden oluşmalıdır. Rota parametre adlarında alt çizgiler (`_`) de kabul edilebilir. Rota parametreleri, sıralarına göre rota geri çağırımlarına / controller'lara enjekte edilir - rota geri çağırımı / controller argümanlarının adları önemli değildir.

#### Parametreler ve Bağımlılık Enjeksiyonu (Parameters and Dependency Injection)
Rotanızın, Laravel servis kabının otomatik olarak rota geri çağırımınıza enjekte etmesini istediğiniz bağımlılıkları varsa, rota parametrelerinizi bağımlılıklarınızdan sonra listelemeniz gerekir:
```php
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User '.$id;
});
```

### Opsiyonel Parametreler (Optional Parameters)
Bazen URI'de her zaman bulunmayabilecek bir rota parametresi belirtmeniz gerekebilir. Bunu, parametre adından sonra bir `?` işareti koyarak yapabilirsiniz. Rotanın karşılık gelen değişkenine bir varsayılan değer vermeyi unutmayın:
```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

### Düzenli İfade Kısıtlamaları (Regular Expression Constraints)
Bir rota örneğindeki `where` metodunu kullanarak rota parametrelerinizin biçimini kısıtlayabilirsiniz. `where` metodu, parametrenin adını ve parametrenin nasıl kısıtlanması gerektiğini tanımlayan bir düzenli ifade (regular expression) kabul eder:
```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```
Kolaylık olması açısından, bazı yaygın olarak kullanılan düzenli ifade kalıplarının, rotalarınıza hızlıca kalıp kısıtlamaları eklemenize izin veren yardımcı metotları (helper methods) vardır:
```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', CategoryEnum::cases());
```
Gelen istek rota kalıbı kısıtlamalarıyla eşleşmezse, 404 HTTP yanıtı döndürülecektir.

#### Global Kısıtlamalar (Global Constraints)
Bir rota parametresinin her zaman belirli bir düzenli ifadeyle kısıtlanmasını istiyorsanız, `pattern` metodunu kullanabilirsiniz. Bu kalıpları, uygulamanızın `App\Providers\AppServiceProvider` sınıfının `boot` metodunda tanımlamalısınız:
```php
use Illuminate\Support\Facades\Route;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```
Kalıp tanımlandıktan sonra, bu parametre adını kullanan tüm rotalara otomatik olarak uygulanır:
```php
Route::get('/user/{id}', function (string $id) {
    // Yalnızca {id} sayısal ise yürütülür...
});
```

#### Kodlanmış Eğik Çizgiler (Encoded Forward Slashes)
Laravel yönlendirme bileşeni, rota parametre değerleri içinde `/` dışındaki tüm karakterlerin bulunmasına izin verir. Bir `where` koşulu düzenli ifadesi kullanarak `/` işaretinin yer tutucunuzun (placeholder) bir parçası olmasına açıkça izin vermelisiniz:
```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```
Kodlanmış eğik çizgiler yalnızca son rota bölümünde (segment) desteklenir.

## Adlandırılmış Rotalar (Named Routes)

Adlandırılmış rotalar, belirli rotalar için uygun URL veya yönlendirme oluşturulmasına olanak tanır. Rota tanımına `name` metodunu zincirleyerek bir rota için bir ad belirtebilirsiniz:
```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```
Controller eylemleri için de rota adları belirtebilirsiniz:
```php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```
Rota adları her zaman benzersiz (unique) olmalıdır.

#### Adlandırılmış Rotalara URL Oluşturma (Generating URLs to Named Routes)
Belirli bir rotaya bir ad atadıktan sonra, Laravel'in `route` ve `redirect` yardımcı fonksiyonları aracılığıyla URL'ler veya yönlendirmeler oluştururken rota adını kullanabilirsiniz:
```php
// URL oluşturma...
$url = route('profile');

// Yönlendirme oluşturma...
return redirect()->route('profile');

return to_route('profile');
```
Adlandırılmış rota parametreler tanımlıyorsa, bu parametreleri `route` fonksiyonuna ikinci argüman olarak iletebilirsiniz. Verilen parametreler, oluşturulan URL'ye otomatik olarak doğru konumlarına yerleştirilecektir:
```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);
```
Dizide ek parametreler iletirseniz, bu anahtar/değer çiftleri otomatik olarak oluşturulan URL'nin sorgu dizesine (query string) eklenecektir:
```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// http://example.com/user/1/profile?photos=yes
```
Bazen, geçerli yerel ayar (locale) gibi URL parametreleri için istek genelinde varsayılan değerler belirtmek isteyebilirsiniz. Bunu başarmak için `URL::defaults` metodunu kullanabilirsiniz.

#### Geçerli Rotayı İnceleme (Inspecting the Current Route)
Geçerli isteğin belirli bir adlandırılmış rotaya yönlendirilip yönlendirilmediğini belirlemek isterseniz, bir `Route` örneği üzerindeki `named` metodunu kullanabilirsiniz. Örneğin, geçerli rota adını bir rota ara yazılımından (middleware) kontrol edebilirsiniz:
```php
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use Symfony\Component\HttpFoundation\Response;

/**
 * Gelen isteği işle.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return \Symfony\Component\HttpFoundation\Response
 */
public function handle(Request $request, Closure $next): Response
{
    if (Route::currentRouteName() === 'profile') {
        // ...
    }

    // Veya mevcut rotanın belirli bir ada sahip olup olmadığını kontrol edin...
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```