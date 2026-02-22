# Laravel 12 Dokümantasyonu: HTTP Testleri (HTTP Tests)

## Giriş

Laravel, uygulamanıza HTTP istekleri yapmak ve yanıtları incelemek için son derece akıcı bir API sağlar. Örneğin, aşağıda tanımlanan özellik testine (feature test) bir göz atın:
```php
<?php

test('uygulama başarılı bir yanıt döndürür', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir test örneği.
     */
    public function test_the_application_returns_a_successful_response(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```
`get` metodu uygulamaya bir `GET` isteği yaparken, `assertStatus` metodu döndürülen yanıtın verilen HTTP durum koduna (status code) sahip olması gerektiğini iddia eder (assert). Bu basit iddianın yanı sıra Laravel, yanıt başlıklarını (headers), içeriğini, JSON yapısını ve daha fazlasını incelemek için çeşitli iddialar (assertions) da içerir.

## İstek Yapma (Making Requests)

Uygulamanıza bir istek yapmak için testinizde `get`, `post`, `put`, `patch` veya `delete` metotlarını çağırabilirsiniz. Bu metotlar aslında uygulamanıza "gerçek" bir HTTP isteği yapmaz. Bunun yerine, tüm ağ isteği dahili olarak simüle edilir (simulated).

Test isteği metotları, bir `Illuminate\Http\Response` örneği döndürmek yerine, uygulamanızın yanıtlarını incelemenize izin veren çeşitli yararlı iddialar sağlayan bir `Illuminate\Testing\TestResponse` örneği döndürür:
```php
<?php

test('temel istek', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir test örneği.
     */
    public function test_a_basic_request(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```
Genel olarak, testlerinizin her biri uygulamanıza yalnızca bir istek yapmalıdır. Tek bir test metodu içinde birden çok istek yürütülürse beklenmeyen davranışlar ortaya çıkabilir.

Kolaylık olması açısından, testleri çalıştırırken CSRF ara yazılımı (middleware) otomatik olarak devre dışı bırakılır.

### İstek Başlıklarını (Headers) Özelleştirme (Customizing Request Headers)
İsteği uygulamaya göndermeden önce isteğin başlıklarını özelleştirmek için `withHeaders` metodunu kullanabilirsiniz. Bu metod, isteğe istediğiniz herhangi bir özel başlığı eklemenize olanak tanır:
```php
<?php

test('başlıklarla etkileşim', function () {
    $response = $this->withHeaders([
        'X-Header' => 'Value',
    ])->post('/user', ['name' => 'Sally']);

    $response->assertStatus(201);
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir fonksiyonel test örneği.
     */
    public function test_interacting_with_headers(): void
    {
        $response = $this->withHeaders([
            'X-Header' => 'Value',
        ])->post('/user', ['name' => 'Sally']);

        $response->assertStatus(201);
    }
}
```

### Çerezler (Cookies)
Bir istek yapmadan önce çerez değerlerini ayarlamak için `withCookie` veya `withCookies` metotlarını kullanabilirsiniz. `withCookie` metodu iki argüman olarak bir çerez adı ve değeri kabul ederken, `withCookies` metodu bir dizi ad/değer çifti kabul eder:
```php
<?php

test('çerezlerle etkileşim', function () {
    $response = $this->withCookie('color', 'blue')->get('/');

    $response = $this->withCookies([
        'color' => 'blue',
        'name' => 'Taylor',
    ])->get('/');

    //
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_cookies(): void
    {
        $response = $this->withCookie('color', 'blue')->get('/');

        $response = $this->withCookies([
            'color' => 'blue',
            'name' => 'Taylor',
        ])->get('/');

        //
    }
}
```

### Oturum (Session) / Kimlik Doğrulama (Authentication)
Laravel, HTTP testleri sırasında oturumla etkileşim kurmak için birkaç yardımcı sağlar. İlk olarak, `withSession` metodunu kullanarak oturum verilerini belirli bir diziye ayarlayabilirsiniz. Bu, uygulamanıza bir istek yapmadan önce oturumu verilerle yüklemek için kullanışlıdır:
```php
<?php

test('oturumla etkileşim', function () {
    $response = $this->withSession(['banned' => false])->get('/');

    //
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_the_session(): void
    {
        $response = $this->withSession(['banned' => false])->get('/');

        //
    }
}
```
Laravel'in oturumu tipik olarak, o anda kimliği doğrulanmış kullanıcının durumunu korumak için kullanılır. Bu nedenle, `actingAs` yardımcı metodu, belirli bir kullanıcının kimliğini geçerli kullanıcı olarak doğrulamanın basit bir yolunu sağlar. Örneğin, bir kullanıcı oluşturmak ve kimliğini doğrulamak için bir model fabrikası (model factory) kullanabiliriz:
```php
<?php

use App\Models\User;

test('kimlik doğrulama gerektiren bir eylem', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->withSession(['banned' => false])
        ->get('/');

    //
});
```
```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_an_action_that_requires_authentication(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->withSession(['banned' => false])
            ->get('/');

        //
    }
}
```
`actingAs` metoduna ikinci argüman olarak koruma (guard) adını ileterek, verilen kullanıcının kimliğini doğrulamak için hangi korumanın kullanılması gerektiğini de belirtebilirsiniz. `actingAs` metoduna sağlanan koruma, test süresince varsayılan koruma haline de gelecektir:
```php
$this->actingAs($user, 'web');
```
İsteğin kimliği doğrulanmamış (unauthenticated) olduğundan emin olmak isterseniz, `actingAsGuest` metodunu kullanabilirsiniz:
```php
$this->actingAsGuest();
```

### Yanıtlarda Hata Ayıklama (Debugging Responses)
Uygulamanıza bir test isteği yaptıktan sonra, `dump`, `dumpHeaders` ve `dumpSession` metotları yanıt içeriğini incelemek ve hata ayıklamak için kullanılabilir:
```php
<?php

test('temel test', function () {
    $response = $this->get('/');

    $response->dump();
    $response->dumpHeaders();
    $response->dumpSession();
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir test örneği.
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dump();
        $response->dumpHeaders();
        $response->dumpSession();
    }
}
```
Alternatif olarak, `dd`, `ddHeaders`, `ddBody`, `ddJson` ve `ddSession` metotlarını kullanarak yanıt hakkındaki bilgileri dökebilir (dump) ve ardından yürütmeyi durdurabilirsiniz (stop execution):
```php
<?php

test('temel test', function () {
    $response = $this->get('/');

    $response->dd();
    $response->ddHeaders();
    $response->ddBody();
    $response->ddJson();
    $response->ddSession();
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir test örneği.
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dd();
        $response->ddHeaders();
        $response->ddBody();
        $response->ddJson();
        $response->ddSession();
    }
}
```

### İstisna (Exception) Yönetimi
Bazen uygulamanızın belirli bir istisna (exception) fırlattığını test etmeniz gerekebilir. Bunu başarmak için, `Exceptions` facade'ı aracılığıyla istisna işleyicisini (exception handler) "sahteleştirebilirsiniz" (fake). İstisna işleyicisi sahteleştirildikten sonra, istek sırasında fırlatılan istisnalar hakkında iddialarda bulunmak için `assertReported` ve `assertNotReported` metotlarını kullanabilirsiniz:
```php
<?php

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;

test('istisna fırlatılır', function () {
    Exceptions::fake();

    $response = $this->get('/order/1');

    // Bir istisnanın fırlatıldığını iddia et...
    Exceptions::assertReported(InvalidOrderException::class);

    // İstisnaya karşı iddiada bulun...
    Exceptions::assertReported(function (InvalidOrderException $e) {
        return $e->getMessage() === 'Sipariş geçersizdi.';
    });
});
```
```php
<?php

namespace Tests\Feature;

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir test örneği.
     */
    public function test_exception_is_thrown(): void
    {
        Exceptions::fake();

        $response = $this->get('/');

        // Bir istisnanın fırlatıldığını iddia et...
        Exceptions::assertReported(InvalidOrderException::class);

        // İstisnaya karşı iddiada bulun...
        Exceptions::assertReported(function (InvalidOrderException $e) {
            return $e->getMessage() === 'Sipariş geçersizdi.';
        });
    }
}
```
`assertNotReported` ve `assertNothingReported` metotları, belirli bir istisnanın istek sırasında fırlatılmadığını veya hiçbir istisna fırlatılmadığını iddia etmek için kullanılabilir:
```php
Exceptions::assertNotReported(InvalidOrderException::class);

Exceptions::assertNothingReported();
```
İsteğinizi yapmadan önce `withoutExceptionHandling` metodunu çağırarak belirli bir istek için istisna yönetimini tamamen devre dışı bırakabilirsiniz:
```php
$response = $this->withoutExceptionHandling()->get('/');
```
Ek olarak, uygulamanızın PHP dili veya uygulamanızın kullandığı kütüphaneler tarafından kullanımdan kaldırılmış (deprecated) özellikleri kullanmadığından emin olmak isterseniz, isteğinizi yapmadan önce `withoutDeprecationHandling` metodunu çağırabilirsiniz. Kullanımdan kaldırma işleme (deprecation handling) devre dışı bırakıldığında, kullanımdan kaldırma uyarıları istisnalara dönüştürülecek ve böylece testinizin başarısız olmasına neden olacaktır:
```php
$response = $this->withoutDeprecationHandling()->get('/');
```
`assertThrows` metodu, belirli bir closure içindeki kodun belirtilen türde bir istisna fırlattığını iddia etmek için kullanılabilir:
```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    OrderInvalid::class
);
```
Fırlatılan istisnayı incelemek ve onunla ilgili iddialarda bulunmak isterseniz, `assertThrows` metoduna ikinci argüman olarak bir closure sağlayabilirsiniz:
```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    fn (OrderInvalid $e) => $e->orderId() === 123;
);
```
`assertDoesntThrow` metodu, belirli bir closure içindeki kodun herhangi bir istisna fırlatmadığını iddia etmek için kullanılabilir:
```php
$this->assertDoesntThrow(fn () => (new ProcessOrder)->execute());
```

## JSON API'lerini Test Etme (Testing JSON APIs)

Laravel ayrıca JSON API'lerini ve yanıtlarını test etmek için birkaç yardımcı sağlar. Örneğin, `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson` ve `optionsJson` metotları, çeşitli HTTP fiilleriyle (verbs) JSON istekleri yapmak için kullanılabilir. Ayrıca bu metotlara kolayca veri ve başlıklar iletebilirsiniz. Başlamak için, `/api/user` adresine bir `POST` isteği yapan ve beklenen JSON verilerinin döndürüldüğünü iddia eden bir test yazalım:
```php
<?php

test('api isteği yapma', function () {
    $response = $this->postJson('/api/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJson([
            'created' => true,
        ]);
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir fonksiyonel test örneği.
     */
    public function test_making_an_api_request(): void
    {
        $response = $this->postJson('/api/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```
Ek olarak, JSON yanıt verilerine yanıt üzerinde dizi değişkenleri olarak erişilebilir, bu da bir JSON yanıtı içinde döndürülen tek tek değerleri incelemeyi kolaylaştırır:
```php
expect($response['created'])->toBeTrue();
```
```php
$this->assertTrue($response['created']);
```
`assertJson` metodu, verilen dizinin uygulama tarafından döndürülen JSON yanıtı içinde mevcut olduğunu doğrulamak için yanıtı bir diziye dönüştürür. Yani, JSON yanıtında başka özellikler varsa, verilen parça mevcut olduğu sürece bu test yine de geçecektir.

#### Tam JSON Eşleşmesini İddia Etme (Asserting Exact JSON Matches)
Daha önce belirtildiği gibi, `assertJson` metodu, bir JSON parçasının JSON yanıtı içinde mevcut olduğunu iddia etmek için kullanılabilir. Belirli bir dizinin uygulamanız tarafından döndürülen JSON ile tam olarak eşleştiğini doğrulamak isterseniz, `assertExactJson` metodunu kullanmalısınız:
```php
<?php

test('tam json eşleşmesini iddia etme', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertExactJson([
            'created' => true,
        ]);
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir fonksiyonel test örneği.
     */
    public function test_asserting_an_exact_json_match(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

#### JSON Yollarında (Paths) İddia Etme (Asserting on JSON Paths)
JSON yanıtının belirtilen bir yolda (path) verilen veriyi içerdiğini doğrulamak isterseniz, `assertJsonPath` metodunu kullanmalısınız:
```php
<?php

test('json yol değerini iddia etme', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJsonPath('team.owner.name', 'Darian');
});
```
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Temel bir fonksiyonel test örneği.
     */
    public function test_asserting_a_json_path_value(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJsonPath('team.owner.name', 'Darian');
    }
}
```