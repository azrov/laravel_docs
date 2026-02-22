# Laravel 12 Dokümantasyonu: Eloquent: Başlarken (Eloquent: Getting Started)

## Giriş

Laravel, veritabanınızla etkileşimi keyifli hale getiren bir nesne-ilişkisel haritalayıcı (object-relational mapper - ORM) olan Eloquent'i içerir. Eloquent'i kullanırken, her veritabanı tablosunun, o tabloyla etkileşim kurmak için kullanılan karşılık gelen bir "Model"i vardır. Eloquent modelleri, veritabanı tablosundan kayıtları almanın yanı sıra, tabloya kayıt eklemenize, güncellemenize ve silmenize de olanak tanır.

Başlamadan önce, uygulamanızın `config/database.php` yapılandırma dosyasında bir veritabanı bağlantısı yapılandırdığınızdan emin olun. Veritabanınızı yapılandırma hakkında daha fazla bilgi için veritabanı yapılandırma dokümantasyonuna göz atın.

## Model Sınıfları Oluşturma (Generating Model Classes)

Başlamak için bir Eloquent modeli oluşturalım. Modeller tipik olarak `app\Models` dizininde bulunur ve `Illuminate\Database\Eloquent\Model` sınıfını genişletir (extend). Yeni bir model oluşturmak için `make:model` Artisan komutunu kullanabilirsiniz:
```bash
php artisan make:model Flight
```
Modeli oluştururken bir veritabanı migration'ı oluşturmak isterseniz, `--migration` veya `-m` seçeneğini kullanabilirsiniz:
```bash
php artisan make:model Flight --migration
```
Bir model oluştururken fabrikalar (factories), tohumlayıcılar (seeders), politikalar (policies), controller'lar ve form istekleri (form requests) gibi çeşitli diğer sınıf türlerini de oluşturabilirsiniz. Ayrıca, bu seçenekler birleştirilerek aynı anda birden çok sınıf oluşturulabilir:
```bash
# Bir model ve bir FlightFactory sınıfı oluştur...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# Bir model ve bir FlightSeeder sınıfı oluştur...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# Bir model ve bir FlightController sınıfı oluştur...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# Bir model, FlightController kaynak sınıfı ve form isteği sınıfları oluştur...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# Bir model ve bir FlightPolicy sınıfı oluştur...
php artisan make:model Flight --policy

# Bir model ve bir migration, fabrika, tohumlayıcı ve controller oluştur...
php artisan make:model Flight -mfsc

# Bir model, migration, fabrika, tohumlayıcı, politika, controller ve form istekleri oluşturma kısayolu...
php artisan make:model Flight --all
php artisan make:model Flight -a

# Bir pivot model oluştur...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

#### Modelleri İnceleme (Inspecting Models)
Bazen bir modelin mevcut tüm niteliklerini ve ilişkilerini yalnızca koduna bakarak belirlemek zor olabilir. Bunun yerine, modelin tüm niteliklerinin ve ilişkilerinin kullanışlı bir özetini sağlayan `model:show` Artisan komutunu deneyin:
```bash
php artisan model:show Flight
```

## Eloquent Model Kuralları (Eloquent Model Conventions)

`make:model` komutuyla oluşturulan modeller `app/Models` dizinine yerleştirilecektir. Temel bir model sınıfını inceleyelim ve Eloquent'in bazı önemli kurallarını (conventions) tartışalım:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // ...
}
```

### Tablo Adları (Table Names)
Yukarıdaki örneğe baktıktan sonra, Eloquent'e `Flight` modelimizin hangi veritabanı tablosuna karşılık geldiğini söylemediğimizi fark etmiş olabilirsiniz. Kural gereği (by convention), sınıfın "yılan_case" (snake case), çoğul adı, başka bir ad açıkça belirtilmedikçe tablo adı olarak kullanılacaktır. Yani, bu durumda Eloquent, `Flight` modelinin kayıtları `flights` tablosunda sakladığını varsayacakken, bir `AirTrafficController` modeli kayıtları `air_traffic_controllers` tablosunda saklayacaktır.

Modelinizin karşılık gelen veritabanı tablosu bu kurala uymuyorsa, model üzerinde bir `table` özelliği tanımlayarak modelin tablo adını manuel olarak belirtebilirsiniz:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Modelle ilişkili tablo.
     *
     * @var string
     */
    protected $table = 'my_flights';
}
```

### Birincil Anahtarlar (Primary Keys)
Eloquent ayrıca her modelin karşılık gelen veritabanı tablosunun `id` adında bir birincil anahtar (primary key) sütununa sahip olduğunu varsayacaktır. Gerekirse, modelinizde birincil anahtar olarak hizmet edecek farklı bir sütunu belirtmek için modelinizde `protected $primaryKey` özelliği tanımlayabilirsiniz:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Tabloyla ilişkili birincil anahtar.
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
```
Ek olarak Eloquent, birincil anahtarın artan (incrementing) bir tamsayı değeri olduğunu varsayar; bu, Eloquent'in birincil anahtarı otomatik olarak bir tamsayıya dönüştüreceği (cast) anlamına gelir. Artan olmayan veya sayısal olmayan bir birincil anahtar kullanmak isterseniz, modelinizde `false` olarak ayarlanmış bir `public $incrementing` özelliği tanımlamalısınız:
```php
<?php

class Flight extends Model
{
    /**
     * Modelin ID'sinin otomatik olarak artıp artmadığını belirtir.
     *
     * @var bool
     */
    public $incrementing = false;
}
```
Modelinizin birincil anahtarı bir tamsayı değilse, modelinizde `protected $keyType` özelliği tanımlamalısınız. Bu özellik `string` değerine sahip olmalıdır:
```php
<?php

class Flight extends Model
{
    /**
     * Birincil anahtar ID'nin veri türü.
     *
     * @var string
     */
    protected $keyType = 'string';
}
```

#### "Bileşik" Birincil Anahtarlar ("Composite" Primary Keys)
Eloquent, her modelin birincil anahtarı olarak hizmet edebilecek en az bir benzersiz tanımlayıcı "ID"ye sahip olmasını gerektirir. "Bileşik" (composite) birincil anahtarlar Eloquent modelleri tarafından desteklenmez. Ancak, tablonun benzersiz tanımlayıcı birincil anahtarına ek olarak veritabanı tablolarınıza ek çok sütunlu, benzersiz indeksler eklemekte özgürsünüz.

### UUID ve ULID Anahtarları
Eloquent modelinizin birincil anahtarları olarak otomatik artan tamsayılar kullanmak yerine, bunun yerine UUID'leri kullanmayı seçebilirsiniz. UUID'ler, 36 karakter uzunluğunda, evrensel olarak benzersiz alfanümerik tanımlayıcılardır.

Bir modelin otomatik artan bir tamsayı anahtarı yerine bir UUID anahtarı kullanmasını istiyorsanız, modelde `Illuminate\Database\Eloquent\Concerns\HasUuids` özelliğini (trait) kullanabilirsiniz. Elbette, modelin UUID eşdeğeri bir birincil anahtar sütununa sahip olduğundan emin olmalısınız:
```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Europe']);

$article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"
```
Varsayılan olarak, `HasUuids` özelliği modelleriniz için "sıralı" (ordered) UUID'ler oluşturacaktır. Bu UUID'ler, sözlükbilimsel olarak (lexicographically) sıralanabildikleri için indekslenmiş veritabanı depolaması için daha verimlidir.

Belirli bir model için UUID oluşturma sürecini, model üzerinde bir `newUniqueId` metodu tanımlayarak geçersiz kılabilirsiniz (override). Ayrıca, model üzerinde bir `uniqueIds` metodu tanımlayarak hangi sütunların UUID alması gerektiğini belirtebilirsiniz:
```php
use Ramsey\Uuid\Uuid;

/**
 * Model için yeni bir UUID oluştur.
 */
public function newUniqueId(): string
{
    return (string) Uuid::uuid4();
}

/**
 * Benzersiz bir tanımlayıcı alması gereken sütunları al.
 *
 * @return array<int, string>
 */
public function uniqueIds(): array
{
    return ['id', 'discount_code'];
}
```
İsterseniz, UUID'ler yerine "ULID"leri kullanmayı seçebilirsiniz. ULID'ler UUID'lere benzer; ancak, yalnızca 26 karakter uzunluğundadırlar. Sıralı UUID'ler gibi, ULID'ler de verimli veritabanı indekslemesi için sözlükbilimsel olarak sıralanabilir. ULID'leri kullanmak için modelinizde `Illuminate\Database\Eloquent\Concerns\HasUlids` özelliğini kullanmalısınız. Ayrıca modelin ULID eşdeğeri bir birincil anahtar sütununa sahip olduğundan emin olmalısınız:
```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Asia']);

$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

### Zaman Damgaları (Timestamps)
Varsayılan olarak Eloquent, modelinizin karşılık gelen veritabanı tablosunda `created_at` ve `updated_at` sütunlarının bulunmasını bekler. Eloquent, modeller oluşturulduğunda veya güncellendiğinde bu sütunların değerlerini otomatik olarak ayarlayacaktır. Bu sütunların Eloquent tarafından otomatik olarak yönetilmesini istemiyorsanız, modelinizde `false` değerine sahip bir `$timestamps` özelliği tanımlamalısınız:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Modelin zaman damgalı olup olmadığını belirtir.
     *
     * @var bool
     */
    public $timestamps = false;
}
```
Modelinizin zaman damgalarının biçimini özelleştirmeniz gerekirse, modelinizde `$dateFormat` özelliğini ayarlayın. Bu özellik, tarih niteliklerinin veritabanında nasıl saklanacağını ve model bir diziye veya JSON'a serileştirildiğinde (serialized) biçimlerini belirler:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Modelin tarih sütunlarının depolama biçimi.
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```
Zaman damgalarını saklamak için kullanılan sütunların adlarını özelleştirmeniz gerekirse, modelinizde `CREATED_AT` ve `UPDATED_AT` sabitlerini (constants) tanımlayabilirsiniz:
```php
<?php

class Flight extends Model
{
    /**
     * "created_at" sütununun adı.
     *
     * @var string|null
     */
    public const CREATED_AT = 'creation_date';

    /**
     * "updated_at" sütununun adı.
     *
     * @var string|null
     */
    public const UPDATED_AT = 'updated_date';
}
```
Model işlemlerini, modelin `updated_at` zaman damgası değiştirilmeden gerçekleştirmek isterseniz, `withoutTimestamps` metoduna verilen bir closure içinde model üzerinde işlem yapabilirsiniz:
```php
Model::withoutTimestamps(fn () => $post->increment('reads'));
```

### Veritabanı Bağlantıları (Database Connections)
Varsayılan olarak, tüm Eloquent modelleri uygulamanız için yapılandırılmış varsayılan veritabanı bağlantısını kullanacaktır. Belirli bir modelle etkileşim kurarken kullanılması gereken farklı bir bağlantı belirtmek isterseniz, model üzerinde bir `$connection` özelliği tanımlamalısınız:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Model tarafından kullanılması gereken veritabanı bağlantısı.
     *
     * @var string
     */
    protected $connection = 'mysql';
}
```

### Varsayılan Nitelik Değerleri (Default Attribute Values)
Varsayılan olarak, yeni oluşturulmuş bir model örneği herhangi bir nitelik değeri içermeyecektir. Modelinizin bazı nitelikleri için varsayılan değerler tanımlamak isterseniz, modelinizde bir `$attributes` özelliği tanımlayabilirsiniz. `$attributes` dizisine yerleştirilen nitelik değerleri, tıpkı veritabanından yeni okunmuş gibi, ham, "depolanabilir" (storable) formatlarında olmalıdır:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Modelin nitelikler için varsayılan değerleri.
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}
```

### Eloquent Sıkılığını (Strictness) Yapılandırma (Configuring Eloquent Strictness)
Laravel, çeşitli durumlarda Eloquent'in davranışını ve "sıkılığını" (strictness) yapılandırmanıza izin veren birkaç yöntem sunar.

İlk olarak, `preventLazyLoading` metodu, tembel yüklemenin (lazy loading) önlenip önlenmeyeceğini belirten isteğe bağlı bir boolean argümanı kabul eder. Örneğin, üretim ortamında tembel yüklemeyi devre dışı bırakmak isteyebilirsiniz, böylece üretim kodunda yanlışlıkla tembel yüklenmiş bir ilişki olsa bile üretim ortamınız normal şekilde çalışmaya devam edecektir. Tipik olarak bu metot, uygulamanızın `AppServiceProvider` sınıfının `boot` metodunda çağrılmalıdır:
```php
use Illuminate\Database\Eloquent\Model;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```
Ayrıca, `preventSilentlyDiscardingAttributes` metodunu çağırarak, doldurulamaz (unfillable) bir niteliği doldurmaya (fill) çalışırken Laravel'e bir istisna (exception) fırlatması talimatını verebilirsiniz. Bu, yerel geliştirme sırasında, modelin `fillable` dizisine eklenmemiş bir niteliği ayarlamaya çalışırken beklenmeyen hataları önlemeye yardımcı olabilir:
```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

## Modelleri Alma (Retrieving Models)

Bir model ve onunla ilişkili veritabanı tablosunu oluşturduktan sonra, veritabanınızdan veri almaya başlamaya hazırsınız. Her Eloquent modelini, modelle ilişkili veritabanı tablosunu akıcı bir şekilde sorgulamanıza izin veren güçlü bir sorgu oluşturucu (query builder) olarak düşünebilirsiniz. Modelin `all` metodu, modelin ilişkili veritabanı tablosundaki tüm kayıtları alacaktır:
```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

#### Sorgu Oluşturma (Building Queries)
Eloquent `all` metodu, modelin tablosundaki tüm sonuçları döndürecektir. Ancak, her Eloquent modeli bir sorgu oluşturucu görevi gördüğünden, sorgulara ek kısıtlamalar ekleyebilir ve ardından sonuçları almak için `get` metodunu çağırabilirsiniz:
```php
$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->limit(10)
    ->get();
```
Eloquent modelleri sorgu oluşturucular olduğundan, Laravel'in sorgu oluşturucusu tarafından sağlanan tüm metotları incelemelisiniz. Eloquent sorgularınızı yazarken bu metotlardan herhangi birini kullanabilirsiniz.

#### Modelleri Yenileme (Refreshing Models)
Veritabanından alınmış bir Eloquent model örneğiniz zaten varsa, `fresh` ve `refresh` metotlarını kullanarak modeli "yenileyebilirsiniz" (refresh). `fresh` metodu, modeli veritabanından yeniden alacaktır. Mevcut model örneği etkilenmeyecektir:
```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```
`refresh` metodu, veritabanından gelen yeni verileri kullanarak mevcut modeli yeniden dolduracaktır (re-hydrate). Ayrıca, yüklenmiş tüm ilişkileri de yenilenecektir:
```php
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

### Koleksiyonlar (Collections)
Gördüğümüz gibi, `all` ve `get` gibi Eloquent metotları veritabanından birden çok kayıt alır. Ancak, bu metotlar bir `Illuminate\Database\Eloquent\Collection` örneği döndürür. `Collection` sınıfı, Eloquent sonuçlarınızla etkileşim kurmak için çeşitli yararlı metotlar sağlar. Örneğin, bir koleksiyondaki her bir modeli yinelemek için `each` metodunu kullanabilirsiniz:
```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

Koleksiyonların normal diziler gibi yinelenebilir olduğunu unutmayın, bu nedenle bunları döngü içinde kullanabilirsiniz. Ancak, koleksiyonlar aynı zamanda çok daha güçlüdür. Örneğin, koleksiyonları filtrelemek için `filter` metodunu kullanabiliriz:
```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->filter(function ($flight) {
    return $flight->cancelled === false;
});
```
Koleksiyonlar hakkında daha fazla bilgi için koleksiyon dokümantasyonuna bakın.

## Sonuçları Parçalama (Chunking Results)
Eğer `all` veya `get` yöntemlerini kullanarak on binlerce Eloquent kaydı yüklüyorsanız, uygulamanızın belleği tükenebilir. Bu kayıtları işlemek için daha az bellek kullanan bir alternatif olarak `chunk` yöntemini kullanabilirsiniz. `chunk` yöntemi, bir Eloquent modeli koleksiyonunu alır ve her bir parçayı işlenmek üzere bir closure'a besler. Belirli bir kümeyi güncellerken olduğu gibi, binlerce kaydı işlerken bu tercih edilen yöntemdir:
```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```
`chunk` yönteminin ilk argümanı, her bir "parçada" (chunk) almak istediğiniz kayıt sayısıdır. İkinci argüman olarak iletilen closure, veritabanından alınan her bir parçayı alacaktır. Sorgu, her bir parçayı almak için yürütülecektir.

Eğer sonuçları parçalarken veritabanı kayıtlarını güncelliyorsanız, parça sonuçlarınız beklenmedik şekillerde değişebilir. Bu nedenle, parçalama yaparken kayıtları güncellerken her zaman `chunkById` yöntemini kullanmak en iyisidir. Bu yöntem, kayıtları birincil anahtara göre otomatik olarak parçalayacaktır:
```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    });
```

## İlk / Tekil Sonuçları Alma (Retrieving Single Models / Aggregates)
Belirli bir sorguyla eşleşen tek bir kayıt almak için `first` yöntemini kullanabilirsiniz. Bu yöntem, tek bir model örneği döndürür:
```php
use App\Models\Flight;

$flight = Flight::where('number', 'FR 900')->first();
```

Eğer eşleşen hiçbir kayıt bulamazsanız, `first` yöntemi `null` döndürecektir. Sorgunuzun eşleşen bir kayıt bulması gerektiğini düşünüyorsanız, eşleşen bir kayıt bulunamazsa bir istisna fırlatacak olan `firstOrFail` yöntemini kullanabilirsiniz:
```php
$model = Flight::findOrFail(1);
$model = Flight::where('legs', '>', 100)->firstOrFail();
```

#### 'find' Yöntemi
Ayrıca, birincil anahtarına göre bir kayıt almak için `find` yöntemini kullanabilirsiniz:
```php
$flight = Flight::find(1);
```

#### 'firstWhere' Yöntemi
`firstWhere` yöntemi, `where` cümlesi için daha kısa bir sözdizimi sağlar:
```php
$flight = Flight::firstWhere('number', 'FR 900');
```

#### Toplu Değerleri Alma (Retrieving Aggregates)
Ayrıca, belirli bir sorguyla eşleşen kayıtların sayısını almak için `count`, `sum`, `max` ve diğer toplu (aggregate) yöntemleri de kullanabilirsiniz. Bu yöntemler, uygun skaler değeri bir tamsayı olarak döndürür:
```php
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

## Modelleri Ekleme, Güncelleme ve Silme (Inserting, Updating, and Deleting Models)

### Ekleme (Inserting)
Yeni bir kayıt eklemek için yeni bir model örneği oluşturun, nitelikleri ayarlayın ve ardından `save` yöntemini çağırın:
```php
<?php

namespace App\Http\Controllers;

use App\Models\Flight;
use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class FlightController extends Controller
{
    /**
     * Store a new flight in the database.
     */
    public function store(Request $request): RedirectResponse
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();

        return redirect('/flights');
    }
}
```
Bu örnekte, HTTP isteğinden gelen `name` alanını `App\Models\Flight` model örneğinin `name` niteliğine atadık. `save` yöntemini çağırdığımızda, veritabanına bir kayıt eklenecektir. `created_at` ve `updated_at` zaman damgaları `save` yöntemi çağrıldığında otomatik olarak ayarlanacaktır, manuel olarak ayarlanmalarına gerek yoktur.

#### Alternatif Olarak 'create' Yöntemi
Alternatif olarak, modelin niteliklerini tek bir PHP dizisi olarak ileten `create` yöntemini kullanarak yeni bir model ekleyebilirsiniz. `create` yöntemi, yeni bir model örneği döndürür:
```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```
Ancak, `create` yöntemini kullanmadan önce, modelinizde bir `fillable` veya `guarded` özelliği tanımlamış olmanız gerekir. Çünkü tüm Eloquent modelleri varsayılan olarak toplu atamaya (mass assignment) karşı korumalıdır. Toplu atama hakkında daha fazla bilgi edinmek için toplu atama dokümantasyonuna bakın.

### Güncelleme (Updating)
`save` yöntemi, veritabanında zaten bulunan bir modeli güncellemek için de kullanılabilir. Bir modeli güncellemek için, onu almalı, güncellemek istediğiniz tüm nitelikleri ayarlamalı ve ardından `save` yöntemini çağırmalısınız. Yine, `updated_at` zaman damgası otomatik olarak güncellenecektir, manuel olarak ayarlanmasına gerek yoktur:
```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```

#### Toplu Güncellemeler (Mass Updates)
Güncellemeler, belirli bir sorguyla eşleşen tüm modellere de uygulanabilir. Bu örnekte, `active` olan ve `destination` değeri `San Diego` olan tüm uçuşlar ertelenmiş olarak işaretlenecektir:
```php
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```

### Toplu Atama (Mass Assignment)
`create` yöntemini kullanarak yeni bir model oluşturmak için modelinizin bir `fillable` veya `guarded` özelliği tanımlamış olması gerekir. Çünkü tüm Eloquent modelleri varsayılan olarak toplu atamaya karşı korumalıdır.

Bir toplu atama güvenlik açığı, bir kullanıcı beklenmedik bir HTTP parametresi gönderdiğinde ve bu parametre, veritabanında değiştirmeyi beklemediğiniz bir sütunu değiştirdiğinde ortaya çıkar. Örneğin, kötü niyetli bir kullanıcı, bir HTTP isteği aracılığıyla bir `is_admin` parametresi gönderebilir ve bu parametre daha sonra modelinizin `create` yöntemine iletilerek kullanıcının kendisini bir yöneticiye yükseltmesine olanak tanır.

Bu nedenle, başlangıç olarak, hangi model niteliklerinin toplu olarak atanabileceğini belirtmelisiniz. Bunu model üzerinde `$fillable` özelliğini kullanarak yapabilirsiniz. Örneğin, `Flight` modelimizin `name` niteliğinin toplu atama yoluyla atanabilmesine izin verelim:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```

Hangi niteliklerin toplu olarak atanabileceğini belirledikten sonra, yeni bir kayıt eklemek için `create` yöntemini kullanabilirsiniz. `create` yöntemi, yeni bir model örneği döndürür:
```php
$flight = Flight::create(['name' => 'London to Paris']);
```

Halihazırda bir model örneğiniz varsa, nitelikleri atamak ve modeli kaydetmek için `fill` yöntemini kullanabilirsiniz:
```php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

#### 'fillable' vs 'guarded'
Eğer `$fillable` bir "izin listesi" (allow list) olarak hizmet ediyorsa, `$guarded` ise bir "engelleme listesi" (block list) olarak hizmet eder:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The attributes that are not mass assignable.
     *
     * @var array
     */
    protected $guarded = ['price'];
}
```

`$guarded` özelliği, toplu olarak atanamayacak nitelikleri içerir. 'price' dışındaki tüm nitelikler toplu olarak atanabilir. Eğer tüm niteliklerin toplu olarak atanabilmesine izin vermek isterseniz, `$guarded` özelliğini boş bir dizi olarak tanımlayabilirsiniz:
```php
/**
 * The attributes that are not mass assignable.
 *
 * @var array
 */
protected $guarded = [];
```

### Diğer Oluşturma Yöntemleri (Other Creation Methods)

#### 'firstOrCreate' / 'firstOrNew'
Toplu atama kullanarak model oluşturmanın yanı sıra, `firstOrCreate` ve `firstOrNew` yöntemlerini de kullanabilirsiniz. `firstOrCreate` yöntemi, verilen anahtar/değer çiftlerini kullanarak bir veritabanı kaydını bulmaya çalışacaktır. Eğer model veritabanında bulunamazsa, ilk dizi argümanındaki nitelikler ve isteğe bağlı ikinci dizi argümanındaki nitelikler birleştirilerek bir kayıt eklenir.

`firstOrNew` yöntemi de `firstOrCreate` gibi, verilen niteliklerle eşleşen bir kaydı bulmaya çalışacaktır. Ancak, eğer bir model bulunamazsa, `firstOrNew` yöntemi yeni bir model örneği döndürecektir. `firstOrNew` tarafından döndürülen modelin henüz veritabanına kaydedilmediğini unutmayın. Kaydetmek için manuel olarak `save` yöntemini çağırmanız gerekecektir:
```php
use App\Models\Flight;

// Adı 'London to Paris' olan bir uçuşu al veya varsa oluştur...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// Adı 'London to Paris' olan bir uçuşu al veya belirtilen niteliklerle oluştur...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Adı 'London to Paris' olan bir uçuşu al veya yeni bir uçuş örneği oluştur...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);

// Adı 'London to Paris' olan bir uçuşu al veya belirtilen niteliklerle yeni bir uçuş örneği oluştur...
$flight = Flight::firstOrNew(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

#### 'updateOrCreate' Yöntemi
Ayrıca, mevcut bir modeli güncellemek veya eğer eşleşen bir model yoksa yeni bir model oluşturmak istediğiniz durumlar da olabilir. Laravel, bunu tek bir adımda yapmak için `updateOrCreate` yöntemini sağlar. `firstOrCreate` gibi, `updateOrCreate` yöntemi de modeli kalıcı hale getirir (persists), bu nedenle `save`'i çağırmanıza gerek yoktur:
```php
$flight = Flight::updateOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'price' => 99]
);
```

Eğer birden çok kaydı güncellemek veya oluşturmak istiyorsanız, bir dizi veri iletebilirsiniz. Bu durumda, sağlanan her bir dizi öğesi için ayrı bir `updateOrCreate` çağrısı yapılacaktır:
```php
$flights = Flight::updateOrCreate(
    [
        ['name' => 'London to Paris'],
        ['name' => 'Frankfurt to Berlin'],
    ],
    [
        ['delayed' => 1, 'price' => 99],
        ['delayed' => 0, 'price' => 129],
    ]
);
```

### Silme (Deleting)
Bir modeli silmek için model örneğinde `delete` yöntemini çağırabilirsiniz:
```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

#### Mevcut Bir Anahtarla Silme (Deleting By Key)
Ancak, model örneğini almadan bir kaydı silmek isterseniz, `destroy` yöntemini kullanabilirsiniz. `destroy` yöntemi, birincil anahtarı argüman olarak kabul eder ve modeli almadan siler. Tek bir birincil anahtarı, birden çok birincil anahtarı bir dizi olarak veya birden çok birincil anahtarı birden çok argüman olarak iletebilirsiniz:
```php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);
```

`destroy` yöntemi, her bir modeli tek tek yükler ve `delete` yöntemini çağırır, böylece her bir model için uygun olaylar (events) gönderilir.

#### Sorgularla Silme (Deleting Using Queries)
Elbette, bir sorgu üzerinde `delete` yöntemini çağırarak bir grup modeli silebilirsiniz. Bu örnekte, `active` olmayan tüm uçuşları sileceğiz. Ancak, toplu silme işlemleri, silinen modeller için herhangi bir model olayı göndermeyecektir:
```php
$deleted = Flight::where('active', 0)->delete();
```

### Yazılı Silme (Soft Deleting)
Gerçekten veritabanınızdan kayıtları kaldırmak yerine, bir modeli "yazılı olarak silebilirsiniz" (soft delete). Yazılı olarak silinen modeller aslında veritabanından kaldırılmaz; bunun yerine, modelin silinip silinmediğini belirten bir `deleted_at` niteliği ayarlanır. Bir model yazılı olarak silinmişse, normal sorgu sonuçlarına dahil edilmeyecektir.

Yazılı silme işlemini etkinleştirmek için, modele `Illuminate\Database\Eloquent\SoftDeletes` özelliğini (trait) ekleyin ve modelin `dates` dizisine `deleted_at` sütununu ekleyin:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

Ayrıca, `deleted_at` sütununu veritabanı tablonuza eklemeniz gerekecektir. Laravel'in şema oluşturucusu (schema builder) bu sütunu oluşturmak için bir yardımcı metot içerir:
```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});
```

Artık, modelde `delete` yöntemini çağırdığınızda, `deleted_at` sütunu geçerli tarih ve saate ayarlanacaktır. Yazılı olarak silinen modelleri içeren sorgular çalıştırırken, yazılı olarak silinen modeller otomatik olarak tüm sorgu sonuçlarından hariç tutulacaktır.

Belirli bir sorguya yazılı olarak silinen modelleri dahil etmek için `withTrashed` yöntemini kullanabilirsiniz:
```php
$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

Yalnızca yazılı olarak silinen modelleri almak için `onlyTrashed` yöntemini kullanabilirsiniz:
```php
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

Yazılı olarak silinmiş bir modeli kalıcı olarak silmek (force delete) için `forceDelete` yöntemini kullanabilirsiniz:
```php
$flight->forceDelete();
```

## Sorgu Kapsamları (Query Scopes)

### Global Kapsamlar (Global Scopes)
Global kapsamlar, belirli bir modelin tüm sorgularına kısıtlamalar eklemenizi sağlar. Laravel'in kendi yazılı silme (soft delete) özelliği, veritabanından "silinmemiş" modelleri almak için global bir kapsam kullanır. Kendi global kapsamlarınızı yazmak, belirli bir model için tüm sorgulara belirli kısıtlamaların uygulandığından emin olmanın kullanışlı bir yolunu sağlayabilir.

Global kapsamlar yazmakla ilgili daha fazla bilgi için Eloquent dokümantasyonunun ilgili bölümüne bakın.

### Yerel Kapsamlar (Local Scopes)
Yerel kapsamlar, uygulamanızda kolayca yeniden kullanılabilecek ortak sorgu kısıtlamaları tanımlamanıza olanak tanır. Örneğin, sık sık "popüler" olarak kabul edilen tüm kullanıcıları almanız gerekebilir. Bunun için bir kapsam tanımlamak için, genellikle bir Eloquent model yönteminin ön ekine `scope` ekleyebilirsiniz.

Kapsamlar her zaman sorgu oluşturucu örneğini döndürmelidir:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     */
    public function scopePopular($query): void
    {
        $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     */
    public function scopeActive($query): void
    {
        $query->where('active', 1);
    }
}
```

Kapsam tanımlandıktan sonra, modeli sorgularken kapsam yöntemini çağırabilirsiniz. Ancak, yöntemi çağırırken `scope` ön ekini dahil etmemelisiniz. Ayrıca, çeşitli kapsam çağrılarını zincirleyebilirsiniz:
```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

Dinamik kapsamlar, parametre kabul eden kapsamlardır. Parametreleri eklemek için, bunları kapsam yönteminize ekleyin:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     */
    public function scopeOfType($query, string $type): void
    {
        $query->where('type', $type);
    }
}
```

Artık kapsam çağrılırken parametreleri iletebilirsiniz:
```php
$users = User::ofType('admin')->get();
```

## Karşılaştırma Modelleri (Comparing Models)
Bazen iki modelin "aynı" olup olmadığını belirlemeniz gerekebilir. `is` yöntemi, iki modelin aynı birincil anahtara, tabloya ve veritabanı bağlantısına sahip olup olmadığını doğrulamak için kullanılabilir:
```php
if ($post->is($anotherPost)) {
    // ...
}
```

`isNot` yöntemi, `is` yönteminin tersidir:
```php
if ($post->isNot($anotherPost)) {
    // ...
}
```

## İlişkili Modeller (Associated Models)
Eğer modeliniz diğer modellerle ilişkilendirilmişse, bu ilişkileri kullanarak modelleri alabilir, ekleyebilir, güncelleyebilir ve silebilirsiniz. İlişkiler hakkında daha fazla bilgi için Eloquent ilişkileri dokümantasyonuna bakın.