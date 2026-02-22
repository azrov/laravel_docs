# Laravel 12 Dokümantasyonu: Eloquent: Serileştirme (Eloquent: Serialization)

## Giriş

Laravel kullanarak API'ler oluştururken, modellerinizi ve ilişkilerinizi sık sık dizilere veya JSON'a dönüştürmeniz gerekecektir. Eloquent, bu dönüşümleri yapmak ve modellerinizin serileştirilmiş temsiline hangi niteliklerin dahil edileceğini kontrol etmek için kullanışlı metotlar içerir.

Eloquent model ve koleksiyon JSON serileştirmesini ele almanın daha da güçlü bir yolu için Eloquent API kaynakları hakkındaki dokümantasyona göz atın.

## Modelleri ve Koleksiyonları Serileştirme (Serializing Models and Collections)

### Dizilere Serileştirme (Serializing to Arrays)
Bir modeli ve yüklenmiş ilişkilerini bir diziye dönüştürmek için `toArray` metodunu kullanmalısınız. Bu metod özyinelemelidir (recursive), bu nedenle tüm nitelikler ve tüm ilişkiler (ilişkilerin ilişkileri dahil) dizilere dönüştürülecektir:
```php
use App\Models\User;

$user = User::with('roles')->first();

return $user->toArray();
```
`attributesToArray` metodu, bir modelin niteliklerini ancak ilişkilerini değil, bir diziye dönüştürmek için kullanılabilir:
```php
$user = User::first();

return $user->attributesToArray();
```
Ayrıca, koleksiyon örneğinde `toArray` metodunu çağırarak tüm model koleksiyonlarını dizilere dönüştürebilirsiniz:
```php
$users = User::all();

return $users->toArray();
```

### JSON'a Serileştirme (Serializing to JSON)
Bir modeli JSON'a dönüştürmek için `toJson` metodunu kullanmalısınız. `toArray` gibi, `toJson` metodu da özyinelemelidir, bu nedenle tüm nitelikler ve ilişkiler JSON'a dönüştürülecektir. Ayrıca, PHP tarafından desteklenen herhangi bir JSON kodlama seçeneğini belirtebilirsiniz:
```php
use App\Models\User;

$user = User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```
Alternatif olarak, bir modeli veya koleksiyonu bir dizeye dönüştürebilirsiniz (cast), bu otomatik olarak model veya koleksiyon üzerinde `toJson` metodunu çağıracaktır:
```php
return (string) User::find(1);
```
Modeller ve koleksiyonlar bir dizeye dönüştürüldüklerinde JSON'a dönüştürüldükleri için, Eloquent nesnelerini doğrudan uygulamanızın rotalarından veya controller'larından döndürebilirsiniz. Laravel, rotalardan veya controller'lardan döndürüldüklerinde Eloquent modellerinizi ve koleksiyonlarınızı otomatik olarak JSON'a serileştirecektir:
```php
Route::get('/users', function () {
    return User::all();
});
```

#### İlişkiler (Relationships)
Bir Eloquent modeli JSON'a dönüştürüldüğünde, yüklenmiş ilişkileri otomatik olarak JSON nesnesinde nitelikler olarak dahil edilecektir. Ayrıca, Eloquent ilişki metotları "camelCase" metot adları kullanılarak tanımlansa da, bir ilişkinin JSON niteliği "snake_case" olacaktır.

## Nitelikleri JSON'dan Gizleme (Hiding Attributes From JSON)

Bazen, parolalar gibi niteliklerin modelinizin dizi veya JSON temsiline dahil edilmesini sınırlamak isteyebilirsiniz. Bunu yapmak için modelinize bir `$hidden` özelliği ekleyin. `$hidden` özelliğinin dizisinde listelenen nitelikler, modelinizin serileştirilmiş temsiline dahil edilmeyecektir:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Serileştirme için gizlenmesi gereken nitelikler.
     *
     * @var array<string>
     */
    protected $hidden = ['password'];
}
```
İlişkileri gizlemek için, ilişkinin metot adını Eloquent modelinizin `$hidden` özelliğine ekleyin.

Alternatif olarak, modelinizin dizi ve JSON temsiline dahil edilmesi gereken niteliklerin bir "izin listesini" (allow list) tanımlamak için `visible` özelliğini kullanabilirsiniz. `$visible` dizisinde bulunmayan tüm nitelikler, model bir diziye veya JSON'a dönüştürüldüğünde gizlenecektir:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Dizilerde görünür olması gereken nitelikler.
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

#### Nitelik Görünürlüğünü Geçici Olarak Değiştirme (Temporarily Modifying Attribute Visibility)
Belirli bir model örneğinde normalde gizli olan bazı nitelikleri görünür hale getirmek isterseniz, `makeVisible` veya `mergeVisible` metotlarını kullanabilirsiniz. `makeVisible` metodu, model örneğini döndürür:
```php
return $user->makeVisible('attribute')->toArray();

return $user->mergeVisible(['name', 'email'])->toArray();
```
Benzer şekilde, normalde görünür olan bazı nitelikleri gizlemek isterseniz, `makeHidden` veya `mergeHidden` metotlarını kullanabilirsiniz:
```php
return $user->makeHidden('attribute')->toArray();

return $user->mergeHidden(['name', 'email'])->toArray();
```
Tüm görünür veya gizli nitelikleri geçici olarak geçersiz kılmak isterseniz, sırasıyla `setVisible` ve `setHidden` metotlarını kullanabilirsiniz:
```php
return $user->setVisible(['id', 'name'])->toArray();

return $user->setHidden(['email', 'password', 'remember_token'])->toArray();
```

## JSON'a Değer Ekleme (Appending Values to JSON)

Bazen, modelleri dizilere veya JSON'a dönüştürürken, veritabanınızda karşılık gelen bir sütunu olmayan nitelikler eklemek isteyebilirsiniz. Bunu yapmak için önce değer için bir erişimci (accessor) tanımlayın:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Kullanıcının yönetici olup olmadığını belirle.
     */
    protected function isAdmin(): Attribute
    {
        return new Attribute(
            get: fn () => 'yes',
        );
    }
}
```
Erişimcinin her zaman modelinizin dizi ve JSON temsillerine eklenmesini istiyorsanız, nitelik adını modelin `appends` özelliğine ekleyebilirsiniz. Erişimcinin PHP metodu "camelCase" kullanılarak tanımlanmış olsa da, nitelik adlarının genellikle "snake_case" serileştirilmiş temsilleri kullanılarak başvurulduğunu unutmayın:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Modelin dizi formuna eklenecek erişimciler.
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```
Nitelik `appends` listesine eklendikten sonra, hem modelin dizi hem de JSON temsillerine dahil edilecektir. `appends` dizisindeki nitelikler ayrıca modelde yapılandırılan `visible` ve `hidden` ayarlarına da saygı gösterecektir.

#### Çalışma Zamanında Ekleme (Appending at Run Time)
Çalışma zamanında, bir model örneğine `append` veya `mergeAppends` metotlarını kullanarak ek nitelikler eklemesi talimatını verebilirsiniz. Veya, belirli bir model örneği için eklenen özelliklerin tüm dizisini geçersiz kılmak üzere `setAppends` metodunu kullanabilirsiniz:
```php
return $user->append('is_admin')->toArray();

return $user->mergeAppends(['is_admin', 'status'])->toArray();

return $user->setAppends(['is_admin'])->toArray();
```
Benzer şekilde, bir modelden eklenmiş tüm özellikleri kaldırmak isterseniz, `withoutAppends` metodunu kullanabilirsiniz:
```php
return $user->withoutAppends()->toArray();
```

## Tarih Serileştirmesi (Date Serialization)

#### Varsayılan Tarih Biçimini Özelleştirme (Customizing the Default Date Format)
`serializeDate` metodunu geçersiz kılarak (override) varsayılan serileştirme biçimini özelleştirebilirsiniz. Bu metod, tarihlerinizin veritabanında depolama için nasıl biçimlendirildiğini etkilemez:
```php
/**
 * Dizi / JSON serileştirmesi için bir tarih hazırla.
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

#### Nitelik Başına Tarih Biçimini Özelleştirme (Customizing the Date Format per Attribute)
Tek tek Eloquent tarih niteliklerinin serileştirme biçimini, modelin dönüşüm bildirimlerinde (cast declarations) tarih biçimini belirterek özelleştirebilirsiniz:
```php
protected function casts(): array
{
    return [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
}
```