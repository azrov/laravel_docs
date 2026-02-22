# Laravel 12 Dokümantasyonu: Eloquent: İlişkiler (Eloquent: Relationships)

## Giriş

Veritabanı tabloları genellikle birbiriyle ilişkilidir. Örneğin, bir blog yazısının birçok yorumu olabilir veya bir sipariş, onu veren kullanıcıyla ilişkilendirilebilir. Eloquent, bu ilişkileri yönetmeyi ve onlarla çalışmayı kolaylaştırır ve bir dizi yaygın ilişki türünü destekler:

*   Bire Bir (One To One)
*   Bire Çok (One To Many)
*   Çoka Çok (Many To Many)
*   Uzaktan Bire Çok (Has Many Through)
*   Çok Biçimli İlişkiler (Polymorphic Relations)
*   Çok Biçimli Bire Çok (Many To Many Polymorphic Relations)

## İlişkileri Tanımlama (Defining Relationships)

Eloquent ilişkileri, Eloquent model sınıflarınızda metotlar olarak tanımlanır. İlişkiler aynı zamanda güçlü sorgu oluşturucular (query builders) olarak da hizmet ettiğinden, bunları metot olarak tanımlamak güçlü metot zincirleme (method chaining) ve sorgulama yetenekleri sağlar. Örneğin, bu `posts` ilişkisine ek sorgu kısıtlamaları ekleyebiliriz:
```php
$user->posts()->where('active', 1)->get();
```
Ancak, ilişkileri kullanmaya çok derinlemesine dalmadan önce, Eloquent tarafından desteklenen her bir ilişki türünün nasıl tanımlanacağını öğrenelim.

### Bire Bir / Has One (One to One / Has One)

Bire bir ilişki, çok temel bir veritabanı ilişkisi türüdür. Örneğin, bir `User` modeli bir `Phone` modeliyle ilişkilendirilebilir. Bu ilişkiyi tanımlamak için, `User` modeline bir `phone` metodu yerleştireceğiz. `phone` metodu, `hasOne` metodunu çağırmalı ve sonucunu döndürmelidir. `hasOne` metodu, modelinize `Illuminate\Database\Eloquent\Model` temel sınıfı (base class) aracılığıyla sunulur:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Kullanıcıyla ilişkili telefonu al.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```
`hasOne` metoduna iletilen ilk argüman, ilişkili model sınıfının adıdır. İlişki tanımlandıktan sonra, Eloquent'in dinamik özelliklerini (dynamic properties) kullanarak ilişkili kaydı alabiliriz. Dinamik özellikler, ilişki metotlarına, modelde tanımlanmış özelliklermiş gibi erişmenize olanak tanır:
```php
$phone = User::find(1)->phone;
```
Eloquent, ilişkinin yabancı anahtarını (foreign key) üst model adına göre belirler. Bu durumda, `Phone` modelinin otomatik olarak bir `user_id` yabancı anahtarına sahip olduğu varsayılır. Bu kuralı geçersiz kılmak isterseniz, `hasOne` metoduna ikinci bir argüman iletebilirsiniz:
```php
return $this->hasOne(Phone::class, 'foreign_key');
```
Ek olarak Eloquent, yabancı anahtarın, üst öğenin birincil anahtar sütunuyla eşleşen bir değere sahip olması gerektiğini varsayar. Başka bir deyişle Eloquent, `Phone` kaydının `user_id` sütununda kullanıcının `id` sütununun değerini arayacaktır. İlişkinin, `id` veya modelinizin `$primaryKey` özelliği dışında bir birincil anahtar değeri kullanmasını istiyorsanız, `hasOne` metoduna üçüncü bir argüman iletebilirsiniz:
```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

#### İlişkinin Tersini Tanımlama (Defining the Inverse of the Relationship)
Böylece, `User` modelimizden `Phone` modeline erişebiliriz. Şimdi, `Phone` modeli üzerinde, telefonun sahibi olan kullanıcıya erişmemizi sağlayacak bir ilişki tanımlayalım. Bir `hasOne` ilişkisinin tersini `belongsTo` metodunu kullanarak tanımlayabiliriz:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Telefona sahip olan kullanıcıyı al.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```
`user` metodunu çağırdığımızda, Eloquent `id`'si `Phone` modelindeki `user_id` sütunuyla eşleşen bir `User` modeli bulmaya çalışacaktır.

Eloquent, yabancı anahtar adını, ilişki metodunun adını inceleyerek ve metod adına `_id` ekleyerek belirler. Yani, bu durumda Eloquent, `Phone` modelinin bir `user_id` sütununa sahip olduğunu varsayar. Ancak, `Phone` modelindeki yabancı anahtar `user_id` değilse, `belongsTo` metoduna ikinci argüman olarak özel bir anahtar adı iletebilirsiniz:
```php
/**
 * Telefona sahip olan kullanıcıyı al.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```
Üst model birincil anahtar olarak `id` kullanmıyorsa veya ilişkili modeli farklı bir sütun kullanarak bulmak istiyorsanız, üst tablonun özel anahtarını belirterek `belongsTo` metoduna üçüncü bir argüman iletebilirsiniz:
```php
/**
 * Telefona sahip olan kullanıcıyı al.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

### Bire Çok / Has Many (One to Many / Has Many)

Bire çok ilişki, tek bir modelin bir veya daha fazla alt modelin üst öğesi olduğu durumları tanımlamak için kullanılır. Örneğin, bir blog yazısının sınırsız sayıda yorumu olabilir. Diğer tüm Eloquent ilişkileri gibi, bire çok ilişkiler de Eloquent modelinizde bir metot tanımlanarak oluşturulur:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Blog yazısının yorumlarını al.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```
Unutmayın, Eloquent `Comment` modeli için uygun yabancı anahtar sütununu otomatik olarak belirleyecektir. Kural gereği Eloquent, üst modelin "yılan_case" (snake case) adını alır ve buna `_id` ekler. Yani, bu örnekte Eloquent, `Comment` modelindeki yabancı anahtar sütununun `post_id` olduğunu varsayacaktır.

İlişki metodu tanımlandıktan sonra, `comments` özelliğine erişerek ilişkili yorumların koleksiyonuna erişebiliriz. Unutmayın, Eloquent "dinamik ilişki özellikleri" sağladığı için, ilişki metotlarına modelde tanımlanmış özelliklermiş gibi erişebiliriz:
```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```
Tüm ilişkiler aynı zamanda sorgu oluşturucu görevi gördüğünden, `comments` metodunu çağırarak ve sorguya koşullar eklemeye devam ederek ilişki sorgusuna daha fazla kısıtlama ekleyebilirsiniz:
```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```
`hasOne` metodu gibi, `hasMany` metoduna ek argümanlar ileterek yabancı ve yerel anahtarları da geçersiz kılabilirsiniz:
```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

#### Alt Modellerde Üst Modelleri Otomatik Olarak Doldurma (Automatically Hydrating Parent Models on Children)
Eloquent'in istekli yüklemesini (eager loading) kullanırken bile, alt modeller arasında döngü yaparken bir alt modelden üst modele erişmeye çalışırsanız "N + 1" sorgu sorunları ortaya çıkabilir:
```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```
Yukarıdaki örnekte, her `Post` modeli için yorumlar istekli olarak yüklenmiş olsa bile, Eloquent her bir alt `Comment` modelinde üst `Post` modelini otomatik olarak doldurmadığı (hydrate) için bir "N + 1" sorgu sorunu ortaya çıkmıştır.

Eloqeunt'in alt modellerine üst modelleri otomatik olarak doldurmasını istiyorsanız, bir `hasMany` ilişkisi tanımlarken `chaperone` (refakatçi) metodunu çağırabilirsiniz:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Blog yazısının yorumlarını al.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```
Veya, çalışma zamanında otomatik üst doldurmayı tercih etmek (opt-in) isterseniz, ilişkiyi istekli olarak yüklerken `chaperone` modelini çağırabilirsiniz:
```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

### Bire Çok (Tersi) / Belongs To (One to Many (Inverse) / Belongs To)

Artık bir yazının tüm yorumlarına erişebildiğimize göre, bir yorumun sahibi olan yazıya erişmesine izin veren bir ilişki tanımlayalım. Bir `hasMany` ilişkisinin tersini tanımlamak için, alt model üzerinde `belongsTo` metodunu çağıran bir ilişki metodu tanımlayın:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * Yoruma sahip olan yazıyı al.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```
İlişki tanımlandıktan sonra, `post` "dinamik ilişki özelliğine" erişerek bir yorumun sahibi olan yazıyı alabiliriz:
```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```
Yukarıdaki örnekte Eloquent, `id`'si `Comment` modelindeki `post_id` sütunuyla eşleşen bir `Post` modeli bulmaya çalışacaktır.

Eloquent, varsayılan yabancı anahtar adını, ilişki metodunun adını inceleyerek ve metod adına bir `_` ve ardından üst modelin birincil anahtar sütununun adını ekleyerek belirler. Yani, bu örnekte Eloquent, `comments` tablosundaki `Post` modeli yabancı anahtarının `post_id` olduğunu varsayacaktır.

Ancak, ilişkinizin yabancı anahtarı bu kurallara uymuyorsa, `belongsTo` metoduna ikinci argüman olarak özel bir yabancı anahtar adı iletebilirsiniz:
```php
/**
 * Yoruma sahip olan yazıyı al.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```
Üst modeliniz birincil anahtar olarak `id` kullanmıyorsa veya ilişkili modeli farklı bir sütun kullanarak bulmak istiyorsanız, üst tablonuzun özel anahtarını belirterek `belongsTo` metoduna üçüncü bir argüman iletebilirsiniz:
```php
/**
 * Yoruma sahip olan yazıyı al.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

#### Varsayılan Modeller (Default Models)
`belongsTo`, `hasOne`, `hasOneThrough` ve `morphOne` ilişkileri, belirtilen ilişki `null` ise döndürülecek bir varsayılan model tanımlamanıza olanak tanır. Bu desen genellikle Null Object deseni olarak adlandırılır ve kodunuzdaki koşullu kontrolleri kaldırmaya yardımcı olabilir. Aşağıdaki örnekte, `Post` modeline hiçbir kullanıcı eklenmemişse `user` ilişkisi boş bir `App\Models\User` modeli döndürecektir:
```php
/**
 * Yazının yazarını al.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```
Varsayılan modeli niteliklerle doldurmak için `withDefault` metoduna bir dizi veya closure iletebilirsiniz:
```php
/**
 * Yazının yazarını al.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Misafir Yazar',
    ]);
}

/**
 * Yazının yazarını al.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Misafir Yazar';
    });
}
```

#### Belongs To İlişkilerini Sorgulama (Querying Belongs To Relationships)
Bir "belongs to" ilişkisinin alt öğelerini sorgularken, ilgili Eloquent modellerini almak için `where` cümlesini manuel olarak oluşturabilirsiniz:
```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```
Ancak, verilen model için uygun ilişkiyi ve yabancı anahtarı otomatik olarak belirleyecek olan `whereBelongsTo` yöntemini kullanmanın daha uygun olduğunu görebilirsiniz:
```php
$posts = Post::whereBelongsTo($user)->get();
```
Ayrıca `whereBelongsTo` yöntemine bir koleksiyon örneği de sağlayabilirsiniz. Bunu yaptığınızda Laravel, koleksiyon içindeki üst modellerden herhangi birine ait olan modelleri alacaktır:
```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```
Varsayılan olarak Laravel, verilen modelle ilişkili ilişkiyi modelin sınıf adına göre belirleyecektir; ancak, `whereBelongsTo` yöntemine ikinci argüman olarak sağlayarak ilişki adını manuel olarak belirtebilirsiniz:
```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

### Birinin Çoğulu / Has One of Many (Has One of Many)
Bazen bir modelin birçok ilişkili modeli olabilir, ancak ilişkinin "en son" veya "en eski" ilişkili modelini kolayca almak isteyebilirsiniz. Örneğin, bir `User` modeli birçok `Order` modeliyle ilişkili olabilir, ancak kullanıcının verdiği en son siparişle etkileşime geçmenin uygun bir yolunu tanımlamak isteyebilirsiniz. Bunu, `hasOne` ilişki türünü `ofMany` metotlarıyla birleştirerek gerçekleştirebilirsiniz:
```php
/**
 * Kullanıcının en son siparişini al.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```
Benzer şekilde, bir ilişkinin "en eski" veya ilk ilişkili modelini almak için bir metot tanımlayabilirsiniz:
```php
/**
 * Kullanıcının en eski siparişini al.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```
Varsayılan olarak, `latestOfMany` ve `oldestOfMany` metotları, sıralanabilir olması gereken modelin birincil anahtarına göre en son veya en eski ilişkili modeli alacaktır. Ancak bazen, farklı bir sıralama ölçütü kullanarak daha büyük bir ilişkiden tek bir model almak isteyebilirsiniz.

Örneğin, `ofMany` yöntemini kullanarak kullanıcının en pahalı siparişini alabilirsiniz. `ofMany` metodu, ilk argüman olarak sıralanabilir sütunu ve ilişkili modeli sorgularken uygulanacak toplama işlevini (`min` veya `max`) kabul eder:
```php
/**
 * Kullanıcının en büyük siparişini al.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```
PostgreSQL, UUID sütunlarına karşı `MAX` işlevinin yürütülmesini desteklemediğinden, şu anda PostgreSQL UUID sütunlarıyla birlikte bire-çok ilişkileri (one-of-many) kullanmak mümkün değildir.

#### "Çok" İlişkilerini "Bire" İlişkilere Dönüştürme (Converting "Many" Relationships to Has One Relationships)
Çoğu zaman, `latestOfMany`, `oldestOfMany` veya `ofMany` metotlarını kullanarak tek bir model alırken, aynı model için zaten bir "has many" ilişkisi tanımlanmış olabilir. Kolaylık olması açısından Laravel, ilişki üzerinde `one` yöntemini çağırarak bu ilişkiyi kolayca bir "has one" ilişkisine dönüştürmenize izin verir:
```php
/**
 * Kullanıcının siparişlerini al.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Kullanıcının en büyük siparişini al.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```