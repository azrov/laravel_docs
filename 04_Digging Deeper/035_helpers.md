# Laravel 12 Dokümantasyonu: Yardımcılar (Helpers)

## Giriş

Laravel, çeşitli global "yardımcı" PHP fonksiyonları içerir. Bu fonksiyonların çoğu framework'ün kendisi tarafından kullanılır; ancak, kullanışlı bulursanız bunları kendi uygulamalarınızda kullanmakta özgürsünüz.

## Mevcut Metotlar (Available Methods)

*   Diziler ve Nesneler (Arrays & Objects)
*   Sayılar (Numbers)
*   Yollar (Paths)
*   URL'ler
*   Çeşitli (Miscellaneous)

## Diziler ve Nesneler (Arrays & Objects)

#### `Arr::accessible()` Metodu
`Arr::accessible` metodu, verilen değerin dizi olarak erişilebilir (array accessible) olup olmadığını belirler:
```php
use Illuminate\Support\Arr;
use Illuminate\Support\Collection;

$isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

// true

$isAccessible = Arr::accessible(new Collection);

// true

$isAccessible = Arr::accessible('abc');

// false

$isAccessible = Arr::accessible(new stdClass);

// false
```

#### `Arr::add()` Metodu
`Arr::add` metodu, verilen anahtar (key) dizide zaten mevcut değilse veya `null` olarak ayarlanmışsa, diziye bir anahtar/değer çifti ekler:
```php
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]

$array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

#### `Arr::array()` Metodu
`Arr::array` metodu, "nokta" notasyonunu kullanarak iç içe geçmiş bir diziden değer alır (tıpkı `Arr::get()` gibi), ancak istenen değer bir dizi değilse bir `InvalidArgumentException` fırlatır:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$value = Arr::array($array, 'languages');

// ['PHP', 'Ruby']

$value = Arr::array($array, 'name');

// throws InvalidArgumentException
```

#### `Arr::boolean()` Metodu
`Arr::boolean` metodu, "nokta" notasyonunu kullanarak iç içe geçmiş bir diziden değer alır (tıpkı `Arr::get()` gibi), ancak istenen değer bir boolean değilse bir `InvalidArgumentException` fırlatır:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'available' => true];

$value = Arr::boolean($array, 'available');

// true

$value = Arr::boolean($array, 'name');

// throws InvalidArgumentException
```

#### `Arr::collapse()` Metodu
`Arr::collapse` metodu, bir dizi veya koleksiyonlar dizisini tek bir dizi halinde birleştirir:
```php
use Illuminate\Support\Arr;

$array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### `Arr::crossJoin()` Metodu
`Arr::crossJoin` metodu, verilen dizileri çapraz birleştirir (cross joins), tüm olası permütasyonlarla bir Kartezyen çarpım (Cartesian product) döndürür:
```php
use Illuminate\Support\Arr;

$matrix = Arr::crossJoin([1, 2], ['a', 'b']);

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

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

#### `Arr::divide()` Metodu
`Arr::divide` metodu iki dizi döndürür: biri verilen dizinin anahtarlarını (keys), diğeri ise değerlerini (values) içerir:
```php
use Illuminate\Support\Arr;

[$keys, $values] = Arr::divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
```

#### `Arr::dot()` Metodu
`Arr::dot` metodu, çok boyutlu bir diziyi, derinliği belirtmek için "nokta" notasyonunu kullanan tek seviyeli bir diziye düzleştirir (flattens):
```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$flattened = Arr::dot($array);

// ['products.desk.price' => 100]
```

#### `Arr::every()` Metodu
`Arr::every` metodu, dizideki tüm değerlerin belirli bir doğruluk testini (truth test) geçtiğinden emin olur:
```php
use Illuminate\Support\Arr;

$array = [1, 2, 3];

Arr::every($array, fn ($i) => $i > 0);

// true

Arr::every($array, fn ($i) => $i > 2);

// false
```

#### `Arr::except()` Metodu
`Arr::except` metodu, verilen anahtar/değer çiftlerini bir diziden kaldırır:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$filtered = Arr::except($array, ['price']);

// ['name' => 'Desk']
```

#### `Arr::exceptValues()` Metodu
`Arr::exceptValues` metodu, belirtilen değerleri bir diziden kaldırır:
```php
use Illuminate\Support\Arr;

$array = ['foo', 'bar', 'baz', 'qux'];

$filtered = Arr::exceptValues($array, ['foo', 'baz']);

// ['bar', 'qux']
```
Filtreleme sırasında sıkı tür karşılaştırmaları (strict type comparisons) kullanmak için `strict` argümanına `true` iletebilirsiniz:
```php
use Illuminate\Support\Arr;

$array = [1, '1', 2, '2'];

$filtered = Arr::exceptValues($array, [1, 2], strict: true);

// ['1', '2']
```

#### `Arr::exists()` Metodu
`Arr::exists` metodu, verilen anahtarın sağlanan dizide mevcut olup olmadığını kontrol eder:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'John Doe', 'age' => 17];

$exists = Arr::exists($array, 'name');

// true

$exists = Arr::exists($array, 'salary');

// false
```

#### `Arr::first()` Metodu
`Arr::first` metodu, belirli bir doğruluk testini geçen bir dizinin ilk elemanını döndürür:
```php
use Illuminate\Support\Arr;

$array = [100, 200, 300];

$first = Arr::first($array, function (int $value, int $key) {
    return $value >= 150;
});

// 200
```
Metoda üçüncü parametre olarak bir varsayılan değer de iletilebilir. Hiçbir değer doğruluk testini geçmezse bu değer döndürülecektir:
```php
use Illuminate\Support\Arr;

$first = Arr::first($array, $callback, $default);
```

#### `Arr::flatten()` Metodu
`Arr::flatten` metodu, çok boyutlu bir diziyi tek seviyeli bir diziye düzleştirir:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$flattened = Arr::flatten($array);

// ['Joe', 'PHP', 'Ruby']
```

#### `Arr::float()` Metodu
`Arr::float` metodu, "nokta" notasyonunu kullanarak iç içe geçmiş bir diziden değer alır (tıpkı `Arr::get()` gibi), ancak istenen değer bir float değilse bir `InvalidArgumentException` fırlatır:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'balance' => 123.45];

$value = Arr::float($array, 'balance');

// 123.45

$value = Arr::float($array, 'name');

// throws InvalidArgumentException
```

#### `Arr::forget()` Metodu
`Arr::forget` metodu, "nokta" notasyonunu kullanarak iç içe geçmiş bir diziden belirli anahtar/değer çiftlerini kaldırır:
```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::forget($array, 'products.desk');

// ['products' => []]
```

#### `Arr::from()` Metodu
`Arr::from` metodu, çeşitli girdi türlerini düz bir PHP dizisine dönüştürür. Diziler, nesneler ve `Arrayable`, `Enumerable`, `Jsonable`, `JsonSerializable` gibi birkaç yaygın Laravel arayüzü dahil olmak üzere çeşitli girdi türlerini destekler. Ayrıca `Traversable` ve `WeakMap` örneklerini de işler:
```php
use Illuminate\Support\Arr;

Arr::from((object) ['foo' => 'bar']); // ['foo' => 'bar']

class TestJsonableObject implements Jsonable
{
    public function toJson($options = 0)
    {
        return json_encode(['foo' => 'bar']);
    }
}

Arr::from(new TestJsonableObject); // ['foo' => 'bar']
```

#### `Arr::get()` Metodu
`Arr::get` metodu, "nokta" notasyonunu kullanarak iç içe geçmiş bir diziden değer alır:
```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```
`Arr::get` metodu ayrıca, belirtilen anahtar dizide yoksa döndürülecek bir varsayılan değer kabul eder:
```php
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

#### `Arr::has()` Metodu
`Arr::has` metodu, "nokta" notasyonunu kullanarak belirli bir öğenin veya öğelerin bir dizide mevcut olup olmadığını kontrol eder:
```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');

// true

$contains = Arr::has($array, ['product.price', 'product.discount']);

// false
```

#### `Arr::hasAll()` Metodu
`Arr::hasAll` metodu, "nokta" notasyonunu kullanarak belirtilen tüm anahtarların verilen dizide mevcut olup olmadığını belirler:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Taylor', 'language' => 'PHP'];

Arr::hasAll($array, ['name']); // true
Arr::hasAll($array, ['name', 'language']); // true
Arr::hasAll($array, ['name', 'IDE']); // false
```

#### `Arr::hasAny()` Metodu
`Arr::hasAny` metodu, "nokta" notasyonunu kullanarak belirli bir kümedeki herhangi bir öğenin bir dizide mevcut olup olmadığını kontrol eder:
```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::hasAny($array, 'product.name');

// true

$contains = Arr::hasAny($array, ['product.name', 'product.discount']);

// true

$contains = Arr::hasAny($array, ['category', 'product.discount']);

// false
```

#### `Arr::integer()` Metodu
`Arr::integer` metodu, "nokta" notasyonunu kullanarak iç içe geçmiş bir diziden değer alır (tıpkı `Arr::get()` gibi), ancak istenen değer bir tamsayı (int) değilse bir `InvalidArgumentException` fırlatır:
```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'age' => 42];

$value = Arr::integer($array, 'age');

// 42

$value = Arr::integer($array, 'name');

// throws InvalidArgumentException
```

#### `Arr::isAssoc()` Metodu
`Arr::isAssoc` metodu, verilen dizi ilişkisel bir dizi (associative array) ise `true` döndürür. Bir dizi, sıfırdan başlayan ardışık sayısal anahtarlara sahip değilse "ilişkisel" olarak kabul edilir:
```php
use Illuminate\Support\Arr;

$isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

// true

$isAssoc = Arr::isAssoc([1, 2, 3]);

// false
```

#### `Arr::isList()` Metodu
`Arr::isList` metodu, verilen dizinin anahtarları sıfırdan başlayan ardışık tamsayılar ise `true` döndürür:
```php
use Illuminate\Support\Arr;

$isList = Arr::isList(['foo', 'bar', 'baz']);

// true

$isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

// false
```

#### `Arr::join()` Metodu
`Arr::join` metodu, dizi elemanlarını bir dize ile birleştirir. Bu metodun üçüncü argümanını kullanarak, dizinin son elemanı için birleştirme dizesini de belirtebilirsiniz:
```php
use Illuminate\Support\Arr;

$array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

$joined = Arr::join($array, ', ');

// Tailwind, Alpine, Laravel, Livewire

$joined = Arr::join($array, ', ', ', and ');

// Tailwind, Alpine, Laravel, and Livewire
```

#### `Arr::keyBy()` Metodu
`Arr::keyBy` metodu, diziyi verilen anahtara göre anahtarlar (keys the array by the given key). Birden çok öğe aynı anahtara sahipse, yeni dizide yalnızca sonuncusu görünecektir:
```php
use Illuminate\Support\Arr;

$array = [
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
];

$keyed = Arr::keyBy($array, 'product_id');

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

#### `Arr::last()` Metodu
`Arr::last` metodu, belirli bir doğruluk testini geçen bir dizinin son elemanını döndürür:
```php
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function (int $value, int $key) {
    return $value >= 150;
});

// 300
```
Metoda üçüncü argüman olarak bir varsayılan değer iletilebilir. Hiçbir değer doğruluk testini geçmezse bu değer döndürülecektir:
```php
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

#### `Arr::map()` Metodu
`Arr::map` metodu, dizi boyunca yinelenir (iterates) ve her değeri ve anahtarı verilen geri çağrıya (callback) iletir. Dizi değeri, geri çağrı tarafından döndürülen değerle değiştirilir:
```php
use Illuminate\Support\Arr;

$array = ['first' => 'james', 'last' => 'kirk'];

$mapped = Arr::map($array, function (string $value, string $key) {
    return ucfirst($value);
});

// ['first' => 'James', 'last' => 'Kirk']
```