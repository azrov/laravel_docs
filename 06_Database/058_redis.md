# Laravel 12 Dokümantasyonu: Redis

## Giriş

Redis, açık kaynaklı, gelişmiş bir anahtar-değer deposudur (key-value store). Anahtarlar (keys) dizeler, hash'ler, listeler, kümeler ve sıralı kümeler içerebildiğinden, genellikle bir veri yapısı sunucusu (data structure server) olarak anılır.

Redis'i Laravel ile kullanmadan önce, PECL aracılığıyla PhpRedis PHP eklentisini (extension) kurup kullanmanızı öneririz. Eklenti, "kullanıcı alanı" (user-land) PHP paketlerine kıyasla kurulumu daha karmaşıktır ancak Redis'i yoğun olarak kullanan uygulamalar için daha iyi performans sağlayabilir. Laravel Sail kullanıyorsanız, bu eklenti uygulamanızın Docker konteynerında zaten kuruludur.

PhpRedis eklentisini kuramıyorsanız, Composer aracılığıyla `predis/predis` paketini kurabilirsiniz. Predis, tamamen PHP ile yazılmış bir Redis istemcisidir ve herhangi bir ek eklenti gerektirmez:
```bash
composer require predis/predis
```

## Yapılandırma (Configuration)

Uygulamanızın Redis ayarlarını `config/database.php` yapılandırma dosyası aracılığıyla yapılandırabilirsiniz. Bu dosyada, uygulamanız tarafından kullanılan Redis sunucularını içeren bir `redis` dizisi göreceksiniz:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

],
```
Yapılandırma dosyanızda tanımlanan her bir Redis sunucusunun, Redis bağlantısını temsil eden tek bir URL tanımlamadığınız sürece, bir adı, ana bilgisayarı (host) ve bir bağlantı noktasına (port) sahip olması gerekir:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => 'tcp://127.0.0.1:6379?database=0',
    ],

    'cache' => [
        'url' => 'tls://user:password@127.0.0.1:6380?database=1',
    ],

],
```

#### Bağlantı Şemasını Yapılandırma (Configuring the Connection Scheme)
Varsayılan olarak, Redis istemcileri Redis sunucularınıza bağlanırken `tcp` şemasını kullanacaktır; ancak, Redis sunucunuzun yapılandırma dizisinde bir `scheme` yapılandırma seçeneği belirterek TLS / SSL şifrelemesi kullanabilirsiniz:
```php
'default' => [
    'scheme' => 'tls',
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
],
```

### Kümeler (Clusters)
Uygulamanız bir Redis sunucu kümesi (cluster) kullanıyorsa, bu kümeleri Redis yapılandırmanızın bir `clusters` anahtarı içinde tanımlamalısınız. Bu yapılandırma anahtarı varsayılan olarak mevcut değildir, bu nedenle onu uygulamanızın `config/database.php` yapılandırma dosyasında oluşturmanız gerekecektir:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'clusters' => [
        'default' => [
            [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'username' => env('REDIS_USERNAME'),
                'password' => env('REDIS_PASSWORD'),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_DB', '0'),
            ],
        ],
    ],

    // ...
],
```
Varsayılan olarak Laravel, `options.cluster` yapılandırma değeri `redis` olarak ayarlandığından yerel Redis kümelemeyi (native Redis clustering) kullanacaktır. Redis kümeleme, hatadan korunmayı (failover) sorunsuz bir şekilde ele aldığı için harika bir varsayılan seçenektir.

Laravel, Predis kullanırken istemci tarafı parçalamayı (client-side sharding) da destekler. Ancak, istemci tarafı parçalama hatadan korunmayı (failover) ele almaz; bu nedenle, öncelikle başka bir birincil veri deposundan (primary data store) elde edilebilen geçici önbelleğe alınmış veriler için uygundur.

Yerel Redis kümeleme yerine istemci tarafı parçalama kullanmak isterseniz, uygulamanızın `config/database.php` yapılandırma dosyasındaki `options.cluster` yapılandırma değerini kaldırabilirsiniz:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'clusters' => [
        // ...
    ],

    // ...
],
```

### Predis Kullanımı
Uygulamanızın Redis ile Predis paketi aracılığıyla etkileşime girmesini istiyorsanız, `REDIS_CLIENT` ortam değişkeninin değerinin `predis` olduğundan emin olmalısınız:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // ...
],
```
Varsayılan yapılandırma seçeneklerine ek olarak Predis, Redis sunucularınızın her biri için tanımlanabilecek ek bağlantı parametrelerini destekler. Bu ek yapılandırma seçeneklerini kullanmak için, bunları uygulamanızın `config/database.php` yapılandırma dosyasındaki Redis sunucu yapılandırmanıza ekleyin:
```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_write_timeout' => 60,
],
```

### PhpRedis Kullanımı
Varsayılan olarak Laravel, Redis ile iletişim kurmak için PhpRedis eklentisini kullanacaktır. Laravel'in Redis ile iletişim kurmak için kullanacağı istemci, tipik olarak `REDIS_CLIENT` ortam değişkeninin değerini yansıtan `redis.client` yapılandırma seçeneğinin değeri tarafından belirlenir:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    // ...
],
```
Varsayılan yapılandırma seçeneklerine ek olarak PhpRedis, aşağıdaki ek bağlantı parametrelerini destekler: `name`, `persistent`, `persistent_id`, `prefix`, `read_timeout`, `retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base`, `backoff_cap`, `timeout` ve `context`. Bu seçeneklerden herhangi birini `config/database.php` yapılandırma dosyasındaki Redis sunucu yapılandırmanıza ekleyebilirsiniz:
```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_timeout' => 60,
    'context' => [
        // 'auth' => ['username', 'secret'],
        // 'stream' => ['verify_peer' => false],
    ],
],
```

#### Yeniden Deneme (Retry) ve Geri Çekilme (Backoff) Yapılandırması
`retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base` ve `backoff_cap` seçenekleri, PhpRedis istemcisinin bir Redis sunucusuna yeniden bağlanmayı nasıl denemesi gerektiğini yapılandırmak için kullanılabilir. Aşağıdaki geri çekilme algoritmaları (backoff algorithms) desteklenir: `default`, `decorrelated_jitter`, `equal_jitter`, `exponential`, `uniform` ve `constant`:
```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'max_retries' => env('REDIS_MAX_RETRIES', 3),
    'backoff_algorithm' => env('REDIS_BACKOFF_ALGORITHM', 'decorrelated_jitter'),
    'backoff_base' => env('REDIS_BACKOFF_BASE', 100),
    'backoff_cap' => env('REDIS_BACKOFF_CAP', 1000),
],
```

#### Unix Soket Bağlantıları (Unix Socket Connections)
Redis bağlantıları, TCP yerine Unix soketleri (Unix sockets) kullanacak şekilde de yapılandırılabilir. Bu, uygulamanızla aynı sunucudaki Redis örneklerine yapılan bağlantılar için TCP yükünü (overhead) ortadan kaldırarak performansı artırabilir. Redis'i bir Unix soketi kullanacak şekilde yapılandırmak için `REDIS_HOST` ortam değişkeninizi Redis soketinin yoluna ve `REDIS_PORT` ortam değişkenini `0` olarak ayarlayın:
```
REDIS_HOST=/run/redis/redis.sock
REDIS_PORT=0
```

#### PhpRedis Serileştirme (Serialization) ve Sıkıştırma (Compression)
PhpRedis eklentisi ayrıca çeşitli serileştiriciler (serializers) ve sıkıştırma algoritmaları (compression algorithms) kullanacak şekilde yapılandırılabilir. Bu algoritmalar, Redis yapılandırmanızın `options` dizisi aracılığıyla yapılandırılabilir:
```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        'serializer' => Redis::SERIALIZER_MSGPACK,
        'compression' => Redis::COMPRESSION_LZ4,
    ],

    // ...
],
```
Şu anda desteklenen serileştiriciler şunlardır: `Redis::SERIALIZER_NONE` (varsayılan), `Redis::SERIALIZER_PHP`, `Redis::SERIALIZER_JSON`, `Redis::SERIALIZER_IGBINARY` ve `Redis::SERIALIZER_MSGPACK`.

Desteklenen sıkıştırma algoritmaları şunlardır: `Redis::COMPRESSION_NONE` (varsayılan), `Redis::COMPRESSION_LZF`, `Redis::COMPRESSION_ZSTD` ve `Redis::COMPRESSION_LZ4`.

## Redis ile Etkileşim (Interacting With Redis)

`Redis` facade'ı üzerinde çeşitli metotlar çağırarak Redis ile etkileşime geçebilirsiniz. `Redis` facade'ı dinamik metotları destekler, yani facade üzerinde herhangi bir Redis komutunu çağırabilirsiniz ve komut doğrudan Redis'e iletilecektir. Bu örnekte, `Redis` facade'ında `get` metodunu çağırarak Redis `GET` komutunu çağıracağız:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Redis;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Belirtilen kullanıcının profilini göster.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => Redis::get('user:profile:'.$id)
        ]);
    }
}
```
Yukarıda belirtildiği gibi, `Redis` facade'ında Redis'in herhangi bir komutunu çağırabilirsiniz. Laravel, komutları Redis sunucusuna iletmek için sihirli metotları (magic methods) kullanır. Bir Redis komutu argümanlar bekliyorsa, bunları facade'ın karşılık gelen metoduna iletmelisiniz:
```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```
Alternatif olarak, ilk argüman olarak komutun adını ve ikinci argüman olarak bir değerler dizisini kabul eden `Redis` facade'ının `command` metodunu kullanarak komutları sunucuya iletebilirsiniz:
```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

#### Birden Çok Redis Bağlantısı Kullanma (Using Multiple Redis Connections)
Uygulamanızın `config/database.php` yapılandırma dosyası, birden çok Redis bağlantısı / sunucusu tanımlamanıza olanak tanır. `Redis` facade'ının `connection` metodunu kullanarak belirli bir Redis bağlantısına erişebilirsiniz:
```php
$redis = Redis::connection('connection-name');
```
Varsayılan Redis bağlantısının bir örneğini elde etmek için `connection` metodunu herhangi bir ek argüman olmadan çağırabilirsiniz:
```php
$redis = Redis::connection();
```

### İşlemler (Transactions)
`Redis` facade'ının `transaction` metodu, Redis'in yerel `MULTI` ve `EXEC` komutları etrafında kullanışlı bir sarmalayıcı (wrapper) sağlar. `transaction` metodu, tek argüman olarak bir closure kabul eder. Bu closure, bir Redis bağlantı örneği alacak ve bu örneğe istediği herhangi bir komutu iletebilir. Closure içinde iletilen tüm Redis komutları, tek bir atomik işlemde (atomic transaction) yürütülecektir:
```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```
Bir Redis işlemi tanımlarken, Redis bağlantısından herhangi bir değer alamazsınız. Unutmayın, işleminiz tek bir atomik işlem olarak yürütülür ve tüm closure'unuz komutlarını yürütmeyi bitirene kadar bu işlem yürütülmez.

#### Lua Betikleri (Lua Scripts)
`eval` metodu, birden çok Redis komutunu tek bir atomik işlemde yürütmenin başka bir yolunu sağlar. Ancak, `eval` metodu, bu işlem sırasında Redis anahtar değerleriyle etkileşime geçme ve onları inceleme avantajına sahiptir. Redis betikleri Lua programlama dilinde yazılır.

`eval` metodu ilk başta biraz korkutucu olabilir, ancak havayı yumuşatmak için basit bir örnek inceleyeceğiz. `eval` metodu birkaç argüman bekler. İlk olarak, Lua betiğini (bir dize olarak) metoda iletmelisiniz. İkinci olarak, betiğin etkileşime girdiği anahtar sayısını (bir tamsayı olarak) iletmelisiniz. Üçüncü olarak, bu anahtarların adlarını iletmelisiniz. Son olarak, betiğiniz içinde erişmeniz gereken diğer ek argümanları iletebilirsiniz.

Bu örnekte, bir sayacı artıracağız, yeni değerini inceleyeceğiz ve sayacın sıfırdan küçük olması durumunda alternatif bir anahtarı artıracağız. Bu, Redis'in kendi içinde dallanma (branching) yapmamızı sağlar:
```php
use Illuminate\Support\Facades\Redis;

$script = <<<'LUA'
    local counter = redis.call('incr', KEYS[1])

    if counter < 10 then
        return counter
    else
        redis.call('set', KEYS[2], 'counter_limit_reached')
        return 'limit_reached'
    end
LUA;

$result = Redis::eval($script, 2, 'user:counter', 'user:counter:limit');
```