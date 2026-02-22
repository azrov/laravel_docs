# Laravel 12 Dokümantasyonu: Eloquent: API Kaynakları (Eloquent: API Resources)

## Giriş

Bir API oluştururken, Eloquent modelleriniz ile uygulamanızın kullanıcılarına gerçekte döndürülen JSON yanıtları arasında yer alan bir dönüşüm katmanına (transformation layer) ihtiyacınız olabilir. Örneğin, belirli bir kullanıcı alt kümesi için bazı nitelikleri görüntülemek ve diğerleri için görüntülememek isteyebilir veya modellerinizin JSON temsiline her zaman belirli ilişkileri dahil etmek isteyebilirsiniz. Eloquent'in kaynak sınıfları (resource classes), modellerinizi ve model koleksiyonlarınızı anlamlı ve kolay bir şekilde JSON'a dönüştürmenize olanak tanır.

Elbette, Eloquent modellerini veya koleksiyonlarını her zaman `toJson` metotlarını kullanarak JSON'a dönüştürebilirsiniz; ancak Eloquent kaynakları, modellerinizin ve ilişkilerinin JSON serileştirmesi üzerinde daha ayrıntılı ve sağlam bir kontrol sağlar.

## Kaynak Oluşturma (Generating Resources)

Bir kaynak sınıfı oluşturmak için `make:resource` Artisan komutunu kullanabilirsiniz. Varsayılan olarak, kaynaklar uygulamanızın `app/Http/Resources` dizinine yerleştirilecektir. Kaynaklar, `Illuminate\Http\Resources\Json\JsonResource` sınıfını genişletir (extend):
```bash
php artisan make:resource UserResource
```

#### Kaynak Koleksiyonları (Resource Collections)
Tekil modelleri dönüştüren kaynaklar oluşturmanın yanı sıra, model koleksiyonlarını dönüştürmekten sorumlu kaynaklar da oluşturabilirsiniz. Bu, JSON yanıtlarınızın, belirli bir kaynağın tüm bir koleksiyonuyla ilgili bağlantılar (links) ve diğer meta bilgileri içermesine olanak tanır.

Bir kaynak koleksiyonu oluşturmak için, kaynağı oluştururken `--collection` bayrağını kullanmalısınız. Veya, kaynak adına `Collection` kelimesini dahil etmek, Laravel'e bir koleksiyon kaynağı oluşturması gerektiğini belirtecektir. Koleksiyon kaynakları, `Illuminate\Http\Resources\Json\ResourceCollection` sınıfını genişletir:
```bash
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

## Kavramsal Genel Bakış (Concept Overview)

Bu, kaynaklar ve kaynak koleksiyonları hakkında üst düzey bir genel bakıştır. Kaynaklar tarafından sunulan özelleştirme ve güç hakkında daha derin bir anlayış kazanmak için bu dokümantasyonun diğer bölümlerini okumanız şiddetle tavsiye edilir.

Kaynak yazarken size sunulan tüm seçeneklere dalmadan önce, kaynakların Laravel içinde nasıl kullanıldığına üst düzey bir bakış atalım. Bir kaynak sınıfı, JSON yapısına dönüştürülmesi gereken tek bir modeli temsil eder. Örneğin, işte basit bir `UserResource` kaynak sınıfı:
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Kaynağı bir diziye dönüştür.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```
Her kaynak sınıfı, bir rota veya controller metodundan yanıt olarak döndürüldüğünde JSON'a dönüştürülmesi gereken nitelikler dizisini döndüren bir `toArray` metodu tanımlar.

Model özelliklerine doğrudan `$this` değişkeninden erişebildiğimize dikkat edin. Bunun nedeni, bir kaynak sınıfının, kolay erişim için özellik ve metot erişimini otomatik olarak temeldeki modele yönlendirmesidir (proxy). Kaynak tanımlandıktan sonra, bir rotadan veya controller'dan döndürülebilir. Kaynak, kurucusu (constructor) aracılığıyla temeldeki model örneğini kabul eder:
```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```
Kolaylık olması açısından, modelin `toResource` metodunu kullanabilirsiniz; bu metod, framework kurallarını kullanarak modelin temeldeki kaynağını otomatik olarak keşfedecektir:
```php
return User::findOrFail($id)->toResource();
```
`toResource` metodunu çağırırken Laravel, modelin ad alanına (namespace) en yakın `Http\Resources` ad alanı içinde, model adıyla eşleşen ve isteğe bağlı olarak `Resource` sonekini içeren bir kaynak bulmaya çalışacaktır.

Kaynak sınıfınız bu adlandırma kuralını takip etmiyorsa veya farklı bir ad alanında bulunuyorsa, model için varsayılan kaynağı `UseResource` niteliğini (attribute) kullanarak belirtebilirsiniz:
```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserResource;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResource;

#[UseResource(CustomUserResource::class)]
class User extends Model
{
    // ...
}
```
Alternatif olarak, kaynak sınıfını `toResource` metoduna ileterek belirtebilirsiniz:
```php
return User::findOrFail($id)->toResource(CustomUserResource::class);
```

### Kaynak Koleksiyonları (Resource Collections)
Bir kaynak koleksiyonu veya sayfalanmış (paginated) bir yanıt döndürüyorsanız, rota veya controller'ınızda kaynak örneğini oluştururken kaynak sınıfınız tarafından sağlanan `collection` metodunu kullanmalısınız:
```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```
Veya, kolaylık olması açısından, Eloquent koleksiyonunun `toResourceCollection` metodunu kullanabilirsiniz; bu metod, framework kurallarını kullanarak modelin temeldeki kaynak koleksiyonunu otomatik olarak keşfedecektir:
```php
return User::all()->toResourceCollection();
```
`toResourceCollection` metodunu çağırırken Laravel, modelin ad alanına en yakın `Http\Resources` ad alanı içinde, model adıyla eşleşen ve `Collection` sonekini içeren bir kaynak koleksiyonu bulmaya çalışacaktır.

Kaynak koleksiyonu sınıfınız bu adlandırma kuralını takip etmiyorsa veya farklı bir ad alanında bulunuyorsa, model için varsayılan kaynak koleksiyonunu `UseResourceCollection` niteliğini kullanarak belirtebilirsiniz:
```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserCollection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResourceCollection;

#[UseResourceCollection(CustomUserCollection::class)]
class User extends Model
{
    // ...
}
```
Alternatif olarak, kaynak koleksiyonu sınıfını `toResourceCollection` metoduna ileterek belirtebilirsiniz:
```php
return User::all()->toResourceCollection(CustomUserCollection::class);
```

#### Özel Kaynak Koleksiyonları (Custom Resource Collections)
Varsayılan olarak, kaynak koleksiyonları, koleksiyonunuzla birlikte döndürülmesi gerekebilecek herhangi bir özel meta verinin eklenmesine izin vermez. Kaynak koleksiyonu yanıtını özelleştirmek isterseniz, koleksiyonu temsil etmek için özel bir kaynak oluşturabilirsiniz:
```bash
php artisan make:resource UserCollection
```
Kaynak koleksiyonu sınıfı oluşturulduktan sonra, yanıta dahil edilmesi gereken herhangi bir meta veriyi kolayca tanımlayabilirsiniz:
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Kaynak koleksiyonunu bir diziye dönüştür.
     *
     * @return array<int|string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```
Kaynak koleksiyonunuzu tanımladıktan sonra, bir rotadan veya controller'dan döndürülebilir:
```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```
Veya, kolaylık olması açısından, Eloquent koleksiyonunun `toResourceCollection` metodunu kullanabilirsiniz; bu metod, framework kurallarını kullanarak modelin temeldeki kaynak koleksiyonunu otomatik olarak keşfedecektir:
```php
return User::all()->toResourceCollection();
```
`toResourceCollection` metodunu çağırırken Laravel, modelin ad alanına en yakın `Http\Resources` ad alanı içinde, model adıyla eşleşen ve `Collection` sonekini içeren bir kaynak koleksiyonu bulmaya çalışacaktır.

#### Koleksiyon Anahtarlarını Koruma (Preserving Collection Keys)
Bir rotadan bir kaynak koleksiyonu döndürürken Laravel, koleksiyonun anahtarlarını sayısal sırada olacak şekilde sıfırlar. Ancak, bir koleksiyonun orijinal anahtarlarının korunması gerekip gerekmediğini belirten bir `preserveKeys` özelliğini kaynak sınıfınıza ekleyebilirsiniz:
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Kaynağın koleksiyon anahtarlarının korunması gerekip gerekmediğini belirtir.
     *
     * @var bool
     */
    public $preserveKeys = true;
}
```
`preserveKeys` özelliği `true` olarak ayarlandığında, koleksiyon bir rotadan veya controller'dan döndürüldüğünde koleksiyon anahtarları korunacaktır:
```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

#### Temel Kaynak Sınıfını Özelleştirme (Customizing the Underlying Resource Class)
Tipik olarak, bir kaynak koleksiyonunun `$this->collection` özelliği, koleksiyondaki her bir öğeyi kendi tekil kaynak sınıfına eşlemenin sonucuyla otomatik olarak doldurulur. Tekil kaynak sınıfının, koleksiyonun sınıf adından sondaki `Collection` kısmı çıkarılmış hali olduğu varsayılır. Ayrıca, kişisel tercihinize bağlı olarak, tekil kaynak sınıfı `Resource` sonekine sahip olabilir veya olmayabilir.

Örneğin, `UserCollection`, verilen kullanıcı örneklerini `UserResource` kaynağına eşlemeye çalışacaktır. Bu davranışı özelleştirmek için, kaynak koleksiyonunuzun `$collects` özelliğini geçersiz kılabilirsiniz (override):
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Bu kaynağın topladığı kaynak.
     *
     * @var string
     */
    public $collects = Member::class;
}
```

## Kaynak Yazma (Writing Resources)

Kavramsal genel bakışı okumadıysanız, bu dokümantasyona devam etmeden önce okumanız şiddetle tavsiye edilir.

Kaynakların yalnızca belirli bir modeli bir diziye dönüştürmesi gerekir. Bu nedenle, her kaynak, modelinizin niteliklerini uygulamanızın rotalarından veya controller'larından döndürülebilecek API dostu bir diziye çeviren bir `toArray` metodu içerir:
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Kaynağı bir diziye dönüştür.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```
Bir kaynak tanımlandıktan sonra, doğrudan bir rotadan veya controller'dan döndürülebilir:
```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toUserResource();
});
```

#### İlişkiler (Relationships)
Yanıtınıza ilgili kaynakları dahil etmek isterseniz, bunları kaynağınızın `toArray` metodu tarafından döndürülen diziye ekleyebilirsiniz. Bu örnekte, kullanıcının blog yazılarını kaynak yanıtına eklemek için `PostResource` kaynağının `collection` metodunu kullanacağız:
```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * Kaynağı bir diziye dönüştür.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
İlişkileri yalnızca daha önce yüklenmişlerse dahil etmek isterseniz, koşullu ilişkiler hakkındaki dokümantasyona göz atın.

#### Kaynak Koleksiyonları (Resource Collections)
Kaynaklar tek bir modeli bir diziye dönüştürürken, kaynak koleksiyonları bir model koleksiyonunu bir diziye dönüştürür. Ancak, tüm Eloquent model koleksiyonları, anında (on the fly) bir "geçici" (ad-hoc) kaynak koleksiyonu oluşturmak için `toResourceCollection` metodu sağladığından, her bir modeliniz için mutlaka bir kaynak koleksiyonu sınıfı tanımlamanız gerekmez:
```php
use App\Models\User;

Route::get('/users', function () {
    return User::all()->toResourceCollection();
});
```
Ancak, koleksiyonla birlikte döndürülen meta verileri özelleştirmeniz gerekiyorsa, kendi kaynak koleksiyonunuzu tanımlamanız gerekir:
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Kaynak koleksiyonunu bir diziye dönüştür.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```
Tekil kaynaklar gibi, kaynak koleksiyonları da doğrudan rotalardan veya controller'lardan döndürülebilir:
```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```
Veya, kolaylık olması açısından, Eloquent koleksiyonunun `toResourceCollection` metodunu kullanabilirsiniz; bu metod, framework kurallarını kullanarak modelin temeldeki kaynak koleksiyonunu otomatik olarak keşfedecektir:
```php
return User::all()->toResourceCollection();
```
`toResourceCollection` metodunu çağırırken Laravel, modelin ad alanına en yakın `Http\Resources` ad alanı içinde, model adıyla eşleşen ve `Collection` sonekini içeren bir kaynak koleksiyonu bulmaya çalışacaktır.

### Veri Sarma (Data Wrapping)
Varsayılan olarak, kaynak yanıtı JSON'a dönüştürüldüğünde en dıştaki kaynağınız bir `data` anahtarı içine sarılır (wrapped). Yani, örneğin, tipik bir kaynak koleksiyonu yanıtı aşağıdaki gibi görünür:
```json
{
    "data": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com"
        },
        {
            "id": 2,
            "name": "Jane Doe",
            "email": "jane@example.com"
        }
    ]
}
```
Tek bir kaynak döndürüyorsanız, kaynak `data` anahtarına sarılmaz. Ancak, özel bir kaynak koleksiyonu sınıfı kullanıyorsanız, kaynağınızın `toArray` metodunda `data` anahtarını manuel olarak döndürmeniz gerekir.

Kaynağınızın en dıştaki sarmasını devre dışı bırakmak isterseniz, kaynak sınıfınıza `withoutWrapping` metodunu çağırmalısınız. Tipik olarak, bu metodu uygulamanızın `AppServiceProvider` sınıfının `boot` metodundan çağırmalısınız:
```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        JsonResource::withoutWrapping();
    }
}
```
`withoutWrapping` metodu yalnızca en dıştaki sarmayı etkiler ve kendi kaynak koleksiyonlarınızın `toArray` metoduna manuel olarak eklediğiniz `data` anahtarlarını kaldırmaz.