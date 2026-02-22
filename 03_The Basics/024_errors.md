# Laravel 12 Dokümantasyonu: Hata Yönetimi (Error Handling)

## Giriş

Yeni bir Laravel projesine başladığınızda, hata ve istisna (exception) yönetimi sizin için zaten yapılandırılmıştır; ancak, istediğiniz zaman uygulamanızın `bootstrap/app.php` dosyasındaki `withExceptions` metodunu kullanarak istisnaların uygulamanız tarafından nasıl raporlanacağını (reported) ve oluşturulacağını (rendered) yönetebilirsiniz.

`withExceptions` closure'ına sağlanan `$exceptions` nesnesi, `Illuminate\Foundation\Configuration\Exceptions`'un bir örneğidir ve uygulamanızda istisna yönetiminden sorumludur. Bu dokümantasyon boyunca bu nesneyi daha derinlemesine inceleyeceğiz.

## Yapılandırma (Configuration)

`config/app.php` yapılandırma dosyanızdaki `debug` seçeneği, bir hata hakkında kullanıcıya aslında ne kadar bilgi gösterileceğini belirler. Varsayılan olarak bu seçenek, `.env` dosyanızda saklanan `APP_DEBUG` ortam değişkeninin değerine saygı gösterecek şekilde ayarlanmıştır.

Yerel geliştirme (local development) sırasında `APP_DEBUG` ortam değişkenini `true` olarak ayarlamalısınız. Üretim ortamınızda bu değer her zaman `false` olmalıdır. Değer üretimde `true` olarak ayarlanırsa, hassas yapılandırma değerlerini uygulamanızın son kullanıcılarına ifşa etme riskiniz vardır.

## İstisnaları Ele Alma (Handling Exceptions)

### İstisnaları Raporlama (Reporting Exceptions)
Laravel'de istisna raporlama (exception reporting), istisnaları günlüğe kaydetmek (log) veya bunları Sentry veya Flare gibi harici bir servise göndermek için kullanılır. Varsayılan olarak, istisnalar günlük kaydı yapılandırmanıza (logging configuration) göre kaydedilecektir. Ancak, istisnaları dilediğiniz gibi kaydetmekte özgürsünüz.

Farklı türdeki istisnaları farklı şekillerde raporlamanız gerekiyorsa, uygulamanızın `bootstrap/app.php` dosyasındaki `report` exception metodunu kullanarak, belirli bir türdeki bir istisnanın raporlanması gerektiğinde yürütülmesi gereken bir closure kaydedebilirsiniz. Laravel, closure'ın raporladığı istisna türünü, closure'ın tür ipucunu (type-hint) inceleyerek belirleyecektir:
```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```
`report` metodunu kullanarak özel bir istisna raporlama geri çağrısı (callback) kaydettiğinizde, Laravel yine de uygulama için varsayılan günlük kaydı yapılandırmasını kullanarak istisnayı günlüğe kaydedecektir. İstisnanın varsayılan günlük kaydı yığınına (default logging stack) iletilmesini durdurmak isterseniz, raporlama geri çağrınızı tanımlarken `stop` metodunu kullanabilir veya geri çağrıdan `false` döndürebilirsiniz:
```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $exceptions->report(function (InvalidOrderException $e) {
        return false;
    });
})
```

#### Global Günlük Bağlamı (Global Log Context)
Mümkün olduğunda, Laravel otomatik olarak geçerli kullanıcının ID'sini bağlamsal veri (contextual data) olarak her istisnanın günlük mesajına ekler. Uygulamanızın `bootstrap/app.php` dosyasındaki `context` exception metodunu kullanarak kendi global bağlamsal verilerinizi tanımlayabilirsiniz. Bu bilgi, uygulamanız tarafından yazılan her istisnanın günlük mesajına dahil edilecektir:
```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

#### İstisna Günlük Bağlamı (Exception Log Context)
Her günlük mesajına bağlam eklemek yararlı olabilirken, bazen belirli bir istisnanın, günlüklerinize dahil etmek isteyeceğiniz benzersiz bir bağlamı olabilir. Uygulamanızın istisnalarından birinde bir `context` metodu tanımlayarak, o istisnayla ilgili, istisnanın günlük kaydına eklenmesi gereken herhangi bir veriyi belirtebilirsiniz:
```php
<?php

namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
    // ...

    /**
     * İstisnanın bağlam bilgilerini al.
     *
     * @return array<string, mixed>
     */
    public function context(): array
    {
        return ['order_id' => $this->orderId];
    }
}
```

#### `report` Yardımcısı (The report Helper)
Bazen bir istisnayı raporlamanız ancak geçerli isteği işlemeye devam etmeniz gerekebilir. `report` yardımcı fonksiyonu, kullanıcıya bir hata sayfası oluşturmadan (rendering) bir istisnayı hızlıca raporlamanıza olanak tanır:
```php
public function isValid(string $value): bool
{
    try {
        // Değeri doğrula...
    } catch (Throwable $e) {
        report($e);

        return false;
    }
}
```

#### Raporlanan İstisnaların Yinelenmesini Önleme (Deduplicating Reported Exceptions)
Uygulamanız boyunca `report` fonksiyonunu kullanıyorsanız, bazen aynı istisnayı birden çok kez raporlayarak günlüklerinizde yinelenen girdiler (duplicate entries) oluşturabilirsiniz.

Bir istisnanın tek bir örneğinin (instance) yalnızca bir kez raporlanmasını sağlamak isterseniz, uygulamanızın `bootstrap/app.php` dosyasında `dontReportDuplicates` exception metodunu çağırabilirsiniz:
```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportDuplicates();
})
```
Artık `report` yardımcısı aynı istisna örneğiyle çağrıldığında, yalnızca ilk çağrı raporlanacaktır:
```php
$original = new RuntimeException('Whoops!');

report($original); // raporlandı

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // yok sayıldı
}

report($original); // yok sayıldı
report($caught); // yok sayıldı
```

### İstisna Günlük Seviyeleri (Exception Log Levels)
Mesajlar uygulamanızın günlüklerine yazıldığında, mesajlar belirli bir günlük seviyesinde (log level) yazılır; bu, günlüğe kaydedilen mesajın ciddiyetini veya önemini gösterir.

Yukarıda belirtildiği gibi, `report` metodunu kullanarak özel bir istisna raporlama geri çağrısı kaydettiğinizde bile, Laravel uygulama için varsayılan günlük kaydı yapılandırmasını kullanarak istisnayı yine de günlüğe kaydedecektir; ancak, günlük seviyesi bazen bir mesajın hangi kanallara (channels) kaydedildiğini etkileyebileceğinden, belirli istisnaların hangi günlük seviyesinde kaydedileceğini yapılandırmak isteyebilirsiniz.

Bunu başarmak için uygulamanızın `bootstrap/app.php` dosyasındaki `level` exception metodunu kullanabilirsiniz. Bu metod, ilk argüman olarak istisna türünü ve ikinci argüman olarak günlük seviyesini alır:
```php
use PDOException;
use Psr\Log\LogLevel;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```

### İstisnaları Türüne Göre Yok Sayma (Ignoring Exceptions by Type)
Uygulamanızı oluştururken, asla raporlamak istemeyeceğiniz bazı istisna türleri olacaktır. Bu istisnaları yok saymak için uygulamanızın `bootstrap/app.php` dosyasındaki `dontReport` exception metodunu kullanabilirsiniz. Bu metoda sağlanan herhangi bir sınıf asla raporlanmayacaktır; ancak, yine de özel oluşturma (custom rendering) mantığına sahip olabilirler:
```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```
Alternatif olarak, bir istisna sınıfını `Illuminate\Contracts\Debug\ShouldntReport` arayüzü ile "işaretleyebilirsiniz" (mark). Bir istisna bu arayüzle işaretlendiğinde, Laravel'in istisna işleyicisi (exception handler) tarafından asla raporlanmayacaktır:
```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Contracts\Debug\ShouldntReport;

class PodcastProcessingException extends Exception implements ShouldntReport
{
    //
}
```

### İstisnaları Oluşturma (Rendering Exceptions)
Varsayılan olarak, Laravel istisna işleyicisi (exception handler) istisnaları sizin için bir HTTP yanıtına dönüştürecektir (convert). Ancak, belirli bir türdeki istisnalar için özel bir oluşturma (rendering) closure'ı kaydetmekte özgürsünüz. Bunu, uygulamanızın `bootstrap/app.php` dosyasındaki `render` exception metodunu kullanarak gerçekleştirebilirsiniz.

`render` metoduna iletilen closure, `response` yardımcısı aracılığıyla oluşturulabilen bir `Illuminate\Http\Response` örneği döndürmelidir. Laravel, closure'ın oluşturduğu istisna türünü, closure'ın tür ipucunu (type-hint) inceleyerek belirleyecektir:
```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 500);
    });
})
```
Ayrıca, yerleşik Laravel veya Symfony istisnaları (örneğin `NotFoundHttpException`) için oluşturma davranışını geçersiz kılmak (override) için `render` metodunu kullanabilirsiniz. `render` metoduna verilen closure bir değer döndürmezse, Laravel'in varsayılan istisna oluşturması kullanılacaktır:
```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
})
```

#### İstisnaları JSON Olarak Oluşturma (Rendering Exceptions as JSON)
Bir istisnayı oluştururken (rendering), Laravel otomatik olarak isteğin `Accept` başlığına bağlı olarak istisnanın HTML mi yoksa JSON yanıtı olarak mı oluşturulması gerektiğini belirleyecektir. Laravel'in HTML veya JSON istisna yanıtlarının nasıl oluşturulacağını belirleme şeklini özelleştirmek isterseniz, `shouldRenderJsonWhen` metodunu kullanabilirsiniz:
```php
use Illuminate\Http\Request;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
        if ($request->is('admin/*')) {
            return true;
        }

        return $request->expectsJson();
    });
})
```

#### İstisna Yanıtını Özelleştirme (Customizing the Exception Response)
Nadiren de olsa, Laravel'in istisna işleyicisi tarafından oluşturulan tüm HTTP yanıtını özelleştirmeniz gerekebilir. Bunu başarmak için `respond` metodunu kullanarak bir yanıt özelleştirme closure'ı kaydedebilirsiniz:
```php
use Symfony\Component\HttpFoundation\Response;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->respond(function (Response $response) {
        if ($response->getStatusCode() === 419) {
            return back()->with([
                'message' => 'The page expired, please try again.',
            ]);
        }

        return $response;
    });
})
```

### Raporlanabilir ve Oluşturulabilir İstisnalar (Reportable and Renderable Exceptions)
Uygulamanızın `bootstrap/app.php` dosyasında özel raporlama ve oluşturma davranışı tanımlamak yerine, `report` ve `render` metotlarını doğrudan uygulamanızın istisnaları üzerinde tanımlayabilirsiniz. Bu metotlar mevcut olduğunda, framework tarafından otomatik olarak çağrılacaklardır:
```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    /**
     * İstisnayı raporla.
     */
    public function report(): void
    {
        // ...
    }

    /**
     * İstisnayı bir HTTP yanıtı olarak oluştur.
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```
İstisnanız, zaten oluşturulabilir (renderable) bir istisnayı (yerleşik bir Laravel veya Symfony istisnası gibi) genişletiyorsa (extend), istisnanın varsayılan HTTP yanıtını oluşturmak için istisnanın `render` metodundan `false` döndürebilirsiniz:
```php
/**
 * İstisnayı bir HTTP yanıtı olarak oluştur.
 */
public function render(Request $request): Response|bool
{
    if (/** İstisnanın özel oluşturma gerektirip gerektirmediğini belirle */) {

        return response(/* ... */);
    }

    return false;
}
```
İstisnanız, yalnızca belirli koşullar karşılandığında gerekli olan özel raporlama mantığı içeriyorsa, Laravel'e bazen istisnayı varsayılan istisna işleme yapılandırmasını kullanarak raporlaması talimatını vermeniz gerekebilir. Bunu başarmak için istisnanın `report` metodundan `false` döndürebilirsiniz:
```php
/**
 * İstisnayı raporla.
 */
public function report(): bool
{
    if (/** İstisnanın özel raporlama gerektirip gerektirmediğini belirle */) {

        // ...

        return true;
    }

    return false;
}
```
`report` metodunun gerektirdiği herhangi bir bağımlılığı tip belirtebilirsiniz (type-hint) ve bunlar Laravel'in servis kabı (service container) tarafından metoda otomatik olarak enjekte edilecektir.

### Raporlanan İstisnaları Sınırlama (Throttling Reported Exceptions)
Uygulamanız çok sayıda istisna raporluyorsa, aslında kaç tane istisnanın günlüğe kaydedildiğini veya uygulamanızın harici hata izleme servisine gönderildiğini sınırlamak (throttle) isteyebilirsiniz.

İstisnaların rastgele bir örnekleme oranını (random sample rate) almak için uygulamanızın `bootstrap/app.php` dosyasındaki `throttle` exception metodunu kullanabilirsiniz. `throttle` metodu, bir `Lottery` örneği döndürmesi gereken bir closure alır:
```php
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```
Ayrıca, istisna türüne göre koşullu olarak örnekleme yapmak da mümkündür. Yalnızca belirli bir istisna sınıfının örneklerini örneklemek isterseniz, yalnızca bu sınıf için bir `Lottery` örneği döndürebilirsiniz:
```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```