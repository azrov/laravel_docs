# Laravel 12 Dokümantasyonu: HTTP Yanıtları (HTTP Responses)

## Yanıt Oluşturma (Creating Responses)

#### Dizeler (Strings) ve Diziler (Arrays)
Tüm rotalar ve controller'lar, kullanıcının tarayıcısına geri gönderilmek üzere bir yanıt (response) döndürmelidir. Laravel, yanıt döndürmek için birkaç farklı yol sağlar. En temel yanıt, bir rotadan veya controller'dan bir dize (string) döndürmektir. Framework, dizeyi otomatik olarak tam bir HTTP yanıtına dönüştürecektir:
```php
Route::get('/', function () {
    return 'Hello World';
});
```
Rotalarınızdan ve controller'larınızdan dizeler döndürmenin yanı sıra, diziler (arrays) de döndürebilirsiniz. Framework, diziyi otomatik olarak bir JSON yanıtına dönüştürecektir:
```php
Route::get('/', function () {
    return [1, 2, 3];
});
```
Rotalarınızdan veya controller'larınızdan Eloquent koleksiyonları da döndürebileceğinizi biliyor muydunuz? Bunlar otomatik olarak JSON'a dönüştürülecektir. Bir deneyin!

#### Yanıt Nesneleri (Response Objects)
Tipik olarak, rota eylemlerinizden yalnızca basit dizeler veya diziler döndürmeyeceksiniz. Bunun yerine, tam `Illuminate\Http\Response` örnekleri veya görünümler (views) döndüreceksiniz.

Tam bir `Response` örneği döndürmek, yanıtın HTTP durum kodunu (status code) ve başlıklarını (headers) özelleştirmenize olanak tanır. Bir `Response` örneği, HTTP yanıtları oluşturmak için çeşitli metotlar sağlayan `Symfony\Component\HttpFoundation\Response` sınıfından miras alır (inherits):
```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

#### Eloquent Modelleri ve Koleksiyonları (Eloquent Models and Collections)
Eloquent ORM modellerini ve koleksiyonlarını doğrudan rotalarınızdan ve controller'larınızdan döndürebilirsiniz. Bunu yaptığınızda, Laravel, modelin gizli (hidden) özniteliklerine saygı göstererek modelleri ve koleksiyonları otomatik olarak JSON yanıtlarına dönüştürecektir:
```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### Yanıtlara Başlık (Header) Ekleme (Attaching Headers to Responses)
Çoğu yanıt metodunun zincirlenebilir (chainable) olduğunu unutmayın; bu, yanıt örneklerinin akıcı (fluent) bir şekilde oluşturulmasına olanak tanır. Örneğin, kullanıcıya geri göndermeden önce yanıta bir dizi başlık eklemek için `header` metodunu kullanabilirsiniz:
```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```
Veya, yanıta eklenecek bir başlıklar dizisi belirtmek için `withHeaders` metodunu kullanabilirsiniz:
```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```
Giden bir yanıttan belirli başlıkları `withoutHeader` metodunu kullanarak kaldırabilirsiniz:
```php
return response($content)->withoutHeader('X-Debug');

return response($content)->withoutHeader(['X-Debug', 'X-Powered-By']);
```

#### Önbellek Kontrol Ara Yazılımı (Cache Control Middleware)
Laravel, bir rota grubu için `Cache-Control` başlığını hızlıca ayarlamak için kullanılabilecek bir `cache.headers` ara yazılımı (middleware) içerir. Yönergeler (directives), ilgili önbellek kontrol yönergesinin "yılan_case" (snake case) karşılığı kullanılarak sağlanmalı ve noktalı virgülle ayrılmalıdır. Yönergeler listesinde `etag` belirtilmişse, yanıt içeriğinin bir MD5 hash'i otomatik olarak ETag tanımlayıcısı olarak ayarlanacaktır:
```php
Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

### Yanıtlara Çerez (Cookie) Ekleme (Attaching Cookies to Responses)
Giden bir `Illuminate\Http\Response` örneğine `cookie` metodunu kullanarak bir çerez (cookie) ekleyebilirsiniz. Bu metoda çerezin adını, değerini ve çerezin geçerli kabul edilmesi gereken dakika sayısını geçmelisiniz:
```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```
`cookie` metodu ayrıca, daha az sıklıkla kullanılan birkaç argüman daha kabul eder. Genel olarak, bu argümanlar PHP'nin yerel `setcookie` metoduna verilen argümanlarla aynı amaç ve anlama sahiptir:
```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```
Giden yanıtla birlikte bir çerezin gönderildiğinden emin olmak istiyor ancak henüz bu yanıtın bir örneğine sahip değilseniz, çerezleri yanıt gönderildiğinde eklenmek üzere "sıraya koymak" (queue) için `Cookie` facade'ını kullanabilirsiniz. `queue` metodu, bir çerez örneği oluşturmak için gereken argümanları kabul eder. Bu çerezler, tarayıcıya gönderilmeden önce giden yanıta eklenecektir:
```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

#### Çerez Örnekleri (Instances) Oluşturma
Daha sonra bir yanıt örneğine eklenebilecek bir `Symfony\Component\HttpFoundation\Cookie` örneği oluşturmak isterseniz, global `cookie` yardımcısını kullanabilirsiniz. Bu çerez, bir yanıt örneğine eklenmediği sürece istemciye geri gönderilmeyecektir:
```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

#### Çerezleri Erken Süresini Doldurma (Expiring Cookies Early)
Giden bir yanıtın `withoutCookie` metodu aracılığıyla bir çerezin süresini doldurarak (expire ederek) onu kaldırabilirsiniz:
```php
return response('Hello World')->withoutCookie('name');
Henüz giden yanıtın bir örneğine sahip değilseniz, bir çerezin süresini doldurmak için `Cookie` facade'ının `expire` metodunu kullanabilirsiniz:
```php
Cookie::expire('name');
```

### Çerezler ve Şifreleme (Cookies and Encryption)
Varsayılan olarak, `Illuminate\Cookie\Middleware\EncryptCookies` ara yazılımı sayesinde, Laravel tarafından oluşturulan tüm çerezler şifrelenir ve imzalanır; böylece istemci tarafından değiştirilemez veya okunamazlar. Uygulamanız tarafından oluşturulan bir çerez alt kümesi için şifrelemeyi devre dışı bırakmak isterseniz, uygulamanızın `bootstrap/app.php` dosyasında `encryptCookies` metodunu kullanabilirsiniz:
```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```
Genel olarak, çerez şifreleme asla devre dışı bırakılmamalıdır, çünkü bu, çerezlerinizi potansiyel istemci tarafı veri ifşasına ve kurcalamaya (tampering) maruz bırakır.

## Yönlendirmeler (Redirects)

Yönlendirme yanıtları, `Illuminate\Http\RedirectResponse` sınıfının örnekleridir ve kullanıcıyı başka bir URL'ye yönlendirmek için gereken uygun başlıkları içerir. Bir `RedirectResponse` örneği oluşturmanın birkaç yolu vardır. En basit yöntem, global `redirect` yardımcısını kullanmaktır:
```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```
Bazen, gönderilen bir form geçersiz olduğunda olduğu gibi, kullanıcıyı önceki konumlarına yönlendirmek isteyebilirsiniz. Bunu global `back` yardımcı fonksiyonunu kullanarak yapabilirsiniz. Bu özellik oturumu (session) kullandığından, `back` fonksiyonunu çağıran rotanın `web` ara yazılım grubunu kullandığından emin olun:
```php
Route::post('/user/profile', function () {
    // İsteği doğrula (Validate)...

    return back()->withInput();
});
```

### Adlandırılmış Rotalara (Named Routes) Yönlendirme (Redirecting to Named Routes)
`redirect` yardımcısını parametresiz çağırdığınızda, `Illuminate\Routing\Redirector`'un bir örneği döndürülür ve `Redirector` örneği üzerinde herhangi bir metodu çağırmanıza olanak tanır. Örneğin, adlandırılmış bir rotaya (named route) `RedirectResponse` oluşturmak için `route` metodunu kullanabilirsiniz:
```php
return redirect()->route('login');
```
Rotanızın parametreleri varsa, bunları `route` metoduna ikinci argüman olarak iletebilirsiniz:
```php
// Şu URI'ye sahip bir rota için: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

#### Parametreleri Eloquent Modelleriyle Doldurma (Populating Parameters via Eloquent Models)
Bir Eloquent modelinden doldurulan bir "ID" parametresine sahip bir rotaya yönlendirme yapıyorsanız, modelin kendisini iletebilirsiniz. ID otomatik olarak çıkarılacaktır:
```php
// Şu URI'ye sahip bir rota için: /profile/{id}

return redirect()->route('profile', [$user]);
```
Rota parametresine yerleştirilen değeri özelleştirmek isterseniz, rota parametresi tanımında sütunu belirtebilirsiniz (`/profile/{id:slug}`) veya Eloquent modelinizdeki `getRouteKey` metodunu geçersiz kılabilirsiniz (override):
```php
/**
 * Modelin rota anahtarının (route key) değerini al.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

### Controller Eylemlerine (Actions) Yönlendirme (Redirecting to Controller Actions)
Ayrıca controller eylemlerine yönlendirmeler de oluşturabilirsiniz. Bunu yapmak için, controller ve eylem adını `action` metoduna iletin:
```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```
Controller rotanız parametreler gerektiriyorsa, bunları `action` metoduna ikinci argüman olarak iletebilirsiniz:
```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

### Harici Alan Adlarına (External Domains) Yönlendirme (Redirecting to External Domains)
Bazen uygulamanızın dışındaki bir alan adına yönlendirme yapmanız gerekebilir. Bunu, herhangi bir ek URL kodlaması, doğrulama veya denetim olmaksızın bir `RedirectResponse` oluşturan `away` metodunu çağırarak yapabilirsiniz:
```php
return redirect()->away('https://www.google.com');
```

### Flash'lanmış Oturum Verileriyle Yönlendirme (Redirecting With Flashed Session Data)
Yeni bir URL'ye yönlendirme yapmak ve oturuma veri flash'lamak genellikle aynı anda yapılır. Tipik olarak bu, bir başarı mesajını oturuma flash'ladığınızda bir eylemi başarıyla gerçekleştirdikten sonra yapılır. Kolaylık olması açısından, tek bir akıcı (fluent) metot zincirinde bir `RedirectResponse` örneği oluşturabilir ve oturuma veri flash'layabilirsiniz:
```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```
Kullanıcı yönlendirildikten sonra, flash'lanmış mesajı oturumdan görüntüleyebilirsiniz. Örneğin, Blade sözdizimini kullanarak:
```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

#### Girdiyle (Input) Yönlendirme (Redirecting With Input)
Kullanıcıyı yeni bir konuma yönlendirmeden önce geçerli isteğin girdi verilerini oturuma flash'lamak için `RedirectResponse` örneği tarafından sağlanan `withInput` metodunu kullanabilirsiniz. Bu, genellikle kullanıcı bir doğrulama (validation) hatasıyla karşılaştıysa yapılır. Girdi oturuma flash'landıktan sonra, formu yeniden doldurmak için bir sonraki istek sırasında onu kolayca alabilirsiniz:
```php
return back()->withInput();
```

## Diğer Yanıt Türleri (Other Response Types)

`response` yardımcısı, diğer yanıt türlerinin örneklerini oluşturmak için kullanılabilir. `response` yardımcısı argümansız çağrıldığında, `Illuminate\Contracts\Routing\ResponseFactory` sözleşmesinin (contract) bir gerçekleştirimi (implementation) döndürülür. Bu sözleşme, yanıtlar oluşturmak için birkaç yararlı metot sağlar.

### Görünüm (View) Yanıtları (View Responses)
Yanıtın durumu ve başlıkları üzerinde kontrole ihtiyacınız varsa ancak yanıtın içeriği olarak bir görünüm (view) döndürmeniz gerekiyorsa, `view` metodunu kullanmalısınız:
```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```
Elbette, özel bir HTTP durum kodu veya özel başlıklar iletmeniz gerekmiyorsa, global `view` yardımcı fonksiyonunu kullanabilirsiniz.

### JSON Yanıtları (JSON Responses)
`json` metodu otomatik olarak `Content-Type` başlığını `application/json` olarak ayarlayacak ve verilen diziyi `json_encode` PHP fonksiyonunu kullanarak JSON'a dönüştürecektir:
```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```
Bir JSONP yanıtı oluşturmak isterseniz, `json` metodunu `withCallback` metoduyla birlikte kullanabilirsiniz:
```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

### Dosya İndirme (File Downloads)
`download` metodu, kullanıcının tarayıcısını verilen yoldaki dosyayı indirmeye zorlayan bir yanıt oluşturmak için kullanılabilir. `download` metodu, ikinci argüman olarak bir dosya adı kabul eder; bu, dosyayı indiren kullanıcı tarafından görülecek dosya adını belirleyecektir. Son olarak, metoda üçüncü argüman olarak bir HTTP başlıkları dizisi iletebilirsiniz:
```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```
Dosya indirmelerini yöneten Symfony HttpFoundation, indirilen dosyanın bir ASCII dosya adına sahip olmasını gerektirir.

### Dosya (File) Yanıtları (File Responses)
`file` metodu, bir indirme başlatmak yerine bir dosyayı (örneğin bir resim veya PDF) doğrudan kullanıcının tarayıcısında görüntülemek için kullanılabilir. Bu metod, ilk argüman olarak dosyanın mutlak yolunu (absolute path) ve ikinci argüman olarak bir başlıklar dizisini kabul eder:
```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## Akışla Yanıtlar (Streamed Responses)

Verileri oluşturuldukça istemciye akışla göndererek (streaming), bellek kullanımını önemli ölçüde azaltabilir ve özellikle çok büyük yanıtlar için performansı artırabilirsiniz. Akışla yanıtlar, sunucu göndermeyi bitirmeden önce istemcinin verileri işlemeye başlamasına olanak tanır:
```php
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // Parçalar arasındaki gecikmeyi simüle et...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```
Kolaylık olması açısından, `stream` metoduna sağladığınız kapatma (closure) bir `Generator` döndürürse, Laravel otomatik olarak oluşturucu tarafından döndürülen dizeler arasında çıktı arabelleğini (output buffer) temizleyecek ve Nginx çıktı arabelleğini devre dışı bırakacaktır:
```php
Route::post('/chat', function () {
    return response()->stream(function (): Generator {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

### Akışla Yanıtları Tüketme (Consuming Streamed Responses)
Akışla yanıtlar, Laravel yanıt ve olay akışlarıyla etkileşim için kullanışlı bir API sağlayan Laravel'in `stream` npm paketi kullanılarak tüketilebilir. Başlamak için `@laravel/stream-react` veya `@laravel/stream-vue` paketini kurun:
```bash
npm install @laravel/stream-react
```
```bash
npm install @laravel/stream-vue
```
Ardından, olay akışını tüketmek için `useStream` kullanılabilir. Akış URL'nizi sağladıktan sonra, Laravel uygulamanızdan içerik döndükçe hook, birleştirilmiş yanıtla birlikte `data`'yı otomatik olarak güncelleyecektir:
```jsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");

    const sendMessage = () => {
        send({
            message: `Current timestamp: ${Date.now()}`,
        });
    };

    return (
        <div>
            <div>{data}</div>
            <button onClick={sendMessage} disabled={isStreaming}>
                Gönder
            </button>
            {isStreaming && <div>Akışlanıyor...</div>}
        </div>
    );
}
```