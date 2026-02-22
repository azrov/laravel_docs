# Laravel 12 Dokümantasyonu: Görev Zamanlama (Task Scheduling)

## Giriş

Geçmişte, sunucunuzda zamanlamanız gereken her görev için bir cron yapılandırma girdisi yazmanız gerekebilirdi. Ancak bu, görev zamanlamanız artık kaynak kontrolünde (source control) olmadığından ve mevcut cron girdilerinizi görüntülemek veya ek girdiler eklemek için sunucunuza SSH ile bağlanmanız gerektiğinden hızla bir zahmet haline gelebilir.

Laravel'in komut zamanlayıcısı (command scheduler), sunucunuzda zamanlanmış görevleri yönetmek için yeni bir yaklaşım sunar. Zamanlayıcı (scheduler), görev zamanlamanızı doğrudan Laravel uygulamanız içinde akıcı (fluently) ve anlamlı bir şekilde tanımlamanıza olanak tanır. Zamanlayıcıyı kullanırken, sunucunuzda yalnızca tek bir cron girdisi gerekir. Görev zamanlamanız genellikle uygulamanızın `routes/console.php` dosyasında tanımlanır.

## Zamanlamaları Tanımlama (Defining Schedules)

Zamanlanmış tüm görevlerinizi uygulamanızın `routes/console.php` dosyasında tanımlayabilirsiniz. Başlamak için bir örneğe bakalım. Bu örnekte, her gün gece yarısı çağrılacak bir kapatma (closure) zamanlayacağız. Kapatma içinde bir veritabanı sorgusu çalıştırarak bir tabloyu temizleyeceğiz:
```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```
Kapatmaları kullanarak zamanlamanın yanı sıra, çağrılabilir nesneleri (invokable objects) de zamanlayabilirsiniz. Çağrılabilir nesneler, bir `__invoke` metodu içeren basit PHP sınıflarıdır:
```php
Schedule::call(new DeleteRecentUsers)->daily();
```
`routes/console.php` dosyanızı yalnızca komut tanımları için ayırmayı tercih ederseniz, uygulamanızın `bootstrap/app.php` dosyasında `withSchedule` metodunu kullanarak zamanlanmış görevlerinizi tanımlayabilirsiniz. Bu metod, zamanlayıcının bir örneğini alan bir closure kabul eder:
```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```
Zamanlanmış görevlerinizin bir özetini ve bir sonraki çalışma zamanlarını görüntülemek isterseniz, `schedule:list` Artisan komutunu kullanabilirsiniz:
```bash
php artisan schedule:list
```

### Artisan Komutlarını Zamanlama (Scheduling Artisan Commands)
Kapatmaları zamanlamanın yanı sıra, Artisan komutlarını ve sistem komutlarını da zamanlayabilirsiniz. Örneğin, bir Artisan komutunu komutun adını veya sınıfını kullanarak zamanlamak için `command` metodunu kullanabilirsiniz.

Artisan komutlarını komutun sınıf adını kullanarak zamanlarken, komut çağrıldığında sağlanması gereken ek komut satırı argümanlarından oluşan bir dizi iletebilirsiniz:
```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

#### Artisan Kapatma Komutlarını Zamanlama (Scheduling Artisan Closure Commands)
Bir kapatma (closure) tarafından tanımlanan bir Artisan komutunu zamanlamak isterseniz, komutun tanımından sonra zamanlamayla ilgili metotları zincirleyebilirsiniz:
```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Son kullanıcıları sil')->daily();
```
Kapatma komutuna argümanlar iletmeniz gerekiyorsa, bunları `schedule` metoduna sağlayabilirsiniz:
```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Belirtilen kullanıcıya e-postalar gönder')->schedule(['Taylor', '--force'])->daily();
```

### Sıraya Konmuş İşleri Zamanlama (Scheduling Queued Jobs)
`job` metodu, sıraya konmuş bir işi (queued job) zamanlamak için kullanılabilir. Bu metod, işi kuyruğa almak için kapatmalar tanımlamak üzere `call` metodunu kullanmadan sıraya konmuş işleri zamanlamanın kullanışlı bir yolunu sağlar:
```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```
`job` metoduna, işin sıraya konması için kullanılması gereken kuyruk adını ve kuyruk bağlantısını (queue connection) belirten isteğe bağlı ikinci ve üçüncü argümanlar sağlanabilir:
```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// İşi "sqs" bağlantısındaki "heartbeats" kuyruğuna gönder...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

### Kabuk (Shell) Komutlarını Zamanlama (Scheduling Shell Commands)
`exec` metodu, işletim sistemine bir komut göndermek için kullanılabilir:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

### Zamanlama Sıklığı Seçenekleri (Schedule Frequency Options)
Bir görevi belirli aralıklarla çalışacak şekilde nasıl yapılandırabileceğinize dair birkaç örnek gördük. Ancak, bir göreve atayabileceğiniz çok daha fazla görev zamanlama sıklığı vardır:

| Metot | Açıklama |
| :--- | :--- |
| `->cron('* * * * *');` | Görevi özel bir cron zamanlamasında çalıştır. |
| `->everySecond();` | Görevi her saniye çalıştır. |
| `->everyTwoSeconds();` | Görevi her iki saniyede bir çalıştır. |
| `->everyFiveSeconds();` | Görevi her beş saniyede bir çalıştır. |
| `->everyTenSeconds();` | Görevi her on saniyede bir çalıştır. |
| `->everyFifteenSeconds();` | Görevi her on beş saniyede bir çalıştır. |
| `->everyTwentySeconds();` | Görevi her yirmi saniyede bir çalıştır. |
| `->everyThirtySeconds();` | Görevi her otuz saniyede bir çalıştır. |
| `->everyMinute();` | Görevi her dakika çalıştır. |
| `->everyTwoMinutes();` | Görevi her iki dakikada bir çalıştır. |
| `->everyThreeMinutes();` | Görevi her üç dakikada bir çalıştır. |
| `->everyFourMinutes();` | Görevi her dört dakikada bir çalıştır. |
| `->everyFiveMinutes();` | Görevi her beş dakikada bir çalıştır. |
| `->everyTenMinutes();` | Görevi her on dakikada bir çalıştır. |
| `->everyFifteenMinutes();` | Görevi her on beş dakikada bir çalıştır. |
| `->everyThirtyMinutes();` | Görevi her otuz dakikada bir çalıştır. |
| `->hourly();` | Görevi her saat çalıştır. |
| `->hourlyAt(17);` | Görevi her saat, saati 17 dakika geçe çalıştır. |
| `->everyOddHour($minutes = 0);` | Görevi her tek saatte çalıştır. |
| `->everyTwoHours($minutes = 0);` | Görevi her iki saatte bir çalıştır. |
| `->everyThreeHours($minutes = 0);` | Görevi her üç saatte bir çalıştır. |
| `->everyFourHours($minutes = 0);` | Görevi her dört saatte bir çalıştır. |
| `->everySixHours($minutes = 0);` | Görevi her altı saatte bir çalıştır. |
| `->daily();` | Görevi her gün gece yarısı çalıştır. |
| `->dailyAt('13:00');` | Görevi her gün 13:00'te çalıştır. |
| `->twiceDaily(1, 13);` | Görevi her gün 1:00 ve 13:00'te çalıştır. |
| `->twiceDailyAt(1, 13, 15);` | Görevi her gün 1:15 ve 13:15'te çalıştır. |
| `->daysOfMonth([1, 10, 20]);` | Görevi ayın belirli günlerinde çalıştır. |
| `->weekly();` | Görevi her Pazar 00:00'da çalıştır. |
| `->weeklyOn(1, '8:00');` | Görevi her hafta Pazartesi 8:00'de çalıştır. |
| `->monthly();` | Görevi her ayın ilk günü 00:00'da çalıştır. |
| `->monthlyOn(4, '15:00');` | Görevi her ayın 4'ünde 15:00'te çalıştır. |
| `->twiceMonthly(1, 16, '13:00');` | Görevi her ayın 1. ve 16. günlerinde 13:00'te çalıştır. |
| `->lastDayOfMonth('15:00');` | Görevi ayın son günü 15:00'te çalıştır. |
| `->quarterly();` | Görevi her çeyreğin ilk günü 00:00'da çalıştır. |
| `->quarterlyOn(4, '14:00');` | Görevi her çeyreğin 4. günü 14:00'te çalıştır. |
| `->yearly();` | Görevi her yılın ilk günü 00:00'da çalıştır. |
| `->yearlyOn(6, 1, '17:00');` | Görevi her yıl 1 Haziran'da 17:00'de çalıştır. |
| `->timezone('America/New_York');` | Görev için saat dilimini ayarla. |

Bu metotlar, yalnızca haftanın belirli günlerinde çalışan daha da ince ayarlanmış zamanlamalar oluşturmak için ek kısıtlamalarla birleştirilebilir. Örneğin, bir komutu haftada bir Pazartesi günü çalışacak şekilde zamanlayabilirsiniz:
```php
use Illuminate\Support\Facades\Schedule;

// Haftada bir Pazartesi günü saat 13:00'te çalıştır...
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// Hafta içi her gün saat 8:00'den 17:00'ye kadar her saat çalıştır...
Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

Ek zamanlama kısıtlamalarının bir listesi aşağıda bulunabilir:

| Metot | Açıklama |
| :--- | :--- |
| `->weekdays();` | Görevi hafta içi günleriyle sınırla. |
| `->weekends();` | Görevi hafta sonu günleriyle sınırla. |
| `->sundays();` | Görevi Pazar günüyle sınırla. |
| `->mondays();` | Görevi Pazartesi günüyle sınırla. |
| `->tuesdays();` | Görevi Salı günüyle sınırla. |
| `->wednesdays();` | Görevi Çarşamba günüyle sınırla. |
| `->thursdays();` | Görevi Perşembe günüyle sınırla. |
| `->fridays();` | Görevi Cuma günüyle sınırla. |
| `->saturdays();` | Görevi Cumartesi günüyle sınırla. |
| `->days(array\|mixed);` | Görevi belirli günlerle sınırla. |
| `->between($startTime, $endTime);` | Görevi başlangıç ve bitiş saatleri arasında çalışacak şekilde sınırla. |
| `->unlessBetween($startTime, $endTime);` | Görevi başlangıç ve bitiş saatleri arasında çalışmayacak şekilde sınırla. |
| `->when(Closure);` | Görevi bir doğruluk testine (truth test) göre sınırla. |
| `->environments($env);` | Görevi belirli ortamlarla (environments) sınırla. |

#### Gün Kısıtlamaları (Day Constraints)
`days` metodu, bir görevin yürütülmesini haftanın belirli günleriyle sınırlamak için kullanılabilir. Örneğin, bir komutu Pazar ve Çarşamba günleri saat başı çalışacak şekilde zamanlayabilirsiniz:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```
Alternatif olarak, bir görevin çalışması gereken günleri tanımlarken `Illuminate\Console\Scheduling\Schedule` sınıfında bulunan sabitleri (constants) kullanabilirsiniz:
```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

#### Zaman Aralığı Kısıtlamaları (Between Time Constraints)
`between` metodu, bir görevin yürütülmesini günün saatine göre sınırlamak için kullanılabilir:
```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```
Benzer şekilde, `unlessBetween` metodu bir görevin yürütülmesini belirli bir zaman aralığında hariç tutmak için kullanılabilir:
```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

#### Doğruluk Testi Kısıtlamaları (Truth Test Constraints)
`when` metodu, bir görevin yürütülmesini belirli bir doğruluk testinin (truth test) sonucuna göre sınırlamak için kullanılabilir. Başka bir deyişle, verilen closure `true` döndürürse, görevi çalıştırılmasını engelleyen başka kısıtlama koşulları yoksa görev çalıştırılacaktır:
```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```
`skip` metodu, `when` metodunun tersi olarak görülebilir. `skip` metodu `true` döndürürse, zamanlanmış görev çalıştırılmayacaktır:
```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```
Zincirlenmiş `when` metotları kullanıldığında, zamanlanmış komut yalnızca tüm `when` koşulları `true` döndürürse çalıştırılacaktır.

#### Ortam Kısıtlamaları (Environment Constraints)
`environments` metodu, görevleri yalnızca belirli ortamlarda (`APP_ENV` ortam değişkeni tarafından tanımlanan) çalıştırmak için kullanılabilir:
```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

### Saat Dilimleri (Timezones)
`timezone` metodunu kullanarak, zamanlanmış bir görevin zamanının belirli bir saat diliminde yorumlanması gerektiğini belirtebilirsiniz:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```
Zamanlanmış tüm görevlerinize aynı saat dilimini tekrar tekrar atıyorsanız, uygulama yapılandırma dosyanızda bir `schedule_timezone` seçeneği tanımlayarak tüm zamanlamalara hangi saat diliminin atanması gerektiğini belirtebilirsiniz:
```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```
Bazı saat dilimlerinin gün ışığından yararlanma saatini (daylight savings time) kullandığını unutmayın. Gün ışığından yararlanma saati değişiklikleri meydana geldiğinde, zamanlanmış göreviniz iki kez çalışabilir veya hiç çalışmayabilir. Bu nedenle, mümkün olduğunda saat dilimi zamanlamasından kaçınmanızı öneririz.

### Görev Çakışmalarını Önleme (Preventing Task Overlaps)
Varsayılan olarak, zamanlanmış görevler, görevin önceki örneği (previous instance) hala çalışıyor olsa bile çalıştırılacaktır. Bunu önlemek için `withoutOverlapping` metodunu kullanabilirsiniz:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```
Bu örnekte, `emails:send` Artisan komutu, zaten çalışmıyorsa her dakika çalıştırılacaktır. `withoutOverlapping` metodu, özellikle yürütme süreleri büyük ölçüde değişen ve belirli bir görevin ne kadar süreceğini tam olarak tahmin etmenizi engelleyen görevleriniz varsa kullanışlıdır.

Gerekirse, "çakışmasız" (without overlapping) kilidin (lock) süresinin dolmadan önce kaç dakika geçmesi gerektiğini belirtebilirsiniz. Varsayılan olarak, kilit 24 saat sonra sona erecektir:
```php
Schedule::command('emails:send')->withoutOverlapping(10);
```
Perde arkasında, `withoutOverlapping` metodu kilitleri elde etmek için uygulamanızın önbelleğini (cache) kullanır. Gerekirse, `schedule:clear-cache` Artisan komutunu kullanarak bu önbellek kilitlerini temizleyebilirsiniz. Bu, genellikle yalnızca bir görev beklenmeyen bir sunucu sorunu nedeniyle takılıp kalırsa gereklidir.

### Görevleri Tek Bir Sunucuda Çalıştırma (Running Tasks on One Server)
Bu özelliği kullanmak için uygulamanızın varsayılan önbellek sürücüsü olarak `database`, `memcached`, `dynamodb` veya `redis` önbellek sürücüsünü kullanıyor olması gerekir. Ayrıca, tüm sunucular aynı merkezi önbellek sunucusuyla iletişim kuruyor olmalıdır.

Uygulamanızın zamanlayıcısı birden çok sunucuda çalışıyorsa, zamanlanmış bir işi yalnızca tek bir sunucuda çalışacak şekilde sınırlayabilirsiniz. Örneğin, her Cuma gecesi yeni bir rapor oluşturan zamanlanmış bir göreviniz olduğunu varsayalım. Görev zamanlayıcı üç işçi sunucusunda (worker servers) çalışıyorsa, zamanlanmış görev her üç sunucuda da çalışacak ve raporu üç kez oluşturacaktır. İyi değil!

Görevin yalnızca bir sunucuda çalışması gerektiğini belirtmek için, zamanlanmış görevi tanımlarken `onOneServer` metodunu kullanın. Görevi ilk alan sunucu, diğer sunucuların aynı anda aynı görevi çalıştırmasını önlemek için iş üzerinde atomik bir kilit (atomic lock) sağlayacaktır:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```
Zamanlayıcının tek sunuculu görevler için gerekli atomik kilitleri elde etmek için kullandığı önbellek deposunu (cache store) özelleştirmek için `useCache` metodunu kullanabilirsiniz:
```php
Schedule::useCache('database');
```

#### Tek Sunuculu İşleri Adlandırma (Naming Single Server Jobs)
Bazen aynı işi farklı parametrelerle gönderilecek şekilde zamanlamanız gerekebilir, ancak yine de Laravel'e işin her bir permütasyonunu tek bir sunucuda çalıştırması talimatını vermek isteyebilirsiniz. Bunu başarmak için, her zamanlama tanımına `name` metodu aracılığıyla benzersiz bir ad atayabilirsiniz:
```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();

Schedule::job(new CheckUptime('https://vapor.laravel.com'))
    ->name('check_uptime:vapor.laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```