# Laravel 12 Dokümantasyonu: Olaylar (Events)

## Giriş

Laravel'in olayları (events), uygulamanızda meydana gelen çeşitli olaylara abone olmanıza ve bunları dinlemenize olanak tanıyan basit bir gözlemci deseni (observer pattern) uygulaması sağlar. Olay sınıfları tipik olarak `app/Events` dizininde saklanırken, dinleyicileri (listeners) `app/Listeners` dizininde saklanır. Uygulamanızda bu dizinleri görmüyorsanız endişelenmeyin, çünkü Artisan konsol komutlarını kullanarak olaylar ve dinleyiciler oluşturdukça sizin için oluşturulacaklardır.

Olaylar, uygulamanızın çeşitli yönlerini ayırmak (decouple) için harika bir yol sunar, çünkü tek bir olay, birbirine bağımlı olmayan birden çok dinleyiciye sahip olabilir. Örneğin, her sipariş gönderildiğinde kullanıcınıza bir Slack bildirimi göndermek isteyebilirsiniz. Sipariş işleme kodunuzu Slack bildirim kodunuza bağlamak (coupling) yerine, bir dinleyicinin alıp bir Slack bildirimi göndermek için kullanabileceği bir `App\Events\OrderShipped` olayı oluşturabilirsiniz.

## Olay ve Dinleyici Oluşturma (Generating Events and Listeners)

Hızlıca olay ve dinleyici oluşturmak için `make:event` ve `make:listener` Artisan komutlarını kullanabilirsiniz:
```bash
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```
Kolaylık olması açısından, `make:event` ve `make:listener` Artisan komutlarını ek argümanlar olmadan da çağırabilirsiniz. Bunu yaptığınızda, Laravel otomatik olarak sizden sınıf adını ve bir dinleyici oluştururken dinlemesi gereken olayı isteyecektir:
```bash
php artisan make:event

php artisan make:listener
```

## Olayları ve Dinleyicileri Kaydetme (Registering Events and Listeners)

### Olay Keşfi (Event Discovery)
Varsayılan olarak Laravel, uygulamanızın `Listeners` dizinini tarayarak olay dinleyicilerinizi otomatik olarak bulacak ve kaydedecektir. Laravel, `handle` veya `__invoke` ile başlayan herhangi bir dinleyici sınıfı metodu bulduğunda, bu metotları, metodun imzasında tip belirtilen (type-hinted) olay için olay dinleyicileri olarak kaydedecektir:
```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * Olayı işle.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```
PHP'nin birleşim türlerini (union types) kullanarak birden çok olayı dinleyebilirsiniz:
```php
/**
 * Olayı işle.
 */
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```
Dinleyicilerinizi farklı bir dizinde veya birden çok dizinde saklamayı planlıyorsanız, uygulamanızın `bootstrap/app.php` dosyasındaki `withEvents` metodunu kullanarak Laravel'e bu dizinleri taraması talimatını verebilirsiniz:
```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```
`*` karakterini joker karakter olarak kullanarak benzer birden çok dizinde tarama yapabilirsiniz:
```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
```
`event:list` komutu, uygulamanızda kayıtlı tüm dinleyicileri listelemek için kullanılabilir:
```bash
php artisan event:list
```

#### Üretimde (Production) Olay Keşfi
Uygulamanıza hız kazandırmak için, `optimize` veya `event:cache` Artisan komutlarını kullanarak uygulamanızın tüm dinleyicilerinin bir manifestosunu önbelleğe almalısınız. Tipik olarak bu komut, uygulamanızın dağıtım (deployment) sürecinin bir parçası olarak çalıştırılmalıdır. Bu manifesto, framework tarafından olay kayıt sürecini hızlandırmak için kullanılacaktır. `event:clear` komutu, olay önbelleğini yok etmek için kullanılabilir.

### Olayları Manuel Olarak Kaydetme (Manually Registering Events)
`Event` facade'ını kullanarak, uygulamanızın `AppServiceProvider` sınıfının `boot` metodu içinde olayları ve bunlara karşılık gelen dinleyicileri manuel olarak kaydedebilirsiniz:
```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

### Kapatma (Closure) Dinleyicileri
Tipik olarak, dinleyiciler sınıflar olarak tanımlanır; ancak, uygulamanızın `AppServiceProvider` sınıfının `boot` metodunda kapatma tabanlı (closure-based) olay dinleyicilerini de manuel olarak kaydedebilirsiniz:
```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

#### Sıraya Konabilir (Queueable) Anonim Olay Dinleyicileri
Kapatma tabanlı olay dinleyicileri kaydederken, dinleyici kapatmasını `Illuminate\Events\queueable` fonksiyonu içine sararak Laravel'e dinleyiciyi kuyruk (queue) kullanarak yürütmesi talimatını verebilirsiniz:
```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```
Sıraya konmuş işler (queued jobs) gibi, sıraya konmuş dinleyicinin yürütülmesini özelleştirmek için `onConnection`, `onQueue` ve `delay` metotlarını kullanabilirsiniz:
```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));
```
Anonim sıraya konmuş dinleyici hatalarını (failures) ele almak isterseniz, `queueable` dinleyiciyi tanımlarken `catch` metoduna bir closure sağlayabilirsiniz. Bu closure, olay örneğini ve dinleyicinin hatasına neden olan `Throwable` örneğini alacaktır:
```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // Sıraya konmuş dinleyici başarısız oldu...
}));
```

#### Joker Karakterli (Wildcard) Olay Dinleyicileri
Ayrıca, `*` karakterini joker parametre olarak kullanarak dinleyiciler kaydedebilir, aynı dinleyicide birden çok olayı yakalamanıza olanak tanıyabilirsiniz. Joker karakterli dinleyiciler, ilk argüman olarak olay adını ve ikinci argüman olarak tüm olay veri dizisini alır:
```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

## Olay Tanımlama (Defining Events)

Bir olay sınıfı, esasen olayla ilgili bilgileri tutan bir veri kabıdır (data container). Örneğin, bir `App\Events\OrderShipped` olayının bir Eloquent ORM nesnesi aldığını varsayalım:
```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Yeni bir olay örneği oluştur.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```
Gördüğünüz gibi, bu olay sınıfı hiçbir mantık içermez. Satın alınan `App\Models\Order` örneği için bir kaptır. Olay tarafından kullanılan `SerializesModels` özelliği (trait), sıraya konmuş dinleyiciler kullanılırken olduğu gibi, olay nesnesi PHP'nin `serialize` fonksiyonu kullanılarak serileştirilirse (serialized) herhangi bir Eloquent modelini zarif bir şekilde serileştirecektir.

## Dinleyici Tanımlama (Defining Listeners)

Şimdi, örnek olayımızın dinleyicisine bir göz atalım. Olay dinleyicileri, `handle` metotlarında olay örneklerini alırlar. `make:listener` Artisan komutu, `--event` seçeneğiyle çağrıldığında, uygun olay sınıfını otomatik olarak içe aktaracak ve `handle` metodunda olayı tip belirtecektir (type-hint). `handle` metodu içinde, olaya yanıt vermek için gerekli tüm eylemleri gerçekleştirebilirsiniz:
```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Olay dinleyicisini oluştur.
     */
    public function __construct() {}

    /**
     * Olayı işle.
     */
    public function handle(OrderShipped $event): void
    {
        // $event->order kullanarak siparişe eriş...
    }
}
```
Olay dinleyicileriniz, kurucularında (constructors) ihtiyaç duydukları herhangi bir bağımlılığı da tip belirtebilir. Tüm olay dinleyicileri Laravel servis kabı (service container) aracılığıyla çözümlenir (resolved), bu nedenle bağımlılıklar otomatik olarak enjekte edilecektir.

#### Bir Olayın Diğer Dinleyicilere Yayılmasını Durdurma (Stopping The Propagation Of An Event)
Bazen, bir olayın diğer dinleyicilere yayılmasını durdurmak isteyebilirsiniz. Bunu, dinleyicinizin `handle` metodundan `false` döndürerek yapabilirsiniz.

## Sıraya Konmuş Olay Dinleyicileri (Queued Event Listeners)

Dinleyiciniz e-posta gönderme veya HTTP isteği yapma gibi yavaş bir görev gerçekleştirecekse, dinleyicileri sıraya koymak faydalı olabilir. Sıraya konmuş dinleyicileri kullanmadan önce, kuyruğunuzu (queue) yapılandırdığınızdan ve sunucunuzda veya yerel geliştirme ortamınızda bir kuyruk çalışanı (queue worker) başlattığınızdan emin olun.

Bir dinleyicinin sıraya konması gerektiğini belirtmek için, dinleyici sınıfına `ShouldQueue` arayüzünü (interface) ekleyin. `make:listener` Artisan komutları tarafından oluşturulan dinleyiciler, bu arayüzü zaten geçerli ad alanına (namespace) aktarmıştır, böylece hemen kullanabilirsiniz:
```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```
Bu kadar! Artık bu dinleyici tarafından işlenen bir olay gönderildiğinde, dinleyici olay dağıtıcısı (event dispatcher) tarafından Laravel'in kuyruk sistemi kullanılarak otomatik olarak sıraya konacaktır. Dinleyici kuyruk tarafından yürütülürken herhangi bir istisna (exception) fırlatılmazsa, sıraya konmuş iş işlemeyi bitirdikten sonra otomatik olarak silinecektir.

#### Kuyruk Bağlantısını, Adını ve Gecikmesini Özelleştirme (Customizing The Queue Connection, Name, & Delay)
Bir olay dinleyicisinin kuyruk bağlantısını (queue connection), kuyruk adını veya kuyruk gecikme süresini özelleştirmek isterseniz, dinleyici sınıfınızda `$connection`, `$queue` veya `$delay` özelliklerini tanımlayabilirsiniz:
```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * İşin gönderilmesi gereken bağlantının adı.
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * İşin gönderilmesi gereken kuyruğun adı.
     *
     * @var string|null
     */
    public $queue = 'listeners';

    /**
     * İşin işlenmesi gereken süre (saniye).
     *
     * @var int
     */
    public $delay = 60;
}
```
Dinleyicinin kuyruk bağlantısını, kuyruk adını veya gecikmesini çalışma zamanında tanımlamak isterseniz, dinleyicide `viaConnection`, `viaQueue` veya `withDelay` metotlarını tanımlayabilirsiniz:
```php
/**
 * Dinleyicinin kuyruk bağlantısının adını al.
 */
public function viaConnection(): string
{
    return 'sqs';
}

/**
 * Dinleyicinin kuyruğunun adını al.
 */
public function viaQueue(): string
{
    return 'listeners';
}

/**
 * İşin işlenmesi gereken saniye sayısını al.
 */
public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

#### Dinleyicileri Koşullu Olarak Sıraya Koyma (Conditionally Queueing Listeners)
Bazen, bir dinleyicinin yalnızca çalışma zamanında mevcut olan bazı verilere dayalı olarak sıraya konması gerekip gerekmediğini belirlemeniz gerekebilir. Bunu başarmak için, dinleyiciye bir `shouldQueue` metodu eklenebilir. `shouldQueue` metodu `false` döndürürse, dinleyici sıraya konmayacaktır:
```php
<?php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * Müşteriye hediye kartı ver.
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }

    /**
     * Dinleyicinin sıraya konması gerekip gerekmediğini belirle.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

### Kuyrukla Manuel Olarak Etkileşim Kurma (Manually Interacting With the Queue)
Dinleyicinin temeldeki kuyruk işinin `delete` ve `release` metotlarına manuel olarak erişmeniz gerekiyorsa, bunu `Illuminate\Queue\InteractsWithQueue` özelliğini (trait) kullanarak yapabilirsiniz. Bu özellik, oluşturulan dinleyicilere varsayılan olarak eklenir ve bu metotlara erişim sağlar:
```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Olayı işle.
     */
    public function handle(OrderShipped $event): void
    {
        if ($condition) {
            $this->release(30);
        }
    }
}
```

### Sıraya Konmuş Olay Dinleyicileri ve Veritabanı İşlemleri (Queued Event Listeners and Database Transactions)
Sıraya konmuş dinleyiciler veritabanı işlemleri (database transactions) içinde gönderildiğinde, veritabanı işlemi tamamlanmadan (commit) önce kuyruk tarafından işlenebilirler. Bu olduğunda, veritabanı işlemi sırasında modellere veya veritabanı kayıtlarına yaptığınız tüm güncellemeler henüz veritabanına yansıtılmamış olabilir. Ayrıca, işlem içinde oluşturulan tüm modeller veya veritabanı kayıtları veritabanında mevcut olmayabilir. Dinleyiciniz bu modellere bağlıysa, sıraya konmuş dinleyiciyi gönderen iş işlendiğinde beklenmeyen hatalar oluşabilir.

Kuyruk bağlantınızın `after_commit` yapılandırma seçeneği `false` olarak ayarlanmışsa, belirli bir sıraya konmuş dinleyicinin tüm açık veritabanı işlemleri tamamlandıktan sonra gönderilmesi gerektiğini, dinleyici sınıfında `ShouldQueueAfterCommit` arayüzünü uygulayarak (implement) belirtebilirsiniz:
```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```
Bu sorunların etrafından nasıl çalışılacağı hakkında daha fazla bilgi edinmek için lütfen sıraya konmuş işler ve veritabanı işlemleri ile ilgili dokümantasyonu inceleyin.

### Sıraya Konmuş Dinleyici Ara Yazılımları (Queued Listener Middleware)
Sıraya konmuş dinleyiciler, iş ara yazılımlarını (job middleware) da kullanabilir. İş ara yazılımları, sıraya konmuş dinleyicilerin yürütülmesi etrafında özel mantık sarmanıza (wrap) olanak tanıyarak dinleyicilerin kendilerindeki tekrar eden kod miktarını (boilerplate) azaltır. İş ara yazılımı oluşturduktan sonra, dinleyicinin `middleware` metodundan döndürerek bir dinleyiciye eklenebilirler:
```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Bu dinleyici için iş ara yazılımlarını al.
     *
     * @return array
     */
    public function middleware(OrderShipped $event): array
    {
        return [
            new CustomMiddleware,
        ];
    }
}
```