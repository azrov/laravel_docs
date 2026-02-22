# Laravel 12 Dokümantasyonu: Günlük Kaydı (Logging)

## Giriş

Uygulamanızın içinde neler olup bittiği hakkında daha fazla bilgi edinmenize yardımcı olmak için Laravel, mesajları dosyalara, sistem hata günlüğüne (system error log) ve hatta tüm ekibinizi bilgilendirmek için Slack'e kaydetmenize olanak tanıyan sağlam günlük kaydı (logging) servisleri sağlar.

Laravel günlük kaydı, "kanallara" (channels) dayanır. Her kanal, günlük bilgilerini yazmanın belirli bir yolunu temsil eder. Örneğin, `single` kanalı günlük dosyalarını tek bir günlük dosyasına yazarken, `slack` kanalı günlük mesajlarını Slack'e gönderir. Günlük mesajları, önem derecelerine (severity) göre birden çok kanala yazılabilir.

Perde arkasında Laravel, çeşitli güçlü günlük işleyicilerini (log handlers) destekleyen Monolog kütüphanesini kullanır. Laravel, bu işleyicileri yapılandırmayı çok kolaylaştırarak, uygulamanızın günlük işlemesini özelleştirmek için bunları birleştirip eşleştirmenize olanak tanır.

## Yapılandırma (Configuration)

Uygulamanızın günlük kaydı davranışını kontrol eden tüm yapılandırma seçenekleri `config/logging.php` yapılandırma dosyasında bulunur. Bu dosya, uygulamanızın günlük kanallarını yapılandırmanıza olanak tanır, bu nedenle mevcut kanalların her birini ve seçeneklerini gözden geçirdiğinizden emin olun. Aşağıda birkaç yaygın seçeneği inceleyeceğiz.

Varsayılan olarak Laravel, mesajları günlüğe kaydederken `stack` kanalını kullanacaktır. `stack` kanalı, birden çok günlük kanalını tek bir kanalda toplamak için kullanılır. Yığınlar (stacks) oluşturma hakkında daha fazla bilgi için aşağıdaki dokümantasyona bakın.

### Mevcut Kanal Sürücüleri (Available Channel Drivers)
Her günlük kanalı bir "sürücü" (driver) tarafından desteklenir. Sürücü, günlük mesajının aslında nasıl ve nereye kaydedileceğini belirler. Aşağıdaki günlük kanalı sürücüleri her Laravel uygulamasında mevcuttur. Bu sürücülerin çoğu için bir girdi, uygulamanızın `config/logging.php` yapılandırma dosyasında zaten bulunur, bu nedenle içeriğine aşina olmak için bu dosyayı incelediğinizden emin olun.

| İsim | Açıklama |
| :--- | :--- |
| `custom` | Bir kanal oluşturmak için belirtilen bir fabrikayı (factory) çağıran bir sürücü. |
| `daily` | Günlük olarak dönen (rotates daily) `RotatingFileHandler` tabanlı Monolog sürücüsü. |
| `errorlog` | `ErrorLogHandler` tabanlı Monolog sürücüsü. |
| `monolog` | Desteklenen herhangi bir Monolog işleyicisini (handler) kullanabilen bir Monolog fabrika sürücüsü. |
| `papertrail` | `SyslogUdpHandler` tabanlı Monolog sürücüsü. |
| `single` | Tek bir dosya veya yol tabanlı günlükçü kanalı (`StreamHandler`). |
| `slack` | `SlackWebhookHandler` tabanlı Monolog sürücüsü. |
| `stack` | "Çok kanallı" kanallar oluşturmayı kolaylaştıran bir sarmalayıcı (wrapper). |
| `syslog` | `SyslogHandler` tabanlı Monolog sürücüsü. |

`monolog` ve `custom` sürücüleri hakkında daha fazla bilgi edinmek için gelişmiş kanal özelleştirme (advanced channel customization) dokümantasyonuna göz atın.

#### Kanal Adını Yapılandırma (Configuring the Channel Name)
Varsayılan olarak Monolog, geçerli ortamla (örneğin `production` veya `local`) eşleşen bir "kanal adı" (channel name) ile başlatılır. Bu değeri değiştirmek için kanalınızın yapılandırmasına bir `name` seçeneği ekleyebilirsiniz:
```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

### Kanal Ön Gereksinimleri (Channel Prerequisites)

#### Tekil (Single) ve Günlük (Daily) Kanalları Yapılandırma
`single` ve `daily` kanallarının üç isteğe bağlı yapılandırma seçeneği vardır: `bubble`, `permission` ve `locking`.

| İsim | Açıklama | Varsayılan |
| :--- | :--- | :--- |
| `bubble` | Mesajların işlendikten sonra diğer kanallara kabarcıklanması (bubble up) gerekip gerekmediğini belirtir. | `true` |
| `locking` | Yazmadan önce günlük dosyasını kilitlemeye (lock) çalışır. | `false` |
| `permission` | Günlük dosyasının izinleri (permissions). | `0644` |

Ek olarak, `daily` kanalı için saklama politikası (retention policy), `LOG_DAILY_DAYS` ortam değişkeni veya `days` yapılandırma seçeneği ayarlanarak yapılandırılabilir.

| İsim | Açıklama | Varsayılan |
| :--- | :--- | :--- |
| `days` | Günlük günlük dosyalarının saklanması gereken gün sayısı. | `14` |

#### Papertrail Kanalını Yapılandırma (Configuring the Papertrail Channel)
`papertrail` kanalı, `host` ve `port` yapılandırma seçeneklerini gerektirir. Bunlar `PAPERTRAIL_URL` ve `PAPERTRAIL_PORT` ortam değişkenleri aracılığıyla tanımlanabilir. Bu değerleri Papertrail'den alabilirsiniz.

#### Slack Kanalını Yapılandırma (Configuring the Slack Channel)
`slack` kanalı bir `url` yapılandırma seçeneği gerektirir. Bu değer `LOG_SLACK_WEBHOOK_URL` ortam değişkeni aracılığıyla tanımlanabilir. Bu URL, Slack ekibiniz için yapılandırdığınız bir gelen webhook'un (incoming webhook) URL'siyle eşleşmelidir.

Varsayılan olarak Slack yalnızca `critical` seviye ve üzerindeki günlükleri alacaktır; ancak, bunu `LOG_LEVEL` ortam değişkenini kullanarak veya Slack günlük kanalınızın yapılandırma dizisindeki `level` yapılandırma seçeneğini değiştirerek ayarlayabilirsiniz.

### Kullanımdan Kaldırma (Deprecation) Uyarılarını Günlüğe Kaydetme
PHP, Laravel ve diğer kütüphaneler genellikle kullanıcılarına bazı özelliklerinin kullanımdan kaldırıldığını (deprecated) ve gelecekteki bir sürümde kaldırılacağını bildirir. Bu kullanımdan kaldırma uyarılarını günlüğe kaydetmek isterseniz, tercih ettiğiniz `deprecations` günlük kanalını `LOG_DEPRECATIONS_CHANNEL` ortam değişkenini kullanarak veya uygulamanızın `config/logging.php` yapılandırma dosyası içinde belirtebilirsiniz:
```php
'deprecations' => [
    'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
    'trace' => env('LOG_DEPRECATIONS_TRACE', false),
],

'channels' => [
    // ...
]
```
Veya, `deprecations` adında bir günlük kanalı tanımlayabilirsiniz. Bu ada sahip bir günlük kanalı varsa, kullanımdan kaldırmaları günlüğe kaydetmek için her zaman kullanılacaktır:
```php
'channels' => [
    'deprecations' => [
        'driver' => 'single',
        'path' => storage_path('logs/php-deprecation-warnings.log'),
    ],
],
```

## Günlük Yığınları Oluşturma (Building Log Stacks)

Daha önce belirtildiği gibi, `stack` sürücüsü, kolaylık olması açısından birden çok kanalı tek bir günlük kanalında birleştirmenize olanak tanır. Günlük yığınlarının nasıl kullanılacağını göstermek için, bir üretim uygulamasında görebileceğiniz örnek bir yapılandırmaya bakalım:
```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'],
        'ignore_exceptions' => false,
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
        'facility' => env('LOG_SYSLOG_FACILITY', LOG_USER),
        'replace_placeholders' => true,
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_USERNAME', 'Laravel Log'),
        'emoji' => env('LOG_SLACK_EMOJI', ':boom:'),
        'level' => env('LOG_LEVEL', 'critical'),
        'replace_placeholders' => true,
    ],
],
```
Bu yapılandırmayı inceleyelim. İlk olarak, `stack` kanalımızın `channels` seçeneği aracılığıyla diğer iki kanalı (`syslog` ve `slack`) bir araya getirdiğine dikkat edin. Bu nedenle, mesajlar günlüğe kaydedilirken, bu kanalların her ikisi de mesajı kaydetme fırsatına sahip olacaktır. Ancak, aşağıda göreceğimiz gibi, bu kanalların mesajı gerçekten kaydedip kaydetmeyeceği, mesajın ciddiyeti / "seviyesi" (level) tarafından belirlenebilir.

#### Günlük Seviyeleri (Log Levels)
Yukarıdaki örnekte `syslog` ve `slack` kanal yapılandırmalarında bulunan `level` yapılandırma seçeneğine dikkat edin. Bu seçenek, bir mesajın kanal tarafından günlüğe kaydedilmesi için sahip olması gereken minimum "seviyeyi" (minimum "level") belirler. Laravel'in günlük kaydı servislerini destekleyen Monolog, RFC 5424 spesifikasyonunda tanımlanan tüm günlük seviyelerini sunar. Azalan ciddiyet sırasına göre bu günlük seviyeleri şunlardır: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info` ve `debug`.

Diyelim ki `debug` yöntemini kullanarak bir mesaj günlüğe kaydediyoruz:
```php
Log::debug('Bilgilendirici bir mesaj.');
```
Yapılandırmamıza göre, `syslog` kanalı mesajı sistem günlüğüne yazacaktır; ancak, hata mesajı `critical` veya daha yüksek olmadığı için Slack'e gönderilmeyecektir. Ancak, bir `emergency` mesajı günlüğe kaydedersek, `emergency` seviyesi her iki kanal için minimum seviye eşiğimizin üzerinde olduğundan, hem sistem günlüğüne hem de Slack'e gönderilecektir:
```php
Log::emergency('Sistem çöktü!');
```

## Günlük Mesajları Yazma (Writing Log Messages)

`Log` facade'ını kullanarak günlüklere bilgi yazabilirsiniz. Daha önce belirtildiği gibi, günlükçü (logger), RFC 5424 spesifikasyonunda tanımlanan sekiz günlük seviyesini sağlar: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info` ve `debug`:
```php
use Illuminate\Support\Facades\Log;

Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```
İlgili seviye için bir mesaj günlüğe kaydetmek üzere bu yöntemlerden herhangi birini çağırabilirsiniz. Varsayılan olarak, mesaj, günlük kaydı yapılandırma dosyanız (`logging.php`) tarafından yapılandırıldığı şekilde varsayılan günlük kanalına yazılacaktır:
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Support\Facades\Log;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Belirtilen kullanıcının profilini göster.
     */
    public function show(string $id): View
    {
        Log::info('Showing the user profile for user: {id}', ['id' => $id]);

        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

### Bağlamsal Bilgi (Contextual Information)
Günlük metotlarına bir bağlamsal veri (contextual data) dizisi iletilebilir. Bu bağlamsal veri, günlük mesajıyla birlikte biçimlendirilecek ve görüntülenecektir:
```php
use Illuminate\Support\Facades\Log;

Log::info('User {id} failed to login.', ['id' => $user->id]);
```
Bazen, belirli bir kanaldaki sonraki tüm günlük girdilerine (log entries) dahil edilmesi gereken bazı bağlamsal bilgiler belirtmek isteyebilirsiniz. Örneğin, uygulamanıza gelen her istekle ilişkili bir istek kimliğini (request ID) günlüğe kaydetmek isteyebilirsiniz. Bunu başarmak için `Log` facade'ının `withContext` metodunu çağırabilirsiniz:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * Gelen bir isteği işle.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::withContext([
            'request-id' => $requestId
        ]);

        $response = $next($request);

        $response->headers->set('Request-Id', $requestId);

        return $response;
    }
}
```
Bağlamsal bilgileri tüm günlük kanallarında paylaşmak isterseniz, `Log::shareContext()` metodunu çağırabilirsiniz. Bu metod, bağlamsal bilgileri oluşturulan tüm kanallara ve daha sonra oluşturulacak herhangi bir kanala sağlayacaktır:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * Gelen bir isteği işle.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::shareContext([
            'request-id' => $requestId
        ]);

        // ...
    }
}
```
Sıraya konmuş işleri (queued jobs) işlerken günlük bağlamını paylaşmanız gerekiyorsa, iş ara yazılımını (job middleware) kullanabilirsiniz.

### Belirli Kanallara Yazma (Writing to Specific Channels)
Bazen bir mesajı uygulamanızın varsayılan kanalı dışındaki bir kanala günlüğe kaydetmek isteyebilirsiniz. Yapılandırma dosyanızda tanımlanan herhangi bir kanalı almak ve ona günlük kaydetmek için `Log` facade'ındaki `channel` metodunu kullanabilirsiniz:
```php
use Illuminate\Support\Facades\Log;

Log::channel('slack')->info('Something happened!');
```
Birden çok kanaldan oluşan, isteğe bağlı (on-demand) bir günlük yığını oluşturmak isterseniz, `stack` metodunu kullanabilirsiniz:
```php
Log::stack(['single', 'slack'])->info('Something happened!');
```

#### İsteğe Bağlı Kanallar (On-Demand Channels)
Uygulamanızın `logging` yapılandırma dosyasında bulunmayan bir yapılandırmayı çalışma zamanında sağlayarak isteğe bağlı bir kanal oluşturmak da mümkündür. Bunu başarmak için `Log` facade'ının `build` metoduna bir yapılandırma dizisi iletebilirsiniz:
```php
use Illuminate\Support\Facades\Log;

Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('Something happened!');
```
İsteğe bağlı bir kanalı, isteğe bağlı bir günlük yığınına dahil etmek isteyebilirsiniz. Bu, isteğe bağlı kanal örneğinizi `stack` metoduna iletilen diziye ekleyerek elde edilebilir:
```php
use Illuminate\Support\Facades\Log;

$channel = Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
]);

Log::stack(['slack', $channel])->info('Something happened!');
```

## Monolog Kanal Özelleştirme (Monolog Channel Customization)

### Mevcut Kanallar için Monolog'u Özelleştirme (Customizing Monolog for Channels)
Bazen mevcut bir kanal için Monolog'un nasıl yapılandırılacağı üzerinde tam kontrole ihtiyacınız olabilir. Örneğin, Laravel'in yerleşik `single` kanalı için özel bir Monolog `FormatterInterface` gerçekleştirimi (implementation) yapılandırmak isteyebilirsiniz.

Başlamak için kanalın yapılandırmasında bir `tap` dizisi tanımlayın. `tap` dizisi, oluşturulduktan sonra Monolog örneğini özelleştirme (veya "tap" ile "dokunma") fırsatına sahip olması gereken sınıfların bir listesini içermelidir. Bu sınıfların yerleştirilmesi gereken geleneksel bir konum yoktur, bu nedenle bu sınıfları içermek için uygulamanız içinde bir dizin oluşturmakta özgürsünüz:
```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'replace_placeholders' => true,
],
```
Kanalınızda `tap` seçeneğini yapılandırdıktan sonra, Monolog örneğinizi özelleştirecek sınıfı tanımlamaya hazırsınız. Bu sınıfın yalnızca tek bir metoda ihtiyacı vardır: `__invoke`, bu metot bir `Illuminate\Log\Logger` örneği alır. `Illuminate\Log\Logger` örneği, tüm metot çağrılarını temeldeki Monolog örneğine iletir (proxies):
```php
<?php

namespace App\Logging;

use Illuminate\Log\Logger;
use Monolog\Formatter\LineFormatter;

class CustomizeFormatter
{
    /**
     * Verilen günlükçü örneğini özelleştir.
     */
    public function __invoke(Logger $logger): void
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
            ));
        }
    }
}
```

### Monolog Kanalı Oluşturma (Creating Monolog Handler Channels)
Laravel'in built-in kanallarının sizin için yeterli olmadığı durumlarda, Monolog'un herhangi bir desteklenen işleyicisini (handler) kullanarak özel bir kanal oluşturmak için `monolog` sürücüsünü kullanabilirsiniz. `monolog` sürücüsü, `handler` bir yapılandırma seçeneği içermelidir. İsteğe bağlı olarak, `handlers`, `processors`, `formatter` ve `formatter_with` seçeneklerini de belirtebilirsiniz.
```php
'logentries' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'handler_with' => [
        'host' => 'my.logentries.com',
        'port' => '10000',
    ],
],
```

#### Özel Monolog İşleyicileri (Custom Monolog Handlers)
Laravel, birçok yaygın Monolog işleyicisi için önceden yapılandırılmış kanallar sağlasa da (`daily`, `slack`, `syslog` vb.), bazen desteklenmeyen özel bir Monolog işleyicisi kullanmanız gerekebilir. Bunu yapmak için `monolog` sürücüsünü kullanabilirsiniz. Bu sürücü, `handler` seçeneğini gerektirir. Alternatif olarak, `handler` bir işleyici örneği oluşturacak bir closure döndüren bir fabrika sınıfına da işaret edebilir.

`monolog` sürücüsü, işleyici yapılandırmasını esnek bir şekilde sağlamanıza olanak tanır:
```php
'custom' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\StreamHandler::class,
    'handler_with' => [
        'stream' => storage_path('logs/custom.log'),
    ],
    'formatter' => Monolog\Formatter\LineFormatter::class,
    'formatter_with' => [
        'format' => "[%datetime%] %channel%: %message%\n",
        'dateFormat' => 'Y-m-d H:i:s',
        'allowInlineLineBreaks' => true,
        'ignoreEmptyContextAndExtra' => true,
    ],
    'processors' => [
        Monolog\Processor\WebProcessor::class,
        // ...
    ],
],
```

### Özel Sürücüler Oluşturma (Creating Custom Drivers via Factories)
Tamamen özel bir sürücü oluşturmak istiyorsanız, `logging.php` yapılandırma dosyanıza `driver` olarak `custom`'ı belirtebilirsiniz. Ardından, Monolog örneği oluşturacak bir fabrika sınıfı belirtmek için bir `via` seçeneği kullanın. Bu fabrika sınıfı, Monolog örneğini döndüren bir `__invoke` metodu içermelidir:
```php
'custom' => [
    'driver' => 'custom',
    'via' => App\Logging\CreateCustomLogger::class,
],
```
Sonra, Monolog örneğini oluşturmaktan sorumlu fabrika sınıfınızı oluşturun:
```php
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * Özel bir Monolog örneği oluştur.
     */
    public function __invoke(array $config): Logger
    {
        return new Logger(/* ... */);
    }
}
```