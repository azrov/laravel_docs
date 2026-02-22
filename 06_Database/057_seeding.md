# Laravel 12 Dokümantasyonu: Veritabanı: Tohumlama (Database: Seeding)

## Giriş

Laravel, veritabanınızı tohum (seed) sınıfları kullanarak veriyle doldurma yeteneği içerir. Tüm tohum sınıfları `database/seeders` dizininde saklanır. Varsayılan olarak, sizin için bir `DatabaseSeeder` sınıfı tanımlanmıştır. Bu sınıftan, diğer tohum sınıflarını çalıştırmak için `call` metodunu kullanabilir, böylece tohumlama sırasını kontrol edebilirsiniz.

Veritabanı tohumlaması sırasında toplu atama koruması (mass assignment protection) otomatik olarak devre dışı bırakılır.

## Tohumlayıcı (Seeder) Yazma

Bir tohumlayıcı oluşturmak için `make:seeder` Artisan komutunu çalıştırın. Framework tarafından oluşturulan tüm tohumlayıcılar `database/seeders` dizinine yerleştirilecektir:
```bash
php artisan make:seeder UserSeeder
```
Bir tohumlayıcı sınıfı varsayılan olarak yalnızca bir metot içerir: `run`. Bu metod, `db:seed` Artisan komutu çalıştırıldığında çağrılır. `run` metodu içinde, verileri istediğiniz şekilde veritabanınıza ekleyebilirsiniz. Verileri manuel olarak eklemek için sorgu oluşturucuyu (query builder) kullanabilir veya Eloquent model fabrikalarını (model factories) kullanabilirsiniz.

Örnek olarak, varsayılan `DatabaseSeeder` sınıfını değiştirelim ve `run` metoduna bir veritabanı ekleme (insert) ifadesi ekleyelim:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * Veritabanı tohumlayıcılarını çalıştır.
     */
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```
`run` metodunun imzasında ihtiyacınız olan herhangi bir bağımlılığı tip belirtebilirsiniz (type-hint). Bunlar Laravel servis kabı (service container) aracılığıyla otomatik olarak çözümlenecektir (resolved).

### Model Fabrikalarını Kullanma (Using Model Factories)
Elbette, her model tohumlaması için nitelikleri manuel olarak belirtmek zahmetlidir. Bunun yerine, büyük miktarlarda veritabanı kaydını kolayca oluşturmak için model fabrikalarını (model factories) kullanabilirsiniz. İlk olarak, fabrikalarınızı nasıl tanımlayacağınızı öğrenmek için model fabrikası dokümantasyonunu inceleyin.

Örneğin, her biriyle ilişkili bir gönderisi (post) olan 50 kullanıcı oluşturalım:
```php
use App\Models\User;

/**
 * Veritabanı tohumlayıcılarını çalıştır.
 */
public function run(): void
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

### Ek Tohumlayıcıları Çağırma (Calling Additional Seeders)
`DatabaseSeeder` sınıfı içinde, ek tohum sınıflarını çalıştırmak için `call` metodunu kullanabilirsiniz. `call` metodunu kullanmak, veritabanı tohumlamanızı birden çok dosyaya bölmenize olanak tanır, böylece tek bir tohumlayıcı sınıfı çok büyümez. `call` metodu, çalıştırılması gereken bir tohumlayıcı sınıfları dizisi kabul eder:
```php
/**
 * Veritabanı tohumlayıcılarını çalıştır.
 */
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

### Model Olaylarını (Events) Susturma (Muting Model Events)
Tohumlama çalıştırırken, modellerin olay göndermesini (dispatching events) engellemek isteyebilirsiniz. Bunu `WithoutModelEvents` özelliğini (trait) kullanarak başarabilirsiniz. Kullanıldığında, `WithoutModelEvents` özelliği, `call` metodu aracılığıyla ek tohum sınıfları çalıştırılsa bile hiçbir model olayının gönderilmemesini sağlar:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    /**
     * Veritabanı tohumlayıcılarını çalıştır.
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

## Tohumlayıcıları Çalıştırma (Running Seeders)

Veritabanınızı tohumlamak için `db:seed` Artisan komutunu çalıştırabilirsiniz. Varsayılan olarak, `db:seed` komutu `Database\Seeders\DatabaseSeeder` sınıfını çalıştırır ve bu sınıf da sırayla diğer tohum sınıflarını çağırabilir. Ancak, tek tek çalıştırmak için belirli bir tohumlayıcı sınıfı belirtmek üzere `--class` seçeneğini kullanabilirsiniz:
```bash
php artisan db:seed

php artisan db:seed --class=UserSeeder
```
Ayrıca, `migrate:fresh` komutunu `--seed` seçeneğiyle birlikte kullanarak veritabanınızı tohumlayabilirsiniz; bu, tüm tabloları silecek ve tüm migration'larınızı yeniden çalıştıracaktır. Bu komut, veritabanınızı tamamen yeniden oluşturmak için kullanışlıdır. `--seeder` seçeneği, çalıştırılacak belirli bir tohumlayıcıyı belirtmek için kullanılabilir:
```bash
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

#### Tohumlayıcıları Üretimde (Production) Çalıştırmaya Zorlama (Forcing Seeders to Run in Production)
Bazı tohumlama işlemleri verilerinizi değiştirmenize veya kaybetmenize neden olabilir. Sizi tohumlama komutlarını üretim veritabanınıza karşı çalıştırmaktan korumak için, `production` ortamında tohumlayıcılar çalıştırılmadan önce onay istenecektir. Tohumlayıcıları onay almadan çalışmaya zorlamak için `--force` bayrağını kullanın:
```bash
php artisan db:seed --force
```