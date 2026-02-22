# Laravel 12 Dokümantasyonu: Görünümler (Views)

## Giriş

Elbette, tüm HTML belgesi dizelerini doğrudan rotalarınızdan ve controller'larınızdan döndürmek pratik değildir. Neyse ki görünümler (views), tüm HTML'imizi ayrı dosyalara yerleştirmek için kullanışlı bir yol sağlar.

Görünümler, controller / uygulama mantığınızı sunum mantığınızdan (presentation logic) ayırır ve `resources/views` dizininde saklanır. Laravel kullanırken, görünüm şablonları genellikle Blade şablonlama dili (templating language) kullanılarak yazılır. Basit bir görünüm şöyle görünebilir:
```blade
<!-- resources/views/greeting.blade.php konumunda saklanan görünüm -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```
Bu görünüm `resources/views/greeting.blade.php` konumunda saklandığından, global `view` yardımcısını kullanarak onu şu şekilde döndürebiliriz:
```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```
Blade şablonlarının nasıl yazılacağı hakkında daha fazla bilgi mi arıyorsunuz? Başlamak için eksiksiz Blade dokümantasyonuna göz atın.

### Görünümleri React / Vue / Svelte ile Yazma
Birçok geliştirici, ön yüz şablonlarını PHP aracılığıyla Blade ile yazmak yerine, bunları React, Vue veya Svelte kullanarak yazmayı tercih etmeye başladı. Laravel, Inertia sayesinde bunu zahmetsiz hale getirir. Inertia, bir SPA oluşturmanın tipik karmaşıklıkları olmadan React / Vue / Svelte ön yüzünüzü Laravel arka ucunuza bağlamayı çok kolaylaştıran bir kütüphanedir.

React, Vue ve Svelte uygulama başlangıç kitlerimiz (starter kits), Inertia tarafından desteklenen bir sonraki Laravel uygulamanız için size harika bir başlangıç noktası sağlar.

## Görünüm Oluşturma ve Render Etme (Creating and Rendering Views)

Uygulamanızın `resources/views` dizinine `.blade.php` uzantılı bir dosya yerleştirerek veya `make:view` Artisan komutunu kullanarak bir görünüm oluşturabilirsiniz:
```bash
php artisan make:view greeting
```
`.blade.php` uzantısı, framework'e dosyanın bir Blade şablonu içerdiğini bildirir. Blade şablonları, değerleri kolayca görüntülemenize, "if" ifadeleri oluşturmanıza, veriler üzerinde yineleme yapmanıza ve daha fazlasına olanak tanıyan HTML ve Blade yönergelerini (directives) içerir.

Bir görünüm oluşturduktan sonra, global `view` yardımcısını kullanarak onu uygulamanızın rotalarından veya controller'larından birinden döndürebilirsiniz:
```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```
Görünümler ayrıca `View` facade'ı kullanılarak da döndürülebilir:
```php
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```
Gördüğünüz gibi, `view` yardımcısına iletilen ilk argüman, `resources/views` dizinindeki görünüm dosyasının adına karşılık gelir. İkinci argüman, görünüme sunulması gereken bir veri dizisidir (array). Bu durumda, görünümde Blade sözdizimi kullanılarak görüntülenen `name` değişkenini iletiyoruz.

### İç İçe Görünüm Dizinleri (Nested View Directories)
Görünümler ayrıca `resources/views` dizininin alt dizinlerinde iç içe yerleştirilebilir. İç içe görünümlere başvurmak için "nokta" notasyonu kullanılabilir. Örneğin, görünümünüz `resources/views/admin/profile.blade.php` konumunda saklanıyorsa, onu uygulamanızın rotalarından / controller'larından birinden şu şekilde döndürebilirsiniz:
```php
return view('admin.profile', $data);
```
Görünüm dizin adları `.` karakterini içermemelidir.

### İlk Mevcut Görünümü Oluşturma (Creating the First Available View)
`View` facade'ının `first` metodunu kullanarak, belirli bir görünüm dizisinde mevcut olan ilk görünümü oluşturabilirsiniz. Bu, uygulamanız veya paketiniz görünümlerin özelleştirilmesine veya geçersiz kılınmasına izin veriyorsa yararlı olabilir:
```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

### Bir Görünümün Var Olup Olmadığını Belirleme (Determining if a View Exists)
Bir görünümün var olup olmadığını belirlemeniz gerekiyorsa, `View` facade'ını kullanabilirsiniz. `exists` metodu, görünüm varsa `true` döndürecektir:
```php
use Illuminate\Support\Facades\View;

if (View::exists('admin.profile')) {
    // ...
}
```

## Görünümlere Veri Aktarma (Passing Data to Views)

Önceki örneklerde gördüğünüz gibi, bu verileri görünüme sunmak için görünümlere bir veri dizisi iletebilirsiniz:
```php
return view('greetings', ['name' => 'Victoria']);
```
Bu şekilde bilgi aktarırken, veriler anahtar/değer çiftlerine sahip bir dizi olmalıdır. Bir görünüme veri sağladıktan sonra, görünümünüz içindeki her bir değere, `<?php echo $name; ?>` gibi verinin anahtarlarını kullanarak erişebilirsiniz.

`view` yardımcı fonksiyonuna tam bir veri dizisi iletmeye bir alternatif olarak, görünüme tek tek veri parçaları eklemek için `with` metodunu kullanabilirsiniz. `with` metodu, görünüm nesnesinin bir örneğini döndürür, böylece görünümü döndürmeden önce metotları zincirlemeye devam edebilirsiniz:
```php
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

### Verileri Tüm Görünümlerle Paylaşma (Sharing Data With All Views)
Bazen, uygulamanız tarafından oluşturulan tüm görünümlerle veri paylaşmanız gerekebilir. Bunu, `View` facade'ının `share` metodunu kullanarak yapabilirsiniz. Tipik olarak, `share` metoduna yapılan çağrıları bir servis sağlayıcının (service provider) `boot` metodu içine yerleştirmelisiniz. Bunları `App\Providers\AppServiceProvider` sınıfına eklemekte veya bunları barındırmak için ayrı bir servis sağlayıcı oluşturmakta özgürsünüz:
```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini kaydet.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        View::share('key', 'value');
    }
}
```

## Görünüm Oluşturucular (View Composers)

Görünüm oluşturucular (view composers), bir görünüm oluşturulduğunda (rendered) çağrılan geri çağrılar (callbacks) veya sınıf metotlarıdır. Bir görünüm her oluşturulduğunda o görünüme bağlanmasını istediğiniz veriler varsa, bir görünüm oluşturucu bu mantığı tek bir konumda düzenlemenize yardımcı olabilir. Aynı görünüm uygulamanızdaki birden çok rota veya controller tarafından döndürülüyorsa ve her zaman belirli bir veri parçasına ihtiyaç duyuyorsa, görünüm oluşturucular özellikle yararlı olabilir.

Tipik olarak, görünüm oluşturucular uygulamanızın servis sağlayıcılarından birinde kaydedilir. Bu örnekte, `App\Providers\AppServiceProvider`'ın bu mantığı barındıracağını varsayacağız.

Görünüm oluşturucuyu kaydetmek için `View` facade'ının `composer` metodunu kullanacağız. Laravel, sınıf tabanlı görünüm oluşturucular için varsayılan bir dizin içermez, bu nedenle onları dilediğiniz gibi düzenlemekte özgürsünüz. Örneğin, uygulamanızın tüm görünüm oluşturucularını barındırmak için bir `app/View/Composers` dizini oluşturabilirsiniz:
```php
<?php

namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini kaydet.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        // Sınıf tabanlı oluşturucuları kullan...
        Facades\View::composer('profile', ProfileComposer::class);

        // Kapatma (closure) tabanlı oluşturucuları kullan...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });

        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```
Artık oluşturucuyu kaydettiğimize göre, `profile` görünümü her oluşturulduğunda `App\View\Composers\ProfileComposer` sınıfının `compose` metodu yürütülecektir. Oluşturucu sınıfının bir örneğine bakalım:
```php
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * Yeni bir profil oluşturucu oluştur.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * Verileri görünüme bağla.
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```
Gördüğünüz gibi, tüm görünüm oluşturucular servis kabı (service container) aracılığıyla çözümlenir (resolve edilir), bu nedenle bir oluşturucunun kurucusunda ihtiyacınız olan herhangi bir bağımlılığı tip belirtebilirsiniz (type-hint).

#### Bir Oluşturucuyu Birden Çok Görünüme Ekleme (Attaching a Composer to Multiple Views)
`composer` metoduna ilk argüman olarak bir görünüm dizisi ileterek bir görünüm oluşturucuyu aynı anda birden çok görünüme ekleyebilirsiniz:
```php
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```
`composer` metodu ayrıca joker karakter olarak `*` karakterini de kabul eder; bu, bir oluşturucuyu tüm görünümlere eklemenize olanak tanır:
```php
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

### Görünüm Yaratıcıları (View Creators)
Görünüm "yaratıcıları" (creators), görünüm oluşturuculara (composers) çok benzer; ancak, görünüm oluşturulmak üzereyken beklemek yerine, görünüm örneklendikten (instantiated) hemen sonra yürütülürler. Bir görünüm yaratıcısı kaydetmek için `creator` metodunu kullanın:
```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

## Görünümleri Optimize Etme (Optimizing Views)

Varsayılan olarak, Blade şablon görünümleri ihtiyaç anında (on demand) derlenir (compiled). Bir görünüm oluşturan bir istek yürütüldüğünde, Laravel görünümün derlenmiş bir sürümünün var olup olmadığını belirleyecektir. Dosya varsa, Laravel daha sonra derlenmemiş görünümün, derlenmiş görünümden daha yeni bir tarihte değiştirilip değiştirilmediğini belirleyecektir. Derlenmiş görünüm mevcut değilse veya derlenmemiş görünüm değiştirilmişse, Laravel görünümü yeniden derleyecektir.

Görünümleri istek sırasında derlemek performans üzerinde küçük bir olumsuz etkiye sahip olabilir, bu nedenle Laravel, uygulamanız tarafından kullanılan tüm görünümleri önceden derlemek için `view:cache` Artisan komutunu sağlar. Artırılmış performans için, bu komutu dağıtım sürecinizin bir parçası olarak çalıştırmak isteyebilirsiniz:
```bash
php artisan view:cache
```
Görünüm önbelleğini temizlemek için `view:clear` komutunu kullanabilirsiniz:
```bash
php artisan view:clear
```