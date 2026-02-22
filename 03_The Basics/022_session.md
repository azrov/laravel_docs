# Laravel 12 Dokümantasyonu: HTTP Oturumu (HTTP Session)

## Giriş

HTTP tabanlı uygulamalar durumsuz (stateless) olduğundan, oturumlar (sessions), kullanıcı hakkındaki bilgileri birden çok istek boyunca saklamanın bir yolunu sağlar. Bu kullanıcı bilgileri tipik olarak, sonraki isteklerden erişilebilen kalıcı bir depoda / arka uçta (persistent store/backend) saklanır.

Laravel, anlamlı, birleşik bir API (expressive, unified API) aracılığıyla erişilen çeşitli oturum arka uçları (session backends) ile birlikte gelir. Memcached, Redis ve veritabanları gibi popüler arka uçlar için destek dahil edilmiştir.

### Yapılandırma (Configuration)
Uygulamanızın oturum yapılandırma dosyası `config/session.php` konumunda saklanır. Bu dosyada size sunulan seçenekleri gözden geçirdiğinizden emin olun. Varsayılan olarak, Laravel `database` oturum sürücüsünü (driver) kullanacak şekilde yapılandırılmıştır.

`session` yapılandırma dosyasındaki `driver` seçeneği, oturum verilerinin her istek için nerede saklanacağını tanımlar. Laravel bir dizi sürücü içerir:

*   `file` - oturumlar `storage/framework/sessions` içinde saklanır.
*   `cookie` - oturumlar güvenli, şifrelenmiş çerezlerde (cookies) saklanır.
*   `database` - oturumlar ilişkisel bir veritabanında (relational database) saklanır.
*   `memcached` / `redis` - oturumlar bu hızlı, önbellek tabanlı depolardan birinde saklanır.
*   `dynamodb` - oturumlar AWS DynamoDB'de saklanır.
*   `array` - oturumlar bir PHP dizisinde saklanır ve kalıcı olmaz (persisted).

`array` sürücüsü öncelikle test sırasında kullanılır ve oturumda saklanan verilerin kalıcı olmasını engeller.

### Sürücü Ön Gereksinimleri (Driver Prerequisites)

#### Veritabanı (Database)
`database` oturum sürücüsünü kullanırken, oturum verilerini içerecek bir veritabanı tablonuzun olduğundan emin olmanız gerekecektir. Tipik olarak bu, Laravel'in varsayılan `0001_01_01_000000_create_users_table.php` veritabanı migration'ında bulunur; ancak, herhangi bir nedenle `sessions` tablonuz yoksa, bu migration'ı oluşturmak için `make:session-table` Artisan komutunu kullanabilirsiniz:
```bash
php artisan make:session-table

php artisan migrate
```

#### Redis
Laravel ile Redis oturumlarını kullanmadan önce, PECL aracılığıyla PhpRedis PHP eklentisini (extension) veya Composer aracılığıyla `predis/predis` paketini (~1.0) kurmanız gerekecektir. Redis'i yapılandırma hakkında daha fazla bilgi için Laravel'in Redis dokümantasyonuna bakın.

`SESSION_CONNECTION` ortam değişkeni veya `session.php` yapılandırma dosyasındaki `connection` seçeneği, oturum depolama için hangi Redis bağlantısının kullanılacağını belirtmek için kullanılabilir.

## Oturumla Etkileşim (Interacting With the Session)

### Veri Alma (Retrieving Data)
Laravel'de oturum verileriyle çalışmanın iki ana yolu vardır: global `session` yardımcısı (helper) ve bir `Request` örneği (instance). İlk olarak, bir rota kapatmasında (route closure) veya controller metodunda tip belirtilebilen (type-hinted) `Request` örneği aracılığıyla oturuma erişmeye bakalım. Controller metodu bağımlılıklarının Laravel servis kabı (service container) aracılığıyla otomatik olarak enjekte edildiğini unutmayın:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Belirtilen kullanıcının profilini göster.
     */
    public function show(Request $request, string $id): View
    {
        $value = $request->session()->get('key');

        // ...

        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```
Oturumdan bir öğe (item) aldığınızda, `get` metoduna ikinci argüman olarak bir varsayılan değer de iletebilirsiniz. Belirtilen anahtar (key) oturumda yoksa bu varsayılan değer döndürülecektir. `get` metoduna varsayılan değer olarak bir closure (kapatma) iletirseniz ve istenen anahtar mevcut değilse, closure yürütülecek ve sonucu döndürülecektir:
```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

#### Global Session Yardımcısı (The Global Session Helper)
Ayrıca, oturumdaki verileri almak ve saklamak için global `session` PHP fonksiyonunu da kullanabilirsiniz. `session` yardımcısı tek bir dize argümanıyla çağrıldığında, bu oturum anahtarının değerini döndürecektir. Yardımcı, anahtar/değer çiftlerinden oluşan bir diziyle çağrıldığında, bu değerler oturumda saklanacaktır:
```php
Route::get('/home', function () {
    // Oturumdan bir veri parçası al...
    $value = session('key');

    // Bir varsayılan değer belirt...
    $value = session('key', 'default');

    // Oturuma bir veri parçası sakla...
    session(['key' => 'value']);
});
```
Oturumu bir HTTP istek örneği (HTTP request instance) aracılığıyla kullanmak ile global `session` yardımcısını kullanmak arasında çok az pratik fark vardır. Her iki yöntem de tüm test durumlarınızda (test cases) mevcut olan `assertSessionHas` metodu aracılığıyla test edilebilir.

#### Tüm Oturum Verilerini Alma (Retrieving All Session Data)
Oturumdaki tüm verileri almak isterseniz, `all` metodunu kullanabilirsiniz:
```php
$data = $request->session()->all();
```

#### Oturum Verilerinin Bir Kısmını Alma (Retrieving a Portion of the Session Data)
`only` ve `except` metotları, oturum verilerinin bir alt kümesini almak için kullanılabilir:
```php
$data = $request->session()->only(['username', 'email']);

$data = $request->session()->except(['username', 'email']);
```

#### Bir Öğenin Oturumda Var Olup Olmadığını Belirleme (Determining if an Item Exists in the Session)
Bir öğenin oturumda mevcut olup olmadığını belirlemek için `has` metodunu kullanabilirsiniz. `has` metodu, öğe mevcutsa ve `null` değilse `true` döndürür:
```php
if ($request->session()->has('users')) {
    // ...
}
```
Bir öğenin oturumda mevcut olup olmadığını, değeri `null` olsa bile belirlemek için `exists` metodunu kullanabilirsiniz:
```php
if ($request->session()->exists('users')) {
    // ...
}
```
Bir öğenin oturumda mevcut olmadığını belirlemek için `missing` metodunu kullanabilirsiniz. `missing` metodu, öğe mevcut değilse `true` döndürür:
```php
if ($request->session()->missing('users')) {
    // ...
}
```

### Veri Saklama (Storing Data)
Oturuma veri saklamak için tipik olarak istek örneğinin `put` metodunu veya global `session` yardımcısını kullanacaksınız:
```php
// Bir istek örneği aracılığıyla...
$request->session()->put('key', 'value');

// Global "session" yardımcısı aracılığıyla...
session(['key' => 'value']);
```

#### Dizi Oturum Değerlerine Ekleme Yapma (Pushing to Array Session Values)
`push` metodu, bir dizi olan oturum değerine yeni bir değer eklemek için kullanılabilir. Örneğin, `user.teams` anahtarı bir takım adları dizisi içeriyorsa, diziye şu şekilde yeni bir değer ekleyebilirsiniz:
```php
$request->session()->push('user.teams', 'developers');
```

#### Bir Öğeyi Alma ve Silme (Retrieving and Deleting an Item)
`pull` metodu, oturumdan bir öğeyi tek bir ifadede alacak ve silecektir:
```php
$value = $request->session()->pull('key', 'default');
```

#### Oturum Değerlerini Artırma ve Azaltma (Incrementing and Decrementing Session Values)
Oturum verileriniz artırmak veya azaltmak istediğiniz bir tamsayı içeriyorsa, `increment` ve `decrement` metotlarını kullanabilirsiniz:
```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

### Flash Veriler (Flash Data)
Bazen öğeleri yalnızca bir sonraki istek için oturumda saklamak isteyebilirsiniz. Bunu `flash` metodunu kullanarak yapabilirsiniz. Bu metodu kullanarak oturumda saklanan veriler hemen ve sonraki HTTP isteği sırasında kullanılabilir olacaktır. Sonraki HTTP isteğinden sonra, flash veriler silinecektir. Flash veriler öncelikle kısa ömürlü durum mesajları (short-lived status messages) için kullanışlıdır:
```php
$request->session()->flash('status', 'Görev başarılı oldu!');
```
Flash verilerinizi birkaç istek boyunca kalıcı kılmanız gerekiyorsa, tüm flash verilerini ek bir istek için saklayacak olan `reflash` metodunu kullanabilirsiniz. Yalnızca belirli flash verilerini saklamanız gerekiyorsa, `keep` metodunu kullanabilirsiniz:
```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```
Flash verilerinizi yalnızca geçerli istek için kalıcı kılmak (persist) için `now` metodunu kullanabilirsiniz:
```php
$request->session()->now('status', 'Görev başarılı oldu!');
```

### Veri Silme (Deleting Data)
`forget` metodu, oturumdan bir veri parçasını kaldıracaktır. Oturumdaki tüm verileri kaldırmak isterseniz, `flush` metodunu kullanabilirsiniz:
```php
// Tek bir anahtarı unut...
$request->session()->forget('name');

// Birden çok anahtarı unut...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

### Oturum ID'sini Yeniden Oluşturma (Regenerating the Session ID)
Oturum ID'sini yeniden oluşturmak (regenerate), genellikle kötü niyetli kullanıcıların uygulamanızda bir oturum sabitleme saldırısından (session fixation attack) yararlanmasını önlemek için yapılır.

Laravel, Laravel uygulama başlangıç kitlerinden (starter kits) birini veya Laravel Fortify'i kullanıyorsanız, kimlik doğrulama sırasında oturum ID'sini otomatik olarak yeniden oluşturur; ancak, oturum ID'sini manuel olarak yeniden oluşturmanız gerekirse, `regenerate` metodunu kullanabilirsiniz:
```php
$request->session()->regenerate();
```
Oturum ID'sini yeniden oluşturmanız ve oturumdaki tüm verileri tek bir ifadede kaldırmanız gerekiyorsa, `invalidate` metodunu kullanabilirsiniz:
```php
$request->session()->invalidate();
```

## Oturum Önbelleği (Session Cache)

Laravel'in oturum önbelleği (session cache), tek bir kullanıcı oturumuyla kapsamlandırılmış (scoped) verileri önbelleğe almanın kullanışlı bir yolunu sağlar. Genel uygulama önbelleğinin aksine, oturum önbellek verileri oturum başına otomatik olarak izole edilir ve oturum sona erdiğinde veya yok edildiğinde temizlenir. Oturum önbelleği, `get`, `put`, `remember`, `forget` ve daha fazlası gibi tüm tanıdık Laravel önbellek metotlarını destekler, ancak geçerli oturumla kapsamlandırılmıştır.

Oturum önbelleği, aynı oturum içinde birden çok istek boyunca kalıcı olmasını istediğiniz, ancak kalıcı olarak saklamanız gerekmeyen geçici, kullanıcıya özgü verileri depolamak için mükemmeldir. Bu, form verileri, geçici hesaplamalar, API yanıtları veya belirli bir kullanıcının oturumuna bağlı olması gereken diğer herhangi bir kısa ömürlü (ephemeral) veri gibi şeyleri içerir.

Oturum önbelleğine, oturum üzerindeki `cache` metodu aracılığıyla erişebilirsiniz:
```php
$discount = $request->session()->cache()->get('discount');

$request->session()->cache()->put(
    'discount', 10, now()->addMinutes(5)
);
```
Laravel'in önbellek metotları hakkında daha fazla bilgi için önbellek dokümantasyonuna bakın.

## Oturum Engelleme (Session Blocking)

Oturum engellemeyi (session blocking) kullanmak için uygulamanızın atomik kilitleri (atomic locks) destekleyen bir önbellek sürücüsü (cache driver) kullanıyor olması gerekir. Şu anda, bu önbellek sürücüleri `memcached`, `dynamodb`, `redis`, `mongodb` (resmi `mongodb/laravel-mongodb` paketinde bulunur), `database`, `file` ve `array` sürücülerini içerir. Ayrıca, `cookie` oturum sürücüsünü kullanamazsınız.

Varsayılan olarak Laravel, aynı oturumu kullanan isteklerin aynı anda (concurrently) yürütülmesine izin verir. Bu nedenle, örneğin, uygulamanıza iki HTTP isteği yapmak için bir JavaScript HTTP kütüphanesi kullanırsanız, her ikisi de aynı anda yürütülecektir. Birçok uygulama için bu bir sorun değildir; ancak, her ikisi de oturuma veri yazan iki farklı uygulama uç noktasına (endpoints) eşzamanlı isteklerde bulunan küçük bir uygulama alt kümesinde oturum verisi kaybı meydana gelebilir.

Bunu azaltmak için Laravel, belirli bir oturum için eşzamanlı istekleri sınırlamanıza izin veren bir işlevsellik sağlar. Başlamak için, rota tanımınıza `block` metodunu eklemeniz yeterlidir. Bu örnekte, `/profile` uç noktasına gelen bir istek bir oturum kilidi (session lock) edinecektir. Bu kilit tutulurken, aynı oturum ID'sini paylaşan `/profile` veya `/order` uç noktalarına yapılan sonraki tüm istekler, yürütmelerine devam etmeden önce ilk isteğin yürütülmesini bitirmesini bekleyecektir:
```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```
`block` metodu iki isteğe bağlı argüman kabul eder. `block` metodu tarafından kabul edilen ilk argüman, oturum kilidinin serbest bırakılmadan önce tutulması gereken maksimum saniye sayısıdır. Elbette, istek bu süreden önce yürütülmeyi bitirirse kilit daha erken serbest bırakılacaktır.

`block` metodu tarafından kabul edilen ikinci argüman, bir isteğin oturum kilidini elde etmeye çalışırken beklemesi gereken saniye sayısıdır. İstek, verilen saniye sayısı içinde bir oturum kilidi elde edemezse, bir `Illuminate\Contracts\Cache\LockTimeoutException` fırlatılacaktır.

Bu argümanlardan hiçbiri iletilmezse, kilit maksimum 10 saniye için elde edilecek ve istekler bir kilit elde etmeye çalışırken maksimum 10 saniye bekleyecektir:
```php
Route::post('/profile', function () {
    // ...
})->block();
```

## Özel Oturum Sürücüleri Ekleme (Adding Custom Session Drivers)

### Sürücüyü Uygulama (Implementing the Driver)
Mevcut oturum sürücülerinden hiçbiri uygulamanızın ihtiyaçlarına uymuyorsa, Laravel kendi oturum işleyicinizi (session handler) yazmanızı mümkün kılar. Özel oturum sürücünüz, PHP'nin yerleşik `SessionHandlerInterface`'ini uygulamalıdır (implement). Bu arayüz sadece birkaç basit metot içerir. Örnek bir MongoDB uygulaması şöyle görünebilir:
```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```
Laravel, uzantılarınızı (extensions) barındırmak için varsayılan bir dizin içermez. Onları dilediğiniz yere yerleştirmekte özgürsünüz. Bu örnekte, `MongoSessionHandler`'ı barındırmak için bir `Extensions` dizini oluşturduk.

Bu metotların amacı hemen anlaşılır olmadığından, işte her bir metodun amacına genel bir bakış:

*   `open` metodu tipik olarak dosya tabanlı oturum depolama sistemlerinde kullanılır. Laravel bir `file` oturum sürücüsüyle geldiğinden, bu metoda nadiren herhangi bir şey koymanız gerekecektir. Bu metodu basitçe boş bırakabilirsiniz.
*   `close` metodu da `open` metodu gibi genellikle göz ardı edilebilir. Çoğu sürücü için gerekli değildir.
*   `read` metodu, verilen `$sessionId` ile ilişkili oturum verilerinin dize (string) sürümünü döndürmelidir. Sürücünüzde oturum verilerini alırken veya saklarken herhangi bir serileştirme (serialization) veya başka bir kodlama yapmanıza gerek yoktur, çünkü Laravel serileştirmeyi sizin için gerçekleştirecektir.
*   `write` metodu, `$sessionId` ile ilişkili verilen `$data` dizesini MongoDB veya seçtiğiniz başka bir depolama sistemi gibi bazı kalıcı depolama sistemlerine yazmalıdır. Yine, herhangi bir serileştirme yapmamalısınız - Laravel bunu sizin için zaten halletmiş olacaktır.
*   `destroy` metodu, kalıcı depodan `$sessionId` ile ilişkili verileri kaldırmalıdır.
*   `gc` metodu, verilen `$lifetime` (UNIX zaman damgası) değerinden daha eski olan tüm oturum verilerini yok etmelidir (destroy). Kendi kendini temizleyen (self-expiring) Redis veya MongoDB gibi sistemler için bu metot boş bırakılabilir.

### Sürücüyü Kaydetme (Registering the Driver)
Sürücünüzü uyguladıktan sonra, onu framework'e kaydetmeye hazırsınız. Laravel'in oturum arka ucuna ek sürücüler eklemek için, `Session` facade'ının `extend` metodunu kullanabilirsiniz. Bu metodu bir servis sağlayıcının (service provider) `boot` metodundan çağırmalısınız. Bunu mevcut `AppServiceProvider`'dan veya yepyeni bir servis sağlayıcı oluşturarak yapabilirsiniz:
```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
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
        Session::extend('mongo', function (Application $app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler();
        });
    }
}
```
Oturum sürücüsü kaydedildikten sonra, `config/session.php` yapılandırma dosyanızda `mongo` sürücüsünü kullanabilirsiniz.