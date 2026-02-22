# Laravel 12 Dokümantasyonu: Kuyruklar (Queues)

## Giriş

Web uygulamanızı oluştururken, yüklenen bir CSV dosyasını ayrıştırmak ve saklamak gibi, tipik bir web isteği sırasında gerçekleştirilmesi çok uzun sürebilecek bazı görevleriniz olabilir. Neyse ki Laravel, arka planda işlenebilecek sıraya konmuş işler (queued jobs) kolayca oluşturmanıza olanak tanır. Zaman yoğun görevleri bir kuyruğa taşıyarak, uygulamanız web isteklerine yüksek hızla yanıt verebilir ve müşterilerinize daha iyi bir kullanıcı deneyimi sağlayabilir.

Laravel kuyrukları, Amazon SQS, Redis veya hatta ilişkisel bir veritabanı gibi çeşitli farklı kuyruk arka uçları (queue backends) arasında birleşik bir kuyruk API'si sağlar.

Laravel'in kuyruk yapılandırma seçenekleri, uygulamanızın `config/queue.php` yapılandırma dosyasında saklanır. Bu dosyada, framework ile birlikte gelen her bir kuyruk sürücüsü (queue driver) için bağlantı yapılandırmalarını bulacaksınız. Bunlar arasında `database`, Amazon SQS, Redis ve Beanstalkd sürücüleri ile işleri hemen yürütecek (geliştirme veya test sırasında kullanım için) bir senkron sürücü (synchronous driver) bulunur. Sıraya konmuş işleri atan (discard) bir `null` kuyruk sürücüsü de dahil edilmiştir.

Laravel Horizon, Redis destekli kuyruklarınız için güzel bir pano (dashboard) ve yapılandırma sistemidir. Daha fazla bilgi için eksiksiz Horizon dokümantasyonuna göz atın.

### Bağlantılar (Connections) ve Kuyruklar (Queues)
Laravel kuyruklarına başlamadan önce, "bağlantılar" (connections) ve "kuyruklar" (queues) arasındaki ayrımı anlamak önemlidir. `config/queue.php` yapılandırma dosyanızda bir `connections` yapılandırma dizisi bulunur. Bu seçenek, Amazon SQS, Beanstalk veya Redis gibi arka uç kuyruk servislerine olan bağlantıları tanımlar. Ancak, herhangi bir kuyruk bağlantısının, sıraya konmuş işlerin farklı yığınları veya kümeleri olarak düşünülebilecek birden çok "kuyruğu" olabilir.

Kuyruk yapılandırma dosyasındaki her bağlantı yapılandırma örneğinin bir `queue` özelliği içerdiğine dikkat edin. Bu, işlerin belirli bir bağlantıya gönderildiğinde (dispatched) gönderileceği varsayılan kuyruktur. Başka bir deyişle, bir işi hangi kuyruğa gönderileceğini açıkça belirtmeden gönderirseniz, iş, bağlantı yapılandırmasının `queue` özelliğinde tanımlanan kuyruğa yerleştirilecektir:
```php
use App\Jobs\ProcessPodcast;

// Bu iş, varsayılan bağlantının varsayılan kuyruğuna gönderilir...
ProcessPodcast::dispatch();

// Bu iş, varsayılan bağlantının "emails" kuyruğuna gönderilir...
ProcessPodcast::dispatch()->onQueue('emails');
```
Bazı uygulamaların, işleri birden çok kuyruğa göndermeleri asla gerekmeyebilir, bunun yerine tek bir basit kuyruğa sahip olmayı tercih edebilirler. Ancak, işleri birden çok kuyruğa göndermek, Laravel kuyruk işçisinin (queue worker) önceliğe göre hangi kuyrukları işlemesi gerektiğini belirtmesine izin verdiğinden, işlerin nasıl işleneceğine öncelik vermek veya bölümlere ayırmak isteyen uygulamalar için özellikle yararlı olabilir. Örneğin, işleri bir `high` (yüksek) kuyruğuna gönderirseniz, onlara daha yüksek işleme önceliği veren bir işçi çalıştırabilirsiniz:
```bash
php artisan queue:work --queue=high,default
```

### Sürücü Notları ve Ön Gereksinimler (Driver Notes and Prerequisites)

#### Veritabanı (Database)
`database` kuyruk sürücüsünü kullanmak için işleri (jobs) tutacak bir veritabanı tablosuna ihtiyacınız olacaktır. Tipik olarak bu, Laravel'in varsayılan `0001_01_01_000002_create_jobs_table.php` veritabanı migration'ında bulunur; ancak, uygulamanız bu migration'ı içermiyorsa, onu oluşturmak için `make:queue-table` Artisan komutunu kullanabilirsiniz:
```bash
php artisan make:queue-table

php artisan migrate
```

#### Redis
`redis` kuyruk sürücüsünü kullanmak için `config/database.php` yapılandırma dosyanızda bir Redis veritabanı bağlantısı yapılandırmalısınız.

`serializer` ve `compression` Redis seçenekleri, `redis` kuyruk sürücüsü tarafından desteklenmez.

##### Redis Kümesi (Redis Cluster)
Redis kuyruk bağlantınız bir Redis Kümesi (Redis Cluster) kullanıyorsa, kuyruk adlarınız bir anahtar karma etiketi (key hash tag) içermelidir. Bu, belirli bir kuyruk için tüm Redis anahtarlarının aynı karma yuvasına (hash slot) yerleştirildiğinden emin olmak için gereklidir:
```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

##### Engelleme (Blocking)
Redis kuyruğunu kullanırken, sürücünün işçi döngüsünde yineleme yapmadan ve Redis veritabanını yeniden yoklamadan (re-polling) önce bir işin kullanılabilir hale gelmesini ne kadar bekleyeceğini belirtmek için `block_for` yapılandırma seçeneğini kullanabilirsiniz.

Bu değeri kuyruk yükünüze göre ayarlamak, yeni işler için Redis veritabanını sürekli yoklamaktan daha verimli olabilir. Örneğin, sürücünün bir işin kullanılabilir hale gelmesini beklerken beş saniye engellemesi gerektiğini belirtmek için değeri `5` olarak ayarlayabilirsiniz:
```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```
`block_for` değerini `0` olarak ayarlamak, kuyruk işçilerinin bir iş kullanılabilir olana kadar süresiz olarak engellenmesine neden olacaktır. Bu ayrıca, bir sonraki iş işlenene kadar `SIGTERM` gibi sinyallerin işlenmesini de engelleyecektir.

#### Diğer Sürücü Ön Gereksinimleri (Other Driver Prerequisites)
Listelenen kuyruk sürücüleri için aşağıdaki bağımlılıklar gereklidir. Bu bağımlılıklar Composer paket yöneticisi aracılığıyla kurulabilir:

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` veya phpredis PHP eklentisi
- MongoDB: `mongodb/laravel-mongodb`

## İş (Job) Oluşturma

### İş Sınıfları Oluşturma (Generating Job Classes)
Varsayılan olarak, uygulamanız için sıraya konabilir tüm işler (queueable jobs) `app/Jobs` dizininde saklanır. `app/Jobs` dizini yoksa, `make:job` Artisan komutunu çalıştırdığınızda oluşturulacaktır:
```bash
php artisan make:job ProcessPodcast
```
Oluşturulan sınıf, Laravel'e işin asenkron olarak çalıştırılmak üzere kuyruğa gönderilmesi gerektiğini belirten `Illuminate\Contracts\Queue\ShouldQueue` arayüzünü (interface) uygulayacaktır (implement).

İş iskeletleri (job stubs), iskelet yayınlama (stub publishing) kullanılarak özelleştirilebilir.

### Sınıf Yapısı (Class Structure)
İş sınıfları çok basittir, normalde yalnızca iş kuyruk tarafından işlendiğinde çağrılan bir `handle` metodu içerir. Başlamak için örnek bir iş sınıfına bakalım. Bu örnekte, bir podcast yayınlama hizmeti yönettiğimizi ve yayınlanmadan önce yüklenen podcast dosyalarını işlememiz gerektiğini varsayalım:
```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Yeni bir iş örneği oluştur.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * İşi yürüt.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Yüklenen podcast'i işle...
    }
}
```
Bu örnekte, sıraya konmuş işin kurucusuna (constructor) doğrudan bir Eloquent modeli aktarabildiğimize dikkat edin. İşin kullandığı `Queueable` özelliği (trait) sayesinde, Eloquent modelleri ve yüklenmiş ilişkileri (loaded relationships), iş işlenirken sorunsuz bir şekilde serileştirilecek (serialized) ve serisi çözülecektir (unserialized).

Sıraya konmuş işiniz kurucusunda bir Eloquent modeli kabul ediyorsa, model için yalnızca tanımlayıcı (identifier) kuyruğa serileştirilecektir. İş fiilen işlendiğinde, kuyruk sistemi tam model örneğini ve yüklenmiş ilişkilerini veritabanından otomatik olarak yeniden alacaktır. Model serileştirmeye yönelik bu yaklaşım, kuyruk sürücünüze çok daha küçük iş yükleri (job payloads) gönderilmesine olanak tanır.

#### `handle` Metodunda Bağımlılık Enjeksiyonu (handle Method Dependency Injection)
`handle` metodu, iş kuyruk tarafından işlendiğinde çağrılır. İşin `handle` metodunda bağımlılıkları tip belirtebildiğimize (type-hint) dikkat edin. Laravel servis kabı (service container) bu bağımlılıkları otomatik olarak enjekte eder.

Kabın (container) `handle` metoduna bağımlılıkları nasıl enjekte ettiği üzerinde tam kontrol sahibi olmak isterseniz, kabın `bindMethod` metodunu kullanabilirsiniz. `bindMethod` metodu, işi ve kabı alan bir geri çağrı (callback) kabul eder. Geri çağrı içinde, `handle` metodunu dilediğiniz gibi çağırmakta özgürsünüz. Tipik olarak, bu metodu `App\Providers\AppServiceProvider` servis sağlayıcınızın `boot` metodundan çağırmalısınız:
```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```
Ham görüntü içerikleri gibi ikili veriler (binary data), sıraya konmuş bir işe aktarılmadan önce `base64_encode` işlevinden geçirilmelidir. Aksi takdirde, iş kuyruğa yerleştirilirken JSON'a düzgün şekilde serileştirilemeyebilir.

#### Sıraya Konmuş İlişkiler (Queued Relationships)
Bir iş sıraya konduğunda yüklenmiş tüm Eloquent model ilişkileri de serileştirildiğinden, serileştirilmiş iş dizesi bazen oldukça büyüyebilir. Ayrıca, bir işin serisi çözüldüğünde ve model ilişkileri veritabanından yeniden alındığında, bunlar bütünüyle alınacaktır. İş kuyruğa alınırken model serileştirilmeden önce uygulanmış olan önceki herhangi bir ilişki kısıtlaması (relationship constraints), işin serisi çözüldüğünde uygulanmayacaktır. Bu nedenle, belirli bir ilişkinin bir alt kümesiyle çalışmak istiyorsanız, bu ilişkiyi sıraya konmuş işiniz içinde yeniden kısıtlamalısınız (re-constrain).

Veya, ilişkilerin serileştirilmesini önlemek için, bir özellik değeri (property value) ayarlarken model üzerinde `withoutRelations` metodunu çağırabilirsiniz. Bu metod, yüklenmiş ilişkileri olmadan modelin bir örneğini döndürecektir:
```php
/**
 * Yeni bir iş örneği oluştur.
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```
PHP kurucu özellik tanıtımını (constructor property promotion) kullanıyorsanız ve bir Eloquent modelinin ilişkilerinin serileştirilmemesi gerektiğini belirtmek isterseniz, `WithoutRelations` niteliğini (attribute) kullanabilirsiniz:
```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * Yeni bir iş örneği oluştur.
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```
Kolaylık olması açısından, tüm modelleri ilişkileri olmadan serileştirmek isterseniz, `WithoutRelations` niteliğini her bir modele uygulamak yerine tüm sınıfa uygulayabilirsiniz:
```php
<?php

namespace App\Jobs;

use App\Models\DistributionPlatform;
use App\Models\Podcast;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\WithoutRelations;

#[WithoutRelations]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Yeni bir iş örneği oluştur.
     */
    public function __construct(
        public Podcast $podcast,
        public DistributionPlatform $platform,
    ) {}
}
```
Bir iş, tek bir model yerine bir Eloquent modelleri koleksiyonu veya dizisi alırsa, bu koleksiyondaki modellerin, işin serisi çözüldüğünde ve yürütüldüğünde ilişkileri geri yüklenmeyecektir. Bu, çok sayıda modelle ilgilenen işlerde aşırı kaynak kullanımını önlemek içindir.

### Benzersiz İşler (Unique Jobs)
Benzersiz işler, kilitleri (locks) destekleyen bir önbellek sürücüsü (cache driver) gerektirir. Şu anda `memcached`, `redis`, `dynamodb`, `database`, `file` ve `array` önbellek sürücüleri atomik kilitleri (atomic locks) destekler.

Benzersiz iş kısıtlamaları, toplu işler (batches) içindeki işler için geçerli değildir.

Bazen, belirli bir işin herhangi bir zamanda kuyrukta yalnızca bir örneğinin (instance) bulunduğundan emin olmak isteyebilirsiniz. Bunu, iş sınıfınızda `ShouldBeUnique` arayüzünü uygulayarak yapabilirsiniz. Bu arayüz, sınıfınızda herhangi bir ek metot tanımlamanızı gerektirmez:
```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...
}
```
Yukarıdaki örnekte, `UpdateSearchIndex` işi benzersizdir. Bu nedenle, işin başka bir örneği zaten kuyruktaysa ve işlemeyi bitirmemişse, iş gönderilmeyecektir (dispatched).

Bazı durumlarda, işi benzersiz kılan belirli bir "anahtar" (key) tanımlamak veya işin artık benzersiz kalmayacağı bir zaman aşımı (timeout) belirtmek isteyebilirsiniz. Bunu başarmak için iş sınıfınızda `uniqueId` ve `uniqueFor` özelliklerini (properties) veya metotlarını tanımlayabilirsiniz:
```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * Ürün örneği.
     *
     * @var \App\Models\Product
     */
    public $product;

    /**
     * İşin benzersiz kilidinin serbest bırakılacağı saniye sayısı.
     *
     * @var int
     */
    public $uniqueFor = 3600;

    /**
     * İş için benzersiz kimliği (ID) al.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```
Yukarıdaki örnekte, `UpdateSearchIndex` işi bir ürün ID'sine göre benzersizdir. Bu nedenle, mevcut iş işlemeyi tamamlayana kadar aynı ürün ID'sine sahip işin yeni gönderimleri yok sayılacaktır. Ayrıca, mevcut iş bir saat içinde işlenmezse, benzersiz kilit serbest bırakılacak ve aynı benzersiz anahtara sahip başka bir iş kuyruğa gönderilebilecektir.

Uygulamanız birden çok web sunucusundan veya kapsayıcıdan (container) işler gönderiyorsa, tüm sunucularınızın aynı merkezi önbellek sunucusuyla iletişim kurduğundan emin olmalısınız, böylece Laravel bir işin benzersiz olup olmadığını doğru bir şekilde belirleyebilir.

#### İşleri İşleme Başlayana Kadar Benzersiz Tutma (Keeping Jobs Unique Until Processing Begins)
Varsayılan olarak, benzersiz işler, bir iş işlemeyi tamamladıktan veya tüm yeniden deneme girişimlerinde başarısız olduktan sonra "kilidi açılır" (unlocked). Ancak, işinizin işlenmeden hemen önce kilidinin açılmasını isteyebileceğiniz durumlar olabilir. Bunu başarmak için işinizin `ShouldBeUnique` sözleşmesi (contract) yerine `ShouldBeUniqueUntilProcessing` sözleşmesini uygulaması gerekir:
```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```