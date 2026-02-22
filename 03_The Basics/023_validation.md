# Laravel 12 Dokümantasyonu: Doğrulama (Validation)

## Giriş

Laravel, uygulamanıza gelen verileri doğrulamak (validate) için birkaç farklı yaklaşım sağlar. Tüm gelen HTTP isteklerinde mevcut olan `validate` metodunu kullanmak en yaygın olanıdır. Ancak, diğer doğrulama yaklaşımlarını da tartışacağız.

Laravel, verilere uygulayabileceğiniz çok çeşitli kullanışlı doğrulama kuralları (validation rules) içerir, hatta değerlerin belirli bir veritabanı tablosunda benzersiz (unique) olup olmadığını doğrulama yeteneği bile sağlar. Laravel'in tüm doğrulama özelliklerine aşina olmanız için bu doğrulama kurallarının her birini ayrıntılı olarak ele alacağız.

## Hızlı Başlangıç (Validation Quickstart)

Laravel'in güçlü doğrulama özelliklerini öğrenmek için, bir formu doğrulama ve hata mesajlarını kullanıcıya geri gösterme konusunda tam bir örneğe bakalım. Bu üst düzey genel bakışı okuyarak, Laravel kullanarak gelen istek verilerini nasıl doğrulayacağınız konusunda iyi bir genel anlayış kazanabileceksiniz.

### Rotaları Tanımlama (Defining the Routes)
İlk olarak, `routes/web.php` dosyamızda aşağıdaki rotaların tanımlandığını varsayalım:
```php
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```
`GET` rotası, kullanıcının yeni bir blog yazısı oluşturması için bir form görüntüleyecek, `POST` rotası ise yeni blog yazısını veritabanında saklayacaktır.

### Controller'ı Oluşturma (Creating the Controller)
Şimdi, bu rotalara gelen istekleri işleyen basit bir controller'a bakalım. `store` metodunu şimdilik boş bırakacağız:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class PostController extends Controller
{
    /**
     * Yeni bir blog yazısı oluşturmak için formu göster.
     */
    public function create(): View
    {
        return view('post.create');
    }

    /**
     * Yeni bir blog yazısı sakla.
     */
    public function store(Request $request): RedirectResponse
    {
        // Blog yazısını doğrula ve sakla...

        $post = /** ... */

        return to_route('post.show', ['post' => $post->id]);
    }
}
```

### Doğrulama Mantığını Yazma (Writing the Validation Logic)
Şimdi `store` metodumuzu, yeni blog yazısını doğrulama mantığıyla doldurmaya hazırız. Bunu yapmak için, `Illuminate\Http\Request` nesnesi tarafından sağlanan `validate` metodunu kullanacağız. Doğrulama kuralları geçerse, kodunuz normal şekilde yürütülmeye devam edecektir; ancak, doğrulama başarısız olursa, bir `Illuminate\Validation\ValidationException` istisnası (exception) fırlatılacak ve uygun hata yanıtı otomatik olarak kullanıcıya geri gönderilecektir.

Geleneksel bir HTTP isteği sırasında doğrulama başarısız olursa, önceki URL'ye bir yönlendirme (redirect) yanıtı oluşturulacaktır. Gelen istek bir XHR isteğiyse, doğrulama hata mesajlarını içeren bir JSON yanıtı döndürülecektir.

`validate` metodunu daha iyi anlamak için `store` metoduna geri dönelim:
```php
/**
 * Yeni bir blog yazısı sakla.
 */
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // Blog yazısı geçerli...

    return redirect('/posts');
}
```
Gördüğünüz gibi, doğrulama kuralları `validate` metoduna iletilir. Endişelenmeyin - mevcut tüm doğrulama kuralları belgelenmiştir. Yine, doğrulama başarısız olursa, uygun yanıt otomatik olarak oluşturulacaktır. Doğrulama geçerse, controller'ımız normal şekilde çalışmaya devam edecektir.

Alternatif olarak, doğrulama kuralları tek bir `|` ile ayrılmış dize yerine bir kurallar dizisi (array of rules) olarak da belirtilebilir:
```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```
Ek olarak, bir isteği doğrulamak ve herhangi bir hata mesajını adlandırılmış bir hata torbasına (named error bag) saklamak için `validateWithBag` metodunu kullanabilirsiniz:
```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

#### İlk Doğrulama Hatasında Durma (Stopping on First Validation Failure)
Bazen bir nitelikteki (attribute) doğrulama kurallarını ilk doğrulama hatasından sonra çalıştırmayı durdurmak isteyebilirsiniz. Bunu yapmak için, niteliğe `bail` kuralını atayın:
```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```
Bu örnekte, `title` niteliğindeki `unique` kuralı başarısız olursa, `max` kuralı kontrol edilmeyecektir. Kurallar, atandıkları sırayla doğrulanacaktır.

#### İç İçe Nitelikler (Nested Attributes) Hakkında Bir Not
Gelen HTTP isteği "iç içe" (nested) alan verileri içeriyorsa, bu alanları doğrulama kurallarınızda "nokta" sözdizimini ("dot" syntax) kullanarak belirtebilirsiniz:
```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```
Öte yandan, alan adınız gerçek bir nokta (literal period) içeriyorsa, noktayı bir ters eğik çizgi (backslash) ile kaçış karakteri olarak kullanarak (escaping) bunun "nokta" sözdizimi olarak yorumlanmasını açıkça engelleyebilirsiniz:
```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

### Doğrulama Hatalarını Görüntüleme (Displaying the Validation Errors)
Peki ya gelen istek alanları verilen doğrulama kurallarını geçemezse? Daha önce belirtildiği gibi, Laravel otomatik olarak kullanıcıyı önceki konumlarına yönlendirecektir. Ek olarak, tüm doğrulama hataları ve istek girdisi (request input) otomatik olarak oturuma (session) flash edilecektir.

Uygulamanızın tüm görünümleriyle (views) `$errors` değişkeni, `web` ara yazılım grubu tarafından sağlanan `Illuminate\View\Middleware\ShareErrorsFromSession` ara yazılımı aracılığıyla paylaşılır. Bu ara yazılım uygulandığında, görünümlerinizde her zaman bir `$errors` değişkeni mevcut olacak ve `$errors` değişkeninin her zaman tanımlandığını ve güvenle kullanılabileceğini varsaymanıza olanak tanır. `$errors` değişkeni, `Illuminate\Support\MessageBag`'in bir örneği olacaktır. Bu nesneyle çalışma hakkında daha fazla bilgi için dokümantasyonuna göz atın.

Bu nedenle, örneğimizde, doğrulama başarısız olduğunda kullanıcı controller'ımızın `create` metoduna yönlendirilecek ve hata mesajlarını görünümde görüntülememize olanak tanıyacaktır:
```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

#### Hata Mesajlarını Özelleştirme (Customizing the Error Messages)
Laravel'in yerleşik doğrulama kurallarının her biri, uygulamanızın `lang/en/validation.php` dosyasında bulunan bir hata mesajına sahiptir. Uygulamanızda bir `lang` dizini yoksa, `lang:publish` Artisan komutunu kullanarak Laravel'e onu oluşturması talimatını verebilirsiniz.

`lang/en/validation.php` dosyası içinde, her doğrulama kuralı için bir çeviri girdisi (translation entry) bulacaksınız. Bu mesajları uygulamanızın ihtiyaçlarına göre değiştirmekte veya düzenlemekte özgürsünüz.

Ek olarak, uygulamanızın dili için mesajları çevirmek üzere bu dosyayı başka bir dil dizinine kopyalayabilirsiniz. Laravel yerelleştirmesi (localization) hakkında daha fazla bilgi edinmek için eksiksiz yerelleştirme dokümantasyonuna göz atın.

Varsayılan olarak, Laravel uygulama iskeleti (skeleton) `lang` dizinini içermez. Laravel'in dil dosyalarını özelleştirmek isterseniz, bunları `lang:publish` Artisan komutu aracılığıyla yayınlayabilirsiniz.

#### XHR İstekleri ve Doğrulama (XHR Requests and Validation)
Bu örnekte, uygulamaya veri göndermek için geleneksel bir form kullandık. Ancak, birçok uygulama JavaScript destekli bir ön uçtan (frontend) XHR istekleri alır. Bir XHR isteği sırasında `validate` metodunu kullanırken, Laravel bir yönlendirme yanıtı oluşturmayacaktır. Bunun yerine Laravel, tüm doğrulama hatalarını içeren bir JSON yanıtı oluşturur. Bu JSON yanıtı, 422 HTTP durum koduyla gönderilecektir.

#### @error Direktifi (The @error Directive)
Belirli bir nitelik için doğrulama hata mesajlarının mevcut olup olmadığını hızlıca belirlemek için `@error` Blade direktifini kullanabilirsiniz. Bir `@error` direktifi içinde, hata mesajını görüntülemek için `$message` değişkenini görüntüleyebilirsiniz:
```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```
Adlandırılmış hata torbaları (named error bags) kullanıyorsanız, `@error` direktifine ikinci argüman olarak hata torbasının adını iletebilirsiniz:
```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

### Formları Yeniden Doldurma (Repopulating Forms)
Laravel bir doğrulama hatası nedeniyle bir yönlendirme yanıtı oluşturduğunda, framework otomatik olarak isteğin tüm girdisini oturuma flash eder. Bu, bir sonraki istek sırasında girdiye kolayca erişebilmeniz ve kullanıcının göndermeye çalıştığı formu yeniden doldurabilmeniz (repopulate) için yapılır.

Önceki istekten flash edilmiş girdiyi almak için, bir `Illuminate\Http\Request` örneği üzerinde `old` metodunu çağırın. `old` metodu, daha önce flash edilmiş girdi verilerini oturumdan çekecektir:
```php
$title = $request->old('title');
```
Laravel ayrıca global bir `old` yardımcısı da sağlar. Bir Blade şablonu içinde eski girdiyi görüntülüyorsanız, formu yeniden doldurmak için `old` yardımcısını kullanmak daha uygundur. Belirtilen alan için eski girdi mevcut değilse, `null` döndürülecektir:
```blade
<input type="text" name="title" value="{{ old('title') }}">
```

### İsteğe Bağlı Alanlar (Optional Fields) Hakkında Bir Not
Varsayılan olarak Laravel, uygulamanızın global ara yazılım yığınına `TrimStrings` ve `ConvertEmptyStringsToNull` ara yazılımlarını içerir. Bu nedenle, doğrulayıcının (validator) `null` değerleri geçersiz olarak kabul etmesini istemiyorsanız, "isteğe bağlı" (optional) istek alanlarınızı sık sık `nullable` olarak işaretlemeniz gerekecektir. Örneğin:
```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```
Bu örnekte, `publish_at` alanının ya `null` ya da geçerli bir tarih gösterimi olabileceğini belirtiyoruz. `nullable` değiştiricisi (modifier) kural tanımına eklenmezse, doğrulayıcı `null` değerini geçersiz bir tarih olarak kabul edecektir.

### Doğrulama Hatası Yanıt Formatı (Validation Error Response Format)
Uygulamanız bir `Illuminate\Validation\ValidationException` istisnası fırlattığında ve gelen HTTP isteği bir JSON yanıtı bekliyorsa, Laravel otomatik olarak hata mesajlarını sizin için biçimlendirecek ve bir `422 Unprocessable Entity` HTTP yanıtı döndürecektir.

Aşağıda, doğrulama hataları için JSON yanıt formatının bir örneğini inceleyebilirsiniz. İç içe hata anahtarlarının (nested error keys) "nokta" notasyonu formatında düzleştirildiğine (flattened) dikkat edin:
```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

## Form İsteği Doğrulaması (Form Request Validation)

### Form İstekleri Oluşturma (Creating Form Requests)
Daha karmaşık doğrulama senaryoları için bir "form isteği" (form request) oluşturmak isteyebilirsiniz. Form istekleri, kendi doğrulama ve yetkilendirme (authorization) mantıklarını kapsülleyen (encapsulate) özel istek sınıflarıdır. Bir form isteği sınıfı oluşturmak için `make:request` Artisan CLI komutunu kullanabilirsiniz:
```bash
php artisan make:request StorePostRequest
```
Oluşturulan form isteği sınıfı, `app/Http/Requests` dizinine yerleştirilecektir. Bu dizin mevcut değilse, `make:request` komutunu çalıştırdığınızda oluşturulacaktır. Laravel tarafından oluşturulan her form isteğinin iki metodu vardır: `authorize` ve `rules`.

Tahmin edebileceğiniz gibi, `authorize` metodu, mevcut kimliği doğrulanmış kullanıcının istek tarafından temsil edilen eylemi gerçekleştirip gerçekleştiremeyeceğini belirlemekten sorumludur, `rules` metodu ise isteğin verilerine uygulanması gereken doğrulama kurallarını döndürür:
```php
/**
 * İsteğe uygulanacak doğrulama kurallarını al.
 *
 * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
 */
public function rules(): array
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```
`rules` metodunun imzasında ihtiyacınız olan herhangi bir bağımlılığı tip belirtebilirsiniz (type-hint). Bunlar otomatik olarak Laravel servis kabı (service container) aracılığıyla çözümlenecektir (resolved).

Peki, doğrulama kuralları nasıl değerlendirilir? Tek yapmanız gereken, controller metodunuzda isteği tip belirtmektir. Gelen form isteği, controller metodu çağrılmadan önce doğrulanır; bu, controller'ınızı herhangi bir doğrulama mantığıyla doldurmanız gerekmediği anlamına gelir:
```php
/**
 * Yeni bir blog yazısı sakla.
 */
public function store(StorePostRequest $request): RedirectResponse
{
    // Gelen istek geçerli...

    // Doğrulanmış girdi verilerini al...
    $validated = $request->validated();

    // Doğrulanmış girdi verilerinin bir kısmını al...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);

    // Blog yazısını sakla...

    return redirect('/posts');
}
```
Doğrulama başarısız olursa, kullanıcıyı önceki konumlarına geri göndermek için bir yönlendirme yanıtı oluşturulacaktır. Hatalar ayrıca görüntülenmek üzere oturuma flash edilecektir. İstek bir XHR isteğiyse, kullanıcıya doğrulama hatalarının bir JSON temsilini içeren 422 durum kodlu bir HTTP yanıtı döndürülecektir.

Inertia destekli Laravel ön yüzünüze gerçek zamanlı form isteği doğrulaması eklemeniz mi gerekiyor? Laravel Precognition'a göz atın.

#### Ek Doğrulama Gerçekleştirme (Performing Additional Validation)
Bazen ilk doğrulamanız tamamlandıktan sonra ek doğrulama yapmanız gerekir. Bunu, form isteğinin `after` metodunu kullanarak gerçekleştirebilirsiniz.

`after` metodu, doğrulama tamamlandıktan sonra çağrılacak bir dizi çağrılabilir (callables) veya kapatma (closure) döndürmelidir. Verilen çağrılabilirler bir `Illuminate\Validation\Validator` örneği alacak ve hata mesajlarını gerektiği gibi eklemenize olanak tanıyacaktır:
```php
use Illuminate\Validation\Validator;

/**
 * İstek için doğrulama sonrası (after) kancalarını yapılandır.
 */
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add(
                    'field',
                    'Something is wrong with this field!'
                );
            }
        }
    ];
}
```

#### İstek Yetkilendirmesi (Request Authorization)
Daha önce bahsedildiği gibi, form isteği sınıfı ayrıca bir `authorize` metodu içerir. Bu metot içinde, kimliği doğrulanmış kullanıcının belirli bir kaynağı güncelleme yetkisine sahip olup olmadığını belirleyebilirsiniz. Örneğin, bir kullanıcının blog yazısına yorum yapmaya çalışıp çalışmadığını belirleyebilirsiniz:
```php
use App\Models\Comment;

/**
 * Kullanıcının bu isteği yapmaya yetkili olup olmadığını belirle.
 */
public function authorize(): bool
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```
Tüm form istekleri uygulamanızın servis kabı üzerinden çözümlendiğinden, `authorize` metodunda bağımlılıkları tip belirtebilirsiniz. Bu durumda, Laravel'in örtülü rota model bağlamasını (implicit route model binding) kullanarak `comment` rota parametresine kolayca erişebiliriz.

`authorize` metodu `false` döndürürse, otomatik olarak 403 HTTP yanıtı gönderen bir HTTP istisnası fırlatılacaktır.

Uygulamanızın başka bir bölümü isteği yetkilendirmeyi ele alacaksa, `authorize` metodundan `true` döndürebilirsiniz:
```php
/**
 * Kullanıcının bu isteği yapmaya yetkili olup olmadığını belirle.
 */
public function authorize(): bool
{
    return true;
}
```

#### Hata Mesajlarını Özelleştirme (Customizing the Error Messages)
Form isteğinin `messages` metodunu geçersiz kılarak (override) form isteği tarafından kullanılan hata mesajlarını özelleştirebilirsiniz. Bu metot, nitelik / kural çiftlerinden oluşan bir dizi ve bunlara karşılık gelen hata mesajlarını döndürmelidir:
```php
/**
 * Doğrulama için hata mesajlarını al.
 *
 * @return array<string, string>
 */
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

#### Doğrulama için Nitelik Adlarını Özelleştirme (Customizing the Validation Attributes)
Form isteğinin `attributes` metodunu geçersiz kılarak, doğrulama hata mesajlarının `:attribute` yer tutucusunu (placeholder) değiştirmek için kullanılan nitelik adlarını özelleştirebilirsiniz. Bu metot, nitelik / ad çiftlerinden oluşan bir dizi ve bunlara karşılık gelen adları döndürmelidir:
```php
/**
 * Doğrulama için özel nitelik adlarını al.
 *
 * @return array<string, string>
 */
public function attributes(): array
{
    return [
        'email' => 'email address',
    ];
}
```

#### Doğrulama Sonrası Kancalar (Validation After Hooks)
Form isteğinin `withValidator` metodunu geçersiz kılarak, doğrulama tamamlandıktan sonra çağrılacak bir doğrulama sonrası kancası (hook) da ekleyebilirsiniz. Bu metot, oluşturulmuş doğrulayıcıyı alır ve doğrulama sonrasında herhangi bir ek işlem yapmanıza olanak tanır:
```php
use Illuminate\Validation\Validator;

/**
 * Doğrulayıcı örneğini yapılandır.
 */
public function withValidator(Validator $validator): void
{
    $validator->after(function (Validator $validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```