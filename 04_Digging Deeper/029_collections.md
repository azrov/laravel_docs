# Laravel 12 Dokümantasyonu: Koleksiyonlar (Collections)

## Giriş

`Illuminate\Support\Collection` sınıfı, dizi verileriyle (arrays of data) çalışmak için akıcı (fluent), kullanışlı bir sarmalayıcı (wrapper) sağlar. Örneğin, aşağıdaki koda bakın. Diziden yeni bir koleksiyon örneği oluşturmak için `collect` yardımcısını kullanacağız, her bir öğede `strtoupper` fonksiyonunu çalıştıracağız ve ardından tüm boş öğeleri kaldıracağız:
```php
$collection = collect(['Taylor', 'Abigail', null])->map(function (?string $name) {
    return strtoupper($name);
})->reject(function (string $name) {
    return empty($name);
});
```
Gördüğünüz gibi, `Collection` sınıfı, temeldeki dizi üzerinde akıcı bir şekilde haritalama (mapping) ve indirgeme (reducing) yapmak için metotlarını zincirlemenize (chain) olanak tanır. Genel olarak, koleksiyonlar değişmezdir (immutable), yani her `Collection` metodu tamamen yeni bir `Collection` örneği döndürür.

### Koleksiyon Oluşturma (Creating Collections)
Yukarıda belirtildiği gibi, `collect` yardımcısı, verilen dizi için yeni bir `Illuminate\Support\Collection` örneği döndürür. Yani, bir koleksiyon oluşturmak şu kadar basittir:
```php
$collection = collect([1, 2, 3]);
```
Ayrıca `make` ve `fromJson` metotlarını kullanarak da bir koleksiyon oluşturabilirsiniz.

Eloquent sorgularının sonuçları her zaman `Collection` örnekleri olarak döndürülür.

### Koleksiyonları Genişletme (Extending Collections)
Koleksiyonlar "genişletilebilir" (macroable), bu da çalışma zamanında `Collection` sınıfına ek metotlar eklemenize olanak tanır. `Illuminate\Support\Collection` sınıfının `macro` metodu, makronuz çağrıldığında yürütülecek bir closure kabul eder. Makro closure'ı, koleksiyon sınıfının gerçek bir metoduymuş gibi `$this` aracılığıyla koleksiyonun diğer metotlarına erişebilir. Örneğin, aşağıdaki kod `Collection` sınıfına bir `toUpper` metodu ekler:
```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```
Genellikle, koleksiyon makrolarını bir servis sağlayıcının (service provider) `boot` metodunda bildirmelisiniz.

#### Makro Argümanları (Macro Arguments)
Gerekirse, ek argümanlar kabul eden makrolar tanımlayabilirsiniz:
```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Lang;

Collection::macro('toLocale', function (string $locale) {
    return $this->map(function (string $value) use ($locale) {
        return Lang::get($value, [], $locale);
    });
});

$collection = collect(['first', 'second']);

$translated = $collection->toLocale('es');

// ['primero', 'segundo'];
```

## Kullanılabilir Metotlar (Available Methods)

Koleksiyon dokümantasyonunun geri kalanının çoğu için, `Collection` sınıfında bulunan her bir metodu tartışacağız. Unutmayın, bu metotların tümü, temeldeki diziyi akıcı bir şekilde işlemek için zincirlenebilir. Ayrıca, hemen hemen her metod yeni bir `Collection` örneği döndürür, bu da gerektiğinde koleksiyonun orijinal kopyasını korumanıza olanak tanır.

## Metot Listesi (Method Listing)

#### `after()` Metodu
`after` metodu, verilen öğeden sonraki öğeyi döndürür. Verilen öğe bulunamazsa veya son öğeyse `null` döndürülür:
```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->after(3);

// 4

$collection->after(5);

// null
```
Bu metod, verilen öğeyi "gevşek" (loose) karşılaştırma kullanarak arar; yani bir tamsayı değeri içeren bir dize, aynı değerdeki bir tamsayı ile eşit kabul edilir. "Sıkı" (strict) karşılaştırma kullanmak için metoda `strict` argümanını sağlayabilirsiniz:
```php
collect([2, 4, 6, 8])->after('4', strict: true);

// null
```
Alternatif olarak, belirli bir doğruluk testini (truth test) geçen ilk öğeyi aramak için kendi closure'ınızı sağlayabilirsiniz:
```php
collect([2, 4, 6, 8])->after(function (int $item, int $key) {
    return $item > 5;
});

// 8
```

#### `all()` Metodu
`all` metodu, koleksiyon tarafından temsil edilen temeldeki diziyi döndürür:
```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

#### `average()` Metodu
`avg` metodu için takma addır (alias).

#### `avg()` Metodu
`avg` metodu, belirli bir anahtarın ortalama değerini döndürür:
```php
$average = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

#### `before()` Metodu
`before` metodu, `after` metodunun tersidir. Verilen öğeden önceki öğeyi döndürür. Verilen öğe bulunamazsa veya ilk öğeyse `null` döndürülür:
```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->before(3);

// 2

$collection->before(1);

// null

collect([2, 4, 6, 8])->before('4', strict: true);

// null

collect([2, 4, 6, 8])->before(function (int $item, int $key) {
    return $item > 5;
});

// 4
```

#### `chunk()` Metodu
`chunk` metodu, koleksiyonu belirli bir boyutta birden çok, daha küçük koleksiyona böler:
```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->all();

// [[1, 2, 3, 4], [5, 6, 7]]
```
Bu metod, özellikle Bootstrap gibi bir ızgara sistemiyle çalışırken görünümlerde (views) kullanışlıdır. Örneğin, bir ızgarada görüntülemek istediğiniz bir Eloquent model koleksiyonunuz olduğunu hayal edin:
```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

#### `chunkWhile()` Metodu
`chunkWhile` metodu, verilen geri çağrının (callback) değerlendirilmesine dayalı olarak koleksiyonu birden çok, daha küçük koleksiyona böler. Closure'a iletilen `$chunk` değişkeni, önceki öğeyi incelemek için kullanılabilir:
```php
$collection = collect(str_split('AABBCCCD'));

$chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
    return $value === $chunk->last();
});

$chunks->all();

// [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]
```

#### `collapse()` Metodu
`collapse` metodu, bir dizi veya koleksiyon koleksiyonunu tek, düz bir koleksiyonda birleştirir:
```php
$collection = collect([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### `collapseWithKeys()` Metodu
`collapseWithKeys` metodu, bir dizi veya koleksiyon koleksiyonunu, orijinal anahtarları bozulmadan tek bir koleksiyonda düzleştirir. Koleksiyon zaten düzse, boş bir koleksiyon döndürecektir:
```php
$collection = collect([
    ['first'  => collect([1, 2, 3])],
    ['second' => [4, 5, 6]],
    ['third'  => collect([7, 8, 9])]
]);

$collapsed = $collection->collapseWithKeys();

$collapsed->all();

// [
//     'first'  => [1, 2, 3],
//     'second' => [4, 5, 6],
//     'third'  => [7, 8, 9],
// ]
```

#### `collect()` Metodu
`collect` metodu, koleksiyonda bulunan öğelerle yeni bir `Collection` örneği döndürür:
```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```
`collect` metodu öncelikle tembel koleksiyonları (lazy collections) standart `Collection` örneklerine dönüştürmek için kullanışlıdır:
```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

$collection::class;

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```
`collect` metodu, özellikle bir `Enumerable` örneğiniz olduğunda ve tembel olmayan (non-lazy) bir koleksiyon örneğine ihtiyacınız olduğunda kullanışlıdır. `collect()`, `Enumerable` sözleşmesinin bir parçası olduğundan, bir `Collection` örneği almak için onu güvenle kullanabilirsiniz.

#### `combine()` Metodu
`combine` metodu, koleksiyonun değerlerini anahtar olarak, başka bir dizi veya koleksiyonun değerleriyle birleştirir:
```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

#### `concat()` Metodu
`concat` metodu, verilen dizi veya koleksiyonun değerlerini başka bir koleksiyonun sonuna ekler:
```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```
`concat` metodu, orijinal koleksiyona eklenen öğeler için anahtarları sayısal olarak yeniden indeksler. İlişkisel (associative) koleksiyonlarda anahtarları korumak için `merge` metoduna bakın.

#### `contains()` Metodu
`contains` metodu, koleksiyonun belirli bir öğeyi içerip içermediğini belirler. Belirli bir doğruluk testine (truth test) uyan bir öğenin koleksiyonda bulunup bulunmadığını belirlemek için `contains` metoduna bir closure iletebilirsiniz:
```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function (int $value, int $key) {
    return $value > 5;
});

// false
```
Alternatif olarak, koleksiyonun belirli bir öğe değerini içerip içermediğini belirlemek için `contains` metoduna bir dize iletebilirsiniz:
```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```
Ayrıca, `contains` metoduna bir anahtar/değer çifti iletebilirsiniz; bu, verilen çiftin koleksiyonda mevcut olup olmadığını belirleyecektir:
```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```
`contains` metodu, öğe değerlerini kontrol ederken "gevşek" (loose) karşılaştırmalar kullanır; yani bir tamsayı değeri içeren bir dize, aynı değerdeki bir tamsayı ile eşit kabul edilir. "Sıkı" (strict) karşılaştırmalar kullanarak filtrelemek için `containsStrict` metodunu kullanın.

`contains` metodunun tersi için `doesntContain` metoduna bakın.

#### `containsStrict()` Metodu
Bu metod, `contains` metoduyla aynı imzaya sahiptir; ancak, tüm değerler "sıkı" (strict) karşılaştırmalar kullanılarak karşılaştırılır.

#### `count()` Metodu
`count` metodu, koleksiyondaki toplam öğe sayısını döndürür:
```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

#### `countBy()` Metodu
`countBy` metodu, koleksiyondaki değerlerin oluşumlarını (occurrences) sayar. Varsayılan olarak, metod her öğenin oluşumlarını sayarak koleksiyondaki belirli "türdeki" öğeleri saymanıza olanak tanır:
```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```
Tüm öğeleri özel bir değere göre saymak için `countBy` metoduna bir closure iletebilirsiniz:
```php
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

$counted = $collection->countBy(function (string $email) {
    return substr(strrchr($email, '@'), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

#### `crossJoin()` Metodu
`crossJoin` metodu, koleksiyonun değerlerini verilen diziler veya koleksiyonlar arasında çapraz birleştirir (cross joins), tüm olası permütasyonlarla bir Kartezyen çarpım (Cartesian product) döndürür:
```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

#### `dd()` Metodu
`dd` metodu, koleksiyonun öğelerini döker (dump) ve betiğin yürütülmesini sonlandırır:
```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
Betiğin yürütülmesini durdurmak istemiyorsanız, bunun yerine `dump` metodunu kullanın.

#### `diff()` Metodu
`diff` metodu, koleksiyonu değerlerine göre başka bir koleksiyon veya düz bir PHP dizisiyle karşılaştırır. Bu metod, orijinal koleksiyonda bulunan ancak verilen koleksiyonda bulunmayan değerleri döndürecektir:
```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

#### `diffAssoc()` Metodu
`diffAssoc` metodu, koleksiyonu anahtarlarına ve değerlerine göre başka bir koleksiyon veya düz bir PHP dizisiyle karşılaştırır. Bu metod, orijinal koleksiyonda bulunan ancak verilen koleksiyonda bulunmayan anahtar/değer çiftlerini döndürecektir:
```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```