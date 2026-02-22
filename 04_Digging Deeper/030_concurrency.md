# Laravel 12 Dokümantasyonu: Eşzamanlılık (Concurrency)

## Giriş

Bazen birbirine bağımlı olmayan birkaç yavaş görevi yürütmeniz gerekebilir. Çoğu durumda, görevleri eşzamanlı (concurrently) yürüterek önemli performans iyileştirmeleri elde edilebilir. Laravel'in `Concurrency` facade'ı, kapatmaları (closures) eşzamanlı olarak yürütmek için basit ve kullanışlı bir API sağlar.

#### Nasıl Çalışır? (How it Works)
Laravel eşzamanlılığı, verilen kapatmaları serileştirerek (serializing) ve bunları gizli bir Artisan CLI komutuna göndererek elde eder. Bu komut, kapatmaların serisini çözer (unserializes) ve kendi PHP süreci içinde onu çağırır. Kapatma çağrıldıktan sonra, ortaya çıkan değer ana sürece (parent process) geri serileştirilir.

`Concurrency` facade'ı üç sürücüyü (drivers) destekler: `process` (varsayılan), `fork` ve `sync`.

`fork` sürücüsü, varsayılan `process` sürücüsüne kıyasla daha iyi performans sunar, ancak yalnızca PHP'nin CLI bağlamında kullanılabilir, çünkü PHP web istekleri sırasında forking'i desteklemez. `fork` sürücüsünü kullanmadan önce, `spatie/fork` paketini kurmanız gerekir:
```bash
composer require spatie/fork
```
`sync` sürücüsü öncelikle test sırasında, tüm eşzamanlılığı devre dışı bırakmak ve verilen kapatmaları ana süreç içinde sırayla (in sequence) yürütmek istediğinizde kullanışlıdır.

## Eşzamanlı Görevleri Çalıştırma (Running Concurrent Tasks)

Eşzamanlı görevleri çalıştırmak için `Concurrency` facade'ının `run` metodunu çağırabilirsiniz. `run` metodu, alt PHP süreçlerinde (child PHP processes) eşzamanlı olarak yürütülmesi gereken bir kapatmalar dizisi (array of closures) kabul eder:
```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```
Belirli bir sürücüyü kullanmak için `driver` metodunu kullanabilirsiniz:
```php
$results = Concurrency::driver('fork')->run(...);
```
Veya, varsayılan eşzamanlılık sürücüsünü değiştirmek için `config:publish` Artisan komutu aracılığıyla `concurrency` yapılandırma dosyasını yayınlamalı ve dosyadaki `default` seçeneğini güncellemelisiniz:
```bash
php artisan config:publish concurrency
```

## Eşzamanlı Görevleri Erteleme (Deferring Concurrent Tasks)

Bir dizi kapatmayı eşzamanlı olarak yürütmek istiyor ancak bu kapatmalar tarafından döndürülen sonuçlarla ilgilenmiyorsanız, `defer` metodunu kullanmayı düşünmelisiniz. `defer` metodu çağrıldığında, verilen kapatmalar hemen yürütülmez. Bunun yerine Laravel, kapatmaları HTTP yanıtı kullanıcıya gönderildikten **sonra** eşzamanlı olarak yürütecektir:
```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```