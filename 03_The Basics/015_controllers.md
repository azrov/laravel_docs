# Laravel 12 Dokümantasyonu: Controller'lar (Denetleyiciler)

## Giriş

Tüm istek işleme mantığınızı (request handling logic) rota dosyalarınızda kapatmalar (closures) olarak tanımlamak yerine, bu davranışı "controller" sınıflarını kullanarak düzenlemek isteyebilirsiniz. Controller'lar, ilgili istek işleme mantığını tek bir sınıfta gruplayabilir. Örneğin, bir `UserController` sınıfı, kullanıcıları gösterme, oluşturma, güncelleme ve silme dahil olmak üzere kullanıcılarla ilgili tüm gelen istekleri işleyebilir. Varsayılan olarak, controller'lar `app/Http/Controllers` dizininde saklanır.

## Controller Yazma (Writing Controllers)

### Temel Controller'lar (Basic Controllers)
Hızlı bir şekilde yeni bir controller oluşturmak için `make:controller` Artisan komutunu çalıştırabilirsiniz. Varsayılan olarak, uygulamanız için tüm controller'lar `app/Http/Controllers` dizininde saklanır:
```bash
php artisan make:controller UserController
```
Temel bir controller örneğine bakalım. Bir controller, gelen HTTP isteklerine yanıt verecek herhangi bir sayıda genel (public) metoda sahip olabilir:
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Belirli bir kullanıcının profilini göster.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```
Bir controller sınıfı ve metodu yazdıktan sonra, controller metoduna giden bir rotayı şu şekilde tanımlayabilirsiniz:
```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```
Gelen bir istek belirtilen rota URI'si ile eşleştiğinde, `App\Http\Controllers\UserController` sınıfındaki `show` metodu çağrılacak ve rota parametreleri metoda iletilecektir.

Controller'ların bir temel sınıfı (base class) genişletmesi (extend) gerekmez. Ancak, tüm controller'larınız arasında paylaşılması gereken metotları içeren bir temel controller sınıfını genişletmek bazen kullanışlı olabilir.

### Tek Eylemli Controller'lar (Single Action Controllers)
Bir controller eylemi (action) özellikle karmaşıksa, tüm bir controller sınıfını bu tek eyleme ayırmanın kullanışlı olduğunu görebilirsiniz. Bunu başarmak için, controller içinde tek bir `__invoke` metodu tanımlayabilirsiniz:
```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Yeni bir web sunucusu sağla (Provision).
     */
    public function __invoke()
    {
        // ...
    }
}
```
Tek eylemli controller'lar için rota kaydederken, bir controller metodu belirtmeniz gerekmez. Bunun yerine, controller'ın adını doğrudan yönlendiriciye (router) iletebilirsiniz:
```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```
`make:controller` Artisan komutunun `--invokable` seçeneğini kullanarak çağrılabilir (invokable) bir controller oluşturabilirsiniz:
```bash
php artisan make:controller ProvisionServer --invokable
```
Controller iskeletleri (stubs), iskelet yayınlama (stub publishing) kullanılarak özelleştirilebilir.

## Controller Ara Yazılımları (Controller Middleware)

Ara yazılımlar (middleware), rota dosyalarınızda controller'ın rotalarına atanabilir:
```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```
Veya, ara yazılımı controller sınıfınız içinde belirtmenin daha kullanışlı olduğunu görebilirsiniz. Bunu yapmak için, controller'ınızın `HasMiddleware` arayüzünü (interface) uygulaması (implement) gerekir; bu arayüz, controller'ın statik bir `middleware` metoduna sahip olması gerektiğini belirtir. Bu metottan, controller'ın eylemlerine uygulanması gereken bir ara yazılım dizisi döndürebilirsiniz:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
{
    /**
     * Controller'a atanması gereken ara yazılımları al.
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }

    // ...
}
```
Controller ara yazılımlarını kapatmalar (closures) olarak da tanımlayabilirsiniz; bu, tüm bir ara yazılım sınıfı yazmadan satır içi (inline) bir ara yazılım tanımlamanın kullanışlı bir yolunu sağlar:
```php
use Closure;
use Illuminate\Http\Request;

/**
 * Controller'a atanması gereken ara yazılımları al.
 */
public static function middleware(): array
{
    return [
        function (Request $request, Closure $next) {
            return $next($request);
        },
    ];
}
```

## Kaynak Controller'lar (Resource Controllers)

Uygulamanızdaki her Eloquent modelini bir "kaynak" (resource) olarak düşünürseniz, uygulamanızdaki her kaynağa karşı aynı eylem setlerini gerçekleştirmek tipiktir. Örneğin, uygulamanızın bir `Photo` modeli ve bir `Movie` modeli içerdiğini hayal edin. Kullanıcıların bu kaynakları oluşturabilmesi, okuyabilmesi, güncelleyebilmesi veya silebilmesi ("CRUD") muhtemeldir.

Bu yaygın kullanım durumu nedeniyle, Laravel kaynak yönlendirmesi (resource routing), tipik oluşturma (create), okuma (read), güncelleme (update) ve silme ("CRUD") rotalarını tek bir kod satırıyla bir controller'a atar. Başlamak için, bu eylemleri işleyecek bir controller'ı hızlıca oluşturmak amacıyla `make:controller` Artisan komutunun `--resource` seçeneğini kullanabiliriz:
```bash
php artisan make:controller PhotoController --resource
```
Bu komut, `app/Http/Controllers/PhotoController.php` konumunda bir controller oluşturacaktır. Controller, mevcut kaynak işlemlerinin her biri için bir metot içerecektir. Ardından, controller'ı işaret eden bir kaynak rotası kaydedebilirsiniz:
```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```
Bu tek rota bildirimi, kaynak üzerinde çeşitli eylemleri işlemek için birden çok rota oluşturur. Oluşturulan controller, bu eylemlerin her biri için önceden hazırlanmış (stubbed) metotlara zaten sahip olacaktır. Unutmayın, uygulamanızın rotalarının hızlı bir özetini her zaman `route:list` Artisan komutunu çalıştırarak alabilirsiniz.

Hatta `resources` metoduna bir dizi ileterek aynı anda birden çok kaynak controller'ı kaydedebilirsiniz:
```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```
`softDeletableResources` metodu, tamamı `withTrashed` metodunu kullanan birçok kaynak controller'ı kaydeder:
```php
Route::softDeletableResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

#### Kaynak Controller'lar Tarafından İşlenen Eylemler (Actions Handled by Resource Controllers)

| Fiil (Verb) | URI | Eylem (Action) | Rota Adı (Route Name) |
| :--- | :--- | :--- | :--- |
| GET | `/photos` | index | photos.index |
| GET | `/photos/create` | create | photos.create |
| POST | `/photos` | store | photos.store |
| GET | `/photos/{photo}` | show | photos.show |
| GET | `/photos/{photo}/edit` | edit | photos.edit |
| PUT/PATCH | `/photos/{photo}` | update | photos.update |
| DELETE | `/photos/{photo}` | destroy | photos.destroy |

#### Eksik Model Davranışını Özelleştirme (Customizing Missing Model Behavior)
Tipik olarak, örtülü olarak bağlanmış (implicitly bound) bir kaynak modeli bulunamazsa 404 HTTP yanıtı oluşturulur. Ancak, kaynak rotanızı tanımlarken `missing` metodunu çağırarak bu davranışı özelleştirebilirsiniz. `missing` metodu, kaynağın rotalarından herhangi biri için örtülü olarak bağlanmış bir model bulunamazsa çağrılacak bir kapatma (closure) kabul eder:
```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::resource('photos', PhotoController::class)
    ->missing(function (Request $request) {
        return Redirect::route('photos.index');
    });
```

#### Yazılı Olarak Silinmiş Modeller (Soft Deleted Models)
Tipik olarak, örtülü model bağlama (implicit model binding), yazılı olarak silinmiş (soft deleted) modelleri almaz ve bunun yerine 404 HTTP yanıtı döndürür. Ancak, kaynak rotanızı tanımlarken `withTrashed` metodunu çağırarak framework'e yazılı olarak silinmiş modellere izin vermesi talimatını verebilirsiniz:
```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```
`withTrashed`'ı argümansız çağırmak, `show`, `edit` ve `update` kaynak rotaları için yazılı olarak silinmiş modellere izin verecektir. `withTrashed` metoduna bir dizi ileterek bu rotaların bir alt kümesini belirtebilirsiniz:
```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

#### Kaynak Modelini Belirtme (Specifying the Resource Model)
Rota model bağlama (route model binding) kullanıyorsanız ve kaynak controller'ın metotlarının bir model örneğini tip belirtmesini (type-hint) istiyorsanız, controller'ı oluştururken `--model` seçeneğini kullanabilirsiniz:
```bash
php artisan make:controller PhotoController --model=Photo --resource
```

#### Form İstekleri (Form Requests) Oluşturma
Bir kaynak controller oluştururken `--requests` seçeneğini sağlayarak Artisan'a controller'ın `storage` ve `update` metotları için form isteği (form request) sınıfları oluşturması talimatını verebilirsiniz:
```bash
php artisan make:controller PhotoController --model=Photo --resource --requests
```

### Kısmi Kaynak Rotaları (Partial Resource Routes)
Bir kaynak rotası bildirirken, controller'ın varsayılan eylemlerin tam seti yerine işlemesi gereken eylemlerin bir alt kümesini belirtebilirsiniz:
```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

#### API Kaynak Rotaları (API Resource Routes)
API'ler tarafından tüketilecek kaynak rotaları bildirirken, genellikle `create` ve `edit` gibi HTML şablonları sunan rotaları hariç tutmak isteyeceksiniz. Kolaylık olması açısından, bu iki rotayı otomatik olarak hariç tutmak için `apiResource` metodunu kullanabilirsiniz:
```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```
`apiResources` metoduna bir dizi ileterek aynı anda birden çok API kaynak controller'ı kaydedebilirsiniz:
```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```
`create` veya `edit` metotlarını içermeyen bir API kaynak controller'ı hızlıca oluşturmak için `make:controller` komutunu çalıştırırken `--api` anahtarını kullanın:
```bash
php artisan make:controller PhotoController --api
```

### İç İçe Kaynaklar (Nested Resources)
Bazen iç içe geçmiş bir kaynağa (nested resource) yönelik rotalar tanımlamanız gerekebilir. Örneğin, bir fotoğraf kaynağına eklenmiş birden çok yorum (comment) olabilir. Kaynak controller'larını iç içe yerleştirmek için rota bildiriminizde "nokta" notasyonunu kullanabilirsiniz:
```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```
Bu rota, aşağıdaki gibi URI'lerle erişilebilen iç içe bir kaynak kaydedecektir:
```
/photos/{photo}/comments/{comment}
```

#### İç İçe Kaynakları Kapsamlandırma (Scoping Nested Resources)
Laravel'in örtülü model bağlama (implicit model binding) özelliği, çözümlenen (resolve edilen) alt modelin üst modele ait olduğunu doğrulayacak şekilde iç içe bağlamaları otomatik olarak kapsamlandırabilir (scope). İç içe kaynağınızı tanımlarken `scoped` metodunu kullanarak otomatik kapsamlandırmayı etkinleştirebilir ve Laravel'e alt kaynağın hangi alana (field) göre alınması gerektiğini söyleyebilirsiniz. Bunun nasıl yapılacağı hakkında daha fazla bilgi için lütfen kaynak rotalarını kapsamlandırma (scoping resource routes) dokümantasyonuna bakın.

#### Sığ İç İçe Yerleştirme (Shallow Nesting)
Genellikle, alt ID zaten benzersiz bir tanımlayıcı (unique identifier) olduğu için bir URI içinde hem üst hem de alt ID'lere sahip olmak tamamen gerekli değildir. Modellerinizi URI bölümlerinde (segments) tanımlamak için otomatik artan birincil anahtarlar (auto-incrementing primary keys) gibi benzersiz tanımlayıcılar kullanırken, "sığ iç içe yerleştirme" (shallow nesting) kullanmayı seçebilirsiniz:
```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```
Bu rota tanımı aşağıdaki rotaları tanımlayacaktır:

| Fiil (Verb) | URI | Eylem (Action) | Rota Adı (Route Name) |
| :--- | :--- | :--- | :--- |
| GET | `/photos/{photo}/comments` | index | photos.comments.index |
| GET | `/photos/{photo}/comments/create` | create | photos.comments.create |
| POST | `/photos/{photo}/comments` | store | photos.comments.store |
| GET | `/comments/{comment}` | show | comments.show |
| GET | `/comments/{comment}/edit` | edit | comments.edit |
| PUT/PATCH | `/comments/{comment}` | update | comments.update |
| DELETE | `/comments/{comment}` | destroy | comments.destroy |

### Kaynak Rotalarını Adlandırma (Naming Resource Routes)
Varsayılan olarak, tüm kaynak controller eylemlerinin bir rota adı vardır; ancak, istediğiniz rota adlarıyla bir `names` dizisi ileterek bu adları geçersiz kılabilirsiniz (override):
```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

### Kaynak Rota Parametrelerini Adlandırma (Naming Resource Route Parameters)
Varsayılan olarak, `Route::resource`, kaynak rotalarınız için rota parametrelerini kaynak adının "tekil hale getirilmiş" (singularized) versiyonuna göre oluşturacaktır. Bunu, `parameters` metodunu kullanarak kaynak bazında kolayca geçersiz kılabilirsiniz. `parameters` metoduna iletilen dizi, kaynak adları ve parametre adlarından oluşan bir ilişkisel dizi (associative array) olmalıdır:
```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```
Yukarıdaki örnek, kaynağın `show` rotası için aşağıdaki URI'yi oluşturur:
```
/users/{admin_user}
```

### Kaynak Rotalarını Kapsamlandırma (Scoping Resource Routes)
Laravel'in kapsamlı örtülü model bağlama (scoped implicit model binding) özelliği, çözümlenen alt modelin üst modele ait olduğunu doğrulayacak şekilde iç içe bağlamaları otomatik olarak kapsamlandırabilir. İç içe kaynağınızı tanımlarken `scoped` metodunu kullanarak otomatik kapsamlandırmayı etkinleştirebilir ve Laravel'e alt kaynağın hangi alana göre alınması gerektiğini söyleyebilirsiniz:
```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```
Bu rota, aşağıdaki gibi URI'lerle erişilebilen kapsamlı bir iç içe kaynak kaydedecektir:
```
/photos/{photo}/comments/{comment:slug}
```
İç içe bir rota parametresi olarak özel anahtarlı (custom keyed) örtülü bağlama kullanıldığında, Laravel, üzerindeki ilişki adını (relationship name) tahmin etmek için kuralları (conventions) kullanarak alt modeli üst modeline göre alacak şekilde sorguyu otomatik olarak kapsamlandıracaktır (scope). Bu durumda, `Photo` modelinin, `Comment` modelini almak için kullanılabilecek `comments` (rota parametresi adının çoğulu) adında bir ilişkisi olduğu varsayılacaktır.

### Kaynak URI'lerini Yerelleştirme (Localizing Resource URIs)
Varsayılan olarak, `Route::resource`, kaynak URI'lerini İngilizce fiiller (verbs) ve çoğul kuralları (plural rules) kullanarak oluşturacaktır. `create` ve `edit` eylem fiillerini yerelleştirmeniz gerekiyorsa, `Route::resourceVerbs` metodunu kullanabilirsiniz. Bu, bir servis sağlayıcının (service provider) `boot` metodunda yapılabilir:
```php
/**
 * Herhangi bir uygulama servisini önyükle.
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'olustur',
        'edit' => 'duzenle',
    ]);
}
```
Fiiller özelleştirildikten sonra, `/photos/create` gibi bir kaynak rotası kaydı, `/photos/olustur` URI'sine yanıt verecektir.