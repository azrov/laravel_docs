# Laravel 12 Dokümantasyonu: Yükseltme Kılavuzu (Upgrade Guide)

## Yüksek Etkili Değişiklikler (High Impact Changes)
*(Bu bölümde, uygulamanızı önemli ölçüde etkileme olasılığı yüksek olan değişiklikler listelenir. Laravel 12 için belirtilmiş yüksek etkili bir değişiklik bulunmamaktadır.)*

## Orta Etkili Değişiklikler (Medium Impact Changes)
*(Bu bölümde, uygulamanızı orta düzeyde etkileme olasılığı olan değişiklikler listelenir. Laravel 12 için belirtilmiş orta etkili bir değişiklik bulunmamaktadır.)*

## Düşük Etkili Değişiklikler (Low Impact Changes)
*(Bu bölümde, uygulamanızı düşük düzeyde etkileme olasılığı olan veya belirli durumlarda geçerli değişiklikler listelenir.)*

## 11.x'ten 12.0'a Yükseltme

#### Tahmini Yükseltme Süresi: 5 Dakika

Olası her geriye dönük uyumsuz değişikliği (breaking change) belgelemeye çalışıyoruz. Bu değişikliklerden bazıları framework'ün az bilinen kısımlarında olduğu için, bunların yalnızca bir kısmı uygulamanızı gerçekten etkileyebilir. Zaman kazanmak ister misiniz? Uygulama yükseltmelerinizi otomatikleştirmeye yardımcı olması için Laravel Shift'i kullanabilirsiniz.

### Bağımlılıkları Güncelleme (Updating Dependencies)

**Etki Olasılığı: Yüksek**

Uygulamanızın `composer.json` dosyasında aşağıdaki bağımlılıkları güncellemelisiniz:

*   `laravel/framework` `^12.0` sürümüne
*   `phpunit/phpunit` `^11.0` sürümüne
*   `pestphp/pest` `^3.0` sürümüne

#### Carbon 3

**Etki Olasılığı: Düşük**

Carbon 2.x desteği kaldırılmıştır. Tüm Laravel 12 uygulamaları artık Carbon 3.x gerektirir.

### Laravel Kurucusunu (Installer) Güncelleme (Updating the Laravel Installer)

Yeni Laravel uygulamaları oluşturmak için Laravel kurucusu CLI aracını kullanıyorsanız, kurucu kurulumunuzu Laravel 12.x ve yeni Laravel başlangıç kitleriyle uyumlu olacak şekilde güncellemelisiniz. Laravel kurucusunu `composer global require` ile kurduysanız, `composer global update` kullanarak güncelleyebilirsiniz:
```bash
composer global update laravel/installer
```
PHP ve Laravel'i başlangıçta `php.new` aracılığıyla kurduysanız, işletim sisteminiz için `php.new` kurulum komutlarını yeniden çalıştırarak PHP'nin ve Laravel kurucusunun en son sürümünü kurabilirsiniz:
```bash
# macOS için...
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"

# Windows için (yönetici olarak çalıştırın)...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))

# Linux için...
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```
Veya, Laravel Herd ile birlikte gelen Laravel kurucusunun kopyasını kullanıyorsanız, Herd kurulumunuzu en son sürüme güncellemelisiniz.

### Kimlik Doğrulama (Authentication)

#### Güncellenmiş `DatabaseTokenRepository` Kurucu İmzası (Updated DatabaseTokenRepository Constructor Signature)

**Etki Olasılığı: Çok Düşük**

`Illuminate\Auth\Passwords\DatabaseTokenRepository` sınıfının kurucusu (constructor) artık `$expires` parametresinin dakika yerine saniye cinsinden verilmesini beklemektedir.

### Eşzamanlılık (Concurrency)

#### Eşzamanlılık Sonucu İndeks Eşlemesi (Concurrency Result Index Mapping)

**Etki Olasılığı: Düşük**

`Concurrency::run` metodu ilişkisel bir dizi (associative array) ile çağrıldığında, eşzamanlı işlemlerin sonuçları artık ilişkili anahtarlarıyla birlikte döndürülmektedir:
```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```

### Container (Container)

#### Container Sınıf Bağımlılık Çözümlemesi (Container Class Dependency Resolution)

**Etki Olasılığı: Düşük**

Bağımlılık enjeksiyon kabı (dependency injection container), artık bir sınıf örneğini (class instance) çözümlerken (resolving) sınıf özelliklerinin varsayılan değerine saygı duymaktadır. Daha önce varsayılan değer olmadan bir sınıf örneğini çözümlemek için container'a güveniyorsanız, bu yeni davranışı hesaba katmak için uygulamanızı ayarlamanız gerekebilir:
```php
class Example
{
    public function __construct(public ?Carbon $date = null) {}
}

$example = resolve(Example::class);

// <= 11.x
$example->date instanceof Carbon;

// >= 12.x
$example->date === null;
```

### Veritabanı (Database)

#### Çok Şemalı (Multi-Schema) Veritabanı İnceleme

**Etki Olasılığı: Düşük**

`Schema::getTables()`, `Schema::getViews()` ve `Schema::getTypes()` metotları artık varsayılan olarak tüm şemalardaki (schemas) sonuçları içerir. Sonucu yalnızca belirtilen şema için almak üzere `schema` argümanını iletebilirsiniz:
```php
// Tüm şemalardaki tüm tablolar...
$tables = Schema::getTables();

// 'main' şemasındaki tüm tablolar...
$tables = Schema::getTables(schema: 'main');

// 'main' ve 'blog' şemalarındaki tüm tablolar...
$tables = Schema::getTables(schema: ['main', 'blog']);
```
`Schema::getTableListing()` metodu artık varsayılan olarak şema nitelikli (schema-qualified) tablo adlarını döndürür. Davranışı istediğiniz gibi değiştirmek için `schemaQualified` argümanını iletebilirsiniz:
```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$tables = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$tables = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```
`db:table` ve `db:show` komutları artık PostgreSQL ve SQL Server'da olduğu gibi MySQL, MariaDB ve SQLite'da da tüm şemaların sonuçlarını çıktı olarak vermektedir.

#### Veritabanı Kurucu İmza Değişiklikleri (Database Constructor Signature Changes)

**Etki Olasılığı: Çok Düşük**

Laravel 12'de, birkaç düşük seviyeli veritabanı sınıfı artık kurucuları aracılığıyla bir `Illuminate\Database\Connection` örneği sağlanmasını gerektirmektedir.

Bu değişiklikler öncelikle veritabanı paketi bakımcılarını (maintainers) ilgilendirir - bu değişikliklerden herhangi birinin normal uygulama geliştirmeyi etkilemesi son derece düşük bir ihtimaldir.

*   **`Illuminate\Database\Schema\Blueprint`:** `Blueprint` sınıfının kurucusu artık ilk argümanı olarak bir `Connection` örneği beklemektedir. Bu, öncelikle `Blueprint` örneklerini manuel olarak oluşturan uygulamaları veya paketleri etkiler.
*   **`Illuminate\Database\Grammar`:** `Grammar` sınıfının kurucusu da artık bir `Connection` örneği gerektirir. Önceki sürümlerde, bağlantı, oluşturma sonrasında `setConnection()` metodu kullanılarak atanıyordu. Bu metod Laravel 12'de kaldırılmıştır:
    ```php
    // Laravel <= 11.x
    $grammar = new MySqlGrammar;
    $grammar->setConnection($connection);

    // Laravel >= 12.x
    $grammar = new MySqlGrammar($connection);
    ```
    Ek olarak, aşağıdaki API'ler kaldırılmış veya kullanımdan kaldırılmıştır (deprecated):
    *   `Blueprint::getPrefix()` metodu kullanımdan kaldırıldı.
    *   `Connection::withTablePrefix()` metodu kaldırıldı.
    *   `Grammar::getTablePrefix()` ve `setTablePrefix()` metotları kullanımdan kaldırıldı.
    *   `Grammar::setConnection()` metodu kaldırıldı.

Tablo ön ekleri (table prefixes) ile çalışırken, artık bunları doğrudan veritabanı bağlantısından almalısınız:
```php
$prefix = $connection->getTablePrefix();
```
Özel veritabanı sürücüleri, şema oluşturucular (schema builders) veya gramer gerçekleştirimleri (grammar implementations) sürdürüyorsanız, kurucularını gözden geçirmeli ve bir `Connection` örneği sağlandığından emin olmalısınız.

### Eloquent

#### Modeller ve UUIDv7

**Etki Olasılığı: Orta**

`HasUuids` özelliği (trait) artık UUID spesifikasyonunun 7. sürümüyle uyumlu (ordered UUIDs) UUID'ler döndürmektedir. Model ID'leriniz için sıralı (ordered) UUIDv4 dizeleri kullanmaya devam etmek istiyorsanız, artık `HasVersion4Uuids` özelliğini kullanmalısınız:
```php
use Illuminate\Database\Eloquent\Concerns\HasUuids; // Artık UUIDv7 döndürür
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids; // UUIDv4 için
```
`HasVersion7Uuids` özelliği kaldırılmıştır. Daha önce bu özelliği kullanıyorsanız, bunun yerine artık aynı davranışı sağlayan `HasUuids` özelliğini kullanmalısınız.

### İstekler (Requests)

#### İç İçe Dizi (Nested Array) İstek Birleştirme (Merging)

**Etki Olasılığı: Düşük**

`$request->mergeIfMissing()` metodu artık "nokta" notasyonunu ("dot" notation) kullanarak iç içe dizi verilerini birleştirmeye (merge) izin vermektedir. Daha önce bu metodun, anahtarın "nokta" notasyonu versiyonunu içeren bir üst düzey dizi anahtarı oluşturmasına güveniyorsanız, bu yeni davranışı hesaba katmak için uygulamanızı ayarlamanız gerekebilir:
```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell', // Artık user dizisi altında 'last_name' olarak birleşir
]);
```

### Yönlendirme (Routing)

#### Rota Önceliği (Route Precedence)

**Etki Olasılığı: Düşük**

Birden çok rotanın aynı ada sahip olduğu durumlarda yönlendirme davranışı, önbelleğe alınmış (cached) ve önbelleğe alınmamış (uncached) yönlendirme arasında birleştirilmiştir. Bu, önbelleğe alınmamış yönlendirmenin artık belirli bir ada sahip ilk kaydedilen rotayı, son kaydedilen yerine eşleştirdiği anlamına gelir.

### Depolama (Storage)

#### Yerel Dosya Sistemi Diski Varsayılan Kök Yolu (Local Filesystem Disk Default Root Path)

**Etki Olasılığı: Düşük**

Uygulamanız dosya sistemleri yapılandırmanızda açıkça bir `local` disk tanımlamıyorsa, Laravel artık `local` diskinin kökünü `storage/app/private` olarak varsayılan hale getirecektir. Önceki sürümlerde bu varsayılan `storage/app` idi. Sonuç olarak, `Storage::disk('local')` çağrıları, aksi yapılandırılmadıkça `storage/app/private` konumundan okuyacak ve bu konuma yazacaktır. Önceki davranışı geri yüklemek için `local` diskini manuel olarak tanımlayabilir ve istediğiniz kök yolunu ayarlayabilirsiniz.

### Doğrulama (Validation)

#### Görüntü (Image) Doğrulaması Artık SVG'leri Hariç Tutuyor

**Etki Olasılığı: Düşük**

`image` doğrulama kuralı artık varsayılan olarak SVG görüntülerine izin vermemektedir. `image` kuralını kullanırken SVG'lere izin vermek isterseniz, bunları açıkça izin vermelisiniz:
```php
use Illuminate\Validation\Rules\File;

'photo' => 'required|image:allow_svg'

// Veya...
'photo' => ['required', File::image(allowSvg: true)],
```

### Çeşitli (Miscellaneous)

Ayrıca `laravel/laravel` GitHub deposundaki değişiklikleri görüntülemenizi teşvik ediyoruz. Bu değişikliklerin çoğu gerekli olmasa da, bu dosyaları uygulamanızla senkronize tutmak isteyebilirsiniz. Bu değişikliklerden bazıları bu yükseltme kılavuzunda ele alınacaktır, ancak yapılandırma dosyalarındaki veya yorumlardaki değişiklikler gibi diğerleri ele alınmayacaktır. Değişiklikleri GitHub karşılaştırma aracıyla kolayca görüntüleyebilir ve sizin için hangi güncellemelerin önemli olduğunu seçebilirsiniz.