# Laravel 12 Dokümantasyonu: Artisan Konsolu (Artisan Console)

## Giriş

Artisan, Laravel ile birlikte gelen komut satırı arayüzüdür (command line interface - CLI). Artisan, uygulamanızın kökünde `artisan` betiği (script) olarak bulunur ve uygulamanızı oluştururken size yardımcı olabilecek bir dizi yararlı komut sağlar. Mevcut tüm Artisan komutlarının bir listesini görüntülemek için `list` komutunu kullanabilirsiniz:
```bash
php artisan list
```
Her komut ayrıca, komutun mevcut argümanlarını ve seçeneklerini görüntüleyen ve açıklayan bir "yardım" (help) ekranı içerir. Bir yardım ekranını görüntülemek için komutun adından önce `help` yazın:
```bash
php artisan help migrate
```

#### Laravel Sail
Yerel geliştirme ortamınız olarak Laravel Sail kullanıyorsanız, Artisan komutlarını çağırmak için `sail` komut satırını kullanmayı unutmayın. Sail, Artisan komutlarınızı uygulamanızın Docker konteynerleri içinde yürütecektir:
```bash
./vendor/bin/sail artisan list
```

### Tinker (REPL)
Laravel Tinker, PsySH paketi tarafından desteklenen, Laravel framework'ü için güçlü bir REPL'dir (Read-Eval-Print Loop).

#### Kurulum (Installation)
Tüm Laravel uygulamaları varsayılan olarak Tinker'ı içerir. Ancak, daha önce uygulamanızdan kaldırdıysanız, Tinker'ı Composer kullanarak kurabilirsiniz:
```bash
composer require laravel/tinker
```
Laravel uygulamanızla etkileşime girerken sıcak yeniden yükleme (hot reloading), çok satırlı kod düzenleme ve otomatik tamamlama (autocompletion) mı arıyorsunuz? Tinkerwell'e göz atın!

#### Kullanım (Usage)
Tinker, Eloquent modelleriniz, işleriniz (jobs), olaylarınız (events) ve daha fazlası dahil olmak üzere tüm Laravel uygulamanızla komut satırında etkileşime girmenize olanak tanır. Tinker ortamına girmek için `tinker` Artisan komutunu çalıştırın:
```bash
php artisan tinker
```
Tinker'ın yapılandırma dosyasını `vendor:publish` komutunu kullanarak yayınlayabilirsiniz:
```bash
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```
`dispatch` yardımcı fonksiyonu ve `Dispatchable` sınıfındaki `dispatch` metodu, işi kuyruğa (queue) yerleştirmek için çöp toplamaya (garbage collection) bağlıdır. Bu nedenle, Tinker kullanırken işleri göndermek için `Bus::dispatch` veya `Queue::push` kullanmalısınız.

#### Komut İzin Listesi (Command Allow List)
Tinker, kabuğu (shell) içinde hangi Artisan komutlarının çalıştırılmasına izin verileceğini belirlemek için bir "izin listesi" (allow list) kullanır. Varsayılan olarak, `clear-compiled`, `down`, `env`, `inspire`, `migrate`, `migrate:install`, `up` ve `optimize` komutlarını çalıştırabilirsiniz. Daha fazla komuta izin vermek isterseniz, bunları `tinker.php` yapılandırma dosyanızdaki `commands` dizisine ekleyebilirsiniz:
```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

#### Takma Ad (Alias) Verilmemesi Gereken Sınıflar
Tipik olarak Tinker, Tinker'da onlarla etkileşime girdiğinizde sınıflara otomatik olarak takma ad (alias) verir. Ancak, bazı sınıflara asla takma ad verilmesini istemeyebilirsiniz. Bunu, sınıfları `tinker.php` yapılandırma dosyanızdaki `dont_alias` dizisinde listeleyerek gerçekleştirebilirsiniz:
```php
'dont_alias' => [
    App\Models\User::class,
],
```

## Komut Yazma (Writing Commands)

Artisan ile sağlanan komutlara ek olarak, kendi özel komutlarınızı da oluşturabilirsiniz. Komutlar tipik olarak `app/Console/Commands` dizininde saklanır; ancak, Laravel'e Artisan komutları için diğer dizinleri taraması talimatını verdiğiniz sürece kendi depolama konumunuzu seçmekte özgürsünüz.

### Komut Oluşturma (Generating Commands)
Yeni bir komut oluşturmak için `make:command` Artisan komutunu kullanabilirsiniz. Bu komut, `app/Console/Commands` dizininde yeni bir komut sınıfı oluşturacaktır. Bu dizin uygulamanızda mevcut değilse endişelenmeyin - `make:command` Artisan komutunu ilk çalıştırdığınızda oluşturulacaktır:
```bash
php artisan make:command SendEmails
```

### Komut Yapısı (Command Structure)
Komutunuzu oluşturduktan sonra, sınıfın `signature` ve `description` özellikleri için uygun değerleri tanımlamalısınız. Bu özellikler, komutunuz `list` ekranında görüntülendiğinde kullanılacaktır. `signature` özelliği ayrıca komutunuzun girdi beklentilerini (input expectations) tanımlamanıza da olanak tanır. Komutunuz yürütüldüğünde `handle` metodu çağrılacaktır. Komut mantığınızı bu metoda yerleştirebilirsiniz.

Örnek bir komuta bakalım. Komutun `handle` metodu aracılığıyla ihtiyacımız olan herhangi bir bağımlılığı isteyebildiğimize dikkat edin. Laravel servis kabı (service container), bu metodun imzasında tip belirtilen (type-hinted) tüm bağımlılıkları otomatik olarak enjekte edecektir:
```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * Konsol komutunun adı ve imzası.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    /**
     * Konsol komutunun açıklaması.
     *
     * @var string
     */
    protected $description = 'Bir kullanıcıya pazarlama e-postası gönder';

    /**
     * Konsol komutunu yürüt.
     */
    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```
Daha fazla kod tekrarı için, konsol komutlarınızı hafif tutmak ve görevlerini yerine getirmek için bunların uygulama servislerine (application services) havale etmesine izin vermek iyi bir uygulamadır. Yukarıdaki örnekte, e-postaları gönderme gibi "ağır işi" yapmak için bir servis sınıfı enjekte ettiğimize dikkat edin.

#### Çıkış Kodları (Exit Codes)
`handle` metodundan hiçbir şey döndürülmezse ve komut başarıyla yürütülürse, komut başarıyı gösteren `0` çıkış koduyla sona erecektir. Ancak, `handle` metodu isteğe bağlı olarak komutun çıkış kodunu manuel olarak belirtmek için bir tamsayı döndürebilir:
```php
$this->error('Bir şeyler yanlış gitti.');

return 1;
```
Komut içindeki herhangi bir metottan komutu "başarısız" kılmak (fail) isterseniz, `fail` metodunu kullanabilirsiniz. `fail` metodu, komutun yürütülmesini hemen sonlandıracak ve `1` çıkış kodu döndürecektir:
```php
$this->fail('Bir şeyler yanlış gitti.');
```

### Kapatma (Closure) Komutları
Kapatma tabanlı komutlar (closure-based commands), konsol komutlarını sınıf olarak tanımlamaya bir alternatif sağlar. Rota kapatmalarının (route closures) controller'lara bir alternatif olması gibi, komut kapatmalarını da komut sınıflarına bir alternatif olarak düşünün.

`routes/console.php` dosyası HTTP rotalarını tanımlamasa da, uygulamanıza konsol tabanlı giriş noktaları (routes) tanımlar. Bu dosya içinde, `Artisan::command` metodunu kullanarak tüm kapatma tabanlı konsol komutlarınızı tanımlayabilirsiniz. `command` metodu iki argüman kabul eder: komut imzası (command signature) ve komutun argümanlarını ve seçeneklerini alan bir kapatma (closure):
```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("E-posta gönderiliyor: {$user}!");
});
```
Kapatma, temeldeki komut örneğine (underlying command instance) bağlıdır, bu nedenle tam bir komut sınıfında normalde erişebileceğiniz tüm yardımcı metotlara tam erişime sahip olursunuz.

#### Bağımlılıkları Tip Belirtme (Type-Hinting Dependencies)
Komutunuzun argümanlarını ve seçeneklerini almanın yanı sıra, komut kapatmaları, servis kabından çözümlenmesini (resolved) istediğiniz ek bağımlılıkları da tip belirtebilir (type-hint):
```php
use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Support\Facades\Artisan;

Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
    $drip->send(User::find($user));
});
```

#### Kapatma Komutu Açıklamaları (Closure Command Descriptions)
Kapatma tabanlı bir komut tanımlarken, komuta bir açıklama eklemek için `purpose` metodunu kullanabilirsiniz. Bu açıklama, `php artisan list` veya `php artisan help` komutlarını çalıştırdığınızda görüntülenecektir:
```php
Artisan::command('mail:send {user}', function (string $user) {
    // ...
})->purpose('Bir kullanıcıya pazarlama e-postası gönder');
```

### Yalıtılabilir Komutlar (Isolatable Commands)
Bu özelliği kullanmak için uygulamanızın varsayılan önbellek sürücüsü olarak `memcached`, `redis`, `dynamodb`, `database`, `file` veya `array` önbellek sürücüsünü kullanıyor olması gerekir. Ayrıca, tüm sunucular aynı merkezi önbellek sunucusuyla iletişim kuruyor olmalıdır.

Bazen bir komutun aynı anda yalnızca bir örneğinin (instance) çalıştığından emin olmak isteyebilirsiniz. Bunu başarmak için komut sınıfınızda `Illuminate\Contracts\Console\Isolatable` arayüzünü (interface) uygulayabilirsiniz (implement):
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // ...
```
Bir komutu `Isolatable` olarak işaretlediğinizde, Laravel, komutun seçeneklerinde açıkça tanımlamaya gerek kalmadan, komut için otomatik olarak `--isolated` seçeneğini kullanılabilir hale getirir. Komut bu seçenekle çağrıldığında, Laravel o komutun başka hiçbir örneğinin zaten çalışmadığından emin olacaktır. Laravel bunu, uygulamanızın varsayılan önbellek sürücüsünü kullanarak atomik bir kilit (atomic lock) edinmeye çalışarak gerçekleştirir. Komutun başka örnekleri çalışıyorsa, komut yürütülmeyecektir; ancak, komut yine de başarılı bir çıkış durum koduyla sona erecektir:
```bash
php artisan mail:send 1 --isolated
```
Komutun yürütülemezse döndürmesi gereken çıkış durum kodunu (exit status code) belirtmek isterseniz, istenen durum kodunu `isolated` seçeneği aracılığıyla sağlayabilirsiniz:
```bash
php artisan mail:send 1 --isolated=12
```

#### Kilit Kimliği (Lock ID)
Varsayılan olarak Laravel, uygulamanızın önbelleğinde atomik kilidi edinmek için kullanılan dize anahtarını (string key) oluşturmak için komutun adını kullanır. Ancak, Artisan komut sınıfınızda bir `isolatableId` metodu tanımlayarak bu anahtarı özelleştirebilir ve komutun argümanlarını veya seçeneklerini anahtara entegre etmenize olanak tanıyabilirsiniz:
```php
/**
 * Komut için yalıtılabilir (isolatable) ID'yi al.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

#### Kilit Son Kullanma Süresi (Lock Expiration Time)
Varsayılan olarak, yalıtım kilitleri (isolation locks) komut bittikten sonra sona erer. Veya komut kesintiye uğrarsa ve bitirilemezse, kilit bir saat sonra sona erecektir. Ancak, komutunuzda bir `isolationLockExpiresAt` metodu tanımlayarak kilit son kullanma süresini ayarlayabilirsiniz:
```php
use DateTimeInterface;
use DateInterval;

/**
 * Komut için bir yalıtım kilidinin (isolation lock) ne zaman sona ereceğini belirle.
 */
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->addMinutes(5);
}
```

## Girdi Beklentilerini Tanımlama (Defining Input Expectations)

Konsol komutları yazarken, kullanıcıdan argümanlar veya seçenekler aracılığıyla girdi toplamak yaygındır. Laravel, komutlarınızdaki `signature` özelliğini kullanarak kullanıcıdan beklediğiniz girdiyi tanımlamayı çok kolaylaştırır. `signature` özelliği, komutun adını, argümanlarını ve seçeneklerini tek, anlamlı, rota benzeri bir sözdiziminde tanımlamanıza olanak tanır.

### Argümanlar (Arguments)
Kullanıcı tarafından sağlanan tüm argümanlar ve seçenekler kaşlı ayraçlar (curly braces) içine alınır. Aşağıdaki örnekte, komut bir gerekli argüman tanımlar: `user`:
```php
/**
 * Konsol komutunun adı ve imzası.
 *
 * @var string
 */
protected $signature = 'mail:send {user}';
```
Ayrıca argümanları isteğe bağlı (optional) hale getirebilir veya argümanlar için varsayılan değerler tanımlayabilirsiniz:
```php
// İsteğe bağlı argüman...
'mail:send {user?}'

// Varsayılan değere sahip isteğe bağlı argüman...
'mail:send {user=foo}'
```

### Seçenekler (Options)
Seçenekler, argümanlar gibi, kullanıcı girdisinin başka bir biçimidir. Seçenekler, komut satırı aracılığıyla sağlandıklarında iki kısa çizgi (`--`) ile ön eklenir (prefixed). İki tür seçenek vardır: değer alanlar ve almayanlar. Değer almayan seçenekler, bir boole "anahtarı" (boolean "switch") görevi görür. Bu tür bir seçeneğin bir örneğine bakalım:
```php
/**
 * Konsol komutunun adı ve imzası.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue}';
```
Bu örnekte, `--queue` anahtarı Artisan komutu çağrılırken belirtilebilir. `--queue` anahtarı iletilirse, seçeneğin değeri `true` olacaktır. Aksi takdirde, değer `false` olacaktır:
```bash
php artisan mail:send 1 --queue
```

#### Değer Alan Seçenekler (Options With Values)
Şimdi, bir değer bekleyen bir seçeneğe bakalım. Kullanıcının bir seçenek için bir değer belirtmesi gerekiyorsa, seçenek adının sonuna bir `=` işareti eklemelisiniz:
```php
/**
 * Konsol komutunun adı ve imzası.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue=}';
```
Bu örnekte, kullanıcı seçenek için şu şekilde bir değer iletebilir. Seçenek komut çağrılırken belirtilmezse, değeri `null` olacaktır:
```bash
php artisan mail:send 1 --queue=default
```
Seçenek adından sonra varsayılan değeri belirterek seçeneklere varsayılan değerler atayabilirsiniz. Kullanıcı tarafından hiçbir seçenek değeri iletilmezse, varsayılan değer kullanılacaktır:
```php
'mail:send {user} {--queue=default}'
```

#### Seçenek Kısayolları (Option Shortcuts)
Bir seçeneği tanımlarken bir kısayol (shortcut) atamak için, bunu seçenek adından önce belirtebilir ve kısayolu tam seçenek adından ayırmak için sınırlayıcı (delimiter) olarak `|` karakterini kullanabilirsiniz:
```php
'mail:send {user} {--Q|queue=}'
```
Komutu terminalinizde çağırırken, seçenek kısayolları tek bir kısa çizgi ile ön eklenmelidir ve seçenek için bir değer belirtirken `=` karakteri dahil edilmemelidir:
```bash
php artisan mail:send 1 -Qdefault
```

### Dizi Girdiler (Input Arrays)
Birden çok girdi değeri bekleyen argümanlar veya seçenekler tanımlamak isterseniz, `*` karakterini kullanabilirsiniz. İlk olarak, böyle bir argümanı belirten bir örneğe bakalım:
```php
'mail:send {user*}'
```
Bu komut çalıştırıldığında, `user` argümanları sırayla komut satırına iletilebilir. Örneğin, aşağıdaki komut `user` değerini, değerleri `1` ve `2` olan bir diziye (array) ayarlayacaktır:
```bash
php artisan mail:send 1 2
```
Bu `*` karakteri, sıfır veya daha fazla argüman örneğine izin vermek için isteğe bağlı bir argüman tanımıyla birleştirilebilir:
```php
'mail:send {user?*}'
```

#### Seçenek Dizileri (Option Arrays)
Birden çok girdi değeri bekleyen bir seçenek tanımlarken, komuta iletilen her seçenek değeri, seçenek adıyla ön eklenmelidir:
```php
'mail:send {--id=*}'
```
Böyle bir komut, birden çok `--id` argümanı ileterek çağrılabilir:
```bash
php artisan mail:send --id=1 --id=2
```

### Girdi Açıklamaları (Input Descriptions)
Girdi argümanlarına ve seçeneklerine, argüman adını açıklamadan iki nokta üst üste (colon) kullanarak ayırarak açıklamalar atayabilirsiniz. Komutunuzu tanımlamak için biraz daha fazla alana ihtiyacınız varsa, tanımı birden çok satıra yaymaktan çekinmeyin:
```php
/**
 * Konsol komutunun adı ve imzası.
 *
 * @var string
 */
protected $signature = 'mail:send
                        {user : Kullanıcının ID\'si}
                        {--queue : İşin sıraya alınıp alınmayacağı}';
```

### Eksik Girdi İçin Soru Sorma (Prompting for Missing Input)
Komutunuz gerekli argümanlar içeriyorsa, kullanıcı onları sağlamadan komutu çalıştırmaya çalıştığında bir hata mesajı alacaktır. Alternatif olarak, eksik argümanlar veya seçenekler için kullanıcıya otomatik olarak soru soracak bir komut yapılandırabilirsiniz. Bunu başarmak için, `{argument?}` gibi isteğe bağlı argümanlar oluşturun ve `promptForMissingArgumentsUsing` metodunu kullanarak Laravel'e kullanıcıya nasıl soru soracağını belirtin. `promptForMissingArgumentsUsing` metodu, argüman adlarını ve sorulacak soruları tanımlayan bir ilişkisel dizi (associative array) kabul eder:
```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

/**
 * Komutun çalıştırılmasını yönet.
 */
public function handle(): void
{
    // ...
}

/**
 * Eksik girdi argümanları için kullanıcıya sor.
 *
 * @return array<string, string>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => 'Hangi kullanıcı ID\'si e-posta gönderilmeli?',
    ];
}
```
Daha karmaşık soru sorma mantığı sağlamanız gerekirse, argüman adına karşılık gelen bir closure da döndürebilirsiniz:
```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

/**
 * Eksik girdi argümanları için kullanıcıya sor.
 *
 * @return array<string, \Closure>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => function () {
            $user = $this->ask('Hangi kullanıcı ID\'si e-posta gönderilmeli?');

            if (! User::find($user)) {
                $this->error('Geçersiz kullanıcı ID\'si.');
                return $this->promptForMissingArgumentsUsing()['user']();
            }

            return $user;
        },
    ];
}
```