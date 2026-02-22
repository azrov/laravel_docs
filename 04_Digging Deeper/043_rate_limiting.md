# Laravel 12 Dokümantasyonu: Hız Sınırlama (Rate Limiting)

## Giriş

Laravel, uygulamanızın önbelleğiyle (cache) birlikte çalışan, kullanımı basit bir hız sınırlama (rate limiting) soyutlaması (abstraction) içerir. Bu, belirli bir zaman aralığında (window of time) herhangi bir eylemi sınırlamanın kolay bir yolunu sağlar.

Gelen HTTP isteklerini hız sınırlamaya ilgi duyuyorsanız, lütfen hız sınırlayıcı ara yazılım (rate limiter middleware) dokümantasyonuna bakın.

### Önbellek Yapılandırması (Cache Configuration)
Tipik olarak, hız sınırlayıcı (rate limiter), uygulamanızın önbellek yapılandırma dosyasındaki `default` anahtarı tarafından tanımlanan varsayılan uygulama önbelleğinizi (default application cache) kullanır. Ancak, uygulamanızın önbellek yapılandırma dosyasında bir `limiter` anahtarı tanımlayarak hız sınırlayıcının hangi önbellek sürücüsünü (cache driver) kullanması gerektiğini belirtebilirsiniz:
```php
'default' => env('CACHE_STORE', 'database'),

'limiter' => 'redis',
```

## Temel Kullanım (Basic Usage)

`Illuminate\Support\Facades\RateLimiter` facade'ı, hız sınırlayıcı ile etkileşim kurmak için kullanılabilir. Hız sınırlayıcı tarafından sunulan en basit metod, belirli bir saniye sayısı için belirli bir geri çağrıyı (callback) hız sınırlayan `attempt` metodudur.

`attempt` metodu, geri çağrının kalan hiçbir deneme hakkı (remaining attempts) kalmadığında `false` döndürür; aksi takdirde, `attempt` metodu geri çağrının sonucunu veya `true` döndürecektir. `attempt` metodu tarafından kabul edilen ilk argüman, hız sınırlayıcı "anahtarıdır" (key) - bu, hız sınırlaması uygulanan eylemi temsil eden, sizin seçeceğiniz herhangi bir dize olabilir:
```php
use Illuminate\Support\Facades\RateLimiter;

$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perMinute = 5,
    function() {
        // Mesaj gönder...
    }
);

if (! $executed) {
    return 'Çok fazla mesaj gönderildi!';
}
```
Gerekirse, `attempt` metoduna dördüncü bir argüman sağlayabilirsiniz; bu, "azalma oranı"dır (decay rate), yani kullanılabilir denemelerin sıfırlanacağı saniye sayısıdır. Örneğin, yukarıdaki örneği her iki dakikada beş denemeye izin verecek şekilde değiştirebiliriz:
```php
$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perTwoMinutes = 5,
    function() {
        // Mesaj gönder...
    },
    $decayRate = 120,
);
```

### Denemeleri Manuel Olarak Artırma (Manually Incrementing Attempts)
Hız sınırlayıcı ile manuel olarak etkileşim kurmak isterseniz, çeşitli başka metotlar mevcuttur. Örneğin, belirli bir hız sınırlayıcı anahtarının dakika başına izin verilen maksimum deneme sayısını aşıp aşmadığını belirlemek için `tooManyAttempts` metodunu çağırabilirsiniz:
```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    return 'Çok fazla deneme!';
}

RateLimiter::increment('send-message:'.$user->id);

// Mesaj gönder...
```
Alternatif olarak, belirli bir anahtar için kalan deneme sayısını almak üzere `remaining` metodunu kullanabilirsiniz. Belirli bir anahtarın kalan deneme hakkı varsa, toplam deneme sayısını artırmak için `increment` metodunu çağırabilirsiniz:
```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
    RateLimiter::increment('send-message:'.$user->id);

    // Mesaj gönder...
}
```
Belirli bir hız sınırlayıcı anahtarı için değeri birden fazla artırmak isterseniz, `increment` metoduna istenen miktarı sağlayabilirsiniz:
```php
RateLimiter::increment('send-message:'.$user->id, amount: 5);
```

#### Sınırlayıcı Kullanılabilirliğini Belirleme (Determining Limiter Availability)
Bir anahtarın hiç deneme hakkı kalmadığında, `availableIn` metodu daha fazla denemenin kullanılabilir olacağı ana kadar kalan saniye sayısını döndürür:
```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    $seconds = RateLimiter::availableIn('send-message:'.$user->id);

    return 'Tekrar deneyebilirsiniz '.$seconds.' saniye sonra.';
}

RateLimiter::increment('send-message:'.$user->id);

// Mesaj gönder...
```

### Denemeleri Temizleme (Clearing Attempts)
Belirli bir hız sınırlayıcı anahtarı için deneme sayısını `clear` metodunu kullanarak sıfırlayabilirsiniz. Örneğin, belirli bir mesaj alıcı tarafından okunduğunda deneme sayısını sıfırlayabilirsiniz:
```php
use App\Models\Message;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Mesajı okundu olarak işaretle.
 */
public function read(Message $message): Message
{
    $message->markAsRead();

    RateLimiter::clear('send-message:'.$message->user_id);

    return $message;
}
```