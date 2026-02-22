# Laravel 12 Dokümantasyonu: HTTP İstemcisi (HTTP Client)

## Giriş

Laravel, Guzzle HTTP istemcisi etrafında anlamlı, minimal bir API sağlayarak diğer web uygulamalarıyla iletişim kurmak için hızlıca giden HTTP istekleri (outgoing HTTP requests) yapmanıza olanak tanır. Laravel'in Guzzle etrafındaki sarmalayıcısı (wrapper), en yaygın kullanım durumlarına ve harika bir geliştirici deneyimine odaklanmıştır.

## İstek Yapma (Making Requests)

İstek yapmak için `Http` facade'ı tarafından sağlanan `head`, `get`, `post`, `put`, `patch` ve `delete` metotlarını kullanabilirsiniz. İlk olarak, başka bir URL'ye nasıl basit bir `GET` isteği yapılacağını inceleyelim:
```php
use Illuminate\Support\Facades\Http;

$response = Http::get('http://example.com');
```
`get` metodu, yanıtı incelemek için kullanılabilecek çeşitli metotlar sağlayan bir `Illuminate\Http\Client\Response` örneği döndürür:
```php
$response->body() : string;
$response->json($key = null, $default = null) : mixed;
$response->object() : object;
$response->collect($key = null) : Illuminate\Support\Collection;
$response->resource() : resource;
$response->status() : int;
$response->successful() : bool;
$response->redirect(): bool;
$response->failed() : bool;
$response->clientError() : bool;
$response->serverError() : bool;
$response->header($header) : string;
$response->headers() : array;
```
`Illuminate\Http\Client\Response` nesnesi ayrıca PHP'nin `ArrayAccess` arayüzünü (interface) uygular, böylece JSON yanıt verilerine doğrudan yanıt üzerinden erişebilirsiniz:
```php
return Http::get('http://example.com/users/1')['name'];
```
Yukarıda listelenen yanıt metotlarına ek olarak, aşağıdaki metotlar yanıtın belirli bir durum koduna (status code) sahip olup olmadığını belirlemek için kullanılabilir:
```php
$response->ok() : bool;                  // 200 OK
$response->created() : bool;             // 201 Created
$response->accepted() : bool;            // 202 Accepted
$response->noContent() : bool;           // 204 No Content
$response->movedPermanently() : bool;    // 301 Moved Permanently
$response->found() : bool;               // 302 Found
$response->badRequest() : bool;          // 400 Bad Request
$response->unauthorized() : bool;        // 401 Unauthorized
$response->paymentRequired() : bool;     // 402 Payment Required
$response->forbidden() : bool;           // 403 Forbidden
$response->notFound() : bool;            // 404 Not Found
$response->requestTimeout() : bool;      // 408 Request Timeout
$response->conflict() : bool;            // 409 Conflict
$response->unprocessableEntity() : bool; // 422 Unprocessable Entity
$response->tooManyRequests() : bool;     // 429 Too Many Requests
$response->serverError() : bool;         // 500 Internal Server Error
```

#### URI Şablonları (URI Templates)
HTTP istemcisi ayrıca URI şablonu spesifikasyonunu (URI template specification) kullanarak istek URL'leri oluşturmanıza olanak tanır. URI şablonunuz tarafından genişletilebilecek URL parametrelerini tanımlamak için `withUrlParameters` metodunu kullanabilirsiniz:
```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '12.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

#### İstekleri Dökme (Dumping Requests)
Giden istek örneğini (outgoing request instance) gönderilmeden önce dökmek (dump) ve betiğin yürütülmesini sonlandırmak isterseniz, istek tanımınızın başına `dd` metodunu ekleyebilirsiniz:
```php
return Http::dd()->get('http://example.com');
```

### İstek Verileri (Request Data)
`POST`, `PUT` ve `PATCH` istekleri yaparken isteğinizle birlikte ek veri göndermek yaygındır, bu nedenle bu metotlar ikinci argüman olarak bir veri dizisi kabul eder. Varsayılan olarak, veriler `application/json` içerik türü (content type) kullanılarak gönderilecektir:
```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://example.com/users', [
    'name' => 'Steve',
    'role' => 'Network Administrator',
]);
```

#### GET İsteği Sorgu Parametreleri (GET Request Query Parameters)
`GET` istekleri yaparken, sorgu dizesini (query string) doğrudan URL'ye ekleyebilir veya `get` metoduna ikinci argüman olarak bir anahtar/değer çiftleri dizisi iletebilirsiniz:
```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);
```
Alternatif olarak, `withQueryParameters` metodu kullanılabilir:
```php
Http::retry(3, 100)->withQueryParameters([
    'name' => 'Taylor',
    'page' => 1,
])->get('http://example.com/users');
```

#### Form URL Kodlu İstekler Gönderme (Sending Form URL Encoded Requests)
Verileri `application/x-www-form-urlencoded` içerik türünü kullanarak göndermek isterseniz, isteğinizi yapmadan önce `asForm` metodunu çağırmalısınız:
```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

#### Ham İstek Gövdesi Gönderme (Sending a Raw Request Body)
Bir istek yaparken ham bir istek gövdesi (raw request body) sağlamak isterseniz, `withBody` metodunu kullanabilirsiniz. İçerik türü, metodun ikinci argümanı aracılığıyla sağlanabilir:
```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

#### Çok Parçalı (Multi-Part) İstekler
Dosyaları çok parçalı istekler olarak göndermek isterseniz, isteğinizi yapmadan önce `attach` metodunu çağırmalısınız. Bu metod, dosyanın adını ve içeriğini kabul eder. Gerekirse, dosyanın dosya adı olarak kabul edilecek üçüncü bir argüman sağlayabilirken, dosyayla ilişkili başlıkları sağlamak için dördüncü bir argüman kullanılabilir:
```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```
Bir dosyanın ham içeriğini iletmek yerine, bir akış kaynağı (stream resource) iletebilirsiniz:
```php
$photo = fopen('photo.jpg', 'r');

$response = Http::attach(
    'attachment', $photo, 'photo.jpg'
)->post('http://example.com/attachments');
```

### Başlıklar (Headers)
`withHeaders` metodu kullanılarak isteklere başlıklar (headers) eklenebilir. Bu `withHeaders` metodu bir anahtar/değer çiftleri dizisi kabul eder:
```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```
`accept` metodunu kullanarak, uygulamanızın isteğinize yanıt olarak beklediği içerik türünü belirtebilirsiniz:
```php
$response = Http::accept('application/json')->get('http://example.com/users');
```
Kolaylık olması açısından, uygulamanızın isteğinize yanıt olarak `application/json` içerik türünü beklediğini hızlıca belirtmek için `acceptJson` metodunu kullanabilirsiniz:
```php
$response = Http::acceptJson()->get('http://example.com/users');
```
`withHeaders` metodu, yeni başlıkları isteğin mevcut başlıklarıyla birleştirir. Gerekirse, `replaceHeaders` metodunu kullanarak tüm başlıkları tamamen değiştirebilirsiniz:
```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

### Kimlik Doğrulama (Authentication)
Temel (basic) ve özet (digest) kimlik doğrulama bilgilerini sırasıyla `withBasicAuth` ve `withDigestAuth` metotlarını kullanarak belirtebilirsiniz:
```php
// Temel kimlik doğrulama...
$response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

// Özet kimlik doğrulama...
$response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);
```

#### Taşıyıcı Token'lar (Bearer Tokens)
İsteğin `Authorization` başlığına hızlıca bir taşıyıcı token (bearer token) eklemek isterseniz, `withToken` metodunu kullanabilirsiniz:
```php
$response = Http::withToken('token')->post(/* ... */);
```

### Zaman Aşımı (Timeout)
`timeout` metodu, bir yanıt için beklenecek maksimum saniye sayısını belirtmek için kullanılabilir. Varsayılan olarak, HTTP istemcisi 30 saniye sonra zaman aşımına uğrar (timeout):
```php
$response = Http::timeout(3)->get(/* ... */);
```
Verilen zaman aşımı aşılırsa, bir `Illuminate\Http\Client\ConnectionException` örneği fırlatılacaktır.

Bir sunucuya bağlanmaya çalışırken beklenecek maksimum saniye sayısını `connectTimeout` metodunu kullanarak belirtebilirsiniz. Varsayılan değer 10 saniyedir:
```php
$response = Http::connectTimeout(3)->get(/* ... */);
```

### Yeniden Denemeler (Retries)
HTTP istemcisinin, bir istemci veya sunucu hatası oluşursa isteği otomatik olarak yeniden denemesini isterseniz, `retry` metodunu kullanabilirsiniz. `retry` metodu, isteğin denenmesi gereken maksimum sayıyı ve Laravel'in denemeler arasında beklemesi gereken milisaniye sayısını kabul eder:
```php
$response = Http::retry(3, 100)->post(/* ... */);
```
Denemeler arasında beklenecek milisaniye sayısını manuel olarak hesaplamak isterseniz, `retry` metoduna ikinci argüman olarak bir closure iletebilirsiniz:
```php
use Exception;

$response = Http::retry(3, function (int $attempt, Exception $exception) {
    return $attempt * 100;
})->post(/* ... */);
```
Kolaylık olması açısından, `retry` metoduna ilk argüman olarak bir dizi de sağlayabilirsiniz. Bu dizi, sonraki denemeler arasında kaç milisaniye bekleneceğini belirlemek için kullanılacaktır:
```php
$response = Http::retry([100, 200])->post(/* ... */);
```
Gerekirse, `retry` metoduna üçüncü bir argüman iletebilirsiniz. Üçüncü argüman, yeniden denemelerin gerçekten denenip denenmeyeceğini belirleyen çağrılabilir bir değer (callable) olmalıdır. Örneğin, yalnızca ilk istek bir `ConnectionException` ile karşılaşırsa isteği yeniden denemek isteyebilirsiniz:
```php
use Exception;
use Illuminate\Http\Client\ConnectionException;

$response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
    return $exception instanceof ConnectionException;
})->post(/* ... */);
```
Bir istek denemesi başarısız olursa, yeni bir deneme yapılmadan önce istekte bir değişiklik yapmak isteyebilirsiniz. Bunu, `retry` metoduna sağladığınız çağrılabilirin `request` argümanını değiştirerek başarabilirsiniz. Örneğin, ilk deneme bir kimlik doğrulama hatası döndürdüyse, isteği yeni bir yetkilendirme token'ı ile yeniden denemek isteyebilirsiniz:
```php
use Exception;
use Illuminate\Http\Client\PendingRequest;
use Illuminate\Http\Client\RequestException;

$response = Http::withToken($this->getToken())->retry(2, 0, function (Exception $exception, PendingRequest $request) {
    if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
        return false;
    }

    $request->withToken($this->getNewToken());

    return true;
})->post(/* ... */);
```
Tüm istekler başarısız olursa, bir `Illuminate\Http\Client\RequestException` örneği fırlatılacaktır. Bu davranışı devre dışı bırakmak isterseniz, `throw` argümanını `false` değeriyle sağlayabilirsiniz. Devre dışı bırakıldığında, tüm yeniden denemeler denendikten sonra istemci tarafından alınan son yanıt döndürülecektir:
```php
$response = Http::retry(3, 100, throw: false)->post(/* ... */);
```
Tüm istekler bir bağlantı sorunu nedeniyle başarısız olursa, `throw` argümanı `false` olarak ayarlanmış olsa bile yine de bir `Illuminate\Http\Client\ConnectionException` fırlatılacaktır.

### Hata Yönetimi (Error Handling)
Guzzle'ın varsayılan davranışının aksine, Laravel'in HTTP istemci sarmalayıcısı, istemci veya sunucu hatalarında (sunuculardan 400 ve 500 seviye yanıtlar) istisna (exception) fırlatmaz. Bu hatalardan birinin döndürülüp döndürülmediğini `successful`, `clientError` veya `serverError` metotlarını kullanarak belirleyebilirsiniz:
```php
// Durum kodunun >= 200 ve < 300 olup olmadığını belirle...
$response->successful();

// Durum kodunun >= 400 olup olmadığını belirle...
$response->failed();

// Yanıtın 400 seviye bir durum koduna sahip olup olmadığını belirle...
$response->clientError();

// Yanıtın 500 seviye bir durum koduna sahip olup olmadığını belirle...
$response->serverError();

// Bir istemci veya sunucu hatası varsa verilen geri çağrıyı hemen yürüt...
$response->onError(callable $callback);
```

#### İstisna Fırlatma (Throwing Exceptions)
Bir yanıt örneğiniz varsa ve yanıt durum kodu bir istemci veya sunucu hatası gösteriyorsa bir `Illuminate\Http\Client\RequestException` örneği fırlatmak isterseniz, `throw` veya `throwIf` metotlarını kullanabilirsiniz:
```php
use Illuminate\Http\Client\Response;

$response = Http::post(/* ... */);

// Bir istemci veya sunucu hatası oluşursa bir istisna fırlat...
$response->throw();

// Bir hata oluşursa ve verilen koşul doğruysa bir istisna fırlat...
$response->throwIf($condition);

// Bir hata oluşursa ve verilen closure true olarak çözümlenirse bir istisna fırlat...
$response->throwIf(fn (Response $response) => true);

// Bir hata oluşursa ve verilen koşul yanlışsa bir istisna fırlat...
$response->throwUnless($condition);

// Bir hata oluşursa ve verilen closure false olarak çözümlenirse bir istisna fırlat...
$response->throwUnless(fn (Response $response) => false);

// Yanıt belirli bir durum koduna sahipse bir istisna fırlat...
$response->throwIfStatus(403);

// Yanıt belirli bir durum koduna sahip değilse bir istisna fırlat...
$response->throwUnlessStatus(200);

return $response['user']['id'];
```
`Illuminate\Http\Client\RequestException` örneği, döndürülen yanıtı incelemenize izin verecek bir `$response` özelliğine sahiptir.

`throw` metodu, herhangi bir hata oluşmadıysa yanıt örneğini döndürür, böylece `throw` metoduna başka işlemler zincirleyebilirsiniz:
```php
return Http::post(/* ... */)->throw()->json();
```
İstisna fırlatılmadan önce bazı ek mantıklar gerçekleştirmek isterseniz, `throw` metoduna bir closure iletebilirsiniz. Closure çağrıldıktan sonra istisna otomatik olarak fırlatılacaktır, bu nedenle closure içinden istisnayı yeniden fırlatmanıza gerek yoktur:
```php
use Illuminate\Http\Client\Response;
use Illuminate\Http\Client\RequestException;

return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
    // ...
})->json();
```
Varsayılan olarak, `RequestException` mesajları günlüğe kaydedildiğinde veya raporlandığında 120 karaktere kısaltılır (truncated). Bu davranışı özelleştirmek veya devre dışı bırakmak için, uygulamanızın `bootstrap/app.php` dosyasında kayıtlı davranışını yapılandırırken `truncateAt` ve `dontTruncate` metotlarını kullanabilirsiniz:
```php
use Illuminate\Http\Client\RequestException;

->registered(function (): void {
    // İstek istisna mesajlarını 240 karaktere kısalt...
    RequestException::truncateAt(240);

    // İstek istisna mesajı kısaltmasını devre dışı bırak...
    RequestException::dontTruncate();
})
```
Alternatif olarak, `truncateExceptionsAt` metodunu kullanarak istek başına istisna kısaltma davranışını özelleştirebilirsiniz:
```php
return Http::truncateExceptionsAt(240)->post(/* ... */);
```

### Guzzle Ara Yazılımları (Guzzle Middleware)
Laravel'in HTTP istemcisi Guzzle tarafından desteklendiğinden, giden isteği işlemek veya gelen yanıtı incelemek için Guzzle Middleware'lerinden yararlanabilirsiniz. Bir isteğe Guzzle middleware'i eklemek için `withMiddleware` metodunu kullanabilirsiniz. Her middleware, bir istek gönderilmeden önce işleme fırsatı veren ve kabul edilen `callable`'ı kabul eden `GuzzleHttp\Middleware::mapRequest` ve `GuzzleHttp\Middleware::mapResponse` metotlarını kullanabilirsiniz:
```php
use GuzzleHttp\Middleware;
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

$response = Http::withMiddleware(
    Middleware::mapRequest(function (RequestInterface $request) {
        $request = $request->withHeader('X-Example', 'Value');
 
        return $request;
    })
)->withMiddleware(
    Middleware::mapResponse(function (ResponseInterface $response) {
        $response = $response->withHeader('X-Example', 'Value');
 
        return $response;
    })
)->get('http://example.com');
```