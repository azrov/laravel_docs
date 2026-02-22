# Laravel 12 Dokümantasyonu: Servis Sağlayıcılar (Service Providers)

## Giriş

Servis sağlayıcılar (service providers), tüm Laravel uygulaması başlatma (bootstrapping) işlemlerinin merkezi yeridir. Kendi uygulamanız ve Laravel'in tüm temel servisleri, servis sağlayıcılar aracılığıyla başlatılır.

Peki, "başlatma" (bootstrapped) ile ne demek istiyoruz? Genel olarak, servis kabı bağlamalarını (service container bindings), olay dinleyicilerini (event listeners), ara yazılımları (middleware) ve hatta rotaları (routes) kaydetmek gibi şeyleri kaydetmeyi kastediyoruz. Servis sağlayıcılar, uygulamanızı yapılandırmak için merkezi yerdir.

Laravel, posta gönderici (mailer), kuyruk (queue), önbellek (cache) ve diğerleri gibi temel servislerini başlatmak için dahili olarak düzinelerce servis sağlayıcı kullanır. Bu sağlayıcıların çoğu "ertelenmiş" (deferred) sağlayıcılardır, yani her istekte yüklenmeyecekler, yalnızca sağladıkları servislere gerçekten ihtiyaç duyulduğunda yükleneceklerdir.

Tüm kullanıcı tanımlı servis sağlayıcılar `bootstrap/providers.php` dosyasında kaydedilir. Aşağıdaki dokümantasyonda, kendi servis sağlayıcılarınızı nasıl yazacağınızı ve bunları Laravel uygulamanıza nasıl kaydedeceğinizi öğreneceksiniz.

Laravel'in istekleri nasıl işlediği ve dahili olarak nasıl çalıştığı hakkında daha fazla bilgi edinmek isterseniz, Laravel istek yaşam döngüsü ile ilgili dokümantasyonumuza göz atın.

## Servis Sağlayıcı Yazma (Writing Service Providers)

Tüm servis sağlayıcılar `Illuminate\Support\ServiceProvider` sınıfını genişletir (extend). Çoğu servis sağlayıcı bir `register` ve bir `boot` metodu içerir. `register` metodu içinde, yalnızca şeyleri servis kabına (service container) bağlamalısınız (bind). Asla `register` metodu içinde herhangi bir olay dinleyicisi (event listener), rota (route) veya başka herhangi bir işlevsellik parçasını kaydetmeye çalışmamalısınız.

Artisan CLI, `make:provider` komutu aracılığıyla yeni bir sağlayıcı oluşturabilir. Laravel, yeni sağlayıcınızı otomatik olarak uygulamanızın `bootstrap/providers.php` dosyasına kaydedecektir:
```bash
php artisan make:provider RiakServiceProvider
```

### Register Metodu (The Register Method)

Daha önce belirtildiği gibi, `register` metodu içinde yalnızca şeyleri servis kabına bağlamalısınız. Asla `register` metodu içinde herhangi bir olay dinleyicisi, rota veya başka herhangi bir işlevsellik parçasını kaydetmeye çalışmamalısınız. Aksi takdirde, henüz yüklenmemiş bir servis sağlayıcı tarafından sağlanan bir servisi yanlışlıkla kullanabilirsiniz.

Basit bir servis sağlayıcıya bakalım. Herhangi bir servis sağlayıcı metodunuzun içinde, servis kabına erişim sağlayan `$app` özelliğine her zaman erişebilirsiniz:
```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini kaydet.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```
Bu servis sağlayıcı yalnızca bir `register` metodu tanımlar ve bu metodu, servis kabında `App\Services\Riak\Connection`'ın bir gerçekleştirimini (implementation) tanımlamak için kullanır. Eğer Laravel'in servis kabına henüz aşina değilseniz, dokümantasyonuna göz atın.

#### bindings ve singletons Özellikleri
Eğer servis sağlayıcınız birçok basit bağlama (binding) kaydediyorsa, her bir container bağlamasını manuel olarak kaydetmek yerine `bindings` ve `singletons` özelliklerini kullanmak isteyebilirsiniz. Sağlayıcı framework tarafından yüklendiğinde, otomatik olarak bu özellikleri kontrol edecek ve bağlamalarını kaydedecektir:
```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Kaydedilmesi gereken tüm container bağlamaları.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * Kaydedilmesi gereken tüm container tekil (singleton) bağlamaları.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

### Boot Metodu (The Boot Method)

Peki ya servis sağlayıcımız içinde bir görünüm oluşturucuyu (view composer) kaydetmemiz gerekirse? Bu, `boot` metodu içinde yapılmalıdır. Bu metod, diğer tüm servis sağlayıcılar kaydedildikten sonra çağrılır, yani framework tarafından kaydedilmiş diğer tüm servislere erişebileceğiniz anlamına gelir:
```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

#### Boot Metodunda Bağımlılık Enjeksiyonu
Servis sağlayıcınızın `boot` metodu için bağımlılıkları tip belirtebilirsiniz (type-hint). Servis kabı, ihtiyacınız olan tüm bağımlılıkları otomatik olarak enjekte edecektir:
```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

## Sağlayıcıları Kaydetme (Registering Providers)

Tüm servis sağlayıcılar `bootstrap/providers.php` yapılandırma dosyasında kaydedilir. Bu dosya, uygulamanızın servis sağlayıcılarının sınıf adlarını içeren bir dizi (array) döndürür:
```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```
`make:provider` Artisan komutunu çağırdığınızda, Laravel otomatik olarak oluşturulan sağlayıcıyı `bootstrap/providers.php` dosyasına ekleyecektir. Ancak, sağlayıcı sınıfını manuel olarak oluşturduysanız, sağlayıcı sınıfını diziye manuel olarak eklemelisiniz:
```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class, // Manuel olarak eklendi
];
```

## Ertelenmiş Sağlayıcılar (Deferred Providers)

Eğer sağlayıcınız yalnızca servis kabına bağlamalar (bindings) kaydediyorsa, kayıtlı bağlamalardan birine gerçekten ihtiyaç duyulana kadar kaydını ertelemeyi seçebilirsiniz. Böyle bir sağlayıcının yüklenmesini ertelemek, uygulamanızın performansını artıracaktır, çünkü her istekte dosya sisteminden yüklenmez.

Laravel, ertelenmiş servis sağlayıcıları tarafından sağlanan tüm servislerin bir listesini, servis sağlayıcı sınıfının adıyla birlikte derler ve saklar. Ardından, yalnızca bu servislerden birini çözümlemeye (resolve) çalıştığınızda Laravel servis sağlayıcıyı yükler.

Bir sağlayıcının yüklenmesini ertelemek için `\Illuminate\Contracts\Support\DeferrableProvider` arayüzünü (interface) uygulayın (implement) ve bir `provides` metodu tanımlayın. `provides` metodu, sağlayıcı tarafından kaydedilen servis kabı bağlamalarını döndürmelidir:
```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Herhangi bir uygulama servisini kaydet.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Sağlayıcı tarafından sağlanan servisleri al.
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```