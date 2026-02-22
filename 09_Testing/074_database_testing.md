# Laravel 12 Dokümantasyonu: Veritabanı Testleri (Database Testing)

## Giriş

Laravel, veritabanı odaklı uygulamalarınızı test etmeyi kolaylaştırmak için çeşitli yararlı araçlar ve iddialar (assertions) sağlar. Ayrıca, Laravel model fabrikaları (model factories) ve tohumlayıcılar (seeders), uygulamanızın Eloquent modellerini ve ilişkilerini kullanarak test veritabanı kayıtları oluşturmayı zahmetsiz hale getirir. Bu güçlü özelliklerin tümünü aşağıdaki dokümantasyonda ele alacağız.

### Her Testten Sonra Veritabanını Sıfırlama (Resetting the Database After Each Test)
Daha fazla ilerlemeden önce, önceki testten gelen verilerin sonraki testlere müdahale etmemesi için her testten sonra veritabanınızı nasıl sıfırlayacağınızı tartışalım. Laravel'in dahili `Illuminate\Foundation\Testing\RefreshDatabase` özelliği (trait) bunu sizin için halledecektir. Test sınıfınızda bu özelliği kullanmanız yeterlidir:
```php
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('temel örnek', function () {
    $response = $this->get('/');

    // ...
});
```
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Temel bir fonksiyonel test örneği.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```
`Illuminate\Foundation\Testing\RefreshDatabase` özelliği, şemanız (schema) güncelse veritabanınızı migrate etmez. Bunun yerine, testi yalnızca bir veritabanı işlemi (database transaction) içinde yürütür. Bu nedenle, bu özelliği kullanmayan test durumları tarafından veritabanına eklenen herhangi bir kayıt veritabanında hala mevcut olabilir.

Veritabanını tamamen sıfırlamak isterseniz, bunun yerine `Illuminate\Foundation\Testing\DatabaseMigrations` veya `Illuminate\Foundation\Testing\DatabaseTruncation` özelliklerini kullanabilirsiniz. Ancak, bu seçeneklerin her ikisi de `RefreshDatabase` özelliğinden önemli ölçüde daha yavaştır.

## Model Fabrikaları (Model Factories)

Test ederken, testinizi yürütmeden önce veritabanınıza birkaç kayıt eklemeniz gerekebilir. Bu test verisini oluştururken her sütunun değerini manuel olarak belirtmek yerine Laravel, model fabrikalarını (model factories) kullanarak Eloquent modellerinizin her biri için bir dizi varsayılan nitelik tanımlamanıza olanak tanır.

Model fabrikalarını oluşturma ve modeller oluşturmak için bunları kullanma hakkında daha fazla bilgi edinmek için lütfen eksiksiz model fabrikası dokümantasyonuna bakın. Bir model fabrikası tanımladıktan sonra, model oluşturmak için testiniz içinde fabrikayı kullanabilirsiniz:
```php
use App\Models\User;

test('modeller örneklendirilebilir', function () {
    $user = User::factory()->create();

    // ...
});
```
```php
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

## Tohumlayıcıları (Seeders) Çalıştırma

Bir özellik testi (feature test) sırasında veritabanınızı doldurmak için veritabanı tohumlayıcılarını (database seeders) kullanmak isterseniz, `seed` metodunu çağırabilirsiniz. Varsayılan olarak, `seed` metodu, diğer tüm tohumlayıcılarınızı çalıştırması gereken `DatabaseSeeder`'ı çalıştıracaktır. Alternatif olarak, `seed` metoduna belirli bir tohumlayıcı sınıf adı iletebilirsiniz:
```php
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('siparişler oluşturulabilir', function () {
    // DatabaseSeeder'ı çalıştır...
    $this->seed();

    // Belirli bir tohumlayıcıyı çalıştır...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // Belirli tohumlayıcıların bir dizisini çalıştır...
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
        // ...
    ]);
});
```
```php
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Yeni bir sipariş oluşturmayı test et.
     */
    public function test_orders_can_be_created(): void
    {
        // DatabaseSeeder'ı çalıştır...
        $this->seed();

        // Belirli bir tohumlayıcıyı çalıştır...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // Belirli tohumlayıcıların bir dizisini çalıştır...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```
Alternatif olarak, `RefreshDatabase` özelliğini kullanan her testten önce Laravel'e veritabanını otomatik olarak tohumlaması talimatını verebilirsiniz. Bunu, temel test sınıfınızda (base test class) bir `$seed` özelliği tanımlayarak gerçekleştirebilirsiniz:
```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    /**
     * Varsayılan tohumlayıcının her testten önce çalıştırılıp çalıştırılmayacağını belirtir.
     *
     * @var bool
     */
    protected $seed = true;
}
```
`$seed` özelliği `true` olduğunda, test, `RefreshDatabase` özelliğini kullanan her testten önce `Database\Seeders\DatabaseSeeder` sınıfını çalıştıracaktır. Ancak, test sınıfınızda bir `$seeder` özelliği tanımlayarak çalıştırılması gereken belirli bir tohumlayıcıyı belirtebilirsiniz:
```php
use Database\Seeders\OrderStatusSeeder;

/**
 * Her testten önce belirli bir tohumlayıcıyı çalıştır.
 *
 * @var string
 */
protected $seeder = OrderStatusSeeder::class;
```

## Mevcut İddialar (Available Assertions)

Laravel, Pest veya PHPUnit özellik testleriniz için birkaç veritabanı iddiası (database assertions) sağlar. Bu iddiaların her birini aşağıda tartışacağız.

#### `assertDatabaseCount`
Veritabanındaki bir tablonun verilen sayıda kayıt içerdiğini iddia eder:
```php
$this->assertDatabaseCount('users', 5);
```

#### `assertDatabaseEmpty`
Veritabanındaki bir tablonun hiç kayıt içermediğini iddia eder:
```php
$this->assertDatabaseEmpty('users');
```

#### `assertDatabaseHas`
Veritabanındaki bir tablonun, verilen anahtar/değer sorgu kısıtlamalarıyla eşleşen kayıtlar içerdiğini iddia eder:
```php
$this->assertDatabaseHas('users', [
    'email' => 'ali@example.com',
]);
```

#### `assertDatabaseMissing`
Veritabanındaki bir tablonun, verilen anahtar/değer sorgu kısıtlamalarıyla eşleşen kayıtlar içermediğini iddia eder:
```php
$this->assertDatabaseMissing('users', [
    'email' => 'ali@example.com',
]);
```

#### `assertSoftDeleted`
Belirli bir Eloquent modelinin "yazılı olarak silinmiş" (soft deleted) olduğunu iddia etmek için `assertSoftDeleted` metodu kullanılabilir:
```php
$this->assertSoftDeleted($user);
```

#### `assertNotSoftDeleted`
Belirli bir Eloquent modelinin "yazılı olarak silinmediğini" iddia etmek için `assertNotSoftDeleted` metodu kullanılabilir:
```php
$this->assertNotSoftDeleted($user);
```

#### `assertModelExists`
Belirli bir modelin veya model koleksiyonunun veritabanında mevcut olduğunu iddia eder:
```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

#### `assertModelMissing`
Belirli bir modelin veya model koleksiyonunun veritabanında mevcut olmadığını iddia eder:
```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

#### `expectsDatabaseQueryCount`
`expectsDatabaseQueryCount` metodu, testinizin başında, test sırasında çalıştırılmasını beklediğiniz toplam veritabanı sorgusu sayısını belirtmek için çağrılabilir. Yürütülen gerçek sorgu sayısı bu beklentiyle tam olarak eşleşmezse, test başarısız olacaktır:
```php
$this->expectsDatabaseQueryCount(5);

// Test...
```