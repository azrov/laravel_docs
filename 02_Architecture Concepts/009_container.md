# Laravel 12 Dokümantasyonu: Servis Kabı (Service Container)

## Giriş

Laravel servis kabı (service container), sınıf bağımlılıklarını (class dependencies) yönetmek ve bağımlılık enjeksiyonu (dependency injection) gerçekleştirmek için güçlü bir araçtır. Bağımlılık enjeksiyonu, süslü bir ifade olup temelde şu anlama gelir: sınıf bağımlılıkları, sınıfa kurucu (constructor) aracılığıyla veya bazı durumlarda "belirleyici" (setter) metotlar aracılığıyla "enjekte edilir".

Basit bir örneğe bakalım:
```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use Illuminate\View\View;

class PodcastController extends Controller
{
    /**
     * Yeni bir controller örneği oluştur.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Belirli bir podcast hakkında bilgi göster.
     */
    public function show(string $id): View
    {
        return view('podcasts.show', [
            'podcast' => $this->apple->findPodcast($id)
        ]);
    }
}
```
Bu örnekte, `PodcastController`'ın Apple Music gibi bir veri kaynağından podcast'leri alması gerekmektedir. Bu nedenle, podcast'leri alabilen bir servisi enjekte edeceğiz. Servis enjekte edildiği için, uygulamamızı test ederken `AppleMusic` servisini kolayca "mock"layabilir veya sahte bir uygulamasını oluşturabiliriz.

Laravel servis kabını derinlemesine anlamak, güçlü ve büyük bir uygulama oluşturmanın yanı sıra Laravel çekirdeğinin kendisine katkıda bulunmak için de gereklidir.

### Sıfır Yapılandırmalı Çözümleme (Zero Configuration Resolution)

Bir sınıfın hiç bağımlılığı yoksa veya yalnızca diğer somut sınıflara (concrete classes) (arayüzler/interface değil) bağımlıysa, kabın bu sınıfı nasıl çözümleyeceği (resolve) konusunda yönlendirilmesine gerek yoktur. Örneğin, aşağıdaki kodu `routes/web.php` dosyanıza yerleştirebilirsiniz:
```php
<?php

class Service
{
    // ...
}

Route::get('/', function (Service $service) {
    dd($service::class);
});
```
Bu örnekte, uygulamanızın `/` rotasına istek yapmak, `Service` sınıfını otomatik olarak çözümleyecek ve rotanızın işleyicisine enjekte edecektir. Bu oyunun kurallarını değiştiren bir özelliktir. Yani, şişirilmiş yapılandırma dosyaları hakkında endişelenmeden uygulamanızı geliştirebilir ve bağımlılık enjeksiyonunun avantajlarından yararlanabilirsiniz.

Neyse ki, bir Laravel uygulaması oluştururken yazacağınız sınıfların çoğu (controller'lar, olay dinleyicileri (event listeners), ara yazılımlar (middleware) ve daha fazlası) bağımlılıklarını otomatik olarak container aracılığıyla alır. Ek olarak, sıraya konmuş işlerin (queued jobs) `handle` metodunda bağımlılıkları tip belirtebilirsiniz (type-hint). Otomatik ve sıfır yapılandırmalı bağımlılık enjeksiyonunun gücünü bir kez tattıktan sonra, onsuz geliştirme yapmak imkansız görünür.

### Container Ne Zaman Kullanılır

Sıfır yapılandırmalı çözümleme sayesinde, rotalarda, controller'larda, olay dinleyicilerinde ve başka yerlerde sık sık bağımlılıkları tip belirtecek (type-hint) ve container ile hiç manuel olarak etkileşime girmeyeceksiniz. Örneğin, geçerli isteğe kolayca erişmek için rota tanımınızda `Illuminate\Http\Request` nesnesini tip belirtebilirsiniz. Bu kodu yazmak için container ile asla etkileşime girmek zorunda olmasak da, container bu bağımlılıkların enjeksiyonunu perde arkasında yönetmektedir:
```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

Çoğu durumda, otomatik bağımlılık enjeksiyonu ve facade'ler sayesinde, container'dan hiçbir şeyi manuel olarak bağlamadan (bind) veya çözümlemeden (resolve) Laravel uygulamaları oluşturabilirsiniz. Peki, container ile ne zaman manuel olarak etkileşime girersiniz? İki durumu inceleyelim.

Birincisi, bir arayüz (interface) uygulayan bir sınıf yazarsanız ve bu arayüzü bir rotada veya sınıf kurucusunda tip belirtmek isterseniz, container'a bu arayüzü nasıl çözümleyeceğini söylemelisiniz. İkincisi, diğer Laravel geliştiricileriyle paylaşmayı planladığınız bir Laravel paketi yazıyorsanız, paketinizin servislerini container'a bağlamanız gerekebilir.

## Bağlama (Binding)

### Bağlama Temelleri (Binding Basics)

#### Basit Bağlamalar (Simple Bindings)
Servis kabı bağlamalarınızın neredeyse tamamı servis sağlayıcılar (service providers) içinde kaydedilecektir, bu nedenle bu örneklerin çoğu container'ın bu bağlamda kullanımını gösterecektir.

Bir servis sağlayıcı içinde, `$this->app` özelliği aracılığıyla container'a her zaman erişebilirsiniz. `bind` metodunu kullanarak, kaydetmek istediğimiz sınıf veya arayüz adını ve sınıfın bir örneğini döndüren bir closure (kapatma) ileterek bir bağlama kaydedebiliriz:
```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```
Çözümleyiciye (resolver) argüman olarak container'ın kendisini aldığımıza dikkat edin. Daha sonra container'ı, oluşturduğumuz nesnenin alt bağımlılıklarını (sub-dependencies) çözümlemek için kullanabiliriz.

Bahsedildiği gibi, tipik olarak servis sağlayıcılar içinde container ile etkileşime geçeceksiniz; ancak bir servis sağlayıcı dışında container ile etkileşime geçmek isterseniz, bunu `App` facade'ı aracılığıyla yapabilirsiniz:
```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;

App::bind(Transistor::class, function (Application $app) {
    // ...
});
```
Yalnızca belirtilen tür için henüz bir bağlama yapılmamışsa bir container bağlaması kaydetmek için `bindIf` metodunu kullanabilirsiniz:
```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```
Kolaylık olması açısından, kaydetmek istediğiniz sınıf veya arayüz adını ayrı bir argüman olarak sağlamaktan vazgeçebilir ve bunun yerine Laravel'in türü `bind` metoduna sağladığınız closure'ın dönüş türünden (return type) çıkarmasına izin verebilirsiniz:
```php
App::bind(function (Application $app): Transistor {
    return new Transistor($app->make(PodcastParser::class));
});
```
Herhangi bir arayüze bağlı değillerse, sınıfları container'a bağlamaya gerek yoktur. Container, bu nesneleri yansıma (reflection) kullanarak otomatik olarak çözümleyebileceği için bunların nasıl oluşturulacağı konusunda yönlendirilmesi gerekmez.

#### Tekil (Singleton) Bağlama
`singleton` metodu, bir sınıfı veya arayüzü container'a yalnızca bir kez çözümlenmesi gereken bir bağlama olarak bağlar. Bir tekil bağlama çözümlendikten sonra, container'a yapılan sonraki çağrılarda aynı nesne örneği döndürülecektir:
```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```
Yalnızca belirtilen tür için henüz bir bağlama yapılmamışsa bir tekil container bağlaması kaydetmek için `singletonIf` metodunu kullanabilirsiniz:
```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

#### Singleton Niteliği (Singleton Attribute)
Alternatif olarak, bir arayüzü veya sınıfı `#[Singleton]` niteliği ile işaretleyerek container'a bunun yalnızca bir kez çözümlenmesi gerektiğini belirtebilirsiniz:
```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Singleton;

#[Singleton]
class Transistor
{
    // ...
}
```

#### Kapsamlı (Scoped) Tekilleri Bağlama (Binding Scoped Singletons)
`scoped` metodu, bir sınıfı veya arayüzü container'a belirli bir Laravel isteği/iş (request/job) yaşam döngüsü içinde yalnızca bir kez çözümlenmesi gereken bir bağlama olarak bağlar. Bu metod `singleton` metoduna benzer olsa da, `scoped` metodu kullanılarak kaydedilen örnekler, Laravel uygulaması yeni bir "yaşam döngüsüne" başladığında (örneğin, bir Laravel Octane çalışanı yeni bir isteği işlediğinde veya bir Laravel kuyruk çalışanı yeni bir işi işlediğinde) temizlenecektir:
```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```
Yalnızca belirtilen tür için henüz bir bağlama yapılmamışsa bir kapsamlı container bağlaması kaydetmek için `scopedIf` metodunu kullanabilirsiniz:
```php
$this->app->scopedIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

#### Scoped Niteliği (Scoped Attribute)
Alternatif olarak, bir arayüzü veya sınıfı `#[Scoped]` niteliği ile işaretleyerek container'a bunun belirli bir Laravel isteği/iş yaşam döngüsü içinde yalnızca bir kez çözümlenmesi gerektiğini belirtebilirsiniz:
```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Scoped;

#[Scoped]
class Transistor
{
    // ...
}
```

#### Örnekleri Bağlama (Binding Instances)
Ayrıca, `instance` metodunu kullanarak mevcut bir nesne örneğini container'a bağlayabilirsiniz. Verilen örnek, container'a yapılan sonraki çağrılarda her zaman döndürülecektir:
```php
use App\Services\Transistor;
use App\Services\PodcastParser;

$service = new Transistor(new PodcastParser);

$this->app->instance(Transistor::class, $service);
```

### Arayüzleri (Interface) Gerçekleştirimlere (Implementation) Bağlama

Servis kabının çok güçlü bir özelliği, bir arayüzü belirli bir gerçekleştirime bağlayabilme yeteneğidir. Örneğin, bir `EventPusher` arayüzümüz ve bir `RedisEventPusher` gerçekleştirimimiz olduğunu varsayalım. Bu arayüzün `RedisEventPusher` gerçekleştirimini kodladıktan sonra, onu servis kabına şöyle kaydedebiliriz:
```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;

$this->app->bind(EventPusher::class, RedisEventPusher::class);
```
Bu ifade, container'a, bir sınıfın `EventPusher` arayüzünün bir gerçekleştirimine ihtiyacı olduğunda `RedisEventPusher`'ı enjekte etmesi gerektiğini söyler. Artık container tarafından çözümlenen bir sınıfın kurucusunda `EventPusher` arayüzünü tip belirtebiliriz (type-hint). Unutmayın, Laravel uygulamalarındaki controller'lar, olay dinleyicileri, ara yazılımlar ve çeşitli diğer sınıf türleri her zaman container kullanılarak çözümlenir:
```php
use App\Contracts\EventPusher;

/**
 * Yeni bir sınıf örneği oluştur.
 */
public function __construct(
    protected EventPusher $pusher,
) {}
```

#### Bind Niteliği (Bind Attribute)
Laravel ayrıca ek kolaylık için bir `Bind` niteliği sağlar. Bu niteliği herhangi bir arayüze uygulayarak, bu arayüz talep edildiğinde Laravel'e hangi gerçekleştirimin otomatik olarak enjekte edilmesi gerektiğini söyleyebilirsiniz. `Bind` niteliğini kullanırken, uygulamanızın servis sağlayıcılarında herhangi bir ek servis kaydı yapmanıza gerek yoktur.

Ayrıca, belirli bir ortam seti için enjekte edilmesi gereken farklı bir gerçekleştirimi yapılandırmak üzere bir arayüze birden çok `Bind` niteliği yerleştirilebilir:
```php
<?php

namespace App\Contracts;

use App\Services\FakeEventPusher;
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;

#[Bind(RedisEventPusher::class)]
#[Bind(FakeEventPusher::class, environments: ['local', 'testing'])]
interface EventPusher
{
    // ...
}
```
Ayrıca, container bağlamalarının bir kez mi yoksa istek/iş yaşam döngüsü başına bir kez mi çözümlenmesi gerektiğini belirtmek için `Singleton` ve `Scoped` nitelikleri uygulanabilir:
```php
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;
use Illuminate\Container\Attributes\Singleton;

#[Bind(RedisEventPusher::class)]
#[Singleton]
interface EventPusher
{
    // ...
}
```

### Bağlamsal Bağlama (Contextual Binding)

Bazen aynı arayüzü kullanan iki sınıfınız olabilir, ancak her bir sınıfa farklı gerçekleştirimler enjekte etmek isteyebilirsiniz. Örneğin, iki controller `Illuminate\Contracts\Filesystem\Filesystem` sözleşmesinin (contract) farklı gerçekleştirimlerine bağımlı olabilir. Laravel bu davranışı tanımlamak için basit, akıcı bir arayüz (fluent interface) sağlar:
```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('local');
    });

$this->app->when([VideoController::class, UploadController::class])
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('s3');
    });
```

### Bağlamsal Nitelikler (Contextual Attributes)

Bağlamsal bağlama genellikle sürücülerin (drivers) veya yapılandırma değerlerinin gerçekleştirimlerini enjekte etmek için kullanıldığından, Laravel bu tür değerleri servis sağlayıcılarınızda bağlamsal bağlamaları manuel olarak tanımlamadan enjekte etmeye izin veren çeşitli bağlamsal bağlama nitelikleri sunar.

Örneğin, `Storage` niteliği belirli bir depolama diskini enjekte etmek için kullanılabilir:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Storage;
use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        #[Storage('local')] protected Filesystem $filesystem
    ) {
        // ...
    }
}
```
`Storage` niteliğine ek olarak, Laravel `Auth`, `Cache`, `Config`, `Context`, `DB`, `Give`, `Log`, `RouteParameter` ve `Tag` niteliklerini de sunar:
```php
<?php

namespace App\Http\Controllers;

use App\Contracts\UserRepository;
use App\Models\Photo;
use App\Repositories\DatabaseRepository;
use Illuminate\Container\Attributes\Auth;
use Illuminate\Container\Attributes\Cache;
use Illuminate\Container\Attributes\Config;
use Illuminate\Container\Attributes\Context;
use Illuminate\Container\Attributes\DB;
use Illuminate\Container\Attributes\Give;
use Illuminate\Container\Attributes\Log;
use Illuminate\Container\Attributes\RouteParameter;
use Illuminate\Container\Attributes\Tag;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Database\Connection;
use Psr\Log\LoggerInterface;

class PhotoController extends Controller
{
    public function __construct(
        #[Auth('web')] protected Guard $auth,
        #[Cache('redis')] protected Repository $cache,
        #[Config('app.timezone')] protected string $timezone,
        #[Context('uuid')] protected string $uuid,
        #[Context('ulid', hidden: true)] protected string $ulid,
        #[DB('mysql')] protected Connection $connection,
        #[Give(DatabaseRepository::class)] protected UserRepository $users,
        #[Log('daily')] protected LoggerInterface $log,
        #[RouteParameter('photo')] protected Photo $photo,
        #[Tag('reports')] protected array $reportServices,
    ) {
        // ...
    }
}
```