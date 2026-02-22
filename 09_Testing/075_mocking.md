# Laravel 12 Dokümantasyonu: Taklit Etme (Mocking)

## Giriş

Laravel uygulamalarını test ederken, uygulamanızın belirli yönlerini "taklit etmek" (mock) isteyebilirsiniz, böylece belirli bir test sırasında gerçekte çalıştırılmazlar. Örneğin, bir olay (event) gönderen (dispatch) bir controller'ı test ederken, olay dinleyicilerini (event listeners) taklit etmek isteyebilirsiniz, böylece test sırasında gerçekte çalıştırılmazlar. Bu, yalnızca controller'ın HTTP yanıtını test etmenize olanak tanır, çünkü olay dinleyicileri kendi test durumlarında (test case) test edilebilir.

Laravel, olayları, işleri (jobs) ve diğer facade'leri kutudan çıktığı gibi taklit etmek için yararlı metotlar sağlar. Bu yardımcılar öncelikle Mockery üzerinde bir kolaylık katmanı sağlar, böylece manuel olarak karmaşık Mockery metot çağrıları yapmak zorunda kalmazsınız.

## Nesneleri Taklit Etme (Mocking Objects)

Laravel'in servis kabı (service container) aracılığıyla uygulamanıza enjekte edilecek bir nesneyi taklit ederken, taklit edilmiş örneğinizi (mocked instance) kaba bir `instance` bağlaması olarak bağlamanız gerekecektir. Bu, nesneyi kendisi oluşturmak yerine kabın, nesnenin taklit edilmiş örneğinizi kullanması talimatını verecektir:
```php
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('bir şey taklit edilebilir', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
});
```
```php
use App\Service;
use Mockery;
use Mockery\MockInterface;

public function test_something_can_be_mocked(): void
{
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
}
```
Bunu daha kolay hale getirmek için, Laravel'in temel test durumu sınıfı (base test case class) tarafından sağlanan `mock` metodunu kullanabilirsiniz. Örneğin, aşağıdaki örnek yukarıdaki örnekle eşdeğerdir:
```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->mock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```
Bir nesnenin yalnızca birkaç metodunu taklit etmeniz gerektiğinde `partialMock` metodunu kullanabilirsiniz. Taklit edilmeyen metotlar çağrıldıklarında normal şekilde yürütülecektir:
```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```
Benzer şekilde, bir nesneyi gözetlemek (spy) isterseniz, Laravel'in temel test durumu sınıfı, `Mockery::spy` metodu etrafında kullanışlı bir sarmalayıcı olarak bir `spy` metodu sunar. Gözetleyiciler (spies), taklitlere (mocks) benzer; ancak gözetleyiciler, gözetleyici ile test edilen kod arasındaki her türlü etkileşimi kaydeder ve kod yürütüldükten sonra iddialarda bulunmanıza olanak tanır:
```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

## Facade'leri Taklit Etme (Mocking Facades)

Geleneksel statik metot çağrılarının aksine, facade'ler (gerçek zamanlı facade'ler dahil) taklit edilebilir. Bu, geleneksel statik metotlara göre büyük bir avantaj sağlar ve geleneksel bağımlılık enjeksiyonu (dependency injection) kullanıyor olsaydınız sahip olacağınız test edilebilirliği size sunar. Test ederken, genellikle controller'larınızdan birinde gerçekleşen bir Laravel facade çağrısını taklit etmek isteyebilirsiniz. Örneğin, aşağıdaki controller eylemini ele alalım:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Uygulamanın tüm kullanıcılarının bir listesini al.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```
`expects` metodunu kullanarak `Cache` facade'ına yapılan çağrıyı taklit edebiliriz. Bu metod, bir Mockery taklidi örneği döndürecektir. Facade'ler aslında Laravel servis kabı tarafından çözümlendiği (resolved) ve yönetildiği için, tipik bir statik sınıftan çok daha fazla test edilebilirliğe sahiptirler. Örneğin, `Cache` facade'ının `get` metoduna yaptığımız çağrıyı taklit edelim:
```php
<?php

use Illuminate\Support\Facades\Cache;

test('index sayfasını al', function () {
    Cache::expects('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/users');

    // ...
});
```
```php
<?php

namespace Tests\Feature;

use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function test_get_index(): void
    {
        Cache::expects('get')
            ->with('key')
            ->andReturn('value');

        $response = $this->get('/users');

        // ...
    }
}
```
`Request` facade'ını taklit etmemelisiniz. Bunun yerine, testinizi çalıştırırken istediğiniz girdiyi `get` ve `post` gibi HTTP test metotlarına iletin. Benzer şekilde, `Config` facade'ını taklit etmek yerine, testlerinizde `Config::set` metodunu çağırın.

### Facade Gözetleyicileri (Facade Spies)
Bir facade'ı gözetlemek isterseniz, ilgili facade üzerinde `spy` metodunu çağırabilirsiniz. Gözetleyiciler, taklitlere benzer; ancak gözetleyiciler, gözetleyici ile test edilen kod arasındaki her türlü etkileşimi kaydeder ve kod yürütüldükten sonra iddialarda bulunmanıza olanak tanır:
```php
<?php

use Illuminate\Support\Facades\Cache;

test('değerler önbellekte saklanır', function () {
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
});
```
```php
use Illuminate\Support\Facades\Cache;

public function test_values_are_stored_in_cache(): void
{
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
}
```

## Zamanla Etkileşim (Interacting With Time)

Test ederken, bazen `now` veya `Illuminate\Support\Carbon::now()` gibi yardımcılar tarafından döndürülen zamanı değiştirmeniz gerekebilir. Neyse ki Laravel'in temel özellik test sınıfı (base feature test class), geçerli zamanı değiştirmenize izin veren yardımcılar içerir:
```php
test('zaman manipüle edilebilir', function () {
    // Geleceğe yolculuk...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Geçmişe yolculuk...
    $this->travel(-5)->hours();

    // Belirli bir zamana yolculuk...
    $this->travelTo(now()->subHours(6));

    // Şimdiki zamana geri dön...
    $this->travelBack();
});
```
```php
public function test_time_can_be_manipulated(): void
{
    // Geleceğe yolculuk...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Geçmişe yolculuk...
    $this->travel(-5)->hours();

    // Belirli bir zamana yolculuk...
    $this->travelTo(now()->subHours(6));

    // Şimdiki zamana geri dön...
    $this->travelBack();
}
```
Ayrıca, çeşitli zaman yolculuğu metotlarına bir closure da sağlayabilirsiniz. Closure, belirtilen zamanda zaman donmuş (frozen) haldeyken çağrılacaktır. Closure yürütüldükten sonra, zaman normal olarak devam edecektir:
```php
$this->travel(5)->days(function () {
    // Beş gün sonraki bir şeyi test et...
});

$this->travelTo(now()->addDays(10), function () {
    // Belirli bir anda bir şeyi test et...
});
```
`freezeTime` metodu, geçerli zamanı dondurmak için kullanılabilir. Benzer şekilde, `freezeSecond` metodu, geçerli zamanı ancak geçerli saniyenin başlangıcında dondurur:
```php
use Illuminate\Support\Carbon;

// Zamanı dondur ve closure yürütüldükten sonra normal zamana devam et...
$this->freezeTime(function (Carbon $time) {
    // ...
});

// Zamanı geçerli saniyede dondur ve closure yürütüldükten sonra normal zamana devam et...
$this->freezeSecond(function (Carbon $time) {
    // ...
})
```
Bekleyeceğiniz gibi, yukarıda tartışılan tüm metotlar öncelikle, bir tartışma forumundaki aktif olmayan gönderileri kilitlemek gibi zamana duyarlı (time sensitive) uygulama davranışlarını test etmek için kullanışlıdır:
```php
use App\Models\Thread;

test('forum başlıkları bir haftalık hareketsizlikten sonra kilitlenir', function () {
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    expect($thread->isLockedByInactivity())->toBeTrue();
});
```
```php
use App\Models\Thread;

public function test_forum_threads_lock_after_one_week_of_inactivity()
{
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    $this->assertTrue($thread->isLockedByInactivity());
}
```