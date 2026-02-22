# Laravel 12 Dokümantasyonu: Yetkilendirme (Authorization)

## Giriş

Laravel, yerleşik kimlik doğrulama (authentication) servisleri sağlamanın yanı sıra, kullanıcı eylemlerini belirli bir kaynağa karşı yetkilendirmek (authorize) için basit bir yol da sunar. Örneğin, bir kullanıcının kimliği doğrulanmış olsa bile, uygulamanız tarafından yönetilen belirli Eloquent modellerini veya veritabanı kayıtlarını güncelleme veya silme yetkisi olmayabilir. Laravel'in yetkilendirme özellikleri, bu tür yetkilendirme kontrollerini yönetmek için kolay ve düzenli bir yol sağlar.

Laravel, eylemleri yetkilendirmek için iki ana yol sağlar: **gates** (kapılar) ve **policies** (politikalar). Gates ve policies'i, rotalar (routes) ve controller'lar gibi düşünün. Gates, yetkilendirme için basit, kapatma tabanlı (closure-based) bir yaklaşım sunarken, policies (controller'lar gibi) mantığı belirli bir model veya kaynak etrafında gruplandırır. Bu dokümantasyonda, önce gates'i inceleyeceğiz, ardından policies'i ele alacağız.

Bir uygulama oluştururken yalnızca gates veya yalnızca policies kullanmayı seçmek zorunda değilsiniz. Çoğu uygulama büyük olasılıkla hem gates hem de policies'in bir karışımını içerecektir ve bu son derece normaldir! Gates, en çok herhangi bir model veya kaynakla ilgili olmayan eylemler için uygundur; örneğin, bir yönetici panosunu görüntüleme. Buna karşılık, policies, belirli bir model veya kaynak için bir eylemi yetkilendirmek istediğinizde kullanılmalıdır.

## Gates

### Gates Yazma (Writing Gates)
Gates, Laravel'in yetkilendirme özelliklerinin temellerini öğrenmek için harika bir yoldur; ancak, sağlam Laravel uygulamaları oluştururken yetkilendirme kurallarınızı düzenlemek için policies kullanmayı düşünmelisiniz.

Gates, basitçe bir kullanıcının belirli bir eylemi gerçekleştirmeye yetkili olup olmadığını belirleyen kapatmalardır (closures). Tipik olarak, gates `App\Providers\AppServiceProvider` sınıfının `boot` metodu içinde `Gate` facade'ı kullanılarak tanımlanır. Gates her zaman ilk argüman olarak bir kullanıcı örneği alır ve isteğe bağlı olarak ilgili bir Eloquent modeli gibi ek argümanlar alabilir.

Bu örnekte, bir kullanıcının belirli bir `App\Models\Post` modelini güncelleyip güncelleyemeyeceğini belirleyen bir gate tanımlayacağız. Bu gate, kullanıcının `id`'sini, gönderiyi oluşturan kullanıcının `user_id`'si ile karşılaştırarak bunu gerçekleştirecektir:
```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```
Controller'lar gibi, gates bir sınıf geri çağrı dizisi (class callback array) kullanılarak da tanımlanabilir:
```php
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Gate::define('update-post', [PostPolicy::class, 'update']);
}
```

### Eylemleri Yetkilendirme (Authorizing Actions)
Gates kullanarak bir eylemi yetkilendirmek için, `Gate` facade'ı tarafından sağlanan `allows` veya `denies` metotlarını kullanmalısınız. Bu metotlara o anda kimliği doğrulanmış kullanıcıyı iletmeniz gerekmediğine dikkat edin. Laravel, kullanıcıyı gate closure'ına iletmeyi otomatik olarak halleder. Yetkilendirme gerektiren bir eylemi gerçekleştirmeden önce, gate yetkilendirme metotlarını uygulamanızın controller'ları içinde çağırmak yaygındır:
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * Verilen gönderiyi güncelle.
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if (! Gate::allows('update-post', $post)) {
            abort(403);
        }

        // Gönderiyi güncelle...

        return redirect('/posts');
    }
}
```
O anda kimliği doğrulanmış kullanıcıdan farklı bir kullanıcının bir eylemi gerçekleştirmeye yetkili olup olmadığını belirlemek isterseniz, `Gate` facade'ındaki `forUser` metodunu kullanabilirsiniz:
```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // Kullanıcı gönderiyi güncelleyebilir...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // Kullanıcı gönderiyi güncelleyemez...
}
```
`any` veya `none` metotlarını kullanarak aynı anda birden çok eylemi yetkilendirebilirsiniz:
```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // Kullanıcı gönderiyi güncelleyebilir veya silebilir...
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // Kullanıcı gönderiyi güncelleyemez veya silemez...
}
```

#### Yetkilendirme veya İstisna Fırlatma (Authorizing or Throwing Exceptions)
Bir eylemi yetkilendirmeye çalışmak ve kullanıcının belirtilen eylemi gerçekleştirmesine izin verilmiyorsa otomatik olarak bir `Illuminate\Auth\Access\AuthorizationException` fırlatmak isterseniz, `Gate` facade'ının `authorize` metodunu kullanabilirsiniz. `AuthorizationException` örnekleri, Laravel tarafından otomatik olarak 403 HTTP yanıtına dönüştürülür:
```php
Gate::authorize('update-post', $post);

// Eylem yetkilendirildi...
```

#### Ek Bağlam Sağlama (Supplying Additional Context)
Yetenekleri (abilities) yetkilendirmek için kullanılan gate metotları (`allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot`) ve yetkilendirme Blade direktifleri (`@can`, `@cannot`, `@canany`) ikinci argüman olarak bir dizi alabilir. Bu dizi öğeleri gate closure'ına parametre olarak iletilir ve yetkilendirme kararları verilirken ek bağlam için kullanılabilir:
```php
use App\Models\Category;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // Kullanıcı gönderiyi oluşturabilir...
}
```

### Gate Yanıtları (Gate Responses)
Şimdiye kadar, yalnızca basit boolean değerler döndüren gate'leri inceledik. Ancak, bazen bir hata mesajı da dahil olmak üzere daha ayrıntılı bir yanıt döndürmek isteyebilirsiniz. Bunu yapmak için, gate'inizden bir `Illuminate\Auth\Access\Response` döndürebilirsiniz:
```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('Yönetici olmalısınız.');
});
```
Gate'inizden bir yetkilendirme yanıtı döndürseniz bile, `Gate::allows` metodu yine de basit bir boolean değer döndürecektir; ancak, gate tarafından döndürülen tam yetkilendirme yanıtını almak için `Gate::inspect` metodunu kullanabilirsiniz:
```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // Eylem yetkilendirildi...
} else {
    echo $response->message();
}
```
Eylemin yetkilendirilmemesi durumunda bir `AuthorizationException` fırlatan `Gate::authorize` metodunu kullanırken, yetkilendirme yanıtı tarafından sağlanan hata mesajı HTTP yanıtına iletilecektir:
```php
Gate::authorize('edit-settings');

// Eylem yetkilendirildi...
```

#### HTTP Yanıt Durumunu Özelleştirme (Customizing The HTTP Response Status)
Bir Gate aracılığıyla bir eylem reddedildiğinde, 403 HTTP yanıtı döndürülür; ancak, bazen alternatif bir HTTP durum kodu döndürmek faydalı olabilir. `Illuminate\Auth\Access\Response` sınıfındaki `denyWithStatus` statik kurucusunu (static constructor) kullanarak başarısız bir yetkilendirme kontrolü için döndürülen HTTP durum kodunu özelleştirebilirsiniz:
```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyWithStatus(404);
});
```
Bir kaynağı 404 yanıtı aracılığıyla gizlemek web uygulamaları için çok yaygın bir desen olduğundan, kolaylık olması açısından `denyAsNotFound` metodu sunulmaktadır:
```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyAsNotFound();
});
```

### Gate Kontrollerini Engelleme (Intercepting Gate Checks)
Bazen, tüm yetenekleri belirli bir kullanıcıya vermek isteyebilirsiniz. Diğer tüm yetkilendirme kontrollerinden önce çalışan bir closure tanımlamak için `before` metodunu kullanabilirsiniz:
```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```
`before` closure'ı null olmayan bir sonuç döndürürse, bu sonuç yetkilendirme kontrolünün sonucu olarak kabul edilecektir.

Diğer tüm yetkilendirme kontrollerinden sonra yürütülecek bir closure tanımlamak için `after` metodunu kullanabilirsiniz:
```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```
`after` closure'ları tarafından döndürülen değerler, gate veya policy `null` döndürmediği sürece yetkilendirme kontrolünün sonucunu geçersiz kılmayacaktır.

### Satır İçi (Inline) Yetkilendirme
Bazen, eyleme karşılık gelen özel bir gate yazmadan, o anda kimliği doğrulanmış kullanıcının belirli bir eylemi gerçekleştirmeye yetkili olup olmadığını belirlemek isteyebilirsiniz. Laravel, bu tür "satır içi" (inline) yetkilendirme kontrollerini `Gate::allowIf` ve `Gate::denyIf` metotları aracılığıyla gerçekleştirmenize izin verir. Satır içi yetkilendirme, tanımlanmış herhangi bir "before" veya "after" yetkilendirme kancasını (hooks) çalıştırmaz:
```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn (User $user) => $user->isAdministrator());

Gate::denyIf(fn (User $user) => $user->banned());
```
Eylem yetkilendirilmemişse veya o anda kimliği doğrulanmış bir kullanıcı yoksa, Laravel otomatik olarak bir `Illuminate\Auth\Access\AuthorizationException` istisnası fırlatacaktır. `AuthorizationException` örnekleri, Laravel'in istisna işleyicisi (exception handler) tarafından otomatik olarak bir 403 HTTP yanıtına dönüştürülür.

## Politika (Policy) Oluşturma

### Politika Oluşturma (Generating Policies)
Policies, yetkilendirme mantığını belirli bir model veya kaynak etrafında düzenleyen sınıflardır. Örneğin, uygulamanız bir blog ise, bir `App\Models\Post` modeliniz ve kullanıcı eylemlerini (gönderi oluşturma veya güncelleme gibi) yetkilendirmek için buna karşılık gelen bir `App\Policies\PostPolicy` sınıfınız olabilir.

`make:policy` Artisan komutunu kullanarak bir politika oluşturabilirsiniz. Oluşturulan politika `app/Policies` dizinine yerleştirilecektir. Bu dizin uygulamanızda mevcut değilse, Laravel onu sizin için oluşturacaktır:
```bash
php artisan make:policy PostPolicy
```
`make:policy` komutu boş bir politika sınıfı oluşturacaktır. Kaynağı görüntüleme, oluşturma, güncelleme ve silme ile ilgili örnek politika metotları içeren bir sınıf oluşturmak isterseniz, komutu çalıştırırken bir `--model` seçeneği sağlayabilirsiniz:
```bash
php artisan make:policy PostPolicy --model=Post
```

### Politikaları Kaydetme (Registering Policies)

#### Politika Keşfi (Policy Discovery)
Varsayılan olarak Laravel, model ve politika standart Laravel adlandırma kurallarını (naming conventions) takip ettiği sürece politikaları otomatik olarak keşfeder. Spesifik olarak, politikalar, modellerinizi içeren dizinle aynı seviyede veya üstünde bir `Policies` dizininde bulunmalıdır. Yani, örneğin modeller `app/Models` dizinine yerleştirilirken, politikalar `app/Policies` dizinine yerleştirilebilir. Bu durumda, Laravel önce `app/Models/Policies` dizininde, ardından `app/Policies` dizininde politika arayacaktır. Ek olarak, politika adı model adıyla eşleşmeli ve bir `Policy` sonekine sahip olmalıdır. Yani, bir `User` modeli, bir `UserPolicy` politika sınıfına karşılık gelecektir.

Kendi politika keşif mantığınızı tanımlamak isterseniz, `Gate::guessPolicyNamesUsing` yöntemini kullanarak özel bir politika keşif geri çağrısı (callback) kaydedebilirsiniz. Tipik olarak, bu yöntem uygulamanızın `AppServiceProvider` sınıfının `boot` metodundan çağrılmalıdır:
```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // Verilen model için politika sınıfının adını döndür...
});
```

#### Politikaları Manuel Olarak Kaydetme (Manually Registering Policies)
`Gate` facade'ını kullanarak, uygulamanızın `AppServiceProvider` sınıfının `boot` metodu içinde politikaları ve bunlara karşılık gelen modelleri manuel olarak kaydedebilirsiniz:
```php
use App\Models\Order;
use App\Policies\OrderPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Gate::policy(Order::class, OrderPolicy::class);
}
```
Alternatif olarak, bir model sınıfına `UsePolicy` niteliğini (attribute) yerleştirerek Laravel'e modelin karşılık gelen politikasını bildirebilirsiniz:
```php
<?php

namespace App\Models;

use App\Policies\OrderPolicy;
use Illuminate\Database\Eloquent\Attributes\UsePolicy;
use Illuminate\Database\Eloquent\Model;

#[UsePolicy(OrderPolicy::class)]
class Order extends Model
{
    //
}
```

## Politika Yazma (Writing Policies)

### Politika Metotları (Policy Methods)
Politika sınıfı kaydedildikten sonra, yetkilendirdiği her eylem için metotlar ekleyebilirsiniz. Örneğin, `PostPolicy` sınıfımızda, belirli bir `App\Models\User`'ın belirli bir `App\Models\Post` örneğini güncelleyip güncelleyemeyeceğini belirleyen bir `update` metodu tanımlayalım.

`update` metodu, argüman olarak bir `User` ve bir `Post` örneği alacak ve kullanıcının verilen `Post`'u güncellemeye yetkili olup olmadığını belirten `true` veya `false` döndürmelidir. Bu nedenle, bu örnekte, kullanıcının `id`'sinin gönderideki `user_id` ile eşleşip eşleşmediğini doğrulayacağız:
```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Verilen gönderinin kullanıcı tarafından güncellenip güncellenemeyeceğini belirle.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```
Yetkilendirdiği çeşitli eylemler için gerektiğinde politika üzerinde ek metotlar tanımlamaya devam edebilirsiniz. Örneğin, çeşitli `Post` ile ilgili eylemleri yetkilendirmek için `view` veya `delete` metotları tanımlayabilirsiniz, ancak politika metotlarınıza istediğiniz adı vermekte özgür olduğunuzu unutmayın.

Politikanızı Artisan konsolu aracılığıyla oluştururken `--model` seçeneğini kullandıysanız, `viewAny`, `view`, `create`, `update`, `delete`, `restore` ve `forceDelete` eylemleri için metotları zaten içerecektir.

Tüm politikalar Laravel servis kabı (service container) aracılığıyla çözümlenir (resolved), bu da politikanın kurucusunda (constructor) ihtiyaç duyulan herhangi bir bağımlılığı tip belirtebileceğiniz ve bunların otomatik olarak enjekte edilmesini sağlayabileceğiniz anlamına gelir.

### Politika Yanıtları (Policy Responses)
Şimdiye kadar, yalnızca basit boolean değerler döndüren politika metotlarını inceledik. Ancak, bazen bir hata mesajı da dahil olmak üzere daha ayrıntılı bir yanıt döndürmek isteyebilirsiniz. Bunu yapmak için, politika metodunuzdan bir `Illuminate\Auth\Access\Response` döndürebilirsiniz:
```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('Yönetici olmalısınız.');
});
```
Politikanızdan bir yetkilendirme yanıtı döndürseniz bile, yetkilendirme kontrolleri yapmak için kullanılan metotlar (`Gate::allows` gibi) yine de basit bir boolean değer döndürecektir; ancak, politika tarafından döndürülen tam yetkilendirme yanıtını almak için `Gate::inspect` metodunu kullanabilirsiniz:
```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // Eylem yetkilendirildi...
} else {
    echo $response->message();
}
```
Eylemin yetkilendirilmemesi durumunda bir `AuthorizationException` fırlatan `Gate::authorize` metodunu kullanırken, yetkilendirme yanıtı tarafından sağlanan hata mesajı HTTP yanıtına iletilecektir.