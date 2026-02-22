# Laravel 12 Dokümantasyonu: Laravel Dusk

## Giriş

Pest 4 artık, Laravel Dusk'a kıyasla önemli performans ve kullanılabilirlik iyileştirmeleri sunan otomatik tarayıcı testleri (automated browser testing) içermektedir. Yeni projeler için, tarayıcı testlerinde Pest kullanmanızı öneririz.

Laravel Dusk, anlamlı, kullanımı kolay bir tarayıcı otomasyonu ve test API'si sağlar. Varsayılan olarak Dusk, yerel bilgisayarınıza JDK veya Selenium kurmanızı gerektirmez. Bunun yerine Dusk, bağımsız bir ChromeDriver kurulumu kullanır. Ancak, istediğiniz diğer Selenium uyumlu sürücüleri kullanmakta özgürsünüz.

## Kurulum (Installation)

Başlamak için Google Chrome'u kurmalı ve projenize `laravel/dusk` Composer bağımlılığını eklemelisiniz:
```bash
composer require laravel/dusk --dev
```
Dusk'un servis sağlayıcısını manuel olarak kaydediyorsanız, bunu asla üretim ortamınızda (production environment) kaydetmemelisiniz, çünkü bu, rastgele kullanıcıların uygulamanızla kimlik doğrulaması yapabilmesine yol açabilir.

Dusk paketini kurduktan sonra, `dusk:install` Artisan komutunu çalıştırın. `dusk:install` komutu bir `tests/Browser` dizini, örnek bir Dusk testi oluşturacak ve işletim sisteminiz için Chrome Driver ikili dosyasını (binary) kuracaktır:
```bash
php artisan dusk:install
```
Ardından, uygulamanızın `.env` dosyasında `APP_URL` ortam değişkenini ayarlayın. Bu değer, uygulamanıza bir tarayıcıda erişmek için kullandığınız URL ile eşleşmelidir.

Yerel geliştirme ortamınızı yönetmek için Laravel Sail kullanıyorsanız, lütfen Dusk testlerini yapılandırma ve çalıştırma hakkındaki Sail dokümantasyonuna da bakın.

### ChromeDriver Kurulumlarını Yönetme (Managing ChromeDriver Installations)
ChromeDriver'ın, Laravel Dusk'un `dusk:install` komutu aracılığıyla kurduğundan farklı bir sürümünü kurmak isterseniz, `dusk:chrome-driver` komutunu kullanabilirsiniz:
```bash
# İşletim sisteminiz için ChromeDriver'ın en son sürümünü kur...
php artisan dusk:chrome-driver

# İşletim sisteminiz için belirli bir ChromeDriver sürümünü kur...
php artisan dusk:chrome-driver 86

# Desteklenen tüm işletim sistemleri için belirli bir ChromeDriver sürümünü kur...
php artisan dusk:chrome-driver --all

# İşletim sisteminiz için algılanan Chrome / Chromium sürümüyle eşleşen ChromeDriver sürümünü kur...
php artisan dusk:chrome-driver --detect
```
Dusk, `chromedriver` ikili dosyalarının çalıştırılabilir (executable) olmasını gerektirir. Dusk'ı çalıştırırken sorun yaşıyorsanız, aşağıdaki komutu kullanarak ikili dosyaların çalıştırılabilir olduğundan emin olmalısınız: `chmod -R 0755 vendor/laravel/dusk/bin/`.

### Diğer Tarayıcıları Kullanma (Using Other Browsers)
Varsayılan olarak Dusk, tarayıcı testlerinizi çalıştırmak için Google Chrome ve bağımsız bir ChromeDriver kurulumu kullanır. Ancak, kendi Selenium sunucunuzu başlatabilir ve testlerinizi istediğiniz herhangi bir tarayıcıda çalıştırabilirsiniz.

Başlamak için, uygulamanız için temel Dusk test durumu olan `tests/DuskTestCase.php` dosyanızı açın. Bu dosya içinde, `startChromeDriver` metoduna yapılan çağrıyı kaldırabilirsiniz. Bu, Dusk'un ChromeDriver'ı otomatik olarak başlatmasını durduracaktır:
```php
/**
 * Dusk test yürütmesi için hazırlan.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```
Ardından, `driver` metodunu seçtiğiniz URL ve bağlantı noktasına (port) bağlanacak şekilde değiştirebilirsiniz. Ek olarak, WebDriver'a iletilmesi gereken "istenen yetenekleri" (desired capabilities) de değiştirebilirsiniz:
```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * RemoteWebDriver örneğini oluştur.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

## Başlarken (Getting Started)

### Test Oluşturma (Generating Tests)
Bir Dusk testi oluşturmak için `dusk:make` Artisan komutunu kullanın. Oluşturulan test, `tests/Browser` dizinine yerleştirilecektir:
```bash
php artisan dusk:make LoginTest
```

### Her Testten Sonra Veritabanını Sıfırlama (Resetting the Database After Each Test)
Yazacağınız testlerin çoğu, uygulamanızın veritabanından veri alan sayfalarla etkileşime girecektir; ancak Dusk testleriniz asla `RefreshDatabase` özelliğini (trait) kullanmamalıdır. `RefreshDatabase` özelliği, HTTP istekleri arasında geçerli veya kullanılabilir olmayacak veritabanı işlemlerini (database transactions) kullanır. Bunun yerine, iki seçeneğiniz vardır: `DatabaseMigrations` özelliği ve `DatabaseTruncation` özelliği.

#### Veritabanı Migration'larını Kullanma (Using Database Migrations)
`DatabaseMigrations` özelliği, her testten önce veritabanı migration'larınızı çalıştıracaktır. Ancak, her test için veritabanı tablolarınızı bırakıp (drop) yeniden oluşturmak, tabloları kesmekten (truncate) tipik olarak daha yavaştır.
```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

//
```
```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    //
}
```
SQLite bellek içi (in-memory) veritabanları, Dusk testleri yürütülürken kullanılamaz. Tarayıcı kendi sürecinde (process) yürütüldüğü için, diğer süreçlerin bellek içi veritabanlarına erişemez.

#### Veritabanı Kesme (Truncation) Kullanma (Using Database Truncation)
`DatabaseTruncation` özelliği, veritabanı tablolarınızın düzgün bir şekilde oluşturulduğundan emin olmak için ilk testte veritabanınızı migrate edecektir. Ancak, sonraki testlerde veritabanının tabloları basitçe kesilecektir (truncated) - bu, tüm veritabanı migration'larınızı yeniden çalıştırmaya kıyasla bir hız artışı sağlar.
```php
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

pest()->use(DatabaseTruncation::class);

//
```
```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseTruncation;

    //
}
```
Varsayılan olarak bu özellik, `migrations` tablosu hariç tüm tabloları kesecektir. Kesilmesi gereken tabloları özelleştirmek isterseniz, test sınıfınızda bir `$tablesToTruncate` özelliği tanımlayabilirsiniz:

Pest kullanıyorsanız, özellikleri veya metotları temel `DuskTestCase` sınıfında veya test dosyanızın genişlettiği herhangi bir sınıfta tanımlamalısınız.
```php
/**
 * Hangi tabloların kesilmesi gerektiğini belirtir.
 *
 * @var array
 */
protected $tablesToTruncate = ['users'];
```
Alternatif olarak, kesme işleminden hariç tutulması gereken tabloları belirtmek için test sınıfınızda bir `$exceptTables` özelliği tanımlayabilirsiniz:
```php
/**
 * Hangi tabloların kesme işleminden hariç tutulması gerektiğini belirtir.
 *
 * @var array
 */
protected $exceptTables = ['users'];
```
Tabloları kesilmesi gereken veritabanı bağlantılarını (connections) belirtmek için test sınıfınızda bir `$connectionsToTruncate` özelliği tanımlayabilirsiniz:
```php
/**
 * Hangi bağlantıların tablolarının kesilmesi gerektiğini belirtir.
 *
 * @var array
 */
protected $connectionsToTruncate = ['mysql'];
```
Veritabanı kesme işlemi gerçekleştirilmeden önce veya sonra kod çalıştırmak isterseniz, test sınıfınızda `beforeTruncatingDatabase` veya `afterTruncatingDatabase` metotlarını tanımlayabilirsiniz:
```php
/**
 * Veritabanı kesilmeye başlamadan önce yapılması gereken işlemleri gerçekleştir.
 */
protected function beforeTruncatingDatabase(): void
{
    //
}

/**
 * Veritabanı kesme işlemi tamamlandıktan sonra yapılması gereken işlemleri gerçekleştir.
 */
protected function afterTruncatingDatabase(): void
{
    //
}
```

### Testleri Çalıştırma (Running Tests)
Tarayıcı testlerinizi çalıştırmak için `dusk` Artisan komutunu çalıştırın:
```bash
php artisan dusk
```
`dusk` komutunu en son çalıştırdığınızda test başarısızlıklarınız olduysa, önce başarısız testleri yeniden çalıştırarak `dusk:fails` komutunu kullanarak zaman kazanabilirsiniz:
```bash
php artisan dusk:fails
```
`dusk` komutu, normalde Pest / PHPUnit test çalıştırıcısı tarafından kabul edilen herhangi bir argümanı kabul eder; örneğin, yalnızca belirli bir gruptaki (group) testleri çalıştırmanıza izin verir:
```bash
php artisan dusk --group=foo
```
Yerel geliştirme ortamınızı yönetmek için Laravel Sail kullanıyorsanız, lütfen Dusk testlerini yapılandırma ve çalıştırma hakkındaki Sail dokümantasyonuna bakın.

#### ChromeDriver'ı Manuel Olarak Başlatma (Manually Starting ChromeDriver)
Varsayılan olarak Dusk, ChromeDriver'ı otomatik olarak başlatmaya çalışacaktır. Bu, belirli sisteminizde çalışmazsa, `dusk` komutunu çalıştırmadan önce ChromeDriver'ı manuel olarak başlatabilirsiniz. ChromeDriver'ı manuel olarak başlatmayı seçerseniz, `tests/DuskTestCase.php` dosyanızdaki aşağıdaki satırı yorum satırı haline getirmelisiniz:
```php
/**
 * Dusk test yürütmesi için hazırlan.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```
Ek olarak, ChromeDriver'ı 9515 dışında bir bağlantı noktasında başlatırsanız, aynı sınıfın `driver` metodunu doğru bağlantı noktasını yansıtacak şekilde değiştirmelisiniz:
```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * RemoteWebDriver örneğini oluştur.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:9515', DesiredCapabilities::chrome()
    );
}
```

### Ortam Yönetimi (Environment Handling)
Dusk'a testleri çalıştırırken kendi ortam dosyasını kullanmasını zorlamak için projenizin kökünde bir `.env.dusk.{environment}` dosyası oluşturun. Örneğin, `dusk` komutunu yerel ortamınızdan başlatacaksanız, bir `.env.dusk.local` dosyası oluşturmalısınız.

Testleri çalıştırırken Dusk, `.env` dosyanızı yedekleyecek ve Dusk ortamınızı `.env` olarak yeniden adlandıracaktır. Testler tamamlandıktan sonra, `.env` dosyanız geri yüklenecektir.

## Tarayıcı Temelleri (Browser Basics)

### Tarayıcı Oluşturma (Creating Browsers)
Başlamak için, uygulamamıza giriş yapabildiğimizi doğrulayan bir test yazalım. Bir test oluşturduktan sonra, onu giriş sayfasına gitmek, bazı kimlik bilgileri girmek ve "Giriş Yap" düğmesine tıklamak için değiştirebiliriz. Bir tarayıcı örneği oluşturmak için Dusk testiniz içinde `browse` metodunu çağırabilirsiniz:
```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

test('temel örnek', function () {
    $user = User::factory()->create([
        'email' => 'ali@example.com',
    ]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Giriş Yap')
            ->assertPathIs('/home');
    });
});
```
```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    /**
     * Temel bir tarayıcı test örneği.
     */
    public function test_basic_example(): void
    {
        $user = User::factory()->create([
            'email' => 'ali@example.com',
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'password')
                ->press('Giriş Yap')
                ->assertPathIs('/home');
        });
    }
}
```
Yukarıdaki örnekte görebileceğiniz gibi, `browse` metodu bir closure kabul eder. Bu closure'a Dusk tarafından otomatik olarak bir tarayıcı örneği iletilecektir ve bu, uygulamanızla etkileşim kurmak ve uygulamanız hakkında iddialarda bulunmak için kullanılan ana nesnedir.

#### Birden Çok Tarayıcı Oluşturma (Creating Multiple Browsers)
Bazen bir testi düzgün bir şekilde gerçekleştirmek için birden çok tarayıcıya ihtiyacınız olabilir. Örneğin, websockets ile etkileşime giren bir sohbet ekranını test etmek için birden çok tarayıcı gerekebilir. Birden çok tarayıcı oluşturmak için, `browse` metoduna verilen closure'ın imzasına daha fazla tarayıcı argümanı eklemeniz yeterlidir:
```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))
        ->visit('/home')
        ->waitForText('Mesaj');

    $second->loginAs(User::find(2))
        ->visit('/home')
        ->waitForText('Mesaj')
        ->type('message', 'Merhaba Ali')
        ->press('Gönder');

    $first->waitForText('Merhaba Ali')
        ->assertSee('Veli Kaya');
});
```

### Gezinme (Navigation)
Uygulamanızda belirli bir URI'ye gitmek için `visit` metodu kullanılabilir:
```php
$browser->visit('/login');
```
Adlandırılmış bir rotaya (named route) gitmek için `visitRoute` metodunu kullanabilirsiniz:
```php
$browser->visitRoute($routeName, $parameters);
```
`back` ve `forward` metotlarını kullanarak "geri" ve "ileri" gidebilirsiniz:
```php
$browser->back();

$browser->forward();
```
Sayfayı yenilemek için `refresh` metodunu kullanabilirsiniz:
```php
$browser->refresh();
```

### Tarayıcı Pencerelerini Yeniden Boyutlandırma (Resizing Browser Windows)
Tarayıcı penceresinin boyutunu ayarlamak için `resize` metodunu kullanabilirsiniz:
```php
$browser->resize(1920, 1080);
```
Tarayıcı penceresini büyütmek (maximize) için `maximize` metodu kullanılabilir:
```php
$browser->maximize();
```
`fitContent` metodu, tarayıcı penceresini içeriğinin boyutuna uyacak şekilde yeniden boyutlandıracaktır:
```php
$browser->fitContent();
```
Bir test başarısız olduğunda Dusk, ekran görüntüsü (screenshot) almadan önce içeriğe sığması için tarayıcıyı otomatik olarak yeniden boyutlandıracaktır. Bu özelliği, testinizde `disableFitOnFailure` metodunu çağırarak devre dışı bırakabilirsiniz:
```php
$browser->disableFitOnFailure();
```
Tarayıcı penceresini ekranınızda farklı bir konuma taşımak için `move` metodunu kullanabilirsiniz:
```php
$browser->move($x = 100, $y = 100);
```

### Tarayıcı Makroları (Browser Macros)
Çeşitli testlerinizde yeniden kullanabileceğiniz özel bir tarayıcı metodu tanımlamak isterseniz, `Browser` sınıfında `macro` metodunu kullanabilirsiniz. Tipik olarak, bu metodu bir servis sağlayıcının `boot` metodundan çağırmalısınız:
```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Dusk'un tarayıcı makrolarını kaydet.
     */
    public function boot(): void
    {
        Browser::macro('scrollToElement', function (string $element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```
`macro` fonksiyonu, ilk argüman olarak bir ad ve ikinci argüman olarak bir closure kabul eder. Makronun closure'ı, makro bir `Browser` örneği üzerinde bir metot olarak çağrıldığında yürütülecektir:
```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
        ->scrollToElement('#credit-card-details')
        ->assertSee('Kredi Kartı Bilgilerini Girin');
});
```

### Kimlik Doğrulama (Authentication)
Genellikle, uygulamanızın kimlik gerektiren sayfalarını test ediyor olacaksınız. `loginAs` metodunu kullanarak, testinizin her senaryosunda giriş yapma sayfasıyla etkileşime girmek zorunda kalmadan bir kullanıcının kimliğini doğrulayabilirsiniz. `loginAs` metodu, birincil anahtarı (primary key) veya bir model örneği ile ilişkili bir kullanıcıyı kabul eder:
```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
          ->visit('/home');
});
```
Bir kullanıcının kimliğini doğruladıktan sonra, uygulamanızın diğer sayfalarında gezinmek için tarayıcıyı kullanmaya devam edebilirsiniz.