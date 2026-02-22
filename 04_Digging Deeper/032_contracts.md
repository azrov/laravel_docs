# Laravel 12 Dokümantasyonu: Sözleşmeler (Contracts)

## Giriş

Laravel'in "sözleşmeleri" (contracts), framework tarafından sağlanan temel hizmetleri tanımlayan bir dizi arayüzdür (interfaces). Örneğin, bir `Illuminate\Contracts\Queue\Queue` sözleşmesi, işleri (jobs) sıraya koymak için gereken metotları tanımlarken, `Illuminate\Contracts\Mail\Mailer` sözleşmesi e-posta göndermek için gereken metotları tanımlar.

Her sözleşmenin, framework tarafından sağlanan karşılık gelen bir gerçekleştirimi (implementation) vardır. Örneğin, Laravel çeşitli sürücülere (drivers) sahip bir kuyruk (queue) gerçekleştirimi ve Symfony Mailer tarafından desteklenen bir e-posta gönderici (mailer) gerçekleştirimi sağlar.

Tüm Laravel sözleşmeleri kendi GitHub depolarında bulunur. Bu, mevcut tüm sözleşmeler için hızlı bir referans noktası ve ayrıca Laravel hizmetleriyle etkileşime giren paketler oluştururken kullanılabilecek tek, ayrıştırılmış (decoupled) bir paket sağlar.

### Sözleşmeler ve Facade'ler (Contracts vs. Facades)
Laravel'in facade'leri ve yardımcı fonksiyonları (helper functions), servis kabında (service container) tip belirtmeye (type-hint) ve sözleşmeleri çözümlemeye (resolve) gerek kalmadan Laravel'in servislerini kullanmanın basit bir yolunu sağlar. Çoğu durumda, her facade'ın eşdeğer bir sözleşmesi vardır.

Sınıfınızın kurucusunda (constructor) onları gerektirmenizi gerektirmeyen facade'lerin aksine, sözleşmeler sınıflarınız için açık bağımlılıklar (explicit dependencies) tanımlamanıza olanak tanır. Bazı geliştiriciler bağımlılıklarını bu şekilde açıkça tanımlamayı tercih eder ve bu nedenle sözleşmeleri kullanmayı tercih ederken, diğer geliştiriciler facade'lerin rahatlığından hoşlanır. Genel olarak, çoğu uygulama geliştirme sırasında facade'leri sorunsuz bir şekilde kullanabilir.

## Sözleşmeler Ne Zaman Kullanılır (When to Use Contracts)
Sözleşmeleri veya facade'leri kullanma kararı, kişisel zevke ve geliştirme ekibinizin zevklerine bağlı olacaktır. Hem sözleşmeler hem de facade'ler, sağlam, iyi test edilmiş Laravel uygulamaları oluşturmak için kullanılabilir. Sözleşmeler ve facade'ler birbirini dışlayan (mutually exclusive) şeyler değildir. Uygulamalarınızın bazı bölümleri facade'leri kullanırken diğerleri sözleşmelere bağlı olabilir. Sınıfınızın sorumluluklarını odaklanmış (focused) tuttuğunuz sürece, sözleşmeler ve facade'leri kullanmak arasında çok az pratik fark fark edeceksiniz.

Genel olarak, çoğu uygulama geliştirme sırasında facade'leri sorunsuz bir şekilde kullanabilir. Birden çok PHP framework'üyle entegre olan bir paket oluşturuyorsanız, paketinizin `composer.json` dosyasında Laravel'in somut gerçekleştirimlerini (concrete implementations) gerektirmeye gerek kalmadan Laravel'in hizmetleriyle entegrasyonunuzu tanımlamak için `illuminate/contracts` paketini kullanmak isteyebilirsiniz.

## Sözleşmeler Nasıl Kullanılır (How to Use Contracts)
Peki, bir sözleşmenin gerçekleştirimini nasıl elde edersiniz? Aslında oldukça basit.

Laravel'deki birçok sınıf türü, controller'lar, olay dinleyicileri (event listeners), ara yazılımlar (middleware), sıraya konmuş işler (queued jobs) ve hatta rota kapatmaları (route closures) dahil olmak üzere servis kabı (service container) aracılığıyla çözümlenir (resolved). Bu nedenle, bir sözleşmenin gerçekleştirimini elde etmek için, çözümlenmekte olan sınıfın kurucusunda (constructor) arayüzü (interface) "tip belirtebilirsiniz" (type-hint).

Örneğin, şu olay dinleyicisine bir bakın:
```php
<?php

namespace App\Listeners;

use App\Events\OrderWasPlaced;
use App\Models\User;
use Illuminate\Contracts\Redis\Factory;

class CacheOrderInformation
{
    /**
     * Olay dinleyicisini oluştur.
     */
    public function __construct(
        protected Factory $redis,
    ) {}

    /**
     * Olayı işle.
     */
    public function handle(OrderWasPlaced $event): void
    {
        // ...
    }
}
```
Olay dinleyicisi çözümlendiğinde, servis kabı sınıfın kurucusundaki tip ipuçlarını (type-hints) okuyacak ve uygun değeri enjekte edecektir. Servis kabına şeyleri kaydetme hakkında daha fazla bilgi edinmek için dokümantasyonuna göz atın.

## Sözleşme Referansı (Contract Reference)

Bu tablo, tüm Laravel sözleşmelerine ve bunların eşdeğer facade'lerine hızlı bir referans sağlar:

| Sözleşme (Contract) | Referans Facade |
| :--- | :--- |
| Illuminate\Contracts\Auth\Access\Authorizable |  |
| Illuminate\Contracts\Auth\Access\Gate | Gate |
| Illuminate\Contracts\Auth\Authenticatable |  |
| Illuminate\Contracts\Auth\CanResetPassword |  |
| Illuminate\Contracts\Auth\Factory | Auth |
| Illuminate\Contracts\Auth\Guard | Auth::guard() |
| Illuminate\Contracts\Auth\PasswordBroker | Password |
| Illuminate\Contracts\Auth\PasswordBrokerFactory |  |
| Illuminate\Contracts\Auth\StatefulGuard |  |
| Illuminate\Contracts\Auth\SupportsBasicAuth |  |
| Illuminate\Contracts\Auth\UserProvider |  |
| Illuminate\Contracts\Bus\Dispatcher | Bus |
| Illuminate\Contracts\Bus\QueueingDispatcher |  |
| Illuminate\Contracts\Broadcasting\Factory | Broadcast |
| Illuminate\Contracts\Broadcasting\Broadcaster | Broadcast::connection() |
| Illuminate\Contracts\Broadcasting\ShouldBroadcast |  |
| Illuminate\Contracts\Broadcasting\ShouldBroadcastNow |  |
| Illuminate\Contracts\Cache\Factory | Cache |
| Illuminate\Contracts\Cache\Lock |  |
| Illuminate\Contracts\Cache\LockProvider |  |
| Illuminate\Contracts\Cache\Repository | Cache::driver() |
| Illuminate\Contracts\Cache\Store |  |
| Illuminate\Contracts\Config\Repository | Config |
| Illuminate\Contracts\Console\Application |  |
| Illuminate\Contracts\Console\Kernel | Artisan |
| Illuminate\Contracts\Container\Container | App |
| Illuminate\Contracts\Cookie\Factory | Cookie |
| Illuminate\Contracts\Cookie\QueueingFactory | Cookie::queue() |
| Illuminate\Contracts\Database\ModelIdentifier |  |
| Illuminate\Contracts\Debug\ExceptionHandler |  |
| Illuminate\Contracts\Encryption\Encrypter | Crypt |
| Illuminate\Contracts\Events\Dispatcher | Event |
| Illuminate\Contracts\Filesystem\Cloud |  |
| Illuminate\Contracts\Filesystem\Factory | Storage |
| Illuminate\Contracts\Filesystem\Filesystem | Storage::disk() |
| Illuminate\Contracts\Foundation\Application | App |
| Illuminate\Contracts\Hashing\Hasher | Hash |
| Illuminate\Contracts\Http\Kernel |  |
| Illuminate\Contracts\Mail\MailQueue | Mail::queue() |
| Illuminate\Contracts\Mail\Mailable |  |
| Illuminate\Contracts\Mail\Mailer | Mail |
| Illuminate\Contracts\Notifications\Dispatcher | Notification |
| Illuminate\Contracts\Notifications\Factory | Notification |
| Illuminate\Contracts\Pagination\LengthAwarePaginator |  |
| Illuminate\Contracts\Pagination\Paginator |  |
| Illuminate\Contracts\Pipeline\Hub |  |
| Illuminate\Contracts\Pipeline\Pipeline |  |
| Illuminate\Contracts\Queue\EntityResolver |  |
| Illuminate\Contracts\Queue\Factory | Queue |
| Illuminate\Contracts\Queue\Job |  |
| Illuminate\Contracts\Queue\Monitor | Queue::size() |
| Illuminate\Contracts\Queue\Queue | Queue::connection() |
| Illuminate\Contracts\Queue\QueueableCollection |  |
| Illuminate\Contracts\Queue\QueueableEntity |  |
| Illuminate\Contracts\Queue\ShouldQueue |  |
| Illuminate\Contracts\Redis\Factory | Redis |
| Illuminate\Contracts\Redis\Connection | Redis::connection() |
| Illuminate\Contracts\Routing\BindingRegistrar | Route |
| Illuminate\Contracts\Routing\Registrar | Route |
| Illuminate\Contracts\Routing\ResponseFactory | Response |
| Illuminate\Contracts\Routing\UrlGenerator | URL |
| Illuminate\Contracts\Routing\UrlRoutable |  |
| Illuminate\Contracts\Session\Session | Session |
| Illuminate\Contracts\Support\Arrayable |  |
| Illuminate\Contracts\Support\Htmlable |  |
| Illuminate\Contracts\Support\Jsonable |  |
| Illuminate\Contracts\Support\MessageBag |  |
| Illuminate\Contracts\Support\MessageProvider |  |
| Illuminate\Contracts\Support\Renderable |  |
| Illuminate\Contracts\Support\Responsable |  |
| Illuminate\Contracts\Translation\Loader |  |
| Illuminate\Contracts\Translation\Translator | Lang |
| Illuminate\Contracts\Validation\Factory | Validator |
| Illuminate\Contracts\Validation\ImplicitRule |  |
| Illuminate\Contracts\Validation\Rule |  |
| Illuminate\Contracts\Validation\Validator | Validator::make() |
| Illuminate\Contracts\View\Engine |  |
| Illuminate\Contracts\View\Factory | View |
| Illuminate\Contracts\View\View | View::make() |