# Laravel 12 Dokümantasyonu: URL Oluşturma (URL Generation)

## Giriş

Laravel, uygulamanız için URL'ler oluşturmanıza yardımcı olacak birkaç yardımcı (helper) sağlar. Bu yardımcılar, öncelikle şablonlarınızda (templates) ve API yanıtlarınızda bağlantılar oluştururken veya uygulamanızın başka bir bölümüne yönlendirme (redirect) yanıtları oluştururken faydalıdır.

## Temel Bilgiler (The Basics)

### URL Oluşturma (Generating URLs)
`url` yardımcısı, uygulamanız için isteğe bağlı (arbitrary) URL'ler oluşturmak için kullanılabilir. Oluşturulan URL, otomatik olarak uygulama tarafından işlenmekte olan geçerli isteğin şemasını (scheme - HTTP veya HTTPS) ve ana bilgisayarını (host) kullanacaktır:
```php
$post = App\Models\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```
Sorgu dizesi parametreleriyle (query string parameters) bir URL oluşturmak için `query` metodunu kullanabilirsiniz:
```php
echo url()->query('/posts', ['search' => 'Laravel']);

// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

// http://example.com/posts?sort=latest&search=Laravel
```
Yol (path) içinde zaten mevcut olan sorgu dizesi parametrelerini sağlamak, onların mevcut değerlerinin üzerine yazacaktır (overwrite):
```php
echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

// http://example.com/posts?sort=oldest
```
Değer dizileri (arrays of values) de sorgu parametreleri olarak iletilebilir. Bu değerler, oluşturulan URL'de uygun şekilde anahtarlanacak (keyed) ve kodlanacaktır (encoded):
```php
echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

// http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

echo urldecode($url);

// http://example.com/posts?columns[0]=title&columns[1]=body
```

### Geçerli URL'ye Erişim (Accessing the Current URL)
`url` yardımcısına hiçbir yol (path) sağlanmazsa, `Illuminate\Routing\UrlGenerator`'ın bir örneği döndürülür ve geçerli URL hakkındaki bilgilere erişmenize olanak tanır:
```php
// Sorgu dizesi olmadan geçerli URL'yi al...
echo url()->current();

// Sorgu dizesi dahil geçerli URL'yi al...
echo url()->full();
```
Bu metotların her birine `URL` facade'ı aracılığıyla da erişilebilir:
```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

#### Önceki URL'ye Erişim (Accessing the Previous URL)
Bazen, kullanıcının geldiği önceki URL'yi (previous URL) bilmek faydalı olabilir. Önceki URL'ye `url` yardımcısının `previous` ve `previousPath` metotları aracılığıyla erişebilirsiniz:
```php
// Önceki isteğin tam URL'sini al...
echo url()->previous();

// Önceki isteğin yolunu (path) al...
echo url()->previousPath();
```
Veya oturum (session) aracılığıyla, önceki URL'ye akıcı bir URI örneği (fluent URI instance) olarak erişebilirsiniz:
```php
use Illuminate\Http\Request;

Route::post('/users', function (Request $request) {
    $previousUri = $request->session()->previousUri();

    // ...
});
```
Daha önce ziyaret edilen URL'nin rota adını (route name) oturum aracılığıyla almak da mümkündür:
```php
$previousRoute = $request->session()->previousRoute();
```

## Adlandırılmış Rotalar için URL'ler (URLs for Named Routes)

`route` yardımcısı, adlandırılmış rotalara (named routes) URL oluşturmak için kullanılabilir. Adlandırılmış rotalar, rotada tanımlanan gerçek URL'ye bağlı kalmadan URL oluşturmanıza olanak tanır. Bu nedenle, rotanın URL'si değişirse, `route` fonksiyonuna yaptığınız çağrılarda herhangi bir değişiklik yapılması gerekmez. Örneğin, uygulamanızın aşağıdaki gibi tanımlanmış bir rota içerdiğini hayal edin:
```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```
Bu rotaya bir URL oluşturmak için `route` yardımcısını şu şekilde kullanabilirsiniz:
```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```
Elbette, `route` yardımcısı birden çok parametreye sahip rotalar için URL oluşturmak için de kullanılabilir:
```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```
Rotanın tanım parametrelerine karşılık gelmeyen ek dizi öğeleri, URL'nin sorgu dizesine eklenecektir:
```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

#### Eloquent Modelleri (Eloquent Models)
Genellikle Eloquent modellerinin rota anahtarını (route key - tipik olarak birincil anahtar) kullanarak URL'ler oluşturacaksınız. Bu nedenle, Eloquent modellerini parametre değerleri olarak iletebilirsiniz. `route` yardımcısı otomatik olarak modelin rota anahtarını çıkaracaktır:
```php
echo route('post.show', ['post' => $post]);
```

### İmzalı URL'ler (Signed URLs)
Laravel, adlandırılmış rotalara kolayca "imzalı" (signed) URL'ler oluşturmanıza olanak tanır. Bu URL'ler, sorgu dizesine eklenmiş bir "imza" (signature) hash'ine sahiptir; bu, Laravel'in URL'nin oluşturulduğundan bu yana değiştirilmediğini doğrulamasına olanak tanır. İmzalı URL'ler, özellikle herkese açık (publicly accessible) olan ancak URL manipülasyonuna karşı bir koruma katmanına ihtiyaç duyan rotalar için kullanışlıdır.

Örneğin, müşterilerinize e-posta ile gönderilen bir "abonelikten çık" (unsubscribe) bağlantısını uygulamak için imzalı URL'ler kullanabilirsiniz. Adlandırılmış bir rotaya imzalı bir URL oluşturmak için `URL` facade'ının `signedRoute` metodunu kullanın:
```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```
`signedRoute` metoduna `absolute` argümanını sağlayarak alan adını (domain) imzalı URL hash'inden hariç tutabilirsiniz:
```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```
Belirli bir süre sonra sona eren geçici bir imzalı rota URL'si oluşturmak isterseniz, `temporarySignedRoute` metodunu kullanabilirsiniz. Laravel geçici bir imzalı rota URL'sini doğrularken, imzalı URL'ye kodlanmış son kullanma zaman damgasının (expiration timestamp) geçmediğinden emin olacaktır:
```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

#### İmzalı Rota İsteklerini Doğrulama (Validating Signed Route Requests)
Gelen bir isteğin geçerli bir imzaya sahip olup olmadığını doğrulamak için, gelen `Illuminate\Http\Request` örneği üzerinde `hasValidSignature` metodunu çağırmalısınız:
```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```
Bazen, uygulamanızın ön yüzünün (frontend), istemci tarafı sayfalandırma (client-side pagination) yaparken olduğu gibi, imzalı bir URL'ye veri eklemesine izin vermeniz gerekebilir. Bu nedenle, `hasValidSignatureWhileIgnoring` metodunu kullanarak imzalı bir URL'yi doğrularken yok sayılması gereken istek sorgu parametrelerini belirtebilirsiniz. Parametreleri yok saymanın, herkesin istek üzerinde bu parametreleri değiştirmesine izin verdiğini unutmayın:
```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```
İmzalı URL'leri gelen istek örneğini kullanarak doğrulamak yerine, `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) ara yazılımını rotaya atayabilirsiniz. Gelen istek geçerli bir imzaya sahip değilse, ara yazılım otomatik olarak bir 403 HTTP yanıtı döndürecektir:
```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```
İmzalı URL'leriniz URL hash'inde alan adını içermiyorsa, ara yazılıma `relative` argümanını sağlamalısınız:
```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

#### Geçersiz İmzalı Rotalara Yanıt Verme (Responding to Invalid Signed Routes)
Birisi süresi dolmuş (expired) bir imzalı URL'yi ziyaret ettiğinde, 403 HTTP durum kodu için genel bir hata sayfası alacaktır. Ancak, uygulamanızın `bootstrap/app.php` dosyasında `InvalidSignatureException` istisnası için özel bir "render" closure'ı tanımlayarak bu davranışı özelleştirebilirsiniz:
```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

## Controller Eylemleri için URL'ler (URLs for Controller Actions)

`action` fonksiyonu, verilen controller eylemi (controller action) için bir URL oluşturur:
```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```
Controller metodu rota parametreleri kabul ediyorsa, fonksiyona ikinci argüman olarak rota parametrelerinin ilişkisel bir dizisini (associative array) iletebilirsiniz:
```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

## Akıcı URI Nesneleri (Fluent URI Objects)

Laravel'in `Uri` sınıfı, URI'leri nesneler aracılığıyla oluşturmak ve değiştirmek için kullanışlı ve akıcı bir arayüz (fluent interface) sağlar. Bu sınıf, temeldeki League URI paketi tarafından sağlanan işlevselliği sarar (wrap) ve Laravel'in yönlendirme sistemiyle sorunsuz bir şekilde entegre olur.

Statik metotları kullanarak kolayca bir `Uri` örneği oluşturabilirsiniz:
```php
use App\Http\Controllers\UserController;
use App\Http\Controllers\InvokableController;
use Illuminate\Support\Uri;

// Verilen dizeden bir URI örneği oluştur...
$uri = Uri::of('https://example.com/path');

// Yollara (paths), adlandırılmış rotalara veya controller eylemlerine URI örnekleri oluştur...
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::temporarySignedRoute('user.index', now()->addMinutes(5));
$uri = Uri::action([UserController::class, 'index']);
$uri = Uri::action(InvokableController::class);

// Geçerli istek URL'sinden bir URI örneği oluştur...
$uri = $request->uri();

// Önceki istek URL'sinden bir URI örneği oluştur...
$uri = $request->session()->previousUri();
```
Bir URI örneğine sahip olduğunuzda, onu akıcı bir şekilde değiştirebilirsiniz:
```php
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```
Akıcı URI nesneleriyle çalışma hakkında daha fazla bilgi için URI dokümantasyonuna bakın.

## Varsayılan Değerler (Default Values)

Bazı uygulamalar için, belirli URL parametreleri için istek genelinde (request-wide) varsayılan değerler belirtmek isteyebilirsiniz. Örneğin, rotalarınızın çoğunun bir `{locale}` parametresi tanımladığını düşünün:
```php
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```
`route` yardımcısını her çağırdığınızda `locale` parametresini iletmek zahmetlidir. Bu nedenle, geçerli istek sırasında her zaman uygulanacak olan bu parametre için bir varsayılan değer tanımlamak üzere `URL::defaults` metodunu kullanabilirsiniz. Geçerli isteğe erişebilmeniz için bu metodu bir rota ara yazılımından (route middleware) çağırmak isteyebilirsiniz:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;

class SetDefaultLocaleForUrls
{
    /**
     * Gelen bir isteği işle.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```
`locale` parametresi için varsayılan değer ayarlandıktan sonra, `route` yardımcısı aracılığıyla URL oluştururken değerini iletmeniz artık gerekmez.

#### URL Varsayılanları ve Ara Yazılım Önceliği (URL Defaults and Middleware Priority)
URL varsayılan değerlerini ayarlamak, Laravel'in örtülü model bağlama (implicit model bindings) işlemesine müdahale edebilir. Bu nedenle, URL varsayılanlarını ayarlayan ara yazılımınızı, Laravel'in kendi `SubstituteBindings` ara yazılımından önce yürütülecek şekilde önceliklendirmelisiniz (prioritize). Bunu, uygulamanızın `bootstrap/app.php` dosyasındaki `priority` ara yazılım metodunu kullanarak gerçekleştirebilirsiniz:
```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```