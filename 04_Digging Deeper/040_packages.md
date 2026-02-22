# Laravel 12 Dokümantasyonu: Paket Geliştirme (Package Development)

## Giriş

Paketler, Laravel'e işlevsellik eklemenin birincil yoludur. Paketler, Carbon gibi tarihlerle çalışmak için harika bir yoldan, Spatie'nin Laravel Media Library'si gibi dosyaları Eloquent modelleriyle ilişkilendirmenize olanak tanıyan bir pakete kadar her şey olabilir.

Farklı türde paketler vardır. Bazı paketler bağımsızdır (stand-alone), yani herhangi bir PHP framework'ü ile çalışırlar. Carbon ve Pest, bağımsız paketlere örnektir. Bu paketlerden herhangi biri, `composer.json` dosyanızda gerektirilerek (require) Laravel ile kullanılabilir.

Öte yandan, diğer paketler özellikle Laravel ile kullanılmak üzere tasarlanmıştır. Bu paketler, bir Laravel uygulamasını geliştirmek için özel olarak tasarlanmış rotalar (routes), controller'lar, görünümler (views) ve yapılandırmaya (configuration) sahip olabilir. Bu kılavuz öncelikle Laravel'e özgü olan bu paketlerin geliştirilmesini kapsar.

### Facade'ler Hakkında Bir Not (A Note on Facades)
Bir Laravel uygulaması yazarken, sözleşmeler (contracts) veya facade'ler kullanmanız genellikle önemli değildir, çünkü her ikisi de temelde eşit düzeyde test edilebilirlik (testability) sağlar. Ancak, paketler yazarken, paketiniz tipik olarak Laravel'in tüm test yardımcılarına (testing helpers) erişime sahip olmayacaktır. Paketinizi, tipik bir Laravel uygulaması içine kurulmuş gibi testler yazabilmek isterseniz, Orchestral Testbench paketini kullanabilirsiniz.

## Paket Keşfi (Package Discovery)

Bir Laravel uygulamasının `bootstrap/providers.php` dosyası, Laravel tarafından yüklenmesi gereken servis sağlayıcıların (service providers) listesini içerir. Ancak, kullanıcıların servis sağlayıcınızı listeye manuel olarak eklemesini gerektirmek yerine, sağlayıcıyı paketinizin `composer.json` dosyasının `extra` bölümünde tanımlayabilirsiniz, böylece Laravel tarafından otomatik olarak yüklenir. Servis sağlayıcılara ek olarak, kaydedilmesini istediğiniz tüm facade'leri de listeleyebilirsiniz:
```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
}
```
Paketiniz keşif (discovery) için yapılandırıldıktan sonra, Laravel kurulduğunda servis sağlayıcılarını ve facade'lerini otomatik olarak kaydedecek ve paketinizin kullanıcıları için kullanışlı bir kurulum deneyimi oluşturacaktır.

#### Paket Keşfini Devre Dışı Bırakma (Opting Out of Package Discovery)
Bir paketin tüketicisi (consumer) iseniz ve bir paket için paket keşfini devre dışı bırakmak isterseniz, paket adını uygulamanızın `composer.json` dosyasının `extra` bölümünde listeleyebilirsiniz:
```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
}
```
Uygulamanızın `dont-discover` yönergesi içinde `*` karakterini kullanarak tüm paketler için paket keşfini devre dışı bırakabilirsiniz:
```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
}
```

## Servis Sağlayıcılar (Service Providers)

Servis sağlayıcılar, paketiniz ile Laravel arasındaki bağlantı noktasıdır. Bir servis sağlayıcı, şeyleri Laravel'in servis kabına (service container) bağlamaktan ve Laravel'e paket kaynaklarını (görünümler, yapılandırma ve dil dosyaları gibi) nerede yükleyeceğini bildirmekten sorumludur.

Bir servis sağlayıcı, `Illuminate\Support\ServiceProvider` sınıfını genişletir (extends) ve iki metot içerir: `register` ve `boot`. Temel `ServiceProvider` sınıfı, kendi paketinizin bağımlılıklarına eklemeniz gereken `illuminate/support` Composer paketinde bulunur. Servis sağlayıcıların yapısı ve amacı hakkında daha fazla bilgi edinmek için dokümantasyonlarına göz atın.

## Kaynaklar (Resources)

### Yapılandırma (Configuration)
Genellikle, paketinizin yapılandırma dosyasını uygulamanın `config` dizinine yayınlamanız (publish) gerekecektir. Bu, paketinizin kullanıcılarının varsayılan yapılandırma seçeneklerinizi kolayca geçersiz kılmasına (override) olanak tanır. Yapılandırma dosyalarınızın yayınlanmasına izin vermek için, servis sağlayıcınızın `boot` metodundan `publishes` metodunu çağırın:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/courier.php' => config_path('courier.php'),
    ]);
}
```
Artık, paketinizin kullanıcıları Laravel'in `vendor:publish` komutunu çalıştırdığında, dosyanız belirtilen yayınlama konumuna kopyalanacaktır. Yapılandırmanız yayınlandıktan sonra, değerlerine diğer tüm yapılandırma dosyaları gibi erişilebilir:
```php
$value = config('courier.option');
```
Yapılandırma dosyalarınızda closure'lar tanımlamamalısınız. Kullanıcılar `config:cache` Artisan komutunu çalıştırdığında doğru şekilde serileştirilemezler (serialized).

#### Varsayılan Paket Yapılandırması (Default Package Configuration)
Ayrıca kendi paket yapılandırma dosyanızı uygulamanın yayınlanmış kopyasıyla birleştirebilirsiniz (merge). Bu, kullanıcılarınızın yapılandırma dosyasının yayınlanmış kopyasında yalnızca gerçekten geçersiz kılmak istedikleri seçenekleri tanımlamasına olanak tanır. Yapılandırma dosyası değerlerini birleştirmek için, servis sağlayıcınızın `register` metodu içinde `mergeConfigFrom` metodunu kullanın.

`mergeConfigFrom` metodu, ilk argüman olarak paketinizin yapılandırma dosyasının yolunu ve ikinci argüman olarak uygulamanın yapılandırma dosyası kopyasının adını kabul eder:
```php
/**
 * Herhangi bir paket servisini kaydet.
 */
public function register(): void
{
    $this->mergeConfigFrom(
        __DIR__.'/../config/courier.php', 'courier'
    );
}
```
Bu metod yalnızca yapılandırma dizisinin ilk seviyesini birleştirir. Kullanıcılarınız çok boyutlu bir yapılandırma dizisini kısmen tanımlarsa, eksik seçenekler birleştirilmeyecektir.

### Rotalar (Routes)
Paketiniz rotalar içeriyorsa, bunları `loadRoutesFrom` metodunu kullanarak yükleyebilirsiniz. Bu metod, uygulamanın rotalarının önbelleğe alınıp alınmadığını otomatik olarak belirleyecek ve rotalar zaten önbelleğe alınmışsa rota dosyanızı yüklemeyecektir:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}
```

### Migration'lar (Migrations)
Paketiniz veritabanı migration'ları içeriyorsa, verilen dizinin veya dosyanın migration'lar içerdiğini Laravel'e bildirmek için `publishesMigrations` metodunu kullanabilirsiniz. Laravel migration'ları yayınlarken, geçerli tarih ve saati yansıtacak şekilde dosya adlarındaki zaman damgasını (timestamp) otomatik olarak güncelleyecektir:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->publishesMigrations([
        __DIR__.'/../database/migrations' => database_path('migrations'),
    ]);
}
```

### Dil Dosyaları (Language Files)
Paketiniz dil dosyaları içeriyorsa, bunların nasıl yükleneceğini Laravel'e bildirmek için `loadTranslationsFrom` metodunu kullanabilirsiniz. Örneğin, paketinizin adı `courier` ise, servis sağlayıcınızın `boot` metoduna aşağıdakini eklemelisiniz:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
}
```
Paket çeviri satırlarına, `package::file.line` sözdizimi kuralı kullanılarak başvurulur. Bu nedenle, `courier` paketinin `messages` dosyasındaki `welcome` satırını şu şekilde yükleyebilirsiniz:
```php
echo trans('courier::messages.welcome');
```
Paketiniz için JSON çeviri dosyalarını `loadJsonTranslationsFrom` metodunu kullanarak kaydedebilirsiniz. Bu metod, paketinizin JSON çeviri dosyalarını içeren dizinin yolunu kabul eder:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->loadJsonTranslationsFrom(__DIR__.'/../lang');
}
```

#### Dil Dosyalarını Yayınlama (Publishing Language Files)
Paketinizin dil dosyalarını uygulamanın `lang/vendor` dizinine yayınlamak isterseniz, servis sağlayıcının `publishes` metodunu kullanabilirsiniz. `publishes` metodu, paket yollarının ve bunların istenen yayınlama konumlarının bir dizisini kabul eder. Örneğin, `courier` paketi için dil dosyalarını yayınlamak üzere aşağıdakileri yapabilirsiniz:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

    $this->publishes([
        __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
    ]);
}
```
Artık, paketinizin kullanıcıları Laravel'in `vendor:publish` Artisan komutunu çalıştırdığında, paketinizin dil dosyaları belirtilen yayınlama konumuna yayınlanacaktır.

### Görünümler (Views)
Paketinizin görünümlerini Laravel'e kaydetmek için Laravel'e görünümlerin nerede bulunduğunu söylemeniz gerekir. Bunu, servis sağlayıcının `loadViewsFrom` metodunu kullanarak yapabilirsiniz. `loadViewsFrom` metodu iki argüman kabul eder: görünüm şablonlarınızın yolu ve paketinizin adı. Örneğin, paketinizin adı `courier` ise, servis sağlayıcınızın `boot` metoduna aşağıdakini eklemelisiniz:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
}
```
Paket görünümlerine, `package::view` sözdizimi kuralı kullanılarak başvurulur. Bu nedenle, görünüm yolunuz bir servis sağlayıcıda kaydedildikten sonra, `courier` paketinden `dashboard` görünümünü şu şekilde yükleyebilirsiniz:
```php
Route::get('/dashboard', function () {
    return view('courier::dashboard');
});
```

#### Paket Görünümlerini Geçersiz Kılma (Overriding Package Views)
`loadViewsFrom` metodunu kullandığınızda, Laravel aslında görünümleriniz için iki konum kaydeder: uygulamanın `resources/views/vendor` dizini ve sizin belirttiğiniz dizin. Bu nedenle, `courier` paketini örnek alırsak, Laravel önce görünümün özelleştirilmiş bir sürümünün geliştirici tarafından `resources/views/vendor/courier` dizinine yerleştirilip yerleştirilmediğini kontrol edecektir. Ardından, görünüm özelleştirilmemişse, Laravel `loadViewsFrom` çağrınızda belirttiğiniz paket görünüm dizinini arayacaktır. Bu, paket kullanıcılarının paketinizin görünümlerini özelleştirmesini / geçersiz kılmasını kolaylaştırır.

#### Görünümleri Yayınlama (Publishing Views)
Görünümlerinizi uygulamanın `resources/views/vendor` dizinine yayınlanabilir hale getirmek isterseniz, servis sağlayıcının `publishes` metodunu kullanabilirsiniz. `publishes` metodu, paket görünüm yollarının ve bunların istenen yayınlama konumlarının bir dizisini kabul eder:
```php
/**
 * Paket servislerini önyükle.
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
    ]);
}
```
Artık, paketinizin kullanıcıları Laravel'in `vendor:publish` Artisan komutunu çalıştırdığında, paketinizin görünümleri belirtilen yayınlama konumuna kopyalanacaktır.

### Görünüm Bileşenleri (View Components)
Blade bileşenlerini kullanan veya bileşenleri geleneksel olmayan dizinlere yerleştiren bir paket oluşturuyorsanız, Laravel'in bileşeni nerede bulacağını bilmesi için bileşen sınıfınızı ve HTML etiket takma adını (tag alias) manuel olarak kaydetmeniz gerekecektir. Bileşenlerinizi tipik olarak paketinizin servis sağlayıcısının `boot` metodunda kaydetmelisiniz:
```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Paketinizin servislerini önyükle.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```
Bileşeniniz kaydedildikten sonra, etiket takma adı kullanılarak oluşturulabilir:
```blade
<x-package-alert/>
```

#### Paket Bileşenlerini Otomatik Yükleme (Autoloading Package Components)
Alternatif olarak, kural gereği (by convention) bileşen sınıflarını otomatik yüklemek için `componentNamespace` metodunu kullanabilirsiniz. Örneğin, bir `Nightshade` paketi, `Nightshade\Views\Components` ad alanı (namespace) içinde bulunan `Calendar` ve `ColorPicker` bileşenlerine sahip olabilir:
```php
use Illuminate\Support\Facades\Blade;

/**
 * Paketinizin servislerini önyükle.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```
Bu, paket bileşenlerinin, `package-name::` sözdizimi kullanılarak satıcı ad alanları (vendor namespace) aracılığıyla kullanılmasına izin verecektir:
```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```
Blade, bileşen adını pascal-case kullanarak bu bileşene bağlı sınıfı otomatik olarak algılayacaktır. Alt dizinler de "nokta" notasyonu kullanılarak desteklenir.

#### Anonim Bileşenler (Anonymous Components)
Paketiniz anonim bileşenler içeriyorsa, bunlar paketinizin "görünümler" dizininin (`loadViewsFrom` metodu tarafından belirtilen) bir `components` dizinine yerleştirilmelidir. Ardından, bileşen adını paketin görünüm ad alanıyla (view namespace) ön ekleyerek (prefix) onları oluşturabilirsiniz:
```blade
<x-courier::alert />
```

### "About" Artisan Komutu
Laravel'in yerleşik `about` Artisan komutu, uygulamanın ortamı ve yapılandırması hakkında bir özet (synopsis) sağlar. Paketler, `AboutCommand` sınıfı aracılığıyla bu komutun çıktısına ek bilgi ekleyebilir. Tipik olarak, bu bilgi paket servis sağlayıcınızın `boot` metodundan eklenebilir:
```php
use Illuminate\Foundation\Console\AboutCommand;

/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
}
```

## Komutlar (Commands)

Paketinizin Artisan komutlarını Laravel'e kaydetmek için `commands` metodunu kullanabilirsiniz. Bu metod, bir komut sınıf adları dizisi bekler. Komutlar kaydedildikten sonra, Artisan CLI kullanılarak yürütülebilirler:
```php
use Courier\Console\Commands\InstallCommand;
use Courier\Console\Commands\NetworkCommand;

/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            InstallCommand::class,
            NetworkCommand::class,
        ]);
    }
}
```

### Optimizasyon Komutları (Optimize Commands)
Laravel'in `optimize` komutu, uygulamanın yapılandırmasını, olaylarını, rotalarını ve görünümlerini önbelleğe alır. `optimizes` metodunu kullanarak, `optimize` ve `optimize:clear` komutları yürütüldüğünde çağrılması gereken kendi Artisan komutlarınızı kaydedebilirsiniz:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->optimizes(
            optimize: 'package:optimize',
            clear: 'package:clear-optimizations',
        );
    }
}
```

### Yeniden Yükleme (Reload) Komutları
Laravel'in `reload` komutu, çalışan tüm servisleri sonlandırır, böylece bir sistem işlem monitörü (system process monitor) tarafından otomatik olarak yeniden başlatılabilirler. `reloads` metodunu kullanarak, `reload` komutu yürütüldüğünde çağrılması gereken kendi Artisan komutlarınızı kaydedebilirsiniz:
```php
/**
 * Herhangi bir paket servisini önyükle (bootstrap).
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->reloads('package:reload');
    }
}
```