# Laravel 12 Dokümantasyonu: Önbellek (Cache)

## Giriş

Uygulamanız tarafından gerçekleştirilen bazı veri alma veya işleme görevleri CPU yoğun olabilir veya tamamlanması birkaç saniye sürebilir. Durum böyle olduğunda, alınan verileri bir süre önbelleğe almak (cache) yaygındır, böylece aynı veri için sonraki isteklerde hızlı bir şekilde alınabilir. Önbelleğe alınan veriler genellikle Memcached veya Redis gibi çok hızlı veri depolarında saklanır.

Neyse ki Laravel, çeşitli önbellek arka uçları (cache backends) için anlamlı, birleşik bir API sağlayarak, onların son derece hızlı veri alma yeteneklerinden yararlanmanıza ve web uygulamanızı hızlandırmanıza olanak tanır.

## Yapılandırma (Configuration)

Uygulamanızın önbellek yapılandırma dosyası `config/cache.php` konumunda bulunur. Bu dosyada, uygulamanız genelinde varsayılan olarak hangi önbellek deposunu (cache store) kullanmak istediğinizi belirtebilirsiniz. Laravel, Memcached, Redis, DynamoDB ve ilişkisel veritabanları gibi popüler önbellek arka uçlarını kutudan çıktığı gibi destekler. Ayrıca, dosya tabanlı bir önbellek sürücüsü (file based cache driver) mevcuttur; `array` ve `null` önbellek sürücüleri ise otomatik testleriniz için kullanışlı önbellek arka uçları sağlar.

Önbellek yapılandırma dosyası ayrıca inceleyebileceğiniz çeşitli başka seçenekler de içerir. Varsayılan olarak Laravel, serileştirilmiş (serialized), önbelleğe alınmış nesneleri uygulamanızın veritabanında saklayan `database` önbellek sürücüsünü kullanacak şekilde yapılandırılmıştır.

### Sürücü Ön Gereksinimleri (Driver Prerequisites)

#### Veritabanı (Database)
`database` önbellek sürücüsünü kullanırken, önbellek verilerini içerecek bir veritabanı tablonuza ihtiyacınız olacaktır. Tipik olarak bu, Laravel'in varsayılan `0001_01_01_000001_create_cache_table.php` veritabanı migration'ında bulunur; ancak, uygulamanız bu migration'ı içermiyorsa, onu oluşturmak için `make:cache-table` Artisan komutunu kullanabilirsiniz:
```bash
php artisan make:cache-table

php artisan migrate
```

#### Memcached
Memcached sürücüsünü kullanmak, Memcached PECL paketinin kurulu olmasını gerektirir. Tüm Memcached sunucularınızı `config/cache.php` yapılandırma dosyasında listeleyebilirsiniz. Bu dosya, başlamanız için zaten bir `memcached.servers` girdisi içerir:
```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```
Gerekirse, `host` seçeneğini bir UNIX soket yoluna (UNIX socket path) ayarlayabilirsiniz. Bunu yaparsanız, `port` seçeneği `0` olarak ayarlanmalıdır:
```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
],
```

#### Redis
Laravel ile bir Redis önbelleği kullanmadan önce, PECL aracılığıyla PhpRedis PHP eklentisini (extension) kurmanız veya Composer aracılığıyla `predis/predis` paketini (~2.0) kurmanız gerekecektir. Laravel Sail bu eklentiyi zaten içerir. Ayrıca, Laravel Cloud ve Laravel Forge gibi resmi Laravel uygulama platformlarında PhpRedis eklentisi varsayılan olarak kuruludur.

Redis'i yapılandırma hakkında daha fazla bilgi için Laravel'in Redis dokümantasyon sayfasına bakın.

#### DynamoDB
DynamoDB önbellek sürücüsünü kullanmadan önce, tüm önbelleğe alınmış verileri depolamak için bir DynamoDB tablosu oluşturmalısınız. Tipik olarak, bu tablo `cache` olarak adlandırılmalıdır. Ancak, tabloyu önbellek yapılandırma dosyasındaki `stores.dynamodb.table` yapılandırma değerine göre adlandırmalısınız. Tablo adı ayrıca `DYNAMODB_CACHE_TABLE` ortam değişkeni aracılığıyla da ayarlanabilir.

Bu tablo ayrıca, adı uygulamanızın önbellek yapılandırma dosyasındaki `stores.dynamodb.attributes.key` yapılandırma öğesinin değerine karşılık gelen bir dize bölümleme anahtarına (string partition key) sahip olmalıdır. Varsayılan olarak, bölümleme anahtarı `key` olarak adlandırılmalıdır.

Tipik olarak DynamoDB, süresi dolmuş öğeleri (expired items) proaktif olarak bir tablodan kaldırmaz. Bu nedenle, tabloda Yaşam Süresi'ni (Time to Live - TTL) etkinleştirmelisiniz. Tablonun TTL ayarlarını yapılandırırken, TTL öznitelik adını (attribute name) `expires_at` olarak ayarlamalısınız.

Ardından, Laravel uygulamanızın DynamoDB ile iletişim kurabilmesi için AWS SDK'sını kurun:
```bash
composer require aws/aws-sdk-php
```
Ek olarak, DynamoDB önbellek deposu yapılandırma seçenekleri için değerlerin sağlandığından emin olmalısınız. Tipik olarak bu seçenekler (`AWS_ACCESS_KEY_ID` ve `AWS_SECRET_ACCESS_KEY` gibi) uygulamanızın `.env` yapılandırma dosyasında tanımlanmalıdır:
```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

#### MongoDB
MongoDB kullanıyorsanız, resmi `mongodb/laravel-mongodb` paketi tarafından bir `mongodb` önbellek sürücüsü sağlanır ve bir `mongodb` veritabanı bağlantısı kullanılarak yapılandırılabilir. MongoDB, süresi dolmuş önbellek öğelerini otomatik olarak temizlemek için kullanılabilen TTL indekslerini destekler.

MongoDB'yi yapılandırma hakkında daha fazla bilgi için lütfen MongoDB Önbellek ve Kilitler dokümantasyonuna bakın.

## Önbellek Kullanımı (Cache Usage)

### Bir Önbellek Örneği (Instance) Alma
Bir önbellek deposu örneği elde etmek için, bu dokümantasyon boyunca kullanacağımız `Cache` facade'ını kullanabilirsiniz. `Cache` facade'ı, Laravel önbellek sözleşmelerinin (cache contracts) temel gerçekleştirimlerine (underlying implementations) kullanışlı, kısa erişim sağlar:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Uygulamanın tüm kullanıcılarının bir listesini göster.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

#### Birden Çok Önbellek Deposuna Erişim (Accessing Multiple Cache Stores)
`Cache` facade'ını kullanarak, `store` metodu aracılığıyla çeşitli önbellek depolarına erişebilirsiniz. `store` metoduna iletilen anahtar, önbellek yapılandırma dosyanızdaki `stores` yapılandırma dizisinde listelenen depolardan birine karşılık gelmelidir:
```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 Dakika
```

### Önbellekten Öğe Alma (Retrieving Items From the Cache)
`Cache` facade'ının `get` metodu, önbellekten öğe almak için kullanılır. Öğe önbellekte yoksa, `null` döndürülecektir. İsterseniz, `get` metoduna ikinci bir argüman ileterek, öğe yoksa döndürülmesini istediğiniz varsayılan değeri belirtebilirsiniz:
```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```
Hatta varsayılan değer olarak bir closure iletebilirsiniz. Belirtilen öğe önbellekte yoksa closure'ın sonucu döndürülecektir. Bir closure iletmek, varsayılan değerlerin bir veritabanından veya başka bir harici servisten alınmasını ertelemenize (defer) olanak tanır:
```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

#### Öğe Varlığını Belirleme (Determining Item Existence)
`has` metodu, bir öğenin önbellekte mevcut olup olmadığını belirlemek için kullanılabilir. Bu metod, öğe mevcutsa ancak değeri `null` ise de `false` döndürecektir:
```php
if (Cache::has('key')) {
    // ...
}
```

#### Değerleri Artırma / Azaltma (Incrementing / Decrementing Values)
`increment` ve `decrement` metotları, önbellekteki tamsayı öğelerin değerini ayarlamak için kullanılabilir. Bu metotların her ikisi de, öğenin değerini artırma veya azaltma miktarını belirten isteğe bağlı bir ikinci argümanı kabul eder:
```php
// Değer yoksa başlat...
Cache::add('key', 0, now()->addHours(4));

// Değeri artır veya azalt...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

#### Al ve Sakla (Retrieve and Store)
Bazen bir öğeyi önbellekten almak, ancak istenen öğe yoksa varsayılan bir değer de saklamak isteyebilirsiniz. Örneğin, tüm kullanıcıları önbellekten almak veya yoksa veritabanından alıp önbelleğe eklemek isteyebilirsiniz. Bunu `Cache::remember` metodunu kullanarak yapabilirsiniz:
```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```
Öğe önbellekte yoksa, `remember` metoduna iletilen closure yürütülecek ve sonucu önbelleğe yerleştirilecektir.

Bir öğeyi önbellekten almak veya yoksa sonsuza kadar saklamak için `rememberForever` metodunu kullanabilirsiniz:
```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

#### Bayatken Yeniden Doğrula (Stale While Revalidate)
`Cache::remember` metodunu kullanırken, önbelleğe alınan değerin süresi dolduysa bazı kullanıcılar yavaş yanıt süreleri yaşayabilir. Belirli veri türleri için, önbellek değeri arka planda yeniden hesaplanırken kısmen eski (stale) verilerin sunulmasına izin vermek, önbellek değerleri hesaplanırken bazı kullanıcıların yavaş yanıt süreleri yaşamasını önleyebilir. Buna genellikle "bayatken yeniden doğrula" (stale-while-revalidate) deseni denir ve `Cache::flexible` metodu bu desenin bir uygulamasını sağlar.

`flexible` metodu, önbelleğe alınan değerin ne kadar süreyle "taze" (fresh) kabul edileceğini ve ne zaman "bayat" (stale) hale geldiğini belirten bir dizi kabul eder. Dizideki ilk değer, önbelleğin taze kabul edildiği saniye sayısını temsil ederken, ikinci değer, yeniden hesaplama gerekli olmadan önce bayat veri olarak ne kadar süre sunulabileceğini tanımlar.

Bir istek taze dönem içinde (ilk değerden önce) yapılırsa, önbellek yeniden hesaplama yapılmadan hemen döndürülür. Bir istek bayat dönem sırasında (iki değer arasında) yapılırsa, kullanıcıya bayat değer sunulur ve yanıt kullanıcıya gönderildikten sonra önbelleğe alınan değeri yenilemek için ertelenmiş bir fonksiyon (deferred function) kaydedilir. Bir istek ikinci değerden sonra yapılırsa, önbelleğin süresi dolmuş kabul edilir ve değer hemen yeniden hesaplanır, bu da kullanıcı için daha yavaş bir yanıta neden olabilir:
```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

#### Al ve Sil (Retrieve and Delete)
Bir öğeyi önbellekten almanız ve ardından öğeyi silmeniz gerekiyorsa, `pull` metodunu kullanabilirsiniz. `get` metodu gibi, öğe önbellekte yoksa `null` döndürülecektir:
```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

### Önbelleğe Öğe Saklama (Storing Items in the Cache)
`Cache` facade'ında `put` metodunu kullanarak önbelleğe öğe saklayabilirsiniz:
```php
Cache::put('key', 'value', $seconds = 10);
```
Saklama süresi `put` metoduna iletilmezse, öğe süresiz olarak saklanacaktır:
```php
Cache::put('key', 'value');
```
Saniye sayısını bir tamsayı olarak iletmek yerine, önbelleğe alınan öğenin istenen son kullanma zamanını (expiration time) temsil eden bir `DateTime` örneği de iletebilirsiniz:
```php
Cache::put('key', 'value', now()->addMinutes(10));
```

#### Yoksa Sakla (Store if Not Present)
`add` metodu, öğeyi yalnızca önbellek deposunda zaten mevcut değilse önbelleğe ekleyecektir. Metod, öğe gerçekten önbelleğe eklendiyse `true` döndürecektir. Aksi takdirde, metod `false` döndürecektir. `add` metodu atomik bir işlemdir (atomic operation):
```php
Cache::add('key', 'value', $seconds);
```

#### Öğeleri Süresiz Saklama (Storing Items Forever)
`forever` metodu, bir öğeyi kalıcı olarak önbellekte saklamak için kullanılabilir. Bu öğelerin süresi dolmayacağından, `forget` metodu kullanılarak manuel olarak önbellekten kaldırılmaları gerekir:
```php
Cache::forever('key', 'value');
```
Memcached sürücüsünü kullanıyorsanız, "süresiz" (forever) saklanan öğeler, önbellek boyut sınırına ulaştığında kaldırılabilir.

### Önbellekten Öğe Kaldırma (Removing Items From the Cache)
Önbellekten öğeleri `forget` metodunu kullanarak kaldırabilirsiniz:
```php
Cache::forget('key');
```
Ayrıca, sıfır veya negatif sayıda saniyelik son kullanma süresi sağlayarak da öğeleri kaldırabilirsiniz:
```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```
Tüm önbelleği `flush` metodunu kullanarak temizleyebilirsiniz:
```php
Cache::flush();
```
Önbelleği temizlemek (flushing), yapılandırılmış önbellek "ön ekinize" (prefix) saygı göstermez ve önbellekteki tüm girdileri kaldıracaktır. Diğer uygulamalar tarafından paylaşılan bir önbelleği temizlerken bunu dikkatlice değerlendirin.

### Önbellek Ezberleme (Cache Memoization)
Laravel'in `memo` önbellek sürücüsü, çözümlenmiş (resolved) önbellek değerlerini tek bir istek veya iş yürütmesi sırasında geçici olarak bellekte saklamanıza olanak tanır. Bu, aynı yürütme içinde tekrarlanan önbellek vuruşlarını (cache hits) önleyerek performansı önemli ölçüde artırır.

Ezberlenmiş (memoized) önbelleği kullanmak için `memo` metodunu çağırın:
```php
use Illuminate\Support\Facades\Cache;

$value = Cache::memo()->get('key');
```
`memo` metodu isteğe bağlı olarak bir önbellek deposunun adını kabul eder; bu, ezberlenmiş sürücünün süsleyeceği (decorate) temel önbellek deposunu belirtir:
```php
// Varsayılan önbellek deposunu kullan...
$value = Cache::memo()->get('key');

// Redis önbellek deposunu kullan...
$value = Cache::memo('redis')->get('key');
```
Belirli bir anahtar için ilk `get` çağrısı, değeri önbellek deponuzdan alır, ancak aynı istek veya iş içindeki sonraki çağrılar değeri bellekten alır:
```php
// Önbelleğe erişir...
$value = Cache::memo()->get('key');

// Önbelleğe erişmez, ezberlenmiş değeri döndürür...
$value = Cache::memo()->get('key');
```
Önbellek değerlerini değiştiren metotları çağırırken (`put`, `increment`, `remember` vb.), ezberlenmiş önbellek otomatik olarak ezberlenmiş değeri unutur ve değiştirme (mutating) metod çağrısını temel önbellek deposuna devreder (delegate):
```php
Cache::memo()->put('name', 'Taylor'); // Temel önbelleğe yazar...
Cache::memo()->get('name');           // Temel önbelleğe erişir...
Cache::memo()->get('name');           // Ezberlendi, önbelleğe erişmez...

Cache::memo()->put('name', 'Tim');    // Ezberlenmiş değeri unutur, yeni değeri yazar...
Cache::memo()->get('name');           // Tekrar temel önbelleğe erişir...
```

### Önbellek Yardımcısı (The Cache Helper)
`Cache` facade'ını kullanmanın yanı sıra, önbellek aracılığıyla veri almak ve saklamak için global `cache` fonksiyonunu da kullanabilirsiniz. `cache` fonksiyonu tek bir dize argümanıyla çağrıldığında, verilen anahtarın değerini döndürecektir:
```php
$value = cache('key');
```
Fonksiyona bir dizi anahtar/değer çifti ve bir son kullanma süresi sağlarsanız, belirtilen süre boyunca değerleri önbellekte saklayacaktır:
```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```
`cache` fonksiyonu herhangi bir argüman olmadan çağrıldığında, `Illuminate\Contracts\Cache\Factory` gerçekleştiriminin bir örneğini döndürerek diğer önbellekleme metotlarını çağırmanıza olanak tanır:
```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```