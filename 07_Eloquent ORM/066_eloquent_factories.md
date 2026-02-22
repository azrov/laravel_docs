# Laravel 12 Dokümantasyonu: Eloquent: Fabrikalar (Eloquent: Factories)

## Giriş

Uygulamanızı test ederken veya veritabanınızı tohumlarken (seeding), veritabanınıza birkaç kayıt eklemeniz gerekebilir. Laravel, her bir sütunun değerini manuel olarak belirtmek yerine, model fabrikalarını (model factories) kullanarak Eloquent modellerinizin her biri için bir dizi varsayılan nitelik tanımlamanıza olanak tanır.

Bir fabrikanın nasıl yazılacağına dair bir örnek görmek için uygulamanızdaki `database/factories/UserFactory.php` dosyasına bakın. Bu fabrika, tüm yeni Laravel uygulamalarıyla birlikte gelir ve aşağıdaki fabrika tanımını içerir:
```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
 */
class UserFactory extends Factory
{
    /**
     * Fabrika tarafından kullanılan geçerli parola.
     */
    protected static ?string $password;

    /**
     * Modelin varsayılan durumunu tanımla.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }

    /**
     * Modelin e-posta adresinin doğrulanmamış olması gerektiğini belirt.
     */
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}
```
Gördüğünüz gibi, en temel halleriyle fabrikalar, Laravel'in temel fabrika sınıfını (base factory class) genişleten ve bir `definition` metodu tanımlayan sınıflardır. `definition` metodu, fabrika kullanılarak bir model oluşturulurken uygulanması gereken varsayılan nitelik değerleri kümesini döndürür.

`fake` yardımcısı aracılığıyla fabrikalar, test ve tohumlama için çeşitli türlerde rastgele veriler oluşturmanıza olanak tanıyan Faker PHP kütüphanesine erişebilir.

Uygulamanızın Faker yerel ayarını (locale), `config/app.php` yapılandırma dosyasındaki `faker_locale` seçeneğini güncelleyerek değiştirebilirsiniz.

## Model Fabrikalarını Tanımlama (Defining Model Factories)

### Fabrika Oluşturma (Generating Factories)
Bir fabrika oluşturmak için `make:factory` Artisan komutunu çalıştırın:
```bash
php artisan make:factory PostFactory
```
Yeni fabrika sınıfı, `database/factories` dizininize yerleştirilecektir.

#### Model ve Fabrika Keşif Kuralları (Model and Factory Discovery Conventions)
Fabrikalarınızı tanımladıktan sonra, o model için bir fabrika örneği oluşturmak amacıyla modellerinize `Illuminate\Database\Eloquent\Factories\HasFactory` özelliği (trait) tarafından sağlanan statik `factory` metodunu kullanabilirsiniz.

`HasFactory` özelliğinin `factory` metodu, özelliğin atandığı model için uygun fabrikayı belirlemek üzere kuralları (conventions) kullanacaktır. Spesifik olarak, metod `Database\Factories` ad alanında (namespace), model adıyla eşleşen ve `Factory` sonekiyle biten bir sınıf adına sahip bir fabrika arayacaktır. Bu kurallar belirli uygulamanız veya fabrikanız için geçerli değilse, modele `UseFactory` niteliğini (attribute) ekleyerek modelin fabrikasını manuel olarak belirtebilirsiniz:
```php
use Illuminate\Database\Eloquent\Attributes\UseFactory;
use Database\Factories\Administration\FlightFactory;

#[UseFactory(FlightFactory::class)]
class Flight extends Model
{
    // ...
}
```
Alternatif olarak, modelinizde `newFactory` metodunu, modelin ilgili fabrikasının bir örneğini doğrudan döndürecek şekilde geçersiz kılabilirsiniz (override):
```php
use Database\Factories\Administration\FlightFactory;

/**
 * Model için yeni bir fabrika örneği oluştur.
 */
protected static function newFactory()
{
    return FlightFactory::new();
}
```
Ardından, ilgili fabrikada bir `model` özelliği tanımlayın:
```php
use App\Administration\Flight;
use Illuminate\Database\Eloquent\Factories\Factory;

class FlightFactory extends Factory
{
    /**
     * Fabrikanın karşılık geldiği modelin adı.
     *
     * @var class-string<\Illuminate\Database\Eloquent\Model>
     */
    protected $model = Flight::class;
}
```

### Fabrika Durumları (Factory States)
Durum manipülasyon metotları (state manipulation methods), model fabrikalarınıza herhangi bir kombinasyonda uygulanabilecek ayrık değişiklikler tanımlamanıza olanak tanır. Örneğin, `Database\Factories\UserFactory` fabrikanız, varsayılan nitelik değerlerinden birini değiştiren bir `suspended` durum metodu içerebilir.

Durum dönüşüm metotları tipik olarak Laravel'in temel fabrika sınıfı tarafından sağlanan `state` metodunu çağırır. `state` metodu, fabrika için tanımlanan ham niteliklerin dizisini alacak ve değiştirilecek niteliklerin bir dizisini döndürmesi gereken bir closure kabul eder:
```php
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Kullanıcının askıya alındığını belirt.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    });
}
```

#### "Silinmiş" (Trashed) Durumu
Eloquent modeliniz yazılı olarak silinebiliyorsa (soft delete), oluşturulan modelin zaten "yazılı olarak silinmiş" olması gerektiğini belirtmek için yerleşik `trashed` durum metodunu çağırabilirsiniz. `trashed` durumunu manuel olarak tanımlamanız gerekmez çünkü tüm fabrikalarda otomatik olarak bulunur:
```php
use App\Models\User;

$user = User::factory()->trashed()->create();
```

### Fabrika Geri Çağrıları (Factory Callbacks)
Fabrika geri çağrıları, `afterMaking` ve `afterCreating` metotları kullanılarak kaydedilir ve bir model oluşturduktan (make) veya yarattıktan (create) sonra ek görevler gerçekleştirmenize olanak tanır. Bu geri çağrıları, fabrika sınıfınızda bir `configure` metodu tanımlayarak kaydetmelisiniz. Bu metod, fabrika örneklendiğinde (instantiated) Laravel tarafından otomatik olarak çağrılacaktır:
```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    /**
     * Model fabrikasını yapılandır.
     */
    public function configure(): static
    {
        return $this->afterMaking(function (User $user) {
            // ...
        })->afterCreating(function (User $user) {
            // ...
        });
    }

    // ...
}
```

Belirli bir duruma özgü ek görevler gerçekleştirmek için fabrika geri çağrılarını durum metotları içinde de kaydedebilirsiniz:
```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Kullanıcının askıya alındığını belirt.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    })->afterMaking(function (User $user) {
        // ...
    })->afterCreating(function (User $user) {
        // ...
    });
}
```

## Fabrikaları Kullanarak Model Oluşturma (Creating Models Using Factories)

### Model Örnekleri Oluşturma (Instantiating Models)
Fabrikalarınızı tanımladıktan sonra, o model için bir fabrika örneği oluşturmak amacıyla modellerinize `Illuminate\Database\Eloquent\Factories\HasFactory` özelliği tarafından sağlanan statik `factory` metodunu kullanabilirsiniz. Model oluşturmanın birkaç örneğine bakalım. İlk olarak, modelleri veritabanına kaydetmeden (persist) oluşturmak için `make` metodunu kullanacağız:
```php
use App\Models\User;

$user = User::factory()->make();
```
`count` metodunu kullanarak birden çok modelden oluşan bir koleksiyon oluşturabilirsiniz:
```php
$users = User::factory()->count(3)->make();
```

#### Durumları Uygulama (Applying States)
Ayrıca durumlarınızdan herhangi birini modellere uygulayabilirsiniz. Modellere birden çok durum dönüşümü uygulamak isterseniz, durum dönüşüm metotlarını doğrudan çağırmanız yeterlidir:
```php
$users = User::factory()->count(5)->suspended()->make();
```

#### Nitelikleri Geçersiz Kılma (Overriding Attributes)
Modellerinizin bazı varsayılan değerlerini geçersiz kılmak isterseniz, `make` metoduna bir değerler dizisi iletebilirsiniz. Yalnızca belirtilen nitelikler değiştirilecek, niteliklerin geri kalanı fabrika tarafından belirtilen varsayılan değerlerinde kalacaktır:
```php
$user = User::factory()->make([
    'name' => 'Abigail Otwell',
]);
```
Alternatif olarak, satır içi (inline) bir durum dönüşümü gerçekleştirmek için `state` metodu doğrudan fabrika örneği üzerinde çağrılabilir:
```php
$user = User::factory()->state([
    'name' => 'Abigail Otwell',
])->make();
```
Fabrikalar kullanılarak model oluşturulurken toplu atama koruması (mass assignment protection) otomatik olarak devre dışı bırakılır.

### Modelleri Kaydetme (Persisting Models)
`create` metodu, model örneklerini oluşturur ve Eloquent'in `save` metodunu kullanarak bunları veritabanına kaydeder (persists):
```php
use App\Models\User;

// Tek bir App\Models\User örneği oluştur...
$user = User::factory()->create();

// Üç App\Models\User örneği oluştur...
$users = User::factory()->count(3)->create();
```
`create` metoduna bir nitelikler dizisi ileterek fabrikanın varsayılan model niteliklerini geçersiz kılabilirsiniz:
```php
$user = User::factory()->create([
    'name' => 'Abigail',
]);
```

### Diziler (Sequences)
Bazen, oluşturulan her model için belirli bir model niteliğinin değerini değiştirmek isteyebilirsiniz. Bunu, bir durum dönüşümünü bir dizi (sequence) olarak tanımlayarak gerçekleştirebilirsiniz. Örneğin, oluşturulan her kullanıcı için bir `admin` sütununun değerini `Y` ve `N` arasında değiştirmek isteyebilirsiniz:
```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```
Bu örnekte, beş kullanıcı `admin` değeri `Y` olarak ve beş kullanıcı `admin` değeri `N` olarak oluşturulacaktır.

Gerekirse, dizi değeri olarak bir closure ekleyebilirsiniz. Closure, dizinin yeni bir değere ihtiyacı olduğu her seferinde çağrılacaktır:
```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
    ))
    ->create();
```
Bir dizi closure'ı içinde, closure'a enjekte edilen dizi örneğindeki `$index` özelliğine erişebilirsiniz. `$index` özelliği, o ana kadar dizide gerçekleşen yineleme (iteration) sayısını içerir:
```php
$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index],
    ))
    ->create();
```
Kolaylık olması açısından, diziler `sequence` metodu kullanılarak da uygulanabilir; bu metod dahili olarak `state` metodunu çağırır. `sequence` metodu bir closure veya sıralanmış nitelikler dizileri kabul eder:
```php
$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User'],
        ['name' => 'Second User'],
    )
    ->create();
```

## Fabrika İlişkileri (Factory Relationships)

### Has Many (Bire Çok) İlişkileri
Şimdi, Laravel'in akıcı fabrika metotlarını kullanarak Eloquent model ilişkileri oluşturmayı keşfedelim. İlk olarak, uygulamamızın bir `App\Models\User` modeli ve bir `App\Models\Post` modeli olduğunu varsayalım. Ayrıca, `User` modelinin `Post` ile bir `hasMany` ilişkisi tanımladığını varsayalım. Laravel fabrikaları tarafından sağlanan `has` metodunu kullanarak üç gönderisi olan bir kullanıcı oluşturabiliriz. `has` metodu bir fabrika örneği kabul eder:
```php
use App\Models\Post;
use App\Models\User;

$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();
```
Kural gereği, `has` metoduna bir `Post` modeli iletildiğinde, Laravel `User` modelinin ilişkiyi tanımlayan bir `posts` metoduna sahip olması gerektiğini varsayacaktır. Gerekirse, işlemek istediğiniz ilişkinin adını açıkça belirtebilirsiniz:
```php
$user = User::factory()
    ->has(Post::factory()->count(3), 'posts')
    ->create();
```
Elbette, ilgili modeller üzerinde durum manipülasyonları gerçekleştirebilirsiniz. Ayrıca, durum değişikliğiniz üst modele erişim gerektiriyorsa, closure tabanlı bir durum dönüşümü iletebilirsiniz:
```php
$user = User::factory()
    ->has(
        Post::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['user_type' => $user->type];
            })
    )
    ->create();
```

#### Sihirli Metotları (Magic Methods) Kullanma
Kolaylık olması açısından, ilişkiler oluşturmak için Laravel'in sihirli fabrika ilişki metotlarını kullanabilirsiniz. Örneğin, aşağıdaki örnek, ilgili modellerin `User` modelinde bir `posts` ilişki metodu aracılığıyla oluşturulması gerektiğini belirlemek için kuralı kullanacaktır:
```php
$user = User::factory()
    ->hasPosts(3)
    ->create();
```
Fabrika ilişkileri oluşturmak için sihirli metotlar kullanırken, ilgili modellerde geçersiz kılmak üzere bir nitelikler dizisi iletebilirsiniz:
```php
$user = User::factory()
    ->hasPosts(3, [
        'published' => false,
    ])
    ->create();
```
Durum değişikliğiniz üst modele erişim gerektiriyorsa, closure tabanlı bir durum dönüşümü sağlayabilirsiniz:
```php
$user = User::factory()
    ->hasPosts(3, function (array $attributes, User $user) {
        return ['user_type' => $user->type];
    })
    ->create();
```

### Belongs To (Ait Olan) İlişkileri
Artık fabrikaları kullanarak "has many" ilişkilerini nasıl oluşturacağımızı keşfettiğimize göre, ilişkinin tersini inceleyelim. `for` metodu, fabrika tarafından oluşturulan modellerin ait olduğu üst modeli tanımlamak için kullanılabilir. Örneğin, tek bir kullanıcıya ait üç `App\Models\Post` model örneği oluşturabiliriz:
```php
use App\Models\Post;
use App\Models\User;

$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state([
        'name' => 'Jessica Archer',
    ]))
    ->create();
```
Oluşturduğunuz modellerle ilişkilendirilmesi gereken zaten bir üst model örneğiniz varsa, model örneğini `for` metoduna iletebilirsiniz:
```php
$user = User::factory()->create();

$posts = Post::factory()
    ->count(3)
    ->for($user)
    ->create();
```

#### Sihirli Metotları (Magic Methods) Kullanma
Kolaylık olması açısından, "belongs to" ilişkilerini tanımlamak için Laravel'in sihirli fabrika ilişki metotlarını kullanabilirsiniz. Örneğin, aşağıdaki örnek, üç gönderinin `Post` modelindeki `user` ilişkisine ait olması gerektiğini belirlemek için kuralı kullanacaktır:
```php
$posts = Post::factory()
    ->count(3)
    ->forUser([
        'name' => 'Jessica Archer',
    ])
    ->create();
```

### Many to Many (Çoka Çok) İlişkileri
Has many ilişkileri gibi, "many to many" ilişkileri de `has` metodu kullanılarak oluşturulabilir:
```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->has(Role::factory()->count(3))
    ->create();
```

#### Pivot Tablo Nitelikleri (Pivot Table Attributes)
Modeller arasındaki ilişkiyi tanımlayan pivot (ara) tabloda ayarlanması gereken nitelikler tanımlamanız gerekiyorsa, `hasAttached` metodunu kullanabilirsiniz. Bu metod, pivot tablosunda ayarlanması gereken bir nitelikler dizisini ikinci argüman olarak kabul eder:
```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```
Durum değişikliğiniz pivot nitelikleri için erişim gerektiriyorsa, closure tabanlı bir durum dönüşümü sağlayabilirsiniz:
```php
$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        function (array $attributes, User $user) {
            return ['active' => $user->active];
        }
    )
    ->create();
```

#### Sihirli Metotları (Magic Methods) Kullanma
Kolaylık olması açısından, "many to many" ilişkileri tanımlamak için Laravel'in sihirli fabrika ilişki metotlarını kullanabilirsiniz. Örneğin, aşağıdaki örnek, `User` modelindeki `roles` ilişki metodunu kullanacaktır:
```php
$user = User::factory()
    ->hasRoles(3, ['active' => true])
    ->create();
```