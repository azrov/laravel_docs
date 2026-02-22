# Laravel 12 Dokümantasyonu: Test Etme: Başlarken (Testing: Getting Started)

## Giriş

Laravel, test etme düşünülerek inşa edilmiştir. Aslında, Pest ve PHPUnit ile test desteği kutudan çıktığı gibi dahildir ve uygulamanız için bir `phpunit.xml` dosyası zaten hazırlanmıştır. Framework ayrıca, uygulamalarınızı anlamlı bir şekilde test etmenize olanak tanıyan kullanışlı yardımcı metotlar (helper methods) ile birlikte gelir.

Varsayılan olarak, uygulamanızın `tests` dizini iki dizin içerir: `Feature` ve `Unit`. Birim testleri (unit tests), kodunuzun çok küçük, izole edilmiş bir bölümüne odaklanan testlerdir. Aslında, çoğu birim testi muhtemelen tek bir metoda odaklanır. "Unit" test dizininizdeki testler Laravel uygulamanızı başlatmaz (boot) ve bu nedenle uygulamanızın veritabanına veya diğer framework servislerine erişemez.

Özellik testleri (feature tests), kodunuzun daha büyük bir bölümünü test edebilir; bu, birkaç nesnenin birbirleriyle nasıl etkileşime girdiğini veya hatta bir JSON uç noktasına (endpoint) yapılan tam bir HTTP isteğini içerebilir. Genel olarak, testlerinizin çoğu özellik testleri olmalıdır. Bu test türleri, sisteminizin bir bütün olarak beklendiği gibi çalıştığına dair en yüksek güveni sağlar.

Hem `Feature` hem de `Unit` test dizinlerinde bir `ExampleTest.php` dosyası sağlanır. Yeni bir Laravel uygulaması kurduktan sonra, testlerinizi çalıştırmak için `vendor/bin/pest`, `vendor/bin/phpunit` veya `php artisan test` komutlarını çalıştırın.

## Ortam (Environment)

Testleri çalıştırırken Laravel, `phpunit.xml` dosyasında tanımlanan ortam değişkenleri (environment variables) nedeniyle yapılandırma ortamını (configuration environment) otomatik olarak `testing` olarak ayarlayacaktır. Laravel ayrıca oturum (session) ve önbelleği (cache) otomatik olarak `array` sürücüsüne yapılandırır, böylece test sırasında hiçbir oturum veya önbellek verisi kalıcı olmaz (persisted).

Gerektiği gibi başka test ortamı yapılandırma değerleri tanımlamakta özgürsünüz. `testing` ortam değişkenleri, uygulamanızın `phpunit.xml` dosyasında yapılandırılabilir, ancak testleri çalıştırmadan önce `config:clear` Artisan komutunu kullanarak yapılandırma önbelleğinizi (configuration cache) temizlediğinizden emin olun!

#### `.env.testing` Ortam Dosyası (The `.env.testing` Environment File)
Ayrıca, projenizin kökünde bir `.env.testing` dosyası oluşturabilirsiniz. Bu dosya, Pest ve PHPUnit testlerini çalıştırırken veya Artisan komutlarını `--env=testing` seçeneğiyle çalıştırırken `.env` dosyası yerine kullanılacaktır.

## Test Oluşturma (Creating Tests)

Yeni bir test durumu (test case) oluşturmak için `make:test` Artisan komutunu kullanın. Varsayılan olarak, testler `tests/Feature` dizinine yerleştirilecektir:
```bash
php artisan make:test UserTest
```
`tests/Unit` dizini içinde bir test oluşturmak isterseniz, `make:test` komutunu çalıştırırken `--unit` seçeneğini kullanabilirsiniz:
```bash
php artisan make:test UserTest --unit
```
Test iskeletleri (stubs), iskelet yayınlama (stub publishing) kullanılarak özelleştirilebilir.

Test oluşturulduktan sonra, normalde Pest veya PHPUnit kullanarak yapacağınız gibi test tanımlayabilirsiniz. Testlerinizi çalıştırmak için terminalinizden `vendor/bin/pest`, `vendor/bin/phpunit` veya `php artisan test` komutunu çalıştırın:
```php
test('temel', function () {
    expect(true)->toBeTrue();
});
```
```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir test örneği.
     */
    public function test_basic_test(): void
    {
        $this->assertTrue(true);
    }
}
```
Bir test sınıfı içinde kendi `setUp` / `tearDown` metotlarınızı tanımlarsanız, ilgili üst sınıfın `parent::setUp()` / `parent::tearDown()` metotlarını çağırdığınızdan emin olun. Tipik olarak, `parent::setUp()`'ı kendi `setUp` metodunuzun başında ve `parent::tearDown()`'ı kendi `tearDown` metodunuzun sonunda çağırmalısınız.

## Testleri Çalıştırma (Running Tests)

Daha önce belirtildiği gibi, testleri yazdıktan sonra `pest` veya `phpunit` kullanarak çalıştırabilirsiniz:
```bash
./vendor/bin/pest
```
```bash
./vendor/bin/phpunit
```
`pest` veya `phpunit` komutlarına ek olarak, testlerinizi çalıştırmak için `test` Artisan komutunu da kullanabilirsiniz. Artisan test çalıştırıcısı, geliştirmeyi ve hata ayıklamayı kolaylaştırmak için ayrıntılı test raporları sağlar:
```bash
php artisan test
```
`pest` veya `phpunit` komutlarına iletilebilen tüm argümanlar, Artisan `test` komutuna da iletilebilir:
```bash
php artisan test --testsuite=Feature --stop-on-failure
```

### Testleri Paralel Olarak Çalıştırma (Running Tests in Parallel)
Varsayılan olarak Laravel ve Pest / PHPUnit, testlerinizi tek bir süreç (process) içinde sırayla (sequentially) yürütür. Ancak, testleri birden çok süreçte aynı anda çalıştırarak testlerinizi çalıştırma süresini büyük ölçüde azaltabilirsiniz. Başlamak için `brianium/paratest` Composer paketini bir "dev" bağımlılığı olarak kurmalısınız. Ardından, `test` Artisan komutunu çalıştırırken `--parallel` seçeneğini ekleyin:
```bash
composer require brianium/paratest --dev

php artisan test --parallel
```
Varsayılan olarak Laravel, makinenizdeki mevcut CPU çekirdeği sayısı kadar süreç oluşturacaktır. Ancak, `--processes` seçeneğini kullanarak süreç sayısını ayarlayabilirsiniz:
```bash
php artisan test --parallel --processes=4
```
Testleri paralel olarak çalıştırırken, bazı Pest / PHPUnit seçenekleri (`--do-not-cache-result` gibi) mevcut olmayabilir.

#### Paralel Test ve Veritabanları (Parallel Testing and Databases)
Birincil veritabanı bağlantısı yapılandırdığınız sürece Laravel, testlerinizi çalıştıran her bir paralel süreç için otomatik olarak bir test veritabanı oluşturmayı ve migration'larını çalıştırmayı (migrate) yönetir. Test veritabanları, süreç başına benzersiz olan bir süreç token'ı (process token) ile son eklenir (suffixed). Örneğin, iki paralel test süreciniz varsa, Laravel `your_db_test_1` ve `your_db_test_2` test veritabanlarını oluşturacak ve kullanacaktır.

Varsayılan olarak, test veritabanları, sonraki test çağrılarında tekrar kullanılabilmeleri için `test` Artisan komutuna yapılan çağrılar arasında kalıcıdır (persist). Ancak, `--recreate-databases` seçeneğini kullanarak bunları yeniden oluşturabilirsiniz:
```bash
php artisan test --parallel --recreate-databases
```

#### Paralel Test Kancaları (Parallel Testing Hooks)
Bazen, uygulamanızın testleri tarafından kullanılan belirli kaynakları, birden çok test süreci tarafından güvenle kullanılabilecek şekilde hazırlamanız gerekebilir.

`ParallelTesting` facade'ını kullanarak, bir süreç (process) veya test durumunun (test case) `setUp` ve `tearDown` işlemlerinde yürütülecek kodu belirtebilirsiniz. Verilen closure'lar, sırasıyla süreç token'ını ve geçerli test durumunu içeren `$token` ve `$testCase` değişkenlerini alır:
```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\ParallelTesting;
use Illuminate\Support\ServiceProvider;
use PHPUnit\Framework\TestCase;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        ParallelTesting::setUpProcess(function (int $token) {
            // ...
        });

        ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        // Bir test veritabanı oluşturulduğunda çalıştırılır...
        ParallelTesting::setUpTestDatabase(function (string $database, int $token) {
            Artisan::call('db:seed');
        });

        ParallelTesting::tearDownTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        ParallelTesting::tearDownProcess(function (int $token) {
            // ...
        });
    }
}
```

#### Paralel Test Token'ına Erişim (Accessing the Parallel Testing Token)
Uygulamanızın test kodundaki başka bir konumdan geçerli paralel süreç "token"ına erişmek isterseniz, `token` metodunu kullanabilirsiniz. Bu token, tek bir test süreci için benzersiz, bir dize tanımlayıcıdır (string identifier) ve kaynakları paralel test süreçleri arasında bölümlendirmek (segment) için kullanılabilir. Örneğin Laravel, bu token'ı her bir paralel test süreci tarafından oluşturulan test veritabanlarının sonuna otomatik olarak ekler:
```php
$token = ParallelTesting::token();
```

### Test Kapsamını (Coverage) Raporlama (Reporting Test Coverage)
Bu özellik Xdebug veya PCOV gerektirir.

Uygulama testlerinizi çalıştırırken, test durumlarınızın gerçekten uygulama kodunu kapsayıp kapsamadığını ve testlerinizi çalıştırırken ne kadar uygulama kodunun kullanıldığını belirlemek isteyebilirsiniz. Bunu başarmak için, `test` komutunu çağırırken `--coverage` seçeneğini sağlayabilirsiniz:
```bash
php artisan test --coverage
```

#### Minimum Kapsam Eşiğini (Coverage Threshold) Zorunlu Kılma (Enforcing a Minimum Coverage Threshold)
Uygulamanız için minimum bir test kapsamı eşiği tanımlamak üzere `--min` seçeneğini kullanabilirsiniz. Bu eşik karşılanmazsa test süiti başarısız olacaktır:
```bash
php artisan test --coverage --min=80.3
```

### Testleri Profilleme (Profiling Tests)
Artisan test çalıştırıcısı ayrıca uygulamanızın en yavaş testlerini listelemek için kullanışlı bir mekanizma içerir. `test` komutunu `--profile` seçeneğiyle çağırın, size en yavaş on testinizin bir listesi sunulacak ve test süitinizi hızlandırmak için hangi testlerin iyileştirilebileceğini kolayca araştırmanıza olanak tanıyacaktır:
```bash
php artisan test --profile
```

## Yapılandırma Önbelleğe Alma (Configuration Caching)

Testleri çalıştırırken Laravel, her bir test metodu için uygulamayı başlatır (boots). Önbelleğe alınmış bir yapılandırma dosyası olmadan, uygulamanızdaki her yapılandırma dosyası bir testin başlangıcında yüklenmelidir. Yapılandırmayı bir kez oluşturmak ve tek bir çalıştırmadaki tüm testler için yeniden kullanmak üzere, `Illuminate\Foundation\Testing\WithCachedConfig` özelliğini (trait) kullanabilirsiniz:
```php
use Illuminate\Foundation\Testing\WithCachedConfig;

pest()->use(WithCachedConfig::class);

// ...
```
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithCachedConfig;
use Tests\TestCase;

class ConfigTest extends TestCase
{
    use WithCachedConfig;

    // ...
}
```