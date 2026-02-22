# Laravel 12 Dokümantasyonu: Eloquent: Koleksiyonlar (Eloquent: Collections)

## Giriş

`get` metodu aracılığıyla alınan veya bir ilişki (relationship) yoluyla erişilen sonuçlar da dahil olmak üzere, birden fazla model sonucu döndüren tüm Eloquent metotları, `Illuminate\Database\Eloquent\Collection` sınıfının örneklerini (instances) döndürecektir. Eloquent koleksiyon nesnesi, Laravel'in temel koleksiyonunu (base collection) genişletir (extend), bu nedenle temeldeki Eloquent modelleri dizisiyle akıcı bir şekilde çalışmak için kullanılan düzinelerce metodu doğal olarak miras alır. Bu yararlı metotların tümünü öğrenmek için Laravel koleksiyon dokümantasyonunu incelemeyi unutmayın!

Tüm koleksiyonlar ayrıca yineleyiciler (iterators) olarak da hizmet eder, böylece onlar üzerinde basit PHP dizileriymiş gibi döngü kurabilirsiniz:
```php
use App\Models\User;

$users = User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```
Ancak, daha önce belirtildiği gibi, koleksiyonlar dizilerden çok daha güçlüdür ve sezgisel bir arayüz kullanılarak zincirlenebilen (chained) çeşitli haritalama / indirgeme (map/reduce) işlemlerini kullanıma sunar. Örneğin, tüm aktif olmayan modelleri kaldırabilir ve ardından kalan her kullanıcı için adı toplayabiliriz:
```php
$names = User::all()->reject(function (User $user) {
    return $user->active === false;
})->map(function (User $user) {
    return $user->name;
});
```

#### Eloquent Koleksiyon Dönüşümü (Eloquent Collection Conversion)
Çoğu Eloquent koleksiyon metodu yeni bir Eloquent koleksiyon örneği döndürürken, `collapse`, `flatten`, `flip`, `keys`, `pluck` ve `zip` metotları bir temel koleksiyon (base collection) örneği döndürür. Benzer şekilde, bir `map` işlemi herhangi bir Eloquent modeli içermeyen bir koleksiyon döndürürse, bu bir temel koleksiyon örneğine dönüştürülecektir.

## Mevcut Metotlar (Available Methods)

Tüm Eloquent koleksiyonları, temel Laravel koleksiyon nesnesini (base Laravel collection object) genişletir; bu nedenle, temel koleksiyon sınıfı tarafından sağlanan tüm güçlü metotları miras alırlar.

Ek olarak, `Illuminate\Database\Eloquent\Collection` sınıfı, model koleksiyonlarınızı yönetmeye yardımcı olacak bir dizi ek metot (superset of methods) sağlar. Çoğu metod `Illuminate\Database\Eloquent\Collection` örnekleri döndürür; ancak, `modelKeys` gibi bazı metotlar bir `Illuminate\Support\Collection` örneği döndürür.

#### `append($attributes)`
`append` metodu, koleksiyondaki her model için bir niteliğin (attribute) eklenmesi gerektiğini belirtmek için kullanılabilir. Bu metod, bir nitelik dizisi veya tek bir nitelik kabul eder:
```php
$users->append('team');

$users->append(['team', 'is_admin']);
```

#### `contains($key, $operator = null, $value = null)`
`contains` metodu, belirli bir model örneğinin koleksiyon tarafından içerilip içerilmediğini belirlemek için kullanılabilir. Bu metod, bir birincil anahtar (primary key) veya bir model örneği kabul eder:
```php
$users->contains(1);

$users->contains(User::find(1));
```

#### `diff($items)`
`diff` metodu, verilen koleksiyonda bulunmayan tüm modelleri döndürür:
```php
use App\Models\User;

$users = $users->diff(User::whereIn('id', [1, 2, 3])->get());
```

#### `except($keys)`
`except` metodu, verilen birincil anahtarlara sahip olmayan tüm modelleri döndürür:
```php
$users = $users->except([1, 2, 3]);
```

#### `find($key)`
`find` metodu, birincil anahtarı verilen anahtarla eşleşen modeli döndürür. Eğer `$key` bir model örneği ise, `find` birincil anahtarı eşleşen bir model döndürmeye çalışacaktır. Eğer `$key` bir anahtarlar dizisi ise, `find` birincil anahtarı verilen dizide olan tüm modelleri döndürecektir:
```php
$users = User::all();

$user = $users->find(1);
```

#### `findOrFail($key)`
`findOrFail` metodu, birincil anahtarı verilen anahtarla eşleşen modeli döndürür veya koleksiyonda eşleşen bir model bulunamazsa bir `Illuminate\Database\Eloquent\ModelNotFoundException` istisnası (exception) fırlatır:
```php
$users = User::all();

$user = $users->findOrFail(1);
```

#### `fresh($with = [])`
`fresh` metodu, koleksiyondaki her bir modelin veritabanından yeni bir örneğini alır. Ayrıca, belirtilen tüm ilişkiler istekli olarak yüklenecektir (eager loaded):
```php
$users = $users->fresh();

$users = $users->fresh('comments');
```

#### `intersect($items)`
`intersect` metodu, verilen koleksiyonda da bulunan tüm modelleri döndürür:
```php
use App\Models\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

#### `load($relations)`
`load` metodu, koleksiyondaki tüm modeller için verilen ilişkileri istekli olarak yükler:
```php
$users->load(['comments', 'posts']);

$users->load('comments.author');

$users->load(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

#### `loadMissing($relations)`
`loadMissing` metodu, eğer ilişkiler zaten yüklenmemişse, koleksiyondaki tüm modeller için verilen ilişkileri istekli olarak yükler:
```php
$users->loadMissing(['comments', 'posts']);

$users->loadMissing('comments.author');

$users->loadMissing(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

#### `modelKeys()`
`modelKeys` metodu, koleksiyondaki tüm modellerin birincil anahtarlarını döndürür:
```php
$users->modelKeys();

// [1, 2, 3, 4, 5]
```

#### `makeVisible($attributes)`
`makeVisible` metodu, koleksiyondaki her modelde normalde "gizli" (hidden) olan nitelikleri görünür (visible) hale getirir:
```php
$users = $users->makeVisible(['address', 'phone_number']);
```

#### `makeHidden($attributes)`
`makeHidden` metodu, koleksiyondaki her modelde normalde "görünür" (visible) olan nitelikleri gizler:
```php
$users = $users->makeHidden(['address', 'phone_number']);
```

#### `mergeVisible($attributes)`
`mergeVisible` metodu, mevcut görünür nitelikleri korurken ek nitelikleri görünür hale getirir:
```php
$users = $users->mergeVisible(['middle_name']);
```

#### `mergeHidden($attributes)`
`mergeHidden` metodu, mevcut gizli nitelikleri korurken ek nitelikleri gizler:
```php
$users = $users->mergeHidden(['last_login_at']);
```

#### `only($keys)`
`only` metodu, verilen birincil anahtarlara sahip olan tüm modelleri döndürür:
```php
$users = $users->only([1, 2, 3]);
```

#### `partition`
`partition` metodu, `Illuminate\Database\Eloquent\Collection` koleksiyon örnekleri içeren bir `Illuminate\Support\Collection` örneği döndürür:
```php
$partition = $users->partition(fn ($user) => $user->age > 18);

dump($partition::class);    // Illuminate\Support\Collection
dump($partition[0]::class); // Illuminate\Database\Eloquent\Collection
dump($partition[1]::class); // Illuminate\Database\Eloquent\Collection
```

#### `setAppends($attributes)`
`setAppends` metodu, koleksiyondaki her modelde eklenen (appended) tüm nitelikleri geçici olarak geçersiz kılar:
```php
$users = $users->setAppends(['is_admin']);
```

#### `setVisible($attributes)`
`setVisible` metodu, koleksiyondaki her modelde tüm görünür nitelikleri geçici olarak geçersiz kılar:
```php
$users = $users->setVisible(['id', 'name']);
```

#### `setHidden($attributes)`
`setHidden` metodu, koleksiyondaki her modelde tüm gizli nitelikleri geçici olarak geçersiz kılar:
```php
$users = $users->setHidden(['email', 'password', 'remember_token']);
```

#### `toQuery()`
`toQuery` metodu, koleksiyon modelinin birincil anahtarlarında bir `whereIn` kısıtlaması içeren bir Eloquent sorgu oluşturucu (query builder) örneği döndürür:
```php
use App\Models\User;

$users = User::where('status', 'VIP')->get();

$users->toQuery()->update([
    'status' => 'Administrator',
]);
```

#### `unique($key = null, $strict = false)`
`unique` metodu, koleksiyondaki tüm benzersiz modelleri döndürür. Koleksiyondaki başka bir modelle aynı birincil anahtara sahip tüm modeller kaldırılır:
```php
$users = $users->unique();
```

#### `withoutAppends()`
`withoutAppends` metodu, koleksiyondaki her modelde eklenen (appended) tüm nitelikleri geçici olarak kaldırır:
```php
$users = $users->withoutAppends();
```

## Özel Koleksiyonlar (Custom Collections)

Belirli bir modelle etkileşime girerken özel bir `Collection` nesnesi kullanmak isterseniz, modelinize `CollectedBy` niteliğini (attribute) ekleyebilirsiniz:
```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Attributes\CollectedBy;
use Illuminate\Database\Eloquent\Model;

#[CollectedBy(UserCollection::class)]
class User extends Model
{
    // ...
}
```
Alternatif olarak, modelinizde bir `newCollection` metodu tanımlayabilirsiniz:
```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Yeni bir Eloquent Collection örneği oluştur.
     *
     * @param  array<int, \Illuminate\Database\Eloquent\Model>  $models
     * @return \Illuminate\Database\Eloquent\Collection<int, \Illuminate\Database\Eloquent\Model>
     */
    public function newCollection(array $models = []): Collection
    {
        $collection = new UserCollection($models);

        if (Model::isAutomaticallyEagerLoadingRelationships()) {
            $collection->withRelationshipAutoloading();
        }

        return $collection;
    }
}
```
Modelinizde bir `newCollection` metodu tanımladıktan veya `CollectedBy` niteliğini ekledikten sonra, Eloquent normalde bir `Illuminate\Database\Eloquent\Collection` örneği döndüreceği her zaman özel koleksiyonunuzun bir örneğini alacaksınız.

Uygulamanızdaki her model için özel bir koleksiyon kullanmak isterseniz, uygulamanızın tüm modelleri tarafından genişletilen bir temel model sınıfında (base model class) `newCollection` metodunu tanımlamalısınız.