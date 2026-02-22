# Laravel 12 Dokümantasyonu: Bildirimler (Notifications)

## Giriş

E-posta gönderme desteğine ek olarak Laravel, e-posta, SMS (Vonage aracılığıyla) ve Slack dahil olmak üzere çeşitli teslimat kanalları (delivery channels) üzerinden bildirim gönderme desteği sağlar. Ayrıca, düzinelerce farklı kanal üzerinden bildirim göndermek için topluluk tarafından oluşturulmuş çok sayıda bildirim kanalı mevcuttur! Bildirimler ayrıca bir veritabanında saklanabilir, böylece web arayüzünüzde görüntülenebilirler.

Tipik olarak bildirimler, kullanıcıları uygulamanızda meydana gelen bir şey hakkında bilgilendiren kısa, bilgilendirici mesajlar olmalıdır. Örneğin, bir faturalandırma uygulaması yazıyorsanız, kullanıcılarınıza e-posta ve SMS kanalları aracılığıyla bir "Fatura Ödendi" bildirimi gönderebilirsiniz.

## Bildirim Oluşturma (Generating Notifications)

Laravel'de her bildirim, tipik olarak `app/Notifications` dizininde saklanan tek bir sınıf (class) tarafından temsil edilir. Uygulamanızda bu dizini görmüyorsanız endişelenmeyin - `make:notification` Artisan komutunu çalıştırdığınızda sizin için oluşturulacaktır:
```bash
php artisan make:notification InvoicePaid
```
Bu komut, `app/Notifications` dizininize yeni bir bildirim sınıfı yerleştirecektir. Her bildirim sınıfı, bir `via` metodu ve bildirimi belirli bir kanal için uyarlanmış bir mesaja dönüştüren `toMail` veya `toDatabase` gibi değişken sayıda mesaj oluşturma metodu içerir.

## Bildirim Gönderme (Sending Notifications)

### Notifiable Trait'ini Kullanma
Bildirimler iki şekilde gönderilebilir: `Notifiable` trait'inin `notify` metodu kullanılarak veya `Notification` facade'ı kullanılarak. `Notifiable` trait'i, varsayılan olarak uygulamanızın `App\Models\User` modeline dahil edilmiştir:
```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```
Bu trait tarafından sağlanan `notify` metodu, bir bildirim örneği (notification instance) almayı bekler:
```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```
Unutmayın, `Notifiable` trait'ini modellerinizden herhangi birinde kullanabilirsiniz. Sadece `User` modelinize dahil etmekle sınırlı değilsiniz.

### Notification Facade'ını Kullanma
Alternatif olarak, bildirimleri `Notification` facade'ı aracılığıyla gönderebilirsiniz. Bu yaklaşım, bir kullanıcı koleksiyonu gibi birden çok bildirilebilir varlığa (notifiable entities) bir bildirim göndermeniz gerektiğinde kullanışlıdır. Facade'ı kullanarak bildirim göndermek için, tüm bildirilebilir varlıkları ve bildirim örneğini `send` metoduna iletin:
```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```
Ayrıca, `sendNow` metodunu kullanarak bildirimleri hemen de gönderebilirsiniz. Bu metod, bildirim `ShouldQueue` arayüzünü uygulasa bile bildirimi hemen gönderecektir:
```php
Notification::sendNow($developers, new DeploymentCompleted($deployment));
```

### Teslimat Kanallarını Belirleme (Specifying Delivery Channels)
Her bildirim sınıfının, bildirimin hangi kanallarda teslim edileceğini belirleyen bir `via` metodu vardır. Bildirimler `database`, `broadcast`, `vonage` ve `slack` kanallarında gönderilebilir.

Telegram veya Pusher gibi diğer teslimat kanallarını kullanmak isterseniz, topluluk odaklı Laravel Notification Channels web sitesine göz atın.

`via` metodu, bildirimin gönderildiği sınıfın bir örneği olacak bir `$notifiable` örneği alır. Bildirimin hangi kanallarda teslim edilmesi gerektiğini belirlemek için `$notifiable`'ı kullanabilirsiniz:
```php
/**
 * Bildirimin teslimat kanallarını al.
 *
 * @return array<int, string>
 */
public function via(object $notifiable): array
{
    return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
}
```

### Bildirimleri Sıraya Koyma (Queueing Notifications)
Bildirimleri sıraya koymadan önce, kuyruğunuzu (queue) yapılandırmalı ve bir işçi (worker) başlatmalısınız.

Bildirim göndermek, özellikle kanalın bildirimi iletmek için harici bir API çağrısı yapması gerekiyorsa zaman alabilir. Uygulamanızın yanıt süresini hızlandırmak için, `ShouldQueue` arayüzünü ve `Queueable` trait'ini sınıfınıza ekleyerek bildiriminizin sıraya konmasını sağlayın. Arayüz ve trait, `make:notification` komutu kullanılarak oluşturulan tüm bildirimler için zaten içe aktarılmıştır, böylece hemen bildirim sınıfınıza ekleyebilirsiniz:
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```
Bildiriminize `ShouldQueue` arayüzü eklendikten sonra, bildirimi normal bir şekilde gönderebilirsiniz. Laravel, sınıftaki `ShouldQueue` arayüzünü algılayacak ve bildirimin teslimatını otomatik olarak sıraya koyacaktır:
```php
$user->notify(new InvoicePaid($invoice));
```
Bildirimleri sıraya koyarken, her alıcı ve kanal kombinasyonu için sıraya konmuş bir iş (queued job) oluşturulacaktır. Örneğin, bildiriminizin üç alıcısı ve iki kanalı varsa, kuyruğa altı iş gönderilecektir.

#### Bildirimleri Geciktirme (Delaying Notifications)
Bildirimin teslimatını geciktirmek isterseniz, bildirim örneğinize `delay` metodunu zincirleyebilirsiniz:
```php
$delay = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```
Belirli kanallar için gecikme miktarını belirtmek üzere `delay` metoduna bir dizi (array) iletebilirsiniz:
```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->addMinutes(5),
    'sms' => now()->addMinutes(10),
]));
```
Alternatif olarak, bildirim sınıfının kendisinde bir `withDelay` metodu tanımlayabilirsiniz. `withDelay` metodu, kanal adları ve gecikme değerlerinden oluşan bir dizi döndürmelidir:
```php
/**
 * Bildirimin teslimat gecikmesini belirle.
 *
 * @return array<string, \Illuminate\Support\Carbon>
 */
public function withDelay(object $notifiable): array
{
    return [
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ];
}
```

#### Bildirim Kuyruk Bağlantısını Özelleştirme (Customizing the Notification Queue Connection)
Varsayılan olarak, sıraya konmuş bildirimler uygulamanızın varsayılan kuyruk bağlantısı (default queue connection) kullanılarak sıraya konacaktır. Belirli bir bildirim için kullanılması gereken farklı bir bağlantı belirtmek isterseniz, bildiriminizin kurucusundan (constructor) `onConnection` metodunu çağırabilirsiniz:
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Yeni bir bildirim örneği oluştur.
     */
    public function __construct()
    {
        $this->onConnection('redis');
    }
}
```
Veya, bildirim tarafından desteklenen her bildirim kanalı için kullanılması gereken belirli bir kuyruk bağlantısı belirtmek isterseniz, bildiriminizde bir `viaConnections` metodu tanımlayabilirsiniz. Bu metod, kanal adı / kuyruk bağlantı adı çiftlerinden oluşan bir dizi döndürmelidir:
```php
/**
 * Her bildirim kanalı için hangi bağlantıların kullanılması gerektiğini belirle.
 *
 * @return array<string, string>
 */
public function viaConnections(): array
{
    return [
        'mail' => 'redis',
        'database' => 'sync',
    ];
}
```

#### Bildirim Kanalı Kuyruklarını Özelleştirme (Customizing Notification Channel Queues)
Bildirim tarafından desteklenen her bildirim kanalı için kullanılması gereken belirli bir kuyruk belirtmek isterseniz, bildiriminizde bir `viaQueues` metodu tanımlayabilirsiniz. Bu metod, kanal adı / kuyruk adı çiftlerinden oluşan bir dizi döndürmelidir:
```php
/**
 * Her bildirim kanalı için hangi kuyrukların kullanılması gerektiğini belirle.
 *
 * @return array<string, string>
 */
public function viaQueues(): array
{
    return [
        'mail' => 'mail-queue',
        'slack' => 'slack-queue',
    ];
}
```

#### Sıraya Konmuş Bildirim İşi Özelliklerini Özelleştirme (Customizing Queued Notification Job Properties)
Bildirim sınıfınızda özellikler (properties) tanımlayarak temeldeki sıraya konmuş işin davranışını özelleştirebilirsiniz. Bu özellikler, bildirimi gönderen sıraya konmuş iş tarafından devralınacaktır:
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Bildirimin denenebileceği maksimum sayı.
     *
     * @var int
     */
    public $tries = 5;

    /**
     * Bildirimin zaman aşımına uğramadan önce çalışabileceği saniye sayısı.
     *
     * @var int
     */
    public $timeout = 120;

    /**
     * Başarısız olmadan önce izin verilen maksimum işlenmemiş istisna sayısı.
     *
     * @var int
     */
    public $maxExceptions = 3;

    // ...
}
```
Sıraya konmuş bir bildirimin verilerinin gizliliğini ve bütünlüğünü şifreleme yoluyla sağlamak isterseniz, bildirim sınıfınıza `ShouldBeEncrypted` arayüzünü ekleyin:
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue, ShouldBeEncrypted
{
    use Queueable;

    // ...
}
```
Bu özellikleri doğrudan bildirim sınıfınızda tanımlamaya ek olarak, sıraya konmuş bildirim işi için geri alma stratejisini (backoff strategy) ve yeniden deneme zaman aşımını (retry timeout) belirtmek üzere `backoff` ve `retryUntil` metotlarını da tanımlayabilirsiniz:
```php
use DateTime;

/**
 * Bildirimi yeniden denemeden önce beklenecek saniye sayısını hesapla.
 */
public function backoff(): int
{
    return 3;
}

/**
 * Bildirimin zaman aşımına uğrayacağı zamanı belirle.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(5);
}
```
Bu iş özellikleri ve metotları hakkında daha fazla bilgi için lütfen sıraya konmuş işlerle ilgili dokümantasyonu inceleyin.

#### Sıraya Konmuş Bildirim Ara Yazılımları (Queued Notification Middleware)
Sıraya konmuş bildirimler, tıpkı sıraya konmuş işler gibi ara yazılım (middleware) tanımlayabilir. Başlamak için bildirim sınıfınızda bir `middleware` metodu tanımlayın. `middleware` metodu `$notifiable` ve `$channel` değişkenlerini alacaktır; bu, döndürülen ara yazılımı bildirimin hedefine göre özelleştirmenize olanak tanır:
```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Bildirim işinin geçmesi gereken ara yazılımları al.
 *
 * @return array<int, object>
 */
public function middleware(object $notifiable, string $channel)
{
    return match ($channel) {
        'mail' => [new RateLimited('postmark')],
        'slack' => [new RateLimited('slack')],
        default => [],
    };
}
```

#### Sıraya Konmuş Bildirimler ve Veritabanı İşlemleri (Queued Notifications and Database Transactions)
Sıraya konmuş bildirimler veritabanı işlemleri (database transactions) içinde gönderildiğinde, veritabanı işlemi tamamlanmadan (commit) önce kuyruk tarafından işlenebilirler. Bu olduğunda, veritabanı işlemi sırasında modellere veya veritabanı kayıtlarına yaptığınız tüm güncellemeler henüz veritabanına yansıtılmamış olabilir. Ayrıca, işlem içinde oluşturulan tüm modeller veya veritabanı kayıtları veritabanında mevcut olmayabilir. Bildiriminiz bu modellere bağlıysa, sıraya konmuş bildirimi gönderen iş işlendiğinde beklenmeyen hatalar oluşabilir.

Kuyruk bağlantınızın `after_commit` yapılandırma seçeneği `false` olarak ayarlanmışsa, belirli bir sıraya konmuş bildirimin tüm açık veritabanı işlemleri tamamlandıktan sonra gönderilmesi gerektiğini, bildirimi gönderirken `afterCommit` metodunu çağırarak belirtebilirsiniz:
```php
use App\Notifications\InvoicePaid;

$user->notify((new InvoicePaid($invoice))->afterCommit());
```
Alternatif olarak, bildiriminizin kurucusundan (constructor) `afterCommit` metodunu çağırabilirsiniz:
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Yeni bir bildirim örneği oluştur.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```
Bu sorunların etrafından nasıl çalışılacağı hakkında daha fazla bilgi edinmek için lütfen sıraya konmuş işler ve veritabanı işlemleri ile ilgili dokümantasyonu inceleyin.

#### Sıraya Konmuş Bir Bildirimin Gönderilip Gönderilmeyeceğini Belirleme (Determining if a Queued Notification Should Be Sent)
Sıraya konmuş bir bildirim arka plan işlemesi için kuyruğa gönderildikten sonra, tipik olarak bir kuyruk işçisi (queue worker) tarafından kabul edilecek ve hedeflenen alıcıya gönderilecektir.

Ancak, sıraya konmuş bildirimin bir kuyruk işçisi tarafından işlendikten sonra gönderilip gönderilmeyeceği konusunda nihai kararı vermek isterseniz, bildirim sınıfında bir `shouldSend` metodu tanımlayabilirsiniz. Bu metod `false` döndürürse, bildirim gönderilmeyecektir:
```php
/**
 * Bildirimin gönderilip gönderilmeyeceğini belirle.
 */
public function shouldSend(object $notifiable, string $channel): bool
{
    return $this->invoice->isPaid();
}
```

#### Bildirim Gönderildikten Sonra (After Sending Notifications)
Bir bildirim gönderildikten sonra kod çalıştırmak isterseniz, bildirim sınıfında bir `afterSending` metodu tanımlayabilirsiniz. Bu metod, bildirilebilir varlığı (notifiable entity), kanal adını ve kanaldan gelen yanıtı (response) alacaktır:
```php
/**
 * Bildirim gönderildikten sonra işle.
 */
public function afterSending(object $notifiable, string $channel, mixed $response): void
{
    // ...
}
```

### İsteğe Bağlı (On-Demand) Bildirimler
Bazen uygulamanızın bir "kullanıcısı" olarak saklanmayan birine bildirim göndermeniz gerekebilir. `Notification` facade'ının `route` metodunu kullanarak, bildirimi göndermeden önce geçici (ad-hoc) bildirim yönlendirme bilgileri belirtebilirsiniz:
```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'taylor@example.com')
    ->route('vonage', '5555555555')
    ->route('slack', '#slack-channel')
    ->route('broadcast', [new Channel('channel-name')])
    ->notify(new InvoicePaid($invoice));
```
E-posta kanalına isteğe bağlı bir bildirim gönderirken alıcının adını sağlamak isterseniz, `route` metoduna ikinci bir argüman olarak bir dizi iletebilirsiniz:
```php
Notification::route('mail', [
    'taylor@example.com' => 'Taylor Otwell',
])->notify(new InvoicePaid($invoice));
```