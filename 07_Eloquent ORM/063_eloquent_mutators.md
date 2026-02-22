# Laravel 12 Dokümantasyonu: Eloquent: Değiştiriciler ve Dönüşüm (Eloquent: Mutators & Casting)

## Giriş

Erişimciler (accessors), değiştiriciler (mutators) ve nitelik dönüşümü (attribute casting), Eloquent model örneklerinde nitelik değerlerini alırken veya ayarlarken dönüştürmenize olanak tanır. Örneğin, veritabanında saklanırken bir değeri şifrelemek için Laravel şifreleyicisini (encrypter) kullanmak ve ardından bir Eloquent modelinde bu niteliğe eriştiğinizde otomatik olarak şifresini çözmek isteyebilirsiniz. Veya, veritabanınızda saklanan bir JSON dizesini, Eloquent modeliniz aracılığıyla erişildiğinde bir diziye dönüştürmek isteyebilirsiniz.

## Erişimciler ve Değiştiriciler (Accessors and Mutators)

### Erişimci (Accessor) Tanımlama
Bir erişimci (accessor), bir Eloquent niteliğine erişildiğinde değerini dönüştürür. Bir erişimci tanımlamak için, modelinizde erişilebilir niteliği temsil edecek şekilde `protected` bir metot oluşturun. Bu metodun adı, uygun olduğunda gerçek temel model niteliğinin / veritabanı sütununun "camelCase" temsiline karşılık gelmelidir.

Bu örnekte, `first_name` niteliği için bir erişimci tanımlayacağız. Erişimci, `first_name` niteliğinin değerini almaya çalışırken Eloquent tarafından otomatik olarak çağrılacaktır. Tüm nitelik erişimci / değiştirici metotları, bir `Illuminate\Database\Eloquent\Casts\Attribute` dönüş türü ipucu (return type-hint) bildirmelidir:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Kullanıcının adını al.
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}
```
Tüm erişimci metotları, niteliğe nasıl erişileceğini ve isteğe bağlı olarak nasıl değiştirileceğini (mutate) tanımlayan bir `Attribute` örneği döndürür. Bu örnekte, yalnızca niteliğe nasıl erişileceğini tanımlıyoruz. Bunu yapmak için, `Attribute` sınıfı kurucusuna `get` argümanını sağlıyoruz.

Gördüğünüz gibi, sütunun orijinal değeri erişimciye iletilir, böylece değeri işleyip döndürebilirsiniz. Erişimcinin değerine erişmek için, bir model örneğinde `first_name` niteliğine erişmeniz yeterlidir:
```php
use App\Models\User;

$user = User::find(1);

$firstName = $user->first_name;
```
Bu hesaplanmış değerlerin modelinizin dizi / JSON temsillerine eklenmesini istiyorsanız, bunları eklemeniz (append) gerekecektir.

#### Birden Çok Nitelikten Değer Nesneleri (Value Objects) Oluşturma
Bazen erişimcinizin birden çok model niteliğini tek bir "değer nesnesine" (value object) dönüştürmesi gerekebilir. Bunu yapmak için `get` closure'ınız, otomatik olarak closure'a sağlanacak ve modelin mevcut tüm niteliklerinin bir dizisini içerecek olan ikinci bir `$attributes` argümanı kabul edebilir:
```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Kullanıcının adresiyle etkileşime gir.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

#### Erişimci Önbellekleme (Accessor Caching)
Erişimcilerden değer nesneleri döndürürken, değer nesnesinde yapılan herhangi bir değişiklik, model kaydedilmeden önce otomatik olarak modele geri senkronize edilecektir. Bu mümkündür çünkü Eloquent, erişimciler tarafından döndürülen örnekleri saklar, böylece erişimci her çağrıldığında aynı örneği döndürebilir:
```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Güncellenmiş Adres Satırı 1 Değeri';
$user->address->lineTwo = 'Güncellenmiş Adres Satırı 2 Değeri';

$user->save();
```
Ancak, bazen özellikle hesaplama açısından yoğunsalar (computationally intensive), dizeler ve boolean'lar gibi ilkel değerler (primitive values) için önbelleğe almayı etkinleştirmek isteyebilirsiniz. Bunu başarmak için, erişimcinizi tanımlarken `shouldCache` metodunu çağırabilirsiniz:
```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```
Niteliklerin nesne önbelleğe alma davranışını devre dışı bırakmak isterseniz, niteliği tanımlarken `withoutObjectCaching` metodunu çağırabilirsiniz:
```php
/**
 * Kullanıcının adresiyle etkileşime gir.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

### Değiştirici (Mutator) Tanımlama
Bir değiştirici (mutator), bir Eloquent niteliği ayarlandığında (set) değerini dönüştürür. Bir değiştirici tanımlamak için, niteliğinizi tanımlarken `set` argümanını sağlayabilirsiniz. `first_name` niteliği için bir değiştirici tanımlayalım. Bu değiştirici, model üzerinde `first_name` niteliğinin değerini ayarlamaya çalıştığımızda otomatik olarak çağrılacaktır:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Kullanıcının adıyla etkileşime gir.
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value),
        );
    }
}
```
Değiştirici closure'ı, nitelikte ayarlanmakta olan değeri alacak ve bu değeri işleyip işlenmiş değeri döndürmenize olanak tanıyacaktır. Değiştiricimizi kullanmak için, bir Eloquent modelinde `first_name` niteliğini ayarlamamız yeterlidir:
```php
use App\Models\User;

$user = User::find(1);

$user->first_name = 'Sally';
```
Bu örnekte, `set` geri çağrısı `Sally` değeriyle çağrılacaktır. Değiştirici daha sonra `strtolower` fonksiyonunu isme uygulayacak ve sonuç değerini modelin dahili `$attributes` dizisinde ayarlayacaktır.

#### Birden Çok Niteliği Değiştirme (Mutating Multiple Attributes)
Bazen değiştiricinizin temeldeki modelde birden çok nitelik ayarlaması gerekebilir. Bunu yapmak için, `set` closure'ından bir dizi döndürebilirsiniz. Dizideki her anahtar, modelle ilişkili temel bir niteliğe / veritabanı sütununa karşılık gelmelidir:
```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Kullanıcının adresiyle etkileşime gir.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

## Nitelik Dönüşümü (Attribute Casting)

Nitelik dönüşümü (attribute casting), modelinizde herhangi bir ek metot tanımlamanızı gerektirmeden erişimciler ve değiştiricilere benzer işlevsellik sağlar. Bunun yerine, modelinizin `casts` metodu, nitelikleri yaygın veri türlerine dönüştürmenin kullanışlı bir yolunu sağlar.

`casts` metodu, anahtarın dönüştürülen niteliğin adı ve değerin sütunu dönüştürmek istediğiniz tür olduğu bir dizi döndürmelidir. Desteklenen dönüşüm türleri şunlardır:

*   `array`
*   `AsFluent::class`
*   `AsStringable::class`
*   `AsUri::class`
*   `boolean`
*   `collection`
*   `date`
*   `datetime`
*   `immutable_date`
*   `immutable_datetime`
*   `decimal:<precision>`
*   `double`
*   `encrypted`
*   `encrypted:array`
*   `encrypted:collection`
*   `encrypted:object`
*   `float`
*   `hashed`
*   `integer`
*   `object`
*   `real`
*   `string`
*   `timestamp`

Nitelik dönüşümünü göstermek için, veritabanımızda bir tamsayı (0 veya 1) olarak saklanan `is_admin` niteliğini bir boolean değerine dönüştürelim:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Dönüştürülmesi gereken nitelikleri al.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'is_admin' => 'boolean',
        ];
    }
}
```
Dönüşüm tanımlandıktan sonra, `is_admin` niteliğine eriştiğinizde, temeldeki değer veritabanında bir tamsayı olarak saklansa bile her zaman bir boole değerine dönüştürülecektir:
```php
$user = App\Models\User::find(1);

if ($user->is_admin) {
    // ...
}
```
Çalışma zamanında yeni, geçici bir dönüşüm eklemeniz gerekirse, `mergeCasts` metodunu kullanabilirsiniz. Bu dönüşüm tanımları, modelde zaten tanımlanmış olan tüm dönüşümlere eklenecektir:
```php
$user->mergeCasts([
    'is_admin' => 'integer',
    'options' => 'object',
]);
```
`null` olan nitelikler dönüştürülmeyecektir. Ayrıca, bir ilişkiyle (relationship) aynı ada sahip bir dönüşüm (veya bir nitelik) asla tanımlamamalı veya modelin birincil anahtarına bir dönüşüm atamamalısınız.

#### Stringable Dönüşüm (Stringable Casting)
Bir model niteliğini akıcı bir `Illuminate\Support\Stringable` nesnesine dönüştürmek için `Illuminate\Database\Eloquent\Casts\AsStringable` dönüşüm sınıfını kullanabilirsiniz:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\AsStringable;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Dönüştürülmesi gereken nitelikleri al.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'directory' => AsStringable::class,
        ];
    }
}
```

### Dizi (Array) ve JSON Dönüşümü
`array` dönüşümü, özellikle serileştirilmiş JSON olarak depolanan sütunlarla çalışırken kullanışlıdır. Örneğin, veritabanınız serileştirilmiş JSON içeren bir JSON veya TEXT alan türüne sahipse, bu niteliğe `array` dönüşümünü eklemek, Eloquent modelinizde ona eriştiğinizde niteliği otomatik olarak bir PHP dizisine dönüştürecektir (deserialize):
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Dönüştürülmesi gereken nitelikleri al.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => 'array',
        ];
    }
}
```
Dönüşüm tanımlandıktan sonra, `options` niteliğine erişebilirsiniz ve otomatik olarak JSON'dan bir PHP dizisine dönüştürülecektir. `options` niteliğinin değerini ayarladığınızda, verilen dizi depolama için otomatik olarak JSON'a geri serileştirilecektir (serialize):
```php
use App\Models\User;

$user = User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```
Bir JSON niteliğinin tek bir alanını daha kısa bir sözdizimiyle güncellemek için, niteliği toplu olarak atanabilir (mass assignable) yapabilir ve `update` yöntemini çağırırken `->` operatörünü kullanabilirsiniz:
```php
$user = User::find(1);

$user->update(['options->key' => 'value']);
```

#### JSON ve Unicode
Bir dizi niteliğini kaçışsız Unicode karakterlerle JSON olarak depolamak isterseniz, `json:unicode` dönüşümünü kullanabilirsiniz:
```php
/**
 * Dönüştürülmesi gereken nitelikleri al.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => 'json:unicode',
    ];
}
```

#### Dizi Nesnesi (Array Object) ve Koleksiyon Dönüşümü
Standart `array` dönüşümü birçok uygulama için yeterli olsa da bazı dezavantajları vardır. `array` dönüşümü ilkel bir tür döndürdüğü için, dizinin bir ofsetini doğrudan değiştirmek mümkün değildir. Örneğin, aşağıdaki kod bir PHP hatasını tetikleyecektir:
```php
$user = User::find(1);

$user->options['key'] = $value;
```
Bunu çözmek için Laravel, JSON niteliğinizi bir `ArrayObject` sınıfına dönüştüren bir `AsArrayObject` dönüşümü sunar. Bu özellik, Laravel'in özel dönüşüm uygulaması (custom cast implementation) kullanılarak uygulanır; bu, Laravel'in değiştirilmiş nesneyi akıllıca önbelleğe almasına ve dönüştürmesine olanak tanır, böylece tek tek ofsetler bir PHP hatasını tetiklemeden değiştirilebilir. `AsArrayObject` dönüşümünü kullanmak için, onu bir niteliğe atamanız yeterlidir:
```php
use Illuminate\Database\Eloquent\Casts\AsArrayObject;

/**
 * Dönüştürülmesi gereken nitelikleri al.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsArrayObject::class,
    ];
}
```
Benzer şekilde Laravel, JSON niteliğinizi bir Laravel `Collection` örneğine dönüştüren bir `AsCollection` dönüşümü sunar:
```php
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Dönüştürülmesi gereken nitelikleri al.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::class,
    ];
}
```
`AsCollection` dönüşümünün Laravel'in temel koleksiyon sınıfı yerine özel bir koleksiyon sınıfı oluşturmasını istiyorsanız, dönüşüm argümanı olarak koleksiyon sınıfı adını sağlayabilirsiniz:
```php
use App\Collections\OptionCollection;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Dönüştürülmesi gereken nitelikleri al.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::using(OptionCollection::class),
    ];
}
```
`of` metodu, koleksiyon öğelerinin, koleksiyonun `mapInto` metodu aracılığıyla belirli bir sınıfa eşlenmesi gerektiğini belirtmek için kullanılabilir:
```php
use App\ValueObjects\Option;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Dönüştürülmesi gereken nitelikleri al.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::of(Option::class)
    ];
}
```
Koleksiyonları nesnelere eşlerken, nesne, örneklerinin JSON olarak veritabanına nasıl serileştirileceğini tanımlamak için `Illuminate\Contracts\Support\Arrayable` ve `JsonSerializable` arayüzlerini uygulamalıdır:
```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Support\Arrayable;
use JsonSerializable;

class Option implements Arrayable, JsonSerializable
{
    public string $name;
    public mixed $value;
    public bool $isLocked;

    /**
     * Yeni bir Option örneği oluştur.
     */
    public function __construct(array $data)
    {
        $this->name = $data['name'];
        $this->value = $data['value'];
        $this->isLocked = $data['is_locked'];
    }

    /**
     * Örneği bir dizi olarak al.
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'data' => $this->value,
            'is_locked' => $this->isLocked,
        ];
    }

    /**
     * Örneği JSON olarak serileştirmek için gereken verileri al.
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function jsonSerialize(): array
    {
        return $this->toArray();
    }
}
```