# Laravel 12 Dokümantasyonu: Veritabanı: Migration'lar (Database: Migrations)

## Giriş

Migration'lar (göçler), veritabanınız için sürüm kontrolü (version control) gibidir ve ekibinizin uygulamanın veritabanı şema tanımını (database schema definition) tanımlamasına ve paylaşmasına olanak tanır. Daha önce bir takım arkadaşınıza, kaynak kontrolünden (source control) değişikliklerinizi çektikten sonra yerel veritabanı şemalarına manuel olarak bir sütun eklemesini söylemek zorunda kaldıysanız, veritabanı migration'larının çözdüğü sorunla karşılaşmışsınız demektir.

Laravel'in `Schema` facade'ı, Laravel'in desteklediği tüm veritabanı sistemlerinde tablolar oluşturmak ve değiştirmek için veritabanından bağımsız (database agnostic) destek sağlar. Tipik olarak, migration'lar veritabanı tablolarını ve sütunlarını oluşturmak ve değiştirmek için bu facade'ı kullanacaktır.

## Migration Oluşturma (Generating Migrations)

Bir veritabanı migration'ı oluşturmak için `make:migration` Artisan komutunu kullanabilirsiniz. Yeni migration, `database/migrations` dizininize yerleştirilecektir. Her migration dosya adı, Laravel'in migration'ların sırasını belirlemesine olanak tanıyan bir zaman damgası (timestamp) içerir:
```bash
php artisan make:migration create_flights_table
```
Laravel, migration adını kullanarak tablo adını ve migration'ın yeni bir tablo oluşturup oluşturmayacağını tahmin etmeye çalışacaktır. Laravel, migration adından tablo adını belirleyebilirse, oluşturulan migration dosyasını belirtilen tabloyla önceden dolduracaktır (pre-fill). Aksi takdirde, tabloyu migration dosyasında manuel olarak belirtebilirsiniz.

Oluşturulan migration için özel bir yol (custom path) belirtmek isterseniz, `make:migration` komutunu çalıştırırken `--path` seçeneğini kullanabilirsiniz. Verilen yol, uygulamanızın temel yoluna (base path) göreli (relative) olmalıdır.

Migration iskeletleri (stubs), iskelet yayınlama (stub publishing) kullanılarak özelleştirilebilir.

### Migration'ları Sıkıştırma (Squashing Migrations)
Uygulamanızı geliştirdikçe, zamanla giderek daha fazla migration biriktirebilirsiniz. Bu, `database/migrations` dizininizin potansiyel olarak yüzlerce migration ile şişmesine neden olabilir. İsterseniz, migration'larınızı tek bir SQL dosyasında "sıkıştırabilirsiniz" (squash). Başlamak için `schema:dump` komutunu çalıştırın:
```bash
php artisan schema:dump

# Mevcut veritabanı şemasını dök ve tüm mevcut migration'ları temizle...
php artisan schema:dump --prune
```
Bu komutu çalıştırdığınızda, Laravel uygulamanızın `database/schema` dizinine bir "schema" dosyası yazacaktır. Şema dosyasının adı, veritabanı bağlantısına (database connection) karşılık gelecektir. Artık, veritabanınızı migrate etmeye çalıştığınızda ve başka hiçbir migration yürütülmemişse, Laravel önce kullanmakta olduğunuz veritabanı bağlantısının şema dosyasındaki SQL ifadelerini yürütecektir. Şema dosyasının SQL ifadelerini yürüttükten sonra Laravel, şema dökümünün (schema dump) parçası olmayan kalan tüm migration'ları yürütecektir.

Uygulamanızın testleri, yerel geliştirme sırasında tipik olarak kullandığınızdan farklı bir veritabanı bağlantısı kullanıyorsa, testlerinizin veritabanınızı oluşturabilmesi için bu veritabanı bağlantısını kullanarak bir şema dosyası döktüğünüzden emin olmalısınız. Bunu, yerel geliştirme sırasında tipik olarak kullandığınız veritabanı bağlantısını döktükten sonra yapmak isteyebilirsiniz:
```bash
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```
Veritabanı şema dosyanızı kaynak kontrole (source control) eklemelisiniz, böylece ekibinizdeki diğer yeni geliştiriciler uygulamanızın ilk veritabanı yapısını hızlıca oluşturabilir.

Migration sıkıştırma yalnızca MariaDB, MySQL, PostgreSQL ve SQLite veritabanları için kullanılabilir ve veritabanının komut satırı istemcisini (command-line client) kullanır.

## Migration Yapısı (Migration Structure)

Bir migration sınıfı iki metot içerir: `up` ve `down`. `up` metodu, veritabanınıza yeni tablolar, sütunlar veya indeksler eklemek için kullanılırken, `down` metodu `up` metodu tarafından gerçekleştirilen işlemleri tersine çevirmelidir (reverse).

Bu metotların her ikisinde de, tabloları anlamlı bir şekilde oluşturmak ve değiştirmek için Laravel şema oluşturucusunu (schema builder) kullanabilirsiniz. `Schema` oluşturucusunda bulunan tüm metotlar hakkında bilgi edinmek için dokümantasyonuna göz atın. Örneğin, aşağıdaki migration bir `flights` tablosu oluşturur:
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Migration'ı çalıştır.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Migration'ı geri al.
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

#### Migration Bağlantısını Ayarlama (Setting the Migration Connection)
Migration'ınız, uygulamanızın varsayılan veritabanı bağlantısı dışında bir veritabanı bağlantısıyla etkileşime girecekse, migration'ınızın `$connection` özelliğini ayarlamalısınız:
```php
/**
 * Migration tarafından kullanılması gereken veritabanı bağlantısı.
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * Migration'ı çalıştır.
 */
public function up(): void
{
    // ...
}
```

#### Migration'ları Atlama (Skipping Migrations)
Bazen bir migration, henüz aktif olmayan bir özelliği desteklemek için tasarlanmış olabilir ve henüz çalıştırılmasını istemeyebilirsiniz. Bu durumda, migration üzerinde bir `shouldRun` metodu tanımlayabilirsiniz. `shouldRun` metodu `false` döndürürse, migration atlanacaktır:
```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * Bu migration'ın çalıştırılıp çalıştırılmayacağını belirle.
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

## Migration'ları Çalıştırma (Running Migrations)

Bekleyen tüm migration'larınızı çalıştırmak için `migrate` Artisan komutunu çalıştırın:
```bash
php artisan migrate
```
Hangi migration'ların zaten çalıştığını ve hangilerinin hala beklemede olduğunu görmek isterseniz, `migrate:status` Artisan komutunu kullanabilirsiniz:
```bash
php artisan migrate:status
```
Migration'lar tarafından çalıştırılacak SQL ifadelerini, onları fiilen çalıştırmadan görmek isterseniz, `migrate` komutuna `--pretend` bayrağını sağlayabilirsiniz:
```bash
php artisan migrate --pretend
```

#### Migration Yürütmeyi İzole Etme (Isolating Migration Execution)
Uygulamanızı birden çok sunucuya dağıtıyorsanız ve migration'ları dağıtım sürecinizin bir parçası olarak çalıştırıyorsanız, muhtemelen iki sunucunun aynı anda veritabanını migrate etmeye çalışmasını istemezsiniz. Bunu önlemek için, `migrate` komutunu çağırırken `isolated` seçeneğini kullanabilirsiniz.

`isolated` seçeneği sağlandığında, Laravel migration'larınızı çalıştırmaya çalışmadan önce uygulamanızın önbellek sürücüsünü (cache driver) kullanarak atomik bir kilit (atomic lock) edinecektir. Bu kilit tutulurken `migrate` komutunu çalıştırmaya yönelik diğer tüm girişimler yürütülmeyecektir; ancak, komut yine de başarılı bir çıkış durum koduyla (exit status code) sona erecektir:
```bash
php artisan migrate --isolated
```
Bu özelliği kullanmak için uygulamanızın varsayılan önbellek sürücüsü olarak `memcached`, `redis`, `dynamodb`, `database`, `file` veya `array` önbellek sürücüsünü kullanıyor olması gerekir. Ayrıca, tüm sunucular aynı merkezi önbellek sunucusuyla iletişim kuruyor olmalıdır.

#### Migration'ları Üretimde (Production) Çalıştırmaya Zorlama (Forcing Migrations to Run in Production)
Bazı migration işlemleri yıkıcıdır (destructive), yani veri kaybına neden olabilirler. Sizi bu komutları üretim veritabanınıza karşı çalıştırmaktan korumak için, komutlar çalıştırılmadan önce onay istenecektir. Komutları onay almadan çalışmaya zorlamak için `--force` bayrağını kullanın:
```bash
php artisan migrate --force
```

### Migration'ları Geri Alma (Rolling Back Migrations)
En son migration işlemini geri almak için `rollback` Artisan komutunu kullanabilirsiniz. Bu komut, birden çok migration dosyası içerebilecek en son "toplu" (batch) migration'ı geri alır:
```bash
php artisan migrate:rollback
```
`rollback` komutuna `step` seçeneğini sağlayarak sınırlı sayıda migration'ı geri alabilirsiniz. Örneğin, aşağıdaki komut son beş migration'ı geri alacaktır:
```bash
php artisan migrate:rollback --step=5
```
`rollback` komutuna `batch` seçeneğini sağlayarak belirli bir "toplu" migration'ı geri alabilirsiniz; burada `batch` seçeneği, uygulamanızın migration'lar veritabanı tablosundaki bir `batch` değerine karşılık gelir. Örneğin, aşağıdaki komut üç numaralı toplu işteki tüm migration'ları geri alacaktır:
```bash
php artisan migrate:rollback --batch=3
```
Migration'lar tarafından çalıştırılacak SQL ifadelerini, onları fiilen çalıştırmadan görmek isterseniz, `migrate:rollback` komutuna `--pretend` bayrağını sağlayabilirsiniz:
```bash
php artisan migrate:rollback --pretend
```
`migrate:reset` komutu, uygulamanızın tüm migration'larını geri alacaktır:
```bash
php artisan migrate:reset
```

#### Tek Bir Komutla Geri Alma ve Migration'ları Çalıştırma (Roll Back and Migrate Using a Single Command)
`migrate:refresh` komutu, tüm migration'larınızı geri alacak ve ardından `migrate` komutunu çalıştıracaktır. Bu komut, tüm veritabanınızı etkili bir şekilde yeniden oluşturur:
```bash
php artisan migrate:refresh

# Veritabanını yenile ve tüm veritabanı tohumlarını (seeds) çalıştır...
php artisan migrate:refresh --seed
```
`refresh` komutuna `step` seçeneğini sağlayarak sınırlı sayıda migration'ı geri alabilir ve yeniden migrate edebilirsiniz. Örneğin, aşağıdaki komut son beş migration'ı geri alacak ve yeniden migrate edecektir:
```bash
php artisan migrate:refresh --step=5
```

#### Tüm Tabloları Sil ve Migration'ları Çalıştır (Drop All Tables and Migrate)
`migrate:fresh` komutu, veritabanındaki tüm tabloları silecek ve ardından `migrate` komutunu çalıştıracaktır:
```bash
php artisan migrate:fresh

php artisan migrate:fresh --seed
```
Varsayılan olarak, `migrate:fresh` komutu yalnızca varsayılan veritabanı bağlantısındaki tabloları siler. Ancak, migration'ların yapılması gereken veritabanı bağlantısını belirtmek için `--database` seçeneğini kullanabilirsiniz. Veritabanı bağlantı adı, uygulamanızın veritabanı yapılandırma dosyasında tanımlanan bir bağlantıya karşılık gelmelidir:
```bash
php artisan migrate:fresh --database=admin
```
`migrate:fresh` komutu, ön eklerine (prefix) bakılmaksızın tüm veritabanı tablolarını silecektir. Bu komut, diğer uygulamalarla paylaşılan bir veritabanında geliştirme yaparken dikkatli kullanılmalıdır.

## Tablolar (Tables)

### Tablo Oluşturma (Creating Tables)
Yeni bir veritabanı tablosu oluşturmak için `Schema` facade'ında `create` metodunu kullanın. `create` metodu iki argüman kabul eder: ilki tablonun adı, ikincisi ise yeni tabloyu tanımlamak için kullanılabilecek bir `Blueprint` nesnesi alan bir closure'dır:
```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```
Tabloyu oluştururken, tablonun sütunlarını tanımlamak için şema oluşturucunun (schema builder) sütun metotlarından herhangi birini kullanabilirsiniz.

#### Tablo / Sütun Varlığını Belirleme (Determining Table / Column Existence)
`hasTable`, `hasColumn` ve `hasIndex` metotlarını kullanarak bir tablonun, sütunun veya indeksin varlığını belirleyebilirsiniz:
```php
if (Schema::hasTable('users')) {
    // "users" tablosu mevcut...
}

if (Schema::hasColumn('users', 'email')) {
    // "users" tablosu mevcut ve bir "email" sütunu var...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // "users" tablosu mevcut ve "email" sütununda benzersiz bir indeks var...
}
```

#### Veritabanı Bağlantısı ve Tablo Seçenekleri (Database Connection and Table Options)
Varsayılan bağlantınız olmayan bir veritabanı bağlantısında şema işlemi yapmak isterseniz, `connection` metodunu kullanın:
```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```
Ek olarak, tablo oluşturmanın diğer yönlerini tanımlamak için birkaç başka özellik ve metot kullanılabilir. `engine` özelliği, MariaDB veya MySQL kullanırken tablonun depolama motorunu (storage engine) belirtmek için kullanılabilir:
```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```
`charset` ve `collation` özellikleri, MariaDB veya MySQL kullanırken oluşturulan tablo için karakter setini ve harmanlamayı (collation) belirtmek için kullanılabilir:
```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```
`temporary` metodu, tablonun "geçici" (temporary) olması gerektiğini belirtmek için kullanılabilir. Geçici tablolar yalnızca geçerli bağlantının veritabanı oturumu tarafından görülebilir ve bağlantı kapatıldığında otomatik olarak silinir:
```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```
Bir veritabanı tablosuna "yorum" (comment) eklemek isterseniz, tablo örneğinde `comment` metodunu çağırabilirsiniz. Tablo yorumları şu anda yalnızca MariaDB, MySQL ve PostgreSQL tarafından desteklenmektedir:
```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

### Tabloları Güncelleme (Updating Tables)
`Schema` facade'ındaki `table` metodu, mevcut tabloları güncellemek için kullanılabilir. `create` metodu gibi, `table` metodu da iki argüman kabul eder: tablonun adı ve tabloya sütunlar veya indeksler eklemek için kullanabileceğiniz bir `Blueprint` örneği alan bir closure:
```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

### Tabloları Yeniden Adlandırma / Silme (Renaming / Dropping Tables)
Mevcut bir veritabanı tablosunu yeniden adlandırmak için `rename` metodunu kullanın:
```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```
Mevcut bir tabloyu silmek için `drop` veya `dropIfExists` metotlarını kullanabilirsiniz:
```php
Schema::drop('users');

Schema::dropIfExists('users');
```

#### Yabancı Anahtarlı (Foreign Key) Tabloları Yeniden Adlandırma (Renaming Tables With Foreign Keys)
Bir tabloyu yeniden adlandırmadan önce, tablodaki herhangi bir yabancı anahtar kısıtlamasının (foreign key constraints), Laravel'in kurala dayalı bir ad atamasına izin vermek yerine migration dosyalarınızda açık bir ada sahip olduğunu doğrulamalısınız. Aksi takdirde, yabancı anahtar kısıtlama adı eski tablo adına başvuracaktır.

## Sütunlar (Columns)

### Sütun Oluşturma (Creating Columns)
`Schema` facade'ındaki `table` metodu, mevcut tabloları güncellemek için kullanılabilir. `create` metodu gibi, `table` metodu da iki argüman kabul eder: tablonun adı ve tabloya sütunlar eklemek için kullanabileceğiniz bir `Illuminate\Database\Schema\Blueprint` örneği alan bir closure:
```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

### Mevcut Sütun Tipleri (Available Column Types)
Şema oluşturucu blueprint (blueprint), farklı sütun türlerine karşılık gelen çeşitli metotlar sunar. Bu metotların listesi oldukça uzun olduğu için, en yaygın olanlardan bazılarını burada belirtmek gerekirse: `bigIncrements`, `bigInteger`, `binary`, `boolean`, `char`, `dateTimeTz`, `dateTime`, `date`, `decimal`, `double`, `enum`, `float`, `foreignId`, `foreignIdFor`, `foreignUlid`, `foreignUuid`, `geography`, `geometry`, `id`, `integer`, `ipAddress`, `json`, `jsonb`, `longText`, `macAddress`, `mediumIncrements`, `mediumInteger`, `mediumText`, `morphs`, `nullableMorphs`, `nullableTimestamps`, `nullableUlidMorphs`, `nullableUuidMorphs`, `rememberToken`, `set`, `smallIncrements`, `smallInteger`, `softDeletesTz`, `softDeletes`, `string`, `text`, `timeTz`, `time`, `timestampTz`, `timestamp`, `timestampsTz`, `timestamps`, `tinyIncrements`, `tinyInteger`, `tinyText`, `unsignedBigInteger`, `unsignedInteger`, `unsignedMediumInteger`, `unsignedSmallInteger`, `unsignedTinyInteger`, `ulidMorphs`, `uuidMorphs`, `ulid`, `uuid`, `year`.