# Laravel 12 Dokümantasyonu: Veritabanı: Sorgu Oluşturucu (Database: Query Builder)

## Giriş

Laravel'in veritabanı sorgu oluşturucusu (query builder), veritabanı sorguları oluşturmak ve çalıştırmak için kullanışlı, akıcı bir arayüz (fluent interface) sağlar. Uygulamanızdaki çoğu veritabanı işlemini gerçekleştirmek için kullanılabilir ve Laravel'in desteklenen tüm veritabanı sistemleriyle mükemmel şekilde çalışır.

Laravel sorgu oluşturucusu, uygulamanızı SQL enjeksiyon saldırılarına (SQL injection attacks) karşı korumak için PDO parametre bağlamayı (PDO parameter binding) kullanır. Sorgu bağlamaları (query bindings) olarak sorgu oluşturucuya iletilen dizeleri temizlemeye veya sterilize etmeye (sanitize) gerek yoktur.

PDO, sütun adlarının bağlanmasını desteklemez. Bu nedenle, kullanıcı girdisinin, sorgularınız tarafından başvurulan sütun adlarını belirlemesine asla izin vermemelisiniz. Buna "order by" sütunları da dahildir.

## Veritabanı Sorguları Çalıştırma (Running Database Queries)

#### Bir Tablodan Tüm Satırları Alma (Retrieving All Rows From a Table)
Bir sorgu başlatmak için `DB` facade'ı tarafından sağlanan `table` metodunu kullanabilirsiniz. `table` metodu, verilen tablo için akıcı bir sorgu oluşturucu örneği döndürür; bu, sorguya daha fazla kısıtlama eklemenize ve ardından `get` metodunu kullanarak sorgunun sonuçlarını almanıza olanak tanır:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Uygulamanın tüm kullanıcılarının bir listesini göster.
     */
    public function index(): View
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```
`get` metodu, her bir sonucun PHP `stdClass` nesnesinin bir örneği olduğu, sorgunun sonuçlarını içeren bir `Illuminate\Support\Collection` örneği döndürür. Her sütunun değerine, sütuna nesnenin bir özelliği olarak erişerek ulaşabilirsiniz:
```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```
Laravel koleksiyonları, verileri haritalamak (mapping) ve indirgemek (reducing) için son derece güçlü çeşitli metotlar sağlar. Laravel koleksiyonları hakkında daha fazla bilgi için koleksiyon dokümantasyonuna göz atın.

#### Bir Tablodan Tek Bir Satır / Sütun Alma (Retrieving a Single Row / Column From a Table)
Yalnızca bir veritabanı tablosundan tek bir satır almanız gerekiyorsa, `DB` facade'ının `first` metodunu kullanabilirsiniz. Bu metod, tek bir `stdClass` nesnesi döndürecektir:
```php
$user = DB::table('users')->where('name', 'John')->first();

return $user->email;
```
Bir veritabanı tablosundan tek bir satır almak, ancak eşleşen satır bulunamazsa bir `Illuminate\Database\RecordNotFoundException` fırlatmak isterseniz, `firstOrFail` metodunu kullanabilirsiniz. `RecordNotFoundException` yakalanmazsa, istemciye otomatik olarak bir 404 HTTP yanıtı gönderilir:
```php
$user = DB::table('users')->where('name', 'John')->firstOrFail();
```
Bir satırın tamamına ihtiyacınız yoksa, `value` metodunu kullanarak bir kayıttan tek bir değer çıkarabilirsiniz. Bu metod, sütunun değerini doğrudan döndürecektir:
```php
$email = DB::table('users')->where('name', 'John')->value('email');
```
Tek bir satırı `id` sütun değerine göre almak için `find` metodunu kullanın:
```php
$user = DB::table('users')->find(3);
```

#### Sütun Değerlerinin Bir Listesini Alma (Retrieving a List of Column Values)
Tek bir sütunun değerlerini içeren bir `Illuminate\Support\Collection` örneği almak isterseniz, `pluck` metodunu kullanabilirsiniz. Bu örnekte, kullanıcı başlıklarından oluşan bir koleksiyon alacağız:
```php
use Illuminate\Support\Facades\DB;

$titles = DB::table('users')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```
Ortaya çıkan koleksiyonun anahtarları (keys) olarak hangi sütunu kullanması gerektiğini, `pluck` metoduna ikinci bir argüman sağlayarak belirtebilirsiniz:
```php
$titles = DB::table('users')->pluck('title', 'name');

foreach ($titles as $name => $title) {
    echo $title;
}
```

### Sonuçları Parçalama (Chunking Results)
Binlerce veritabanı kaydıyla çalışmanız gerekiyorsa, `DB` facade'ı tarafından sağlanan `chunk` metodunu kullanmayı düşünün. Bu metod, bir seferde küçük bir sonuç parçası (chunk) alır ve her bir parçayı işleme için bir closure'a besler. Örneğin, `users` tablosunun tamamını 100 kayıtlık parçalar halinde alalım:
```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    foreach ($users as $user) {
        // ...
    }
});
```
Closure'dan `false` döndürerek daha fazla parçanın işlenmesini durdurabilirsiniz:
```php
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    // Kayıtları işle...

    return false;
});
```
Sonuçları parçalarken veritabanı kayıtlarını güncelliyorsanız, parça sonuçlarınız beklenmedik şekillerde değişebilir. Parçalama sırasında alınan kayıtları güncellemeyi planlıyorsanız, bunun yerine her zaman `chunkById` metodunu kullanmak en iyisidir. Bu metod, sonuçları kaydın birincil anahtarına (primary key) göre otomatik olarak sayfalandıracaktır:
```php
DB::table('users')->where('active', false)
    ->chunkById(100, function (Collection $users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```
`chunkById` ve `lazyById` metotları, yürütülen sorguya kendi "where" koşullarını eklediğinden, kendi koşullarınızı genellikle bir closure içinde mantıksal olarak gruplandırmalısınız:
```php
DB::table('users')->where(function ($query) {
    $query->where('credits', 1)->orWhere('credits', 2);
})->chunkById(100, function (Collection $users) {
    foreach ($users as $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['credits' => 3]);
    }
});
```
Parça geri çağrısı (chunk callback) içinde kayıtları güncellerken veya silerken, birincil anahtardaki veya yabancı anahtarlardaki (foreign keys) herhangi bir değişiklik parça sorgusunu etkileyebilir. Bu, potansiyel olarak kayıtların parçalanmış sonuçlara dahil edilmemesine neden olabilir.

### Sonuçları Tembelce (Lazily) Akışla Alma (Streaming Results Lazily)
`lazy` metodu, sorguyu parçalar halinde yürütmesi açısından `chunk` metoduna benzer şekilde çalışır. Ancak, her bir parçayı bir geri çağrıya iletmek yerine, `lazy()` metodu sonuçlarla tek bir akış olarak etkileşime girmenizi sağlayan bir `LazyCollection` döndürür:
```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // ...
});
```
Yine, yineleme sırasında alınan kayıtları güncellemeyi planlıyorsanız, bunun yerine `lazyById` veya `lazyByIdDesc` metotlarını kullanmak en iyisidir. Bu metotlar, sonuçları kaydın birincil anahtarına göre otomatik olarak sayfalandıracaktır:
```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```
Yineleme sırasında kayıtları güncellerken veya silerken, birincil anahtardaki veya yabancı anahtarlardaki herhangi bir değişiklik parça sorgusunu etkileyebilir. Bu, potansiyel olarak kayıtların sonuçlara dahil edilmemesine neden olabilir.

### Toplu (Aggregate) İşlemler
Sorgu oluşturucu ayrıca `count`, `max`, `min`, `avg` ve `sum` gibi toplu değerleri almak için çeşitli metotlar sağlar. Sorgunuzu oluşturduktan sonra bu metotlardan herhangi birini çağırabilirsiniz:
```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```
Elbette, bu metotları diğer cümlelerle (clauses) birleştirerek toplu değerinizin nasıl hesaplanacağını ince ayarlayabilirsiniz:
```php
$price = DB::table('orders')
    ->where('finalized', 1)
    ->avg('price');
```

#### Kayıtların Var Olup Olmadığını Belirleme (Determining if Records Exist)
Sorgunuzun kısıtlamalarıyla eşleşen herhangi bir kaydın var olup olmadığını belirlemek için `count` metodunu kullanmak yerine, `exists` ve `doesntExist` metotlarını kullanabilirsiniz:
```php
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

## Select İfadeleri (Select Statements)

#### Bir Select Cümlesi Belirtme (Specifying a Select Clause)
Bir veritabanı tablosundan her zaman tüm sütunları seçmek istemeyebilirsiniz. `select` metodunu kullanarak sorgu için özel bir "select" cümlesi belirleyebilirsiniz:
```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();
```
`distinct` metodu, sorgunun benzersiz (distinct) sonuçlar döndürmesini zorlamanıza olanak tanır:
```php
$users = DB::table('users')->distinct()->get();
```
Zaten bir sorgu oluşturucu örneğiniz varsa ve mevcut select cümlesine bir sütun eklemek isterseniz, `addSelect` metodunu kullanabilirsiniz:
```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

## Ham İfadeler (Raw Expressions)

Bazen bir sorguya rastgele bir dize eklemeniz gerekebilir. Ham bir dize ifadesi (raw string expression) oluşturmak için `DB` facade'ı tarafından sağlanan `raw` metodunu kullanabilirsiniz:
```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```
Ham ifadeler sorguya dizeler olarak enjekte edilecektir, bu nedenle SQL enjeksiyon güvenlik açıkları oluşturmaktan kaçınmak için son derece dikkatli olmalısınız.

### Ham Metotlar (Raw Methods)
`DB::raw` metodunu kullanmak yerine, sorgunuzun çeşitli bölümlerine ham bir ifade eklemek için aşağıdaki metotları da kullanabilirsiniz. Unutmayın, Laravel ham ifadeler kullanan herhangi bir sorgunun SQL enjeksiyonuna karşı korunduğunu garanti edemez.

#### `selectRaw`
`selectRaw` metodu, `addSelect(DB::raw(/* ... */))` yerine kullanılabilir. Bu metod, ikinci argüman olarak isteğe bağlı bir bağlamalar dizisi (array of bindings) kabul eder:
```php
$orders = DB::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

#### `whereRaw` / `orWhereRaw`
`whereRaw` ve `orWhereRaw` metotları, sorgunuza ham bir "where" cümlesi eklemek için kullanılabilir. Bu metotlar, ikinci argüman olarak isteğe bağlı bir bağlamalar dizisi kabul eder:
```php
$orders = DB::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

#### `havingRaw` / `orHavingRaw`
`havingRaw` ve `orHavingRaw` metotları, "having" cümlesinin değeri olarak ham bir dize sağlamak için kullanılabilir. Bu metotlar, ikinci argüman olarak isteğe bağlı bir bağlamalar dizisi kabul eder:
```php
$orders = DB::table('orders')
    ->select('department', DB::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

#### `orderByRaw`
`orderByRaw` metodu, "order by" cümlesinin değeri olarak ham bir dize sağlamak için kullanılabilir:
```php
$orders = DB::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

#### `groupByRaw`
`groupByRaw` metodu, group by cümlesinin değeri olarak ham bir dize sağlamak için kullanılabilir:
```php
$orders = DB::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

## Birleştirmeler (Joins)

#### İç Birleştirme (Inner Join) Cümlesi
Sorgu oluşturucu, sorgularınıza birleştirme cümleleri (join clauses) eklemek için de kullanılabilir. Temel bir "iç birleştirme" (inner join) gerçekleştirmek için bir sorgu oluşturucu örneğinde `join` metodunu kullanabilirsiniz. `join` metoduna iletilen ilk argüman, birleştirme yapmanız gereken tablonun adıdır, kalan argümanlar ise birleştirme için sütun kısıtlamalarını belirtir. Tek bir sorguda birden çok tabloyu bile birleştirebilirsiniz:
```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

#### Sol Birleştirme (Left Join) / Sağ Birleştirme (Right Join) Cümlesi
Bir "iç birleştirme" yerine "sol birleştirme" (left join) veya "sağ birleştirme" (right join) yapmak isterseniz, `leftJoin` veya `rightJoin` metotlarını kullanın. Bu metotlar `join` metoduyla aynı imzaya (signature) sahiptir:
```php
$users = DB::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = DB::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

#### Çapraz Birleştirme (Cross Join) Cümlesi
Bir "çapraz birleştirme" (cross join) gerçekleştirmek için `crossJoin` metodunu kullanabilirsiniz. Çapraz birleştirmeler, ilk tablo ile birleştirilen tablo arasında bir kartezyen çarpım (cartesian product) oluşturur:
```php
$sizes = DB::table('sizes')
    ->crossJoin('colors')
    ->get();
```

#### Gelişmiş Birleştirme Cümleleri (Advanced Join Clauses)
Daha gelişmiş birleştirme cümleleri de belirtebilirsiniz. Başlamak için, `join` metoduna ikinci argüman olarak bir closure iletin. Closure, "join" cümlesi üzerinde kısıtlamalar belirlemenize izin veren bir `Illuminate\Database\Query\JoinClause` örneği alacaktır:
```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
    })
    ->get();
```
Birleştirmelerinizde bir "where" cümlesi kullanmak isterseniz, `JoinClause` örneği tarafından sağlanan `where` ve `orWhere` metotlarını kullanabilirsiniz. Bu metotlar, iki sütunu karşılaştırmak yerine sütunu bir değerle karşılaştıracaktır:
```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

#### Alt Sorgu (Subquery) Birleştirmeleri
Bir sorguyu bir alt sorguyla (subquery) birleştirmek için `joinSub`, `leftJoinSub` ve `rightJoinSub` metotlarını kullanabilirsiniz. Bu metotların her biri üç argüman alır: alt sorgu, tablo takma adı (alias) ve ilgili sütunları tanımlayan bir closure. Bu örnekte, her bir kullanıcı kaydının aynı zamanda kullanıcının en son yayınladığı blog gönderisinin `created_at` zaman damgasını da içerdiği bir kullanıcı koleksiyonu alacağız:
```php
$latestPosts = DB::table('posts')
    ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
    ->where('is_published', true)
    ->groupBy('user_id');

$users = DB::table('users')
    ->joinSub($latestPosts, 'latest_posts', function (JoinClause $join) {
        $join->on('users.id', '=', 'latest_posts.user_id');
    })->get();
```

#### Yanal Birleştirmeler (Lateral Joins)
Yanal birleştirmeler şu anda PostgreSQL, MySQL >= 8.0.14 ve SQL Server tarafından desteklenmektedir.

Bir alt sorguyla "yanal birleştirme" (lateral join) gerçekleştirmek için `joinLateral` ve `leftJoinLateral` metotlarını kullanabilirsiniz. Bu metotların her biri iki argüman alır: alt sorgu ve tablo takma adı. Birleştirme koşulu(ları), verilen alt sorgunun `where` cümlesi içinde belirtilmelidir. Yanal birleştirmeler her satır için değerlendirilir ve alt sorgu dışındaki sütunlara başvurabilir.

Bu örnekte, bir kullanıcı koleksiyonunun yanı sıra kullanıcının en son üç blog gönderisini de alacağız. Her kullanıcı, sonuç kümesinde en fazla üç satır üretebilir: her gönderi için bir tane.