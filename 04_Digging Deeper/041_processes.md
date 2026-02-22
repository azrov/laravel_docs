# Laravel 12 Dokümantasyonu: Süreçler (Processes)

## Giriş

Laravel, Symfony Process bileşeni etrafında anlamlı, minimal bir API sağlayarak Laravel uygulamanızdan harici süreçleri (external processes) kolayca çağırmanıza olanak tanır. Laravel'in süreç özellikleri, en yaygın kullanım durumlarına ve harika bir geliştirici deneyimine odaklanmıştır.

## Süreçleri Çağırma (Invoking Processes)

Bir süreci çağırmak için `Process` facade'ı tarafından sunulan `run` ve `start` metotlarını kullanabilirsiniz. `run` metodu bir süreci çağıracak ve sürecin yürütmeyi bitirmesini bekleyecek, `start` metodu ise asenkron (asynchronous) süreç yürütme için kullanılır. Bu dokümantasyonda her iki yaklaşımı da inceleyeceğiz. İlk olarak, temel bir senkron (synchronous) sürecin nasıl çağrılacağını ve sonucunun nasıl inceleneceğini görelim:
```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```
Elbette, `run` metodu tarafından döndürülen `Illuminate\Contracts\Process\ProcessResult` örneği, süreç sonucunu incelemek için kullanılabilecek çeşitli yararlı metotlar sunar:
```php
$result = Process::run('ls -la');

$result->command();
$result->successful();
$result->failed();
$result->output();
$result->errorOutput();
$result->exitCode();
```

#### İstisna Fırlatma (Throwing Exceptions)
Bir süreç sonucunuz varsa ve çıkış kodu (exit code) sıfırdan büyükse (bu başarısızlığı gösterir) bir `Illuminate\Process\Exceptions\ProcessFailedException` örneği fırlatmak isterseniz, `throw` ve `throwIf` metotlarını kullanabilirsiniz. Süreç başarısız olmadıysa, `ProcessResult` örneği döndürülecektir:
```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

### Süreç Seçenekleri (Process Options)
Elbette, bir süreci çağırmadan önce davranışını özelleştirmeniz gerekebilir. Neyse ki Laravel, çalışma dizini (working directory), zaman aşımı (timeout) ve ortam değişkenleri (environment variables) gibi çeşitli süreç özelliklerini ayarlamanıza izin verir.

#### Çalışma Dizini Yolu (Working Directory Path)
Sürecin çalışma dizinini belirtmek için `path` metodunu kullanabilirsiniz. Bu metod çağrılmazsa, süreç mevcut çalışan PHP betiğinin çalışma dizinini devralacaktır:
```php
$result = Process::path(__DIR__)->run('ls -la');
```

#### Girdi (Input)
`input` metodunu kullanarak sürecin "standart girdisi" (standard input) aracılığıyla girdi sağlayabilirsiniz:
```php
$result = Process::input('Hello World')->run('cat');
```

#### Zaman Aşımları (Timeouts)
Varsayılan olarak, süreçler 60 saniyeden daha uzun süre yürütüldükten sonra bir `Illuminate\Process\Exceptions\ProcessTimedOutException` örneği fırlatacaktır. Ancak, bu davranışı `timeout` metodu aracılığıyla özelleştirebilirsiniz:
```php
$result = Process::timeout(120)->run('bash import.sh');
```
Veya, süreç zaman aşımını tamamen devre dışı bırakmak isterseniz, `forever` metodunu çağırabilirsiniz:
```php
$result = Process::forever()->run('bash import.sh');
```
`idleTimeout` metodu, sürecin herhangi bir çıktı vermeden çalışabileceği maksimum saniye sayısını belirtmek için kullanılabilir:
```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

#### Ortam Değişkenleri (Environment Variables)
Sürece ortam değişkenleri `env` metodu aracılığıyla sağlanabilir. Çağrılan süreç ayrıca sisteminiz tarafından tanımlanan tüm ortam değişkenlerini de devralacaktır:
```php
$result = Process::forever()
    ->env(['IMPORT_PATH' => __DIR__])
    ->run('bash import.sh');
```
Devralınan bir ortam değişkenini çağrılan süreçten kaldırmak isterseniz, bu ortam değişkenini `false` değeriyle sağlayabilirsiniz:
```php
$result = Process::forever()
    ->env(['LOAD_PATH' => false])
    ->run('bash import.sh');
```

#### TTY Modu (TTY Mode)
`tty` metodu, süreciniz için TTY modunu etkinleştirmek için kullanılabilir. TTY modu, sürecin girdisini ve çıktısını programınızın girdisine ve çıktısına bağlayarak sürecinizin bir süreç olarak Vim veya Nano gibi bir düzenleyici açmasına izin verir:
```php
Process::forever()->tty()->run('vim');
```
TTY modu Windows'ta desteklenmez.

### Süreç Çıktısı (Process Output)
Daha önce tartışıldığı gibi, süreç çıktısına bir süreç sonucu üzerindeki `output` (stdout) ve `errorOutput` (stderr) metotları kullanılarak erişilebilir:
```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```
Ancak, çıktı ayrıca `run` metoduna ikinci argüman olarak bir closure ileterek gerçek zamanlı olarak da toplanabilir. Closure, iki argüman alacaktır: çıktının "türü" (`stdout` veya `stderr`) ve çıktı dizesinin kendisi:
```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```
Laravel ayrıca, belirli bir dizenin sürecin çıktısında bulunup bulunmadığını belirlemenin uygun bir yolunu sağlayan `seeInOutput` ve `seeInErrorOutput` metotlarını da sunar:
```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

#### Süreç Çıktısını Devre Dışı Bırakma (Disabling Process Output)
Süreciniz, ilgilenmediğiniz önemli miktarda çıktı yazıyorsa, çıktı almayı tamamen devre dışı bırakarak bellekten tasarruf edebilirsiniz. Bunu başarmak için süreci oluştururken `quietly` metodunu çağırın:
```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

### Ardışık Düzenler (Pipelines)
Bazen bir sürecin çıktısını başka bir sürecin girdisi yapmak isteyebilirsiniz. Buna genellikle bir sürecin çıktısını diğerine "aktarmak" (piping) denir. `Process` facade'ı tarafından sağlanan `pipe` metodu bunu kolayca yapmayı sağlar. `pipe` metodu, ardışık düzenlenmiş süreçleri senkron olarak yürütecek ve ardışık düzendeki son süreç için süreç sonucunu döndürecektir:
```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```
Ardışık düzeni oluşturan bireysel süreçleri özelleştirmeniz gerekmiyorsa, `pipe` metoduna basitçe bir komut dizeleri dizisi iletebilirsiniz:
```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```
Süreç çıktısı, `pipe` metoduna ikinci argüman olarak bir closure iletilerek gerçek zamanlı olarak toplanabilir. Closure, iki argüman alacaktır: çıktının "türü" (`stdout` veya `stderr`) ve çıktı dizesinin kendisi:
```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```
Laravel ayrıca, `as` metodu aracılığıyla bir ardışık düzen içindeki her bir sürece dize anahtarları atamanıza izin verir. Bu anahtar, `pipe` metoduna sağlanan çıktı closure'ına da iletilecek ve çıktının hangi sürece ait olduğunu belirlemenize olanak tanıyacaktır:
```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
}, function (string $type, string $output, string $key) {
    // ...
});
```

## Asenkron Süreçler (Asynchronous Processes)

`run` metodu süreçleri senkron olarak çağırırken, `start` metodu bir süreci asenkron olarak çağırmak için kullanılabilir. Bu, süreç arka planda çalışırken uygulamanızın diğer görevleri gerçekleştirmeye devam etmesine olanak tanır. Süreç çağrıldıktan sonra, sürecin hala çalışıp çalışmadığını belirlemek için `running` metodunu kullanabilirsiniz:
```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```
Fark etmiş olabileceğiniz gibi, sürecin yürütmeyi bitirmesini beklemek ve `ProcessResult` örneğini almak için `wait` metodunu çağırabilirsiniz:
```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

### Süreç ID'leri ve Sinyalleri (Process IDs and Signals)
`id` metodu, çalışan sürecin işletim sistemi tarafından atanmış süreç kimliğini (process ID) almak için kullanılabilir:
```php
$process = Process::start('bash import.sh');

return $process->id();
```
Çalışan sürece bir "sinyal" (signal) göndermek için `signal` metodunu kullanabilirsiniz. Önceden tanımlanmış sinyal sabitlerinin (predefined signal constants) bir listesi PHP dokümantasyonunda bulunabilir:
```php
$process->signal(SIGUSR2);
```

### Asenkron Süreç Çıktısı (Asynchronous Process Output)
Bir asenkron süreç çalışırken, mevcut tüm çıktısına `output` ve `errorOutput` metotlarını kullanarak erişebilirsiniz; ancak, çıktının en son alındığı andan itibaren süreçten gelen çıktıya erişmek için `latestOutput` ve `latestErrorOutput`'u kullanabilirsiniz:
```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```
`run` metodu gibi, `start` metoduna ikinci argüman olarak bir closure ileterek asenkron süreçlerden de çıktı gerçek zamanlı olarak toplanabilir. Closure, iki argüman alacaktır: çıktının "türü" (`stdout` veya `stderr`) ve çıktı dizesinin kendisi:
```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```
Süreç bitene kadar beklemek yerine, sürecin çıktısına bağlı olarak beklemeyi durdurmak için `waitUntil` metodunu kullanabilirsiniz. Laravel, `waitUntil` metoduna verilen closure `true` döndürdüğünde sürecin bitmesini beklemeyi durduracaktır:
```php
$process = Process::start('bash import.sh');

$process->waitUntil(function (string $type, string $output) {
    return $output === 'Ready...';
});
```

### Asenkron Süreç Zaman Aşımları (Asynchronous Process Timeouts)
Bir asenkron süreç çalışırken, `ensureNotTimedOut` metodunu kullanarak sürecin zaman aşımına uğramadığını doğrulayabilirsiniz. Bu metod, süreç zaman aşımına uğradıysa bir zaman aşımı istisnası (timeout exception) fırlatacaktır:
```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    $process->ensureNotTimedOut();

    // ...

    sleep(1);
}
```

## Eşzamanlı Süreçler (Concurrent Processes)

Laravel ayrıca bir dizi eşzamanlı, asenkron süreci yönetmeyi de çok kolaylaştırarak aynı anda birçok görevi kolayca yürütmenize olanak tanır. Başlamak için, bir `Illuminate\Process\Pool` örneği alan bir closure kabul eden `pool` metodunu çağırın.

Bu closure içinde, havuza (pool) ait süreçleri tanımlayabilirsiniz. Bir süreç havuzu `start` metodu aracılığıyla başlatıldıktan sonra, `running` metodu aracılığıyla çalışan süreçlerin koleksiyonuna erişebilirsiniz:
```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```
Gördüğünüz gibi, havuzdaki tüm süreçlerin yürütmeyi bitirmesini bekleyebilir ve sonuçlarını `wait` metodu aracılığıyla çözümleyebilirsiniz. `wait` metodu, havuzdaki her bir sürecin `ProcessResult` örneğine anahtarıyla erişmenizi sağlayan, dizi olarak erişilebilen (array accessible) bir nesne döndürür:
```php
$results = $pool->wait();

echo $results[0]->output();
```
Veya, kolaylık olması açısından, `concurrently` metodu bir asenkron süreç havuzu başlatmak ve hemen sonuçlarını beklemek için kullanılabilir. Bu, özellikle PHP'nin dizi yapıbozum (array destructuring) yetenekleriyle birleştirildiğinde oldukça anlamlı bir sözdizimi sağlayabilir:
```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

### Havuz Süreçlerini Adlandırma (Naming Pool Processes)
Süreç havuzu sonuçlarına sayısal bir anahtarla erişmek pek anlamlı değildir; bu nedenle Laravel, `as` metodu aracılığıyla bir havuz içindeki her bir sürece dize anahtarları atamanıza izin verir. Bu anahtar, `start` metoduna sağlanan closure'a da iletilecek ve çıktının hangi sürece ait olduğunu belirlemenize olanak tanıyacaktır:
```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

### Havuz Süreç ID'leri ve Sinyalleri (Pool Process IDs and Signals)
Süreç havuzunun `running` metodu, havuz içindeki tüm çağrılmış süreçlerin bir koleksiyonunu sağladığından, temeldeki havuz süreç ID'lerine kolayca erişebilirsiniz:
```php
$processIds = $pool->running()->each->id();
```
Ve kolaylık olması açısından, bir süreç havuzundaki her bir sürece bir sinyal göndermek için `signal` metodunu çağırabilirsiniz:
```php
$pool->signal(SIGUSR2);
```

## Test Etme (Testing)

Birçok Laravel servisi, kolayca ve anlamlı bir şekilde testler yazmanıza yardımcı olacak işlevsellik sağlar ve Laravel'in süreç servisi de bir istisna değildir. `Process` facade'ının `fake` metodu, süreçler çağrıldığında Laravel'e sahte / hazır (stubbed/dummy) sonuçlar döndürmesi talimatını vermenizi sağlar.

### Süreçleri Sahteleme (Faking Processes)
Laravel'in süreçleri sahteleme yeteneğini keşfetmek için, bir süreç çağıran bir rota hayal edelim:
```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```
Bu rotayı test ederken, `Process` facade'ında `fake` metodunu argümansız çağırarak Laravel'e çağrılan her süreç için sahte, başarılı bir süreç sonucu döndürmesi talimatını verebiliriz. Ayrıca, belirli bir sürecin "çalıştırıldığını" (run) doğrulayabiliriz:
```php
<?php

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;

test('process is invoked', function () {
    Process::fake();

    $response = $this->get('/import');

    // Basit süreç iddiası (assertion)...
    Process::assertRan('bash import.sh');

    // Veya, süreç yapılandırmasını inceleme...
    Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
        return $process->command === 'bash import.sh' &&
               $process->timeout === 60;
    });
});
```
```php
<?php

namespace Tests\Feature;

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // Basit süreç iddiası (assertion)...
        Process::assertRan('bash import.sh');

        // Veya, süreç yapılandırmasını inceleme...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

#### Belirli Süreçleri Sahteleme (Faking Specific Processes)
Önceki örnekte gösterildiği gibi, `Process` facade'ında `fake` metodunu argümansız çağırırsanız, her sürecin sahteleneceğini belirtmiş olursunuz. Ancak, yalnızca belirli bir dizi süreci sahtelemeniz ve diğerlerinin normal şekilde yürütülmesine izin vermeniz gerekebilir.

Bunu başarmak için, her biri sahte bir sonuç döndürmesi gereken süreç komutlarına karşılık gelen anahtarlara sahip bir dizi anahtar/değer çiftini `fake` metoduna iletebilirsiniz. Boş dize (`''`) anahtarı, eşleşmeyen tüm süreçler için varsayılan sahte sonucu belirtmek için kullanılabilir:
```php
Process::fake([
    '*' => Process::result(
        output: 'Successful output',
        errorOutput: 'Error output',
        exitCode: 0,
    ),
]);

Process::fake([
    'cat *' => Process::result(
        output: 'Contents of file...',
    ),
    'bash import.sh' => Process::result(
        output: 'Done',
    ),
]);
```

#### Sahte Süreç Sonuçlarını Özelleştirme (Customizing Fake Process Results)
Laravel'in sahte süreç sonuçlarını test durumlarınız için özelleştirmek isteyebilirsiniz. `result` ve `failed` metotları, sahte süreç sonuçları oluşturmanıza izin verir:
```php
Process::fake([
    '*' => Process::result('Successful output', 'Error output', 0),
    'bash import.sh' => Process::result(output: 'Done', exitCode: 0),
    'cat *' => Process::failed(errorOutput: 'File not found', exitCode: 1),
]);
```