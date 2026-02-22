# Laravel 12 Dokümantasyonu: Facade'ler (Yüzeyler/Arayüzler)

## Giriş

Laravel dokümantasyonu boyunca, "facade"ler aracılığıyla Laravel'in özellikleriyle etkileşime giren kod örnekleri göreceksiniz. Facade'ler, uygulamanın servis kabında (service container) bulunan sınıflara "statik" bir arayüz sağlar. Laravel, neredeyse tüm Laravel özelliklerine erişim sağlayan birçok facade ile birlikte gelir.

Laravel facade'leri, servis kabındaki temel sınıflara "statik proxy'ler" olarak hizmet eder; geleneksel statik metotlardan daha fazla test edilebilirlik ve esneklik sağlarken kısa, anlamlı bir sözdiziminin avantajını sunar. Facade'lerin nasıl çalıştığını tam olarak anlamasanız bile sorun değil - sadece akışa devam edin ve Laravel'i öğrenmeye devam edin.

Tüm Laravel facade'leri `Illuminate\Support\Facades` ad alanı (namespace) altında tanımlanmıştır. Bu nedenle, bir facade'a şu şekilde kolayca erişebiliriz:
```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```
Laravel dokümantasyonu boyunca, örneklerin çoğu framework'ün çeşitli özelliklerini göstermek için facade'leri kullanacaktır.

#### Yardımcı Fonksiyonlar (Helper Functions)

Facade'leri tamamlamak için Laravel, yaygın Laravel özellikleriyle etkileşimi daha da kolaylaştıran çeşitli global "yardımcı fonksiyonlar" sunar. Etkileşimde bulunabileceğiniz bazı yaygın yardımcı fonksiyonlar `view`, `response`, `url`, `config` ve daha fazlasıdır. Laravel tarafından sunulan her yardımcı fonksiyon, ilgili özellikleriyle birlikte belgelenmiştir; ancak, özel yardımcı dokümantasyon içinde eksiksiz bir liste mevcuttur.

Örneğin, bir JSON yanıtı oluşturmak için `Illuminate\Support\Facades\Response` facade'ını kullanmak yerine, basitçe `response` fonksiyonunu kullanabiliriz. Yardımcı fonksiyonlar global olarak kullanılabildiğinden, bunları kullanmak için herhangi bir sınıfı içe aktarmanıza gerek yoktur:
```php
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

## Facade'ler Ne Zaman Kullanılır

Facade'lerin birçok faydası vardır. Kısa, akılda kalıcı bir sözdizimi sağlayarak, manuel olarak enjekte edilmesi veya yapılandırılması gereken uzun sınıf adlarını hatırlamadan Laravel'in özelliklerini kullanmanıza olanak tanırlar. Ayrıca, PHP'nin dinamik metotlarını benzersiz kullanımları sayesinde test edilmeleri kolaydır.

Ancak, facade'leri kullanırken dikkatli olunmalıdır. Facade'lerin ana tehlikesi, sınıf "kapsam kayması"dır (scope creep). Facade'lerin kullanımı çok kolay olduğu ve enjeksiyon gerektirmediği için, sınıflarınızın büyümeye devam etmesine ve tek bir sınıfta birçok facade kullanmasına izin vermek kolay olabilir. Bağımlılık enjeksiyonu kullanıldığında, bu potansiyel, büyük bir kurucunun (constructor) size sınıfınızın çok büyüdüğüne dair verdiği görsel geri bildirimle azaltılır. Bu nedenle, facade'leri kullanırken, sorumluluk kapsamının dar kalması için sınıfınızın boyutuna özellikle dikkat edin. Sınıfınız çok büyüyorsa, onu daha küçük birden çok sınıfa bölmeyi düşünün.

### Facade'ler ve Bağımlılık Enjeksiyonu (Dependency Injection)

Bağımlılık enjeksiyonunun temel faydalarından biri, enjekte edilen sınıfın gerçekleştirimlerini (implementations) değiştirebilme yeteneğidir. Bu, test sırasında bir mock veya stub enjekte edebileceğiniz ve stub üzerinde çeşitli metotların çağrıldığını doğrulayabileceğiniz (assert) için kullanışlıdır.

Genellikle, gerçekten statik bir sınıf metodunu mock'lamak veya stub'lamak mümkün olmazdı. Ancak, facade'ler metot çağrılarını servis kabından çözümlenen (resolve edilen) nesnelere yönlendirmek için dinamik metotlar kullandığından, aslında facade'leri, enjekte edilmiş bir sınıf örneğini test edeceğimiz gibi test edebiliriz. Örneğin, aşağıdaki rotayı ele alalım:
```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```
Laravel'in facade test metotlarını kullanarak, `Cache::get` metodunun beklediğimiz argümanla çağrıldığını doğrulamak için aşağıdaki testi yazabiliriz:
```php
use Illuminate\Support\Facades\Cache;

test('basit örnek', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```
```php
use Illuminate\Support\Facades\Cache;

/**
 * Temel bir fonksiyonel test örneği.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

### Facade'ler ve Yardımcı Fonksiyonlar (Helper Functions)

Facade'lere ek olarak, Laravel görünümler (views) oluşturma, olaylar (events) tetikleme, işler (jobs) gönderme veya HTTP yanıtları gönderme gibi yaygın görevleri gerçekleştirebilen çeşitli "yardımcı" fonksiyonlar içerir. Bu yardımcı fonksiyonların çoğu, karşılık gelen bir facade ile aynı işlevi yerine getirir. Örneğin, bu facade çağrısı ve yardımcı çağrısı eşdeğerdir:
```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```
Facade'ler ve yardımcı fonksiyonlar arasında kesinlikle pratik bir fark yoktur. Yardımcı fonksiyonları kullanırken, onları yine de karşılık gelen facade'i test edeceğiniz gibi test edebilirsiniz. Örneğin, aşağıdaki rotayı ele alalım:
```php
Route::get('/cache', function () {
    return cache('key');
});
```
`cache` yardımcısı, `Cache` facade'ının altında yatan sınıfın `get` metodunu çağıracaktır. Bu nedenle, yardımcı fonksiyonu kullanıyor olsak bile, metodun beklediğimiz argümanla çağrıldığını doğrulamak için aşağıdaki testi yazabiliriz:
```php
use Illuminate\Support\Facades\Cache;

/**
 * Temel bir fonksiyonel test örneği.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

## Facade'ler Nasıl Çalışır

Bir Laravel uygulamasında, facade, container'dan bir nesneye erişim sağlayan bir sınıftır. Bunu mümkün kılan mekanizma `Facade` sınıfındadır. Laravel'in facade'leri ve oluşturacağınız herhangi bir özel facade, temel `Illuminate\Support\Facades\Facade` sınıfını genişletecektir (extend edecektir).

Temel `Facade` sınıfı, facade'inizden container'dan çözümlenen bir nesneye çağrıları yönlendirmek için `__callStatic()` sihirli-metodunu (magic-method) kullanır. Aşağıdaki örnekte, Laravel önbellek sistemine bir çağrı yapılmaktadır. Bu koda bakıldığında, `Cache` sınıfı üzerinde statik `get` metodunun çağrıldığı düşünülebilir:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Belirtilen kullanıcının profilini göster.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```
Dosyanın üst kısmına yakın bir yerde `Cache` facade'ını "içe aktardığımıza" (import) dikkat edin. Bu facade, `Illuminate\Contracts\Cache\Factory` arayüzünün temel gerçekleştirimine erişmek için bir proxy görevi görür. Facade'i kullanarak yaptığımız tüm çağrılar, Laravel'in önbellek servisinin temel örneğine (underlying instance) iletilecektir.

Eğer `Illuminate\Support\Facades\Cache` sınıfına bakarsak, orada statik bir `get` metodu olmadığını görürüz:
```php
class Cache extends Facade
{
    /**
     * Bileşenin kayıtlı adını al.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```
Bunun yerine, `Cache` facade'ı temel `Facade` sınıfını genişletir ve `getFacadeAccessor()` metodunu tanımlar. Bu metodun görevi, bir servis container bağlamasının (binding) adını döndürmektir. Bir kullanıcı `Cache` facade'ında herhangi bir statik metoda başvurduğunda, Laravel `cache` bağlamasını servis container'dan çözümler (resolve eder) ve istenen metodu (bu durumda `get`) bu nesneye karşı çalıştırır.

## Gerçek Zamanlı Facade'ler (Real-Time Facades)

Gerçek zamanlı facade'leri kullanarak, uygulamanızdaki herhangi bir sınıfı bir facade'miş gibi ele alabilirsiniz. Bunun nasıl kullanılabileceğini göstermek için, önce gerçek zamanlı facade'leri kullanmayan bazı kodları inceleyelim. Örneğin, `Podcast` modelimizin bir `publish` metodu olduğunu varsayalım. Ancak, podcast'i yayınlamak için bir `Publisher` örneği enjekte etmemiz gerekiyor:
```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Podcast'i yayınla.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```
Metoda bir yayıncı (publisher) gerçekleştirimi enjekte etmek, enjekte edilen yayıncıyı mock'layabileceğimiz için metodu izole bir şekilde kolayca test etmemizi sağlar. Ancak, `publish` metodunu her çağırdığımızda her zaman bir yayıncı örneği geçirmemizi gerektirir. Gerçek zamanlı facade'leri kullanarak, açıkça bir `Publisher` örneği geçirme zorunluluğu olmadan aynı test edilebilirliği koruyabiliriz. Gerçek zamanlı bir facade oluşturmak için, içe aktarılan sınıfın ad alanının (namespace) önüne `Facades` ekleyin:
```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // Eski yol
use Facades\App\Contracts\Publisher; // Gerçek zamanlı facade yolu
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Podcast'i yayınla.
     */
    public function publish(Publisher $publisher): void // Eski yol
    public function publish(): void // Yeni yol
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this); // Eski yol
        Publisher::publish($this); // Yeni yol
    }
}
```
Gerçek zamanlı facade kullanıldığında, yayıncı gerçekleştirimi, `Facades` ön ekinden sonra gelen arayüz veya sınıf adının kısmı kullanılarak servis container'ından çözümlenecektir. Test sırasında, bu metod çağrısını mock'lamak için Laravel'in yerleşik facade test yardımcılarını kullanabiliriz:
```php
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podcast yayınlanabilir', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```
```php
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Bir test örneği.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

## Facade Sınıf Referansı (Facade Class Reference)

Aşağıda her facade'ı ve onun temelindeki sınıfı bulacaksınız. Bu, belirli bir facade kökü için API dokümantasyonuna hızlıca dalmak için yararlı bir araçtır. Servis container bağlama anahtarı (service container binding key) da uygun olan yerlerde dahil edilmiştir.

| Facade | Sınıf | Servis Container Bağlama Anahtarı |
| :--- | :--- | :--- |
| App | `Illuminate\Foundation\Application` | `app` |
| Artisan | `Illuminate\Contracts\Console\Kernel` | `artisan` |
| Auth (Örnek) | `Illuminate\Contracts\Auth\Guard` | `auth.driver` |
| Auth | `Illuminate\Auth\AuthManager` | `auth` |
| Blade | `Illuminate\View\Compilers\BladeCompiler` | `blade.compiler` |
| Broadcast (Örnek) | `Illuminate\Contracts\Broadcasting\Broadcaster` | |
| Broadcast | `Illuminate\Contracts\Broadcasting\Factory` | |
| Bus | `Illuminate\Contracts\Bus\Dispatcher` | |
| Cache (Örnek) | `Illuminate\Cache\Repository` | `cache.store` |
| Cache | `Illuminate\Cache\CacheManager` | `cache` |
| Config | `Illuminate\Config\Repository` | `config` |
| Context | `Illuminate\Log\Context\Repository` | |
| Cookie | `Illuminate\Cookie\CookieJar` | `cookie` |
| Crypt | `Illuminate\Encryption\Encrypter` | `encrypter` |
| Date | `Illuminate\Support\DateFactory` | `date` |
| DB (Örnek) | `Illuminate\Database\Connection` | `db.connection` |
| DB | `Illuminate\Database\DatabaseManager` | `db` |
| Event | `Illuminate\Events\Dispatcher` | `events` |
| Exceptions (Örnek) | `Illuminate\Contracts\Debug\ExceptionHandler` | |
| Exceptions | `Illuminate\Foundation\Exceptions\Handler` | |
| File | `Illuminate\Filesystem\Filesystem` | `files` |
| Gate | `Illuminate\Contracts\Auth\Access\Gate` | |
| Hash | `Illuminate\Contracts\Hashing\Hasher` | `hash` |
| Http | `Illuminate\Http\Client\Factory` | |
| Lang | `Illuminate\Translation\Translator` | `translator` |
| Log | `Illuminate\Log\LogManager` | `log` |
| Mail | `Illuminate\Mail\Mailer` | `mailer` |
| Notification | `Illuminate\Notifications\ChannelManager` | |
| Password (Örnek) | `Illuminate\Auth\Passwords\PasswordBroker` | `auth.password.broker` |
| Password | `Illuminate\Auth\Passwords\PasswordBrokerManager` | `auth.password` |
| Pipeline (Örnek) | `Illuminate\Pipeline\Pipeline` | |
| Process | `Illuminate\Process\Factory` | |
| Queue (Temel Sınıf) | `Illuminate\Queue\Queue` | |
| Queue (Örnek) | `Illuminate\Contracts\Queue\Queue` | `queue.connection` |
| Queue | `Illuminate\Queue\QueueManager` | `queue` |
| RateLimiter | `Illuminate\Cache\RateLimiter` | |
| Redirect | `Illuminate\Routing\Redirector` | `redirect` |
| Redis (Örnek) | `Illuminate\Redis\Connections\Connection` | `redis.connection` |
| Redis | `Illuminate\Redis\RedisManager` | `redis` |
| Request | `Illuminate\Http\Request` | `request` |
| Response (Örnek) | `Illuminate\Http\Response` | |
| Response | `Illuminate\Contracts\Routing\ResponseFactory` | |
| Route | `Illuminate\Routing\Router` | `router` |
| Schedule | `Illuminate\Console\Scheduling\Schedule` | |
| Schema | `Illuminate\Database\Schema\Builder` | |
| Session (Örnek) | `Illuminate\Session\Store` | `session.store` |
| Session | `Illuminate\Session\SessionManager` | `session` |
| Storage (Örnek) | `Illuminate\Contracts\Filesystem\Filesystem` | `filesystem.disk` |
| Storage | `Illuminate\Filesystem\FilesystemManager` | `filesystem` |
| URL | `Illuminate\Routing\UrlGenerator` | `url` |
| Validator (Örnek) | `Illuminate\Validation\Validator` | |
| Validator | `Illuminate\Validation\Factory` | `validator` |
| View (Örnek) | `Illuminate\View\View` | |
| View | `Illuminate\View\Factory` | `view` |
| Vite | `Illuminate\Foundation\Vite` | |