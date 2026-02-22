# Laravel 12 Dokümantasyonu: HTTP İstekleri (HTTP Requests)

## Giriş

Laravel'in `Illuminate\Http\Request` sınıfı, uygulamanız tarafından işlenmekte olan geçerli HTTP isteğiyle etkileşime geçmek ve istekle birlikte gönderilen girdileri (input), çerezleri (cookies) ve dosyaları (files) almak için nesne yönelimli (object-oriented) bir yol sağlar.

## İstekle Etkileşim (Interacting With The Request)

### İsteğe Erişim (Accessing the Request)
Geçerli HTTP isteğinin bir örneğini (instance) bağımlılık enjeksiyonu (dependency injection) yoluyla elde etmek için, rota kapatmanızda (route closure) veya controller metodunuzda `Illuminate\Http\Request` sınıfını tip belirtmelisiniz (type-hint). Gelen istek örneği, Laravel servis kabı (service container) tarafından otomatik olarak enjekte edilecektir:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Yeni bir kullanıcı sakla (Store).
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Kullanıcıyı sakla...

        return redirect('/users');
    }
}
```
Bahsedildiği gibi, `Illuminate\Http\Request` sınıfını bir rota kapatmasında da tip belirtebilirsiniz. Servis kabı, yürütüldüğünde gelen isteği otomatik olarak kapatmaya enjekte edecektir:
```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

#### Bağımlılık Enjeksiyonu ve Rota Parametreleri (Dependency Injection and Route Parameters)
Controller metodunuz ayrıca bir rota parametresinden girdi bekliyorsa, rota parametrelerinizi diğer bağımlılıklarınızdan sonra listelemelisiniz. Örneğin, rotanız şu şekilde tanımlanmışsa:
```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```
Yine de `Illuminate\Http\Request` sınıfını tip belirtebilir ve `id` rota parametrenize controller metodunuzu aşağıdaki gibi tanımlayarak erişebilirsiniz:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Belirtilen kullanıcıyı güncelle.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Kullanıcıyı güncelle...

        return redirect('/users');
    }
}
```

### İstek Yolu, Ana Bilgisayar (Host) ve Metodu (Request Path, Host, and Method)
`Illuminate\Http\Request` örneği, gelen HTTP isteğini incelemek için çeşitli metotlar sağlar ve `Symfony\Component\HttpFoundation\Request` sınıfını genişletir (extend eder). Aşağıda en önemli metotlardan birkaçını tartışacağız.

#### İstek Yolunu (Path) Alma
`path` metodu, isteğin yol bilgisini döndürür. Yani, gelen istek `http://example.com/foo/bar` adresine yönlendirilmişse, `path` metodu `foo/bar` döndürecektir:
```php
$uri = $request->path();
```

#### İstek Yolunu / Rotasını İnceleme (Inspecting the Request Path / Route)
`is` metodu, gelen istek yolunun belirli bir kalıpla (pattern) eşleşip eşleşmediğini doğrulamanıza olanak tanır. Bu metodu kullanırken joker karakter olarak `*` karakterini kullanabilirsiniz:
```php
if ($request->is('admin/*')) {
    // ...
}
```
`routeIs` metodunu kullanarak, gelen isteğin adlandırılmış bir rotayla (named route) eşleşip eşleşmediğini belirleyebilirsiniz:
```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

#### İstek URL'sini Alma (Retrieving the Request URL)
Gelen isteğin tam URL'sini almak için `url` veya `fullUrl` metotlarını kullanabilirsiniz. `url` metodu URL'yi sorgu dizesi (query string) olmadan döndürürken, `fullUrl` metodu sorgu dizesini içerir:
```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```
Geçerli URL'ye sorgu dizesi verileri eklemek isterseniz, `fullUrlWithQuery` metodunu çağırabilirsiniz. Bu metod, verilen sorgu dizesi değişkenleri dizisini geçerli sorgu dizesiyle birleştirir:
```php
$request->fullUrlWithQuery(['type' => 'phone']);
```
Belirli bir sorgu dizesi parametresi olmadan geçerli URL'yi almak isterseniz, `fullUrlWithoutQuery` metodunu kullanabilirsiniz:
```php
$request->fullUrlWithoutQuery(['type']);
```

#### İstek Ana Bilgisayarını (Host) Alma
Gelen isteğin "ana bilgisayarını" (host) `host`, `httpHost` ve `schemeAndHttpHost` metotları aracılığıyla alabilirsiniz:
```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

#### İstek Metodunu (Method) Alma
`method` metodu, istek için HTTP fiilini (verb) döndürecektir. HTTP fiilinin belirli bir dizeyle eşleşip eşleşmediğini doğrulamak için `isMethod` metodunu kullanabilirsiniz:
```php
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

### İstek Başlıkları (Request Headers)
`Illuminate\Http\Request` örneğinden `header` metodunu kullanarak bir istek başlığı (request header) alabilirsiniz. Başlık istekte yoksa, `null` döndürülecektir. Ancak, `header` metodu, başlık istekte yoksa döndürülecek olan isteğe bağlı ikinci bir argüman kabul eder:
```php
$value = $request->header('X-Header-Name');

$value = $request->header('X-Header-Name', 'default');
```
`hasHeader` metodu, isteğin belirli bir başlığı içerip içermediğini belirlemek için kullanılabilir:
```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```
Kolaylık olması açısından, `bearerToken` metodu, `Authorization` başlığından bir bearer token almak için kullanılabilir. Böyle bir başlık yoksa, boş bir dize döndürülecektir:
```php
$token = $request->bearerToken();
```

### İstek IP Adresi (Request IP Address)
`ip` metodu, uygulamanıza isteği yapan istemcinin IP adresini almak için kullanılabilir:
```php
$ipAddress = $request->ip();
```
Proxy'ler tarafından iletilen tüm istemci IP adresleri de dahil olmak üzere bir IP adresleri dizisi almak isterseniz, `ips` metodunu kullanabilirsiniz. "Orijinal" istemci IP adresi dizinin sonunda olacaktır:
```php
$ipAddresses = $request->ips();
```
Genel olarak, IP adresleri güvenilmeyen, kullanıcı kontrollü girdiler olarak kabul edilmeli ve yalnızca bilgilendirme amaçlı kullanılmalıdır.

### İçerik Anlaşması (Content Negotiation)
Laravel, gelen isteğin `Accept` başlığı aracılığıyla talep edilen içerik türlerini incelemek için çeşitli metotlar sağlar. İlk olarak, `getAcceptableContentTypes` metodu, istek tarafından kabul edilen tüm içerik türlerini içeren bir dizi döndürecektir:
```php
$contentTypes = $request->getAcceptableContentTypes();
```
`accepts` metodu bir içerik türleri dizisi kabul eder ve bu içerik türlerinden herhangi biri istek tarafından kabul ediliyorsa `true` döndürür. Aksi takdirde, `false` döndürülecektir:
```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```
Verilen bir içerik türleri dizisinden hangi içerik türünün istek tarafından en çok tercih edildiğini belirlemek için `prefers` metodunu kullanabilirsiniz. Sağlanan içerik türlerinden hiçbiri istek tarafından kabul edilmiyorsa, `null` döndürülecektir:
```php
$preferred = $request->prefers(['text/html', 'application/json']);
```
Birçok uygulama yalnızca HTML veya JSON sunduğundan, gelen isteğin bir JSON yanıtı bekleyip beklemediğini hızlıca belirlemek için `expectsJson` metodunu kullanabilirsiniz:
```php
if ($request->expectsJson()) {
    // ...
}
```

### PSR-7 İstekleri (PSR-7 Requests)
PSR-7 standardı, istekler ve yanıtlar dahil olmak üzere HTTP mesajları için arayüzler (interfaces) belirtir. Bir Laravel isteği yerine bir PSR-7 isteğinin örneğini almak isterseniz, önce birkaç kütüphane kurmanız gerekecektir. Laravel, tipik Laravel isteklerini ve yanıtlarını PSR-7 uyumlu gerçekleştirimlere (implementations) dönüştürmek için Symfony HTTP Message Bridge bileşenini kullanır:
```bash
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```
Bu kütüphaneleri yükledikten sonra, rota kapatmanızda veya controller metodunuzda istek arayüzünü tip belirterek bir PSR-7 isteği elde edebilirsiniz:
```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```
Bir rota veya controller'dan bir PSR-7 yanıt örneği döndürürseniz, otomatik olarak bir Laravel yanıt örneğine geri dönüştürülecek ve framework tarafından görüntülenecektir.

## Girdi (Input)

### Girdi Alma (Retrieving Input)

#### Tüm Girdi Verilerini Alma (Retrieving All Input Data)
Gelen isteğin tüm girdi verilerini bir `dizi` (array) olarak `all` metodunu kullanarak alabilirsiniz. Bu metod, gelen isteğin bir HTML formundan mı yoksa bir XHR isteğinden mi olduğuna bakılmaksızın kullanılabilir:
```php
$input = $request->all();
```
`collect` metodunu kullanarak, gelen isteğin tüm girdi verilerini bir koleksiyon (collection) olarak alabilirsiniz:
```php
$input = $request->collect();
```
`collect` metodu ayrıca, gelen isteğin girdisinin bir alt kümesini bir koleksiyon olarak almanıza da olanak tanır:
```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

#### Bir Girdi Değeri Alma (Retrieving an Input Value)
Birkaç basit metot kullanarak, istek için hangi HTTP fiilinin kullanıldığı konusunda endişelenmeden `Illuminate\Http\Request` örneğinizden tüm kullanıcı girdilerine erişebilirsiniz. HTTP fiili ne olursa olsun, `input` metodu kullanıcı girdisini almak için kullanılabilir:
```php
$name = $request->input('name');
```
`input` metoduna ikinci argüman olarak bir varsayılan değer iletebilirsiniz. İstenen girdi değeri istekte yoksa bu değer döndürülecektir:
```php
$name = $request->input('name', 'Sally');
```
Dizi girdileri içeren formlarla çalışırken, dizilere erişmek için "nokta" notasyonunu kullanın:
```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```
Tüm girdi değerlerini ilişkisel bir dizi (associative array) olarak almak için `input` metodunu herhangi bir argüman olmadan çağırabilirsiniz:
```php
$input = $request->input();
```

#### Sorgu Dizesinden (Query String) Girdi Alma
`input` metodu değerleri tüm istek yükünden (request payload) (sorgu dizesi dahil) alırken, `query` metodu yalnızca sorgu dizesinden değerleri alacaktır:
```php
$name = $request->query('name');
```
İstenen sorgu dizesi değer verisi yoksa, bu metodun ikinci argümanı döndürülecektir:
```php
$name = $request->query('name', 'Helen');
```
Tüm sorgu dizesi değerlerini ilişkisel bir dizi olarak almak için `query` metodunu herhangi bir argüman olmadan çağırabilirsiniz:
```php
$query = $request->query();
```

#### JSON Girdi Değerlerini Alma (Retrieving JSON Input Values)
Uygulamanıza JSON istekleri gönderirken, isteğin `Content-Type` başlığı uygun şekilde `application/json` olarak ayarlandığı sürece JSON verilerine `input` metodu aracılığıyla erişebilirsiniz. JSON dizileri / nesneleri içinde iç içe geçmiş değerleri almak için "nokta" sözdizimini bile kullanabilirsiniz:
```php
$name = $request->input('user.name');
```

#### Stringable (Dize Benzeri) Girdi Değerlerini Alma
İsteğin girdi verilerini ilkel bir `string` olarak almak yerine, istek verilerini bir `Illuminate\Support\Stringable` örneği olarak almak için `string` metodunu kullanabilirsiniz:
```php
$name = $request->string('name')->trim();
```

#### Tamsayı (Integer) Girdi Değerlerini Alma
Girdi değerlerini tamsayı olarak almak için `integer` metodunu kullanabilirsiniz. Bu metod, girdi değerini bir tamsayıya dönüştürmeye (cast) çalışacaktır. Girdi yoksa veya dönüştürme başarısız olursa, belirttiğiniz varsayılan değeri döndürecektir. Bu, özellikle sayfalandırma (pagination) veya diğer sayısal girdiler için kullanışlıdır:
```php
$perPage = $request->integer('per_page');
```

#### Boolean (Mantıksal) Girdi Değerlerini Alma
Onay kutuları (checkboxes) gibi HTML öğeleriyle uğraşırken, uygulamanız aslında dizeler olan "doğruluk değeri taşıyan" (truthy) değerler alabilir. Örneğin, "true" veya "on". Kolaylık olması açısından, bu değerleri boolean olarak almak için `boolean` metodunu kullanabilirsiniz. `boolean` metodu, `1`, `"1"`, `true`, `"true"`, `"on"` ve `"yes"` için `true` döndürür. Diğer tüm değerler `false` döndürecektir:
```php
$archived = $request->boolean('archived');
```

#### Dizi (Array) Girdi Değerlerini Alma
Dizi içeren girdi değerleri `array` metodu kullanılarak alınabilir. Bu metod, girdi değerini her zaman bir diziye dönüştürecektir (cast). İstek, verilen ada sahip bir girdi değeri içermiyorsa, boş bir dizi döndürülecektir:
```php
$versions = $request->array('versions');
```

#### Tarih (Date) Girdi Değerlerini Alma
Kolaylık olması açısından, tarih / saat içeren girdi değerleri `date` metodu kullanılarak Carbon örnekleri olarak alınabilir. İstek, verilen ada sahip bir girdi değeri içermiyorsa, `null` döndürülecektir:
```php
$birthday = $request->date('birthday');
```
`date` metodu tarafından kabul edilen ikinci ve üçüncü argümanlar, sırasıyla tarihin biçimini (format) ve saat dilimini (timezone) belirtmek için kullanılabilir:
```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```
Girdi değeri mevcutsa ancak geçersiz bir biçime sahipse, bir `InvalidArgumentException` fırlatılacaktır; bu nedenle, `date` metodunu çağırmadan önce girdiyi doğrulamanız (validate) önerilir.

#### Enum Girdi Değerlerini Alma (Retrieving Enum Input Values)
PHP enum'larına karşılık gelen girdi değerleri de istekten alınabilir. İstek, verilen ada sahip bir girdi değeri içermiyorsa veya enum, girdi değeriyle eşleşen bir destekleyici değere (backing value) sahip değilse, `null` döndürülecektir. `enum` metodu, girdi değerinin adını ve enum sınıfını birinci ve ikinci argümanları olarak kabul eder:
```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```
Ayrıca, değer eksik veya geçersizse döndürülecek bir varsayılan değer de sağlayabilirsiniz:
```php
$status = $request->enum('status', Status::class, Status::Pending);
```
Girdi değeri, bir PHP enum'ına karşılık gelen bir değerler dizisiyse, değerler dizisini enum örnekleri olarak almak için `enums` metodunu kullanabilirsiniz:
```php
use App\Enums\Product;

$products = $request->enums('products', Product::class);
```

#### Dinamik Özellikler (Dynamic Properties) Aracılığıyla Girdi Alma
`Illuminate\Http\Request` örneği üzerinde dinamik özellikleri (dynamic properties) kullanarak da kullanıcı girdilerine erişebilirsiniz. Örneğin, uygulamanızın formlarından biri bir `name` alanı içeriyorsa, alanın değerine şu şekilde erişebilirsiniz:
```php
$name = $request->name;
```
Dinamik özellikler kullanıldığında, Laravel önce parametrenin değerini istek yükünde (request payload) arar. Eğer mevcut değilse, Laravel alanı eşleşen rotanın parametrelerinde arar.

#### Girdi Verilerinin Bir Kısmını Alma (Retrieving a Portion of the Input Data)
Girdi verilerinin bir alt kümesini almanız gerekiyorsa, `only` ve `except` metotlarını kullanabilirsiniz. Bu metotların her ikisi de tek bir dizi veya dinamik bir argüman listesi kabul eder:
```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```
`only` metodu, talep ettiğiniz tüm anahtar/değer çiftlerini döndürür; ancak, istekte bulunmayan anahtar/değer çiftlerini döndürmez.

### Girdi Varlığı (Input Presence)
Bir değerin istekte mevcut olup olmadığını belirlemek için `has` metodunu kullanabilirsiniz. `has` metodu, değer istekte mevcutsa `true` döndürür. Ayrıca, `hasAny` metodu, belirtilen değerlerden herhangi biri mevcutsa `true` döndürür:
```php
if ($request->has('name')) {
    // ...
}

if ($request->hasAny(['name', 'email'])) {
    // ...
}
```
Bir değerin istekte mevcut olup olmadığını ve boş olmadığını belirlemek isterseniz, `filled` metodunu kullanabilirsiniz:
```php
if ($request->filled('name')) {
    // ...
}
```
Bir değerin istekte mevcut olmadığını veya boş olduğunu belirlemek için `missing` metodunu kullanabilirsiniz:
```php
if ($request->missing('name')) {
    // ...
}
```
`whenFilled` metodu, bir değer istekte mevcut olduğunda ve boş olmadığında belirli bir geri çağrıyı (callback) yürütmek için kullanılabilir:
```php
$request->whenFilled('name', function (string $input) {
    // ...
}, function () {
    // ...
});
```

### Eski Girdi (Old Input)
Laravel, bir isteğin girdilerini bir sonraki istek sırasında saklamanıza olanak tanır. Bu, özellikle doğrulama (validation) hatalarını tespit ettikten sonra form alanlarını yeniden doldurmak için kullanışlıdır. Ancak, Laravel'in yerleşik doğrulama hizmetlerini kullanıyorsanız, bu özelliklerden bazılarını manuel olarak kullanmanız gerekmeyebilir, çünkü Laravel'in yerleşik doğrulama özellikleri bunları otomatik olarak çağırır.

#### Girdiyi Oturuma (Session) Flash'lama
`flash` metodu, geçerli girdiyi, uygulamanın bir sonraki isteği sırasında yalnızca kullanılabilir olacak şekilde oturuma (session) flash'lar:
```php
$request->flash();
```
Ayrıca, girdinin yalnızca bir alt kümesini oturuma flash'lamak için `flashOnly` ve `flashExcept` metotlarını da kullanabilirsiniz. Bu metotlar, korumalı alanlardan (sensitive fields) girdileri oturumun dışında tutmak için kullanışlıdır:
```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

#### Girdiyi Flash'lama ve Sonra Yönlendirme
Genellikle, girdiyi oturuma flash'lamak ve ardından bir önceki sayfaya yönlendirme yapmak isteyebilirsiniz. Yönlendirici (router) üzerindeki `withInput` metodu kullanılarak bu işlem kolayca gerçekleştirilebilir:
```php
return redirect('form')->withInput();

return redirect()->route('user.create')->withInput();

return back()->withInput();
```

#### Eski Girdiyi Alma
Bir sonraki istek sırasında flash'lanmış girdiyi almak için `old` metodunu kullanabilirsiniz. `old` metodu, daha önce oturuma flash'lanmış girdi verilerini oturumdan çekecektir:
```php
$username = $request->old('username');
```
Laravel ayrıca bir `global old` yardımcı fonksiyonu da sağlar. Bir Blade şablonu içinde eski girdiyi görüntülemek istiyorsanız, `old` yardımcı fonksiyonunu kullanmak daha uygundur. Belirtilen alan için eski girdi mevcut değilse, `null` döndürülecektir:
```blade
<input type="text" name="username" value="{{ old('username') }}">
```

### Çerezler (Cookies)

#### İstekten Çerez Alma (Retrieving Cookies from Requests)
Laravel framework'ü tarafından oluşturulan tüm çerezler (cookies) şifrelenir ve bir imza ile imzalanır; bu sayede istemci tarafından değiştirilmişlerse geçersiz sayılırlar. İstekten bir çerez değeri almak için `Illuminate\Http\Request` örneği üzerindeki `cookie` metodunu kullanın:
```php
$value = $request->cookie('name');
```

#### Çerezleri Yanıtlara Ekleme (Attaching Cookies to Responses)
Bir giden `Illuminate\Http\Response` örneğine çerez eklemek için `cookie` metodunu kullanabilirsiniz. Bu metoda çerezin adını, değerini ve çerezin geçerli olacağı dakika sayısını geçmelisiniz:
```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```
`cookie` metodu ayrıca, daha az sıklıkla kullanılan bazı argümanları da kabul eder. Genel olarak, bu argümanlar PHP'nin yerel setcookie metoduna verilen argümanlarla aynı amaç ve anlama sahiptir:
```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```
Yanıtı framework'e göndermeden önce bir çerezin yanıta eklendiğinden emin olmak istiyorsanız, `queueCookie` metodunu kullanabilirsiniz:
```php
/** @var \Illuminate\Http\Response $response */
$response->queueCookie('name', 'value', $minutes);
```

#### Çerez Oluşturma ve Örnek Oluşturma
Yanıta eklenebilecek bir `Symfony\Component\HttpFoundation\Cookie` örneği oluşturmak için `Cookie::make` yardımcı fonksiyonunu kullanabilirsiniz. Bu örnek daha sonra `cookie` metodu aracılığıyla yanıt örneğine eklenebilir:
```php
use Illuminate\Support\Facades\Cookie;

$cookie = Cookie::make('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

### Dosyalar

#### Dosya Yükleme İşleme (Retrieving Uploaded Files)
Yüklenen dosyaları `Illuminate\Http\Request` örneğinden `file` metodu veya dinamik özellikler kullanarak alabilirsiniz. `file` metodu, `Illuminate\Http\UploadedFile` sınıfının bir örneğini döndürür; bu sınıf, PHP'nin `SplFileInfo` sınıfını genişletir (extend) ve dosyayla etkileşim için çeşitli metotlar sağlar:
```php
$file = $request->file('photo');

$file = $request->photo;
```
`hasFile` metodunu kullanarak istekte bir dosyanın mevcut olup olmadığını belirleyebilirsiniz:
```php
if ($request->hasFile('photo')) {
    // ...
}
```

#### Başarılı Yüklemeleri Doğrulama (Validating Successful Uploads)
Dosyanın mevcut olup olmadığını kontrol etmenin yanı sıra, dosyayla ilgili herhangi bir sorun olmadığını `isValid` metodu aracılığıyla doğrulayabilirsiniz:
```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

#### Dosya Yolları ve Uzantılar (File Paths and Extensions)
`UploadedFile` sınıfı ayrıca dosyanın tam nitelikli yoluna (fully qualified path) ve uzantısına (extension) erişim sağlar. `path` metodu dosyanın geçici yükleme konumunu döndürür. `extension` metodu dosyanın uzantısını tahmin edebilir:
```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

#### Diğer Dosya Metotları
`UploadedFile` örneğinde birçok başka metot mevcuttur. Bu metotlar hakkında daha fazla bilgi için sınıfın API belgelerine göz atın.