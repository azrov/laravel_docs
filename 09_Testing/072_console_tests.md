# Laravel 12 Dokümantasyonu: Konsol Testleri (Console Tests)

## Giriş

Laravel, HTTP testlerini basitleştirmenin yanı sıra, uygulamanızın özel konsol komutlarını (custom console commands) test etmek için basit bir API de sağlar.

## Başarı / Başarısızlık Beklentileri (Success / Failure Expectations)

Başlamak için, bir Artisan komutunun çıkış kodu (exit code) hakkında nasıl iddialarda (assertions) bulunacağımızı keşfedelim. Bunu başarmak için, testimizden bir Artisan komutunu çağırmak üzere `artisan` metodunu kullanacağız. Ardından, komutun belirli bir çıkış koduyla tamamlandığını iddia etmek için `assertExitCode` metodunu kullanacağız:
```php
test('konsol komutu', function () {
    $this->artisan('inspire')->assertExitCode(0);
});
```
```php
/**
 * Bir konsol komutunu test et.
 */
public function test_console_command(): void
{
    $this->artisan('inspire')->assertExitCode(0);
}
```
Komutun belirli bir çıkış koduyla çıkmadığını iddia etmek için `assertNotExitCode` metodunu kullanabilirsiniz:
```php
$this->artisan('inspire')->assertNotExitCode(1);
```
Elbette, tüm terminal komutları başarılı olduklarında tipik olarak `0` durum koduyla, başarısız olduklarında ise sıfır olmayan bir çıkış koduyla çıkarlar. Bu nedenle, kolaylık olması açısından, belirli bir komutun başarılı bir çıkış koduyla çıkıp çıkmadığını veya başarısız olup olmadığını iddia etmek için `assertSuccessful` ve `assertFailed` iddialarını kullanabilirsiniz:
```php
$this->artisan('inspire')->assertSuccessful();

$this->artisan('inspire')->assertFailed();
```

## Girdi / Çıktı Beklentileri (Input / Output Expectations)

Laravel, `expectsQuestion` metodunu kullanarak konsol komutlarınız için kullanıcı girdisini kolayca "mock"lamanıza (taklit etmenize) olanak tanır. Ek olarak, `assertExitCode` ve `expectsOutput` metotlarını kullanarak konsol komutu tarafından çıktı olarak verilmesini beklediğiniz metni ve çıkış kodunu belirtebilirsiniz. Örneğin, aşağıdaki konsol komutunu ele alalım:
```php
Artisan::command('question', function () {
    $name = $this->ask('Adınız nedir?');

    $language = $this->choice('Hangi dili tercih edersiniz?', [
        'PHP',
        'Ruby',
        'Python',
    ]);

    $this->line('Adınız '.$name.' ve tercih ettiğiniz dil '.$language.'.');
});
```
Bu komutu aşağıdaki testle test edebilirsiniz:
```php
test('konsol komutu', function () {
    $this->artisan('question')
        ->expectsQuestion('Adınız nedir?', 'Ali Yılmaz')
        ->expectsQuestion('Hangi dili tercih edersiniz?', 'PHP')
        ->expectsOutput('Adınız Ali Yılmaz ve tercih ettiğiniz dil PHP.')
        ->doesntExpectOutput('Adınız Ali Yılmaz ve tercih ettiğiniz dil Ruby.')
        ->assertExitCode(0);
});
```
```php
/**
 * Bir konsol komutunu test et.
 */
public function test_console_command(): void
{
    $this->artisan('question')
        ->expectsQuestion('Adınız nedir?', 'Ali Yılmaz')
        ->expectsQuestion('Hangi dili tercih edersiniz?', 'PHP')
        ->expectsOutput('Adınız Ali Yılmaz ve tercih ettiğiniz dil PHP.')
        ->doesntExpectOutput('Adınız Ali Yılmaz ve tercih ettiğiniz dil Ruby.')
        ->assertExitCode(0);
}
```
Laravel Prompts tarafından sağlanan `search` veya `multisearch` fonksiyonlarını kullanıyorsanız, kullanıcının girdisini, arama sonuçlarını ve seçimini taklit etmek için `expectsSearch` iddiasını kullanabilirsiniz:
```php
test('konsol komutu', function () {
    $this->artisan('example')
        ->expectsSearch('Adınız nedir?', search: 'Ali', answers: [
            'Ali Yılmaz',
            'Ali Demir',
            'Veli Kaya'
        ], answer: 'Ali Yılmaz')
        ->assertExitCode(0);
});
```
```php
/**
 * Bir konsol komutunu test et.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsSearch('Adınız nedir?', search: 'Ali', answers: [
            'Ali Yılmaz',
            'Ali Demir',
            'Veli Kaya'
        ], answer: 'Ali Yılmaz')
        ->assertExitCode(0);
}
```
Ayrıca, bir konsol komutunun herhangi bir çıktı üretmediğini `doesntExpectOutput` metodunu kullanarak iddia edebilirsiniz:
```php
test('konsol komutu', function () {
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
});
```
```php
/**
 * Bir konsol komutunu test et.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
}
```
`expectsOutputToContain` ve `doesntExpectOutputToContain` metotları, çıktının bir kısmı hakkında iddialarda bulunmak için kullanılabilir:
```php
test('konsol komutu', function () {
    $this->artisan('example')
        ->expectsOutputToContain('Ali')
        ->assertExitCode(0);
});
```
```php
/**
 * Bir konsol komutunu test et.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsOutputToContain('Ali')
        ->assertExitCode(0);
}
```

#### Onay (Confirmation) Beklentileri
"Evet" veya "hayır" şeklinde bir yanıt bekleyen bir onaylama içeren bir komut yazarken, `expectsConfirmation` metodunu kullanabilirsiniz:
```php
$this->artisan('module:import')
    ->expectsConfirmation('Bu komutu gerçekten çalıştırmak istiyor musunuz?', 'hayır')
    ->assertExitCode(1);
```

#### Tablo (Table) Beklentileri
Komutunuz, Artisan'ın `table` metodunu kullanarak bir bilgi tablosu görüntülüyorsa, tüm tablo için çıktı beklentileri yazmak zahmetli olabilir. Bunun yerine, `expectsTable` metodunu kullanabilirsiniz. Bu metod, ilk argüman olarak tablonun başlıklarını (headers) ve ikinci argüman olarak tablonun verilerini kabul eder:
```php
$this->artisan('users:all')
    ->expectsTable([
        'ID',
        'E-posta',
    ], [
        [1, 'ali@example.com'],
        [2, 'veli@example.com'],
    ]);
```

## Konsol Olayları (Console Events)

Varsayılan olarak, `Illuminate\Console\Events\CommandStarting` ve `Illuminate\Console\Events\CommandFinished` olayları, uygulamanızın testlerini çalıştırırken gönderilmez (dispatched). Ancak, sınıfa `Illuminate\Foundation\Testing\WithConsoleEvents` özelliğini (trait) ekleyerek bu olayları belirli bir test sınıfı için etkinleştirebilirsiniz:
```php
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

pest()->use(WithConsoleEvents::class);

// ...
```
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithConsoleEvents;
use Tests\TestCase;

class ConsoleEventTest extends TestCase
{
    use WithConsoleEvents;

    // ...
}
```