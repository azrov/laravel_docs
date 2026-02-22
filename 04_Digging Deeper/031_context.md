# Laravel 12 Dokümantasyonu: Bağlam (Context)

## Giriş

Laravel'in "bağlam" (context) yetenekleri, uygulamanızda çalışan istekler (requests), işler (jobs) ve komutlar (commands) boyunca bilgileri yakalamanıza, almanıza ve paylaşmanıza olanak tanır. Bu yakalanan bilgiler, uygulamanız tarafından yazılan günlüklere (logs) de dahil edilerek, bir günlük girdisi (log entry) yazılmadan önceki çevreleyen kod yürütme geçmişi hakkında size daha derin bir görüş sağlar ve dağıtık bir sistem (distributed system) boyunca yürütme akışlarını izlemenize (trace) olanak tanır.

### Nasıl Çalışır? (How it Works)
Laravel'in bağlam yeteneklerini anlamanın en iyi yolu, yerleşik günlük kaydı (logging) özelliklerini kullanarak onu eylemde görmektir. Başlamak için, `Context` facade'ını kullanarak bağlama bilgi ekleyebilirsiniz. Bu örnekte, bir ara yazılım (middleware) kullanarak her gelen istekte istek URL'sini ve benzersiz bir izleme kimliğini (trace ID) bağlama ekleyeceğiz:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddContext
{
    /**
     * Gelen bir isteği işle.
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());

        return $next($request);
    }
}
```
Bağlama eklenen bilgiler, istek boyunca yazılan tüm günlük girdilerine otomatik olarak meta veri (metadata) olarak eklenir. Bağlamı meta veri olarak eklemek, tek tek günlük girdilerine iletilen bilgilerin `Context` aracılığıyla paylaşılan bilgilerden ayırt edilmesini sağlar. Örneğin, aşağıdaki günlük girdisini yazdığımızı hayal edin:
```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```
Yazılan günlük, günlük girdisine iletilen `auth_id`'yi içerecek, ancak aynı zamanda bağlamın `url` ve `trace_id`'sini meta veri olarak içerecektir:
```
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```
Bağlama eklenen bilgiler, kuyruğa (queue) gönderilen işlere (jobs) de sunulur. Örneğin, bağlama biraz bilgi ekledikten sonra kuyruğa bir `ProcessPodcast` işi gönderdiğimizi hayal edin:
```php
// Ara yazılımımızda...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// Controller'ımızda...
ProcessPodcast::dispatch($podcast);
```
İş gönderildiğinde, bağlamda o anda saklanan herhangi bir bilgi yakalanır ve işle paylaşılır. Yakalanan bilgi daha sonra iş yürütülürken geçerli bağlama geri yüklenir (hydrated). Yani, işimizin `handle` metodu günlüğe yazacak olsaydı:
```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * İşi yürüt.
     */
    public function handle(): void
    {
        Log::info('Processing podcast.', [
            'podcast_id' => $this->podcast->id,
        ]);

        // ...
    }
}
```
Ortaya çıkan günlük girdisi, başlangıçta işi gönderen istek sırasında bağlama eklenen bilgileri içerecektir:
```
Processing podcast. {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```
Laravel bağlamının yerleşik günlük kaydıyla ilgili özelliklerine odaklanmış olsak da, aşağıdaki dokümantasyon, bağlamın HTTP isteği / sıraya konmuş iş sınırı boyunca bilgileri nasıl paylaşmanıza izin verdiğini ve hatta günlük girdileriyle yazılmayan gizli bağlam verilerini nasıl ekleyeceğinizi gösterecektir.

## Bağlamı Yakalama (Capturing Context)

Geçerli bağlamda bilgi saklamak için `Context` facade'ının `add` metodunu kullanabilirsiniz:
```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```
Aynı anda birden çok öğe eklemek için `add` metoduna ilişkisel bir dizi (associative array) iletebilirsiniz:
```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```
`add` metodu, aynı anahtarı paylaşan mevcut herhangi bir değeri geçersiz kılacaktır (override). Bağlama yalnızca anahtar zaten mevcut değilse bilgi eklemek isterseniz, `addIf` metodunu kullanabilirsiniz:
```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```
Bağlam ayrıca belirli bir anahtarı artırmak (increment) veya azaltmak (decrement) için kullanışlı metotlar sağlar. Bu metotların her ikisi de en az bir argüman kabul eder: izlenecek anahtar. Anahtarın artırılması veya azaltılması gereken miktarı belirtmek için ikinci bir argüman sağlanabilir:
```php
Context::increment('records_added');
Context::increment('records_added', 5);

Context::decrement('records_added');
Context::decrement('records_added', 5);
```

#### Koşullu Bağlam (Conditional Context)
`when` metodu, belirli bir koşula bağlı olarak bağlama veri eklemek için kullanılabilir. `when` metoduna sağlanan ilk closure, verilen koşul `true` olarak değerlendirilirse çağrılırken, ikinci closure koşul `false` olarak değerlendirilirse çağrılacaktır:
```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

#### Kapsamlı Bağlam (Scoped Context)
`scope` metodu, belirli bir geri çağrının (callback) yürütülmesi sırasında bağlamı geçici olarak değiştirmenin ve geri çağrı yürütmeyi bitirdiğinde bağlamı orijinal durumuna geri yüklemenin bir yolunu sağlar. Ayrıca, closure yürütülürken bağlama birleştirilmesi gereken ekstra verileri (ikinci ve üçüncü argümanlar olarak) iletebilirsiniz.
```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\Log;

Context::add('trace_id', 'abc-999');
Context::addHidden('user_id', 123);

Context::scope(
    function () {
        Context::add('action', 'adding_friend');

        $userId = Context::getHidden('user_id');

        Log::debug("Adding user [{$userId}] to friends list.");
        // Adding user [987] to friends list.  {"trace_id":"abc-999","user_name":"taylor_otwell","action":"adding_friend"}
    },
    data: ['user_name' => 'taylor_otwell'],
    hidden: ['user_id' => 987],
);

Context::all();
// [
//     'trace_id' => 'abc-999',
// ]

Context::allHidden();
// [
//     'user_id' => 123,
// ]
```
Bağlam içindeki bir nesne, kapsamlı closure içinde değiştirilirse, bu değişiklik kapsamın dışına da yansıyacaktır.

### Yığınlar (Stacks)
Bağlam, eklendikleri sırayla saklanan veri listeleri olan "yığınlar" (stacks) oluşturma yeteneği sunar. `push` metodunu çağırarak bir yığına bilgi ekleyebilirsiniz:
```php
use Illuminate\Support\Facades\Context;

Context::push('breadcrumbs', 'first_value');

Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```
Yığınlar, bir istek hakkında, uygulamanız boyunca gerçekleşen olaylar gibi tarihsel bilgileri yakalamak için kullanışlı olabilir. Örneğin, bir sorgu her yürütüldüğünde yığına ekleme yapmak, sorgu SQL'ini ve süresini bir demet (tuple) olarak yakalamak için bir olay dinleyicisi (event listener) oluşturabilirsiniz:
```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

// AppServiceProvider.php içinde...
DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```
Bir değerin yığında olup olmadığını `stackContains` ve `hiddenStackContains` metotlarını kullanarak belirleyebilirsiniz:
```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```
`stackContains` ve `hiddenStackContains` metotları ayrıca ikinci argüman olarak bir closure kabul eder, bu da değer karşılaştırma işlemi üzerinde daha fazla kontrol sağlar:
```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

## Bağlamı Alma (Retrieving Context)

`Context` facade'ının `get` metodunu kullanarak bağlamdan bilgi alabilirsiniz:
```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```
`only` ve `except` metotları, bağlamdaki bilgilerin bir alt kümesini almak için kullanılabilir:
```php
$data = Context::only(['first_key', 'second_key']);

$data = Context::except(['first_key']);
```
`pull` metodu, bağlamdan bilgi almak ve onu hemen bağlamdan kaldırmak için kullanılabilir:
```php
$value = Context::pull('key');
```
Bağlam verileri bir yığında saklanıyorsa, `pop` metodunu kullanarak yığından öğeler çıkarabilirsiniz (pop):
```php
Context::push('breadcrumbs', 'first_value', 'second_value');

Context::pop('breadcrumbs');
// second_value

Context::get('breadcrumbs');
// ['first_value']
```
`remember` ve `rememberHidden` metotları, bağlamdan bilgi almak için kullanılabilirken, istenen bilgi yoksa bağlam değerini verilen closure tarafından döndürülen değere ayarlar:
```php
$permissions = Context::remember(
    'user-permissions',
    fn () => $user->permissions,
);
```
Bağlamda saklanan tüm bilgileri almak isterseniz, `all` metodunu çağırabilirsiniz:
```php
$data = Context::all();
```

### Öğe Varlığını Belirleme (Determining Item Existence)
`has` ve `missing` metotlarını kullanarak bağlamın verilen anahtar için herhangi bir değer saklayıp saklamadığını belirleyebilirsiniz:
```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```
`has` metodu, saklanan değer ne olursa olsun `true` döndürecektir. Yani, örneğin, `null` değerine sahip bir anahtar mevcut kabul edilecektir:
```php
Context::add('key', null);

Context::has('key');
// true
```

## Bağlamı Kaldırma (Removing Context)

`forget` metodu, geçerli bağlamdan bir anahtarı ve değerini kaldırmak için kullanılabilir:
```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```
`forget` metoduna bir dizi sağlayarak aynı anda birden çok anahtarı kaldırabilirsiniz:
```php
Context::forget(['first_key', 'second_key']);
```

## Gizli Bağlam (Hidden Context)

Bağlam, "gizli" (hidden) veri saklama yeteneği sunar. Bu gizli bilgiler günlüklere eklenmez ve yukarıda belgelenen veri alma metotları aracılığıyla erişilemez. Bağlam, gizli bağlam bilgileriyle etkileşim kurmak için farklı bir metot seti sağlar:
```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```
"Gizli" metotlar, yukarıda belgelenen gizli olmayan metotların işlevselliğini yansıtır:
```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::popHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::exceptHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::missingHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

## Olaylar (Events)

Bağlam, bağlamın hidrasyon (hydration) ve dehidrasyon (dehydration) sürecine bağlanmanıza (hook) izin veren iki olay gönderir.

Bu olayların nasıl kullanılabileceğini göstermek için, uygulamanızın bir ara yazılımında `app.locale` yapılandırma değerini gelen HTTP isteğinin `Accept-Language` başlığına göre ayarladığınızı hayal edin. Bağlamın olayları, bu değeri istek sırasında yakalamanıza ve kuyrukta geri yüklemenize olanak tanır, böylece kuyruğa gönderilen bildirimlerin doğru `app.locale` değerine sahip olmasını sağlar. Bunu başarmak için bağlamın olaylarını ve gizli verilerini kullanabiliriz; aşağıdaki dokümantasyon bunu gösterecektir.

### Dehidrasyon (Dehydrating)
Kuyruğa bir iş gönderildiğinde, bağlamdaki veriler "dehidre edilir" (dehydrated) ve işin yükü (payload) ile birlikte yakalanır. `Context::dehydrating` metodu, dehidrasyon işlemi sırasında çağrılacak bir closure kaydetmenize olanak tanır. Bu closure içinde, sıraya konmuş işle paylaşılacak verilerde değişiklikler yapabilirsiniz.

Tipik olarak, `dehydrating` geri çağrılarını uygulamanızın `AppServiceProvider` sınıfının `boot` metodu içinde kaydetmelisiniz:
```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```
`dehydrating` geri çağrısı içinde `Context` facade'ını kullanmamalısınız, çünkü bu geçerli sürecin bağlamını değiştirecektir. Yalnızca geri çağrıya iletilen `Repository` üzerinde değişiklik yaptığınızdan emin olun.

### Hidrasyon (Hydrated)
Kuyrukta sıraya konmuş bir iş yürütülmeye başladığında, işle paylaşılan herhangi bir bağlam geçerli bağlama geri "hidre edilir" (hydrated). `Context::hydrated` metodu, hidrasyon işlemi sırasında çağrılacak bir closure kaydetmenize olanak tanır.

Tipik olarak, `hydrated` geri çağrılarını uygulamanızın `AppServiceProvider` sınıfının `boot` metodu içinde kaydetmelisiniz:
```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```
`hydrated` geri çağrısı içinde `Context` facade'ını kullanmamalı ve bunun yerine yalnızca geri çağrıya iletilen `Repository` üzerinde değişiklik yaptığınızdan emin olmalısınız.